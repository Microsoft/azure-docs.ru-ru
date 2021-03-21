---
title: Анализ рабочей нагрузки для выделенного пула SQL
description: Методы анализа приоритетов запросов для выделенного пула SQL в Azure синапсе Analytics.
services: synapse-analytics
author: ronortloff
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: rortloff
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 14c3ad30bac7cec4c11822d825323bb9db2ba440
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96454531"
---
# <a name="analyze-your-workload-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Анализ рабочей нагрузки для выделенного пула SQL в Azure синапсе Analytics

Методы анализа выделенной рабочей нагрузки пула SQL в Azure синапсе Analytics. 

## <a name="resource-classes"></a>Классы ресурсов

Выделенный пул SQL предоставляет классы ресурсов для назначения системных ресурсов запросам.  Дополнительные сведения о классах ресурсов см. в разделе [классы ресурсов & Управление рабочей нагрузкой](resource-classes-for-workload-management.md).  Запросы будут ожидать, если класс ресурсов, назначенный запросу, требует больше ресурсов, чем доступно в данный момент.

## <a name="queued-query-detection-and-other-dmvs"></a>Представление, используемое для определения запросов, поставленных в очередь, и другие динамические административные представления

Определить запросы, попавшие в очередь параллельной обработки, можно с помощью динамического административного представления `sys.dm_pdw_exec_requests` . Запросы, ожидающие выделения слота параллелизма, находятся в **приостановленном** состоянии.

```sql
SELECT  r.[request_id]                           AS Request_ID
,       r.[status]                               AS Request_Status
,       r.[submit_time]                          AS Request_SubmitTime
,       r.[start_time]                           AS Request_StartTime
,       DATEDIFF(ms,[submit_time],[start_time])  AS Request_InitiateDuration_ms
,       r.resource_class                         AS Request_resource_class
FROM    sys.dm_pdw_exec_requests r
;
```

Чтобы просмотреть роли управления рабочей нагрузкой, можно использовать представление `sys.database_principals`.

```sql
SELECT  ro.[name]           AS [db_role_name]
FROM    sys.database_principals ro
WHERE   ro.[type_desc]      = 'DATABASE_ROLE'
AND     ro.[is_fixed_role]  = 0
;
```

Следующий запрос позволяет определить, в какие роли добавлены пользователи.

```sql
SELECT  r.name AS role_principal_name
,       m.name AS member_principal_name
FROM    sys.database_role_members rm
JOIN    sys.database_principals AS r            ON rm.role_principal_id      = r.principal_id
JOIN    sys.database_principals AS m            ON rm.member_principal_id    = m.principal_id
WHERE   r.name IN ('mediumrc','largerc','xlargerc')
;
```

Выделенный пул SQL имеет следующие типы ожидания:

* **LocalQueriesConcurrencyResourceType** относится к запросам, которые не входят в платформу слотов выдачи. В качестве примеров таких запросов можно привести запросы и системные функции динамических административных представлений, такие как `SELECT @@VERSION` .
* **UserConcurrencyResourceType** относится к запросам, которые входят в платформу слотов выдачи. В качестве примеров использования этого типа ресурсов можно привести запросы к таблицам пользователя.
* **DmsConcurrencyResourceType** относится к ожиданиям, связанным с операциями перемещения данных.
* **BackupConcurrencyResourceType** может использоваться при создании резервной копии базы данных. Максимальное значение для этого типа ресурсов равно 1. При одновременном запросе сразу нескольких резервных копий все остальные запросы помещаются в очередь. Как правило, рекомендуется использовать минимальное время между последовательными моментальными снимками за 10 минут.

Определить, какие ресурсы необходимы для запроса, можно при помощи динамического административного представления `sys.dm_pdw_waits` .

```sql
SELECT  w.[wait_id]
,       w.[session_id]
,       w.[type]                                           AS Wait_type
,       w.[object_type]
,       w.[object_name]
,       w.[request_id]
,       w.[request_time]
,       w.[acquire_time]
,       w.[state]
,       w.[priority]
,       SESSION_ID()                                       AS Current_session
,       s.[status]                                         AS Session_status
,       s.[login_name]
,       s.[query_count]
,       s.[client_id]
,       s.[sql_spid]
,       r.[command]                                        AS Request_command
,       r.[label]
,       r.[status]                                         AS Request_status
,       r.[submit_time]
,       r.[start_time]
,       r.[end_compile_time]
,       r.[end_time]
,       DATEDIFF(ms,r.[submit_time],r.[start_time])        AS Request_queue_time_ms
,       DATEDIFF(ms,r.[start_time],r.[end_compile_time])   AS Request_compile_time_ms
,       DATEDIFF(ms,r.[end_compile_time],r.[end_time])     AS Request_execution_time_ms
,       r.[total_elapsed_time]
FROM    sys.dm_pdw_waits w
JOIN    sys.dm_pdw_exec_sessions s  ON w.[session_id] = s.[session_id]
JOIN    sys.dm_pdw_exec_requests r  ON w.[request_id] = r.[request_id]
WHERE    w.[session_id] <> SESSION_ID()
;
```

В `sys.dm_pdw_resource_waits` динамическом административном отображении отображаются сведения о времени ожидания для заданного запроса. Время ожидания ресурсов измеряет время ожидания предоставляемых ресурсов. Время ожидания сигнала — это время, которое занимается базовыми серверами SQL для планирования запроса на загрузку ЦП.

```sql
SELECT  [session_id]
,       [type]
,       [object_type]
,       [object_name]
,       [request_id]
,       [request_time]
,       [acquire_time]
,       DATEDIFF(ms,[request_time],[acquire_time])  AS acquire_duration_ms
,       [concurrency_slots_used]                    AS concurrency_slots_reserved
,       [resource_class]
,       [wait_id]                                   AS queue_position
FROM    sys.dm_pdw_resource_waits
WHERE    [session_id] <> SESSION_ID()
;
```

Вы также можете использовать DMV `sys.dm_pdw_resource_waits`, чтобы вычислить, сколько слотов выдачи было предоставлено.

```sql
SELECT  SUM([concurrency_slots_used]) as total_granted_slots
FROM    sys.[dm_pdw_resource_waits]
WHERE   [state]           = 'Granted'
AND     [resource_class] is not null
AND     [session_id]     <> session_id()
;
```

Для анализа тенденций времени ожидания за прошедший период используется динамическое административное представление `sys.dm_pdw_wait_stats` .

```sql
SELECT   w.[pdw_node_id]
,        w.[wait_name]
,        w.[max_wait_time]
,        w.[request_count]
,        w.[signal_time]
,        w.[completed_count]
,        w.[wait_time]
FROM    sys.dm_pdw_wait_stats w
;
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об управлении пользователями и безопасностью баз данных см. [в разделе Защита выделенного пула SQL (ранее — хранилище SQL DW)](sql-data-warehouse-overview-manage-security.md). Дополнительные сведения о том, как крупные классы ресурсов могут улучшить качество кластеризованных индексов columnstore, см. [в разделе перестроение индексов для улучшения качества сегментов](sql-data-warehouse-tables-index.md#rebuilding-indexes-to-improve-segment-quality).
