---
title: Балансировка нагрузки в конфигурациях с несколькими IP-адресами — Azure CLI
titleSuffix: Azure Load Balancer
description: Из этой статьи вы узнаете о балансировке нагрузки в основной и дополнительной IP-конфигурациях с помощью Azure CLI.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18, devx-track-azurecli
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: allensu
ms.openlocfilehash: a2916f28be0b45eec6e9c1a85c0b8db3fb611381
ms.sourcegitcommit: df1930c9fa3d8f6592f812c42ec611043e817b3b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/13/2021
ms.locfileid: "103417879"
---
# <a name="load-balancing-on-multiple-ip-configurations-using-powershell"></a>Балансировка нагрузки в конфигурациях с несколькими IP-адресами с помощью PowerShell

> [!div class="op_single_selector"]
> * [Портал](load-balancer-multiple-ip.md)
> * [CLI](load-balancer-multiple-ip-cli.md)
> * [PowerShell](load-balancer-multiple-ip-powershell.md)

В этой статье описывается, как использовать Azure Load Balancer в конфигурации, когда каждому дополнительному сетевому интерфейсу (сетевой карты) назначено несколько IP-адресов. В этом сценарии у нас есть две виртуальные машины под управлением Windows, оснащенные основной и дополнительной сетевыми картами. В каждом из дополнительных сетевых интерфейсов есть по две IP-конфигурации. На каждой виртуальной машине размещены веб-сайты contoso.com и fabrikam.com. Каждый веб-сайт привязан к одной из IP-конфигураций дополнительной сетевой карты. Мы используем Azure Load Balancer, чтобы предоставить два интерфейсных IP-адреса, по одному для каждого веб-сайта. Это позволит направлять трафик в соответствующую IP-конфигурацию для веб-сайта. В данном сценарии используется одинаковый номер порта для обоих внешних интерфейсов, как и для обоих IP-адресов внутренних пулов.

