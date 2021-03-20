---
title: Переход на новые задания обработки эластичных баз данных (Предварительная версия)
description: Миграция к новым заданиям обработки эластичных баз данных.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: johnpaulkee
ms.author: joke
ms.reviewer: sstein
ms.date: 03/13/2019
ms.openlocfilehash: e8c222d7cdcdb203cd323cde38818e5b51643159
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91443373"
---
# <a name="migrate-to-the-new-elastic-database-jobs-preview"></a>Переход на новый Задания обработки эластичных баз данных (Предварительная версия)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Доступна новая версия [задания обработки эластичных баз данных](elastic-jobs-overview.md).

Если у вас есть существующая версия задания обработки эластичных баз данных, установленная на стороне пользователя, миграцию на последнюю версию можно упростить с помощью следующих командлетов и скриптов миграции.


## <a name="prerequisites"></a>Предварительные требования

Новая версия задания обработки эластичных баз данных обладает новым набором командлетов PowerShell, которые можно использовать во время процесса миграции. С помощью новых командлетов будут перенесены все существующие учетные данные задания, объекты (базы данных, сервера, пользовательские коллекции), триггеры задания, расписания задания, содержимое задания на новый агент заданий обработки эластичных баз данных.

### <a name="install-the-latest-elastic-jobs-cmdlets"></a>Установка агента задания обработки эластичных баз данных

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.

Установите модуль **Az.Sql** предварительной версии 1.1.1, чтобы получать актуальные командлеты заданий обработки эластичных баз данных. Выполните приведенные ниже команды в PowerShell с правами администратора.

```powershell
# Installs the latest PackageManagement powershell package which PowerShellGet v1.6.5 is dependent on
Find-Package PackageManagement -RequiredVersion 1.1.7.2 | Install-Package -Force

# Installs the latest PowerShellGet module which adds the -AllowPrerelease flag to Install-Module
Find-Package PowerShellGet -RequiredVersion 1.6.5 | Install-Package -Force

# Restart your powershell session with administrative access

# Places Az.Sql preview cmdlets side by side with existing Az.Sql version
Install-Module -Name Az.Sql -RequiredVersion 1.1.1-preview -AllowPrerelease

# Import the Az.Sql module
Import-Module Az.Sql -RequiredVersion 1.1.1

# Confirm if module successfully imported - if the imported version is 1.1.1, then continue
Get-Module Az.Sql
```

### <a name="create-a-new-elastic-job-agent"></a>Создание нового агента заданий обработки эластичных баз данных

Установив новые командлеты, создайте нового агента заданий обработки эластичных баз данных.

```powershell
# Register your subscription for the for the Elastic Jobs public preview feature
Register-AzProviderFeature -FeatureName sqldb-JobAccounts -ProviderNamespace Microsoft.Sql

# Get an existing database to use as the job database - or create a new one if necessary
$db = Get-AzSqlDatabase -ResourceGroupName <resourceGroupName> -ServerName <serverName> -DatabaseName <databaseName>
# Create a new elastic job agent
$agent = $db | New-AzSqlElasticJobAgent -Name <agentName>
```

### <a name="install-the-old-elastic-database-jobs-cmdlets"></a>Установка устаревших версий командлетов задания обработки эластичных баз данных

Для миграции необходимо использовать некоторые из *старых* версий командлетов заданий обработки эластичных баз данных, поэтому, если они не установлены, необходимо выполнить следующие команды.

```powershell
# Install the old elastic job cmdlets if necessary and initialize the old jobs cmdlets
.\nuget install Microsoft.Azure.SqlDatabase.Jobs -prerelease

# Install the old jobs cmdlets
cd Microsoft.Azure.SqlDatabase.Jobs.x.x.xxxx.x*\tools
Unblock-File .\InstallElasticDatabaseJobsCmdlets.ps1
.\InstallElasticDatabaseJobsCmdlets.ps1

# Choose the subscription where your existing jobs are
Select-AzSubscription -SubscriptionId <subscriptionId>
Use-AzureSqlJobConnection -CurrentAzureSubscription -Credential (Get-Credential)
```



## <a name="migration"></a>Миграция

