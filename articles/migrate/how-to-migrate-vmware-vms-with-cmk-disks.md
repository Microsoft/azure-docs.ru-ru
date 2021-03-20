---
title: Перенос виртуальных машин VMware в Azure с помощью шифрования на стороне сервера (SSE) и ключей, управляемых клиентом (CMK), с помощью миграции сервера миграция Azure
description: Узнайте, как перенести виртуальные машины VMware в Azure с помощью шифрования на стороне сервера (SSE) и ключей, управляемых клиентом (CMK), с помощью миграции сервера миграцией Azure.
author: anvar-ms
ms.author: anvar
ms.manager: bsiva
ms.topic: how-to
ms.date: 03/12/2020
ms.openlocfilehash: 8a174c3b2bfb390eb7d691ae1bdcb0e28dde9032
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96751093"
---
# <a name="migrate-vmware-vms-to-azure-vms-enabled-with-server-side-encryption-and-customer-managed-keys"></a>Перенос виртуальных машин VMware на виртуальные машины Azure, поддерживающие шифрование на стороне сервера и управляемые клиентом ключи

В этой статье описывается, как перенести виртуальные машины VMware на виртуальные машины Azure с помощью шифрования на стороне сервера (SSE) с ключами, управляемыми клиентом (CMK), с использованием миграции сервера в Azure Migration (без агента).

Портал миграции сервера Azure Migration позволяет [переносить виртуальные машины VMware в Azure с помощью репликации без агента.](tutorial-migrate-vmware.md) Сейчас интерфейс портала не поддерживает возможность включения SSE с CMK для реплицированных дисков в Azure. Возможность включения SSE с CMK для реплицируемых дисков в настоящее время доступна только с помощью REST API. В этой статье вы узнаете, как создать и развернуть [шаблон Azure Resource Manager](../azure-resource-manager/templates/overview.md) для репликации виртуальной машины VMware и настроить реплицированные диски в Azure для использования SSE с CMK.

В примерах, приведенных в этой статье, используется [Azure PowerShell](/powershell/azure/new-azureps-module-az) для выполнения задач, необходимых для создания и развертывания шаблона диспетчер ресурсов.

Дополнительные [сведения](../virtual-machines/disk-encryption.md) о шифровании на стороне сервера (SSE) с помощью управляемых пользователем ключей (CMK) для управляемых дисков.

## <a name="prerequisites"></a>Предварительные условия

- [Ознакомьтесь с руководством](tutorial-migrate-vmware.md) по переносу виртуальных машин VMware в Azure с помощью безагентной репликации, чтобы понять требования к инструментам.
- [Выполните эти инструкции](./create-manage-projects.md) , чтобы создать проект службы "миграция Azure" и добавить в проект средство **миграции для Azure Migration: Server** .
- [Выполните эти инструкции](how-to-set-up-appliance-vmware.md) , чтобы настроить устройство "миграция Azure" для VMware в локальной среде и завершить обнаружение.

## <a name="prepare-for-replication"></a>Подготовка к репликации

После завершения обнаружения виртуальных машин в строке обнаруженные серверы на плитке миграция сервера отобразится число виртуальных машин VMware, обнаруженных устройством.

Перед началом репликации виртуальных машин необходимо подготовить инфраструктуру репликации.

1. Создайте экземпляр служебной шины в целевом регионе. Шина обслуживания используется локальным устройством миграции Azure для взаимодействия со службой миграции сервера для координации репликации и миграции.
2. Создайте учетную запись хранения для перемещения журналов операций из репликации.
3. Создайте учетную запись хранения, в которую устройство "миграция Azure" отправляет данные репликации.
4. Создайте Key Vault и настройте Key Vault для управления маркерами подписанного общего доступа для доступа к BLOB-объектам в учетных записях хранения, созданных на шаге 3 и 4.
5. Создайте маркер подписанного URL-канала для служебной шины, созданной на шаге 1, и создать секрет для маркера в Key Vault, созданном на предыдущем шаге.
6. Создайте политику доступа Key Vault, чтобы предоставить локальному устройству миграция Azure (с помощью приложения AAD устройства) и службе миграции сервера доступ к Key Vault.
7. Создайте политику репликации и настройте в службе миграции сервера сведения о инфраструктуре репликации, созданной на предыдущем шаге.

