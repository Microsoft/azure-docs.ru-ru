---
title: Создание агента заданий обработки эластичных баз данных с помощью PowerShell (предварительная версия)
description: Дополнительные сведения о создании агента заданий обработки эластичных баз данных с помощью PowerShell.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, devx-track-azurepowershell
ms.devlang: ''
ms.topic: tutorial
author: johnpaulkee
ms.author: joke
ms.reviwer: sstein
ms.date: 10/21/2020
ms.openlocfilehash: 95e9ef340328bb4c1835e966cc9c3019bca88c09
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100368835"
---
# <a name="create-an-elastic-job-agent-using-powershell-preview"></a>Создание агента заданий обработки эластичных баз данных с помощью PowerShell (предварительная версия)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[Задания обработки эластичных баз данных (предварительная версия)](job-automation-overview.md) позволяют выполнять скрипты Transact-SQL (T-SQL) в нескольких базах данных одновременно.

В рамках этого учебника вы узнаете о задачах, необходимых для выполнения запроса в нескольких базах данных:

> [!div class="checklist"]
> * создание агента заданий обработки эластичных баз данных;
> * создание учетных данных заданий для выполнения скриптов на целевых объектах с помощью этих заданий;
> * определение целевых объектов (серверы, эластичные пулы, базы данных, карты сегментов), для которых следует выполнить задание;
> * создание учетных данных для базы данных в целевых базах данных для подключения агента и выполнения заданий;
> * Создание задания
> * добавление шагов задания;
> * запуск выполнения задания;
> * мониторинг задания.

## <a name="prerequisites"></a>Предварительные требования

Новая версия задания обработки эластичных баз данных обладает новым набором командлетов PowerShell, которые можно использовать во время процесса миграции. С помощью новых командлетов будут перенесены все существующие учетные данные задания, объекты (базы данных, сервера, пользовательские коллекции), триггеры задания, расписания задания, содержимое задания на новый агент заданий обработки эластичных баз данных.

### <a name="install-the-latest-elastic-jobs-cmdlets"></a>Установка агента задания обработки эластичных баз данных

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

Установите модуль **Az.Sql**, чтобы получать актуальные командлеты заданий обработки эластичных баз данных. Выполните приведенные ниже команды в PowerShell с правами администратора.

```powershell
# installs the latest PackageManagement and PowerShellGet packages
Find-Package PackageManagement | Install-Package -Force
Find-Package PowerShellGet | Install-Package -Force

# Restart your powershell session with administrative access

# Install and import the Az.Sql module, then confirm
Install-Module -Name Az.Sql
Import-Module Az.Sql

Get-Module Az.Sql
```

Для работы с этим руководством кроме модуля **Az.Sql** также требуется модуль *SqlServer* PowerShell. Дополнительные сведения см. в статье [Установка модуля SQL Server PowerShell](/sql/powershell/download-sql-server-ps-module).

## <a name="create-required-resources"></a>Создание необходимых ресурсов