![Схема балансировки нагрузки для сценария](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

## <a name="steps-to-load-balance-on-multiple-ip-configurations"></a>Инструкции по балансировке нагрузки в конфигурациях с несколькими IP-адресами

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Выполните следующие действия, чтобы реализовать сценарий, описанный в этой статье.

1. Установите Azure PowerShell. Сведения об установке последней версии Azure PowerShell, а также о выборе нужной подписки и входе в учетную запись Azure см. в статье [Как установить и настроить службы Azure PowerShell](/powershell/azure/).
2. Создайте группу ресурсов, указав приведенные ниже параметры.

    ```powershell
    $location = "westcentralus".
    $myResourceGroup = "contosofabrikam"
    ```

    Чтобы узнать больше, ознакомьтесь с шагом 2 в разделе [Создание группы ресурсов](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm?toc=%2fazure%2fload-balancer%2ftoc.json).

3. [Создайте группу доступности](../virtual-machines/windows/tutorial-availability-sets.md?toc=%2fazure%2fload-balancer%2ftoc.json) для виртуальных машин. В рамках данного сценария воспользуйтесь следующей командой.

    ```powershell
    New-AzAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset" -Location "West Central US"
    ```

4. Следуйте инструкциям для шагов 3–5 в разделе [Создание виртуальной машины Windows](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm?toc=%2fazure%2fload-balancer%2ftoc.json), чтобы подготовить среду к созданию виртуальной машины с одной сетевой картой. Выполните шаг 6.1, затем вместо шага 6.2 выполните следующую команду.

    ```powershell
    $availset = Get-AzAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset"
    New-AzVMConfig -VMName "VM1" -VMSize "Standard_DS1_v2" -AvailabilitySetId $availset.Id
    ```

    После этого выполните шаги 6.3–6.8 раздела [Создание виртуальной машины Windows](/previous-versions/azure/virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm?toc=%2fazure%2fload-balancer%2ftoc.json).

5. Добавьте вторую IP-конфигурации в каждую виртуальную машину. Следуйте инструкциям в статье [Назначение виртуальным машинам нескольких IP-адресов](../virtual-network/virtual-network-multiple-ip-addresses-powershell.md#add). Используйте следующие параметры конфигурации.

    ```powershell
    $NicName = "VM1-NIC2"
    $RgName = "contosofabrikam"
    $NicLocation = "West Central US"
    $IPConfigName4 = "VM1-ipconfig2"
    $Subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $myVnet
    ```

    В рамках данного руководства нет необходимости связывать дополнительные IP-конфигурации с общедоступными IP-адресами. Измените команду, чтобы удалить связывание с общедоступным IP-адресом.

6. Повторите шаги 4–6 данной статьи для VM2. Не забудьте заменить в командах VM на VM2. Обратите внимание, что для второй виртуальной машины не нужно создавать виртуальную сеть. Вы можете создать или не создавать новую подсеть в зависимости от требований ситуации.

7. Создайте два общедоступных IP-адреса и сохраните их в соответствующих переменных, как показано ниже.

    ```powershell
    $publicIP1 = New-AzPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel contoso
    $publicIP2 = New-AzPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel fabrikam

    $publicIP1 = Get-AzPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam
    $publicIP2 = Get-AzPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam
    ```

8. Создайте две интерфейсные IP-конфигурации.

    ```powershell
    $frontendIP1 = New-AzLoadBalancerFrontendIpConfig -Name contosofe -PublicIpAddress $publicIP1
    $frontendIP2 = New-AzLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2
    ```

9. Создайте внутренние пулы адресов, пробу и правила балансировки нагрузки.

    ```powershell
    $beaddresspool1 = New-AzLoadBalancerBackendAddressPoolConfig -Name contosopool
    $beaddresspool2 = New-AzLoadBalancerBackendAddressPoolConfig -Name fabrikampool

    $healthProbe = New-AzLoadBalancerProbeConfig -Name HTTP -RequestPath 'index.html' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

    $lbrule1 = New-AzLoadBalancerRuleConfig -Name HTTPc -FrontendIpConfiguration $frontendIP1 -BackendAddressPool $beaddresspool1 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    $lbrule2 = New-AzLoadBalancerRuleConfig -Name HTTPf -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    ```

10. После создания этих ресурсов создайте подсистему балансировки нагрузки.

    ```powershell
    $mylb = New-AzLoadBalancer -ResourceGroupName contosofabrikam -Name mylb -Location 'West Central US' -FrontendIpConfiguration $frontendIP1 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe
    ```

11. Добавьте второй внутренний пул адресов и интерфейсную IP-конфигурацию в созданную подсистему балансировки нагрузки.

    ```powershell
    $mylb = Get-AzLoadBalancer -Name "mylb" -ResourceGroupName $myResourceGroup | Add-AzLoadBalancerBackendAddressPoolConfig -Name fabrikampool | Set-AzLoadBalancer

    $mylb | Add-AzLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2 | Set-AzLoadBalancer
    
    Add-AzLoadBalancerRuleConfig -Name HTTP -LoadBalancer $mylb -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80 | Set-AzLoadBalancer
    ```

12. Приведенные ниже команды используются для получения сетевых карт и добавления обеих IP-конфигураций каждой дополнительной сетевой карты во внутренний пул адресов подсистемы балансировки нагрузки.

    ```powershell
    $nic1 = Get-AzNetworkInterface -Name "VM1-NIC2" -ResourceGroupName "MyResourcegroup";
    $nic2 = Get-AzNetworkInterface -Name "VM2-NIC2" -ResourceGroupName "MyResourcegroup";

    $nic1.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic1.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);
    $nic2.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic2.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);

    $mylb = $mylb | Set-AzLoadBalancer

    $nic1 | Set-AzNetworkInterface
    $nic2 | Set-AzNetworkInterface
    ```

13. Наконец, необходимо настроить записи ресурсов DNS, чтобы они указывали на соответствующие интерфейсные IP-адреса подсистемы балансировки нагрузки. Домены можно разместить в Azure DNS. Дополнительные сведения об использовании Azure DNS с подсистемой балансировки нагрузки см. в разделе [Использование Azure DNS с другими службами Azure](../dns/dns-for-azure-services.md).

## <a name="next-steps"></a>Дальнейшие действия
- Узнайте больше о том, как объединять службы балансировки нагрузки, в статье [Использование служб балансировки нагрузки в Azure](../traffic-manager/traffic-manager-load-balancing-azure.md).
- Узнайте, как можно использовать различные типы журналов в Azure для управления подсистемой балансировки нагрузки и устранения неполадок в [журналах Azure Monitor для Azure Load Balancer](../load-balancer/load-balancer-monitor-log.md).