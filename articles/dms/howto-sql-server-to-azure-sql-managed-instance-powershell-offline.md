---
title: 'PowerShell: миграция SQL Server в SQL Управляемый экземпляр в автономном режиме'
titleSuffix: Azure Database Migration Service
description: Сведения о переходе в автономный режим с SQL Server в Azure SQL Управляемый экземпляр с помощью Azure PowerShell и Azure Database Migration Service.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019,fasttrack-edit, devx-track-azurepowershell
ms.topic: how-to
ms.date: 12/16/2020
ms.openlocfilehash: 90663b6beb4f1e3f7ade32e603a53c8b9d9158f5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98697854"
---
# <a name="migrate-sql-server-to-sql-managed-instance-offline-with-powershell--azure-database-migration-service"></a>Миграция SQL Server в SQL Управляемый экземпляр в автономном режиме с помощью PowerShell & Azure Database Migration Service

В этой статье автономная миграция базы данных **Adventureworks2016** , восстановленной в локальном экземпляре SQL Server 2005 или более поздней версии, в управляемый экземпляр SQL Azure SQL с помощью Microsoft Azure PowerShell. Можно выполнить миграцию баз данных из SQL Server экземпляра в Управляемый экземпляр SQL с помощью `Az.DataMigration` модуля в Microsoft Azure PowerShell.

Вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
>
> * Создайте группу ресурсов.
> * Создайте экземпляр Azure Database Migration Service.
> * Создайте проект миграции в экземпляре Azure Database Migration Service.
> * Запустите миграцию в автономном режиме.

[!INCLUDE [online-offline](../../includes/database-migration-service-offline-online.md)]

Эта статья содержит шаги для автономной миграции, но ее также можно перенести в режим «в [сети](howto-sql-server-to-azure-sql-managed-instance-powershell-online.md)».


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


## <a name="sign-in-to-your-microsoft-azure-subscription"></a>Вход в подписку Microsoft Azure

Войдите в подписку Azure с помощью PowerShell. Дополнительные сведения см. в статье [Вход с помощью Azure PowerShell](/powershell/azure/authenticate-azureps).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью [`New-AzResourceGroup`](/powershell/module/az.resources/new-azresourcegroup) команды.

В следующем примере создается группа ресурсов с именем *myResourceGroup* в регионе " *Восточная часть США* ".

```powershell
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

## <a name="create-an-instance-of-azure-database-migration-service"></a>Создание экземпляра Azure Database Migration Service

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

### <a name="create-a-database-connection-info-object-for-the-source-and-target-connections"></a>Создание объекта сведений о подключении к базе данных для исходных и целевых подключений

Объект сведений о подключении к базе данных можно создать с помощью `New-AzDmsConnInfo` командлета, который предполагает наличие следующих параметров:

* *ServerType*. Тип запрошенного подключения к базе данных, например SQL, Oracle или MySQL. Используйте SQL для SQL Server и SQL Azure.
* *Источник данных*. Имя или IP-адрес экземпляра SQL Server или экземпляра базы данных SQL Azure.
* *AuthType*. Тип проверки подлинности для подключения. Это может быть SqlAuthentication или WindowsAuthentication.
* *TrustServerCertificate*. Этот параметр задает значение, указывающее, шифруется ли канал при прохождении обхода цепочки сертификатов для проверки доверия. Значением может быть `$true` или `$false`.

В следующем примере создается объект сведений о соединении для источника SQL Server с именем *мисаурцесклсервер* с использованием проверки подлинности SQL:

```powershell
$sourceConnInfo = New-AzDmsConnInfo -ServerType SQL `
  -DataSource MySourceSQLServer `
  -AuthType SqlAuthentication `
  -TrustServerCertificate:$true
```

В следующем примере показано создание сведений о подключении для Управляемый экземпляр Azure SQL с именем "таржетманажединстанце":

```powershell
$targetResourceId = (Get-AzSqlInstance -Name "targetmanagedinstance").Id
$targetConnInfo = New-AzDmsConnInfo -ServerType SQLMI -MiResourceId $targetResourceId
```

