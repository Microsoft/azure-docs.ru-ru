---
title: Создание и обновление статистики по таблицам
description: Рекомендации и примеры для создания и обновления статистики оптимизации запросов для таблиц в выделенном пуле SQL.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 05/09/2018
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 3ade41c51cbb8065734e8957cfc8b9f0c22b2df3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98673372"
---
# <a name="table-statistics-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Статистика таблиц для выделенного пула SQL в Azure синапсе Analytics

В этой статье вы найдете рекомендации и примеры создания и обновления статистики оптимизации запросов для таблиц в выделенном пуле SQL.

## <a name="why-use-statistics"></a>Для чего используется статистика?

Чем более выделенный пул SQL знает о ваших данных, тем быстрее она может выполнять запросы к ней. После загрузки данных в выделенный пул SQL сбор статистики по данным является одним из наиболее важных вещей, которые можно выполнить для оптимизации запросов.

Оптимизатор запросов выделенного пула SQL является оптимизатором на основе затрат. Он сравнивает стоимость разных планов запроса, а затем выбирает план с наименьшей стоимостью. В большинстве случаев он выбирает план, который выполняется быстрее всего.

Например, если оптимизатор вычислит, что дата, указанная в фильтре запроса, вернет одну строку, будет выбран один план. Если он вычислит, что выбранная дата вернет 1 000 000 строк, будет возвращен другой план.

## <a name="automatic-creation-of-statistic"></a>Автоматическое создание статистики

Если параметр Database AUTO_CREATE_STATISTICS включен, выделенный пул SQL анализирует входящие запросы пользователей на отсутствие статистики.

Если она отсутствует, оптимизатор запросов создает статистику по отдельным столбцам в предикате запроса или условии соединения, чтобы улучшить оценку кратности для плана запроса.

> [!NOTE]
> Автоматическое создание статистики в настоящее время включено по умолчанию.

Чтобы проверить, настроен ли выделенный пул SQL, AUTO_CREATE_STATISTICS настроить, выполнив следующую команду:

```sql
SELECT name, is_auto_create_stats_on
FROM sys.databases
```

Если в выделенном пуле SQL не настроен AUTO_CREATE_STATISTICS, рекомендуется включить это свойство, выполнив следующую команду:

```sql
ALTER DATABASE <yourdatawarehousename>
SET AUTO_CREATE_STATISTICS ON
```

Эти инструкции активируют автоматическое создание статистики:

- SELECT
- INSERT SELECT
- CTAS
- UPDATE
- DELETE
- EXPLAIN, если они содержат операцию соединения или обнаружено наличие предиката.

> [!NOTE]
> Автоматическое создание статистики не выполняется для временных и внешних таблиц.

Автоматическое создание статистики выполняется синхронно, поэтому, если статистика для столбцов отсутствует, возможно небольшое замедление выполнения запросов. Время создания статистики для одного столбца зависит от размера таблицы.

Чтобы избежать ощутимого замедления, убедитесь, что сначала создается статистика, выполняя рабочую нагрузку теста производительности перед профилированием системы.

> [!NOTE]
> Создание статистики будет зарегистрировано в [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) в другом контексте пользователя.

При автоматическом создании статистики используется следующий формат: _WA_Sys_<8-разрядный идентификатор столбца в шестнадцатеричном формате>_<8-разрядный идентификатор таблицы в шестнадцатеричном формате>. Вы можете просмотреть статистику, которая уже была создана, выполнив команду [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) :

```sql
DBCC SHOW_STATISTICS (<table_name>, <target>)
```

Table_name — это имя таблицы, содержащей статистику для вывода. Эта таблица не может быть внешней таблицей. Аргумент target — имя целевого индекса, статистики или столбца, для которого нужно отобразить статистические данные.

## <a name="update-statistics"></a>Обновите статистику

Лучше всего обновлять статистику в столбцах дат каждый день, когда добавляются новые даты. Каждый раз, когда новые строки загружаются в выделенный пул SQL, добавляются новые даты загрузки или даты транзакций. Эти дополнения изменяют распределение данных и делают статистику устаревшей.

Статистика по столбцу страны или региона в таблице клиентов может никогда не обновляться, так как распределение значений обычно не изменяется. Если предположить, что распределение между клиентами постоянно, добавление новых строк в вариант таблицы не изменит распределение данных.

