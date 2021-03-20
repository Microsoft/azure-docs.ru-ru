---
title: Настройка масштабируемого набора виртуальных машин с помощью существующего Azure Load Balancer Azure CLI
description: Узнайте, как настроить в масштабируемом наборе виртуальных машин существующую Azure Load Balancer с помощью Azure CLI.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: how-to
ms.date: 03/25/2020
ms.openlocfilehash: a60a6889217ce6ca8dccd5ebf5ee74b8f67a7757
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94518215"
---
# <a name="configure-a-virtual-machine-scale-set-with-an-existing-azure-load-balancer-using-the-azure-cli"></a>Настройка масштабируемого набора виртуальных машин с использованием существующего Azure Load Balancer с помощью Azure CLI

В этой статье вы узнаете, как настроить в масштабируемом наборе виртуальных машин существующую Azure Load Balancer.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные условия 

- В подписке, в которой будет развернут масштабируемый набор виртуальных машин, необходим существующий стандартный балансировщик нагрузки SKU.

- Для масштабируемого набора виртуальных машин требуется виртуальная сеть Azure.
 
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="deploy-a-virtual-machine-scale-set-with-existing-load-balancer"></a>Развертывание масштабируемого набора виртуальных машин с помощью существующей подсистемы балансировки нагрузки

Замените значения в квадратных скобках именами ресурсов в конфигурации.

```azurecli-interactive
az vmss create \
    --resource-group <resource-group> \
    --name <vmss-name>\
    --image <your-image> \
    --admin-username <admin-username> \
    --generate-ssh-keys  \
    --upgrade-policy-mode Automatic \
    --instance-count 3 \
    --vnet-name <virtual-network-name> \
    --subnet <subnet-name> \
    --lb <load-balancer-name> \
    --backend-pool-name <backend-pool-name>
```

В приведенном ниже примере выполняется развертывание масштабируемого набора виртуальных машин с помощью:

- Масштабируемый набор виртуальных машин с именем **myVMSS**
- Azure Load Balancer с именем **myLoadBalancer**
- Внутренний пул подсистемы балансировки нагрузки с именем **myBackendPool**
- Виртуальная сеть Azure с именем **myVnet**
- Подсеть с именем **mySubnet**
- Группа ресурсов с именем **myResourceGroup**
- Образ сервера Ubuntu для масштабируемого набора виртуальных машин

```azurecli-interactive
az vmss create \
    --resource-group myResourceGroup \
    --name myVMSS \
    --image Canonical:UbuntuServer:18.04-LTS:latest \
    --admin-username adminuser \
    --generate-ssh-keys \
    --upgrade-policy-mode Automatic \
    --instance-count 3 \
    --vnet-name myVnet\
    --subnet mySubnet \
    --lb myLoadBalancer \
    --backend-pool-name myBackendPool
```
> [!NOTE]
> После создания масштабируемого набора невозможно изменить внутренний порт для правила балансировки нагрузки, используемого зондом работоспособности балансировщика нагрузки. Чтобы изменить порт, можно удалить проверку работоспособности, обновив масштабируемый набор виртуальных машин Azure, обновить порт, а затем снова настроить зонд работоспособности.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы развернули масштабируемый набор виртуальных машин с существующим Azure Load Balancer.  Дополнительные сведения о масштабируемых наборах виртуальных машин и подсистеме балансировки нагрузки см. в следующих статьях:

- [Что такое Azure Load Balancer](load-balancer-overview.md)
- [Что такое масштабируемый набор виртуальных машин?](../virtual-machine-scale-sets/overview.md)
                                