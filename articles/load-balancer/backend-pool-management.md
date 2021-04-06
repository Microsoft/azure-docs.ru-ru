---
title: Управление внутренним пулом
titleSuffix: Azure Load Balancer
description: Базовые сведения о настройке внутреннего пула Azure Load Balancer и управлении им
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 01/28/2021
ms.author: allensu
ms.openlocfilehash: c49a721a4db758965c9cf8d71f5d73b5754b6088
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104654481"
---
# <a name="backend-pool-management"></a>Управление внутренним пулом
Внутренний пул является важнейшим компонентом подсистемы балансировки нагрузки. Внутренний пул определяет группу ресурсов, которая будет обрабатывать трафик по определенному правилу балансировки нагрузки.

У вас есть два способа настроить внутренний пул:
* с помощью сетевой карты;
* с использованием сочетания IP-адреса и идентификатора ресурса виртуальной сети.

Для настройки внутреннего пула следует выбрать сетевую карту, если вы используете существующие виртуальные машины и масштабируемые наборы виртуальных машин. Этот метод создает наиболее прямую связь между ресурсом и внутренним пулом. 

При предварительном выделении внутреннего пула с диапазоном IP-адресов, который позже будет использоваться вами для создания виртуальных машин и масштабируемых наборов виртуальных машин, настройте внутренний пул по сочетанию IP-адресов и идентификаторов виртуальных сетей.

Вы можете настроить несколько серверных пулов на основе IP-адресов и сетевых карт для одной и той же подсистемы балансировки нагрузки, но нельзя создать один серверный пул с поддерживаемыми адресами, которые используются для сетевой карты и IP-адресов.

В разделах этой статьи, посвященных настройке, рассматриваются следующие темы:

* Azure PowerShell
* Azure CLI
* REST API
* Шаблоны Azure Resource Manager 

В этих разделах представлены сведения о структуре внутренних пулов для каждого варианта конфигурации.

## <a name="configuring-backend-pool-by-nic"></a>Настройка внутреннего пула по сетевой карте
Внутренний пул создается в рамках работы подсистемы балансировки нагрузки. Свойство IP-конфигурации для сетевой карты позволяет добавлять члены внутреннего пула.

Следующие примеры демонстрируют создание и заполнение внутреннего пула, позволяя лучше понять рабочий процесс и соответствующие отношения.

  >[!NOTE] 
  >Важно отметить, что настроенные через сетевой интерфейс внутренние пулы нельзя обновлять в рамках операций с серверным пулом. Любое добавление или удаление внутренних ресурсов должно выполняться в сетевом интерфейсе ресурса.

### <a name="powershell"></a>PowerShell
Создайте новый внутренний пул:
 
```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"

$backendPool = 
New-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -BackendAddressPoolName $backendPoolName  
```

Создайте новый сетевой интерфейс и добавьте его во внутренний пул:

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"
$nicname = "myNic"
$location = "eastus"
$vnetname = <your-vnet-name>

$vnet = 
Get-AzVirtualNetwork -Name $vnetname -ResourceGroupName $resourceGroup

$nic = 
New-AzNetworkInterface -ResourceGroupName $resourceGroup -Location $location -Name $nicname -LoadBalancerBackendAddressPool $backendPoolName -Subnet $vnet.Subnets[0]
```

Получите сведения о внутреннем пуле для подсистемы балансировки нагрузки и убедитесь, что этот сетевой интерфейс добавлен во внутренний пул:

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"

$lb =
Get-AzLoadBalancer -ResourceGroupName $res
Get-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -BackendAddressPoolName $backendPoolName 
```

Создайте новую виртуальную машину и подключите к ней сетевой интерфейс, чтобы поместить ее во внутренний пул:

```azurepowershell-interactive
# Create a username and password for the virtual machine
$cred = Get-Credential

# Create a virtual machine configuration
$vmname = "myVM1"
$vmsize = "Standard_DS1_v2"
$pubname = "MicrosoftWindowsServer"
$nicname = "myNic"
$off = "WindowsServer"
$sku = "2019-Datacenter"
$resourceGroup = "myResourceGroup"
$location = "eastus"

$nic =
Get-AzNetworkInterface -Name $nicname -ResourceGroupName $resourceGroup

$vmConfig = 
New-AzVMConfig -VMName $vmname -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmname -Credential $cred | Set-AzVMSourceImage -PublisherName $pubname -Offer $off -Skus $sku -Version latest | Add-AzVMNetworkInterface -Id $nic.Id
 
# Create a virtual machine using the configuration
$vm1 = New-AzVM -ResourceGroupName $resourceGroup -Zone 1 -Location $location -VM $vmConfig
```

