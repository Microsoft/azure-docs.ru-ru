---
title: Устранение неполадок с производительностью запросов в службе "База данных Azure для MariaDB"
description: Узнайте, как использовать инструкцию EXPLAIN для устранения неполадок с производительностью запросов в Базе данных Azure для MariaDB.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: troubleshooting
ms.date: 3/18/2020
ms.openlocfilehash: 2b7491723ffcff73e4b243fe54ef18608167d636
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98664721"
---
# <a name="how-to-use-explain-to-profile-query-performance-in-azure-database-for-mariadb"></a>Использование инструкции EXPLAIN для профилирования производительности запросов в Базе данных Azure для MariaDB
Инструкция **EXPLAIN** представляет собой удобное средство для оптимизации запросов. С помощью инструкций EXPLAIN можно получать информацию о том, как выполняются инструкции SQL. Ниже показан пример выполнения инструкции EXPLAIN.

```sql
mysql> EXPLAIN SELECT * FROM tb1 WHERE id=100\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 995789
     filtered: 10.00
        Extra: Using where
```

Как видно из этого примера, значение *key* (ключ) имеет значение NULL. Этот результат означает, что MariaDB не может найти индексы, оптимизированные для запроса, и выполняет полное сканирование таблицы. Оптимизируем этот запрос, добавив индекс в столбце **ID** (идентификатор).

```sql
mysql> ALTER TABLE tb1 ADD KEY (id);
mysql> EXPLAIN SELECT * FROM tb1 WHERE id=100\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ref
possible_keys: id
          key: id
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

Новая инструкция EXPLAIN показывает, что MariaDB теперь использует индекс для ограничения числа строк со значением 1, что значительно сокращает время поиска.
 
## <a name="covering-index"></a>Индекс покрытия
Индекс покрытия состоит из всех столбцов запроса в индексе для сокращения извлечения значений из таблиц данных. Ниже приведена иллюстрация следующей инструкции **GROUP BY**.
 
```sql
mysql> EXPLAIN SELECT MAX(c1), c2 FROM tb1 WHERE c2 LIKE '%100' GROUP BY c1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 995789
     filtered: 11.11
        Extra: Using where; Using temporary; Using filesort
```

Как видно из выходных данных, MariaDB не использует индексы, так как соответствующие индексы недоступны. Кроме того, отображается строка *Using temporary; Using file sort* (Использование временной таблицы; использование сортировки файлов), означающая, что MariaDB создает временную таблицу для соответствия предложению **GROUP BY**.
 
Создание индекса по столбцу **c2** ничего не меняет, и для MariaDB по-прежнему требуется создать временную таблицу:

```sql 
mysql> ALTER TABLE tb1 ADD KEY (c2);
mysql> EXPLAIN SELECT MAX(c1), c2 FROM tb1 WHERE c2 LIKE '%100' GROUP BY c1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 995789
     filtered: 11.11
        Extra: Using where; Using temporary; Using filesort
```

В этом случае можно создать **охватываемый индекс** по обоим столбцам **c1** и **c2**, добавив при этом значение столбца **c2** непосредственно в индекс для исключения дальнейшего поиска данных.

```sql 
mysql> ALTER TABLE tb1 ADD KEY covered(c1,c2);
mysql> EXPLAIN SELECT MAX(c1), c2 FROM tb1 WHERE c2 LIKE '%100' GROUP BY c1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: index
possible_keys: covered
          key: covered
      key_len: 108
          ref: NULL
         rows: 995789
     filtered: 11.11
        Extra: Using where; Using index
```

Как показано в инструкции EXPLAIN выше, теперь MariaDB использует охватываемый индекс и не создает временную таблицу. 

## <a name="combined-index"></a>Составной индекс
Составной индекс состоит из значений нескольких столбцов и может считаться массивом строк, отсортированных путем объединения значений индексированных столбцов.  Этот метод может быть полезен в инструкции **GROUP BY**.

```sql
mysql> EXPLAIN SELECT c1, c2 from tb1 WHERE c2 LIKE '%100' ORDER BY c1 DESC LIMIT 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 995789
     filtered: 11.11
        Extra: Using where; Using filesort
```

MariaDB выполняет операцию *сортировки файла* довольно медленно, особенно когда требуется отсортировать много строк. Для оптимизации этого запроса можно создать составной индекс по обоим столбцам, которые сортируются.

```sql 
mysql> ALTER TABLE tb1 ADD KEY my_sort2 (c1, c2);
mysql> EXPLAIN SELECT c1, c2 from tb1 WHERE c2 LIKE '%100' ORDER BY c1 DESC LIMIT 10\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb1
   partitions: NULL
         type: index
possible_keys: NULL
          key: my_sort2
      key_len: 108
          ref: NULL
         rows: 10
     filtered: 11.11
        Extra: Using where; Using index
```

Инструкция EXPLAIN теперь показывает, что MariaDB с помощью составного индекса может не выполнять дополнительную сортировку, так как индекс уже отсортирован.
 
## <a name="conclusion"></a>Заключение
 
Использование инструкции EXPLAIN и разных типов индексов позволяет существенно повысить производительность. Наличие индекса для таблицы не обязательно означает, что MariaDB сможет использовать его в запросах. Следует всегда проверять свои предположения с помощью инструкции EXPLAIN и оптимизировать запросы с помощью индексов.

## <a name="next-steps"></a>Дальнейшие действия
- Найти ответы на важные для вас вопросы, а также задать вопрос или ответить на него можно на [странице вопросов и ответов на сайте Майкрософт](/answers/topics/azure-database-mariadb.html) или на [сайте Stack Overflow](https://stackoverflow.com/questions/tagged/azure-database-mariadb).