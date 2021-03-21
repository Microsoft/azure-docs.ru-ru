---
title: Обновление приложения IPv4 до IPv6 в виртуальной сети Azure с помощью PowerShell
titlesuffix: Azure Virtual Network
description: В этой статье показано, как развернуть IPv6-адреса в существующем приложении в виртуальной сети Azure с помощью Azure PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: 9d2ff583b032ff1f9aa5dbc9706ea6c981fc7265
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96009982"
---
# <a name="upgrade-an-ipv4-application-to-ipv6-in-azure-virtual-network---powershell"></a>Обновление приложения IPv4 до IPv6 в виртуальной сети Azure с помощью PowerShell

В этой статье показано, как добавить IPv6-подключение к существующему приложению IPv4 в виртуальной сети Azure с помощью Load Balancer (цен. категория "Стандартный") и общедоступного IP-адреса. Обновление на месте включает в себя:
- Адресное пространство IPv6 для виртуальной сети и подсети
- Load Balancer (цен. категория "Стандартный") с многоинтерфейсными конфигурациями IPv4 и IPV6
- Виртуальные машины с сетевыми адаптерами с конфигурацией IPv4 и IPv6
- Общедоступный IP-адрес IPv6, поэтому подсистема балансировки нагрузки использует IPv6-подключение к Интернету

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этой статьей, вам понадобится модуль Azure PowerShell 6.9.0 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="prerequisites"></a>Предварительные условия

В этой статье предполагается, что вы развернули Load Balancer (цен. категория "Стандартный"), как описано в разделе [Краткое руководство по созданию Load Balancer (цен. категория "Стандартный") Azure PowerShell](../load-balancer/quickstart-load-balancer-standard-public-powershell.md).

## <a name="retrieve-the-resource-group"></a>Получение группы ресурсов

Прежде чем создавать виртуальную сеть с двумя стеками, необходимо получить группу ресурсов с помощью [Get-азресаурцеграуп](/powershell/module/az.resources/get-azresourcegroup).

```azurepowershell-interactive
$rg = Get-AzResourceGroup  -ResourceGroupName "myResourceGroupSLB"
```

## <a name="create-an-ipv6-ip-addresses"></a>Создание IP-адресов IPv6

Создайте общедоступный адрес IPv6 с помощью [New-азпублиЦипаддресс](/powershell/module/az.network/new-azpublicipaddress) для Load Balancer (цен. Категория "Стандартный"). В следующем примере создается общедоступный IP-адрес IPv6 с именем *PublicIP_v6* в группе ресурсов *myResourceGroupSLB* :

```azurepowershell-interactive  
$PublicIP_v6 = New-AzPublicIpAddress `
  -Name "PublicIP_v6" `
  -ResourceGroupName $rg.ResourceGroupName `
  -Location $rg.Location  `
  -Sku Standard  `
  -AllocationMethod Static `
  -IpAddressVersion IPv6
```

## <a name="configure-load-balancer-frontend"></a>Настройка внешнего интерфейса балансировщика нагрузки

Извлеките существующую конфигурацию подсистемы балансировки нагрузки и добавьте новый IP-адрес IPv6 с помощью [Add-азлоадбаланцерфронтендипконфиг](/powershell/module/az.network/Add-AzLoadBalancerFrontendIpConfig) следующим образом:

```azurepowershell-interactive
# Retrieve the load balancer configuration
$lb = Get-AzLoadBalancer -ResourceGroupName $rg.ResourceGroupName -Name "MyLoadBalancer"

# Add IPv6 components to the local copy of the load balancer configuration
$lb | Add-AzLoadBalancerFrontendIpConfig `
  -Name "dsLbFrontEnd_v6" `
  -PublicIpAddress $PublicIP_v6

#Update the running load balancer with the new frontend
$lb | Set-AzLoadBalancer
```

## <a name="configure-load-balancer-backend-pool"></a>Настройка внутреннего пула подсистемы балансировки нагрузки

Создайте внутренний пул в локальной копии конфигурации подсистемы балансировки нагрузки и обновите работающую подсистему балансировки нагрузки с помощью новой конфигурации серверного пула следующим образом:

```azurepowershell-interactive
$lb | Add-AzLoadBalancerBackendAddressPoolConfig -Name "LbBackEndPool_v6"

# Update the running load balancer with the new backend pool
$lb | Set-AzLoadBalancer
```

## <a name="configure-load-balancer-rules"></a>Настройка правил подсистемы балансировки нагрузки.
Извлеките существующую конфигурацию внешнего интерфейса и внутреннего пула подсистемы балансировки нагрузки, а затем добавьте новые правила балансировки нагрузки с помощью [Add-азлоадбаланцеррулеконфиг](/powershell/module/az.network/Add-AzLoadBalancerRuleConfig).

