---
title: Пример скрипта Azure PowerShell. Настройка VPN-подключения между виртуальными сетями | Документация Майкрософт
description: Сведения о том, как с помощью скрипта PowerShell настроить подключение VPN-шлюза между сайтами и между виртуальными сетями.
services: vpn-gateway
documentationcenter: vpn-gateway
author: kumudD
ms.service: vpn-gateway
ms.devlang: powershell
ms.topic: sample
ms.date: 01/09/2020
ms.author: alzam
ms.openlocfilehash: 949c2b7810bc6b4f5a80b9932427f4aa3565794e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88030373"
---
# <a name="use-powershell-to-configure-a-vnet-to-vnet-vpn-gateway-connection"></a>Настройка подключения VPN-шлюза между виртуальными сетями с помощью PowerShell

Этот скрипт устанавливает подключение "виртуальная сеть — виртуальная сеть" между двумя виртуальными сетями.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

```azurepowershell-interactive
# Declare variables for VNET 1
  $RG1 = "TestRG1"
  $VNetName1  = "VNet1"
  $FESubName1 = "FrontEnd"
  $GWSubName1 = "GatewaySubnet"
  $VNetPrefix11 = "10.1.0.0/16"
  $FESubPrefix1 = "10.1.0.0/24"
  $GWSubPrefix1 = "10.1.255.0/27"
  $Location1 = "EastUS"
  $GWName1 = "VNet1GW"
  $GWIPName1 = "VNet1GWIP"
  $GWIPconfName1 = "gwipconfig1"
  $Connection12 = "VNet1toVNet2"

# Declare variables for VNET 2  
  $RG2 = "TestRG2"
  $VNetName2  = "VNet2"
  $FESubName2 = "FrontEnd"
  $GWSubName2 = "GatewaySubnet"
  $VNetPrefix21 = "10.2.0.0/16"
  $FESubPrefix2 = "10.2.0.0/24"
  $GWSubPrefix2 = "10.2.255.0/27"
  $Location2 = "EastUS"
  $GWName2 = "VNet2GW"
  $GWIPName2 = "VNet2GWIP"
  $GWIPconfName2 = "gwipconfig2"
  $Connection21 = "VNet2toVNet1"

# Create first resource group
New-AzResourceGroup -Name $RG1 -Location $Location1

# Create a virtual network 1
$virtualNetwork1 = New-AzVirtualNetwork `
  -ResourceGroupName $RG1 `
  -Location $Location1 `
  -Name $VNetName1 `
  -AddressPrefix $VNetPrefix11

# Create a subnet configuration
Add-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1 -VirtualNetwork $virtualNetwork1

# Set the subnet configuration for virtual network 1
$virtualNetwork1 | Set-AzVirtualNetwork

# Add a gateway subnet
Add-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1 -VirtualNetwork $virtualNetwork1

# Set the subnet configuration for the virtual network
$virtualNetwork1 | Set-AzVirtualNetwork

# Request a public IP address
$gwpip1= New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 `
 -AllocationMethod Dynamic

# Create the gateway IP address configuration
$vnet1 = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1 = Get-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -VirtualNetwork $vnet1
$gwipconfig1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -SubnetId $subnet1.Id -PublicIpAddressId $gwpip1.Id

# Create the VPN gateway (takes 20-40 minutes)
New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 `
 -Location $Location1 -IpConfigurations $gwipconfig1 -GatewayType Vpn `
 -VpnType RouteBased -GatewaySku VpnGw1 

# Create the second resource group
New-AzResourceGroup -Name $RG2 -Location $Location2

# Create a virtual network 2
$virtualNetwork2 = New-AzVirtualNetwork `
  -ResourceGroupName $RG2 `
  -Location $Location2 `
  -Name $VNetName2 `
  -AddressPrefix $VNetPrefix21

# Create a subnet configuration
Add-AzVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2 -VirtualNetwork $virtualNetwork2

# Set the subnet configuration for virtual network 2
$virtualNetwork2 | Set-AzVirtualNetwork

# Add a gateway subnet
Add-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2 -VirtualNetwork $virtualNetwork2

# Set the subnet configuration for the virtual network
$virtualNetwork2 | Set-AzVirtualNetwork

# Request a public IP address
$gwpip2 = New-AzPublicIpAddress -Name $GWIPName2 -ResourceGroupName $RG2 -Location $Location2 `
 -AllocationMethod Dynamic

# Create the gateway IP address configuration
$vnet2 = Get-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2 = Get-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -VirtualNetwork $vnet2
$gwipconfig2 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName2 -SubnetId $subnet2.Id -PublicIpAddressId $gwpip2.Id

# Create the VPN gateway (takes 20-40 minutes)
New-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 `
 -Location $Location2 -IpConfigurations $gwipconfig2 -GatewayType Vpn `
 -VpnType RouteBased -GatewaySku VpnGw1

# Create the connections
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet2gw = Get-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2
New-AzVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 `
-VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 `
-ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
New-AzVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 `
-VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 `
-ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
 ```
 
## <a name="clean-up-resources"></a>Очистка ресурсов

Когда созданные ресурсы станут не нужны, удалите группу ресурсов с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). При этом будут удалены группы ресурсов и все содержащиеся в них ресурсы.

```azurepowershell-interactive
Remove-AzResourceGroup -Name TestRG1
Remove-AzResourceGroup -Name TestRG2
```
 
 
## <a name="script-explanation"></a>Описание скрипта

Чтобы создать развертывание, скрипт использует следующие команды. Для каждого элемента в таблице приведены ссылки на документацию по команде.

| Get-Help | Примечания |
|---|---|
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Добавление конфигурации подсети. Эта конфигурация используется в процессе создания виртуальной сети. |
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Получение сведений о виртуальной сети. |
| [Get-AzVirtualNetworkGateway](/powershell/module/az.network/get-azvirtualnetworkgateway) | Получение сведений о шлюзе виртуальной сети. |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Получение сведений о конфигурации подсети в виртуальной сети. |
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Создает конфигурацию подсети. Эта конфигурация используется в процессе создания виртуальной сети. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес. |
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Создание конфигурации IP-адреса шлюза. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Создание VPN-шлюза. |
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Создает подключение между виртуальными сетями. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов и все ресурсы, содержащиеся в ней. |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Установка конфигурации подсети для виртуальной сети. |
| [Set-AzVirtualNetworkGateway](/powershell/module/az.network/set-azvirtualnetworkgateway) | Установка конфигурации для VPN-шлюза. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).
