---
title: Создание и обновление статистики с помощью ресурсов Azure синапсе SQL
description: Рекомендации и примеры для создания и обновления статистики оптимизации запросов для таблиц в Synapse SQL.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/19/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.custom: ''
ms.openlocfilehash: 450a089c6cc1c77ac26cb0aa339277d5c49b41c8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104594790"
---
# <a name="statistics-in-synapse-sql"></a>Статистика в Synapse SQL

В этой статье приведены рекомендации и примеры создания и обновления статистики оптимизации запросов с помощью ресурсов синапсе SQL: выделенного пула SQL и бессерверного пула SQL.

## <a name="statistics-in-dedicated-sql-pool"></a>Статистика в выделенном пуле SQL

### <a name="why-use-statistics"></a>Для чего используется статистика?

Чем более выделенный пул SQL знает о ваших данных, тем быстрее она может выполнять запросы. После загрузки данных в выделенный пул SQL сбор статистики по данным является одной из наиболее важных вещей, которые можно выполнить для оптимизации запросов.  

Оптимизатор запросов выделенного пула SQL является оптимизатором на основе затрат. Он сравнивает стоимость разных планов запроса, а затем выбирает план с наименьшей стоимостью. В большинстве случаев он выбирает план, который выполняется быстрее всего.

Например, если оптимизатор вычислит, что дата, указанная в фильтре запроса, вернет одну строку, будет выбран один план. Если он вычислит, что выбранная дата вернет 1 000 000 строк, будет возвращен другой план.

### <a name="automatic-creation-of-statistics"></a>Автоматическое создание статистики

Выделенный модуль пула SQL анализирует входящие запросы пользователей на отсутствие статистики, если параметр Database AUTO_CREATE_STATISTICS имеет значение `ON` .  Если статистика отсутствует, оптимизатор запросов создает статистику по отдельным столбцам в предикате запроса или условии соединения. 

Эта функция используется для улучшения оценки кратности для плана запроса.

> [!IMPORTANT]
> Автоматическое создание статистики в настоящее время включено по умолчанию.

Можно проверить, настроено ли свойство AUTO_CREATE_STATISTICS для хранилища данных, выполнив следующую команду.

```sql
SELECT name, is_auto_create_stats_on
FROM sys.databases
```

Если для хранилища данных не настроено свойство AUTO_CREATE_STATISTICS, рекомендуется включить его, выполнив следующую команду.

```sql
ALTER DATABASE <yourdatawarehousename>
SET AUTO_CREATE_STATISTICS ON
```

Приведенные ниже инструкции активируют автоматическое создание статистики:

- SELECT
- INSERT SELECT
- CTAS
- UPDATE
- DELETE
- EXPLAIN, если они содержат операцию соединения или обнаружено наличие предиката.

> [!NOTE]
> Автоматическое создание статистики не выполняется для временных и внешних таблиц.

Автоматическое создание статистики выполняется синхронно. Поэтому, если в столбцах отсутствует статистика, может наблюдаться незначительное замедление выполнения запросов. Время создания статистики для одного столбца зависит от размера таблицы.

Чтобы избежать ощутимого замедления, убедитесь, что сначала создается статистика, выполняя рабочую нагрузку теста производительности перед профилированием системы.

> [!NOTE]
> Создание статистики регистрируется в журнале [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?view=azure-sqldw-latest&preserve-view=true) в контексте другого пользователя.

При автоматическом создании статистики используется следующий формат: _WA_Sys_<8-разрядный идентификатор столбца в шестнадцатеричном формате>_<8-разрядный идентификатор таблицы в шестнадцатеричном формате>. Созданную статистику можно просмотреть, выполнив команду [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?view=azure-sqldw-latest&preserve-view=true).

```sql
DBCC SHOW_STATISTICS (<table_name>, <target>)
```

Аргумент table_name — имя таблицы, содержащей статистику для вывода. Это не может быть внешняя таблица. Аргумент target — имя целевого индекса, статистики или столбца, для которого нужно отобразить статистические данные.

