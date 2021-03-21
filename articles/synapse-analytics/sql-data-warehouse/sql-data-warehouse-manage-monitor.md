---
title: Мониторинг выделенной рабочей нагрузки пула SQL с помощью динамических административных представлений
description: Узнайте, как отслеживать выделенную рабочую нагрузку пула SQL Azure синапсе Analytics и выполнение запросов с помощью динамических административных представлений.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/24/2020
ms.author: rortloff
ms.reviewer: igorstan
ms.custom: synapse-analytics
ms.openlocfilehash: 62064eaae6aa7fb3438845170497035473227d30
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98685214"
---
# <a name="monitor-your-azure-synapse-analytics-dedicated-sql-pool-workload-using-dmvs"></a>Мониторинг выделенной рабочей нагрузки пула SQL Azure синапсе Analytics с помощью динамических административных представлений

В этой статье описывается, как использовать динамические административные представления (DMV) для наблюдения за рабочей нагрузкой, включая анализ выполнения запросов в пуле SQL.

## <a name="permissions"></a>Разрешения

Чтобы запросить динамические административные представления в этой статье, необходимо либо **Просмотреть состояние базы данных** , либо разрешение **Control** . Как правило, предоставление **состояния базы данных View** является предпочтительным разрешением, так как оно является гораздо более узким.

```sql
GRANT VIEW DATABASE STATE TO myuser;
```

## <a name="monitor-connections"></a>Мониторинг подключений

