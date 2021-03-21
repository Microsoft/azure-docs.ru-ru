---
title: Разработка таблиц с помощью синапсе SQL
description: Общие сведения о проектировании таблиц в синапсе SQL.
services: synapse-analytics
author: filippopovic
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 27cc53c3eef1bb2a9962d2c21ae80db3c8b0383d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104585440"
---
# <a name="design-tables-using-synapse-sql-in-azure-synapse-analytics"></a>Разработка таблиц с помощью синапсе SQL в Azure синапсе Analytics

Этот документ содержит основные понятия для проектирования таблиц с выделенным пулом SQL и бессерверным пулом SQL Server.  

[Бессерверный пул SQL](on-demand-workspace-overview.md) — это служба запросов к данным в Data Lake. Он не имеет локального хранилища для приема данных. [Выделенный пул SQL](best-practices-dedicated-sql-pool.md) представляет коллекцию аналитических ресурсов, которые подготавливаются при использовании синапсе SQL. Размер выделенного пула SQL определяется единицами использования хранилища данных (DWU).

В следующей таблице перечислены разделы, относящиеся к выделенному пулу SQL и бессерверному пулу SQL:

| Раздел                                                        | выделенный пул SQL | Бессерверный пул SQL |
| ------------------------------------------------------------ | ------------------ | ----------------------- |
| [Определение категории таблицы](#determine-table-category)        | Да                | Нет                      |
| [имена схем;](#schema-names)                                | Да                | Да                     |
| [Имена таблиц](#table-names)                                  | Да                | Нет                      |
| [Сохраняемость таблицы](#table-persistence)                      | Да                | Нет                      |
| [Обычная таблица](#regular-table)                              | Да                | Нет                      |
| [Временная таблица](#temporary-table)                          | Да                | Да                     |
| [Внешняя таблица](#external-table)                            | Да                | Да                     |
| [Типы данных](#data-types)                                    | Да                | Да                     |
| [Распределенные таблицы](#distributed-tables)                    | Да                | Нет                      |
| [Таблицы с хэш-распределением](#hash-distributed-tables)          | Да                | Нет                      |
| [Реплицированные таблицы](#replicated-tables)                      | Да                | Нет                      |
| [Таблицы с распределением методом циклического перебора](#round-robin-tables)                    | Да                | Нет                      |
| [Общие методы распределения для таблиц](#common-distribution-methods-for-tables) | Да                | Нет                      |
| [Секции](#partitions)                                    | Да                | Да                     |
| [Индексы columnstore](#columnstore-indexes)                  | Да                | Нет                      |
| [Статистика](#statistics)                                    | Да                | Да                     |
| [Первичный ключ и уникальный ключ](#primary-key-and-unique-key)    | Да                | Нет                      |
| [Команды для создания таблиц](#commands-for-creating-tables) | Да                | Нет                      |
| [Согласование исходных данных с хранилищем данных](#align-source-data-with-the-data-warehouse) | Да                | Нет                      |
| [Неподдерживаемые функции таблиц](#unsupported-table-features)    | Да                | Нет                      |
| [Запросы размера таблицы](#table-size-queries)                    | Да                | Нет                      |

## <a name="determine-table-category"></a>Определение категории таблицы

При [схеме типа "звезда"](https://en.wikipedia.org/wiki/Star_schema) данные упорядочиваются в таблицы фактов и измерений. Некоторые таблицы используются для интеграции или промежуточного хранения данных перед переходом в таблицу фактов или измерений. При конструировании таблицы решите, к какой таблице относятся данные: фактов, измерений или интеграции. Это решение влияет на соответствующую структуру и распределение таблицы.

- **Таблицы фактов** содержат количественные данные, которые обычно создаются в транзакционной системе, а затем загружаются в хранилище данных. Например, розничное предприятие ежедневно создает транзакции продаж, а затем загружает данные в таблицу фактов хранилища данных для анализа.

- **Таблицы измерений** содержат данные атрибутов, которые могут измениться, хотя обычно это происходит редко. Например, имя и адрес клиента хранятся в таблице измерений и обновляются только при изменении профиля клиента. Чтобы максимально ограничить размер большой таблицы фактов, имя и адрес клиента не обязательно должны находиться в каждой строке таблицы фактов. Вместо этого в таблице фактов и таблице измерений может совместно использоваться идентификатор клиента. Запрос может объединить две таблицы, чтобы связать профиль и транзакции клиента.

- **Таблицы интеграции** служат для интеграции или промежуточного размещения данных. Можно создать таблицу интеграции в виде обычной таблицы, внешней таблицы или временной таблицы. Например, вы можете загружать данные в промежуточную таблицу, выполнять преобразования данных в режиме промежуточного размещения, а затем вставлять их в рабочую таблицу.

## <a name="schema-names"></a>имена схем;

Схемы — это хороший способ группирования объектов, которые используются аналогичным образом. Следующий код создает [определяемую пользователем схему](/sql/t-sql/statements/create-schema-transact-sql?view=azure-sqldw-latest&preserve-view=true) с именем WWI.

```sql
CREATE SCHEMA wwi;
```

## <a name="table-names"></a>Имена таблиц

При переносе нескольких баз данных из локального решения в выделенный пул SQL рекомендуется перенести все таблицы фактов, измерений и интеграции в одну схему пула SQL. Например, можно хранить все таблицы в образце хранилища данных [WideWorldImportersDW](/sql/samples/wide-world-importers-dw-database-catalog?view=azure-sqldw-latest&preserve-view=true) в одной схеме с именем WWI.

Чтобы отобразить организацию таблиц в выделенном пуле SQL, в качестве префиксов для имен таблиц можно использовать факт, Dim и int. В таблице ниже показаны некоторые имена схем и таблиц для WideWorldImportersDW.  

| Таблица WideWorldImportersDW  | Тип таблицы | выделенный пул SQL |
|:-----|:-----|:------|:-----|
| City | Измерение | wwi.DimCity |
| Номер | Факты | wwi.FactOrder |

## <a name="table-persistence"></a>Сохраняемость таблицы

Таблицы хранят данные постоянно в службе хранилища Azure, временно в службе хранилища Azure или в хранилище данных, внешнем по отношению к хранилищу данных.

### <a name="regular-table"></a>Обычная таблица

Данные обычной таблицы хранятся в службе хранилища Azure как часть хранилища данных. Таблица и данные сохраняются независимо от того, открыт ли сеанс.  В приведенном ниже примере создается обычная таблица с двумя столбцами.

```sql
CREATE TABLE MyTable (col1 int, col2 int );  
```

### <a name="temporary-table"></a>Временная таблица

Временная таблица существует, только пока отрыт сеанс. Можно использовать временную таблицу, чтобы другие пользователи не могли просматривать временные результаты. Использование временных таблиц также сокращает потребность в очистке.  Временные таблицы используют локальное хранилище, а в выделенных пулах SQL могут обеспечить более высокую производительность.  

Бессерверный пул SQL поддерживает временные таблицы. Однако его использование ограничено, так как вы можете выбрать из временной таблицы, но не можете присоединить ее к файлам в хранилище.

Дополнительные сведения см. в статье [Временные таблицы в хранилище данных SQL](develop-tables-temporary.md).

### <a name="external-table"></a>Внешняя таблица

[Внешние таблицы](develop-tables-external-tables.md) указывают на данные, расположенные в хранилище BLOB-объектов Azure или Azure Data Lake Storage.

Импорт данных из внешних таблиц в выделенные пулы SQL с помощью [CREATE TABLE в качестве инструкции SELECT](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) . Руководство по загрузке см. [в статье Загрузка данных из хранилища BLOB-объектов Azure с помощью polybase](../sql-data-warehouse/load-data-from-azure-blob-storage-using-copy.md?bc=%2fazure%2fsynapse-analytics%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fsynapse-analytics%2ftoc.json).

Для несерверного пула SQL можно использовать [CETAS](develop-tables-cetas.md) , чтобы сохранить результат запроса во внешней таблице в службе хранилища Azure.

## <a name="data-types"></a>Типы данных

Выделенный пул SQL поддерживает наиболее часто используемые типы данных. Список поддерживаемых типов данных представлен в [справочнике по типах данных в инструкции CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=azure-sqldw-latest#DataTypes&preserve-view=true). Дополнительные сведения об использовании типов данных см. в разделе [типы данных](../sql/develop-tables-data-types.md).

## <a name="distributed-tables"></a>Распределенные таблицы

Фундаментальным компонентом выделенного пула SQL является способ хранения таблиц в [распределениях](../sql-data-warehouse/massively-parallel-processing-mpp-architecture.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#distributions)и работы с ними.  Выделенный пул SQL поддерживает три метода распространения данных:

- Циклический перебор (по умолчанию)
- Хэш
- Реплицированный

### <a name="hash-distributed-tables"></a>Таблицы с хэш-распределением

Распределенная хэш-таблица распределяет строки на основе значения в столбце распределения. Распределенная хэш-таблица предназначена для обеспечения высокой производительности запросов к большим таблицам. При выборе столбца распределения необходимо учитывать несколько факторов.

Дополнительные сведения см. в статье [Руководство по проектированию распределенных таблиц в хранилище данных SQL Azure](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="replicated-tables"></a>Реплицированные таблицы

Копия реплицированной таблицы хранится на каждом вычислительном узле. Запросы в реплицированных таблицах выполняются быстро, поскольку для соединений с реплицируемыми таблицами не требуется перемещение данных. Репликация требует дополнительного хранилища, но непрактично для больших таблиц.

Дополнительные сведения см. в статье [Руководство по проектированию для использования реплицированных таблиц в хранилище данных SQL Azure](../sql-data-warehouse/design-guidance-for-replicated-tables.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="round-robin-tables"></a>Таблицы с распределением методом циклического перебора

Строки таблицы с распределением методом циклического перебора распределяются равномерно по всем распределениям. Строки распределяются случайным образом. Загрузка данных в таблицу с распределением методом циклического перебора происходит быстро.  Однако запросы могут потребовать больше перемещений данных, чем другие методы распределения.

Дополнительные сведения см. в статье [Руководство по проектированию распределенных таблиц в хранилище данных SQL Azure](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

### <a name="common-distribution-methods-for-tables"></a>Общие методы распределения для таблиц

Категория таблицы часто определяет оптимальный вариант распределения таблиц.

| Категории таблицы | Рекомендуемый вариант распределения |
|:---------------|:--------------------|
| Факты           | Используйте хэш-распределение с кластеризованным индексом columnstore. Производительность улучшается, когда две хэш-таблицы объединены по одному столбцу распределения. |
| Измерение      | Используйте репликацию для небольших таблиц. Если таблицы слишком велики для хранения на каждом вычислительном узле, используйте хэш-распределение. |
| Промежуточная        | Используйте циклический перебор для промежуточных таблиц. Загрузка с помощью инструкции CTAS выполняется быстро. Когда данные помещается в промежуточную таблицу, используйте инструкцию INSERT... Выберите, чтобы переместить данные в рабочие таблицы. |

## <a name="partitions"></a>Секции

В выделенных пулах SQL секционированная таблица хранит и выполняет операции над строками таблицы в соответствии с диапазонами данных. Например, таблицу можно разделить по дню, месяцу или году. Вы можете улучшить производительность запросов путем исключения секций, что ограничивает проверку запросов к данным в секции.

Вы также можете управлять данными путем переключения разделов. Поскольку данные в выделенном пуле SQL уже распределены, слишком большое количество секций может снизить производительность запросов. Дополнительные сведения см. в статье [Секционирование таблиц в хранилище данных SQL](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).  

> [!TIP]
> При переключении секций в секции таблицы, которые не являются пустыми, рекомендуется использовать параметр TRUNCATE_TARGET в инструкции [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?view=azure-sqldw-latest&preserve-view=true) , если существующие данные должны быть усечены.

Приведенный ниже код переключает преобразованные ежедневные данные в секцию SalesFact и перезаписывает существующие данные.

```sql
ALTER TABLE SalesFact_DailyFinalLoad SWITCH PARTITION 256 TO SalesFact PARTITION 256 WITH (TRUNCATE_TARGET = ON);  
```

В бессерверном пуле SQL можно ограничить файлы и папки (разделы), которые будут считываться запросом. Секционирование по пути поддерживается с помощью функций FilePath и FileInfo, описанных в разделе [запросы к файлам хранилища](develop-storage-files-overview.md). В следующем примере считывается папка с данными за год 2017.

```sql
SELECT
    nyc.filepath(1) AS [year],
    payment_type,
    SUM(fare_amount) AS fare_total
FROM  
    OPENROWSET(
        BULK 'https://sqlondemandstorage.blob.core.windows.net/parquet/taxi/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS nyc
WHERE
    nyc.filepath(1) = 2017
GROUP BY
    nyc.filepath(1),
    payment_type
ORDER BY
    nyc.filepath(1),
    payment_type
```

## <a name="columnstore-indexes"></a>Индексы columnstore

По умолчанию выделенный пул SQL хранит таблицу как кластеризованный индекс columnstore. Такая форма хранения данных обеспечивает высокое сжатие данных и производительность запросов в больших таблицах.  Кластеризованный индекс columnstore обычно является лучшим выбором, но в некоторых случаях подходящей структурой хранения являются кластеризованный индекс или куча.  

> [!TIP]
> Таблица кучи может быть особенно полезной для загрузки временных данных, таких как промежуточная таблица, которая преобразуется в окончательную таблицу.

Список функций индексов columnstore см. в статье [Новые возможности индексов columnstore](/sql/relational-databases/indexes/columnstore-indexes-what-s-new?view=azure-sqldw-latest&preserve-view=true). Сведения о повышении производительности индекса columnstore см. в статье [Максимальное повышение качества группы строк для индекса columnstore](../sql/data-load-columnstore-compression.md).

## <a name="statistics"></a>Статистика

Оптимизатор запросов использует статистику уровня столбца при создании плана выполнения запроса. Чтобы повысить производительность запросов, важно иметь статистические данные по отдельным столбцам, особенно столбцы, используемые в объединениях запросов. Синапсе SQL поддерживает автоматическое создание статистики. 

Статистическое обновление не происходит автоматически. Обновите статистику после добавления или изменения значительного числа строк. Например, обновление статистики после загрузки. Дополнительные сведения приведены в статье [руководство по статистике](develop-tables-statistics.md) .

## <a name="primary-key-and-unique-key"></a>Первичный ключ и уникальный ключ

Для выделенного пула SQL ПЕРВИЧный ключ поддерживается только в том случае, если используются некластеризованные и непринудительные.  Ограничение UNIQUE поддерживается только в том случае, если используется не ПРИНУДИТЕЛЬно.  Дополнительные сведения см. в статье [ограничения таблицы пула SQL](../sql-data-warehouse/sql-data-warehouse-table-constraints.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) .

## <a name="commands-for-creating-tables"></a>Команды для создания таблиц

Для выделенного пула SQL можно создать таблицу как новую пустую таблицу. Вы также можете создать и заполнить таблицу результатами инструкции Select. Ниже приведены команды T-SQL для создания таблицы.

| Инструкция Т-SQL | Описание |
|:----------------|:------------|
| [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) | Создает пустую таблицу, определив все столбцы и параметры таблицы. |
| [CREATE EXTERNAL TABLE](/sql/t-sql/statements/create-external-table-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Создает внешнюю таблицу. Определение таблицы хранится в выделенном пуле SQL. Данные таблицы хранятся в хранилище BLOB-объектов Azure или Azure Data Lake Storage. |
| [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) | Задает новую таблицу с результатами инструкции Select. Столбцы и типы данных таблицы основаны на результатах инструкции Select. Чтобы импортировать данные, эта инструкция может выбрать данные из внешней таблицы. |
| [CREATE EXTERNAL TABLE AS SELECT](/sql/t-sql/statements/create-external-table-as-select-transact-sql?view=azure-sqldw-latest&preserve-view=true) | Создает новую внешнюю таблицу, экспортируя результаты инструкции Select во внешнее расположение.  Расположением может быть хранилище BLOB-объектов Azure или Azure Data Lake Storage. |

## <a name="align-source-data-with-the-data-warehouse"></a>Сопоставление исходных данных с хранилищем данных

Выделенные таблицы пула SQL заполняются путем загрузки данных из другого источника данных. Чтобы добиться успешной загрузки, число и типы данных столбцов в исходных данных должны быть согласованы с определением таблицы в хранилище данных.

> [!NOTE]
> Согласование данных может оказаться самой сложной частью разработки таблиц.

Если данные поступают из нескольких хранилищ данных, можно перенести данные в хранилище данных и сохранить их в таблице интеграции. После того как данные находятся в таблице интеграции, можно использовать возможности выделенного пула SQL для реализации операций преобразования. После подготовки данных их можно вставить в рабочие таблицы.

## <a name="unsupported-table-features"></a>Неподдерживаемые функции таблиц

Выделенный пул SQL поддерживает многие, но не все функции таблиц, предлагаемые другими базами данных.  В следующем списке показаны некоторые функции таблиц, которые не поддерживаются в выделенном пуле SQL.

- Внешний ключ, проверка [ограничений таблицы](/sql/t-sql/statements/alter-table-table-constraint-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [Вычисляемые столбцы](/sql/t-sql/statements/alter-table-computed-column-definition-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [Индексированные представления](/sql/relational-databases/views/create-indexed-views?view=azure-sqldw-latest&preserve-view=true)
- [Последовательность](/sql/t-sql/statements/create-sequence-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [Разреженные столбцы](/sql/relational-databases/tables/use-sparse-columns?view=azure-sqldw-latest&preserve-view=true)
- Суррогатные ключи, реализация с помощью [Identity](../sql-data-warehouse/sql-data-warehouse-tables-identity.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Синонимы](/sql/t-sql/statements/create-synonym-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [Триггеры](/sql/t-sql/statements/create-trigger-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [Уникальные индексы](/sql/t-sql/statements/create-index-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [Определяемые пользователем типы](/sql/relational-databases/native-client/features/using-user-defined-types?view=azure-sqldw-latest&preserve-view=true)

## <a name="table-size-queries"></a>Запросы размера таблицы

В выделенном пуле SQL один простой способ выделить пространство и строки, используемые таблицей в каждом распределении 60, — использовать [DBCC PDW_SHOWSPACEUSED](/sql/t-sql/database-console-commands/dbcc-pdw-showspaceused-transact-sql?view=azure-sqldw-latest&preserve-view=true).

```sql
DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');
```

Помните, что использование команд DBCC может быть достаточно ограничено.  Динамические административные представления (DMV) отображают больше сведений, чем команды DBCC. Начните с создания представления ниже.

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

Этот запрос возвращает количество строк и объем каждой таблицы.  Сводка по табличному пространству позволяет увидеть, какие таблицы являются наиболее крупными таблицами. Вы также узнаете, будут ли они циклически перераспределяться, реплицированы или распределены по хэшу.  Для таблиц с хэш-распределением запрос показывает столбец распределения.  

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

После создания таблицы для хранилища данных загрузите данные в таблицу.  Руководство по загрузке см. [в разделе Загрузка данных в выделенный пул SQL](../sql-data-warehouse/load-data-wideworldimportersdw.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#load-the-data-into-sql-pool).