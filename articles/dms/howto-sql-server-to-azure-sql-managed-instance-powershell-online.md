---
title: 'PowerShell: миграция SQL Server в SQL Управляемый экземпляр Online'
titleSuffix: Azure Database Migration Service
description: Сведения о переходе в оперативный режим с SQL Server на Управляемый экземпляр SQL Azure с помощью Azure PowerShell и Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: ''
ms.topic: how-to
ms.date: 12/16/2020
ms.openlocfilehash: 7f09db2e1f98d48e91dfea2642969ff4ca360967
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98697830"
---
# <a name="migrate-sql-server-to-sql-managed-instance-online-with-powershell--azure-database-migration-service"></a>Миграция SQL Server в SQL Управляемый экземпляр в сети с помощью PowerShell & Azure Database Migration Service

В этой статье вы перенесите базу данных **Adventureworks2016** , восстановленную в локальный экземпляр SQL Server 2005 или более поздней версии, в управляемый экземпляр SQL Azure с помощью Microsoft Azure PowerShell. Можно выполнить миграцию баз данных из SQL Server экземпляра в Управляемый экземпляр SQL с помощью `Az.DataMigration` модуля в Microsoft Azure PowerShell.

Вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
>
> * Создайте группу ресурсов.
> * Создайте экземпляр Azure Database Migration Service.
> * Создайте проект миграции в экземпляре Azure Database Migration Service.
> * Запустите миграцию в режиме в сети.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

В этой статье приводятся шаги для оперативной миграции, но можно также выполнить миграцию в [автономном режиме](howto-sql-server-to-azure-sql-managed-instance-powershell-offline.md).


## <a name="prerequisites"></a>Предварительные условия

Для выполнения этих действий вам потребуется следующее:

* [SQL Server 2016 или более поздней](https://www.microsoft.com/sql-server/sql-server-downloads) версии (любой выпуск).
* Локальная копия базы данных **AdventureWorks2016** , доступная для скачивания [здесь](/sql/samples/adventureworks-install-configure).
* При установке SQL Server Express включите протокол TCP/IP, который отключен по умолчанию. Для этого выполните инструкции в разделе [Использование диспетчера конфигурации SQL Server](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol#SSMSProcedure) статьи "Включение или отключение сетевого протокола сервера".
* [Настройте брандмауэр Windows для доступа к ядру СУБД](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access).
* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начать работу.
* Управляемый экземпляр SQL. Вы можете создать Управляемый экземпляр SQL, следуя сведениям в статье [Создание аскл управляемый экземпляр](../azure-sql/managed-instance/instance-create-quickstart.md).
* Для загрузки и установки [Помощник по миграции данных](https://www.microsoft.com/download/details.aspx?id=53595) v 3.3 или более поздней версии.
* Виртуальная сеть Microsoft Azure, созданная с помощью модели развертывания Azure Resource Manager, которая предоставляет Azure Database Migration Service подключения типа "сеть — сеть" к локальным исходным серверам с помощью [ExpressRoute](../expressroute/expressroute-introduction.md) или [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* Завершенная оценка локальной базы данных и миграции схем с помощью Помощник по миграции данных, как описано в статье [выполнение SQL Serverной оценки миграции](/sql/dma/dma-assesssqlonprem).
* Чтобы скачать и установить `Az.DataMigration` модуль (0.7.2 или более поздней версии) из коллекция PowerShell с помощью [командлета PowerShell Install-Module](/powershell/module/powershellget/Install-Module).
* Чтобы убедиться, что учетные данные, используемые для подключения к исходному SQL Server экземпляру, имеют разрешение [Control Server](/sql/t-sql/statements/grant-server-permissions-transact-sql) .
* Чтобы гарантировать, что учетные данные, используемые для подключения к целевой базе данных SQL Управляемый экземпляр, имеют разрешение CONTROL DATABASE на целевые базы данных SQL Управляемый экземпляр.

    > [!IMPORTANT]
    > Для оперативной миграции необходимо предварительно настроить учетные данные Azure Active Directory. Дополнительные сведения см. в статье [Использование портала для создания приложения Azure AD и субъекта-службы, которые имеют доступ к ресурсам](../active-directory/develop/howto-create-service-principal-portal.md).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью [`New-AzResourceGroup`](/powershell/module/az.resources/new-azresourcegroup) команды.

В следующем примере создается группа ресурсов с именем *myResourceGroup* в регионе " *Восточная часть США* ".

```powershell
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

## <a name="create-an-instance-of-dms"></a>Создание экземпляра DMS

Вы можете создать экземпляр Azure Database Migration Service, выполнив командлет `New-AzDataMigrationService`.
Для этого командлета нужно передать следующие обязательные параметры:

* *Имя группы ресурсов Azure*. Вы можете использовать [`New-AzResourceGroup`](/powershell/module/az.resources/new-azresourcegroup) команду, чтобы создать группу ресурсов Azure, как показано выше, и указать ее имя в качестве параметра.
* *Имя службы*. Строка, соответствующая нужному уникальному имени службы для Azure Database Migration Service.
* *Расположение*. Указывает расположение службы. Укажите расположение центра обработки данных Azure, например "Западная часть США" или "Юго-Восточная Азия".
* *Номер SKU*. Этот параметр соответствует имени SKU DMS. В настоящее время поддерживаются имена SKU *Basic_1vCore*, *Basic_2vCores*, *GeneralPurpose_4vCores*.
* *VirtualSubnetId.* [`New-AzVirtualNetworkSubnetConfig`](/powershell/module/az.network/new-azvirtualnetworksubnetconfig)Для создания подсети можно использовать командлет.

В следующем примере создается служба с именем *mydms* в группе ресурсов *MyDMSResourceGroup* , расположенной в регионе *восточной части США* , с использованием виртуальной сети с именем *MyVNET* и подсети *MySubnet*.


```powershell
$vNet = Get-AzVirtualNetwork -ResourceGroupName MyDMSResourceGroup -Name MyVNET

$vSubNet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vNet -Name MySubnet

$service = New-AzDms -ResourceGroupName myResourceGroup `
  -ServiceName MyDMS `
  -Location EastUS `
  -Sku Basic_2vCores `  
  -VirtualSubnetId $vSubNet.Id`
```

## <a name="create-a-migration-project"></a>Создание проекта миграции

Создав экземпляр Azure Database Migration Service, создайте проект миграции. Для проекта Azure Database Migration Service необходимо иметь сведения о подключении для исходных и целевых экземпляров, а также список баз данных, которые требуется перенести как часть проекта.
Определите строки подключения исходной и целевой подключений.

Следующий скрипт определяет сведения о подключении SQL Server источника: 

```powershell
# Source connection properties
$sourceDataSource = "<mysqlserver.domain.com/privateIP of source SQL>"
$sourceUserName = "domain\user"
$sourcePassword = "mypassword"
```

В следующем скрипте определяются сведения о подключении к целевому SQL Управляемый экземпляр. 

```powershell
# Target MI connection properties
$targetMIResourceId = "/subscriptions/<subid>/resourceGroups/<rg>/providers/Microsoft.Sql/managedInstances/<myMI>"
$targetUserName = "<user>"
$targetPassword = "<password>"
```



### <a name="define-source-and-target-database-mapping"></a>Определение сопоставления исходной и целевой баз данных

Укажите базы данных для переноса в этом проекте миграции

Следующий сценарий сопоставляет базу данных источника с соответствующей новой базой данных в целевом SQL Управляемый экземпляр с указанным именем.

```powershell
# Selected databases (Source database name to target database name mapping)
$selectedDatabasesMap = New-Object System.Collections.Generic.Dictionary"[String,String]" 
$selectedDatabasesMap.Add("<source  database name>", "<target database name> ")
```

Для нескольких баз данных добавьте список баз данных к указанному выше сценарию в следующем формате:

```powershell
$selectedDatabasesMap = New-Object System.Collections.Generic.Dictionary"[String,String]" 
$selectedDatabasesMap.Add("<source  database name1>", "<target database name1> ")
$selectedDatabasesMap.Add("<source  database name2>", "<target database name2> ")
```

### <a name="create-dms-project"></a>Создание проекта DMS

Проект Azure Database Migration Service можно создать в экземпляре DMS.

```powershell
# Create DMS project
$project = New-AzDataMigrationProject `
  -ResourceGroupName $dmsResourceGroupName `
  -ServiceName $dmsServiceName `
  -ProjectName $dmsProjectName `
  -Location $dmsLocation `
  -SourceType SQL `
  -TargetType SQLMI `

# Create selected databases object
$selectedDatabases = @();
foreach ($sourceDbName in $selectedDatabasesMap.Keys){
    $targetDbName = $($selectedDatabasesMap[$sourceDbName])
    $selectedDatabases += New-AzDmsSelectedDB -MigrateSqlServerSqlDbMi `
      -Name $sourceDbName `
      -TargetDatabaseName $targetDbName `
      -BackupFileShare $backupFileShare `
}
```



### <a name="create-a-backup-fileshare-object"></a>Создание объекта общей папки для архивации

Теперь создайте объект общего ресурса, представляющий локальную сетевую папку SMB, в которую Azure Database Migration Service может принимать резервные копии базы данных-источника с помощью командлета New-AzDmsFileShare.

```powershell
# SMB Backup share properties
$smbBackupSharePath = "\\shareserver.domain.com\mybackup"
$smbBackupShareUserName = "domain\user"
$smbBackupSharePassword = "<password>"

# Create backup file share object
$smbBackupSharePasswordSecure = ConvertTo-SecureString -String $smbBackupSharePassword -AsPlainText -Force
$smbBackupShareCredentials = New-Object System.Management.Automation.PSCredential ($smbBackupShareUserName, $smbBackupSharePasswordSecure)
$backupFileShare = New-AzDmsFileShare -Path $smbBackupSharePath -Credential $smbBackupShareCredentials
```

### <a name="define-the-azure-storage"></a>Определение службы хранилища Azure 

Выберите контейнер хранилища Azure, который будет использоваться для миграции: 

```powershell
# Storage resource id
$storageAccountResourceId = "/subscriptions/<subscriptionname>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<mystorage>"
```


### <a name="configure-azure-active-directory-app"></a>Настройка приложения Azure Active Directory

Укажите необходимые сведения для Azure Active Directory оперативной миграции Управляемый экземпляр SQL: 

```powershell
# AAD properties
$AADAppId = "<appid-guid>"
$AADAppKey = "<app-key>"

# Create AAD object
$AADAppKeySecure = ConvertTo-SecureString $AADAppKey -AsPlainText -Force
$AADApp = New-AzDmsAadApp -ApplicationId $AADAppId -AppKey $AADAppKeySecure
```


## <a name="create-and-start-a-migration-task"></a>Создание и запуск задачи миграции

Затем создайте и запустите задачу Azure Database Migration Service. Вызывайте исходный и целевой объекты с помощью переменных и перечислите таблицы базы данных, подлежат миграции: 


```powershell
# Managed Instance online migration properties
$dmsTaskName = "testmigration1"

# Create source connection info
$sourceConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource $sourceDataSource `
  -AuthType WindowsAuthentication `
  -TrustServerCertificate:$true
$sourcePasswordSecure = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCredentials = New-Object System.Management.Automation.PSCredential ($sourceUserName, $sourcePasswordSecure)

# Create target connection info
$targetConnInfo = New-AzDmsConnInfo -ServerType SQLMI `
    -MiResourceId $targetMIResourceId
$targetPasswordSecure = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCredentials = New-Object System.Management.Automation.PSCredential ($targetUserName, $targetPasswordSecure)
```

В следующем примере создается и запускается задача оперативной миграции. 

```powershell
# Create DMS migration task
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDbMiSync `
  -ResourceGroupName $dmsResourceGroupName `
  -ServiceName $dmsServiceName `
  -ProjectName $dmsProjectName `
  -TaskName $dmsTaskName `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCredentials `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCredentials `
  -SelectedDatabase  $selectedDatabases `
  -BackupFileShare $backupFileShare `
  -AzureActiveDirectoryApp $AADApp `
  -StorageResourceId $storageAccountResourceId
```

Дополнительные сведения см. в разделе [New-аздатамигратионтаск](/powershell/module/az.datamigration/new-azdatamigrationtask).

## <a name="monitor-the-migration"></a>Мониторинг миграции

Чтобы отслеживать миграцию, выполните следующие задачи.

### <a name="check-the-status-of-task"></a>Проверка состояния задачи

```powershell
# Get migration task status details
$migTask = Get-AzDataMigrationTask `
                    -ResourceGroupName $dmsResourceGroupName `
                    -ServiceName $dmsServiceName `
                    -ProjectName $dmsProjectName `
                    -Name $dmsTaskName `
                    -ResultType DatabaseLevelOutput `
                    -Expand

# Task state will be either of 'Queued', 'Running', 'Succeeded', 'Failed', 'FailedInputValidation' or 'Faulted'
$taskState = $migTask.ProjectTask.Properties.State

# Display task state
$taskState | Format-List
```

Для получения списка ошибок используйте следующую команду:

```powershell
# Get task errors
$taskErrors = $migTask.ProjectTask.Properties.Errors

# Display task errors
foreach($taskError in $taskErrors){
    $taskError |  Format-List
}


# Get database level details
$databaseLevelOutputs = $migTask.ProjectTask.Properties.Output

# Display database level details
foreach($databaseLevelOutput in $databaseLevelOutputs){

    # This is the source database name.
    $databaseName = $databaseLevelOutput.SourceDatabaseName;

    Write-Host "=========="
    Write-Host "Start migration details for database " $databaseName
    # This is the status for that database - It will be either of:
    # INITIAL, FULL_BACKUP_UPLOADING, FULL_BACKUP_UPLOADED, LOG_FILES_UPLOADING,
    # CUTOVER_IN_PROGRESS, CUTOVER_INITIATED, CUTOVER_COMPLETED, COMPLETED, CANCELLED, FAILED
    $databaseMigrationState = $databaseLevelOutput.MigrationState;

    # Details about last restored backup. This contains file names, LSN, backup date, etc 
    $databaseLastRestoredBackup = $databaseLevelOutput.LastRestoredBackupSetInfo
        
    # Details about last restored backup. This contains file names, LSN, backup date, etc 
    $databaseLastRestoredBackup = $databaseLevelOutput.LastRestoredBackupSetInfo

    # Details about last Currently active/most recent backups. This contains file names, LSN, backup date, etc 
    $databaseActiveBackpusets = $databaseLevelOutput.ActiveBackupSets

    # Display info
    $databaseLevelOutput | Format-List

    Write-Host "Currently active/most recent backupset details:"
    $databaseActiveBackpusets  | select BackupStartDate, BackupFinishedDate, FirstLsn, LastLsn -ExpandProperty ListOfBackupFiles | Format-List

    Write-Host "Last restored backupset details:"
    $databaseLastRestoredBackupFiles  | Format-List

    Write-Host "End migration details for database " $databaseName
    Write-Host "=========="
}
```

## <a name="performing-the-cutover"></a>Выполнение прямую миграцию 

При оперативной миграции выполняется полное резервное копирование и восстановление баз данных, после чего работа продолжается при восстановлении журналов транзакций, хранящихся в Баккупфилешаре.

Когда база данных в Управляемый экземпляр Azure SQL обновляется последними данными и синхронизируется с базой данных-источником, можно выполнить прямую миграцию.

В следующем примере выполняется завершение кутовер\мигратион. Пользователи вызывают эту команду по собственному усмотрению.

```powershell
$command = Invoke-AzDmsCommand -CommandType CompleteSqlMiSync `
                               -ResourceGroupName myResourceGroup `
                               -ServiceName $service.Name `
                               -ProjectName $project.Name `
                               -TaskName myDMSTask `
                               -DatabaseName "Source DB Name"
```

## <a name="deleting-the-instance-of-azure-database-migration-service"></a>Удаление экземпляра Azure Database Migration Service

После завершения миграции можно удалить экземпляр Azure Database Migration Service.

```powershell
Remove-AzDms -ResourceGroupName myResourceGroup -ServiceName MyDMS
```

## <a name="additional-resources"></a>Дополнительные ресурсы

Дополнительные сведения о дополнительных сценариях миграции (пары "источник — Целевая версия") см. в разделе [руководств по миграции баз данных](https://datamigration.microsoft.com/)Майкрософт.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о Azure Database Migration Service в статье [что такое Azure Database Migration Service?](./dms-overview.md).