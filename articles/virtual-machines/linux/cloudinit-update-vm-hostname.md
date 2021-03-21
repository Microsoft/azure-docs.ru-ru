---
title: Использование Cloud-init для задания имени узла для виртуальной машины Linux
description: Как с помощью cloud-init и Azure CLI настроить создаваемую виртуальную машину Linux
author: rickstercdn
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: 5c3195fd338d17e2dae3ff2aadfb90790519715b
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102559398"
---
# <a name="use-cloud-init-to-set-hostname-for-a-linux-vm-in-azure"></a>Задание имени узла с помощью cloud-init для виртуальной машины Linux в Azure
В этой статье показано, как с помощью [cloud-init](https://cloudinit.readthedocs.io) настроить определенное имя узла на виртуальной машине или в масштабируемом наборе виртуальных машин при подготовке в Azure. Эти скрипты cloud-init выполняются при первой загрузке, если в Azure подготовлены все нужные ресурсы. Дополнительные сведения о встроенной поддержке cloud-init в Azure и поддерживаемых дистрибутивах Linux см. в [обзоре cloud-init](using-cloud-init.md).

## <a name="set-the-hostname-with-cloud-init"></a>Указание имени узла с помощью cloud-init
По умолчанию имя узла совпадает с именем виртуальной машины при ее создании в Azure.  Для выполнения скрипта cloud-init, чтобы изменить имя узла по умолчанию при создании виртуальной машины в Azure с помощью команды [az vm create](/cli/azure/vm), укажите файл cloud-init с параметром `--custom-data`.  

Чтобы увидеть процесс обновления в действии, создайте файл с именем *cloud_init_hostname.txt* в текущей оболочке и вставьте в него конфигурацию, приведенную ниже. Для этого примера создайте файл в Cloud Shell (не на локальном компьютере). Вы можете использовать любой редактор. Введите `sensible-editor cloud_init_hostname.txt`, чтобы создать файл и просмотреть список доступных редакторов. Выберите первый пункт, чтобы использовать редактор **nano**. Убедитесь, что весь файл cloud-init скопирован правильно, особенно первая строка.  

```yaml
#cloud-config
hostname: myhostname
```

Прежде чем развернуть этот образ, необходимо создать группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Теперь создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm) и укажите файл cloud-init с помощью `--custom-data cloud_init_hostname.txt`, как показано ниже.

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_hostname.txt \
  --generate-ssh-keys 
```

Когда создание завершится, в Azure CLI отобразятся сведения о виртуальной машине. Используйте `publicIpAddress`, чтобы подключиться к виртуальной машине по протоколу SSH. Введите свой адрес, как показано здесь:

```bash
ssh <publicIpAddress>
```

Чтобы увидеть имя виртуальной машины, используйте команду `hostname` следующим образом:

```bash
hostname
```

Виртуальная машина должна вернуть имя узла, заданное в файле cloud-init, как показано в этом примере выходных данных:

```bash
myhostname
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные примеры изменения конфигурации с помощью cloud-init см. в следующих статьях:
 
- [Добавление пользователя Linux к виртуальной машине](cloudinit-add-user.md)
- [Запуск диспетчера пакетов для обновления существующих пакетов при первой загрузке](cloudinit-update-vm.md)
- [Изменение имени локального узла виртуальной машины](cloudinit-update-vm-hostname.md) 
- [Установка пакета приложения, обновление файлов конфигурации и внедрение ключей](tutorial-automate-vm-deployment.md)
