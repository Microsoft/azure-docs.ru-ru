---
title: Секционирование таблиц
description: Рекомендации и примеры использования секций таблиц в выделенном пуле SQL
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/18/2019
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 8a59c24100b433719ccfd3a9ea1b6a676695d381
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98673440"
---
# <a name="partitioning-tables-in-dedicated-sql-pool"></a>Секционирование таблиц в выделенном пуле SQL

Рекомендации и примеры использования секций таблиц в выделенном пуле SQL.

## <a name="what-are-table-partitions"></a>Что такое секции таблицы

Секции таблицы позволяют разделять данные на более мелкие группы данных. В большинстве случаев секции таблицы создаются по столбцу с датами. Секционирование поддерживается во всех выделенных типах таблиц пула SQL. включая кластеризованный columnstore, кластеризованный индекс и кучу. Секционирование также поддерживается для всех типов распределения, включая распределение с помощью хэша или методом циклического перебора.  

Секционирование позволяет упростить операции обслуживания данных и повысить производительность запросов. Однако не всегда можно получить оба преимущества. Это зависит от способа загрузки данных и возможности использования одного столбца для обоих целей, так как секционирование может быть выполнено только по одному столбцу.

### <a name="benefits-to-loads"></a>Преимущества для нагрузок

Основное преимущество секционирования в выделенном пуле SQL — повышение эффективности и производительности загрузки данных с помощью удаления, переключения и объединения секций. В большинстве случаев данные секционированы по столбцу дат, тесно связанному с порядком, в котором данные загружаются в пул SQL. Одним из лучших преимуществ использования секций для поддержания данных является избежание ведения журнала транзакций. Хотя намного проще использовать обычные операции вставки, обновления и удаления данных, которые не требуют много внимания и усилий, применение секционирования при загрузке может существенно повысить производительность.

С помощью переключения секций можно быстро удалить или заменить часть таблицы.  Например, таблица фактов по продажам может содержать только данные за последние 36 месяцев. В конце каждого месяца данные о продажах за самый давний месяц удаляются из таблицы.  Удалить эти данные можно с использованием инструкции DELETE. 

Тем не менее построчное удаление большого объема данных с помощью инструкции DELETE может длиться слишком долго, а также создать риск больших транзакций, для отката которых в случае неполадки также понадобится много времени. Более оптимальный подход — удалить самую давнюю секцию данных. Если удаление отдельных строк может занять несколько часов, то удаление всей секции — несколько секунд.

### <a name="benefits-to-queries"></a>Преимущества для запросов

С помощью секционирования также можно повысить производительность запросов. Запрос, который применяет фильтр к секционированным данным, может ограничить сканирование только соответствующими секциями. Этот метод фильтрации позволяет не сканировать всю таблицу, а проверить только небольшое подмножество данных. С появлением кластеризованных индексов Columnstore преимущества производительности исключений предикатов менее существенны, но в некоторых случаях они могут быть эффективны для запросов. 

Например, если таблица фактов по продажам секционирована в 36 месяцев с помощью поля Дата продажи, то запросы, которые фильтруются по дате продажи, могут пропускать Поиск в секциях, которые не соответствуют фильтру.

## <a name="sizing-partitions"></a>Изменение размера секций

Хотя секционирование повышает производительность в ряде сценариев, в некоторых случаях создание таблицы со **слишком большим количеством** секций может повлечь снижение производительности.  В частности, это свойственно для таблиц с кластеризированными индексами Columnstore. 

Чтобы секционирование было эффективным, нужно понимать, когда следует его использовать и сколько секций необходимо создать. Не существует строгих правил в отношении количества секций. Выбор зависит от характера данных и числа секций, к которым одновременно применяется нагрузка. Правильно составленная схема секционирования обычно содержит от нескольких десятков до нескольких сотен секций (но не тысяч).

При секционировании таблиц с **кластеризованными индексами columnstore** важно учитывать, сколько строк будет содержать каждая секция. Для оптимального сжатия и производительности таких таблиц требуется как минимум 1 миллион строк на одно распределение и одну секцию. Перед созданием секций выделенный пул SQL уже делит каждую таблицу на 60 распределенных баз данных. 

