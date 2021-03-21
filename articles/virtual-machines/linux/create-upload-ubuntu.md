---
title: Создание и передача виртуального жесткого диска с ОС Ubuntu Linux в Azure
description: Узнайте, как создать и передать виртуальный жесткий диск (VHD-файл) Azure, содержащий операционную систему Ubuntu Linux.
author: danielsollondon
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 06/06/2020
ms.author: danis
ms.openlocfilehash: 92ceecd16a428593764fe5ab6478cc4ea7ab91d7
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102554621"
---
# <a name="prepare-an-ubuntu-virtual-machine-for-azure"></a>Подготовка виртуальной машины Ubuntu для Azure


Теперь Ubuntu публикует официальные виртуальные жесткие диски Azure для загрузки по адресу [https://cloud-images.ubuntu.com/](https://cloud-images.ubuntu.com/) . Если вам нужно создать свой собственный, особый образ Ubuntu для Azure, не выполняйте описанную ниже процедуру, а начните с таких заведомо рабочих виртуальных жестких дисков и настройте их так, как вам требуется. Последние выпуски образов можно всегда найти в следующих расположениях:

* Ubuntu 16.04/Xenial: [Ubuntu-16,04-Server-клаудимг-AMD64-disk1. vmdk](https://cloud-images.ubuntu.com/releases/xenial/release/ubuntu-16.04-server-cloudimg-amd64-disk1.vmdk)
* Ubuntu 18.04/Бионик: [Бионик-Server-клаудимг-AMD64. vmdk](https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.vmdk)

## <a name="prerequisites"></a>Предварительные условия
В этой статье предполагается, что вы уже установили операционную систему Ubuntu Linux на виртуальный жесткий диск. Существует несколько средств для создания VHD-файлов, например решение для виртуализации, такое как Hyper-V. Инструкции см. в разделе [Установка роли Hyper-V и настройка виртуальной машины](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

**Замечания по установке Ubuntu**

* Дополнительные сведения о подготовке Linux для Azure см. в разделе [Общие замечания по установке Linux](create-upload-generic.md#general-linux-installation-notes).
* Формат VHDX не поддерживается в Azure, поддерживается только **фиксированный VHD**.  Диск можно преобразовать в формат VHD с помощью диспетчера Hyper-V или `Convert-VHD` командлета.
* При установке системы Linux рекомендуется использовать стандартные разделы, а не LVM (как правило, значение по умолчанию во многих дистрибутивах). Это позволит избежать конфликта имен LVM при клонировании виртуальных машин, особенно если диск с OC может быть подключен к другой ВМ в целях устранения неполадок. Для дисков данных можно использовать [LVM](/previous-versions/azure/virtual-machines/linux/configure-lvm) или [RAID](/previous-versions/azure/virtual-machines/linux/configure-raid).
* Не настраивайте раздел swap или файл подкачки на диске операционной системы. Агент подготовки Cloud-init можно настроить для создания файла подкачки или раздела подкачки на диске временных ресурсов. Дополнительные сведения описаны далее.
* Размер виртуальной памяти всех VHD в Azure должен быть округлен до 1 МБ. При конвертации диска в формате RAW в виртуальный жесткий диск убедитесь, что размер диска RAW в несколько раз превышает 1 МБ. См. дополнительные сведения в [примечаниях по установке Linux](create-upload-generic.md#general-linux-installation-notes).

## <a name="manual-steps"></a>Создание вручную
> [!NOTE]
> Прежде чем создавать собственный пользовательский образ Ubuntu для Azure, рекомендуется использовать предварительно созданные и проверенные образы [https://cloud-images.ubuntu.com/](https://cloud-images.ubuntu.com/) .
> 
> 

1. На центральной панели диспетчера Hyper-V выберите виртуальную машину.

2. Щелкните **Подключение** , чтобы открыть окно виртуальной машины.

3. Замените текущие репозитории в образе, чтобы использовать репозиторий Azure Ubuntu.

    Перед редактированием `/etc/apt/sources.list` рекомендуется сделать резервную копию:

    ```console
    # sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
    ```

    Ubuntu 16,04 и Ubuntu 18,04:

    ```console
    # sudo sed -i 's/http:\/\/archive\.ubuntu\.com\/ubuntu\//http:\/\/azure\.archive\.ubuntu\.com\/ubuntu\//g' /etc/apt/sources.list
    # sudo sed -i 's/http:\/\/[a-z][a-z]\.archive\.ubuntu\.com\/ubuntu\//http:\/\/azure\.archive\.ubuntu\.com\/ubuntu\//g' /etc/apt/sources.list
    # sudo apt-get update
    ```

4. Образы Ubuntu Azure теперь используют [специализированное ядро Azure](https://ubuntu.com/blog/microsoft-and-canonical-increase-velocity-with-azure-tailored-kernel). Обновите операционную систему до последней версии ядра Azure и установите инструменты Azure Linux (включая зависимости Hyper-V), выполнив следующие команды:

    Ubuntu 16,04 и Ubuntu 18,04:

    ```console
    # sudo apt update
    # sudo apt install linux-azure linux-image-azure linux-headers-azure linux-tools-common linux-cloud-tools-common linux-tools-azure linux-cloud-tools-azure
    (recommended) # sudo apt full-upgrade

    # sudo reboot
    ```

5. Измените строку загрузки ядра в конфигурации Grub, чтобы включить дополнительные параметры ядра для Azure. Для этого откройте файл `/etc/default/grub` в текстовом редакторе, найдите переменную `GRUB_CMDLINE_LINUX_DEFAULT` (или добавьте ее, если это необходимо) и измените ее, включив следующие параметры:

    ```text
    GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300 quiet splash"
    ```

    Сохраните и закройте файл, а затем выполните команду `sudo update-grub`. Это гарантирует отправку всех сообщений консоли на первый последовательный порт, что может помочь технической поддержке Azure в плане отладки.

6. Убедитесь, что SSH-сервер установлен и настроен для включения во время загрузки.  Обычно это сделано по умолчанию.

7. Установите Cloud-init (агент подготовки) и агент Linux для Azure (обработчик гостевых расширений). Cloud-init использует нетплан для настройки системной конфигурации сети во время подготовки и каждой последующей загрузки.

    ```console
    # sudo apt update
    # sudo apt install cloud-init netplan.io walinuxagent && systemctl stop walinuxagent
    ```

   > [!Note]
   >  Установка пакета `walinuxagent` приведет к удалению пакетов `NetworkManager` и `NetworkManager-gnome` (если они установлены).

8. Удалите конфигурации Cloud-init по умолчанию и оставшиеся артефакты нетплан, которые могут конфликтовать с подготовкой Cloud-init в Azure:

    ```console
    # rm -f /etc/cloud/cloud.cfg.d/50-curtin-networking.cfg /etc/cloud/cloud.cfg.d/curtin-preserve-sources.cfg
    # rm -f /etc/cloud/ds-identify.cfg
    # rm -f /etc/netplan/*.yaml
    ```

9. Настройте Cloud-init для инициализации системы с помощью источника данных Azure.

    ```console
    # cat > /etc/cloud/cloud.cfg.d/90_dpkg.cfg << EOF
    datasource_list: [ Azure ]
    EOF

    # cat > /etc/cloud/cloud.cfg.d/90-azure.cfg << EOF
    system_info:
       package_mirrors:
         - arches: [i386, amd64]
           failsafe:
             primary: http://archive.ubuntu.com/ubuntu
             security: http://security.ubuntu.com/ubuntu
           search:
             primary:
               - http://azure.archive.ubuntu.com/ubuntu/
             security: []
         - arches: [armhf, armel, default]
           failsafe:
             primary: http://ports.ubuntu.com/ubuntu-ports
             security: http://ports.ubuntu.com/ubuntu-ports
    EOF

    # cat > /etc/cloud/cloud.cfg.d/10-azure-kvp.cfg << EOF
    reporting:
      logging:
        type: log
      telemetry:
        type: hyperv
    EOF
    ```

10. Настройте агент Linux для Azure, чтобы он зависел от Cloud-init, чтобы выполнить подготовку. Дополнительные сведения об этих параметрах см. в [проекте WALinuxAgent](https://github.com/Azure/WALinuxAgent) .

    ```console
    sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf
    sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
    sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf

    cat >> /etc/waagent.conf << EOF
    # For Azure Linux agent version >= 2.2.45, this is the option to configure,
    # enable, or disable the provisioning behavior of the Linux agent.
    # Accepted values are auto (default), waagent, cloud-init, or disabled.
    # A value of auto means that the agent will rely on cloud-init to handle
    # provisioning if it is installed and enabled, which in this case it will.
    Provisioning.Agent=auto
    EOF
    ```

11. Очистите журналы и артефакты среды выполнения агента Linux для облака и Azure:

    ```console
    # sudo cloud-init clean --logs --seed
    # sudo rm -rf /var/lib/cloud/
    # sudo systemctl stop walinuxagent.service
    # sudo rm -rf /var/lib/waagent/
    # sudo rm -f /var/log/waagent.log
    ```

12. Выполните следующие команды, чтобы отменить подготовку виртуальной машины и подготовить ее в Azure:

    > [!NOTE]
    > `sudo waagent -force -deprovision+user`Команда будет пытаться очистить систему и сделать ее подходящей для повторной подготовки. `+user`Параметр удаляет последнюю подготовленную учетную запись пользователя и связанные с ней данные.

    > [!WARNING]
    > При использовании приведенной выше команды не гарантируется, что образ удаляется из всех конфиденциальных данных и подготавливается к повторному распространению.

    ```console
    # sudo waagent -force -deprovision+user
    # rm -f ~/.bash_history
    # export HISTSIZE=0
    # logout
    ```

13. В диспетчере Hyper-V выберите **Действие -> Завершение работы**.

14. Azure принимает только виртуальные жесткие диски фиксированного размера. Если диск ОС виртуальной машины не является VHD фиксированного размера, используйте `Convert-VHD` командлет PowerShell и укажите `-VHDType Fixed` параметр. Просмотрите документацию по `Convert-VHD` этой ссылке: [Convert-VHD](/powershell/module/hyper-v/convert-vhd).


## <a name="next-steps"></a>Дальнейшие действия
Теперь виртуальный жесткий диск Ubuntu Linux можно использовать для создания новых виртуальных машин Azure. Если вы отправляете VHD-файл в Azure впервые, см. раздел [Вариант 1. Передача VHD](upload-vhd.md#option-1-upload-a-vhd).