Инфраструктура репликации должна быть создана в целевом регионе Azure для миграции и в целевой подписке Azure, в которую выполняется миграция виртуальных машин.

Портал миграции сервера упрощает подготовку инфраструктуры репликации, автоматически выполняя эту процедуру при репликации виртуальной машины в первый раз в проекте. В этой статье предполагается, что вы уже реплицируете одну или несколько виртуальных машин с помощью интерфейса портала и что инфраструктура репликации уже создана. Мы рассмотрим, как найти сведения о существующей инфраструктуре репликации и как использовать эти сведения в качестве входных данных для шаблона диспетчер ресурсов, который будет использоваться для настройки репликации с помощью CMK.

### <a name="identifying-replication-infrastructure-components"></a>Определение компонентов инфраструктуры репликации

1. На портал Azure перейдите на страницу группы ресурсов и выберите группу ресурсов, в которой был создан проект службы "миграция Azure".
2. В меню слева выберите **развертывания** и найдите имя развертывания, начинающееся со строки *"Microsoft. MigrateV2. VMwareV2EnableMigrate"*. Вы увидите список шаблонов диспетчер ресурсов, созданных с помощью портала, чтобы настроить репликацию для виртуальных машин в этом проекте. Мы скачиваем один шаблон и используем его в качестве основы для подготовки шаблона к репликации с помощью CMK.
3. Чтобы скачать шаблон, выберите любое развертывание, соответствующее шаблону строки на предыдущем шаге > выберите **шаблон** в меню слева > щелкните **скачать** в верхнем меню. Сохраните template.jsв файле локально. Вы измените этот файл шаблона на предыдущем шаге.

## <a name="create-a-disk-encryption-set"></a>Создание набора шифрования дисков

Объект набора шифрования дисков сопоставляет управляемые диски с Key Vault, где содержится CMK для использования в SSE. Чтобы реплицировать виртуальные машины с помощью CMK, необходимо создать набор шифрования дисков и передать его в качестве входных данных для операции репликации.

Выполните [приведенный ниже пример,](../virtual-machines/windows/disks-enable-customer-managed-keys-powershell.md) чтобы создать набор шифрования дисков с помощью Azure PowerShell. Убедитесь, что набор шифрования дисков создан в целевой подписке, в которую выполняется миграция виртуальных машин, и в целевом регионе Azure для миграции.

Набор шифрования дисков можно настроить для шифрования управляемых дисков с помощью ключа, управляемого клиентом, или для двойного шифрования с ключом, управляемым клиентом, и ключом платформы. Чтобы использовать параметр двойное шифрование при неактивных данных, настройте набор шифрования диска, как описано [здесь](../virtual-machines/windows/disks-enable-double-encryption-at-rest-powershell.md).

В примере, показанном ниже, для набора шифрования диска настроено использование ключа, управляемого клиентом.

```azurepowershell
$Location = "southcentralus"                           #Target Azure region for migration 
$TargetResourceGroupName = "ContosoMigrationTarget"
$KeyVaultName = "ContosoCMKKV"
$KeyName = "ContosoCMKKey"
$KeyDestination = "Software"
$DiskEncryptionSetName = "ContosoCMKDES"

$KeyVault = New-AzKeyVault -Name $KeyVaultName -ResourceGroupName $TargetResourceGroupName -Location $Location -EnableSoftDelete -EnablePurgeProtection

$Key = Add-AzKeyVaultKey -VaultName $KeyVaultName -Name $KeyName -Destination $KeyDestination

$desConfig = New-AzDiskEncryptionSetConfig -Location $Location -SourceVaultId $KeyVault.ResourceId -KeyUrl $Key.Key.Kid -IdentityType SystemAssigned

$des = New-AzDiskEncryptionSet -Name $DiskEncryptionSetName -ResourceGroupName $TargetResourceGroupName -InputObject $desConfig

Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ObjectId $des.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get

New-AzRoleAssignment -ResourceName $KeyVaultName -ResourceGroupName $TargetResourceGroupName -ResourceType "Microsoft.KeyVault/vaults" -ObjectId $des.Identity.PrincipalId -RoleDefinitionName "Reader"
```

