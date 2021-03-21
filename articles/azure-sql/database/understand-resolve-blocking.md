---
title: Изучение и устранение проблем с блокировкой SQL Azure
titleSuffix: Azure SQL Database
description: Общие сведения о блокировке и устранении неполадок в конкретных базах данных SQL Azure.
services: sql-database
dev_langs:
- TSQL
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: ''
ms.date: 3/02/2021
ms.openlocfilehash: 3d64336184450514d52095097343a4588213f111
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102034903"
---
# <a name="understand-and-resolve-azure-sql-database-blocking-problems"></a>Изучение и устранение проблем с блокировкой базы данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

## <a name="objective"></a>Назначение

В статье описывается блокировка в базах данных SQL Azure и демонстрируется, как устранять неполадки и разрешать блокировку. 

В этой статье термин «соединение» относится к сеансу с одним входом в систему базы данных. Каждое соединение отображается как идентификатор сеанса (SPID) или session_id во многих динамических административных представлениях. Каждый из этих SPID часто называется процессом, хотя он не является отдельным контекстом процесса в обычном смысле. Вместо этого каждый SPID состоит из ресурсов сервера и структур данных, необходимых для обслуживания запросов одного соединения от данного клиента. У одного клиентского приложения может быть одно или несколько подключений. С точки зрения базы данных SQL Azure не существует различий между несколькими подключениями из одного клиентского приложения на одном клиентском компьютере и несколькими клиентскими приложениями или несколькими клиентскими компьютерами. они являются атомарными. Одно подключение может блокировать другое соединение независимо от исходного клиента.

> [!NOTE]
> **Это содержимое сосредоточено на базе данных SQL Azure.** База данных SQL Azure основана на новейшей стабильной версии ядра СУБД Microsoft SQL Server, поэтому большая часть содержимого схожа с тем, что параметры и средства устранения неполадок могут отличаться. Дополнительные сведения о блокировке в SQL Server см. в разделе [изучение и устранение проблем с блокировкой SQL Server](/troubleshoot/sql/performance/understand-resolve-blocking).

## <a name="understand-blocking"></a>Общие сведения о блокировке 
 
Блокировка — это неиспользуемая и некоторая характеристика любой системы управления реляционными базами данных (РСУБД) с параллелизмом на основе блокировок. Как упоминалось ранее, в SQL Server блокировка происходит, когда один сеанс удерживает блокировку определенного ресурса, а второй SPID пытается получить конфликтующий тип блокировки в том же ресурсе. Как правило, интервал времени, для которого первый SPID блокирует ресурс, является небольшим. Когда сеанс-владелец освобождает блокировку, второе подключение освобождается, чтобы получить собственную блокировку ресурса и продолжить обработку. Это нормальное поведение, которое может происходить несколько раз в течение дня без заметного воздействия на производительность системы.

Длительность и контекст транзакции запроса определяют время удержания блокировок и, таким образом, их воздействие на другие запросы. Если запрос не выполняется в рамках транзакции (и указания блокировки не используются), блокировки для инструкций SELECT будут храниться только на ресурсе во время чтения, а не во время выполнения запроса. Для инструкций INSERT, UPDATE и DELETE блокировки удерживаются во время запроса, как для согласованности данных, так и для разрешения отката запроса при необходимости.

Для запросов, выполняемых в рамках транзакции, продолжительность удержания блокировок определяется типом запроса, уровнем изоляции транзакции и тем, используются ли в запросе подсказки блокировки. Описание блокировок, подсказок блокировки и уровней изоляции транзакций см. в следующих статьях:

