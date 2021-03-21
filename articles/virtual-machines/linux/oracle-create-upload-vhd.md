---
title: Создание и передача виртуального жесткого диска Oracle Linux
description: Узнайте, как создать и передать виртуальный жесткий диск (VHD-файл) Azure, содержащий операционную систему Oracle Linux.
author: danielsollondon
ms.service: virtual-machines
ms.collection: linux
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 12/01/2020
ms.author: danis
ms.openlocfilehash: 9984589b19f15ab00e895bca75c295a92a68d0fe
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102557800"
---
# <a name="prepare-an-oracle-linux-virtual-machine-for-azure"></a>Подготовка виртуальной машины Oracle Linux для Azure

В этой статье предполагается, что вы уже установили операционную систему Oracle Linux на виртуальный жесткий диск. Существует несколько средств для создания VHD-файлов, например решение для виртуализации, такое как Hyper-V. Инструкции см. в разделе [Установка роли Hyper-V и настройка виртуальной машины](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

## <a name="oracle-linux-installation-notes"></a>Замечания по установке Oracle Linux
* Дополнительные сведения о подготовке Linux для Azure см. в разделе [Общие замечания по установке Linux](create-upload-generic.md#general-linux-installation-notes).
* Hyper-V и Azure поддерживают Oracle Linux с ядром Unbreakable Enterprise Kernel (UEK) или ядром, совместимым с Red Hat.
* Oracle UEK2 не поддерживается в Hyper-V и Azure, поскольку не включает необходимые драйверы.
* Формат VHDX не поддерживается в Azure, поддерживается только **фиксированный VHD**.  Можно преобразовать диск в формат VHD с помощью диспетчера Hyper-V или командлета convert-vhd.
* **Требуется поддержка ядра для монтирования файловых систем UDF.** При первой загрузке в Azure конфигурация подготовки передается в виртуальную машину Linux через UDF-носитель, подключенный к гостевой машине. Агент Azure Linux должен иметь возможность подключать файловую систему UDF для считывания конфигурации и подготовки виртуальной машины.
* При установке системы Linux рекомендуется использовать стандартные разделы, а не LVM (как правило, значение по умолчанию во многих дистрибутивах). Это позволит избежать конфликта имен LVM при клонировании виртуальных машин, особенно если диск с OC может быть подключен к другой ВМ в целях устранения неполадок. Для дисков данных можно использовать [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) или [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid).
* Версии ядра Linux ниже 2.6.37 не поддерживают NUMA в Hyper-V с виртуальными машинами большего размера. Эта проблема влияет в основном на дистрибутивы более ранних версий, в которых используется вышестоящее ядро Red Hat 2.6.32, и была исправлена в Oracle Linux 6.6 и более поздних версий.
* Не настраивайте раздел подкачки на диске с ОС. Дополнительные сведения описаны далее.
* Размер виртуальной памяти всех VHD в Azure должен быть округлен до 1 МБ. При конвертации диска в формате RAW в виртуальный жесткий диск убедитесь, что размер диска RAW в несколько раз превышает 1 МБ. См. дополнительные сведения в [примечаниях по установке Linux](create-upload-generic.md#general-linux-installation-notes).
* Убедитесь, что `Addons` включен репозиторий. Измените файл `/etc/yum.repos.d/public-yum-ol6.repo` (Oracle Linux 6) или `/etc/yum.repos.d/public-yum-ol7.repo` (Oracle Linux 7) и замените в нем строку `enabled=0` строкой `enabled=1` в разделе **[ol6_addons]** или **[ol7_addons]** .

## <a name="oracle-linux-64-and-later"></a>Oracle Linux 6.4 и более поздние версии
Необходимо выполнить определенные действия с конфигурацией операционной системы, чтобы виртуальная машина запускалась в среде Azure.

1. На центральной панели диспетчера Hyper-V выберите виртуальную машину.
2. Щелкните **Подключение** , чтобы открыть окно виртуальной машины.
3. Удалите NetworkManager, выполнив следующую команду:

    ```console
    # sudo rpm -e --nodeps NetworkManager
    ```

    **Примечание.** Если пакет еще не установлен, эта команда завершится с сообщением об ошибке. Это ожидаемое поведение.
4. Создайте файл с именем **network** в каталоге `/etc/sysconfig/`. Файл должен содержать следующий текст:

    ```config   
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

5. Создайте файл с именем **ifcfg-eth0** в каталоге `/etc/sysconfig/network-scripts/`. Файл должен содержать следующий текст:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

6. Переместите или удалите правила udev, чтобы не создавать статические правила для интерфейса Ethernet. Эти правила приводят к появлению проблем при клонировании виртуальной машины в Microsoft Azure или Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

7. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```console
    # chkconfig network on
    ```

8. Установите python-pyasn1, выполнив следующую команду:

    ```console
    # sudo yum install python-pyasn1
    ```

9. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл /boot/grub/menu.lst в текстовом редакторе и убедитесь в том, что ядро включает следующие параметры:

    ```config-grub
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

   Это гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки.
   
   Помимо вышесказанного, рекомендуется *удалить* следующие параметры:

    ```config-grub
    rhgb quiet crashkernel=auto
    ```

   Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт.
   
   При желании можно настроить параметр `crashkernel` , однако учитывайте, что он сокращает объем доступной памяти в виртуальной машине на 128 МБ или более, что может оказаться проблемой в виртуальных машинах небольшого размера.
10. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки.  Обычно это сделано по умолчанию.
11. Установите агент Linux для Azure, выполнив следующую команду. Последней является версия 2.0.15.

    ```console
    # sudo yum install WALinuxAgent
    ```

    Обратите внимание, что установка пакета WALinuxAgent приведет к удалению пакетов NetworkManager и NetworkManager-gnome, если они еще не были удалены, как описано в шаге 2.
12. Не создавайте пространство подкачки на диске с ОС.
    
    Агент Linux для Azure может автоматически настраивать пространство подкачки с использованием диска на локальном ресурсе, подключенном к виртуальной машине после подготовки для работы в среде Azure. Следует отметить, что локальный диск ресурсов является *временным* диском и должен быть очищен при отмене подготовки виртуальной машины. После установки агента Linux для Azure (см. предыдущий шаг) измените следующие параметры в /etc/waagent.conf соответствующим образом:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

13. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

    ```console
    # sudo waagent -force -deprovision
    # export HISTSIZE=0
    # logout
    ```

14. В диспетчере Hyper-V выберите **Действие -> Завершение работы**. Виртуальный жесткий диск Linux готов к передаче в Azure.

---
## <a name="oracle-linux-70-and-later"></a>Oracle Linux 7.0 и более поздние версии
**Изменения в Oracle Linux 7**

Подготовка виртуальной машины Oracle Linux 7 для Azure в значительной степени аналогична версии Oracle Linux 6, однако есть несколько важных отличий:

* Azure поддерживает Oracle Linux с ядром Unbreakable Enterprise Kernel (UEK) или ядром, совместимым с Red Hat. Рекомендуется использовать Oracle Linux с UEK.
* Пакет NetworkManager больше не конфликтует с агентом Azure Linux. Этот пакет устанавливается по умолчанию, и мы рекомендуем не удалять его.
* В качестве загрузчика по умолчанию теперь используется GRUB2, поэтому изменилась процедура правки параметров ядра (см. ниже).
* XFS теперь является файловой системой по умолчанию. При желании можно продолжать использование файловой системы ext4.

**Шаги настройки**

1. В диспетчере Hyper-V выберите виртуальную машину.
2. Щелкните **Подключение** , чтобы открыть окно консоли для виртуальной машины.
3. Создайте файл с именем **network** в каталоге `/etc/sysconfig/`. Файл должен содержать следующий текст:

    ```config
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

4. Создайте файл с именем **ifcfg-eth0** в каталоге `/etc/sysconfig/network-scripts/`. Файл должен содержать следующий текст:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

5. Переместите или удалите правила udev, чтобы не создавать статические правила для интерфейса Ethernet. Эти правила приводят к появлению проблем при клонировании виртуальной машины в Microsoft Azure или Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    ```

6. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```console
    # sudo chkconfig network on
    ```

7. Установите пакет python-pyasn1, выполнив следующую команду:

    ```console
    # sudo yum install python-pyasn1
    ```

8. Выполните следующую команду, чтобы очистить текущие метаданные yum и установить все обновления:

    ```console 
    # sudo yum clean all
    # sudo yum -y update
    ```

9. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл /etc/default/grub в текстовом редакторе и измените параметр `GRUB_CMDLINE_LINUX`, например:

    ```config-grub
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   Это также гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки. При этом также отключаются соглашения об именовании сетевых карт в Oracle Linux 7 с Unbreakable Enterprise Kernel. Помимо вышесказанного, рекомендуется *удалить* следующие параметры:

    ```config-grub
       rhgb quiet crashkernel=auto
    ```
 
   Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт.
   
   При желании можно настроить параметр `crashkernel` , однако учитывайте, что он сокращает объем доступной памяти в виртуальной машине на 128 МБ или более, что может оказаться проблемой в виртуальных машинах небольшого размера.
10. После внесения изменений в файл "/etc/default/grub", как указано выше, выполните следующую команду для повторного создания конфигурации grub:

    ```console
    # sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

11. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки.  Обычно это сделано по умолчанию.
12. Установите агент Linux для Azure и зависимости:

    ```bash
    sudo yum install WALinuxAgent
    sudo systemctl enable waagent
    ```

13. Установка Cloud-init для решения подготовки

    ```console
    yum install -y cloud-init cloud-utils-growpart gdisk hyperv-daemons

    # Configure waagent for cloud-init
    sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
    sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf


    echo "Adding mounts and disk_setup to init stage"
    sed -i '/ - mounts/d' /etc/cloud/cloud.cfg
    sed -i '/ - disk_setup/d' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - mounts' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - disk_setup' /etc/cloud/cloud.cfg

    echo "Allow only Azure datasource, disable fetching network setting via IMDS"
    cat > /etc/cloud/cloud.cfg.d/91-azure_datasource.cfg <<EOF
    datasource_list: [ Azure ]
    datasource:
    Azure:
        apply_network_config: False
    EOF

    if [[ -f /mnt/resource/swapfile ]]; then
    echo Removing swapfile - RHEL uses a swapfile by default
    swapoff /mnt/resource/swapfile
    rm /mnt/resource/swapfile -f
    fi

    echo "Add console log file"
    cat >> /etc/cloud/cloud.cfg.d/05_logging.cfg <<EOF

    # This tells cloud-init to redirect its stdout and stderr to
    # 'tee -a /var/log/cloud-init-output.log' so the user can see output
    # there without needing to look on the console.
    output: {all: '| tee -a /var/log/cloud-init-output.log'}
    EOF

    ```

14. Конфигурация переключения не создает пространство подкачки на диске операционной системы.

    Ранее агент Linux для Azure использовал автоматическую настройку области подкачки с помощью локального диска ресурсов, подключенного к виртуальной машине после подготовки виртуальной машины в Azure. Однако это теперь обрабатывается с помощью Cloud-init, поэтому **не следует** использовать агент Linux для форматирования диска ресурсов. Создайте файл подкачки, измените следующие параметры соответствующим образом `/etc/waagent.conf` :

    ```console
    sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf
    ```

    Если требуется подключить, отформатировать и создать подкачку, можно выполнить одно из следующих действий.
    * Передавайте это в качестве конфигурации Cloud-init при каждом создании виртуальной машины.
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
    # Note: if you are migrating a specific virtual machine and do not wish to create a generalized image,
    # skip the deprovision step
    # sudo rm -rf /var/lib/waagent/
    # sudo rm -f /var/log/waagent.log

    # waagent -force -deprovision+user
    # rm -f ~/.bash_history
    

    # export HISTSIZE=0

    # logout
    ```

16. В диспетчере Hyper-V выберите **Действие -> Завершение работы**. Виртуальный жесткий диск Linux готов к передаче в Azure.

## <a name="next-steps"></a>Дальнейшие действия
Теперь вы можете создавать новые виртуальные машины в Azure с помощью VHD-файла системы Oracle Linux. Если вы отправляете VHD-файл в Azure впервые, см. раздел [Вариант 1. Передача VHD](upload-vhd.md#option-1-upload-a-vhd).