### <a name="update-statistics"></a>Обновите статистику

Лучше всего обновлять статистику в столбцах дат каждый день, когда добавляются новые даты. При каждой загрузке строк в хранилище данных добавляются новые даты загрузки или даты транзакций. Эти дополнения изменяют распределение данных и делают статистику устаревшей.

Статистика по столбцу страны или региона в таблице клиентов может никогда не обновляться, потому что распределение значений меняется не часто. Если предположить, что распределение между клиентами постоянно, добавление новых строк в вариант таблицы не изменит распределение данных.

Тем не менее, если хранилище данных содержит только одну страну или регион и добавляются данные из новой страны или региона, то статистику по столбцу страны или региона необходимо будет обновить.

Ниже приведены рекомендации по обновлению статистики.

|Тип|Рекомендация|
|-|-|
| **Частота обновления статистики**  | Консервативная: Ежедневно </br> После загрузки или преобразования данных |
| **Выборка** |  Если менее 1 млрд строк, используйте выборку по умолчанию (20 %). </br> Если строк более 1 000 000 000, используйте выборку в 2 %. |

### <a name="determine-last-statistics-update"></a>Определение времени последнего обновления статистики

Один из первых вопросов, задаваемых при устранении неполадок запроса: **Актуальна ли статистика?**

На этот вопрос невозможно ответить, исходя только из возраста данных. Актуальный объект статистики может быть старым, если исходные данные существенно не менялись. Если количество строк или распределение значений для столбца значительно изменяется, то *в этот момент* необходимо обновить статистику.

Динамическое административное представление, в котором можно определить, изменились ли данные в таблице со времени последнего обновления статистики, отсутствует. Знание возраста статистики может дать вам частичное представление об этом. 

Можно использовать следующий запрос, чтобы определить время последнего обновления статистики в каждой таблице.

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

Например, для **столбцов дат** в хранилище данных обычно требуется часто обновлять статистику. При каждой загрузке строк в хранилище данных добавляются новые даты загрузки или даты транзакций. Эти дополнения изменяют распределение данных и делают статистику устаревшей.

Статистика по столбцу пола в таблице клиентов может никогда не обновляться. Если предположить, что распределение между клиентами постоянно, добавление новых строк в вариант таблицы не изменит распределение данных.

Тем не менее, если хранилище данных содержит сведения только о клиентах одного пола, а новые требования предписывают гендерные различия, вам надо обновить статистику по столбцу пола. 

Дополнительные сведения см. в статье [Статистика](/sql/relational-databases/statistics/statistics).

### <a name="implement-statistics-management"></a>Реализация управления статистикой

Часто рекомендуется расширить процесс загрузки данных, чтобы обеспечить обновление статистики в конце загрузки. Загрузка данных происходит, когда для таблицы часто меняется размер, распределение значений либо и то, и другое. Поэтому логично реализовать в процессе загрузки некоторые процессы управления.

Ниже приведены некоторые основные принципы обновления статистики в процессе загрузки:

- Убедитесь, что для каждой загружаемой таблицы обновляется по крайней мере один объект статистики. Тогда в процессе обновления статистики будет обновляться информация о размере таблицы (число строк и страниц).
- Сосредоточьтесь на столбцах, участвующих в предложениях JOIN, GROUP BY, ORDER BY и DISTINCT.
- Рекомендуется чаще обновлять столбцы "с возрастающим порядком ключа", например даты транзакций, потому что эти значения не будут включены в гистограмму статистики.
- Рекомендуется реже обновлять столбцы со статическим распределением.
- Помните, что каждый объект статистики обновляется последовательно. Просто реализовать `UPDATE STATISTICS <TABLE_NAME>` может быть не идеальным решением, особенно для обширных таблиц с большим количеством объектов статистики.

Дополнительные сведения см. в разделе об [оценке кратности](/sql/relational-databases/performance/cardinality-estimation-sql-server).

### <a name="examples-create-statistics"></a>Примеры: Создание статистики