## <a name="get-details-of-the-vmware-vm-to-migrate"></a>Получение сведений о переносимой виртуальной машине VMware

На этом шаге вы будете использовать Azure PowerShell для получения сведений о виртуальной машине, которую необходимо перенести. Эти сведения будут использоваться для создания шаблона диспетчер ресурсов для репликации. В частности, есть два свойства, представляющие интерес:

- Идентификатор ресурса компьютера для обнаруженных виртуальных машин.
- Список дисков виртуальной машины и их идентификаторов дисков.

```azurepowershell

$ProjectResourceGroup = "ContosoVMwareCMK"   #Resource group that the Azure Migrate Project is created in
$ProjectName = "ContosoVMwareCMK"            #Name of the Azure Migrate Project

$solution = Get-AzResource -ResourceGroupName $ProjectResourceGroup -ResourceType Microsoft.Migrate/MigrateProjects/solutions -ExpandProperties -ResourceName $ProjectName | where Name -eq "Servers-Discovery-ServerDis
covery"

# Displays one entry for each appliance in the project mapping the appliance to the VMware sites discovered through the appliance.
$solution.Properties.details.extendedDetails.applianceNameToSiteIdMapV2 | ConvertFrom-Json | select ApplianceName, SiteId
```
```Output
ApplianceName  SiteId
-------------  ------
VMwareApplianc /subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite
```

Скопируйте значение строки сайта, соответствующее устройству миграции Azure, с помощью которого была обнаружена виртуальная машина. В приведенном выше примере идентификатор узла имеет значение *«/Subscriptions/509099b2-9d2c-4636-B43E-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite»* .

```azurepowershell

#Replace value with SiteId from the previous step
$SiteId = "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite"
$SiteName = Get-AzResource -ResourceId $SiteId -ExpandProperties | Select-Object -ExpandProperty Name
$DiscoveredMachines = Get-AzResource -ResourceGroupName $ProjectResourceGroup -ResourceType Microsoft.OffAzure/VMwareSites/machines  -ExpandProperties -ResourceName $SiteName

#Get machine details
PS /home/bharathram> $MachineName = "FPL-W19-09"     #Replace string with VMware VM name of the machine to migrate
PS /home/bharathram> $machine = $Discoveredmachines | where {$_.Properties.displayName -eq $MachineName}
PS /home/bharathram> $machine.count   #Validate that only 1 VM was found matching this name.
```

Скопируйте значения ResourceId, Name и UUID диска для переносимого компьютера.
```Output
PS > $machine.Name
10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210
PS > $machine.ResourceId
/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210

PS > $machine.Properties.disks | select uuid, label, name, maxSizeInBytes

uuid                                 label       name    maxSizeInBytes
----                                 -----       ----    --------------
6000C291-5106-2aac-7a74-4f33c3ddb78c Hard disk 1 scsi0:0    42949672960
6000C293-39a1-bd70-7b24-735f0eeb79c4 Hard disk 2 scsi0:1    53687091200
6000C29e-cbee-4d79-39c7-d00dd0208aa9 Hard disk 3 scsi0:2    53687091200

```

## <a name="create-resource-manager-template-for-replication"></a>Создание шаблона диспетчер ресурсов для репликации

