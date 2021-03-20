---
title: Мониторинг производительности с помощью динамических административных представлений
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Узнайте, как обнаруживать и диагностировать распространенные проблемы с производительностью, используя динамические административные представления для мониторинга База данных SQL Microsoft Azure и Управляемый экземпляр SQL Azure.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 03/15/2021
ms.openlocfilehash: 5c0de2c1589bfa495ab6ad287b998c403041674c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104592172"
---
# <a name="monitoring-microsoft-azure-sql-database-and-azure-sql-managed-instance-performance-using-dynamic-management-views"></a>Наблюдение за производительностью Базы данных SQL Microsoft Azure и Управляемого экземпляра SQL Azure с помощью динамических административных представлений
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

База данных SQL Microsoft Azure и Azure SQL Управляемый экземпляр позволяют подмножества динамических административных представлений для диагностики проблем с производительностью, которые могут быть вызваны блокированием или длительными запросами, узкими местами ресурсов, неудачными планами запросов и т. д. Эта статья содержит сведения о том, как обнаружить распространенные проблемы с производительностью с помощью динамических административных представлений.

База данных SQL Microsoft Azure и Azure SQL Управляемый экземпляр частично поддерживают три категории динамических административных представлений:

- динамические представления управления, относящиеся к базам данных;
- динамические представления управления, относящиеся к выполнению;
- динамические представления управления, относящиеся к транзакциям.

Подробные сведения о динамических административных представлениях см. в разделе [динамические административные представления и функции (Transact-SQL)](/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views).

## <a name="monitor-with-sql-insights"></a>Мониторинг с помощью SQL Insights

[Azure Monitor SQL Insights](../../azure-monitor/insights/sql-insights-overview.md) — это средство для мониторинга управляемых экземпляров Azure SQL, баз данных SQL Azure и экземпляров SQL Server на виртуальных машинах SQL Azure. Эта служба использует удаленный агент для записи данных из динамических административных представлений и направляет их в Log Analytics Azure, где их можно отслеживать и анализировать. Эти данные можно просмотреть [Azure Monitor](../../azure-monitor/overview.md) в предоставленных представлениях или получить доступ к данным журнала непосредственно для выполнения запросов и анализа тенденций. Инструкции по началу работы с Azure Monitor SQL Insights см. в разделе [Включение SQL](../../azure-monitor/insights/sql-insights-enable.md)Insights.

## <a name="permissions"></a>Разрешения

В базе данных SQL Azure для запросов к динамическому административному представлению требуется разрешение **Просмотр состояния базы данных** . Разрешение **VIEW DATABASE STATE** возвращает сведения обо всех объектах в текущей базе данных.
Чтобы предоставить разрешение **VIEW DATABASE STATE** определенному пользователю базы данных, выполните следующий запрос:

```sql
GRANT VIEW DATABASE STATE TO database_user;
```

