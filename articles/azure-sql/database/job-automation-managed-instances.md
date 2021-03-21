---
title: Автоматизация заданий с помощью заданий агента SQL в Azure SQL Управляемый экземпляр
description: Параметры автоматизации для выполнения скриптов Transact-SQL (T-SQL) в Azure SQL Управляемый экземпляр
services: sql-database
ms.service: sql-db-mi
ms.subservice: features
ms.custom: sqldbrb=1
dev_langs:
- TSQL
ms.topic: conceptual
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: ''
ms.date: 02/01/2021
ms.openlocfilehash: 3be01c304a40317e0d21baf6789ef1376cd89b80
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104608084"
---
# <a name="automate-management-tasks-using-sql-agent-jobs-in-azure-sql-managed-instance"></a>Автоматизация задач управления с помощью заданий агента SQL в Управляемый экземпляр Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

С помощью [Агент SQL Server](/sql/ssms/agent/sql-server-agent) в SQL Server и [SQL управляемый экземпляр](../../azure-sql/managed-instance/sql-managed-instance-paas-overview.md)можно создавать и планировать задания, которые могут периодически выполняться в одной или нескольких базах данных для выполнения запросов Transact-SQL (T-SQL) и выполнения задач обслуживания. В этой статье представлен SQL Agent для SQL Управляемый экземпляр.

> [!Note]
> Агент SQL Server недоступен в базе данных SQL Azure или Azure синапсе Analytics. Вместо этого мы рекомендуем [автоматизировать задания с помощью заданий обработки эластичных БД](job-automation-overview.md).

### <a name="sql-agent-job-limitations-in-azure-sql-managed-instance"></a>Ограничения заданий агента SQL в управляемом экземпляре SQL Azure

