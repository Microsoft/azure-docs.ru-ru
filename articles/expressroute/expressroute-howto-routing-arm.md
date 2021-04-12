---
title: Руководство по Настройка пиринга для канала ExpressRoute — Azure PowerShell
description: В этом руководстве показано, как создать конфигурацию маршрутизации для канала ExpressRoute (модель развертывания Resource Manager) и управлять ею с помощью PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: duau
ms.openlocfilehash: 7cfd378ae621192cd98b482b66c85c3dcd3ca454
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101721945"
---
# <a name="tutorial-create-and-modify-peering-for-an-expressroute-circuit-using-powershell"></a>Руководство по Создание и изменение пиринга для канала ExpressRoute с помощью PowerShell

В этом руководстве показано, как создать конфигурацию маршрутизации для канала ExpressRoute (модель развертывания Resource Manager) и управлять ею с помощью PowerShell. Вы также сможете проверить состояние, обновить, удалить и отозвать пиринги для канала ExpressRoute. Если вы хотите использовать для работы с каналом другой метод, выберите подходящую статью из списка ниже.

> [!div class="op_single_selector"]
> * [Портал Azure](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [Azure CLI](howto-routing-cli.md)
> * [Общедоступный пиринг](about-public-peering.md)
> * [Видео — частный пиринг](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [Видео — пиринг Майкрософт](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (классическая модель)](expressroute-howto-routing-classic.md)
> 

Эти инструкции распространяются только на каналы от поставщиков, предоставляющих услуги подключения второго уровня. Если ваш поставщик услуг подключения предлагает услуги третьего уровня (обычно это IPVPN, например MPLS), он отвечает за настройку маршрутизации и управление ею.

> [!IMPORTANT]
> Мы пока не предлагаем использовать пиринги, которые настроены поставщиками услуг на портале управления службами. Такая возможность будет предоставлена позже. Перед настройкой пирингов BGP уточните информацию у поставщика услуг.
> 

Вы можете настроить частный пиринг и пиринг Майкрософт для канала ExpressRoute (общедоступный пиринг Azure не поддерживается для новых каналов). Пиринги можно настраивать в любом порядке, главное, выполнять их конфигурацию по очереди. Дополнительную информацию о доменах маршрутизации и пиринге см. в статье [Каналы ExpressRoute и домены маршрутизации](expressroute-circuit-peerings.md). Сведения об общедоступном пиринге см. в статье [Создание общедоступного пиринга ExpressRoute и управление им](about-public-peering.md).

В этом руководстве описано следующее:
> [!div class="checklist"]
> - настройка, обновление и удаление пиринга Майкрософт для канала;
> - настройка, обновление и удаление частного пиринга Azure для канала.

## <a name="prerequisites"></a>Предварительные требования

* Перед началом настройки просмотрите следующие страницы:
    * [Предварительные требования](expressroute-prerequisites.md) 
    * [Требования к маршрутизации](expressroute-routing.md)
    * [Рабочие процессы](expressroute-workflows.md)
* Вам потребуется активный канал ExpressRoute. Приступая к работе, [создайте канал ExpressRoute](expressroute-howto-circuit-arm.md). Его должен включить поставщик услуг подключения. Для выполнения командлетов, описанных в статье, нужно подготовить и включить канал ExpressRoute.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="microsoft-peering"></a><a name="msft"></a>Пиринг Майкрософт

Этот раздел поможет создать, получить, обновить и (или) удалить конфигурацию пиринга Майкрософт для канала ExpressRoute.

> [!IMPORTANT]
> Пиринг Майкрософт для каналов ExpressRoute, которые были настроены до 1 августа 2017 г., позволяет объявлять все префиксы служб, даже если фильтры маршрутов не определены. Пиринг Майкрософт для каналов ExpressRoute, настроенных 1 августа 2017 г. или позднее, не будет объявлять префиксы, пока к каналу не будет присоединен фильтр маршрутов. Дополнительные сведения см. в руководстве по [настройке фильтра маршрута для пиринга Майкрософт](how-to-routefilter-powershell.md).
> 

### <a name="to-create-microsoft-peering"></a>Создание пиринга Майкрософт

1. Выполните вход и выберите подписку.

   Если вы установили PowerShell на локальном компьютере, выполните вход. Если вы используете Azure Cloud Shell, этот шаг можно пропустить.

   ```azurepowershell
   Connect-AzAccount
   ```

   Выберите подписку для создания канала ExpressRoute.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionId "<subscription ID>"
   ```
2. Создайте канал ExpressRoute.

   Выполните инструкции по созданию [канала ExpressRoute](expressroute-howto-circuit-arm.md). Поставщик услуг подключения должен подготовить его. Если поставщик услуг подключения оказывает услуги третьего уровня, он может включить для вас пиринг Майкрософт. Инструкции в следующих разделах выполнять не нужно. Если же поставщик услуг подключения не управляет маршрутизацией за вас, после создания канала сделайте следующее. 

3. Убедитесь, что канал ExpressRoute подготовлен и включен. Используйте следующий пример:

   ```azurepowershell-interactive
   Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```

   Ответ будет выглядеть примерно так:

   ```
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
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
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
   ServiceKey                       : **************************************
   Peerings                         : []
   ```
4. Настройте пиринг Майкрософт для этого канала. Прежде чем продолжить, убедитесь, что у вас есть следующие сведения.

   * Подсеть /30 или /126 для основной ссылки. Блок адресов должен быть представлен префиксом допустимого общедоступного адреса IPv4 или IPv6, принадлежащего вам и зарегистрированного в RIR/IRR.
   * Подсеть /30 или /126 для дополнительной ссылки. Блок адресов должен быть представлен префиксом допустимого общедоступного адреса IPv4 или IPv6, принадлежащего вам и зарегистрированного в RIR/IRR.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS.
   * Объявленные префиксы: предоставьте список всех префиксов, которые вы планируете объявить во время сеанса BGP. Допускаются только общедоступные префиксы IP-адресов. Если вы планируете отправить набор префиксов, их можно оформить в виде списка, разделенного запятыми. Эти префиксы должны быть зарегистрированы в RIR/IRR на ваше имя. Для сеансов BGP IPv4 требуются объявленные префиксы IPv4, а для сеансов BGP IPv6 — объявленные префиксы IPv6. 
   * Имя реестра маршрутизации: можно указать RIR/IRR, в котором зарегистрированы номер AS и префиксы.
   * Необязательное действие:
     * Клиент ASN: для объявления префиксов, не зарегистрированных с номером AS для пиринга, можно указать номер AS, с которым они зарегистрированы.
     * Хэш MD5, если вы решите его использовать.

> [!IMPORTANT]
> Корпорация Майкрософт проверяет, назначены ли вам указанные объявленные открытые префиксы (Advertised public prefixes) и одноранговый узел ASN (Peer ASN) или ASN клиента (Customer ASN) в реестре маршрутизации в Интернете. Если вы получаете открытые префиксы из другой сущности и если назначение не записано в реестре маршрутизации, автоматическая проверка не будет завершена и потребуется проверка вручную. Если автоматическая проверка не удалась, вы увидите для AdvertisedPublicPrefixesState значение Validation needed (Требуется проверка) в выходных данных Get-AzExpressRouteCircuitPeeringConfig (см. раздел "Получение сведений о пиринге Майкрософт" ниже). 
> 
> Если вы видите это сообщение, соберите документы, подтверждающие, что открытые префиксы назначены вашей организации сущностью, указанной в качестве владельца префиксов в реестре маршрутизации. Отправьте эти документы для проверки вручную, открыв запрос в службу поддержки. 
> 

   Чтобы настроить пиринг Майкрософт для своего канала, воспользуйтесь следующим примером:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv4 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "123.1.0.0/24" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

   Add-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv6 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "3FFE:FFFF:0:CD31::/120" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

### <a name="to-get-microsoft-peering-details"></a><a name="getmsft"></a>Получение сведений о пиринге Майкрософт

Для получения сведений о конфигурации можно использовать следующий пример:

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

Get-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt
```

### <a name="to-update-microsoft-peering-configuration"></a><a name="updatemsft"></a>Обновление конфигурации пиринга Майкрософт

С помощью следующего примера можно обновить любую часть конфигурации:

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig  -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv4 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "124.1.0.0/24" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

Set-AzExpressRouteCircuitPeeringConfig  -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv6 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "3FFE:FFFF:0:CD31::/120" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="azure-private-peering"></a><a name="private"></a>Частный пиринг Azure

Этот раздел поможет вам создать, получить, обновить и (или) удалить конфигурацию частного пиринга Azure для канала ExpressRoute.

> [!IMPORTANT]
> Поддержка IPv6 для частного пиринга сейчас предоставляется в **общедоступной предварительной версии**. 
> 
> 

### <a name="to-create-azure-private-peering"></a>Создание частного пиринга Azure

1. Импортируйте модуль PowerShell для ExpressRoute.

   Установите последнюю версию установщика PowerShell из [коллекции PowerShell](https://www.powershellgallery.com/). Затем импортируйте модули Azure Resource Manager в сеанс PowerShell, чтобы начать использовать командлеты ExpressRoute. Вам нужно будет запустить PowerShell от имени администратора.

   ```azurepowershell-interactive
   Install-Module Az
   ```

   Импортируйте все модули Az.\* в рамках диапазона известных семантических версий.

   ```azurepowershell-interactive
   Import-Module Az
   ```

   Можно также просто импортировать выбранный модуль в рамках диапазона известных семантических версий.

   ```azurepowershell-interactive
   Import-Module Az.Network 
   ```

   Войдите в свою учетную запись.

   ```azurepowershell-interactive
   Connect-AzAccount
   ```

   Выберите подписку для создания канала ExpressRoute.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionId "<subscription ID>"
   ```
2. Создайте канал ExpressRoute.

   Выполните инструкции по созданию [канала ExpressRoute](expressroute-howto-circuit-arm.md). Поставщик услуг подключения должен подготовить его. Если поставщик услуг подключения предоставляет управляемые службы уровня 3, он может включить для вас частный пиринг Azure. Инструкции в следующих разделах выполнять не нужно. Если же поставщик услуг подключения не управляет маршрутизацией за вас, после создания канала сделайте следующее.

3. Убедитесь, что канал ExpressRoute подготовлен и включен. Используйте следующий пример:

   ```azurepowershell-interactive
   Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```

   Ответ будет выглядеть примерно так:

   ```
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
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
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
   ServiceKey                       : **************************************
   Peerings                         : []
   ```
4. Настройте для канала частный пиринг Azure. Прежде чем продолжить, убедитесь в наличии следующих элементов:

   * Пара подсетей, которые не являются частью какого-либо диапазона адресов, зарезервированного для виртуальных сетей. Одна подсеть будет использоваться в качестве основной ссылки, а вторая — в качестве дополнительной. Из каждой подсети вы назначите первый готовый к применению IP-адрес для своего маршрутизатора, так как корпорация Майкрософт использует второй IP-адрес для маршрутизатора Майкрософт. Существует три варианта пары подсетей:
       * IPv4: две подсети /30;
       * IPv6: две подсети /126;
       * Оба: две подсети /30 и две подсети /126.
   * Действительный идентификатор виртуальной локальной сети для установки пиринга. Идентификатор не должен использоваться ни одним другим пирингом в канале.
   * Номер AS для пиринга. Можно использовать 2-байтовые и 4-байтовые номера AS. Для этого пиринга можно использовать частный номер AS. Убедитесь, что вы не используете 65515.
   * Необязательное действие:
     * Хэш MD5, если вы решите его использовать.

   Чтобы настроить для канала частный пиринг Azure, выполните код из следующего примера:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200

   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 200 -PeerAddressType IPv6

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   Если вы решили использовать MD5-хэш, используйте следующий пример:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200  -SharedKey "A1B2C3D4"

   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 200 -PeerAddressType IPv6 -SharedKey "A1B2C3D4"
   ```

   > [!IMPORTANT]
   > Номер AS должен быть указан в качестве ASN пиринга, а не ASN клиента.
   > 
   >

### <a name="to-get-azure-private-peering-details"></a><a name="getprivate"></a>Получение сведений о частном пиринге Azure

Для получения сведений о конфигурации можно использовать следующий пример кода:

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
```

### <a name="to-update-azure-private-peering-configuration"></a><a name="updateprivate"></a>Обновление конфигурации частного пиринга Azure

С помощью следующего примера можно обновить любую часть конфигурации. В следующем примере значение идентификатора виртуальной локальной сети для канала изменяется с 200 на 500:

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 500

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="clean-up-resources"></a>Очистка ресурсов

### <a name="to-delete-microsoft-peering"></a><a name="deletemsft"></a>Удаление пиринга Майкрософт

Для удаления конфигурации пиринга выполните следующий командлет.

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-delete-azure-private-peering"></a><a name="deleteprivate"></a>Удаление частного пиринга Azure

Для удаления конфигурации пиринга выполните следующий пример кода:

> [!WARNING]
> Перед запуском этого примера обязательно убедитесь, что все виртуальные сети и подключения Global Reach к ExpressRoute удалены. 
> 

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="next-steps"></a>Дальнейшие шаги

После настройки частного пиринга Azure можно создать шлюз ExpressRoute, чтобы связать виртуальную сеть с каналом. 

> [!div class="nextstepaction"]
> [Настройка шлюза виртуальной сети для ExpressRoute](expressroute-howto-add-gateway-resource-manager.md)
