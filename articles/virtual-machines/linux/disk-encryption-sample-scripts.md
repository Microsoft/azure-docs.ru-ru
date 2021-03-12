---
title: Примеры скриптов шифрования дисков Azure
description: В этой статье описывается, как Microsoft Azure шифрование дисков для виртуальных машин Linux.
author: msmbaldwin
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.topic: how-to
ms.author: mbaldwin
ms.date: 08/06/2019
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: f11677d9ebc31f1c1f7cc6332b07b69f8e35ad52
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102561183"
---
# <a name="azure-disk-encryption-sample-scripts-for-linux-vms"></a>Примеры сценариев шифрования дисков Azure для виртуальных машин Linux

В этой статье приводятся примеры сценариев для подготовки предварительно зашифрованных виртуальных жестких дисков и других задач.  

> [!NOTE]
> Все скрипты относятся к последней версии файла ADE, отличной от AAD, за исключением указанных выше.

## <a name="sample-powershell-scripts-for-azure-disk-encryption"></a>Примеры сценариев PowerShell для шифрования дисков Azure 

- **Вывод списка всех зашифрованных виртуальных машин в подписке**
  
  Все виртуальные машины, зашифрованные с помощью ADE, и версия расширения во всех группах ресурсов, имеющихся в подписке, можно найти с помощью [этого сценария PowerShell](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/Find_1passAdeVersion_VM.ps1).

  Кроме того, эти командлеты будут показывать все виртуальные машины, зашифрованные с помощью ADE (но не версия расширения):

   ```azurepowershell-interactive
   $osVolEncrypted = {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).OsVolumeEncrypted}
   $dataVolEncrypted= {(Get-AzVMDiskEncryptionStatus -ResourceGroupName $_.ResourceGroupName -VMName $_.Name).DataVolumesEncrypted}
   Get-AzVm | Format-Table @{Label="MachineName"; Expression={$_.Name}}, @{Label="OsVolumeEncrypted"; Expression=$osVolEncrypted}, @{Label="DataVolumesEncrypted"; Expression=$dataVolEncrypted}
   ```

- **Вывод списка всех зашифрованных экземпляров VMSS в вашей подписке**
    
    Все экземпляры VMSS, зашифрованные с помощью ADE, и версия расширения во всех группах ресурсов, имеющихся в подписке, можно найти с помощью [этого сценария PowerShell](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/Find_1passAdeVersion_VMSS.ps1).

- **Вывод списка всех секретов шифрования дисков, используемых для шифрования виртуальных машин в хранилище ключей** 

   ```azurepowershell-interactive
   Get-AzKeyVaultSecret -VaultName $KeyVaultName | where {$_.Tags.ContainsKey('DiskEncryptionKeyFileName')} | format-table @{Label="MachineName"; Expression={$_.Tags['MachineName']}}, @{Label="VolumeLetter"; Expression={$_.Tags['VolumeLetter']}}, @{Label="EncryptionKeyURL"; Expression={$_.Id}}
   ```

