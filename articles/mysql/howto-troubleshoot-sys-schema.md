---
title: Использование sys_schema — База данных Azure для MySQL
description: Узнайте, как использовать средство sys_schema для поиска проблем с производительностью и обслуживания базы данных в службе "База данных Azure для MySQL".
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: troubleshooting
ms.date: 3/30/2020
ms.openlocfilehash: a20510ee2800a54f9a51a2f498ee8ae8a3e51d55
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94543155"
---
# <a name="how-to-use-sys_schema-for-performance-tuning-and-database-maintenance-in-azure-database-for-mysql"></a>Как использовать sys_schema для настройки производительности и обслуживания базы данных в службе "База данных Azure для MySQL"

Схема performance_schema MySQL, которая стала впервые доступна в MySQL 5.5, предоставляет способ инструментирования многих важных ресурсов серверов, таких как выделение памяти, хранимые программы, блокировка метаданных и т. д. Однако performance_schema содержит более 80 таблиц. Для получения необходимой информации часто требуется объединение таблиц в performance_schema, а также таблицы из information_schema. Созданная на основе performance_schema и information_schema схема sys_schema предоставляет большую коллекцию [понятных представлений](https://dev.mysql.com/doc/refman/5.7/en/sys-schema-views.html) в базе данных только для чтения и полностью поддерживается в службе "База данных Azure для MySQL" версии 5.7.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/sys-schema-views.png" alt-text="Представления sys_schema":::

В sys_schema есть 52 представления, каждое из которых имеет один из следующих префиксов:

- Host_summary или операции ввода-вывода: задержки, связанные с операциями ввода-вывода.
- InnoDB: состояние и блокировки буфера InnoDB.
- Память: использование памяти узлом и пользователями.
- Схема. информация, связанная со схемой, такая как автоматическое увеличение, индексы и т. д.
- Statement: информация об инструкциях SQL. Это может быть инструкция, выполнение которой привело к полному сканированию таблицы или длительному времени запроса.
- User: Потребляемые ресурсы, сгруппированные по пользователям. Примерами являются операции ввода-вывода файлов, подключения и память.
- Wait: события ожидания, сгруппированные по узлу или пользователю.

Теперь рассмотрим некоторые общие шаблоны использования sys_schema. Для начала сгруппируем шаблоны использования в две категории: **Настройка производительности** и **Обслуживание баз данных**.

## <a name="performance-tuning"></a>Настройка производительности

### <a name="sysuser_summary_by_file_io"></a>*sys.user_summary_by_file_io*

Операции ввода-вывода являются наиболее ресурсоемкими операциями в базе данных. Мы можем узнать среднюю задержку операций ввода-вывода, запросив представление *sys.user_summary_by_file_io*. При использовании подготовленного хранилища по умолчанию размером 125 ГБ задержка ввода-вывода составляет около 15 секунд.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/io-latency-125GB.png" alt-text="Задержка ввода-вывода: 125 ГБ":::

Так как База данных Azure для MySQL масштабирует операции ввода-вывода в соответствии с хранилищем, после увеличения объема подготовленного хранилища до 1 ТБ задержка операций ввода-вывода уменьшается до 571 мс.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/io-latency-1TB.png" alt-text="Задержка ввода-вывода: 1 ТБ":::

### <a name="sysschema_tables_with_full_table_scans"></a>*sys.schema_tables_with_full_table_scans*

Несмотря на тщательное планирование, многие запросы могут по-прежнему привести к сканированию всей таблицы. Дополнительные сведения о типах индексов и способах их оптимизации см. в следующей статье: [Решение проблем с производительностью запросов](./howto-troubleshoot-query-performance.md). Полное сканирование таблиц является ресурсоемким и снижает производительность вашей базы данных. Самый быстрый способ поиска таблиц с полным сканированием — запросить представление *sys.schema_tables_with_full_table_scans*.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/full-table-scans.png" alt-text="Сканирование всей таблицы":::

### <a name="sysuser_summary_by_statement_type"></a>*sys.user_summary_by_statement_type*

Чтобы устранить проблемы с производительностью базы данных, может быть полезно просто выявить события, происходящие внутри базы данных, а представление *sys.user_summary_by_statement_type* выведет необходимые сведения по типу инструкций.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/summary-by-statement.png" alt-text="Сводка по инструкциям":::

В этом примере База данных Azure для MySQL потратила 53 минуты на сканирование журнала подробных поисковых запросов 44 579 раз. Это занимает много времени и требует множества операций ввода-вывода. Вы можете уменьшить эту активность, отключив журнал медленных запросов либо уменьшив частоту внесения в него записей на портале Azure.

## <a name="database-maintenance"></a>Обслуживание базы данных

### <a name="sysinnodb_buffer_stats_by_table"></a>*sys.innodb_buffer_stats_by_table*

[!IMPORTANT]
> Запрос этого представления может повлиять на производительность. Устранение неполадок следует запланировать на нерабочее время.

Буферный пул InnoDB находится в памяти и является основным механизмом кэширования между СУБД и хранилищем. Размер буферного пула привязан к уровню производительности. Его можно изменить, только выбрав другой номер SKU продукта. Как и с памятью в операционной системе, старые страницы выгружаются, чтобы освободить место для новых данных. Чтобы узнать, какие таблицы используют больше всего памяти буферного пула InnoDB, можно запросить представление *sys.innodb_buffer_stats_by_table*.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/innodb-buffer-status.png" alt-text="Состояние буфера InnoDB":::

На рисунке выше видно, что, за исключением системных таблиц и представлений, каждая таблица базы данных mysqldatabase033, которая размещает один из сайтов WordPress, занимает 16 КБ или 1 страницу данных в памяти.

### <a name="sysschema_unused_indexes--sysschema_redundant_indexes"></a>*Sys.schema_unused_indexes* & *sys.schema_redundant_indexes*

Индексы являются эффективным инструментом повышения производительности чтения, однако они влекут дополнительные затраты, связанные с операциями вставки и хранением. *sys.schema_unused_indexes* и *sys.schema_redundant_indexes* предоставляют сведения об неиспользуемых или повторяющихся индексах.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/unused-indexes.png" alt-text="Неиспользуемые индексы":::

:::image type="content" source="./media/howto-troubleshoot-sys-schema/redundant-indexes.png" alt-text="Избыточные индексы":::

## <a name="conclusion"></a>Заключение

Таким образом, sys_schema является мощным инструментом, который подходит и для настройки производительности, и для обслуживания базы данных. Обязательно воспользуйтесь этой функцией в Базе данных Azure для MySQL. 

## <a name="next-steps"></a>Дальнейшие действия
- Найти ответы на важные для вас вопросы, а также задать вопрос или ответить на него можно на [странице вопросов и ответов на сайте Майкрософт](/answers/topics/azure-database-mysql.html) или на [сайте Stack Overflow](https://stackoverflow.com/questions/tagged/azure-database-mysql).