---
title: 'VPN-шлюз Azure: Настройка BGP: PowerShell'
description: В этой статье описана поэтапная настройка протокола BGP для VPN-шлюзов Azure с помощью Azure Resource Manager и PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8573d9e55299382392927b532966a6e6fdd8c439
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94659766"
---
# <a name="how-to-configure-bgp-on-azure-vpn-gateways-using-powershell"></a>Настройка BGP на VPN-шлюзах Azure с помощью PowerShell
В этой статье содержится пошаговое описание процесса, который позволит с помощью модели развертывания Resource Manager и PowerShell включить BGP для VPN-подключения типа "сеть — сеть" (S2S), настроенного между локальными сетями, или для подключения между виртуальными сетями.



## <a name="about-bgp"></a>О протоколе BGP
Стандартный протокол маршрутизации BGP обычно используется в Интернете для обмена данными о маршрутизации и доступности между двумя или несколькими сетями. Протокол BGP используется VPN-шлюзами Azure и локальными VPN-устройствами (так называемые узлы BGP) для обмена данными о маршрутах. Это позволяет информировать участников обмена о доступности и возможности передавать трафик для определенных сетей через шлюзы или маршрутизаторы. Также BGP позволяет передавать трафик транзитом через несколько сетей. Для этого шлюз BGP распространяет на все известные ему узлы BGP информацию о маршрутах, полученную от остальных узлов BGP.

В статье [Обзор использования BGP с VPN-шлюзами Azure](vpn-gateway-bgp-overview.md) приведены дополнительные сведения о преимуществах BGP, о технических требованиях и важных аспектах использования BGP.

## <a name="getting-started-with-bgp-on-azure-vpn-gateways"></a>Приступая к работе с BGP на VPN-шлюзах Azure

В этой статье описано выполнение следующих действий:

* [Часть 1 — активация BGP на VPN-шлюзе Azure](#enablebgp)
* Часть 2 — создание подключения между локальными сетями с использованием BGP
* [Часть 3 — создание подключения между виртуальными сетями с использованием BGP](#v2vbgp)

Каждая часть этой инструкции является базовым блоком для использования BGP в вашей сети. Выполнив инструкции, приведенные в трех частях, вы создадите топологию, которая представлена на следующей схеме:

![Топология BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

Вы можете объединять блоки для создания более сложных, многоскачковых и транзитных сетей, в соответствии со своими задачами.

## <a name="part-1---configure-bgp-on-the-azure-vpn-gateway"></a><a name ="enablebgp"></a>Часть 1 — настройка BGP для VPN-шлюза Azure
Описанные ниже действия позволят настроить параметры BGP для VPN-шлюза Azure в соответствии со следующей схемой:

![Шлюз BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>Перед началом
* Убедитесь в том, что у вас уже есть подписка Azure. Если у вас нет подписки Azure, вы можете [активировать преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) или [зарегистрировать бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/).
* Установите командлеты PowerShell для Azure Resource Manager. См. дополнительные сведения об [установке и настройке командлетов Azure PowerShell](/powershell/azure/). 

### <a name="step-1---create-and-configure-vnet1"></a>Шаг 1. Создание и настройка VNet1
#### <a name="1-declare-your-variables"></a>1. Объявите переменные
В этом упражнении мы начнем с объявления переменных. В примере ниже объявлены переменные со значениями для этого упражнения. Обязательно замените значения своими при настройке для рабочей среды. Эти переменные можно использовать для ознакомления с этим типом конфигурации. Измените переменные, а затем скопируйте и вставьте код в консоль PowerShell.

```powershell
$Sub1 = "Replace_With_Your_Subscription_Name"
$RG1 = "TestBGPRG1"
$Location1 = "East US"
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
$GWIPName1 = "VNet1GWIP"
$GWIPconfName1 = "gwipconf1"
$Connection12 = "VNet1toVNet2"
$Connection15 = "VNet1toSite5"
```

#### <a name="2-connect-to-your-subscription-and-create-a-new-resource-group"></a>2. Подключение к подписке Azure и создание группы ресурсов
Для работы с командлетами Resource Manager необходимо переключиться в режим PowerShell. Дополнительные сведения см. в статье [Использование Azure PowerShell с диспетчером ресурсов Azure](../azure-resource-manager/management/manage-resources-powershell.md).

Откройте консоль PowerShell и подключитесь к своей учетной записи. Для подключения используйте следующий пример.

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
New-AzResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-testvnet1"></a>3. Создание TestVNet1
В примере ниже создается виртуальная сеть с именем TestVNet1 и три подсети: GatewaySubnet, FrontEnd и Backend. При замене значений важно, чтобы вы назвали подсеть шлюза именем GatewaySubnet. Если вы используете другое имя, создание шлюза завершится сбоем.

```powershell
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
```

### <a name="step-2---create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>Шаг 2. Создание VPN-шлюза для TestVNet1 с параметрами BGP
#### <a name="1-create-the-ip-and-subnet-configurations"></a>1. Создание конфигураций IP-адресов и подсетей
Запросите выделение общедоступного IP-адреса для шлюза, который будет создан для виртуальной сети. Также следует определить необходимые конфигурации подсети и IP-адресов.

```powershell
$gwpip1 = New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic

$vnet1 = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -Subnet $subnet1 -PublicIpAddress $gwpip1
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2. Создайте VPN-шлюз с номером AS.
Создайте шлюз для виртуальной сети TestVNet1. Для использования BGP обязательно нужен VPN-шлюз на основе маршрутов, а также дополнительный параметр -Asn, который задает номер ASN для TestVNet1. Если параметр ASN не указан, будет назначено значение ASN 65515. Создание шлюза может занять некоторое время (30 минут или более).

```powershell
New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3. получение IP-адреса узла BGP Azure
После создания шлюза необходимо получить IP-адрес для узла BGP на VPN-шлюзе Azure. Этот адрес нужен, чтобы VPN-шлюз Azure мог выполнять роль узла BGP для локальных VPN-устройств.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet1gw.BgpSettingsText
```

Последняя команда выводит соответствующую конфигурацию BGP для VPN-шлюза Azure.

```powershell
$vnet1gw.BgpSettingsText
{
    "Asn": 65010,
    "BgpPeeringAddress": "10.12.255.30",
    "PeerWeight": 0
}
```

С помощью созданного шлюза можно установить подключение между локальными или виртуальными сетями с использованием BGP. В следующих разделах описано, как это сделать.

## <a name="part-2---establish-a-cross-premises-connection-with-bgp"></a><a name ="crossprembbgp"></a>Часть 2 — создание подключения между локальными сетями с использованием BGP

Чтобы установить подключение между локальными сетями, нужно создать локальный сетевой шлюз, который будет представлять локальное VPN-устройство, а также подключение между VPN-шлюзом и шлюзом локальной сети. Несмотря на то что эти действия подробно описаны в других статьях, в этой статье указаны дополнительные свойства, необходимые для параметров конфигурации BGP.

![BGP между локальными сетями](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)

Прежде чем продолжить, убедитесь, что вы выполнили инструкции из [первой части](#enablebgp) этой статьи.

### <a name="step-1---create-and-configure-the-local-network-gateway"></a>Шаг 1. Создание и настройка локального сетевого шлюза

#### <a name="1-declare-your-variables"></a>1. Объявите переменные

В этом подразделе мы продолжим создание конфигурации, которая представлена на схеме. Не забудьте заменить значения теми, которые вы хотите использовать для конфигурации.

```powershell
$RG5 = "TestBGPRG5"
$Location5 = "East US 2"
$LNGName5 = "Site5"
$LNGPrefix50 = "10.52.255.254/32"
$LNGIP5 = "Your_VPN_Device_IP"
$LNGASN5 = 65050
$BGPPeerIP5 = "10.52.255.254"
```

Несколько важных замечаний о параметрах локального сетевого шлюза.

* Локальный сетевой шлюз может находиться в том же расположении, что и VPN-шлюз, или в любом другом. Это же справедливо в отношении групп ресурсов. В нашем примере они располагаются в разных группах ресурсов и расположениях.
* Префикс, который необходимо объявить для локального сетевого шлюза — это IP-адрес узла BGP на вашем VPN-устройстве. В нашем примере это префикс /32 для адреса 10.52.255.254/32.
* Не забывайте, что для локальных сетей и виртуальной сети Azure должны быть указаны разные номера ASN BGP. Если они совпадают, а локальное VPN-устройство уже использует свой ASN для связи с другими соседями BGP, необходимо изменить ASN вашей виртуальной сети.

Прежде чем продолжить, убедитесь, что вы все еще подключены к подписке 1.

#### <a name="2-create-the-local-network-gateway-for-site5"></a>2. Создание шлюза локальной сети для site5

Прежде чем создавать локальный сетевой шлюз, не забудьте создать группу ресурсов, если она не была создана ранее. Обратите внимание на два дополнительных параметра локального сетевого шлюза: Asn и BgpPeerAddress.

```powershell
New-AzResourceGroup -Name $RG5 -Location $Location5

New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix50 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5
```

### <a name="step-2---connect-the-vnet-gateway-and-local-network-gateway"></a>Шаг 2. Подключение шлюза виртуальной сети к локальному сетевому шлюзу

#### <a name="1-get-the-two-gateways"></a>1. получение двух шлюзов

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG5
```

#### <a name="2-create-the-testvnet1-to-site5-connection"></a>2. Создание подключения TestVNet1 к site5

На этом шаге вы создадите подключение между TestVNet1 и Site5. Чтобы активировать BGP для этого подключения, укажите параметр -EnableBGP True. Как обсуждалось ранее, на одном VPN-шлюзе можно одновременно создавать подключения с использованием BGP и без него. Если BGP не указано в свойствах подключения явным образом, Azure не будет использовать BGP для этого подключения, даже если параметры BGP настроены для обоих шлюзов.

```powershell
New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

В следующем примере перечислены параметры, которые следует ввести в разделе конфигурации BGP на локальном VPN-устройстве для нашего упражнения.

```

- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.254
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP    : 10.12.255.30
- Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

Подключение установится через несколько минут. Сразу после создания подключения IPsec начнется сеанс пиринга BGP.

## <a name="part-3---establish-a-vnet-to-vnet-connection-with-bgp"></a><a name ="v2vbgp"></a>Часть 3 — создание подключения между виртуальными сетями с использованием BGP

В этом разделе мы добавим подключение между виртуальными сетями с использованием BGP, как показано на следующей схеме:

![Схема, показывающая сетевое подключение V-v к сети.](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

Приведенные ниже инструкции продолжают действия, описанные выше. Чтобы создать и настроить сеть TestVNet1 и VPN-шлюз с использованием BGP, сначала следует выполнить инструкции из [первой части](#enablebgp) . 

### <a name="step-1---create-testvnet2-and-the-vpn-gateway"></a>Шаг 1. Создание TestVNet2 и VPN-шлюза

Важно убедиться в том, что пространство IP-адресов для новой виртуальной сети TestVNet2 не пересекается с диапазонами любой из ваших виртуальных сетей.

В этом примере виртуальные сети относятся к одной подписке. Вы можете создавать подключения между виртуальным сетями из разных подписок. Дополнительные сведения см. в статье [Настройка подключения VPN-шлюза между виртуальными сетями с помощью PowerShell](vpn-gateway-vnet-vnet-rm-ps.md). Чтобы использовать для подключения протокол BGP, обязательно укажите параметр -EnableBgp True при создании подключения.

#### <a name="1-declare-your-variables"></a>1. Объявите переменные

Не забудьте заменить значения теми, которые вы хотите использовать для конфигурации.

```powershell
$RG2 = "TestBGPRG2"
$Location2 = "West US"
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
$GWIPName2 = "VNet2GWIP"
$GWIPconfName2 = "gwipconf2"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2. Создайте TestVNet2 в новой группе ресурсов.

```powershell
New-AzResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2
```

#### <a name="3-create-the-vpn-gateway-for-testvnet2-with-bgp-parameters"></a>3. Создание VPN-шлюза для TestVNet2 с параметрами BGP

Запросите выделение общедоступного IP-адреса для шлюза, который будет создан для виртуальной сети, и определите необходимые конфигурации подсети и IP-адресов.

```powershell
$gwpip2    = New-AzPublicIpAddress -Name $GWIPName2 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic

$vnet2     = Get-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2   = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gwipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName2 -Subnet $subnet2 -PublicIpAddress $gwpip2
```

Создайте VPN-шлюз с номером AS. Номер ASN по умолчанию необходимо переопределить для ваших VPN-шлюзов Azure. Номера ASN для подключенных виртуальных сетей должны быть разными, чтобы работал протокол BGP и транзитная маршрутизация.

```powershell
New-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gwipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet2ASN
```

### <a name="step-2---connect-the-testvnet1-and-testvnet2-gateways"></a>Шаг 2. Подключение шлюзов TestVNet1 и TestVNet2

В этом примере оба шлюза находятся в одной подписке. Этот шаг можно выполнить в одном сеансе PowerShell.

#### <a name="1-get-both-gateways"></a>1. получите оба шлюза.

Обязательно войдите и подключитесь к подписке 1.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet2gw = Get-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2
```

#### <a name="2-create-both-connections"></a>2. Создайте оба подключения.

На этом шаге вы создадите подключение из TestVNet1 к TestVNet2, а также подключение из TestVNet2 к TestVNet1.

```powershell
New-AzVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True

New-AzVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True
```

> [!IMPORTANT]
> Не забудьте включить BGP для ОБОИХ подключений.
> 
> 

Подключение будет установлено через несколько минут после выполнения этих действий. Сразу после создания подключения между виртуальными сетями начнется сеанс пиринга BGP.

Если вы выполнили все три части этого упражнения, ваша сеть будет иметь примерно такую топологию:

![BGP между виртуальными сетями](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

## <a name="next-steps"></a>Дальнейшие действия

Установив подключение, можно добавить виртуальные машины в виртуальные сети. Инструкции см. в статье о [создании виртуальной машины](../virtual-machines/windows/quick-create-portal.md).