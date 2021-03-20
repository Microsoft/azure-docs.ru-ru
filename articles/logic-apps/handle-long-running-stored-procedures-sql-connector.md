---
title: Обработку длительно выполняемых хранимых процедур в соединителе SQL
description: Как работать с хранимыми процедурами, для которых истекло время ожидания при использовании соединителя SQL, в Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: camerost, logicappspm
ms.topic: conceptual
ms.date: 10/27/2020
ms.openlocfilehash: f5b04c563dc81497f591788dc4890d379c0f898f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93103112"
---
# <a name="handle-stored-procedure-timeouts-in-the-sql-connector-for-azure-logic-apps"></a>Время ожидания обработки хранимых процедур в соединителе SQL для Azure Logic Apps

Если приложение логики работает с результирующими наборами так, чтобы [соединитель SQL](../connectors/connectors-create-api-sqlazure.md) не возвращал все результаты одновременно, или если требуется больший контроль над размером и структурой для результирующих наборов, можно создать [хранимую процедуру](/sql/relational-databases/stored-procedures/stored-procedures-database-engine) , которая упорядочивает результаты нужным образом. Соединитель SQL предоставляет множество серверных функций, к которым можно получить доступ с помощью [Azure Logic Apps](../logic-apps/logic-apps-overview.md) , чтобы упростить автоматизацию бизнес-задач, работающих с таблицами базы данных SQL.

