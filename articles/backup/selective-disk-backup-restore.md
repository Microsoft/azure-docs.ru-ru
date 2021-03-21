---
title: Выборочное резервное копирование и восстановление для виртуальных машин Azure
description: В этой статье вы узнаете о выборочном резервном копировании и восстановлении диска с помощью решения для резервного копирования виртуальных машин Azure.
ms.topic: conceptual
ms.date: 07/17/2020
ms.custom: references_regions , devx-track-azurecli
ms.openlocfilehash: e82c959dc63222e8565243cc9ac805283cab6617
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102501846"
---
# <a name="selective-disk-backup-and-restore-for-azure-virtual-machines"></a>Выборочное резервное копирование и восстановление для виртуальных машин Azure

Azure Backup поддерживает резервное копирование всех дисков (операционной системы и данных) на ВИРТУАЛЬНОЙ машине вместе с помощью решения для резервного копирования виртуальных машин. Теперь с помощью функций резервного копирования и восстановления выборочных дисков можно создать резервную копию подмножества дисков данных в виртуальной машине. Это обеспечивает эффективное и экономичное решение для резервного копирования и восстановления. Каждая точка восстановления содержит только диски, входящие в операцию резервного копирования. Это позволяет получить подмножество дисков, восстановленных из заданной точки восстановления во время операции восстановления. Это относится как к восстановлению из моментальных снимков, так и к хранилищу.

## <a name="scenarios"></a>Сценарии

Это решение особенно полезно в следующих сценариях:

1. Если у вас есть важные данные для резервного копирования только на одном диске или в подмножестве дисков и вы не хотите создавать резервные копии остальных дисков, подключенных к виртуальной машине, чтобы снизить затраты на хранение резервных копий.
2. При наличии других решений резервного копирования для части виртуальной машины или данных. Например, если вы создаете резервную копию баз данных или данных с помощью другого решения резервного копирования рабочей нагрузки и хотите использовать резервное копирование на уровне виртуальной машины Azure для работы с остальными данными или дисками, чтобы создать эффективную и надежную систему с использованием лучших доступных возможностей.

С помощью PowerShell или Azure CLI можно настроить выборочную архивацию на диске виртуальной машины Azure.  С помощью скрипта можно включать или исключать диски данных, используя их номера LUN.  В настоящее время возможность настройки выборочного резервного копирования с помощью портал Azure ограничена параметром " **архивировать только диск ОС** ". Поэтому вы можете настроить резервное копирование виртуальной машины Azure с помощью диска операционной системы и исключить все подключенные к нему диски данных.

>[!NOTE]
> Диск ОС по умолчанию добавляется в резервную копию виртуальной машины и не может быть исключен.

## <a name="using-azure-cli"></a>Использование Azure CLI

Убедитесь, что вы используете AZ CLI версии 2.0.80 или более поздней. Версию CLI можно получить с помощью следующей команды:

```azurecli
az --version
```

Войдите в систему с ИДЕНТИФИКАТОРом подписки, в котором существует хранилище служб восстановления и виртуальная машина.

```azurecli
az account set -s {subscriptionID}
```

>[!NOTE]
>В каждой из приведенных ниже команд требуется только имя **resourcegroup** (не объект), соответствующее хранилищу.

### <a name="configure-backup-with-azure-cli"></a>Настройка резервного копирования с помощью Azure CLI

Во время операции настройки защиты необходимо указать параметр списка дисков с   /  параметром **исключения** включения, указав номера LUN дисков, которые должны быть включены в резервную копию или исключены из нее.

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --disk-list-setting include --diskslist {LUN number(s) separated by space}
```

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --disk-list-setting exclude --diskslist 0 1
```

Если виртуальная машина находится не в той же группе ресурсов, что и хранилище, то Группа ресурсов, в которой было создано хранилище **, относится к** этой группе. Вместо имени виртуальной машины укажите идентификатор виртуальной машины, как показано ниже.

