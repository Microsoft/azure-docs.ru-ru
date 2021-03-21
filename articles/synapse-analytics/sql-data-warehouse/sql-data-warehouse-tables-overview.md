---
title: Проектирование таблиц
description: Общие сведения о проектировании таблиц с помощью выделенного пула SQL в Azure синапсе Analytics.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/15/2019
ms.author: xiaoyul
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: c55edbd24553189c11070999ddc5d3b3516f2d97
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98737937"
---
# <a name="design-tables-using-dedicated-sql-pool-in-azure-synapse-analytics"></a>Разработка таблиц с помощью выделенного пула SQL в Azure синапсе Analytics

В этой статье приводятся ключевые понятия для разработки таблиц в выделенном пуле SQL.

## <a name="determine-table-category"></a>Определение категории таблицы

При [схеме типа "звезда"](https://en.wikipedia.org/wiki/Star_schema) данные упорядочиваются в таблицы фактов и измерений. Некоторые таблицы используются для интеграции или хранения данных до того, как они перейдут в таблицу фактов или измерений. При конструировании таблицы решите, к какой таблице относятся данные: фактов, измерений или интеграции. Это решение влияет на соответствующую структуру и распределение таблицы.

- **Таблицы фактов** содержат количественные данные, которые обычно создаются в транзакционной системе, а затем загружаются в выделенный пул SQL. Например, розничная компания создает транзакции по продажам каждый день, а затем загружает данные в отдельную таблицу фактов пула SQL для анализа.

- **Таблицы измерений** содержат данные атрибутов, которые могут измениться, хотя обычно это происходит редко. Например, имя и адрес клиента хранятся в таблице измерений и обновляются только при изменении профиля клиента. Чтобы максимально ограничить размер большой таблицы фактов, имя и адрес клиента не обязательно должны находиться в каждой строке таблицы фактов. Вместо этого в таблице фактов и таблице измерений может совместно использоваться идентификатор клиента. Запрос может объединить две таблицы, чтобы связать профиль и транзакции клиента.

- **Таблицы интеграции** служат для интеграции или промежуточного размещения данных. Можно создать таблицу интеграции в виде обычной таблицы, внешней таблицы или временной таблицы. Например, вы можете загружать данные в промежуточную таблицу, выполнять преобразования данных в режиме промежуточного размещения, а затем вставлять их в рабочую таблицу.

## <a name="schema-and-table-names"></a>Имена схем и таблиц

Схемы — это хороший способ группировки таблиц, используемых подобным образом.  При переносе нескольких баз данных из локального решения в выделенный пул SQL лучше всего перенести все таблицы фактов, измерений и интеграции в одну схему в выделенном пуле SQL.

Например, можно хранить все таблицы в выделенном пуле SQL [WideWorldImportersDW](/sql/sample/world-wide-importers/database-catalog-wwi-olap?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) в одной схеме под названием WWI. Следующий код создает [определяемую пользователем схему](/sql/t-sql/statements/create-schema-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) с именем WWI.

```sql
CREATE SCHEMA wwi;
```

Чтобы отобразить организацию таблиц в выделенном пуле SQL, в качестве префиксов для имен таблиц можно использовать факт, Dim и int. В следующей таблице показаны некоторые имена схем и таблиц для WideWorldImportersDW.  

| Таблица WideWorldImportersDW  | Тип таблицы | Выделенный пул SQL |
|:-----|:-----|:------|:-----|
| Город | Измерение | wwi.DimCity |
| Номер | Факты | wwi.FactOrder |

## <a name="table-persistence"></a>Сохраняемость таблицы

Таблицы хранят данные в хранилище Azure постоянно, временно в службе хранилища Azure или в хранилище данных, внешнем по выделенному пулу SQL.

### <a name="regular-table"></a>Обычная таблица

В обычной таблице данные хранятся в службе хранилища Azure как часть выделенного пула SQL. Таблица и данные сохраняются независимо от того, открыт ли сеанс.  В следующем примере создается обычная таблица с двумя столбцами.

```sql
CREATE TABLE MyTable (col1 int, col2 int );  
```

### <a name="temporary-table"></a>Временная таблица

Временная таблица существует, только пока отрыт сеанс. Можно использовать временную таблицу, чтобы другие пользователи не могли просматривать временные результаты, а также сокращать потребность в очистке.  

Для обеспечения высокой производительности временные таблицы используют локальное хранилище.  Дополнительные сведения см. в статье [Временные таблицы в хранилище данных SQL](sql-data-warehouse-tables-temporary.md).

### <a name="external-table"></a>Внешняя таблица

Внешняя таблица указывает на данные, расположенные в большом двоичном объекте службы хранилища Azure или Azure Data Lake Store. При использовании в сочетании с инструкцией CREATE TABLE как SELECT, при выборе из внешней таблицы данные импортируются в выделенный пул SQL.

Таким образом, внешние таблицы удобны для загрузки данных. Руководство по загрузке см. [в статье Загрузка данных из хранилища BLOB-объектов Azure с помощью polybase](./load-data-from-azure-blob-storage-using-copy.md).

## <a name="data-types"></a>Типы данных

Выделенный пул SQL поддерживает наиболее часто используемые типы данных. Список поддерживаемых типов данных представлен в [справочнике по типах данных в инструкции CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json?view=azure-sqldw-latest&preserve-view=true#datatypes). Рекомендации по использованию типов данных см. в статье [Руководство по определению типов данных для таблиц в хранилище данных SQL](sql-data-warehouse-tables-data-types.md).

## <a name="distributed-tables"></a>Распределенные таблицы

Фундаментальным компонентом выделенного пула SQL является способ хранения таблиц в [распределениях](massively-parallel-processing-mpp-architecture.md#distributions)и работы с ними.  Выделенный пул SQL поддерживает три метода распространения данных: циклический перебор (по умолчанию), хэширование и репликация.

### <a name="hash-distributed-tables"></a>Таблицы с хэш-распределением

Распределенная хэш-таблица распределяет строки на основе значения в столбце распределения. Распределенная хэш-таблица предназначена для обеспечения высокой производительности запросов к большим таблицам. При выборе столбца распределения необходимо учитывать несколько факторов.

Дополнительные сведения см. в статье [Руководство по проектированию распределенных таблиц в хранилище данных SQL Azure](sql-data-warehouse-tables-distribute.md).

### <a name="replicated-tables"></a>Реплицированные таблицы

Копия реплицированной таблицы хранится на каждом вычислительном узле. Запросы в реплицированных таблицах выполняются быстро, поскольку для соединений в реплицированных таблицах не требуется перемещение данных. Репликация требует дополнительного хранилища, но непрактично для больших таблиц.

Дополнительные сведения см. в статье [Руководство по проектированию для использования реплицированных таблиц в хранилище данных SQL Azure](design-guidance-for-replicated-tables.md).

### <a name="round-robin-tables"></a>Таблицы с распределением методом циклического перебора

Строки таблицы с распределением методом циклического перебора распределяются равномерно по всем распределениям. Строки распределяются случайным образом. Загрузка данных в таблицу с распределением методом циклического перебора происходит быстро.  Помните, что запросы могут потребовать больше перемещений данных, чем другие методы распределения.

Дополнительные сведения см. в статье [Руководство по проектированию распределенных таблиц в хранилище данных SQL Azure](sql-data-warehouse-tables-distribute.md).

### <a name="common-distribution-methods-for-tables"></a>Общие методы распределения для таблиц

Вариант распределения таблицы часто зависит от категории таблицы.

| Категории таблицы | Рекомендуемый вариант распределения |
|:---------------|:--------------------|
| Факты           | Используйте хэш-распределение с кластеризованным индексом columnstore. Производительность улучшается, когда две хэш-таблицы объединены по одному столбцу распределения. |
| Измерение      | Используйте репликацию для небольших таблиц. Если таблицы слишком велики для хранения на каждом вычислительном узле, используйте хэш-распределение. |
| Промежуточная        | Используйте циклический перебор для промежуточных таблиц. Загрузка с помощью инструкции CTAS выполняется быстро. Когда данные помещается в промежуточную таблицу, используйте инструкцию INSERT... Выберите, чтобы переместить данные в рабочие таблицы. |

## <a name="table-partitions"></a>Разделы таблицы

Секционированная таблица хранит данные и выполняет операции со строками таблицы в соответствии с диапазонами данных. Например, таблицу можно разделить по дню, месяцу или году. Вы можете улучшить производительность запросов путем исключения секций, что ограничивает проверку запросов к данным в секции. Вы также можете управлять данными путем переключения разделов. Поскольку данные в пуле SQL уже распределены, слишком большое количество секций может снизить производительность запросов. Дополнительные сведения см. в статье [Секционирование таблиц в хранилище данных SQL](sql-data-warehouse-tables-partition.md).  При переключении секций в секции таблицы, которые не являются пустыми, рекомендуется использовать параметр TRUNCATE_TARGET в инструкции [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) , если существующие данные должны быть усечены. Приведенный ниже код переключается в преобразованных ежедневных данных в SalesFact перезаписи существующих данных.

```sql
ALTER TABLE SalesFact_DailyFinalLoad SWITCH PARTITION 256 TO SalesFact PARTITION 256 WITH (TRUNCATE_TARGET = ON);  
```

## <a name="columnstore-indexes"></a>Индексы columnstore

По умолчанию выделенный пул SQL хранит таблицу как кластеризованный индекс columnstore. Такая форма хранения данных обеспечивает высокое сжатие данных и производительность запросов в больших таблицах.  

Кластеризованный индекс columnstore обычно является лучшим выбором, но в некоторых случаях подходящей структурой хранения являются кластеризованный индекс или куча.  

> [!TIP]
> Таблица кучи может быть особенно полезной для загрузки временных данных, например промежуточной таблицы, которая преобразуется в окончательную таблицу.

Список функций индексов columnstore см. в статье [Новые возможности индексов columnstore](/sql/relational-databases/indexes/columnstore-indexes-what-s-new?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). Сведения о повышении производительности индекса columnstore см. в статье [Максимальное повышение качества группы строк для индекса columnstore](sql-data-warehouse-memory-optimizations-for-columnstore-compression.md).

## <a name="statistics"></a>Статистика

Оптимизатор запросов использует статистику уровня столбца при создании плана выполнения запроса.

Чтобы повысить производительность запросов, важно иметь статистические данные по отдельным столбцам, особенно столбцы, используемые в объединениях запросов. [Создание статистики](sql-data-warehouse-tables-statistics.md#automatic-creation-of-statistic) происходит автоматически.  

Обновление статистики не происходит автоматически. Обновите статистику после добавления или изменения значительного числа строк. Например, обновите статистику после загрузки. Дополнительные сведения см. в статье [Управление статистикой таблиц в хранилище данных SQL](sql-data-warehouse-tables-statistics.md).

## <a name="primary-key-and-unique-key"></a>Первичный ключ и уникальный ключ

ПЕРВИЧный ключ поддерживается только в том случае, если используются некластеризованные и непринудительные.  Ограничение UNIQUE поддерживается только при использовании не применяется.  Проверьте [выделенные ограничения таблицы пула SQL](sql-data-warehouse-table-constraints.md).

## <a name="commands-for-creating-tables"></a>Команды для создания таблиц

Вы можете создать пустую таблицу. Вы также можете создать и заполнить таблицу результатами инструкции Select. Ниже приведены команды T-SQL для создания таблицы.

| Инструкция Т-SQL | Описание |
|:----------------|:------------|
| [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Создает пустую таблицу, определив все столбцы и параметры таблицы. |
| [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Создает внешнюю таблицу. Определение таблицы хранится в выделенном пуле SQL. Данные таблицы хранятся в хранилище BLOB-объектов Azure или в Azure Data Lake Store. |
| [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Задает новую таблицу с результатами инструкции Select. Столбцы и типы данных таблицы основаны на результатах инструкции Select. Чтобы импортировать данные, эта инструкция может выбрать данные из внешней таблицы. |
| [CREATE EXTERNAL TABLE AS SELECT](/sql/t-sql/statements/create-external-table-as-select-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) | Создает новую внешнюю таблицу, экспортируя результаты инструкции Select во внешнее расположение.  Расположением является либо хранилище BLOB-объектов Azure, либо Azure Data Lake Store. |

## <a name="aligning-source-data-with-dedicated-sql-pool"></a>Согласование исходных данных с выделенным пулом SQL

Выделенные таблицы пула SQL заполняются путем загрузки данных из другого источника данных. Чтобы успешно выполнить загрузку, число и типы данных столбцов в исходных данных должны быть согласованы с определением таблицы в выделенном пуле SQL. Согласование данных может оказаться самой сложной частью разработки таблиц.

Если данные поступают из нескольких хранилищ данных, данные загружаются в выделенный пул SQL и сохраняются в таблице интеграции. После того как данные находятся в таблице интеграции, можно использовать возможности выделенного пула SQL для выполнения операций преобразования. После подготовки данных их можно вставить в рабочие таблицы.

## <a name="unsupported-table-features"></a>Неподдерживаемые функции таблиц

Выделенный пул SQL поддерживает многие, но не все функции таблиц, предлагаемые другими базами данных.  В следующем списке показаны некоторые функции таблиц, которые не поддерживаются в выделенном пуле SQL:

- Внешний ключ, проверка [ограничений таблицы](/sql/t-sql/statements/alter-table-table-constraint-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Вычисляемые столбцы](/sql/t-sql/statements/alter-table-computed-column-definition-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Индексированные представления](/sql/relational-databases/views/create-indexed-views?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Последовательность](/sql/t-sql/statements/create-sequence-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Разреженные столбцы](/sql/relational-databases/tables/use-sparse-columns?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- Суррогатные ключи (реализация посредством [удостоверения](sql-data-warehouse-tables-identity.md));
- [Синонимы](/sql/t-sql/statements/create-synonym-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Триггеры](/sql/t-sql/statements/create-trigger-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Уникальные индексы](/sql/t-sql/statements/create-index-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)
- [Определяемые пользователем типы](/sql/relational-databases/native-client/features/using-user-defined-types?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

## <a name="table-size-queries"></a>Запросы размера таблицы

Простой способ определить пространство и строки, используемые таблицей в каждом из 60 распределений, — применить инструкцию [DBCC PDW_SHOWSPACEUSED](/sql/t-sql/database-console-commands/dbcc-pdw-showspaceused-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

```sql
DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');
```

Однако возможности команд DBCC достаточно ограничены.  Динамические административные представления (DMV) отображают больше сведений, чем команды DBCC. Начните с создания этого представления:

```sql
CREATE VIEW dbo.vTableSizes
AS
WITH base
AS
(
SELECT
 GETDATE()                                                             AS  [execution_time]
, DB_NAME()                                                            AS  [database_name]
, s.name                                                               AS  [schema_name]
, t.name                                                               AS  [table_name]
, QUOTENAME(s.name)+'.'+QUOTENAME(t.name)                              AS  [two_part_name]
, nt.[name]                                                            AS  [node_table_name]
, ROW_NUMBER() OVER(PARTITION BY nt.[name] ORDER BY (SELECT NULL))     AS  [node_table_name_seq]
, tp.[distribution_policy_desc]                                        AS  [distribution_policy_name]
, c.[name]                                                             AS  [distribution_column]
, nt.[distribution_id]                                                 AS  [distribution_id]
, i.[type]                                                             AS  [index_type]
, i.[type_desc]                                                        AS  [index_type_desc]
, nt.[pdw_node_id]                                                     AS  [pdw_node_id]
, pn.[type]                                                            AS  [pdw_node_type]
, pn.[name]                                                            AS  [pdw_node_name]
, di.name                                                              AS  [dist_name]
, di.position                                                          AS  [dist_position]
, nps.[partition_number]                                               AS  [partition_nmbr]
, nps.[reserved_page_count]                                            AS  [reserved_space_page_count]
, nps.[reserved_page_count] - nps.[used_page_count]                    AS  [unused_space_page_count]
, nps.[in_row_data_page_count]
    + nps.[row_overflow_used_page_count]
    + nps.[lob_used_page_count]                                        AS  [data_space_page_count]
, nps.[reserved_page_count]
 - (nps.[reserved_page_count] - nps.[used_page_count])
 - ([in_row_data_page_count]
         + [row_overflow_used_page_count]+[lob_used_page_count])       AS  [index_space_page_count]
, nps.[row_count]                                                      AS  [row_count]
from
    sys.schemas s
INNER JOIN sys.tables t
    ON s.[schema_id] = t.[schema_id]
INNER JOIN sys.indexes i
    ON  t.[object_id] = i.[object_id]
    AND i.[index_id] <= 1
INNER JOIN sys.pdw_table_distribution_properties tp
    ON t.[object_id] = tp.[object_id]
INNER JOIN sys.pdw_table_mappings tm
    ON t.[object_id] = tm.[object_id]
INNER JOIN sys.pdw_nodes_tables nt
    ON tm.[physical_name] = nt.[name]
INNER JOIN sys.dm_pdw_nodes pn
    ON  nt.[pdw_node_id] = pn.[pdw_node_id]
INNER JOIN sys.pdw_distributions di
    ON  nt.[distribution_id] = di.[distribution_id]
INNER JOIN sys.dm_pdw_nodes_db_partition_stats nps
    ON nt.[object_id] = nps.[object_id]
    AND nt.[pdw_node_id] = nps.[pdw_node_id]
    AND nt.[distribution_id] = nps.[distribution_id]
LEFT OUTER JOIN (select * from sys.pdw_column_distribution_properties where distribution_ordinal = 1) cdp
    ON t.[object_id] = cdp.[object_id]
LEFT OUTER JOIN sys.columns c
    ON cdp.[object_id] = c.[object_id]
    AND cdp.[column_id] = c.[column_id]
WHERE pn.[type] = 'COMPUTE'
)
, size
AS
(
SELECT
   [execution_time]
,  [database_name]
,  [schema_name]
,  [table_name]
,  [two_part_name]
,  [node_table_name]
,  [node_table_name_seq]
,  [distribution_policy_name]
,  [distribution_column]
,  [distribution_id]
,  [index_type]
,  [index_type_desc]
,  [pdw_node_id]
,  [pdw_node_type]
,  [pdw_node_name]
,  [dist_name]
,  [dist_position]
,  [partition_nmbr]
,  [reserved_space_page_count]
,  [unused_space_page_count]
,  [data_space_page_count]
,  [index_space_page_count]
,  [row_count]
,  ([reserved_space_page_count] * 8.0)                                 AS [reserved_space_KB]
,  ([reserved_space_page_count] * 8.0)/1000                            AS [reserved_space_MB]
,  ([reserved_space_page_count] * 8.0)/1000000                         AS [reserved_space_GB]
,  ([reserved_space_page_count] * 8.0)/1000000000                      AS [reserved_space_TB]
,  ([unused_space_page_count]   * 8.0)                                 AS [unused_space_KB]
,  ([unused_space_page_count]   * 8.0)/1000                            AS [unused_space_MB]
,  ([unused_space_page_count]   * 8.0)/1000000                         AS [unused_space_GB]
,  ([unused_space_page_count]   * 8.0)/1000000000                      AS [unused_space_TB]
,  ([data_space_page_count]     * 8.0)                                 AS [data_space_KB]
,  ([data_space_page_count]     * 8.0)/1000                            AS [data_space_MB]
,  ([data_space_page_count]     * 8.0)/1000000                         AS [data_space_GB]
,  ([data_space_page_count]     * 8.0)/1000000000                      AS [data_space_TB]
,  ([index_space_page_count]  * 8.0)                                   AS [index_space_KB]
,  ([index_space_page_count]  * 8.0)/1000                              AS [index_space_MB]
,  ([index_space_page_count]  * 8.0)/1000000                           AS [index_space_GB]
,  ([index_space_page_count]  * 8.0)/1000000000                        AS [index_space_TB]
FROM base
)
SELECT *
FROM size
;
```

### <a name="table-space-summary"></a>Сводка табличного пространства

Этот запрос возвращает количество строк и объем каждой таблицы.  Он позволяет видеть, какие таблицы являются наиболее крупными таблицами, а также осуществляется ли их циклический перебор, репликация или распределение по хэшу.  Для таблиц с хэш-распределением запрос показывает столбец распределения.  

```sql
SELECT
     database_name
,    schema_name
,    table_name
,    distribution_policy_name
,      distribution_column
,    index_type_desc
,    COUNT(distinct partition_nmbr) as nbr_partitions
,    SUM(row_count)                 as table_row_count
,    SUM(reserved_space_GB)         as table_reserved_space_GB
,    SUM(data_space_GB)             as table_data_space_GB
,    SUM(index_space_GB)            as table_index_space_GB
,    SUM(unused_space_GB)           as table_unused_space_GB
FROM
    dbo.vTableSizes
GROUP BY
     database_name
,    schema_name
,    table_name
,    distribution_policy_name
,      distribution_column
,    index_type_desc
ORDER BY
    table_reserved_space_GB desc
;
```

### <a name="table-space-by-distribution-type"></a>Табличное пространство по типу распределения

```sql
SELECT
     distribution_policy_name
,    SUM(row_count)                as table_type_row_count
,    SUM(reserved_space_GB)        as table_type_reserved_space_GB
,    SUM(data_space_GB)            as table_type_data_space_GB
,    SUM(index_space_GB)           as table_type_index_space_GB
,    SUM(unused_space_GB)          as table_type_unused_space_GB
FROM dbo.vTableSizes
GROUP BY distribution_policy_name
;
```

### <a name="table-space-by-index-type"></a>Табличное пространство по типу индекса

```sql
SELECT
     index_type_desc
,    SUM(row_count)                as table_type_row_count
,    SUM(reserved_space_GB)        as table_type_reserved_space_GB
,    SUM(data_space_GB)            as table_type_data_space_GB
,    SUM(index_space_GB)           as table_type_index_space_GB
,    SUM(unused_space_GB)          as table_type_unused_space_GB
FROM dbo.vTableSizes
GROUP BY index_type_desc
;
```

### <a name="distribution-space-summary"></a>Сводка пространства распределения

```sql
SELECT
    distribution_id
,    SUM(row_count)                as total_node_distribution_row_count
,    SUM(reserved_space_MB)        as total_node_distribution_reserved_space_MB
,    SUM(data_space_MB)            as total_node_distribution_data_space_MB
,    SUM(index_space_MB)           as total_node_distribution_index_space_MB
,    SUM(unused_space_MB)          as total_node_distribution_unused_space_MB
FROM dbo.vTableSizes
GROUP BY     distribution_id
ORDER BY    distribution_id
;
```

## <a name="next-steps"></a>Дальнейшие действия

После создания таблиц для выделенного пула SQL следующим шагом является загрузка данных в таблицу.  Руководство по загрузке см. в разделе [Загрузка данных в выделенный пул SQL](load-data-wideworldimportersdw.md).