Теперь когда обе версии (новая и старая) командлетов задания обработки эластичных баз данных инициализированы, можно выполнить миграцию учетных данных задания, и заданий в новую *базу данных заданий*.

### <a name="setup"></a>Настройка

```powershell
$ErrorActionPreference = "Stop";

# Helper function to show starting write output
function Log-StartOutput ($output) {
  Write-Output ("`r--------------------- " + $output + " ---------------------")
}

# Helper function to show starting write output
function Log-ChildOutput ($output) {
  Write-Output ("  - " + $output)
}
```



### <a name="migrate-credentials"></a>Миграция учетных данных

```powershell
function Migrate-Credentials ($agent) {
    Log-StartOutput "Migrating credentials"

    $oldCreds = Get-AzureSqlJobCredential
    $oldCreds | % {
        $oldCredName = $_.CredentialName
        $oldUserName = $_.UserName
        Write-Output ("Credential " + $oldCredName)
        $oldCredential = Get-Credential -UserName $oldUserName `
                         -Message ("Please enter in the password that was used for your credential " + $oldCredName)
        try
        {
            $cred = New-AzSqlElasticJobCredential -ParentObject $agent -Name $oldCredName -Credential $oldCredential
        }
        catch [System.Management.Automation.PSArgumentException]
        {
            $cred = Get-AzSqlElasticJobCredential -ParentObject $agent -Name $oldCredName
            $cred = Set-AzSqlElasticJobCredential -InputObject $cred -Credential $oldCredential
        }

        Log-ChildOutput ("Added user " + $oldUserName)
    }
}
```

Чтобы перенести свои учетные данные, выполните следующую команду, передав объект `$agent` PowerShell из более ранней версии.

```powershell
Migrate-Credentials $agent
```

Пример выходных данных

```powershell
# You should see similar output after executing the above
# --------------------- Migrating credentials ---------------------
# Credential cred1
#  - Added user user1
# Credential cred2
#  - Added user user2
# Credential cred3
#  - Added user user3
```

### <a name="migrate-targets"></a>Миграция объектов

```powershell
function Migrate-TargetGroups ($agent) {
    Log-StartOutput "Migrating target groups"

    # Setup hash of target groups
    $targetGroups = [ordered]@{}

    # Fetch root job targets from old service
    $rootTargets = Get-AzureSqlJobTarget

    # Return if no root targets are found
    if ($rootTargets.Count -eq 0)
    {
        Write-Output "No targets found - no need for migration"
        return
    }

    # Create list of target groups to create
    # We format the target group name as such:
    # - If root target is server type, then target group name is "(serverName)"
    # - If root target is database type, then target group name is "(serverName,databaseName)"
    # - If root target is shard map type, then target group name is "(serverName,databaseName,shardMapName)"
    # - If root target is custom collection, then target group name is "customCollectionName"
    $rootTargets | % {
        $tgName = Format-OldTargetName -target $_
        $childTargets = Get-ChildTargets -target $_
        $targetGroups.Add($tgName, $childTargets)
    }

    # Flatten list
    for ($i=$targetGroups.Count - 1; $i -ge 0; $i--)
    {
        # Fetch target group's initial list of targets unexpanded
        $targets = $targetGroups[$i]

        # Expand custom collection targets
        $j = 0;
        while ($j -lt $targets.Count)
        {
            $target = $targets[$j]
            if ($target.TargetType -eq "CustomCollection")
            {
                $targets = [System.Collections.ArrayList] $targets
                $targets.Remove($target) # Remove this target from the list

                $expandedTargets = $targetGroups[$target.TargetDescription.CustomCollectionName]

                foreach ($expandedTarget in $expandedTargets)
                {
                    $targets.Add($expandedTarget) | Out-Null
                }

                # Set updated list of targets for tg
                $targetGroups[$i] = $targets
                # Note we don't increment here in case we need to expand further
            }
            else
            {
                # Skip if no custom collection target needs to be expanded
                $j++
            }
        }
    }

    # Add targets to target group
    foreach ($targetGroup in $targetGroups.Keys)
    {
        $tg = Setup-TargetGroup -tgName $targetGroup -agent $agent
        $targets = $targetGroups[$targetGroup]
        Migrate-Targets -targets $targets -tg $tg
        $targetsAdded = (Get-AzSqlElasticJobTargetGroup -ParentObject $agent -Name $tg.TargetGroupName).Targets
        foreach ($targetAdded in $targetsAdded)
        {
            Log-ChildOutput ("Added target " + (Format-NewTargetName $targetAdded))
        }
    }
}

## Target group helpers
# Migrate shard map target from old jobs to new job's target group
function Migrate-Targets ($targets, $tg) {
  Write-Output ("Target group " + $tg.TargetGroupName)
  foreach ($target in $targets) {
    if ($target.TargetType -eq "Server") {
      Add-ServerTarget -target $target -tg $tg
    }
    elseif ($target.TargetType -eq "Database") {
      Add-DatabaseTarget -target $target -tg $tg
    }
    elseif ($target.TargetType -eq "ShardMap") {
      Add-ShardMapTarget -target $target -tg $tg
    }
  }
}

# Migrate server target from old jobs to new job's target group
function Add-ServerTarget ($target, $tg) {
  $jobTarget = Get-AzureSqlJobTarget -TargetId $target.TargetId
  $serverName = $jobTarget.ServerName
  $credName = $jobTarget.MasterDatabaseCredentialName
  $t = Add-AzSqlElasticJobTarget -ParentObject $tg -ServerName $serverName -RefreshCredentialName $credName
}

# Migrate database target from old jobs to new job's target group
function Add-DatabaseTarget ($target, $tg) {
  $jobTarget = Get-AzureSqlJobTarget -TargetId $target.TargetId
  $serverName = $jobTarget.ServerName
  $databaseName = $jobTarget.DatabaseName
  $exclude = $target.Membership

  if ($exclude -eq "Exclude") {
    $t = Add-AzSqlElasticJobTarget -ParentObject $tg -ServerName $serverName -DatabaseName $databaseName -Exclude
  }
  else {
    $t = Add-AzSqlElasticJobTarget -ParentObject $tg -ServerName $serverName -DatabaseName $databaseName
  }
}

# Migrate shard map target from old jobs to new job's target group
function Add-ShardMapTarget ($target, $tg) {
  $jobTarget = Get-AzureSqlJobTarget -TargetId $target.TargetId
  $smName = $jobTarget.ShardMapName
  $serverName = $jobTarget.ShardMapManagerServerName
  $databaseName = $jobTarget.ShardMapManagerDatabaseName
  $credName = $jobTarget.ShardMapManagerCredentialName
  $exclude = $target.Membership

  if ($exclude -eq "Exclude") {
    $t = Add-AzSqlElasticJobTarget -ParentObject $tg -ServerName $serverName -ShardMapName $smName -DatabaseName $databasename -RefreshCredentialName $credName -Exclude
  }
  else {
    $t = Add-AzSqlElasticJobTarget -ParentObject $tg -ServerName $serverName -ShardMapName $smName -DatabaseName $databasename -RefreshCredentialName $credName
  }
}

# Helper to format target old target names
function Format-OldTargetName ($target) {
  if ($target.TargetType -eq "Server") {
    $tgName = "(" + $target.ServerName + ")"
  }
  elseif ($target.TargetType -eq "Database") {
    $tgName = "(" + $target.ServerName + "," + $target.DatabaseName + ")"
  }
  elseif ($target.TargetType -eq "ShardMap") {
    $tgName = "(" + $target.ShardMapManagerServerName + "," +
    $target.ShardMapManagerDatabaseName + "," + `
      $target.ShardMapName + ")"
  }
  elseif ($target.TargetType -eq "CustomCollection") {
    $tgName = $target.CustomCollectionName
  }

  return $tgName
}