```azurepowershell-interactive
# Retrieve the updated (live) versions of the frontend and backend pool
$frontendIPv6 = Get-AzLoadBalancerFrontendIpConfig -Name "dsLbFrontEnd_v6" -LoadBalancer $lb
$backendPoolv6 = Get-AzLoadBalancerBackendAddressPoolConfig -Name "LbBackEndPool_v6" -LoadBalancer $lb

# Create new LB rule with the frontend and backend
$lb | Add-AzLoadBalancerRuleConfig `
  -Name "dsLBrule_v6" `
  -FrontendIpConfiguration $frontendIPv6 `
  -BackendAddressPool $backendPoolv6 `
  -Protocol Tcp `
  -FrontendPort 80 `
  -BackendPort 80

#Finalize all the load balancer updates on the running load balancer
$lb | Set-AzLoadBalancer
```
## <a name="add-ipv6-address-ranges"></a>Добавить диапазоны IPv6-адресов

Добавьте диапазоны IPv6-адресов в виртуальную сеть и подсеть, в которых размещаются виртуальные машины, следующим образом:

```azurepowershell-interactive
#Add IPv6 ranges to the VNET and subnet
#Retreive the VNET object
$vnet = Get-AzVirtualNetwork  -ResourceGroupName $rg.ResourceGroupName -Name "myVnet" 

#Add IPv6 prefix to the VNET
$vnet.addressspace.addressprefixes.add("fd00:db8:deca::/48")

#Update the running VNET
$vnet |  Set-AzVirtualNetwork

#Retrieve the subnet object from the local copy of the VNET
$subnet= $vnet.subnets[0]

#Add IPv6 prefix to the Subnet (subnet of the VNET prefix, of course)
$subnet.addressprefix.add("fd00:db8:deca::/64")

#Update the running VNET with the new subnet configuration
$vnet |  Set-AzVirtualNetwork
```

## <a name="add-ipv6-configuration-to-nic"></a>Добавление конфигурации IPv6 к сетевой карте

Настройте все сетевые карты виртуальных машин с IPv6-адресом с помощью [Add-азнетворкинтерфацеипконфиг](/powershell/module/az.network/Add-AzNetworkInterfaceIpConfig) следующим образом:

```azurepowershell-interactive
#Retrieve the NIC objects
$NIC_1 = Get-AzNetworkInterface -Name "myNic1" -ResourceGroupName $rg.ResourceGroupName
$NIC_2 = Get-AzNetworkInterface -Name "myNic2" -ResourceGroupName $rg.ResourceGroupName
$NIC_3 = Get-AzNetworkInterface -Name "myNic3" -ResourceGroupName $rg.ResourceGroupName

#Add an IPv6 IPconfig to NIC_1 and update the NIC on the running VM
$NIC_1 | Add-AzNetworkInterfaceIpConfig -Name MyIPv6Config -Subnet $vnet.Subnets[0]  -PrivateIpAddressVersion IPv6 -LoadBalancerBackendAddressPool $backendPoolv6 
$NIC_1 | Set-AzNetworkInterface

#Add an IPv6 IPconfig to NIC_2 and update the NIC on the running VM
$NIC_2 | Add-AzNetworkInterfaceIpConfig -Name MyIPv6Config -Subnet $vnet.Subnets[0]  -PrivateIpAddressVersion IPv6 -LoadBalancerBackendAddressPool $backendPoolv6 
$NIC_2 | Set-AzNetworkInterface

#Add an IPv6 IPconfig to NIC_3 and update the NIC on the running VM
$NIC_3 | Add-AzNetworkInterfaceIpConfig -Name MyIPv6Config -Subnet $vnet.Subnets[0]  -PrivateIpAddressVersion IPv6 -LoadBalancerBackendAddressPool $backendPoolv6 
$NIC_3 | Set-AzNetworkInterface
```

## <a name="view-ipv6-dual-stack-virtual-network-in-azure-portal"></a>Просмотр двух виртуальных сетей IPv6 с двумя стеками в портал Azure

Виртуальную сеть с двумя стеками IPv6 можно просмотреть в портал Azure следующим образом:
1. На панели поиска портала введите *myVnet*.
2. Когда в результатах поиска появится **myVnet** , выберите его. Откроется страница **обзора** для виртуальной сети с двумя стеками с именем *myVNet*. В виртуальной сети с двумя стеками показаны три сетевых адаптера с конфигурациями IPv4 и IPv6, расположенными в подсети с двойным стеком с именем *mySubnet*.

  ![Виртуальная сеть с двумя стеками IPv6 в Azure](./media/ipv6-add-to-existing-vnet-powershell/ipv6-dual-stack-vnet.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyAzureResourceGroupSLB
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы обновили существующую Load Balancer (цен. категория "Стандартный") с многоинтерфейсной IP-конфигурацией IPv4 для конфигурации с двумя стеками (IPv4 и IPv6). Вы также добавили конфигурации IPv6 в сетевые карты виртуальных машин в серверном пуле и в виртуальную сеть, в которой они размещены. Дополнительные сведения о поддержке IPv6 в виртуальных сетях Azure см. в статье [что такое IPv6 для виртуальной сети Azure?](ipv6-overview.md)
