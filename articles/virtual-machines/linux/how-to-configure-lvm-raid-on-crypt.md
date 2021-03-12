---
title: Настройка LVM и RAID на зашифрованных устройствах — шифрование дисков Azure
description: В этой статье приводятся инструкции по настройке LVM и RAID на зашифрованных устройствах для виртуальных машин Linux.
author: jofrance
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.topic: how-to
ms.author: jofrance
ms.date: 03/17/2020
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: ec9f99d0a13b5b92bc267f184d364ebabe36a050
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102566113"
---
# <a name="configure-lvm-and-raid-on-encrypted-devices"></a>Настройка LVM и RAID на зашифрованных устройствах

Эта статья представляет собой пошаговый процесс выполнения логического управления томами (LVM) и RAID на зашифрованных устройствах. Этот процесс применяется к следующим средам:

- Дистрибутивах Linux
    - RHEL 7.6 +
    - Ubuntu 18.04 +
    - SUSE 12 +
- Расширение для одного прохода шифрования дисков Azure
- Расширение двойного прохода шифрования дисков Azure


## <a name="scenarios"></a>Сценарии

Процедуры, описанные в этой статье, поддерживают следующие сценарии.  

- Настройка LVM поверх зашифрованных устройств (LVM-on-Encrypting)
- Настройка RAID поверх зашифрованных устройств (RAID-On-Encrypting)

После шифрования устройства или устройства можно создать структуры LVM или RAID поверх этого зашифрованного слоя. 

Физические тома (постоянного хранилища версий) создаются поверх зашифрованного слоя. Для создания группы томов используются физические тома. Создайте тома и добавьте необходимые записи в/etc/fstab.. 

![Схема слоев структур LVM](./media/disk-encryption/lvm-raid-on-crypt/000-lvm-raid-crypt-diagram.png)

Аналогичным образом, устройство RAID создается поверх зашифрованного слоя на дисках. Файловая система создается поверх устройства RAID и добавляется в/etc/fstab как обычное устройство.

## <a name="considerations"></a>Рекомендации

Рекомендуется использовать LVM-on-шифрования. RAID можно использовать, если LVM не может использоваться из-за ограничений конкретного приложения или среды.