Однако если выделенный пул SQL содержит только одну страну или регион, и вы перенесете данные из новой страны или региона, что приведет к хранению данных из нескольких стран или регионов, необходимо обновить статистику по столбцу страна или регион.

Ниже приведены рекомендации по обновлению статистики.

|||
|-|-|
| **Частота обновления статистики**  | Консервативная: Ежедневно </br> После загрузки или преобразования данных |
| **Выборка** |  Если менее 1 млрд строк, используйте выборку по умолчанию (20 %). </br> Если строк более 1 000 000 000, используйте выборку в 2 %. |

Один из первых вопросов, задаваемых при устранении неполадок запроса: **Актуальна ли статистика?**

На этот вопрос невозможно ответить, исходя только из возраста данных. Актуальный объект статистики может быть старым, если исходные данные существенно не менялись. Если количество строк или распределение значений для столбца значительно изменяется, то *в этот момент* необходимо обновить статистику. 

Динамическое административное представление не позволяет определить, изменились ли данные в таблице с момента последнего обновления статистики.  Следующие два запроса могут помочь определить, устарела ли статистика.

**Запрос 1:**  Найдите разницу между числом строк из статистики (**stats_row_count**) и фактическим числом строк (**actual_row_count**). 

```sql
select 
objIdsWithStats.[object_id], 
actualRowCounts.[schema], 
actualRowCounts.logical_table_name, 
statsRowCounts.stats_row_count, 
actualRowCounts.actual_row_count,
row_count_difference = CASE
    WHEN actualRowCounts.actual_row_count >= statsRowCounts.stats_row_count THEN actualRowCounts.actual_row_count - statsRowCounts.stats_row_count
    ELSE statsRowCounts.stats_row_count - actualRowCounts.actual_row_count
END,
percent_deviation_from_actual = CASE
    WHEN actualRowCounts.actual_row_count = 0 THEN statsRowCounts.stats_row_count
    WHEN statsRowCounts.stats_row_count = 0 THEN actualRowCounts.actual_row_count
    WHEN actualRowCounts.actual_row_count >= statsRowCounts.stats_row_count THEN CONVERT(NUMERIC(18, 0), CONVERT(NUMERIC(18, 2), (actualRowCounts.actual_row_count - statsRowCounts.stats_row_count)) / CONVERT(NUMERIC(18, 2), actualRowCounts.actual_row_count) * 100)
    ELSE CONVERT(NUMERIC(18, 0), CONVERT(NUMERIC(18, 2), (statsRowCounts.stats_row_count - actualRowCounts.actual_row_count)) / CONVERT(NUMERIC(18, 2), actualRowCounts.actual_row_count) * 100)
END
from
(
    select distinct object_id from sys.stats where stats_id > 1
) objIdsWithStats
left join
(
    select object_id, sum(rows) as stats_row_count from sys.partitions group by object_id
) statsRowCounts
on objIdsWithStats.object_id = statsRowCounts.object_id 
left join
(
    SELECT sm.name [schema] ,
    tb.name logical_table_name ,
    tb.object_id object_id ,
    SUM(rg.row_count) actual_row_count
    FROM sys.schemas sm
    INNER JOIN sys.tables tb ON sm.schema_id = tb.schema_id
    INNER JOIN sys.pdw_table_mappings mp ON tb.object_id = mp.object_id
    INNER JOIN sys.pdw_nodes_tables nt ON nt.name = mp.physical_name
    INNER JOIN sys.dm_pdw_nodes_db_partition_stats rg
    ON rg.object_id = nt.object_id
    AND rg.pdw_node_id = nt.pdw_node_id
    AND rg.distribution_id = nt.distribution_id
    WHERE 1 = 1
    GROUP BY sm.name, tb.name, tb.object_id
) actualRowCounts
on objIdsWithStats.object_id = actualRowCounts.object_id

```

**Запрос 2:** Выясните возраст статистики, проверив время последнего обновления статистики в каждой таблице. 

> [!NOTE]
> Если распределение значений для столбца существенно изменилось, то статистику необходимо обновить, независимо от времени ее последнего обновления.

