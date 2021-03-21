---
title: Репликация виртуальных машин Azure, выполняемых в группах размещения близкого взаимодействия
description: Узнайте, как реплицировать виртуальные машины Azure, выполняемые в группах размещения близкого взаимодействия, с использованием Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.topic: how-to
ms.date: 02/11/2021
ms.openlocfilehash: 681b635099d450f061e0bcdb5b2c5d60d56c20a3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100380770"
---
# <a name="replicate-azure-virtual-machines-running-in-proximity-placement-groups-to-another-region"></a>Репликация виртуальных машин Azure, выполняемых в группах размещения близкого взаимодействия, в другой регион

В этой статье описывается репликация, отработка отказа и восстановление виртуальных машин, выполняемых в группах размещения близкого взаимодействия, в дополнительном регионе.

[Группы размещения близкого взаимодействия](../virtual-machines/windows/proximity-placement-groups-portal.md) — это реализованная в виртуальных машинах Azure возможность логической группировки, благодаря которой можно сократить сетевую задержку приложений при обмене данными между виртуальными машинами. Если виртуальные машины развернуты в одной группе размещения близкого взаимодействия, они физически расположены как можно ближе друг к другу. Группы размещения близкого взаимодействия особенно полезны при выполнении рабочих нагрузок, чувствительных к величине задержки.

## <a name="disaster-recovery-with-proximity-placement-groups"></a>Аварийное восстановление с использованием групп размещения близкого взаимодействия

В стандартном сценарии виртуальные машины могут работать в группах размещения близкого взаимодействия во избежание сетевой задержки между различными уровнями приложения. Хотя это может обеспечить оптимальную сетевую задержку для вашего приложения, нужно защитить эти приложения с помощью Site Recovery от любых сбоев на региональном уровне. Site Recovery реплицирует данные из одного региона в другой регион Azure и вызывает компьютеры в регионе аварийного восстановления в случае сбоя.

## <a name="considerations"></a>Рекомендации

- Лучше всего организовать отработку отказа/восстановление виртуальных машин в группе размещения близкого взаимодействия. Однако если восстановить виртуальную машину внутри групп размещения близкого взаимодействия в процессе отработки отказа не удастся, отработка отказа все равно произойдет, но виртуальные машины будут созданы вне группы размещения.
- Если группа доступности закреплена в группе размещения близкого взаимодействия и во время отработки отказа/восстановления виртуальных машин в группе доступности действует ограничение на распределение, виртуальные машины будут созданы за пределами группы доступности и группы размещения близкого взаимодействия.
- Site Recovery для групп размещения близкого взаимодействия не поддерживается для неуправляемых дисков.

> [!NOTE]
> Azure Site Recovery не поддерживает восстановление размещения с управляемых дисков для сценариев Hyper-V в Azure. Следовательно, восстановление размещения из группы размещения в Azure на Hyper-V не поддерживается.

## <a name="set-up-disaster-recovery-for-vms-in-proximity-placement-groups-via-portal"></a>Настройка аварийного восстановления для виртуальных машин в группах размещения с учетом расположения через портал

### <a name="azure-to-azure-via-portal"></a>С Azure в Azure с помощью портала

Вы можете включить репликацию для виртуальной машины на странице аварийного восстановления виртуальной машины или в предварительно созданном хранилище, перейдя к разделу Site Recovery, а затем включив репликацию. Давайте посмотрим, как Site Recovery можно настроить для виртуальных машин в ППГ с помощью обоих подходов:

- Как выбрать ППГ в регионе аварийного восстановления, включив репликацию через колонку "аварийное восстановление ВМ IaaS":
  1. Перейдите к виртуальной машине. В боковой колонке слева в разделе "операции" выберите "аварийное восстановление".
  2. На вкладке "основы" выберите регион аварийного восстановления, в который требуется реплицировать виртуальную машину. Перейдите в раздел "Дополнительные параметры".
  3. Здесь можно увидеть группу размещения с учетом расположения на виртуальной машине и выбрать ППГ в регионе аварийного восстановления. Site Recovery также дает возможность использовать новую группу размещения, которая создается для вас при выборе этого параметра по умолчанию. Вы можете выбрать нужную группу размещения, а затем перейти к разделу "Проверка и запуск репликации", а затем включить репликацию.

   :::image type="content" source="media/how-to-enable-replication-proximity-placement-groups/proximity-placement-group-a2a-1.png" alt-text="Включение репликации.":::