Стоит отметить различия между агентом SQL, доступным в SQL Server и в составе SQL Управляемый экземпляр. Дополнительные сведения о различиях в поддерживаемых функциях между SQL Server и SQL Управляемый экземпляр см. в статье [отличия T-SQL в Azure sql управляемый экземпляр от SQL Server](../../azure-sql/managed-instance/transact-sql-tsql-differences-sql-server.md#sql-server-agent). 

Некоторые функции агента SQL Server, доступные в SQL Server, не поддерживаются в Управляемый экземпляр SQL:

- Параметры агента SQL Server доступны только для чтения. 
    - Системная хранимая процедура `sp_set_agent_properties` не поддерживается в SQL управляемый экземпляр.
- Включение и отключение агента SQL в настоящее время не поддерживается в Управляемый экземпляр SQL. Агент SQL работает всегда.
- Уведомления поддерживаются частично:
  - Пейджер не поддерживается.
  - NetSend не поддерживается.
  - Оповещения не поддерживаются.
- Прокси-серверы не поддерживаются.
- Eventlog не поддерживается.
- Триггер расписания задания на основе неактивного ЦП не поддерживается.

## <a name="when-to-use-sql-agent-jobs"></a>Когда следует использовать задания агента SQL Server 

Существует несколько сценариев, в которых можно использовать задания агента SQL:

- Автоматизация задач управления, а также их добавление в расписание для запуска каждый рабочий день, в нерабочие часы и т. д.
  - Развертывание изменений схемы, учетных данных, сбора данных о производительности или телеметрии клиента.
  - Обновление эталонных данных (например, сведений о продукте, которые являются общими для всех баз данных), загрузка данных из хранилища BLOB-объектов Azure.
  - Общие задачи обслуживания, в том числе DBCC CHECKDB, чтобы обеспечить целостность данных или обслуживание индекса для повышения производительности запросов. Настройка заданий для выполнения в коллекции баз данных на постоянной основе, например в часы наименьшей нагрузки.
  - Собирайте результаты запросов из набора баз данных в центральную таблицу на постоянной основе. Запросы производительности могут выполняться непрерывно и вызывать дополнительные задачи.
- Сбор данных для отчетов
  - Сбор данных из коллекции баз данных в одну целевую таблицу.
  - Выполняйте запросы обработки данных с повышенным временем выполнения для большого набора баз данных, например коллекции телеметрии клиентов. Результаты собираются в одну целевую таблицу для дальнейшего анализа.
- Перемещения данных
  - Создание заданий, которые реплицируют изменения, внесенные в базах данных, в другие базы данных, или собирают обновления, выполняемые в удаленных базах данных, и применяют изменения в базе данных.
  - Создание заданий, загружающих данные в базы данных и обратно, с помощью SQL Server Integration Services (SSIS).

## <a name="sql-agent-jobs-in-azure-sql-managed-instance"></a>Задания агента SQL в управляемом экземпляре SQL Azure

Задания агента SQL Server выполняются службой агента SQL Server, которая по-своему используется для автоматизации задач в SQL Server и SQL Управляемый экземпляр. 

Задания агента SQL — указанный ряд сценариев T-SQL в базе данных. Используйте задания, чтобы определить задачу администрирования, которую можно запустить один или несколько раз и отслеживать ее успешное выполнение или сбой. 

Задание может выполняться на одном локальном или на нескольких удаленных серверах. Задания агента SQL являются внутренним компонентом ядро СУБД, который выполняется в службе SQL Управляемый экземпляр.

В заданиях агента SQL существует несколько ключевых концепций:

- Набор **шагов задания**, состоящий из одного или нескольких шагов, которые должны выполняться в рамках задания. Для каждого шага задания вы можете определить стратегию повторов и действие, которое должно произойти, если шаг задания завершился успешно или с ошибкой.
- **Расписания** определяют, когда должно выполняться задание.
- **Уведомления** позволяют определять правила, которые будут использоваться для уведомления операторов через электронную почту по завершении задания.

### <a name="sql-agent-job-steps"></a>Шаги задания агента SQL Server

Шаги задания агента SQL — это последовательности действий, которые должны выполняться агентом SQL. После каждого шага предусмотрен следующий шаг, который должен выполняться, если предыдущий шаг выполнен успешно или неуспешно, а также если было предпринято определенное количество попыток повторного выполнения в случае сбоя.

Агент SQL позволяет создавать различные типы шагов заданий, например шаги задания Transact-SQL, выполняющие один пакет Transact-SQL для базы данных, или шаги командной строки или PowerShell, которые могут выполнять пользовательский скрипт операционной системы, [шаги задания служб SSIS](../../data-factory/how-to-invoke-ssis-package-managed-instance-agent.md) , позволяющие загружать данные с помощью среды выполнения служб SSIS, или шаги [репликации](../managed-instance/replication-transactional-overview.md) , которые могут публиковать изменения из базы данных в других базах данных.

> [!Note]
> Дополнительные сведения об использовании Integration Runtime Azure SSIS с SSISDB, размещенной в Управляемый экземпляр SQL Azure, см. [в статье использование управляемый экземпляр Azure SQL с SQL Server Integration Services (SSIS) в фабрике данных Azure](../../data-factory/how-to-use-sql-managed-instance-with-ir.md).

[Репликация транзакций](../managed-instance/replication-transactional-overview.md) может реплицировать изменения из таблиц в другие базы данных в управляемый экземпляр Azure SQL, в базе данных SQL azure или SQL Server. Дополнительные сведения см. [в статье Настройка репликации в Azure SQL управляемый экземпляр](../../azure-sql/managed-instance/replication-between-two-instances-configure-tutorial.md). 

Другие типы шагов заданий в настоящее время не поддерживаются в Управляемый экземпляр SQL, включая:

- Шаг задания репликации слиянием не поддерживается.
- Читатель очереди пока не поддерживается.
- Analysis Services не поддерживаются.
 
### <a name="sql-agent-job-schedules"></a>Расписания заданий агента SQL Server

Расписание определяет время выполнения задания. Несколько заданий могут выполняться по тому же расписанию, а несколько расписаний могут применяться для одного задания.

Расписание может определить следующие условия для времени выполнения задания:

- при каждом запуске агента SQL Server; Задание активируется после каждой отработки отказа.
- Один раз в определенную дату и время, что удобно для некоторых заданий, выполнение которых необходимо отложить.
- Для повторяющегося расписания.

> [!Note]
> SQL Управляемый экземпляр в настоящее время не позволяет запускать задание, когда ЦП бездействует.

### <a name="sql-agent-job-notifications"></a>Уведомления о заданиях агента SQL

Задания агента SQL позволяют получать уведомления при успешном завершении задания или в случае возникновения ошибки. Уведомления можно получать по электронной почте.

Если он еще не включен, необходимо настроить [функцию Database Mail](/sql/relational-databases/database-mail/database-mail) в управляемый экземпляр SQL Azure:

```sql
GO
EXEC sp_configure 'show advanced options', 1;
GO
RECONFIGURE;
GO
EXEC sp_configure 'Database Mail XPs', 1;
GO
RECONFIGURE
```

В качестве примера упражнения настройте учетную запись электронной почты, которая будет использоваться для отправки уведомлений по электронной почте. Назначьте учетную запись профилю электронной почты с именем `AzureManagedInstance_dbmail_profile` . Чтобы отправить сообщение электронной почты с помощью заданий агента SQL Server в Управляемый экземпляр SQL, необходимо вызвать профиль, который должен быть вызван `AzureManagedInstance_dbmail_profile` . В противном случае SQL Управляемый экземпляр не сможет отправить сообщения электронной почты через агент SQL. См. Следующий пример:

```sql
-- Create a Database Mail account
EXECUTE msdb.dbo.sysmail_add_account_sp
    @account_name = 'SQL Agent Account',
    @description = 'Mail account for Azure SQL Managed Instance SQL Agent system.',
    @email_address = '$(loginEmail)',
    @display_name = 'SQL Agent Account',
    @mailserver_name = '$(mailserver)' ,
    @username = '$(loginEmail)' ,
    @password = '$(password)';

-- Create a Database Mail profile
EXECUTE msdb.dbo.sysmail_add_profile_sp
    @profile_name = 'AzureManagedInstance_dbmail_profile',
    @description = 'E-mail profile used for messages sent by Managed Instance SQL Agent.';

-- Add the account to the profile
EXECUTE msdb.dbo.sysmail_add_profileaccount_sp
    @profile_name = 'AzureManagedInstance_dbmail_profile',
    @account_name = 'SQL Agent Account',
    @sequence_number = 1;
```

Протестируйте конфигурацию Database Mail с помощью T-SQL, используя системную хранимую процедуру [sp_send_db_mail](/sql/relational-databases/system-stored-procedures/sp-send-dbmail-transact-sql) :

```sql
DECLARE @body VARCHAR(4000) = 'The email is sent from ' + @@SERVERNAME;
EXEC msdb.dbo.sp_send_dbmail  
    @profile_name = 'AzureManagedInstance_dbmail_profile',  
    @recipients = 'ADD YOUR EMAIL HERE',  
    @body = 'Add some text',  
    @subject = 'Azure SQL Instance - test email';  
```

Вы можете уведомить оператора, что с вашими заданиями агента SQL что-то произошло. Оператор определяет контактные сведения о лице, ответственном за обслуживание одного или нескольких экземпляров в Управляемом экземпляре SQL. Иногда обязанности оператора возлагаются на одно лицо.

В системах с несколькими экземплярами в Управляемом экземпляре SQL или сервере SQL Server обязанности оператора могут разделяться между несколькими лицами. Оператор не содержит сведений о безопасности и не определяет субъект безопасности. В идеале оператор не является сотрудником, обязанности которого могут измениться, но группа рассылки электронной почты.   

Операторы можно [создавать](/sql/relational-databases/system-stored-procedures/sp-add-operator-transact-sql) с помощью SQL Server Management Studio (SSMS) или скрипта TRANSACT-SQL, как показано в следующем примере:

```sql
EXEC msdb.dbo.sp_add_operator
    @name=N'AzureSQLTeam',
    @enabled=1,
    @email_address=N'AzureSQLTeamn@contoso.com';
```

Подтвердите успешность или сбой электронной почты с помощью [журнала Database Mail](/sql/relational-databases/database-mail/database-mail-log-and-audits) в SSMS.

Затем можно [изменить любое задание агента SQL](/sql/relational-databases/system-stored-procedures/sp-update-job-transact-sql) и назначить операторы, которые будут уведомлены по электронной почте, если задание завершается, завершается сбоем или завершается с помощью SSMS или следующего скрипта TRANSACT-SQL:

```sql
EXEC msdb.dbo.sp_update_job @job_name=N'Load data using SSIS',
    @notify_level_email=3, -- Options are: 1 on succeed, 2 on failure, 3 on complete
    @notify_email_operator_name=N'AzureSQLTeam';
```

### <a name="sql-agent-job-history"></a>Журнал заданий агента SQL

В настоящее время Управляемый экземпляр Azure SQL не позволяет изменять какие-либо свойства агента SQL Server, так как они хранятся в базовых значениях реестра. Это означает, что параметры настройки политики хранения агента для записей журнала заданий фиксированы по умолчанию — 1000 записей и максимум 100 записей по каждому заданию.

### <a name="sql-agent-fixed-database-role-membership"></a>Членство в предопределенной роли базы данных агента SQL

Если пользователи, связанные с именами входа, не являющимися администраторами, добавляются в любую из трех предопределенных ролей базы данных агента SQL в системной базе данных msdb, существует проблема, в которой для работы этих имен входа необходимо предоставить явные разрешения на выполнение главным хранимым процедурам. В случае возникновения этой проблемы появляется сообщение об ошибке "недопустимое разрешение на выполнение объекта <object_name будет показано> (Microsoft SQL Server, ошибка: 229)". 

После добавления пользователей в предопределенную роль базы данных агента SQL (SQLAgentUserRole, SQLAgentReaderRole или SQLAgentOperatorRole) в msdb для каждого имени входа пользователя, добавленного к этим ролям, выполните приведенный ниже скрипт T-SQL, чтобы явно предоставить разрешения на выполнение для системных хранимых процедур. В этом примере предполагается, что имя пользователя и имя входа одинаковы. 

```sql
USE [master]
GO
CREATE USER [login_name] FOR LOGIN [login_name];
GO
GRANT EXECUTE ON master.dbo.xp_sqlagent_enum_jobs TO [login_name];
GRANT EXECUTE ON master.dbo.xp_sqlagent_is_starting TO [login_name];
GRANT EXECUTE ON master.dbo.xp_sqlagent_notify TO [login_name];
```

## <a name="learn-more"></a>Материалы для дальнейшего изучения

- [Что такое Управляемый экземпляр SQL Azure?](../managed-instance/sql-managed-instance-paas-overview.md)
- [Что нового в базе данных SQL Azure & Управляемый экземпляр SQL?](../../azure-sql/database/doc-changes-updates-release-notes.md?tabs=managed-instance)
- [Отличия T-SQL Управляемый экземпляр SQL Azure от SQL Server](../../azure-sql/managed-instance/transact-sql-tsql-differences-sql-server.md#sql-server-agent)
- [Сравнение функций: база данных SQL Azure и Управляемый экземпляр Azure SQL](../../azure-sql/database/features-comparison.md)