```sql
SELECT
    sm.[name] AS [schema_name],
    tb.[name] AS [table_name],
    co.[name] AS [stats_column_name],
    st.[name] AS [stats_name],
    STATS_DATE(st.[object_id],st.[stats_id]) AS [stats_last_updated_date]
FROM
    sys.objects ob
    JOIN sys.stats st
        ON  ob.[object_id] = st.[object_id]
    JOIN sys.stats_columns sc
        ON  st.[stats_id] = sc.[stats_id]
        AND st.[object_id] = sc.[object_id]
    JOIN sys.columns co
        ON  sc.[column_id] = co.[column_id]
        AND sc.[object_id] = co.[object_id]
    JOIN sys.types  ty
        ON  co.[user_type_id] = ty.[user_type_id]
    JOIN sys.tables tb
        ON  co.[object_id] = tb.[object_id]
    JOIN sys.schemas sm
        ON  tb.[schema_id] = sm.[schema_id]
WHERE
    st.[user_created] = 1;
```

Для **столбцов даты** в выделенном пуле SQL, например, обычно требуется частое обновление статистики. Каждый раз, когда новые строки загружаются в выделенный пул SQL, добавляются новые даты загрузки или даты транзакций. Эти дополнения изменяют распределение данных и делают статистику устаревшей.

И наоборот, статистика по столбцу пола в таблице клиентов может никогда не обновляться. Если предположить, что распределение между клиентами постоянно, добавление новых строк в вариант таблицы не изменит распределение данных.

Если выделенный пул SQL содержит только один пол, а новое требование дает несколько пола, то необходимо обновить статистику по столбцу пола.

Дополнительные сведения можно получить в общем руководстве по [статистике](/sql/relational-databases/statistics/statistics?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="implementing-statistics-management"></a>Реализация управления статистикой

Часто рекомендуется расширять процесс загрузки данных, чтобы гарантировать, что статистика будет обновлена в конце нагрузки, чтобы избежать/сокращать блокировку или состязание за ресурсы между параллельными запросами.  

Загрузка данных происходит, когда таблицы часто меняют свой размер и (или) распределение значений. Загрузка данных — это логическое место для реализации некоторых процессов управления.

Ниже приведены основные принципы обновления статистики.

- Убедитесь, что для каждой загружаемой таблицы обновляется по крайней мере один объект статистики. Тогда при обновлении статистики обновляется информация о размере таблицы (число строк и страниц).
- Сосредоточьтесь на столбцах, участвующих в предложениях JOIN, GROUP BY, ORDER BY и DISTINCT.
- Рекомендуется чаще обновлять столбцы "с возрастающим порядком ключа", например даты транзакций, потому что эти значения не будут включены в гистограмму статистики.
- Рекомендуется реже обновлять столбцы со статическим распределением.
- Помните, что каждый объект статистики обновляется последовательно. Просто реализовать `UPDATE STATISTICS <TABLE_NAME>` может быть не идеальным решением, особенно для обширных таблиц с большим количеством объектов статистики.

Дополнительные сведения см. в разделе об [оценке кратности](/sql/relational-databases/performance/cardinality-estimation-sql-server?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="examples-create-statistics"></a>Примеры: Создание статистики

Эти примеры показывают, как использовать различные параметры для создания статистики. Параметры, которые можно использовать для каждого столбца, зависят от характеристик данных и того, как столбец будет использован в запросах.

### <a name="create-single-column-statistics-with-default-options"></a>Создание одностолбцовой статистики с параметрами по умолчанию

Чтобы создать одностолбцовую статистику, достаточно указать имя объекта статистики и имя столбца.

В этом синтаксисе все параметры используются по умолчанию. По умолчанию при создании статистики используется выборка **20 процентов** таблицы.

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]);
```

Пример:

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1);
```

### <a name="create-single-column-statistics-by-examining-every-row"></a>Создание одностолбцовой статистики путем проверки всех строк

В большинстве случаев достаточно использовать частоту выборки по умолчанию, 20 процентов. Однако вы можете настроить частоту выборки.

Для выборки всей таблицы используйте следующий синтаксис:

```sql
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]) WITH FULLSCAN;
```