- Как выбрать ППГ в регионе аварийного восстановления при включении репликации в колонке хранилища:
  1. Перейдите к хранилищу служб восстановления и перейдите на вкладку Site Recovery
  2. Щелкните "+ Enable Site Recovery", а затем выберите "1: включить репликацию" на виртуальных машинах Azure (так как вы хотите реплицировать виртуальную машину Azure).
  3. Заполните обязательные поля на вкладке "источник" и нажмите кнопку "Далее".
  4. Выберите список виртуальных машин, для которых требуется включить репликацию, на вкладке "виртуальные машины" и нажмите кнопку "Далее".
  5. Здесь можно увидеть возможность выбора ППГ в регионе аварийного восстановления. Site Recovery также дает возможность использовать новый ППГ, который он создает для вас при выборе этого параметра по умолчанию. Вы можете выбрать нужный ППГ, а затем перейти к включению репликации.

   :::image type="content" source="media/how-to-enable-replication-proximity-placement-groups/proximity-placement-group-a2a-2.png" alt-text="Включите репликацию через хранилище.":::

Обратите внимание, что вы можете легко обновить выбор ППГ в регионе аварийного восстановления после включения репликации для виртуальной машины.

1. Перейдите к виртуальной машине и в колонке слева в разделе "операции" выберите "аварийное восстановление".
2. Перейдите в колонку "расчеты и сеть" и щелкните "Изменить" в верхней части страницы.
3. Вы можете увидеть параметры для изменения нескольких целевых параметров, включая Target ППГ. Выберите ППГ, в которую вы хотите выполнить отработку отказа виртуальной машины, и нажмите кнопку "Сохранить".

### <a name="vmware-to-azure-via-portal"></a>VMware в Azure с помощью портала

Группу размещения с учетом расположения для целевой виртуальной машины можно настроить после включения репликации для виртуальной машины. Убедитесь, что вы самостоятельно создаете ППГ в целевом регионе в соответствии с вашими требованиями. Впоследствии можно легко обновить выбор ППГ в регионе аварийного восстановления после включения репликации для виртуальной машины.

1. Выберите виртуальную машину из хранилища и в колонке слева в разделе "операции" выберите "аварийное восстановление".
2. Перейдите в колонку "расчеты и сеть" и щелкните "Изменить" в верхней части страницы.
3. Вы можете увидеть параметры для изменения нескольких целевых параметров, включая Target ППГ. Выберите ППГ, в которую вы хотите выполнить отработку отказа виртуальной машины, и нажмите кнопку "Сохранить".

   :::image type="content" source="media/how-to-enable-replication-proximity-placement-groups/proximity-placement-groups-update-v2a.png" alt-text="Обновление ППГ V2A":::

### <a name="hyper-v-to-azure-via-portal"></a>Hyper-V в Azure с помощью портала

Группу размещения с учетом расположения для целевой виртуальной машины можно настроить после включения репликации для виртуальной машины. Убедитесь, что вы самостоятельно создаете ППГ в целевом регионе в соответствии с вашими требованиями. Впоследствии можно легко обновить выбор ППГ в регионе аварийного восстановления после включения репликации для виртуальной машины.

1. Выберите виртуальную машину из хранилища и в колонке слева в разделе "операции" выберите "аварийное восстановление".
2. Перейдите в колонку "расчеты и сеть" и щелкните "Изменить" в верхней части страницы.
3. Вы можете увидеть параметры для изменения нескольких целевых параметров, включая Target ППГ. Выберите ППГ, в которую вы хотите выполнить отработку отказа виртуальной машины, и нажмите кнопку "Сохранить".

   :::image type="content" source="media/how-to-enable-replication-proximity-placement-groups/proximity-placement-groups-update-h2a.png" alt-text="Обновление ППГ H2A":::

## <a name="set-up-disaster-recovery-for-vms-in-proximity-placement-groups-via-powershell"></a>Настройка аварийного восстановления для виртуальных машин в группах размещения с учетом расположения с помощью PowerShell

### <a name="prerequisites"></a>Предварительные требования 