### <a name="provide-databases-for-the-migration-project"></a>Указание баз данных для проекта миграции

Создайте список `AzDataMigrationDatabaseInfo` объектов, указывающих базы данных в составе проекта Azure Database Migration Service, который можно указать в качестве параметра для создания проекта. Для создания можно использовать командлет `New-AzDataMigrationDatabaseInfo` `AzDataMigrationDatabaseInfo` .

В следующем примере создается `AzDataMigrationDatabaseInfo` проект для базы данных **AdventureWorks2016** и добавляется в список, который предоставляется в качестве параметра для создания проекта.

```powershell
$dbInfo1 = New-AzDataMigrationDatabaseInfo -SourceDatabaseName AdventureWorks
$dbList = @($dbInfo1)
```

### <a name="create-a-project-object"></a>Создание объекта проекта

Наконец, можно создать Azure Database Migration Service проект с именем *мидмспрожект* , расположенный в *восточной части США* , с помощью `New-AzDataMigrationProject` и добавить ранее созданное исходное и целевое подключения и список баз данных для миграции.

```powershell
$project = New-AzDataMigrationProject -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName MyDMSProject `
  -Location EastUS `
  -SourceType SQL `
  -TargetType SQLMI `
  -SourceConnection $sourceConnInfo `
  -TargetConnection $targetConnInfo `
  -DatabaseInfo $dbList
```

## <a name="create-and-start-a-migration-task"></a>Создание и запуск задачи миграции

Затем создайте и запустите задачу Azure Database Migration Service. Для этой задачи требуются учетные данные подключения как для источника, так и для целевого объекта, а также список таблиц базы данных для переноса и сведения, уже предоставленные вместе с проектом, созданным в качестве необходимого компонента.

### <a name="create-credential-parameters-for-source-and-target"></a>Создание параметров учетных данных для исходного и целевого подключения

Создайте учетные данные безопасности подключения в виде объекта [PSCredential](/dotnet/api/system.management.automation.pscredential) .

В следующем примере показано создание объектов *PSCredential* как для исходных, так и для целевых подключений, предоставляя пароли как строковые переменные *$sourcePassword* и *$targetPassword*.

```powershell
$secpasswd = ConvertTo-SecureString -String $sourcePassword -AsPlainText -Force
$sourceCred = New-Object System.Management.Automation.PSCredential ($sourceUserName, $secpasswd)
$secpasswd = ConvertTo-SecureString -String $targetPassword -AsPlainText -Force
$targetCred = New-Object System.Management.Automation.PSCredential ($targetUserName, $secpasswd)
```

### <a name="create-a-backup-fileshare-object"></a>Создание объекта общей папки для архивации

Теперь создайте объект общей папки, представляющий локальную сетевую папку SMB, в которую Azure Database Migration Service может принимать резервные копии базы данных-источника с помощью `New-AzDmsFileShare` командлета.

```powershell
$backupPassword = ConvertTo-SecureString -String $password -AsPlainText -Force
$backupCred = New-Object System.Management.Automation.PSCredential ($backupUserName, $backupPassword)

$backupFileSharePath="\\10.0.0.76\SharedBackup"
$backupFileShare = New-AzDmsFileShare -Path $backupFileSharePath -Credential $backupCred
```

### <a name="create-selected-database-object"></a>Создание объекта выбранной базы данных

Следующим шагом является выбор исходной и целевой баз данных с помощью `New-AzDmsSelectedDB` командлета.

Следующий пример предназначен для миграции одной базы данных из SQL Server в Управляемый экземпляр SQL Azure:

```powershell
$selectedDbs = @()
$selectedDbs += New-AzDmsSelectedDB -MigrateSqlServerSqlDbMi `
  -Name AdventureWorks2016 `
  -TargetDatabaseName AdventureWorks2016 `
  -BackupFileShare $backupFileShare `
```

