---
title: Настройка предпочтительного варианта маршрутизации для общедоступного IP-адреса с помощью Azure CLI
titlesuffix: Azure Virtual Network
description: Узнайте, как создать общедоступный IP-адрес с предпочтением маршрутизации трафика Интернета с помощью Azure CLI.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2021
ms.author: mnayak
ms.openlocfilehash: 2291767c162953f8339fb8cc27e55b96290ef795
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101665955"
---
# <a name="configure-routing-preference-for-a-public-ip-address-using-azure-cli"></a>Настройка предпочтительного варианта маршрутизации для общедоступного IP-адреса с помощью Azure CLI

В этой статье показано, как настроить предпочтительный вариант маршрутизации через сеть ISP (параметр **Интернет**) для общедоступного IP-адреса с помощью Azure CLI. Созданный общедоступный IP-адрес можно связать со следующими ресурсами Azure для входящего и исходящего Интернет-трафика.

* Виртуальная машина
* Набор масштабирования виртуальных машин
* Служба Azure Kubernetes (AKS)
* подсистема балансировки нагрузки с доступом в Интернет;
* Шлюз приложений
* Брандмауэр Azure

По умолчанию предпочтительный вариант маршрутизации для общедоступного IP-адреса настроен на глобальную сеть Майкрософт для всех служб Azure и его можно связать с любой службой Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется версия 2.0.49 или более поздняя Azure CLI. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов
Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create). В следующем примере создается группа ресурсов в регионе Azure **Восточная часть США**.

```azurecli
  az group create --name myResourceGroup --location eastus
```
## <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса

Создайте общедоступный IP-адрес с предпочтением маршрутизации типа **Интернет** с помощью команды [AZ Network public-IP Create](/cli/azure/network/public-ip#az-network-public-ip-create), используя формат, как показано ниже.

Следующая команда создает новый общедоступный IP-адрес с параметром предпочтительного варианта маршрутизации **Интернет** в регионе Azure **Восточная часть США**.

```azurecli
az network public-ip create \
--name MyRoutingPrefIP \
--resource-group MyResourceGroup \
--location eastus \
--ip-tags 'RoutingPreference=Internet' \
--sku STANDARD \
--allocation-method static \
--version IPv4
```

> [!NOTE]
>  Сейчас при выборе предпочтительного варианта маршрутизации поддерживаются только общедоступные IP-адреса IPV4.

Вы можете связать созданный выше общедоступный IP-адрес с виртуальной машиной [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) или [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Используйте раздел руководства, посвященный CLI: [Свяжите общедоступный IP-адрес с виртуальной машиной.](associate-public-ip-address-vm.md#azure-cli) Вы также можете связать общедоступный IP-адрес, созданный выше, с [Azure Load Balancer](../load-balancer/load-balancer-overview.md), назначив его конфигурации **внешнего интерфейса** подсистемы балансировки нагрузки. Общедоступный IP-адрес будет выполнять функцию виртуального IP-адреса с балансировкой нагрузки.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [предпочтительном варианте маршрутизации в общедоступных IP-адресах](routing-preference-overview.md). 
- [Настройка предпочтительного варианта маршрутизации для виртуальной машины с помощью Azure CLI.](configure-routing-preference-virtual-machine-cli.md)

