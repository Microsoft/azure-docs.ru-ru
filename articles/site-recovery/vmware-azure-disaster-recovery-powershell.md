---
title: Настройка аварийного восстановления VMware с помощью PowerShell на сайте Azure Ревоери
description: Сведения о том, как настроить репликацию и отработку отказа в Azure для аварийного восстановления виртуальных машин VMware с помощью PowerShell и Azure Site Recovery.
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.date: 01/10/2020
ms.topic: conceptual
ms.author: sutalasi
ms.openlocfilehash: de25a3f9df04b09a7337dc889a688a171d98db28
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86129908"
---
# <a name="set-up-disaster-recovery-of-vmware-vms-to-azure-with-powershell"></a>Настройка аварийного восстановления виртуальных машин VMware в Azure с помощью PowerShell

В этой статье описывается репликация и отработка отказа виртуальных машин VMware в Azure с помощью Azure PowerShell.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> - Создание служб восстановления и указание контекста хранилища.
> - Проверка регистрации сервера в хранилище.
> - Настройка репликации, включая политику репликации. Добавление сервера vCenter и обнаружение виртуальных машин.
> - Добавление сервера vCenter и обнаружение
> - Создание учетных записей хранения для хранения журналов репликации или данных, а также репликация виртуальных машин.
> - Выполнение отработки отказа. Настройте параметры отработки отказа, выполняйте параметры для репликации виртуальных машин.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы

- Вам должны быть понятны [архитектура и компоненты сценария](vmware-azure-architecture.md).
- [Ознакомьтесь](./vmware-physical-azure-support-matrix.md) с требованиями поддержки для всех компонентов.
- У вас есть `Az`  модуль Azure PowerShell. Если вам необходимо установить или обновить Azure PowerShell, ознакомьтесь с этим [руководством по установке и настройке Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="log-into-azure"></a>Вход на портал Azure

Войдите в подписку Azure с помощью командлета Connect-AzAccount:

```azurepowershell
Connect-AzAccount
```
Выберите подписку Azure, в которую нужно реплицировать виртуальные машины VMware. Используйте командлет Get-AzSubscription, чтобы получить список подписок Azure, к которым у вас есть доступ. Выберите подписку Azure для работы с помощью командлета Select-AzSubscription.

```azurepowershell
Select-AzSubscription -SubscriptionName "ASR Test Subscription"
```
## <a name="set-up-a-recovery-services-vault"></a>Настройка хранилища служб восстановления

1. Создайте группу ресурсов, в которой будет создано хранилище служб восстановления. В следующем примере группа ресурсов VMwareDRtoAzurePS создана в регионе "Восточная Азия".

   ```azurepowershell
   New-AzResourceGroup -Name "VMwareDRtoAzurePS" -Location "East Asia"
   ```
   ```
   ResourceGroupName : VMwareDRtoAzurePS
   Location          : eastasia
   ProvisioningState : Succeeded
   Tags              :
   ResourceId        : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRtoAzurePS
   ```

2. Создайте хранилище служб восстановления. В следующем примере хранилище служб восстановления VMwareDRToAzurePs создано в регионе "Восточная Азия", в группе ресурсов, созданной на предыдущем шаге.

   ```azurepowershell
   New-AzRecoveryServicesVault -Name "VMwareDRToAzurePs" -Location "East Asia" -ResourceGroupName "VMwareDRToAzurePs"
   ```
   ```
   Name              : VMwareDRToAzurePs
   ID                : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs
   Type              : Microsoft.RecoveryServices/vaults
   Location          : eastasia
   ResourceGroupName : VMwareDRToAzurePs
   SubscriptionId    : xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
   ```

3. Скачайте ключ регистрации в хранилище. Ключ регистрации в хранилище используется для регистрации локального сервера конфигурации в этом хранилище. Регистрация является частью процесса установки программного обеспечения сервера конфигурации.

   ```azurepowershell
   #Get the vault object by name and resource group and save it to the $vault PowerShell variable 
   $vault = Get-AzRecoveryServicesVault -Name "VMwareDRToAzurePS" -ResourceGroupName "VMwareDRToAzurePS"

   #Download vault registration key to the path C:\Work
   Get-AzRecoveryServicesVaultSettingsFile -SiteRecovery -Vault $Vault -Path "C:\Work\"
   ```
   ```
   FilePath
   --------
   C:\Work\VMwareDRToAzurePs_2017-11-23T19-52-34.VaultCredentials
   ```

4. Используйте скачанный ключ регистрации в хранилище и следуйте инструкциям в статьях, представленных ниже, чтобы выполнить установку и регистрацию сервера конфигурации.
   - [Выбор целевых объектов для защиты](vmware-azure-set-up-source.md#choose-your-protection-goals)
   - [Настройка исходной среды](vmware-azure-set-up-source.md#set-up-the-configuration-server)

### <a name="set-the-vault-context"></a>Задание контекста хранилища

Задайте контекст хранилища с помощью командлета Set-ASRVaultContext. После этого последующие операции Azure Site Recovery в сеансе PowerShell будут выполняться в контексте выбранного хранилища.

> [!TIP]
> Модуль Azure Site Recovery PowerShell (az. RecoveryServices Module) поставляется с простым использованием псевдонимов для большинства командлетов. Командлеты в модуле принимают форму *\<Operation> - **азрековерисервицесаср** \<Object>* и имеют эквивалентные псевдонимы, которые принимают форму *\<Operation> - **ASR** \<Object>*. Можно заменить псевдонимы командлетов для простоты использования.

В следующем примере данные хранилища из переменной $vault используется для указания контекста хранилища для сеанса PowerShell.

   ```azurepowershell
   Set-ASRVaultContext -Vault $vault
   ```
   ```
   ResourceName      ResourceGroupName ResourceNamespace          ResourceType
   ------------      ----------------- -----------------          -----------
   VMwareDRToAzurePs VMwareDRToAzurePs Microsoft.RecoveryServices vaults
   ```

В качестве альтернативы командлету Set-ASRVaultContext можно также использовать командлет Import-AzRecoveryServicesAsrVaultSettingsFile, чтобы задать контекст хранилища. Укажите путь, по которому файл ключа регистрации хранилища будет располагаться в качестве параметра-Path для командлета Import-AzRecoveryServicesAsrVaultSettingsFile. Пример:

   ```azurepowershell
   Get-AzRecoveryServicesVaultSettingsFile -SiteRecovery -Vault $Vault -Path "C:\Work\"
   Import-AzRecoveryServicesAsrVaultSettingsFile -Path "C:\Work\VMwareDRToAzurePs_2017-11-23T19-52-34.VaultCredentials"
   ```
Последующих разделах данной статьи предполагается, контекст хранилища для операций Azure Site Recovery уже задан.

## <a name="validate-vault-registration"></a>Проверка регистрации в хранилище

В этом примере используются следующие компоненты:

- Сервер конфигурации **ConfigurationServer**, зарегистрированный в этом хранилище.
- Дополнительный сервер обработки **ScaleOut-ProcessServer**, зарегистрированный на сервере *ConfigurationServer*.
- Учетные записи **vCenter_account**, **WindowsAccount** и **LinuxAccount**, настроенные на сервере конфигурации. Эти учетные записи используются, чтобы добавить сервер vCenter для обнаружения виртуальных машин, а также чтобы принудительно установить программное обеспечение Mobility Service на серверах Windows и Linux, которые должны быть реплицированы.

1. Зарегистрированные серверы конфигурации представлены объектом структуры в Site Recovery. Получите список объектов структуры в хранилище и определите сервер конфигурации.

   ```azurepowershell
   # Verify that the Configuration server is successfully registered to the vault
   $ASRFabrics = Get-AzRecoveryServicesAsrFabric
   $ASRFabrics.count
   ```
   ```
   1
   ```
   ```azurepowershell
   #Print details of the Configuration Server
   $ASRFabrics[0]
   ```
   ```
   Name                  : 2c33d710a5ee6af753413e97f01e314fc75938ea4e9ac7bafbf4a31f6804460d
   FriendlyName          : ConfigurationServer
   ID                    : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs/replicationFabrics
                           /2c33d710a5ee6af753413e97f01e314fc75938ea4e9ac7bafbf4a31f6804460d
   Type                  : Microsoft.RecoveryServices/vaults/replicationFabrics
   FabricType            : VMware
   SiteIdentifier        : ef7a1580-f356-4a00-aa30-7bf80f952510
   FabricSpecificDetails : Microsoft.Azure.Commands.RecoveryServices.SiteRecovery.ASRVMWareSpecificDetails
   ```

2. Определите серверы обработки, которые можно использовать для репликации компьютеров.

   ```azurepowershell
   $ProcessServers = $ASRFabrics[0].FabricSpecificDetails.ProcessServers
   for($i=0; $i -lt $ProcessServers.count; $i++) {
    "{0,-5} {1}" -f $i, $ProcessServers[$i].FriendlyName
   }
   ```
   ```
   0     ScaleOut-ProcessServer
   1     ConfigurationServer
   ```

   Из выходных данных выше ***$ProcessServers [0]** _ соответствует _scaleing-ProcessServer * и ***$ProcessServers [1]**_ соответствует роли сервера обработки на _ConfigurationServer *

3. Определите учетные записи, которые были настроены на сервере конфигурации.

   ```azurepowershell
   $AccountHandles = $ASRFabrics[0].FabricSpecificDetails.RunAsAccounts
   #Print the account details
   $AccountHandles
   ```
   ```
   AccountId AccountName
   --------- -----------
   1         vCenter_account
   2         WindowsAccount
   3         LinuxAccount
   ```

   Из выходных данных выше ***$AccountHandles [0]** _ соответствует учетной записи _vCenter_account *, ***$AccountHandles [1]**_ в учетную запись _WindowsAccount *, а ***$AccountHandles [2]** —_ _LinuxAccount *.

## <a name="create-a-replication-policy"></a>Создание политики репликации

На этом шаге создаются две политики репликации. Одна политика предназначена для репликации виртуальных машин VMware в Azure, а другая — для репликации виртуальных машин, работающих в Azure, после отработки отказа обратно на локальный сайт VMware.

> [!NOTE]
> Большинство операций Azure Site Recovery выполняется асинхронно. При запуске операции отправляется задание Azure Site Recovery и возвращается объект отслеживания задания. Этот объект отслеживания задания можно использовать для наблюдения за состоянием операции.

1. Создайте политику репликации *ReplicationPolicy* для репликации виртуальных машин VMware в Azure с указанными свойствами.

   ```azurepowershell
   $Job_PolicyCreate = New-AzRecoveryServicesAsrPolicy -VMwareToAzure -Name "ReplicationPolicy" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4 -RPOWarningThresholdInMinutes 60

   # Track Job status to check for completion
   while (($Job_PolicyCreate.State -eq "InProgress") -or ($Job_PolicyCreate.State -eq "NotStarted")){
           sleep 10;
           $Job_PolicyCreate = Get-ASRJob -Job $Job_PolicyCreate
   }

   #Display job status
   $Job_PolicyCreate
   ```
   ```
   Name             : 8d18e2d9-479f-430d-b76b-6bc7eb2d0b3e
   ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs/replicationJobs/8d18e2d
                      9-479f-430d-b76b-6bc7eb2d0b3e
   Type             :
   JobType          : AddProtectionProfile
   DisplayName      : Create replication policy
   ClientRequestId  : a162b233-55d7-4852-abac-3d595a1faac2 ActivityId: 9895234a-90ea-4c1a-83b5-1f2c6586252a
   State            : Succeeded
   StateDescription : Completed
   StartTime        : 11/24/2017 2:49:24 AM
   EndTime          : 11/24/2017 2:49:23 AM
   TargetObjectId   : ab31026e-4866-5440-969a-8ebcb13a372f
   TargetObjectType : ProtectionProfile
   TargetObjectName : ReplicationPolicy
   AllowedActions   :
   Tasks            : {Prerequisites check for creating the replication policy, Creating the replication policy}
   Errors           : {}
   ```

2. Создайте политику репликации для восстановления размещения из Azure на локальный сайт VMware.

   ```azurepowershell
   $Job_FailbackPolicyCreate = New-AzRecoveryServicesAsrPolicy -AzureToVMware -Name "ReplicationPolicy-Failback" -RecoveryPointRetentionInHours 24 -ApplicationConsistentSnapshotFrequencyInHours 4 -RPOWarningThresholdInMinutes 60
   ```

   Используйте сведения о задании в *$Job_FailbackPolicyCreate* для отслеживания операции до завершения.

   * Создайте сопоставление контейнера защиты для сопоставления политик репликации с сервером конфигурации.

   ```azurepowershell
   #Get the protection container corresponding to the Configuration Server
   $ProtectionContainer = Get-AzRecoveryServicesAsrProtectionContainer -Fabric $ASRFabrics[0]

   #Get the replication policies to map by name.
   $ReplicationPolicy = Get-AzRecoveryServicesAsrPolicy -Name "ReplicationPolicy"
   $FailbackReplicationPolicy = Get-AzRecoveryServicesAsrPolicy -Name "ReplicationPolicy-Failback"

   # Associate the replication policies to the protection container corresponding to the Configuration Server.

   $Job_AssociatePolicy = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "PolicyAssociation1" -PrimaryProtectionContainer $ProtectionContainer -Policy $ReplicationPolicy

   # Check the job status
   while (($Job_AssociatePolicy.State -eq "InProgress") -or ($Job_AssociatePolicy.State -eq "NotStarted")){
           sleep 10;
           $Job_AssociatePolicy = Get-ASRJob -Job $Job_AssociatePolicy
   }
   $Job_AssociatePolicy.State

   <# In the protection container mapping used for failback (replicating failed over virtual machines
      running in Azure, to the primary VMware site.) the protection container corresponding to the
      Configuration server acts as both the Primary protection container and the recovery protection
      container
   #>
    $Job_AssociateFailbackPolicy = New-AzRecoveryServicesAsrProtectionContainerMapping -Name "FailbackPolicyAssociation" -PrimaryProtectionContainer $ProtectionContainer -RecoveryProtectionContainer $ProtectionContainer -Policy $FailbackReplicationPolicy

   # Check the job status
   while (($Job_AssociateFailbackPolicy.State -eq "InProgress") -or ($Job_AssociateFailbackPolicy.State -eq "NotStarted")){
           sleep 10;
           $Job_AssociateFailbackPolicy = Get-ASRJob -Job $Job_AssociateFailbackPolicy
   }
   $Job_AssociateFailbackPolicy.State

   ```

## <a name="add-a-vcenter-server-and-discover-vms"></a>Добавление сервера vCenter Server и обнаружение виртуальных машин

Добавьте сервер vCenter Server, указав его IP-адрес или имя узла. Параметр **-Port** задает порт на сервере vCenter Server для подключения, параметр **-Name** задает понятное имя, используемое для сервера vCenter Server, а параметр **-Account** задает дескриптор учетной записи на сервере конфигурации, который следует использовать для поиска виртуальных машин, управляемых сервером vCenter Server.

```azurepowershell
# The $AccountHandles[0] variable holds details of vCenter_account

$Job_AddvCenterServer = New-AzRecoveryServicesAsrvCenter -Fabric $ASRFabrics[0] -Name "MyvCenterServer" -IpOrHostName "10.150.24.63" -Account $AccountHandles[0] -Port 443

#Wait for the job to complete and ensure it completed successfully

while (($Job_AddvCenterServer.State -eq "InProgress") -or ($Job_AddvCenterServer.State -eq "NotStarted")) {
        sleep 30;
        $Job_AddvCenterServer = Get-ASRJob -Job $Job_AddvCenterServer
}
$Job_AddvCenterServer
```
```
Name             : 0f76f937-f9cf-4e0e-bf27-10c9d1c252a4
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs/replicationJobs/0f76f93
                   7-f9cf-4e0e-bf27-10c9d1c252a4
Type             :
JobType          : DiscoverVCenter
DisplayName      : Add vCenter server
ClientRequestId  : a2af8892-5686-4d64-a528-10445bc2f698 ActivityId: 7ec05aad-002e-4da0-991f-95d0de7a9f3a
State            : Succeeded
StateDescription : Completed
StartTime        : 11/24/2017 2:41:47 AM
EndTime          : 11/24/2017 2:44:37 AM
TargetObjectId   : 10.150.24.63
TargetObjectType : VCenter
TargetObjectName : MyvCenterServer
AllowedActions   :
Tasks            : {Adding vCenter server}
Errors           : {}
```

## <a name="create-storage-accounts-for-replication"></a>Создание учетных записей хранения для репликации

**Для записи на управляемый диск используйте [PowerShell AZ. RecoveryServices Module 2.0.0](https://www.powershellgallery.com/packages/Az.RecoveryServices/2.0.0-preview) .** Он требует только создания учетной записи хранения журнала. Рекомендуется использовать стандартный тип учетной записи и избыточность LRS, так как она используется для хранения только временных журналов. Убедитесь, что учетная запись хранения создана в том же регионе Azure, что и хранилище.

Если вы используете версию AZ. RecoveryServices Module старше 2.0.0, выполните следующие действия, чтобы создать учетные записи хранения. Они будут использоваться позже для репликации виртуальных машин. Убедитесь, что эти учетные записи хранения создаются в том же регионе Azure, в котором находится хранилище. Этот шаг можно пропустить, если вы планируете использовать для репликации существующую учетную запись хранения.

> [!NOTE]
> При репликации локальных виртуальных машин в учетную запись хранения уровня "Премиум" необходимо указать дополнительную учетную запись уровня "Стандартный" (учетную запись хранения журналов). В учетной записи хранения журналов временно хранятся журналы репликации, пока их не можно будет применить к целевому хранилищу уровня "Премиум".
>

```azurepowershell

$PremiumStorageAccount = New-AzStorageAccount -ResourceGroupName "VMwareDRToAzurePs" -Name "premiumstorageaccount1" -Location "East Asia" -SkuName Premium_LRS

$LogStorageAccount = New-AzStorageAccount -ResourceGroupName "VMwareDRToAzurePs" -Name "logstorageaccount1" -Location "East Asia" -SkuName Standard_LRS

$ReplicationStdStorageAccount= New-AzStorageAccount -ResourceGroupName "VMwareDRToAzurePs" -Name "replicationstdstorageaccount1" -Location "East Asia" -SkuName Standard_LRS
```

## <a name="replicate-vmware-vms"></a>Репликация виртуальных машин VMware

Обнаружение виртуальных машин на сервере vCenter Server занимает около 15–20 минут. После этого для каждой обнаруженной виртуальной машины в Azure Site Recovery создается объект защищаемого элемента. На этом шаге три обнаруженные виртуальных машины реплицируются в учетные записи хранения Azure, созданные на предыдущем шаге.

Для защиты обнаруженной виртуальной машины потребуется следующее:

* Защищаемый элемент, который требуется реплицировать.
* Учетная запись хранения для репликации виртуальной машины (только в том случае, если выполняется репликация в учетную запись хранения). 
* Хранилище журналов требуется для защиты виртуальных машин в учетной записи хранения класса Premium или на управляемом диске.
* Сервер обработки для репликации. Список доступных серверов обработки был получен и сохранен в переменных ***$ProcessServers [0]** _ _(scaleing-ProcessServer) * и ***$ProcessServers [1]**_ _ (ConfigurationServer) *.
* Учетная запись, используемая для принудительной установки программного обеспечения Mobility Service на компьютерах. Список доступных учетных записей должен быть получен и сохранен в переменной ***$AccountHandles***.
* Сопоставление контейнера защиты для политики репликации, используемой для репликации.
* Группа ресурсов, в которой должны создаваться виртуальные машины при отработке отказа.
* (Необязательно.) Виртуальная сеть Azure и подсеть, к которым должны подключаться виртуальные машины после отработки отказа.

Теперь реплицируйте следующие виртуальные машины, используя параметры, заданные в этой таблице.


|Виртуальная машина  |Сервер обработки        |Учетная запись хранения              |Учетная запись хранения журналов  |Политика           |Учетная запись для установки службы Mobility Service|Целевая группа ресурсов  | Целевая виртуальная сеть  |Целевая подсеть  |
|-----------------|----------------------|-----------------------------|---------------------|-----------------|-----------------------------------------|-----------------------|-------------------------|---------------|
|CentOSVM1       |ConfigurationServer   |Н/Д| logstorageaccount1                 |ReplicationPolicy|LinuxAccount                             |VMwareDRToAzurePs      |ASR-vnet                 |Subnet-1       |
|Win2K12VM1       |ScaleOut-ProcessServer|premiumstorageaccount1       |logstorageaccount1   |ReplicationPolicy|WindowsAccount                           |VMwareDRToAzurePs      |ASR-vnet                 |Subnet-1       |   
|CentOSVM2       |ConfigurationServer   |replicationstdstorageaccount1| Н/Д                 |ReplicationPolicy|LinuxAccount                             |VMwareDRToAzurePs      |ASR-vnet                 |Subnet-1       |   


```azurepowershell

#Get the target resource group to be used
$ResourceGroup = Get-AzResourceGroup -Name "VMwareToAzureDrPs"

#Get the target virtual network to be used
$RecoveryVnet = Get-AzVirtualNetwork -Name "ASR-vnet" -ResourceGroupName "asrrg" 

#Get the protection container mapping for replication policy named ReplicationPolicy
$PolicyMap  = Get-AzRecoveryServicesAsrProtectionContainerMapping -ProtectionContainer $ProtectionContainer | where PolicyFriendlyName -eq "ReplicationPolicy"

#Get the protectable item corresponding to the virtual machine CentOSVM1
$VM1 = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $ProtectionContainer -FriendlyName "CentOSVM1"

# Enable replication for virtual machine CentOSVM1 using the Az.RecoveryServices module 2.0.0 onwards to replicate to managed disks
# The name specified for the replicated item needs to be unique within the protection container. Using a random GUID to ensure uniqueness
$Job_EnableReplication1 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM1 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -ProcessServer $ProcessServers[1] -Account $AccountHandles[2] -RecoveryResourceGroupId $ResourceGroup.ResourceId -logStorageAccountId $LogStorageAccount.Id -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1"

# Alternatively, if the virtual machine CentOSVM1 has CMK enabled disks, enable replication using Az module 3.3.0 onwards as below
# $diskID is the Disk Encryption Set ID to be used for all replica managed disks and target managed disks in the target region
$Job_EnableReplication1 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM1 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -ProcessServer $ProcessServers[1] -Account $AccountHandles[2] -RecoveryResourceGroupId $ResourceGroup.ResourceId -logStorageAccountId -DiskEncryptionSetId $diskId $LogStorageAccount.Id -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1"

#Get the protectable item corresponding to the virtual machine Win2K12VM1
$VM2 = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $ProtectionContainer -FriendlyName "Win2K12VM1"

# Enable replication for virtual machine Win2K12VM1
$Job_EnableReplication2 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM2 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -RecoveryAzureStorageAccountId $PremiumStorageAccount.Id -LogStorageAccountId $LogStorageAccount.Id -ProcessServer $ProcessServers[0] -Account $AccountHandles[1] -RecoveryResourceGroupId $ResourceGroup.ResourceId -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1"

#Get the protectable item corresponding to the virtual machine CentOSVM2
$VM3 = Get-AzRecoveryServicesAsrProtectableItem -ProtectionContainer $ProtectionContainer -FriendlyName "CentOSVM2"

# Enable replication for virtual machine CentOSVM2
$Job_EnableReplication3 = New-AzRecoveryServicesAsrReplicationProtectedItem -VMwareToAzure -ProtectableItem $VM3 -Name (New-Guid).Guid -ProtectionContainerMapping $PolicyMap -RecoveryAzureStorageAccountId $ReplicationStdStorageAccount.Id  -ProcessServer $ProcessServers[1] -Account $AccountHandles[2] -RecoveryResourceGroupId $ResourceGroup.ResourceId -RecoveryAzureNetworkId $RecoveryVnet.Id -RecoveryAzureSubnetName "Subnet-1"

```

После успешного завершения задания включения репликации для виртуальных машин запускается начальная репликация. Начальная репликация может занять некоторое время. Это зависит от объема реплицируемых данных и пропускной способности, доступной для репликации. После завершения начальной репликации виртуальная машина переходит в защищенное состояние. После перехода виртуальной машины в защищенное состояние можно выполнить ее тестовую отработку отказа, добавить эту виртуальную машину в планы восстановления и т. д.

Состояние репликации и работоспособность репликации виртуальной машины можно проверить с помощью командлета Get-ASRReplicationProtectedItem.

```azurepowershell
Get-AzRecoveryServicesAsrReplicationProtectedItem -ProtectionContainer $ProtectionContainer | Select FriendlyName, ProtectionState, ReplicationHealth
```
```
FriendlyName ProtectionState                 ReplicationHealth
------------ ---------------                 -----------------
CentOSVM1    Protected                       Normal
CentOSVM2    InitialReplicationInProgress    Normal
Win2K12VM1   Protected                       Normal
```

## <a name="configure-failover-settings"></a>Настройка параметров отработки отказа

Параметры отработки отказа для защищенных виртуальных машин можно изменить с помощью командлета Set-ASRReplicationProtectedItem. Ниже приведены некоторые из параметров, которые можно изменить с помощью этого командлета.
* Имя виртуальной машины, создаваемой при отработке отказа.
* Размер виртуальной машины, создаваемой при отработке отказа.
* Виртуальная сеть Azure и подсеть, к которым должны быть подключены сетевые адаптеры виртуальной машины при отработке отказа.
* Отработка отказа на управляемые диски.
* Применение преимуществ гибридного использования Azure.
* Назначение статического IP-адреса из целевой виртуальной сети для виртуальной машины при отработке отказа.

В этом примере мы изменяем размер виртуальной машины, которая будет создана при отработке отказа, для виртуальной машины *Win2K12VM1*, и указываем, что для отработки отказа используются управляемые диски.

```azurepowershell
$ReplicatedVM1 = Get-AzRecoveryServicesAsrReplicationProtectedItem -FriendlyName "Win2K12VM1" -ProtectionContainer $ProtectionContainer

Set-AzRecoveryServicesAsrReplicationProtectedItem -InputObject $ReplicatedVM1 -Size "Standard_DS11" -UseManagedDisk True
```
```
Name             : cafa459c-44a7-45b0-9de9-3d925b0e7db9
ID               : /Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/VMwareDRToAzurePs/providers/Microsoft.RecoveryServices/vaults/VMwareDRToAzurePs/replicationJobs/cafa459
                   c-44a7-45b0-9de9-3d925b0e7db9
Type             :
JobType          : UpdateVmProperties
DisplayName      : Update the virtual machine
ClientRequestId  : b0b51b2a-f151-4e9a-a98e-064a5b5131f3 ActivityId: ac2ba316-be7b-4c94-a053-5363f683d38f
State            : InProgress
StateDescription : InProgress
StartTime        : 11/24/2017 2:04:26 PM
EndTime          :
TargetObjectId   : 88bc391e-d091-11e7-9484-000c2955bb50
TargetObjectType : ProtectionEntity
TargetObjectName : Win2K12VM1
AllowedActions   :
Tasks            : {Update the virtual machine properties}
Errors           : {}
```

## <a name="run-a-test-failover"></a>Запуск тестовой отработки отказа

1. Запустите тестирование аварийного восстановления (тестовую отработку отказа), выполнив следующий код:

   ```azurepowershell
   #Test failover of Win2K12VM1 to the test virtual network "V2TestNetwork"

   #Get details of the test failover virtual network to be used
   TestFailovervnet = Get-AzVirtualNetwork -Name "V2TestNetwork" -ResourceGroupName "asrrg" 

   #Start the test failover operation
   $TFOJob = Start-AzRecoveryServicesAsrTestFailoverJob -ReplicationProtectedItem $ReplicatedVM1 -AzureVMNetworkId $TestFailovervnet.Id -Direction PrimaryToRecovery
   ```
2. После успешного выполнения тестового задания отработки отказа можно заметить, что в Azure создана виртуальная машина с добавленным к имени суффиксом *-Test* (в данном случае это Win2K12VM1-Test).
3. Теперь можно подключиться к этой виртуальной машине и проверить результаты тестовой отработки отказа.
4. Очистите результаты тестовой отработки отказа с помощью командлета Start-ASRTestFailoverCleanupJob. Эта операция удаляет виртуальную машину, созданную в рамках операции тестовой отработки отказа.

   ```azurepowershell
   $Job_TFOCleanup = Start-AzRecoveryServicesAsrTestFailoverCleanupJob -ReplicationProtectedItem $ReplicatedVM1
   ```

## <a name="fail-over-to-azure"></a>Отработка отказа с переходом в Azure

На этом шаге выполняется отработка отказа виртуальной машины Win2K12VM1 до определенной точки восстановления.

1. Получите список точек восстановления, доступных для отработки отказа:
   ```azurepowershell
   # Get the list of available recovery points for Win2K12VM1
   $RecoveryPoints = Get-AzRecoveryServicesAsrRecoveryPoint -ReplicationProtectedItem $ReplicatedVM1
   "{0} {1}" -f $RecoveryPoints[0].RecoveryPointType, $RecoveryPoints[0].RecoveryPointTime
   ```
   ```
   CrashConsistent 11/24/2017 5:28:25 PM
   ```
   ```azurepowershell

   #Start the failover job
   $Job_Failover = Start-AzRecoveryServicesAsrUnplannedFailoverJob -ReplicationProtectedItem $ReplicatedVM1 -Direction PrimaryToRecovery -RecoveryPoint $RecoveryPoints[0]
   do {
           $Job_Failover = Get-ASRJob -Job $Job_Failover;
           sleep 60;
   } while (($Job_Failover.State -eq "InProgress") -or ($JobFailover.State -eq "NotStarted"))
   $Job_Failover.State
   ```
   ```
   Succeeded
   ```

2. После успешной отработки отказа можно зафиксировать эту операцию и настроить обратную репликацию из Azure в локальное расположение VMware.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте, как автоматизировать дополнительные задачи с помощью [справочника по Azure Site Recovery PowerShell](/powershell/module/Az.RecoveryServices).
