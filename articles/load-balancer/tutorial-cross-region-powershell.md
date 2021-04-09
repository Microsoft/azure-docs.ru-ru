---
title: Руководство по созданию подсистемы балансировки нагрузки между регионами с помощью Azure PowerShell
titleSuffix: Azure Load Balancer
description: В этом руководстве вы узнаете, как развернуть Azure Load Balancer между регионами с помощью Azure PowerShell.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: tutorial
ms.date: 02/10/2021
ms.openlocfilehash: 88e400cea764be84521c003a681aa74885dc29ce
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101721350"
---
# <a name="tutorial-create-a-cross-region-azure-load-balancer-using-azure-powershell"></a>Руководство по созданию Azure Load Balancer между регионами с помощью Azure PowerShell

Подсистема балансировки нагрузки в нескольких регионах обеспечивает глобальную доступность службы в разных регионах Azure. В случае сбоя одного региона его трафик направляется на подсистему балансировки нагрузки в ближайшем работоспособном регионе.  

В этом руководстве описано следующее.

> [!div class="checklist"]
> * создание подсистемы балансировки нагрузки в нескольких регионах;
> * Создайте правило балансировщика нагрузки.
> * создание серверного пула с двумя региональными подсистемами балансировки нагрузки;
> * тестирование подсистемы балансировки нагрузки.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure.
- Два Azure Load Balancer ценовой категории **Стандартный**, серверные пулы которых развернуты в двух разных регионах Azure.
    - Сведения о том, как создать региональную подсистему балансировки нагрузки ценовой категории "Стандартный" и виртуальные машины для серверных пулов см. в кратком руководстве [ Создание общедоступной подсистемы балансировки нагрузки для распределения нагрузки между виртуальными машинами с помощью Azure PowerShell](quickstart-load-balancer-standard-public-powershell.md).
        - К именам подсистем балансировки нагрузки и виртуальных машин в каждом регионе добавьте **-R1** и **-R2**. 
- Локальная установка Azure PowerShell или Azure Cloud Shell.


Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-cross-region-load-balancer"></a>Создание подсистемы балансировки нагрузки в нескольких регионах


### <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).


```azurepowershell-interactive
$rg = @{
    Name = 'MyResourceGroupLB-CR'
    Location = 'westus'
}
New-AzResourceGroup @rg

```

### <a name="create-cross-region-load-balancer-resources"></a>Создание ресурсов подсистемы балансировки нагрузки между регионами

В этом разделе показано, как создать ресурсы, необходимые для подсистемы балансировки нагрузки между регионами.

Для внешнего интерфейса подсистемы балансировки нагрузки между регионами используется глобальный общедоступный IP-адрес SKU уровня "Стандартный".

* Создайте общедоступный IP-адрес с помощью командлета [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress).

* Создайте конфигурацию интерфейсных IP-адресов с помощью командлета [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig).

* Создайте серверный пул адресов с помощью командлета [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig).

* Создайте правило подсистемы балансировки нагрузки с помощью командлета [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig).

* Создайте подсистему балансировки нагрузки между регионами с помощью командлета [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer).

```azurepowershell-interactive
## Create global IP address for load balancer ##
$ip = @{
    Name = 'myPublicIP-CR'
    ResourceGroupName = 'MyResourceGroupLB-CR'
    Location = 'westus'
    Sku = 'Standard'
    Tier = 'Global'
    AllocationMethod = 'Static'
}
$publicIP = New-AzPublicIpAddress @ip

## Create frontend configuration ##
$fe = @{
    Name = 'myFrontEnd-CR'
    PublicIpAddress = $publicIP
}
$feip = New-AzLoadBalancerFrontendIpConfig @fe

## Create back-end address pool ##
$be = @{
    Name = 'myBackEndPool-CR'
}
$bepool = New-AzLoadBalancerBackendAddressPoolConfig @be

## Create the load balancer rule ##
$rul = @{
    Name = 'myHTTPRule-CR'
    Protocol = 'tcp'
    FrontendPort = '80'
    BackendPort = '80'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bepool
}
$rule = New-AzLoadBalancerRuleConfig @rul

## Create cross-region load balancer resource ##
$lbp = @{
    ResourceGroupName = 'myResourceGroupLB-CR'
    Name = 'myLoadBalancer-CR'
    Location = 'westus'
    Sku = 'Standard'
    Tier = 'Global'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bepool
    LoadBalancingRule = $rule
}
$lb = New-AzLoadBalancer @lbp
```

## <a name="configure-backend-pool"></a>Настройка внутреннего пула

В этом разделе показано, как добавить в серверный пул две подсистемы балансировки нагрузки ценовой категории "Стандартный".

> [!IMPORTANT]
> Чтобы выполнить эти действия, в подписке должны быть развернуты две региональные подсистемы балансировки нагрузки с серверными пулами.  Дополнительные сведения см. в кратком руководстве **[ Создание общедоступной подсистемы балансировки нагрузки для распределения нагрузки между виртуальными машинами с помощью Azure PowerShell](quickstart-load-balancer-standard-public-powershell.md)** .