# Helper to format new target names
function Format-NewTargetName ($target) {
  if ($target.TargetType -eq "SqlServer") {
    $tgName = "(" + $target.TargetServerName + ")"
  }
  elseif ($target.TargetType -eq "SqlDatabase") {
    $tgName = "(" + $target.TargetServerName + "," + $target.TargetDatabaseName + ")"
  }
  elseif ($target.TargetType -eq "SqlShardMap") {
    $tgName = "(" + $target.TargetServerName + "," +
    $target.TargetDatabaseName + "," + `
      $target.TargetShardMapName + ")"
  }
  elseif ($target.TargetType -eq "SqlElasticPool") {
    $tgName = "(" + $target.TargetServerName + "," +
    $target.TargetDatabaseName + "," + `
      $target.TargetElasticPoolName + ")"
  }

  return $tgName
}

# Get child targets
function Get-ChildTargets($target) {
  if ($target.TargetType -eq "CustomCollection") {
    $children = Get-AzureSqlJobChildTarget -TargetId $target.TargetId
    if ($children.Count -eq 1)
    {
        $arr = New-Object System.Collections.ArrayList($null)
        $arr.Add($children)
        $children = $arr
    }
    return $children
  }
  else {
    return $target
  }
}

# Migrates target groups
function Setup-TargetGroup ($tgName, $agent) {
  try {
    $tg = New-AzSqlElasticJobTargetGroup -ParentObject $agent -Name $tgName
    return $tg
  }
  catch [System.Management.Automation.PSArgumentException] {
    $tg = Get-AzSqlElasticJobTargetGroup -ParentObject $agent -Name $tgName
    return $tg
  }
}
```

Чтобы перенести целевые объекты (серверы, базы данных и пользовательские коллекции) в новую базу данных заданий, выполните командлет **Migrate-TargetGroups**.

- Целевые объекты корневого уровня, которые являются серверами и базами данных, будут перенесены в новую целевую группу с именем "( \<serverName\> , \<databaseName\> )", содержащую только целевой объект корневого уровня.
- Пользовательская коллекция будет перенесена в новую целевую группу, содержащую все дочерние объекты.

```powershell
Migrate-TargetGroups $agent
```

Пример результатов выполнения:

```powershell
# --------------------- Migrating target groups ---------------------
# Target group cc1
#   - Added target (s1)
#   - Added target (s1,db1)
# Target group cc2
#   - Added target (s1,db1)
# Target group cc3
#   - Added target (s1)
#   - Added target (s1,db1)
# Target group (s1,db1)
#   - Added target (s1,db1)
# Target group (s1,db2)
#   - Added target (s1,db2)
# Target group (s1)
#   - Added target (s1)
# Target group (s1,db1,sm1)
#   - Added target (s1,db1,sm1)
```



### <a name="migrate-jobs"></a>Миграция заданий

```powershell
function Migrate-Jobs ($agent)
{
    Log-StartOutput "Migrating jobs and job steps"

    $oldJobs = Get-AzureSqlJob
    $newJobs = [System.Collections.ArrayList] @()

    foreach ($oldJob in $oldJobs)
    {
        # Ignore system jobs
        if ($oldJob.ContentName -eq $null)
        {
            continue
        }

        # Schedule
        $oldJobTriggers = Get-AzureSqlJobTrigger -JobName $oldJob.JobName

        if ($oldJobTriggers.Count -ge 1)
        {
            foreach ($trigger in $oldJobTriggers)
            {

                $schedule = Get-AzureSqlJobSchedule -ScheduleName $trigger.ScheduleName
                $newJob = [PSCustomObject] @{
                    JobName  = ($trigger.JobName + " (" + $trigger.ScheduleName + ")");
                    Description = $oldJob.ContentName
                    Schedule = $schedule
                    TargetGroupName = (Format-OldTargetName(Get-AzureSqlJobTarget -TargetId $oldJob.TargetId))
                    CredentialName = $oldJob.CredentialName
                    Output = $oldJob.ResultSetDestination
                }
                $newJobs.Add($newJob) | Out-Null
            }
        }
        else
        {
            $newJob = [PSCustomObject] @{
                JobName  = $oldJob.JobName
                Description = $oldJob.ContentName
                Schedule = $null
                TargetGroupName = (Format-OldTargetName(Get-AzureSqlJobTarget -TargetId $oldJob.TargetId))
                CredentialName = $oldJob.CredentialName
                Output = $oldJob.ResultSetDestination
            }
            $newJobs.Add($newJob) | Out-Null
        }
    }

    # At this point, we should have an organized list of jobs to create
    foreach ($newJob in $newJobs)
    {
        Write-Output ("Job " + $newJob.JobName)
        $job = Setup-Job $newJob $agent
        If ($job.Interval -ne $null)
        {
            Log-ChildOutput ("Schedule with start time " + $job.StartTime + " and end time at " + $job.EndTime)
            Log-ChildOutput ("Repeats every " + $job.Interval)
        }
        else {
            Log-ChildOutput ("Repeats once")
        }

        Setup-JobStep $newJob $job
    }
}

