---
title: Краткое руководство. Создание внутренней подсистемы балансировки нагрузки — Azure PowerShell
titleSuffix: Azure Load Balancer
description: В этом кратком руководстве показано, как создать внутреннюю подсистему балансировки нагрузки с помощью Azure PowerShell
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/27/2020
ms.author: allensu
ms.openlocfilehash: 618ca8722cef1959fddc5dcd24e8e8bdbb1f620a
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530043"
---
# <a name="quickstart-create-an-internal-load-balancer-to-load-balance-vms-using-azure-powershell"></a>Краткое руководство. Создание внутренней подсистемы балансировки нагрузки с помощью Azure PowerShell для распределения нагрузки между виртуальными машинами

Начните работу с Azure Load Balancer, создав с помощью Azure PowerShell внутреннюю подсистему балансировки нагрузки и две виртуальные машины.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Локальная установка Azure PowerShell или Azure Cloud Shell

Чтобы установить и использовать PowerShell локально, для работы с этой статьей вам понадобится модуль Azure PowerShell 5.4.1 или более поздней версии. Выполните командлет `Get-Module -ListAvailable Az`, чтобы узнать установленную версию. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-Az-ps). При использовании PowerShell на локальном компьютере также нужно запустить `Connect-AzAccount`, чтобы создать подключение к Azure.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

Создайте группу ресурсов с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

```azurepowershell-interactive
New-AzResourceGroup -Name 'CreateIntLBQS-rg' -Location 'eastus'
```
---

# <a name="standard-sku"></a>[**SKU "Стандартный"**](#tab/option-1-create-load-balancer-standard)

>[!NOTE]
>Для производственных рабочих нагрузок рекомендуется использовать подсистему балансировки нагрузки ценовой категории "Стандартный". Дополнительные сведения о доступных ценовых категориях см. в статье **[Номера SKU для Azure Load Balancer](skus.md)** .

В этом разделе показано, как создать подсистему балансировки нагрузки, которая распределяет нагрузку между виртуальными машинами. 

При создании внутренней подсистемы балансировки нагрузки виртуальная сеть настраивается в качестве сети для подсистемы балансировки нагрузки. 

На следующей схеме показаны ресурсы, созданные при работе с этим кратким руководством:

:::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/resources-diagram-internal.png" alt-text="Ресурсы подсистемы балансировки нагрузки ценовой категории &quot;Стандартный&quot;, созданные для работы с этим кратким руководством." border="false":::

## <a name="configure-virtual-network---standard"></a>Настройка виртуальной сети, категория "Стандартный"

Прежде чем развертывать виртуальные машины и тестировать подсистему балансировки нагрузки, создайте вспомогательные ресурсы виртуальной сети.

Создайте виртуальную сеть для серверных виртуальных машин.

Создайте группу безопасности сети для определения входящих подключений к виртуальной сети.

### <a name="create-virtual-network-network-security-group-and-bastion-host"></a>Создание виртуальной сети, группы безопасности сети и узла-бастиона

* Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

* Создайте правило группы безопасности сети с помощью командлета [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig):

* Создайте узел Бастиона Azure с помощью командлета [New-AzBastion](/powershell/module/az.network/new-azbastion):

