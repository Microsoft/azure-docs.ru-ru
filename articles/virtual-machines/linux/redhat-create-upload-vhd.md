---
title: Создание и передача виртуального жесткого диска Red Hat Enterprise Linux для использования в Azure
description: Узнайте, как создать и передать виртуальный жесткий диск (VHD) Azure, содержащий операционную систему RedHat Linux.
author: danielsollondon
ms.service: virtual-machines
ms.subservice: redhat
ms.collection: linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: how-to
ms.date: 12/01/2020
ms.author: danis
ms.openlocfilehash: ff22010f12a7d76e60b629c4c1c4e24946d4af5a
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102552972"
---
# <a name="prepare-a-red-hat-based-virtual-machine-for-azure"></a>Подготовка виртуальной машины на основе Red Hat для Azure

В этой статье вы узнаете, как подготовить виртуальную машину Red Hat Enterprise Linux (RHEL) для использования в Azure. В статье описываются версии RHEL 6.7+ и 7.1+. Низкоуровневые оболочки для подготовки, о которых идет речь в этой статье, — это Hyper-V, Kernel-based Virtual Machine (KVM) и VMware. Подробнее о требованиях к участникам в программе Red Hat Cloud Access см. на [веб-сайте Red Hat Cloud Access](https://www.redhat.com/en/technologies/cloud-computing/cloud-access) и странице [запуска RHEL в Azure](https://access.redhat.com/ecosystem/ccsp/microsoft-azure). Способы автоматизации создания образов RHEL см. в разделе [Azure Image Builder](../image-builder-overview.md).

## <a name="hyper-v-manager"></a>В диспетчере Hyper-V

В этом разделе показано, как подготовить виртуальную машину [RHEL 6](#rhel-6-using-hyper-v-manager), [RHEL 7](#rhel-7-using-hyper-v-manager)или [RHEL 8](#rhel-8-using-hyper-v-manager) с помощью диспетчера Hyper-V.

### <a name="prerequisites"></a>Предварительные условия
В этом разделе предполагается, что вы уже получили ISO-файл с веб-сайта Red Hat и установили образ RHEL на виртуальный жесткий диск. Дополнительные сведения о том, как использовать диспетчер Hyper-V для установки образа операционной системы, см. в статье [Установка Hyper-V и создание виртуальной машины](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

**Замечания по установке RHEL**

* Формат VHDX не поддерживается в Azure. В Azure поддерживается только формат фиксированного виртуального жесткого диска. Вы можете преобразовать диск в формат VHD с помощью диспетчера Hyper-V или командлета convert-vhd. Если вы используете VirtualBox, при создании диска выберите **фиксированный размер** вместо динамически выделяемого (по умолчанию).
* Azure поддерживает виртуальные машины Gen1 (Boot BIOS) & Gen2 (Загрузка UEFI).
* Максимально допустимый размер виртуального жесткого диска составляет 1023 ГБ.
* Диспетчер логических томов (LVM) поддерживается и может использоваться для дисков операционной системы или дисков с данными на виртуальных машинах Azure. Однако, как правило, для дисков операционной системы рекомендуется использовать стандартные разделы, а не LVM. Это позволит избежать конфликта имен LVM при клонировании виртуальных машин, особенно если диск с OC может быть подключен к другой идентичной виртуальной машине в целях устранения неполадок. См. также документацию по [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) и [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid).
* **Требуется поддержка ядра для подключения файловых систем UDF**. При первой загрузке в Azure UDF-носитель, подключенный к гостевой машине, передает конфигурацию подготовки в виртуальную машину Linux. Агент Linux для Azure должен иметь возможность подключить файловую систему UDF для считывания конфигурации и подготовки виртуальной машины без этого, подготовка завершится ошибкой.
* Не настраивайте раздел swap на диске операционной системы. Дополнительные сведения описаны ниже.

* Размер виртуальной памяти всех VHD в Azure должен быть округлен до 1 МБ. При конвертации диска в формате RAW в виртуальный жесткий диск убедитесь, что размер диска RAW в несколько раз превышает 1 МБ. Дополнительные сведения можно найти в инструкциях ниже. См. также дополнительные сведения в [примечаниях по установке Linux](create-upload-generic.md#general-linux-installation-notes).

### <a name="rhel-6-using-hyper-v-manager"></a>RHEL 6 с использованием диспетчера Hyper-V

1. В диспетчере Hyper-V выберите виртуальную машину.

1. Щелкните **Подключение** , чтобы открыть окно консоли для виртуальной машины.

1. В RHEL 6 NetworkManager может мешать работе агента Linux для Azure. Установите пакет, выполнив следующую команду:

    ```console
    # sudo rpm -e --nodeps NetworkManager
    ```

1. Создайте или измените файл `/etc/sysconfig/network`, добавив следующий текст:

    ```config
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Создайте или измените файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, добавив следующий текст:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

1. Переместите (или удалите) правила udev, чтобы не создавать статические правила для интерфейса Ethernet. Эти правила приводят к появлению проблем при клонировании виртуальной машины в Microsoft Azure или Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

1. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```console
    # sudo chkconfig network on
    ```

1. Зарегистрируйте подписку Red Hat, чтобы активировать установку пакетов из репозитория RHEL, запустив следующую команду:

    ```console
    # sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Пакет WALinuxAgent `WALinuxAgent-<version>` был отправлен в репозиторий дополнений Red Hat. Включите репозиторий дополнений, выполнив следующую команду:

    ```console
    # subscription-manager repos --enable=rhel-6-server-extras-rpms
    ```

1. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/boot/grub/menu.lst` в текстовом редакторе и укажите для ядра по умолчанию следующие параметры:

    ```config-grub
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

    Это также гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки.
    
    В дополнение к вышесказанному мы рекомендуем удалить следующие параметры:

    ```config-grub
    rhgb quiet crashkernel=auto
    ```
    
    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт.  При желании можно оставить параметр `crashkernel`. Обратите внимание, что этот параметр сокращает объем доступной памяти на виртуальной машине на 128 MБ и более. Это может оказаться проблемой для виртуальных машин небольшого размера.


1. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки (что обычно задано по умолчанию). Измените файл /etc/ssh/sshd_config, включив в него следующую строку:

    ```config
    ClientAliveInterval 180
    ```

1. Установите агент Linux для Azure, выполнив следующую команду:

    ```console
    # sudo yum install WALinuxAgent

    # sudo chkconfig waagent on
    ```

    Установка пакета WALinuxAgent приводит к удалению пакетов NetworkManager и NetworkManager-gnome, если они еще не были удалены, как описано на шаге 3.

1. Не создавайте пространство подкачки на диске ОС.

    Агент Linux для Azure может автоматически настраивать размер области подкачки с использованием локального диска ресурсов, подключенного к виртуальной машине после ее подготовки в Azure. Обратите внимание, что локальный диск ресурсов является временным диском и может быть очищен, если виртуальная машина освобождается. После установки агента Linux для Azure (см. предыдущий шаг) измените соответствующим образом следующие параметры в файле /etc/waagent.conf:

    ```config-cong
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

1. Отмените регистрацию подписки (при необходимости), выполнив следующую команду:

    ```console
    # sudo subscription-manager unregister
    ```

1. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

    ```console
    # Note: if you are migrating a specific virtual machine and do not wish to create a generalized image,
    # skip the deprovision step
    # sudo waagent -force -deprovision

    # export HISTSIZE=0

    # logout
    ```

1. Щелкните **действие**  >  **Завершение работы** в диспетчере Hyper-V. Виртуальный жесткий диск Linux готов к передаче в Azure.


### <a name="rhel-7-using-hyper-v-manager"></a>RHEL 7 с помощью диспетчера Hyper-V

1. В диспетчере Hyper-V выберите виртуальную машину.

1. Щелкните **Подключение** , чтобы открыть окно консоли для виртуальной машины.

1. Создайте или измените файл `/etc/sysconfig/network`, добавив следующий текст:

    ```config
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Создайте или измените файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, добавив следующий текст:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    PERSISTENT_DHCLIENT=yes
    NM_CONTROLLED=yes
    ```

1. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```console
    # sudo systemctl enable network
    ```

1. Зарегистрируйте подписку Red Hat, чтобы активировать установку пакетов из репозитория RHEL, запустив следующую команду:

    ```console
    # sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/etc/default/grub` в текстовом редакторе и измените параметр `GRUB_CMDLINE_LINUX`. Пример:

    
    ```config-grub
    GRUB_CMDLINE_LINUX="rootdelay=300 console=tty1 console=ttyS0,115200n8 earlyprintk=ttyS0,115200 earlyprintk=ttyS0 net.ifnames=0"
    GRUB_TERMINAL_OUTPUT="serial console"
    GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1
    ```
   
    Это также обеспечит отправку всех сообщений консоли первому последовательному порту и включение взаимодействия с последовательной консолью, что может помочь в поддержке Azure с проблемами отладки. Также будут отключены новые соглашения об именовании RHEL 7 для сетевых карт.

    ```config
    rhgb quiet crashkernel=auto
    ```
   
    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт. При желании можно оставить параметр `crashkernel`. Обратите внимание, что этот параметр сокращает объем доступной памяти на виртуальной машине на 128 MБ и более. Это может оказаться проблемой для виртуальных машин небольшого размера.

1. После внесения изменений в файл `/etc/default/grub`выполните следующую команду, чтобы повторно создать конфигурацию grub:

    ```console
    # sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```
    > [!NOTE]
    > При отправке виртуальной машины с поддержкой UEFI команда для обновления GRUB имеет значение `grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg` .

1. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки (что обычно задано по умолчанию). Измените файл `/etc/ssh/sshd_config` , добавив в него следующую строку:

    ```config
    ClientAliveInterval 180
    ```

1. Пакет WALinuxAgent `WALinuxAgent-<version>` был отправлен в репозиторий дополнений Red Hat. Включите репозиторий дополнений, выполнив следующую команду:

    ```console
    # subscription-manager repos --enable=rhel-7-server-extras-rpms
    ```

1. Установите агент Linux для Azure, Cloud-init и другие необходимые служебные программы, выполнив следующую команду:

    ```console
    # sudo yum install -y WALinuxAgent cloud-init cloud-utils-growpart gdisk hyperv-daemons

    # sudo systemctl enable waagent.service
    # sudo systemctl enable cloud-init.service
    ```

1. Настройте Cloud-init для выполнения подготовки:

    1. Настройка waagent для Cloud-init:

    ```console
    sed -i 's/Provisioning.Agent=auto/Provisioning.Agent=cloud-init/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf
    ```
    > [!NOTE]
    > Если вы переносите определенную виртуальную машину и не хотите создавать обобщенные образы, установите `Provisioning.Agent=disabled` в `/etc/waagent.conf` конфигурации.
    
    1. Настройка подключений:

    ```console
    echo "Adding mounts and disk_setup to init stage"
    sed -i '/ - mounts/d' /etc/cloud/cloud.cfg
    sed -i '/ - disk_setup/d' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - mounts' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - disk_setup' /etc/cloud/cloud.cfg
    ```
    
    1. Настройка источника данных Azure:

    ```console
    echo "Allow only Azure datasource, disable fetching network setting via IMDS"
    cat > /etc/cloud/cloud.cfg.d/91-azure_datasource.cfg <<EOF
    datasource_list: [ Azure ]
    datasource:
        Azure:
            apply_network_config: False
    EOF
    ```

    1. Если настроено, удалите существующие файл подкачки:

    ```console
    if [[ -f /mnt/resource/swapfile ]]; then
    echo "Removing swapfile" #RHEL uses a swapfile by defaul
    swapoff /mnt/resource/swapfile
    rm /mnt/resource/swapfile -f
    fi
    ```
    1. Настройка ведения журнала Cloud-init:
    ```console
    echo "Add console log file"
    cat >> /etc/cloud/cloud.cfg.d/05_logging.cfg <<EOF

    # This tells cloud-init to redirect its stdout and stderr to
    # 'tee -a /var/log/cloud-init-output.log' so the user can see output
    # there without needing to look on the console.
    output: {all: '| tee -a /var/log/cloud-init-output.log'}
    EOF

    ```

1. Конфигурация переключения не создает пространство подкачки на диске операционной системы.

    Ранее агент Linux для Azure использовал автоматическую настройку области подкачки с помощью локального диска ресурсов, подключенного к виртуальной машине после подготовки виртуальной машины в Azure. Однако это теперь обрабатывается с помощью Cloud-init, поэтому **не следует** использовать агент Linux для форматирования диска ресурсов. Создайте файл подкачки, измените следующие параметры соответствующим образом `/etc/waagent.conf` :

    ```console
    ResourceDisk.Format=n
    ResourceDisk.EnableSwap=n
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
1. Чтобы отменить регистрацию подписки, выполните следующую команду:

    ```console
    # sudo subscription-manager unregister
    ```

1. Отзыв

    Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

    > [!CAUTION]
    > Если вы переносите определенную виртуальную машину и не хотите создавать обобщенные образы, пропустите этап отмены наполнения. Выполнение команды `waagent -force -deprovision` сделает исходный компьютер непригодным для использования. Этот шаг предназначен только для создания обобщенного образа.
    ```console
    # sudo waagent -force -deprovision

    # export HISTSIZE=0

    # logout
    ```
    

1. Щелкните **действие**  >  **Завершение работы** в диспетчере Hyper-V. Виртуальный жесткий диск Linux готов к передаче в Azure.

### <a name="rhel-8-using-hyper-v-manager"></a>RHEL 8 с помощью диспетчера Hyper-V

1. В диспетчере Hyper-V выберите виртуальную машину.

1. Щелкните **Подключение** , чтобы открыть окно консоли для виртуальной машины.

1. Убедитесь, что служба диспетчера сети запустится во время загрузки, выполнив следующую команду:

    ```console
    # sudo systemctl enable NetworkManager.service
    ```

1. Настройте сетевой интерфейс для автоматического запуска при загрузке и используйте DHCP:

    ```console
    # nmcli con mod eth0 connection.autoconnect yes ipv4.method auto
    ```


1. Зарегистрируйте подписку Red Hat, чтобы активировать установку пакетов из репозитория RHEL, запустив следующую команду:

    ```console
    # sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Измените строку загрузки ядра в конфигурации GRUB, чтобы включить дополнительные параметры ядра для Azure и включить последовательную консоль. 

    1. Удалить текущие параметры GRUB:
    ```console
    # grub2-editenv - unset kernelopts
    ```

    1. Измените `/etc/default/grub` в текстовом редакторе и добавьте следующий примечанием:

    ```config-grub
    GRUB_CMDLINE_LINUX="rootdelay=300 console=tty1 console=ttyS0,115200n8 earlyprintk=ttyS0,115200 earlyprintk=ttyS0 net.ifnames=0"
    GRUB_TERMINAL_OUTPUT="serial console"
    GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
    ```
   
   Это также обеспечит отправку всех сообщений консоли первому последовательному порту и включение взаимодействия с последовательной консолью, что может помочь в поддержке Azure с проблемами отладки. Эта конфигурация также отключает новые соглашения об именовании сетевых интерфейсов.
   
   1. Кроме того, рекомендуется удалить следующие параметры:

    ```config
    rhgb quiet crashkernel=auto
    ```
   
    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт. При желании можно оставить параметр `crashkernel`. Обратите внимание, что этот параметр сокращает объем доступной памяти на виртуальной машине на 128 MБ и более. Это может оказаться проблемой для виртуальных машин небольшого размера.

1. После внесения изменений в файл `/etc/default/grub`выполните следующую команду, чтобы повторно создать конфигурацию grub:

    ```console
    # sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```
    Для виртуальной машины с включенной UEFI выполните следующую команду:

    ```console
    # sudo grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
    ```

1. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки (что обычно задано по умолчанию). Измените файл `/etc/ssh/sshd_config` , добавив в него следующую строку:

    ```config
    ClientAliveInterval 180
    ```

1. Установите агент Linux для Azure, Cloud-init и другие необходимые служебные программы, выполнив следующую команду:

    ```console
    # sudo yum install -y WALinuxAgent cloud-init cloud-utils-growpart gdisk hyperv-daemons

    # sudo systemctl enable waagent.service
    # sudo systemctl enable cloud-init.service
    ```

1. Настройте Cloud-init для выполнения подготовки:

    1. Настройка waagent для Cloud-init:

    ```console
    sed -i 's/Provisioning.Agent=auto/Provisioning.Agent=cloud-init/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf
    ```
    > [!NOTE]
    > Если вы переносите определенную виртуальную машину и не хотите создавать обобщенные образы, установите `Provisioning.Agent=disabled` в `/etc/waagent.conf` конфигурации.
    
    1. Настройка подключений:

    ```console
    echo "Adding mounts and disk_setup to init stage"
    sed -i '/ - mounts/d' /etc/cloud/cloud.cfg
    sed -i '/ - disk_setup/d' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - mounts' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - disk_setup' /etc/cloud/cloud.cfg
    ```
    
    1. Настройка источника данных Azure:

    ```console
    echo "Allow only Azure datasource, disable fetching network setting via IMDS"
    cat > /etc/cloud/cloud.cfg.d/91-azure_datasource.cfg <<EOF
    datasource_list: [ Azure ]
    datasource:
        Azure:
            apply_network_config: False
    EOF
    ```

    1. Если настроено, удалите существующие файл подкачки:

    ```console
    if [[ -f /mnt/resource/swapfile ]]; then
    echo "Removing swapfile" #RHEL uses a swapfile by defaul
    swapoff /mnt/resource/swapfile
    rm /mnt/resource/swapfile -f
    fi
    ```
    1. Настройка ведения журнала Cloud-init:
    ```console
    echo "Add console log file"
    cat >> /etc/cloud/cloud.cfg.d/05_logging.cfg <<EOF

    # This tells cloud-init to redirect its stdout and stderr to
    # 'tee -a /var/log/cloud-init-output.log' so the user can see output
    # there without needing to look on the console.
    output: {all: '| tee -a /var/log/cloud-init-output.log'}
    EOF

    ```

1. Конфигурация переключения не создает пространство подкачки на диске операционной системы.

    Ранее агент Linux для Azure использовал автоматическую настройку области подкачки с помощью локального диска ресурсов, подключенного к виртуальной машине после подготовки виртуальной машины в Azure. Однако это теперь обрабатывается с помощью Cloud-init, поэтому **не следует** использовать агент Linux для форматирования диска ресурсов. Создайте файл подкачки, измените следующие параметры соответствующим образом `/etc/waagent.conf` :

    ```console
    ResourceDisk.Format=n
    ResourceDisk.EnableSwap=n
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
1. Чтобы отменить регистрацию подписки, выполните следующую команду:

    ```console
    # sudo subscription-manager unregister
    ```

1. Отзыв

    Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

    ```console
    # sudo waagent -force -deprovision

    # export HISTSIZE=0

    # logout
    ```
    > [!CAUTION]
    > Если вы переносите определенную виртуальную машину и не хотите создавать обобщенные образы, пропустите этап отмены наполнения. Выполнение команды `waagent -force -deprovision` сделает исходный компьютер непригодным для использования. Этот шаг предназначен только для создания обобщенного образа.


1. Щелкните **действие**  >  **Завершение работы** в диспетчере Hyper-V. Виртуальный жесткий диск Linux готов к передаче в Azure.


## <a name="kvm"></a>KVM

В этом разделе показано, как использовать KVM для подготовки [RHEL 6](#rhel-6-using-kvm) или [RHEL 7](#rhel-7-using-kvm) дистрибутив к передаче в Azure. 

### <a name="rhel-6-using-kvm"></a>RHEL 6 Использование KVM

1. Скачайте образ KVM RHEL 6 с веб-сайта Red Hat.

1. Задайте пароль пользователя root.

    Создайте зашифрованный пароль и скопируйте результат выполнения команды:

    ```console
    # openssl passwd -1 changeme
    ```

    Установите корневой пароль с помощью Guestfish:

    ```console
    # guestfish --rw -a <image-name>
    > <fs> run
    > <fs> list-filesystems
    > <fs> mount /dev/sda1 /
    > <fs> vi /etc/shadow
    > <fs> exit
    ```

   Во втором поле привилегированного пользователя замените "!!" зашифрованным паролем.

1. Создайте виртуальную машину в KVM из образа qcow2. Укажите тип диска **qcow2** и задайте для модели устройства интерфейса виртуальной сети значение **virtio**. Затем запустите виртуальную машину и войдите в систему как привилегированный пользователь.

1. Создайте или измените файл `/etc/sysconfig/network`, добавив следующий текст:

    ```config
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Создайте или измените файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, добавив следующий текст:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

1. Переместите (или удалите) правила udev, чтобы не создавать статические правила для интерфейса Ethernet. Эти правила приводят к появлению проблем при клонировании виртуальной машины в Azure или Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

1. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```console
    # chkconfig network on
    ```

1. Зарегистрируйте подписку Red Hat, чтобы активировать установку пакетов из репозитория RHEL, запустив следующую команду:

    ```console
    # subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/boot/grub/menu.lst` в текстовом редакторе и укажите для ядра по умолчанию следующие параметры:

    ```config-grub
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

    Это также гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки.
    
    В дополнение к вышесказанному мы рекомендуем удалить следующие параметры:

    ```config-grub
    rhgb quiet crashkernel=auto
    ```

    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт. При желании можно оставить параметр `crashkernel`. Обратите внимание, что этот параметр сокращает объем доступной памяти на виртуальной машине на 128 MБ и более. Это может оказаться проблемой для виртуальных машин небольшого размера.


1. Добавьте модули Hyper-V в initramfs.  

    Измените файл `/etc/dracut.conf`, добавив следующее содержимое:

    ```config-conf
    add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
    ```

    Повторно создайте initramfs:

    ```config-conf
    # dracut -f -v
    ```

1. Удаление cloud-init:

    ```console
    # yum remove cloud-init
    ```

1. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки:

    ```console
    # chkconfig sshd on
    ```

    Измените файл /etc/ssh/sshd_config, включив в него следующие строки:

    ```config
    PasswordAuthentication yes
    ClientAliveInterval 180
    ```

1. Пакет WALinuxAgent `WALinuxAgent-<version>` был отправлен в репозиторий дополнений Red Hat. Включите репозиторий дополнений, выполнив следующую команду:

    ```console
    # subscription-manager repos --enable=rhel-6-server-extras-rpms
    ```

1. Установите агент Linux для Azure, выполнив следующую команду:

    ```console
    # yum install WALinuxAgent

    # chkconfig waagent on
    ```

1. Агент Linux для Azure может автоматически настраивать размер области подкачки с использованием локального диска ресурсов, подключенного к виртуальной машине после ее подготовки в Azure. Обратите внимание, что локальный диск ресурсов является временным и может быть очищен, если виртуальная машина будет отменена. После установки агента Linux для Azure на предыдущем шаге измените следующие параметры в **/etc/waagent.conf** соответствующим образом:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

1. Отмените регистрацию подписки (при необходимости), выполнив следующую команду:

    ```console-conf
    # subscription-manager unregister
    ```

1. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

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

1. Остановите работу виртуальной машины в KVM.

1. Конвертируйте образ qcow2 в формат VHD.

    > [!NOTE]
    > В команде qemu-img версии 2.2.1 и более поздних версиях есть ошибка, которая приводит к созданию неверного формата VHD. Эта проблема устранена в QEMU версии 2.6. Рекомендуется использовать qemu-img версии 2.2.0 или более ранних версий. Либо выполните обновление до версии 2.6 или выше. Справочные материалы: https://bugs.launchpad.net/qemu/+bug/1490611.
    >

    Для начала конвертируйте образ в формат RAW:

    ```console
    # qemu-img convert -f qcow2 -O raw rhel-6.9.qcow2 rhel-6.9.raw
    ```

    Убедитесь, что размер образа в формате RAW соответствует 1 МБ. Если это не так, округлите размер до 1 МБ:

    ```console
    # MB=$((1024*1024))
    # size=$(qemu-img info -f raw --output json "rhel-6.9.raw" | \
      gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

    # rounded_size=$((($size/$MB + 1)*$MB))
    # qemu-img resize rhel-6.9.raw $rounded_size
    ```

    Конвертируйте диск в формате RAW в виртуальный жесткий диск фиксированного размера:

    ```console
    # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-6.9.raw rhel-6.9.vhd
    ```

    Кроме того, QEMU версии **2.6 +** включает `force_size` параметр:

    ```console
    # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-6.9.raw rhel-6.9.vhd
    ```

        
### <a name="rhel-7-using-kvm"></a>RHEL 7 с помощью KVM

1. Скачайте образ KVM RHEL 7 с веб-сайта Red Hat. В примере используется RHEL 7.

1. Задайте пароль пользователя root.

    Создайте зашифрованный пароль и скопируйте результат выполнения команды:

    ```console
    # openssl passwd -1 changeme
    ```

    Установите корневой пароль с помощью Guestfish:

    ```console
    # guestfish --rw -a <image-name>
    > <fs> run
    > <fs> list-filesystems
    > <fs> mount /dev/sda1 /
    > <fs> vi /etc/shadow
    > <fs> exit
    ```

   Во втором поле привилегированного пользователя замените "!!" зашифрованным паролем.

1. Создайте виртуальную машину в KVM из образа qcow2. Укажите тип диска **qcow2** и задайте для модели устройства интерфейса виртуальной сети значение **virtio**. Затем запустите виртуальную машину и войдите в систему как привилегированный пользователь.

1. Создайте или измените файл `/etc/sysconfig/network`, добавив следующий текст:

    ```config
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Создайте или измените файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, добавив следующий текст:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    PERSISTENT_DHCLIENT=yes
    NM_CONTROLLED=yes
    ```

1. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```console
    # sudo systemctl enable network
    ```

1. Зарегистрируйте подписку Red Hat, чтобы активировать установку пакетов из репозитория RHEL, запустив следующую команду:

    ```console
    # subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/etc/default/grub` в текстовом редакторе и измените параметр `GRUB_CMDLINE_LINUX`. Пример:

    ```config-grub
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   Эта команда также гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки. Также будут отключены новые соглашения об именовании RHEL 7 для сетевых карт. В дополнение к вышесказанному мы рекомендуем удалить следующие параметры:

    ```config-grub
    rhgb quiet crashkernel=auto
    ```

    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт. При желании можно оставить параметр `crashkernel`. Обратите внимание, что этот параметр сокращает объем доступной памяти на виртуальной машине на 128 MБ и более. Это может оказаться проблемой для виртуальных машин небольшого размера.

1. После внесения изменений в файл `/etc/default/grub`выполните следующую команду, чтобы повторно создать конфигурацию grub:

    ```console
    # grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. Добавьте модули Hyper-V в initramfs.

    Измените файл `/etc/dracut.conf` , добавив в него следующее содержимое:

    ```config-conf
    add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
    ```

    Повторно создайте initramfs:

    ```config-conf
    # dracut -f -v
    ```

1. Удаление cloud-init:

    ```console
    # yum remove cloud-init
    ```

1. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки:

    ```console
    # systemctl enable sshd
    ```

    Измените файл /etc/ssh/sshd_config, включив в него следующие строки:

    ```config
    PasswordAuthentication yes
    ClientAliveInterval 180
    ```

1. Пакет WALinuxAgent `WALinuxAgent-<version>` был отправлен в репозиторий дополнений Red Hat. Включите репозиторий дополнений, выполнив следующую команду:

    ```config
    # subscription-manager repos --enable=rhel-7-server-extras-rpms
    ```

1. Установите агент Linux для Azure, выполнив следующую команду:

    ```console
    # yum install WALinuxAgent
    ```

    Включите службу waagent:

    ```console
    # systemctl enable waagent.service
    ```

1. Установка Cloud-init выполните действия, описанные в подразделе "Подготовка виртуальной машины RHEL 7 из диспетчера Hyper-V", шаг 12, "Установка Cloud-init для обработки подготовки".

1. Переключить конфигурацию 

    Не создавайте пространство подкачки на диске ОС.
    Выполните действия, описанные в подразделе "Подготовка виртуальной машины RHEL 7 из диспетчера Hyper-V", шаг 13, "переключить конфигурацию"


1. Отмените регистрацию подписки (при необходимости), выполнив следующую команду:

    ```console
    # subscription-manager unregister
    ```


1. Отзыв

    Выполните действия, описанные в подразделе "Подготовка виртуальной машины RHEL 7 из диспетчера Hyper-V", шаг 15, "отменить подготовку".

1. Остановите работу виртуальной машины в KVM.

1. Конвертируйте образ qcow2 в формат VHD.

    > [!NOTE]
    > В команде qemu-img версии 2.2.1 и более поздних версиях есть ошибка, которая приводит к созданию неверного формата VHD. Эта проблема устранена в QEMU версии 2.6. Рекомендуется использовать qemu-img версии 2.2.0 или более ранних версий. Либо выполните обновление до версии 2.6 или выше. Справочные материалы: https://bugs.launchpad.net/qemu/+bug/1490611.
    >

    Для начала конвертируйте образ в формат RAW:

    ```console
    # qemu-img convert -f qcow2 -O raw rhel-7.4.qcow2 rhel-7.4.raw
    ```

    Убедитесь, что размер образа в формате RAW соответствует 1 МБ. Если это не так, округлите размер до 1 МБ:

    ```console
    # MB=$((1024*1024))
    # size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
      gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

    # rounded_size=$((($size/$MB + 1)*$MB))
    # qemu-img resize rhel-7.4.raw $rounded_size
    ```

    Конвертируйте диск в формате RAW в виртуальный жесткий диск фиксированного размера:

    ```console
    # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

    Кроме того, QEMU версии **2.6 +** включает `force_size` параметр:

    ```console
    # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

## <a name="vmware"></a>VMware

В этом разделе показано, как подготовить [RHEL 6](#rhel-6-using-vmware) или [RHEL 7](#rhel-6-using-vmware)  дистрибутив из VMware.

### <a name="prerequisites"></a>Предварительные условия
В этом разделе предполагается, что вы уже установили виртуальную машину RHEL в VMWare. Дополнительные сведения об установке операционной системы на виртуальной машине VMware см. [здесь](https://partnerweb.vmware.com/GOSIG/home.html).

* При установке операционной системы Linux мы рекомендуем использовать стандартные разделы, а не LVM (который зачастую используется по умолчанию при установке). Это позволит избежать конфликта имен LVM при клонировании виртуальных машин, особенно если диск с OC может быть подключен к другой идентичной виртуальной машине в целях устранения неполадок. При желании на дисках с данными можно использовать LVM или RAID.
* Не настраивайте раздел swap на диске операционной системы. Вы можете настроить агент Linux для создания файла подкачки на временном диске с ресурсами. Дополнительные сведения приведены ниже.
* При создании виртуального жесткого диска выберите параметр **Store virtual disk as a single file**(Сохранять виртуальный диск как один файл).

### <a name="rhel-6-using-vmware"></a>RHEL 6 с использованием VMware
1. В RHEL 6 NetworkManager может мешать работе агента Linux для Azure. Установите пакет, выполнив следующую команду:

    ```console
    # sudo rpm -e --nodeps NetworkManager
    ```

1. Создайте файл с именем **network** в каталоге /etc/sysconfig/. Файл должен содержать следующий текст:

    ```console
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Создайте или измените файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, добавив следующий текст:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

1. Переместите (или удалите) правила udev, чтобы не создавать статические правила для интерфейса Ethernet. Эти правила приводят к появлению проблем при клонировании виртуальной машины в Azure или Hyper-V:

    ```console
    # sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules

    # sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

1. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```console
    # sudo chkconfig network on
    ```

1. Зарегистрируйте подписку Red Hat, чтобы активировать установку пакетов из репозитория RHEL, запустив следующую команду:

    ```console
    # sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Пакет WALinuxAgent `WALinuxAgent-<version>` был отправлен в репозиторий дополнений Red Hat. Включите репозиторий дополнений, выполнив следующую команду:

    ```console
    # subscription-manager repos --enable=rhel-6-server-extras-rpms
    ```

1. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/etc/default/grub` в текстовом редакторе и измените параметр `GRUB_CMDLINE_LINUX`. Пример:

    ```config-grub
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0"
    ```

   Это также гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки. В дополнение к вышесказанному мы рекомендуем удалить следующие параметры:

    ```config-grub
    rhgb quiet crashkernel=auto
    ```
   
    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт. При желании можно оставить параметр `crashkernel`. Обратите внимание, что этот параметр сокращает объем доступной памяти на виртуальной машине на 128 MБ и более. Это может оказаться проблемой для виртуальных машин небольшого размера.

1. Добавьте модули Hyper-V в initramfs.

    Измените файл `/etc/dracut.conf`, добавив следующее содержимое:

    ```config-conf
    add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
    ```

    Повторно создайте initramfs:

    ```config-cong
    # dracut -f -v
    ```

1. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки (что обычно задано по умолчанию). Измените файл `/etc/ssh/sshd_config` , добавив в него следующую строку:

    ```config
    ClientAliveInterval 180
    ```

1. Установите агент Linux для Azure, выполнив следующую команду:

    ```console
    # sudo yum install WALinuxAgent

    # sudo chkconfig waagent on
    ```

1. Не создавайте пространство подкачки на диске ОС.

    Агент Linux для Azure может автоматически настраивать размер области подкачки с использованием локального диска ресурсов, подключенного к виртуальной машине после ее подготовки в Azure. Обратите внимание, что локальный диск ресурсов является временным и может быть очищен, если виртуальная машина будет отменена. После установки агента Linux для Azure (см. предыдущий шаг) измените соответствующим образом следующие параметры в файле `/etc/waagent.conf`:

    ```config-conf
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048    ## NOTE: set this to whatever you need it to be.
    ```

1. Отмените регистрацию подписки (при необходимости), выполнив следующую команду:

    ```console
    # sudo subscription-manager unregister
    ```

1. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

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

1. Завершите работу виртуальной машины и конвертируйте файл VMDK в формат VHD.

    > [!NOTE]
    > В команде qemu-img версии 2.2.1 и более поздних версиях есть ошибка, которая приводит к созданию неверного формата VHD. Эта проблема устранена в QEMU версии 2.6. Рекомендуется использовать qemu-img версии 2.2.0 или более ранних версий. Либо выполните обновление до версии 2.6 или выше. Справочные материалы: https://bugs.launchpad.net/qemu/+bug/1490611.
    >

    Для начала конвертируйте образ в формат RAW:

    ```console
    # qemu-img convert -f vmdk -O raw rhel-6.9.vmdk rhel-6.9.raw
    ```

    Убедитесь, что размер образа в формате RAW соответствует 1 МБ. Если это не так, округлите размер до 1 МБ:

    ```console
    # MB=$((1024*1024))
    # size=$(qemu-img info -f raw --output json "rhel-6.9.raw" | \
      gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

    # rounded_size=$((($size/$MB + 1)*$MB))
    # qemu-img resize rhel-6.9.raw $rounded_size
    ```

    Конвертируйте диск в формате RAW в виртуальный жесткий диск фиксированного размера:

    ```console
    # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-6.9.raw rhel-6.9.vhd
    ```

    Кроме того, QEMU версии **2.6 +** включает `force_size` параметр:

    ```console
    # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-6.9.raw rhel-6.9.vhd
    ```

### <a name="rhel-7-using-vmware"></a>RHEL 7 с использованием VMware
1. Создайте или измените файл `/etc/sysconfig/network`, добавив следующий текст:

    ```config
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

1. Создайте или измените файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, добавив следующий текст:

    ```config
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    PERSISTENT_DHCLIENT=yes
    NM_CONTROLLED=yes
    ```

1. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```console
    # sudo systemctl enable network
    ```

1. Зарегистрируйте подписку Red Hat, чтобы активировать установку пакетов из репозитория RHEL, запустив следующую команду:

    ```console
    # sudo subscription-manager register --auto-attach --username=XXX --password=XXX
    ```

1. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/etc/default/grub` в текстовом редакторе и измените параметр `GRUB_CMDLINE_LINUX`. Пример:

    ```config-grub
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   Это также гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки. Также будут отключены новые соглашения об именовании RHEL 7 для сетевых карт. В дополнение к вышесказанному мы рекомендуем удалить следующие параметры:

    ```config-grub
    rhgb quiet crashkernel=auto
    ```

    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт. При желании можно оставить параметр `crashkernel`. Обратите внимание, что этот параметр сокращает объем доступной памяти на виртуальной машине на 128 MБ и более. Это может оказаться проблемой для виртуальных машин небольшого размера.

1. После внесения изменений в файл `/etc/default/grub`выполните следующую команду, чтобы повторно создать конфигурацию grub:

    ```console
    # sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1. Добавьте модули Hyper-V в initramfs.

    Измените файл `/etc/dracut.conf`, добавив в него следующее содержимое:

    ```config-conf
    add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
    ```

    Повторно создайте initramfs:

    ```console
    # dracut -f -v
    ```

1. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки. Обычно этот параметр задан по умолчанию. Измените файл `/etc/ssh/sshd_config` , добавив в него следующую строку:

    ```config
    ClientAliveInterval 180
    ```

1. Пакет WALinuxAgent `WALinuxAgent-<version>` был отправлен в репозиторий дополнений Red Hat. Включите репозиторий дополнений, выполнив следующую команду:

    ```console
    # subscription-manager repos --enable=rhel-7-server-extras-rpms
    ```

1. Установите агент Linux для Azure, выполнив следующую команду:

    ```console
    # sudo yum install WALinuxAgent

    # sudo systemctl enable waagent.service
    ```

1. Установка Cloud-init

    Выполните действия, описанные в подразделе "Подготовка виртуальной машины RHEL 7 из диспетчера Hyper-V", шаг 12, "Установка Cloud-init для обработки подготовки".

1. Переключить конфигурацию

    Не создавайте пространство подкачки на диске ОС.
    Выполните действия, описанные в подразделе "Подготовка виртуальной машины RHEL 7 из диспетчера Hyper-V", шаг 13, "переключить конфигурацию"

1. Чтобы отменить регистрацию подписки, выполните следующую команду:

    ```console
    # sudo subscription-manager unregister
    ```

1. Отзыв

    Выполните действия, описанные в подразделе "Подготовка виртуальной машины RHEL 7 из диспетчера Hyper-V", шаг 15, "отменить подготовку".


1. Завершите работу виртуальной машины и конвертируйте файл VMDK в формат VHD.

    > [!NOTE]
    > В команде qemu-img версии 2.2.1 и более поздних версиях есть ошибка, которая приводит к созданию неверного формата VHD. Эта проблема устранена в QEMU версии 2.6. Рекомендуется использовать qemu-img версии 2.2.0 или более ранних версий. Либо выполните обновление до версии 2.6 или выше. Справочные материалы: https://bugs.launchpad.net/qemu/+bug/1490611.
    >

    Для начала конвертируйте образ в формат RAW:

    ```console
    # qemu-img convert -f vmdk -O raw rhel-7.4.vmdk rhel-7.4.raw
    ```

    Убедитесь, что размер образа в формате RAW соответствует 1 МБ. Если это не так, округлите размер до 1 МБ:

    ```console
    # MB=$((1024*1024))
    # size=$(qemu-img info -f raw --output json "rhel-7.4.raw" | \
      gawk 'match($0, /"virtual-size": ([0-9]+),/, val) {print val[1]}')

    # rounded_size=$((($size/$MB + 1)*$MB))
    # qemu-img resize rhel-7.4.raw $rounded_size
    ```

    Конвертируйте диск в формате RAW в виртуальный жесткий диск фиксированного размера:

    ```console
    # qemu-img convert -f raw -o subformat=fixed -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

    Кроме того, QEMU версии **2.6 +** включает `force_size` параметр:

    ```console
    # qemu-img convert -f raw -o subformat=fixed,force_size -O vpc rhel-7.4.raw rhel-7.4.vhd
    ```

## <a name="kickstart-file"></a>Файл Kickstart

В этом разделе показано, как подготовить RHEL 7 дистрибутив из ISO-образа с помощью файла Kickstart.

### <a name="rhel-7-from-a-kickstart-file"></a>RHEL 7 из файла Kickstart

1.  Создайте файл kickstart, который будет включать содержимое ниже, и сохраните его. Дополнительные сведения об установке kickstart см. [здесь](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/chap-kickstart-installations.html).

    ```text
    # Kickstart for provisioning a RHEL 7 Azure VM

    # System authorization information
      auth --enableshadow --passalgo=sha512

    # Use graphical install
    text

    # Do not run the Setup Agent on first boot
    firstboot --disable

    # Keyboard layouts
    keyboard --vckeymap=us --xlayouts='us'

    # System language
    lang en_US.UTF-8

    # Network information
    network  --bootproto=dhcp

    # Root password
    rootpw --plaintext "to_be_disabled"

    # System services
    services --enabled="sshd,waagent,NetworkManager"

    # System timezone
    timezone Etc/UTC --isUtc --ntpservers 0.rhel.pool.ntp.org,1.rhel.pool.ntp.org,2.rhel.pool.ntp.org,3.rhel.pool.ntp.org

    # Partition clearing information
    clearpart --all --initlabel

    # Clear the MBR
    zerombr

    # Disk partitioning information
    part /boot --fstype="xfs" --size=500
    part / --fstyp="xfs" --size=1 --grow --asprimary

    # System bootloader configuration
    bootloader --location=mbr

    # Firewall configuration
    firewall --disabled

    # Enable SELinux
    selinux --enforcing

    # Don't configure X
    skipx

    # Power down the machine after install
    poweroff

    %packages
    @base
    @console-internet
    chrony
    sudo
    parted
    -dracut-config-rescue

    %end

    %post --log=/var/log/anaconda/post-install.log

    #!/bin/bash

    # Register Red Hat Subscription
    subscription-manager register --username=XXX --password=XXX --auto-attach --force

    # Install latest repo update
    yum update -y

    # Enable extras repo
    subscription-manager repos --enable=rhel-7-server-extras-rpms

    # Install WALinuxAgent
    yum install -y WALinuxAgent

    # Unregister Red Hat subscription
    subscription-manager unregister

    # Enable waaagent at boot-up
    systemctl enable waagent

    # Install cloud-init
    yum install -y cloud-init cloud-utils-growpart gdisk hyperv-daemons

    # Configure waagent for cloud-init
    sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
    sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf

    echo "Adding mounts and disk_setup to init stage"
    sed -i '/ - mounts/d' /etc/cloud/cloud.cfg
    sed -i '/ - disk_setup/d' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - mounts' /etc/cloud/cloud.cfg
    sed -i '/cloud_init_modules/a\\ - disk_setup' /etc/cloud/cloud.cfg

    # Disable the root account
    usermod root -p '!!'

    # Disabke swap in WALinuxAgent
    ResourceDisk.Format=n
    ResourceDisk.EnableSwap=n

    # Configure swap using cloud-init
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

    # Set the cmdline
    sed -i 's/^\(GRUB_CMDLINE_LINUX\)=".*"$/\1="console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300"/g' /etc/default/grub

    # Enable SSH keepalive
    sed -i 's/^#\(ClientAliveInterval\).*$/\1 180/g' /etc/ssh/sshd_config

    # Build the grub cfg
    grub2-mkconfig -o /boot/grub2/grub.cfg

    # Configure network
    cat << EOF > /etc/sysconfig/network-scripts/ifcfg-eth0
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    PERSISTENT_DHCLIENT=yes
    NM_CONTROLLED=yes
    EOF

    # Deprovision and prepare for Azure if you are creating a generalized image
    sudo cloud-init clean --logs --seed
    sudo rm -rf /var/lib/cloud/
    sudo rm -rf /var/lib/waagent/
    sudo rm -f /var/log/waagent.log

    sudo waagent -force -deprovision+user
    rm -f ~/.bash_history
    export HISTSIZE=0

    %end
    ```

1. Поместите файл kickstart в место, доступное для системы установки.

1. Создайте новую виртуальную машину в диспетчере Hyper-V. На странице **Подключить виртуальный жесткий диск** выберите параметр **Подключить виртуальный жесткий диск позднее** и выполните инструкции в мастере создания виртуальной машины.

1. Откройте параметры виртуальной машины:

    a.  Подключите новый виртуальный жесткий диск к виртуальной машине. Выберите параметры **VHD Format** (Формат VHD) и **Фиксированный размер**.

    b.  Подключите установочный ISO-образ к DVD-дисководу.

    c.  В BIOS выберите загрузку с компакт-диска.

1. Запустите виртуальную машину. Когда отобразится руководство по установке, нажмите клавишу **TAB** , чтобы настроить параметры загрузки.

1. Введите `inst.ks=<the location of the kickstart file>` в конце параметров загрузки и нажмите клавишу **ВВОД**.

1. Дождитесь завершения процесса установки. По окончании процесса виртуальная машина завершит работу автоматически. Виртуальный жесткий диск Linux готов к передаче в Azure.

## <a name="known-issues"></a>Известные проблемы
### <a name="the-hyper-v-driver-could-not-be-included-in-the-initial-ram-disk-when-using-a-non-hyper-v-hypervisor"></a>Не удалось включить драйвер Hyper-V в начальный электронный диск при использовании низкоуровневой оболочки Hyper V

В некоторых случаях установщики Linux могут не включать драйверы для Hyper-V в начальный электронный диск (initrd или initramfs), если только не обнаружат, что он работает в среде Hyper-V.

Если вы используете другую систему виртуализации (то есть VirtualBox, Xen и т. д.) для подготовки образа Linux, может потребоваться перестроить initrd, чтобы убедиться, что на начальном ЭЛЕКТРОНном диске доступны по крайней мере hv_vmbus и hv_storvsc модули ядра. Это известная проблема по крайней мере в системах на основе предшествующего дистрибутива Red Hat.

Чтобы устранить эту проблему, необходимо добавить модули Hyper-V в initramfs и выполнить повторную сборку:

Измените файл `/etc/dracut.conf`, добавив следующее содержимое:

```config-conf
add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
```

Повторно создайте initramfs:

```console
# dracut -f -v
```

Дополнительные сведения см. в описании [повторного создания initramfs](https://access.redhat.com/solutions/1958).

## <a name="next-steps"></a>Дальнейшие действия
* Теперь виртуальный жесткий диск Red Hat Enterprise Linux можно использовать для создания новых виртуальных машин Azure. Если вы отправляете VHD-файл в Azure впервые, см. раздел [Вариант 1. Передача VHD](upload-vhd.md#option-1-upload-a-vhd).
* Чтобы получить дополнительные сведения о низкоуровневых оболочках, сертифицированных для запуска Red Hat Enterprise Linux, посетите [веб-сайт Red Hat](https://access.redhat.com/certified-hypervisors).
* Чтобы узнать больше об использовании готовых образов RHEL BYOS, перейдите на страницу документации по [BYOS](../workloads/redhat/byos.md).