Вы будете использовать параметр **енкриптформаталл** . Дополнительные сведения об этом параметре см. [в статье Использование функции енкриптформаталл для дисков данных на виртуальных машинах Linux](./disk-encryption-linux.md#use-encryptformatall-feature-for-data-disks-on-linux-vms).

Несмотря на то, что этот метод можно использовать при шифровании операционной системы, мы просто зашифрованы диски с данными.

В процедурах предполагается, что вы уже проверили предварительные требования в [сценариях шифрования дисков Azure на виртуальных машинах Linux](./disk-encryption-linux.md) и в [кратком руководстве создание и шифрование виртуальной машины Linux с Azure CLI](./disk-encryption-cli-quickstart.md).

Версия с двумя проходами шифрования дисков Azure находится на пути устаревания и больше не должна использоваться в новых шифрованиях.

## <a name="general-steps"></a>основные шаги

При использовании конфигураций "при шифровании" используйте процесс, описанный в следующих процедурах.

>[!NOTE] 
>Мы используем переменные в этой статье. Заменяйте значения соответствующим образом.

### <a name="deploy-a-vm"></a>Развертывание виртуальной машины 
Следующие команды являются необязательными, но их рекомендуется применять к только что развернутой виртуальной машине.

PowerShell.

```powershell
New-AzVm -ResourceGroupName ${RGNAME} `
-Name ${VMNAME} `
-Location ${LOCATION} `
-Size ${VMSIZE} `
-Image ${OSIMAGE} `
-Credential ${creds} `
-Verbose
```
Azure CLI:

```azurecli
az vm create \
-n ${VMNAME} \
-g ${RGNAME} \
--image ${OSIMAGE} \
--admin-username ${username} \
--admin-password ${password} \
-l ${LOCATION} \
--size ${VMSIZE} \
-o table
```
### <a name="attach-disks-to-the-vm"></a>Присоединение дисков к виртуальной машине
Повторите следующие команды для `$N` количества новых дисков, которые нужно подключить к виртуальной машине.

PowerShell.

```powershell
$storageType = 'Standard_LRS'
$dataDiskName = ${VMNAME} + '_datadisk0'
$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $LOCATION -CreateOption Empty -DiskSizeGB 5
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName ${RGNAME}
$vm = Get-AzVM -Name ${VMNAME} -ResourceGroupName ${RGNAME} 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 0
Update-AzVM -VM ${VM} -ResourceGroupName ${RGNAME}
```

Azure CLI:

```azurecli
az vm disk attach \
-g ${RGNAME} \
--vm-name ${VMNAME} \
--name ${VMNAME}datadisk1 \
--size-gb 5 \
--new \
-o table
```

### <a name="verify-that-the-disks-are-attached-to-the-vm"></a>Убедитесь, что диски подключены к виртуальной машине.
PowerShell.
```powershell
$VM = Get-AzVM -ResourceGroupName ${RGNAME} -Name ${VMNAME}
$VM.StorageProfile.DataDisks | Select-Object Lun,Name,DiskSizeGB
```
![Список подключенных дисков в PowerShell](./media/disk-encryption/lvm-raid-on-crypt/001-lvm-raid-check-disks-powershell.png)

Azure CLI:

```azurecli
az vm show -g ${RGNAME} -n ${VMNAME} --query storageProfile.dataDisks -o table
```
![Список подключенных дисков в Azure CLI](./media/disk-encryption/lvm-raid-on-crypt/002-lvm-raid-check-disks-cli.png)

Портал:

![Список подключенных дисков на портале](./media/disk-encryption/lvm-raid-on-crypt/003-lvm-raid-check-disks-portal.png)

ОС:

```bash
lsblk 
```
![Список подключенных дисков в ОС](./media/disk-encryption/lvm-raid-on-crypt/004-lvm-raid-check-disks-os.png)

### <a name="configure-the-disks-to-be-encrypted"></a>Настройка шифрования дисков
Эта конфигурация выполняется на уровне операционной системы. Для соответствующих дисков настраивается традиционное шифрование с помощью шифрования дисков Azure:

- Файловые системы создаются поверх дисков.
- Для подключения файловых систем создаются временные точки подключения.
- Файловые системы настраиваются в/etc/fstab для подключения во время загрузки.

Проверьте букву устройства, назначенную новым дискам. В этом примере мы используем четыре диска данных.

```bash
lsblk 
```
![Диски данных, подключенные к операционной системе](./media/disk-encryption/lvm-raid-on-crypt/004-lvm-raid-check-disks-os.png)

### <a name="create-a-file-system-on-top-of-each-disk"></a>Создание файловой системы поверх каждого диска
Эта команда выполняет итерацию по созданию файловой системы ext4 на каждом диске, определенном в части цикла "for".

```bash
for disk in c d e f; do echo mkfs.ext4 -F /dev/sd${disk}; done |bash
```
![Создание файловой системы ext4](./media/disk-encryption/lvm-raid-on-crypt/005-lvm-raid-create-temp-fs.png)

Найдите универсальный уникальный идентификатор (UUID) для недавно созданных файловых систем, создайте временную папку, добавьте соответствующие записи в/etc/fstab и подключите все файловые системы.

Эта команда также выполняет итерацию по каждому диску, определенному в части цикла "for":

```bash
for disk in c d e f; do diskuuid="$(blkid -s UUID -o value /dev/sd${disk})"; \
mkdir /tempdata${disk}; \
echo "UUID=${diskuuid} /tempdata${disk} ext4 defaults,nofail 0 0" >> /etc/fstab; \
mount -a; \
done
``` 

### <a name="verify-that-the-disks-are-mounted-properly"></a>Убедитесь, что диски подключены правильно
```bash
lsblk
```
![Список подключенных временных файловых систем](./media/disk-encryption/lvm-raid-on-crypt/006-lvm-raid-verify-temp-fs.png)

Также убедитесь, что диски настроены:

```bash
cat /etc/fstab
```
![Сведения о конфигурации через fstab](./media/disk-encryption/lvm-raid-on-crypt/007-lvm-raid-verify-temp-fstab.png)

### <a name="encrypt-the-data-disks"></a>Шифрование дисков данных
PowerShell с использованием ключа шифрования ключей (KEK):

```powershell
$sequenceVersion = [Guid]::NewGuid() 
Set-AzVMDiskEncryptionExtension -ResourceGroupName $RGNAME `
-VMName ${VMNAME} `
-DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl `
-DiskEncryptionKeyVaultId $KeyVaultResourceId `
-KeyEncryptionKeyUrl $keyEncryptionKeyUrl `
-KeyEncryptionKeyVaultId $KeyVaultResourceId `
-VolumeType 'DATA' `
-EncryptFormatAll `
-SequenceVersion $sequenceVersion `
-skipVmBackup;
```

Azure CLI с помощью KEK:

```azurecli
az vm encryption enable \
--resource-group ${RGNAME} \
--name ${VMNAME} \
--disk-encryption-keyvault ${KEYVAULTNAME} \
--key-encryption-key ${KEYNAME} \
--key-encryption-keyvault ${KEYVAULTNAME} \
--volume-type "DATA" \
--encrypt-format-all \
-o table
```
### <a name="verify-the-encryption-status"></a>Проверка состояния шифрования

Переходите к следующему шагу, только если все диски зашифрованы.

PowerShell.

```powershell
Get-AzVmDiskEncryptionStatus -ResourceGroupName ${RGNAME} -VMName ${VMNAME}
```
![Состояние шифрования в PowerShell](./media/disk-encryption/lvm-raid-on-crypt/008-lvm-raid-verify-encryption-status-ps.png)

Azure CLI:

```azurecli
az vm encryption show -n ${VMNAME} -g ${RGNAME} -o table
```
![Состояние шифрования в Azure CLI](./media/disk-encryption/lvm-raid-on-crypt/009-lvm-raid-verify-encryption-status-cli.png)

Портал:

![Состояние шифрования на портале](./media/disk-encryption/lvm-raid-on-crypt/010-lvm-raid-verify-encryption-status-portal.png)

Уровень ОС:

```bash
lsblk
```
![Состояние шифрования в ОС](./media/disk-encryption/lvm-raid-on-crypt/011-lvm-raid-verify-encryption-status-os.png)

Расширение добавит файловые системы в/Вар/либ/azure_disk_encryption_config/azure_crypt_mount (старое шифрование) или в/etc/crypttab (новые шифрования).

>[!NOTE] 
>Не изменяйте ни один из этих файлов.

Этот файл позаботится о активации этих дисков во время процесса загрузки, чтобы LVM или RAID мог их использовать позже. 

Не беспокойтесь о точках подключения этого файла. Шифрование дисков Azure потеряет возможность получения дисков, подключенных в качестве обычной файловой системы, после создания физического тома или устройства RAID на основе этих зашифрованных устройств. (Это приведет к удалению формата файловой системы, который мы использовали во время процесса подготовки.)

### <a name="remove-the-temporary-folders-and-temporary-fstab-entries"></a>Удаление временных папок и временных записей fstab
Отсоедините файловые системы на дисках, которые будут использоваться в составе LVM.

```bash
for disk in c d e f; do unmount /tempdata${disk}; done
```
И удалите записи/etc/fstab:

```bash
vi /etc/fstab
```
### <a name="verify-that-the-disks-are-not-mounted-and-that-the-entries-on-etcfstab-were-removed"></a>Убедитесь, что диски не подключены и записи в/etc/fstab были удалены.

```bash
lsblk
```
![Проверка отключения временных файловых систем](./media/disk-encryption/lvm-raid-on-crypt/012-lvm-raid-verify-disks-not-mounted.png)

И убедитесь, что диски настроены:
```bash
cat /etc/fstab
```
![Проверка удаления временных записей fstab](./media/disk-encryption/lvm-raid-on-crypt/013-lvm-raid-verify-fstab-temp-removed.png)

## <a name="steps-for-lvm-on-crypt"></a>Шаги для LVM-on-шифрования
Теперь, когда базовые диски зашифрованы, можно создать структуры LVM.

Вместо имени устройства используйте пути/Дев/маппер для каждого диска, чтобы создать физический том (на уровне шифрования поверх диска, а не на самом диске).

### <a name="configure-lvm-on-top-of-the-encrypted-layers"></a>Настройка LVM поверх зашифрованных слоев
#### <a name="create-the-physical-volumes"></a>Создание физических томов
Вы получите предупреждение о том, что можете очистить подпись файловой системы. Продолжайте, введя **y**, или используйте **echo "y"** , как показано ниже:

```bash
echo "y" | pvcreate /dev/mapper/c49ff535-1df9-45ad-9dad-f0846509f052
echo "y" | pvcreate /dev/mapper/6712ad6f-65ce-487b-aa52-462f381611a1
echo "y" | pvcreate /dev/mapper/ea607dfd-c396-48d6-bc54-603cf741bc2a
echo "y" | pvcreate /dev/mapper/4159c60a-a546-455b-985f-92865d51158c
```
![Проверка создания физического тома](./media/disk-encryption/lvm-raid-on-crypt/014-lvm-raid-pvcreate.png)

>[!NOTE] 
>Имена/Дев/маппер/девице необходимо заменить на фактические значения на основе выходных данных **лсблк**.

#### <a name="verify-the-information-for-physical-volumes"></a>Проверьте данные для физических томов.
```bash
pvs
```

![Сведения о физических томах](./media/disk-encryption/lvm-raid-on-crypt/015-lvm-raid-pvs.png)

#### <a name="create-the-volume-group"></a>Создание группы томов
Создайте группу томов с помощью тех же устройств, которые уже инициализированы:

```bash
vgcreate vgdata /dev/mapper/
```

### <a name="check-the-information-for-the-volume-group"></a>Проверьте сведения о группе томов

```bash
vgdisplay -v vgdata
```
```bash
pvs
```
![Сведения о группе томов](./media/disk-encryption/lvm-raid-on-crypt/016-lvm-raid-pvs-on-vg.png)

#### <a name="create-logical-volumes"></a>Создание логических томов

```bash
lvcreate -L 10G -n lvdata1 vgdata
lvcreate -L 7G -n lvdata2 vgdata
``` 

#### <a name="check-the-created-logical-volumes"></a>Проверка созданных логических томов

```bash
lvdisplay
lvdisplay vgdata/lvdata1
lvdisplay vgdata/lvdata2
```
![Сведения о логических томах](./media/disk-encryption/lvm-raid-on-crypt/017-lvm-raid-lvs.png)

#### <a name="create-file-systems-on-top-of-the-structures-for-logical-volumes"></a>Создание файловых систем поверх структур для логических томов

```bash
echo "yes" | mkfs.ext4 /dev/vgdata/lvdata1
echo "yes" | mkfs.ext4 /dev/vgdata/lvdata2
```

#### <a name="create-the-mount-points-for-the-new-file-systems"></a>Создание точек подключения для новых файловых систем

```bash
mkdir /data0
mkdir /data1
```

#### <a name="add-the-new-file-systems-to-etcfstab-and-mount-them"></a>Добавьте новые файловые системы в/etc/fstab и подключите их.

```bash
echo "/dev/mapper/vgdata-lvdata1 /data0 ext4 defaults,nofail 0 0" >>/etc/fstab
echo "/dev/mapper/vgdata-lvdata2 /data1 ext4 defaults,nofail 0 0" >>/etc/fstab
mount -a
```

#### <a name="verify-that-the-new-file-systems-are-mounted"></a>Убедитесь, что новые файловые системы подключены

```bash
lsblk -fs
df -h
```
![На снимке экрана показано окно консоли с файловыми системами, подключенными как data0 и файл1.](./media/disk-encryption/lvm-raid-on-crypt/018-lvm-raid-lsblk-after-lvm.png)

В этом варианте **лсблк** перечислены устройства, на которых показаны зависимости в обратном порядке. Этот параметр позволяет обнаруживать устройства, сгруппированные по логическому тому, а не исходные имена устройств/Дев/СД [диск].

Важно убедиться в том, что параметр "не **пройден** " добавляется к параметрам точки подключения томов LVM, созданных на основе устройства, зашифрованного с помощью шифрования дисков Azure. Это предотвращает зависание операционной системы во время процесса загрузки (или в режиме обслуживания).

Если вы не используете параметр "не **удалось** ":

- Операционная система никогда не пойдет в этап, где запущено шифрование дисков Azure, а диски с данными будут разблокированы и подключены. 
- Зашифрованные диски будут разблокированы в конце процесса загрузки. Тома и файловые системы LVM будут автоматически подключены, пока шифрование дисков Azure не разблокирует их. 

Вы можете протестировать перезагрузку виртуальной машины и проверить, что файловые системы также автоматически подключаются после времени загрузки. Этот процесс может занять несколько минут в зависимости от числа и размеров файловых систем.

#### <a name="reboot-the-vm-and-verify-after-reboot"></a>Перезагрузите виртуальную машину и проверьте после перезагрузки.

```bash
shutdown -r now
```
```bash
lsblk
df -h
```
## <a name="steps-for-raid-on-crypt"></a>Шаги для RAID-включения и шифрования
Теперь, когда базовые диски зашифрованы, можно продолжить создание структур RAID. Этот процесс аналогичен тому, который используется для LVM, но вместо имени устройства используйте пути/Дев/маппер для каждого диска.

#### <a name="configure-raid-on-top-of-the-encrypted-layer-of-the-disks"></a>Настройка RAID поверх зашифрованного уровня дисков
```bash
mdadm --create /dev/md10 \
--level 0 \
--raid-devices=4 \
/dev/mapper/c49ff535-1df9-45ad-9dad-f0846509f052 \
/dev/mapper/6712ad6f-65ce-487b-aa52-462f381611a1 \
/dev/mapper/ea607dfd-c396-48d6-bc54-603cf741bc2a \
/dev/mapper/4159c60a-a546-455b-985f-92865d51158c
```
![Сведения о настроенном RAID-массиве с помощью команды mdadm](./media/disk-encryption/lvm-raid-on-crypt/019-lvm-raid-md-creation.png)

>[!NOTE] 
>Имена/Дев/маппер/девице необходимо заменить фактическими значениями на основе выходных данных **лсблк**.

### <a name="checkmonitor-raid-creation"></a>Проверка и мониторинг создания RAID-массива
```bash
watch -n1 cat /proc/mdstat
mdadm --examine /dev/mapper/[]
mdadm --detail /dev/md10
```
![Состояние RAID](./media/disk-encryption/lvm-raid-on-crypt/020-lvm-raid-md-details.png)

### <a name="create-a-file-system-on-top-of-the-new-raid-device"></a>Создание файловой системы поверх нового устройства RAID
```bash
mkfs.ext4 /dev/md10
```

Создайте новую точку подключения для файловой системы, добавьте новую файловую систему в/etc/fstab и подключите ее:

```bash
for device in md10; do diskuuid="$(blkid -s UUID -o value /dev/${device})"; \
mkdir /raiddata; \
echo "UUID=${diskuuid} /raiddata ext4 defaults,nofail 0 0" >> /etc/fstab; \
mount -a; \
done
```

Убедитесь, что новая файловая система подключена:

```bash
lsblk -fs
df -h
```
![На снимке экрана показано окно консоли с файловой системой, смонтированной как раиддата.](./media/disk-encryption/lvm-raid-on-crypt/021-lvm-raid-lsblk-md-details.png)

Важно убедиться в том, что параметр не был добавлен **к параметрам** точки подключения томов RAID, созданных на основе устройства, зашифрованного с помощью шифрования дисков Azure. Это предотвращает зависание операционной системы во время процесса загрузки (или в режиме обслуживания).

Если вы не используете параметр "не **удалось** ":

- Операционная система никогда не пойдет в этап, где запущено шифрование дисков Azure, а диски с данными будут разблокированы и подключены.
- Зашифрованные диски будут разблокированы в конце процесса загрузки. Тома и файловые системы RAID будут автоматически подключены, пока шифрование дисков Azure не разблокирует их.

Вы можете протестировать перезагрузку виртуальной машины и проверить, что файловые системы также автоматически подключаются после времени загрузки. Этот процесс может занять несколько минут в зависимости от числа и размеров файловых систем.

```bash
shutdown -r now
```

И когда вы можете войти в систему:

```bash
lsblk
df -h
```
## <a name="next-steps"></a>Дальнейшие действия

- [Изменение размера устройств управления логическими томами, зашифрованных с помощью шифрования дисков Azure](how-to-resize-encrypted-lvm.md)
- [Устранение неполадок с шифрованием дисков Azure](disk-encryption-troubleshooting.md)