Если в Управляемый экземпляр SQL Azure весь экземпляр SQL Server нуждается в точности и сдвиге, то для получения всех баз данных из источника следует использовать цикл. В следующем примере для $Server, $SourceUserName и $SourcePassword укажите сведения об источнике SQL Server.

```powershell
$Query = "(select name as Database_Name from master.sys.databases where Database_id>4)";
$Databases= (Invoke-Sqlcmd -ServerInstance "$Server" -Username $SourceUserName
-Password $SourcePassword -database master -Query $Query)
$selectedDbs=@()
foreach($DataBase in $Databases.Database_Name)
    {
      $SourceDB=$DataBase
      $TargetDB=$DataBase
      
$selectedDbs += New-AzureRmDmsSelectedDB -MigrateSqlServerSqlDbMi `
                                              -Name $SourceDB `
                                              -TargetDatabaseName $TargetDB `
                                              -BackupFileShare $backupFileShare
      }
```

### <a name="sas-uri-for-azure-storage-container"></a>Универсальный код ресурса (URI) SAS для контейнера службы хранилища Azure

Создайте переменную, содержащую URI SAS, который позволяет Azure Database Migration Service обращаться к контейнеру учетной записи хранения для отправки файлов резервных копий.

```powershell
$blobSasUri="https://mystorage.blob.core.windows.net/test?st=2018-07-13T18%3A10%3A33Z&se=2019-07-14T18%3A10%3A00Z&sp=rwdl&sv=2018-03-28&sr=c&sig=qKlSA512EVtest3xYjvUg139tYSDrasbftY%3D"
```

> [!NOTE]
> Azure Database Migration Service не поддерживает использование маркера SAS уровня учетной записи. Для контейнера учетной записи хранения необходимо использовать универсальный код ресурса (URI) SAS. [Получение SAS для контейнера больших двоичных объектов](../vs-azure-tools-storage-explorer-blobs.md#get-the-sas-for-a-blob-container).

### <a name="additional-configuration-requirements"></a>Дополнительные требования к конфигурации

Существует несколько дополнительных требований, которые необходимо решить:


* **Выберите имена входа**. Создайте список имен входа для переноса, как показано в следующем примере:

    ```powershell
    $selectedLogins = @("user1", "user2")
    ```

    > [!IMPORTANT]
    > В настоящее время Azure Database Migration Service поддерживает только миграцию имен входа SQL.

* **Выберите задания агента**. Создайте список заданий агента для переноса, как показано в следующем примере:

    ```powershell
    $selectedAgentJobs = @("agentJob1", "agentJob2")
    ```

    > [!IMPORTANT]
    > В настоящее время Azure Database Migration Service поддерживает только задания с шагами заданий подсистемы T-SQL.



### <a name="create-and-start-the-migration-task"></a>Создание и запуск задачи миграции

Выполните командлет `New-AzDataMigrationTask`, чтобы создать и запустить задачу миграции.

#### <a name="specify-parameters"></a>Указание параметров

`New-AzDataMigrationTask`Командлету требуются следующие параметры:

* *TaskType.* Тип задачи миграции, создаваемой для SQL Server в Azure SQL Управляемый экземпляр тип миграции *мигратесклсерверсклдбми* . 
* *Имя группы ресурсов*. Имя группы ресурсов Azure, в которой будет создана задача.
* *ServiceName*. Экземпляр Azure Database Migration Service, в котором будет создана задача.
* *Имя_проекта*. Имя проекта Azure Database Migration Service, в котором будет создана задача. 
* *Имя_задания*. Имя задачи, которая будет создана. 
* *SourceConnection*. Объект Аздмсконнинфо, представляющий соединение с источником SQL Server.
* *TargetConnection.* Объект Аздмсконнинфо, представляющий целевое подключение Управляемый экземпляр SQL Azure.
* *SourceCred.* Объект [PSCredential](/dotnet/api/system.management.automation.pscredential) для подключения к исходному серверу.
* *TargetCred.* Объект [PSCredential](/dotnet/api/system.management.automation.pscredential) для подключения к целевому серверу.
* *SelectedDatabase.* Объект Аздатамигратионселектеддб, представляющий сопоставление исходного и целевого базы данных.
* *BackupFileShare.* Объект FileShare, представляющий локальную сетевую папку, в которую Azure Database Migration Service может поместить резервные копии базы данных-источника.
* *BackupBlobSasUri.* URI SAS, который позволяет Azure Database Migration Service обращаться к контейнеру учетной записи хранения для отправки файлов резервных копий. Узнайте, как получить URI SAS для контейнера больших двоичных объектов.
* *Значение selectedlogins*. Список выбранных для переноса имен для входа.
* *SelectedAgentJobs.* Список выбранных заданий агента для переноса.
* *Значение selectedlogins*. Список выбранных для переноса имен для входа.
* *SelectedAgentJobs.* Список выбранных заданий агента для переноса.



#### <a name="create-and-start-a-migration-task"></a>Создание и запуск задачи миграции

В следующем примере создается и запускается задача автономной миграции с именем **мидмстаск**:

```powershell
$migTask = New-AzDataMigrationTask -TaskType MigrateSqlServerSqlDbMi `
  -ResourceGroupName myResourceGroup `
  -ServiceName $service.Name `
  -ProjectName $project.Name `
  -TaskName myDMSTask `
  -SourceConnection $sourceConnInfo `
  -SourceCred $sourceCred `
  -TargetConnection $targetConnInfo `
  -TargetCred $targetCred `
  -SelectedDatabase  $selectedDbs `
  -BackupFileShare $backupFileShare `
  -BackupBlobSasUri $blobSasUri `
  -SelectedLogins $selectedLogins `
  -SelectedAgentJobs $selectedJobs `
