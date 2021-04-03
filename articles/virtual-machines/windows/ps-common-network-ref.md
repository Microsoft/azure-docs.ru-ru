---
title: Общие команды PowerShell для виртуальных сетей Azure
description: Общие команды PowerShell, позволяющие приступить к созданию виртуальной сети и связанных с ней ресурсов для виртуальных машин.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 07/17/2017
ms.author: cynthn
ms.openlocfilehash: b4d6b20e63c42616aad0f8776fae159a0f2aa455
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87088382"
---
# <a name="common-powershell-commands-for-azure-virtual-networks"></a>Общие команды PowerShell для виртуальных сетей Azure

Если вы хотите создать виртуальную машину, необходимо создать [виртуальную сеть](../../virtual-network/virtual-networks-overview.md) или иметь данные существующей виртуальной сети, в которую можно добавить виртуальную машину. Обычно при создании виртуальной машины требуется также рассмотреть возможность создания ресурсов, приведенных в этой статье.

Сведения об установке последней версии Azure PowerShell, а также о выборе нужной подписки и входе в учетную запись Azure см. в статье [Как установить и настроить службы Azure PowerShell](/powershell/azure/).

При выполнении нескольких команд, описываемых в этой статье, могут пригодиться некоторые переменные.

- $location — расположение сетевых ресурсов. Вы можете использовать командлет [Get-AzLocation](/powershell/module/az.resources/get-azlocation), чтобы найти используемый [географический регион](https://azure.microsoft.com/regions/).
- $myResourceGroup — имя группы ресурсов, в которой находятся сетевые ресурсы.

## <a name="create-network-resources"></a>Создание сетевых ресурсов

| Задача | Get-Help |
| ---- | ------- |
| Создание конфигураций подсети |$subnet1 = [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) -Name "mySubnet1" -AddressPrefix XX.X.X.X/XX<BR>$subnet2 = New-AzVirtualNetworkSubnetConfig -Name "mySubnet2" -AddressPrefix XX.X.X.X/XX<BR><BR>В обычной сети может быть подсеть для [балансировщика нагрузки с выходом в Интернет](../../load-balancer/load-balancer-overview.md) и отдельная подсеть для [внутреннего балансировщика нагрузки](../../load-balancer/load-balancer-overview.md). |
| Создание виртуальной сети |$vnet = [New-AzureRmVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup -Location $location -AddressPrefix XX.X.X.X/XX -Subnet $subnet1, $subnet2 |
| Проверка доменного имени на уникальность |[Test-AzDnsAvailability](/powershell/module/az.network/test-azdnsavailability) -DomainNameLabel "myDNS" -Location $location<BR><BR>Вы можете указать DNS-имя домена для [ресурса с общедоступным IP-адресом](../../virtual-network/public-ip-addresses.md). В результате этого будут сопоставлены адрес доменное_имя.расположение.cloudapp.azure.com и общедоступный IP-адрес на DNS-серверах под управлением Azure. Имя может содержать только буквы, цифры и дефисы. Первым и последним знаком должна быть буква или цифра, а доменное имя должно быть уникальным в пределах его расположения Azure. Если возвращается значение **True**, то предложенное имя является глобально уникальным. |
| Создание общедоступного IP-адреса |$pip = [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) -Name "myPublicIp" -ResourceGroupName $myResourceGroup -DomainNameLabel "myDNS" -Location $location -AllocationMethod Dynamic<BR><BR>В общедоступном IP-адресе используется доменное имя, которое вы ранее проверили и которое используется интерфейсной конфигурацией балансировщика нагрузки. |
| Создание интерфейсной IP-конфигурации |$frontendIP = [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) -Name "myFrontendIP" -PublicIpAddress $pip<BR><BR>Конфигурация внешнего интерфейса содержит общедоступный IP-адрес, созданный ранее для входящего сетевого трафика. |
| Создание внутреннего пула адресов |$beAddressPool = [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) -Name "myBackendAddressPool"<BR><BR>Предоставляет внутренние адреса для серверной части балансировщика нагрузки, доступной через сетевой интерфейс. |
| Создание пробы |$healthProbe = [New-AzLoadBalancerProbeConfig](/powershell/module/az.network/new-azloadbalancerprobeconfig) -Name "myProbe" -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2<BR><BR>Содержит пробы работоспособности, используемые для проверки доступности экземпляров виртуальных машин в пуле внутренних адресов. |
| Создание правила балансировки нагрузки |$lbRule = [New-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerruleconfig) -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80<BR><BR>Содержит правила, которые назначают общедоступный порт в балансировщике нагрузки порту во внутреннем пуле адресов. |
| Создание правила NAT для входящего трафика |$inboundNATRule = [New-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) -Name "myInboundRule1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389<BR><BR>Содержит правила сопоставления общедоступного порта в балансировщике нагрузки с портом конкретной виртуальной машины в пуле внутренних адресов. |
| Создание балансировщика нагрузки |$loadBalancer = [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) -ResourceGroupName $myResourceGroup -Name "myLoadBalancer" -Location $location -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule -LoadBalancingRule $lbRule -BackendAddressPool $beAddressPool -Probe $healthProbe |
| Создание сетевого интерфейса |$nic1= [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) -ResourceGroupName $myResourceGroup -Name "myNIC" -Location $location -PrivateIpAddress XX.X.X.X -Subnet $subnet2 -LoadBalancerBackendAddressPool $loadBalancer.BackendAddressPools[0] -LoadBalancerInboundNatRule $loadBalancer.InboundNatRules[0]<BR><BR>Создает сетевой интерфейс с использованием общедоступного IP-адреса и подсети виртуальной сети, созданной ранее. |

## <a name="get-information-about-network-resources"></a>Получение сведений о сетевых ресурсах

| Задача | Get-Help |
| ---- | ------- |
| Получение списка виртуальных сетей |[Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) -ResourceGroupName $myResourceGroup<BR><BR>Выводит список всех виртуальных сетей в группе ресурсов. |
| Получение сведений о виртуальной сети |Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup |
| Получение списка подсетей в виртуальной сети |Get-AzVirtualNetwork -Name "myVNet" -ResourceGroupName $myResourceGroup | Выбор подсетей |
| Получение сведений о подсети |[Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>Возвращает сведения о подсети в указанной виртуальной сети. Значение $vnet представляет собой объект, возвращаемый командлетом Get-AzVirtualNetwork. |
| Получение списка IP-адресов |[Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) -ResourceGroupName $myResourceGroup<BR><BR>Выводит список общедоступных IP-адресов в группе ресурсов. |
| Получение списка балансировщиков нагрузки |[Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer) -ResourceGroupName $myResourceGroup<BR><BR>Выводит список всех балансировщиков нагрузки в группе ресурсов. |
| Получение списка сетевых интерфейсов |[Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface) -ResourceGroupName $myResourceGroup<BR><BR>Выводит список всех виртуальных интерфейсов в группе ресурсов. |
| Получение сведений о виртуальном интерфейсе |Get-AzNetworkInterface -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>Получение сведений об определенном сетевом интерфейсе. |
| Получение IP-конфигурации сетевого интерфейса |[Get-AzNetworkInterfaceIPConfig](/powershell/module/az.network/get-aznetworkinterfaceipconfig) -Name "myNICIP" -NetworkInterface $nic<BR><BR>Возвращает сведения об IP-конфигурации указанного сетевого интерфейса. Значение $nic представляет собой объект, возвращаемый командлетом Get-AzNetworkInterface. |

## <a name="manage-network-resources"></a>Управление сетевыми ресурсами

| Задача | Get-Help |
| ---- | ------- |
| Добавление подсети в виртуальную сеть |[Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) -AddressPrefix XX.X.X.X/XX -Name "mySubnet1" -VirtualNetwork $vnet<BR><BR>Добавляет подсеть в существующую виртуальную сеть. Значение $vnet представляет собой объект, возвращаемый командлетом Get-AzVirtualNetwork. |
| Удаление виртуальной сети |[Remove-AzVirtualNetwork](/powershell/module/az.network/remove-azvirtualnetwork) -Name "myVNet" -ResourceGroupName $myResourceGroup<BR><BR>Удаляет указанную виртуальную сеть из группы ресурсов. |
| Удаление сетевого интерфейса |[Remove-AzNetworkInterface](/powershell/module/az.network/remove-aznetworkinterface) -Name "myNIC" -ResourceGroupName $myResourceGroup<BR><BR>Удаляет указанный сетевой интерфейс из группы ресурсов. |
| Удаление балансировщика нагрузки |[Remove-AzLoadBalancer](/powershell/module/az.network/remove-azloadbalancer) -Name "myLoadBalancer" -ResourceGroupName $myResourceGroup<BR><BR>Удаляет указанный балансировщик нагрузки из группы ресурсов. |
| Удаление общедоступного IP-адреса |[Remove-AzPublicIpAddress](/powershell/module/az.network/remove-azpublicipaddress)-Name "myIPAddress" -ResourceGroupName $myResourceGroup<BR><BR>Удаляет указанный общедоступный IP-адрес из группы ресурсов. |

## <a name="next-steps"></a>Next Steps
Использование созданного сетевого интерфейса при [создании виртуальной машины](./quick-create-powershell.md?toc=/azure/virtual-machines/windows/toc.json).
