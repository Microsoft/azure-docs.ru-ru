---
title: 'Azure ExpressRoute: сброс пиринга цепи'
description: Узнайте, как включить и отключить пирингы для канала Azure ExpressRoute с помощью Azure PowerShell.
services: expressroute
author: charwen
ms.service: expressroute
ms.topic: how-to
ms.date: 12/15/2020
ms.author: duau
ms.openlocfilehash: 0bde96ae5f4a9aff6f4a16a4f1544d9b39e5cb66
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97559579"
---
# <a name="reset-expressroute-circuit-peerings"></a>Сброс пиринга канала ExpressRoute

В этой статье описывается включение и отключение пиринга канала ExpressRoute с помощью PowerShell. Пиринга по умолчанию включаются при их создании. При отключении пиринга сеанс BGP на основном и дополнительном подключении канала ExpressRoute будет закрыт. Вы потеряли возможность подключения для этого пиринга в корпорацию Майкрософт. При включении пиринга будет установлен сеанс BGP как на основном, так и в дополнительном подключении канала ExpressRoute. Подключение к Майкрософт будет восстановлено для этого пиринга. Вы можете включить и отключить пиринг для пиринга Майкрософт и частного пиринга Azure независимо от канала ExpressRoute.

Существует два сценария, в которых может оказаться полезным сбросить пиринг ExpressRoute.
* Если вы хотите протестировать проектирование и реализацию аварийного восстановления. Например, существует две цепи ExpressRoute. Можно отключить пирингы в одном канале и принудительно выполнить отработку отказа сетевого трафика в другой канал.
* Включите обнаружение двунаправленной переадресации (БФД) для частного пиринга Azure или пиринга Майкрософт для канала ExpressRoute. БФД по умолчанию включается в частном пиринга Azure, если вы создали канал ExpressRoute после 1 августа 2018 и для пиринга Майкрософт после 10 января 2020. Если ваш канал был создан до указанной даты, потребуется сбросить пиринг, чтобы включить БФД. 

### <a name="working-with-azure-powershell"></a>Работа с Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

## <a name="reset-a-peering"></a>Сброс пиринга

1. Если вы используете PowerShell локально, откройте консоль PowerShell с повышенными привилегиями и подключитесь к своей учетной записи. Для подключения используйте следующий пример кода:

   ```azurepowershell
   Connect-AzAccount
   ```
2. Если у вас есть несколько подписок Azure, проверьте подписки для этой учетной записи.

   ```azurepowershell-interactive
   Get-AzSubscription
   ```
3. Укажите подписку, которую нужно использовать.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
   ```
4. Для извлечения цепи ExpressRoute необходимо выполнить следующие команды.

   ```azurepowershell-interactive
   $ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```
5. Определите пиринг, который требуется включить или отключить. *Пиринг* является массивом. В следующем примере Peerings[0] — это частный пиринг Azure, а Peerings[1] — это пиринг корпорации Майкрософт.

   ```azurepowershell-interactive
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
   Id                               : /subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
   Etag                             : W/"cd011bef-dc79-49eb-b4c6-81fb6ea5d178"
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
                                     "ServiceProviderName": "Coresite",
                                     "PeeringLocation": "Los Angeles",
                                     "BandwidthInMbps": 50
                                   }
   ServiceKey                       : ########-####-####-####-############
   Peerings                         : [
                                     {
                                       "Name": "AzurePrivatePeering",
                                       "Etag": "W/\"cd011bef-dc79-49eb-b4c6-81fb6ea5d178\"",
                                       "Id": "/subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit/peerings/AzurePrivatePeering",
                                       "PeeringType": "AzurePrivatePeering",
                                       "State": "Enabled",
                                       "AzureASN": 12076,
                                       "PeerASN": 123,
                                       "PrimaryPeerAddressPrefix": "10.0.0.0/30",
                                       "SecondaryPeerAddressPrefix": "10.0.0.4/30",
                                       "PrimaryAzurePort": "",
                                       "SecondaryAzurePort": "",
                                       "VlanId": 789,
                                       "MicrosoftPeeringConfig": {
                                         "AdvertisedPublicPrefixes": [],
                                         "AdvertisedCommunities": [],
                                         "AdvertisedPublicPrefixesState": "NotConfigured",
                                         "CustomerASN": 0,
                                         "LegacyMode": 0,
                                         "RoutingRegistryName": "NONE"
                                       },
                                       "ProvisioningState": "Succeeded",
                                       "GatewayManagerEtag": "",
                                       "LastModifiedBy": "Customer",
                                       "Connections": []
                                     },
                                     {
                                       "Name": "MicrosoftPeering",
                                       "Etag": "W/\"cd011bef-dc79-49eb-b4c6-81fb6ea5d178\"",
                                       "Id": "/subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit/peerings/MicrosoftPeering",
                                       "PeeringType": "MicrosoftPeering",
                                       "State": "Enabled",
                                       "AzureASN": 12076,
                                       "PeerASN": 123,
                                       "PrimaryPeerAddressPrefix": "3.0.0.0/30",
                                       "SecondaryPeerAddressPrefix": "3.0.0.4/30",
                                       "PrimaryAzurePort": "",
                                       "SecondaryAzurePort": "",
                                       "VlanId": 345,
                                       "MicrosoftPeeringConfig": {
                                         "AdvertisedPublicPrefixes": [
                                           "3.0.0.3/32"
                                         ],
                                         "AdvertisedCommunities": [],
                                         "AdvertisedPublicPrefixesState": "ValidationNeeded",
                                         "CustomerASN": 0,
                                         "LegacyMode": 0,
                                         "RoutingRegistryName": "NONE"
                                       },
                                       "ProvisioningState": "Succeeded",
                                       "GatewayManagerEtag": "",
                                       "LastModifiedBy": "Customer",
                                       "Connections": []
                                     }
                                   ]
   Authorizations                   : []
   AllowClassicOperations           : False
   GatewayManagerEtag               :
   ```
6. Чтобы изменить состояние пиринга, выполните следующие команды.

   ```azurepowershell-interactive
   $ckt.Peerings[0].State = "Disabled"
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```
   Теперь пиринг должен находиться в состоянии, установленном пользователем. 

## <a name="next-steps"></a>Дальнейшие действия
Если вам нужна помощь с устранением неполадок ExpressRoute, см. следующие статьи:
* [Проверка подключения ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* [Устранение проблем с производительностью сети](expressroute-troubleshooting-network-performance.md)