Эти примеры показывают, как использовать различные параметры для создания статистики. Параметры, которые можно использовать для каждого столбца, зависят от характеристик данных и того, как столбец будет использован в запросах.

#### <a name="create-single-column-statistics-with-default-options"></a>Создание одностолбцовой статистики с параметрами по умолчанию

Чтобы создать одностолбцовую статистику, достаточно указать имя объекта статистики и имя столбца.
В этом синтаксисе все параметры используются по умолчанию. По умолчанию выделенный пул SQL выбирает **20 процентов** таблицы при создании статистики.

```sql
CREATE STATISTICS [statistics_name]
    ON [schema_name].[table_name]([column_name]);
```

Пример:

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1);
```

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Создание одностолбцовой статистики путем проверки всех строк

В большинстве случаев достаточно использовать частоту выборки по умолчанию, 20 процентов. Однако вы можете настроить частоту выборки. Для выборки всей таблицы используйте следующий синтаксис:

```sql
CREATE STATISTICS [statistics_name]
    ON [schema_name].[table_name]([column_name])
    WITH FULLSCAN;
```

Пример:

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1)
    WITH FULLSCAN;
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Создание одностолбцовой статистики с указанием размера выборки

Другой вариант — указать размер выборки в процентах.

```sql
CREATE STATISTICS col1_stats
    ON dbo.table1 (col1)
    WITH SAMPLE = 50 PERCENT;
```

#### <a name="create-single-column-statistics-on-only-some-of-the-rows"></a>Создание одностолбцовой статистики только для некоторых строк

Можно также создать статистику для части строк в таблице. Это называется отфильтрованной статистикой.

Например, отфильтрованную статистику можно использовать при планировании запроса определенной секции большой секционированной таблицы. Создавая статистику только по значениям секции, можно повысить точность статистики. Кроме того, вы можете повысить производительность запросов.

Этот пример создает статистику по диапазону значений. Значения можно легко определить для сопоставления с диапазоном значений в секции.

```sql
CREATE STATISTICS stats_col1
    ON table1(col1)
    WHERE col1 > '2000101' AND col1 < '20001231';
```

> [!NOTE]
> Чтобы оптимизатор запросов рассмотрел возможность использования отфильтрованной статистики, когда выбирает план распределенного запроса, запрос должен быть в пределах определения объекта статистики. Если взять приведенный выше пример, предложение WHERE запроса должно указать значения col1 от 2000101 до 20001231.

#### <a name="create-single-column-statistics-with-all-the-options"></a>Создание одностолбцовой статистики со всеми параметрами

Можно также комбинировать параметры. В приведенном ниже примере создается отфильтрованный объект статистики с настраиваемым размером выборки:

```sql
CREATE STATISTICS stats_col1
    ON table1 (col1)
    WHERE col1 > '2000101' AND col1 < '20001231'
    WITH SAMPLE = 50 PERCENT;
```

Полные справочные сведения см. в статье [CREATE STATISTICS (Transact-SQL)](/sql/t-sql/statements/create-statistics-transact-sql?view=azure-sqldw-latest&preserve-view=true).

#### <a name="create-multi-column-statistics"></a>Создание многостолбцовой статистики

Для создания объекта статистики с несколькими столбцами используйте предыдущие примеры, но укажите больше столбцов.

> [!NOTE]
> Гистограмма, используемая для оценки количества строк в результатах запроса, доступна только для первого столбца, указанного в определении объекта статистики.

В этом примере гистограмма создана для *product\_category*. Статистика между столбцами вычисляется по *product\_category* и *product\_* .

```sql
CREATE STATISTICS stats_2cols
    ON table1 (product_category, product_sub_category)
    WHERE product_category > '2000101' AND product_category < '20001231'
    WITH SAMPLE = 50 PERCENT;
