---
title: Индексирование таблиц
description: Рекомендации и примеры индексирования таблиц в выделенном пуле SQL.
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
ms.openlocfilehash: fabbdf330d43737ffa85379f9cc4d5ac59c4a734
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98673524"
---
# <a name="indexing-dedicated-sql-pool-tables-in-azure-synapse-analytics"></a>Индексирование выделенных таблиц пула SQL в Azure синапсе Analytics

Рекомендации и примеры индексирования таблиц в выделенном пуле SQL.

## <a name="index-types"></a>Типы индексов

Выделенный пул SQL предлагает несколько вариантов индексирования, включая [кластеризованные индексы columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), [кластеризованные индексы и некластеризованные индексы](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), а также параметр, не являющийся индексом, известный как [куча](/sql/relational-databases/indexes/heaps-tables-without-clustered-indexes?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  

Чтобы создать таблицу с индексом, см. документацию по [CREATE TABLE (выделенному пулу SQL)](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) .

## <a name="clustered-columnstore-indexes"></a>Кластеризованные индексы Columnstore

По умолчанию выделенный пул SQL создает кластеризованный индекс columnstore, если для таблицы не заданы параметры индекса. Кластеризованные таблицы Columnstore обеспечивают максимальную производительность запросов, а также самый высокий уровень сжатия данных.  Обычно эти таблицы более эффективны, чем таблицы с кластеризованными индексами и таблицы без кластеризованных индексов, и их рекомендуется использовать в больших таблицах.  Таким образом, если вы не уверены, какой метод индексации применить к вашей таблице, используйте кластеризованные индексы Columnstore.  

Чтобы создать таблицу с кластеризованным индексом Columnstore, опустите предложение WITH или укажите в нем параметр CLUSTERED COLUMNSTORE INDEX.

```SQL
CREATE TABLE myTable
  (  
    id int NOT NULL,  
    lastName varchar(20),  
    zipCode varchar(6)  
  )  
WITH ( CLUSTERED COLUMNSTORE INDEX );
```

Есть несколько сценариев, в которых не рекомендуется использовать кластеризованный индекс Columnstore.

- Таблицы Columnstore не поддерживают типы данных VARCHAR(MAX), NVARCHAR(MAX) и VARBINARY(MAX). В этом случае рекомендуется использовать кучу или кластеризованный индекс.
- Таблицы Columnstore менее эффективны для хранения временных данных. В этом случае рекомендуется использовать таблицы без кластеризованных индексов и, возможно, даже временные таблицы.
- Небольшие таблицы, содержащие менее 60 000 000 строк. В этом случае рекомендуется использовать таблицы без кластеризованных индексов.

## <a name="heap-tables"></a>Таблицы без кластеризованных индексов

При временном размещении данных в выделенном пуле SQL может оказаться, что использование таблицы кучи позволяет ускорить общий процесс. Это связано с тем, что загрузка в кучи выполняется быстрее, чем в таблицы с кластеризованными индексами, и в некоторых случаях из кэша можно выполнять последующее считывание.  Загрузка таблицы в таблицу без кластеризованных индексов перед выполнением преобразования данных выполняется намного быстрее, чем загрузка данных в таблицу с кластеризованными индексами Columnstore. Кроме того, загрузка данных во [временную таблицу](sql-data-warehouse-tables-temporary.md) выполняется быстрее, чем загрузка таблицы в постоянное хранилище.  После загрузки данных можно создавать индексы в таблице для повышения производительности запросов.  

Таблицы в кластере columnstore начинают оптимальное сжатие, когда количество строк превышает 60 000 000.  Для небольших таблиц уточняющих запросов, менее 60 000 000 строк, рассмотрите возможность использования КУЧИ или кластеризованного индекса для повышения производительности запросов. 

Чтобы создать таблицу без кластеризованных индексов, укажите в предложении WITH параметр HEAP.

```SQL
CREATE TABLE myTable
  (  
    id int NOT NULL,  
    lastName varchar(20),  
    zipCode varchar(6)  
  )  
WITH ( HEAP );
```

## <a name="clustered-and-nonclustered-indexes"></a>Кластеризованные и некластеризованные индексы

Кластеризованные индексы могут быть более эффективны, когда требуется извлечь отдельную строку. Для запросов, в которых требуется один или несколько уточняющих запросов строки для выполнения с максимальной скоростью, рассмотрим индекс кластера или некластеризованный вторичный индекс. Недостаток использования кластеризованного индекса заключается в том, что эффективными будут только те запросы, в которых используется высокоизбирательный фильтр для столбца с кластеризованным индексом. Чтобы улучшить фильтр в других столбцах, добавьте в них некластеризованный индекс. Однако каждый добавленный в таблицу индекс увеличивает пространство и время обработки загрузки.

Чтобы создать таблицу с кластеризованным индексом, укажите в предложении WITH параметр CLUSTERED INDEX.

```SQL
CREATE TABLE myTable
  (  
    id int NOT NULL,  
    lastName varchar(20),  
    zipCode varchar(6)  
  )  
WITH ( CLUSTERED INDEX (id) );
```

Чтобы добавить некластеризованный индекс для таблицы, используйте следующий синтаксис:

```SQL
CREATE INDEX zipCodeIndex ON myTable (zipCode);
```

## <a name="optimizing-clustered-columnstore-indexes"></a>Оптимизация кластеризованных индексов Columnstore

В кластеризованных таблицах Columnstore данные упорядочены по сегментам.  Качество сегментов существенно влияет на эффективность выполнения запросов в таблице Columnstore.  Его можно определить по числу строк в сжатой группе строк.  Наиболее оптимальное качество сегментов достигается в таблицах, где на сжатую группу строк приходится по крайней мере 100 тысяч строк, а производительность увеличивается при достижении максимального количества строк на группу — 1 048 576.

Для вычисления среднего количества строк для каждой группы строк и определения неоптимальных кластеризованных индексов Columnstore можно создать и использовать представление, подобное приведенному ниже.  Последний столбец в этом представлении создает инструкцию SQL, которую можно использовать для перестроения индексов.

```sql
CREATE VIEW dbo.vColumnstoreDensity
AS
SELECT
        GETDATE()                                                               AS [execution_date]
,       DB_Name()                                                               AS [database_name]
,       s.name                                                                  AS [schema_name]
,       t.name                                                                  AS [table_name]
,    COUNT(DISTINCT rg.[partition_number])                    AS [table_partition_count]
,       SUM(rg.[total_rows])                                                    AS [row_count_total]
,       SUM(rg.[total_rows])/COUNT(DISTINCT rg.[distribution_id])               AS [row_count_per_distribution_MAX]
,    CEILING    ((SUM(rg.[total_rows])*1.0/COUNT(DISTINCT rg.[distribution_id]))/1048576) AS [rowgroup_per_distribution_MAX]
,       SUM(CASE WHEN rg.[State] = 0 THEN 1                   ELSE 0    END)    AS [INVISIBLE_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE 0    END)    AS [INVISIBLE_rowgroup_rows]
,       MIN(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_AVG]
,       SUM(CASE WHEN rg.[State] = 1 THEN 1                   ELSE 0    END)    AS [OPEN_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE 0    END)    AS [OPEN_rowgroup_rows]
,       MIN(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_AVG]
,       SUM(CASE WHEN rg.[State] = 2 THEN 1                   ELSE 0    END)    AS [CLOSED_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE 0    END)    AS [CLOSED_rowgroup_rows]
,       MIN(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_AVG]
,       SUM(CASE WHEN rg.[State] = 3 THEN 1                   ELSE 0    END)    AS [COMPRESSED_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE 0    END)    AS [COMPRESSED_rowgroup_rows]
,       SUM(CASE WHEN rg.[State] = 3 THEN rg.[deleted_rows]   ELSE 0    END)    AS [COMPRESSED_rowgroup_rows_DELETED]
,       MIN(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_AVG]
,       'ALTER INDEX ALL ON ' + s.name + '.' + t.NAME + ' REBUILD;'             AS [Rebuild_Index_SQL]
FROM    sys.[pdw_nodes_column_store_row_groups] rg
JOIN    sys.[pdw_nodes_tables] nt                   ON  rg.[object_id]          = nt.[object_id]
                                                    AND rg.[pdw_node_id]        = nt.[pdw_node_id]
                                                    AND rg.[distribution_id]    = nt.[distribution_id]
JOIN    sys.[pdw_table_mappings] mp                 ON  nt.[name]               = mp.[physical_name]
JOIN    sys.[tables] t                              ON  mp.[object_id]          = t.[object_id]
JOIN    sys.[schemas] s                             ON t.[schema_id]            = s.[schema_id]
GROUP BY
        s.[name]
,       t.[name]
;
```

После создания представления выполните этот запрос, чтобы определить таблицы с группами строк, содержащими менее 100 тысяч строк. Если требуется достичь более оптимального качества сегментов, пороговое значение числа строк можно увеличить.

```sql
SELECT    *
FROM    [dbo].[vColumnstoreDensity]
WHERE    COMPRESSED_rowgroup_rows_AVG < 100000
        OR INVISIBLE_rowgroup_rows_AVG < 100000
```

После выполнения запроса можно начать просмотр данных и анализ результатов. В таблице ниже показано, на какие данные следует обратить внимание при анализе групп строк.

| Столбец | Как использовать эти данные |
| --- | --- |
| [table_partition_count] |Если таблица секционирована, то счетчики открытых групп строк могут быть больше. Каждая секция в распределении теоретически может иметь связанную с ней открытую группу строк. Это следует учитывать при анализе. Небольшую секционированную таблицу можно оптимизировать, полностью отменив секционирование, так как это улучшит сжатие. |
| [row_count_total] |Общее количество строк в таблице. Например, это значение можно использовать для вычисления процента строк в сжатом состоянии. |
| [row_count_per_distribution_MAX] |Если все строки распределены равномерно, это значение будет целевым числом строк для каждого распределения. Сравните это значение с compressed_rowgroup_count. |
| [COMPRESSED_rowgroup_rows] |Общее количество строк в формате columnstore для таблицы. |
| [COMPRESSED_rowgroup_rows_AVG] |Если среднее количество строк существенно меньше максимального числа строк для группы строк, возможно, стоит использовать инструкции CTAS или ALTER INDEX REBUILD для повторного сжатия данных. |
| [COMPRESSED_rowgroup_count] |Число групп строк в формате columnstore. Очень большое относительно таблицы число указывает на то, что плотность columnstore низка. |
| [COMPRESSED_rowgroup_rows_DELETED] |Логически удаленные строки в формате columnstore. Если это число велико относительно размера таблицы, возможно, стоит создать секцию заново или перестроить индекс, так как в этом случае они удаляются физически. |
| [COMPRESSED_rowgroup_rows_MIN] |Используйте это значение вместе со столбцами AVG и MAX, чтобы оценить диапазон размеров для групп строк в индексе Columnstore. Небольшое значение относительно порогового значения загрузки (102 400 для распределения с выравниванием по секциям) говорит о том, что загрузку данных можно оптимизировать. |
| [COMPRESSED_rowgroup_rows_MAX] |То же, что и выше |
| [OPEN_rowgroup_count] |Открытые группы строк допускаются. Ожидается наличие одной открытой группы строк на каждое распределение таблиц (60). Очень большие числа говорят о загрузке данных в разные секции. Внимательно проверьте стратегию секционирования, чтобы убедиться в ее эффективности |
| [OPEN_rowgroup_rows] |Каждая группа строк может иметь максимум 1 048 576 строк. Используйте это значение для оценки текущей наполненности открытых групп строк. |
| [OPEN_rowgroup_rows_MIN] |Наличие открытых групп указывает, что либо данные медленно загружаются в таблицу, либо предыдущая загрузка привела к переносу оставшихся строк в эту группу строк. С помощью столбцов MIN, MAX, AVG можно узнать, сколько данных находится в открытых группах строк. Для небольших таблиц это может быть 100 % всех данных! В этом случае следует использовать инструкцию ALTER INDEX REBUILD, чтобы принудительно перенести данные в columnstore. |
| [OPEN_rowgroup_rows_MAX] |То же, что и выше |
| [OPEN_rowgroup_rows_AVG] |То же, что и выше |
| [CLOSED_rowgroup_rows] |Для проверки оцените число строк в закрытых группах строк. |
| [CLOSED_rowgroup_count] |Число закрытых групп строк должно быть небольшим (если они вообще присутствуют). Закрытые группы строк можно преобразовать в сжатые группы строк с помощью инструкции ALTER INDEX... Команда реорганизации. Однако обычно это не требуется. Закрытые группы автоматически преобразуются в группы строк Columnstore фоновым процессом "переноса кортежей". |
| [CLOSED_rowgroup_rows_MIN] |Закрытые группы строк должны иметь очень высокий коэффициент заполнения. Если коэффициент заполнения для закрытой группы строк низок, то требуется дальнейший анализ Columnstore. |
| [CLOSED_rowgroup_rows_MAX] |То же, что и выше |
| [CLOSED_rowgroup_rows_AVG] |То же, что и выше |
| [Rebuild_Index_SQL] |Инструкция SQL, которая используется для перестроения индекса Columnstore таблицы. |

## <a name="causes-of-poor-columnstore-index-quality"></a>Причины низкого качества индекса Columnstore

Если вы обнаружили таблицы с сегментами низкого качества, необходимо определить причину.  Ниже описаны распространенные причины низкого качества сегментов:

1. Нехватка памяти при создании индекса.
2. Высокая интенсивность операций DML.
3. Небольшой объем операций загрузки или потоковые загрузки
4. Слишком много секций

По приведенным выше причинам количество строк в каждой группе будет значительно меньше 1 миллиона. Кроме того, вместо перехода в сжатую группу строки могут попасть в разностную группу строк.

### <a name="memory-pressure-when-index-was-built"></a>Нехватка памяти при создании индекса.

Количество строк в сжатой группе строк непосредственно зависит от ширины строк и объема памяти, доступного для обработки группы строк.  Если во время записи строк в таблицы Columnstore возникает нехватка памяти, качество сегмента Columnstore может ухудшиться.  Поэтому для записи строк в таблицы Columnstore рекомендуется выделить максимально возможный объем памяти.  Так как показатели памяти и параллелизма взаимосвязаны, выделение памяти зависит от класса данных в каждой строке таблицы, числа единиц базы данных хранилища данных, выделенного для системы, а также числа слотов выдачи, доступного для записи данных в таблицу.

### <a name="high-volume-of-dml-operations"></a>Высокая интенсивность операций DML.

Интенсивные операции DML по обновлению и удалению строк приводят к неэффективной работе Columnstore. Это особенно верно в случае изменения большей части строк в группе строк.

- Если строка удаляется из сжатой группы строк, она лишь логически помечается как удаленная. Строка остается в сжатой группе строк до перестроения секции или таблицы.
- При вставке строка добавляется во внутреннюю таблицу хранилища строк, называемую разностной группой строк. Вставленная строка не преобразуется в Columnstore до тех пор, пока разностная группа строк не будет заполнена и помечена как закрытая. Группы строк закрываются после достижения максимального объема в 1 048 576 строк.
- Обновление строки в формате columnstore обрабатывается как логическое удаление, а затем — как вставка. Вставленную строку можно сохранить в разностном хранилище.

Пакетные операции обновления и вставки, превышающие пороговое значение в 102 400 строк для распределения с выравниванием по секциям, записываются непосредственно в формате columnstore. Чтобы это произошло, при условии равномерного распределения потребуется изменить более 6,144 млн строк в рамках одной операции. Если число строк для данного распределения с распределением по секциям меньше 102 400, то строки попадают в разностное хранилище и остаются там, пока не будут вставлены или изменены достаточное количество строк, чтобы закрыть группу строк или перестроить индекс.

### <a name="small-or-trickle-load-operations"></a>Небольшой объем операций загрузки или потоковые загрузки

Небольшие нагрузки, которые помещаются в выделенный пул SQL, иногда также называют тонкогоной загрузкой. Обычно они представляют собой практически постоянный поток данных, принимаемых системой. Однако поскольку этот поток является не совсем непрерывным, объем строк невелик. Чаще всего количество данных значительно ниже порогового значения, необходимого для непосредственной загрузки в формате columnstore.

В таких ситуациях рекомендуется сначала разместить данные в хранилище BLOB-объектов Azure, накопить их и лишь затем загрузить. Этот метод часто называют *микропакетной обработкой*.

### <a name="too-many-partitions"></a>Слишком много секций

На качество индекса также влияет секционирование в кластеризованных таблицах Columnstore.  Перед секционированием выделенный пул SQL уже разделяет данные на базы данных 60.  Во время секционирования выполняется дальнейшее разделение данных.  Чтобы воспользоваться преимуществами кластеризованного индекса columnstore при секционировании данных, **каждая** секция должна содержать по крайней мере 1 миллион строк.  При секционировании таблицы на 100 секций таблица должна иметь по крайней мере 6 000 000 000 строк, чтобы получить преимущество от кластеризованного индекса columnstore (60 распределений *100 секции* 1 000 000 строк). Если эта таблица не содержит такого количества строк, рекомендуется уменьшить количество секций или использовать таблицу без кластеризованных индексов.

Когда в таблицу будут загружены данные, чтобы определить и перестроить таблицы с неоптимальными кластеризованными индексами Columnstore, выполните приведенные ниже действия.

## <a name="rebuilding-indexes-to-improve-segment-quality"></a>Повышение качества сегментов за счет перестроения индексов

### <a name="step-1-identify-or-create-user-which-uses-the-right-resource-class"></a>Шаг 1. Определение или создание пользователя, который использует соответствующий класс ресурсов

Простой способ быстро повысить качество сегментов — перестроить индекс.  Инструкция SQL, возвращенная в предыдущем представлении, возвращает инструкцию ALTER INDEX REBUILD, которую можно использовать для перестроения индексов. При перестроении индексов необходимо выделить достаточный объем памяти для сеанса.  Для этого повысьте класс ресурсов для пользователя, который имеет разрешение на перестроение индекса для этой таблицы, до рекомендованного минимального класса ресурсов.

Ниже приведен пример того, как можно выделить дополнительный объем памяти для пользователя, увеличив класс ресурсов. Сведения об использовании классов ресурсов см. в статье [Классы ресурсов для управления рабочими нагрузками](resource-classes-for-workload-management.md).

```sql
EXEC sp_addrolemember 'xlargerc', 'LoadUser'
```

### <a name="step-2-rebuild-clustered-columnstore-indexes-with-higher-resource-class-user"></a>Шаг 2. Перестройка кластеризованных индексов Columnstore, используя пользователя с более высоким классом ресурсов

Войдите от имени пользователя из шага 1 (например, LoadUser), который теперь использует более высокий класс ресурсов, и выполните инструкции ALTER INDEX. Убедитесь, что этот пользователь имеет разрешение ALTER в отношении таблиц, в которых будет выполнятся перестроение индекса. В этих примерах показано перестроение всего индекса Columnstore и перестроение одной секции. В больших таблицах целесообразно перестроить только одну секцию за раз.

Кроме того, вместо перестроения индекса можно скопировать таблицу в новую таблицу [с помощью CTAS](sql-data-warehouse-develop-ctas.md). Какой способ лучше? Для больших объемов данных инструкция CTAS обычно выполняется быстрее, чем [ALTER INDEX](/sql/t-sql/statements/alter-index-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true). Что же касается небольших объемов данных, рекомендуем использовать инструкцию ALTER INDEX. Она проще в использовании и не требует замены таблицы.

```sql
-- Rebuild the entire clustered index
ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD
```

```sql
-- Rebuild a single partition
ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5
```

```sql
-- Rebuild a single partition with archival compression
ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5 WITH (DATA_COMPRESSION = COLUMNSTORE_ARCHIVE)
```

```sql
-- Rebuild a single partition with columnstore compression
ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5 WITH (DATA_COMPRESSION = COLUMNSTORE)
```

Перестроение индекса в выделенном пуле SQL является автономной операцией.  Дополнительные сведения о перестройке индексов см. в разделе об использовании инструкции ALTER INDEX REORGANIZE в статьях [Дефрагментация индексов columnstore](/sql/relational-databases/indexes/columnstore-indexes-defragmentation?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) и работе с [ALTER INDEX (Transact-SQL)](/sql/t-sql/statements/alter-index-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

### <a name="step-3-verify-clustered-columnstore-segment-quality-has-improved"></a>Шаг 3. Проверка улучшения качества кластеризованных сегментов Columnstore

Повторно выполните запрос на определение таблицы с сегментами низкого качества и убедитесь, что качество сегментов улучшилось.  Если это не так, возможно, в таблице слишком широкие строки.  Если для перестроения индексов требуется более высокий объем памяти,

## <a name="rebuilding-indexes-with-ctas-and-partition-switching"></a>Перестроение индексов с помощью инструкции CTAS и переключения секций

В этом примере для перестроения секции таблицы используется инструкция [CREATE TABLE AS SELECT (CTAS)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) и переключение секций.

```sql
-- Step 1: Select the partition of data and write it out to a new table using CTAS
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
FROM    [dbo].[FactInternetSales]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

-- Step 2: Switch IN the rebuilt data with TRUNCATE_TARGET option
ALTER TABLE [dbo].[FactInternetSales_20000101_20010101] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales] PARTITION 2 WITH (TRUNCATE_TARGET = ON);
```

Дополнительные сведения о повторном создании секций с помощью CTAS см. [в разделе Использование секций в выделенном пуле SQL](sql-data-warehouse-tables-partition.md).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о разработке таблиц см. в статье [Общие сведения о проектировании таблиц в хранилище данных SQL Azure](sql-data-warehouse-tables-overview.md).
