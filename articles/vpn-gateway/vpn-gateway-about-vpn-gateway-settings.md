---
title: 'VPN-шлюз Azure: параметры конфигурации'
description: Сведения о ресурсах и параметрах VPN-шлюза для виртуальной сети, созданной в модели развертывания диспетчер ресурсов.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 10/21/2020
ms.author: cherylmc
ms.openlocfilehash: 1aba87b2139fb8a7d395fb3180d2074e47310fa9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96010877"
---
# <a name="about-vpn-gateway-configuration-settings"></a>Сведения о параметрах конфигурации VPN-шлюза

VPN-шлюз — это разновидность шлюза виртуальной сети, который передает зашифрованный трафик между виртуальной сетью и локальным расположением через общедоступное подключение. VPN-шлюз можно также использовать для передачи трафика между виртуальными сетями в магистрали Azure.

Подключение VPN-шлюза зависит от конфигурации нескольких ресурсов, каждый из которых содержит настраиваемые параметры. В разделах этой статьи рассматриваются ресурсы и параметры, относящиеся к VPN-шлюзу для виртуальной сети, созданной на основе модели развертывания с помощью Resource Manager. Описания и схемы топологий для каждого варианта подключения можно найти в статье [Основные сведения о VPN-шлюзах Azure](vpn-gateway-about-vpngateways.md).

Значения в этой статье применяются к шлюзам VPN (шлюзам виртуальной сети, использующим -GatewayType Vpn). В этой статье рассматриваются не все типы шлюзов и избыточных в пределах зоны шлюзов.

* Значения, которые применяются к -GatewayType ExpressRoute, см. в статье [Сведения о шлюзах виртуальных сетей ExpressRoute](../expressroute/expressroute-about-virtual-network-gateways.md).

* Сведения об избыточных в пределах зоны шлюзах см. в разделе [Избыточные в пределах зоны шлюзы](about-zone-redundant-vnet-gateways.md).

* Сведения о виртуальной глобальной сети см. в разделе [Виртуальная глобальная сеть](../virtual-wan/virtual-wan-about.md).

## <a name="gateway-types"></a><a name="gwtype"></a>Типы шлюзов

У каждой виртуальной сети может быть только один шлюз виртуальной сети каждого типа. При создании шлюза виртуальной сети необходимо убедиться, что в конфигурации указан правильный тип шлюза.

Для -GatewayType доступны приведенные ниже значения.

* Vpn
* ExpressRoute

Для VPN-шлюза требуется `-GatewayType` *Vpn*.

Пример

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased
```

## <a name="gateway-skus"></a><a name="gwsku"></a>SKU шлюзов

[!INCLUDE [vpn-gateway-gwsku-include](../../includes/vpn-gateway-gwsku-include.md)]

### <a name="configure-a-gateway-sku"></a>Настройка номера SKU для шлюза

**Портал Azure**

Если для создания шлюза виртуальной сети Resource Manager используется портал Azure, выбрать SKU шлюза можно в раскрывающемся списке. Представленные параметры соответствуют выбранным типу шлюза и типу VPN.

**PowerShell**

В следующем примере PowerShell используется `-GatewaySku` со значением VpnGw1. Чтобы создать шлюз с помощью PowerShell, сначала создайте IP-конфигурацию, а затем для ссылки на нее используйте переменную. В этом примере переменная конфигурации — $gwipconfig.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name VNet1GW -ResourceGroupName TestRG1 `
-Location 'US East' -IpConfigurations $gwipconfig -GatewaySku VpnGw1 `
-GatewayType Vpn -VpnType RouteBased
```

**Azure CLI**

```azurecli
az network vnet-gateway create --name VNet1GW --public-ip-address VNet1GWPIP --resource-group TestRG1 --vnet VNet1 --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1 --no-wait
```

###  <a name="resizing-or-changing-a-sku"></a><a name="resizechange"></a>Изменение размера номера SKU или переход на другой номер SKU

Если у вас есть VPN-шлюз и вы хотите использовать другой номер SKU шлюза, можно либо изменить размер SKU шлюза, или перейти на другой номер SKU. При переходе на другой номер SKU шлюза существующий шлюз полностью удаляется и создается новый. Создание шлюза может занять до 45 минут. В сравнении, при изменении размера SKU шлюза немало простоев, поскольку нет необходимости удалять и перестраивать шлюз. Если у вас есть возможность изменить размер SKU шлюза, а не перейти на другой номер SKU, выберите именно этот вариант. Но для изменения размера есть определенные правила:

1. За исключением номера SKU уровня "базовый" можно изменить размер SKU VPN-шлюза на другой SKU VPN-шлюза в том же выколении (Generation1 или Generation2). Например, можно изменить размер VpnGw1 Generation1 до VpnGw2 Generation1, но не VpnGw2 Generation2.
2. Для номеров SKU предыдущих версий можно изменять размер, выбирая между следующими вариантами: Basic, Standard и HighPerformance.
3. **Нельзя** изменить размер из номеров SKU Basic/Standard/HighPerformance на VpnGw SKU. Вместо этого нужно [изменить](#change) номера SKU на новые.

#### <a name="to-resize-a-gateway"></a><a name="resizegwsku"></a>Изменение размера шлюза

**Портал Azure**

[!INCLUDE [Resize a SKU - portal](../../includes/vpn-gateway-resize-gw-portal-include.md)]

**PowerShell**

[!INCLUDE [Resize a SKU](../../includes/vpn-gateway-gwsku-resize-include.md)]

####  <a name="to-change-from-an-old-legacy-sku-to-a-new-sku"></a><a name="change"></a>Изменение прежнего (устаревшего) номера SKU на новый

[!INCLUDE [Change a SKU](../../includes/vpn-gateway-gwsku-change-legacy-sku-include.md)]

## <a name="connection-types"></a><a name="connectiontype"></a>Типы подключений

В модели развертывания с помощью Resource Manager каждой конфигурации необходим определенный тип подключения шлюза виртуальной сети. Для `-ConnectionType` в PowerShell можно указать такие значения (развертывание Resource Manager):

* IPsec
* Vnet2Vnet
* ExpressRoute
* VPNClient

В следующем примере PowerShell создается подключение типа "сеть — сеть", для которого требуется тип *IPsec*.

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name localtovon -ResourceGroupName testrg `
-Location 'West US' -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
-ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'
```

## <a name="vpn-types"></a><a name="vpntype"></a>Типы VPN

При создании шлюза виртуальной сети для конфигурации VPN-шлюза необходимо указать тип VPN. Выбор типа VPN зависит от топологии подключений, которую вы хотите создать. Например, для подключения типа "точка — сеть" требуется тип VPN на основе маршрутов. Тип VPN может также зависеть от используемого оборудования. Для конфигураций "сеть — сеть" требуется VPN-устройство. Некоторые VPN-устройства поддерживают только определенный тип VPN.

Выбранный тип VPN должен удовлетворять всем требованиям к подключению решения, которое вы хотите создать. Например, если вы хотите создать подключение VPN-шлюза типа "сеть — сеть" и подключение VPN-шлюза типа "точка — сеть" для одной и той же виртуальной сети, то вам следует использовать тип VPN *RouteBased* , так как он необходим для подключения типа "точка — сеть". Необходимо также проверить, поддерживает ли ваше VPN-устройство VPN-подключение на основе маршрутов. 

После создания шлюза виртуальной сети изменить тип VPN невозможно. Для этого потребуется удалить шлюз виртуальной сети и создать новый. Существуют два типа VPN:

[!INCLUDE [vpn-gateway-vpntype](../../includes/vpn-gateway-vpntype-include.md)]

В следующем примере PowerShell используется `-VpnType` со значением *RouteBased*. При создании шлюза необходимо убедиться, что в конфигурации указано правильное значение -VpnType.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg `
-Location 'West US' -IpConfigurations $gwipconfig `
-GatewayType Vpn -VpnType RouteBased
```

## <a name="gateway-requirements"></a><a name="requirements"></a>Требования к шлюзу

[!INCLUDE [vpn-gateway-table-requirements](../../includes/vpn-gateway-table-requirements-include.md)]

## <a name="gateway-subnet"></a><a name="gwsub"></a>Подсеть шлюза

Прежде чем создать VPN-шлюз, необходимо создать подсеть для него. Подсеть шлюза содержит IP-адреса, которые используют виртуальные машины и службы шлюза виртуальной сети. При создании шлюза виртуальной сети его виртуальные машины развертываются в подсети шлюза и на них настраиваются необходимые параметры VPN-шлюза. Никогда не развертывайте все остальное (например, дополнительные виртуальные машины) в подсети шлюза. Чтобы подсеть шлюза работала правильно, ее нужно назвать GatewaySubnet. Такое имя позволяет Azure определить, что это подсеть для развертывания виртуальных машин и служб шлюза виртуальной сети.

>[!NOTE]
>[!INCLUDE [vpn-gateway-gwudr-warning.md](../../includes/vpn-gateway-gwudr-warning.md)]
>

При создании подсети шлюза указывается количество IP-адресов, которое содержит подсеть. IP-адреса в подсети шлюза выделяются виртуальным машинам и службам шлюза. Некоторым конфигурациям требуется больше IP-адресов, чем прочим. 

При планировании размера подсети шлюза обратитесь к документации по конфигурации, которую вы планируете создать. Например, для конфигурации с сосуществованием шлюза ExpressRoute или VPN требуется более крупная подсеть шлюза, чем большинство других конфигураций. Кроме того, необходимо убедиться, что подсеть шлюза содержит достаточно IP-адресов для дополнительных конфигураций, которые могут быть добавлены в будущем. Хотя вы можете создать подсеть шлюза размером/29, мы рекомендуем создать подсеть шлюза размером/27 или больше (/27,/26 и т. д.), если у вас есть доступное адресное пространство. Это будет соответствовать большинству конфигураций.

В примере PowerShell для Resource Manager ниже показана подсеть шлюза GatewaySubnet. Здесь приведена нотация CIDR, указывающая размер /27, при котором можно использовать достаточно IP-адресов для большинства существующих конфигураций.

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/27
```

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="local-network-gateways"></a><a name="lng"></a>Локальные сетевые шлюзы

Локальный сетевой шлюз отличается от шлюза виртуальной сети. При создании конфигурации VPN-шлюза локальный сетевой шлюз обычно представляет локальную сеть и соответствующее VPN-устройство. В классической модели развертывания шлюз локальной сети называется "локальным сайтом".

Вы присвойте шлюзу локальной сети имя, общедоступный IP-адрес или полное доменное имя (FQDN) локального VPN-устройства и укажите префиксы адресов, которые находятся в локальном расположении. Azure проверяет наличие сетевого трафика по префиксам адресов назначения, учитывает конфигурацию, указанную для локального сетевого шлюза, и соответствующим образом направляет пакеты. Если на VPN-устройстве используется протокол BGP (BGP), вы получите одноранговый IP-адрес устройства VPN и номер автономной системы (ASN) локальной сети. Можно также указать шлюзы локальной сети для конфигураций типа "виртуальная сеть — виртуальная сеть", использующих подключение к VPN-шлюзу.

Следующий пример PowerShell создает шлюз локальной сети.

```azurepowershell-interactive
New-AzLocalNetworkGateway -Name LocalSite -ResourceGroupName testrg `
-Location 'West US' -GatewayIpAddress '23.99.221.164' -AddressPrefix '10.5.51.0/24'
```

Иногда возникает необходимость изменить параметры локального сетевого шлюза. Например, при добавлении или изменении диапазона адресов, либо при изменении IP-адреса VPN-устройства. См. статью [Изменение параметров шлюза локальной сети с помощью PowerShell](vpn-gateway-modify-local-network-gateway.md).

## <a name="rest-apis-powershell-cmdlets-and-cli"></a><a name="resources"></a>Интерфейсы REST API, командлеты PowerShell и CLI

Дополнительные технические материалы и специальные требования к синтаксису, действующие при использовании интерфейсов REST API, командлетов PowerShell или Azure CLI для настройки конфигураций VPN-шлюзов, доступны на приведенных ниже страницах.

| **Классический** | **Resource Manager** |
| --- | --- |
| [PowerShell](/powershell/module/az.network/#networking) |[PowerShell](/powershell/module/az.network#vpn) |
| [REST API](/previous-versions/azure/reference/jj154113(v=azure.100)) |[REST API](/rest/api/network/virtualnetworkgateways) |
| Не поддерживается | [Azure CLI](/cli/azure/network/vnet-gateway)|

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о доступных конфигурациях подключений см. в статье [Основные сведения о VPN-шлюзах Azure](vpn-gateway-about-vpngateways.md).