### <a name="using-the-azure-disk-encryption-prerequisites-powershell-script"></a> Выполнение сценария PowerShell для установки компонентов, необходимых при шифровании дисков Azure
Если вы уже знакомы с предварительными требованиями для шифрования дисков Azure, можно использовать [соответствующий сценарий PowerShell предварительных требований](https://raw.githubusercontent.com/Azure/azure-powershell/master/src/Compute/Compute/Extension/AzureDiskEncryption/Scripts/AzureDiskEncryptionPreRequisiteSetup.ps1 ). Пример использования этого сценария PowerShell см. в статье [Краткое руководство. Шифрование виртуальной машины IaaS под управлением Windows с помощью Azure PowerShell](disk-encryption-powershell-quickstart.md). Вы можете удалить комментарии из раздела сценария, начиная со строки 211, чтобы шифровать все диски имеющихся виртуальных машин в имеющейся группе ресурсов. 

В следующей таблице показано, какие параметры могут использоваться в сценарии PowerShell: 


|Параметр|Описание|Обязательное?|
|------|------|------|
|$resourceGroupName| Имя группы ресурсов, к которой принадлежит хранилище ключей.  При отсутствии группы ресурсов с таким именем — она будет создана.| True|
|$keyVaultName|Имя хранилища ключей, в котором будут размещаться ключи шифрования. При отсутствии хранилища ключей с таким именем — оно будет создано.| True|
|$location|Расположение хранилища ключей. Убедитесь, что хранилище ключей и виртуальные машины, которые предстоит зашифровать, находятся в одном расположении. Получите список расположений с помощью команды `Get-AzLocation`.|True|
|$subscriptionId|Идентификатор подписки Azure для использования.  Вы можете получить идентификатор подписки с помощью команды `Get-AzSubscription`.|True|
|$aadAppName|Имя приложения Azure AD, которое будет использоваться для записи секретов в хранилище ключей. Будет создано приложение с таким именем (если оно еще не создано). Если это приложение уже есть, передайте параметр aadClientSecret в сценарий.|False|
|$aadClientSecret|Секрет клиента приложения Azure AD, который был создан ранее.|False|
|$keyEncryptionKeyName|Имя дополнительного ключа шифрования ключа в хранилище ключей. При отсутствии ключа с таким именем — он будет создан.|False|

### <a name="encrypt-or-decrypt-vms-without-an-azure-ad-app"></a>Шифрование или расшифровка виртуальных машин без приложения Azure AD

- [Включение шифрования дисков на существующей или работающей виртуальной машине Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad)  
- [Отключение шифрования на работающей виртуальной машине Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm-without-aad) 
    - Для виртуальных машин Linux отключение шифрования возможно только для томов данных.  

### <a name="encrypt-or-decrypt-vms-with-an-azure-ad-app-previous-release"></a>Шифрование или расшифровка виртуальных машин с приложением Azure AD (предыдущий выпуск)
 
- [Включение шифрования дисков на существующей или работающей виртуальной машине Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm)    


-  [Отключение шифрования на работающей виртуальной машине Linux](https://github.com/Azure/azure-quickstart-templates/tree/master/201-decrypt-running-linux-vm) 
    - Для виртуальных машин Linux отключение шифрования возможно только для томов данных. 


- [Создание нового зашифрованного управляемого диска из предварительно зашифрованного виртуального жесткого диска или большого двоичного объекта хранилища](https://github.com/Azure/azure-quickstart-templates/tree/master/201-create-encrypted-managed-disk)
    - Создает зашифрованный управляемый диск на основе предварительно зашифрованного виртуального жесткого диска и его соответствующих параметров шифрования.

## <a name="encrypting-an-os-drive-on-a-running-linux-vm"></a>Шифрование диска ОС на работающей виртуальной машине Linux

### <a name="prerequisites-for-os-disk-encryption"></a>Предварительные требования для шифрования диска ОС

* Виртуальная машина должна использовать дистрибутив, совместимый с шифрованием дисков ОС, как указано в [операционных системах, поддерживаемых шифрованием дисков Azure](disk-encryption-overview.md#supported-vms) . 
* Виртуальная машина должна быть создана из образа Marketplace в Azure Resource Manager.
* Виртуальная машина Azure по крайней мере с 4 ГБ ОЗУ (рекомендуемый размер — 7 ГБ).
* (Для RHEL и CentOS.) Отключите SELinux. Чтобы отключить SELinux на виртуальной машине, ознакомьтесь с разделом "4.4.2. Disabling SELinux" (4.4.2. Отключение SELinux) [руководства пользователя и администратора SELinux](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/sect-Security-Enhanced_Linux-Working_with_SELinux-Changing_SELinux_Modes.html#sect-Security-Enhanced_Linux-Enabling_and_Disabling_SELinux-Disabling_SELinux).
* После отключения SELinux перезагрузите виртуальную машину по крайней мере один раз.

### <a name="steps"></a>Шаги
1. Создайте виртуальную машину с помощью одного из дистрибутивов, указанных ранее.

   Для CentOS 7.2 шифрование диска операционной системы можно включить через специальный образ. Чтобы использовать этот образ, задайте при создании виртуальной машины для параметра SKU значение 7.2n.

   ```powershell
    Set-AzVMSourceImage -VM $VirtualMachine -PublisherName "OpenLogic" -Offer "CentOS" -Skus "7.2n" -Version "latest"
   ```
2. Настройте виртуальную машину в соответствии с потребностями. Если вы собираетесь включить шифрование всех дисков (диска ОС и дисков данных), то диски данных необходимо указать и подключить в файле /etc/fstab.

   > [!NOTE]
   > Чтобы указать диски данных в файле /etc/fstab, используйте значение UUID=... (в этом случае не нужно указывать имя блочного устройства, например /dev/sdb1). В процессе шифрования порядок дисков на виртуальной машине изменится. Если на виртуальной машине используется определенный порядок блочных устройств, то при их подключении после шифрования произойдет сбой.

3. Выйдите из сеансов SSH.

4. Чтобы зашифровать операционную систему, укажите параметра volumetype значение как **ALL** или **OS** при включении шифрования.

   > [!NOTE]
   > Все пользовательские процессы, не запущенные как службы `systemd`, необходимо завершить с помощью `SIGKILL`. Перезагрузите виртуальную машину. В случае включения шифрования диска ОС на работающей виртуальной запланируйте период простоя.

5. Периодически отслеживайте ход выполнения шифрования, следуя инструкциям из [следующего раздела](#monitoring-os-encryption-progress).

6. После Get-AzVmDiskEncryptionStatus отобразится "VMRestartPending", перезапустите виртуальную машину, войдя на нее или используя портал, PowerShell или CLI.
    ```powershell
    C:\> Get-AzVmDiskEncryptionStatus  -ResourceGroupName $ResourceGroupName -VMName $VMName
    -ExtensionName $ExtensionName

    OsVolumeEncrypted          : VMRestartPending
    DataVolumesEncrypted       : NotMounted
    OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
    ProgressMessage            : OS disk successfully encrypted, reboot the VM
    ```
   Перед перезагрузкой рекомендуется сохранить [диагностические данные загрузки](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/) виртуальной машины.

## <a name="monitoring-os-encryption-progress"></a>Мониторинг хода выполнения шифрования операционной системы
Мониторинг хода выполнения шифрования ОС можно выполнять тремя способами.

* Используйте командлет `Get-AzVmDiskEncryptionStatus` и просмотрите поле ProgressMessage.
    ```powershell
    OsVolumeEncrypted          : EncryptionInProgress
    DataVolumesEncrypted       : NotMounted
    OsVolumeEncryptionSettings : Microsoft.Azure.Management.Compute.Models.DiskEncryptionSettings
    ProgressMessage            : OS disk encryption started
    ```
  На виртуальных машинах с хранилищем класса Premium шифрование диска ОС длится 40–50 минут с момента появления сообщения "OS disk encryption started" (Начато шифрование диска ОС).

  В связи с [ошибкой № 388](https://github.com/Azure/WALinuxAgent/issues/388) в WALinuxAgent в некоторых дистрибутивах `OsVolumeEncrypted` и `DataVolumesEncrypted` отображаются как `Unknown`. В WALinuxAgent 2.1.5 и более поздних версиях эта проблема исправляется автоматически. Если в выходных данных отображается `Unknown`, состояние шифрования дисков можно проверить с помощью обозревателя ресурсов Azure.

  Откройте [обозреватель ресурсов Azure](https://resources.azure.com/) и на панели выбора слева разверните иерархическую структуру, как показано ниже.

  ```
  |-- subscriptions
     |-- [Your subscription]
          |-- resourceGroups
               |-- [Your resource group]
                    |-- providers
                         |-- Microsoft.Compute
                              |-- virtualMachines
                                   |-- [Your virtual machine]
                                        |-- InstanceView
  ```                

  В InstanceView прокрутите вниз, чтобы увидеть состояние шифрования дисков.

  ![Представление экземпляра виртуальной машины](./media/disk-encryption/vm-instanceview.png)

* Просмотрите [диагностические данные загрузки](https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/). Сообщения расширения шифрования дисков Azure должны начинаться с `[AzureDiskEncryption]`.

* Выполните вход на виртуальную машину с помощью протокола SSH и скопируйте журнал расширения, расположенный по следующему пути:

    /var/log/azure/Microsoft.Azure.Security.AzureDiskEncryptionForLinux

  Мы советуем не входить на виртуальную машину, пока шифруется ее диск ОС. Копируйте журналы только в том случае, когда два других способа не дали результата.

## <a name="prepare-a-pre-encrypted-linux-vhd"></a>Подготовка предварительно зашифрованного виртуального жесткого диска Linux
Шаги подготовки предварительно зашифрованных виртуальных жестких дисков могут отличаться в зависимости от дистрибутива. Доступны примеры для подготовки Ubuntu 16, openSUSE 13.2 и CentOS 7. 

### <a name="ubuntu-16"></a>Ubuntu 16
Настройте шифрование во время установки дистрибутива следующим образом.

1. Во время настройки разделов дисков выберите **Configure encrypted volumes** (Настройка зашифрованных томов).

   ![Настройка зашифрованных томов при настройке Ubuntu 16.04](./media/disk-encryption/ubuntu-1604-preencrypted-fig1.png)

2. Создайте отдельный загрузочный диск (незашифрованный). Зашифруйте корневой диск.

   ![Выбор устройств для шифрования при настройке Ubuntu 16.04](./media/disk-encryption/ubuntu-1604-preencrypted-fig2.png)

3. Укажите парольную фразу. Это парольная фраза, которую вы передали в хранилище ключей.

   ![Указание парольной фразы при настройке Ubuntu 16.04](./media/disk-encryption/ubuntu-1604-preencrypted-fig3.png)

4. Завершите настройку разделов.

   ![Завершение секционирования при настройке Ubuntu 16.04](./media/disk-encryption/ubuntu-1604-preencrypted-fig4.png)

5. После загрузки виртуальной машины отобразится запрос парольной фразы. Введите парольную фразу, указанную на шаге 3.

   ![Указание парольной фразы во время загрузки при настройке Ubuntu 16.04](./media/disk-encryption/ubuntu-1604-preencrypted-fig5.png)

6. Подготовьте виртуальную машину к передаче в Azure, следуя [этим инструкциям](./create-upload-ubuntu.md?toc=/azure/virtual-machines/linux/toc.json). Пока не выполняйте последний шаг (отзыв виртуальной машины).

Настройте шифрование для Azure, выполнив следующие шаги.

1. Создайте файл /usr/local/sbin/azure_crypt_key.sh с приведенным ниже содержимым. Обратите внимание на параметр KeyFileName — это имя файла парольной фразы, используемое Azure.

    ```bash
    #!/bin/sh
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint
    modprobe vfat >/dev/null 2>&1
    modprobe ntfs >/dev/null 2>&1
    sleep 2
    OPENED=0
    cd /sys/block
    for DEV in sd*; do

        echo "> Trying device: $DEV ..." >&2
        mount -t vfat -r /dev/${DEV}1 $MountPoint >/dev/null||
        mount -t ntfs -r /dev/${DEV}1 $MountPoint >/dev/null
        if [ -f $MountPoint/$KeyFileName ]; then
                cat $MountPoint/$KeyFileName
                umount $MountPoint 2>/dev/null
                OPENED=1
                break
        fi
        umount $MountPoint 2>/dev/null
    done

      if [ $OPENED -eq 0 ]; then
        echo "FAILED to find suitable passphrase file ..." >&2
        echo -n "Try to enter your password: " >&2
        read -s -r A </dev/console
        echo -n "$A"
     else
        echo "Success loading keyfile!" >&2
    fi
   ```

2. Измените настройки шифрования в */etc/crypttab*. Он должен выглядеть так:
   ```
    xxx_crypt uuid=xxxxxxxxxxxxxxxxxxxxx none luks,discard,keyscript=/usr/local/sbin/azure_crypt_key.sh
    ```

4. Добавьте в сценарий разрешения для исполняемого файла.
   ```
    chmod +x /usr/local/sbin/azure_crypt_key.sh
   ```
5. Измените */etc/initramfs-tools/modules*, добавив приведенные ниже строки.
   ```
    vfat
    ntfs
    nls_cp437
    nls_utf8
    nls_iso8859-1
   ```
6. Выполните `update-initramfs -u -k all` и обновите initramfs, чтобы `keyscript` вступил в действие.

7. Теперь виртуальную машину можно отозвать.

   ![Обновление initramfs при настройке Ubuntu 16.04](./media/disk-encryption/ubuntu-1604-preencrypted-fig6.png)

8. Перейдите к следующему шагу и передайте свой виртуальный жесткий диск в Azure.

### <a name="opensuse-132"></a>openSUSE 13.2
Чтобы настроить шифрование во время установки дистрибутива, сделайте следующее:
1. При настройке разделов дисков выберите **Encrypt Volume Group** (Шифрование группы томов), а затем введите пароль. Это пароль, который потребуется передать в хранилище ключей.

   ![Шифрование группы томов при настройке openSUSE 13.2](./media/disk-encryption/opensuse-encrypt-fig1.png)

2. Загрузите виртуальную машину, используя указанный пароль.

   ![Указание парольной фразы во время загрузки при настройке openSUSE 13.2](./media/disk-encryption/opensuse-encrypt-fig2.png)

3. Подготовьте виртуальную машину для передачи в Azure, следуя инструкциям в разделе [Подготовка виртуальной машины SLES или openSUSE для Azure](./suse-create-upload-vhd.md?toc=/azure/virtual-machines/linux/toc.json#prepare-opensuse-131). Пока не выполняйте последний шаг (отзыв виртуальной машины).

Чтобы настроить шифрование для Azure, сделайте следующее:
1. Измените файл /etc/dracut.conf и добавьте приведенную ниже строку.
    ```
    add_drivers+=" vfat ntfs nls_cp437 nls_iso8859-1"
    ```
2. Закомментируйте приведенные строки в конце файла /usr/lib/dracut/modules.d/90crypt/module-setup.sh.
   ```bash
    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator
   ```

3. В начале файла /usr/lib/dracut/modules.d/90crypt/parse-crypt.sh добавьте следующую строку.
   ```bash
    DRACUT_SYSTEMD=0
   ```
   Затем измените все вхождения
   ```bash
    if [ -z "$DRACUT_SYSTEMD" ]; then
   ```
   на:
   ```bash
    if [ 1 ]; then
   ```
4. Измените файл/usr/lib/dracut/modules.d/90crypt/cryptroot-Ask.sh и добавьте его в "# Open LUKS Device":

    ```bash
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        cp $MountPoint/$KeyFileName /tmp-keyfile >&2
        luksfile=/tmp-keyfile
        umount $MountPoint >&2
        break
    fi
    done
    ```
5. Выполните `/usr/sbin/dracut -f -v`, чтобы обновить initrd.

6. Теперь виртуальную машину можно отозвать и передать свой виртуальный жесткий диск в Azure.

### <a name="centos-7-and-rhel-7"></a>CentOS 7 и RHEL 7

Чтобы настроить шифрование во время установки дистрибутива, сделайте следующее:
1. Во время настройки разделов дисков выберите **Encrypt my data** (Шифрование личных данных).

   ![Установка назначения при настройке CentOS 7](./media/disk-encryption/centos-encrypt-fig1.png)

2. Включите шифрование для корневого раздела (параметр **Encrypt**).

   ![Выбор шифрования для корневого раздела при настройке CentOS 7](./media/disk-encryption/centos-encrypt-fig2.png)

3. Укажите парольную фразу. Это парольная фраза, которую потребуется передать в хранилище ключей.

   ![Указание парольной фразы при настройке CentOS 7](./media/disk-encryption/centos-encrypt-fig3.png)

4. После загрузки виртуальной машины отобразится запрос парольной фразы. Введите парольную фразу, указанную на шаге 3.

   ![Введение парольной фразы во время загрузки при настройке CentOS 7](./media/disk-encryption/centos-encrypt-fig4.png)

5. Подготовьте виртуальную машину для передачи в Azure, следуя инструкциям в разделе "CentOS 7.0+" статьи [Подготовка виртуальной машины на основе CentOS для Azure](./create-upload-centos.md?toc=/azure/virtual-machines/linux/toc.json#centos-70). Пока не выполняйте последний шаг (отзыв виртуальной машины).

6. Теперь виртуальную машину можно отозвать и передать свой виртуальный жесткий диск в Azure.

Чтобы настроить шифрование для Azure, сделайте следующее:

1. Измените файл /etc/dracut.conf и добавьте приведенную ниже строку.
    ```
    add_drivers+=" vfat ntfs nls_cp437 nls_iso8859-1"
    ```

2. Закомментируйте приведенные строки в конце файла /usr/lib/dracut/modules.d/90crypt/module-setup.sh.
   ```bash
    #        inst_multiple -o \
    #        $systemdutildir/system-generators/systemd-cryptsetup-generator \
    #        $systemdutildir/systemd-cryptsetup \
    #        $systemdsystemunitdir/systemd-ask-password-console.path \
    #        $systemdsystemunitdir/systemd-ask-password-console.service \
    #        $systemdsystemunitdir/cryptsetup.target \
    #        $systemdsystemunitdir/sysinit.target.wants/cryptsetup.target \
    #        systemd-ask-password systemd-tty-ask-password-agent
    #        inst_script "$moddir"/crypt-run-generator.sh /sbin/crypt-run-generator
   ```

3. В начале файла /usr/lib/dracut/modules.d/90crypt/parse-crypt.sh добавьте следующую строку.
   ```bash
    DRACUT_SYSTEMD=0
   ```
   Затем измените все вхождения
   ```bash
    if [ -z "$DRACUT_SYSTEMD" ]; then
   ```
   значение
   ```bash
    if [ 1 ]; then
   ```
4. Измените файл/usr/lib/dracut/modules.d/90crypt/cryptroot-Ask.sh и добавьте следующий текст после "# Open LUKS Device":
    ```bash
    MountPoint=/tmp-keydisk-mount
    KeyFileName=LinuxPassPhraseFileName
    echo "Trying to get the key from disks ..." >&2
    mkdir -p $MountPoint >&2
    modprobe vfat >/dev/null >&2
    modprobe ntfs >/dev/null >&2
    for SFS in /dev/sd*; do
    echo "> Trying device:$SFS..." >&2
    mount ${SFS}1 $MountPoint -t vfat -r >&2 ||
    mount ${SFS}1 $MountPoint -t ntfs -r >&2
    if [ -f $MountPoint/$KeyFileName ]; then
        echo "> keyfile got..." >&2
        cp $MountPoint/$KeyFileName /tmp-keyfile >&2
        luksfile=/tmp-keyfile
        umount $MountPoint >&2
        break
    fi
    done
    ```    
5. Выполните команду "/usr/sbin/dracut-f-v", чтобы обновить initrd.

    ![Выполнение команды /usr/sbin/dracut -f -v при настройке CentOS 7](./media/disk-encryption/centos-encrypt-fig5.png)

## <a name="upload-encrypted-vhd-to-an-azure-storage-account"></a>Передача зашифрованного виртуального жесткого диска в учетную запись хранения Azure
После включения шифрования DM-Crypt необходимо отправить локальный зашифрованный виртуальный жесткий диск в учетную запись хранения.
```powershell
    Add-AzVhd [-Destination] <Uri> [-LocalFilePath] <FileInfo> [[-NumberOfUploaderThreads] <Int32> ] [[-BaseImageUriToPatch] <Uri> ] [[-OverWrite]] [ <CommonParameters>]
```
## <a name="upload-the-secret-for-the-pre-encrypted-vm-to-your-key-vault"></a> Передача секрета для предварительно зашифрованной виртуальной машины в хранилище ключей
При шифровании с использованием приложения Azure AD (предыдущая версия) секрет шифрования дисков, полученный ранее, необходимо передать в хранилище ключей в качестве секрета. В хранилище ключей нужно включить разрешения для клиента Azure AD и шифрование дисков.

```powershell 
 $AadClientId = "My-AAD-Client-Id"
 $AadClientSecret = "My-AAD-Client-Secret"

 $key vault = New-AzKeyVault -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -Location $Location

 Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -ServicePrincipalName $AadClientId -PermissionsToKeys all -PermissionsToSecrets all
 Set-AzKeyVaultAccessPolicy -VaultName $KeyVaultName -ResourceGroupName $ResourceGroupName -EnabledForDiskEncryption
``` 

### <a name="disk-encryption-secret-not-encrypted-with-a-kek"></a>Секрет дискового шифрования не шифруется с помощью KEK
Чтобы настроить секрет в хранилище ключей, используйте [Set-азкэйваултсекрет](/powershell/module/az.keyvault/set-azkeyvaultsecret). Парольная фраза кодируется как строка Base64, а затем передается в хранилище ключей. Убедитесь также, что при создании секрета в хранилище ключей были установлены следующие теги.

```powershell

 # This is the passphrase that was provided for encryption during the distribution installation
 $passphrase = "contoso-password"

 $tags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
 $secretName = [guid]::NewGuid().ToString()
 $secretValue = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($passphrase))
 $secureSecretValue = ConvertTo-SecureString $secretValue -AsPlainText -Force

 $secret = Set-AzKeyVaultSecret -VaultName $KeyVaultName -Name $secretName -SecretValue $secureSecretValue -tags $tags
 $secretUrl = $secret.Id
```


На следующем шаге используйте `$secretUrl`, чтобы [подключить диск ОС, не применяя ключ шифрования ключей](#without-using-a-kek).

### <a name="disk-encryption-secret-encrypted-with-a-kek"></a>Секрет дискового шифрования, зашифрованный с помощью KEK
Перед передачей секрета в хранилище ключей его можно дополнительно зашифровать с помощью ключа шифрования ключей. Используйте [API](/rest/api/keyvault/wrapkey) для создания оболочки, чтобы сначала зашифровать секрет с использованием ключа шифрования ключей. Выходными данными этой операции по переносу является строка в кодировке Base64, которую можно затем передать в качестве секрета с помощью [`Set-AzKeyVaultSecret`](/powershell/module/az.keyvault/set-azkeyvaultsecret) командлета.

```powershell
    # This is the passphrase that was provided for encryption during the distribution installation
    $passphrase = "contoso-password"

    Add-AzKeyVaultKey -VaultName $KeyVaultName -Name "keyencryptionkey" -Destination Software
    $KeyEncryptionKey = Get-AzKeyVaultKey -VaultName $KeyVault.OriginalVault.Name -Name "keyencryptionkey"

    $apiversion = "2015-06-01"

    ##############################
    # Get Auth URI
    ##############################

    $uri = $KeyVault.VaultUri + "/keys"
    $headers = @{}

    $response = try { Invoke-RestMethod -Method GET -Uri $uri -Headers $headers } catch { $_.Exception.Response }

    $authHeader = $response.Headers["www-authenticate"]
    $authUri = [regex]::match($authHeader, 'authorization="(.*?)"').Groups[1].Value

    Write-Host "Got Auth URI successfully"

    ##############################
    # Get Auth Token
    ##############################

    $uri = $authUri + "/oauth2/token"
    $body = "grant_type=client_credentials"
    $body += "&client_id=" + $AadClientId
    $body += "&client_secret=" + [Uri]::EscapeDataString($AadClientSecret)
    $body += "&resource=" + [Uri]::EscapeDataString("https://vault.azure.net")
    $headers = @{}

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $access_token = $response.access_token

    Write-Host "Got Auth Token successfully"

    ##############################
    # Get KEK info
    ##############################

    $uri = $KeyEncryptionKey.Id + "?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token}

    $response = Invoke-RestMethod -Method GET -Uri $uri -Headers $headers

    $keyid = $response.key.kid

    Write-Host "Got KEK info successfully"

    ##############################
    # Encrypt passphrase using KEK
    ##############################

    $passphraseB64 = [Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($Passphrase))
    $uri = $keyid + "/encrypt?api-version=" + $apiversion
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"alg" = "RSA-OAEP"; "value" = $passphraseB64}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method POST -Uri $uri -Headers $headers -Body $body

    $wrappedSecret = $response.value

    Write-Host "Encrypted passphrase successfully"

    ##############################
    # Store secret
    ##############################

    $secretName = [guid]::NewGuid().ToString()
    $uri = $KeyVault.VaultUri + "/secrets/" + $secretName + "?api-version=" + $apiversion
    $secretAttributes = @{"enabled" = $true}
    $secretTags = @{"DiskEncryptionKeyEncryptionAlgorithm" = "RSA-OAEP"; "DiskEncryptionKeyFileName" = "LinuxPassPhraseFileName"}
    $headers = @{"Authorization" = "Bearer " + $access_token; "Content-Type" = "application/json"}
    $bodyObj = @{"value" = $wrappedSecret; "attributes" = $secretAttributes; "tags" = $secretTags}
    $body = $bodyObj | ConvertTo-Json

    $response = Invoke-RestMethod -Method PUT -Uri $uri -Headers $headers -Body $body

    Write-Host "Stored secret successfully"

    $secretUrl = $response.id
```

На следующем шаге мы используем `$KeyEncryptionKey` и `$secretUrl`, чтобы [подключить диск ОС, применив ключ шифрования ключей](#using-a-kek).

##  <a name="specify-a-secret-url-when-you-attach-an-os-disk"></a>Указание URL-адреса секрета при подключении диска ОС

###  <a name="without-using-a-kek"></a>Без использования ключа шифрования ключа
При подключении диска ОС необходимо передать `$secretUrl`. Этот URL-адрес был создан в разделе "Секрет шифрования дисков, не зашифрованный с помощью ключа шифрования ключей".
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $VhdUri `
            -VhdUri $OSDiskUri `
            -Linux `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl
```
### <a name="using-a-kek"></a>С использованием ключа шифрования ключа
При подключении диска ОС передайте `$KeyEncryptionKey` и `$secretUrl`. Этот URL-адрес был создан в разделе "Секрет шифрования дисков, зашифрованный с помощью ключа шифрования ключей".
```powershell
    Set-AzVMOSDisk `
            -VM $VirtualMachine `
            -Name $OSDiskName `
            -SourceImageUri $CopiedTemplateBlobUri `
            -VhdUri $OSDiskUri `
            -Linux `
            -CreateOption FromImage `
            -DiskEncryptionKeyVaultId $KeyVault.ResourceId `
            -DiskEncryptionKeyUrl $SecretUrl `
            -KeyEncryptionKeyVaultId $KeyVault.ResourceId `
            -KeyEncryptionKeyURL $KeyEncryptionKey.Id
```
