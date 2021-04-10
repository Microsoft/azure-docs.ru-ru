---
title: Пример скрипта Azure PowerShell для создания VPN-шлюза
titleSuffix: Azure VPN Gateway
description: Сведения о том, как с помощью скрипта PowerShell создать виртуальную сеть, подсеть и VPN-шлюз на основе маршрутов.
services: vpn-gateway
documentationcenter: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.devlang: powershell
ms.topic: sample
ms.date: 02/10/2021
ms.author: alzam
ms.openlocfilehash: d41ec492f1f694a8b39f16a7932c4476d9614a1a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100385886"
---
# <a name="create-a-vpn-gateway---powershell-script-sample"></a>Создание VPN-шлюза — пример скрипта PowerShell

С помощью этого скрипта PowerShell создается VPN-шлюз на основе маршрутов.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

```azurepowershell-interactive
# Declare variables
  $RG = "TestRG1"
  $Location = "East US"
  $VNetName  = "VNet1"
  $FESubName = "FrontEnd"
  $VNetPrefix1 = "10.1.0.0/16"
  $FESubPrefix = "10.1.0.0/24"
  $GWSubPrefix = "10.1.255.0/27"
  $GWName = "VNet1GW"
  $GWIPName = "VNet1GWIP"

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
Add-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix $GWSubPrefix -VirtualNetwork $vnet
# Set the subnet configuration for the virtual network
$vnet | Set-AzVirtualNetwork
# Request a public IP address
$gwpip= New-AzPublicIpAddress -Name $GWIPName -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
# Create the gateway IP address configuration
$vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $RG
$subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
$gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id
# Create the VPN gateway
New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG `
-Location $Location -IpConfigurations $gwipconfig -GatewayType Vpn `
-VpnType RouteBased -GatewaySku VpnGw1
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
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Создает группу ресурсов, в которой хранятся все ресурсы. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Создает конфигурацию подсети. Эта конфигурация используется в процессе создания виртуальной сети. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Создает виртуальную сеть. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Создает общедоступный IP-адрес. |
|[New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Создание VPN-шлюза. |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Удаляет группу ресурсов и все ресурсы, содержащиеся в ней. |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Установка конфигурации подсети для виртуальной сети. |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о модуле Azure PowerShell см. в [документации по Azure PowerShell](/powershell/azure/).
