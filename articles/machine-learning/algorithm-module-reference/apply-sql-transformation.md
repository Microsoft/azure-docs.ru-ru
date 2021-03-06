---
title: Применение преобразования SQL
titleSuffix: Azure Machine Learning
description: Узнайте, как использовать модуль «применение преобразования SQL» в Машинное обучение Azure для выполнения запроса SQLite к входным наборам данных, чтобы преобразовать данные.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 11/12/2020
ms.openlocfilehash: c66fbe59fd5b2660d02bfca285f78666d64569fe
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94555606"
---
# <a name="apply-sql-transformation"></a>Применение преобразования SQL

В этой статье описывается модуль конструктора Машинное обучение Azure.

С помощью модуля Apply SQL преобразование можно:
  
-   создать таблицу для результатов, а затем сохранить наборы данных в переносимой базе данных;  
  
-   выполнять пользовательские преобразования типов данных или создавать статистические выражения;  
  
-   выполнять инструкции SQL-запроса для фильтрации или изменения данных и получения результатов запроса в виде таблицы данных.  

> [!IMPORTANT]
> Ядро SQL, используемое в этом модуле, — **SQLite**. Дополнительные сведения о синтаксисе SQLite см. в разделе [SQL как понятное для SQLite](https://www.sqlite.org/index.html).
> Этот модуль будет выполнять данные SQLite, который находится в базе данных памяти, поэтому выполнение модуля требует значительно большего объема памяти и может привести к `Out of memory` ошибке. Убедитесь, что на компьютере достаточно ОЗУ.

## <a name="how-to-configure-apply-sql-transformation"></a>Настройка преобразования «применение SQL»  

В модуле может содержаться до трех наборов в качестве входных данных. При ссылке на наборы данных, подключенные к каждому порту ввода, необходимо использовать имена `t1` , `t2` и `t3` . Номер таблицы соответствует индексу входного порта.  

Ниже приведен пример кода, демонстрирующий соединение двух таблиц. T1 и T2 — это два набора данных, подключенных к левому и среднему входным портам для **применения преобразования SQL**.

```sql
SELECT t1.*
    , t3.Average_Rating
FROM t1 join
    (SELECT placeID
        , AVG(rating) AS Average_Rating
    FROM t2
    GROUP BY placeID
    ) as t3
on t1.placeID = t3.placeID
```
  
Оставшийся параметр — SQL-запрос, в котором используется синтаксис SQLite. При вводе нескольких строк в текстовом поле **скрипт SQL** используйте точку с запятой для завершения каждой инструкции. В противном случае разрывы строки преобразуются в пробелы.  

Этот модуль поддерживает все стандартные операторы синтаксиса SQLite. Список неподдерживаемых инструкций см. в разделе [Технические примечания](#technical-notes) .

##  <a name="technical-notes"></a>Технические примечания  

В этом разделе содержатся сведения о реализации, советы и ответы на часто задаваемые вопросы.

-   Входные данные всегда требуются для порта 1.  
  
-   Для идентификаторов столбцов, содержащих пробелы или другие специальные символы, всегда заключите идентификатор столбца в квадратные или двойные кавычки при ссылке на столбец в `SELECT` `WHERE` предложениях или.  
  
### <a name="unsupported-statements"></a>Неподдерживаемые инструкции  

Хотя SQLite поддерживает большую часть стандарта ANSI SQL, в него не включены несколько функций, поддерживаемых коммерческими системами реляционных баз данных. Дополнительные сведения см. в разделе [SQL как понятное для SQLite](http://www.sqlite.org/lang.html). Кроме того, при создании инструкций SQL учитывайте следующие ограничения.  
  
- SQLite использует динамическую типизацию значений, а не присвоение типа столбцу, как в большинстве систем реляционных баз данных. Кроме того, для нее характерна слабая типизация. Система также позволяет выполнять неявное преобразование типов.  
  
- `LEFT OUTER JOIN` реализован, но не `RIGHT OUTER JOIN` или `FULL OUTER JOIN` .  

- Можно использовать инструкции `RENAME TABLE` и `ADD COLUMN` с командой `ALTER TABLE`, но другие предложения не поддерживаются, в том числе `DROP COLUMN`, `ALTER COLUMN` и `ADD CONSTRAINT`.  
  
- Вы можете создать VIEW в SQLite, но последующие представления будут доступны только для чтения. Невозможно выполнить инструкцию `DELETE`, `INSERT` или `UPDATE` для представления. Но вы можете создать триггер, срабатывающий при попытке выполнить инструкцию `DELETE`, `INSERT` или `UPDATE` для представления, и выполнить другие операции в теле триггера.  
  

Помимо списка неподдерживаемых функций, предоставляемых на официальном веб-сайте SQLite, следующий вики-сайт предоставляет список других неподдерживаемых функций: [SQLite — неподдерживаемый SQL](http://www2.sqlite.org/cvstrac/wiki?p=UnsupportedSql)  
    
## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [набором доступных модулей](module-reference.md) в службе Машинного обучения Azure. 