```azurecli
az backup protection enable-for-vm  --resource-group {ResourceGroup} --vault-name {vaultname} --vm $(az vm show -g VMResourceGroup -n MyVm --query id --output tsv) --policy-name {policyname} --disk-list-setting include --diskslist {LUN number(s) separated by space}
```

### <a name="modify-protection-for-already-backed-up-vms-with-azure-cli"></a>Изменить защиту для уже резервных копий виртуальных машин с Azure CLI

```azurecli
az backup protection update-for-vm --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --disk-list-setting exclude --diskslist {LUN number(s) separated by space}
```

### <a name="backup-only-os-disk-during-configure-backup-with-azure-cli"></a>Создавать резервную копию только диска ОС во время настройки резервного копирования с помощью Azure CLI

```azurecli
az backup protection enable-for-vm --resource-group {resourcegroup} --vault-name {vaultname} --vm {vmname} --policy-name {policyname} --exclude-all-data-disks
```

### <a name="backup-only-os-disk-during-modify-protection-with-azure-cli"></a>Создавать резервную копию только диска ОС во время защиты от изменения с помощью Azure CLI

```azurecli
az backup protection update-for-vm --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --exclude-all-data-disks
```

### <a name="restore-disks-with-azure-cli"></a>Восстановление дисков с помощью Azure CLI

```azurecli
az backup restore restore-disks --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} -r {restorepoint} --target-resource-group {targetresourcegroup} --storage-account {storageaccountname} --diskslist {LUN number of the disk(s) to be restored}
```

### <a name="restore-only-os-disk-with-azure-cli"></a>Восстановление только диска ОС с Azure CLI

```azurecli
az backup restore restore-disks --resource-group {resourcegroup} --vault-name {vaultname} -c {vmname} -i {vmname} -r {restorepoint} } --target-resource-group {targetresourcegroup} --storage-account {storageaccountname} --restore-only-osdisk
```

### <a name="get-protected-item-to-get-disk-exclusion-details-with-azure-cli"></a>Получение защищенного элемента для получения сведений об исключении диска с Azure CLI

```azurecli
az backup item show -c {vmname} -n {vmname} --vault-name {vaultname} --resource-group {resourcegroup} --backup-management-type AzureIaasVM
```

К защищенному элементу добавляется дополнительный параметр **дискексклусионпропертиес** , как показано ниже:

```azurecli
"extendedProperties": {
      "diskExclusionProperties": {
        "diskLunList": [
          0,
          1
        ],
        "isInclusionList": true
      }
```

### <a name="get-backup-job-with-azure-cli"></a>Получение задания резервного копирования с помощью Azure CLI

```azurecli
az backup job show --vault-name {vaultname} --resource-group {resourcegroup} -n {BackupJobID}
```

Эта команда позволяет получить сведения о резервных дисках и исключенных дисках, как показано ниже:

```output
   "Backed-up disk(s)": "diskextest_OsDisk_1_170808a95d214428bad92efeecae626b; diskextest_DataDisk_0; diskextest_DataDisk_1",  "Backup Size": "0 MB",
   "Excluded disk(s)": "diskextest_DataDisk_2",
```

### <a name="list-recovery-points-with-azure-cli"></a>Перечисление точек восстановления с помощью Azure CLI

```azurecli
az backup recoverypoint list --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM
```

Это позволяет получить сведения о количестве подключенных дисков и их резервной копии на виртуальной машине.

```azurecli
      "recoveryPointDiskConfiguration": {
        "excludedDiskList": null,
        "includedDiskList": null,
        "numberOfDisksAttachedToVm": 4,
        "numberOfDisksIncludedInBackup": 3
};
```

### <a name="get-recovery-point-with-azure-cli"></a>Получение точки восстановления с помощью Azure CLI

```azurecli
az backup recoverypoint show --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM -n {recoverypointID}
```

Каждая точка восстановления содержит сведения о включенных и исключенных дисках:

```azurecli
  "recoveryPointDiskConfiguration": {
      "excludedDiskList": [
        {
          "lun": 2,
          "name": "diskextest_DataDisk_2"
        }
      ],
      "includedDiskList": [
        {
          "lun": -1,
          "name": "diskextest_OsDisk_1_170808a95d214428bad92efeecae626b"
        },
        {
          "lun": 0,
          "name": "diskextest_DataDisk_0"
        },
        {
          "lun": 1,
          "name": "diskextest_DataDisk_1"
        }
      ],
      "numberOfDisksAttachedToVm": 4,
      "numberOfDisksIncludedInBackup": 3
```

### <a name="remove-disk-exclusion-settings-and-get-protected-item-with-azure-cli"></a>Удаление параметров исключения диска и получение защищенного элемента с помощью Azure CLI

```azurecli
az backup protection update-for-vm --vault-name {vaultname} --resource-group {resourcegroup} -c {vmname} -i {vmname} --backup-management-type AzureIaasVM --disk-list-setting resetexclusionsettings

az backup item show -c {vmname} -n {vmname} --vault-name {vaultname} --resource-group {resourcegroup} --backup-management-type AzureIaasVM
```

При выполнении этих команд вы увидите `"diskExclusionProperties": null` .

## <a name="using-powershell"></a>Использование PowerShell

Убедитесь, что вы используете Azure PowerShell версии 3.7.0 или более поздней.

Во время операции настройки защиты необходимо указать параметр списка дисков с параметром включения или исключения, указав номера LUN дисков, которые должны быть включены в резервную копию или исключены из нее.

### <a name="enable-backup-with-powershell"></a>Включение резервного копирования с помощью PowerShell

Пример:

```azurepowershell
$disks = ("0","1")
$targetVault = Get-AzRecoveryServicesVault -ResourceGroupName "rg-p-recovery_vaults" -Name "rsv-p-servers"
Get-AzRecoveryServicesBackupProtectionPolicy
$pol = Get-AzRecoveryServicesBackupProtectionPolicy -Name "P-Servers"
```

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -InclusionDisksList $disks -VaultId $targetVault.ID
```

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -ExclusionDisksList $disks -VaultId $targetVault.ID
```

### <a name="backup-only-os-disk-during-configure-backup-with-powershell"></a>Создание резервной копии только диска ОС во время настройки резервного копирования с помощью PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"  -ExcludeAllDataDisks -VaultId $targetVault.ID
```

### <a name="get-backup-item-object-to-be-passed-in-modify-protection-with-powershell"></a>Получение объекта резервной копии для передачи в качестве защиты на изменение с помощью PowerShell

```azurepowershell
$item= Get-AzRecoveryServicesBackupItem -BackupManagementType "AzureVM" -WorkloadType "AzureVM" -VaultId $targetVault.ID -FriendlyName "V2VM"
```

Необходимо передать полученный выше объект **$Item** в параметр **– Item** в следующих командлетах.

### <a name="modify-protection-for-already-backed-up-vms-with-powershell"></a>Изменение защиты для уже резервных копий виртуальных машин с помощью PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -InclusionDisksList[Strings] -VaultId $targetVault.ID
```

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -ExclusionDisksList[Strings] -VaultId $targetVault.ID
```

### <a name="backup-only-os-disk-during-modify-protection-with-powershell"></a>Создавать резервную копию только диска ОС во время защиты от изменения с помощью PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item  -ExcludeAllDataDisks -VaultId $targetVault.ID
```

### <a name="reset-disk-exclusion-setting-with-powershell"></a>Сброс параметров исключения диска с помощью PowerShell

```azurepowershell
Enable-AzRecoveryServicesBackupProtection -Item $item -ResetExclusionSettings -VaultId $targetVault.ID
```

### <a name="restore-selective-disks-with-powershell"></a>Восстановление выборочных дисков с помощью PowerShell

```azurepowershell
$startDate = (Get-Date).AddDays(-7)
$endDate = Get-Date
$rp = Get-AzRecoveryServicesBackupRecoveryPoint -Item $item -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime() -VaultId $targetVault.ID
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID -RestoreDiskList [$disks]
```

