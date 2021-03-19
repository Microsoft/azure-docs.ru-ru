---
title: Развертывание брандмауэра Azure с несколькими общедоступными IP-адресами с помощью Azure PowerShell
description: Развертывание брандмауэра с несколькими общедоступными IP-адресами для защиты виртуального концентратора
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: how-to
ms.date: 07/09/2020
ms.author: victorh
ms.openlocfilehash: 652c7cbfbe63ef2ae9a0d54e05407152ea300f1d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87007016"
---
# <a name="deploy-an-azure-firewall-with-multiple-public-ip-addresses"></a>Развертывание брандмауэра Azure с несколькими общедоступными IP-адресами

Если вы хотите защитить виртуальный концентратор с помощью брандмауэра Azure, можно развернуть брандмауэр с несколькими общедоступными IP-адресами с помощью Azure PowerShell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-the-firewall"></a>Развертывание брандмауэра

Используйте следующий пример Azure PowerShell, чтобы развернуть брандмауэр Azure с несколькими общедоступными IP-адресами для защиты виртуального концентратора.

```azurepowershell
Select-AzSubscription -SubscriptionId <subscription ID> 

$rgName = <resource group name> 
$vHub = Get-AzVirtualHub -Name <hub name> 
$vHubId = $vHub.Id 
$fwpips = New-AzFirewallHubPublicIpAddress -Count 3
$hubIpAddresses = New-AzFirewallHubIpAddress -PublicIPs $fwpips 
$fw = New-AzFirewall -Name <firewall name> -ResourceGroupName $rgName `
     -Location <location> -Sku AZFW_Hub -HubIPAddresses $hubIpAddresses `
     -VirtualHubId $vHubId 
```

### <a name="update-a-public-ip-address"></a>Обновление общедоступного IP-адреса

Azure PowerShell можно использовать для обновления общедоступного IP-адреса для брандмауэра Azure. В следующем примере один общедоступный IP-адрес удаляется из брандмауэра. Он начинается с трех общедоступных IP-адресов.

```azurepowershell
Select-AzSubscription -SubscriptionId <subscription ID>

$azfw = get-azfirewall -Name <firewall name> -ResourceGroupName <resource group name>
$ip1 = New-AzFirewallPublicIpAddress -Address <first ip address to keep>
$ip2 = New-AzFirewallPublicIpAddress -Address <second ip address to keep>
$ipAddresses = $ip1,$ip2
$azfw.HubIPAddresses.publicIPs.Addresses = $ipAddresses
$azfw.HubIPAddresses.publicIPs.count = 2
Set-AzFirewall -AzureFirewall $azfw
```

## <a name="next-steps"></a>Дальнейшие действия

[Что такое защищенный виртуальный концентратор?](secured-virtual-hub.md)