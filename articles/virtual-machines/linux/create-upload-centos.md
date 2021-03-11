---
title: Создание и передача виртуального жесткого диска Linux на основе CentOS
description: Узнайте, как создать и передать виртуальный жесткий диск (VHD-файл) Azure, содержащий операционную систему Linux на основе CentOS
author: danielsollondon
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 12/01/2020
ms.author: danis
ms.openlocfilehash: 4745e631bd92675f8dd1ef0d390baa88f7666552
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102554672"
---
# <a name="prepare-a-centos-based-virtual-machine-for-azure"></a>Подготовка виртуальной машины на основе CentOS для Azure

Узнайте, как создать и передать виртуальный жесткий диск (VHD-файл) Azure, содержащий операционную систему Linux на основе CentOS

* [Подготовка виртуальной машины CentOS 6.x для Azure](#centos-6x)
* [Подготовка виртуальной машины CentOS 7.0+ для Azure](#centos-70)


## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что вы уже установили операционную систему CentOS Linux (или аналогичную производную) на виртуальный жесткий диск. Существует несколько средств для создания VHD-файлов, например решение для виртуализации, такое как Hyper-V. Инструкции см. в разделе [Установка роли Hyper-V и настройка виртуальной машины](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

**Замечания по установке CentOS**

* Дополнительные сведения о подготовке Linux для Azure см. в разделе [Общие замечания по установке Linux](create-upload-generic.md#general-linux-installation-notes).
* Формат VHDX не поддерживается в Azure, поддерживается только **фиксированный VHD**.  Можно преобразовать диск в формат VHD с помощью диспетчера Hyper-V или командлета convert-vhd. Если вы используете VirtualBox, при создании диска по умолчанию нужно выбрать **фиксированный размер** вместо динамически выделяемого.
* При установке системы Linux *рекомендуется* использовать стандартные секции, а не LVM (как правило, по умолчанию для многих установок). Это позволит избежать конфликта имен LVM c клонированными виртуальными машинами, особенно если диск с OC может быть подключен к другой идентичной виртуальной машине в целях устранения неполадок. Для дисков данных можно использовать [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) или [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid).
* **Требуется поддержка ядра для монтирования файловых систем UDF.** При первой загрузке в Azure конфигурация подготовки передается в виртуальную машину Linux через UDF-носитель, подключенный к гостевой машине. Агент Azure Linux должен иметь возможность подключать файловую систему UDF для считывания конфигурации и подготовки виртуальной машины.
* Версии ядра Linux ниже 2.6.37 не поддерживают NUMA в Hyper-V с виртуальными машинами большего размера. Эта проблема влияет в основном на дистрибутивы более ранних версий, в которых используется исходное ядро Red Hat 2.6.32, и была исправлена в RHEL 6.6 (kernel-2.6.32-504). В системах под управлением модифицированных ядер старше версии 2.6.37 или ядер RHEL старше 2.6.32-504 в командной строке ядра необходимо задать параметр загрузки `numa=off` в файле grub.conf. Дополнительные сведения см. в статье Red Hat [KB 436883](https://access.redhat.com/solutions/436883).
* Не настраивайте раздел подкачки на диске с ОС. Дополнительные сведения описаны далее.
* Размер виртуальной памяти всех VHD в Azure должен быть округлен до 1 МБ. При конвертации диска в формате RAW в виртуальный жесткий диск убедитесь, что размер диска RAW в несколько раз превышает 1 МБ. См. дополнительные сведения в [примечаниях по установке Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="centos-6x"></a>CentOS 6.x

1. В диспетчере Hyper-V выберите виртуальную машину.

2. Щелкните **Подключение** , чтобы открыть окно консоли для виртуальной машины.

3. В CentOS 6 NetworkManager может мешать выполнению агента Linux для Azure. Установите пакет, выполнив следующую команду:

    ```bash
    sudo rpm -e --nodeps NetworkManager
    ```

4. Создайте или измените файл `/etc/sysconfig/network`, добавив следующий текст:

    ```console
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

5. Создайте или измените файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, добавив следующий текст:

    ```console
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    ```

6. Переместите или удалите правила udev, чтобы не создавать статические правила для интерфейса Ethernet. Эти правила приводят к появлению проблем при клонировании виртуальной машины в Microsoft Azure или Hyper-V:

    ```bash
    sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    sudo rm -f /etc/udev/rules.d/70-persistent-net.rules
    ```

7. Убедитесь, что сетевая служба запускается во время загрузки, выполнив следующую команду:

    ```bash
    sudo chkconfig network on
    ```

8. Если вы хотите использовать зеркала OpenLogic, размещенные в центрах обработки данных Azure, замените файл `/etc/yum.repos.d/CentOS-Base.repo` следующими репозиториями.  При этом также будет добавлен репозиторий **[openlogic]**, который включает дополнительные пакеты, например для агента Linux для Azure:

   ```console
   [openlogic]
   name=CentOS-$releasever - openlogic packages for $basearch
   baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
   enabled=1
   gpgcheck=0

   [base]
   name=CentOS-$releasever - Base
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #released updates
   [updates]
   name=CentOS-$releasever - Updates
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #additional packages that may be useful
   [extras]
   name=CentOS-$releasever - Extras
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #additional packages that extend functionality of existing packages
   [centosplus]
   name=CentOS-$releasever - Plus
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6

   #contrib - packages by Centos Users
   [contrib]
   name=CentOS-$releasever - Contrib
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/contrib/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
   ```

    > [!Note]
    > В этом руководстве предполагается, что вы используете по крайней мере репозиторий `[openlogic]`, который будет применяться при установке агента Linux для Azure.

9. Добавьте следующую строку в файл /etc/yum.conf:

    ```console
    http_caching=packages
    ```

10. Выполните следующую команду, чтобы очистить текущие метаданные yum и установить в системе последние обновления:

    ```bash
    yum clean all
    ```

    Если создается образ для более ранней версии CentOS, рекомендуется выполнить обновление всех пакетов до последней версии:

    ```bash
    sudo yum -y update
    ```

    После выполнения этой команды может потребоваться перезагрузка.

11. (Необязательно) Установите драйверы для служб интеграции Linux.

    > [!IMPORTANT]
    > Этот шаг является **обязательным** для CentOS 6.3 и более ранних версий и необязательным для более поздних.

    ```bash
    sudo rpm -e hypervkvpd  ## (may return error if not installed, that's OK)
    sudo yum install microsoft-hyper-v
    ```

    Или же следуйте инструкциям по установке на [странице скачивания служб интеграции Linux](https://www.microsoft.com/download/details.aspx?id=55106), чтобы установить RPM в образе.

12. Установите агент Linux для Azure и зависимости. Запуск и включение службы waagent:

    ```bash
    sudo yum install python-pyasn1 WALinuxAgent
    sudo service waagent start
    sudo chkconfig waagent on
    ```


    Установка пакета WALinuxAgent приведет к удалению пакетов NetworkManager и NetworkManager-gnome, если они еще не удалены, как описано в шаге 3.

13. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/boot/grub/menu.lst` в текстовом редакторе и укажите для ядра по умолчанию следующие параметры:

    ```console
    console=ttyS0 earlyprintk=ttyS0 rootdelay=300
    ```

    Это также гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки.

    Помимо вышесказанного, рекомендуется *удалить* следующие параметры:

    ```console
    rhgb quiet crashkernel=auto
    ```

    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт.  При желании можно настроить параметр `crashkernel` , однако учитывайте, что он сокращает объем доступной памяти в виртуальной машине на 128 МБ или более, что может оказаться проблемой в виртуальных машинах небольшого размера.

    > [!Important]
    > Для CentOS 6.5 и более ранних версий необходимо также указать параметр ядра `numa=off`. См. посвященный Red Hat [раздел 436883](https://access.redhat.com/solutions/436883).

14. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки.  Обычно это сделано по умолчанию.

15. Не создавайте пространство подкачки на диске с ОС.

    Агент Linux для Azure может автоматически настраивать пространство подкачки с использованием диска на локальном ресурсе, подключенном к виртуальной машине после подготовки для работы в среде Azure. Следует отметить, что локальный диск ресурсов является *временным* диском и должен быть очищен при отмене подготовки виртуальной машины. После установки агента Linux для Azure (см. предыдущий шаг) соответствующим образом измените следующие параметры в файле `/etc/waagent.conf`.

    ```console
    ResourceDisk.Format=y
    ResourceDisk.Filesystem=ext4
    ResourceDisk.MountPoint=/mnt/resource
    ResourceDisk.EnableSwap=y
    ResourceDisk.SwapSizeMB=2048 ## NOTE: set this to whatever you need it to be.
    ```

16. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

    ```bash
    sudo waagent -force -deprovision+user
    export HISTSIZE=0
    logout
    ```

17. В диспетчере Hyper-V выберите **Действие -> Завершение работы**. Виртуальный жесткий диск Linux готов к передаче в Azure.



## <a name="centos-70"></a>CentOS 7.0+

**Изменения в CentOS 7 (и аналогичных производных версиях)**

Подготовка виртуальной машины CentOS 7 для Azure в значительной степени аналогична версии CentOS 6, однако есть несколько важных отличий:

* Пакет NetworkManager больше не конфликтует с агентом Azure Linux. Этот пакет устанавливается по умолчанию, и мы рекомендуем не удалять его.
* В качестве загрузчика по умолчанию теперь используется GRUB2, поэтому изменилась процедура правки параметров ядра (см. ниже).
* XFS теперь является файловой системой по умолчанию. При желании можно продолжать использование файловой системы ext4.

**Этапы настройки**

1. В диспетчере Hyper-V выберите виртуальную машину.

2. Щелкните **Подключение** , чтобы открыть окно консоли для виртуальной машины.

3. Создайте или измените файл `/etc/sysconfig/network`, добавив следующий текст:

    ```console
    NETWORKING=yes
    HOSTNAME=localhost.localdomain
    ```

4. Создайте или измените файл `/etc/sysconfig/network-scripts/ifcfg-eth0`, добавив следующий текст:

    ```console
    DEVICE=eth0
    ONBOOT=yes
    BOOTPROTO=dhcp
    TYPE=Ethernet
    USERCTL=no
    PEERDNS=yes
    IPV6INIT=no
    NM_CONTROLLED=no
    ```

5. Переместите или удалите правила udev, чтобы не создавать статические правила для интерфейса Ethernet. Эти правила приводят к появлению проблем при клонировании виртуальной машины в Microsoft Azure или Hyper-V:

    ```bash
    sudo ln -s /dev/null /etc/udev/rules.d/75-persistent-net-generator.rules
    ```

6. Если вы хотите использовать зеркала OpenLogic, размещенные в центрах обработки данных Azure, замените файл `/etc/yum.repos.d/CentOS-Base.repo` следующими репозиториями.  При этом также будет добавлен репозиторий **[openlogic]** , включающий пакеты для агента Linux для Azure:

   ```console
   [openlogic]
   name=CentOS-$releasever - openlogic packages for $basearch
   baseurl=http://olcentgbl.trafficmanager.net/openlogic/$releasever/openlogic/$basearch/
   enabled=1
   gpgcheck=0
    
   [base]
   name=CentOS-$releasever - Base
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/os/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #released updates
   [updates]
   name=CentOS-$releasever - Updates
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/updates/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #additional packages that may be useful
   [extras]
   name=CentOS-$releasever - Extras
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/extras/$basearch/
   gpgcheck=1
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    
   #additional packages that extend functionality of existing packages
   [centosplus]
   name=CentOS-$releasever - Plus
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
   baseurl=http://olcentgbl.trafficmanager.net/centos/$releasever/centosplus/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
   ```
    
   > [!Note]
   > В этом руководстве предполагается, что вы используете по крайней мере репозиторий `[openlogic]`, который будет применяться при установке агента Linux для Azure.

7. Выполните следующую команду, чтобы очистить текущие метаданные yum и установить все обновления:

    ```bash
    sudo yum clean all
    ```

    Если создается образ для более ранней версии CentOS, рекомендуется выполнить обновление всех пакетов до последней версии:

    ```bash
    sudo yum -y update
    ```

    После выполнения этой команды может потребоваться перезагрузка.

8. Измените строку загрузки ядра в конфигурации grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/etc/default/grub` в текстовом редакторе и измените параметр `GRUB_CMDLINE_LINUX`, например:

    ```console
    GRUB_CMDLINE_LINUX="rootdelay=300 console=ttyS0 earlyprintk=ttyS0 net.ifnames=0"
    ```

   Это также гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки. Также будут отключены новые соглашения CentOS 7 об именовании для сетевых адаптеров. Помимо вышесказанного, рекомендуется *удалить* следующие параметры:

    ```console
    rhgb quiet crashkernel=auto
    ```

    Графическая и "тихая" загрузка бесполезны в облачной среде, в которой нам нужно, чтобы все журналы отправлялись на последовательный порт. При желании можно настроить параметр `crashkernel` , однако учитывайте, что он сокращает объем доступной памяти в виртуальной машине на 128 МБ или более, что может оказаться проблемой в виртуальных машинах небольшого размера.

9. После внесения изменений в файл `/etc/default/grub` , как указано выше, выполните следующую команду для повторного создания конфигурации grub:

    ```bash
    sudo grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

10. При создании образа из **VMware, VirtualBox или KVM** убедитесь, что драйверы Hyper-V включены в initramfs:

    Измените файл `/etc/dracut.conf`, добавив в него следующее содержимое:

    ```console
    add_drivers+=" hv_vmbus hv_netvsc hv_storvsc "
    ```

    Выполните сборку initramfs заново:

    ```bash
    sudo dracut -f -v
    ```

11. Установите агент Linux для Azure и зависимости для расширений виртуальной машины Azure.

    ```bash
    sudo yum install python-pyasn1 WALinuxAgent
    sudo systemctl enable waagent
    ```

12. Установка Cloud-init для решения подготовки

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


13. Конфигурация переключения не создает пространство подкачки на диске операционной системы.

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

14. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

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

15. В диспетчере Hyper-V выберите **Действие -> Завершение работы**. Виртуальный жесткий диск Linux готов к передаче в Azure.

## <a name="next-steps"></a>Дальнейшие действия

Теперь виртуальный жесткий диск CentOS Linux можно использовать для создания новых виртуальных машин Azure. Если вы отправляете VHD-файл в Azure впервые, см. раздел [Вариант 1. Передача VHD](upload-vhd.md#option-1-upload-a-vhd).