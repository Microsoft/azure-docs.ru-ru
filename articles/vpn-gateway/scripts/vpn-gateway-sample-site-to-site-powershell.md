---
title: Пример скрипта Azure PowerShell для настройки VPN типа "сеть — сеть"
description: С помощью PowerShell вы можете создать VPN-шлюз на основе маршрутов и настроить свое VPN-устройство на добавление подключений между сайтами.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: sample
ms.date: 02/09/2021
ms.author: cherylmc
ms.openlocfilehash: 9778942dc24a81c839e14e095a755a280a17d9c9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100379137"
---
# <a name="create-a-vpn-gateway-and-add-a-site-to-site-connection-using-powershell"></a>Создание VPN-шлюза и добавление подключения типа "сеть — сеть" с помощью PowerShell

Этот скрипт создает VPN-шлюз на основе маршрутов и добавляется конфигурация "точка — сеть". Для создания подключения, необходимо также настроить устройство VPN. См. дополнительные сведения об [VPN-устройствах и параметрах IPsec/IKE для подключений "сеть — сеть" через VPN-шлюз](../vpn-gateway-about-vpn-devices.md).

```azurepowershell-interactive
# Declare variables
  $VNetName  = "VNet1"
  $RG = "TestRG1"
  $Location = "East US"
  $FESubName = "FrontEnd"
  $BESubName = "BackEnd"
  $GWSubName = "GatewaySubnet"
  $VNetPrefix1 = "10.1.0.0/16"
  $FESubPrefix = "10.1.0.0/24"
  $BESubPrefix = "10.1.1.0/24"
  $GWSubPrefix = "10.1.255.0/27"
  $VPNClientAddressPool = "192.168.0.0/24"
  $GWName = "VNet1GW"
  $GWIPName = "VNet1GWIP"
  $GWIPconfName = "gwipconf"
  $LNGName = "Site1"
# Create a resource group
New-AzResourceGroup -Name $RG -Location $Location
# Create a virtual network
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName $RG `
  -Location $Location `
  -Name $VNetName `
  -AddressPrefix $VNetPrefix1
# Create a subnet configuration
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name $FESubName `
  -AddressPrefix $FESubPrefix `
  -VirtualNetwork $virtualNetwork
# Set the subnet configuration for the virtual network
$virtualNetwork | Set-AzVirtualNetwork
# Add a gateway subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName $RG -Name $VNetName
Add-AzVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix -VirtualNetwork $vnet
# Set the subnet configuration for the virtual network
$vnet | Set-AzVirtualNetwork
# Request a public IP address
$gwpip= New-AzPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location `
 -AllocationMethod Dynamic
# Create the gateway IP address configuration
$vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $GWSubName -VirtualNetwork $vnet
$gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
# Create the VPN gateway
New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
 -Location $Location -IpConfigurations $gwipconfig -GatewayType Vpn `
 -VpnType RouteBased -GatewaySku VpnGw1
# Create the local network gateway
New-AzLocalNetworkGateway -Name $LNGName -ResourceGroupName $RG `
 -Location $Location -GatewayIpAddress '23.99.221.164' -AddressPrefix @('10.101.0.0/24','10.101.1.0/24')
# Configure your on-premises VPN device
# Create the VPN connection
$gateway1 = Get-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
$local = Get-AzLocalNetworkGateway -Name $LNGName -ResourceGroupName $RG
New-AzVirtualNetworkGatewayConnection -Name VNet1toSite1 -ResourceGroupName $RG `
-Location $Location -VirtualNetworkGateway1 $gateway1 -LocalNetworkGateway2 $local `
-ConnectionType IPsec -ConnectionProtocol IKEv2 -RoutingWeight 10 -SharedKey 'abc123'
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Когда созданные ресурсы станут не нужны, удалите группу ресурсов с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). При этом будет удалена группа ресурсов и все содержащиеся в ней ресурсы. 

```azurepowershell-interactive
Remove-AzResourceGroup -Name TestRG1
```

## <a name="script-explanation"></a>Описание скрипта

Чтобы создать развертывание, скрипт использует следующие команды. Для каждого элемента в таблице приведены ссылки на документацию по команде.

| Get-Help | Примечания |
|---|---|
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Добавление конфигурации подсети. Эта конфигурация используется в процессе создания виртуальной сети. |
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Получает сведения о виртуальной сети. |
| [Get-AzVirtualNetworkGateway](/powershell/module/az.network/get-azvirtualnetworkgateway) | Получение сведений о шлюзе виртуальной сети. |
| [Get-AzLocalNetworkGateway](/powershell/module/az.network/get-azvirtualnetworkgateway) | Получение сведений о шлюзе локальной сети. |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Получение сведений о конфигурации подсети в виртуальной сети. |
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Создает конфигурацию подсети. Эта конфигурация используется в процессе создания виртуальной сети. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес. |
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Создание конфигурации IP-адреса шлюза. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Создание VPN-шлюза. |
| [New-AzLocalNetworkGateway](/powershell/module/az.network/new-azlocalnetworkgateway) | Создание шлюза локальной сети. |
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Создание подключения типа "сеть — сеть". |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов и все ресурсы, содержащиеся в ней. |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Установка конфигурации подсети для виртуальной сети. |
| [Set-AzVirtualNetworkGateway](/powershell/module/az.network/set-azvirtualnetworkgateway) | Установка конфигурации для VPN-шлюза. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).