* Создайте группу безопасности сети с помощью командлета [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

```azurepowershell-interactive
## Create backend subnet config ##
$subnet = @{
    Name = 'myBackendSubnet'
    AddressPrefix = '10.1.0.0/24'
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create Azure Bastion subnet. ##
$bastsubnet = @{
    Name = 'AzureBastionSubnet' 
    AddressPrefix = '10.1.1.0/24'
}
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig @bastsubnet

## Create the virtual network ##
$net = @{
    Name = 'myVNet'
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    AddressPrefix = '10.1.0.0/16'
    Subnet = $subnetConfig,$bastsubnetConfig
}
$vnet = New-AzVirtualNetwork @net

## Create public IP address for bastion host. ##
$ip = @{
    Name = 'myBastionIP'
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicip = New-AzPublicIpAddress @ip

## Create bastion host ##
$bastion = @{
    ResourceGroupName = 'CreateIntLBQS-rg'
    Name = 'myBastion'
    PublicIpAddress = $publicip
    VirtualNetwork = $vnet
}
New-AzBastion @bastion -AsJob

## Create rule for network security group and place in variable. ##
$nsgrule = @{
    Name = 'myNSGRuleHTTP'
    Description = 'Allow HTTP'
    Protocol = '*'
    SourcePortRange = '*'
    DestinationPortRange = '80'
    SourceAddressPrefix = 'Internet'
    DestinationAddressPrefix = '*'
    Access = 'Allow'
    Priority = '2000'
    Direction = 'Inbound'
}
$rule1 = New-AzNetworkSecurityRuleConfig @nsgrule

## Create network security group ##
$nsg = @{
    Name = 'myNSG'
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    SecurityRules = $rule1
}
New-AzNetworkSecurityGroup @nsg

```
## <a name="create-standard-load-balancer"></a>Создание подсистемы балансировки нагрузки (цен. категория "Стандартный")

В этом разделе описано, как создать и настроить следующие компоненты подсистемы балансировки нагрузки:

* Создайте с помощью командлета [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) интерфейсный IP-адрес для интерфейсного пула IP-адресов. Этот IP-адрес получает входящий трафик в подсистеме балансировки нагрузки.

* Создайте с помощью командлета [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) пул внутренних адресов для трафика, отправляемого из интерфейсного сервера подсистемы балансировки нагрузки. В этом пуле вы будете развертывать серверные виртуальные машины.

* Создайте с помощью командлета [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) пробу работоспособности, которая определяет работоспособность экземпляров серверной виртуальной машины.

* Создайте с помощью командлета [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) правило подсистемы балансировки нагрузки, которое определяет способ распределения трафика между виртуальными машинами.

* Теперь создайте общедоступную подсистему балансировки нагрузки с помощью командлета [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer).


```azurepowershell-interactive
## Place virtual network created in previous step into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreateIntLBQS-rg'

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
    ResourceGroupName = 'CreateIntLBQS-rg'
    Name = 'myLoadBalancer'
    Location = 'eastus'
    Sku = 'Standard'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
    LoadBalancingRule = $rule
    Probe = $healthprobe
}
New-AzLoadBalancer @loadbalancer

```

## <a name="create-virtual-machines---standard"></a>Создание стандартных виртуальных машин

В этом разделе вы создадите три виртуальные машины для внутреннего пула подсистемы балансировки нагрузки.

* Создайте три сетевых интерфейса с помощью командлета [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface).

* Укажите имя и пароль администратора для виртуальной машины с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential).

* Создайте виртуальные машины с помощью следующих командлетов:
    * [New-AzVM](/powershell/module/az.compute/new-azvm)
    * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
    * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
    * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
    * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)

```azurepowershell-interactive
# Set the administrator and password for the VMs. ##
$cred = Get-Credential

## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreateIntLBQS-rg'

## Place the load balancer into a variable. ##
$lb = @{
    Name = 'myLoadBalancer'
    ResourceGroupName = 'CreateIntLBQS-rg'
}
$bepool = Get-AzLoadBalancer @lb  | Get-AzLoadBalancerBackendAddressPoolConfig

## Place the network security group into a variable. ##
$nsg = Get-AzNetworkSecurityGroup -Name 'myNSG' -ResourceGroupName 'CreateIntLBQS-rg'

## For loop with variable to create virtual machines for load balancer backend pool. ##
for ($i=1; $i -le 3; $i++)
{
## Command to create network interface for VMs ##
$nic = @{
    Name = "myNicVM$i"
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
    NetworkSecurityGroup = $nsg
    LoadBalancerBackendAddressPool = $bepool
}
$nicVM = New-AzNetworkInterface @nic

## Create a virtual machine configuration for VMs ##
$vmsz = @{
    VMName = "myVM$i"
    VMSize = 'Standard_DS1_v2'  
}
$vmos = @{
    ComputerName = "myVM$i"
    Credential = $cred
}
$vmimage = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'    
}
$vmConfig = New-AzVMConfig @vmsz `
    | Set-AzVMOperatingSystem @vmos -Windows `
    | Set-AzVMSourceImage @vmimage `
    | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine for VMs ##
