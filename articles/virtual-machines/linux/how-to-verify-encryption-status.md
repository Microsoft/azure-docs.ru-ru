---
title: Проверка состояния шифрования для Linux — шифрование дисков Azure
description: В этой статье приведены инструкции по проверке состояния шифрования на уровнях платформы и операционной системы.
author: kailashmsft
ms.service: virtual-machines
ms.subservice: disks
ms.topic: how-to
ms.author: kaib
ms.date: 03/11/2020
ms.custom: seodec18, devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 04654672cc5806465ec9f75b695772dcb2037eab
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102564141"
---
# <a name="verify-encryption-status-for-linux"></a>Проверка состояния шифрования для Linux 

Эта статья охватывает проверку состояния шифрования виртуальной машины с помощью различных методов: портала Azure, PowerShell, Azure CLI или операционной системы виртуальной машины. 

Состояние шифрования можно проверить во время или после его выполнения, выполнив одно из следующих действий.

- Проверка дисков, подключенных к определенной виртуальной машине. 
- Запрос параметров шифрования на каждом диске, подключен ли диск или нет.

Этот сценарий относится расширениям шифрования дисков Azure с двойным или одиночным проходом. Дистрибутивы Linux являются единственной средой для этого сценария.

>[!NOTE] 
>Мы используем переменные в этой статье. Заменяйте значения соответствующим образом.

## <a name="portal"></a>Портал

В портал Azure в разделе **расширения** выберите расширение шифрования дисков Azure в списке. Информация для **сообщения о состоянии** указывает текущее состояние шифрования:

![Проверка портала с выделением состояния, версии и сообщения о состоянии](./media/disk-encryption/verify-encryption-linux/portal-check-001.png)

В списке расширений вы увидите соответствующую версию расширения шифрования дисков Azure. Версия 0.x соответствует двум проходам шифрования дисков Azure, а версия 1. x — одному проходу шифрования дисков Azure.

Чтобы получить дополнительные сведения, выберите расширение и затем выберите **Просмотр подробных сведений о состоянии**. Подробное состояние процесса шифрования отображается в формате JSON.

![Проверка портала с выделенной ссылкой "Просмотр подробных сведений о состоянии"](./media/disk-encryption/verify-encryption-linux/portal-check-002.png)

![Подробное состояние в формате JSON](./media/disk-encryption/verify-encryption-linux/portal-check-003.png)

Еще один способ проверить состояние шифрования — просмотреть раздел **Параметры диска**.

![Состояние шифрования для дисков ОС и дисков данных](./media/disk-encryption/verify-encryption-linux/portal-check-004.png)

>[!NOTE] 
> Это состояние означает, что на дисках помечены параметры шифрования, а не то, что все они были зашифрованы на уровне операционной системы.
>
> Диски сперва помечаются, а затем шифруются, это сделано намеренно. В случае сбоя процесса шифрования диски могут оказаться помеченными, но не зашифрованными. 
>
> Чтобы убедиться, что диски действительно зашифрованы, можно еще раз проверить шифрование каждого диска на уровне операционной системы.

## <a name="powershell"></a>PowerShell

Вы можете проверить *общее* состояние шифрования зашифрованной виртуальной машины с помощью следующей команды PowerShell:

```azurepowershell
   $VMNAME="VMNAME"
   $RGNAME="RGNAME"
   Get-AzVmDiskEncryptionStatus -ResourceGroupName  ${RGNAME} -VMName ${VMNAME}
```
![Общее состояние шифрования в PowerShell](./media/disk-encryption/verify-encryption-linux/verify-status-ps-01.png)

Вы можете записать параметры шифрования с каждого диска, используя следующие команды PowerShell.

### <a name="single-pass"></a>Одиночный проход
При одном проходе параметры шифрования помечаются на каждом из дисков (ОС и данных). Параметры шифрования для диска ОС можно записать одним проходом следующим образом:

```powershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$VM = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME}  
 $Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $VM.StorageProfile.OsDisk.Name
 Write-Host "============================================================================================================================================================="
 Write-Host "Encryption Settings:"
 Write-Host "============================================================================================================================================================="
 Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
 Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
 Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
 Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
 Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
 Write-Host "============================================================================================================================================================="
```
![Параметры шифрования для диска ОС](./media/disk-encryption/verify-encryption-linux/verify-os-single-ps-001.png)

Если на диске не помечены параметры шифрования, выходные данные будут пустыми:

![Пустые выходные данные](./media/disk-encryption/verify-encryption-linux/os-encryption-settings-2.png)

Используйте следующие команды для записи параметров шифрования дисков данных.

```azurepowershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$VM = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME}
 clear
 foreach ($i in $VM.StorageProfile.DataDisks|ForEach-Object{$_.Name})
 {
 Write-Host "============================================================================================================================================================="
 Write-Host "Encryption Settings:"
 Write-Host "============================================================================================================================================================="
 Write-Host "Checking Disk:" $i
 $Disk=(Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $i)
 Write-Host "Encryption Enable: " $Sourcedisk.EncryptionSettingsCollection.Enabled
 Write-Host "Encryption KeyEncryptionKey: " $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl;
 Write-Host "Encryption DiskEncryptionKey: " $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl;
 Write-Host "============================================================================================================================================================="
 }
```
![Параметры шифрования для дисков данных](./media/disk-encryption/verify-encryption-linux/verify-data-single-ps-001.png)

### <a name="dual-pass"></a>Двойной проход
При двойном проходе параметры шифрования помечаются в модели виртуальной машины, а не на каждом отдельном диске.

Чтобы убедиться, что параметры шифрования были помечены в двойном проходе, используйте следующие команды:

```azurepowershell
$RGNAME = "RGNAME"
$VMNAME = "VMNAME"

$vm = Get-AzVm -ResourceGroupName ${RGNAME} -Name ${VMNAME};
$Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName $VM.StorageProfile.OsDisk.Name
clear
Write-Host "============================================================================================================================================================="
Write-Host "Encryption Settings:"
Write-Host "============================================================================================================================================================="
Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
Write-Host "============================================================================================================================================================="
```
![Параметры шифрования при двойном проходе](./media/disk-encryption/verify-encryption-linux/verify-dual-ps-001.png)

### <a name="unattached-disks"></a>Неподключенные диски

Проверьте параметры шифрования для дисков, которые не подключены к виртуальной машине.

### <a name="managed-disks"></a>Управляемые диски
```powershell
$Sourcedisk = Get-AzDisk -ResourceGroupName ${RGNAME} -DiskName ${TARGETDISKNAME}
Write-Host "============================================================================================================================================================="
Write-Host "Encryption Settings:"
Write-Host "============================================================================================================================================================="
Write-Host "Enabled:" $Sourcedisk.EncryptionSettingsCollection.Enabled
Write-Host "Version:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettingsVersion
Write-Host "Source Vault:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SourceVault.Id
Write-Host "Secret URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.DiskEncryptionKey.SecretUrl
Write-Host "Key URL:" $Sourcedisk.EncryptionSettingsCollection.EncryptionSettings.KeyEncryptionKey.KeyUrl
Write-Host "============================================================================================================================================================="
```
## <a name="azure-cli"></a>Azure CLI

Вы можете проверить *общее* состояние шифрования зашифрованной виртуальной машины с помощью следующих команд Azure CLI:

```azurecli
VMNAME="VMNAME"
RGNAME="RGNAME"
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} --query "substatus"
```
![Общее состояние шифрования из Azure CLI ](./media/disk-encryption/verify-encryption-linux/verify-gen-cli.png)

### <a name="single-pass"></a>Одиночный проход
Вы можете проверить параметры шифрования для каждого диска, используя следующие команды Azure CLI.

```azurecli
az vm encryption show -g ${RGNAME} -n ${VMNAME} --query "disks[*].[name, statuses[*].displayStatus]"  -o table
```

![Параметры шифрования данных](./media/disk-encryption/verify-encryption-linux/data-encryption-settings-2.png)

>[!IMPORTANT]
> Если на диске не помечены параметры шифрования, вы увидите текст **Диск не зашифрован**.

Чтобы получить подробные сведения о состоянии и параметрах шифрования, используйте следующие команды.

Диск ОС:

```bash
RGNAME="RGNAME"
VMNAME="VNAME"

disk=`az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.osDisk.name -o tsv`
for disk in $disk; do \
echo "============================================================================================================================================================="
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![Подробные сведения о состоянии и параметрах шифрования для диска ОС](./media/disk-encryption/verify-encryption-linux/os-single-cli.png)

Диски данных

```azurecli
RGNAME="RGNAME"
VMNAME="VMNAME"
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} --query "substatus"

for disk in `az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.dataDisks[].name -o tsv`; do \
echo "============================================================================================================================================================="; \
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![Подробные сведения о состоянии и параметрах шифрования для дисков данных.](./media/disk-encryption/verify-encryption-linux/data-single-cli.png)

### <a name="dual-pass"></a>Двойной проход

```azurecli
az vm encryption show --name ${VMNAME} --resource-group ${RGNAME} -o table
```

![Общие параметры шифрования для двойного прохода посредством Azure CLI](./media/disk-encryption/verify-encryption-linux/verify-gen-dual-cli.png)

Вы также можете проверить параметры шифрования в профиле хранилища модели виртуальной машины на диске ОС:

