---
title: Настройка подключений VPN-шлюза Azure типа "активный — активный"
description: В этой статье описана поэтапная настройка подключений в режиме "активный — активный" для VPN-шлюзов Azure с помощью Azure Resource Manager и PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: yushwang
ms.reviewer: cherylmc
ms.openlocfilehash: 022ccaab0b210cd2d656b69f505791d1a2aa963f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89440785"
---
# <a name="configure-active-active-s2s-vpn-connections-with-azure-vpn-gateways"></a>Настройка VPN-подключений типа "сеть — сеть" в режиме "активный — активный" для VPN-шлюзов Azure

В этой статье содержится пошаговое описание процесса, который позволит создать подключения между локальными и виртуальными сетями в режиме "активный — активный" с помощью модели развертывания Resource Manager и PowerShell. Также можно настроить шлюз "активный — активный" в портал Azure.

## <a name="about-highly-available-cross-premises-connections"></a>О высокодоступных распределенных подключениях
Чтобы добиться высокого уровня доступности для подключений между локальными и виртуальными сетями, следует развернуть несколько VPN-шлюзов и установить несколько параллельных подключений между сетями и Azure. Сведения о вариантах подключения и топологии подключений см. в статье [Настройка высокодоступных подключений: распределенных и между виртуальными сетями](vpn-gateway-highlyavailable.md).

Эта статья содержит инструкции по настройке распределенного VPN-подключения в режиме "активный — активный" и подключения в режиме "активный — активный" между двумя виртуальными сетями.