Таким образом, секционирование таблицы выступает дополнением к распределениям, созданным в фоновом режиме. В этом примере, если таблица фактов продаж содержала 36 ежемесячных секций и при условии, что выделенный пул SQL содержит 60, то таблица фактов Sales должна 60 000 000 содержать строки в месяц или 2 100 000 000 строки, если все месяцы заполнены. Если в таблице намного меньше строк, чем рекомендуемый минимум на одну секцию, уменьшите количество секций, чтобы на каждую из них приходилось больше строк. 

Дополнительные сведения см. в статье [Индексирование таблиц в хранилище данных SQL](sql-data-warehouse-tables-index.md). В ней рассматриваются запросы, с помощью которых можно оценить качество кластеризованных индексов columnstore.

## <a name="syntax-differences-from-sql-server"></a>Отличия синтаксиса от SQL Server

Выделенный пул SQL предоставляет способ определения секций, которые проще, чем SQL Server. Функции секционирования и схемы не используются в выделенном пуле SQL, так как они находятся в SQL Server. Вам нужно только определить секционированный столбец и граничные точки. 

Хотя синтаксис секционирования может немного отличаться от синтаксиса SQL Server, основные понятия одинаковые. SQL Server и выделенный пул SQL поддерживают один столбец секционирования для каждой таблицы, который может быть секционирован по диапазону. Дополнительные сведения о секционировании см. в статье [Секционированные таблицы и индексы](/sql/relational-databases/partitions/partitioned-tables-and-indexes?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

В приведенном ниже примере таблица FactInternetSales секционируется по столбцу OrderDateKey с помощью инструкции [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

```sql
CREATE TABLE [dbo].[FactInternetSales]
(
    [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101,20010101,20020101
                    ,20030101,20040101,20050101
                    )
                )
)
;
```

## <a name="migrating-partitioning-from-sql-server"></a>Перенос секционирования из SQL Server

Чтобы выполнить миграцию SQL Server определений секций в выделенный пул SQL, просто сделайте следующее:

- удалить [схему секционирования](/sql/t-sql/statements/create-partition-scheme-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) SQL Server;
- добавить определение [функции секционирования](/sql/t-sql/statements/create-partition-function-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) в инструкцию CREATE TABLE.

Если вы переносите секционированную таблицу из экземпляра SQL Server, количество строк в каждой секции можно запросить с помощью приведенного ниже алгоритма SQL. Помните, что если в выделенном пуле SQL используется одна и та же гранулярность секционирования, число строк на секцию уменьшается на 60.  

```sql
-- Partition information for a SQL Server Database
SELECT      s.[name]                        AS      [schema_name]
,           t.[name]                        AS      [table_name]
,           i.[name]                        AS      [index_name]
,           p.[partition_number]            AS      [partition_number]
,           SUM(a.[used_pages]*8.0)         AS      [partition_size_kb]
,           SUM(a.[used_pages]*8.0)/1024    AS      [partition_size_mb]
,           SUM(a.[used_pages]*8.0)/1048576 AS      [partition_size_gb]
,           p.[rows]                        AS      [partition_row_count]
,           rv.[value]                      AS      [partition_boundary_value]
,           p.[data_compression_desc]       AS      [partition_compression_desc]
FROM        sys.schemas s
JOIN        sys.tables t                    ON      t.[schema_id]         = s.[schema_id]
JOIN        sys.partitions p                ON      p.[object_id]         = t.[object_id]
JOIN        sys.allocation_units a          ON      a.[container_id]      = p.[partition_id]
JOIN        sys.indexes i                   ON      i.[object_id]         = p.[object_id]
                                            AND     i.[index_id]          = p.[index_id]
JOIN        sys.data_spaces ds              ON      ds.[data_space_id]    = i.[data_space_id]
LEFT JOIN   sys.partition_schemes ps        ON      ps.[data_space_id]    = ds.[data_space_id]
LEFT JOIN   sys.partition_functions pf      ON      pf.[function_id]      = ps.[function_id]
LEFT JOIN   sys.partition_range_values rv   ON      rv.[function_id]      = pf.[function_id]
                                            AND     rv.[boundary_id]      = p.[partition_number]
WHERE       p.[index_id] <=1
GROUP BY    s.[name]
,           t.[name]
,           i.[name]
,           p.[partition_number]
,           p.[rows]
,           rv.[value]
,           p.[data_compression_desc]
;
```

## <a name="partition-switching"></a>Переключение секций

Выделенный пул SQL поддерживает разделение, слияние и переключение секций. Для каждой из этих функций используется инструкция [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

Для переключения секций между двумя таблицами необходимо убедиться, что секции выровнены по соответствующим границам и их определения таблиц совпадают. Вы не можете установить диапазон значений в таблице с помощью ограничений CHECK, а значит, исходная таблица должна иметь такие же границы секций, как целевая. Если границы не будут совпадать, переключение секций завершится ошибкой из-за невозможности синхронизировать метаданные.

### <a name="how-to-split-a-partition-that-contains-data"></a>Как разделить секцию, которая содержит данные

Наиболее эффективный способ разделения секции, которая уже содержит данные, — использовать инструкцию `CTAS` . Если секционированная таблица является кластеризованным индексом columnstore, то разделять в такой таблице можно только пустые секции.

В приведенном ниже примере создается секционированная таблица columnstore. Затем в каждую секцию вставляется одна строка:

```sql
CREATE TABLE [dbo].[FactInternetSales]
(
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101
                    )
                )
)
;

INSERT INTO dbo.FactInternetSales
VALUES (1,19990101,1,1,1,1,1,1);
INSERT INTO dbo.FactInternetSales
VALUES (1,20000101,1,1,1,1,1,1);
```

Следующий запрос позволяет найти количество строк с помощью представления каталога `sys.partitions`.

```sql
SELECT  QUOTENAME(s.[name])+'.'+QUOTENAME(t.[name]) as Table_name
,       i.[name] as Index_name
,       p.partition_number as Partition_nmbr
,       p.[rows] as Row_count
,       p.[data_compression_desc] as Data_Compression_desc
FROM    sys.partitions p
JOIN    sys.tables     t    ON    p.[object_id]   = t.[object_id]
JOIN    sys.schemas    s    ON    t.[schema_id]   = s.[schema_id]
JOIN    sys.indexes    i    ON    p.[object_id]   = i.[object_Id]
                            AND   p.[index_Id]    = i.[index_Id]
WHERE t.[name] = 'FactInternetSales'
;
```

Следующая команда разделения завершится ошибкой:

```sql
ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

Сообщение 35346, уровень 15, состояние 1, строка 44. Использование предложения SPLIT в инструкции ALTER PARTITION привело к ошибке, поскольку секция непустая. Если для таблицы имеется индекс columnstore, разделять можно только пустые секции. Рекомендуется отключить индекс columnstore перед выполнением инструкции ALTER PARTITION, а после завершения ALTER PARTITION перестроить индекс columnstore.

Можно создать новую таблицу для хранения данных при помощи `CTAS`.

```sql
CREATE TABLE dbo.FactInternetSales_20000101
    WITH    (   DISTRIBUTION = HASH(ProductKey)
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101
                                )
                            )
            )
AS
SELECT *
FROM    FactInternetSales
WHERE   1=2
;
```

Так как границы этих секций выровнены, переключение допускается. Так вы получите пустую секцию в исходной таблице, чтобы впоследствии разделить ее.

```sql
ALTER TABLE FactInternetSales SWITCH PARTITION 2 TO  FactInternetSales_20000101 PARTITION 2;

ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

Теперь нам осталось только выровнять данные по новым границам секции, используя `CTAS`, и переключить данные на основную таблицу.

```sql
CREATE TABLE [dbo].[FactInternetSales_20000101_20010101]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales_20000101]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

ALTER TABLE dbo.FactInternetSales_20000101_20010101 SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2;
```

Когда вы завершите перемещение данных, рекомендуем обновить статистику для целевой таблицы. Благодаря обновлению статистика будет точно отражать новое распределение данных по секциям.

```sql
UPDATE STATISTICS [dbo].[FactInternetSales];
```

### <a name="load-new-data-into-partitions-that-contain-data-in-one-step"></a>Загрузка новых данных в секции, содержащие данные за один шаг

Загрузка данных в секции с переключением секций — это удобный способ размещения новых данных в таблице, которая не видна пользователям.  В занятых системах может быть проблематично работать с конфликтами блокировки, связанными с переключением секций.  

Чтобы очистить существующие данные в секции, необходимо `ALTER TABLE` использовать для переключения данных.  Затем `ALTER TABLE` для переключения новых данных требовалось другое.  

В выделенном пуле SQL `TRUNCATE_TARGET` параметр поддерживается в `ALTER TABLE` команде.  `TRUNCATE_TARGET` `ALTER TABLE` Команда перезаписывает существующие данные в секции новыми данными.  Ниже приведен пример использования `CTAS` для создания новой таблицы с существующими данными, вставки новых данных, а затем переключения всех данных обратно в целевую таблицу с перезаписью существующих данных.

```sql
CREATE TABLE [dbo].[FactInternetSales_NewSales]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

INSERT INTO dbo.FactInternetSales_NewSales
VALUES (1,20000101,2,2,2,2,2,2);

ALTER TABLE dbo.FactInternetSales_NewSales SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2 WITH (TRUNCATE_TARGET = ON);  
```

### <a name="table-partitioning-source-control"></a>Система управления версиями секционирования таблиц

Чтобы избежать **повреждения** определения таблицы в системе управления версиями, попробуйте применить следующий подход:

1. Создайте таблицу как секционированную, но без значений секций.

    ```sql
    CREATE TABLE [dbo].[FactInternetSales]
    (
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
    )
    WITH
    (   CLUSTERED COLUMNSTORE INDEX
    ,   DISTRIBUTION = HASH([ProductKey])
    ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES () )
    )
    ;
    ```

1. `SPLIT` таблицы в процессе развертывания:

    ```sql
     -- Create a table containing the partition boundaries

    CREATE TABLE #partitions
    WITH
    (
        LOCATION = USER_DB
    ,   DISTRIBUTION = HASH(ptn_no)
    )
    AS
    SELECT  ptn_no
    ,       ROW_NUMBER() OVER (ORDER BY (ptn_no)) as seq_no
    FROM    (
        SELECT CAST(20000101 AS INT) ptn_no
        UNION ALL
        SELECT CAST(20010101 AS INT)
        UNION ALL
        SELECT CAST(20020101 AS INT)
        UNION ALL
        SELECT CAST(20030101 AS INT)
        UNION ALL
        SELECT CAST(20040101 AS INT)
    ) a
    ;

     -- Iterate over the partition boundaries and split the table

    DECLARE @c INT = (SELECT COUNT(*) FROM #partitions)
    ,       @i INT = 1                                 --iterator for while loop
    ,       @q NVARCHAR(4000)                          --query
    ,       @p NVARCHAR(20)     = N''                  --partition_number
    ,       @s NVARCHAR(128)    = N'dbo'               --schema
    ,       @t NVARCHAR(128)    = N'FactInternetSales' --table
    ;

    WHILE @i <= @c
    BEGIN
        SET @p = (SELECT ptn_no FROM #partitions WHERE seq_no = @i);
        SET @q = (SELECT N'ALTER TABLE '+@s+N'.'+@t+N' SPLIT RANGE ('+@p+N');');

        -- PRINT @q;
        EXECUTE sp_executesql @q;
        SET @i+=1;
    END

     -- Code clean-up

    DROP TABLE #partitions;
    ```

При таком подходе код в системе управления версиями остается статическим, а граничные значения секционирования могут быть динамическими. постепенное развитие с использованием пула SQL с течением времени.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о разработке таблиц см. в статье [Общие сведения о проектировании таблиц в хранилище данных SQL Azure](sql-data-warehouse-tables-overview.md).