- Откройте файл шаблона диспетчер ресурсов, скачанный на шаге **Определение компонентов инфраструктуры репликации** в любом редакторе.
- Удалите все определения ресурсов из шаблона, за исключением ресурсов типа *"Microsoft. RecoveryServices/Vault/репликатионфабрикс/репликатионпротектионконтаинерс/репликатионмигратионитемс"* .
- Если имеется несколько определений ресурсов указанного выше типа, удалите все, кроме одного. Удалите все определения свойств **dependsOn** из определения ресурса.
- В конце этого шага у вас должен быть файл, который выглядит, как в примере ниже, и имеет тот же набор свойств.

```
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems",
            "apiVersion": "2018-01-10",
            "name": "ContosoMigration7371rsvault/VMware104e4replicationfabric/VMware104e4replicationcontainer/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_500937f3-805e-9414-11b1-f22923456e08",
            "properties": {
                "policyId": "/Subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.RecoveryServices/vaults/ContosoMigration7371rsvault/replicationPolicies/migrateVMware104e4sitepolicy",
                "providerSpecificDetails": {
                    "instanceType": "VMwareCbt",
                    "vmwareMachineId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_500937f3-805e-9414-11b1-f22923456e08",
                    "targetResourceGroupId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/PayrollRG",
                    "targetNetworkId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/PayrollRG/providers/Microsoft.Network/virtualNetworks/PayrollNW",
                    "targetSubnetName": "PayrollSubnet",
                    "licenseType": "NoLicenseType",
                    "disksToInclude": [
                        {
                            "diskId": "6000C295-dafe-a0eb-906e-d47cb5b05a1d",
                            "isOSDisk": "true",
                            "logStorageAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.Storage/storageAccounts/migratelsa1432469187",
                            "logStorageAccountSasSecretName": "migratelsa1432469187-cacheSas",
                            "diskType": "Standard_LRS"
                        }
                    ],
                    "dataMoverRunAsAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "snapshotRunAsAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.OffAzure/VMwareSites/VMware104e4site/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "targetBootDiagnosticsStorageAccountId": "/subscriptions/6785ea1f-ac40-4244-a9ce-94b12fd832ca/resourceGroups/ContosoMigration/providers/Microsoft.Storage/storageAccounts/migratelsa1432469187",
                    "targetVmName": "PayrollWeb04"
                }
            }
        }
    ]
}
```

- Измените свойство **Name** в определении ресурса. Замените строку, следующую за последней "/" в свойстве Name, значением *$Machine. Имя*(из предыдущего шага).
- Измените значение свойства **Properties. провидерспеЦификдетаилс. вмваремачинеид** на значение *$Machine. ResourceId*(из предыдущего шага).
- Задайте для параметров **таржетресаурцеграупид**, **ТАРЖЕТНЕТВОРКИД**, **таржетсубнетнаме** идентификатор целевой группы ресурсов, идентификатор ресурса целевой виртуальной сети и имя целевой подсети соответственно.
- Присвойте параметру **licenseType** значение "WindowsServer", чтобы применить преимущество гибридного использования Azure для этой виртуальной машины. Если эта виртуальная машина не подходит для Преимущество гибридного использования Azure, присвойте параметру **licenseType** значение нолиценсетипе.
- Измените значение свойства **таржетвмнаме** на нужное имя виртуальной машины Azure для ПЕРЕНЕСЕННОЙ виртуальной машины.
- При необходимости добавьте свойство с именем **таржетвмсизе** под свойством **таржетвмнаме** . В качестве значения свойства **таржетвмсизе** задайте требуемый размер виртуальной машины Azure для ПЕРЕНЕСЕННОЙ виртуальной машины.
- Свойство **дискстоинклуде** представляет собой список входных данных диска для репликации с каждым элементом списка, представляющим один локальный диск. Создайте столько элементов списка, сколько количество дисков на локальной виртуальной машине. Замените свойство **diskId** в элементе списка на UUID дисков, определенных на предыдущем шаге. Задайте для параметра **исосдиск** значение true для диска операционной системы виртуальной машины и значение false для всех остальных дисков. Оставьте свойства **логсторажеаккаунтид** и **логсторажеаккаунтсассекретнаме** без изменений. Задайте для параметра **дисктипе** тип управляемого диска Azure (*Standard_LRS, Premium_LRS, StandardSSD_LRS*), который будет использоваться для диска. Для дисков, которые необходимо зашифровать с помощью CMK, добавьте свойство с именем **дискенкриптионсетид** и задайте значение идентификатора ресурса созданного набора шифрования диска (**$DES. ID**) на шаге *Создание набора шифрования диска*
- Сохраните измененный файл шаблона. В приведенном выше примере измененный файл шаблона выглядит следующим образом:

```
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "Microsoft.RecoveryServices/vaults/replicationFabrics/replicationProtectionContainers/replicationMigrationItems",
            "apiVersion": "2018-01-10",
            "name": "ContosoVMwareCMK00ddrsvault/VMwareApplianca8bareplicationfabric/VMwareApplianca8bareplicationcontainer/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210",
            "properties": {
                "policyId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.RecoveryServices/vaults/ContosoVMwareCMK00ddrsvault/replicationPolicies/migrateVMwareApplianca8basitepolicy",
                "providerSpecificDetails": {
                    "instanceType": "VMwareCbt",
                    "vmwareMachineId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/machines/10-150-8-52-b090bef3-b733-5e34-bc8f-eb6f2701432a_50098f99-f949-22ca-642b-724ec6595210",
                    "targetResourceGroupId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoMigrationTarget",
                    "targetNetworkId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/cmkRTest/providers/Microsoft.Network/virtualNetworks/cmkvm1_vnet",
                    "targetSubnetName": "cmkvm1_subnet",
                    "licenseType": "NoLicenseType",
                    "disksToInclude": [
                        {
                            "diskId": "6000C291-5106-2aac-7a74-4f33c3ddb78c",
                            "isOSDisk": "true",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        },
                        {
                            "diskId": "6000C293-39a1-bd70-7b24-735f0eeb79c4",
                            "isOSDisk": "false",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        },
                        {
                            "diskId": "6000C29e-cbee-4d79-39c7-d00dd0208aa9",
                            "isOSDisk": "false",
                            "logStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                            "logStorageAccountSasSecretName": "migratelsa1671875959-cacheSas",
                            "diskEncryptionSetId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/CONTOSOMIGRATIONTARGET/providers/Microsoft.Compute/diskEncryptionSets/ContosoCMKDES",
                            "diskType": "Standard_LRS"
                        }
                    ],
                    "dataMoverRunAsAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "snapshotRunAsAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.OffAzure/VMwareSites/VMwareApplianca8basite/runasaccounts/b090bef3-b733-5e34-bc8f-eb6f2701432a",
                    "targetBootDiagnosticsStorageAccountId": "/subscriptions/509099b2-9d2c-4636-b43e-bd5cafb6be69/resourceGroups/ContosoVMwareCMK/providers/Microsoft.Storage/storageAccounts/migratelsa1671875959",
                    "performAutoResync": "true",
                    "targetVmName": "FPL-W19-09"
                }
            }
        }
    ]
}
```

## <a name="set-up-replication"></a>Настройка репликации

Теперь можно развернуть измененный шаблон диспетчер ресурсов в группе ресурсов проекта, чтобы настроить репликацию для виртуальной машины. Узнайте, как [развертывать ресурсы с помощью шаблонов Azure Resource Manager и Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName $ProjectResourceGroup -TemplateFile "C:\Users\Administrator\Downloads\template.json"
```

```Output
DeploymentName          : template
ResourceGroupName       : ContosoVMwareCMK
ProvisioningState       : Succeeded
Timestamp               : 3/11/2020 8:52:00 PM
Mode                    : Incremental
TemplateLink            :
Parameters              :
Outputs                 :
DeploymentDebugLogLevel :

```

## <a name="next-steps"></a>Дальнейшие действия

[Мониторинг состояния репликации](tutorial-migrate-vmware.md#track-and-monitor) с помощью портала и выполнение тестов миграции и миграции.