# Migrates jobs
function Setup-Job ($job, $agent) {
  $jobName = $newJob.JobName
  $jobDescription = $newJob.Description

  # Create or update a job has a recurring schedule
  if ($newJob.Schedule -ne $null) {
    $schedule = $newJob.Schedule
    $startTime = $schedule.StartTime.UtcTime
    $endTime = $schedule.EndTime.UtcTime
    $intervalType = $schedule.Interval.IntervalType.ToString()
    $intervalType = $intervalType.Substring(0, $intervalType.Length - 1) # Remove the last letter (s)
    $intervalCount = $schedule.Interval.Count

    try {
      $job = New-AzSqlElasticJob -ParentObject $agent -Name $jobName `
        -Description $jobDescription -IntervalType $intervalType -IntervalCount $intervalCount `
        -StartTime $startTime -EndTime $endTime
      return $job
    }
    catch [System.Management.Automation.PSArgumentException] {
      $job = Get-AzSqlElasticJob -ParentObject $agent -Name $jobName
      $job = $job | Set-AzSqlElasticJob -Description $jobDescription -IntervalType $intervalType -IntervalCount $intervalCount `
        -StartTime $startTime -EndTime $endTime
      return $job
    }
  }
  # Create or update a job that runs once
  else {
    try {
      $job = New-AzSqlElasticJob -ParentObject $agent -Name $jobName `
        -Description $jobDescription -RunOnce
      return $job
    }
    catch [System.Management.Automation.PSArgumentException] {
      $job = Get-AzSqlElasticJob -ParentObject $agent -Name $jobName
      $job = $job | Set-AzSqlElasticJob -Description $jobDescription -RunOnce
      return $job
    }
  }
}
# Migrates job steps
function Setup-JobStep ($newJob, $job) {
  $defaultJobStepName = 'JobStep'
  $contentName = $newJob.Description
  $commandText = (Get-AzureSqlJobContentDefinition -ContentName $contentName).CommandText
  $targetGroupName = $newJob.TargetGroupName
  $credentialName = $newJob.CredentialName

  $output = $newJob.Output

  if ($output -ne $null) {
    $outputServerName = $output.TargetDescription.ServerName
    $outputDatabaseName = $output.TargetDescription.DatabaseName
    $outputCredentialName = $output.CredentialName
    $outputSchemaName = $output.SchemaName
    $outputTableName = $output.TableName
    $outputDatabase = Get-AzSqlDatabase -ResourceGroupName $job.ResourceGroupName -ServerName $outputServerName -Databasename $outputDatabaseName

    try {
      $jobStep = $job | Add-AzSqlElasticJobStep -Name $defaultJobStepName `
        -TargetGroupName $targetGroupName -CredentialName $credentialName -CommandText $commandText `
        -OutputDatabaseObject $outputDatabase `
        -OutputSchemaName $outputSchemaName -OutputTableName $outputTableName `
        -OutputCredentialName $outputCredentialName
    }
    catch [System.Management.Automation.PSArgumentException] {
      $jobStep = $job | Get-AzSqlElasticJobStep -Name $defaultJobStepName
      $jobStep = $jobStep | Set-AzSqlElasticJobStep -TargetGroupName $targetGroupName `
        -CredentialName $credentialName -CommandText $commandText `
        -OutputDatabaseObject $outputDatabase `
        -OutputSchemaName $outputSchemaName -OutputTableName $outputTableName `
        -OutputCredentialName $outputCredentialName
    }
  }
  else {
    try {
      $jobStep = $job | Add-AzSqlElasticJobStep -Name $defaultJobStepName -TargetGroupName $targetGroupName -CredentialName $credentialName -CommandText $commandText
    }
    catch [System.Management.Automation.PSArgumentException] {
      $jobStep = $job | Get-AzSqlElasticJobStep -Name $defaultJobStepName
      $jobStep = $jobStep | Set-AzSqlElasticJobStep -TargetGroupName $targetGroupName -CredentialName $credentialName -CommandText $commandText
    }
  }
  Log-ChildOutput ("Added step " + $jobStep.StepName + " using target group " + $jobStep.TargetGroupName + " using credential " + $jobStep.CredentialName)
  Log-ChildOutput("Command text script taken from content name " + $contentName)

  if ($jobStep.Output -ne $null) {
    Log-ChildOutput ("With output target as (" + $jobStep.Output.ServerName + "," + $jobStep.Output.DatabaseName + "," + $jobStep.Output.SchemaName + "," + $jobStep.Output.TableName + ")")
  }
}
```

Для выполнения миграции заданий, содержимого заданий, триггеров заданий и расписания заданий в новую базу данных агента заданий обработки эластичных баз данных, выполните командлет **Migrate-Jobs**, который появился в агенте.

- Задания с несколькими триггерами с разными расписаниями разделяются на несколько заданий с использованием схемы именования: " \<jobName\> ( \<scheduleName\> )".
- Содержимое задания переносится в задание путем добавления шага задания с именем JobStep с соответствующим текстом команды по умолчанию.
- По умолчанию задания отключены, поэтому до момента включения их невозможно проверить.

```powershell
Migrate-Jobs $agent
```

Пример результатов выполнения:
```powershell
--------------------- Migrating jobs and job steps ---------------------
Job job1
  - Repeats once
  - Added step JobStep using target group cc2 using credential cred1
  - Command text script taken from content name SampleContext
