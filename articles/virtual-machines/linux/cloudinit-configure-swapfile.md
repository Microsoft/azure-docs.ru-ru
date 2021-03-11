---
title: Настройка раздела подкачки на виртуальной машине Linux с помощью Cloud-init
description: Как использовать Cloud-init для настройки раздела подкачки на виртуальной машине Linux во время создания с помощью Azure CLI
author: rickstercdn
manager: gwallace
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: b9f4adc4e1e980db2af4fcc20b3a4492309c89f3
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102559381"
---
# <a name="use-cloud-init-to-configure-a-swap-partition-on-a-linux-vm"></a>Настройка раздела подкачки на виртуальной машине Linux с помощью Cloud-init
В этой статье показано, как с помощью [Cloud-init](https://cloudinit.readthedocs.io) настроить раздел подкачки в различных дистрибутивах Linux. Раздел подкачки традиционно настроился агентом Linux (WALA) в зависимости от того, какие дистрибутивы необходимы.  В этом документе описывается процесс создания раздела подкачки по запросу во время подготовки с помощью Cloud-init.  Дополнительные сведения о встроенной поддержке cloud-init в Azure и поддерживаемых дистрибутивах Linux см. в [обзоре cloud-init](using-cloud-init.md).

## <a name="create-swap-partition-for-ubuntu-based-images"></a>Создание раздела подкачки для образов на основе Ubuntu
По умолчанию в Azure образы из коллекции Ubuntu не создают разделы подкачки. Чтобы включить конфигурацию раздела подкачки во время подготовки виртуальной машины с помощью Cloud-init, см. [документ здесь](https://wiki.ubuntu.com/AzureSwapPartitions) на сайте Ubuntu.

## <a name="create-swap-partition-for-red-hat-and-centos-based-images"></a>Создание раздела подкачки для изображений на основе Red Hat и CentOS

Создайте файл в текущей оболочке с именем *cloud_init_swappart.txt* и вставьте следующую конфигурацию. Для этого примера создайте файл в Cloud Shell (не на локальном компьютере). Вы можете использовать любой редактор. Введите `sensible-editor cloud_init_swappart.txt`, чтобы создать файл и просмотреть список доступных редакторов. Выберите первый пункт, чтобы использовать редактор **nano**. Убедитесь, что весь файл cloud-init скопирован правильно, особенно первая строка.  

```yaml
#cloud-config
disk_setup:
  ephemeral0:
    table_type: gpt
    layout: [66, [33,82]]
    overwrite: true
fs_setup:
  - device: ephemeral0.1
    filesystem: ext4
  - device: ephemeral0.2
    filesystem: swap
mounts:
  - ["ephemeral0.1", "/mnt"]
  - ["ephemeral0.2", "none", "swap", "sw", "0", "0"]
```

Прежде чем развернуть этот образ, необходимо создать группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Теперь создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm) и укажите файл cloud-init с помощью `--custom-data cloud_init_swappart.txt`, как показано ниже.

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_swappart.txt \
  --generate-ssh-keys 
```

## <a name="verify-swap-partition-was-created"></a>Проверка создания раздела подкачки
Подключитесь по протоколу SSH к общедоступному IP-адресу виртуальной машины, который указан в выходных данных предыдущей команды. Введите свой общедоступный IP-адрес в качестве значения для **publicIpAddress**, как показано здесь:

```bash
ssh <publicIpAddress>
```

Сш'ед в виртуальную машину, проверьте, был ли создан раздел подкачки.

```bash
swapon -s
```

Выходные данные этой команды должны выглядеть следующим образом:

```text
Filename                Type        Size    Used    Priority
/dev/sdb2  partition   2494440 0   -1
```

> [!NOTE] 
> Если у вас есть образ Azure с настроенной секцией подкачки и вы хотите изменить конфигурацию раздела подкачки для новых образов, следует удалить существующую секцию подкачки. Дополнительные сведения см. в документе "Customize Images to provision by cloud-init" (Настройка образов для подготовки с помощью cloud-init).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные примеры изменения конфигурации с помощью cloud-init см. в следующих статьях:
 
- [Добавление пользователя Linux к виртуальной машине](cloudinit-add-user.md)
- [Запуск диспетчера пакетов для обновления существующих пакетов при первой загрузке](cloudinit-update-vm.md)
- [Изменение имени локального узла виртуальной машины](cloudinit-update-vm-hostname.md) 
- [Установка пакета приложения, обновление файлов конфигурации и внедрение ключей](tutorial-automate-vm-deployment.md)