1. Убедитесь в наличии модуля Azure PowerShell Az. Если вам необходимо установить или обновить Azure PowerShell, ознакомьтесь с этим [руководством по установке и настройке Azure PowerShell](/powershell/azure/install-az-ps).
2. Минимальная Azure PowerShell AZ version должна быть 4.1.0. Чтобы проверить текущую версию, используйте следующую команду:

    ```
    Get-InstalledModule -Name Az
    ```

### <a name="set-up-site-recovery-for-virtual-machines-in-proximity-placement-group"></a>Настройка Site Recovery для виртуальных машин в группе размещения близкого взаимодействия

> [!NOTE]
> Убедитесь, что у вас есть уникальный идентификатор целевой группы размещения с учетом расположения. Если вы создаете новую группу размещения с учетом расположения, проверьте команду [здесь](../virtual-machines/windows/proximity-placement-groups.md#create-a-proximity-placement-group) и, если вы используете существующую группу размещения с учетом расположения, выполните команду [здесь](../virtual-machines/windows/proximity-placement-groups.md#list-proximity-placement-groups).

### <a name="azure-to-azure"></a>Из Azure в Azure

1. [Войдите](./azure-to-azure-powershell.md#sign-in-to-your-microsoft-azure-subscription) в свою учетную запись и настройте подписку.
2. Получите сведения о виртуальной машине, которую вы планируете реплицировать, как указано [здесь](./azure-to-azure-powershell.md#get-details-of-the-virtual-machine-to-be-replicated).
3. [Создайте](./azure-to-azure-powershell.md#create-a-recovery-services-vault) хранилище служб восстановления и [задайте](./azure-to-azure-powershell.md#set-the-vault-context) контекст хранилища.
4. Подготовьте хранилище к запуску виртуальной машины репликации. Это включает создание [объекта фабрики службы](./azure-to-azure-powershell.md#create-a-site-recovery-fabric-object-to-represent-the-primary-source-region) для основного региона и региона восстановления.
5. [Создайте](./azure-to-azure-powershell.md#create-a-site-recovery-protection-container-in-the-primary-fabric) защитный контейнер Site Recovery для основной фабрики и фабрики восстановления.
6. [Создайте](./azure-to-azure-powershell.md#create-a-replication-policy) политику репликации.
7. Создайте сопоставление защитных контейнеров (основного контейнера и контейнера восстановления), выполнив [эти](./azure-to-azure-powershell.md#create-a-protection-container-mapping-between-the-primary-and-recovery-protection-container) шаги, а также сопоставление защитных контейнеров для восстановления, следуя инструкциям [здесь](./azure-to-azure-powershell.md#create-a-protection-container-mapping-for-failback-reverse-replication-after-a-failover).
8. Создайте учетную запись хранения кэша, выполнив [следующие](./azure-to-azure-powershell.md#create-cache-storage-account-and-target-storage-account) действия.
9. Создайте необходимые сетевые сопоставления, как указано [здесь](./azure-to-azure-powershell.md#create-network-mappings).
10. Чтобы реплицировать виртуальную машину Azure с управляемыми дисками, используйте следующий командлет PowerShell:

```azurepowershell
#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration)
#Make sure to replace the variables $OSdiskName with OS disk name.

#OS Disk
$OSdisk = Get-AzDisk -DiskName $OSdiskName -ResourceGroupName "A2AdemoRG"
$OSdiskId = $OSdisk.Id
$RecoveryOSDiskAccountType = $OSdisk.Sku.Name
$RecoveryReplicaDiskAccountType = $OSdisk.Sku.Name

$OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $OSdiskId -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

#Make sure to replace the variables $datadiskName with data disk name.

#Data disk
$datadisk = Get-AzDisk -DiskName $datadiskName -ResourceGroupName "A2AdemoRG"
$datadiskId1 = $datadisk[0].Id
$RecoveryReplicaDiskAccountType = $datadisk[0].Sku.Name
$RecoveryTargetDiskAccountType = $datadisk[0].Sku.Name

$DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $datadiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType

#Create a list of disk replication configuration objects for the disks of the virtual machine that are to be replicated.

$diskconfigs = @()
$diskconfigs += $OSDiskReplicationConfig, $DataDisk1ReplicationConfig

#Start replication by creating replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.

$TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id
```

При включении репликации для нескольких дисков данных используйте следующий командлет PowerShell:

```azurepowershell
#Get the resource group that the virtual machine must be created in when failed over.
$RecoveryRG = Get-AzResourceGroup -Name "a2ademorecoveryrg" -Location "West US 2"

#Specify replication properties for each disk of the VM that is to be replicated (create disk replication configuration)
#Make sure to replace the variables $OSdiskName with OS disk name.

#OS Disk
$OSdisk = Get-AzDisk -DiskName $OSdiskName -ResourceGroupName "A2AdemoRG"
$OSdiskId = $OSdisk.Id
$RecoveryOSDiskAccountType = $OSdisk.Sku.Name
$RecoveryReplicaDiskAccountType = $OSdisk.Sku.Name

$OSDiskReplicationConfig = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id -DiskId $OSdiskId -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType -RecoveryTargetDiskAccountType $RecoveryOSDiskAccountType

$diskconfigs = @()
$diskconfigs.Add($OSDiskReplicationConfig)

#Data disk

# Add data disks
Foreach( $disk in $VM.StorageProfile.DataDisks)
{
    $datadisk = Get-AzDisk -DiskName $datadiskName -ResourceGroupName "A2AdemoRG"
    $dataDiskId1 = $datadisk[0].Id
    $RecoveryReplicaDiskAccountType = $datadisk[0].Sku.Name
    $RecoveryTargetDiskAccountType = $datadisk[0].Sku.Name
    $DataDisk1ReplicationConfig  = New-AzRecoveryServicesAsrAzureToAzureDiskReplicationConfig -ManagedDisk -LogStorageAccountId $EastUSCacheStorageAccount.Id `
         -DiskId $dataDiskId1 -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryReplicaDiskAccountType $RecoveryReplicaDiskAccountType `
         -RecoveryTargetDiskAccountType $RecoveryTargetDiskAccountType
    $diskconfigs.Add($DataDisk1ReplicationConfig)
}

#Start replication by creating replication protected item. Using a GUID for the name of the replication protected item to ensure uniqueness of name.

$TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id
```

При включении зоны для репликации зоны с помощью ППГ команда для запуска репликации будет заменена командлетом PowerShell.

```azurepowershell
$TempASRJob = New-AzRecoveryServicesAsrReplicationProtectedItem -AzureToAzure -AzureVmId $VM.Id -Name (New-Guid).Guid -ProtectionContainerMapping $EusToWusPCMapping -AzureToAzureDiskReplicationConfiguration $diskconfigs -RecoveryResourceGroupId $RecoveryRG.ResourceId -RecoveryProximityPlacementGroupId $targetPpg.Id -RecoveryAvailabilityZone "2"
```

Когда начальная репликация успешно завершится, все данные виртуальной машины будут реплицированы в регион восстановления.

Процесс репликации начинается с того, что в регионе восстановления создаются копии реплицируемых дисков виртуальной машины. Этот этап называется начальной репликацией.

По окончании первоначальной репликации начинается процесс разностной синхронизации. На этом этапе виртуальная машина считается защищенной и для нее можно выполнять тестовую отработку отказа. Когда завершится начальная репликация, реплицируемый элемент, представляющий виртуальную машину, примет состояние репликации "Защищено".

Чтобы отследить текущее состояние и работоспособность репликации для виртуальной машины, получите сведения о репликации для соответствующего защищенного элемента.

```azurepowershell
Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $PrimaryProtContainer | Select FriendlyName, ProtectionState, ReplicationHealth
```

11. Чтобы выполнить тестовую отработку отказа, проверить и очистить данные после тестовой отработки, выполните [эти](./azure-to-azure-powershell.md#do-a-test-failover-validate-and-cleanup-test-failover) действия.
12. Для отработки отказа выполните действия [здесь](./azure-to-azure-powershell.md#fail-over-to-azure).
13. Для защиты и восстановления исходного региона используйте следующий командлет PowerShell:

```azurepowershell
#Create Cache storage account for replication logs in the primary region
$WestUSCacheStorageAccount = New-AzStorageAccount -Name "a2acachestoragewestus" -ResourceGroupName "A2AdemoRG" -Location 'West US' -SkuName Standard_LRS -Kind Storage


#Use the recovery protection container, new cache storage account in West US and the source region VM resource group 
Update-AzRecoveryServicesAsrProtectionDirection -ReplicationProtectedItem $ReplicationProtectedItem -AzureToAzure -ProtectionContainerMapping $WusToEusPCMapping -LogStorageAccountId $WestUSCacheStorageAccount.Id -RecoveryResourceGroupID $sourceVMResourcegroup.ResourceId -RecoveryProximityPlacementGroupId $vm.ProximityPlacementGroup.Id
```

14. Для отключения репликации выполните описанные [здесь](./azure-to-azure-powershell.md#disable-replication) действия.

### <a name="vmware-to-azure-via-powershell"></a>VMware в Azure с помощью PowerShell

1. Убедитесь, что для аварийного восстановления в Azure [подготовлены локальные серверы VMware](./vmware-azure-tutorial-prepare-on-premises.md).
2. Войдите в свою учетную запись и настройте подписку, как указано [здесь](./vmware-azure-disaster-recovery-powershell.md#log-into-azure).
3. [Настройте](./vmware-azure-disaster-recovery-powershell.md#set-up-a-recovery-services-vault) хранилище служб восстановления и [задайте контекст восстановления](./vmware-azure-disaster-recovery-powershell.md#set-the-vault-context).
4. [Подтвердите](./vmware-azure-disaster-recovery-powershell.md#validate-vault-registration) регистрацию хранилища.
5. [Создайте](./vmware-azure-disaster-recovery-powershell.md#create-a-replication-policy) политику репликации.
6. [Добавьте](./vmware-azure-disaster-recovery-powershell.md#add-a-vcenter-server-and-discover-vms) сервер vCenter, обнаруживайте виртуальные машины и [создавайте](./vmware-azure-disaster-recovery-powershell.md#create-storage-accounts-for-replication) учетные записи хранения для репликации.
7. Сведения о репликации виртуальных машин VMware доступны здесь. Кроме того, необходимо выполнить следующий командлет PowerShell:

```azurepowershell
#Get the target resource group to be used
$ResourceGroup = Get-AzResourceGroup -Name "VMwareToAzureDrPs"

#Get the target virtual network to be used
$RecoveryVnet = Get-AzVirtualNetwork -Name "ASR-vnet" -ResourceGroupName "asrrg"

#Get the protection container mapping for replication policy named ReplicationPolicy
$PolicyMap = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $ProtectionContainer | where PolicyFriendlyName -eq "ReplicationPolicy"

#Get the protectable item corresponding to the virtual machine CentOSVM1
$VM1 = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $ProtectionContainer -FriendlyName "CentOSVM1"

# Enable replication for virtual machine CentOSVM1 using the Az.RecoveryServices module 2.0.0 onwards to replicate to managed disks
# The name specified for the replicated item needs to be unique within the protection container. Using a random GUID to ensure uniqueness
$Job_EnableReplication1 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM1 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -ProcessServer $ProcessServers[1] -Account $AccountHandles[2] -RecoveryResourceGroupId $ResourceGroup.ResourceId -logStorageAccountId $LogStorageAccount.Id -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1" -RecoveryProximityPlacementGroupId $targetPpg.Id
```

8. Состояние репликации и работоспособность репликации виртуальной машины можно проверить с помощью командлета Get-ASRReplicationProtectedItem.

```azurepowershell
Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $ProtectionContainer | Select FriendlyName, ProtectionState, ReplicationHealth
```

9. Настройте параметры отработки отказа, выполняя описанные [здесь](./vmware-azure-disaster-recovery-powershell.md#configure-failover-settings) действия.
10. [Выполните](./vmware-azure-disaster-recovery-powershell.md#run-a-test-failover) тестовую отработку отказа.
11. Выполните отработку отказа в Azure, следуя инструкциям [здесь](./vmware-azure-disaster-recovery-powershell.md#fail-over-to-azure).

### <a name="hyper-v-to-azure-via-powershell"></a>Hyper-V в Azure с помощью PowerShell

1. Убедитесь, что для аварийного восстановления в Azure [подготовлены локальные серверы Hyper-V](./hyper-v-prepare-on-premises-tutorial.md).
2. [Выполните вход](./hyper-v-azure-powershell-resource-manager.md#step-1-sign-in-to-your-azure-account) в Azure.
3. [Настройте](./hyper-v-azure-powershell-resource-manager.md#step-2-set-up-the-vault) хранилище и [задайте](./hyper-v-azure-powershell-resource-manager.md#step-3-set-the-recovery-services-vault-context) контекст хранилища служб восстановления.
4. [Создайте](./hyper-v-azure-powershell-resource-manager.md#step-4-create-a-hyper-v-site) сайт Hyper-V.
5. [Установите](./hyper-v-azure-powershell-resource-manager.md#step-5-install-the-provider-and-agent) поставщик и агент.
6. [Создайте](./hyper-v-azure-powershell-resource-manager.md#step-6-create-a-replication-policy) политику репликации.
7. Включите репликацию, выполнив инструкции ниже. 
    
    а. Получите защищаемый элемент, соответствующий виртуальной машине, которую необходимо защитить, выполнив следующую команду:

    ```azurepowershell
    $VMFriendlyName = "Fabrikam-app"          #Name of the VM
    $ProtectableItem = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName
    ```
    b. Включите защиту виртуальной машины. Если к виртуальной машине, которую необходимо защитить, подключено несколько дисков, нужно указать диск операционной системы (ОС) с помощью параметра OSDiskName.
    
    ```azurepowershell
    $OSType = "Windows"          # "Windows" or "Linux"
    $DRjob = New-AzRecoveryServicesAsrReplicationProtectedItem -ProtectableItem $VM -Name $VM.Name -ProtectionContainerMapping $ProtectionContainerMapping -RecoveryAzureStorageAccountId   $StorageAccountID -OSDiskName $OSDiskNameList[$i] -OS $OSType -RecoveryResourceGroupId $ResourceGroupID -RecoveryProximityPlacementGroupId $targetPpg.Id
    ```
    c. Дождитесь, пока виртуальные машины перейдут в защищенное состояние после начальной репликации. Это может занять некоторое время в зависимости от таких факторов, как объем данных для репликации и доступная пропускная способность вышестоящих служб в Azure. Когда виртуальные машины перейдут в защищенное состояние, параметры State и StateDescription задания обновятся следующим образом: 
    
    ```azurepowershell
    $DRjob = Get-AzRecoveryServicesAsrJob -Job $DRjob
    $DRjob | Select-Object -ExpandProperty State

    $DRjob | Select-Object -ExpandProperty StateDescription
    ```
    d. Обновите свойства восстановления (например, размер роли виртуальной машины) и сеть Azure, к которой подключится сетевой адаптер виртуальной машины после отработки отказа.

    ```azurepowershell
    $nw1 = Get-AzVirtualNetwork -Name "FailoverNw" -ResourceGroupName "MyRG"

    $VMFriendlyName = "Fabrikam-App"

    $rpi = Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $protectionContainer -FriendlyName $VMFriendlyName

    $UpdateJob = Set-AzRecoveryServicesAsrReplicationProtectedItem -InputObject $rpi -PrimaryNic $VM.NicDetailsList[0].NicId -RecoveryNetworkId $nw1.Id -RecoveryNicSubnetName $nw1.Subnets[0].Name

    $UpdateJob = Get-AzRecoveryServicesAsrJob -Job $UpdateJob

    $UpdateJob | Select-Object -ExpandProperty state

    Get-AzRecoveryServicesAsrJob -Job $job | Select-Object -ExpandProperty state
    ```
8. Выполните тестовую [отработку отказа](./hyper-v-azure-powershell-resource-manager.md#step-8-run-a-test-failover).


## <a name="next-steps"></a>Дальнейшие действия

Для повторной защиты и отработки отказа VMware в Azure выполните описанные [здесь](./vmware-azure-prepare-failback.md) действия.

Чтобы выполнить отработку отказа для Hyper-V в Azure, выполните действия, описанные [здесь](./site-recovery-failover.md), а чтобы выполнить восстановление, выполните действия, описанные [здесь](./hyper-v-azure-failback.md).

Дополнительные сведения см. в статье [Отработка отказа в Site Recovery](site-recovery-failover.md).