```


## <a name="monitor-the-migration"></a>Мониторинг миграции

Чтобы отслеживать миграцию, выполните следующие задачи.

1. Объедините все сведения о миграции в переменную с именем $CheckTask.

    Чтобы объединить сведения о миграции, такие как свойства, состояние и сведения о базе данных, связанные с миграцией, используйте следующий фрагмент кода:

    ```powershell
    $CheckTask= Get-AzDataMigrationTask     -ResourceGroupName myResourceGroup `
                                            -ServiceName $service.Name `
                                        -ProjectName $project.Name `
                                            -Name myDMSTask `
                                            -ResultType DatabaseLevelOutput `
                        -Expand 
    Write-Host ‘$CheckTask.ProjectTask.Properties.Output’
    ```

2. Используйте `$CheckTask` переменную для получения текущего состояния задачи миграции.

    Чтобы использовать `$CheckTask` переменную для получения текущего состояния задачи миграции, можно отслеживать выполнение задачи миграции, запросив свойство State задачи, как показано в следующем примере:

    ```powershell
    if (($CheckTask.ProjectTask.Properties.State -eq "Running") -or ($CheckTask.ProjectTask.Properties.State -eq "Queued"))
    {
      Write-Host "migration task running"
    }
    else if($CheckTask.ProjectTask.Properties.State -eq "Succeeded")
    { 
      Write-Host "Migration task is completed Successfully"
    }
    else if($CheckTask.ProjectTask.Properties.State -eq "Failed" -or $CheckTask.ProjectTask.Properties.State -eq "FailedInputValidation" -or $CheckTask.ProjectTask.Properties.State -eq "Faulted")
    { 
      Write-Host "Migration Task Failed"
    }
    ```


## <a name="delete-the-instance-of-azure-database-migration-service"></a>Удаление экземпляра Azure Database Migration Service

После завершения миграции можно удалить экземпляр Azure Database Migration Service.

```powershell
Remove-AzDms -ResourceGroupName myResourceGroup -ServiceName MyDMS
```


## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о Azure Database Migration Service в статье [что такое Azure Database Migration Service?](./dms-overview.md).

Дополнительные сведения о дополнительных сценариях миграции (пары "источник — Целевая версия") см. в разделе [руководств по миграции баз данных](https://datamigration.microsoft.com/)Майкрософт.