Все имена входа в хранилище данных регистрируются в [sys.dm_pdw_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-sessions-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  Это динамическое административное представление содержит записи о последних 10 000 операций входа.  Идентификатор session_id является первичным ключом и назначается последовательно для каждого нового входа.

```sql
-- Other Active Connections
SELECT * FROM sys.dm_pdw_exec_sessions where status <> 'Closed' and session_id <> session_id();
```

## <a name="monitor-query-execution"></a>Наблюдение за выполнением запросов

Все запросы, выполняемые в пуле SQL, регистрируются в [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).  Это динамическое административное представление содержит записи о последних 10 000 запросах.  Идентификатор request_id уникально идентифицирует каждый запрос и является первичным ключом для этого динамического административного представления.  Идентификатор request_id назначается последовательно для каждого нового запроса с добавлением префикса QID, означающего идентификатор запроса.  При запросе конкретного session_id из этого динамического административного представления будут показаны все запросы для данной операции входа.

> [!NOTE]
> Хранимые процедуры используют несколько идентификаторов request_id.  Идентификатора запросов назначаются последовательно.

Ниже приведено несколько действий для проверки планов выполнения запросов и длительности конкретных запросов.

### <a name="step-1-identify-the-query-you-wish-to-investigate"></a>Шаг 1. Определение запроса, который нужно исследовать

```sql
-- Monitor active queries
SELECT *
FROM sys.dm_pdw_exec_requests
WHERE status not in ('Completed','Failed','Cancelled')
  AND session_id <> session_id()
ORDER BY submit_time DESC;

-- Find top 10 queries longest running queries
SELECT TOP 10 *
FROM sys.dm_pdw_exec_requests
ORDER BY total_elapsed_time DESC;

```

**Запишите идентификатор запроса**, который вы хотите исследовать. Он указан в приведенных выше результатах запроса.

Запросы с состоянием **Приостановлено** можно поставить в очередь ввиду большого количества активных выполняющихся запросов. Эти запросы также отображаются в запросе [sys.dm_pdw_waits](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-waits-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) ожидания с типом усерконкурренциресаурцетипе. Сведения об ограничениях параллелизма см. в разделе [ограничения памяти и параллелизма](memory-concurrency-limits.md) или [классы ресурсов для управления рабочей нагрузкой](resource-classes-for-workload-management.md). Запросы также могут быть отложены по другим причинам, в том числе из-за блокировки объектов.  Если запрос ожидает ресурс, см. раздел [исследование запросов, ожидающих ресурсы](#monitor-waiting-queries) далее в этой статье.

Чтобы упростить поиск запроса в [sys.dm_pdw_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-exec-requests-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) таблице, используйте [Label](/sql/t-sql/queries/option-clause-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) , чтобы назначить комментарий для запроса, который можно найти в представлении sys.dm_pdw_exec_requests.

```sql
-- Query with Label
SELECT *
FROM sys.tables
OPTION (LABEL = 'My Query')
;

-- Find a query with the Label 'My Query'
-- Use brackets when querying the label column, as it it a key word
SELECT  *
FROM    sys.dm_pdw_exec_requests
WHERE   [label] = 'My Query';
```

### <a name="step-2-investigate-the-query-plan"></a>Шаг 2. Изучение плана запроса

Используйте идентификатор запроса для получения плана распределенного SQL (DSQL) запроса из [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true)

```sql
-- Find the distributed query plan steps for a specific query.
-- Replace request_id with value from Step 1.

SELECT * FROM sys.dm_pdw_request_steps
WHERE request_id = 'QID####'
ORDER BY step_index;
```

Если план DSQL выполняется больше времени, чем ожидалось, причина может быть в том, что это сложный план со множеством действий DSQL или только одним этапом, который выполняется слишком долго.  Если план содержит множество действий с несколькими операциями перемещения, рассмотрите возможность оптимизировать распределение таблиц, чтобы сократить перемещение данных. В статье [Распределение таблиц](sql-data-warehouse-tables-distribute.md) поясняется, почему для решения запроса требуется переместить данные. В статье также объясняются стратегии распределения для сведения к минимуму перемещения данных.

Чтобы просмотреть дополнительные сведения об одном шаге, *operation_type* столбец длительно выполняемого шага запроса и обратите внимание на **Индекс шага**:

* Перейдите к шагу 3 для **операций SQL**: OnOperation, RemoteOperation, ReturnOperation.
* Перейдите к шагу 4 для **операций перемещения данных**: ShuffleMoveOperation, BroadcastMoveOperation, TrimMoveOperation, PartitionMoveOperation, MoveOperation, CopyOperation.

### <a name="step-3-investigate-sql-on-the-distributed-databases"></a>Шаг 3. изучение SQL в распределенных базах данных

Используйте идентификатор запроса и индекс шага для получения сведений из [sys.dm_pdw_sql_requests](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true), который содержит сведения о выполнении шага запроса для всех распределенных баз данных.

```sql
-- Find the distribution run times for a SQL step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_sql_requests
WHERE request_id = 'QID####' AND step_index = 2;
```

При выполнении шага запроса можно использовать [DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) для получения оценочного плана SQL Server из кэша планов SQL Server для шага, выполняемого в определенном распределении.

```sql
-- Find the SQL Server execution plan for a query running on a specific SQL pool or control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(1, 78);
```

### <a name="step-4-investigate-data-movement-on-the-distributed-databases"></a>Шаг 4. изучение перемещения данных в распределенных базах данных

Используйте идентификатор запроса и индекс шага, чтобы получить сведения о шаге перемещения данных, выполняемом в каждом распределении, из [sys.dm_pdw_dms_workers](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-dms-workers-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

```sql
-- Find information about all the workers completing a Data Movement Step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_dms_workers
WHERE request_id = 'QID####' AND step_index = 2;
```

* Перейдите к столбцу *total_elapsed_time*, чтобы просмотреть, есть ли какая-то операция распространения, перемещение данных для которой значительно больше времени по сравнению с другими операциями.
* Для длительной операции распространения обратитесь к столбцу *rows_processed* и проверьте, является ли количество перемещаемых строк для этой операции значительно большим по сравнению с другими операциями. Если это так, такой результат может означать отклонение базовых данных. Одна из причин неравномерного распределения данных — распределение по столбцу с большим количеством значений NULL (строки которых будут находиться в одном распределении). Предотвратите снижение производительности запросов за счет предотвращения распространения для этих типов столбцов или фильтрация запроса, чтобы исключить значения NULL, если это возможно. 

Если запрос выполняется, можно использовать [инструкцию DBCC PDW_SHOWEXECUTIONPLAN](/sql/t-sql/database-console-commands/dbcc-pdw-showexecutionplan-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) , чтобы получить SQL Server предполагаемый план из кэша планов SQL Server для текущего этапа SQL в определенном распределении.

```sql
-- Find the SQL Server estimated plan for a query running on a specific SQL pool Compute or control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(55, 238);
```

<a name="waiting"></a>

## <a name="monitor-waiting-queries"></a>Наблюдение за ожидающими запросами

Если вы обнаружили, что запрос не выполняется, так как ожидает ресурс, то можете отобразить все ожидаемые запросом ресурсы, выполнив приведенный ниже запрос.

```sql
-- Find queries
-- Replace request_id with value from Step 1.

SELECT waits.session_id,
      waits.request_id,  
      requests.command,
      requests.status,
      requests.start_time,  
      waits.type,
      waits.state,
      waits.object_type,
      waits.object_name
FROM   sys.dm_pdw_waits waits
   JOIN  sys.dm_pdw_exec_requests requests
   ON waits.request_id=requests.request_id
WHERE waits.request_id = 'QID####'
ORDER BY waits.object_name, waits.object_type, waits.state;
```

Если запрос активно ожидает ресурсы из другого запроса, то его состоянием будет **AcquireResources**.  Если запрос содержит все необходимые ресурсы, то его состоянием будет **Granted**.

## <a name="monitor-tempdb"></a>Мониторинг tempdb

База данных tempdb используется для хранения промежуточных результатов во время выполнения запроса. Высокий уровень использования базы данных tempdb может привести к снижению производительности запросов. Для каждого настроенного DW100c выделено 399 ГБ пространства tempdb (DW1000c будет иметь 3,99 ТБ общего пространства tempdb).  Ниже приведены советы по мониторингу использования tempdb и снижению использования базы данных tempdb в запросах.

### <a name="monitoring-tempdb-with-views"></a>Мониторинг tempdb с помощью представлений

Чтобы отслеживать использование tempdb, сначала установите представление [Microsoft.vw_sql_requests](https://github.com/Microsoft/sql-data-warehouse-samples/blob/master/solutions/monitoring/scripts/views/microsoft.vw_sql_requests.sql) из [набора Microsoft Toolkit для пула SQL](https://github.com/Microsoft/sql-data-warehouse-samples/tree/master/solutions/monitoring). Затем можно выполнить следующий запрос, чтобы просмотреть использование tempdb на каждом узле для всех выполненных запросов:

```sql
-- Monitor tempdb
SELECT
    sr.request_id,
    ssu.session_id,
    ssu.pdw_node_id,
    sr.command,
    sr.total_elapsed_time,
    es.login_name AS 'LoginName',
    DB_NAME(ssu.database_id) AS 'DatabaseName',
    (es.memory_usage * 8) AS 'MemoryUsage (in KB)',
    (ssu.user_objects_alloc_page_count * 8) AS 'Space Allocated For User Objects (in KB)',
    (ssu.user_objects_dealloc_page_count * 8) AS 'Space Deallocated For User Objects (in KB)',
    (ssu.internal_objects_alloc_page_count * 8) AS 'Space Allocated For Internal Objects (in KB)',
    (ssu.internal_objects_dealloc_page_count * 8) AS 'Space Deallocated For Internal Objects (in KB)',
    CASE es.is_user_process
    WHEN 1 THEN 'User Session'
    WHEN 0 THEN 'System Session'
    END AS 'SessionType',
    es.row_count AS 'RowCount'
FROM sys.dm_pdw_nodes_db_session_space_usage AS ssu
    INNER JOIN sys.dm_pdw_nodes_exec_sessions AS es ON ssu.session_id = es.session_id AND ssu.pdw_node_id = es.pdw_node_id
    INNER JOIN sys.dm_pdw_nodes_exec_connections AS er ON ssu.session_id = er.session_id AND ssu.pdw_node_id = er.pdw_node_id
    INNER JOIN microsoft.vw_sql_requests AS sr ON ssu.session_id = sr.spid AND ssu.pdw_node_id = sr.pdw_node_id
WHERE DB_NAME(ssu.database_id) = 'tempdb'
    AND es.session_id <> @@SPID
    AND es.login_name <> 'sa'
ORDER BY sr.request_id;
```

Если у вас есть запрос, который потребляет большой объем памяти или получил сообщение об ошибке, связанное с выделением базы данных tempdb, это могло произойти из-за очень большого CREATE TABLE в том случае, если инструкция [SELECT (CTAS)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse) или [INSERT SELECT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) выполняется с ошибкой в окончательной операции перемещения данных. Обычно это может быть определено как операция Шуффлемове в плане распределенного запроса непосредственно перед завершающим ВЫДЕЛЕНИЕм INSERT.  Используйте [sys.dm_pdw_request_steps](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-request-steps-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) для мониторинга операций шуффлемове.

Наиболее распространенный способ устранения проблемы состоит в том, чтобы разбить инструкцию CTAS или INSERT SELECT на несколько инструкций Load, чтобы объем данных не превышал ограничение в 1 ТБ на узел. Можно также масштабировать кластер до большего размера, который будет распределять размер базы данных tempdb между большим количеством узлов, уменьшая базу данных tempdb на каждом отдельном узле.

В дополнение к инструкциям CTAS и INSERT SELECT, большие сложные запросы, работающие с недостаточным объемом памяти, могут сбрасываться в базу данных tempdb, что приводит к сбою запросов.  Рассмотрите возможность запуска с большим [классом ресурсов](resource-classes-for-workload-management.md) , чтобы избежать переноса данных в tempdb.

## <a name="monitor-memory"></a>Мониторинг памяти

Объем может стать основной причиной низкой производительности и нехватки памяти. Вы можете рассмотреть масштабирование хранилища данных, если во время выполнения запроса наблюдается активное использование памяти SQL Server (приближение к пределам).

Следующий запрос возвращает использование памяти SQL Server и нехватку памяти на каждом узле:

```sql
-- Memory consumption
SELECT
  pc1.cntr_value as Curr_Mem_KB,
  pc1.cntr_value/1024.0 as Curr_Mem_MB,
  (pc1.cntr_value/1048576.0) as Curr_Mem_GB,
  pc2.cntr_value as Max_Mem_KB,
  pc2.cntr_value/1024.0 as Max_Mem_MB,
  (pc2.cntr_value/1048576.0) as Max_Mem_GB,
  pc1.cntr_value * 100.0/pc2.cntr_value AS Memory_Utilization_Percentage,
  pc1.pdw_node_id
FROM
-- pc1: current memory
sys.dm_pdw_nodes_os_performance_counters AS pc1
-- pc2: total memory allowed for this SQL instance
JOIN sys.dm_pdw_nodes_os_performance_counters AS pc2
ON pc1.object_name = pc2.object_name AND pc1.pdw_node_id = pc2.pdw_node_id
WHERE
pc1.counter_name = 'Total Server Memory (KB)'
AND pc2.counter_name = 'Target Server Memory (KB)'
```

## <a name="monitor-transaction-log-size"></a>Мониторинг размера журнала транзакций

Следующий запрос возвращает размер журнала транзакций для каждого распределения. Если один из файлов журнала достигает размера 160 ГБ, следует рассмотреть вертикальное масштабирование экземпляра или ограничение размера транзакции.

```sql
-- Transaction log size
SELECT
  instance_name as distribution_db,
  cntr_value*1.0/1048576 as log_file_size_used_GB,
  pdw_node_id
FROM sys.dm_pdw_nodes_os_performance_counters
WHERE
instance_name like 'Distribution_%'
AND counter_name = 'Log File(s) Used Size (KB)'
```

## <a name="monitor-transaction-log-rollback"></a>Мониторинг отката журнала транзакций

Если запросы завершаются сбоем или требуют слишком много времени на выполнение, вы можете отследить, происходит ли откат транзакций.

```sql
-- Monitor rollback
SELECT
    SUM(CASE WHEN t.database_transaction_next_undo_lsn IS NOT NULL THEN 1 ELSE 0 END),
    t.pdw_node_id,
    nod.[type]
FROM sys.dm_pdw_nodes_tran_database_transactions t
JOIN sys.dm_pdw_nodes nod ON t.pdw_node_id = nod.pdw_node_id
GROUP BY t.pdw_node_id, nod.[type]
```

## <a name="monitor-polybase-load"></a>Мониторинг загрузки Polybase

Следующий запрос позволяет приблизительно оценить ход выполнения загрузки. В запросе отображаются только те файлы, которые обрабатываются в данный момент.

```sql

-- To track bytes and files
SELECT
    r.command,
    s.request_id,
    r.status,
    count(distinct input_name) as nbr_files,
    sum(s.bytes_processed)/1024/1024/1024 as gb_processed
FROM
    sys.dm_pdw_exec_requests r
    inner join sys.dm_pdw_dms_external_work s
        on r.request_id = s.request_id
GROUP BY
    r.command,
    s.request_id,
    r.status
ORDER BY
    nbr_files desc,
    gb_processed desc;
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о динамических административных представлениях см. в статье о [системных представлениях](../sql/reference-tsql-system-views.md).
