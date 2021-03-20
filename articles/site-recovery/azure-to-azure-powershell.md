---
title: Аварийное восстановление для виртуальных машин Azure с помощью Azure PowerShell и Azure Site Recovery
description: Сведения о настройке аварийного восстановления в Azure Site Recovery для виртуальных машин Azure с помощью Azure PowerShell.
services: site-recovery
author: sujayt
manager: rochakm
ms.topic: article
ms.date: 3/29/2019
ms.author: sutalasi
ms.openlocfilehash: 1570bd9dfa62caa749d5a3983b93c2555be058ec
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93348735"
---
# <a name="set-up-disaster-recovery-for-azure-virtual-machines-using-azure-powershell"></a>Настройка аварийного восстановления для виртуальных машин Azure с помощью Azure PowerShell

Из этой статьи вы узнаете, как настроить и проверить аварийное восстановление для виртуальных машин Azure с помощью Azure PowerShell.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> - Создайте хранилище служб восстановления,
> - настройка контекста хранилища для сеанса PowerShell;
> - подготовка хранилища к началу репликации виртуальных машин Azure;
> - создание сетевых сопоставлений;
> - Создание целевых учетных записей хранения для репликации виртуальных машин.
> - репликация виртуальных машин Azure в регион восстановления для аварийного восстановления;
> - Выполнение тестовой отработки отказа, проверка и очистка после тестовой отработки отказа.
> - Отработка отказа в регион восстановления.