* [Часть 1. Создание и настройка VPN-шлюза Azure в режиме "активный — активный"](#aagateway)
* [Часть 2. Создание подключения между локальными сетями в режиме "активный — активный"](#aacrossprem)
* [Часть 3. Создание подключения между виртуальными сетями в режиме "активный — активный"](#aav2v)

Если у вас уже имеется VPN-шлюз, то вы можете:
* [Обновить существующий VPN-шлюз в режиме "активный — резервный", переведя его в режим "активный — активный" или наоборот.](#aaupdate)

Вы можете комбинировать эти блоки для создания более сложной, высокодоступной топологии сети в соответствии со своими задачами.

> [!IMPORTANT]
> Режим "активный — активный" доступен для всех номеров SKU, кроме Basic.

## <a name="part-1---create-and-configure-active-active-vpn-gateways"></a><a name ="aagateway"></a>Часть 1. Создание и настройка VPN-шлюзов в режиме "активный — активный"
Здесь приведены действия по настройке VPN-шлюза Azure в режиме "активный — активный". Между шлюзами в режиме "активный — активный" и "активный — резервный" существуют такие основные различия:

* Необходимо создать две конфигурации IP-адресов шлюза с двумя общедоступными IP-адресами.
* Требуется задать параметр EnableActiveActiveFeature.
* Номер SKU шлюза должен быть VpnGw1, VpnGw2, VpnGw3 или HighPerformance (устаревший SKU).

Другие свойства такие же, как у шлюзов не в режиме "активный — активный". 

### <a name="before-you-begin"></a>Перед началом
* Убедитесь в том, что у вас уже есть подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/).
* Если вы не хотите использовать CloudShell в браузере, необходимо установить командлеты PowerShell для Azure Resource Manager. Дополнительные сведения об установке командлетов PowerShell см. в разделе [Общие сведения об Azure PowerShell](/powershell/azure/).

### <a name="step-1---create-and-configure-vnet1"></a>Шаг 1. Создание и настройка VNet1
#### <a name="1-declare-your-variables"></a>1. Объявите переменные

В этом упражнении мы начнем с объявления переменных. Если вы используете Cloud Shell "попробовать", вы автоматически будете подключаться к вашей учетной записи. Если вы используете PowerShell локально, используйте следующий пример для подключения:

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
```

В примере ниже объявлены переменные со значениями для этого упражнения. Обязательно замените значения своими при настройке для рабочей среды. Эти переменные можно использовать для ознакомления с этим типом конфигурации. Измените переменные, а затем скопируйте и вставьте код в консоль PowerShell.

```azurepowershell-interactive
$Sub1 = "Ross"
$RG1 = "TestAARG1"
$Location1 = "West US"
$VNetName1 = "TestVNet1"
$FESubName1 = "FrontEnd"
$BESubName1 = "Backend"
$GWSubName1 = "GatewaySubnet"
$VNetPrefix11 = "10.11.0.0/16"
$VNetPrefix12 = "10.12.0.0/16"
$FESubPrefix1 = "10.11.0.0/24"
$BESubPrefix1 = "10.12.0.0/24"
$GWSubPrefix1 = "10.12.255.0/27"
$VNet1ASN = 65010
$DNS1 = "8.8.8.8"
$GWName1 = "VNet1GW"
$GW1IPName1 = "VNet1GWIP1"
$GW1IPName2 = "VNet1GWIP2"
$GW1IPconf1 = "gw1ipconf1"
$GW1IPconf2 = "gw1ipconf2"
$Connection12 = "VNet1toVNet2"
$Connection151 = "VNet1toSite5_1"
$Connection152 = "VNet1toSite5_2"
```

#### <a name="2-create-a-new-resource-group"></a>2. Создайте новую группу ресурсов.

Используйте приведенный ниже пример, чтобы создать новую группу ресурсов.

```azurepowershell-interactive
New-AzResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-testvnet1"></a>3. Создание TestVNet1
В примере ниже создается виртуальная сеть с именем TestVNet1 и три подсети: GatewaySubnet, FrontEnd и Backend. При замене значений важно, чтобы вы назвали подсеть шлюза именем GatewaySubnet. Если вы используете другое имя, создание шлюза завершится сбоем.

```azurepowershell-interactive
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
```

### <a name="step-2---create-the-vpn-gateway-for-testvnet1-with-active-active-mode"></a>Шаг 2. Создание VPN-шлюза для TestVNet1 в режиме "активный — активный"
#### <a name="1-create-the-public-ip-addresses-and-gateway-ip-configurations"></a>1. Создайте общедоступные IP-адреса и IP-конфигурации шлюза.
Запросите выделение двух общедоступных IP-адресов для шлюза, который будет создан для виртуальной сети. Также следует определить настройки подсети и IP-адресов.

```azurepowershell-interactive
$gw1pip1 = New-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$gw1pip2 = New-AzPublicIpAddress -Name $GW1IPName2 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic

$vnet1 = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gw1ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1
$gw1ipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf2 -Subnet $subnet1 -PublicIpAddress $gw1pip2
```

#### <a name="2-create-the-vpn-gateway-with-active-active-configuration"></a>2. Создание VPN-шлюза с конфигурацией "активный — активный"
Создайте шлюз для виртуальной сети TestVNet1. Обратите внимание, что есть две записи GatewayIpConfig и задан параметр EnableActiveActiveFeature. Создание шлюза может занять некоторое время (45 минут или более).

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1,$gw1ipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN -EnableActiveActiveFeature -Debug
```

#### <a name="3-obtain-the-gateway-public-ip-addresses-and-the-bgp-peer-ip-address"></a>3. получите общедоступные IP-адреса шлюза и IP-адрес узла BGP
После создания шлюза необходимо получить IP-адрес для узла Azure BGP на VPN-шлюзе Azure. Этот адрес нужен, чтобы VPN-шлюз Azure мог выполнять роль узла BGP для локальных VPN-устройств.

```azurepowershell-interactive
$gw1pip1 = Get-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1
$gw1pip2 = Get-AzPublicIpAddress -Name $GW1IPName2 -ResourceGroupName $RG1
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
```

Чтобы отобразить два общедоступных IP-адреса, выделенных для VPN-шлюза, и их соответствующие IP-адреса узла BGP для каждого экземпляра шлюза, используйте следующие командлеты:

```azurepowershell-interactive
PS D:\> $gw1pip1.IpAddress
40.112.190.5

PS D:\> $gw1pip2.IpAddress
138.91.156.129

PS D:\> $vnet1gw.BgpSettingsText
{
  "Asn": 65010,
  "BgpPeeringAddress": "10.12.255.4,10.12.255.5",
  "PeerWeight": 0
}
```

Порядок общедоступных IP-адресов для экземпляров шлюза и соответствующих адресов пиринга BGP одинаковый. В приведенном примере виртуальная машина шлюза с общедоступным IP-адресом 40.112.190.5 использует адрес 10.12.255.4 в качестве адреса пиринга BGP, а шлюз с адресом 138.91.156.129 использует адрес 10.12.255.5. Эти сведения необходимы при настройке локальных VPN-устройств, которые подключаются к шлюзу в режиме "активный — активный". На схеме ниже показан шлюз вместе со всеми адресами.

![Шлюз в режиме "активный — активный"](./media/vpn-gateway-activeactive-rm-powershell/active-active-gw.png)

С помощью созданного шлюза можно установить подключение в режиме "активный — активный" между локальными или виртуальными сетями. В следующих разделах описано, как это сделать.

## <a name="part-2---establish-an-active-active-cross-premises-connection"></a><a name ="aacrossprem"></a>Часть 2. Создание подключения между локальными сетями в режиме "активный — активный"
Чтобы установить подключение между локальными сетями, нужно создать локальный сетевой шлюз, который будет представлять локальное VPN-устройство, а также подключение между VPN-шлюзом Azure и шлюзом локальной сети. В этом примере VPN-шлюз Azure находится в режиме "активный — активный". Таким образом, несмотря на то, что есть только одно локальное VPN-устройство (шлюз локальной сети) и один ресурс подключения, оба экземпляра VPN-шлюза Azure установят VPN-туннели типа "сеть — сеть" с локального устройства.

Прежде чем продолжить, убедитесь, что вы выполнили инструкции из [первой части](#aagateway) этой статьи.

### <a name="step-1---create-and-configure-the-local-network-gateway"></a>Шаг 1. Создание и настройка локального сетевого шлюза
#### <a name="1-declare-your-variables"></a>1. Объявите переменные
В этом упражнении мы продолжим создание конфигурации, которая представлена на схеме. Не забудьте заменить значения теми, которые вы хотите использовать для конфигурации.

```azurepowershell-interactive
$RG5 = "TestAARG5"
$Location5 = "West US"
$LNGName51 = "Site5_1"
$LNGPrefix51 = "10.52.255.253/32"
$LNGIP51 = "131.107.72.22"
$LNGASN5 = 65050
$BGPPeerIP51 = "10.52.255.253"
```

Несколько важных замечаний о параметрах локального сетевого шлюза.

* Локальный сетевой шлюз может находиться в том же расположении, что и VPN-шлюз, или в любом другом. Это же справедливо в отношении групп ресурсов. В этом примере они представлены в разных группах ресурсов, но в одном и том же расположении Azure.
* Если, как показано выше, есть только одно локальное VPN-устройство, подключение в режиме "активный — активный" может работать как с протоколом BGP, так и без него. В этом примере для подключения между локальными сетями используется BGP.
* Если включен BGP, то префикс, который необходимо объявить для шлюза локальной сети — это IP-адрес узла BGP на VPN-устройстве. В нашем примере это префикс /32 для адреса 10.52.255.253/32.
* Не забывайте, что для локальных сетей и виртуальной сети Azure должны быть указаны разные номера ASN BGP. Если они совпадают, а локальное VPN-устройство уже использует свой ASN для связи с другими соседями BGP, необходимо изменить ASN вашей виртуальной сети.

#### <a name="2-create-the-local-network-gateway-for-site5"></a>2. Создание шлюза локальной сети для site5
Прежде чем продолжить, убедитесь, что вы все еще подключены к подписке 1. Создайте группу ресурсов, если она еще не создана.

```azurepowershell-interactive
New-AzResourceGroup -Name $RG5 -Location $Location5
New-AzLocalNetworkGateway -Name $LNGName51 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP51 -AddressPrefix $LNGPrefix51 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP51
```

### <a name="step-2---connect-the-vnet-gateway-and-local-network-gateway"></a>Шаг 2. Подключение шлюза виртуальной сети к локальному сетевому шлюзу
#### <a name="1-get-the-two-gateways"></a>1. получение двух шлюзов

```azurepowershell-interactive
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw1 = Get-AzLocalNetworkGateway  -Name $LNGName51 -ResourceGroupName $RG5
```

#### <a name="2-create-the-testvnet1-to-site5-connection"></a>2. Создание подключения TestVNet1 к site5
На этом шаге создается подключение между TestVNet1 и Site5_1, при этом параметру EnableBGP задано значение $True.

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name $Connection151 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw1 -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

#### <a name="3-vpn-and-bgp-parameters-for-your-on-premises-vpn-device"></a>3. параметры VPN и BGP для локального VPN-устройства
В приведенном ниже примере перечислены параметры, которые следует ввести в разделе конфигурации BGP на локальном VPN-устройстве для нашего тестового задания.

```
- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.253
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP 1  : 10.12.255.4 for tunnel to 40.112.190.5
- Azure VNet BGP IP 2  : 10.12.255.5 for tunnel to 138.91.156.129
- Static routes        : Destination 10.12.255.4/32, nexthop the VPN tunnel interface to 40.112.190.5
                         Destination 10.12.255.5/32, nexthop the VPN tunnel interface to 138.91.156.129
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

Подключение будет установлено через несколько минут. Сразу после создания подключения IPsec начнется сессия пиринга BGP. В этом примере настроено только одно локальное VPN-устройство, как показано на схеме ниже.

![Подключение между локальными сетями в режиме "активный — активный"](./media/vpn-gateway-activeactive-rm-powershell/active-active.png)

### <a name="step-3---connect-two-on-premises-vpn-devices-to-the-active-active-vpn-gateway"></a>Шаг 3. Подключение двух локальных VPN-устройств к VPN-шлюзу в режиме "активный — активный"
При наличии двух VPN-устройств в одной локальной сети можно добиться двойной избыточности, подключив VPN-шлюз Azure ко второму VPN-устройству.

#### <a name="1-create-the-second-local-network-gateway-for-site5"></a>1. Создайте второй шлюз локальной сети для site5.
IP-адрес шлюза, префикс адреса и адрес пиринга BGP для второго шлюза локальной сети не должны перекрываться с предыдущим шлюзом локальной сети для одной локальной сети.

```azurepowershell-interactive
$LNGName52 = "Site5_2"
$LNGPrefix52 = "10.52.255.254/32"
$LNGIP52 = "131.107.72.23"
$BGPPeerIP52 = "10.52.255.254"
```

```azurepowershell-interactive
New-AzLocalNetworkGateway -Name $LNGName52 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP52 -AddressPrefix $LNGPrefix52 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP52
```

#### <a name="2-connect-the-vnet-gateway-and-the-second-local-network-gateway"></a>2. Подключите шлюз виртуальной сети и второй локальный сетевой шлюз.
Создайте подключение между TestVNet1 и Site5_2, где параметру EnableBGP задано значение $True.

```azurepowershell-interactive
$lng5gw2 = Get-AzLocalNetworkGateway -Name $LNGName52 -ResourceGroupName $RG5
```

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name $Connection152 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw2 -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

#### <a name="3-vpn-and-bgp-parameters-for-your-second-on-premises-vpn-device"></a>3. параметры VPN и BGP для второго локального VPN-устройства
Аналогичным образом ниже перечислены параметры, которые нужно ввести на втором VPN-устройстве.

```
- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.254
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP 1  : 10.12.255.4 for tunnel to 40.112.190.5
- Azure VNet BGP IP 2  : 10.12.255.5 for tunnel to 138.91.156.129
- Static routes        : Destination 10.12.255.4/32, nexthop the VPN tunnel interface to 40.112.190.5
                         Destination 10.12.255.5/32, nexthop the VPN tunnel interface to 138.91.156.129
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

После того, как подключение (туннели) установится, VPN-устройства с двойной избыточностью и туннели подключатся к локальной сети и Azure.

![Подключение между локальными сетями с двойной избыточностью](./media/vpn-gateway-activeactive-rm-powershell/dual-redundancy.png)

## <a name="part-3---establish-an-active-active-vnet-to-vnet-connection"></a><a name ="aav2v"></a>Часть 3. Создание подключения между виртуальными сетями в режиме "активный — активный"
В этом разделе описано, как создать подключение между виртуальными сетями в режиме "активный — активный" с использованием BGP. 

Приведенные ниже инструкции продолжают действия, описанные выше. Чтобы создать и настроить сеть TestVNet1 и VPN-шлюз с использованием BGP, сначала следует выполнить инструкции из [первой части](#aagateway) . 

### <a name="step-1---create-testvnet2-and-the-vpn-gateway"></a>Шаг 1. Создание TestVNet2 и VPN-шлюза
Важно убедиться в том, что пространство IP-адресов для новой виртуальной сети TestVNet2 не пересекается с диапазонами любой из ваших виртуальных сетей.

В этом примере виртуальные сети относятся к одной подписке. Вы можете создавать подключения и между виртуальными сетями из разных подписок. Дополнительные сведения об этом в статье [Настройка подключения между виртуальными сетями в развертывании Resource Manager с помощью PowerShell](vpn-gateway-vnet-vnet-rm-ps.md). Чтобы использовать для подключения протокол BGP, обязательно укажите параметр -EnableBgp True при создании подключения.

#### <a name="1-declare-your-variables"></a>1. Объявите переменные
Не забудьте заменить значения теми, которые вы хотите использовать для конфигурации.

```azurepowershell-interactive
$RG2 = "TestAARG2"
$Location2 = "East US"
$VNetName2 = "TestVNet2"
$FESubName2 = "FrontEnd"
$BESubName2 = "Backend"
$GWSubName2 = "GatewaySubnet"
$VNetPrefix21 = "10.21.0.0/16"
$VNetPrefix22 = "10.22.0.0/16"
$FESubPrefix2 = "10.21.0.0/24"
$BESubPrefix2 = "10.22.0.0/24"
$GWSubPrefix2 = "10.22.255.0/27"
$VNet2ASN = 65020
$DNS2 = "8.8.8.8"
$GWName2 = "VNet2GW"
$GW2IPName1 = "VNet2GWIP1"
$GW2IPconf1 = "gw2ipconf1"
$GW2IPName2 = "VNet2GWIP2"
$GW2IPconf2 = "gw2ipconf2"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2. Создайте TestVNet2 в новой группе ресурсов.

```azurepowershell-interactive
New-AzResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2
```

#### <a name="3-create-the-active-active-vpn-gateway-for-testvnet2"></a>3. Создание VPN-шлюза "активный — активный" для TestVNet2
Запросите выделение двух общедоступных IP-адресов для шлюза, который будет создан для виртуальной сети. Также следует определить настройки подсети и IP-адресов.

```azurepowershell-interactive
$gw2pip1 = New-AzPublicIpAddress -Name $GW2IPName1 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic
$gw2pip2 = New-AzPublicIpAddress -Name $GW2IPName2 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic

$vnet2 = Get-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gw2ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW2IPconf1 -Subnet $subnet2 -PublicIpAddress $gw2pip1
$gw2ipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GW2IPconf2 -Subnet $subnet2 -PublicIpAddress $gw2pip2
```

Создайте VPN-шлюз с номером AS и включенным параметром EnableActiveActiveFeature. Обратите внимание, что нужно переназначить номер ASN по умолчанию для ваших VPN-шлюзов Azure. Номера ASN для подключенных виртуальных сетей должны быть разными, чтобы работал протокол BGP и транзитная маршрутизация.

```azurepowershell-interactive
New-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gw2ipconf1,$gw2ipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet2ASN -EnableActiveActiveFeature
```

### <a name="step-2---connect-the-testvnet1-and-testvnet2-gateways"></a>Шаг 2. Подключение шлюзов TestVNet1 и TestVNet2
В этом примере оба шлюза находятся в одной подписке. Этот шаг можно выполнить в одном сеансе PowerShell.

#### <a name="1-get-both-gateways"></a>1. получите оба шлюза.
Обязательно войдите и подключитесь к подписке 1.

```azurepowershell-interactive
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet2gw = Get-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2
```

#### <a name="2-create-both-connections"></a>2. Создайте оба подключения.
На этом шаге вы создадите подключение от TestVNet1 к TestVNet2, а также подключение от TestVNet2 к TestVNet1.

```azurepowershell-interactive
New-AzVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True

New-AzVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True
```

> [!IMPORTANT]
> Не забудьте включить BGP для ОБОИХ подключений.
> 
> 

Подключение будет установлено через несколько минут после выполнения этих шагов, а после создания подключения между виртуальными сетями с двойной избыточностью запустится сеанс пиринга BGP.

![Подключение между виртуальными сетями в режиме "активный — активный"](./media/vpn-gateway-activeactive-rm-powershell/vnet-to-vnet.png)

## <a name="update-an-existing-vpn-gateway"></a><a name ="aaupdate"></a>Обновление существующего VPN-шлюза

При изменении режима шлюза "активный — резервный" на "активный — активный" необходимо создать другой общедоступный IP-адрес, затем добавить вторую IP-конфигурацию шлюза. Этот раздел поможет вам изменить существующий VPN-шлюз Azure с режима "активный — резервный" на "активный — активный" или наоборот с помощью PowerShell. Шлюз можно также изменить в портал Azure на странице **настройки** шлюза виртуальной сети.

### <a name="change-an-active-standby-gateway-to-an-active-active-gateway"></a>Настройка режима "активный — активный" для шлюза в режиме "активный — резервный"

В следующем примере шлюз в режиме "активный — резервный" переходит в режим "активный — активный". 

#### <a name="1-declare-your-variables"></a>1. Объявите переменные

Замените приведенные ниже параметры, используемые для примера, собственными параметрами конфигурации, после чего объявите эти переменные.

```azurepowershell-interactive
$GWName = "TestVNetAA1GW"
$VNetName = "TestVNetAA1"
$RG = "TestVPNActiveActive01"
$GWIPName2 = "gwpip2"
$GWIPconf2 = "gw1ipconf2"
```

После объявления переменных можно скопировать этот пример и вставить его в консоль PowerShell.

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
$subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gw = Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
$location = $gw.Location
```

#### <a name="2-create-the-public-ip-address-then-add-the-second-gateway-ip-configuration"></a>2. Создайте общедоступный IP-адрес, а затем добавьте конфигурацию IP-адреса второго шлюза.

```azurepowershell-interactive
$gwpip2 = New-AzPublicIpAddress -Name $GWIPName2 -ResourceGroupName $RG -Location $location -AllocationMethod Dynamic
Add-AzVirtualNetworkGatewayIpConfig -VirtualNetworkGateway $gw -Name $GWIPconf2 -Subnet $subnet -PublicIpAddress $gwpip2
```

#### <a name="3-enable-active-active-mode-and-update-the-gateway"></a>3. Включение режима "активный — активный" и обновление шлюза

На этом шаге вы включаете режим "активный — активный" и обновляете шлюз. В приведенном примере VPN-шлюз использует устаревший номер SKU "Стандартный". Однако режим "активный — активный" не поддерживает его. Чтобы изменить размер устаревшего номера SKU на один из поддерживаемых размеров (в данном случае — HighPerformance), просто укажите поддерживаемый устаревший номер SKU, который вы хотите использовать.

* Невозможно изменить устаревший номер SKU на один из новых номеров SKU с помощью этого действия. Можно только изменить размер устаревшего номера SKU на другой поддерживаемый устаревший номер SKU. Например, невозможно изменить номер SKU "Стандартный" на VpnGw1 (хотя VpnGw1 и поддерживается для режима "активный — активный"), так как номер SKU "Стандартный" является устаревшим, а VpnGw1 — современным номером SKU. Дополнительные сведения об изменении размера и переходе с различных номеров SKU см. в разделе [SKU шлюзов](vpn-gateway-about-vpngateways.md#gwsku).

* Если вы хотите изменить размер современного номера SKU, например изменить VpnGw1 на VpnGw3, это можно сделать с помощью этого действия, так как эти номера SKU из одного семейства SKU. Для этого используется значение ```-GatewaySku VpnGw3```.

Если при использовании этой операции в своей среде вам не требуется изменять размер шлюза, то указывать параметр -GatewaySku не нужно. Обратите внимание на то, что на этом шаге нужно задать объект шлюза в PowerShell, чтобы активировать фактическое обновление. Это обновление может занять 30–45 минут, даже если вы не изменяете размер шлюза.

```azurepowershell-interactive
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -EnableActiveActiveFeature -GatewaySku HighPerformance
```

### <a name="change-an-active-active-gateway-to-an-active-standby-gateway"></a>Настройка режима "активный — резервный" для шлюза в режиме "активный — активный"
#### <a name="1-declare-your-variables"></a>1. Объявите переменные

Замените приведенные ниже параметры, используемые для примера, собственными параметрами конфигурации, после чего объявите эти переменные.

```azurepowershell-interactive
$GWName = "TestVNetAA1GW"
$RG = "TestVPNActiveActive01"
```

После объявления переменных получите имя IP-конфигурации, которую нужно удалить.

```azurepowershell-interactive
$gw = Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
$ipconfname = $gw.IpConfigurations[1].Name
```

#### <a name="2-remove-the-gateway-ip-configuration-and-disable-the-active-active-mode"></a>2. Удалите IP-конфигурацию шлюза и отключите режим "активный — активный".

Используйте этот пример, чтобы удалить IP-конфигурацию шлюза и отключить режим "активный — активный". Обратите внимание на то, что нужно задать объект шлюза в PowerShell, чтобы активировать фактическое обновление.

```azurepowershell-interactive
Remove-AzVirtualNetworkGatewayIpConfig -Name $ipconfname -VirtualNetworkGateway $gw
Set-AzVirtualNetworkGateway -VirtualNetworkGateway $gw -DisableActiveActiveFeature
```

Для обновления может потребоваться 30–45 минут.

## <a name="next-steps"></a>Дальнейшие действия
Установив подключение, можно добавить виртуальные машины в виртуальные сети. Инструкции см. в статье о [создании виртуальной машины](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
