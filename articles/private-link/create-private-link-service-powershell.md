---
title: Краткое руководство. Создание службы "Приватный канал Azure" с помощью Azure PowerShell
description: В этом кратком руководстве показано, как создать службу Приватного канала Azure с помощью Azure PowerShell.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.date: 01/24/2021
ms.author: allensu
ms.openlocfilehash: 366be37135808a6d3d5cc1a277e2de3e3d6da8ae
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102566411"
---
# <a name="quickstart-create-a-private-link-service-using-azure-powershell"></a>Краткое руководство. Создание службы Приватного канала с помощью Azure PowerShell

Приступите к созданию службы приватного канала, которая ссылается на вашу службу.  Предоставьте Приватному каналу доступ к службе или ресурсу, которые развернуты за Azure Load Balancer (цен. категория "Стандартный").  Пользователи службы имеют закрытый доступ из своей виртуальной сети.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure PowerShell или Azure Cloud Shell

Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```azurepowershell-interactive
New-AzResourceGroup -Name 'CreatePrivLinkService-rg' -Location 'eastus2'

```

## <a name="create-an-internal-load-balancer"></a>Создание внутреннего балансировщика нагрузки

В этом разделе показано, как создать виртуальную сеть и внутренний экземпляр Azure Load Balancer.

### <a name="virtual-network"></a>Виртуальная сеть

В этом разделе показано, как создать виртуальную сеть и подсеть для размещения подсистемы балансировки нагрузки, которая используется для доступа к службе приватного канала.

* Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

```azurepowershell-interactive
## Create backend subnet config ##
$subnet = @{
    Name = 'mySubnet'
    AddressPrefix = '10.1.0.0/24'
    PrivateLinkServiceNetworkPolicies = 'Disabled'
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create the virtual network ##
$net = @{
    Name = 'myVNet'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Location = 'eastus2'
    AddressPrefix = '10.1.0.0/16'
    Subnet = $subnetConfig
}
$vnet = New-AzVirtualNetwork @net

```

### <a name="create-standard-load-balancer"></a>Создание подсистемы балансировки нагрузки (цен. категория "Стандартный")

В этом разделе описано, как создать и настроить следующие компоненты подсистемы балансировки нагрузки:

* Создайте с помощью командлета [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) интерфейсный IP-адрес для интерфейсного пула IP-адресов. Этот IP-адрес получает входящий трафик в подсистеме балансировки нагрузки.

* Создайте с помощью командлета [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) пул внутренних адресов для трафика, отправляемого из интерфейсного сервера подсистемы балансировки нагрузки. В этом пуле вы будете развертывать серверные виртуальные машины.

* Создайте с помощью командлета [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) пробу работоспособности, которая определяет работоспособность экземпляров серверной виртуальной машины.

* Создайте с помощью командлета [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) правило подсистемы балансировки нагрузки, которое определяет способ распределения трафика между виртуальными машинами.

* Теперь создайте общедоступную подсистему балансировки нагрузки с помощью командлета [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer).


```azurepowershell-interactive
## Place virtual network created in previous step into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreatePrivLinkService-rg'

## Create load balancer frontend configuration and place in variable. ##
$lbip = @{
    Name = 'myFrontEnd'
    PrivateIpAddress = '10.1.0.4'
    SubnetId = $vnet.subnets[0].Id
}
$feip = New-AzLoadBalancerFrontendIpConfig @lbip

## Create backend address pool configuration and place in variable. ##
$bepool = New-AzLoadBalancerBackendAddressPoolConfig -Name 'myBackEndPool'

## Create the health probe and place in variable. ##
$probe = @{
    Name = 'myHealthProbe'
    Protocol = 'http'
    Port = '80'
    IntervalInSeconds = '360'
    ProbeCount = '5'
    RequestPath = '/'
}
$healthprobe = New-AzLoadBalancerProbeConfig @probe

## Create the load balancer rule and place in variable. ##
$lbrule = @{
    Name = 'myHTTPRule'
    Protocol = 'tcp'
    FrontendPort = '80'
    BackendPort = '80'
    IdleTimeoutInMinutes = '15'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
}
$rule = New-AzLoadBalancerRuleConfig @lbrule -EnableTcpReset

## Create the load balancer resource. ##
$loadbalancer = @{
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Name = 'myLoadBalancer'
    Location = 'eastus2'
    Sku = 'Standard'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
    LoadBalancingRule = $rule
    Probe = $healthprobe
}
New-AzLoadBalancer @loadbalancer

```

## <a name="create-a-private-link-service"></a>Создание службы "Приватный канал"

В этом разделе описывается, как создать службу "Приватный канал", которая использует созданный на предыдущем шаге Azure Load Balancer цен. категории "Стандартный.

* Создайте IP-конфигурацию службы "Приватный канал" с помощью командлета [New-AzPrivateLinkServiceIpConfig](/powershell/module/az.network/new-azprivatelinkserviceipconfig).

* Создайте службу "Приватный канал" с помощью командлета [New-AzPrivateLinkService](/powershell/module/az.network/new-azprivatelinkservice).

