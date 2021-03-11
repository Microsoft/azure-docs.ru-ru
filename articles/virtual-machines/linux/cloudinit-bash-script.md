---
title: Использование Cloud-init для запуска скрипта bash на виртуальной машине Linux в Azure
description: Как с помощью cloud-init выполнить bash-сценарий на виртуальной машине Linux при ее создании через Azure CLI.
author: rickstercdn
ms.service: virtual-machines
ms.collection: linux
ms.topic: how-to
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: 1dabd7db85d4222ca2e9f8387b716af291ee6a69
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102559432"
---
# <a name="use-cloud-init-to-run-a-bash-script-in-a-linux-vm-in-azure"></a>Выполнение bash-скрипта на виртуальной машине Linux в Azure с помощью cloud-init
В этой статье показано, как с помощью [cloud-init](https://cloudinit.readthedocs.io) выполнить существующий bash-скрипт на виртуальной машине или в масштабируемом наборе виртуальных машин при их подготовке в Azure. Эти скрипты cloud-init выполняются при первой загрузке, если в Azure подготовлены все нужные ресурсы. Дополнительные сведения о встроенной поддержке cloud-init в Azure и поддерживаемых дистрибутивах Linux см. в [обзоре cloud-init](using-cloud-init.md).

## <a name="run-a-bash-script-with-cloud-init"></a>Запуск bash-скрипта с помощью cloud-init
При использовании cloud-init нет необходимости преобразовывать скрипты в облачный формат (cloud-config), так как cloud-init поддерживает множество типов входных данных, в том числе bash-скрипты.

Если для запуска скриптов вы ранее использовали расширение Azure для пользовательских скриптов Linux, значит их можно без проблем перенести на cloud-init. Но не забывайте, что расширения Azure имеют встроенный механизм извещения об ошибках скриптов, поэтому развертывания образов cloud-init НЕ БУДУТ завершаться сбоем при сбоях скриптов.

Чтобы увидеть, как это работает, создайте простой bash-скрипт для тестирования. Как и файл `#cloud-config` для cloud-init, этот скрипт должен размещаться локально там, где вы будете запускать команды AzureCLI для подготовки виртуальной машины.  Для этого примера создайте файл в Cloud Shell (не на локальном компьютере). Вы можете использовать любой редактор. Введите `sensible-editor simple_bash.sh`, чтобы создать файл и просмотреть список доступных редакторов. Выберите первый пункт, чтобы использовать редактор **nano**. Убедитесь, что весь файл cloud-init скопирован правильно, особенно первая строка.  

```bash
#!/bin/sh
echo "this has been written via cloud-init" + $(date) >> /tmp/myScript.txt
```

Прежде чем развернуть этот образ, необходимо создать группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Теперь создайте виртуальную машину с помощью команды [az vm create](/cli/azure/vm) и укажите нужный bash-скрипт с помощью `--custom-data simple_bash.sh`, как показано ниже.

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data simple_bash.sh \
  --generate-ssh-keys 
```
## <a name="verify-bash-script-has-run"></a>Проверка выполнения bash-скрипта
Подключитесь по протоколу SSH к общедоступному IP-адресу виртуальной машины, который указан в выходных данных предыдущей команды. Введите свой общедоступный IP-адрес в качестве значения для **publicIpAddress**, как показано здесь:

```bash
ssh <publicIpAddress>
```

Перейдите в каталог **/tmp** и убедитесь, что файл myScript.txt существует и содержит правильный текст.  Если это не так, изучите журнал **/var/log/cloud-init.log**, чтобы получить дополнительные сведения.  Найдите следующую запись:

```bash
Running config-scripts-user using lock Running command ['/var/lib/cloud/instance/scripts/part-001']
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные примеры изменения конфигурации с помощью cloud-init см. в следующих статьях:
 
- [Добавление пользователя Linux к виртуальной машине](cloudinit-add-user.md)
- [Запуск диспетчера пакетов для обновления существующих пакетов при первой загрузке](cloudinit-update-vm.md)
- [Изменение имени локального узла виртуальной машины](cloudinit-update-vm-hostname.md) 
- [Установка пакета приложения, обновление файлов конфигурации и внедрение ключей](tutorial-automate-vm-deployment.md)