$vm = @{
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    VM = $vmConfig
    Zone = "$i"
}
New-AzVM @vm -AsJob
}

```

Развертывания виртуальных машин и узла-бастиона отправляются как задания PowerShell. Чтобы просмотреть состояние заданий, используйте команду [Get-Job](/powershell/module/microsoft.powershell.core/get-job).

```azurepowershell-interactive
Get-Job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzBastion
2      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
3      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
4      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

# <a name="basic-sku"></a>[**SKU "Базовый"**](#tab/option-1-create-load-balancer-basic)

>[!NOTE]
>Для производственных рабочих нагрузок рекомендуется использовать подсистему балансировки нагрузки ценовой категории "Стандартный". Дополнительные сведения о доступных ценовых категориях см. в статье **[Номера SKU для Azure Load Balancer](skus.md)** .

В этом разделе показано, как создать подсистему балансировки нагрузки, которая распределяет нагрузку между виртуальными машинами. 

При создании внутренней подсистемы балансировки нагрузки виртуальная сеть настраивается в качестве сети для подсистемы балансировки нагрузки. 

На следующей схеме показаны ресурсы, созданные при работе с этим кратким руководством:

:::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/resources-diagram-internal-basic.png" alt-text="Ресурсы подсистемы балансировки нагрузки ценовой категории &quot;Базовый&quot;, созданные при работе с этим кратким руководством." border="false":::

## <a name="configure-virtual-network---basic"></a>Настройка виртуальной сети, категория "Базовый"

Прежде чем развертывать виртуальные машины и тестировать подсистему балансировки нагрузки, создайте вспомогательные ресурсы виртуальной сети.

Создайте виртуальную сеть для серверных виртуальных машин.

Создайте группу безопасности сети для определения входящих подключений к виртуальной сети.

### <a name="create-virtual-network-network-security-group-and-bastion-host"></a>Создание виртуальной сети, группы безопасности сети и узла-бастиона

* Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork).

* Создайте правило группы безопасности сети с помощью командлета [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig):

* Создайте узел Бастиона Azure с помощью командлета [New-AzBastion](/powershell/module/az.network/new-azbastion):

