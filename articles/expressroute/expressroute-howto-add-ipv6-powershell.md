---
title: 'Azure ExpressRoute: Добавление поддержки IPv6 с помощью Azure PowerShell'
description: Узнайте, как добавить поддержку IPv6 для подключения к развертываниям Azure с помощью Azure PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 03/02/2021
ms.author: duau
ms.openlocfilehash: e893216eb8e2d7e44e3f272f6b965b84c6253f7f
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104870336"
---
# <a name="add-ipv6-support-for-private-peering-using-azure-powershell-preview"></a>Добавление поддержки IPv6 для частного пиринга с помощью Azure PowerShell (Предварительная версия)

В этой статье описывается, как добавить поддержку IPv6 для подключения через ExpressRoute к ресурсам в Azure с помощью Azure PowerShell.

> [!Note]
> Сейчас эта функция доступна для предварительной версии в [регионах Azure с зоны доступности](https://docs.microsoft.com/azure/availability-zones/az-region#azure-regions-with-availability-zones). Таким образом, канал ExpressRoute можно создать с помощью любого расположения пиринга, но развертываемые на основе IPv6 развертывания должны находиться в регионе с Зоны доступности.

## <a name="working-with-azure-powershell"></a>Работа с Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

## <a name="register-for-public-preview"></a>Зарегистрируйтесь для просмотра общедоступной предварительной версии
Перед добавлением поддержки IPv6 сначала необходимо зарегистрировать подписку. Для регистрации выполните следующие действия с помощью Azure PowerShell.
1.  Войдите в Azure и выберите подписку. Это необходимо сделать для подписки, содержащей канал ExpressRoute, а также подписки, содержащей развертывания Azure (если они различаются).

    ```azurepowershell-interactive
    Connect-AzAccount 

    Select-AzSubscription -Subscription "<SubscriptionID or SubscriptionName>"
    ```

2. Запрос на регистрацию подписки на общедоступную предварительную версию с помощью следующей команды:
    ```azurepowershell-interactive
    Register-AzProviderFeature -FeatureName AllowIpv6PrivatePeering -ProviderNamespace Microsoft.Network
    ```

Ваш запрос будет утвержден командой ExpressRoute в течение 2-3 рабочих дней.

## <a name="add-ipv6-private-peering-to-your-expressroute-circuit"></a>Добавление частного пиринга IPv6 в канал ExpressRoute

1. [Создайте канал ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-howto-circuit-arm) или используйте существующий канал. Получите канал, выполнив команду **Get-азекспрессраутеЦиркуит** :

    ```azurepowershell-interactive
    $ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
    ```

2. Получите конфигурацию частного пиринга для канала, выполнив команду **Get-азекспрессраутеЦиркуитпирингконфиг**:

    ```azurepowershell-interactive
    Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
    ```

3. Добавьте частный пиринг IPv6 в существующую конфигурацию частного пиринга IPv4. Предоставьте пару из/126 подсетей IPv6, которыми вы владеете, для основной ссылки и дополнительных ссылок. Из каждой подсети вы назначите первый готовый к применению IP-адрес для своего маршрутизатора, так как корпорация Майкрософт использует второй IP-адрес для маршрутизатора Майкрософт.

    > [!Note]
    > Узлы ASN и VlanId должны соответствовать значениям в конфигурации частного пиринга IPv4.

    ```azurepowershell-interactive
    Set-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 200 -PeerAddressType IPv6

    Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
    ```

4. После успешного сохранения конфигурации снова получите канал, выполнив команду **Get-азекспрессраутеЦиркуит** . Ответ должен выглядеть, как в следующем примере:

    ```azurepowershell
    Name                             : ExpressRouteARMCircuit
    ResourceGroupName                : ExpressRouteResourceGroup
    Location                         : eastus
    Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
    Etag                             : W/"################################"
    ProvisioningState                : Succeeded
    Sku                              : {
                                         "Name": "Standard_MeteredData",
                                         "Tier": "Standard",
                                         "Family": "MeteredData"
                                       }
    CircuitProvisioningState         : Enabled
    ServiceProviderProvisioningState : Provisioned
    ServiceProviderNotes             :
    ServiceProviderProperties        : {
                                         "ServiceProviderName": "Equinix",
                                         "PeeringLocation": "Washington DC",
                                         "BandwidthInMbps": 50
                                       }
    ExpressRoutePort                 : null
    BandwidthInGbps                  :
    Stag                             : 29
    ServiceKey                       : **************************************
    Peerings                         : [
                                         {
                                           "Name": "AzurePrivatePeering",
                                           "Etag": "W/\"facc8972-995c-4861-a18d-9a82aaa7167e\"",
                                           "Id": "/subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit/peerings/AzurePrivatePeering",
                                           "PeeringType": "AzurePrivatePeering",
                                           "State": "Enabled",
                                           "AzureASN": 12076,
                                           "PeerASN": 100,
                                           "PrimaryPeerAddressPrefix": "192.168.15.16/30",
                                           "SecondaryPeerAddressPrefix": "192.168.15.20/30",
                                           "PrimaryAzurePort": "",
                                           "SecondaryAzurePort": "",
                                           "VlanId": 200,
                                           "ProvisioningState": "Succeeded",
                                           "GatewayManagerEtag": "",
                                           "LastModifiedBy": "Customer",
                                           "Ipv6PeeringConfig": {
                                             "State": "Enabled",
                                             "PrimaryPeerAddressPrefix": "3FFE:FFFF:0:CD30::/126",
                                             "SecondaryPeerAddressPrefix": "3FFE:FFFF:0:CD30::4/126"
                                           },
                                           "Connections": [],
                                           "PeeredConnections": []
                                         },
                                       ]
    Authorizations                   : []
    AllowClassicOperations           : False
    GatewayManagerEtag               :
    ```

## <a name="update-your-connection-to-an-existing-virtual-network"></a>Обновление подключения к существующей виртуальной сети

Выполните следующие действия, если у вас есть среда ресурсов Azure в регионе с Зоны доступности, с которыми вы хотите использовать частный пиринг IPv6 с.

1. Получите виртуальную сеть, к которой подключен канал ExpressRoute.

    ```azurepowershell-interactive
    $vnet = Get-AzVirtualNetwork -Name "VirtualNetwork" -ResourceGroupName "ExpressRouteResourceGroup"
    ```

2. Добавьте адресное пространство IPv6 в виртуальную сеть.

    ```azurepowershell-interactive
    $vnet.AddressSpace.AddressPrefixes.add("ace:daa:daaa:deaa::/64")
    Set-AzVirtualNetwork -VirtualNetwork $vnet
    ```

3. Добавьте адресное пространство IPv6 в подсеть шлюза. Подсеть IPv6 шлюза должна иметь значение/64 или больше.

    ```azurepowershell-interactive
    Set-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix "10.0.0.0/26", "ace:daa:daaa:deaa::/64"
    Set-AzVirtualNetwork -VirtualNetwork $vnet
    ```

4. Если у вас есть шлюз, избыточный в зоне, выполните следующую команду, чтобы включить IPv6-подключение. В противном случае [Создайте шлюз виртуальной сети](https://docs.microsoft.com/azure/expressroute/expressroute-howto-add-gateway-resource-manager) с помощью SKU, избыточного в виде зоны (ErGw1AZ, ErGw2AZ, ErGw3AZ).

    ```azurepowershell-interactive
    $gw = Get-AzVirtualNetworkGateway -Name "GatewayName" -ResourceGroupName "ExpressRouteResourceGroup"
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw
    ```

## <a name="create-a-connection-to-a-new-virtual-network"></a>Создание подключения к новой виртуальной сети

Если вы планируете подключиться к новому набору ресурсов Azure в регионе с Зоны доступности помощью частного пиринга IPv6, выполните следующие действия.

1. Создайте виртуальную сеть с двумя стеками с адресным пространством IPv4 и IPv6. Дополнительные сведения см. [в разделе Создание виртуальной сети](https://docs.microsoft.com/azure/virtual-network/quick-create-portal#create-a-virtual-network).

2. [Создайте подсеть шлюза с двойным стеком](https://docs.microsoft.com/azure/expressroute/expressroute-howto-add-gateway-resource-manager#add-a-gateway).

3. [Создайте шлюз виртуальной сети](https://docs.microsoft.com/azure/expressroute/expressroute-howto-add-gateway-resource-manager#add-a-gateway) с помощью SKU, избыточного в виде зоны (ErGw1AZ, ErGw2AZ, ErGw3AZ). Если вы планируете использовать Фастпас, используйте ErGw3AZ (Обратите внимание, что эта возможность доступна только для каналов с помощью ExpressRoute Direct).

4. [Свяжите виртуальную сеть с каналом ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-howto-linkvnet-arm).

## <a name="limitations"></a>Ограничения
Хотя поддержка IPv6 доступна для подключений к развертываниям в регионах с Зоны доступности, она не поддерживает следующие варианты использования:

* Подключения к развертываниям в Azure с помощью SKU шлюза, отличного от AZ ExpressRoute
* Подключения к развертываниям в регионах, не относящихся к AZ
* Global Reach соединений между каналами ExpressRoute
* Использование ExpressRoute с виртуальной глобальной сетью
* Фастпас с прямыми каналами, отличными от ExpressRoute
* Фастпас с каналами в следующих расположениях пиринга: Дубаи
* Сосуществование с VPN-шлюзом

## <a name="next-steps"></a>Дальнейшие действия

Сведения об устранении неполадок ExpressRoute см. в следующих статьях:

* [Проверка подключения ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* [Устранение проблем с производительностью сети](expressroute-troubleshooting-network-performance.md)
