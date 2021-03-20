---
title: Добавление IPv6 в приложение IPv4 в виртуальной сети Azure Azure CLI
titlesuffix: Azure Virtual Network
description: В этой статье показано, как развернуть IPv6-адреса в существующем приложении в виртуальной сети Azure с помощью Azure CLI.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: 9a321687a755f8a3d6e6d9139138d61c58764ef4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98932601"
---
# <a name="add-ipv6-to-an-ipv4-application-in-azure-virtual-network---azure-cli"></a>Добавление IPv6 в приложение IPv4 в виртуальной сети Azure Azure CLI

В этой статье показано, как добавить IPv6-адреса в приложение, которое использует общедоступный IP-адрес IPv4 в виртуальной сети Azure для Load Balancer (цен. категория "Стандартный") с помощью Azure CLI. Обновление на месте включает виртуальную сеть и подсеть, Load Balancer (цен. категория "Стандартный") с многоадресными конфигурациями IPv4 и IPV6, виртуальные машины с сетевыми картами, которые имеют конфигурации IPv4 + IPv6, группу безопасности сети и общедоступные IP-адреса.

## <a name="prerequisites"></a>Предварительные условия

- В этой статье предполагается, что вы развернули Load Balancer (цен. категория "Стандартный"), как описано в разделе [Краткое руководство по созданию Load Balancer (цен. категория "Стандартный") Azure CLI](../load-balancer/quickstart-load-balancer-standard-public-cli.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.28 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-ipv6-addresses"></a>Создание IPv6-адресов

Создайте Общедоступный IPv6-адрес с помощью команды [AZ Network public-IP Create](/cli/azure/network/public-ip) для Load Balancer (цен. Категория "Стандартный"). В следующем примере создается общедоступный IP-адрес IPv6 с именем *PublicIP_v6* в группе ресурсов *myResourceGroupSLB* :

```azurecli-interactive
az network public-ip create \
--name PublicIP_v6 \
--resource-group MyResourceGroupSLB \
--location EastUS \
--sku Standard \
--allocation-method static \
--version IPv6
```

## <a name="configure-ipv6-load-balancer-frontend"></a>Настройка внешнего интерфейса балансировщика нагрузки IPv6

Настройте подсистему балансировки нагрузки с новым IP-адресом IPv6 с помощью команды [AZ Network фунтов интерфейсного IP-адреса Create](/cli/azure/network/lb/frontend-ip#az-network-lb-frontend-ip-create) , как показано ниже.

```azurecli-interactive
az network lb frontend-ip create \
--lb-name myLoadBalancer \
--name dsLbFrontEnd_v6 \
--resource-group MyResourceGroupSLB \
--public-ip-address PublicIP_v6
```

## <a name="configure-ipv6-load-balancer-backend-pool"></a>Настройка внутреннего пула подсистемы балансировки нагрузки IPv6

Создайте внутренний пул для сетевых карт с IPv6-адресами, используя команду [AZ Network фунтов Address-Pool Create](/cli/azure/network/lb/address-pool#az-network-lb-address-pool-create) , как показано ниже.

```azurecli-interactive
az network lb address-pool create \
--lb-name myLoadBalancer \
--name dsLbBackEndPool_v6 \
--resource-group MyResourceGroupSLB
```

## <a name="configure-ipv6-load-balancer-rules"></a>Настройка правил подсистемы балансировки нагрузки IPv6

Создайте правила подсистемы балансировки нагрузки IPv6 с помощью команды [AZ Network фунтов Rule Create](/cli/azure/network/lb/rule#az-network-lb-rule-create).

```azurecli-interactive
az network lb rule create \
--lb-name myLoadBalancer \
--name dsLBrule_v6 \
--resource-group MyResourceGroupSLB \
--frontend-ip-name dsLbFrontEnd_v6 \
--protocol Tcp \
--frontend-port 80 \
--backend-port 80 \
--backend-pool-name dsLbBackEndPool_v6
```

## <a name="add-ipv6-address-ranges"></a>Добавить диапазоны IPv6-адресов

Добавьте диапазоны IPv6-адресов в виртуальную сеть и подсеть, в которых размещен балансировщик нагрузки, следующим образом:

```azurecli-interactive
az network vnet update \
--name myVnet  \
--resource-group MyResourceGroupSLB \
--address-prefixes  "10.0.0.0/16"  "fd00:db8:deca::/48"

az network vnet subnet update \
--vnet-name myVnet \
--name mySubnet \
--resource-group MyResourceGroupSLB \
--address-prefixes  "10.0.0.0/24"  "fd00:db8:deca:deed::/64"  
```

## <a name="add-ipv6-configuration-to-nics"></a>Добавление конфигурации IPv6 в сетевые карты

Настройте сетевые карты виртуальных машин с адресом IPv6 с помощью команды [AZ Network NIC IP-config Create](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-create) , как показано ниже.

```azurecli-interactive
az network nic ip-config create \
--name dsIp6Config_NIC1 \
--nic-name myNicVM1 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name dsLB

az network nic ip-config create \
--name dsIp6Config_NIC2 \
--nic-name myNicVM2 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name myLoadBalancer

az network nic ip-config create \
--name dsIp6Config_NIC3 \
--nic-name myNicVM3 \
--resource-group MyResourceGroupSLB \
--vnet-name myVnet \
--subnet mySubnet \
--private-ip-address-version IPv6 \
--lb-address-pools dsLbBackEndPool_v6 \
--lb-name myLoadBalancer
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Просмотр двух виртуальных сетей IPv6 с двумя стеками в портал Azure

Виртуальную сеть с двумя стеками IPv6 можно просмотреть в портал Azure следующим образом:
1. На панели поиска портала введите *myVnet*.
2. Когда в результатах поиска появится **myVnet** , выберите его. Откроется страница **обзора** для виртуальной сети с двумя стеками с именем *myVNet*. В виртуальной сети с двумя стеками показаны три сетевых адаптера с конфигурациями IPv4 и IPv6, расположенными в подсети с двойным стеком с именем *mySubnet*.

  ![Виртуальная сеть с двумя стеками IPv6 в Azure](./media/ipv6-add-to-existing-vnet-powershell/ipv6-dual-stack-vnet.png)


## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurecli-interactive
az group delete --name MyAzureResourceGroupSLB
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы обновили существующую Load Balancer (цен. категория "Стандартный") с многоинтерфейсной IP-конфигурацией IPv4 для конфигурации с двумя стеками (IPv4 и IPv6). Вы также добавили конфигурации IPv6 в сетевые адаптеры виртуальных машин в серверном пуле. Дополнительные сведения о поддержке IPv6 в виртуальных сетях Azure см. в статье [что такое IPv6 для виртуальной сети Azure?](ipv6-overview.md)
