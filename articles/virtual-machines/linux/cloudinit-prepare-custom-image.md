---
title: Подготовка образа виртуальной машины Azure для использования с cloud-init
description: Подготовка существующего образа виртуальной машины Azure к развертыванию с помощью cloud-init
author: danis
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: how-to
ms.date: 06/24/2019
ms.author: danis
ms.openlocfilehash: 3aed65b0319f9a80c5ebc45428ff0c380c33fc3d
ms.sourcegitcommit: 6d6030de2d776f3d5fb89f68aaead148c05837e2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/05/2021
ms.locfileid: "97883271"
---
# <a name="prepare-an-existing-linux-azure-vm-image-for-use-with-cloud-init"></a>Подготовка существующего образа виртуальной машины Azure Linux к использованию с cloud-init
В этой статье показано, как выбрать и подготовить существующую виртуальную машину Azure к повторному развертыванию и применению cloud-init. Из полученного образа можно развернуть новую виртуальную машину или масштабируемый набор виртуальных машин. И то, и другое вы сможете дополнительно настроить во время развертывания с помощью cloud-init.  Эти скрипты cloud-init выполняются при первой загрузке, если в Azure подготовлены все нужные ресурсы. Дополнительные сведения о встроенной поддержке cloud-init в Azure и поддерживаемых дистрибутивах Linux см. в [обзоре cloud-init](using-cloud-init.md).

## <a name="prerequisites"></a>Предварительные требования
В этом документе предполагается, что у вас уже есть виртуальная машина Azure под управлением поддерживаемой версии ОС Linux. На этой виртуальной машине должны быть выполнены все необходимые настройки, установлены все необходимые модули, применены все необходимые обновления и проведены все проверки на соответствие требованиям. 

## <a name="preparing-rhel-76--centos-76"></a>Подготовка RHEL 7.6 и CentOS 7.6
Чтобы установить cloud-init, войдите на виртуальную машину Linux с помощью SSH и выполните следующие команды.

```bash
sudo yum makecache fast
sudo yum install -y gdisk cloud-utils-growpart
sudo yum install -y cloud-init 
```

Обновите раздел `cloud_init_modules` в `/etc/cloud/cloud.cfg`, включив добавление следующих модулей:

```bash
- disk_setup
- mounts
```

В следующем примере показано, как должен выглядеть универсальный раздел `cloud_init_modules`.

```bash
cloud_init_modules:
 - migrator
 - bootcmd
 - write-files
 - growpart
 - resizefs
 - disk_setup
 - mounts
 - set_hostname
 - update_hostname
 - update_etc_hosts
 - rsyslog
 - users-groups
 - ssh
```

Также нужно обновить в `/etc/waagent.conf` ряд задач, связанных с инициализацией и обработкой временных дисков. Выполните следующие команды, чтобы применить соответствующие параметры.

```bash
sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf
sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf
cloud-init clean
```

Укажите Azure в качестве единственного источника данных агента Linux для Azure, создав в любом текстовом редакторе новый файл `/etc/cloud/cloud.cfg.d/91-azure_datasource.cfg`, содержащий следующую строку:

```bash
# Azure Data Source config
datasource_list: [ Azure ]
```

Если в существующем образе Azure настроен файл подкачки, и вы хотите с помощью cloud-init изменить настройки файла подкачки для нового образа, необходимо удалить существующий файл подкачки.

Для образов на основе Red Hat нужно выполнить инструкции по [удалению файла подкачки](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/storage_administration_guide/swap-removing-file), предоставленные в документации по Red Hat.

Для образов на основе CentOS, в которых настроен файл подкачки, для его отключения можно выполнить следующую команду:

```bash
sudo swapoff /mnt/resource/swapfile
```

Обязательно удалите ссылку на файл подкачки из файла `/etc/fstab`, который должен выглядеть примерно так:

```output
# /etc/fstab
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=99cf66df-2fef-4aad-b226-382883643a1c / xfs defaults 0 0
UUID=7c473048-a4e7-4908-bad3-a9be22e9d37d /boot xfs defaults 0 0
```

Чтобы освободить место и удалить файл подкачки, можно запустить следующую команду:

```bash
rm /mnt/resource/swapfile
```

## <a name="extra-step-for-cloud-init-prepared-image"></a>Дополнительный этап для образов, подготовленных с помощью cloud-init
> [!NOTE]
> Если используемый образ был ранее подготовлен и настроен с помощью **cloud-init**, необходимо сделать следующее.

Следующие три команды используются, только если вы хотите создать новый специализированный исходный образ из виртуальной машины, которая ранее была создана с помощью cloud-init.  Их НЕ СЛЕДУЕТ запускать, если для настройки образа применялся агент Linux для Azure.

```bash
sudo cloud-init clean --logs
sudo waagent -deprovision+user -force
```

## <a name="finalizing-linux-agent-setting"></a>Завершение настройки агента Linux 
Все образы платформы Azure содержат установленный агент Linux для Azure, даже если они были настроены с помощью cloud-init.  Запустите следующую команду, чтобы окончательно удалить этого пользователя с компьютера Linux. 

```bash
sudo waagent -deprovision+user -force
```

Дополнительные сведения о командах удаления агента Linux для Azure см. в статье [Агент Linux для Azure](../extensions/agent-linux.md).

Завершите сеанс SSH, а затем выполните в оболочке bash следующие команды AzureCLI, которые освободят, обобщат и создадут новый образ виртуальной машины Azure.  Вместо `myResourceGroup` и `sourceVmName` укажите реальную информацию об исходной виртуальной машине.

```azurecli
az vm deallocate --resource-group myResourceGroup --name sourceVmName
az vm generalize --resource-group myResourceGroup --name sourceVmName
az image create --resource-group myResourceGroup --name myCloudInitImage --source sourceVmName
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные примеры изменения конфигурации с помощью cloud-init см. в следующих статьях:
 
- [Добавление пользователя Linux к виртуальной машине](cloudinit-add-user.md)
- [Запуск диспетчера пакетов для обновления существующих пакетов при первой загрузке](cloudinit-update-vm.md)
- [Изменение имени локального узла виртуальной машины](cloudinit-update-vm-hostname.md) 
- [Установка пакета приложения, обновление файлов конфигурации и внедрение ключей](tutorial-automate-vm-deployment.md)