* [Блокировка в ядре СУБД](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide)
* [Настройка блокировки и управления версиями строк](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#customizing-locking-and-row-versioning)
* [Режимы блокировки](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#lock_modes)
* [Совместимость блокировок](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide#lock_compatibility)
* [Транзакции](/sql/t-sql/language-elements/transactions-transact-sql)

Если блокировка и блокировка сохраняются в момент возникновения негативного воздействия на производительность системы, это вызвано одной из следующих причин:

* Идентификатор SPID удерживает блокировки на наборе ресурсов в течение продолжительного периода времени, прежде чем их выпустить. Этот тип блокировки разрешается с течением времени, но может привести к снижению производительности.

* Идентификатор SPID содержит блокировки на наборе ресурсов и никогда не освобождает их. Этот тип блокировки не позволяет устранить неполадки и запрещает доступ к затронутым ресурсам в течение неограниченного времени.

В первом случае ситуация может быть очень гибкой, так как различные идентификаторы SPID приводят к блокировке различных ресурсов с течением времени, создавая цель для перемещения. В таких ситуациях трудно устранять неполадки с помощью [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) , чтобы устранить проблему в отдельных запросах. В отличие от этого, вторая ситуация приводит к последовательному состоянию, который легче диагностировать.

## <a name="applications-and-blocking"></a>Приложения и блокировка

Может возникнуть тенденция сосредоточиться на проблемах настройки и платформы на стороне сервера при возникновении проблем с блокировкой. Тем не менее, если внимание было оплачено только в базе данных, это может не привести к решению проблемы, а время и энергопотребление лучше повлиять на изучение клиентского приложения и отправленных им запросов. Независимо от уровня видимости, предоставляемого приложением в отношении сделанных вызовов базы данных, проблемная блокировка часто требует как проверки точных инструкций SQL, переданных приложением, так и точного поведения приложения в отношении отмены запросов, управления подключениями, получения всех результирующих строк и т. д. Если средство разработки не разрешает явное управление подключениями, отмену запросов, время ожидания запроса, выборка результатов и т. д., проблемы с блокировкой могут быть не разрешены. Этот потенциал следует тщательно изучить перед выбором средства разработки приложений для базы данных SQL Azure, особенно для сред OLTP, учитывающих производительность. 

Обратите внимание на производительность базы данных на этапе проектирования и построения базы данных и приложения. В частности, для каждого запроса следует оценивать потребление ресурсов, уровень изоляции и длину пути транзакции. Каждый запрос и транзакция должны быть максимально простыми. Хорошая дисциплина управления подключениями должна быть продолжена, без нее, приложение может иметь приемлемую производительность при нехватке числа пользователей, но производительность может значительно снизиться, так как число пользователей масштабируется вверх. 

Благодаря правильному проектированию приложений и запросов База данных SQL Azure может поддерживать множество тысяч одновременных пользователей на одном сервере с небольшой блокировкой.

> [!Note]
> Дополнительные рекомендации по разработке приложений см. [в статьях устранение неполадок подключения и другие ошибки в базе данных SQL Azure и управляемый экземпляр Azure SQL](troubleshoot-common-errors-issues.md) и [обработка временных сбоев](/aspnet/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/transient-fault-handling).

## <a name="troubleshoot-blocking"></a>Устранение неполадок блокировки

Независимо от ситуации блокировки, в которой мы работаем, методология устранения неполадок в ней одинакова. Эти логические цветоделения зависят от оставшейся части композиции этой статьи. Эта концепция предназначена для поиска и определения того, что делает запрос и почему он блокируется. После определения проблемного запроса (т. е. блокировки на длительный период) следующим шагом является анализ и определение причины возникновения блокировки. После того, как мы понимаем, почему, мы можем внести изменения, переопределив запрос и транзакцию.

Инструкции по устранению неполадок:

1. Определите основной сеанс блокировки (головной блок)

2. Поиск запроса и транзакции, вызывающей блокировку (что удерживает блокировки в течение длительного периода)

3. Анализ и понимание того, почему происходит длительная блокировка

4. Устранение проблемы с блокировкой путем перепроектирования запроса и транзакции

Теперь давайте обсудим, как точно определить основной сеанс блокировки с подходящей записью данных.

## <a name="gather-blocking-information"></a>Сбор сведений о блокировании

Чтобы отрушить сложность устранения проблем с блокировкой, администратор базы данных может использовать скрипты SQL, которые постоянно отслеживают состояние блокировки и блокировки в базе данных SQL Azure. Для сбора этих данных по сути, существует два метода. 

Первый — запрос динамических управляющих объектов (дмос) и сохранение результатов для сравнения с течением времени. Некоторые объекты, упоминаемые в этой статье, являются динамическими административными представлениями (DMV), а некоторые — динамическими функциями управления (DMF). Второй способ — использовать XEvents для записи выполняемого действия. 


## <a name="gather-information-from-dmvs"></a>Сбор данных из динамических административных представлений

Ссылки на динамические административные представления для устранения неполадок блокируются с целью определения SPID (идентификатора сеанса) в заголовке цепочки блокировок и инструкции SQL. Найдите идентификаторы SPID жертвы, которые блокируются. Если какой-либо SPID блокируется другим идентификатором SPID, Исследуйте идентификатор SPID, который владеет ресурсом (блокировка SPID). Блокируется ли также владелец SPID? Вы можете пройти по цепочке, чтобы найти головной блок, а затем выяснить, почему он обслуживает блокировку.

Не забудьте выполнить каждый из этих сценариев в целевой базе данных SQL Azure.

* Команды sp_who и sp_who2 являются более старыми командами для отображения всех текущих сеансов. Sys.dm_exec_sessions динамического административного представления возвращает больше данных в результирующем наборе, который проще запрашивать и фильтровать. Вы обнаружите sys.dm_exec_sessions в ядре других запросов. 

* Если у вас уже есть определенный сеанс, можно использовать `DBCC INPUTBUFFER(<session_id>)` для поиска последней инструкции, отправленной сеансом. Аналогичные результаты могут возвращаться с помощью функции динамического управления sys.dm_exec_input_buffer (DMF) в результирующем наборе, который проще запрашивать и фильтровать, предоставляя session_id и request_id. Например, чтобы получить последний запрос, отправленный session_id 66 и request_id 0:

```sql
SELECT * FROM sys.dm_exec_input_buffer (66,0);
```

* См. sys.dm_exec_requests и ссылки на столбец blocking_session_id. Если blocking_session_id = 0, сеанс не блокируется. Хотя sys.dm_exec_requests содержит только запросы, выполняемые в данный момент, все подключения (активные или не) будут перечислены в sys.dm_exec_sessions. Постройте это общее соединение между sys.dm_exec_requests и sys.dm_exec_sessions в следующем запросе.

* Выполните этот пример запроса, чтобы найти активные выполняемые запросы и текущий текст пакета SQL или текст входного буфера с помощью [sys.dm_exec_sql_text](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sql-text-transact-sql) или [sys.dm_exec_input_buffer](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-input-buffer-transact-sql) DMV. Если данные, возвращаемые `text` полем sys.dm_exec_sql_text, имеют значение null, запрос в данный момент не выполняется. В этом случае `event_info` поле sys.dm_exec_input_buffer будет содержать последнюю командную строку, переданную в ядро SQL. Этот запрос также можно использовать для обнаружения сеансов, блокирующих другие сеансы, включая список session_ids заблокированных на session_id. 

```sql
WITH cteBL (session_id, blocking_these) AS 
(SELECT s.session_id, blocking_these = x.blocking_these FROM sys.dm_exec_sessions s 
CROSS APPLY    (SELECT isnull(convert(varchar(6), er.session_id),'') + ', '  
                FROM sys.dm_exec_requests as er
                WHERE er.blocking_session_id = isnull(s.session_id ,0)
                AND er.blocking_session_id <> 0
                FOR XML PATH('') ) AS x (blocking_these)
)
SELECT s.session_id, blocked_by = r.blocking_session_id, bl.blocking_these
, batch_text = t.text, input_buffer = ib.event_info, * 
FROM sys.dm_exec_sessions s 
LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
INNER JOIN cteBL as bl on s.session_id = bl.session_id
OUTER APPLY sys.dm_exec_sql_text (r.sql_handle) t
OUTER APPLY sys.dm_exec_input_buffer(s.session_id, NULL) AS ib
WHERE blocking_these is not null or r.blocking_session_id > 0
ORDER BY len(bl.blocking_these) desc, r.blocking_session_id desc, r.session_id;
```

* Запустите этот более сложный пример запроса, предоставляемый служба поддержки Майкрософт, чтобы указать заголовок цепочки многосеансовой блокировки, включая текст запроса сеансов, участвующих в цепочке блокировок.

```sql
WITH cteHead ( session_id,request_id,wait_type,wait_resource,last_wait_type,is_user_process,request_cpu_time
,request_logical_reads,request_reads,request_writes,wait_time,blocking_session_id,memory_usage
,session_cpu_time,session_reads,session_writes,session_logical_reads
,percent_complete,est_completion_time,request_start_time,request_status,command
,plan_handle,sql_handle,statement_start_offset,statement_end_offset,most_recent_sql_handle
,session_status,group_id,query_hash,query_plan_hash) 
AS ( SELECT sess.session_id, req.request_id, LEFT (ISNULL (req.wait_type, ''), 50) AS 'wait_type'
    , LEFT (ISNULL (req.wait_resource, ''), 40) AS 'wait_resource', LEFT (req.last_wait_type, 50) AS 'last_wait_type'
    , sess.is_user_process, req.cpu_time AS 'request_cpu_time', req.logical_reads AS 'request_logical_reads'
    , req.reads AS 'request_reads', req.writes AS 'request_writes', req.wait_time, req.blocking_session_id,sess.memory_usage
    , sess.cpu_time AS 'session_cpu_time', sess.reads AS 'session_reads', sess.writes AS 'session_writes', sess.logical_reads AS 'session_logical_reads'
    , CONVERT (decimal(5,2), req.percent_complete) AS 'percent_complete', req.estimated_completion_time AS 'est_completion_time'
    , req.start_time AS 'request_start_time', LEFT (req.status, 15) AS 'request_status', req.command
    , req.plan_handle, req.[sql_handle], req.statement_start_offset, req.statement_end_offset, conn.most_recent_sql_handle
    , LEFT (sess.status, 15) AS 'session_status', sess.group_id, req.query_hash, req.query_plan_hash
    FROM sys.dm_exec_sessions AS sess
    LEFT OUTER JOIN sys.dm_exec_requests AS req ON sess.session_id = req.session_id
    LEFT OUTER JOIN sys.dm_exec_connections AS conn on conn.session_id = sess.session_id 
    )
, cteBlockingHierarchy (head_blocker_session_id, session_id, blocking_session_id, wait_type, wait_duration_ms,
wait_resource, statement_start_offset, statement_end_offset, plan_handle, sql_handle, most_recent_sql_handle, [Level])
AS ( SELECT head.session_id AS head_blocker_session_id, head.session_id AS session_id, head.blocking_session_id
    , head.wait_type, head.wait_time, head.wait_resource, head.statement_start_offset, head.statement_end_offset
    , head.plan_handle, head.sql_handle, head.most_recent_sql_handle, 0 AS [Level]
    FROM cteHead AS head
    WHERE (head.blocking_session_id IS NULL OR head.blocking_session_id = 0)
    AND head.session_id IN (SELECT DISTINCT blocking_session_id FROM cteHead WHERE blocking_session_id != 0)
    UNION ALL
    SELECT h.head_blocker_session_id, blocked.session_id, blocked.blocking_session_id, blocked.wait_type,
    blocked.wait_time, blocked.wait_resource, h.statement_start_offset, h.statement_end_offset,
    h.plan_handle, h.sql_handle, h.most_recent_sql_handle, [Level] + 1
    FROM cteHead AS blocked
    INNER JOIN cteBlockingHierarchy AS h ON h.session_id = blocked.blocking_session_id and h.session_id!=blocked.session_id --avoid infinite recursion for latch type of blocking
    WHERE h.wait_type COLLATE Latin1_General_BIN NOT IN ('EXCHANGE', 'CXPACKET') or h.wait_type is null
    )
SELECT bh.*, txt.text AS blocker_query_or_most_recent_query 
FROM cteBlockingHierarchy AS bh 
OUTER APPLY sys.dm_exec_sql_text (ISNULL ([sql_handle], most_recent_sql_handle)) AS txt;
```

* Чтобы перехватить длительные или незафиксированные транзакции, используйте другой набор динамических административных представлений для просмотра текущих открытых транзакций, включая [sys.dm_tran_database_transactions](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-database-transactions-transact-sql), [sys.dm_tran_session_transactions](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-session-transactions-transact-sql), [sys.dm_exec_connections](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-connections-transact-sql)и sys.dm_exec_sql_text. Существует несколько динамических административных представлений, связанных с транзакциями отслеживания. Дополнительные сведения см. в статье дополнительные [представления DMV для транзакций](/sql/relational-databases/system-dynamic-management-views/transaction-related-dynamic-management-views-and-functions-transact-sql) . 

```sql
SELECT [s_tst].[session_id],
[database_name] = DB_NAME (s_tdt.database_id),
[s_tdt].[database_transaction_begin_time], 
[sql_text] = [s_est].[text] 
FROM sys.dm_tran_database_transactions [s_tdt]
INNER JOIN sys.dm_tran_session_transactions [s_tst] ON [s_tst].[transaction_id] = [s_tdt].[transaction_id]
INNER JOIN sys.dm_exec_connections [s_ec] ON [s_ec].[session_id] = [s_tst].[session_id]
CROSS APPLY sys.dm_exec_sql_text ([s_ec].[most_recent_sql_handle]) AS [s_est];
```

* Ссылка [sys.dm_os_waiting_tasks](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-waiting-tasks-transact-sql) , которая находится на уровне потока или задачи SQL. Он возвращает сведения о типе ожидания SQL, в котором в данный момент находится запрос. Как и sys.dm_exec_requests, sys.dm_os_waiting_tasks возвращаются только активные запросы. 

> [!Note]
> Дополнительные сведения о типах ожидания, включая агрегированную статистику ожидания с течением времени, см. в [sys.dm_db_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-wait-stats-azure-sql-database)динамического административного представления. Это динамическое административное представление возвращает совокупную статистику ожидания только для текущей базы данных.

* Используйте [sys.dm_tran_locksное](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql) административное представление для более детальной информации о том, какие блокировки были помещены запросами. Это динамическое административное представление может возвращать большие объемы данных в рабочей SQL Server и полезно для диагностики того, какие блокировки в настоящее время удерживаются. 

Из-за внутреннего объединения на sys.dm_os_waiting_tasks следующий запрос ограничит выходные данные sys.dm_tran_locks только для запросов, заблокированных в настоящий момент, их состояния ожидания и блокировки.

```sql
SELECT table_name = schema_name(o.schema_id) + '.' + o.name
, wt.wait_duration_ms, wt.wait_type, wt.blocking_session_id, wt.resource_description
, tm.resource_type, tm.request_status, tm.request_mode, tm.request_session_id
FROM sys.dm_tran_locks AS tm
INNER JOIN sys.dm_os_waiting_tasks as wt ON tm.lock_owner_address = wt.resource_address
LEFT OUTER JOIN sys.partitions AS p on p.hobt_id = tm.resource_associated_entity_id
LEFT OUTER JOIN sys.objects o on o.object_id = p.object_id or tm.resource_associated_entity_id = o.object_id
WHERE resource_database_id = DB_ID()
AND object_name(p.object_id) = '<table_name>';
```

* При использовании динамических административных представлений для хранения результатов запроса с течением времени будут предоставлены точки данных, позволяющие просматривать блокировку в течение заданного интервала времени для выявления сохраненных блокировок или тенденций. 

## <a name="gather-information-from-extended-events"></a>Сбор данных из расширенных событий

В дополнение к предыдущим сведениям часто требуется записать трассировку действий на сервере, чтобы тщательно исследовать проблему блокировки в базе данных SQL Azure. Например, если сеанс выполняет несколько инструкций внутри транзакции, будет представлена только последняя отправленная инструкция. Однако одной из предыдущих инструкций могут быть все еще удерживаемые блокировки. Трассировка позволяет просматривать все команды, выполняемые сеансом в текущей транзакции.

Существует два способа записи трассировок в SQL Server. Расширенные события (XEvents) и трассировки профилировщика. Однако [SQL Server Profiler](/sql/tools/sql-server-profiler/sql-server-profiler) является устаревшей технологией трассировки, не поддерживаемой для базы данных SQL Azure. [Расширенные события](/sql/relational-databases/extended-events/extended-events) — это более новая технология трассировки, которая обеспечивает большую гибкость и снижает влияние на наблюдаемую систему, а ее интерфейс интегрирован в SQL Server Management Studio (SSMS). 

См. документ, в котором объясняется, как использовать [Мастер расширенных событий нового сеанса](/sql/relational-databases/extended-events/quick-start-extended-events-in-sql-server) в SSMS. Однако для баз данных SQL Azure среда SSMS предоставляет вложенную папку расширенных событий для каждой базы данных в обозревателе объектов. Используйте мастер сеансов расширенных событий для сбора этих полезных событий: 

-   Ошибки категории:
    -   Внимание
    -   Error_reported  
    -   Execution_warning

-   Предупреждения категории: 
    -   Missing_join_predicate

-   Выполнение категории:
    -   Rpc_completed
    -   Rpc_starting 
    -   Sql_batch_completed
    -   Sql_batch_starting

-   Блокировка
    -   Lock_deadlock

-   Сеанс
    -   Existing_connection
    -   Имя входа
    -   Logout

## <a name="identify-and-resolve-common-blocking-scenarios"></a>Выявление и устранение распространенных сценариев блокировки

Изучив предыдущие сведения, можно определить причину большинства проблем с блокировкой. Оставшаяся часть этой статьи — обсуждение того, как использовать эту информацию для определения и разрешения некоторых распространенных сценариев блокировки. В этом обсуждении предполагается, что вы использовали блокирующие сценарии (упомянутые ранее) для сбора сведений о блокировках идентификаторов SPID и сбора действий приложения с помощью сеанса XEvent.

## <a name="analyze-blocking-data"></a>Анализ блокирующих данных 

* Изучите выходные данные динамических административных представлений sys.dm_exec_requests и sys.dm_exec_sessions, чтобы определить заголовки цепочек блокировки с помощью blocking_these и session_id. Это наиболее ясно определяет, какие запросы блокируются и какие блокируются. Проанализируйте сеансы, которые блокируются и блокируются. Существует ли общая или корневая цепочка блокировок? Скорее всего, они совместно используют общую таблицу, и один или несколько сеансов, участвующих в цепочке блокировок, выполняют операцию записи. 

* Изучите выходные данные динамических административных представлений sys.dm_exec_requests и sys.dm_exec_sessions, чтобы получить сведения о SPID в заголовке цепочки блокировок. Найдите следующие поля: 

    -    `sys.dm_exec_requests.status`  
    В этом столбце отображается состояние конкретного запроса. Как правило, состояние ожидания означает, что SPID завершил выполнение и ожидает, пока приложение отправит другой запрос или пакет. Состояние готовности или выполнения указывает на то, что идентификатор SPID в настоящее время обрабатывает запрос. В следующей таблице приведены краткие объяснения различных значений состояния.

    | Состояние | Значение |
    |:-|:-|
    | Историческая справка | SPID выполняет фоновую задачу, например обнаружение взаимоблокировки, модуль записи журнала или контрольную точку. |
    | В режиме ожидания | Идентификатор SPID в данный момент не выполняется. Обычно это означает, что SPID ожидает команду из приложения. |
    | Запущен | Идентификатор SPID в настоящее время выполняется в планировщике. |
    | Готово к запуску | Идентификатор SPID находится в очереди готовности планировщика и ожидает получения времени планировщика. |
    | Приостановлена | SPID ожидает ресурса, например блокировки или кратковременной блокировки. | 
                       
    -    `sys.dm_exec_sessions.open_transaction_count`  
    В этом поле указывается количество открытых транзакций в этом сеансе. Если это значение больше 0, то SPID находится в открытой транзакции и может удерживать блокировки, полученные любой инструкцией внутри транзакции.

    -    `sys.dm_exec_requests.open_transaction_count`  
    Аналогичным образом в этом поле указывается количество открытых транзакций в этом запросе. Если это значение больше 0, то SPID находится в открытой транзакции и может удерживать блокировки, полученные любой инструкцией внутри транзакции.

    -   `sys.dm_exec_requests.wait_type`, `wait_time` и `last_wait_type`.  
    Если значение  `sys.dm_exec_requests.wait_type`   равно null, запрос в настоящее время не ожидает какого-либо  `last_wait_type`   значения и указывает на Последнее  `wait_type`   , что был обнаружен запрос. Дополнительные сведения  `sys.dm_os_wait_stats` и описание наиболее распространенных типов ожидания см. в разделе [sys.dm_os_wait_stats](/sql/relational-databases/system-dynamic-management-views/sys-dm-os-wait-stats-transact-sql).  `wait_time`   Значение можно использовать, чтобы определить, выполняется ли запрос. Если запрос к sys.dm_exec_requests таблице возвращает значение в  `wait_time`   столбце, которое меньше  `wait_time`   значения из предыдущего запроса sys.dm_exec_requests, это означает, что предыдущая блокировка была получена и освобождена и теперь ожидает новой блокировки (при условии, что она не равна нулю `wait_time` ). Это можно проверить с помощью сравнения `wait_resource`   между выходными данными sys.dm_exec_requests, в которых отображается ресурс, для которого запрос находится в состоянии ожидания.

    -   `sys.dm_exec_requests.wait_resource` В этом поле указывается ресурс, который ожидает заблокированный запрос. В следующей таблице перечислены стандартные  `wait_resource`   форматы и их значения.

    | Ресурс | Формат | Пример | Пояснение | 
    |:-|:-|:-|:-|
    | Таблица | DatabaseID: ObjectID: Индексид | ВКЛАДКА: 5:261575970:1 | В этом случае база данных с ИДЕНТИФИКАТОРом 5 является образцом базы данных pubs, а объект с ИДЕНТИФИКАТОРом 261575970 — таблицей titles, а 1 — кластеризованным индексом. |
    | Страница | DatabaseID: ИД: идентификатор страницы | СТРАНИЦА: 5:1:104 | В этом случае база данных с ИДЕНТИФИКАТОРом 5 — Pubs, файл с ИДЕНТИФИКАТОРом 1 — первичный файл данных, а страница 104 — страница, принадлежащая таблице titles. Чтобы указать object_id, к которой принадлежит страница, используйте функцию динамического управления [sys.dm_db_page_info](/sql/relational-databases/system-dynamic-management-views/sys-dm-db-page-info-transact-sql), передав DatabaseID, идентификаторы идентификатор страницы из `wait_resource` . | 
    | Ключ | DatabaseID: Hobt_id (хэш-значение для ключа индекса) | КЛЮЧ: 5:72057594044284928 (3300a4f361aa) | В этом случае база данных с ИДЕНТИФИКАТОРом 5 — это Pubs, Hobt_ID 72057594044284928 соответствует index_id 2 для object_id 261575970 (таблица titles). Используйте представление каталога sys. partitions, чтобы связать hobt_id с определенным index_id и object_id. Невозможно выполнить дехэширование хэша ключа индекса до определенного значения ключа. |
    | Строка | DatabaseID: ИД столбца: идентификатор страницы: слот (строка) | RID: 5:1:104:3 | В этом случае база данных с ИДЕНТИФИКАТОРом 5 — Pubs, файл с ИДЕНТИФИКАТОРом 1 — первичный файл данных, страница 104 — страница, принадлежащая таблице titles, а в слоте 3 — расположение строки на странице. |
    | Компилятор  | DatabaseID: ИД столбца: идентификатор страницы: слот (строка) | RID: 5:1:104:3 | В этом случае база данных с ИДЕНТИФИКАТОРом 5 — Pubs, файл с ИДЕНТИФИКАТОРом 1 — первичный файл данных, страница 104 — страница, принадлежащая таблице titles, а в слоте 3 — расположение строки на странице. |

    -    `sys.dm_tran_active_transactions`[Sys.dm_tran_active_transactions](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-active-transactions-transact-sql) динамическое административное представление содержит данные об открытых транзакциях, которые можно объединить с другими динамическими представлениями DMV для создания полной картины транзакций, ожидающих фиксации или отката. Используйте следующий запрос, чтобы получить сведения об открытых транзакциях, Объединенных в другие динамические административные представления, включая [sys.dm_tran_session_transactions](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-session-transactions-transact-sql). Рассмотрим текущее состояние транзакции, `transaction_begin_time` и другие данные о ситуации, чтобы определить, может ли это быть источником блокировки.

    ```sql
    SELECT tst.session_id, [database_name] = db_name(s.database_id)
    , tat.transaction_begin_time
    , transaction_duration_s = datediff(s, tat.transaction_begin_time, sysdatetime()) 
    , transaction_type = CASE tat.transaction_type  WHEN 1 THEN 'Read/write transaction'
                                                    WHEN 2 THEN 'Read-only transaction'
                                                    WHEN 3 THEN 'System transaction'
                                                    WHEN 4 THEN 'Distributed transaction' END
    , input_buffer = ib.event_info, tat.transaction_uow     
    , transaction_state  = CASE tat.transaction_state    
                WHEN 0 THEN 'The transaction has not been completely initialized yet.'
                WHEN 1 THEN 'The transaction has been initialized but has not started.'
                WHEN 2 THEN 'The transaction is active - has not been committed or rolled back.'
                WHEN 3 THEN 'The transaction has ended. This is used for read-only transactions.'
                WHEN 4 THEN 'The commit process has been initiated on the distributed transaction.'
                WHEN 5 THEN 'The transaction is in a prepared state and waiting resolution.'
                WHEN 6 THEN 'The transaction has been committed.'
                WHEN 7 THEN 'The transaction is being rolled back.'
                WHEN 8 THEN 'The transaction has been rolled back.' END 
    , transaction_name = tat.name, request_status = r.status
    , azure_dtc_state = CASE tat.dtc_state 
                        WHEN 1 THEN 'ACTIVE'
                        WHEN 2 THEN 'PREPARED'
                        WHEN 3 THEN 'COMMITTED'
                        WHEN 4 THEN 'ABORTED'
                        WHEN 5 THEN 'RECOVERED' END
    , tst.is_user_transaction, tst.is_local
    , session_open_transaction_count = tst.open_transaction_count  
    , s.host_name, s.program_name, s.client_interface_name, s.login_name, s.is_user_process
    FROM sys.dm_tran_active_transactions tat 
    INNER JOIN sys.dm_tran_session_transactions tst  on tat.transaction_id = tst.transaction_id
    INNER JOIN Sys.dm_exec_sessions s on s.session_id = tst.session_id 
    LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
    CROSS APPLY sys.dm_exec_input_buffer(s.session_id, null) AS ib;
    ```

    -   Другие столбцы

        Остальные столбцы в [sys.dm_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql) и [sys.dm_exec_request](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql) могут также получить информацию о корне проблемы. Их полезность зависит от обстоятельств проблемы. Например, можно определить, возникает ли проблема только с определенными клиентами (hostname) в определенных сетевых библиотеках (net_library), когда последний пакет, отправленный SPID, был `last_request_start_time` в sys.dm_exec_sessions, как долго выполнялся запрос с помощью `start_time` в sys.dm_exec_requests и т. д.


## <a name="common-blocking-scenarios"></a>Распространенные сценарии блокировки

В приведенной ниже таблице сопоставлены общие признаки их возможных причин.  

`wait_type`Столбцы, `open_transaction_count` и `status` ссылаются на сведения, возвращаемые [sys.dm_exec_request](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-requests-transact-sql). другие столбцы могут возвращаться [sys.dm_exec_sessions](/sql/relational-databases/system-dynamic-management-views/sys-dm-exec-sessions-transact-sql). "Разрешает?" Указывает, будет ли блокировка самостоятельно разрешаться, или же сеанс должен быть прерван с помощью `KILL` команды. Дополнительные сведения см. в разделе [Kill (Transact-SQL)](/sql/t-sql/language-elements/kill-transact-sql).

| Сценарий | Столбцы waittype | Open_Tran | Состояние | Разрешает? | Другие признаки |  
|:-|:-|:-|:-|:-|:-|--|
| 1 | NOT NULL | >= 0 | готов к запуску | Да, по завершении запроса. | В sys.dm_exec_sessions столбцы, **операции чтения**, **cpu_time** и (или) **memory_usage** увеличиваются со временем. Длительность запроса будет высока по завершении. |
| 2 | NULL | \>0 | ожидание | Нет, но SPID может быть уничтожен. | В сеансе расширенного события для этого SPID может появиться сигнал внимания, указывающий на время ожидания запроса или отмены. |
| 3 | NULL | \>= 0 | готов к запуску | Нет. Не будет разрешаться, пока клиент не выберет все строки или закроет соединение. SPID может быть уничтожен, но может занять до 30 секунд. | Если open_transaction_count = 0, а SPID удерживает блокировки, в то время как уровень изоляции транзакции по умолчанию равен (READ КОМММИТТЕД), это вероятная причина. |  
| 4 | Различается | \>= 0 | готов к запуску | Нет. Не будет разрешаться, пока клиент не отменит запросы или не закроет соединения. SPID могут быть уничтожены, но может занять до 30 секунд. | Столбец **HostName** в sys.DM_EXEC_SESSIONS для SPID в заголовке цепочки блокировок будет таким же, как и один из SPID, который он блокирует. |  
| 5 | NULL | \>0 | откат; | Да. | В сеансе расширенных событий для этого SPID может появиться сигнал об ошибке, указывающий на время ожидания запроса или отмену, либо выдана простая инструкция ROLLBACK. |  
| 6 | NULL | \>0 | ожидание | Со временем. Когда система Windows NT определит, что сеанс больше не активен, подключение к базе данных SQL Azure будет разорвано. | `last_request_start_time`Значение в sys.dm_exec_sessions намного раньше текущего времени. |

## <a name="detailed-blocking-scenarios"></a>Подробные сценарии блокировки

1.  Блокировка, вызванная обычным выполнением запроса с длительным временем выполнения

    **Решение. решение** для этого типа проблемы блокировки заключается в поиске способов оптимизации запроса. На самом деле, этот класс проблемы с блокировкой может просто быть проблемой с производительностью и потребовать от нее решения. Сведения об устранении неполадок, связанных с заданными задолгоми запросами, см. в статье [Устранение неполадок с выполнением запросов в SQL Server](/troubleshoot/sql/performance/troubleshoot-slow-running-queries). Дополнительные сведения см. в статье [Наблюдение и настройка производительности](/sql/relational-databases/performance/monitor-and-tune-for-performance). 

    Отчеты из [хранилища запросов](/sql/relational-databases/performance/best-practice-with-the-query-store) в среде SSMS также являются очень рекомендуемым и ценным инструментом для определения наиболее ресурсоемких запросов, неоптимальных планов выполнения. Также ознакомьтесь с разделом [Интеллектуальная производительность](intelligent-insights-overview.md) портал Azure для базы данных SQL Azure, в том числе [Анализ производительности запросов](query-performance-insight-use.md).

    При наличии долго выполняющегося запроса, который блокирует других пользователей и не может быть оптимизирован, рассмотрите возможность перемещения его из среды OLTP в выделенную систему отчетов — [синхронную реплику базы данных, доступную только для чтения](read-scale-out.md).

1.  Блокировка вызвана спящим SPID с незафиксированной транзакцией

    Этот тип блокировки часто может быть идентифицирован по SPID, который находится в спящем режиме, или ожидает команды, для которой уровень вложенности транзакций `@@TRANCOUNT` ( `open_transaction_count` от sys.dm_exec_requests) больше нуля. Это может произойти, если время ожидания запроса в приложении истекло или при отмене не выдается необходимое число инструкций ROLLBACK или COMMIT. Когда SPID получает время ожидания запроса или отмену, он прерывает текущий запрос и пакет, но не выполняет автоматический откат или фиксацию транзакции. Это приложение отвечает за это, так как база данных SQL Azure не может предположить, что должна быть произведена откат всей транзакции из-за отмены одного запроса. Время ожидания запроса или Cancel (Отмена) будет отображаться как событие сигнала предупреждения для SPID в сеансе расширенного события.

    Чтобы продемонстрировать незафиксированную явную транзакцию, выполните следующий запрос:

    ```sql
    CREATE TABLE #test (col1 INT);
    INSERT INTO #test SELECT 1;
    BEGIN TRAN
    UPDATE #test SET col1 = 2 where col1 = 1;
    ```

    Затем выполните этот запрос в том же окне:
    ```sql
    SELECT @@TRANCOUNT;
    ROLLBACK TRAN
    DROP TABLE #test;
    ```

    Выходные данные второго запроса указывают на то, что уровень вложенности транзакций равен единице. Все блокировки, полученные в транзакции, по-прежнему удерживаются до момента фиксации или отката транзакции. Если приложения явно открывают и фиксируют транзакции, связь или другая ошибка могут покинуть сеанс и его транзакцию в открытом состоянии. 

    Используйте скрипт, приведенный ранее в этой статье, на основе sys.dm_tran_active_transactions для обнаружения незафиксированных транзакций в экземпляре.

    **Способы решения**:

     -   Кроме того, этот класс проблем с блокировкой может также быть проблемой с производительностью и потребовать от нее решения. Если время выполнения запроса может быть уменьшено, время ожидания запроса или отмены не произойдет. Важно, чтобы приложение могло справиться с истечением времени ожидания или отмены сценариев, но вы также можете воспользоваться преимуществами анализа производительности запроса. 
     
     -    Приложения должны должным образом управлять уровнями вложенности транзакций, или же они могут привести к возникновению блокирующей проблемы, следующей за отменой запроса таким образом. Рассмотрим следующий пример.  

            *    В обработчике ошибок клиентского приложения выполните `IF @@TRANCOUNT > 0 ROLLBACK TRAN` следующие ошибки, даже если клиентское приложение не считает, что транзакция открыта. Проверка наличия открытых транзакций необходима, так как хранимая процедура, вызываемая во время выполнения пакета, могла запустить транзакцию без ведома клиентского приложения. Некоторые условия, такие как отмена запроса, препятствуют выполнению процедуры после текущей инструкции, поэтому даже если процедура имеет логику для проверки `IF @@ERROR <> 0` и прерывания транзакции, этот код отката не будет выполняться в таких случаях.  
            *    Если пул соединений используется в приложении, которое открывает подключение, и выполняет небольшое количество запросов перед тем, как освободить подключение к пулу, например веб-приложение, временное отключение пулов соединений может помочь устранить проблему, пока клиентское приложение не будет изменено для правильной поддержки ошибок. При отключении пула подключений освобождение подключения приведет к физическому отключению подключения к базе данных SQL Azure, в результате чего сервер будет выполнять откат всех открытых транзакций.  
            *    Используется `SET XACT_ABORT ON` для соединения или в любых хранимых процедурах, которые начинают транзакции и не удаляются после ошибки. В случае ошибки времени выполнения этот параметр прервет все открытые транзакции и возвратит управление клиенту. Дополнительные сведения см. в инструкции [SET XACT_ABORT (Transact-SQL)](/sql/t-sql/statements/set-xact-abort-transact-sql).

    > [!NOTE]
    > Соединение не сбрасывается, пока оно не будет повторно использовано из пула соединений, поэтому возможно, что пользователь сможет открыть транзакцию, а затем отпустить подключение к пулу подключений, но может быть не использован повторно в течение нескольких секунд, в течение которого транзакция будет оставаться открытой. Если соединение не используется повторно, транзакция будет прервана при истечении времени ожидания соединения и удалена из пула соединений. Таким образом, для клиентского приложения оптимально прерывать транзакции в обработчике ошибок или использовать `SET XACT_ABORT ON` во избежание этой потенциальной задержки.

    > [!CAUTION]
    > Далее `SET XACT_ABORT ON` инструкции T-SQL, следующие за инструкцией, которая вызывает ошибку, не будут выполнены. Это может повлиять на предполагаемый поток существующего кода.

1.  Блокировка вызвана идентификатором SPID, соответствующее клиентское приложение не получало все строки результатов до завершения

    После отправки запроса на сервер все приложения должны сразу же получить все строки результатов до завершения. Если приложение не извлекает все результирующие строки, то блокировки могут остаться в таблицах, блокируя других пользователей. Если вы используете приложение, которое прозрачно отправляет инструкции SQL на сервер, приложение должно получить все строки результатов. Если это не так (и не может быть настроено для этого), возможно, не удастся устранить проблему с блокировкой. Чтобы избежать этой проблемы, можно ограничить неверно работающие приложения в отчетности или базе данных поддержки принятия решений, отдельно от основной базы данных OLTP.
    
    > [!NOTE]
    > См. [руководство по логике повторных попыток](./troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors) для приложений, подключающихся к базе данных SQL Azure. 
    
    **Решение**. чтобы получить все строки результата до завершения, необходимо перезаписать приложение. Это не повлечет за собой использование [смещения и выборки в предложении ORDER BY](/sql/t-sql/queries/select-order-by-clause-transact-sql#using-offset-and-fetch-to-limit-the-rows-returned) запроса для выполнения разбиения на страницы на стороне сервера.

1.  Блокировка, вызванная сеансом в состоянии отката

    Будет выполнен откат запроса на изменение данных, который был прерван или отменен вне определенной пользователем транзакции. Это также может происходить в качестве побочного результата отключения сетевого сеанса клиента или при выборе запроса в качестве жертвы взаимоблокировки. Это часто можно определить, просматривая выходные данные sys.dm_exec_requests, которые могут указывать на **команду** Rollback, а **столбец percent_complete** может показывать ход выполнения. 

    Благодаря [функции ускоренного восстановления баз данных](../accelerated-database-recovery.md) , представленной в 2019, длинные откаты должны быть редкими.
    
    **Решение**. Дождитесь, пока SPID завершит откат выполненных изменений. 

    Чтобы избежать такой ситуации, не следует выполнять большие операции пакетной записи или создавать индексы или операции обслуживания в часы работы систем OLTP. Если возможно, выполните такие операции в периоды низкой активности.

1.  Блокировка вызвана потерянным соединением

    Если клиентское приложение вызывает ошибки или клиентская рабочая станция перезапускается, сетевой сеанс сервера может быть не немедленно отменен при некоторых условиях. С точки зрения базы данных SQL Azure клиент по-прежнему отображается, и все полученные блокировки могут по-прежнему оставаться в системе. Дополнительные сведения см. [в разделе Устранение неполадок потерянных подключений в SQL Server](/sql/t-sql/language-elements/kill-transact-sql#remarks). 

    **Решение**. Если клиентское приложение отключено без соответствующей очистки ресурсов, можно завершить SPID с помощью `KILL` команды. `KILL`Команда принимает значение SPID в качестве входных данных. Например, чтобы уничтожить SPID 99, выполните следующую команду:

    ```sql
    KILL 99
    ```

## <a name="see-also"></a>См. также раздел

* [Мониторинг и настройка производительности Базы данных SQL Azure и Управляемого экземпляра SQL Azure](./monitor-tune-overview.md)
* [Мониторинг производительности с использованием хранилища запросов](/sql/relational-databases/performance/monitoring-performance-by-using-the-query-store)
* [Руководство по блокировке и управлению версиями строк транзакций](/sql/relational-databases/sql-server-transaction-locking-and-row-versioning-guide)
* [SET TRANSACTION ISOLATION LEVEL](/sql/t-sql/statements/set-transaction-isolation-level-transact-sql)
* [Краткое руководство. Расширенные события в SQL Server](/sql/relational-databases/extended-events/quick-start-extended-events-in-sql-server)
* [Функция Intelligent Insights использует искусственный интеллект для мониторинга производительности базы данных и устранения связанных с ней проблем.](intelligent-insights-overview.md)

## <a name="learn-more"></a>Дополнительные сведения

* [База данных SQL Azure: улучшение настройки производительности с помощью автоматической настройки](https://channel9.msdn.com/Shows/Data-Exposed/Azure-SQL-Database-Improving-Performance-Tuning-with-Automatic-Tuning)
* [Повышение производительности базы данных SQL Azure с помощью автоматической настройки](https://channel9.msdn.com/Shows/Azure-Friday/Improve-Azure-SQL-Database-Performance-with-Automatic-Tuning)
* [Обеспечьте стабильную производительность с помощью SQL Azure](/learn/modules/azure-sql-performance/)
* [Устранение проблем с подключением и другие ошибки в базе данных SQL Azure и Azure SQL Управляемый экземпляр](troubleshoot-common-errors-issues.md)
* [Обработка временных сбоев](/aspnet/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/transient-fault-handling)
