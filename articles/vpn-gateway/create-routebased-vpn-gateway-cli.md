---
title: 'Создание VPN-шлюза Azure на основе маршрутов: CLI'
description: Быстрое создание VPN-шлюза Azure на основе маршрутов с помощью Azure CLI для VPN-подключения к локальной сети или для подключения виртуальных сетей.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 2bbd7e39ee65ba304ec62697b6fcc77bea133b41
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94737258"
---
# <a name="create-a-route-based-vpn-gateway-using-cli"></a>Создание VPN-шлюза на основе маршрута с помощью CLI

В статье кратко описано создание VPN-шлюза в Azure с использованием CLI в Azure. VPN-шлюз используется при создании VPN-подключения к локальной сети. Также вы можете использовать VPN-шлюз для подключения виртуальных сетей.

Инструкции в этой статье позволяют создать виртуальную сеть, подсеть, подсеть шлюза и VPN-шлюз на основе маршрутов (шлюз виртуальной сети). Создание шлюза в виртуальной сети может занять 45 минут или более. Создав шлюз, можно создавать подключения. Для этих действий требуется подписка Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. 


```azurecli-interactive
az group create --name TestRG1 --location eastus
```

## <a name="create-a-virtual-network"></a><a name="vnet"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью команды [az network vnet create](/cli/azure/network/vnet). В следующем примере создается виртуальная сеть с именем **VNet1** в расположении **EastUS**.

```azurecli-interactive
az network vnet create \
  -n VNet1 \
  -g TestRG1 \
  -l eastus \
  --address-prefix 10.1.0.0/16 \
  --subnet-name Frontend \
  --subnet-prefix 10.1.0.0/24
```

## <a name="add-a-gateway-subnet"></a><a name="gwsubnet"></a>Добавление подсети шлюза

Подсеть шлюза содержит зарезервированные IP-адреса, используемые службами шлюза виртуальной сети. Используйте следующие примеры для добавления подсети шлюза:

```azurecli-interactive
az network vnet subnet create \
  --vnet-name VNet1 \
  -n GatewaySubnet \
  -g TestRG1 \
  --address-prefix 10.1.255.0/27 
```

## <a name="request-a-public-ip-address"></a><a name="PublicIP"></a>Запрос общедоступного IP-адреса

VPN-шлюз должен иметь динамически выделяемый общедоступный IP-адрес. VPN-шлюзу, созданному для виртуальной сети, выделяется общедоступный IP-адрес. Воспользуйтесь следующим примером для запроса общедоступного IP-адреса:

```azurecli-interactive
az network public-ip create \
  -n VNet1GWIP \
  -g TestRG1 \
  --allocation-method Dynamic 
```

## <a name="create-the-vpn-gateway"></a><a name="CreateGateway"></a>Создание VPN-шлюза

Чтобы создать VPN-шлюз, используйте команду [az network vnet-gateway create](/cli/azure/group).

При выполнении этой команды с использованием параметра `--no-wait` вы не увидите ответа или выходных данных. Параметр `--no-wait` позволяет создать шлюз в фоновом режиме. Это не означает, что создание VPN-шлюза завершается немедленно.

```azurecli-interactive
az network vnet-gateway create \
  -n VNet1GW \
  -l eastus \
  --public-ip-address VNet1GWIP \
  -g TestRG1 \
  --vnet VNet1 \
  --gateway-type Vpn \
  --sku VpnGw1 \
  --vpn-type RouteBased \
  --no-wait
```

Для создания VPN-шлюза требуется не менее 45 минут.

## <a name="view-the-vpn-gateway"></a><a name="viewgw"></a>Просмотр VPN-шлюза

```azurecli-interactive
az network vnet-gateway show \
  -n VNet1GW \
  -g TestRG1
```

Результат выглядит примерно так.

```output
{
  "activeActive": false,
  "bgpSettings": null,
  "enableBgp": false,
  "etag": "W/\"6c61f8cb-d90f-4796-8697\"",
  "gatewayDefaultSite": null,
  "gatewayType": "Vpn",
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW",
  "ipConfigurations": [
    {
      "etag": "W/\"6c61f8cb-d90f-4796-8697\"",
      "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/vnetGatewayConfig0",
      "name": "vnetGatewayConfig0",
      "privateIpAllocationMethod": "Dynamic",
      "provisioningState": "Updating",
      "publicIpAddress": {
        "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/publicIPAddresses/VNet1GWIP",
        "resourceGroup": "TestRG1"
      },
      "resourceGroup": "TestRG1",
      "subnet": {
        "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworks/VNet1/subnets/GatewaySubnet",
        "resourceGroup": "TestRG1"
      }
    }
  ],
  "location": "eastus",
  "name": "VNet1GW",
  "provisioningState": "Updating",
  "resourceGroup": "TestRG1",
  "resourceGuid": "69c269e3-622c-4123-9231",
  "sku": {
    "capacity": 2,
    "name": "VpnGw1",
    "tier": "VpnGw1"
  },
  "tags": null,
  "type": "Microsoft.Network/virtualNetworkGateways",
  "vpnClientConfiguration": null,
  "vpnType": "RouteBased"
}
```

### <a name="view-the-public-ip-address"></a>Просмотр общедоступного IP-адреса

Для просмотра общедоступного IP-адреса, назначенного шлюзу, воспользуйтесь следующим примером.

```azurecli-interactive
az network public-ip show \
  --name VNet1GWIP \
  --resource-group TestRG11
```

Значение в поле **ipAddress** представляет собой общедоступный IP-адрес VPN-шлюза.

Пример ответа:

```output
{
  "dnsSettings": null,
  "etag": "W/\"a12d4d03-b27a-46cc-b222-8d9364b8166a\"",
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/publicIPAddresses/VNet1GWIP",
  "idleTimeoutInMinutes": 4,
  "ipAddress": "13.90.195.184",
  "ipConfiguration": {
    "etag": null,
    "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/vnetGatewayConfig0",
```

## <a name="clean-up-resources"></a>Очистка ресурсов

При отсутствии необходимости дальнейшего использования созданных ресурсов группу ресурсов можно удалить командой [az group delete](/cli/azure/group). При этом будет удалена группа ресурсов и все содержащиеся в ней ресурсы.

```azurecli-interactive 
az group delete --name TestRG1 --yes
```

## <a name="next-steps"></a>Дальнейшие действия

Создав шлюз, можно создать подключение между вашей и другой виртуальной сетью. Также можно создать подключение между виртуальной сетью и локальным расположением.

> [!div class="nextstepaction"]
> [Создание подключения "сеть — сеть"](vpn-gateway-create-site-to-site-rm-powershell.md)<br><br>
> [Создание подключения "точка — сеть"](vpn-gateway-howto-point-to-site-rm-ps.md)<br><br>
> [Создание подключения к другой виртуальной сети](vpn-gateway-vnet-vnet-rm-ps.md)
