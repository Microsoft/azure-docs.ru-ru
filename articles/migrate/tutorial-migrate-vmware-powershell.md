---
title: Миграция виртуальных машин VMware в Azure (без агента) — PowerShell
description: Узнайте, как выполнить миграцию виртуальных машин VMware с помощью службы "Миграция Azure" без агента, используя PowerShell.
author: rahulg1190
ms.author: rahugup
manager: bsiva
ms.topic: tutorial
ms.date: 03/02/2021
ms.openlocfilehash: 24dd33495915a9f4d47a00fbbfe9e894df839d4d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101715077"
---
# <a name="migrate-vmware-vms-to-azure-agentless---powershell"></a>Миграция виртуальных машин VMware в Azure (без агента) — PowerShell

В этой статье вы узнаете, как перенести обнаруженные виртуальные машины VMware без агента, используя Azure PowerShell для средства [Миграция Azure: Server Migration](migrate-services-overview.md#azure-migrate-server-migration-tool) (Миграция Azure: миграция сервера).

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Извлечение обнаруженных виртуальных машин VMware в проекте Миграции Azure.
> * Запуск репликации виртуальных машин.
> * Обновление свойств, выполняющих репликацию виртуальных машин.
> * Мониторинг репликации.
> * Выполнение тестовой миграции, позволяющей убедиться в правильной работе всех компонентов.
> * Выполнение полной миграции виртуальной машины.

> [!NOTE]
> В руководствах показан простейший путь развертывания сценария, использование которого позволяет быстро настроить проверку концепции. В них используются значения по умолчанию (при возможности) и описаны только некоторые возможные параметры и пути.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/), прежде чем начинать работу.

## <a name="1-prerequisites"></a>1. Предварительные требования

Для работы с этим руководством вам потребуется:

1. Ознакомьтесь со статьей [Руководство. Обнаружение виртуальных машин VMware с помощью средства оценки серверов](tutorial-discover-vmware.md), чтобы подготовить Azure и VMware к миграции.
2. Ознакомьтесь со статьей [Руководство. Оценка готовности виртуальных машин VMware к переносу на виртуальные машины Azure](./tutorial-assess-vmware-azure-vm.md), прежде чем выполнять миграцию в Azure.
3. [Установите модуль Az PowerShell.](/powershell/azure/install-az-ps)

## <a name="2-install-azure-migrate-powershell-module"></a>2. Установка модуля PowerShell для Миграции Azure

Модуль PowerShell для Миграции Azure доступен в составе Azure PowerShell (`Az`). Выполните команду `Get-InstalledModule -Name Az.Migrate`, чтобы проверить, установлен ли на компьютере модуль PowerShell для Миграции Azure.  

## <a name="3-sign-in-to-your-microsoft-azure-subscription"></a>3. Вход в подписку Microsoft Azure

