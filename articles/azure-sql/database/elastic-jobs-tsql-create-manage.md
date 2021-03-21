---
title: Создание заданий обработки эластичных баз данных и управление ими (Предварительная версия) с помощью Transact-SQL (T-SQL)
description: Выполнение сценариев с участием нескольких баз данных с помощью агента обработки заданий эластичной базы данных и Transact-SQL (T-SQL).
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
dev_langs:
- TSQL
ms.topic: how-to
ms.author: jaredmoo
author: jaredmoo
ms.reviewer: sstein
ms.date: 02/01/2021
ms.openlocfilehash: 11b94ba5bcedf56f0115b8730dc58f808aff5c58
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100371606"
---
# <a name="use-transact-sql-t-sql-to-create-and-manage-elastic-database-jobs-preview"></a>Использование Transact-SQL (T-SQL) для создания заданий обработки эластичных баз данных и управления ими (Предварительная версия)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этой статье приведено множество примеров сценариев, позволяющих приступить к использованию эластичных заданий с помощью T-SQL.

В этих примерах используются [хранимые процедуры](#job-stored-procedures) и [представления](#job-views), доступные в [*базе данных задания*](job-automation-overview.md#elastic-job-database).

Transact-SQL (T-SQL) используется для создания, настройки, выполнения заданий и управления ими. Создание агента заданий обработки эластичных баз данных в T-SQL не поддерживается, поэтому нужно сначала создать *агент заданий обработки эластичных баз данных* с помощью портала или [PowerShell](elastic-jobs-powershell-create.md#create-the-elastic-job-agent).

## <a name="create-a-credential-for-job-execution"></a>Создание учетных данных для выполнения заданий

Учетные данные используются для подключения к целевым базам данных при выполнении сценария. Для успешного выполнения сценария этим учетным данным необходимы соответствующие разрешения для баз данных, указанных в целевой группе. При использовании логического члена целевой группы [SQL Server](logical-servers.md) или пула настоятельно рекомендуется создать учетные данные для использования при обновлении учетных данных до расширения сервера и (или) пула во время выполнения задания. Учетные данные базы данных должны быть созданы в базе данных агента заданий. Эти же учетные данные должны использоваться для *создания имени для входа* и *создания пользователя на основании имени для входа для предоставления разрешений на вход в базу данных* для целевых баз данных.

```sql
--Connect to the new job database specified when creating the Elastic Job agent

-- Create a database master key if one does not already exist, using your own password.  
CREATE MASTER KEY ENCRYPTION BY PASSWORD='<EnterStrongPasswordHere>';  
  
-- Create two database scoped credentials.  
-- The credential to connect to the Azure SQL logical server, to execute jobs
CREATE DATABASE SCOPED CREDENTIAL job_credential WITH IDENTITY = 'job_credential',
    SECRET = '<EnterStrongPasswordHere>';
GO
-- The credential to connect to the Azure SQL logical server, to refresh the database metadata in server
CREATE DATABASE SCOPED CREDENTIAL refresh_credential WITH IDENTITY = 'refresh_credential',
    SECRET = '<EnterStrongPasswordHere>';
GO
```

## <a name="create-a-target-group-servers"></a>Создание целевой группы (серверов)

В приведенном ниже примере показано, как выполнить задание со всеми базами данных на сервере.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
-- Connect to the job database specified when creating the job agent

-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group 'ServerGroup1';

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = 'ServerGroup1',
@target_type = 'SqlServer',
@refresh_credential_name = 'refresh_credential', --credential required to refresh the databases in a server
@server_name = 'server1.database.windows.net';

--View the recently created target group and target group members
SELECT * FROM jobs.target_groups WHERE target_group_name='ServerGroup1';
SELECT * FROM jobs.target_group_members WHERE target_group_name='ServerGroup1';
```

## <a name="exclude-an-individual-database"></a>Исключение отдельной базы данных

В следующем примере показано, как выполнить задание для всех баз данных на сервере, за исключением базы данных с именем *маппингдб*.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

-- Add a target group containing server(s)
EXEC [jobs].sp_add_target_group N'ServerGroup';
GO

-- Add a server target member
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@target_type = N'SqlServer',
@refresh_credential_name = N'refresh_credential', --credential required to refresh the databases in a server
@server_name = N'London.database.windows.net';
GO

-- Add a server target member
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@target_type = N'SqlServer',
@refresh_credential_name = N'refresh_credential', --credential required to refresh the databases in a server
@server_name = 'server2.database.windows.net';
GO

--Exclude a database target member from the server target group
EXEC [jobs].sp_add_target_group_member
@target_group_name = N'ServerGroup',
@membership_type = N'Exclude',
@target_type = N'SqlDatabase',
@server_name = N'server1.database.windows.net',
@database_name = N'MappingDB';
GO

--View the recently created target group and target group members
SELECT * FROM [jobs].target_groups WHERE target_group_name = N'ServerGroup';
SELECT * FROM [jobs].target_group_members WHERE target_group_name = N'ServerGroup';
```

## <a name="create-a-target-group-pools"></a>Создание целевой группы (пулов)

В примере ниже показано, как выбрать в качестве цели все базы данных в одном или нескольких эластичных пулах.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

-- Add a target group containing pool(s)
EXEC jobs.sp_add_target_group 'PoolGroup';

-- Add an elastic pool(s) target member
EXEC jobs.sp_add_target_group_member
@target_group_name = 'PoolGroup',
@target_type = 'SqlElasticPool',
@refresh_credential_name = 'refresh_credential', --credential required to refresh the databases in a server
@server_name = 'server1.database.windows.net',
@elastic_pool_name = 'ElasticPool-1';

-- View the recently created target group and target group members
SELECT * FROM jobs.target_groups WHERE target_group_name = N'PoolGroup';
SELECT * FROM jobs.target_group_members WHERE target_group_name = N'PoolGroup';
```

## <a name="deploy-new-schema-to-many-databases"></a>Развертывание новой схемы в нескольких базах данных

В приведенном ниже примере показано, как развернуть новую схему во всех базах данных.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

--Add job for create table
EXEC jobs.sp_add_job @job_name = 'CreateTableTest', @description = 'Create Table Test';

-- Add job step for create table
EXEC jobs.sp_add_jobstep @job_name = 'CreateTableTest',
@command = N'IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id(''Test''))
CREATE TABLE [dbo].[Test]([TestId] [int] NOT NULL);',
@credential_name = 'job_credential',
@target_group_name = 'PoolGroup';
```

## <a name="data-collection-using-built-in-parameters"></a>Сбор данных с помощью встроенных параметров

Во множестве сценариев сбора данных может быть удобно добавить некоторые из приведенных переменных сценариев, чтобы упростить последующую обработку результатов задания.

- $(job_name)
- $(job_id)
- $(job_version)
- $(step_id)
- $(step_name)
- $(job_execution_id)
- $(job_execution_create_time)
- $(target_group_name)

Например, чтобы сгруппировать все результаты выполнения одного задания, используйте *$(job_execution_id)*, как показано в команде ниже.

```sql
@command= N' SELECT DB_NAME() DatabaseName, $(job_execution_id) AS job_execution_id, * FROM sys.dm_db_resource_stats WHERE end_time > DATEADD(mi, -20, GETDATE());'
```

## <a name="monitor-database-performance"></a>Мониторинг производительности базы данных

Приведенный ниже пример создает задание для сбора данных производительности из нескольких баз данных.

По умолчанию агент заданий создаст выходную таблицу для хранения возвращаемых результатов. Поэтому участник базы данных, связанный с выходными учетными данными, должен иметь как минимум следующие разрешения: `CREATE TABLE` в базе данных,,, `ALTER` `SELECT` `INSERT` `DELETE` в выходной таблице или ее схеме, а также `SELECT` в представлении каталога [sys. indexes](/sql/relational-databases/system-catalog-views/sys-indexes-transact-sql) .

Если необходимо вручную создать таблицу заранее, необходимо иметь следующие свойства:

1. Столбцы с правильными именами и типами данных для результирующего набора.
2. Дополнительный столбец для internal_execution_id с типом данных uniqueidentifier.
3. Некластеризованный индекс, указанный `IX_<TableName>_Internal_Execution_ID` в столбце internal_execution_id.
4. Все перечисленные выше разрешения, за исключением `CREATE TABLE` разрешения для базы данных.

Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команды, приведенные ниже.

```sql
--Connect to the job database specified when creating the job agent

-- Add a job to collect perf results
EXEC jobs.sp_add_job @job_name ='ResultsJob', @description='Collection Performance data from all customers'

-- Add a job step w/ schedule to collect results
EXEC jobs.sp_add_jobstep
@job_name = 'ResultsJob',
@command = N' SELECT DB_NAME() DatabaseName, $(job_execution_id) AS job_execution_id, * FROM sys.dm_db_resource_stats WHERE end_time > DATEADD(mi, -20, GETDATE());',
@credential_name = 'job_credential',
@target_group_name = 'PoolGroup',
@output_type = 'SqlDatabase',
@output_credential_name = 'job_credential',
@output_server_name = 'server1.database.windows.net',
@output_database_name = '<resultsdb>',
@output_table_name = '<resutlstable>';

--Create a job to monitor pool performance

--Connect to the job database specified when creating the job agent

-- Add a target group containing Elastic Job database
EXEC jobs.sp_add_target_group 'ElasticJobGroup';

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = 'ElasticJobGroup',
@target_type = 'SqlDatabase',
@server_name = 'server1.database.windows.net',
@database_name = 'master';

-- Add a job to collect perf results
EXEC jobs.sp_add_job
@job_name = 'ResultsPoolsJob',
@description = 'Demo: Collection Performance data from all pools',
@schedule_interval_type = 'Minutes',
@schedule_interval_count = 15;

-- Add a job step w/ schedule to collect results
EXEC jobs.sp_add_jobstep
@job_name='ResultsPoolsJob',
@command=N'declare @now datetime
DECLARE @startTime datetime
DECLARE @endTime datetime
DECLARE @poolLagMinutes datetime
DECLARE @poolStartTime datetime
DECLARE @poolEndTime datetime
SELECT @now = getutcdate ()
SELECT @startTime = dateadd(minute, -15, @now)
SELECT @endTime = @now
SELECT @poolStartTime = dateadd(minute, -30, @startTime)
SELECT @poolEndTime = dateadd(minute, -30, @endTime)

SELECT elastic_pool_name , end_time, elastic_pool_dtu_limit, avg_cpu_percent, avg_data_io_percent, avg_log_write_percent, max_worker_percent, max_session_percent,
        avg_storage_percent, elastic_pool_storage_limit_mb FROM sys.elastic_pool_resource_stats
        WHERE end_time > @poolStartTime and end_time <= @poolEndTime;
'),
@credential_name = 'job_credential',
@target_group_name = 'ElasticJobGroup',
@output_type = 'SqlDatabase',
@output_credential_name = 'job_credential',
@output_server_name = 'server1.database.windows.net',
@output_database_name = 'resultsdb',
@output_table_name = 'resutlstable';
```

## <a name="view-job-definitions"></a>Просмотр определений заданий

В следующем примере показано, как просмотреть определения текущих заданий.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

-- View all jobs
SELECT * FROM jobs.jobs;

-- View the steps of the current version of all jobs
SELECT js.* FROM jobs.jobsteps js
JOIN jobs.jobs j
  ON j.job_id = js.job_id AND j.job_version = js.job_version;

-- View the steps of all versions of all jobs
SELECT * FROM jobs.jobsteps;
```

## <a name="begin-unplanned-execution-of-a-job"></a>Начало незапланированного выполнения задания

В приведенном ниже примере показано, как запустить задание немедленно.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

-- Execute the latest version of a job
EXEC jobs.sp_start_job 'CreateTableTest';

-- Execute the latest version of a job and receive the execution id
declare @je uniqueidentifier;
exec jobs.sp_start_job 'CreateTableTest', @job_execution_id = @je output;
select @je;

select * from jobs.job_executions where job_execution_id = @je;

-- Execute a specific version of a job (e.g. version 1)
exec jobs.sp_start_job 'CreateTableTest', 1;
```

## <a name="schedule-execution-of-a-job"></a>Планирование выполнения задания

В следующем примере показано, как запланировать выполнение задания в будущем.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

EXEC jobs.sp_update_job
@job_name = 'ResultsJob',
@enabled=1,
@schedule_interval_type = 'Minutes',
@schedule_interval_count = 15;
```

## <a name="monitor-job-execution-status"></a>Отслеживание состояния выполнения задания

В приведенном ниже примере показано, как просмотреть сведения о состоянии выполнения всех заданий.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

--View top-level execution status for the job named 'ResultsPoolJob'
SELECT * FROM jobs.job_executions
WHERE job_name = 'ResultsPoolsJob' and step_id IS NULL
ORDER BY start_time DESC;

--View all top-level execution status for all jobs
SELECT * FROM jobs.job_executions WHERE step_id IS NULL
ORDER BY start_time DESC;

--View all execution statuses for job named 'ResultsPoolsJob'
SELECT * FROM jobs.job_executions
WHERE job_name = 'ResultsPoolsJob'
ORDER BY start_time DESC;

-- View all active executions
SELECT * FROM jobs.job_executions
WHERE is_active = 1
ORDER BY start_time DESC;
```

## <a name="cancel-a-job"></a>Отмена задания

В следующем примере показано, как отменить задание.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

-- View all active executions to determine job execution id
SELECT * FROM jobs.job_executions
WHERE is_active = 1 AND job_name = 'ResultPoolsJob'
ORDER BY start_time DESC;
GO

-- Cancel job execution with the specified job execution id
EXEC jobs.sp_stop_job '01234567-89ab-cdef-0123-456789abcdef';
```

## <a name="delete-old-job-history"></a>Удаление старого журнала заданий

В приведенном ниже примере показано, как удалить старый журнал заданий до определенной даты.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

-- Delete history of a specific job's executions older than the specified date
EXEC jobs.sp_purge_jobhistory @job_name='ResultPoolsJob', @oldest_date='2016-07-01 00:00:00';

--Note: job history is automatically deleted if it is >45 days old
```

## <a name="delete-a-job-and-all-its-job-history"></a>Удаление задания и его журнала заданий

В приведенном ниже примере показано, как удалить задание и связанный с ним журнал заданий.  
Подключитесь к [*базе данных заданий*](job-automation-overview.md#elastic-job-database) и выполните команду, приведенную ниже.

```sql
--Connect to the job database specified when creating the job agent

EXEC jobs.sp_delete_job @job_name='ResultsPoolsJob';

--Note: job history is automatically deleted if it is >45 days old
```

## <a name="job-stored-procedures"></a>Хранимые процедуры задания

В [базе данных заданий](job-automation-overview.md#elastic-job-database) доступны следующие хранимые процедуры.

|Хранимая процедура  |Описание  |
|---------|---------|
|[sp_add_job](#sp_add_job)     |     Добавляет новое задание.    |
|[sp_update_job, хранимая процедура](#sp_update_job)    |      Обновляет существующее задание.   |
|[sp_delete_job](#sp_delete_job)     |      Удаляет существующее задание.   |
|[sp_add_jobstep](#sp_add_jobstep)    |    Добавляет шаг в задание.     |
|[sp_update_jobstep](#sp_update_jobstep)     |     Обновляет шаг задания.    |
|[sp_delete_jobstep](#sp_delete_jobstep)     |     Удаляет шаг задания.    |
|[sp_start_job](#sp_start_job)    |  Запускает выполнение задания.       |
|[sp_stop_job](#sp_stop_job)     |     Останавливает выполнение задания.   |
|[sp_add_target_group](#sp_add_target_group)    |     Добавляет целевую группу.    |
|[sp_delete_target_group](#sp_delete_target_group)     |    Удаляет целевую группу.     |
|[sp_add_target_group_member](#sp_add_target_group_member)     |    Добавляет базу данных или несколько баз данных в целевую группу.     |
|[sp_delete_target_group_member](#sp_delete_target_group_member)     |     Удаляет элемент из целевой группы.    |
|[sp_purge_jobhistory, хранимая процедура](#sp_purge_jobhistory)    |    Удаляет записи журнала для задания.     |

### <a name="sp_add_job"></a><a name="sp_add_job"></a>sp_add_job

Добавляет новое задание.
  
#### <a name="syntax"></a>Синтаксис  
  
```syntaxsql
[jobs].sp_add_job [ @job_name = ] 'job_name'  
  [ , [ @description = ] 'description' ]
  [ , [ @enabled = ] enabled ]
  [ , [ @schedule_interval_type = ] schedule_interval_type ]  
  [ , [ @schedule_interval_count = ] schedule_interval_count ]
  [ , [ @schedule_start_time = ] schedule_start_time ]
  [ , [ @schedule_end_time = ] schedule_end_time ]
  [ , [ @job_id = ] job_id OUTPUT ]
```

#### <a name="arguments"></a>Аргументы  

[ **\@ job_name =** ] ' job_name '  
Имя задания. Это имя должно быть уникальным и не должно содержать символ процента (%). Типом job_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ Description =** ] "Description"  
Описание задания. Типом description является nvarchar(512), значение по умолчанию — NULL. Если описание не указано, используется пустая строка.

[ **\@ включено =** ] включено  
Включено ли расписание задания. Типом enabled является bit, значение по умолчанию — 0 (отключено). При значении 0 задание не включено и не выполняется по заданному расписанию, однако его можно запустить вручную. При значении 1 задание выполняется по своему расписанию, а также может быть выполнено вручную.

[ **\@ schedule_interval_type =**] schedule_interval_type  
Это значение указывает, когда должно выполняться задание. Типом schedule_interval_type является nvarchar(50), значение по умолчанию — Once. Допускаются следующие значения:

- Once
- Minutes
- Hours
- Days
- Weeks
- Months

[ **\@ schedule_interval_count =** ] schedule_interval_count  
Число периодов schedule_interval_count между выполнениями задания. Типом schedule_interval_count является int, значение по умолчанию — 1. Это значение должно быть больше или равно 1.

[ **\@ schedule_start_time =** ] schedule_start_time  
Дата, когда может начаться выполнение задания. Типом schedule_start_time является DATETIME2, значение по умолчанию — 0001-01-01 00:00:00.0000000.

[ **\@ schedule_end_time =** ] schedule_end_time  
Дата, когда может быть остановлено выполнение задания. Типом schedule_end_time является DATETIME2, значение по умолчанию — 9999-12-31 11:59:59.0000000.

[ **\@ job_id =** ] job_id выходные данные  
Идентификационный номер задания, присваиваемый заданию после успешного создания. job_id является выходной переменно типа uniqueidentifier.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

sp_add_job необходимо выполнять из базы данных агента заданий, указанной при создании этого агента.
После выполнения sp_add_job для добавления задания можно выполнить sp_add_jobstep, чтобы добавить в задание шаги для выполнения действий. Первоначальный номер версии задания равен 0, что будет увеличиваться до 1 при добавлении первого шага.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_update_job"></a><a name="sp_update_job"></a>sp_update_job, хранимая процедура

Обновляет существующее задание.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_update_job [ @job_name = ] 'job_name'  
  [ , [ @new_name = ] 'new_name' ]
  [ , [ @description = ] 'description' ]
  [ , [ @enabled = ] enabled ]
  [ , [ @schedule_interval_type = ] schedule_interval_type ]  
  [ , [ @schedule_interval_count = ] schedule_interval_count ]
  [ , [ @schedule_start_time = ] schedule_start_time ]
  [ , [ @schedule_end_time = ] schedule_end_time ]
```

#### <a name="arguments"></a>Аргументы

[ **\@ job_name =** ] ' job_name '  
Имя обновляемого задания. Типом job_name является nvarchar(128).

[ **\@ new_name =** ] ' new_name '  
Новое имя задания. Типом new_name является nvarchar(128).

[ **\@ Description =** ] "Description"  
Описание задания. Типом description является nvarchar(512).

[ **\@ включено =** ] включено  
Указывает, включено ли расписание задания (1) или не включено (0). Типом enabled является bit.

[ **\@ schedule_interval_type =** ] schedule_interval_type  
Это значение указывает, когда должно выполняться задание. Типом schedule_interval_type является nvarchar(50). Допускаются следующие значения:

- Once
- Minutes
- Hours
- Days
- Weeks
- Months

[ **\@ schedule_interval_count =** ] schedule_interval_count  
Число периодов schedule_interval_count между выполнениями задания. Типом schedule_interval_count является int, значение по умолчанию — 1. Это значение должно быть больше или равно 1.

[ **\@ schedule_start_time =** ] schedule_start_time  
Дата, когда может начаться выполнение задания. Типом schedule_start_time является DATETIME2, значение по умолчанию — 0001-01-01 00:00:00.0000000.

[ **\@ schedule_end_time =** ] schedule_end_time  
Дата, когда может быть остановлено выполнение задания. Типом schedule_end_time является DATETIME2, значение по умолчанию — 9999-12-31 11:59:59.0000000.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

После выполнения sp_add_job для добавления задания можно выполнить sp_add_jobstep, чтобы добавить в задание шаги для выполнения действий. Первоначальный номер версии задания равен 0, что будет увеличиваться до 1 при добавлении первого шага.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_delete_job"></a><a name="sp_delete_job"></a>sp_delete_job

Удаляет существующее задание.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_delete_job [ @job_name = ] 'job_name'
  [ , [ @force = ] force ]
```

#### <a name="arguments"></a>Аргументы

[ **\@ job_name =** ] ' job_name '  
Имя удаляемого задания. Типом job_name является nvarchar(128).

[ **\@ Force =** ] принудительно  
Указывает, следует ли выполнить удаление, отменив все текущие выполнения задания, если оно выполняется (1), или следует выдать ошибку (0). Типом force является bit.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

Журнал заданий автоматически удаляется вместе с заданием.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_add_jobstep"></a><a name="sp_add_jobstep"></a>sp_add_jobstep

Добавляет шаг в задание.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_add_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] step_name ]
     [ , [ @command_type = ] 'command_type' ]
     [ , [ @command_source = ] 'command_source' ]  
     , [ @command = ] 'command'
     , [ @credential_name = ] 'credential_name'
     , [ @target_group_name = ] 'target_group_name'
     [ , [ @initial_retry_interval_seconds = ] initial_retry_interval_seconds ]
     [ , [ @maximum_retry_interval_seconds = ] maximum_retry_interval_seconds ]
     [ , [ @retry_interval_backoff_multiplier = ] retry_interval_backoff_multiplier ]
     [ , [ @retry_attempts = ] retry_attempts ]
     [ , [ @step_timeout_seconds = ] step_timeout_seconds ]
     [ , [ @output_type = ] 'output_type' ]
     [ , [ @output_credential_name = ] 'output_credential_name' ]
     [ , [ @output_subscription_id = ] 'output_subscription_id' ]
     [ , [ @output_resource_group_name = ] 'output_resource_group_name' ]
     [ , [ @output_server_name = ] 'output_server_name' ]
     [ , [ @output_database_name = ] 'output_database_name' ]
     [ , [ @output_schema_name = ] 'output_schema_name' ]
     [ , [ @output_table_name = ] 'output_table_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
     [ , [ @max_parallelism = ] max_parallelism ]
```

#### <a name="arguments"></a>Аргументы

[ **\@ job_name =** ] ' job_name '  
Имя задания, к которому добавляется этап. Типом job_name является nvarchar(128).

[ **\@ step_id =** ] step_id  
Порядковый идентификационный номер для шага задания. Последовательные идентификационные номера начинаются с 1 и увеличиваются без пропусков. Если у существующего шага уже есть этот идентификатор, его идентификатор будет увеличиваться, чтобы можно было вставить этот новый шаг в последовательность. Если идентификатор не указан, то step_id назначается автоматически всем шагам в последовательности. Типом step_id является int.

[ **\@ step_name =** ] step_name  
Имя шага этапа. Необходимо указать, за исключением первого шага задания, которое (для удобства) имеет имя по умолчанию "JobStep". Типом step_name является nvarchar(128).

[ **\@ command_type =** ] ' command_type '  
Тип команды, выполняемой данным шагом задания. Типом command_type является nvarchar(50), значение по умолчанию — TSql (указывает, что значением параметра @command_type является сценарий T-SQL).

Если этот аргумент указывается, его значением должно быть TSql.

[ **\@ command_source =** ] ' command_source '  
Тип расположения, в котором хранится команда. Типом command_source является nvarchar(50), значение по умолчанию — Inline (указывает, что значением параметра @command_source является литеральный ткст команды).

Если этот аргумент указывается, его значением должно быть Inline.

[ **\@ команда =** ] ' Command '  
Аргументом command должен быть допустимый сценарий T-SQL, который выполняется данным шагом задания. Типом command является nvarchar(max), значение по умолчанию — NULL.

[ **\@ credential_name =** ] ' credential_name '  
Имя учетных данных базы данных, которые хранятся в этой базе данных управления заданиями и используются для подключения к целевым базам данных в целевой группе при выполнении данного шага. Типом credential_name является nvarchar(128).

[ **\@ target_group_name =** ] ' target-group_name '  
Имя целевой группы, содержащей целевые базы данных, для которых будет выполнен шаг задания. Типом target_group_name является nvarchar(128).

[ **\@ initial_retry_interval_seconds =** ] initial_retry_interval_seconds  
Задержка перед первой повторной попыткой в случае сбоя при первой попытке выполнения задания. Типом initial_retry_interval_seconds является int, значение по умолчанию — 1.

[ **\@ maximum_retry_interval_seconds =** ] maximum_retry_interval_seconds  
Максимальная задержка между повторными попытками. Если задержка между повторными попытками превышает это значение, она ограничивается указанным значением. Типом maximum_retry_interval_seconds является int, значение по умолчанию — 120.

[ **\@ retry_interval_backoff_multiplier =** ] retry_interval_backoff_multiplier  
Коэффициент, на который умножается значение задержки перед повторной попыткой в случае нескольких попыток выполнения шага задания. Например, если перед первой попыткой задержка составляет 5 секунд, а коэффициент отхода равен 2,0, то задержка перед второй попыткой составит 10 секунд, а перед третьей — 20 секунд. Типом retry_interval_backoff_multiplier является real, значение по умолчанию — 2,0.

[ **\@ retry_attempts =** ] retry_attempts  
Число попыток выполнения в случае сбоя первоначальной попытки. Например, если значение retry_attempts равно 10, то будет выполнена 1 первоначальная попытка и 10 повторных попыток, то есть всего 11 попыток. В случае сбоя последней попытки выполнение задания прекращается с состоянием жизненного цикла Failed. Типом retry_attempts является int, значение по умолчанию — 10.

[ **\@ step_timeout_seconds =** ] step_timeout_seconds  
Максимальная продолжительность выполнения шага. При превышении этого значения выполнение задания завершается с состоянием жизненного цикла TimedOut. Типом step_timeout_seconds является int, значение по умолчанию — 43 200 секунд (12 часов).

[ **\@ output_type =** ] ' output_type '  
Если значение не равно null, то тип назначения, в который записывается первый результирующий набор команды. Типом output_type является nvarchar(50), значение по умолчанию — NULL.

Если этот аргумент указан, его значением должно быть SqlDatabase.

[ **\@ output_credential_name =** ] ' output_credential_name '  
Если значение не равно NULL, указывает имя учетных данных базы данных, используемых для подключения к выходной базе данных. Должен указываться, если значение output_type равно SqlDatabase. Типом output_credential_name является nvarchar(128), значение по умолчанию — NULL.

[ **\@ output_subscription_id =** ] ' output_subscription_id '  
Требуется описание.

[ **\@ output_resource_group_name =** ] ' output_resource_group_name '  
Требуется описание.

[ **\@ output_server_name =** ] ' output_server_name '  
Если значение не равно NULL, указывает полное DNS-имя сервера, содержащего целевую выходную базу данных. Должен указываться, если значение output_type равно SqlDatabase. Типом output_server_name является nvarchar(256), значение по умолчанию — NULL.

[ **\@ output_database_name =** ] ' output_database_name '  
Если значение не равно NULL, указывает имя базы данных, содержащей целевую выходную таблицу. Должен указываться, если значение output_type равно SqlDatabase. Типом output_database_name является nvarchar(128), значение по умолчанию — NULL.

[ **\@ output_schema_name =** ] ' output_schema_name '  
Если значение не равно NULL, указывает имя схемы SQL, содержащей целевую выходную таблицу. Если значение output_type равно SqlDatabase, то значение по умолчанию — dbo. Типом output_schema_name является nvarchar(128).

[ **\@ output_table_name =** ] ' output_table_name '  
Если не равно null, то имя таблицы, в которую будет записан первый результирующий набор команды. Если таблица не существует, она будет создана в соответствии со схемой возвращенного результирующего набора. Должен указываться, если значение output_type равно SqlDatabase. Типом output_table_name является nvarchar(128), значение по умолчанию — NULL.

[ **\@ job_version =** ] job_version выходные данные  
Параметр вывода, которому будет присвоен номер версии нового задания. Типом job_version является int.

[ **\@ max_parallelism =** ] max_parallelism выходные данные  
Максимальный уровень параллелизма на эластичный пул. Если этот аргумент задан, то шаг задания сможет обрабатывать не больше указанного максимального числа баз данных на эластичный пул. Это относится к каждому эластичному пулу, который либо непосредственно добавлен в целевую группу, либо размещен на сервере, добавленном в целевую группу. Типом max_parallelism является int.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

При завершении sp_add_jobstep номер текущей версии задания увеличивается. При следующем выполнении задания будет использоваться его новая версия. Если идет выполнение задания, то оно не будет включать в себя новый шаг.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.  

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_update_jobstep"></a><a name="sp_update_jobstep"></a>sp_update_jobstep

Обновляет шаг задания.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_update_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] 'step_name' ]
     [ , [ @new_id = ] new_id ]
     [ , [ @new_name = ] 'new_name' ]
     [ , [ @command_type = ] 'command_type' ]
     [ , [ @command_source = ] 'command_source' ]  
     , [ @command = ] 'command'
     , [ @credential_name = ] 'credential_name'
     , [ @target_group_name = ] 'target_group_name'
     [ , [ @initial_retry_interval_seconds = ] initial_retry_interval_seconds ]
     [ , [ @maximum_retry_interval_seconds = ] maximum_retry_interval_seconds ]
     [ , [ @retry_interval_backoff_multiplier = ] retry_interval_backoff_multiplier ]
     [ , [ @retry_attempts = ] retry_attempts ]
     [ , [ @step_timeout_seconds = ] step_timeout_seconds ]
     [ , [ @output_type = ] 'output_type' ]
     [ , [ @output_credential_name = ] 'output_credential_name' ]
     [ , [ @output_server_name = ] 'output_server_name' ]
     [ , [ @output_database_name = ] 'output_database_name' ]
     [ , [ @output_schema_name = ] 'output_schema_name' ]
     [ , [ @output_table_name = ] 'output_table_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
     [ , [ @max_parallelism = ] max_parallelism ]
```

#### <a name="arguments"></a>Аргументы

[ **\@ job_name =** ] ' job_name '  
Имя задания, которому принадлежит шаг. Типом job_name является nvarchar(128).

[ **\@ step_id =** ] step_id  
Идентификационный номер шага задания, которое необходимо изменить. Требуется указать step_id или step_name. Типом step_id является int.

[ **\@ step_name =** ] ' step_name '  
Имя изменяемого шага. Требуется указать step_id или step_name. Типом step_name является nvarchar(128).

[ **\@ new_id =** ] new_id  
Новый последовательный идентификационный номер шага задания. Последовательные идентификационные номера начинаются с 1 и увеличиваются без пропусков. Если порядок шага изменяется, остальные шаги автоматически перенумеровываются.

[ **\@ new_name =** ] ' new_name '  
Новое имя шага. Типом new_name является nvarchar(128).

[ **\@ command_type =** ] ' command_type '  
Тип команды, выполняемой данным шагом задания. Типом command_type является nvarchar(50), значение по умолчанию — TSql (указывает, что значением параметра @command_type является сценарий T-SQL).

Если этот аргумент указывается, его значением должно быть TSql.

[ **\@ command_source =** ] ' command_source '  
Тип расположения, в котором хранится команда. Типом command_source является nvarchar(50), значение по умолчанию — Inline (указывает, что значением параметра @command_source является литеральный ткст команды).

Если этот аргумент указывается, его значением должно быть Inline.

[ **\@ команда =** ] ' Command '  
Аргументом command должен быть допустимый сценарий T-SQL, который выполняется данным шагом задания. Типом command является nvarchar(max), значение по умолчанию — NULL.

[ **\@ credential_name =** ] ' credential_name '  
Имя учетных данных базы данных, которые хранятся в этой базе данных управления заданиями и используются для подключения к целевым базам данных в целевой группе при выполнении данного шага. Типом credential_name является nvarchar(128).

[ **\@ target_group_name =** ] ' target-group_name '  
Имя целевой группы, содержащей целевые базы данных, для которых будет выполнен шаг задания. Типом target_group_name является nvarchar(128).

[ **\@ initial_retry_interval_seconds =** ] initial_retry_interval_seconds  
Задержка перед первой повторной попыткой в случае сбоя при первой попытке выполнения задания. Типом initial_retry_interval_seconds является int, значение по умолчанию — 1.

[ **\@ maximum_retry_interval_seconds =** ] maximum_retry_interval_seconds  
Максимальная задержка между повторными попытками. Если задержка между повторными попытками превышает это значение, она ограничивается указанным значением. Типом maximum_retry_interval_seconds является int, значение по умолчанию — 120.

[ **\@ retry_interval_backoff_multiplier =** ] retry_interval_backoff_multiplier  
Коэффициент, на который умножается значение задержки перед повторной попыткой в случае нескольких попыток выполнения шага задания. Например, если перед первой попыткой задержка составляет 5 секунд, а коэффициент отхода равен 2,0, то задержка перед второй попыткой составит 10 секунд, а перед третьей — 20 секунд. Типом retry_interval_backoff_multiplier является real, значение по умолчанию — 2,0.

[ **\@ retry_attempts =** ] retry_attempts  
Число попыток выполнения в случае сбоя первоначальной попытки. Например, если значение retry_attempts равно 10, то будет выполнена 1 первоначальная попытка и 10 повторных попыток, то есть всего 11 попыток. В случае сбоя последней попытки выполнение задания прекращается с состоянием жизненного цикла Failed. Типом retry_attempts является int, значение по умолчанию — 10.

[ **\@ step_timeout_seconds =** ] step_timeout_seconds  
Максимальная продолжительность выполнения шага. При превышении этого значения выполнение задания завершается с состоянием жизненного цикла TimedOut. Типом step_timeout_seconds является int, значение по умолчанию — 43 200 секунд (12 часов).

[ **\@ output_type =** ] ' output_type '  
Если значение не равно null, то тип назначения, в который записывается первый результирующий набор команды. Чтобы сбросить значение output_type до NULL, задайте для этого параметра значение '' (пустая строка). Типом output_type является nvarchar(50), значение по умолчанию — NULL.

Если этот аргумент указан, его значением должно быть SqlDatabase.

[ **\@ output_credential_name =** ] ' output_credential_name '  
Если значение не равно NULL, указывает имя учетных данных базы данных, используемых для подключения к выходной базе данных. Должен указываться, если значение output_type равно SqlDatabase. Чтобы сбросить значение output_credential_name до NULL, задайте для этого параметра значение '' (пустая строка). Типом output_credential_name является nvarchar(128), значение по умолчанию — NULL.

[ **\@ output_server_name =** ] ' output_server_name '  
Если значение не равно NULL, указывает полное DNS-имя сервера, содержащего целевую выходную базу данных. Должен указываться, если значение output_type равно SqlDatabase. Чтобы сбросить значение output_server_name до NULL, задайте для этого параметра значение '' (пустая строка). Типом output_server_name является nvarchar(256), значение по умолчанию — NULL.

[ **\@ output_database_name =** ] ' output_database_name '  
Если значение не равно NULL, указывает имя базы данных, содержащей целевую выходную таблицу. Должен указываться, если значение output_type равно SqlDatabase. Чтобы сбросить значение output_database_name до NULL, задайте для этого параметра значение '' (пустая строка). Типом output_database_name является nvarchar(128), значение по умолчанию — NULL.

[ **\@ output_schema_name =** ] ' output_schema_name '  
Если значение не равно NULL, указывает имя схемы SQL, содержащей целевую выходную таблицу. Если значение output_type равно SqlDatabase, то значение по умолчанию — dbo. Чтобы сбросить значение output_schema_name до NULL, задайте для этого параметра значение '' (пустая строка). Типом output_schema_name является nvarchar(128).

[ **\@ output_table_name =** ] ' output_table_name '  
Если не равно null, то имя таблицы, в которую будет записан первый результирующий набор команды. Если таблица не существует, она будет создана в соответствии со схемой возвращенного результирующего набора. Должен указываться, если значение output_type равно SqlDatabase. Чтобы сбросить значение output_server_name до NULL, задайте для этого параметра значение '' (пустая строка). Типом output_table_name является nvarchar(128), значение по умолчанию — NULL.

[ **\@ job_version =** ] job_version выходные данные  
Параметр вывода, которому будет присвоен номер версии нового задания. Типом job_version является int.

[ **\@ max_parallelism =** ] max_parallelism выходные данные  
Максимальный уровень параллелизма на эластичный пул. Если этот аргумент задан, то шаг задания сможет обрабатывать не больше указанного максимального числа баз данных на эластичный пул. Это относится к каждому эластичному пулу, который либо непосредственно добавлен в целевую группу, либо размещен на сервере, добавленном в целевую группу. Чтобы сбросить значение max_parallelism до NULL, задайте для этого параметра значение -1. Типом max_parallelism является int.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

Какие-либо текущие выполнения задания не затрагиваются. При завершении sp_update_jobstep номер версии задания увеличивается. При следующем выполнении задания будет использоваться его новая версия.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_delete_jobstep"></a><a name="sp_delete_jobstep"></a>sp_delete_jobstep

Удаляет шаг задания.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_delete_jobstep [ @job_name = ] 'job_name'
     [ , [ @step_id = ] step_id ]
     [ , [ @step_name = ] 'step_name' ]
     [ , [ @job_version = ] job_version OUTPUT ]
```

#### <a name="arguments"></a>Аргументы

[ **\@ job_name =** ] ' job_name '  
Имя задания, из которого удаляется этап. Типом job_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ step_id =** ] step_id  
Идентификационный номер удаляемого шага задания. Требуется указать step_id или step_name. Типом step_id является int.

[ **\@ step_name =** ] ' step_name '  
Имя удаляемого шага. Требуется указать step_id или step_name. Типом step_name является nvarchar(128).

[ **\@ job_version =** ] job_version выходные данные  
Параметр вывода, которому будет присвоен номер версии нового задания. Типом job_version является int.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

Какие-либо текущие выполнения задания не затрагиваются. При завершении sp_update_jobstep номер версии задания увеличивается. При следующем выполнении задания будет использоваться его новая версия.

Другие шаги задания будут автоматически перенумерованы, чтобы заполнить пропуск из-за удаленного шага задания.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_start_job"></a><a name="sp_start_job"></a>sp_start_job

Запускает выполнение задания.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_start_job [ @job_name = ] 'job_name'
     [ , [ @job_execution_id = ] job_execution_id OUTPUT ]
```

#### <a name="arguments"></a>Аргументы

[ **\@ job_name =** ] ' job_name '  
Имя задания, из которого удаляется этап. Типом job_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ job_execution_id =** ] job_execution_id выходные данные  
Выходной параметр, которому будет назначен идентификатор выполнения задания. job_version имеет тип uniqueidentifier.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

Нет.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_stop_job"></a><a name="sp_stop_job"></a>sp_stop_job

Останавливает выполнение задания.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_stop_job [ @job_execution_id = ] ' job_execution_id '
```

#### <a name="arguments"></a>Аргументы

[ **\@ job_execution_id =** ] job_execution_id  
Идентификационный номер останавливаемого выполнения задания. Типом job_execution_id является uniqueidentifier, значение по умолчанию — NULL.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

Нет.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_add_target_group"></a><a name="sp_add_target_group"></a>sp_add_target_group

Добавляет целевую группу.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_add_target_group [ @target_group_name = ] 'target_group_name'
     [ , [ @target_group_id = ] target_group_id OUTPUT ]
```

#### <a name="arguments"></a>Аргументы

[ **\@ target_group_name =** ] ' target_group_name '  
Имя создаваемой целевой группы. Типом target_group_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ target_group_id =** ] target_group_id вывод идентификационного номера целевой группы, назначенного заданию при успешном создании задания. target_group_id является выходной переменной типа uniqueidentifier, значение по умолчанию — NULL.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

С помощью целевых групп можно легко нацелить задание на коллекцию баз данных.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_delete_target_group"></a><a name="sp_delete_target_group"></a>sp_delete_target_group

Удаляет целевую группу.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_delete_target_group [ @target_group_name = ] 'target_group_name'
```

#### <a name="arguments"></a>Аргументы

[ **\@ target_group_name =** ] ' target_group_name '  
Имя удаляемой целевой группы. Типом target_group_name является nvarchar(128), значение по умолчанию отсутствует.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

Нет.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

### <a name="sp_add_target_group_member"></a><a name="sp_add_target_group_member"></a>sp_add_target_group_member

Добавляет базу данных или несколько баз данных в целевую группу.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_add_target_group_member [ @target_group_name = ] 'target_group_name'
         [ @membership_type = ] 'membership_type' ]
        [ , [ @target_type = ] 'target_type' ]
        [ , [ @refresh_credential_name = ] 'refresh_credential_name' ]
        [ , [ @server_name = ] 'server_name' ]
        [ , [ @database_name = ] 'database_name' ]
        [ , [ @elastic_pool_name = ] 'elastic_pool_name' ]
        [ , [ @shard_map_name = ] 'shard_map_name' ]
        [ , [ @target_id = ] 'target_id' OUTPUT ]
```

#### <a name="arguments"></a>Аргументы

[ **\@ target_group_name =** ] ' target_group_name '  
Имя целевой группы для добавления элемента. Типом target_group_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ membership_type =** ] ' membership_type '  
Указывает, добавляется или исключается ли элемент целевой группы. target_group_name имеет тип nvarchar (128) и значение по умолчанию "include". Допустимые значения для membership_type: "include" или "Exclude".

[ **\@ target_type =** ] ' target_type '  
Тип целевой базы данных или коллекции баз данных, включая все базы данных на сервере, все базы данных в эластичном пуле, все базы данных в сопоставлении сегментов или отдельную базу данных. Типом target_type является nvarchar(128), значение по умолчанию отсутствует. Допустимые значения для target_type: "SqlServer", "Склеластикпул", "SqlDatabase" или "Склшардмап".

[ **\@ refresh_credential_name =** ] ' refresh_credential_name '  
Имя учетных данных области базы данных. Типом refresh_credential_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ server_name =** ] ' server_name '  
Имя сервера, который необходимо добавить в указанную целевую группу. server_name следует указывать, если target_type имеет "SqlServer". Типом server_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ database_name =** ] ' database_name '  
Имя базы данных, добавляемой в указанную целевую группу. database_name следует указывать, если target_type имеет "SqlDatabase". Типом database_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ elastic_pool_name =** ] ' elastic_pool_name '  
Имя эластичного пула, добавляемого в указанную целевую группу. elastic_pool_name следует указывать, если target_type имеет "Склеластикпул". Типом elastic_pool_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ shard_map_name =** ] ' shard_map_name '  
Имя сопоставления сегментов, добавляемого в указанную целевую группу. elastic_pool_name следует указывать, если target_type имеет "Склшардмап". Типом shard_map_name является nvarchar(128), значение по умолчанию отсутствует.

[ **\@ target_id =** ] target_group_id выходные данные  
Целевой идентификационный номер, который назначается элементу целевой группы при его успешном создании и добавлении в целевую группу. target_id является выходной переменой типа uniqueidentifier, значение по умолчанию — NULL.
Значения кода возврата: 0 (успех) или 1 (сбой).

#### <a name="remarks"></a>Remarks

Задание выполняется во всех отдельных базах данных на сервере или в эластичном пуле во время выполнения, если в целевую группу входит сервер или эластичный пул.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

#### <a name="examples"></a>Примеры

Приведенный ниже пример добавляет все базы данных на серверах London и NewYork в группу Servers Maintaining Customer Information. Необходимо подключиться к базе данных заданий, указанной при создании агента заданий. В данном случае это база данных ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent
USE ElasticJobs;
GO

-- Add a target group containing server(s)
EXEC jobs.sp_add_target_group @target_group_name =  N'Servers Maintaining Customer Information';
GO

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_type = N'SqlServer',
@refresh_credential_name=N'refresh_credential', --credential required to refresh the databases in server
@server_name=N'London.database.windows.net';
GO

-- Add a server target member
EXEC jobs.sp_add_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_type = N'SqlServer',
@refresh_credential_name=N'refresh_credential', --credential required to refresh the databases in server
@server_name=N'NewYork.database.windows.net';
GO

--View the recently added members to the target group
SELECT * FROM [jobs].target_group_members WHERE target_group_name= N'Servers Maintaining Customer Information';
GO
```

### <a name="sp_delete_target_group_member"></a><a name="sp_delete_target_group_member"></a>sp_delete_target_group_member

Удаляет элемент из целевой группы.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_delete_target_group_member [ @target_group_name = ] 'target_group_name'
   [ , [ @target_id = ] 'target_id']
```

#### <a name="arguments"></a>Аргументы

[ @target_group_name =] "target_group_name"  
Имя целевой группы, из которой удаляется элемент. Типом target_group_name является nvarchar(128), значение по умолчанию отсутствует.

[ @target_id = ] target_id  
 Целевой идентификационный номер, который назначен удаляемому элементу целевой группы. Типом target_id является uniqueidentifier, значение по умолчанию — NULL.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

С помощью целевых групп можно легко нацелить задание на коллекцию баз данных.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

#### <a name="examples"></a>Примеры

Приведенный ниже пример удаляет сервер London из группы Servers Maintaining Customer Information. Необходимо подключиться к базе данных заданий, указанной при создании агента заданий. В данном случае это база данных ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent
USE ElasticJobs ;
GO

-- Retrieve the target_id for a target_group_members
declare @tid uniqueidentifier
SELECT @tid = target_id FROM [jobs].target_group_members WHERE target_group_name = 'Servers Maintaining Customer Information' and server_name = 'London.database.windows.net';

-- Remove a target group member of type server
EXEC jobs.sp_delete_target_group_member
@target_group_name = N'Servers Maintaining Customer Information',
@target_id = @tid;
GO
```

### <a name="sp_purge_jobhistory"></a><a name="sp_purge_jobhistory"></a>sp_purge_jobhistory, хранимая процедура

Удаляет записи журнала для задания.

#### <a name="syntax"></a>Синтаксис

```syntaxsql
[jobs].sp_purge_jobhistory [ @job_name = ] 'job_name'
      [ , [ @job_id = ] job_id ]
      [ , [ @oldest_date = ] oldest_date []
```

#### <a name="arguments"></a>Аргументы

[ **\@ job_name =** ] ' job_name '  
Имя задания, для которого удаляются записи журнала. Типом job_name является nvarchar(128), значение по умолчанию — NULL. Необходимо указать job_id или job_name, но не одновременно.

[ **\@ job_id =** ] job_id  
 Идентификатор задания, для которого удаляются записи. Типом job_id является uniqueidentifier, значение по умолчанию — NULL. Необходимо указать job_id или job_name, но не одновременно.

[ **\@ oldest_date =** ] oldest_date  
 Самая ранняя запись журнала, которую необходимо сохранить. Типом oldest_date является DATETIME2, значение по умолчанию — NULL. Если указан аргумент oldest_date, то sp_purge_jobhistory удаляет только записи, которые старше указанного значения.

#### <a name="return-code-values"></a>Значения кода возврата

0 (успешное завершение) или 1 (неуспешное завершение)

#### <a name="remarks"></a>Remarks

С помощью целевых групп можно легко нацелить задание на коллекцию баз данных.

#### <a name="permissions"></a>Разрешения

По умолчанию участники фиксированной роли сервера sysadmin могут выполнять эту хранимую процедуру. Чтобы ограничить возможности пользователя исключительно отслеживанием заданий, его можно добавить в приведенную ниже роль базы данных агента заданий, указанной при создании этого агента.

- jobs_reader

Описание разрешений этих ролей приведено в разделе "Разрешения" этого документа. Только участники роли sysadmin могут использовать эту хранимую процедуру для редактирования атрибутов заданий, которыми владеют другие пользователи.

#### <a name="examples"></a>Примеры

Приведенный ниже пример добавляет все базы данных на серверах London и NewYork в группу Servers Maintaining Customer Information. Необходимо подключиться к базе данных заданий, указанной при создании агента заданий. В данном случае это база данных ElasticJobs.

```sql
--Connect to the jobs database specified when creating the job agent

EXEC sp_delete_target_group_member
    @target_group_name = N'Servers Maintaining Customer Information',  
    @server_name = N'London.database.windows.net';  
GO
```

## <a name="job-views"></a>Представления заданий

Ниже приведены представления, доступные в [базе данных заданий](job-automation-overview.md#elastic-job-database).

|Представление  |Описание  |
|---------|---------|
|[job_executions](#job_executions-view)     |  Отображает журнал выполнения задания.      |
|[Вакансии](#jobs-view)     |   Отображает все задания.      |
|[job_versions](#job_versions-view)     |   Отображает все версии задания.      |
|[JobSteps](#jobsteps-view)     |     Отображает все шаги в текущей версии каждого задания.    |
|[jobstep_versions](#jobstep_versions-view)     |     Отображает все шаги во всех версиях каждого задания.    |
|[target_groups](#target_groups-view)     |      Отображает все целевые группы.   |
|[target_group_members](#target_group_members-view)     |   Отображает все элементы всех целевых групп.      |

### <a name="job_executions-view"></a><a name="job_executions-view"></a>представление job_executions

[задания]. [job_executions]

Отображает журнал выполнения задания.

|Имя столбца | Тип данных | Описание |
|---------|---------|---------|
|**job_execution_id** | UNIQUEIDENTIFIER | Уникальный идентификатор экземпляра выполнения задания.
|**job_name** | NVARCHAR(128) | Имя задания.
|**job_id** | UNIQUEIDENTIFIER | Уникальный идентификатор задания.
|**job_version** | INT | Версия задания (автоматически обновляется каждый раз при изменении задания).
|**step_id** |INT | Уникальный для данного задания идентификатор этапа. Значение NULL означает, что выполняется родительское задание.
|**is_active** | bit | Указывает, является ли задание активным или неактивным. Значение 1 указывает активные задания, а значение 0 — неактивные.
|**файлу** | nvarchar(50) | Значение, указывающее состояние задания: "создано", "выполняется", "сбой", "успешно", "пропущено", "Сукцеедедвисскиппед"|
|**create_time**| datetime2(7) | Дата и время создания задания.
|**start_time** | datetime2(7) | Дата и время начала выполнения задания. Имеет значение NULL, если задание еще не выполнено.
|**end_time** | datetime2(7) | Дата и время завершения выполнения задания. Имеет значение NULL, если задание еще не выполнено или его выполнение не завершено.
|**current_attempts** | INT | Число попыток выполнения шага. Для родительского задания будет отображено значение 0, а число выполнений дочернего задания будет равно 1 или больше, в зависимости от политики выполнения.
|**current_attempt_start_time** | datetime2(7) | Дата и время начала выполнения задания. Значение NULL означает, что выполняется родительское задание.
|**last_message** | nvarchar(max) | Запись в журнале о задании или этапе.
|**target_type** | NVARCHAR(128) | Тип целевой базы данных или коллекции баз данных, включая все базы данных на сервере, все базы данных в эластичном пуле или отдельную базу данных. Допустимые значения для target_type: "SqlServer", "Склеластикпул" или "SqlDatabase". Значение NULL означает, что выполняется родительское задание.
|**target_id** | UNIQUEIDENTIFIER | Уникальный идентификатор элемента целевой группы.  Значение NULL означает, что выполняется родительское задание.
|**target_group_name** | NVARCHAR(128) | Имя целевой группы. Значение NULL означает, что выполняется родительское задание.
|**target_server_name** | nvarchar(256)  | Имя сервера, содержащегося в Целевой группе. Указывается, только если target_type имеет значение "SqlServer". Значение NULL означает, что выполняется родительское задание.
|**target_database_name** | NVARCHAR(128) | Имя базы данных в целевой группе. Указывается, только если target_type имеет "SqlDatabase". Значение NULL означает, что выполняется родительское задание.

### <a name="jobs-view"></a>Представление jobs

[jobs].[jobs]

Отображает все задания.

|Имя столбца | Тип данных |Описание|
|------|------|-------|
|**job_name** | NVARCHAR(128) | Имя задания.|
|**job_id**| UNIQUEIDENTIFIER |Уникальный идентификатор задания.|
|**job_version** |INT |Версия задания (автоматически обновляется каждый раз при изменении задания).|
|**description** |nvarchar(512)| Описание задания. Бит включено: указывает, включено ли задание или отключено. Значение 1 указывает включенные задания, а значение 0 — отключенные задания.|
|**schedule_interval_type**|nvarchar(50) |Значение, указывающее дату и время выполнения задания: Once, Minutes, Hours, Days, Weeks, Months.
|**schedule_interval_count**|INT|Число периодов schedule_interval_type между выполнениями задания.|
|**schedule_start_time**|datetime2(7)|Дата и время последнего начала выполнения задания.|
|**schedule_end_time**|datetime2(7)|Дата и время последнего успешного выполнения задания.|

### <a name="job_versions-view"></a><a name="job_versions-view"></a>Представление job_versions

[jobs].[job_versions]

Отображает все версии задания.

|Имя столбца|Тип данных|Описание|
|------|------|-------|
|**job_name**|NVARCHAR(128)|Имя задания.|
|**job_id**|UNIQUEIDENTIFIER|Уникальный идентификатор задания.|
|**job_version**|INT|Версия задания (автоматически обновляется каждый раз при изменении задания).|

### <a name="jobsteps-view"></a>Представление jobsteps

[jobs].[jobsteps]

Отображает все шаги в текущей версии каждого задания.

|Имя столбца|Тип данных|Описание|
|------|------|-------|
|**job_name**|NVARCHAR(128)|Имя задания.|
|**job_id**|UNIQUEIDENTIFIER|Уникальный идентификатор задания.|
|**job_version**|INT|Версия задания (автоматически обновляется каждый раз при изменении задания).|
|**step_id**|INT|Уникальный для данного задания идентификатор этапа.|
|**step_name**|NVARCHAR(128)|Уникальное (в пределах задания) имя шага.|
|**command_type**|nvarchar(50)|Тип команды, выполняемой на шаге задания. Для v1 значение должно быть равно, а по умолчанию — TSql.|
|**command_source**|nvarchar(50)|Расположение команды. Для версии v1 параметр "inline" является значением по умолчанию и принимает только значение.|
|**command**|nvarchar(max)|Команды, выполняемые эластичными заданиями в соответствии с command_type.|
|**credential_name**|NVARCHAR(128)|Имя учетных данных базы данных, используемых для выполнения задания.|
|**target_group_name**|NVARCHAR(128)|Имя целевой группы.|
|**target_group_id**|UNIQUEIDENTIFIER|Уникальный идентификатор целевой группы.|
|**initial_retry_interval_seconds**|INT|Задержка перед первой повторной попыткой. Значение по умолчанию — 1.|
|**maximum_retry_interval_seconds**|INT|Максимальная задержка между повторными попытками. Если задержка между повторными попытками превышает это значение, она ограничивается указанным значением. Значение по умолчанию — 120.|
|**retry_interval_backoff_multiplier**|real|Коэффициент, на который умножается значение задержки перед повторной попыткой в случае нескольких попыток выполнения шага задания. Значение по умолчанию — 2,0.|
|**retry_attempts**|INT|Число повторных попыток, предпринимаемых при завершении данного шага с ошибкой. Значение по умолчанию — 10. Оно означает отсутствие повторных попыток.|
|**step_timeout_seconds**|INT|Время ожидания в минутах между попытками повтора. Значение по умолчанию — 0. Оно означает интервал в 0 минут.|
|**output_type**|nvarchar(11)|Расположение команды. Для текущей предварительной версии допускается только значение по умолчанию — Inline.|
|**output_credential_name**|NVARCHAR(128)|Имя учетных данных для подключения к целевому серверу для хранения результирующего набора.|
|**output_subscription_id**|UNIQUEIDENTIFIER|Уникальный идентификатор подписки целевого сервера или базы данных для хранения результирующего набора после выполнения запроса.|
|**output_resource_group_name**|NVARCHAR(128)|Имя группы ресурсов, в которой размещен целевой сервер.|
|**output_server_name**|nvarchar(256)|Имя целевого сервера для хранения результирующего набора.|
|**output_database_name**|NVARCHAR(128)|Имя целевой базы данных для хранения результирующего набора.|
|**output_schema_name**|nvarchar(max)|Имя целевой схемы. Если не указано, используется значение по умолчанию — dbo.|
|**output_table_name**|nvarchar(max)|Имя таблицы для хранения результирующего набора запроса. Таблица будет создана автоматически на основе схемы набора результатов, если она еще не существует. Схема должна совпадать со схемой результирующего набора.|
|**max_parallelism**|INT|Максимальное число баз данных на эластичный пул, которое может одновременно обрабатывать шаг задания. Значение по умолчанию — NULL. Оно означает отсутствие ограничения. |

### <a name="jobstep_versions-view"></a><a name="jobstep_versions-view"></a>Представление jobstep_versions

[jobs].[jobstep_versions]

Отображает все шаги во всех версиях каждого задания. Схема идентична схеме [jobsteps](#jobsteps-view).

### <a name="target_groups-view"></a><a name="target_groups-view"></a>Представление target_groups

[jobs].[target_groups]

Отображает все целевые группы.

|Имя столбца|Тип данных|Описание|
|-----|-----|-----|
|**target_group_name**|NVARCHAR(128)|Имя целевой группы, содержащей коллекцию баз данных.
|**target_group_id**|UNIQUEIDENTIFIER|Уникальный идентификатор целевой группы.

### <a name="target_group_members-view"></a><a name="target_group_members-view"></a>представление target_group_members

[задания]. [target_group_members]

Отображает все элементы всех целевых групп.

|Имя столбца|Тип данных|Описание|
|-----|-----|-----|
|**target_group_name**|nvarchar(128)|Имя целевой группы, содержащей коллекцию баз данных. |
|**target_group_id**|UNIQUEIDENTIFIER|Уникальный идентификатор целевой группы.|
|**membership_type**|INT|Указывает, добавляется элемент в целевую группу или исключается из нее. Допустимые значения для target_group_name: "include" или "Exclude".|
|**target_type**|NVARCHAR(128)|Тип целевой базы данных или коллекции баз данных, включая все базы данных на сервере, все базы данных в эластичном пуле или отдельную базу данных. Допустимые значения для target_type: "SqlServer", "Склеластикпул", "SqlDatabase" или "Склшардмап".|
|**target_id**|UNIQUEIDENTIFIER|Уникальный идентификатор элемента целевой группы.|
|**refresh_credential_name**|NVARCHAR(128)|Имя учетных данных базы данных, используемых для подключения к элементу целевой группы.|
|**subscription_id**|UNIQUEIDENTIFIER|Уникальный идентификатор подписки.|
|**resource_group_name**|NVARCHAR(128)|Имя группы ресурсов, в которой находится элемент целевой группы.|
|**server_name**|NVARCHAR(128)|Имя сервера, содержащегося в Целевой группе. Указывается, только если target_type имеет значение "SqlServer". |
|**database_name**|NVARCHAR(128)|Имя базы данных в целевой группе. Указывается, только если target_type имеет "SqlDatabase".|
|**elastic_pool_name**|NVARCHAR(128)|Имя эластичного пула в целевой группе. Указывается, только если target_type имеет "Склеластикпул".|
|**shard_map_name**|NVARCHAR(128)|Имя карт сегментов, содержащихся в Целевой группе. Указывается, только если target_type имеет "Склшардмап".|

## <a name="resources"></a>Ресурсы

- ![Значок ссылки на раздел](/sql/database-engine/configure-windows/media/topic-link.gif "Значок ссылки на раздел") [Синтаксические обозначения в Transact-SQL](/sql/t-sql/language-elements/transact-sql-syntax-conventions-transact-sql)  

## <a name="next-steps"></a>Дальнейшие действия

- [Создание заданий обработки эластичных баз данных и управление ими с помощью PowerShell](elastic-jobs-powershell-create.md)
- [Авторизация и разрешения](/dotnet/framework/data/adonet/sql/authorization-and-permissions-in-sql-server)