```

Так как между *product\_category* и *product\_sub\_category* существует корреляция, объект статистики с несколькими столбцами может быть полезен, когда к этим столбцам обращаются одновременно.

#### <a name="create-statistics-on-all-columns-in-a-table"></a>Создание статистики для всех столбцов в таблице

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

#### <a name="use-a-stored-procedure-to-create-statistics-on-all-columns-in-a-database"></a>Использование хранимой процедуры для создания статистики для всех столбцов в базе данных

В пуле SQL нет системной хранимой процедуры, эквивалентной sp_create_stats в SQL Server. Эта хранимая процедура создает объект одностолбцовой статистики для каждого столбца базы данных, для которого статистики еще нет.

Следующий пример поможет приступить к проектированию базы данных. Вы можете адаптировать код в соответствии со своими потребностями.

```sql
CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
(   @create_type    tinyint -- 1 default, 2 Fullscan, 3 Sample
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

Чтобы создать статистику для всех столбцов в таблице с помощью полного сканирования, просто вызовите эту процедуру.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 2, NULL;
```

Чтобы создать выборочную статистику для всех столбцов в таблице, введите 3 и процент выборки. Приведенная ниже процедура использует частоту выборки в 20 %.

```sql
EXEC [dbo].[prc_sqldw_create_stats] 3, 20;
```

### <a name="examples-update-statistics"></a>Примеры: Обновите статистику

Чтобы обновить статистику, можно:

- Обновить один объект статистики. Указать имя объекта статистики, который вы хотите обновить.
- Обновить все объекты статистики для таблицы. Указать имя таблицы, а не один объект статистики.

#### <a name="update-one-specific-statistics-object"></a>Обновление одного указанного объекта статистики

Для обновления указанного объекта статистики используйте следующий синтаксис:

```sql
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

Пример:

```sql
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

Обновляя определенные объекты статистики, можно свести к минимуму затраты времени и ресурсов на управление статистикой. Необходимо тщательно подумать, чтобы выбрать наиболее подходящие объекты статистики для обновления.

#### <a name="update-all-statistics-on-a-table"></a>Обновление всей статистики для таблицы

Ниже показан простой метод обновления всех объектов статистики для таблицы.

```sql
UPDATE STATISTICS [schema_name].[table_name];
```

Пример:

```sql
UPDATE STATISTICS dbo.table1;
```

Инструкцию UPDATE STATISTICS легко использовать. Просто помните, что она обновляет *всю* статистику для таблицы и, следовательно, может выполнять больше работы, чем необходимо. 

Если производительность не является проблемой, то это самый простой и действенный способ гарантировать, что статистика актуальна.

> [!NOTE]
> При обновлении всей статистики для таблицы выделенный пул SQL выполняет проверку для выборки таблицы для каждого объекта статистики. Если таблица большого размера и содержит много столбцов и статистики, может оказаться эффективнее обновлять отдельные данные статистики по необходимости.

Реализация процедуры `UPDATE STATISTICS` приведена в разделе о [временных таблицах](develop-tables-temporary.md). Метод реализации слегка отличается от процедуры `CREATE STATISTICS`, описанной выше, но результат одинаков.
Полный синтаксис приведен в разделе об [обновлении статистики](/sql/t-sql/statements/update-statistics-transact-sql?view=azure-sqldw-latest&preserve-view=true).

### <a name="statistics-metadata"></a>Метаданные статистики

Существует несколько системных представлений и функций, которые можно использовать для поиска информации о статистике. Например, можно узнать, устарел ли объект статистики, воспользовавшись функцией STATS_DATE(). Функция STATS_DATE() позволяет просмотреть время последнего создания или обновления статистики.

#### <a name="catalog-views-for-statistics"></a>Представления каталога для статистики

Вот какие системные представления показывают информацию о статистике:

| Представление каталога | Описание |
|:--- |:--- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого столбца. |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого объекта в базе данных. |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждой схемы в базе данных. |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого объекта статистики. |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого столбца в объекте статистики. Ссылается на sys.columns. |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждой таблицы (включая внешние таблицы). |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Одна строка для каждого типа данных. |

#### <a name="system-functions-for-statistics"></a>Системные функции для статистики

Эти системные функции полезны для работы со статистикой:

| Системная функция | Описание |
|:--- |:--- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Дата последнего обновления объекта статистики. |
| [DBCC SHOW_STATISTICS](/sql/t-sql/database-console-commands/dbcc-show-statistics-transact-sql?view=azure-sqldw-latest&preserve-view=true) |Сводная и подробная информация о распределении значений согласно объекту статистики. |

#### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Сочетание столбцов и функций статистики в одном представлении

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
JOIN    sys.tables          AS tb ON    co.[object_id]      = tb.[object_id]
JOIN    sys.schemas         AS sm ON    tb.[schema_id]      = sm.[schema_id]
WHERE   1=1
AND     st.[user_created] = 1
;
```

### <a name="dbcc-show_statistics-examples"></a>Примеры DBCC SHOW_STATISTICS()

DBCC SHOW_STATISTICS() отображает данные, хранящиеся в объекте статистики. Эти данные состоят из трех частей:

- Заголовок
- Вектор плотности
- Гистограмма

Заголовок — это метаданные статистики. Гистограмма отображает распределение значений в первом ключевом столбце объекта статистики. 

Вектор плотности измеряет корреляцию между столбцами. Выделенный пул SQL вычисляет оценку количества элементов с помощью любого из данных в объекте статистики.

#### <a name="show-header-density-and-histogram"></a>Отображение заголовка, плотности и гистограммы

Этот простой пример показывает все три части объекта статистики.

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

Пример:

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

#### <a name="show-one-or-more-parts-of-dbcc-show_statistics"></a>Отображение одной или нескольких частей DBCC SHOW_STATISTICS()

Если вы заинтересованы только в просмотре определенных частей, используйте предложение `WITH` и укажите, какие части требуется показать:

```sql
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
    WITH stat_header, histogram, density_vector
```

Пример:

```sql
DBCC SHOW_STATISTICS (dbo.table1, stats_col1)
    WITH histogram, density_vector
```

### <a name="dbcc-show_statistics-differences"></a>Отличия DBCC SHOW_STATISTICS()

`DBCC SHOW_STATISTICS()` более строго реализуется в выделенном пуле SQL по сравнению с SQL Server:

- Недокументированные возможности не поддерживаются.
- Невозможно использовать Stats_stream.
- Невозможно соединить результаты для определенных подмножеств данных статистики. Например, STAT_HEADER JOIN DENSITY_VECTOR.
- Невозможно задать NO_INFOMSGS для отключения сообщений.
- Невозможно использовать квадратные скобки вокруг имен статистики.
- Невозможно использовать имена столбцов для идентификации объектов статистики.
- Пользовательская ошибка 2767 не поддерживается.


## <a name="statistics-in-serverless-sql-pool"></a>Статистика в бессерверном пуле SQL

Статистика создается по каждому отдельному столбцу в наборе данных (пути к хранилищу).

> [!NOTE]
> Невозможно создать статистику для столбцов LOB.

### <a name="why-use-statistics"></a>Для чего используется статистика?

Более чем серверный пул SQL знает о ваших данных, тем быстрее он может выполнять запросы к ней. Сбор статистики данных — один из самых важных факторов, позволяющих оптимизировать запросы. 

Оптимизатор запросов пула SQL, который не зависит от сервера, является оптимизатором на основе стоимости. Он сравнивает стоимость разных планов запроса, а затем выбирает план с наименьшей стоимостью. В большинстве случаев он выбирает план, который выполняется быстрее всего. 

Например, если оптимизатор вычислит, что дата, указанная в фильтре запроса, вернет одну строку, будет выбран один план. Если он вычислит, что выбранная дата вернет 1 000 000 строк, будет возвращен другой план.

### <a name="automatic-creation-of-statistics"></a>Автоматическое создание статистики

Бессерверный пул SQL анализирует входящие запросы пользователей на отсутствие статистики. Если она отсутствует, оптимизатор запросов создает статистику по отдельным столбцам в предикате запроса или условии соединения, чтобы улучшить оценку кратности для плана запроса.

Инструкция SELECT активирует автоматическое создание статистики.

> [!NOTE]
> Автоматическое создание статистики включено по умолчанию для файлов Parquet. Для CSV-файлов необходимо создавать статистику вручную, пока не будет реализована поддержка автоматического создания статистически для CSV-файлов.

Автоматическое создание статистики выполняется синхронно, поэтому, если статистика для столбцов отсутствует, возможно небольшое замедление выполнения запросов. Время создания статистики для одного столбца зависит от размера выбранных файлов.

### <a name="manual-creation-of-statistics"></a>Ручное создание статистики

Бессерверный пул SQL позволяет создавать статистику вручную. Для CSV-файлов необходимо создавать статистику вручную, так как автоматическое создание статистики для них не включено. 

Инструкции по созданию статистики вручную приведены в следующих примерах.

### <a name="update-statistics"></a>Обновите статистику

Изменение данных в файлах, удаление и добавление файлов приводит к изменению распределения данных и устареванию статистики. В этом случае необходимо обновить статистику.

Несерверный пул SQL автоматически повторно создает статистику, если данные значительно изменяются. Каждый раз, когда статистика создается автоматически, также сохраняется текущее состояние набора данных: пути к файлам, их размер и дата последнего изменения.

Если статистика устарела, она создается заново. Алгоритм обходит данные и сравнивает их с текущим состоянием набора данных. Если объем изменений превышает заданный порог, старая статистика удаляется и создается статистика на основе нового набора данных.

Статистика, созданная вручную, никогда не объявляется устаревшей.

> [!NOTE]
> Автоматическое повторное создание статистики включено по умолчанию для файлов Parquet. Для CSV-файлов необходимо удалять и затем создавать статистику вручную, пока не будет реализована поддержка автоматического создания статистически для CSV-файлов. Ознакомьтесь с приведенными ниже примерами удаления и создания статистики.

Один из первых вопросов, задаваемых при устранении неполадок запроса: **Актуальна ли статистика?**

Если количество строк или распределение значений для столбца значительно изменяется, то *в этот момент* необходимо обновить статистику.

> [!NOTE]
> Если распределение значений для столбца существенно изменилось, то статистику необходимо обновить, независимо от времени ее последнего обновления.

### <a name="implement-statistics-management"></a>Реализация управления статистикой

Вам может потребоваться расширить конвейер данных, чтобы обеспечить обновление статистики при значительном изменении данных из-за добавления, удаления или изменения файлов.

Ниже приведены основные принципы обновления статистики.

- Убедитесь, что для набора данных обновляется по крайней мере один объект статистики. Тогда при обновлении статистики будет обновляться информация о размере (число строк и страниц).
- Сосредоточьтесь на столбцах, участвующих в предложениях WHERE, JOIN, GROUP BY, ORDER BY и DISTINCT.
- Чаще обновляйте столбцы "с возрастающим порядком ключа", например даты транзакций, потому что эти значения не будут включены в гистограмму статистики.
- Реже обновляйте столбцы со статическим распределением.

Дополнительные сведения см. в разделе об [оценке кратности](/sql/relational-databases/performance/cardinality-estimation-sql-server).

### <a name="examples-create-statistics-for-column-in-openrowset-path"></a>Примеры: Создание статистики для столбца по пути OPENROWSET

В примерах ниже показано, как использовать различные параметры для создания статистики. Параметры, которые можно использовать для каждого столбца, зависят от характеристик данных и того, как столбец будет использован в запросах.

> [!NOTE]
> В данный момент можно создать только статистику по одному столбцу.
>
> Для выполнения sp_create_openrowset_statistics и sp_drop_openrowset_statistics требуются следующие разрешения: АДМИНИСТРировать групповые операции или АДМИНИСТРировать групповые операции базы данных.

Приведенная далее хранимая процедура используется для создания статистики.

```sql
sys.sp_create_openrowset_statistics [ @stmt = ] N'statement_text'
```

Аргументы: [ @stmt = ] N'текст_инструкции' — указывает инструкцию Transact-SQL, которая будет возвращать значения столбцов, используемых в статистике. Предложение TABLESAMPLE можно использовать для указания используемых примеров данных. Если предложение TABLESAMPLE не задано, то используется FULLSCAN.

```syntaxsql
<tablesample_clause> ::= TABLESAMPLE ( sample_number PERCENT )
```

> [!NOTE]
> Выборка по CSV-файлам в настоящее время не поддерживается, для CSV-файлов поддерживается только FULLSCAN.

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Создание одностолбцовой статистики путем проверки всех строк

Чтобы создать статистику по столбцу, укажите запрос, возвращающий столбец, для которого требуется статистика.

По умолчанию, если не указано иное, в бессерверном пуле SQL используется 100% данных, предоставленных в наборе данных при создании статистики.

Например, вот как можно создать статистику с параметрами по умолчанию (FULLSCAN) для столбца year набора данных на основе файла population.csv.

```sql
/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_openrowset_statistics N'SELECT year
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/csv/population/population.csv'',
        FORMAT = ''CSV'',
        FIELDTERMINATOR ='','',
        ROWTERMINATOR = ''\n''
    )
WITH (
    [country_code] VARCHAR (5) COLLATE Latin1_General_BIN2,
    [country_name] VARCHAR (100) COLLATE Latin1_General_BIN2,
    [year] smallint,
    [population] bigint
) AS [r]
'
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Создание одностолбцовой статистики с указанием размера выборки

Можно указать размер выборки в процентах.

```sql
/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_openrowset_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
```

### <a name="examples-update-statistics"></a>Примеры: Обновите статистику

Чтобы обновить статистику, необходимо удалить ее и создать заново. Приведенная ниже хранимая процедура используется для удаления статистики.

```sql
sys.sp_drop_openrowset_statistics [ @stmt = ] N'statement_text'
```

> [!NOTE]
> Для выполнения sp_create_openrowset_statistics и sp_drop_openrowset_statistics требуются следующие разрешения: АДМИНИСТРировать групповые операции или АДМИНИСТРировать групповые операции базы данных.

Аргументы: [ @stmt = ] N'текст_инструкции' — задает ту же инструкцию Transact-SQL, которая использовалась при создании статистики.

Чтобы обновить статистику для столбца year в наборе данных, основанном на файле population.csv, необходимо удалить и создать статистику заново.

```sql
EXEC sys.sp_drop_openrowset_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
GO

/* make sure you have credentials for storage account access created
IF EXISTS (SELECT * FROM sys.credentials WHERE name = 'https://azureopendatastorage.blob.core.windows.net/censusdatacontainer')
DROP CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]
GO

CREATE CREDENTIAL [https://azureopendatastorage.blob.core.windows.net/censusdatacontainer]  
WITH IDENTITY='SHARED ACCESS SIGNATURE',  
SECRET = ''
GO
*/

EXEC sys.sp_create_openrowset_statistics N'SELECT payment_type
FROM OPENROWSET(
        BULK ''https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=2018/month=6/*.parquet'',
         FORMAT = ''PARQUET''
    ) AS [nyc]
    TABLESAMPLE(5 PERCENT)
'
```

### <a name="examples-create-statistics-for-external-table-column"></a>Примеры: Создание статистики по столбцу внешней таблицы

В примерах ниже показано, как использовать различные параметры для создания статистики. Параметры, которые можно использовать для каждого столбца, зависят от характеристик данных и того, как столбец будет использован в запросах.

> [!NOTE]
> В данный момент можно создать только статистику по одному столбцу.

Чтобы создать одностолбцовую статистику, достаточно указать имя объекта статистики и имя столбца.

```sql
CREATE STATISTICS statistics_name
ON { external_table } ( column )
    WITH
        { FULLSCAN
          | [ SAMPLE number PERCENT ] }
        , { NORECOMPUTE }
```

Аргументы: external_table — указывает внешнюю таблицу, для которой должна быть создана статистика.

Операция FULLSCAN вычисляет статистику путем сканирования всех строк. FULLSCAN и SAMPLE 100 PERCENT имеют одинаковые результаты. FULLSCAN невозможно использовать с параметром SAMPLE.

SAMPLE <число> PERCENT — указывает приблизительное процентное соотношение или число строк в таблице или индексированном представлении для оптимизатора запросов, которые используются при создании статистики. Число может быть в диапазоне от 0 до 100.

SAMPLE невозможно использовать вместе с параметром FULLSCAN.

> [!NOTE]
> Выборка по CSV-файлам в настоящее время не поддерживается, для CSV-файлов поддерживается только FULLSCAN.

#### <a name="create-single-column-statistics-by-examining-every-row"></a>Создание одностолбцовой статистики путем проверки всех строк

```sql
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH FULLSCAN, NORECOMPUTE
```

#### <a name="create-single-column-statistics-by-specifying-the-sample-size"></a>Создание одностолбцовой статистики с указанием размера выборки

```sql
-- following sample creates statistics with sampling 20%
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH SAMPLE 5 percent, NORECOMPUTE
```

### <a name="examples-update-statistics"></a>Примеры: Обновите статистику

Чтобы обновить статистику, необходимо удалить ее и создать заново. Сначала удалите статистику.

```sql
DROP STATISTICS census_external_table.sState
```

Затем создайте статистику.

```sql
CREATE STATISTICS sState
    on census_external_table (STATENAME)
    WITH FULLSCAN, NORECOMPUTE
```

### <a name="statistics-metadata"></a>Метаданные статистики

Существует несколько системных представлений и функций, которые можно использовать для поиска информации о статистике. Например, можно узнать, устарел ли объект статистики, воспользовавшись функцией STATS_DATE(). Функция STATS_DATE() позволяет просмотреть время последнего создания или обновления статистики.

> [!NOTE]
> Метаданные статистики доступны только для столбцов внешней таблицы. Метаданные статистики недоступны для столбцов OPENROWSET.

#### <a name="catalog-views-for-statistics"></a>Представления каталога для статистики

Вот какие системные представления показывают информацию о статистике:

| Представление каталога                                                 | Описание                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [sys.columns](/sql/relational-databases/system-catalog-views/sys-columns-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Одна строка для каждого столбца.                                     |
| [sys.objects](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Одна строка для каждого объекта в базе данных.                     |
| [sys.schemas](/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Одна строка для каждой схемы в базе данных.                     |
| [sys.stats](/sql/relational-databases/system-catalog-views/sys-stats-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Одна строка для каждого объекта статистики.                          |
| [sys.stats_columns](/sql/relational-databases/system-catalog-views/sys-stats-columns-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Одна строка для каждого столбца в объекте статистики. Ссылается на sys.columns. |
| [sys.tables](/sql/relational-databases/system-catalog-views/sys-tables-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Одна строка для каждой таблицы (включая внешние таблицы).           |
| [sys.table_types](/sql/relational-databases/system-catalog-views/sys-table-types-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Одна строка для каждого типа данных.                                  |

#### <a name="system-functions-for-statistics"></a>Системные функции для статистики

Эти системные функции полезны для работы со статистикой:

| Системная функция                                              | Описание                                  |
| :----------------------------------------------------------- | :------------------------------------------- |
| [STATS_DATE](/sql/t-sql/functions/stats-date-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Дата последнего обновления объекта статистики. |

#### <a name="combine-statistics-columns-and-functions-into-one-view"></a>Сочетание столбцов и функций статистики в одном представлении

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
JOIN    sys.tables          AS tb ON    co.[object_id]      = tb.[object_id]
JOIN    sys.schemas         AS sm ON    tb.[schema_id]      = sm.[schema_id]
WHERE   st.[user_created] = 1
;
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о повышении производительности запросов для выделенного пула SQL см. в статье [мониторинг рабочей нагрузки](../sql-data-warehouse/sql-data-warehouse-manage-monitor.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) и рекомендации [для выделенного пула SQL](best-practices-dedicated-sql-pool.md#maintain-statistics).

Дополнительные сведения о производительности запросов для несерверного пула SQL см. в статье [рекомендации по использованию бессерверного пула SQL](best-practices-serverless-sql-pool.md) .