Пример:

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH FULLSCAN;
```

### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Создание одностолбцовой статистики с указанием размера выборки

В качестве альтернативы можно указать размер выборки в процентах:

```sql
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH SAMPLE = 50 PERCENT;
```

### <a name="create-single-column-statistics-on-only-some-of-the-rows"></a>Создание одностолбцовой статистики только для некоторых строк

Можно также создать статистику для части строк в таблице. Это называется отфильтрованной статистикой.

Например, отфильтрованную статистику можно использовать при планировании запроса определенной секции большой секционированной таблицы. Создавая статистику только по значениям секции, можно повысить точность статистики и, таким образом, увеличить производительность запросов.

Этот пример создает статистику по диапазону значений. Значения можно легко определить для сопоставления с диапазоном значений в секции.

```sql
CREATE STATISTICS stats_col1 ON table1(col1) WHERE col1 > '2000101' AND col1 < '20001231';
```

> [!NOTE]
> Чтобы оптимизатор запросов рассмотрел возможность использования отфильтрованной статистики, когда выбирает план распределенного запроса, запрос должен быть в пределах определения объекта статистики. Если взять приведенный выше пример, предложение WHERE запроса должно указать значения col1 от 2000101 до 20001231.

### <a name="create-single-column-statistics-with-all-the-options"></a>Создание одностолбцовой статистики со всеми параметрами

Можно также комбинировать параметры. В приведенном ниже примере создается отфильтрованный объект статистики с настраиваемым размером выборки:

```sql
CREATE STATISTICS stats_col1 ON table1 (col1) WHERE col1 > '2000101' AND col1 < '20001231' WITH SAMPLE = 50 PERCENT;
```

Полные справочные сведения см. в статье [CREATE STATISTICS (Transact-SQL)](/sql/t-sql/statements/create-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

### <a name="create-multi-column-statistics"></a>Создание многостолбцовой статистики

Для создания объекта статистики с несколькими столбцами используйте предыдущие примеры, но укажите больше столбцов.

> [!NOTE]
> Гистограмма, используемая для оценки количества строк в результатах запроса, доступна только для первого столбца, указанного в определении объекта статистики.

В этом примере гистограмма создана для *product\_category*. Статистика между столбцами вычисляется по *product\_category* и *product\_* .

```sql
CREATE STATISTICS stats_2cols ON table1 (product_category, product_sub_category) WHERE product_category > '2000101' AND product_category < '20001231' WITH SAMPLE = 50 PERCENT;
```

Так как между *product\_category* и *product\_sub\_category* настроена корреляция, объект статистики с несколькими столбцами может быть полезен, когда к этим столбцам обращаются одновременно.

### <a name="create-statistics-on-all-columns-in-a-table"></a>Создание статистики для всех столбцов в таблице

Один из способов создать статистику — выполнить команды CREATE STATISTICS после создания таблицы.

```sql
CREATE TABLE dbo.table1
(
   col1 int
,  col2 int
,  col3 int
)
WITH
  (
    CLUSTERED COLUMNSTORE INDEX
  )
;

CREATE STATISTICS stats_col1 on dbo.table1 (col1);
CREATE STATISTICS stats_col2 on dbo.table2 (col2);
CREATE STATISTICS stats_col3 on dbo.table3 (col3);
```

### <a name="use-a-stored-procedure-to-create-statistics-on-all-columns-in-a-sql-pool"></a>Использование хранимой процедуры для создания статистики по всем столбцам в пуле SQL

Выделенный пул SQL не имеет системных хранимых процедур, эквивалентного sp_create_stats в SQL Server. Эта хранимая процедура создает объект статистики по одному столбцу для каждого столбца в пуле SQL, который еще не имеет статистики.

Следующий пример поможет приступить к проектированию пула SQL. Вы можете адаптировать код в соответствии со своими нуждами.

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
(   @create_type    tinyint -- 1 default 2 Fullscan 3 Sample
,   @sample_pct     tinyint
)
AS

IF @create_type IS NULL
BEGIN
    SET @create_type = 1;
END;

IF @create_type NOT IN (1,2,3)
BEGIN
    THROW 151000,'Invalid value for @stats_type parameter. Valid range 1 (default), 2 (fullscan) or 3 (sample).',1;
END;

IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN;
    DROP TABLE #stats_ddl;
END;

CREATE TABLE #stats_ddl
WITH    (   DISTRIBUTION    = HASH([seq_nmbr])
        ,   LOCATION        = USER_DB
        )
AS
WITH T
AS
(
SELECT      t.[name]                        AS [table_name]
,           s.[name]                        AS [table_schema_name]
,           c.[name]                        AS [column_name]
,           c.[column_id]                   AS [column_id]
,           t.[object_id]                   AS [object_id]
,           ROW_NUMBER()
            OVER(ORDER BY (SELECT NULL))    AS [seq_nmbr]
FROM        sys.[tables] t
JOIN        sys.[schemas] s         ON  t.[schema_id]       = s.[schema_id]
JOIN        sys.[columns] c         ON  t.[object_id]       = c.[object_id]
LEFT JOIN   sys.[stats_columns] l   ON  l.[object_id]       = c.[object_id]
                                    AND l.[column_id]       = c.[column_id]
                                    AND l.[stats_column_id] = 1
LEFT JOIN    sys.[external_tables] e    ON    e.[object_id]        = t.[object_id]
WHERE       l.[object_id] IS NULL
AND            e.[object_id] IS NULL -- not an external table
)
SELECT  [table_schema_name]
,       [table_name]
,       [column_name]
,       [column_id]
,       [object_id]
,       [seq_nmbr]
,       CASE @create_type
        WHEN 1
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+')' AS VARCHAR(8000))
        WHEN 2
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH FULLSCAN' AS VARCHAR(8000))
        WHEN 3
        THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH SAMPLE '+CONVERT(varchar(4),@sample_pct)+' PERCENT' AS VARCHAR(8000))
        END AS create_stat_ddl
FROM T
;

DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''
;

WHILE @i <= @t
BEGIN
    SET @s=(SELECT create_stat_ddl FROM #stats_ddl WHERE seq_nmbr = @i);

    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END

DROP TABLE #stats_ddl;
```

