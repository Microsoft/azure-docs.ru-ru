---
title: Создание и передача виртуального жесткого диска с операционной системой SUSE Linux в Azure
description: Узнайте, как создать и передать виртуальный жесткий диск (VHD-файл) Azure, содержащий операционную систему SUSE Linux.
author: danielsollondon
ms.service: virtual-machines
ms.subservice: imaging
ms.collection: linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 12/01/2020
ms.author: danis
ms.openlocfilehash: 276f5f4542ecea42c665764b8c4e5f66f2531126
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102552717"
---
# <a name="prepare-a-sles-or-opensuse-virtual-machine-for-azure"></a>Подготовка виртуальной машины SLES или openSUSE для Azure


В этой статье предполагается, что вы уже установили операционную систему SUSE или openSUSE Linux на виртуальный жесткий диск. Существует несколько средств для создания VHD-файлов, например решение для виртуализации, такое как Hyper-V. Инструкции см. в разделе [Установка роли Hyper-V и настройка виртуальной машины](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

## <a name="sles--opensuse-installation-notes"></a>Замечания по установке SLES и openSUSE
* Дополнительные сведения о подготовке Linux для Azure см. в разделе [Общие замечания по установке Linux](create-upload-generic.md#general-linux-installation-notes).
* Формат VHDX не поддерживается в Azure, поддерживается только **фиксированный VHD**.  Можно преобразовать диск в формат VHD с помощью диспетчера Hyper-V или командлета convert-vhd.
* При установке системы Linux рекомендуется использовать стандартные разделы, а не LVM (как правило, значение по умолчанию во многих дистрибутивах). Это позволит избежать конфликта имен LVM при клонировании виртуальных машин, особенно если диск с OC может быть подключен к другой ВМ в целях устранения неполадок. Для дисков данных можно использовать [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) или [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid).
* Не настраивайте раздел подкачки на диске с ОС. Можно настроить агент Linux для создания файла подкачки на временном диске ресурсов.  Дополнительные сведения описаны далее.
* Размер виртуальной памяти всех VHD в Azure должен быть округлен до 1 МБ. При конвертации диска в формате RAW в виртуальный жесткий диск убедитесь, что размер диска RAW в несколько раз превышает 1 МБ. См. дополнительные сведения в [примечаниях по установке Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="use-suse-studio"></a>Использование SUSE Studio
[SUSE Studio](https://studioexpress.opensuse.org/) можно с легкостью использовать для создания образов SLES и openSUSE для Azure и Hyper-V, а также для управления этими образами. Этот подход рекомендуется для настройки собственных образов SLES и openSUSE.

В качестве альтернативы созданию собственного виртуального жесткого диска SUSE также публикует образы BYOS (присвоить собственные подписки) для SLES на [виртуальной машине хранилище](https://www.microsoft.com/research/wp-content/uploads/2016/04/using-and-contributing-vms-to-vm-depot.pdf).

## <a name="prepare-suse-linux-enterprise-server-for-azure"></a>Подготовка SUSE Linux Enterprise Server для Azure
1. На центральной панели диспетчера Hyper-V выберите виртуальную машину.
2. Щелкните **Подключение** , чтобы открыть окно виртуальной машины.
3. Зарегистрируйте систему SUSE Linux Enterprise, чтобы позволить ей скачивать обновления и устанавливать пакеты.
4. Установите в системе последние исправления:

    ```console
    # sudo zypper update
    ```
    
5. Установка агента Linux для Azure и Cloud-init

    ```console
    # SUSEConnect -p sle-module-public-cloud/15.2/x86_64  (SLES 15 SP2)
    # sudo zypper refresh
    # sudo zypper install python-azure-agent
    # sudo zypper install cloud-init
    ```

6. Включение waagent & Cloud-init для запуска при загрузке

    ```console
    # sudo chkconfig waagent on
    # systemctl enable cloud-init-local.service
    # systemctl enable cloud-init.service
    # systemctl enable cloud-config.service
    # systemctl enable cloud-final.service
    # systemctl daemon-reload
    # cloud-init clean
    ```

7. Обновление конфигурации waagent и Cloud-init

    ```console
    # sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
    # sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf

    # sudo sh -c 'printf "datasource:\n  Azure:" > /etc/cloud/cloud.cfg.d/91-azure_datasource.cfg'
    # sudo sh -c 'printf "reporting:\n  logging:\n    type: log\n  telemetry:\n    type: hyperv" > /etc/cloud/cloud.cfg.d/10-azure-kvp.cfg'
    ```

8. Измените файл/etc/default/grub, чтобы обеспечить отправку журналов консоли в последовательный порт, а затем обновите основной файл конфигурации с помощью grub2-mkconfig-o/Boot/grub2/GRUB.cfg

    ```config-grub
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```
    Это гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки.
    
9. Убедитесь, что файл/etc/fstab ссылается на диск по его UUID (по UUID)
         
10. Переместите или удалите правила udev, чтобы не создавать статические правила для интерфейса Ethernet. Эти правила приводят к появлению проблем при клонировании виртуальной машины в Microsoft Azure или Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```
   
11. Рекомендуется отредактировать файл /etc/sysconfig/network/dhcp и изменить параметр `DHCLIENT_SET_HOSTNAME` следующим образом:

    ```config
    DHCLIENT_SET_HOSTNAME="no"
    ```

12. В "/etc/sudoers" закомментируйте или удалите следующие строки, если они существуют:

    ```text
    Defaults targetpw   # ask for the password of the target user i.e. root
    ALL    ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!
    ```

13. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки. Обычно это сделано по умолчанию.

14. Переключить конфигурацию
 
    Не создавайте пространство подкачки на диске ОС.

    Ранее агент Linux для Azure использовал автоматическую настройку области подкачки с помощью локального диска ресурсов, подключенного к виртуальной машине после подготовки виртуальной машины в Azure. Однако это теперь обрабатывается с помощью Cloud-init, поэтому **не следует** использовать агент Linux для форматирования диска ресурсов. Создайте файл подкачки, измените следующие параметры соответствующим образом `/etc/waagent.conf` :

    ```console
    # sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    # sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf
    ```

    Если требуется подключить, отформатировать и создать подкачку, можно выполнить одно из следующих действий.
    * Передайте это в качестве конфигурации Cloud-init при каждом создании виртуальной машины.
    * Используйте директиву Cloud-init, помогут в образ, который будет выполнять это при каждом создании виртуальной машины:

        ```console
        cat > /etc/cloud/cloud.cfg.d/00-azure-swap.cfg << EOF
        #cloud-config
        # Generated by Azure cloud image build
        disk_setup:
          ephemeral0:
            table_type: mbr
            layout: [66, [33, 82]]
            overwrite: True
        fs_setup:
          - device: ephemeral0.1
            filesystem: ext4
          - device: ephemeral0.2
            filesystem: swap
        mounts:
          - ["ephemeral0.1", "/mnt"]
          - ["ephemeral0.2", "none", "swap", "sw", "0", "0"]
        EOF
        ```

15. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

    ```console
    # sudo rm -rf /var/lib/waagent/
    # sudo rm -f /var/log/waagent.log

    # waagent -force -deprovision+user
    # rm -f ~/.bash_history
    

    # export HISTSIZE=0

    # logout
    ```
16. В диспетчере Hyper-V выберите **Действие -> Завершение работы**. Виртуальный жесткий диск Linux готов к передаче в Azure.

---
## <a name="prepare-opensuse-131"></a>Подготовка openSUSE 13.1+
1. На центральной панели диспетчера Hyper-V выберите виртуальную машину.
2. Щелкните **Подключение** , чтобы открыть окно виртуальной машины.
3. В консоли оболочки выполните команду`zypper lr`. Если эта команда возвращает результат следующего вида, то репозитории настроены надлежащим образом, и никаких изменений не требуется (обратите внимание, что номера версий могут отличаться):

   | # | Псевдоним                 | Имя                  | Активировано | Обновить
   | - | :-------------------- | :-------------------- | :------ | :------
   | 1 | Облако: Tools_13.1      | Облако: Tools_13.1      | Да     | Да
   | 2 | openSUSE_13 openSUSE_13.1_OSS     | openSUSE_13 openSUSE_13.1_OSS     | Да     | Да
   | 3 | openSUSE_13 openSUSE_13.1_Updates | openSUSE_13 openSUSE_13.1_Updates | Да     | Да

    Если команда возвращает сообщение "No repositories defined..." (Репозитории не определены...), используйте следующие команды для добавления этих репозиториев:

    ```console
    # sudo zypper ar -f http://download.opensuse.org/repositories/Cloud:Tools/openSUSE_13.1 Cloud:Tools_13.1
    # sudo zypper ar -f https://download.opensuse.org/distribution/13.1/repo/oss openSUSE_13.1_OSS
    # sudo zypper ar -f http://download.opensuse.org/update/13.1 openSUSE_13.1_Updates
    ```

    Чтобы убедиться, что репозитории добавлены, можно еще раз выполнить команду`zypper lr`. Если один из соответствующих репозиториев обновлений не включен, включите его, выполнив следующую команду:

    ```console
    # sudo zypper mr -e [NUMBER OF REPOSITORY]
    ```

4. Обновите ядро до последней доступной версии:

    ```console
    # sudo zypper up kernel-default
    ```

    Либо установите в системе все последние исправления:

    ```console
    # sudo zypper update
    ```

5. Установите агент Linux для Azure.

    ```console
    # sudo zypper install WALinuxAgent
    ```

6. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл /boot/grub/menu.lst в текстовом редакторе и убедитесь, что ядро по умолчанию включает следующие параметры:

    ```config-grub
     console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

   Это гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки. Кроме того, удалите следующие параметры из строки загрузки ядра, если таковые существуют:

    ```config-grub
     libata.atapi_enabled=0 reserve=0x1f0,0x8
    ```

7. Рекомендуется отредактировать файл /etc/sysconfig/network/dhcp и изменить параметр `DHCLIENT_SET_HOSTNAME` следующим образом:

    ```config
     DHCLIENT_SET_HOSTNAME="no"
    ```

8. **Внимание!** В "/etc/sudoers" закомментируйте или удалите следующие строки, если они существуют:

    ```text
    Defaults targetpw   # ask for the password of the target user i.e. root
    ALL    ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!
    ```

9. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки.  Обычно это сделано по умолчанию.
10. Не создавайте пространство подкачки на диске с ОС.

    Агент Linux для Azure может автоматически настраивать пространство подкачки с использованием диска на локальном ресурсе, подключенном к виртуальной машине после подготовки для работы в среде Azure. Следует отметить, что локальный диск ресурсов является *временным* диском и должен быть очищен при отмене подготовки виртуальной машины. После установки агента Linux для Azure (см. предыдущий шаг) измените следующие параметры в /etc/waagent.conf соответствующим образом:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

11. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

    ```console
    # sudo waagent -force -deprovision
    # export HISTSIZE=0
    # logout
    ```

12. Убедитесь, что агент Linux для Azure запускается при загрузке:

    ```console
    # sudo systemctl enable waagent.service
    ```

13. В диспетчере Hyper-V выберите **Действие -> Завершение работы**. Виртуальный жесткий диск Linux готов к передаче в Azure.

## <a name="next-steps"></a>Дальнейшие действия
Теперь виртуальный жесткий диск SUSE Linux можно использовать для создания новых виртуальных машин Azure. Если вы отправляете VHD-файл в Azure впервые, см. раздел [Вариант 1. Передача VHD](upload-vhd.md#option-1-upload-a-vhd).