### <a name="restore-only-os-disk-with-powershell"></a>Восстановление только диска ОС с помощью PowerShell

```azurepowershell
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName "DestAccount" -StorageAccountResourceGroupName "DestRG" -TargetResourceGroupName "DestRGforManagedDisks" -VaultId $targetVault.ID -RestoreOnlyOSDisk
```

## <a name="using-the-azure-portal"></a>Использование портала Azure

[!INCLUDE [backup-center.md](../../includes/backup-center.md)]

С помощью портал Azure можно просмотреть включенные и Исключенные диски в области сведения о резервном копировании виртуальной машины и области сведения о задании резервного копирования.  Во время восстановления при выборе точки восстановления для восстановления можно просмотреть резервные диски в этой точке восстановления.

Здесь можно просмотреть включенные и Исключенные диски для виртуальной машины на портале в области сведения о резервном копировании виртуальной машины:

![Просмотр включенных и исключенных дисков из области сведений о резервном копировании](./media/selective-disk-backup-restore/backup-details.png)

Здесь можно просмотреть включенные и Исключенные диски в резервной копии из области сведений о задании.

![Просмотр включенных и исключенных дисков из области сведений о задании](./media/selective-disk-backup-restore/job-details.png)

Здесь можно просмотреть резервные диски во время восстановления, выбрав точку восстановления для восстановления из:

![Просмотр резервных копий дисков во время восстановления](./media/selective-disk-backup-restore/during-restore.png)

Настройка возможности резервного копирования выборочных дисков для виртуальной машины с помощью портал Azure ограничена параметром " **архивировать только диск ОС** ". Чтобы использовать резервную копию выборочных дисков в уже созданной резервной копии виртуальной машины или для расширенного включения или исключения конкретных дисков данных виртуальной машины, используйте PowerShell или Azure CLI.

>[!NOTE]
>Если данные распределяются между дисками, убедитесь, что все зависимые диски включены в резервную копию. Если вы не создаете резервную копию всех зависимых дисков в томе, то во время восстановления тома, содержащие некоторые Нерезервные диски, не будут созданы.

### <a name="backup-os-disk-only-in-the-azure-portal"></a>Резервное копирование диска ОС в портал Azure

При включении резервного копирования с помощью портал Azure можно выбрать параметр **архивировать только диск ОС** . Поэтому вы можете настроить резервное копирование виртуальной машины Azure с помощью диска операционной системы и исключить все подключенные к нему диски данных.

![Настройка резервного копирования только для диска ОС](./media/selective-disk-backup-restore/configure-backup-operating-system-disk.png)

## <a name="using-azure-rest-api"></a>Использование Azure REST API

