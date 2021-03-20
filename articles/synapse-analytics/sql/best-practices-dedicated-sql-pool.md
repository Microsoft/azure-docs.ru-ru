---
title: Рекомендации по выделенным пулам SQL
description: Рекомендации и рекомендации, которые следует учитывать при работе с выделенными пулами SQL.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 03/17/2021
ms.author: martinle
ms.reviewer: igorstan
ms.openlocfilehash: b11a76be94fc52285482e13dadbc8c7c92af1374
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104609785"
---
# <a name="best-practices-for-dedicated-sql-pools-in-azure-synapse-analytics"></a>Рекомендации по выделенным пулам SQL в Azure синапсе Analytics

Эта статья содержит набор рекомендаций, которые помогут добиться оптимальной производительности для выделенных пулов SQL в Azure синапсе Analytics. Ниже вы найдете основные рекомендации и важные аспекты, которые помогут вам при создании решения. В каждом разделе содержится краткое описание определенных понятий и ссылки на статьи с более подробными сведениями.

## <a name="dedicated-sql-pools-loading"></a>Загрузка выделенных пулов SQL

Руководство по [загрузке данных](data-loading-best-practices.md)для выделенных пулов SQL см. в этой статье.

## <a name="reduce-cost-with-pause-and-scale"></a>Снижение расходов за счет приостановки и масштабирования ресурсов