Для создания агента заданий обработки эластичных баз данных требуется база данных (S0 или более поздней версии). Она будет использоваться в качестве [базы данных заданий](job-automation-overview.md#elastic-job-database).

Следующий скрипт позволяет создать группу ресурсов, сервер и базу данных, которые можно использовать в базе данных заданий. Второй скрипт также создает второй сервер с двумя пустыми базами данных для выполнения заданий.

Для задания обработки эластичных баз данных нет особых требований к именованию, поэтому можно использовать любые соглашения об именовании (при условии, что они соответствуют всем [требованиям Azure](/azure/architecture/best-practices/resource-naming)).

```powershell
# sign in to Azure account
Connect-AzAccount

# create a resource group
Write-Output "Creating a resource group..."
$resourceGroupName = Read-Host "Please enter a resource group name"
$location = Read-Host "Please enter an Azure Region"
$rg = New-AzResourceGroup -Name $resourceGroupName -Location $location
$rg

# create a server
Write-Output "Creating a server..."
$agentServerName = Read-Host "Please enter an agent server name"
$agentServerName = $agentServerName + "-" + [guid]::NewGuid()
$adminLogin = Read-Host "Please enter the server admin name"
$adminPassword = Read-Host "Please enter the server admin password"
$adminPasswordSecure = ConvertTo-SecureString -String $AdminPassword -AsPlainText -Force
$adminCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $adminLogin, $adminPasswordSecure
$agentServer = New-AzSqlServer -ResourceGroupName $resourceGroupName -Location $location `
    -ServerName $agentServerName -ServerVersion "12.0" -SqlAdministratorCredentials ($adminCred)

# set server firewall rules to allow all Azure IPs
Write-Output "Creating a server firewall rule..."
$agentServer | New-AzSqlServerFirewallRule -AllowAllAzureIPs
$agentServer

# create the job database
Write-Output "Creating a blank database to be used as the Job Database..."
$jobDatabaseName = "JobDatabase"
$jobDatabase = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $agentServerName -DatabaseName $jobDatabaseName -RequestedServiceObjectiveName "S0"
$jobDatabase
```

```powershell
# create a target server and sample databases - uses the same credentials
Write-Output "Creating target server..."
$targetServerName = Read-Host "Please enter a target server name"
$targetServerName = $targetServerName + "-" + [guid]::NewGuid()
$targetServer = New-AzSqlServer -ResourceGroupName $resourceGroupName -Location $location `
    -ServerName $targetServerName -ServerVersion "12.0" -SqlAdministratorCredentials ($adminCred)

# set target server firewall rules to allow all Azure IPs
$targetServer | New-AzSqlServerFirewallRule -AllowAllAzureIPs
$targetServer | New-AzSqlServerFirewallRule -StartIpAddress 0.0.0.0 -EndIpAddress 255.255.255.255 -FirewallRuleName AllowAll
$targetServer

# create sample databases to execute jobs against
$db1 = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $targetServerName -DatabaseName "database1"
$db1
$db2 = New-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $targetServerName -DatabaseName "database2"
$db2
```

### <a name="create-the-elastic-job-agent"></a>Создание агента заданий обработки эластичных баз данных

Агент заданий обработки эластичных баз данных является ресурсом Azure для создания, запуска заданий и управления ими. Агент выполняет запланированные или разовые задания.

Командлет **New-AzSqlElasticJobAgent** требует наличия базы данных в службе "База данных SQL Azure", поэтому параметры *resourceGroupName*, *serverName* и *databaseName* должны указывать на существующие ресурсы.

```powershell
Write-Output "Creating job agent..."
$agentName = Read-Host "Please enter a name for your new Elastic Job agent"
$jobAgent = $jobDatabase | New-AzSqlElasticJobAgent -Name $agentName
$jobAgent
```

### <a name="create-the-job-credentials"></a>Создание учетных данных заданий

Задания используют учетные данные для базы данных, чтобы подключаться к целевым базам данных, указанным целевой группой при выполнении и запуске скриптов. Эти учетные данные для баз данных также используются для подключения к базе данных master. Это позволяет перечислить все базы данных сервера или эластичного пула, когда они используются как тип элемента целевой группы.

Учетные данные для базы данных должны быть созданы в базе данных заданий. Для успешного завершения задания все целевые базы данных должны иметь имя для входа с достаточными полномочиями.

![Учетные данные заданий обработки эластичных баз данных](./media/elastic-jobs-powershell-create/job-credentials.png)

Помимо учетных данных в образе обратите внимание на добавление команд **GRANT** в следующем скрипте. Эти разрешения являются обязательными для скрипта, выбранного в этом примере задания. Так как в этом примере создается таблица в целевых базах данных, каждой из этих баз данных необходимо предоставить соответствующие разрешения для успешного выполнения.

Чтобы создать требуемые учетные данные задания (в базе данных заданий), выполните следующий скрипт:

```powershell
# in the master database (target server)
# create the master user login, master user, and job user login
$params = @{
  'database' = 'master'
  'serverInstance' =  $targetServer.ServerName + '.database.windows.net'
  'username' = $adminLogin
  'password' = $adminPassword
  'outputSqlErrors' = $true
  'query' = 'CREATE LOGIN masteruser WITH PASSWORD=''password!123'''
}
Invoke-SqlCmd @params
$params.query = "CREATE USER masteruser FROM LOGIN masteruser"
Invoke-SqlCmd @params
$params.query = 'CREATE LOGIN jobuser WITH PASSWORD=''password!123'''
Invoke-SqlCmd @params

# for each target database
# create the jobuser from jobuser login and check permission for script execution
$targetDatabases = @( $db1.DatabaseName, $Db2.DatabaseName )
$createJobUserScript =  "CREATE USER jobuser FROM LOGIN jobuser"
$grantAlterSchemaScript = "GRANT ALTER ON SCHEMA::dbo TO jobuser"
$grantCreateScript = "GRANT CREATE TABLE TO jobuser"

$targetDatabases | % {
  $params.database = $_
  $params.query = $createJobUserScript
  Invoke-SqlCmd @params
  $params.query = $grantAlterSchemaScript
  Invoke-SqlCmd @params
  $params.query = $grantCreateScript
  Invoke-SqlCmd @params
}

# create job credential in Job database for master user
Write-Output "Creating job credentials..."
$loginPasswordSecure = (ConvertTo-SecureString -String 'password!123' -AsPlainText -Force)

$masterCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList "masteruser", $loginPasswordSecure
$masterCred = $jobAgent | New-AzSqlElasticJobCredential -Name "masteruser" -Credential $masterCred

$jobCred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList "jobuser", $loginPasswordSecure
$jobCred = $jobAgent | New-AzSqlElasticJobCredential -Name "jobuser" -Credential $jobCred
```

### <a name="define-the-target-databases-to-run-the-job-against"></a>Определение целевых баз данных, для которых нужно запустить задание

[Целевая группа](job-automation-overview.md#target-group) определяет набор из одной или нескольких баз данных, в которых будет выполняться шаг задания.

Следующий фрагмент кода позволяет создать две целевые группы: *serverGroup* и *serverGroupExcludingDb2*. *serverGroup* охватывает все базы данных, имеющиеся на сервере во время выполнения, а *serverGroupExcludingDb2* охватывает все базы данных сервера, за исключением *targetDb2*:

```powershell
Write-Output "Creating test target groups..."
# create ServerGroup target group
$serverGroup = $jobAgent | New-AzSqlElasticJobTargetGroup -Name 'ServerGroup'
$serverGroup | Add-AzSqlElasticJobTarget -ServerName $targetServerName -RefreshCredentialName $masterCred.CredentialName

# create ServerGroup with an exclusion of db2
$serverGroupExcludingDb2 = $jobAgent | New-AzSqlElasticJobTargetGroup -Name 'ServerGroupExcludingDb2'
$serverGroupExcludingDb2 | Add-AzSqlElasticJobTarget -ServerName $targetServerName -RefreshCredentialName $masterCred.CredentialName
$serverGroupExcludingDb2 | Add-AzSqlElasticJobTarget -ServerName $targetServerName -Database $db2.DatabaseName -Exclude
```

### <a name="create-a-job-and-steps"></a>Создание задания и шагов

В этом примере определяется задание и два шага для выполнения задания. Первый шаг задания (*step1*) позволяет создать таблицу (*Step1Table*) в каждой базе данных целевой группы *ServerGroup*. Второй шаг задания (*step2*) позволяет создать таблицу (*Step2Table*) в каждой базе данных, за исключением *TargetDb2*, так как целевая группа, определенная ранее, предназначена для исключения этой базы данных.

```powershell
Write-Output "Creating a new job..."
$jobName = "Job1"
$job = $jobAgent | New-AzSqlElasticJob -Name $jobName -RunOnce
$job

Write-Output "Creating job steps..."
$sqlText1 = "IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id('Step1Table')) CREATE TABLE [dbo].[Step1Table]([TestId] [int] NOT NULL);"
$sqlText2 = "IF NOT EXISTS (SELECT * FROM sys.tables WHERE object_id = object_id('Step2Table')) CREATE TABLE [dbo].[Step2Table]([TestId] [int] NOT NULL);"

$job | Add-AzSqlElasticJobStep -Name "step1" -TargetGroupName $serverGroup.TargetGroupName -CredentialName $jobCred.CredentialName -CommandText $sqlText1
$job | Add-AzSqlElasticJobStep -Name "step2" -TargetGroupName $serverGroupExcludingDb2.TargetGroupName -CredentialName $jobCred.CredentialName -CommandText $sqlText2
```

### <a name="run-the-job"></a>Запуск задания

Для немедленного запуска задания выполните следующую команду:

```powershell
Write-Output "Start a new execution of the job..."
$jobExecution = $job | Start-AzSqlElasticJob
$jobExecution
```

После успешного завершения команды появятся две новые таблицы в TargetDb1 и только одна новая таблица в TargetDb2:

   ![проверка новых таблиц в SSMS](./media/elastic-jobs-powershell-create/job-execution-verification.png)

Кроме того, вы можете запланировать более позднее выполнение задания. Чтобы запланировать выполнение задания в определенное время, выполните следующую команду:

```powershell
# run every hour starting from now
$job | Set-AzSqlElasticJob -IntervalType Hour -IntervalCount 1 -StartTime (Get-Date) -Enable
```

### <a name="monitor-status-of-job-executions"></a>Мониторинг состояния выполнения задания

С помощью следующих фрагментов кода можно получить сведения о выполнении задания:

```powershell
# get the latest 10 executions run
$jobAgent | Get-AzSqlElasticJobExecution -Count 10

# get the job step execution details
$jobExecution | Get-AzSqlElasticJobStepExecution

# get the job target execution details
$jobExecution | Get-AzSqlElasticJobTargetExecution -Count 2
```

В приведенной ниже таблице указаны возможные состояния выполнения заданий.

|Состояние|Описание|
|:---|:---|
|**Создан** | Выполнение задания было только что создано и еще не началось.|
|**InProgress** | Задание сейчас находится в процессе выполнения.|
|**WaitingForRetry** | Не удалось завершить выполнение задания. Ожидается повторная попытка.|
|**Успешно** | Выполнение задания успешно завершено.|
|**SucceededWithSkipped** | Выполнение задания успешно завершено, но некоторые из его дочерних элементов были пропущены.|
|**Сбой** | Выполнение задания завершилось ошибкой. Количество повторных попыток исчерпано.|
|**TimedOut** | Истекло время ожидания выполнения задания.|
|**Canceled** | Выполнение задания было отменено.|
|**Пропущено** | Выполнение задания было пропущено по причине другого выполнения шага этого же задания.|
|**WaitingForChildJobExecutions** | Выполнение задания ожидает завершения выполнения дочерних элементов.|

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите ресурсы Azure, созданные в рамках этого руководства, удалив группу ресурсов.

> [!TIP]
> Если вы планируете продолжить работу с этими событиями, не очищайте ресурсы, созданные при работе с этой статьей.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого руководства вы выполняли сценарий Transact-SQL для набора баз данных. Вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * создание агента заданий обработки эластичных баз данных;
> * создание учетных данных заданий для выполнения скриптов на целевых объектах с помощью этих заданий;
> * определение целевых объектов (серверы, эластичные пулы, базы данных, карты сегментов), для которых следует выполнить задание;
> * создание учетных данных для базы данных в целевых базах данных для подключения агента и выполнения заданий;
> * Создание задания
> * Добавление шага задания.
> * Запуск выполнения задания.
> * Отслеживание задания

> [!div class="nextstepaction"]
> [Use Transact-SQL (T-SQL) to create and manage Elastic Database Jobs](elastic-jobs-tsql-create-manage.md) (Создание заданий обработки эластичных баз данных и управление ими с помощью Transact-SQL (T-SQL))