Вы можете настроить резервное копирование виртуальной машины Azure с помощью нескольких выбранных дисков или изменить защиту существующей виртуальной машины, чтобы включить или исключить несколько дисков, как описано [здесь](backup-azure-arm-userestapi-backupazurevms.md#excluding-disks-in-azure-vm-backup).

## <a name="selective-disk-restore"></a>Выборочное восстановление диска

Выборочное восстановление диска — это дополнительная функция, получаемая при включении функции резервного копирования выборочных дисков. С помощью этой функции можно восстановить выборочные диски со всех дисков, которые были архивированы в точке восстановления. Это более эффективно и помогает экономить время в сценариях, где известно, какие диски необходимо восстановить.

- Диск операционной системы по умолчанию включен в резервную копию и восстановление виртуальной машины и не может быть исключен.
- Выборочное восстановление диска поддерживается только для точек восстановления, созданных после включения функции исключения диска.
- Резервные копии с параметром исключения диска **для** поддерживают только параметр **восстановления диска** . **Восстановление виртуальной машины** или **Замена существующих** параметров восстановления в этом случае не поддерживается.

![Параметр восстановления виртуальной машины и замены существующих недоступен во время операции восстановления](./media/selective-disk-backup-restore/options-not-available.png)

## <a name="limitations"></a>Ограничения

Функции резервного копирования выборочных дисков не поддерживаются для классических виртуальных машин и зашифрованных виртуальных машин. Так что виртуальные машины Azure, зашифрованные с помощью шифрования дисков Azure (ADE), используют BitLocker для шифрования виртуальных машин Windows, а функция DM-Encryption для виртуальных машин Linux не поддерживается.

Параметры восстановления для **создания новой виртуальной машины** и **замены существующих** не поддерживаются для виртуальной машины, для которой включена функция резервного копирования выборочных дисков.

Сейчас служба архивации виртуальных машин Azure не поддерживает виртуальные машины с подключенными к ним виртуальными дисками Ultra-Disks или общие диски. В таких случаях нельзя использовать выборочную резервную копию диска, что позволяет исключить диск и создать резервную копию виртуальной машины.

## <a name="billing"></a>Выставление счетов

Резервное копирование виртуальных машин Azure соответствует существующей модели ценообразования, подробно описанной [здесь](https://azure.microsoft.com/pricing/details/backup/).

**Стоимость защищенного экземпляра (PI)** вычисляется для диска ОС только в том случае, если выбран вариант резервного копирования с использованием параметра **только диск операционной системы** .  Если вы настроили резервное копирование и выбрали по крайней мере один диск данных, стоимость PI будет рассчитана для всех дисков, подключенных к виртуальной машине. **Стоимость хранилища резервных копий** вычисляется на основе только включенных дисков, поэтому вы получаете возможность сэкономить стоимость хранилища. **Стоимость моментального снимка** всегда вычисляется для всех дисков виртуальной машины (включенных и исключенных дисков).

Если вы выбрали функцию "восстановление между регионами" (КРР), [цены на КРР](https://azure.microsoft.com/pricing/details/backup/) применяются к стоимости хранилища резервных копий после исключения диска.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="how-is-protected-instance-pi-cost-calculated-for-only-os-disk-backup-in-windows-and-linux"></a>Как вычисляется стоимость защищенного экземпляра (PI) только для резервного копирования диска ОС в Windows и Linux?

Стоимость PI вычисляется на основе фактического (используемого) размера виртуальной машины.

- Для Windows: вычисление используемого пространства основано на диске, где хранится операционная система (обычно C:).
- Для Linux: используемое вычисление пространства основано на устройстве, в котором подключена корневая файловая система (/).

### <a name="i-have-configured-only-os-disk-backup-why-is-the-snapshot-happening-for-all-the-disks"></a>Я настроил только резервную копию диска ОС, почему моментальный снимок происходит для всех дисков?

Функции выборочной архивации диска позволяют экономить затраты на хранение резервных копий за счет усиления количества включенных дисков, которые являются частью резервной копии. Однако моментальный снимок создается для всех дисков, подключенных к виртуальной машине. Поэтому стоимость моментального снимка всегда вычисляется для всех дисков виртуальной машины (включенных и исключенных дисков). Дополнительные сведения см. в разделе [выставление счетов](#billing).

### <a name="i-cant-configure-backup-for-the-azure-virtual-machine-by-excluding-ultra-disk-or-shared-disks-attached-to-the-vm"></a>Не удается настроить резервное копирование для виртуальной машины Azure за исключением Ultra Disk или общих дисков, подключенных к виртуальной машине

Функция выборочной архивации диска — это возможность, предоставляемая в верхней части решения для резервного копирования виртуальных машин Azure. Сейчас служба архивации виртуальных машин Azure не поддерживает виртуальные машины с подключенными к ним виртуальными дисками Ultra-Disk или Shared.

## <a name="next-steps"></a>Дальнейшие действия

- [Матрица поддержки для резервного копирования виртуальных машин Azure](backup-support-matrix-iaas.md)
- [Часто задаваемые вопросы. Резервное копирование виртуальных машин Azure](backup-azure-vm-backup-faq.yml)