### <a name="cli"></a>CLI
Создайте внутренний пул:

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool 
```

Создайте новый сетевой интерфейс и добавьте его во внутренний пул:

```azurecli-interactive
az network nic create \
--resource-group myResourceGroup \
--name myNic \
--vnet-name myVnet \
--subnet mySubnet \
--network-security-group myNetworkSecurityGroup \
--lb-name myLB \
--lb-address-pools myBackEndPool
```

Получите внутренний пул и убедитесь, что IP-адрес добавлен правильно:

```azurecli-interactive
az network lb address-pool show \
--resource-group myResourceGroup \
--lb-name myLb \
--name myBackendPool
```

Создайте новую виртуальную машину и подключите к ней сетевой интерфейс, чтобы поместить ее во внутренний пул:

```azurecli-interactive
az vm create \
--resource-group myResourceGroup \
--name myVM \
--nics myNic \
--image UbuntuLTS \
--admin-username azureuser \
--generate-ssh-keys
```

### <a name="resource-manager-template"></a>Шаблон Resource Manager

Выполните инструкции из этого [краткого руководства по применению шаблона Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-load-balancer-standard-create/), чтобы развернуть подсистему балансировки нагрузки и виртуальные машины, а затем добавить эти виртуальные машины во внутренний пул с помощью сетевого интерфейса.

Воспользуйтесь этим [шаблоном быстрого запуска для Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-load-balancer-ip-configured-backend-pool), чтобы развернуть подсистему балансировки нагрузки и виртуальные машины, а затем добавить эти виртуальные машины во внутренний пул с помощью IP-адреса.


## <a name="configure-backend-pool-by-ip-address-and-virtual-network"></a>Настройка внутреннего пула по IP-адресу и виртуальной сети
В сценариях с использованием предварительно заполненных внутренних пулов следует применять сочетание IP-адресов и виртуальной сети.

Все управление внутренним пулом выполняется непосредственно на объекте внутреннего пула, как видно из приведенных ниже примеров.

### <a name="powershell"></a>PowerShell
Создайте внутренний пул:

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$loadBalancerName = "myLoadBalancer"
$backendPoolName = "myBackendPool"
$vnetName = "myVnet"
$location = "eastus"
$nicName = "myNic"

$backendPool = New-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -Name $backendPoolName  
```

Присвойте внутреннему пулу новый IP-адрес из существующей виртуальной сети:
 
```azurepowershell-interactive
$virtualNetwork = 
Get-AzVirtualNetwork -Name $vnetName -ResourceGroupName $resourceGroup 
 
$ip1 = New-AzLoadBalancerBackendAddressConfig -IpAddress "10.0.0.5" -Name "TestVNetRef" -VirtualNetwork $virtualNetwork  
 
$backendPool.LoadBalancerBackendAddresses.Add($ip1) 

Set-AzLoadBalancerBackendAddressPool -InputObject $backendPool
```

Получите сведения о внутреннем пуле для подсистемы балансировки нагрузки и убедитесь, что внутренние адреса добавлены во внутренний пул:

```azurepowershell-interactive
Get-AzLoadBalancerBackendAddressPool -ResourceGroupName $resourceGroup -LoadBalancerName $loadBalancerName -Name $backendPoolName 
```
Создайте сетевой интерфейс и добавьте его во внутренний пул. Присвойте IP-адрес одному из внутренних адресов:

```azurepowershell-interactive
$nic = 
New-AzNetworkInterface -ResourceGroupName $resourceGroup -Location $location -Name $nicName -PrivateIpAddress 10.0.0.4 -Subnet $virtualNetwork.Subnets[0]
```