Чтобы создать статистику для всех столбцов в таблице с помощью параметров по умолчанию, выполните хранимую процедуру.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 1, NULL;
```

Чтобы создать статистику по всем столбцам в таблице с помощью FULLSCAN, вызовите эту процедуру.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 2, NULL;
```

Чтобы создать выборочную статистику для всех столбцов в таблице, введите 3 и процент выборки. В этой процедуре используется частота выборки в 20%.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 3, 20;
```

## <a name="examples-update-statistics"></a>Примеры: Обновите статистику

Чтобы обновить статистику, можно:

- Обновить один объект статистики. Указать имя объекта статистики, который вы хотите обновить.
- Обновить все объекты статистики для таблицы. Указать имя таблицы, а не один объект статистики.

### <a name="update-one-specific-statistics-object"></a>Обновление одного указанного объекта статистики

Для обновления указанного объекта статистики используйте следующий синтаксис:

```sql
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

Пример:

```sql
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

Обновляя определенные объекты статистики, можно свести к минимуму затраты времени и ресурсов на управление статистикой. Это требует некоторых усилий, чтобы выбрать наиболее подходящие объекты статистики для обновления.

### <a name="update-all-statistics-on-a-table"></a>Обновление всей статистики для таблицы

Ниже показан простой метод обновления всех объектов статистики для таблицы.

```sql
UPDATE STATISTICS [schema_name].[table_name];
```

Пример:

```sql
UPDATE STATISTICS dbo.table1;
```

Инструкцию UPDATE STATISTICS легко использовать. Просто помните, что она обновляет *всю* статистику для таблицы и, следовательно, может выполнять больше работы, чем необходимо. Если производительность не является проблемой, это самый простой и наиболее полный способ гарантировать актуальность статистики.

> [!NOTE]
> При обновлении всей статистики для таблицы выделенный пул SQL выполняет проверку для выборки таблицы для каждого объекта статистики. Если таблица большого размера и содержит много столбцов и статистики, может оказаться эффективнее обновлять отдельные данные статистики по необходимости.

Сведения о реализации `UPDATE STATISTICS` процедуры см. в разделе [временные таблицы](sql-data-warehouse-tables-temporary.md). Метод реализации слегка отличается от процедуры `CREATE STATISTICS`, описанной выше, но результат одинаков.

Полный синтаксис см. в разделе [Обновление статистики](/sql/t-sql/statements/update-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="statistics-metadata"></a>Метаданные статистики

Существует несколько системных представлений и функций, которые можно использовать для поиска информации о статистике. Например, можно узнать, устарел ли объект статистики, воспользовавшись функцией stats-date, чтобы посмотреть, когда статистика была в последний раз создана или обновлена.

### <a name="catalog-views-for-statistics"></a>Представления каталога для статистики

Вот какие системные представления показывают информацию о статистике:

| Представление каталога | Описание |
|:--- |:--- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого столбца. |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого объекта в базе данных. |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждой схемы в базе данных. |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого объекта статистики. |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого столбца в объекте статистики. Ссылается на sys.columns. |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждой таблицы (включая внешние таблицы). |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого типа данных. |

### <a name="system-functions-for-statistics"></a>Системные функции для статистики

Эти системные функции полезны для работы со статистикой:

| Системная функция | Описание |
|:--- |:--- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Дата последнего обновления объекта статистики. |
| [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) |Сводная и подробная информация о распределении значений согласно объекту статистики. |

### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Сочетание столбцов и функций статистики в одном представлении

Это представление содержит столбцы, относящиеся к статистике, и результаты функции STATS_DATE().

```sql
CREATE VIEW dbo.vstats_columns
AS
SELECT
        sm.[name]                           AS [schema_name]