```bash
disk=`az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.osDisk.name -o tsv`
for disk in $disk; do \
echo "============================================================================================================================================================="; \
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${disk} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${disk} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
done
```

![Профиль виртуальной машины для двойного прохода посредством Azure CLI](./media/disk-encryption/verify-encryption-linux/verify-vm-profile-dual-cli.png)

### <a name="unattached-disks"></a>Неподключенные диски

Проверьте параметры шифрования для дисков, которые не подключены к виртуальной машине.

### <a name="managed-disks"></a>Управляемые диски

```bash
RGNAME="RGNAME"
TARGETDISKNAME="DISKNAME"
echo "============================================================================================================================================================="
echo -ne "Disk Name: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query name -o tsv; \
echo -ne "Encryption Enabled: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.enabled -o tsv; \
echo -ne "Version: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettingsVersion -o tsv; \
echo -ne "Disk Encryption Key: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettings[].diskEncryptionKey.secretUrl -o tsv; \
echo -ne "key Encryption Key: "; az disk show -g ${RGNAME} -n ${TARGETDISKNAME} --query encryptionSettingsCollection.encryptionSettings[].keyEncryptionKey.keyUrl -o tsv; \
echo "============================================================================================================================================================="
```
### <a name="unmanaged-disks"></a>Неуправляемые диски

Неуправляемые диски — это VHD-файлы, которые хранятся в учетных записях хранения Azure как страничные BLOB-объекты.

Чтобы получить сведения о конкретном диске, необходимо указать:

- имя учетной записи хранения, содержащей диск;
- строку подключения для этой конкретной учетной записи хранения;
- имя контейнера, в котором хранится диск;
- имя диска.

Эта команда выводит список всех идентификаторов для всех учетных записей хранения:

```azurecli
az storage account list --query [].[id] -o tsv
```
Идентификаторы учетной записи хранения перечислены в следующем формате:

/subscriptions/\<subscription id>/resourceGroups/\<resource group name>/providers/Microsoft.Storage/storageAccounts/\<storage account name>

Выберите соответствующий идентификатор и сохраните его в переменной:
```bash
id="/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name>"
```

Эта команда возвращает строку подключения для одной конкретной учетной записи хранения и сохраняет ее в переменной:

```bash
ConnectionString=$(az storage account show-connection-string --ids $id --query connectionString -o tsv)
```

Следующая команда выводит список всех контейнеров в учетной записи хранения:
```azurecli
az storage container list --connection-string $ConnectionString --query [].[name] -o tsv
```
Контейнер, используемый для дисков, обычно называется vhds.

Сохраните имя контейнера в переменной: 
```bash
ContainerName="name of the container"
```

Используйте эту команду, чтобы вывести список всех больших двоичных объектов в определенном контейнере:
```azurecli 
az storage blob list -c ${ContainerName} --connection-string $ConnectionString --query [].[name] -o tsv
```
Выберите диск, который требуется запросить, и сохраните его имя в переменной:
```bash
DiskName="diskname.vhd"
```
Запросите параметры шифрования диска:
```azurecli
az storage blob show -c ${ContainerName} --connection-string ${ConnectionString} -n ${DiskName} --query metadata.DiskEncryptionSettings
```

## <a name="operating-system"></a>Операционная система
Проверьте, зашифрованы ли разделы диска данных (и не зашифрованы ли они на диске ОС).

При шифровании раздела или диска он отображается как тип **crypt**. Если он не зашифрован, он отображается в виде типа **part/disk**.

```bash
lsblk
```

![Уровень шифрования ОС для секции](./media/disk-encryption/verify-encryption-linux/verify-os-crypt-layer.png)

Для получения дополнительных сведений можно использовать следующий вариант **lsblk**. 

Вы увидите уровень типа **crypt**, подключенный с помощью расширения. В следующем примере показаны логические тома и обычные диски с **crypto\_LUKS FSTYPE**.

```bash
lsblk -o NAME,TYPE,FSTYPE,LABEL,SIZE,RO,MOUNTPOINT
```
![Уровень шифрования ОС для логических томов и обычных дисков](./media/disk-encryption/verify-encryption-linux/verify-os-crypt-layer-2.png)

В качестве дополнительного шага можно проверить, загружены ли на диск данных ключи.

```bash
cryptsetup luksDump /dev/VGNAME/LVNAME
```

```bash
cryptsetup luksDump /dev/sdd1
```

Вы также можете проверить, какие устройства **dm** перечислены как **crypt**:

```bash
dmsetup ls --target crypt
```

## <a name="next-steps"></a>Дальнейшие действия

- [Устранение неполадок с шифрованием дисков Azure](disk-encryption-troubleshooting.md)