Войдите в подписку Azure с помощью командлета [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

```azurepowershell-interactive
Connect-AzAccount
```

### <a name="select-your-azure-subscription"></a>Выбор подписки Azure

Используйте командлет [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription), чтобы получить список подписок Azure, к которым у вас есть доступ. Выберите подписку Azure, в которой находится необходимый проект Миграции Azure, с помощью командлета [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="4-retrieve-the-azure-migrate-project"></a>4. Извлечение проекта Миграции Azure

Проект Миграции Azure используется для хранения метаданных обнаружения, оценки и миграции, собранных из среды, для которой выполняется оценка или миграция.
В проекте можно отслеживать обнаруженные ресурсы, оркестрировать оценки и выполнять миграции.

Проект Миграции Azure уже был создан при выполнении предварительных требований. Используйте командлет [Get-AzMigrateProject](/powershell/module/az.migrate/get-azmigrateproject), чтобы получить сведения о проекте Миграции Azure. Необходимо указать имя проекта Миграции Azure (`Name`) и имя группы ресурсов этого проекта (`ResourceGroupName`).

```azurepowershell-interactive
# Get resource group of the Azure Migrate project
$ResourceGroup = Get-AzResourceGroup -Name MyResourceGroup

# Get details of the Azure Migrate project
$MigrateProject = Get-AzMigrateProject -Name MyMigrateProject -ResourceGroupName $ResourceGroup.ResourceGroupName

# View Azure Migrate project details
Write-Output $MigrateProject
```

## <a name="5-retrieve-discovered-vms-in-an-azure-migrate-project"></a>5. Извлечение обнаруженных виртуальных машин в проекте Миграции Azure

Служба "Миграция Azure" использует упрощенное [устройство Миграции Azure](migrate-appliance-architecture.md). При выполнении предварительных требований вы развернули устройство Миграции Azure в качестве виртуальной машины VMware.

Чтобы извлечь конкретную виртуальную машину VMware в проекте Миграции Azure, укажите имя проекта Миграции Azure (`ProjectName`), группу ресурсов этого проекта (`ResourceGroupName`) и имя виртуальной машины (`DisplayName`).


```azurepowershell-interactive
# Get a specific VMware VM in an Azure Migrate project
$DiscoveredServer = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -DisplayName MyTestVM | Format-Table DisplayName, Name, Type

# View discovered server details
Write-Output $DiscoveredServer
```

Из этого учебника вы узнаете, как выполнить миграцию такой виртуальной машины.

Вы также можете извлечь все виртуальные машины VMware в проекте Миграции Azure с помощью параметров (`ProjectName`) и (`ResourceGroupName`).

```azurepowershell-interactive
# Get all VMware VMs in an Azure Migrate project
$DiscoveredServers = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName
```

Если в проекте Миграции Azure имеется несколько устройств, можно воспользоваться параметрами (`ProjectName`), (`ResourceGroupName`) и (`ApplianceName`), чтобы получить все виртуальные машины, обнаруженные с помощью определенного устройства Миграции Azure.

```azurepowershell-interactive
# Get all VMware VMs discovered by an Azure Migrate Appliance in an Azure Migrate project
$DiscoveredServers = Get-AzMigrateDiscoveredServer -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -ApplianceName MyMigrateAppliance

```

## <a name="6-initialize-replication-infrastructure"></a>6. Инициализация инфраструктуры репликации

[Azure Migrate: средство переноса сервера](migrate-services-overview.md#azure-migrate-server-migration-tool) использует несколько ресурсов Azure для миграции виртуальных машин. Оно автоматически подготавливает эти ресурсы в той же группе ресурсов, что и проект.

- **Служебная шина** Средство переноса сервера использует служебную шину для отправки на устройство сообщений оркестрации репликации.
- **Учетная запись хранения шлюза**. Миграция сервера использует учетную запись шлюза для хранения информации о состоянии реплицируемых виртуальных машин.
- **Учетная запись хранения журналов**. Устройство Миграции Azure загружает журналы репликации для виртуальных машин в учетную запись хранения журналов. Миграция Azure применяет сведения о репликации к управляемым дискам реплики.
- **Хранилище ключей**. Устройство Миграции Azure использует хранилище ключей для управления строками подключения для служебной шины и ключами доступа для учетных записей хранения, используемых при репликации.

Перед репликацией первой виртуальной машины в проекте Миграции Azure выполните следующую команду, чтобы подготовить инфраструктуру репликации. Эта команда подготавливает и настраивает упомянутые выше ресурсы для запуска миграции виртуальных машин VMware.

> [!NOTE]
> Один проект Миграции Azure поддерживает миграцию только в один регион Azure. После выполнения этого скрипта целевой регион, в который будут перенесены виртуальные машины VMware, изменить нельзя.
> При настройке нового устройства в проекте Миграции Azure необходимо выполнить команду `Initialize-AzMigrateReplicationInfrastructure`.

Из этой статьи вы узнаете, как инициализировать инфраструктуру репликации, чтобы перенести виртуальные машины в регион `Central US`.

```azurepowershell-interactive
# Initialize replication infrastructure for the current Migrate project
Initialize-AzMigrateReplicationInfrastructure -ResourceGroupName $ResourceGroup.ResourceGroupName -ProjectName $MigrateProject. Name -Scenario agentlessVMware -TargetRegion "CentralUS" 

```

## <a name="7-replicate-vms"></a>7. Репликация виртуальных машин

После завершения обнаружения и инициализации инфраструктуры репликации можно начать репликацию виртуальных машин VMware в Azure. Одновременно можно выполнять до 500 репликаций.

Свойства репликации можно задать следующим образом.

- **Целевая подписка и группа ресурсов**. Укажите подписку и группу ресурсов, в которую следует перенести виртуальную машину. Для этого задайте идентификатор группы ресурсов с помощью параметра (`TargetResourceGroupId`).
- **Целевая виртуальная сеть и подсеть**. С помощью параметров (`TargetNetworkId`) и (`TargetSubnetName`) укажите, соответственно, идентификатор виртуальной сети Azure и имя подсети, в которую должна быть перенесена виртуальная машина.
- **Имя целевой виртуальной машины**. С помощью параметра (`TargetVMName`) укажите имя виртуальной машины Azure, которую следует создать.
- **Размер целевой виртуальной машины**. С помощью параметра (`TargetVMSize`) укажите размер виртуальной машины Azure, который будет использоваться для репликации виртуальной машины. Например, чтобы перенести виртуальную машину в D2_v2 в Azure, укажите для (`TargetVMSize`) значение "Standard_D2_v2".
- **Лицензия**. Чтобы применить Преимущество гибридного использования Azure к компьютерам с Windows Server, включенным в активные программы Software Assurance или подписки Windows Server, укажите для параметра (`LicenseType`) значение **WindowsServer**. В противном случае укажите для параметра (`LicenseType`) значение "NoLicenseType".
- **Диск ОС**. Укажите уникальный идентификатор диска с загрузчиком операционной системы и установщиком. Используемый идентификатор диска — это свойство, определяющее уникальный идентификатор (UUID) для диска, извлеченного с помощью командлета [Get-AzMigrateDiscoveredServer](/powershell/module/az.migrate/get-azmigratediscoveredserver).
- **Тип диска**. Укажите значение для параметра (`DiskType`), как показано ниже.
    - Чтобы использовать управляемые диски уровня "Премиум", укажите для параметра (`DiskType`) значение "Premium_LRS".
    - Чтобы использовать стандартные диски SSD, укажите для параметра (`DiskType`) значение "StandardSSD_LRS".
    - Чтобы использовать стандартные диски HDD, укажите для параметра (`DiskType`) значение "Standard_LRS".
- **Избыточность инфраструктуры**. Задайте параметр избыточности инфраструктуры так, как описано ниже.
    - "Зона доступности", чтобы закрепить перенесенный компьютер в определенной зоне доступности в регионе. Используйте этот параметр для распределения серверов, образующих уровень приложения с несколькими узлами, по зонам доступности. Этот параметр доступен только в том случае, если в целевом регионе, выбранном для миграции, поддерживаются Зоны доступности. Чтобы использовать зоны доступности, укажите зону доступности в качестве значения параметра (`TargetAvailabilityZone`).
    - "Группа доступности", чтобы поместить перенесенную виртуальную машину в группу доступности. Чтобы использовать этот параметр, выбранная целевая группа ресурсов должна содержать одну или несколько групп доступности. Чтобы использовать группу доступности, укажите ее идентификатор в качестве значения параметра (`TargetAvailabilitySet`).
 - **Учетная запись хранения для диагностики загрузки**. Чтобы использовать учетную запись хранения для диагностики загрузки, укажите ее идентификатор в качестве значения параметра (`TargetBootDiagnosticStorageAccount`).
    -  Учетная запись хранения, используемая для диагностики загрузки, должна находиться в той же подписке, куда вы переносите виртуальные машины.  
    - По умолчанию значение для этого параметра не задано. 

### <a name="replicate-vms-with-all-disks"></a>Репликация виртуальных машин со всеми дисками

Из этого учебника вы узнаете, как реплицировать все диски обнаруженной виртуальной машины и указать новое имя для виртуальной машины в Azure. Первый диск обнаруженного сервера будет указан в качестве диска ОС. Все диски будут перенесены как HDD (цен. категория "Стандартный"). Диск ОС — это диск с загрузчиком операционной системы и установщиком. Командлет возвращает задание, которое можно отслеживать для мониторинга состояния операции.

```azurepowershell-interactive
# Retrieve the resource group that you want to migrate to
$TargetResourceGroup = Get-AzResourceGroup -Name MyTargetResourceGroup

# Retrieve the Azure virtual network and subnet that you want to migrate to
$TargetVirtualNetwork = Get-AzVirtualNetwork -Name MyVirtualNetwork

# Start replication for a discovered VM in an Azure Migrate project
$MigrateJob =  New-AzMigrateServerReplication -InputObject $DiscoveredServer -TargetResourceGroupId $TargetResourceGroup.ResourceId -TargetNetworkId $TargetVirtualNetwork.Id -LicenseType NoLicenseType -OSDiskID $DiscoveredServer.Disk[0].Uuid -TargetSubnetName $TargetVirtualNetwork.Subnets[0].Name -DiskType Standard_LRS -TargetVMName MyMigratedTestVM -TargetVMSize Standard_DS2_v2

# Track job status to check for completion
while (($MigrateJob.State -eq 'InProgress') -or ($MigrateJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $MigrateJob.State
```

### <a name="replicate-vms-with-select-disks"></a>Репликация виртуальных машин с выбранными дисками

Вы также можете выборочно реплицировать диски обнаруженной виртуальной машины. Для этого воспользуйтесь командлетом [New-AzMigrateDiskMapping](/powershell/module/az.migrate/new-azmigratediskmapping) и предоставьте его результат в качестве входных данных для параметра (`DiskToInclude`) в командлете [New-AzMigrateServerReplication](/powershell/module/az.migrate/new-azmigrateserverreplication). С помощью командлета [New-AzMigrateDiskMapping](/powershell/module/az.migrate/new-azmigratediskmapping) можно также отдельно указать разные типы целевых дисков для каждого реплицируемого диска.

Укажите значения для следующих параметров командлета [New-AzMigrateDiskMapping](/powershell/module/az.migrate/new-azmigratediskmapping).

- **DiskId**. Укажите уникальный идентификатор для диска, который требуется перенести. Используемый идентификатор диска — это свойство, определяющее уникальный идентификатор (UUID) для диска, извлеченного с помощью командлета [Get-AzMigrateDiscoveredServer](/powershell/module/az.migrate/get-azmigratediscoveredserver).
- **IsOSDisk**. Укажите значение true, если диск, который требуется перенести, является диском ОС виртуальной машины. В противном случае укажите false.
- **DiskType**. Укажите тип диска, который будет использоваться в Azure.

В следующем примере показано, как реплицировать только два диска обнаруженной виртуальной машины. Мы укажем диск ОС и будем использовать разные типы дисков для каждого реплицируемого диска. Командлет возвращает задание, которое можно отслеживать для мониторинга состояния операции.

```azurepowershell-interactive
# View disk details of the discovered server
Write-Output $DiscoveredServer.Disk

# Create a new disk mapping for the disks to be replicated
$DisksToReplicate = @()
$OSDisk = New-AzMigrateDiskMapping -DiskID $DiscoveredServer.Disk[0].Uuid -DiskType StandardSSD_LRS -IsOSDisk true
$DataDisk = New-AzMigrateDiskMapping -DiskID $DiscoveredServer.Disk[1].Uuid -DiskType Premium_LRS -IsOSDisk false

$DisksToReplicate += $OSDisk
$DisksToReplicate += $DataDisk

# Retrieve the resource group that you want to migrate to
$TargetResourceGroup = Get-AzResourceGroup -Name MyTargetResourceGroup

# Retrieve the Azure virtual network and subnet that you want to migrate to
$TargetVirtualNetwork = Get-AzVirtualNetwork -Name MyVirtualNetwork

# Start replication for the VM
$MigrateJob =  New-AzMigrateServerReplication -InputObject $DiscoveredServer -TargetResourceGroupId $TargetResourceGroup.ResourceId -TargetNetworkId $TargetVirtualNetwork.Id -LicenseType NoLicenseType -DiskToInclude $DisksToReplicate -TargetSubnetName $TargetVirtualNetwork.Subnets[0].Name -TargetVMName MyMigratedTestVM -TargetVMSize Standard_DS2_v2

# Track job status to check for completion
while (($MigrateJob.State -eq 'InProgress') -or ($MigrateJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
# Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $MigrateJob.State
```

## <a name="8-monitor-replication"></a>8. Мониторинг репликации

Репликация выполняется следующим образом:

- После успешного завершения задания "Запуск репликации" компьютеры начинают первоначальную репликацию в Azure.
- Во время начальной репликации создается снимок виртуальной машины. Дисковые данные из моментального снимка реплицируются в управляемые диски реплики в Azure.
- После завершения начальной репликации начинается разностная репликация. Добавочные изменения локальных дисков периодически реплицируются на диски реплики в Azure.

Отследить состояние репликации можно с помощью командлета [Get-AzMigrateServerReplication](/powershell/module/az.migrate/get-azmigrateserverreplication).



```azurepowershell-interactive
# List replicating VMs and filter the result for selecting a replicating VM. This cmdlet will not return all properties of the replicating VM.
$ReplicatingServer = Get-AzMigrateServerReplication -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -MachineName MyTestVM

# Retrieve all properties of a replicating VM 
$ReplicatingServer = Get-AzMigrateServerReplication -TargetObjectID $ReplicatingServer.Id
```

В выходных данных можно отслеживать свойства **Migration State** и **Migration State Description**.

- При начальной репликации свойства **Migration State** и **Migration State Description** будут иметь значения `InitialSeedingInProgress` и `Initial replication` соответственно.
- При разностной репликации свойства **Migration State** и **Migration State Description** будут иметь значения `Replicating` и `Ready to migrate` соответственно.
- После завершения репликации свойства **Migration State** и **Migration State Description** будут иметь значения `Migration succeeded` и `Migrated` соответственно.

```Output
AllowedOperation            : {DisableMigration, TestMigrate, Migrate}
CurrentJobId                : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationJobs/None
CurrentJobName              : None
CurrentJobStartTime         : 1/1/1753 1:01:01 AM
EventCorrelationId          : 9d435c55-4660-41a5-a8ed-dd74213d85fa
Health                      : Normal
HealthError                 : {}
Id                          : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationFabrics/xxx/replicationProtectionContainers/xxx/
                              replicationMigrationItems/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_5009e941-3e40-
                              39b2-1e14-f90584522703
LastTestMigrationStatus     :
LastTestMigrationTime       :
Location                    :
MachineName                 : MyTestVM
MigrationState              : InitialSeedingInProgress
MigrationStateDescription   : Initial replication
Name                        : 10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_5009e941-3e40-39b2-1e14-f90584522703
PolicyFriendlyName          : xxx
PolicyId                    : /Subscriptions/xxx/resourceGroups/xxx/providers/Micr
                              osoft.RecoveryServices/vaults/xxx/replicationPolicies/xxx
ProviderSpecificDetail      : Microsoft.Azure.PowerShell.Cmdlets.Migrate.Models.Api20180110.VMwareCbtMigrationDetails
TestMigrateState            : None
TestMigrateStateDescription : None
Type                        : Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems
```

Чтобы получить подробные сведения о ходе репликации, воспользуйтесь следующим командлетом.

```azurepowershell-interactive
Write-Output $replicatingserver.ProviderSpecificDetail
```

Ход выполнения начальной репликации можно отслеживать с помощью свойств **Initial Seeding Progress Percentage** в выходных данных.

```Output
    "DataMoverRunAsAccountId": "/subscriptions/xxx/resourceGroups/xxx/providers/Microsoft.OffAzure/VMwareSites/xxx/runasaccounts/xxx",
    "FirmwareType":  "BIOS",
    "InitialSeedingProgressPercentage": 20,
    "InstanceType":  "VMwareCbt",
    "LastRecoveryPointReceived":  "\/Date(1601733591427)\/",
    "LicenseType":  "NoLicenseType",
    "MigrationProgressPercentage":  null,
    "MigrationRecoveryPointId":  null,
    "OSType":  "Windows",
    "PerformAutoResync":  "true",
```

Репликация выполняется следующим образом:

- После успешного завершения задания "Запуск репликации" компьютеры начинают первоначальную репликацию в Azure.
- Во время начальной репликации создается снимок виртуальной машины. Дисковые данные из моментального снимка реплицируются в управляемые диски реплики в Azure.
- После завершения начальной репликации начинается разностная репликация. Добавочные изменения локальных дисков периодически реплицируются на диски реплики в Azure.

## <a name="9-retrieve-the-status-of-a-job"></a>9. Получение сведений о состоянии задания

Состояние задания можно отслеживать с помощью командлета [Get-AzMigrateJob](/powershell/module/az.migrate/get-azmigratejob).

```azurepowershell-interactive
# Retrieve the updated status for a job
$job = Get-AzMigrateJob -InputObject $job
```

## <a name="10-update-properties-of-a-replicating-vm"></a>10. Обновление свойств выполняющей репликацию виртуальной машины

Средство [Миграция Azure: миграция серверов](migrate-services-overview.md#azure-migrate-server-migration-tool) позволяет изменять целевые значения свойств, например имя, размер, группу ресурсов, конфигурацию сетевой карты и т. п., для выполняющей репликацию виртуальной машины.

Для виртуальной машины можно обновить следующие свойства.

- **Имя виртуальной машины**. С помощью параметра [`TargetVMName`] укажите имя виртуальной машины Azure, которую следует создать.
- **Размер виртуальной машины**. С помощью параметра [`TargetVMSize`] укажите размер виртуальной машины Azure, который будет использоваться для репликации виртуальной машины. Например, чтобы перенести виртуальную машину в D2_v2 в Azure, укажите для [`TargetVMSize`] значение `Standard_D2_v2`.
- **Виртуальная сеть**. С помощью параметра [`TargetNetworkId`] укажите идентификатор виртуальной сети Azure, в которую должна быть перенесена виртуальная машина.
- **Группа ресурсов**. Укажите идентификатор группы ресурсов, в которую следует перенести виртуальную машину. Для этого задайте идентификатор группы ресурсов с помощью параметра [`TargetResourceGroupId`].
- **Сетевой интерфейс.** Конфигурацию сетевой карты можно указать с помощью командлета [New-AzMigrateNicMapping](/powershell/module/az.migrate/new-azmigratenicmapping). В этом случае объект передается в качестве входных данных в параметр [`NicToUpdate`] командлета [Set-AzMigrateServerReplication](/powershell/module/az.migrate/set-azmigrateserverreplication).

    - **Изменение выделения IP-адресов**. Чтобы указать статический IP-адрес для сетевой карты, с помощью параметра [`TargetNicIP`] укажите адрес IPv4, который следует использовать в качестве статического для виртуальной машины. Чтобы IP-адрес назначался сетевой карте динамически, задайте для параметра **TargetNicIP** значение `auto`.
    - Используйте значения `Primary`, `Secondary` или `DoNotCreate` для параметра [`TargetNicSelectionType`], чтобы указать, какой должна быть сетевая карта на перенесенной виртуальной машине (основной, дополнительной или не должна быть создана вообще). Для виртуальной машины можно задать только одну основную сетевую карту.
    - Чтобы сделать сетевую карту основной, потребуется также указать другие сетевые карты. На перенесенной виртуальной машине они будут дополнительными или же не будут созданы вовсе.
    - Чтобы изменить подсеть для сетевой карты, укажите имя подсети с помощью параметра [`TargetNicSubnet`].

 - **Зона доступности**. Чтобы использовать зоны доступности, укажите зону доступности в качестве значения параметра [`TargetAvailabilityZone`].
 - **Группа доступности**. Чтобы использовать группу доступности, укажите ее идентификатор в качестве значения параметра [`TargetAvailabilitySet`].

Командлет [Get-AzMigrateServerReplication](/powershell/module/az.migrate/get-azmigrateserverreplication) возвращает задание, которое можно отслеживать для мониторинга состояния операции.

```azurepowershell-interactive
# List replicating VMs and filter the result for selecting a replicating VM. This cmdlet will not return all properties of the replicating VM.
$ReplicatingServer = Get-AzMigrateServerReplication -ProjectName $MigrateProject.Name -ResourceGroupName $ResourceGroup.ResourceGroupName -MachineName MyTestVM

# Retrieve all properties of a replicating VM 
$ReplicatingServer = Get-AzMigrateServerReplication -TargetObjectID $ReplicatingServer.Id

# View NIC details of the replicating server
Write-Output $ReplicatingServer.ProviderSpecificDetail.VMNic
```

В следующем примере показано, как обновить конфигурацию сетевой карты, назначив первую сетевую карту в качестве основной и присвоив ей статический IP-адрес. Мы удалим вторую сетевую карту при миграции и обновим имя и размер целевой виртуальной машины.

```azurepowershell-interactive
# Specify the NIC properties to be updated for a replicating VM.
$NicMapping = @()
$NicMapping1 = New-AzMigrateNicMapping -NicId $ReplicatingServer.ProviderSpecificDetail.VMNic[0].NicId -TargetNicIP ###.###.###.### -TargetNicSelectionType Primary
$NicMapping2 = New-AzMigrateNicMapping -NicId $ReplicatingServer.ProviderSpecificDetail.VMNic[1].NicId -TargetNicSelectionType DoNotCreate

$NicMapping += $NicMapping1
$NicMapping += $NicMapping2

# Update the name, size and NIC configuration of a replicating server
$UpdateJob = Set-AzMigrateServerReplication -InputObject $ReplicatingServer -TargetVMSize Standard_DS13_v2 -TargetVMName MyMigratedVM -NicToUpdate $NicMapping

# Track job status to check for completion
while (($UpdateJob.State -eq 'InProgress') -or ($UpdateJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $UpdateJob = Get-AzMigrateJob -InputObject $UpdateJob
}
# Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $UpdateJob.State
```



## <a name="11-run-a-test-migration"></a>11. Выполнение тестовой миграции

Когда начинается разностная репликация, вы можете запустить тестовую миграцию для виртуальных машин перед выполнением полной миграции в Azure. Мы настоятельно рекомендуем сначала выполнить тестовую миграцию каждой машины хотя бы один раз. Командлет возвращает задание, которое можно отслеживать для мониторинга состояния операции.

- Тестовая миграция позволяет проверить, будет ли миграция выполняться должным образом. Она не влияет на локальный компьютер, который остается работоспособным и продолжает выполнять репликацию.
- Тестовая миграция моделирует миграцию путем создания виртуальной машины Azure с использованием реплицированных данных (обычно выполняется миграция в нерабочую виртуальную сеть в вашей подписке Azure).
- Вы можете использовать реплицированную тестовую виртуальную машину Azure для проверки миграции, выполнения тестирования приложений и решения любых проблем перед полной миграцией.

Выберите виртуальную сеть Azure, которая будет использоваться для тестирования, указав ее идентификатор с помощью параметра [`TestNetworkID`].

```azurepowershell-interactive
# Retrieve the Azure virtual network created for testing
$TestVirtualNetwork = Get-AzVirtualNetwork -Name MyTestVirtualNetwork

# Start test migration for a replicating server
$TestMigrationJob = Start-AzMigrateTestMigration -InputObject $ReplicatingServer -TestNetworkID $TestVirtualNetwork.Id

# Track job status to check for completion
while (($TestMigrationJob.State -eq 'InProgress') -or ($TestMigrationJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $TestMigrationJob = Get-AzMigrateJob -InputObject $TestMigrationJob
}
# Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $TestMigrationJob.State
```

После завершения тестирования выполните очистку тестовой миграции с помощью командлета [Start-AzMigrateTestMigrationCleanup](/powershell/module/az.migrate/start-azmigratetestmigrationcleanup). Командлет возвращает задание, которое можно отслеживать для мониторинга состояния операции.

```azurepowershell-interactive
# Clean-up test migration for a replicating server
$CleanupTestMigrationJob = Start-AzMigrateTestMigrationCleanup -InputObject $ReplicatingServer

# Track job status to check for completion
while (($CleanupTestMigrationJob.State -eq "InProgress") -or ($CleanupTestMigrationJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $CleanupTestMigrationJob = Get-AzMigrateJob -InputObject $CleanupTestMigrationJob
}
# Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $CleanupTestMigrationJob.State
```

## <a name="12-migrate-vms"></a>12. Перенос виртуальных машин

Убедившись, что тестовая миграция работает должным образом, вы можете выполнить миграцию выполняющего репликацию сервера с помощью следующего командлета. Командлет возвращает задание, которое можно отслеживать для мониторинга состояния операции.

Если вы не хотите отключать исходный сервер, не используйте параметр [`TurnOffSourceServer`].

```azurepowershell-interactive
# Start migration for a replicating server and turn off source server as part of migration
$MigrateJob = Start-AzMigrateServerMigration -InputObject $ReplicatingServer -TurnOffSourceServer

# Track job status to check for completion
while (($MigrateJob.State -eq 'InProgress') -or ($MigrateJob.State -eq 'NotStarted')){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $MigrateJob = Get-AzMigrateJob -InputObject $MigrateJob
}
#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
Write-Output $MigrateJob.State
```

## <a name="13-complete-the-migration"></a>13. Выполнение миграции

1. После завершения миграции отключите репликацию для локального компьютера и очистите сведения о состоянии репликации для виртуальной машины с помощью следующего командлета. Командлет возвращает задание, которое можно отслеживать для мониторинга состояния операции.

   ```azurepowershell-interactive
   # Stop replication for a migrated server
   $StopReplicationJob = Remove-AzMigrateServerReplication -InputObject $ReplicatingServer

   # Track job status to check for completion
   while (($StopReplicationJob.State -eq 'InProgress') -or ($StopReplicationJob.State -eq 'NotStarted')){
           #If the job hasn't completed, sleep for 10 seconds before checking the job status again
           sleep 10;
           $StopReplicationJob = Get-AzMigrateJob -InputObject $StopReplicationJob
   }
   # Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded".
   Write-Output $StopReplicationJob.State
   ```

1. Установите агент [Linux](../virtual-machines/extensions/agent-linux.md) на перенесенных компьютерах, если на них используется ОС Linux. Агент виртуальных машин автоматически устанавливается для виртуальных машин Windows во время миграции.
1. Выполните любые действия по настройке после миграции приложения, такие как обновление строк подключения к базе данных и конфигурация веб-сервера.
1. Выполните приемочное тестирование конечного приложения и миграции на перенесенном приложении, работающем в Azure.
1. Остановите трафик для перенесенного экземпляра виртуальной машины Azure.
1. Удалите локальные виртуальные машины из списка локальных виртуальных машин.
1. Удалите локальные виртуальные машины из локальных заданий резервного копирования.
1. Обновите внутренние документы и отразите в них новое расположение и IP-адреса виртуальных машин Azure.

## <a name="14-post-migration-best-practices"></a>14. Рекомендации, выполняемые после миграции

- Для повышения устойчивости:
    - Обеспечьте безопасность данных путем резервного копирования виртуальных машин Azure с помощью службы Azure Backup. [Подробнее](../backup/quick-backup-vm-portal.md).
    - Обеспечьте непрерывную работу и постоянную доступность рабочих нагрузок за счет репликации виртуальных машин Azure в дополнительный регион с помощью Site Recovery. [Подробнее](../site-recovery/azure-to-azure-tutorial-enable-replication.md).
- Для повышения уровня безопасности:
    - Заблокируйте и ограничьте доступ входящего трафика с помощью [JIT-администрирования](../security-center/security-center-just-in-time.md) центра безопасности Azure.
    - Ограничьте сетевой трафик конечными точками с помощью [групп безопасности сети](../virtual-network/network-security-groups-overview.md).
    - Разверните [шифрование дисков Azure](../security/fundamentals/azure-disk-encryption-vms-vmss.md), чтобы обеспечить безопасность дисков и защитить данные от кражи и несанкционированного доступа.
    - Ознакомьтесь с дополнительными сведениями о [защите ресурсов IaaS](https://azure.microsoft.com/services/virtual-machines/secure-well-managed-iaas/) и посетите [центр безопасности Azure](https://azure.microsoft.com/services/security-center/).
- Для мониторинга и управления:
-  Рассмотрите возможность развертывания [службы "Управление затратами Azure"](../cost-management-billing/cloudyn/overview.md) для мониторинга использования ресурсов и затрат.