,       tb.[name]                           AS [table_name]
,       st.[name]                           AS [stats_name]
,       st.[filter_definition]              AS [stats_filter_definition]
,       st.[has_filter]                     AS [stats_is_filtered]
,       STATS_DATE(st.[object_id],st.[stats_id])
                                            AS [stats_last_updated_date]
,       co.[name]                           AS [stats_column_name]
,       ty.[name]                           AS [column_type]
,       co.[max_length]                     AS [column_max_length]
,       co.[precision]                      AS [column_precision]
,       co.[scale]                          AS [column_scale]
,       co.[is_nullable]                    AS [column_is_nullable]
,       co.[collation_name]                 AS [column_collation_name]
,       QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS two_part_name
,       QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                            AS three_part_name
FROM    sys.objects                         AS ob
JOIN    sys.stats           AS st ON    ob.[object_id]      = st.[object_id]
JOIN    sys.stats_columns   AS sc ON    st.[stats_id]       = sc.[stats_id]
                            AND         st.[object_id]      = sc.[object_id]
JOIN    sys.columns         AS co ON    sc.[column_id]      = co.[column_id]
                            AND         sc.[object_id]      = co.[object_id]
JOIN    sys.types           AS ty ON    co.[user_type_id]   = ty.[user_type_id]
JOIN    sys.tables          AS tb ON  co.[object_id]        = tb.[object_id]
JOIN    sys.schemas         AS sm ON  tb.[schema_id]        = sm.[schema_id]
WHERE   1=1
AND     st.[user_created] = 1
;
```

## <a name="dbcc-show_statistics-examples"></a>Примеры DBCC SHOW_STATISTICS()

DBCC SHOW_STATISTICS() отображает данные, хранящиеся в объекте статистики. Эти данные состоят из трех частей:

- Заголовок
- Вектор плотности
- Гистограмма

Заголовок метаданных о статистике. Гистограмма отображает распределение значений в первом ключевом столбце объекта статистики. Вектор плотности измеряет корреляцию между столбцами.

> [!NOTE]
> Выделенный пул SQL вычисляет оценку количества элементов с помощью любого из данных в объекте статистики.

### <a name="show-header-density-and-histogram"></a>Отображение заголовка, плотности и гистограммы

Этот простой пример показывает все три части объекта статистики.

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

Пример:

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

### <a name="show-one-or-more-parts-of-dbcc-show_statistics"></a>Отображение одной или нескольких частей DBCC SHOW_STATISTICS()

Если вы заинтересованы только в просмотре определенных частей, используйте предложение `WITH` и укажите, какие части требуется показать:

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>) WITH stat_header, histogram, density_vector
```

Пример:

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1) WITH histogram, density_vector
```

## <a name="dbcc-show_statistics-differences"></a>Отличия DBCC SHOW_STATISTICS()

Инструкция DBCC SHOW_STATISTICS () более строго реализована в выделенном пуле SQL по сравнению с SQL Server:

- Недокументированные возможности не поддерживаются.
- Нельзя использовать Stats_stream.
- Нельзя соединить результаты для определенных подмножеств данных статистики. Например, STAT_HEADER JOIN DENSITY_VECTOR.
- Невозможно задать NO_INFOMSGS для подавления сообщений.
- Нельзя использовать квадратные скобки вокруг имен статистики.
- Нельзя использовать имена столбцов для идентификации объектов статистики.
- Пользовательская ошибка 2767 не поддерживается.

## <a name="next-steps"></a>Дальнейшие действия

Для дальнейшего повышения производительности запросов ознакомьтесь с разделом [Мониторинг рабочей нагрузки с помощью динамических административных представлений](sql-data-warehouse-manage-monitor.md).