Например, при получении или вставке нескольких строк приложение логики может выполнить итерацию по этим строкам, используя [цикл **until**](../logic-apps/logic-apps-control-flow-loops.md#until-loop) в пределах этих [ограничений](../logic-apps/logic-apps-limits-and-config.md). Однако если приложение логики должно работать с тысячами или миллионами строк, необходимо сократить затраты, возникающие в результате вызовов к базе данных. Дополнительные сведения см. [в разделе Работа с массовыми данными с помощью СОЕДИНИТЕЛЯ SQL](../connectors/connectors-create-api-sqlazure.md#handle-bulk-data).

<a name="timeout-limit"></a>

## <a name="timeout-limit-on-stored-procedure-execution"></a>Предельное время ожидания выполнения хранимой процедуры

В соединителе SQL есть ограничение времени ожидания хранимой процедуры, которое [меньше 2 минут](/connectors/sql/#known-issues-and-limitations). Некоторые хранимые процедуры могут занять больше времени, чем это ограничение, что приведет к `504 Timeout` ошибке. Иногда эти долгосрочные процессы кодируются в качестве хранимых процедур явным образом для этой цели. Из-за предельного времени ожидания вызов этих процедур из Azure Logic Apps может привести к возникновению проблем. Хотя соединитель SQL изначально не поддерживает асинхронный режим, эту проблему можно обойти и имитировать этот режим с помощью триггера завершения SQL, собственного сквозного запроса SQL, таблицы состояний и заданий на стороне сервера. Для этой задачи можно использовать [агент заданий эластичной службы Azure](../azure-sql/database/elastic-jobs-overview.md) для [базы данных SQL Azure](../azure-sql/database/sql-database-paas-overview.md). Для [SQL Server локальных](/sql/sql-server/sql-server-technical-documentation) и [управляемый экземпляр SQL Azure](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)можно использовать [Агент SQL Server](/sql/ssms/agent/sql-server-agent).

Например, предположим, что имеется следующая длительная хранимая процедура, которая занимает больше времени, чем предельное время ожидания для завершения выполнения. Если запустить эту хранимую процедуру из приложения логики с помощью соединителя SQL, `HTTP 504 Gateway Timeout` в качестве результата появится сообщение об ошибке.

```sql
CREATE PROCEDURE [dbo].[WaitForIt]
   @delay char(8) = '00:03:00'
AS
BEGIN
   SET NOCOUNT ON;
   WAITFOR DELAY @delay
END
```

Вместо непосредственного вызова хранимой процедуры можно асинхронно выполнять процедуру в фоновом режиме с помощью *агента заданий*. Входные и выходные данные можно хранить в таблице состояний, с помощью которой можно взаимодействовать через приложение логики. Если входные и выходные данные не нужны, или если вы уже записываете результаты в таблицу в хранимой процедуре, этот подход можно упростить.

> [!IMPORTANT]
> Убедитесь, что хранимая процедура и все задания являются *идемпотентными*. Это означает, что они могут выполняться несколько раз, не влияя на результаты. Если асинхронная обработка завершается сбоем или истекает время ожидания, агент заданий может повторить шаг и, таким образом, хранимую процедуру несколько раз. Чтобы избежать дублирования выходных данных, перед созданием объектов ознакомьтесь с [рекомендациями и подходами](../azure-sql/database/elastic-jobs-overview.md#idempotent-scripts).

В следующем разделе описывается, как можно использовать агент заданий эластичной службы Azure для базы данных SQL Azure. Для SQL Server и Управляемый экземпляр SQL Azure можно использовать агент SQL Server. Некоторые сведения об управлении будут отличаться, но фундаментальные шаги остаются теми же, что и настройка агента заданий для базы данных SQL Azure.

<a name="azure-sql-database"></a>

## <a name="job-agent-for-azure-sql-database"></a>Агент заданий для базы данных SQL Azure

Чтобы создать задание, которое может выполнять хранимую процедуру для [базы данных SQL Azure](../azure-sql/database/sql-database-paas-overview.md), используйте [агент заданий обработки эластичных БД Azure](../azure-sql/database/elastic-jobs-overview.md). Создайте агент заданий в портал Azure. Этот подход приведет к добавлению нескольких хранимых процедур в базу данных, используемую агентом, также называемую *базой данных агента*. Затем можно создать задание, которое запускает хранимую процедуру в целевой базе данных и захватывает выходные данные по завершении.

Прежде чем можно будет создать задание, необходимо настроить разрешения, группы и целевые объекты, как описано в [полной документации по агенту заданий эластичных](../azure-sql/database/elastic-jobs-overview.md)баз данных Azure. Кроме того, необходимо создать вспомогательную таблицу в целевой базе данных, как описано в следующих разделах.

<a name="create-state-table"></a>

### <a name="create-state-table-for-registering-parameters-and-storing-inputs"></a>Создание таблицы состояний для регистрации параметров и хранения входных данных

Задания агента SQL Server не принимают входные параметры. Вместо этого в целевой базе данных создайте таблицу состояний, в которой вы регистрируете параметры и сохраняете входные данные, используемые для вызова хранимых процедур. Все шаги задания агента выполняются с целевой базой данных, но хранимые процедуры задания выполняются в базе данных агента. 

Чтобы создать таблицу состояний, используйте следующую схему:

```sql
CREATE TABLE [dbo].[LongRunningState](
   [jobid] [uniqueidentifier] NOT NULL,
   [rowversion] [timestamp] NULL,
   [parameters] [nvarchar](max) NULL,
   [start] [datetimeoffset](7) NULL,
   [complete] [datetimeoffset](7) NULL,
   [code] [int] NULL,
   [result] [nvarchar](max) NULL,
   CONSTRAINT [PK_LongRunningState] PRIMARY KEY CLUSTERED
      (   [jobid] ASC
      )WITH (STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF) ON [PRIMARY]
      ) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
```

Вот как будет выглядеть полученная таблица в [SQL Server Management Studio (SMSS)](/sql/ssms/download-sql-server-management-studio-ssms):

![Снимок экрана, показывающий созданную таблицу состояний, в которой хранятся входные данные для хранимой процедуры.](media/handle-long-running-stored-procedures-sql-connector/state-table-input-parameters.png)

Чтобы обеспечить хорошую производительность и убедиться, что задание агента может найти связанную запись, в таблице используется идентификатор выполнения задания () в `jobid` качестве первичного ключа. При необходимости можно также добавить отдельные столбцы для входных параметров. Ранее Описанная схема может более широко обрабатывала несколько параметров, но ограничена размером, вычисленным `NVARCHAR(MAX)` .

<a name="create-top-level-job"></a>

### <a name="create-a-top-level-job-to-run-the-stored-procedure"></a>Создание задания верхнего уровня для выполнения хранимой процедуры

Чтобы выполнить долгосрочную хранимую процедуру, создайте этот агент заданий верхнего уровня в базе данных агента:

```sql
EXEC jobs.sp_add_job 
   @job_name='LongRunningJob',
   @description='Execute Long-Running Stored Proc',
   @enabled = 1
```

Теперь добавьте в задание шаги, которые параметризуются, запускают и завершают хранимую процедуру. По умолчанию шаг задания истекает через 12 часов. Если для выполнения хранимой процедуры требуется больше времени или если вы хотите, чтобы процедура использовалась для истечения времени ожидания, можно изменить `step_timeout_seconds` параметр на другое значение, указанное в секундах. По умолчанию на шаге имеется 10 встроенных повторных попыток с временем ожидания перехода между повторными попытками, которые можно использовать в качестве своего преимущества.

Ниже приведены шаги для добавления.

1. Дождитесь появления параметров в `LongRunningState` таблице.

   Первый шаг ожидает добавления параметров в `LongRunningState` таблицу, что происходит вскоре после запуска задания. Если идентификатор выполнения задания ( `jobid` ) не добавляется в `LongRunningState` таблицу, шаг просто завершается ошибкой, а время ожидания по умолчанию или истечения задержки истекает:

   ```sql
   EXEC jobs.sp_add_jobstep
      @job_name='LongRunningJob',
      @step_name= 'Parameterize WaitForIt',
      @step_timeout_seconds = 30,
      @command= N'
         IF NOT EXISTS(SELECT [jobid] FROM [dbo].[LongRunningState]
            WHERE jobid = $(job_execution_id))
            THROW 50400, ''Failed to locate call parameters (Step1)'', 1',
      @credential_name='JobRun',
      @target_group_name='DatabaseGroupLongRunning'
   ```

1. Запросите параметры из таблицы состояний и передайте их в хранимую процедуру. Этот шаг также выполняет процедуру в фоновом режиме. 

   Если хранимая процедура не требует параметров, просто вызовите хранимую процедуру напрямую. В противном случае для передачи `@timespan` параметра используйте `@callparams` , который можно также расширить для передачи дополнительных параметров.

   ```sql
   EXEC jobs.sp_add_jobstep
      @job_name='LongRunningJob',
      @step_name='Execute WaitForIt',
      @command=N'
         DECLARE @timespan char(8)
         DECLARE @callparams NVARCHAR(MAX)
         SELECT @callparams = [parameters] FROM [dbo].[LongRunningState]
            WHERE jobid = $(job_execution_id))
         SET @timespan = @callparams
         EXECUTE [dbo].[WaitForIt] @delay = @timespan', 
      @credential_name='JobRun',
      @target_group_name='DatabaseGroupLongRunning'
   ```

1. Завершите задание и запишите результаты.

   ```sql
   EXEC jobs.sp_add_jobstep
      @job_name='LongRunningJob',
      @step_name='Complete WaitForIt',
      @command=N'
         UPDATE [dbo].[LongRunningState]
            SET [complete] = GETUTCDATE(),
               [code] = 200,
               [result] = ''Success''
            WHERE jobid = $(job_execution_id)',
      @credential_name='JobRun',
      @target_group_name='DatabaseGroupLongRunning'
   ```

<a name="start-job-pass-parameters"></a>

### <a name="start-the-job-and-pass-the-parameters"></a>Запустите задание и передайте параметры.

Чтобы запустить задание, используйте транзитный собственный запрос с [действием **выполнить запрос SQL**](/connectors/sql/#execute-a-sql-query-(v2)) и немедленно отправить параметры задания в таблицу состояний. Чтобы предоставить входные данные `jobid` атрибуту в целевой таблице, Logic Apps добавляет цикл **for each** , который выполняет итерацию по выходным данным таблицы из предыдущего действия. Для каждого идентификатора выполнения задания выполните действие **Вставить строку** , которое использует выходные данные динамического типа данных, `ResultSets JobExecutionId` , чтобы добавить параметры задания для распаковки и передачи в целевую хранимую процедуру.

![Снимок экрана, на котором показаны действия, которые необходимо выполнить для запуска задания и передачи параметров в хранимую процедуру.](media/handle-long-running-stored-procedures-sql-connector/start-job-actions.png)

После завершения задания оно обновляет `LongRunningState` таблицу, чтобы можно было легко активировать результат с помощью [триггера **при изменении элемента**](/connectors/sql/#when-an-item-is-modified-(v2)). Если выходные данные не нужны или если у вас уже есть триггер, отслеживающий выходную таблицу, эту часть можно пропустить.

![Снимок экрана, на котором показан триггер SQL для изменения элемента.](media/handle-long-running-stored-procedures-sql-connector/trigger-on-results-after-job-completes.png)

<a name="sql-on-premises-or-managed-instance"></a>

## <a name="job-agent-for-sql-server-or-azure-sql-managed-instance"></a>Агент заданий для SQL Server или Azure SQL Управляемый экземпляр

Для того же сценария можно использовать [Агент SQL Server](/sql/ssms/agent/sql-server-agent) для [SQL Server локальной](/sql/sql-server/sql-server-technical-documentation) среды и [управляемый экземпляр SQL Azure](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md). Хотя некоторые сведения об управлении различны, фундаментальные шаги остаются теми же, что и настройка агента заданий для базы данных SQL Azure.

## <a name="next-steps"></a>Дальнейшие действия

[Подключение к SQL Server, базе данных SQL Azure или Управляемый экземпляр SQL Azure](../connectors/connectors-create-api-sqlazure.md)