Создайте виртуальную машину и подключите к ней сетевую карту с IP-адресом из внутреннего пула:
```azurepowershell-interactive
# Create a username and password for the virtual machine
$cred = Get-Credential

# Create a virtual machine configuration
$vmname = "myVM1"
$vmsize = "Standard_DS1_v2"
$pubname = "MicrosoftWindowsServer"
$nicname = "myNic"
$off = "WindowsServer"
$sku = "2019-Datacenter"
$resourceGroup = "myResourceGroup"
$location = "eastus"

$nic =
Get-AzNetworkInterface -Name $nicname -ResourceGroupName $resourceGroup

$vmConfig = 
New-AzVMConfig -VMName $vmname -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmname -Credential $cred | Set-AzVMSourceImage -PublisherName $pubname -Offer $off -Skus $sku -Version latest | Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine using the configuration
$vm1 = New-AzVM -ResourceGroupName $resourceGroup -Zone 1 -Location $location -VM $vmConfig
```

### <a name="cli"></a>CLI
С помощью CLI вы можете заполнить внутренний пул, указав аргументы командной строки либо применив файл конфигурации в формате JSON. 

Создайте и заполните внутренний пул с помощью аргументов командной строки:

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool \
--vnet {VNET resource ID} \
--backend-address name=addr1 ip-address=10.0.0.4 \
--backend-address name=addr2 ip-address=10.0.0.5
```

Создайте и заполните внутренний пул, используя файл конфигурации в формате JSON:

```azurecli-interactive
az network lb address-pool create \
--resource-group myResourceGroup \
--lb-name myLB \
--name myBackendPool \
--vnet {VNET resource ID} \
--backend-address-config-file @config_file.json
```

Файл конфигурации в формате JSON:
```JSON
        [
          {
            "name": "address1",
            "virtualNetwork": "/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}",
            "ipAddress": "10.0.0.4"
          },
          {
            "name": "address2",
            "virtualNetwork": "/subscriptions/{subscriptionId}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/virtualNetworks/{vnet-name}",
            "ipAddress": "10.0.0.5"
          }
        ]
```

Получите сведения о внутреннем пуле для подсистемы балансировки нагрузки и убедитесь, что внутренние адреса добавлены во внутренний пул:

```azurecli-interactive
az network lb address-pool show \
--resource-group myResourceGroup \
--lb-name MyLb \
--name MyBackendPool
```

Создайте сетевой интерфейс и добавьте его во внутренний пул. Присвойте IP-адрес одному из внутренних адресов:

```azurecli-interactive
az network nic create \
  --resource-group myResourceGroup \
  --name myNic \
  --vnet-name myVnet \
  --subnet mySubnet \
  --network-security-group myNetworkSecurityGroup \
  --lb-name myLB \
  --private-ip-address 10.0.0.4
```

Создайте виртуальную машину и подключите к ней сетевую карту с IP-адресом из внутреннего пула:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --nics myNic \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```
 
### <a name="limitations"></a>Ограничения
Внутренний пул, настроенный с использованием IP-адреса, имеет следующие ограничения:
  * Может использоваться только для Load Balancer цен. категории "Стандартный".
  * внутренний пул поддерживает до 100 IP-адресов;
  * ресурсы внутреннего пула должны находиться в той же виртуальной сети, что и подсистема балансировки нагрузки;
  * Load Balancer с внутренним пулом на основе IP-адресов не может функционировать как служба Приватного канала.
  * пока эта возможность не поддерживается на портале Azure;
  * контейнеры ACI в настоящее время этой функцией не поддерживаются;
  * подсистемы балансировки нагрузки или службы, такие как Шлюз приложений, нельзя помещать во внутренний пул подсистемы балансировки нагрузки;
  * Правила NAT для входящего трафика не могут определяться IP-адресом.

## <a name="next-steps"></a>Дальнейшие действия
Из этой статьи вы узнали об управлении внутренним пулом Azure Load Balancer и о настройке внутреннего пула с использованием IP-адреса и виртуальной сети.

[Дополнительные сведения об Azure Load Balancer](load-balancer-overview.md).

Проверьте [REST API](https://docs.microsoft.com/rest/api/load-balancer/loadbalancerbackendaddresspools/createorupdate) для управления серверным пулом на основе IP-адресов.
