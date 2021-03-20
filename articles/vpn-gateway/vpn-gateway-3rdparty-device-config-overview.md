---
title: Конфигурации партнерских VPN-устройств для подключения к VPN-шлюзам Azure
description: В этой статье содержится обзор конфигураций партнерских VPN-устройств, позволяющих подключаться к VPN-шлюзам Azure.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 484dad65310efaa60e8744b2f122b5e44ae13565
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98880158"
---
# <a name="overview-of-partner-vpn-device-configurations"></a>Обзор конфигураций партнерских VPN-устройств
В этой статье содержится обзор процесса настройки локальных VPN-устройств для подключения к VPN-шлюзам Azure. Для демонстрации подключения к различным конфигурациям локальных VPN-устройств с помощью одинаковых параметров используются примеры виртуальной сети Azure и настройки VPN-шлюза.



## <a name="device-requirements"></a>Требования к устройствам
В VPN-шлюзах Azure используются стандартные наборы протоколов IPsec/IKE для VPN-туннелей типа "сайт — сайт" (S2S). Список параметров IPsec/IKE и алгоритмы шифрования для VPN-шлюзов Azure приведены в статье [VPN-устройства и параметры IPsec/IKE для подключений типа "сеть — сеть" через VPN-шлюз](vpn-gateway-about-vpn-devices.md). Также можно указать точные алгоритмы и уровни стойкости ключа для определенного подключения, как описано в статье [Требования к шифрованию и VPN-шлюзы Azure](vpn-gateway-about-compliance-crypto.md).

## <a name="single-vpn-tunnel"></a><a name ="singletunnel"></a>Одиночный VPN-туннель
Первая конфигурация в примере состоит из одного VPN-туннеля S2S между VPN-шлюзом Azure и локальным устройством VPN. При необходимости можно также настроить [протокол BGP поверх VPN-туннеля](#bgp).

![Схема одиночного VPN-туннеля S2S](./media/vpn-gateway-3rdparty-device-config-overview/singletunnel.png)

Пошаговые инструкции по настройке одиночного VPN-туннеля см. в статье [Создание подключения типа "сеть — сеть" на портале Azure](./tutorial-site-to-site-portal.md). В следующих разделах указаны параметры подключения для примера конфигурации, а также приводится сценарий PowerShell для начала работы.

### <a name="connection-parameters"></a>Параметры подключения
В этом разделе перечислены параметры для примеров, описанных в предыдущих разделах.

| **Параметр**                | **Значение**                    |
| ---                          | ---                          |
| Префиксы адресов виртуальной сети        | 10.11.0.0/16;<br>10.12.0.0/16 |
| IP-адрес VPN-шлюза Azure         | IP-адрес VPN-шлюза Azure         |
| Префиксы адресов в локальной среде | 10.51.0.0/16<br>10.52.0.0/16 |
| IP-адрес VPN-устройства в локальной среде    | IP-адрес VPN-устройства в локальной среде    |
| *ASN для BGP виртуальной сети                | 65010                        |
| *IP-адрес узла BGP Azure           | 10.12.255.30                 |
| *ASN для BGP в локальной среде         | 65050                        |
| *IP-адрес узла BGP в локальной среде     | 10.52.255.254                |

\\* Необязательный параметр только для BGP.

### <a name="sample-powershell-script"></a>Пример скрипта PowerShell
В этом разделе приведен пример скрипта для начала работы. Подробные инструкции приводятся в статье [Создание виртуальной сети с VPN-подключением типа "сеть — сеть" с помощью PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md).

```powershell
# Declare your variables

$Sub1          = "Replace_With_Your_Subscription_Name"
$RG1           = "TestRG1"
$Location1     = "East US 2"
$VNetName1     = "TestVNet1"
$FESubName1    = "FrontEnd"
$BESubName1    = "Backend"
$GWSubName1    = "GatewaySubnet"
$VNetPrefix11  = "10.11.0.0/16"
$VNetPrefix12  = "10.12.0.0/16"
$FESubPrefix1  = "10.11.0.0/24"
$BESubPrefix1  = "10.12.0.0/24"
$GWSubPrefix1  = "10.12.255.0/27"
$VNet1ASN      = 65010
$DNS1          = "8.8.8.8"
$GWName1       = "VNet1GW"
$GWIPName1     = "VNet1GWIP"
$GWIPconfName1 = "gwipconf1"
$Connection15  = "VNet1toSite5"
$LNGName5      = "Site5"
$LNGPrefix50   = "10.52.255.254/32"
$LNGPrefix51   = "10.51.0.0/16"
$LNGPrefix52   = "10.52.0.0/16"
$LNGIP5        = "Your_VPN_Device_IP"
$LNGASN5       = 65050
$BGPPeerIP5    = "10.52.255.254"

# Connect to your subscription and create a new resource group

Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
New-AzResourceGroup -Name $RG1 -Location $Location1

# Create virtual network

$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1 $besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1

# Create VPN gateway

$gwpip1    = New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$vnet1     = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1   = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -Subnet $subnet1 -PublicIpAddress $gwpip1

New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN

# Create local network gateway

New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix51,$LNGPrefix52 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5

# Create the S2S VPN connection

$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1

New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $False
```

### <a name="optional-use-custom-ipsecike-policy-with-usepolicybasedtrafficselectors"></a><a name ="policybased"></a>Используемых Использование настраиваемой политики IPsec/IKE с UsePolicyBasedTrafficSelectors
Если ваши VPN-устройства не поддерживают селекторы трафика "любой к любому", такие как конфигурация на основе маршрутов или на основе VTI, то создайте настраиваемую политику IPsec/IKE с параметром [UsePolicyBasedTrafficSelectors](vpn-gateway-connect-multiple-policybased-rm-ps.md).

> [!IMPORTANT]
> Необходимо создать политику IPsec/IKE для включения параметра **UsePolicyBasedTrafficSelectors** для подключения.


Пример сценария создает политику IPsec/IKE со следующими параметрами и алгоритмами:
* IKEv2: AES256, SHA384, DHGroup24
* IPsec: AES256, SHA1, PFS24, время существования сопоставления безопасности 7200 секунд, размер 20 480 000 КБ (20 ГБ).

Сценарий применяет политику IPsec/IKE и включает параметр **UsePolicyBasedTrafficSelectors** для подключения.

```powershell
$ipsecpolicy5 = New-AzIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA1 -PfsGroup PFS24 -SALifeTimeSeconds 7200 -SADataSizeKilobytes 20480000

$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1

New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $False -IpsecPolicies $ipsecpolicy5 -UsePolicyBasedTrafficSelectors $True
```

### <a name="optional-use-bgp-on-s2s-vpn-connection"></a><a name ="bgp"></a>Используемых Использовать VPN-подключение BGP на S2S
При создании VPN-подключения типа "сеть — сеть" (S2S) можно использовать [BGP для VPN-шлюза](vpn-gateway-bgp-resource-manager-ps.md). Такой подход имеет два отличия:

* Префиксы адресов в локальной среде могут быть адресом одиночного хоста. IP-адрес узла BGP в локальной среде указывается следующим образом:

    ```powershell
    New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix50 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5
    ```

* При создании подключения необходимо задать для параметра **-EnableBGP** значение $True:

    ```powershell
    New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
    ```

## <a name="next-steps"></a>Дальнейшие действия
Пошаговые инструкции по настройке VPN-шлюзов в режиме "активный — активный" см. в статье [Настройка VPN-подключений типа "сеть — сеть" в режиме "активный — активный" для VPN-шлюзов Azure](vpn-gateway-activeactive-rm-powershell.md).