В Управляемый экземпляр SQL Azure для запроса динамического административного представления требуются разрешения **View Server State** . Дополнительные сведения см. в разделе [системные динамические административные представления](/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views#required-permissions).

В экземпляре SQL Server и в Управляемый экземпляр SQL Azure динамические административные представления возвращают сведения о состоянии сервера. В базе данных SQL Azure они возвращают сведения только о текущей логической базе данных.

Эта статья содержит коллекцию запросов динамического административного представления, которые можно выполнять с помощью SQL Server Management Studio или Azure Data Studio, чтобы обнаружить следующие типы проблем с производительностью запросов.

- [Определение запросов, связанных с чрезмерным потреблением ресурсов ЦП](#identify-cpu-performance-issues)
- [PAGELATCH_ * и WRITE_LOG ожидания, связанные с узкими местами ввода-вывода](#identify-io-performance-issues)
- [PAGELATCH_ * ожидания вызваны состязанием за Биттемпдб](#identify-tempdb-performance-issues)
- [RESOURCE_SEMAHPORE ожиданий, вызванных проблемами предоставления памяти](#identify-memory-grant-wait-performance-issues)
- [Определение размеров базы данных и объекта](#calculating-database-and-objects-sizes)
- [Получение сведений об активных сеансах](#monitoring-connections)
- [Получение сведений об использовании ресурсов в масштабе всей системы и базы данных](#monitor-resource-use)
- [Получение сведений о производительности запроса](#monitoring-query-performance)

## <a name="identify-cpu-performance-issues"></a>Выявление проблем производительности ЦП

Если загрузка ЦП превышает 80 % на длительное время, попробуйте выполнить следующие действия:

### <a name="the-cpu-issue-is-occurring-now"></a>Проблема с ЦП происходит сейчас

Если проблема происходит прямо сейчас, существует два возможных сценария:

#### <a name="many-individual-queries-that-cumulatively-consume-high-cpu"></a>Много отдельных запросов, которые вместе потребляют большой объем ЦП

Используйте следующий запрос для идентификации самых ресурсоемких хэшей запросов:

```sql
PRINT '-- top 10 Active CPU Consuming Queries (aggregated)--';
SELECT TOP 10 GETDATE() runtime, *
FROM (SELECT query_stats.query_hash, SUM(query_stats.cpu_time) 'Total_Request_Cpu_Time_Ms', SUM(logical_reads) 'Total_Request_Logical_Reads', MIN(start_time) 'Earliest_Request_start_Time', COUNT(*) 'Number_Of_Requests', SUBSTRING(REPLACE(REPLACE(MIN(query_stats.statement_text), CHAR(10), ' '), CHAR(13), ' '), 1, 256) AS "Statement_Text"
    FROM (SELECT req.*, SUBSTRING(ST.text, (req.statement_start_offset / 2)+1, ((CASE statement_end_offset WHEN -1 THEN DATALENGTH(ST.text)ELSE req.statement_end_offset END-req.statement_start_offset)/ 2)+1) AS statement_text
          FROM sys.dm_exec_requests AS req
                CROSS APPLY sys.dm_exec_sql_text(req.sql_handle) AS ST ) AS query_stats
    GROUP BY query_hash) AS t
ORDER BY Total_Request_Cpu_Time_Ms DESC;
```

#### <a name="long-running-queries-that-consume-cpu-are-still-running"></a>Длительные запросы, использующие ЦП, по-прежнему выполняются

Используйте следующий запрос для идентификации этих запросов:

```sql
PRINT '--top 10 Active CPU Consuming Queries by sessions--';
SELECT TOP 10 req.session_id, req.start_time, cpu_time 'cpu_time_ms', OBJECT_NAME(ST.objectid, ST.dbid) 'ObjectName', SUBSTRING(REPLACE(REPLACE(SUBSTRING(ST.text, (req.statement_start_offset / 2)+1, ((CASE statement_end_offset WHEN -1 THEN DATALENGTH(ST.text)ELSE req.statement_end_offset END-req.statement_start_offset)/ 2)+1), CHAR(10), ' '), CHAR(13), ' '), 1, 512) AS statement_text
FROM sys.dm_exec_requests AS req
    CROSS APPLY sys.dm_exec_sql_text(req.sql_handle) AS ST
ORDER BY cpu_time DESC;
GO
```

### <a name="the-cpu-issue-occurred-in-the-past"></a>Проблема с ЦП была в прошлом

Если проблема возникла в прошлом и вы хотите найти первопричину, используйте [хранилище запросов](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store). Пользователи с доступом к базе данных могут использовать T-SQL для запроса данных из хранилища запросов. В конфигурации хранилища запросов по умолчанию используется степень детализации 1 час. Используйте следующий запрос для поиска деятельности ресурсоемких запросов. Этот запрос возвращает 15 самых ресурсоемких запросов. Не забудьте изменить `rsi.start_time >= DATEADD(hour, -2, GETUTCDATE()`:

```sql
-- Top 15 CPU consuming queries by query hash
-- note that a query  hash can have many query id if not parameterized or not parameterized properly
-- it grabs a sample query text by min
WITH AggregatedCPU AS (SELECT q.query_hash, SUM(count_executions * avg_cpu_time / 1000.0) AS total_cpu_millisec, SUM(count_executions * avg_cpu_time / 1000.0)/ SUM(count_executions) AS avg_cpu_millisec, MAX(rs.max_cpu_time / 1000.00) AS max_cpu_millisec, MAX(max_logical_io_reads) max_logical_reads, COUNT(DISTINCT p.plan_id) AS number_of_distinct_plans, COUNT(DISTINCT p.query_id) AS number_of_distinct_query_ids, SUM(CASE WHEN rs.execution_type_desc='Aborted' THEN count_executions ELSE 0 END) AS Aborted_Execution_Count, SUM(CASE WHEN rs.execution_type_desc='Regular' THEN count_executions ELSE 0 END) AS Regular_Execution_Count, SUM(CASE WHEN rs.execution_type_desc='Exception' THEN count_executions ELSE 0 END) AS Exception_Execution_Count, SUM(count_executions) AS total_executions, MIN(qt.query_sql_text) AS sampled_query_text
                       FROM sys.query_store_query_text AS qt
                            JOIN sys.query_store_query AS q ON qt.query_text_id=q.query_text_id
                            JOIN sys.query_store_plan AS p ON q.query_id=p.query_id
                            JOIN sys.query_store_runtime_stats AS rs ON rs.plan_id=p.plan_id
                            JOIN sys.query_store_runtime_stats_interval AS rsi ON rsi.runtime_stats_interval_id=rs.runtime_stats_interval_id
                       WHERE rs.execution_type_desc IN ('Regular', 'Aborted', 'Exception')AND rsi.start_time>=DATEADD(HOUR, -2, GETUTCDATE())
                       GROUP BY q.query_hash), OrderedCPU AS (SELECT query_hash, total_cpu_millisec, avg_cpu_millisec, max_cpu_millisec, max_logical_reads, number_of_distinct_plans, number_of_distinct_query_ids, total_executions, Aborted_Execution_Count, Regular_Execution_Count, Exception_Execution_Count, sampled_query_text, ROW_NUMBER() OVER (ORDER BY total_cpu_millisec DESC, query_hash ASC) AS RN
                                                              FROM AggregatedCPU)
SELECT OD.query_hash, OD.total_cpu_millisec, OD.avg_cpu_millisec, OD.max_cpu_millisec, OD.max_logical_reads, OD.number_of_distinct_plans, OD.number_of_distinct_query_ids, OD.total_executions, OD.Aborted_Execution_Count, OD.Regular_Execution_Count, OD.Exception_Execution_Count, OD.sampled_query_text, OD.RN
FROM OrderedCPU AS OD
WHERE OD.RN<=15
ORDER BY total_cpu_millisec DESC;
```

Когда вы найдете проблемные запросы, настройте их для снижения нагрузки на ЦП.  Если у вас нет времени на настройку запросов, вы также можете обновить SLO базы данных, чтобы обойти эту проблему.

## <a name="identify-io-performance-issues"></a>Поиск проблем производительности операций ввода-вывода

При определении проблем с производительностью операций ввода-вывода учтите главные типы ожидания, связанные с проблемами ввода-вывода:

- `PAGEIOLATCH_*`

  Для ошибок ввода-вывода в файлах данных (включая `PAGEIOLATCH_SH`, `PAGEIOLATCH_EX`, `PAGEIOLATCH_UP`).  Если имя типа ожидания имеет в нем **операции ввода-вывода** , оно указывает на ошибку ввода-вывода. Если в имени ожидания кратковременной блокировки страницы нет **операций ввода-вывода** , он указывает на другой тип проблемы (например, за состязанием за tempdb).

- `WRITE_LOG`

  Для проблем ввода-вывода в журнале транзакций.

### <a name="if-the-io-issue-is-occurring-right-now"></a>Если проблема ввода-вывода происходит прямо сейчас

Используйте [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) или [sys.dm_os_waiting_tasks](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql), чтобы посмотреть `wait_type` и `wait_time`.

#### <a name="identify-data-and-log-io-usage"></a>Определение использования ввода-вывода для данных и журнала

Используйте следующий запрос для идентификации использования ввода-вывода для данных и журналов. Если значение ввода-вывода данных или журнала превышает 80%, это означает, что пользователи использовали доступные операции ввода-вывода для уровня службы базы данных SQL.

```sql
SELECT end_time, avg_data_io_percent, avg_log_write_percent
FROM sys.dm_db_resource_stats
ORDER BY end_time DESC;
```

Если достигнут лимит операций ввода-вывода, у вас есть два варианта:

- Вариант 1: повышение объема вычислительных ресурсов или уровня служб.
- Вариант 2: определение и настройка запросов, потребляющих наибольший объем операций ввода-вывода.

#### <a name="view-buffer-related-io-using-the-query-store"></a>Просмотр операций ввода-вывода, связанных с буфером, с помощью хранилища запросов

Во втором варианте используйте следующий запрос к хранилищу запросов об операциях ввода-вывода, связанных с буфером, для просмотра действий за последние два часа:

```sql
-- top queries that waited on buffer
-- note these are finished queries
WITH Aggregated AS (SELECT q.query_hash, SUM(total_query_wait_time_ms) total_wait_time_ms, SUM(total_query_wait_time_ms / avg_query_wait_time_ms) AS total_executions, MIN(qt.query_sql_text) AS sampled_query_text, MIN(wait_category_desc) AS wait_category_desc
                    FROM sys.query_store_query_text AS qt
                         JOIN sys.query_store_query AS q ON qt.query_text_id=q.query_text_id
                         JOIN sys.query_store_plan AS p ON q.query_id=p.query_id
                         JOIN sys.query_store_wait_stats AS waits ON waits.plan_id=p.plan_id
                         JOIN sys.query_store_runtime_stats_interval AS rsi ON rsi.runtime_stats_interval_id=waits.runtime_stats_interval_id
                    WHERE wait_category_desc='Buffer IO' AND rsi.start_time>=DATEADD(HOUR, -2, GETUTCDATE())
                    GROUP BY q.query_hash), Ordered AS (SELECT query_hash, total_executions, total_wait_time_ms, sampled_query_text, wait_category_desc, ROW_NUMBER() OVER (ORDER BY total_wait_time_ms DESC, query_hash ASC) AS RN
                                                        FROM Aggregated)
SELECT OD.query_hash, OD.total_executions, OD.total_wait_time_ms, OD.sampled_query_text, OD.wait_category_desc, OD.RN
FROM Ordered AS OD
WHERE OD.RN<=15
ORDER BY total_wait_time_ms DESC;
GO
```

#### <a name="view-total-log-io-for-writelog-waits"></a>Просмотр всех операций ввода-вывода журнала для ожиданий WRITELOG

Если тип ожидания — `WRITELOG`, используйте следующий запрос, чтобы просмотреть все операции ввода-вывода журнала по оператору:

```sql
-- Top transaction log consumers
-- Adjust the time window by changing
-- rsi.start_time >= DATEADD(hour, -2, GETUTCDATE())
WITH AggregatedLogUsed
AS (SELECT q.query_hash,
           SUM(count_executions * avg_cpu_time / 1000.0) AS total_cpu_millisec,
           SUM(count_executions * avg_cpu_time / 1000.0) / SUM(count_executions) AS avg_cpu_millisec,
           SUM(count_executions * avg_log_bytes_used) AS total_log_bytes_used,
           MAX(rs.max_cpu_time / 1000.00) AS max_cpu_millisec,
           MAX(max_logical_io_reads) max_logical_reads,
           COUNT(DISTINCT p.plan_id) AS number_of_distinct_plans,
           COUNT(DISTINCT p.query_id) AS number_of_distinct_query_ids,
           SUM(   CASE
                      WHEN rs.execution_type_desc = 'Aborted' THEN
                          count_executions
                      ELSE
                          0
                  END
              ) AS Aborted_Execution_Count,
           SUM(   CASE
                      WHEN rs.execution_type_desc = 'Regular' THEN
                          count_executions
                      ELSE
                          0
                  END
              ) AS Regular_Execution_Count,
           SUM(   CASE
                      WHEN rs.execution_type_desc = 'Exception' THEN
                          count_executions
                      ELSE
                          0
                  END
              ) AS Exception_Execution_Count,
           SUM(count_executions) AS total_executions,
           MIN(qt.query_sql_text) AS sampled_query_text
    FROM sys.query_store_query_text AS qt
        JOIN sys.query_store_query AS q
            ON qt.query_text_id = q.query_text_id
        JOIN sys.query_store_plan AS p
            ON q.query_id = p.query_id
        JOIN sys.query_store_runtime_stats AS rs
            ON rs.plan_id = p.plan_id
        JOIN sys.query_store_runtime_stats_interval AS rsi
            ON rsi.runtime_stats_interval_id = rs.runtime_stats_interval_id
    WHERE rs.execution_type_desc IN ( 'Regular', 'Aborted', 'Exception' )
          AND rsi.start_time >= DATEADD(HOUR, -2, GETUTCDATE())
    GROUP BY q.query_hash),
     OrderedLogUsed
AS (SELECT query_hash,
           total_log_bytes_used,
           number_of_distinct_plans,
           number_of_distinct_query_ids,
           total_executions,
           Aborted_Execution_Count,
           Regular_Execution_Count,
           Exception_Execution_Count,
           sampled_query_text,
           ROW_NUMBER() OVER (ORDER BY total_log_bytes_used DESC, query_hash ASC) AS RN
    FROM AggregatedLogUsed)
SELECT OD.total_log_bytes_used,
       OD.number_of_distinct_plans,
       OD.number_of_distinct_query_ids,
       OD.total_executions,
       OD.Aborted_Execution_Count,
       OD.Regular_Execution_Count,
       OD.Exception_Execution_Count,
       OD.sampled_query_text,
       OD.RN
FROM OrderedLogUsed AS OD
WHERE OD.RN <= 15
ORDER BY total_log_bytes_used DESC;
GO
```

## <a name="identify-tempdb-performance-issues"></a>Поиск проблем производительности `tempdb`

При определении проблем с производительностью операций ввода-вывода главные типы ожидания, связанные с проблемами `tempdb`, — `PAGELATCH_*` (не `PAGEIOLATCH_*`). Однако ожидания `PAGELATCH_*` не всегда означают состязание `tempdb`.  Этот тип ожидания может также указывать на состязание за страницы данных объекта пользователя из-за одновременных запросов к одной странице данных. Чтобы еще больше подтвердить `tempdb` состязание, используйте [sys.dm_exec_requests](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) , чтобы убедиться, что значение wait_resource начинается с, `2:x:y` где 2 — `tempdb` идентификатор базы данных, `x` — идентификатор файла, а `y` — идентификатор страницы.  

Для состязаний tempdb распространенным методом является сокращение или перезапись кода приложения, который зависит от `tempdb` .  Распространенные области использования `tempdb`:

- Временные таблицы
- Табличные переменные
- Возвращающие табличные значения параметры
- Использование хранилища версий (связано с долго выполняющимися транзакциями)
- Запросы с планами запросов, которые используют сортировку, хэш-соединения и буферы

### <a name="top-queries-that-use-table-variables-and-temporary-tables"></a>Основные запросы, использующие табличные переменные и временные таблицы

Используйте следующий запрос для идентификации основных запросов, использующих табличные переменные и временные таблицы:

```sql
SELECT plan_handle, execution_count, query_plan
INTO #tmpPlan
FROM sys.dm_exec_query_stats
     CROSS APPLY sys.dm_exec_query_plan(plan_handle);
GO

WITH XMLNAMESPACES('http://schemas.microsoft.com/sqlserver/2004/07/showplan' AS sp)
SELECT plan_handle, stmt.stmt_details.value('@Database', 'varchar(max)') 'Database', stmt.stmt_details.value('@Schema', 'varchar(max)') 'Schema', stmt.stmt_details.value('@Table', 'varchar(max)') 'table'
INTO #tmp2
FROM(SELECT CAST(query_plan AS XML) sqlplan, plan_handle FROM #tmpPlan) AS p
    CROSS APPLY sqlplan.nodes('//sp:Object') AS stmt(stmt_details);
GO

SELECT t.plan_handle, [Database], [Schema], [table], execution_count
FROM(SELECT DISTINCT plan_handle, [Database], [Schema], [table]
     FROM #tmp2
     WHERE [table] LIKE '%@%' OR [table] LIKE '%#%') AS t
    JOIN #tmpPlan AS t2 ON t.plan_handle=t2.plan_handle;
```

### <a name="identify-long-running-transactions"></a>Выявление длительных транзакций

Используйте следующий запрос, чтобы определить длительные транзакции. Длительные транзакции мешают очистке хранилища версий.

```sql
SELECT DB_NAME(dtr.database_id) 'database_name',
       sess.session_id,
       atr.name AS 'tran_name',
       atr.transaction_id,
       transaction_type,
       transaction_begin_time,
       database_transaction_begin_time transaction_state,
       is_user_transaction,
       sess.open_transaction_count,
       LTRIM(RTRIM(REPLACE(
                              REPLACE(
                                         SUBSTRING(
                                                      SUBSTRING(
                                                                   txt.text,
                                                                   (req.statement_start_offset / 2) + 1,
                                                                   ((CASE req.statement_end_offset
                                                                         WHEN -1 THEN
                                                                             DATALENGTH(txt.text)
                                                                         ELSE
                                                                             req.statement_end_offset
                                                                     END - req.statement_start_offset
                                                                    ) / 2
                                                                   ) + 1
                                                               ),
                                                      1,
                                                      1000
                                                  ),
                                         CHAR(10),
                                         ' '
                                     ),
                              CHAR(13),
                              ' '
                          )
                  )
            ) Running_stmt_text,
       recenttxt.text 'MostRecentSQLText'
FROM sys.dm_tran_active_transactions AS atr
    INNER JOIN sys.dm_tran_database_transactions AS dtr
        ON dtr.transaction_id = atr.transaction_id
    LEFT JOIN sys.dm_tran_session_transactions AS sess
        ON sess.transaction_id = atr.transaction_id
    LEFT JOIN sys.dm_exec_requests AS req
        ON req.session_id = sess.session_id
           AND req.transaction_id = sess.transaction_id
    LEFT JOIN sys.dm_exec_connections AS conn
        ON sess.session_id = conn.session_id
    OUTER APPLY sys.dm_exec_sql_text(req.sql_handle) AS txt
    OUTER APPLY sys.dm_exec_sql_text(conn.most_recent_sql_handle) AS recenttxt
WHERE atr.transaction_type != 2
      AND sess.session_id != @@spid
ORDER BY start_time ASC;
```

## <a name="identify-memory-grant-wait-performance-issues"></a>Определение проблем с производительностью ожидания временно предоставляемого буфера памяти

Если основной тип ожидания — `RESOURCE_SEMAHPORE` и у вас нет проблем с загрузкой ЦП, у вас проблема с ожиданием временно предоставляемого буфера памяти.

### <a name="determine-if-a-resource_semahpore-wait-is-a-top-wait"></a>Определение типа ожидания `RESOURCE_SEMAHPORE` в качестве основного

Используйте следующий запрос для определения того, что тип ожидания `RESOURCE_SEMAHPORE` является основным

```sql
SELECT wait_type,
       SUM(wait_time) AS total_wait_time_ms
FROM sys.dm_exec_requests AS req
    JOIN sys.dm_exec_sessions AS sess
        ON req.session_id = sess.session_id
WHERE is_user_process = 1
GROUP BY wait_type
ORDER BY SUM(wait_time) DESC;
```

### <a name="identify-high-memory-consuming-statements"></a>Выявление чрезмерно потребляемой памятью инструкций

Используйте следующий запрос для определения операторов, потребляющих большой объем памяти:

```sql
SELECT IDENTITY(INT, 1, 1) rowId,
    CAST(query_plan AS XML) query_plan,
    p.query_id
INTO #tmp
FROM sys.query_store_plan AS p
    JOIN sys.query_store_runtime_stats AS r
        ON p.plan_id = r.plan_id
    JOIN sys.query_store_runtime_stats_interval AS i
        ON r.runtime_stats_interval_id = i.runtime_stats_interval_id
WHERE start_time > '2018-10-11 14:00:00.0000000'
      AND end_time < '2018-10-17 20:00:00.0000000';
GO
;WITH cte
AS (SELECT query_id,
        query_plan,
        m.c.value('@SerialDesiredMemory', 'INT') AS SerialDesiredMemory
    FROM #tmp AS t
        CROSS APPLY t.query_plan.nodes('//*:MemoryGrantInfo[@SerialDesiredMemory[. > 0]]') AS m(c) )
SELECT TOP 50
    cte.query_id,
    t.query_sql_text,
    cte.query_plan,
    CAST(SerialDesiredMemory / 1024. AS DECIMAL(10, 2)) SerialDesiredMemory_MB
FROM cte
    JOIN sys.query_store_query AS q
        ON cte.query_id = q.query_id
    JOIN sys.query_store_query_text AS t
        ON q.query_text_id = t.query_text_id
ORDER BY SerialDesiredMemory DESC;
```

### <a name="identify-the-top-10-active-memory-grants"></a>Определение десяти основных временно предоставляемых буферов памяти

Используйте следующий запрос для определения десяти основных временно предоставляемых буферов памяти:

```sql
SELECT TOP 10
    CONVERT(VARCHAR(30), GETDATE(), 121) AS runtime,
       r.session_id,
       r.blocking_session_id,
       r.cpu_time,
       r.total_elapsed_time,
       r.reads,
       r.writes,
       r.logical_reads,
       r.row_count,
       wait_time,
       wait_type,
       r.command,
       OBJECT_NAME(txt.objectid, txt.dbid) 'Object_Name',
       LTRIM(RTRIM(REPLACE(
                              REPLACE(
                                         SUBSTRING(
                                                      SUBSTRING(
                                                                   text,
                                                                   (r.statement_start_offset / 2) + 1,
                                                                   ((CASE r.statement_end_offset
                                                                         WHEN -1 THEN
                                                                             DATALENGTH(text)
                                                                         ELSE
                                                                             r.statement_end_offset
                                                                     END - r.statement_start_offset
                                                                    ) / 2
                                                                   ) + 1
                                                               ),
                                                      1,
                                                      1000
                                                  ),
                                         CHAR(10),
                                         ' '
                                     ),
                              CHAR(13),
                              ' '
                          )
                  )
            ) stmt_text,
       mg.dop,                                               --Degree of parallelism
       mg.request_time,                                      --Date and time when this query requested the memory grant.
       mg.grant_time,                                        --NULL means memory has not been granted
       mg.requested_memory_kb / 1024.0 requested_memory_mb,  --Total requested amount of memory in megabytes
       mg.granted_memory_kb / 1024.0 AS granted_memory_mb,   --Total amount of memory actually granted in megabytes. NULL if not granted
       mg.required_memory_kb / 1024.0 AS required_memory_mb, --Minimum memory required to run this query in megabytes.
       max_used_memory_kb / 1024.0 AS max_used_memory_mb,
       mg.query_cost,                                        --Estimated query cost.
       mg.timeout_sec,                                       --Time-out in seconds before this query gives up the memory grant request.
       mg.resource_semaphore_id,                             --Non-unique ID of the resource semaphore on which this query is waiting.
       mg.wait_time_ms,                                      --Wait time in milliseconds. NULL if the memory is already granted.
       CASE mg.is_next_candidate --Is this process the next candidate for a memory grant
           WHEN 1 THEN
               'Yes'
           WHEN 0 THEN
               'No'
           ELSE
               'Memory has been granted'
       END AS 'Next Candidate for Memory Grant',
       qp.query_plan
FROM sys.dm_exec_requests AS r
    JOIN sys.dm_exec_query_memory_grants AS mg
        ON r.session_id = mg.session_id
           AND r.request_id = mg.request_id
    CROSS APPLY sys.dm_exec_sql_text(mg.sql_handle) AS txt
    CROSS APPLY sys.dm_exec_query_plan(r.plan_handle) AS qp
ORDER BY mg.granted_memory_kb DESC;
```

## <a name="calculating-database-and-objects-sizes"></a>Вычисление размера базы данных и объектов

Следующий запрос возвращает размер базы данных в мегабайтах:

```sql
-- Calculates the size of the database.
SELECT SUM(CAST(FILEPROPERTY(name, 'SpaceUsed') AS bigint) * 8192.) / 1024 / 1024 AS DatabaseSizeInMB
FROM sys.database_files
WHERE type_desc = 'ROWS';
GO
```

Следующий запрос возвращает размер отдельных объектов базы данных в мегабайтах:

```sql
-- Calculates the size of individual database objects.
SELECT sys.objects.name, SUM(reserved_page_count) * 8.0 / 1024
FROM sys.dm_db_partition_stats, sys.objects
WHERE sys.dm_db_partition_stats.object_id = sys.objects.object_id
GROUP BY sys.objects.name;
GO
```

## <a name="monitoring-connections"></a>Мониторинг подключений

Представление [sys.dm_exec_connections](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-connections-transact-sql) можно использовать для получения сведений о соединениях, установленных для конкретного сервера и управляемого экземпляра, а также сведения о каждом соединении. Кроме того, представление [sys.dm_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql) позволяет получить сведения обо всех активных подключениях пользователя и внутренних задачах.

Следующий запрос получает информацию о текущем подключении:

```sql
SELECT
    c.session_id, c.net_transport, c.encrypt_option,
    c.auth_scheme, s.host_name, s.program_name,
    s.client_interface_name, s.login_name, s.nt_domain,
    s.nt_user_name, s.original_login_name, c.connect_time,
    s.login_time
FROM sys.dm_exec_connections AS c
JOIN sys.dm_exec_sessions AS s
    ON c.session_id = s.session_id
WHERE c.session_id = @@SPID;
```

> [!NOTE]
> При выполнении представлений **sys.dm_exec_requests** и **sys.dm_exec_sessions views** с разрешением **VIEW DATABASE STATE** для базы данных вы увидите все выполняющиеся сеансы в базе данных. В противном случае вы увидите только текущий сеанс.

## <a name="monitor-resource-use"></a>Отслеживание использования ресурсов

Вы можете отслеживать использование ресурсов базы данных SQL Azure с помощью [Анализ производительности запросов базы данных SQL](query-performance-insight-use.md). Для базы данных SQL Azure и Управляемый экземпляр Azure SQL можно выполнять мониторинг с помощью [хранилища запросов](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store).

Вы также можете отслеживать использование с помощью следующих представлений:

- База данных SQL Azure: [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database)
- Управляемый экземпляр Azure SQL: [sys.server_resource_stats](/sql/relational-databases/system-catalog-views/sys-server-resource-stats-azure-sql-database)
- База данных SQL Azure и Управляемый экземпляр Azure SQL: [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database)

### <a name="sysdm_db_resource_stats"></a>sys.dm_db_resource_stats

Можно использовать представление [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database) в каждой базе данных. В представлении **sys.dm_db_resource_stats** отображаются последние данные использования ресурсов в соответствии с уровнем служб. Средний процент нагрузки ЦП, показатели операций ввода-вывода данных, записи в журнал и данные о памяти записываются каждые 15 секунд и хранятся 1 час.

Так как это представление содержит более подробные сведения об использовании ресурсов, сначала используйте представление **sys.dm_db_resource_stats** для любого анализа текущего состояния и устранения неполадок. Например, этот запрос показывает среднее и максимальное использование ресурсов для текущей базы данных за последний час:

```sql
SELECT  
    AVG(avg_cpu_percent) AS 'Average CPU use in percent',
    MAX(avg_cpu_percent) AS 'Maximum CPU use in percent',
    AVG(avg_data_io_percent) AS 'Average data IO in percent',
    MAX(avg_data_io_percent) AS 'Maximum data IO in percent',
    AVG(avg_log_write_percent) AS 'Average log write use in percent',
    MAX(avg_log_write_percent) AS 'Maximum log write use in percent',
    AVG(avg_memory_usage_percent) AS 'Average memory use in percent',
    MAX(avg_memory_usage_percent) AS 'Maximum memory use in percent'
FROM sys.dm_db_resource_stats;  
```

Примеры других запросов см. в [sys.dm_db_resource_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-resource-stats-azure-sql-database).

### <a name="sysserver_resource_stats"></a>sys.server_resource_stats

Вы можете использовать [sys.server_resource_stats](/sql/relational-databases/system-catalog-views/sys-server-resource-stats-azure-sql-database) для возврата сведений об использовании ЦП, операций ввода-вывода и хранения данных для управляемый экземпляр SQL Azure. Эти данные собираются и объединяются с пятиминутными интервалами. Для отчетов каждые 15 секунд имеется одна строка. Возвращаемые данные включают загрузку ЦП, размер хранилища, использование операций ввода-вывода и SKU управляемого экземпляра. Данные предыстории хранятся приблизительно в течение 14 суток.

```sql
DECLARE @s datetime;  
DECLARE @e datetime;  
SET @s= DateAdd(d,-7,GetUTCDate());  
SET @e= GETUTCDATE();  
SELECT resource_name, AVG(avg_cpu_percent) AS Average_Compute_Utilization
FROM sys.server_resource_stats
WHERE start_time BETWEEN @s AND @e  
GROUP BY resource_name  
HAVING AVG(avg_cpu_percent) >= 80;
```

### <a name="sysresource_stats"></a>sys.resource_stats

Представление [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) в базе данных **master** содержит дополнительные сведения, которые могут помочь в мониторинге производительности базы данных на уровне служб и вычислений. Данные собираются каждые 5 минут и хранятся приблизительно 14 дней. Это представление полезно для долгосрочного анализа использования ресурсов в базе данных.

На следующей диаграмме показано почасовое использование ресурсов процессора для базы данных уровня служб "Премиум" с объемом вычислительных ресурсов P2 в течение недели. Этот граф начинается в понедельник, показывает пять рабочих дней, а затем показывает выходные дни, когда в приложении происходит гораздо меньше.

![Использование ресурсов базы данных](./media/monitoring-with-dmvs/sql_db_resource_utilization.png)

Судя по этим данным, пиковая нагрузка на процессор составляет чуть более 50 % от максимальной нагрузки для объема вычислительных ресурсов P2 (полдень вторника). Если ЦП является главным фактором в профиле ресурса приложения, можно решить, что P2 — это правильный вычислительный размер, гарантирующий, что Рабочая нагрузка всегда будет соответствовать. Если ожидается рост нагрузки на приложение с течением времени, рекомендуется увеличить запас ресурсов, чтобы приложения не достигло предела производительности. Увеличив объем вычислительных ресурсов, можно избежать заметных пользователю ошибок, которые могут возникнуть из-за нехватки в базе данных мощности для эффективной обработки запросов, особенно в средах, чувствительных к задержкам. Это может быть база данных, поддерживающая приложение, создающее веб-страницы на основе запросов к базе данных.

Для других приложений эту диаграмму можно интерпретировать иначе. Например, если приложение обрабатывало данные по зарплате каждый день и получало ту же диаграмму, то выполнение таких "пакетных заданий" будет эффективным и для объема вычислительных ресурсов P1. Объем вычислительных ресурсов Р1 предоставляет 100 единиц DTU, а P2 — 200 единиц DTU. Таким образом, объем вычислительных ресурсов P1 предоставляет половину объема вычислительных ресурсов P2. Таким образом, 50 процентов использования ЦП на уровне P2 равняется 100 процентам использования ЦП на уровне P1. Если в работе приложения не возникает пауз, возможно, не имеет значения, сколько времени выполняется задание — 2 или 2,5 часа, а важно только, чтобы оно было завершено сегодня. Приложение такой категории может использовать объем вычислительных ресурсов Р1. Вы можете воспользоваться тем, что в определенное время дня использование ресурсов ниже, и перенести пиковую нагрузку именно на этот период. Объем вычислительных ресурсов Р1 может отлично подойти для такого приложения (и сэкономить деньги), если задания будут завершаться вовремя в течение одного дня.

Ядро СУБД предоставляет сведения о потребленных ресурсах для каждой активной базы данных в **sys.resource_stats** представлении базы данных **master** на каждом сервере. Данные в таблице агрегируются каждые 5 минут. На уровнях служб "Базовый", "Стандартный" и "Премиум" может потребоваться более 5 минут, прежде чем данные появятся в таблице, то есть эти данные лучше подходят для ретроспективного анализа, чем для анализа в режиме реального времени. Выполните запрос представления **sys.resource_stats**, чтобы просмотреть журнал базы данных и проверить, обеспечил ли выбранный уровень резервирования требуемую производительность.

> [!NOTE]
> В базе данных SQL Azure необходимо подключиться к базе данных **master** , чтобы запросить **sys.resource_stats** в следующих примерах.

В этом примере показано, как отображаются данные в этом представлении.

```sql
SELECT TOP 10 *
FROM sys.resource_stats
WHERE database_name = 'resource1'
ORDER BY start_time DESC;
```

![Представление каталога sys.resource_stats](./media/monitoring-with-dmvs/sys_resource_stats.png)

В следующем примере показаны различные способы использования представления каталога **sys.resource_stats** для получения сведений о том, как база данных использует ресурсы.

1. Чтобы просмотреть использование ресурсов за прошлую неделю для базы данных userdb1, можно выполнить следующий запрос:

    ```sql
    SELECT *
    FROM sys.resource_stats
    WHERE database_name = 'userdb1' AND
        start_time > DATEADD(day, -7, GETDATE())
    ORDER BY start_time DESC;
    ```

2. Чтобы определить, какой объем вычислительных ресурсов лучше всего подходит для конкретной рабочей нагрузки, нужно детализировать все значения использования ресурсов: ОЗУ, операции чтения и записи, количество рабочих ролей и количество сеансов. Ниже приведен измененный запрос, использующий **sys.resource_stats** для получения отчета о средних и максимальных значениях использования ресурсов.

    ```sql
    SELECT
        avg(avg_cpu_percent) AS 'Average CPU use in percent',
        max(avg_cpu_percent) AS 'Maximum CPU use in percent',
        avg(avg_data_io_percent) AS 'Average physical data IO use in percent',
        max(avg_data_io_percent) AS 'Maximum physical data IO use in percent',
        avg(avg_log_write_percent) AS 'Average log write use in percent',
        max(avg_log_write_percent) AS 'Maximum log write use in percent',
        avg(max_session_percent) AS 'Average % of sessions',
        max(max_session_percent) AS 'Maximum % of sessions',
        avg(max_worker_percent) AS 'Average % of workers',
        max(max_worker_percent) AS 'Maximum % of workers'
    FROM sys.resource_stats
    WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
    ```

3. С помощью средних и максимальных значений по каждому ресурсу можно оценить, насколько выбранный объем вычислительных ресурсов подходит для вашей рабочей нагрузки. Как правило, средние значения из **sys.resource_stats** дают хорошую основу для определения целевого резервирования. Эти сведения следует использовать в качестве отправной точки анализа. Например, вы можете использовать уровень служб "Стандартный" с объемом вычислительных ресурсов S2. При этом средняя нагрузка на процессор и число операций чтения и записи ввода-вывода составляют меньше 40 %, среднее число рабочих ролей — меньше 50, а среднее количество сеансов — меньше 200. Для такой рабочей нагрузки может подойти объем вычислительных ресурсов S1. Вы легко можете определить, отвечает ли уровень базы данных ограничениям рабочих ролей и сеансов. Чтобы определить, соответствует ли база данных более низкому размеру вычислений относительно ЦП, операций чтения и записи, разделите число DTU меньшего размера на единицу DTU текущего размера вычислений, а затем умножьте результат на 100:

    `S1 DTU / S2 DTU * 100 = 20 / 50 * 100 = 40`

    Результатом будет относительная разница производительности между двумя объемами вычислительных ресурсов в процентах. Если использование ресурсов не превышает это значение, для рабочей нагрузки может подойти более низкий объем вычислительных ресурсов. Однако необходимо рассмотреть все диапазоны значений использования ресурсов, а также определить (в процентном отношении), как часто рабочая нагрузка базы банных будет вписываться в рамки более низкого объема вычислительных ресурсов. Следующий запрос отображает процентный показатель измерения ресурсов, исходя из 40-процентного порога, вычисленного в предыдущем примере.

   ```sql
    SELECT
        (COUNT(database_name) - SUM(CASE WHEN avg_cpu_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'CPU Fit Percent',
        (COUNT(database_name) - SUM(CASE WHEN avg_log_write_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Log Write Fit Percent',
        (COUNT(database_name) - SUM(CASE WHEN avg_data_io_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Physical Data IO Fit Percent'
    FROM sys.resource_stats
    WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
    ```

    В зависимости от уровня служб базы данных вы можете определить, подходит ли более низкий объем вычислительных ресурсов для вашей рабочей нагрузки. Если целевой показатель рабочей нагрузки составляет 99,9 % и указанный выше запрос возвращает значение больше 99,9 % для всех трех измерений ресурсов, весьма вероятно, что рабочую нагрузку можно выполнять с более низким объемов вычислительных ресурсов.

    Процентное значение также поможет вам понять, следует ли перейти на следующий объем вычислительных ресурсов для выполнения требований. Например, для userdb1 мы видим следующую нагрузку ЦП за прошлую неделю.

   | Средняя нагрузка ЦП, % | Максимальная нагрузка ЦП, % |
   | --- | --- |
   | 24,5 |100,00 |

    Средняя нагрузка ЦП равна приблизительно одной четвертой ограничения объема вычислительных ресурсов, что вполне соответствует объему вычислительных ресурсов базы данных. Однако максимальное значение показывает, что база данных достигла предела объема вычислительных ресурсов. Требуется ли перейти на более высокий объем вычислительных ресурсов? Определите, сколько раз рабочая нагрузка достигает 100 %, и сравните это значение с целевым показателем рабочей нагрузки базы данных.

    ```sql
    SELECT
        (COUNT(database_name) - SUM(CASE WHEN avg_cpu_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'CPU fit percent'
        ,(COUNT(database_name) - SUM(CASE WHEN avg_log_write_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Log write fit percent'
        ,(COUNT(database_name) - SUM(CASE WHEN avg_data_io_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Physical data IO fit percent'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
    ```

    Если этот запрос возвращает значение менее 99,9% для любого из трех измерений ресурсов, рассмотрите возможность перехода к следующему более высокому размеру вычислений или использование методик настройки приложений для снижения нагрузки на базу данных.

4. В таких расчетах также следует учитывать возможное увеличение рабочей нагрузки в будущем.

Для эластичных пулов можно отслеживать отдельные базы данных в пуле с помощью способов, описанных в этом разделе. Но также можно отслеживать и пул в целом. Дополнительные сведения см. в статье [Мониторинг пула эластичных баз данных и управление им на портале Azure](elastic-pool-overview.md).

### <a name="maximum-concurrent-requests"></a>Максимальное количество параллельных запросов

Чтобы просмотреть количество одновременных запросов, выполните этот запрос Transact-SQL в базе данных:

```sql
SELECT COUNT(*) AS [Concurrent_Requests]
FROM sys.dm_exec_requests R;
```

Чтобы проанализировать рабочую нагрузку SQL Server базы данных, измените этот запрос для фильтрации по конкретной базе данных, которую необходимо проанализировать. Например, если у вас есть локальная база данных с именем MyDatabase, для получения числа параллельных запросов в этой базе данных можно использовать следующий запрос Transact-SQL:

```sql
SELECT COUNT(*) AS [Concurrent_Requests]
FROM sys.dm_exec_requests R
INNER JOIN sys.databases D ON D.database_id = R.database_id
AND D.name = 'MyDatabase';
```

Это только моментальный снимок в один момент времени. Для лучшего понимания рабочей нагрузки и требований к параллельным запросам потребуется собрать большое количество примеров с течением времени.

### <a name="maximum-concurrent-logins"></a>Максимальное число параллельных операций входа

Чтобы получить представление о частоте входа, можно проанализировать шаблоны работы пользователей и приложений. Кроме того, можно запустить реальные нагрузки в тестовой среде, чтобы убедиться в том, что вы не приближаетесь к этим или другим ограничениям, описанным в этой статье. Не существует отдельного запроса или динамического административного представления, которое может показывать одновременно текущие количество входов или историю.

Если несколько клиентов используют ту же строку подключения, служба проверяет подлинность каждого входа. Если 10 пользователей одновременно подключаются к базе данных с использованием того же имени пользователя и пароля, это будет 10 параллельных операций входа. Это ограничение применяется только на время входа и проверки подлинности. Если те же 10 пользователей последовательно подключатся к базе данных, количество параллельных операций входа никогда не будет больше 1.

> [!NOTE]
> Сейчас это ограничение не применимо к базам данных в пулах эластичных баз данных.

### <a name="maximum-sessions"></a>Максимальное число сеансов

Чтобы просмотреть число текущих активных сеансов, выполните этот запрос Transact-SQL в базе данных:

```sql
SELECT COUNT(*) AS [Sessions]
FROM sys.dm_exec_connections;
```

При анализе рабочей нагрузки SQL Server измените запрос, чтобы сосредоточиться на определенной базе данных. Этот запрос позволяет определить возможные потребности в сеансе для базы данных, если вы собираетесь переместить ее в Azure.

```sql
SELECT COUNT(*) AS [Sessions]
FROM sys.dm_exec_connections C
INNER JOIN sys.dm_exec_sessions S ON (S.session_id = C.session_id)
INNER JOIN sys.databases D ON (D.database_id = S.database_id)
WHERE D.name = 'MyDatabase';
```

Опять же, эти запросы возвращают значение счетчика на определенный момент времени. При сборе нескольких выборок с течением времени вы получите лучшее представление об использовании сеанса.

Историю статистики по сеансам можно получить, запросив представление [sys.resource_stats](/sql/relational-databases/system-catalog-views/sys-resource-stats-azure-sql-database) и просмотрев столбец **active_session_count** .

## <a name="monitoring-query-performance"></a>Мониторинг производительности запросов

Медленные или длительные запросы могут потреблять значительные системные ресурсы. В этом разделе показано, как использовать динамические представления управления для выявления нескольких распространенных проблем производительности запросов.

### <a name="finding-top-n-queries"></a>Поиск верхних N запросов

В следующем примере возвращаются сведения о пяти первых запросах, отсортированных по среднему времени ЦП. В этом примере выполняется сбор запросов по хэшу запроса, то есть логически схожие запросы группируются по общему потреблению ресурсов.

```sql
SELECT TOP 5 query_stats.query_hash AS "Query Hash",
    SUM(query_stats.total_worker_time) / SUM(query_stats.execution_count) AS "Avg CPU Time",
     MIN(query_stats.statement_text) AS "Statement Text"
FROM
    (SELECT QS.*,
        SUBSTRING(ST.text, (QS.statement_start_offset/2) + 1,
            ((CASE statement_end_offset
                WHEN -1 THEN DATALENGTH(ST.text)
                ELSE QS.statement_end_offset END
            - QS.statement_start_offset)/2) + 1) AS statement_text
FROM sys.dm_exec_query_stats AS QS
CROSS APPLY sys.dm_exec_sql_text(QS.sql_handle) as ST) as query_stats
GROUP BY query_stats.query_hash
ORDER BY 2 DESC;
```

### <a name="monitoring-blocked-queries"></a>Мониторинг заблокированных запросов

Медленные или длительные запросы могут вызывать избыточное потребление ресурсов, что приводит к блокировке запросов. Причиной блокировки может быть неэффективная структура приложений, неудачные планы запросов, отсутствие полезных индексов и т. д. Представление sys.dm_tran_locks можно использовать для получения сведений о текущем действии блокировки в базе данных. Пример кода см. в разделе [sys.dm_tran_locks (Transact-SQL)](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql). Дополнительные сведения об устранении неполадок с блокировкой см. в статье [изучение и устранение проблем с БЛОКИРОВКОЙ SQL Azure](understand-resolve-blocking.md).

### <a name="monitoring-query-plans"></a>Мониторинг планов запросов

Неэффективный план запросов может повысить потребление ресурсов ЦП. В следующем примере представление [sys.dm_exec_query_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-query-stats-transact-sql) используется, чтобы определить, какой запрос использует наибольшее количество ресурсов ЦП.

```sql
SELECT
    highest_cpu_queries.plan_handle,
    highest_cpu_queries.total_worker_time,
    q.dbid,
    q.objectid,
    q.number,
    q.encrypted,
    q.[text]
FROM
    (SELECT TOP 50
        qs.plan_handle,
        qs.total_worker_time
    FROM
        sys.dm_exec_query_stats qs
ORDER BY qs.total_worker_time desc) AS highest_cpu_queries
CROSS APPLY sys.dm_exec_sql_text(plan_handle) AS q
ORDER BY highest_cpu_queries.total_worker_time DESC;
```

## <a name="see-also"></a>См. также раздел

[Введение в базу данных SQL Azure и Управляемый экземпляр Azure SQL](sql-database-paas-overview.md)