* Используйте командлеты [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer) и [Get-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/get-azloadbalancerfrontendipconfig), чтобы сохранить сведения о региональных подсистемах балансировки нагрузки в переменных.

* Используйте командлет [New-AzLoadBalancerBackendAddressConfig](/powershell/module/az.network/new-azloadbalancerbackendaddressconfig), чтобы создать конфигурации внутреннего пула адресов для подсистемы балансировки нагрузки.

* Используйте командлет [Set-AzLoadBalancerBackendAddressPool](/powershell/module/az.network/new-azloadbalancerbackendaddresspool), чтобы добавить внешний интерфейс региональной подсистемы балансировки нагрузки во внутренний пул между регионами.

```azurepowershell-interactive
## Place the region one load balancer configuration in a variable ##
$region1 = @{
    Name = 'myLoadBalancer-R1'
    ResourceGroupName = 'CreatePubLBQS-rg-r1'
}
$R1 = Get-AzLoadBalancer @region1

## Place the region two load balancer configuration in a variable ##
$region2 = @{
    Name = 'myLoadBalancer-R2'
    ResourceGroupName = 'CreatePubLBQS-rg-r2'
}
$R2 = Get-AzLoadBalancer @region2

## Place the region one load balancer front-end configuration in a variable ##
$region1fe = @{
    Name = 'MyFrontEnd-R1'
    LoadBalancer = $R1
}
$R1FE = Get-AzLoadBalancerFrontendIpConfig @region1fe

## Place the region two load balancer front-end configuration in a variable ##
$region2fe = @{
    Name = 'MyFrontEnd-R2'
    LoadBalancer = $R2
}
$R2FE = Get-AzLoadBalancerFrontendIpConfig @region2fe

## Create the cross-region backend address pool configuration for region 1 ##
$region1ap = @{
    Name = 'MyBackendPoolConfig-R1'
    LoadBalancerFrontendIPConfigurationId = $R1FE.Id
}
$beaddressconfigR1 = New-AzLoadBalancerBackendAddressConfig @region1ap

## Create the cross-region backend address pool configuration for region 2 ##
$region2ap = @{
    Name = 'MyBackendPoolConfig-R2'
    LoadBalancerFrontendIPConfigurationId = $R2FE.Id
}
$beaddressconfigR2 = New-AzLoadBalancerBackendAddressConfig @region2ap

## Apply the backend address pool configuration for the cross-region load balancer ##
$bepoolcr = @{
    ResourceGroupName = 'myResourceGroupLB-CR'
    LoadBalancerName = 'myLoadBalancer-CR'
    Name = 'myBackEndPool-CR'
    LoadBalancerBackendAddress = $beaddressconfigR1,$beaddressconfigR2
}
Set-AzLoadBalancerBackendAddressPool @bepoolcr

```

## <a name="test-the-load-balancer"></a>Тестирование подсистемы балансировки нагрузки

В этом разделе вы проверите подсистему балансировки нагрузки в нескольких регионах. С помощью веб-браузера вы подключитесь к общедоступному IP-адресу.  Вы остановите работу виртуальных машин в одном из серверных пулов подсистемы балансировки нагрузки, а затем отследите выполнение отработки отказа.

1. Используйте командлет [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress), чтобы получить сведения об общедоступном IP-адресе подсистемы балансировки нагрузки:

```azurepowershell-interactive
$ip = @{
    Name = 'myPublicIP-CR'
    ResourceGroupName = 'myResourceGroupLB-CR'
}  
Get-AzPublicIPAddress @ip | select IpAddress

```
2. Скопируйте общедоступный IP-адрес и вставьте его в адресную строку браузера. В браузере отобразится страница по умолчанию веб-сервера IIS.

3. Остановите все виртуальные машины в серверном пуле одной из региональных подсистем балансировки нагрузки.

4. Обновите страницу в веб-браузере и убедитесь, что отработка отказа в другую региональную подсистему балансировки нагрузки выполнена успешно.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить группу ресурсов, подсистему балансировки нагрузки и все остальные ресурсы, можно воспользоваться командлетом [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'myResourceGroupLB-CR'
```

## <a name="next-steps"></a>Дальнейшие действия

Изучив это руководство, вы:

* создали глобальный IP-адрес;
* создали подсистему балансировки нагрузки в нескольких регионах;
* создали правило балансировки нагрузки;
* добавили региональные подсистемы балансировки нагрузки в серверный пул подсистемы балансировки нагрузки в нескольких регионах;
* тестировать подсистему балансировки нагрузки.


Дополнительные сведения см. в следующей статье:
> [!div class="nextstepaction"]
> [Виртуальные машины, использующие подсистему балансировки нагрузки в пределах зон доступности](tutorial-load-balancer-standard-public-zone-redundant-portal.md)