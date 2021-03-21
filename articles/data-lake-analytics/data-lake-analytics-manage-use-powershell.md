---
title: Управление аналитикой озера данных Azure с помощью Azure PowerShell
description: В этой статье описано, как с помощью Azure PowerShell управлять учетными записями, источниками данных, пользователями и заданиями Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/29/2018
ms.openlocfilehash: f1d5d732433a78b8e3e7ff94bd6aaa2eea8f4a87
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102051792"
---
# <a name="manage-azure-data-lake-analytics-using-azure-powershell"></a>Управление аналитикой озера данных Azure с помощью Azure PowerShell

[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

В этой статье описано, как управлять учетными записями, источниками данных, пользователями и заданиями Azure Data Lake Analytics с помощью Azure PowerShell.

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Чтобы использовать PowerShell с Data Lake Analytics, получите следующие сведения:

* **Идентификатор подписки** — идентификатор подписки Azure, в которую входит ваша учетная запись Data Lake Analytics.
* **Группа ресурсов** — имя группы ресурсов Azure, содержащей учетную запись Data Lake Analytics.
* **Имя учетной записи** — имя вашей учетной записи Data Lake Analytics.
* **Имя учетной записи Data Lake Store по умолчанию** — каждая учетная запись Data Lake Analytics содержит учетную запись Data Lake Store по умолчанию.
* **Расположение** — расположение учетной записи Data Lake Analytics, например "Восточная часть США 2" или другое поддерживаемое расположение.

Во фрагментах кода PowerShell в этом руководстве для хранения такой информации используются следующие переменные:

```powershell
$subId = "<SubscriptionId>"
$rg = "<ResourceGroupName>"
$adla = "<DataLakeAnalyticsAccountName>"
$adls = "<DataLakeStoreAccountName>"
$location = "<Location>"
```

## <a name="log-in-to-azure"></a>Вход в Azure

### <a name="log-in-using-interactive-user-authentication"></a>Вход с использованием интерактивной аутентификации пользователей

Войдите, используя идентификатор или имя подписки.

```powershell
# Using subscription id
Connect-AzAccount -SubscriptionId $subId

# Using subscription name
Connect-AzAccount -SubscriptionName $subname
```

## <a name="saving-authentication-context"></a>Сохранение контекста аутентификации

При использовании командлета `Connect-AzAccount` всегда запрашиваются учетные данные. Избежать появления запроса можно с помощью следующих командлетов:

```powershell
# Save login session information
Save-AzAccounts -Path D:\profile.json  

# Load login session information
Select-AzAccounts -Path D:\profile.json
```

### <a name="log-in-using-a-service-principal-identity-spi"></a>Вход с использованием удостоверения субъекта-службы (SPI)

```powershell
$tenantid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"  
$spi_appname = "appname"
$spi_appid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
$spi_secret = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

$pscredential = New-Object System.Management.Automation.PSCredential ($spi_appid, (ConvertTo-SecureString $spi_secret -AsPlainText -Force))
Login-AzAccount -ServicePrincipal -TenantId $tenantid -Credential $pscredential -Subscription $subid
```

## <a name="manage-accounts"></a>Управление учетными записями

### <a name="list-accounts"></a>Список учетных записей

```powershell
# List Data Lake Analytics accounts within the current subscription.
Get-AdlAnalyticsAccount

# List Data Lake Analytics accounts within a specific resource group.
Get-AdlAnalyticsAccount -ResourceGroupName $rg
```

### <a name="create-an-account"></a>Создание учетной записи

Для каждой учетной записи Data Lake Analytics необходима учетная запись Data Lake Store по умолчанию, которая используется для хранения журналов. Можно повторно использовать существующую учетную запись или создать новую.

```powershell
# Create a data lake store if needed, or you can re-use an existing one
New-AdlStore -ResourceGroupName $rg -Name $adls -Location $location
New-AdlAnalyticsAccount -ResourceGroupName $rg -Name $adla -Location $location -DefaultDataLake $adls
```

### <a name="get-account-information"></a>Получение данных об учетной записи

Получение сведений об учетной записи.

```powershell
Get-AdlAnalyticsAccount -Name $adla
```

### <a name="check-if-an-account-exists"></a>Проверка наличия учетной записи

```powershell
Test-AdlAnalyticsAccount -Name $adla
```

## <a name="manage-data-sources"></a>Управление источниками данных

Azure Data Lake Analytics в настоящее время поддерживает следующие источники данных:

* [Azure Data Lake Storage](../data-lake-store/data-lake-store-overview.md)
* [Хранилище Azure](../storage/common/storage-introduction.md)

Для каждой учетной записи Data Lake Analytics существует учетная запись Data Lake Store по умолчанию. Учетная запись хранения озера данных по умолчанию используется для хранения метаданных задания и журналов аудита задания.

### <a name="find-the-default-data-lake-store-account"></a>Поиск учетной записи хранения озера данных по умолчанию

```powershell
$adla_acct = Get-AdlAnalyticsAccount -Name $adla
$dataLakeStoreName = $adla_acct.DefaultDataLakeAccount
```

Чтобы найти учетную запись по умолчанию для Data Lake Store, отфильтруйте список источников данных по свойству `IsDefault`:

```powershell
Get-AdlAnalyticsDataSource -Account $adla  | ? { $_.IsDefault }
```

### <a name="add-a-data-source"></a>Добавление источника данных

```powershell

# Add an additional Storage (Blob) account.
$AzureStorageAccountName = "<AzureStorageAccountName>"
$AzureStorageAccountKey = "<AzureStorageAccountKey>"
Add-AdlAnalyticsDataSource -Account $adla -Blob $AzureStorageAccountName -AccessKey $AzureStorageAccountKey

# Add an additional Data Lake Store account.
$AzureDataLakeStoreName = "<AzureDataLakeStoreAccountName"
Add-AdlAnalyticsDataSource -Account $adla -DataLakeStore $AzureDataLakeStoreName
```

### <a name="list-data-sources"></a>Получение списка источников данных

```powershell
# List all the data sources
Get-AdlAnalyticsDataSource -Account $adla

# List attached Data Lake Store accounts
Get-AdlAnalyticsDataSource -Account $adla | where -Property Type -EQ "DataLakeStore"

# List attached Storage accounts
Get-AdlAnalyticsDataSource -Account $adla | where -Property Type -EQ "Blob"
```

## <a name="submit-u-sql-jobs"></a>Отправка заданий U-SQL

### <a name="submit-a-string-as-a-u-sql-job"></a>Отправка строки как задания U-SQL

```powershell
$script = @"
@a  =
    SELECT * FROM
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
"@

$scriptpath = "d:\test.usql"
$script | Out-File $scriptpath

Submit-AdlJob -AccountName $adla -Script $script -Name "Demo"
```

### <a name="submit-a-file-as-a-u-sql-job"></a>Отправка файла как задания U-SQL

```powershell
$scriptpath = "d:\test.usql"
$script | Out-File $scriptpath
Submit-AdlJob -AccountName $adla –ScriptPath $scriptpath -Name "Demo"
```

### <a name="list-jobs"></a>Список заданий

Результаты включают в себя текущие и недавно завершенные задания.

```powershell
Get-AdlJob -Account $adla
```

### <a name="list-the-top-n-jobs"></a>Получение списка последних N заданий

По умолчанию список заданий сортируется по времени отправки. Поэтому в его начале находятся последние отправленные задания. По умолчанию в учетной записи ADLA сохраняются сведения о заданиях за 180 дней, но командлет Ge-AdlJob по умолчанию возвращает только первые 500 заданий. Чтобы получить список определенного количества заданий, используйте параметр -Top.

```powershell
$jobs = Get-AdlJob -Account $adla -Top 10
```

### <a name="list-jobs-by-job-state"></a>Получение списка заданий по их состоянию

Использование параметра `-State`. Вы можете использовать любое сочетание следующих значений:

* `Accepted`
* `Compiling`
* `Ended`
* `New`
* `Paused`
* `Queued`
* `Running`
* `Scheduling`
* `Start`

```powershell
# List the running jobs
Get-AdlJob -Account $adla -State Running

# List the jobs that have completed
Get-AdlJob -Account $adla -State Ended

# List the jobs that have not started yet
Get-AdlJob -Account $adla -State Accepted,Compiling,New,Paused,Scheduling,Start
```

### <a name="list-jobs-by-job-result"></a>Получение списка заданий по их результатам

Используйте параметр `-Result`, чтобы определить, успешно ли выполнено завершенное задание. Возможны следующие значения:

* Отменено
* Сбой
* Нет
* Выполнено

``` powershell
# List Successful jobs.
Get-AdlJob -Account $adla -State Ended -Result Succeeded

# List Failed jobs.
Get-AdlJob -Account $adla -State Ended -Result Failed
```

### <a name="list-jobs-by-job-submitter"></a>Получение списка заданий по их отправителям

Параметр `-Submitter` позволяет определить, кто отправил это задание.

```powershell
Get-AdlJob -Account $adla -Submitter "joe@contoso.com"
```

### <a name="list-jobs-by-submission-time"></a>Получение списка заданий по времени отправки

Параметр `-SubmittedAfter` используется для фильтрации по диапазону времени.

```powershell
# List  jobs submitted in the last day.
$d = [DateTime]::Now.AddDays(-1)
Get-AdlJob -Account $adla -SubmittedAfter $d

# List  jobs submitted in the last seven day.
$d = [DateTime]::Now.AddDays(-7)
Get-AdlJob -Account $adla -SubmittedAfter $d
```

### <a name="get-job-status"></a>Получение состояния задания

Получите состояние конкретного задания.

```powershell
Get-AdlJob -AccountName $adla -JobId $job.JobId
```

### <a name="cancel-a-job"></a>Отмена задания

```powershell
Stop-AdlJob -Account $adla -JobID $jobID
```

### <a name="wait-for-a-job-to-finish"></a>Ожидание завершения задания

Вместо того чтобы повторно выполнять `Get-AdlAnalyticsJob`, пока задание не завершится, можно использовать командлет `Wait-AdlJob`, чтобы дождаться завершения задания.

```powershell
Wait-AdlJob -Account $adla -JobId $job.JobId
```


## <a name="list-job-pipelines-and-recurrences"></a>Получение списка конвейеров и повторений заданий

Используйте командлет `Get-AdlJobPipeline`, чтобы получить сведения о конвейерах для ранее отправленных заданий.

```powershell
$pipelines = Get-AdlJobPipeline -Account $adla
$pipeline = Get-AdlJobPipeline -Account $adla -PipelineId "<pipeline ID>"
```

Используйте командлет `Get-AdlJobRecurrence`, чтобы получить сведения о повторениях для ранее отправленных заданий.

```powershell
$recurrences = Get-AdlJobRecurrence -Account $adla

$recurrence = Get-AdlJobRecurrence -Account $adla -RecurrenceId "<recurrence ID>"
```

## <a name="manage-compute-policies"></a>Управление политиками вычислений

### <a name="list-existing-compute-policies"></a>Список существующих политик вычислений

Командлет `Get-AdlAnalyticsComputePolicy` извлекает информацию о политиках вычислений для учетной записи Data Lake Analytics.

```powershell
$policies = Get-AdlAnalyticsComputePolicy -Account $adla
```

### <a name="create-a-compute-policy"></a>Создание политики вычислений

Командлет `New-AdlAnalyticsComputePolicy` создает новую политику вычислений для учетной записи Data Lake Analytics. В этом примере для указанного пользователя устанавливается максимальное число единиц использования аналитики (50) и минимальный приоритет задания (250).

```powershell
$userObjectId = (Get-AzAdUser -SearchString "garymcdaniel@contoso.com").Id

New-AdlAnalyticsComputePolicy -Account $adla -Name "GaryMcDaniel" -ObjectId $objectId -ObjectType User -MaxDegreeOfParallelismPerJob 50 -MinPriorityPerJob 250
```

## <a name="manage-files"></a>Управление файлами

### <a name="check-for-the-existence-of-a-file"></a>Проверка существования файла

```powershell
Test-AdlStoreItem -Account $adls -Path "/data.csv"
```

### <a name="uploading-and-downloading"></a>Отправка и скачивание

Отправка файла.

```powershell
Import-AdlStoreItem -AccountName $adls -Path "c:\data.tsv" -Destination "/data_copy.csv"
```

Рекурсивная отправка всей папки.

```powershell
Import-AdlStoreItem -AccountName $adls -Path "c:\myData\" -Destination "/myData/" -Recurse
```

Скачивание файла.

```powershell
Export-AdlStoreItem -AccountName $adls -Path "/data.csv" -Destination "c:\data.csv"
```

Рекурсивное скачивание всей папки.

```powershell
Export-AdlStoreItem -AccountName $adls -Path "/" -Destination "c:\myData\" -Recurse
```

> [!NOTE]
> Если процесс отправки или скачивания прервался, вы можете попытаться возобновить его, выполнив командлет еще раз с флагом ``-Resume``.

## <a name="manage-the-u-sql-catalog"></a>Управление каталогом U-SQL

Каталог U-SQL используется для структурирования данных и кода, чтобы их могли совместно использовать сценарии U-SQL. Каталог обеспечивает максимальную производительность, возможную с данными в озере данных Azure. Дополнительные сведения см. в разделе [Использование каталога U-SQL](./data-lake-analytics-u-sql-get-started.md).

### <a name="list-items-in-the-u-sql-catalog"></a>Получение списка элементов в каталоге U-SQL

```powershell
# List U-SQL databases
Get-AdlCatalogItem -Account $adla -ItemType Database

# List tables within a database
Get-AdlCatalogItem -Account $adla -ItemType Table -Path "database"

# List tables within a schema.
Get-AdlCatalogItem -Account $adla -ItemType Table -Path "database.schema"
```

### <a name="list-all-the-assemblies-the-u-sql-catalog"></a>Получение списка всех сборок в каталоге U-SQL

```powershell
$dbs = Get-AdlCatalogItem -Account $adla -ItemType Database

foreach ($db in $dbs)
{
    $asms = Get-AdlCatalogItem -Account $adla -ItemType Assembly -Path $db.Name

    foreach ($asm in $asms)
    {
        $asmname = "[" + $db.Name + "].[" + $asm.Name + "]"
        Write-Host $asmname
    }
}
```

### <a name="get-details-about-a-catalog-item"></a>Получение сведений об элементе каталога

```powershell
# Get details of a table
Get-AdlCatalogItem  -Account $adla -ItemType Table -Path "master.dbo.mytable"

# Test existence of a U-SQL database.
Test-AdlCatalogItem  -Account $adla -ItemType Database -Path "master"
```

### <a name="store-credentials-in-the-catalog"></a>Хранение учетных данных в каталоге

В базе данных U-SQL создайте объект учетных данных для базы данных, размещенной в Azure. В настоящее время учетные данные U-SQL — это единственный тип элементов каталога, который можно создавать с помощью PowerShell.

```powershell
$dbName = "master"
$credentialName = "ContosoDbCreds"
$dbUri = "https://contoso.database.windows.net:8080"

New-AdlCatalogCredential -AccountName $adla `
          -DatabaseName $db `
          -CredentialName $credentialName `
          -Credential (Get-Credential) `
          -Uri $dbUri
```

## <a name="manage-firewall-rules"></a>Управление правилами брандмауэра

### <a name="list-firewall-rules"></a>Вывод списка правил брандмауэра

```powershell
Get-AdlAnalyticsFirewallRule -Account $adla
```

### <a name="add-a-firewall-rule"></a>Добавление правила брандмауэра

```powershell
$ruleName = "Allow access from on-prem server"
$startIpAddress = "<start IP address>"
$endIpAddress = "<end IP address>"

Add-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName -StartIpAddress $startIpAddress -EndIpAddress $endIpAddress
```

### <a name="modify-a-firewall-rule"></a>Изменение правила брандмауэра

```powershell
Set-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName -StartIpAddress $startIpAddress -EndIpAddress $endIpAddress
```

### <a name="remove-a-firewall-rule"></a>Удаление правила брандмауэра

```powershell
Remove-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName
```

### <a name="allow-azure-ip-addresses"></a>Разрешение IP-адресов Azure

```powershell
Set-AdlAnalyticsAccount -Name $adla -AllowAzureIpState Enabled
```

```powershell
Set-AdlAnalyticsAccount -Name $adla -FirewallState Enabled
Set-AdlAnalyticsAccount -Name $adla -FirewallState Disabled
```

## <a name="working-with-azure"></a>Работа с Azure

### <a name="get-error-details"></a>Получение сведений об ошибке

```powershell
Resolve-AzError -Last
```

### <a name="verify-if-you-are-running-as-an-administrator-on-your-windows-machine"></a>Проверка того, что вход выполнен от имени администратора на компьютере Windows

```powershell
function Test-Administrator  
{  
    $user = [Security.Principal.WindowsIdentity]::GetCurrent();
    $p = New-Object Security.Principal.WindowsPrincipal $user
    $p.IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator)  
}
```

### <a name="find-a-tenantid"></a>Поиск TenantID

По имени подписки:

```powershell
function Get-TenantIdFromSubscriptionName( [string] $subname )
{
    $sub = (Get-AzSubscription -SubscriptionName $subname)
    $sub.TenantId
}

Get-TenantIdFromSubscriptionName "ADLTrainingMS"
```

По идентификатору подписки:

```powershell
function Get-TenantIdFromSubscriptionId( [string] $subid )
{
    $sub = (Get-AzSubscription -SubscriptionId $subid)
    $sub.TenantId
}

$subid = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
Get-TenantIdFromSubscriptionId $subid
```

По адресу домена, например contoso.com:

```powershell
function Get-TenantIdFromDomain( $domain )
{
    $url = "https://login.windows.net/" + $domain + "/.well-known/openid-configuration"
    return (Invoke-WebRequest $url|ConvertFrom-Json).token_endpoint.Split('/')[3]
}

$domain = "contoso.com"
Get-TenantIdFromDomain $domain
```

### <a name="list-all-your-subscriptions-and-tenant-ids"></a>Получение списка всех подписок и идентификаторов клиентов

```powershell
$subs = Get-AzSubscription
foreach ($sub in $subs)
{
    Write-Host $sub.Name "("  $sub.Id ")"
    Write-Host "`tTenant Id" $sub.TenantId
}
```

## <a name="create-a-data-lake-analytics-account-using-a-template"></a>Создание учетной записи Data Lake Analytics с помощью шаблона

Вы также можете применить шаблон группы ресурсов Azure, используя следующий пример: [Create a Data Lake Analytics account using a template](https://github.com/Azure-Samples/data-lake-analytics-create-account-with-arm-template) (Создание учетной записи Data Lake Analytics с помощью шаблона).

## <a name="next-steps"></a>Дальнейшие действия
* [Обзор аналитики озера данных Microsoft Azure](data-lake-analytics-overview.md)
* Приступая к работе с Data Lake Analytics с помощью Azure PowerShell [портал Azure](data-lake-analytics-get-started-portal.md)  |  [](data-lake-analytics-get-started-powershell.md)  |  [Azure CLI](data-lake-analytics-get-started-cli.md)
* Управление Azure Data Lake Analytics с помощью [портал Azure](data-lake-analytics-manage-use-portal.md)  |  [Azure PowerShell](data-lake-analytics-manage-use-powershell.md)  |  [CLI](data-lake-analytics-manage-use-cli.md)