```azurepowershell
## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreatePrivLinkService-rg'

## Create the IP configuration for the private link service. ##
$ipsettings = @{
    Name = 'myIPconfig'
    PrivateIpAddress = '10.1.0.5'
    Subnet = $vnet.subnets[0]
}
$ipconfig = New-AzPrivateLinkServiceIpConfig @ipsettings

## Place the load balancer frontend configuration into a variable. ##
$par = @{
    Name = 'myLoadBalancer'
    ResourceGroupName = 'CreatePrivLinkService-rg'
}
$fe = Get-AzLoadBalancer @par | Get-AzLoadBalancerFrontendIpConfig

## Create the private link service for the load balancer. ##
$privlinksettings = @{
    Name = 'myPrivateLinkService'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Location = 'eastus2'
    LoadBalancerFrontendIpConfiguration = $fe
    IpConfiguration = $ipconfig
}
New-AzPrivateLinkService @privlinksettings

```

Ваша служба приватного канала создана и может принимать трафик. Если вы хотите просмотреть потоки трафика, настройте приложение за подсистемой Load Balancer ценовой категории "Стандартный".

## <a name="create-private-endpoint"></a>Создание частной конечной точки

В этом разделе вы сопоставите службу приватного канала с частной конечной точкой. Виртуальная сеть содержит частную конечную точку для службы приватного канала. Эта виртуальная сеть содержит ресурсы, которые будут обращаться к службе приватного канала.

### <a name="create-private-endpoint-virtual-network"></a>Создание виртуальной сети с частной конечной точкой

* Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

```azurepowershell-interactive
## Create backend subnet config ##
$subnet = @{
    Name = 'mySubnetPE'
    AddressPrefix = '11.1.0.0/24'
    PrivateEndpointNetworkPolicies = 'Disabled'
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create the virtual network ##
$net = @{
    Name = 'myVNetPE'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Location = 'eastus2'
    AddressPrefix = '11.1.0.0/16'
    Subnet = $subnetConfig
}
$vnetpe = New-AzVirtualNetwork @net

```

### <a name="create-endpoint-and-connection"></a>Создание конечной точки и подключения

* С помощью командлета [Get-AzPrivateLinkService](/powershell/module/az.network/get-azprivatelinkservice) поместите конфигурацию службы "Приватный канал", созданной ранее, в переменную для последующего использования.

* С помощью командлета [New-AzPrivateLinkServiceConnection](/powershell/module/az.network/new-azprivatelinkserviceconnection) создайте конфигурацию подключения.

* С помощью командлета [New-AzPrivateEndpoint](/powershell/module/az.network/new-azprivateendpoint) создайте конечную точку.



```azurepowershell-interactive
## Place the private link service configuration into variable. ##
$par1 = @{
    Name = 'myPrivateLinkService'
    ResourceGroupName = 'CreatePrivLinkService-rg'
}
$pls = Get-AzPrivateLinkService @par1

## Create the private link configuration and place in variable. ##
$par2 = @{
    Name = 'myPrivateLinkConnection'
    PrivateLinkServiceId = $pls.Id
}
$plsConnection = New-AzPrivateLinkServiceConnection @par2

## Place the virtual network into a variable. ##
$par3 = @{
    Name = 'myVNetPE'
    ResourceGroupName = 'CreatePrivLinkService-rg'
}
$vnetpe = Get-AzVirtualNetwork @par3

## Create private endpoint ##
$par4 = @{
    Name = 'MyPrivateEndpoint'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Location = 'eastus2'
    Subnet = $vnetpe.subnets[0]
    PrivateLinkServiceConnection = $plsConnection
}
New-AzPrivateEndpoint @par4 -ByManualRequest
```

### <a name="approve-the-private-endpoint-connection"></a>Утверждение подключения частной конечной точки

В этом разделе описывается, как утвердить подключение, созданное на предыдущих шагах.

* С помощью командлета [Approve-AzPrivateEndpointConnection](/powershell/module/az.network/approve-azprivateendpointconnection) выполните утверждение подключения.

```azurepowershell-interactive
## Place the private link service configuration into variable. ##
$par1 = @{
    Name = 'myPrivateLinkService'
    ResourceGroupName = 'CreatePrivLinkService-rg'
}
$pls = Get-AzPrivateLinkService @par1

$par2 = @{
    Name = $pls.PrivateEndpointConnections[0].Name
    ServiceName = 'myPrivateLinkService'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    Description = 'Approved'
}
Approve-AzPrivateEndpointConnection @par2

```

### <a name="ip-address-of-private-endpoint"></a>IP-адрес частной конечной точки

В этом разделе вы найдете IP-адрес частной конечной точки, которая соответствует Load Balancer и службе приватного канала.

* С помощью командлета [Get-AzPrivateEndpoint](/powershell/module/az.network/get-azprivateendpoint) получите IP-адрес.

```azurepowershell-interactive
## Get private endpoint and the IP address and place in a variable for display. ##
$par1 = @{
    Name = 'myPrivateEndpoint'
    ResourceGroupName = 'CreatePrivLinkService-rg'
    ExpandResource = 'networkinterfaces'
}
$pe = Get-AzPrivateEndpoint @par1

## Display the IP address by expanding the variable. ##
$pe.NetworkInterfaces[0].IpConfigurations[0].PrivateIpAddress
```

```bash
❯ $pe.NetworkInterfaces[0].IpConfigurations[0].PrivateIpAddress
11.1.0.4
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить группу ресурсов, подсистему балансировки нагрузки и все остальные ресурсы, можно воспользоваться командлетом [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'CreatePrivLinkService-rg'
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве показано, как:

* создать виртуальную сеть и внутренний экземпляр Azure Load Balancer;
* создать службу приватного канала.

Чтобы узнать больше о частной конечной точке Azure, ознакомьтесь со следующей статьей:
> [!div class="nextstepaction"]
> [Краткое руководство. Создание частной конечной точки с помощью Azure Powershell](create-private-endpoint-powershell.md)