> [!NOTE]
> Azure PowerShell пока поддерживает не все возможности этого сценария, доступные через портал. Ниже перечислены некоторые возможности, которые пока недоступны через Azure PowerShell:
> - возможность сразу выбрать для репликации все диски виртуальной машины, чтобы не указывать их отдельно по одному.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы
- Вам должны быть понятны [архитектура и компоненты сценария](azure-to-azure-architecture.md).
- [Ознакомьтесь](azure-to-azure-support-matrix.md) с требованиями поддержки для всех компонентов.
- Установить модуль `Az` Azure PowerShell. Если вам необходимо установить или обновить Azure PowerShell, ознакомьтесь с этим [руководством по установке и настройке Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="sign-in-to-your-microsoft-azure-subscription"></a>Вход в подписку Microsoft Azure

Войдите в подписку Azure с помощью командлета `Connect-AzAccount`.

```azurepowershell
Connect-AzAccount
```

Выберите подписку Azure. Используйте командлет `Get-AzSubscription`, чтобы получить список подписок Azure, к которым у вас есть доступ. Выберите подписку Azure для работы с помощью `Set-AzContext` командлета.

```azurepowershell
Set-AzContext -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

## <a name="get-details-of-the-virtual-machine-to-be-replicated"></a>Получение сведений о реплицируемой виртуальной машине

В этой статье виртуальная машина в регионе "Восточная часть США" реплицируется и восстанавливается в регионе "Западная часть США 2". Виртуальная машина, для которой выполняется репликация, имеет диск операционной системы и один диск данных. В примере используется имя виртуальной машины `AzureDemoVM` .

```azurepowershell
# Get details of the virtual machine
$VM = Get-AzVM -ResourceGroupName "A2AdemoRG" -Name "AzureDemoVM"

Write-Output $VM
```

```Output
ResourceGroupName  : A2AdemoRG
Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/A2AdemoRG/providers/Microsoft.Compute/virtualMachines/AzureDemoVM
VmId               : 1b864902-c7ea-499a-ad0f-65da2930b81b
Name               : AzureDemoVM
Type               : Microsoft.Compute/virtualMachines
Location           : eastus
Tags               : {}
DiagnosticsProfile : {BootDiagnostics}
HardwareProfile    : {VmSize}
NetworkProfile     : {NetworkInterfaces}
OSProfile          : {ComputerName, AdminUsername, WindowsConfiguration, Secrets}
ProvisioningState  : Succeeded
StorageProfile     : {ImageReference, OsDisk, DataDisks}
```

Получение сведений о диске для дисков виртуальной машины. Сведения о дисках потребуются позже для запуска репликации виртуальной машины.

```azurepowershell
$OSDiskVhdURI = $VM.StorageProfile.OsDisk.Vhd
$DataDisk1VhdURI = $VM.StorageProfile.DataDisks[0].Vhd
```

## <a name="create-a-recovery-services-vault"></a>Создание хранилища Служб восстановления

Создайте группу ресурсов, в которой будет создано хранилище служб восстановления.

> [!IMPORTANT]
> * Хранилище служб восстановления и защищаемые виртуальные машины должны находиться в разных расположениях Azure.
> * Группа ресурсов для хранилища служб восстановления и защищаемые виртуальные машины должны находиться в разных расположениях Azure.
> * Хранилище служб восстановления и группа ресурсов, к которой оно относится, могут размещаться в одном расположении Azure.

В примере для этой статьи защищаемая виртуальная машина находится в регионе "Восточная часть США". Для аварийного восстановления выбран регион "Западная часть США 2". Хранилище служб восстановления и группа ресурсов хранилища находятся в регионе восстановления, Западная часть США 2.

```azurepowershell
#Create a resource group for the recovery services vault in the recovery Azure region
New-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"
```

```Output
ResourceGroupName : a2ademorecoveryrg
Location          : westus2
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg
```

Создайте хранилище служб восстановления. В этом примере хранилище служб восстановления с именем `a2aDemoRecoveryVault` создается в регионе "Западная часть США 2".

```azurepowershell
#Create a new Recovery services vault in the recovery region
$vault = New-AzRecoveryServicesVault -Name "a2aDemoRecoveryVault" -ResourceGroupName "a2ademorecoveryrg" -Location "West US 2"

Write-Output $vault
```

```Output
Name              : a2aDemoRecoveryVault
ID                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoRecoveryVault
Type              : Microsoft.RecoveryServices/vaults
Location          : westus2
ResourceGroupName : a2ademorecoveryrg
SubscriptionId    : xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

## <a name="set-the-vault-context"></a>Задание контекста хранилища

Настройте контекст хранилища для сеанса PowerShell. После установки контекста хранилища Azure Site Recovery операции в сеансе PowerShell выполняются в контексте выбранного хранилища.

```azurepowershell
#Setting the vault context.
Set-AzRecoveryServicesAsrVaultContext -Vault $vault
```

```Output
ResourceName         ResourceGroupName ResourceNamespace          ResourceType
------------         ----------------- -----------------          -----------
a2aDemoRecoveryVault a2ademorecoveryrg Microsoft.RecoveryServices Vaults
```

```azurepowershell
#Delete the downloaded vault settings file
Remove-Item -Path $Vaultsettingsfile.FilePath
```

Для миграции из Azure в Azure можно задать контекст хранилища для вновь созданного хранилища:

```azurepowershell
#Set the vault context for the PowerShell session.
Set-AzRecoveryServicesAsrVaultContext -Vault $vault
```

## <a name="prepare-the-vault-to-start-replicating-azure-virtual-machines"></a>Подготовка хранилища к началу репликации виртуальных машин Azure

### <a name="create-a-site-recovery-fabric-object-to-represent-the-primary-source-region"></a>Создание объекта структуры Site Recovery, который будет представлять исходный регион (источник)

Объект структуры в хранилище представляет регион Azure. Объект исходной структуры создается для представления региона Azure, в котором размещаются защищенные виртуальные машины. В примере для этой статьи защищаемая виртуальная машина находится в регионе "Восточная часть США".

- Можно создать только один объект структуры для каждого региона.
- Если была ранее включена репликация Site Recovery для виртуальной машины на портале Azure, Site Recovery автоматически создает объект структуры. Если в регионе уже существует объект структуры, новый создать не удастся.

Прежде чем начать, изучите, что операции Site Recovery выполняются асинхронно. При запуске операции отправляется задание Azure Site Recovery и возвращается объект отслеживания задания. Используйте объект отслеживания заданий, чтобы получить Последнее состояние задания ( `Get-AzRecoveryServicesAsrJob` ), а также для наблюдения за состоянием операции.

```azurepowershell
#Create Primary ASR fabric
$TempASRJob = New-AzRecoveryServicesAsrFabric -Azure -Location 'East US'  -Name "A2Ademo-EastUS"

# Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        #If the job hasn't completed, sleep for 10 seconds before checking the job status again
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$PrimaryFabric = Get-AzRecoveryServicesAsrFabric -Name "A2Ademo-EastUS"
```

Если в одном хранилище будут защищаться виртуальные машины из нескольких регионов Azure, создайте отдельный объект структуры для каждого исходного региона Azure.

### <a name="create-a-site-recovery-fabric-object-to-represent-the-recovery-region"></a>Создание объекта структуры Site Recovery, который будет представлять регион восстановления

Объект целевой структуры представляет расположение восстановления в Azure. При отработке отказа виртуальные машины реплицируются и восстанавливаются в регион восстановления, представленный структурой восстановления. В нашем примере регионом восстановления (целевым) Azure является регион "Западная часть США 2".

```azurepowershell
#Create Recovery ASR fabric
$TempASRJob = New-AzRecoveryServicesAsrFabric -Azure -Location 'West US 2'  -Name "A2Ademo-WestUS"

# Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$RecoveryFabric = Get-AzRecoveryServicesAsrFabric -Name "A2Ademo-WestUS"
```

### <a name="create-a-site-recovery-protection-container-in-the-primary-fabric"></a>Создание контейнера защиты Site Recovery в исходной структуре

Контейнер защиты — это контейнер, в котором сгруппированы реплицируемые элементы структуры.

```azurepowershell
#Create a Protection container in the primary Azure region (within the Primary fabric)
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainer -InputObject $PrimaryFabric -Name "A2AEastUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

Write-Output $TempASRJob.State

$PrimaryProtContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $PrimaryFabric -Name "A2AEastUSProtectionContainer"
```

### <a name="create-a-site-recovery-protection-container-in-the-recovery-fabric"></a>Создание контейнера защиты Site Recovery в структуре восстановления

```azurepowershell
#Create a Protection container in the recovery Azure region (within the Recovery fabric)
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainer -InputObject $RecoveryFabric -Name "A2AWestUSProtectionContainer"

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"

Write-Output $TempASRJob.State

$RecoveryProtContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $RecoveryFabric -Name "A2AWestUSProtectionContainer"
```

#### <a name="fabric-and-container-creation-when-enabling-zone-to-zone-replication"></a>Создание структуры и контейнера при включении зоны для репликации зоны

При включении зоны для репликации зоны будет создана только одна структура. Но будет два контейнера. Предполагая, что регион находится в Западной Европе, используйте следующие команды для получения первичного контейнера и контейнеров защиты.

```azurepowershell
$primaryProtectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $fabric -Name "asr-a2a-default-westeurope-container"
$recoveryPprotectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $fabric -Name "asr-a2a-default-westeurope-t-container"
```

### <a name="create-a-replication-policy"></a>Создание политики репликации

```azurepowershell
#Create replication policy
$TempASRJob = New-AzRecoveryServicesAsrPolicy -AzureToAzure -Name "A2APolicy" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$ReplicationPolicy = Get-AzRecoveryServicesAsrPolicy -Name "A2APolicy"
```

### <a name="create-a-protection-container-mapping-between-the-primary-and-recovery-protection-container"></a>Создание сопоставления между исходным контейнером защиты и целевым контейнером защиты

Сопоставление контейнеров защиты связывает между собой исходный контейнер защиты, целевой контейнер защиты и политику репликации. Создайте отдельное сопоставление для каждой политики репликации, которую вы будете применять для репликации виртуальных машин между каждой парой контейнеров защиты.

```azurepowershell
#Create Protection container mapping between the Primary and Recovery Protection Containers with the Replication policy
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "A2APrimaryToRecovery" -Policy $ReplicationPolicy -PrimaryProtectionContainer $PrimaryProtContainer -RecoveryProtectionContainer $RecoveryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$EusToWusPCMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $PrimaryProtContainer -Name "A2APrimaryToRecovery"
```

#### <a name="protection-container-mapping-creation-when-enabling-zone-to-zone-replication"></a>Создание сопоставления контейнеров защиты при включении зоны для репликации зоны

При включении зоны для репликации зоны используйте приведенную ниже команду, чтобы создать сопоставление контейнеров защиты. Если регионом является Западная Европа, команда будет иметь значение-

```azurepowershell
$protContainerMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $PrimprotectionContainer -Name "westeurope-westeurope-24-hour-retention-policy-s"
```

### <a name="create-a-protection-container-mapping-for-failback-reverse-replication-after-a-failover"></a>Создание сопоставления контейнеров защиты для восстановления размещения (обратной репликации после отработки отказа)

После отработки отказа, когда вы будете готовы перевести виртуальную машину, для которой выполнен переход на другой ресурс, обратно в исходный регион Azure, восстановление размещения выполняется. Для восстановления после сбоя виртуальная машина, для которой выполнена отработка отказа, реплицируется из отработки отказа в исходный регион. Для обратной репликации исходный и целевой регионы меняются ролями. Исходный регион становится новым регионом восстановления, а бывший регион восстановления становится исходным регионом. Сопоставление контейнеров защиты для обратной репликации отражает эту смену ролей исходного и целевого регионов.

```azurepowershell
#Create Protection container mapping (for fail back) between the Recovery and Primary Protection Containers with the Replication policy
$TempASRJob = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "A2ARecoveryToPrimary" -Policy $ReplicationPolicy -PrimaryProtectionContainer $RecoveryProtContainer -RecoveryProtectionContainer $PrimaryProtContainer

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State

$WusToEusPCMapping = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $RecoveryProtContainer -Name "A2ARecoveryToPrimary"
```

## <a name="create-cache-storage-account-and-target-storage-account"></a>Создать учетную запись хранения кэша и целевую учетную запись хранения

Учетная запись хранения кэша — это стандартная учетная запись хранения в том же регионе Azure, где расположена реплицируемая виртуальная машина. Учетная запись хранения кэша используется для временного хранения реплицируемых изменений перед их перемещением в регион восстановления Azure. Вы можете выбрать, но это необязательно, чтобы указать разные учетные записи хранения кэша для разных дисков виртуальной машины.

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$EastUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestorage" -ResourceGroupName "A2AdemoRG" -Location 'East US' -SkuName Standard_LRS -Kind Storage
```

Для виртуальных машин, **не использующих управляемые диски**, Целевая учетная запись хранения — это учетная запись хранения в регионе восстановления, в которой реплицируются диски виртуальной машины. Целевая учетная запись хранения может иметь категорию "Стандартная" или "Премиум". Выберите тип учетной записи хранения, необходимый в зависимости от скорости изменения данных (скорость записи ввода-вывода) для дисков и Azure Site Recovery поддерживаемые ограничения на количество обновлений для типа хранилища.

```azurepowershell
#Create Target storage account in the recovery region. In this case a Standard Storage account
$WestUSTargetStorageAccount = New-AzStorageAccount -Name "a2atargetstorage" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -SkuName Standard_LRS -Kind Storage
```

## <a name="create-network-mappings"></a>Создание сетевых сопоставлений

Сетевое сопоставление связывает виртуальные сети исходного региона с виртуальными сетями в целевом регионе. Сетевое сопоставление указывает виртуальную сеть Azure в регионе восстановления, на которую должна выполняться отработка отказа виртуальной машины в основной виртуальной сети. Виртуальную сеть Azure можно сопоставить только с одной виртуальной сетью Azure в регионе восстановления.

- Создайте виртуальную сеть Azure в регионе восстановления, чтобы выполнить отработку отказа на:

   ```azurepowershell
    #Create a Recovery Network in the recovery region
    $WestUSRecoveryVnet = New-AzVirtualNetwork -Name "a2arecoveryvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.0.0.0/16"

    Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $WestUSRecoveryVnet -AddressPrefix "10.0.0.0/20" | Set-AzVirtualNetwork

    $WestUSRecoveryNetwork = $WestUSRecoveryVnet.Id
   ```

- Получите основную виртуальную сеть. Виртуальной сети, к которой подключена виртуальная машина:

   ```azurepowershell
    #Retrieve the virtual network that the virtual machine is connected to

    #Get first network interface card(nic) of the virtual machine
    $SplitNicArmId = $VM.NetworkProfile.NetworkInterfaces[0].Id.split("/")

    #Extract resource group name from the ResourceId of the nic
    $NICRG = $SplitNicArmId[4]

    #Extract resource name from the ResourceId of the nic
    $NICname = $SplitNicArmId[-1]

    #Get network interface details using the extracted resource group name and resource name
    $NIC = Get-AzNetworkInterface -ResourceGroupName $NICRG -Name $NICname

    #Get the subnet ID of the subnet that the nic is connected to
    $PrimarySubnet = $NIC.IpConfigurations[0].Subnet

    # Extract the resource ID of the Azure virtual network the nic is connected to from the subnet ID
    $EastUSPrimaryNetwork = (Split-Path(Split-Path($PrimarySubnet.Id))).Replace("\","/")
   ```

- Создайте Сетевое сопоставление между основной виртуальной сетью и виртуальной сетью восстановления.

   ```azurepowershell
    #Create an ASR network mapping between the primary Azure virtual network and the recovery Azure virtual network
    $TempASRJob = New-AzRecoveryServicesAsrNetworkMapping -AzureToAzure -Name "A2AEusToWusNWMapping" -PrimaryFabric $PrimaryFabric -PrimaryAzureNetworkId $EastUSPrimaryNetwork -RecoveryFabric $RecoveryFabric -RecoveryAzureNetworkId $WestUSRecoveryNetwork

    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State
   ```

- Создать сетевое сопоставление для обратного направления (восстановления после сбоя):

    ```azurepowershell
    #Create an ASR network mapping for fail back between the recovery Azure virtual network and the primary Azure virtual network
    $TempASRJob = New-AzRecoveryServicesAsrNetworkMapping -AzureToAzure -Name "A2AWusToEusNWMapping" -PrimaryFabric $RecoveryFabric -PrimaryAzureNetworkId $WestUSRecoveryNetwork -RecoveryFabric $PrimaryFabric -RecoveryAzureNetworkId $EastUSPrimaryNetwork

    #Track Job status to check for completion
    while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
            sleep 10;
            $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
    }

    #Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
    Write-Output $TempASRJob.State
    ```

## <a name="replicate-azure-virtual-machine"></a>Репликация виртуальной машины Azure

Выполните репликацию виртуальной машины Azure с помощью **управляемых дисков**.

```azurepowershell
#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration)

#OsDisk
$OSdiskId = $vm.StorageProfile.OsDisk.ManagedDisk.Id
$RecoveryOSDiskAccountType = $vm.StorageProfile.OsDisk.ManagedDisk.StorageAccountType
$RecoveryReplicaDiskAccountType = $vm.StorageProfile.OsDisk.ManagedDisk.StorageAccountType

$OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
         -DiskId $OSdiskId -RecoveryResourceGroupId  $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType  $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

# Data disk
$datadiskId1 = $vm.StorageProfile.DataDisks[0].ManagedDisk.Id
$RecoveryReplicaDiskAccountType = $vm.StorageProfile.DataDisks[0].ManagedDisk.StorageAccountType
$RecoveryTargetDiskAccountType = $vm.StorageProfile.DataDisks[0].ManagedDisk.StorageAccountType

$DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
         -DiskId $datadiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

#Create a list of disk replication configuration objects for the disks of the virtual machine that are to be replicated.
$diskconfigs = @()
$diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

#Start replication by creating replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.
$TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId
```

Выполните репликацию виртуальной машины Azure с помощью **неуправляемых дисков**.

```azurepowershell
#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration)

#Disk replication configuration for the OS disk
$OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -VhdUri $OSDiskVhdURI.Uri -LogStorageAccountId $EastUSCacheStorageAccount.Id -RecoveryAzureStorageAccountId $WestUSTargetStorageAccount.Id

#Disk replication configuration for data disk
$DataDisk1ReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -VhdUri $DataDisk1VhdURI.Uri -LogStorageAccountId $EastUSCacheStorageAccount.Id -RecoveryAzureStorageAccountId $WestUSTargetStorageAccount.Id

#Create a list of disk replication configuration objects for the disks of the virtual machine that are to be replicated.
$diskconfigs = @()
$diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Start replication by creating replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.
$TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId

#Track Job status to check for completion
while (($TempASRJob.State -eq "InProgress") -or ($TempASRJob.State -eq "NotStarted")){
        sleep 10;
        $TempASRJob = Get-AzRecoveryServicesAsrJob -Job $TempASRJob
}

#Check if the Job completed successfully. The updated job state of a successfully completed job should be "Succeeded"
Write-Output $TempASRJob.State
```

Когда начальная репликация успешно завершится, все данные виртуальной машины будут реплицированы в регион восстановления.

Процесс репликации начинается с того, что в регионе восстановления создаются копии реплицируемых дисков виртуальной машины. Этот этап называется начальной репликацией.

После завершения начальной репликации репликация перемещается на фазу разностной синхронизации. На этом этапе виртуальная машина считается защищенной и для нее можно выполнять тестовую отработку отказа. Состояние репликации реплицированного элемента, представляющего виртуальную машину, переходит в **защищенное** состояние после завершения начальной репликации.

Чтобы отследить текущее состояние и работоспособность репликации для виртуальной машины, получите сведения о репликации для соответствующего защищенного элемента.

```azurepowershell
Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $PrimaryProtContainer | Select FriendlyName, ProtectionState, ReplicationHealth
```

```
FriendlyName ProtectionState ReplicationHealth
------------ --------------- -----------------
AzureDemoVM  Protected       Normal
```

## <a name="do-a-test-failover-validate-and-cleanup-test-failover"></a>Проверка отработки отказа, проверка и очистка тестовой отработки отказа

После того как репликация для виртуальной машины достигнет защищенного состояния, на виртуальной машине можно выполнить операцию тестовой отработки отказа (в элементе, защищенном репликацией виртуальной машины).

```azurepowershell
#Create a separate network for test failover (not connected to my DR network)
$TFOVnet = New-AzVirtualNetwork -Name "a2aTFOvnet" -ResourceGroupName "a2ademorecoveryrg" -Location 'West US 2' -AddressPrefix "10.3.0.0/16"

Add-AzVirtualNetworkSubnetConfig -Name "default" -VirtualNetwork $TFOVnet -AddressPrefix "10.3.0.0/20" | Set-AzVirtualNetwork

$TFONetwork= $TFOVnet.Id
```

Выполните тестовую отработку отказа.

```azurepowershell
$ReplicationProtectedItem = Get-AzRecoveryServicesAsrReplicationProtectedItem -FriendlyName "AzureDemoVM" -ProtectionContainer $PrimaryProtContainer

$TFOJob = Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -AzureVMNetworkId $TFONetwork -Direction PrimaryToRecovery
```

Дождитесь завершения этой операции.

```azurepowershell
Get-AzRecoveryServicesAsrJob -Job $TFOJob
```

```Output
Name             : 3dcb043e-3c6d-4e0e-a42e-8d4245668547
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/3dcb043e-3c6d-4e0e-a42e-8d4245668547
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : TestFailover
DisplayName      : Test failover
ClientRequestId  : 1ef8515b-b130-4452-a44d-91aaf071931c ActivityId: 907bb2bc-ebe6-4732-8b66-77d0546eaba8
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:29:43 AM
EndTime          : 4/25/2018 4:33:06 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   :
Tasks            : {Prerequisites check for test failover, Create test virtual machine, Preparing the virtual machine, Start the virtual machine}
Errors           : {}
```

После успешного завершения задания тестовой отработки отказа можно подключиться к виртуальной машине теста, для которого произошел сбой, и проверить тестовую отработку отказа.

Когда проверка созданной тестовой виртуальной машины завершится, удалите эту тестовую копию, запустив операцию очистки после тестовой отработки отказа. Эта операция удаляет тестовую копию виртуальной машины, созданную в процессе тестовой отработки отказа.

```azurepowershell
$Job_TFOCleanup = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $Job_TFOCleanup | Select State
```

```Output
State
-----
Succeeded
```

## <a name="fail-over-to-azure"></a>Отработка отказа с переходом в Azure

Отработка отказа виртуальной машины на определенную точку восстановления.

```azurepowershell
$RecoveryPoints = Get-AzRecoveryServicesAsrRecoveryPoint -ReplicationProtectedItem $ReplicationProtectedItem

#The list of recovery points returned may not be sorted chronologically and will need to be sorted first, in order to be able to find the oldest or the latest recovery points for the virtual machine.
"{0} {1}" -f $RecoveryPoints[0].RecoveryPointType, $RecoveryPoints[-1].RecoveryPointTime
```

```Output
CrashConsistent 4/24/2018 11:10:25 PM
```

```azurepowershell
#Start the fail over job
$Job_Failover = Start-AzRecoveryServicesAsrUnplannedFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem -Direction PrimaryToRecovery -RecoveryPoint $RecoveryPoints[-1]

do {
        $Job_Failover = Get-AzRecoveryServicesAsrJob -Job $Job_Failover;
        sleep 30;
} while (($Job_Failover.State -eq "InProgress") -or ($JobFailover.State -eq "NotStarted"))

$Job_Failover.State
```

```Output
Succeeded
```

После успешного выполнения задания отработки отказа можно применить операцию отработки отказа.

```azurepowershell
$CommitFailoverJOb = Start-AzRecoveryServicesAsrCommitFailoverJob -ReplicationProtectedItem $ReplicationProtectedItem

Get-AzRecoveryServicesAsrJob -Job $CommitFailoverJOb
```

```Output
Name             : 58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/a2ademorecoveryrg/providers/Microsoft.RecoveryServices/vaults/a2aDemoR
                   ecoveryVault/replicationJobs/58afc2b7-5cfe-4da9-83b2-6df358c6e4ff
Type             : Microsoft.RecoveryServices/vaults/replicationJobs
JobType          : CommitFailover
DisplayName      : Commit
ClientRequestId  : 10a95d6c-359e-4603-b7d9-b7ee3317ce94 ActivityId: 8751ada4-fc42-4238-8de6-a82618408fcf
State            : Succeeded
StateDescription : Completed
StartTime        : 4/25/2018 4:50:58 AM
EndTime          : 4/25/2018 4:51:01 AM
TargetObjectId   : ce86206c-bd78-53b4-b004-39b722c1ac3a
TargetObjectType : ProtectionEntity
TargetObjectName : azuredemovm
AllowedActions   :
Tasks            : {Prerequisite check, Commit}
Errors           : {}
```

## <a name="reprotect-and-fail-back-to-the-source-region"></a>Повторное включение защиты и возвращение к исходному региону

После отработки отказа, когда вы будете готовы вернуться к исходному региону, запустите обратную репликацию для защищенного элемента репликации с помощью `Update-AzRecoveryServicesAsrProtectionDirection` командлета.

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$WestUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestoragewestus" -ResourceGroupName "A2AdemoRG" -Location 'West US' -SkuName Standard_LRS -Kind Storage
```

```azurepowershell
#Use the recovery protection container, new cache storage account in West US and the source region VM resource group
Update-AzRecoveryServicesAsrProtectionDirection -ReplicationProtectedItem $ReplicationProtectedItem -AzureToAzure
-ProtectionContainerMapping $WusToEusPCMapping -LogStorageAccountId $WestUSCacheStorageAccount.Id -RecoveryResourceGroupID $sourceVMResourcegroup.ResourceId
```

После завершения повторной защиты можно выполнить отработку отказа в обратном направлении, Западная часть США — Восточная часть США и восстановить расположение в исходном регионе.

## <a name="disable-replication"></a>Отключение репликации

Репликацию можно отключить с помощью `Remove-AzRecoveryServicesAsrReplicationProtectedItem` командлета.

```azurepowershell
Remove-AzRecoveryServicesAsrReplicationProtectedItem -ReplicationProtectedItem $ReplicationProtectedItem
```

## <a name="next-steps"></a>Дальнейшие действия

Просмотрите [Справочник по Azure Site Recovery PowerShell](/powershell/module/az.RecoveryServices) , чтобы узнать, как можно выполнять другие задачи, такие как создание планов восстановления и тестирование отработки отказа планов восстановления с помощью PowerShell.