Job job2
  - Repeats once
  - Added step JobStep using target group (s1,db1) using credential cred1
  - Command text script taken from content name SampleContent
  - With output target as (s1,db1,dbo,sampleTable)
Job job3 (repeat every 10 min)
  - Schedule with start time 05/16/2018 22:05:28 and end time at 12/31/9999 11:59:59
  - Repeats every PT10M
  - Added step JobStep using target group cc1 using credential cred1
  - Command text script taken from content name SampleContent
Job job3 (repeat every 5 min)
  - Schedule with start time 05/16/2018 22:05:31 and end time at 12/31/9999 11:59:59
  - Repeats every PT5M
  - Added step JobStep using target group cc1 using credential cred1
  - Command text script taken from content name SampleContent
Job job4
  - Repeats once
  - Added step JobStep using target group (s1,db1) using credential cred1
  - Command text script taken from content name SampleContent
```



## <a name="migration-complete"></a>Миграция завершена

В *задание базы данных* теперь должны быть указаны все учетные данные задания, объекты, триггеры заданий, расписания заданий и перенесенные задания.

Чтобы убедиться, что миграция выполнено правильно, используйте следующие скрипты:

```powershell
$creds = $agent | Get-AzSqlElasticJobCredential
$targetGroups = $agent | Get-AzSqlElasticJobTargetGroup
$jobs = $agent | Get-AzSqlElasticJob
$steps = $jobs | Get-AzSqlElasticJobStep
```

Чтобы проверить правильность выполнения этих заданий, запустите их.

```powershell
$jobs | Start-AzSqlElasticJob
```

Не забудьте их включить для любых заданий, которые выполнялись по расписанию, чтобы они могли работать в фоновом режиме.

```powershell
$jobs | Set-AzSqlElasticJob -Enable
```

## <a name="next-steps"></a>Дальнейшие действия

- [Создание заданий обработки эластичных баз данных и управление ими с помощью PowerShell](elastic-jobs-powershell-create.md)
- [Use Transact-SQL (T-SQL) to create and manage Elastic Database Jobs](elastic-jobs-tsql-create-manage.md) (Создание заданий обработки эластичных БД и управление ими с использованием Transact-SQL (T-SQL))