* Создайте группу безопасности сети с помощью командлета [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

```azurepowershell-interactive
## Create backend subnet config ##
$subnet = @{
    Name = 'myBackendSubnet'
    AddressPrefix = '10.1.0.0/24'
}
$subnetConfig = New-AzVirtualNetworkSubnetConfig @subnet 

## Create Azure Bastion subnet. ##
$bastsubnet = @{
    Name = 'AzureBastionSubnet' 
    AddressPrefix = '10.1.1.0/24'
}
$bastsubnetConfig = New-AzVirtualNetworkSubnetConfig @bastsubnet

## Create the virtual network ##
$net = @{
    Name = 'myVNet'
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    AddressPrefix = '10.1.0.0/16'
    Subnet = $subnetConfig,$bastsubnetConfig
}
$vnet = New-AzVirtualNetwork @net

## Create public IP address for bastion host. ##
$ip = @{
    Name = 'myBastionIP'
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    Sku = 'Standard'
    AllocationMethod = 'Static'
}
$publicip = New-AzPublicIpAddress @ip

## Create bastion host ##
$bastion = @{
    ResourceGroupName = 'CreateIntLBQS-rg'
    Name = 'myBastion'
    PublicIpAddress = $publicip
    VirtualNetwork = $vnet
}
New-AzBastion @bastion -AsJob

## Create rule for network security group and place in variable. ##
$nsgrule = @{
    Name = 'myNSGRuleHTTP'
    Description = 'Allow HTTP'
    Protocol = '*'
    SourcePortRange = '*'
    DestinationPortRange = '80'
    SourceAddressPrefix = 'Internet'
    DestinationAddressPrefix = '*'
    Access = 'Allow'
    Priority = '2000'
    Direction = 'Inbound'
}
$rule1 = New-AzNetworkSecurityRuleConfig @nsgrule

## Create network security group ##
$nsg = @{
    Name = 'myNSG'
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    SecurityRules = $rule1
}
New-AzNetworkSecurityGroup @nsg

```
## <a name="create-basic-load-balancer"></a>Создание подсистемы балансировки нагрузки уровня "Базовый"

В этом разделе описано, как создать и настроить следующие компоненты подсистемы балансировки нагрузки:

* Создайте с помощью командлета [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) интерфейсный IP-адрес для интерфейсного пула IP-адресов. Этот IP-адрес получает входящий трафик в подсистеме балансировки нагрузки.

* Создайте с помощью командлета [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) пул внутренних адресов для трафика, отправляемого из интерфейсного сервера подсистемы балансировки нагрузки. В этом пуле вы будете развертывать серверные виртуальные машины.

* Создайте с помощью командлета [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) пробу работоспособности, которая определяет работоспособность экземпляров серверной виртуальной машины.

* Создайте с помощью командлета [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) правило подсистемы балансировки нагрузки, которое определяет способ распределения трафика между виртуальными машинами.

* Теперь создайте общедоступную подсистему балансировки нагрузки с помощью командлета [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer).

```azurepowershell-interactive
## Place virtual network created in previous step into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreateIntLBQS-rg'

## Create load balancer frontend configuration and place in variable. ##
$lbip = @{
    Name = 'myFrontEnd'
    PrivateIpAddress = '10.1.0.4'
    SubnetId = $vnet.subnets[0].Id
}
$feip = New-AzLoadBalancerFrontendIpConfig @lbip

## Create load balancer frontend configuration and place in variable. ##
$feip = New-AzLoadBalancerFrontendIpConfig -Name 'myFrontEnd' -PublicIpAddress $publicIp

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
$rule = New-AzLoadBalancerRuleConfig @lbrule

## Create the load balancer resource. ##
$loadbalancer = @{
    ResourceGroupName = 'CreateIntLBQS-rg'
    Name = 'myLoadBalancer'
    Location = 'eastus'
    Sku = 'Basic'
    FrontendIpConfiguration = $feip
    BackendAddressPool = $bePool
    LoadBalancingRule = $rule
    Probe = $healthprobe
}
New-AzLoadBalancer @loadbalancer

```

## <a name="create-virtual-machines---basic"></a>Создание базовых виртуальных машин

В этом разделе вы создадите виртуальные машины для внутреннего пула подсистемы балансировки нагрузки.

* Создайте три сетевых интерфейса с помощью командлета [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface).

* Укажите имя и пароль администратора для виртуальной машины с помощью командлета [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential).

* Создайте группу доступности для виртуальных машин с помощью командлета [New-AzAvailabilitySet](/powershell/module/az.compute/new-azvm):

* Создайте виртуальные машины с помощью следующих командлетов:
    * [New-AzVM](/powershell/module/az.compute/new-azvm)
    * [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
    * [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
    * [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
    * [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)

```azurepowershell-interactive
# Set the administrator and password for the VMs. ##
$cred = Get-Credential

## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreateIntLBQS-rg'

## Place the load balancer into a variable. ##
$lb = @{
    Name = 'myLoadBalancer'
    ResourceGroupName = 'CreateIntLBQS-rg'
}
$bepool = Get-AzLoadBalancer @lb  | Get-AzLoadBalancerBackendAddressPoolConfig

## Place the network security group into a variable. ##
$nsg = Get-AzNetworkSecurityGroup -Name 'myNSG' -ResourceGroupName 'CreateIntLBQS-rg'

## Create availability set for the virtual machines. ##
$set = @{
    Name = 'myAvailabilitySet'
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    Sku = 'Aligned'
    PlatformFaultDomainCount = '2'
    PlatformUpdateDomainCount =  '2'
}
$avs = New-AzAvailabilitySet @set

## For loop with variable to create virtual machines. ##
for ($i=1; $i -le 3; $i++)
{
## Command to create network interface for VMs ##
$nic = @{
    Name = "myNicVM$i"
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
    NetworkSecurityGroup = $nsg
    LoadBalancerBackendAddressPool = $bepool
}
$nicVM = New-AzNetworkInterface @nic

## Create a virtual machine configuration for VMs ##
$vmsz = @{
    VMName = "myVM$i"
    VMSize = 'Standard_DS1_v2'
    AvailabilitySetId = $avs.Id   
}
$vmos = @{
    ComputerName = "myVM$i"
    Credential = $cred
}
$vmimage = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'    
}
$vmConfig = New-AzVMConfig @vmsz `
    | Set-AzVMOperatingSystem @vmos -Windows `
    | Set-AzVMSourceImage @vmimage `
    | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine for VMs ##
$vm = @{
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    VM = $vmConfig
}
New-AzVM @vm -AsJob
}

```

Развертывания виртуальных машин и узла-бастиона отправляются как задания PowerShell. Чтобы просмотреть состояние заданий, используйте команду [Get-Job](/powershell/module/microsoft.powershell.core/get-job).

```azurepowershell-interactive
Get-Job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzBastion
2      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
3      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
4      Long Running O… AzureLongRunni… Completed     True            localhost            New-AzVM
```

[!INCLUDE [ephemeral-ip-note.md](../../includes/ephemeral-ip-note.md)]

---

## <a name="install-iis"></a>Установка служб IIS

Воспользуйтесь командлетом [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension), чтобы установить расширение пользовательских скриптов. 

Это расширение запускает `PowerShell Add-WindowsFeature Web-Server` для установки веб-сервера IIS, а затем обновляет страницу Default.htm для отображения имени узла виртуальной машины.

> [!IMPORTANT]
> Прежде чем продолжить убедитесь, что развертывания виртуальной машины, выполняемые на предыдущих шагах, завершились.  Используйте `Get-Job`, чтобы проверить состояние заданий развертывания виртуальной машины.

```azurepowershell-interactive
## For loop with variable to install custom script extension on virtual machines. ##
for ($i=1; $i -le 3; $i++)
{
$ext = @{
    Publisher = 'Microsoft.Compute'
    ExtensionType = 'CustomScriptExtension'
    ExtensionName = 'IIS'
    ResourceGroupName = 'CreateIntLBQS-rg'
    VMName = "myVM$i"
    Location = 'eastus'
    TypeHandlerVersion = '1.8'
    SettingString = '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}'
}
Set-AzVMExtension @ext -AsJob
}
```

Расширения развертываются как задания PowerShell. Чтобы просмотреть состояние заданий установки, используйте [Get-Job](/powershell/module/microsoft.powershell.core/get-job):


```azurepowershell-interactive
Get-Job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
8      Long Running O… AzureLongRunni… Running       True            localhost            Set-AzVMExtension
9      Long Running O… AzureLongRunni… Running       True            localhost            Set-AzVMExtension
10     Long Running O… AzureLongRunni… Running       True            localhost            Set-AzVMExtension
```


## <a name="test-the-load-balancer"></a>Тестирование подсистемы балансировки нагрузки

### <a name="create-virtual-machine"></a>Создание виртуальной машины

Создайте виртуальную машину с помощью следующих командлетов:

* [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface)
* [New-AzVM](/powershell/module/az.compute/new-azvm)
* [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig)
* [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem)
* [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage)
* [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface)

```azurepowershell-interactive
# Set the administrator and password for the VM. ##
$cred = Get-Credential

## Place the virtual network into a variable. ##
$vnet = Get-AzVirtualNetwork -Name 'myVNet' -ResourceGroupName 'CreateIntLBQS-rg'

## Place the network security group into a variable. ##
$nsg = Get-AzNetworkSecurityGroup -Name 'myNSG' -ResourceGroupName 'CreateIntLBQS-rg'

## Command to create network interface for VM ##
$nic = @{
    Name = "myNicTestVM"
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    Subnet = $vnet.Subnets[0]
    NetworkSecurityGroup = $nsg
}
$nicVM = New-AzNetworkInterface @nic

## Create a virtual machine configuration for VMs ##
$vmsz = @{
    VMName = "myTestVM"
    VMSize = 'Standard_DS1_v2' 
}
$vmos = @{
    ComputerName = "myTestVM"
    Credential = $cred
}
$vmimage = @{
    PublisherName = 'MicrosoftWindowsServer'
    Offer = 'WindowsServer'
    Skus = '2019-Datacenter'
    Version = 'latest'    
}
$vmConfig = New-AzVMConfig @vmsz `
    | Set-AzVMOperatingSystem @vmos -Windows `
    | Set-AzVMSourceImage @vmimage `
    | Add-AzVMNetworkInterface -Id $nicVM.Id

## Create the virtual machine for VMs ##
$vm = @{
    ResourceGroupName = 'CreateIntLBQS-rg'
    Location = 'eastus'
    VM = $vmConfig
}
New-AzVM @vm

```

### <a name="test"></a>Тест

1. [Войдите](https://portal.azure.com) на портал Azure.

1. Найдите частный IP-адрес для подсистемы балансировки нагрузки на экране **обзора**. В меню слева щелкните **Все службы**, выберите **Все ресурсы**, а затем — **myLoadBalancer**.

2. Запишите или скопируйте адрес рядом с **частным IP-адресом** на экране **обзора** **myLoadBalancer**.

3. В меню слева щелкните **Все службы**, выберите **Все ресурсы**, а затем в списке ресурсов выберите виртуальную машину **myTestVM**, расположенную в группе ресурсов **CreateIntLBQS-rg**.

4. На странице **Обзор** выберите **Подключиться** и **Бастион**.

6. Введите имя пользователя и пароль, введенные в процессе создания виртуальной машины.

7. На виртуальной машине **myTestVM** откройте браузер **Internet Explorer**.

8. Введите IP-адрес с предыдущего шага в адресную строку браузера. В браузере отобразится страница по умолчанию веб-сервера IIS.

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/load-balancer-test.png" alt-text="Создание внутренней подсистемы балансировки нагрузки уровня Стандартный" border="true":::
   
Чтобы подсистема балансировки нагрузки распределяла трафик между всеми тремя виртуальными машинами, вы можете настроить стандартную страницу веб-сервера IIS каждой из них, а затем принудительно обновить браузер с клиентского компьютера.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить группу ресурсов, подсистему балансировки нагрузки и все остальные ресурсы, можно воспользоваться командлетом [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
Remove-AzResourceGroup -Name 'CreateIntLBQS-rg'
```

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве:

* создать внутреннюю подсистему балансировки нагрузки уровня "Стандартный" или "Базовый";
* подключить к ней виртуальные машины; 
* настроить правило трафика подсистемы балансировки нагрузки и пробу работоспособности;
* тестировать подсистему балансировки нагрузки.

Чтобы узнать больше об Azure Load Balancer, ознакомьтесь со следующей статьей:
> [!div class="nextstepaction"]
> [Что такое Azure Load Balancer](load-balancer-overview.md)