Дополнительные сведения о сокращении затрат при помощи приостановки и масштабирования см. в статье об [управлении вычислительными ресурсами](../sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="maintain-statistics"></a>Обеспечение статистики

Выделенный пул SQL можно настроить для автоматического обнаружения и создания статистики по столбцам.  Планы запросов, созданные с помощью оптимизатора, зависят от доступной статистики.  

Рекомендуется включить AUTO_CREATE_STATISTICS для баз данных и обновлять статистику ежедневно или после каждой нагрузки, чтобы гарантировать, что статистика по столбцам, используемых в запросах, всегда будет актуальной.

Чтобы сократить время, затрачиваемое на обслуживание статистики, необходимо указать, какие столбцы содержат данные статистики или требуют наиболее частого обновления. Например, можно ежедневно обновлять столбцы дат, в которые добавляются новые значения. Статистику рекомендуется вести для столбцов, которые являются частью объединения, используются в предложении WHERE или входят в GROUP BY.

Дополнительные сведения о статистике можно найти в статье об [управлении статистикой таблицы](develop-tables-statistics.md), а также статьях [CREATE STATISTICS](/sql/t-sql/statements/create-statistics-transact-sql?view=azure-sqldw-latest&preserve-view=true) и [UPDATE STATISTICS](/sql/t-sql/statements/update-statistics-transact-sql?view=azure-sqldw-latest&preserve-view=true).

## <a name="tune-query-performance-with-new-product-enhancements"></a>Настройка производительности запросов с помощью новых усовершенствований продукта

- [Performance tuning with materialized views](../sql-data-warehouse/performance-tuning-materialized-views.md) (Настройка производительности с помощью материализованных представлений)
- [Performance tuning with result set caching](../sql-data-warehouse/performance-tuning-ordered-cci.md) (Настройка производительности с помощью упорядоченного кластеризованного индекса columnstore)
- [Performance tuning with result set caching](../sql-data-warehouse/performance-tuning-result-set-caching.md) (Настройка производительности путем кэширования результирующего набора)


## <a name="group-insert-statements-into-batches"></a>Объединение инструкций INSERT в группы

В зависимости от ваших потребностей, однократная загрузка в небольшую таблицу с помощью инструкции INSERT, например `INSERT INTO MyLookup VALUES (1, 'Type 1')`, может быть лучшим решением, но для загрузки тысяч или миллионов строк на протяжении дня отдельные инструкции INSERT, скорее всего, не будут оптимальным выбором.

Одним из способов решения этой проблемы является разработка одного процесса для записи в файл, за которым следует другой процесс для периодической загрузки этого файла. Дополнительные сведения см. в статье [INSERT](/sql/t-sql/statements/insert-transact-sql?view=azure-sqldw-latest&preserve-view=true).

## <a name="use-polybase-to-load-and-export-data-quickly"></a>Быстрая загрузка и экспорт данных с помощью PolyBase

Выделенный пул SQL поддерживает загрузку и экспорт данных с помощью нескольких средств, включая фабрику данных Azure, Polybase и BCP.  При работе с данными небольшого объема, что не требует высокой производительности, можно использовать любой инструмент.  

> [!NOTE]
> Для загрузки или экспорта большого объема данных, что требует высокой производительности, лучше всего использовать PolyBase.

Загрузку данных с помощью PolyBase можно выполнить, используя команды CTAS или INSERT INTO. Функция CTAS позволяет свести к минимуму нагрузку ведения журнала транзакций и быстрее всего выполнить загрузку данных. Фабрика данных Azure также поддерживает загрузки PolyBase и способна достичь производительности, сопоставимой с CTAS. PolyBase поддерживает различные форматы файлов, включая формат GZIP.

Чтобы максимально повысить пропускную способность при использовании текстовых файлов в формате GZIP, разбейте файлы на 60 или больше частей. Это позволит достичь максимальной степени параллелизма для загрузки. Кроме того, для повышения общей пропускной способности можно загружать данные одновременно. Дополнительные сведения на темы, относящиеся к этому разделу, приведены в следующих статьях:

- [Загрузка данных](../sql-data-warehouse/design-elt-data-loading.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [руководством по использованию PolyBase](data-loading-best-practices.md)
- [Шаблоны и стратегии загрузки выделенного пула SQL](/archive/blogs/sqlcat/azure-sql-data-warehouse-loading-patterns-and-strategies)
- [Загрузка данных с помощью фабрики данных Azure](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Перенос данных с помощью фабрики данных Azure](../../data-factory/transform-data-using-machine-learning.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [Инструкция Create table as select (CTAS)](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)

## <a name="load-then-query-external-tables"></a>Загрузка внешних таблиц и отправка запросов к ним

Polybase не подходит для запросов. Таблицы polybase для выделенных пулов SQL в настоящее время поддерживают только файлы больших двоичных объектов Azure и хранилище Azure Data Lake. Эти файлы не обслуживаются какими-либо вычислительными ресурсами. В результате выделенные пулы SQL не могут отгружать эту работу и считывать весь файл, загружая его в базу данных tempdb, чтобы он мог считывать данные.

Если будет отправлено несколько запросов к этим данным, то лучше один раз загрузить их в локальную таблицу и предоставить ее для выполнения запросов. Дополнительные рекомендации по использованию Polybase приведены в [этой статье](data-loading-best-practices.md).

## <a name="hash-distribute-large-tables"></a>Хэш-распределение больших таблиц

По умолчанию таблицы распределяются по методу циклического перебора.   Это позволяет упростить процесс создания таблиц, так как пользователям не нужно принимать решение о типе распределения. Таблицы с циклическим перебором могут быть достаточно эффективными для некоторых рабочих нагрузок. Но в большинстве случаев столбец распределения обеспечивает лучшую производительность.  

Наглядно это превосходство можно увидеть при объединении больших таблиц фактов.  

Например, во время выполнения запроса на объединение таблицы Orders, распределенной по идентификатору order_id, с таблицей Transactions, распределенной по тому же идентификатору, этот запрос превращается в запрос к серверу. Затем операции перемещения данных удаляются. Чем меньше в запросе действий, тем быстрее он выполняется. Скорость выполнения запроса также зависит от объема перемещаемых данных.

> [!TIP]
> При загрузке распределенной таблицы входящие данные не должны быть отсортированы по ключу распределения. Это приведет к увеличению времени нагрузки.

По приведенным ниже ссылкам на статьи вы можете получите дополнительные сведения о повышении производительности с помощью выбора столбца распределения. Кроме того, вы найдете сведения о том, как определить распределенную таблицу в предложении WITH инструкции CREATE TABLE:

- [Общие сведения о таблицах](develop-tables-overview.md)
- [Distributing tables in SQL Data Warehouse (Распределение таблиц в хранилище данных SQL)](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Выбор распределения таблицы](/archive/blogs/sqlcat/choosing-hash-distributed-table-vs-round-robin-distributed-table-in-azure-sql-dw-service)
- [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true)
- [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true)

## <a name="do-not-over-partition"></a>Недопущение избыточного секционирования

Хотя секционирование данных — это эффективный способ управления данными с помощью переключения секций или оптимизации сканирования путем исключения секций, наличие большого количества секций может замедлить выполнение запросов.  Зачастую стратегия секционирования с высокой степенью детализации, которая хорошо работает на SQL Server может не работать в выделенном пуле SQL.  

Слишком большое количество секций снижает эффективность кластеризованных индексов columnstore, если в секции содержится менее 1 миллиона строк. выделенные пулы SQL автоматически разделяют данные на базы данных 60. Таким образом, если создается таблица со 100 секциями, получится 6000 секций. Рабочие нагрузки отличаются друг от друга, поэтому рекомендуется поэкспериментировать с секционированием, чтобы выбрать наиболее подходящее количество секций для вашей рабочей нагрузки.  

Одним из вариантов является использование степени детализации, которая ниже, чем вы реализовали с помощью SQL Server. Например, попробуйте использовать секционирование по неделям или месяцам, а не по дням.

Дополнительные сведения о секционировании см. в статье [Секционирование таблиц](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="minimize-transaction-sizes"></a>Уменьшение размера транзакций

Инструкции INSERT, UPDATE и DELETE выполняются в транзакции. В случае сбоя их нужно откатить. Чтобы сократить время выполнения отката, необходимо по возможности уменьшить размеры транзакций.  Это можно сделать, разделив инструкции INSERT, UPDATE и DELETE на части. Например, если вы ожидаете, что операция INSERT будет выполняться 1 час, ее можно разделить на четыре части. Таким образом, время каждого выполнения сократится до 15 минут.  

> [!TIP]
> К пустым таблицам можно применять специальные операции, которые сопровождаются записью в журнал минимальных сведений (такие как CTAS, TRUNCATE, DROP TABLE или INSERT). Это снизит риск отката.  

Устранить откаты также можно, используя для управления данными только операции с метаданными (например, переключение секций).  Например, вместо выполнения инструкции DELETE для удаления всех строк в таблице, для которых в параметре order_date указан октябрь 2001 года, данные можно секционировать по месяцам. Затем можно записать в секцию данные для пустой секции из другой таблицы (см. примеры ALTER TABLE).  

Для несекционированной таблицы можно применить вместо DELETE инструкцию CTAS, которая запишет данные в таблицу.  Если на выполнение этих двух инструкций требуется одинаковое время, целесообразно использовать инструкцию CTAS, так как для ее выполнения необходима минимальная нагрузка ведения журнала транзакций, и ее можно быстро отменить.

Дополнительные сведения о содержимом, связанном с этим разделом, приведены в следующих статьях:

- [Инструкция Create table as select (CTAS)](../sql-data-warehouse/sql-data-warehouse-develop-ctas.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Основные сведения о транзакциях](develop-transactions.md)
- [Оптимизация транзакций](../sql-data-warehouse/sql-data-warehouse-develop-best-practices-transactions.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Секционирование таблиц](../sql-data-warehouse/sql-data-warehouse-tables-partition.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [TRUNCATE TABLE](/sql/t-sql/statements/truncate-table-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [ALTER TABLE](/sql/t-sql/statements/alter-table-transact-sql?view=azure-sqldw-latest&preserve-view=true)

## <a name="reduce-query-result-sizes"></a>Уменьшение размера результатов запроса

Уменьшение размера результатов запроса помогает избежать проблем на стороне клиента, вызванных большими результатами запросов.  Запрос можно изменить, чтобы уменьшить число возвращаемых строк. Некоторые инструменты создания запросов позволяют добавлять в каждый запрос синтаксис "первые N".  Можно также использовать инструкцию CETAS, чтобы записать результат запроса во временную таблицу, а затем использовать экспорт PolyBase для обработки на нижнем уровне.

## <a name="use-the-smallest-possible-column-size"></a>Использование минимального размера столбца

При определении DDL рекомендуется использовать поддерживаемый тип данных с наименьшим размером. Это позволит повысить производительность запросов.  Эта рекомендация особенно важна для столбцов CHAR и VARCHAR.  Если самое длинное значение в столбце состоит из 25 знаков, столбец необходимо определить как VARCHAR(25).  Не рекомендуется использовать по умолчанию длинные значения столбцов.  Кроме того, по возможности определяйте столбцы как VARCHAR, а не NVARCHAR.

Дополнительные сведения о важных концепциях, связанных с приведенными выше данными, см. в [обзоре таблиц](develop-tables-overview.md), статьях о [табличных типах данных](develop-tables-data-types.md) и [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true).

## <a name="use-temporary-heap-tables-for-transient-data"></a>Использование временных таблиц без кластеризованных индексов для хранения временных данных

При временном размещении данных в выделенных пулах SQL таблицы кучи обычно делают общий процесс быстрее.  Если вы загружаете данные только для промежуточного хранения перед их последующими преобразованиями, загрузка таблицы в таблицу кучи будет происходить быстрее по сравнению с загрузкой данных в кластеризованную таблицу columnstore.  

Загрузка данных во временную таблицу выполняется гораздо быстрее, чем загрузка таблицы в постоянное хранилище.  Временные таблицы начинаются с символа "#" и доступны только создавшему их сеансу. Следовательно, они подходят только для определенных сценариев использования. Таблицы без кластеризованных индексов определены в предложении WITH инструкции CREATE TABLE.  При использовании временной таблицы рекомендуется создавать в ней статистику.

Дополнительные рекомендации см. в статьях [Временные таблицы](/sql/t-sql/statements/alter-table-transact-sql?view=azure-sqldw-latest&preserve-view=true), [CREATE TABLE](/sql/t-sql/statements/create-table-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true) и [CREATE TABLE AS SELECT](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?view=azure-sqldw-latest&preserve-view=true).

## <a name="optimize-clustered-columnstore-tables"></a>Оптимизация таблиц с кластеризованными индексами columnstore

Кластеризованные индексы columnstore — это один из наиболее эффективных способов хранения данных в выделенном пуле SQL.  По умолчанию таблицы в выделенном пуле SQL создаются как кластеризованный ColumnStore.  Качество кластеризованного сегмента Columnstore существенно влияет на эффективность выполнения запросов в таблицах с кластеризованными индексами Columnstore.  Если во время записи строк в таблицы Columnstore возникает нехватка памяти, качество сегмента Columnstore может ухудшиться.  

Качество сегмента можно определить по числу строк в сжатой группе строк. Дополнительные сведения об определении и улучшении качества сегмента для таблиц с кластеризованными индексами columnstore см. в разделе [Причины низкого качества индекса columnstore](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json#causes-of-poor-columnstore-index-quality) статьи [Индексирование таблиц в пуле Synapse SQL](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).  

Так как высококачественные сегменты columnstore важны, рекомендуется использовать для загрузки данных идентификаторы пользователей, которым назначен класс ресурсов среднего или большого размера. Использование [единиц использования хранилища данных](resource-consumption-models.md) меньшего размера требует присвоения пользователю загрузки класса ресурсов большего размера.

Таблицы columnstore обычно не отправляют данные в сжатый сегмент columnstore до тех пор, пока в таблице не будет более 1 миллиона строк. Каждая выделенная таблица пула SQL разделяется на таблицы 60. Таким образом, таблицы columnstore не смогут улучшить запрос, если таблица содержит менее 60 миллионов строк.  

> [!TIP]
> Для таблиц, в которых менее 60 миллионов строк, индекс сolumstore, как правило, лучше не использовать.  

При секционировании данных в каждой секции должен быть 1 миллион строк, чтобы можно было получить преимущество от кластеризованного индекса columnstore.  Если таблица состоит из 100 секций, то чтобы получить преимущество от использования кластеризованного индекса columnstore, она должна состоять как минимум из 6 миллиардов строк (60 распределений *100 секций* 1 миллион строк).  

Если таблица не содержит 6 миллиардов строк, у вас есть два основных варианта. Либо сократите количество секций, либо рассмотрите возможность использования таблицы кучи.  Чтобы получить более высокую производительность, возможно, вместо кластеризованной таблицы columnstore стоит использовать таблицу кучи, содержащую вторичные индексы.

Если выбрать только необходимые столбцы, запросы к таблице ColumnStore будут выполняться быстрее.  Дополнительные сведения о таблицах и индексах columnstore. их можно найти в следующих статьях:
- [Индексы таблиц](../sql-data-warehouse/sql-data-warehouse-tables-index.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Руководство по индексам columnstore](/sql/relational-databases/indexes/columnstore-indexes-overview?view=azure-sqldw-latest&preserve-view=true)
- [перестроением индексов columnstore](../sql-data-warehouse/sql-data-warehouse-tables-index.md?view=azure-sqldw-latest&preserve-view=true#rebuilding-indexes-to-improve-segment-quality) 
- [Performance tuning with result set caching](../sql-data-warehouse/performance-tuning-ordered-cci.md) (Настройка производительности с помощью упорядоченного кластеризованного индекса columnstore)

## <a name="use-larger-resource-class-to-improve-query-performance"></a>Использование класса ресурсов большого размера для повышения производительности запросов

Чтобы выделить память для выполнения запросов, пулы SQL используют группы ресурсов. Изначально для всех пользователей настроен класс ресурсов небольшого размера, предусматривающий 100 МБ памяти для каждого распределения.  Всегда существует 60 распределений. Каждому распределению предоставляется минимум 100 МБ. Общий объем памяти, выделяемой для всей системы, составляет 6 000 МБ или чуть меньше 6 ГБ.  

На выполнение некоторых запросов (например, на объединение таблиц с кластеризованными индексами Columnstore или добавления в них данных) требуется больше памяти.  На выполнение некоторых запросов, таких как запросы на обычное сканирование, ресурсы памяти не требуются. Использование больших классов ресурсов влияет на параллелизм. Поэтому учтите это, прежде чем перемещать всех пользователей в большой класс ресурсов.

Дополнительные сведения о классах ресурсов см. в статье [Классы ресурсов для управления рабочими нагрузками](../sql-data-warehouse/resource-classes-for-workload-management.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

## <a name="use-smaller-resource-class-to-increase-concurrency"></a>Использование класса ресурсов небольшого размера для увеличения параллелизма

Если вы заметите длительную задержку при выполнении пользовательских запросов, возможно, ваши пользователи обрабатывают большие классы ресурсов. Такая ситуация увеличивает потребление слотов параллелизма, что может привести к постановке других запросов в очередь.  Чтобы определить очередь выполнения пользовательских запросов, выполните команду `SELECT * FROM sys.dm_pdw_waits`, которая позволяет просмотреть возвращенные строки.

Дополнительные сведения приведены в статьях [Классы ресурсов для управления рабочими нагрузками](../sql-data-warehouse/resource-classes-for-workload-management.md) и [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?view=azure-sqldw-latest&preserve-view=true).

## <a name="use-dmvs-to-monitor-and-optimize-your-queries"></a>Использование динамических административных представлений для отслеживания и оптимизации запросов

Выделенные пулы SQL имеют несколько динамических административных представлений, которые можно использовать для наблюдения за выполнением запросов.  В первой статье в списке ниже предоставлены пошаговые инструкции по отслеживанию выполнения запроса.  Чтобы быстро определить запросы в динамических административных представлениях, используйте в запросах параметр LABEL. Дополнительные сведения см. в статьях, перечисленных в списке ниже.

- [Мониторинг рабочей нагрузки с помощью динамических административных представлений](../sql-data-warehouse/sql-data-warehouse-manage-monitor.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)

- [LABEL](develop-label.md)
- [OPTION](/sql/t-sql/queries/option-clause-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_sql_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-sql-requests-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?view=azure-sqldw-latest&preserve-view=true)
- [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?view=azure-sqldw-latest&preserve-view=true)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о распространенных проблемах и способах их решения см. в статье об [устранении неполадок](../sql-data-warehouse/sql-data-warehouse-troubleshoot.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

Если вам нужна информация, не указанная в этой статье, выполните поиск на [странице Microsoft Q&A вопроса по Azure синапсе](/answers/topics/azure-synapse-analytics.html) , где вы хотите задать вопросы другим пользователям и к группе продуктов Azure синапсе Analytics.  

Мы регулярно просматриваем этот форум и следим за тем, чтобы другие пользователи или наши специалисты ответили на интересующие вас вопросы.  Если вы предпочитаете задавать вопросы на Stack Overflow, у нас также есть [форум Stack overflow Azure синапсе Analytics](https://stackoverflow.com/questions/tagged/azure-synapse).

Для получения запросов функций используйте страницу [обратной связи Azure синапсе Analytics](https://feedback.azure.com/forums/307516-sql-data-warehouse) .  Добавление запросов или голосование за другие запросы помогает нам сосредоточить усилия на работе над наиболее востребованными функциями.