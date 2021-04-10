---
title: Настройка предпочтительного варианта маршрутизации для виртуальной машины — Azure PowerShell
description: Узнайте, как создать виртуальную машину с общедоступным IP-адресом и выбором предпочтительного варианта маршрутизации с помощью Azure PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2021
ms.author: mnayak
ms.openlocfilehash: 8ffa7ef7b3539e3cefc4e2f8621d7c67cd3b56b0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101672965"
---
# <a name="configure-routing-preference-for-a-vm-using-azure-powershell"></a>Настройка предпочтительного варианта маршрутизации для виртуальной машины с помощью Azure PowerShell

В этой статье показано, как настроить предпочтительный вариант маршрутизации для виртуальной машины. Связанный с Интернетом трафик от виртуальной машины будет направляться через сеть поставщика услуг Интернета при выборе параметра **Интернет** в качестве предпочтительного варианта маршрутизации. Маршрутизация по умолчанию осуществляется через глобальную сеть Майкрософт.

В этой статье показано, как создать виртуальную машину с общедоступным IP-адресом, настроенным для маршрутизации трафика через сеть поставщика услуг Интернета с помощью Azure PowerShell.

## <a name="create-a-resource-group"></a>Создание группы ресурсов
1. При использовании Cloud Shell перейдите к шагу 2. Откройте сеанс командной строки и войдите в Azure с помощью команды `Connect-AzAccount`.
2. Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов в регионе Azure "Восточная часть США".

    ```azurepowershell
    $rg = New-AzResourceGroup -Name MyResourceGroup -Location EastUS
    ```

## <a name="create-a-public-ip-address"></a>Создание общедоступного IP-адреса

Чтобы получить доступ к виртуальным машинам из Интернета, необходимы общедоступные IP-адреса. Создайте общедоступные IP-адреса с помощью командлета [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). В следующем примере создается общедоступный IP-адрес IPv4 с именем *MyPublicIP* и предпочтительным вариантом маршрутизации через *Интернет*, размещенный в группе ресурсов *MyResourceGroup* в *Восточном регионе США*:

```azurepowershell-interactive
$iptagtype="RoutingPreference"
$tagName = "Internet"
$ipTag = New-AzPublicIpTag -IpTagType $iptagtype -Tag $tagName 
# attach the tag
$publicIp = New-AzPublicIpAddress  `
-Name "MyPublicIP" `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location `
-IpTag $ipTag `
-AllocationMethod Static `
-Sku Standard `
-IpAddressVersion IPv4
```

## <a name="create-network-resources"></a>Создание сетевых ресурсов

Перед развертыванием виртуальной машины необходимо создать вспомогательные сетевые ресурсы — группу безопасности сети, виртуальную сеть и виртуальный сетевой адаптер.

### <a name="create-a-network-security-group"></a>Создание группы безопасности сети

Создайте группу безопасности сети с помощью командлета [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). В следующем примере создается группа безопасности сети с именем *myNSG*

```azurepowershell
$nsg = New-AzNetworkSecurityGroup `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "myNSG"
```

### <a name="create-a-virtual-network"></a>Создание виртуальной сети

Создайте виртуальную сеть с помощью командлета [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). В следующем примере создаются виртуальная сеть *myVNET* и подсеть *mySubNet*.

### <a name="create-a-subnet"></a>Создание подсети

```azurepowershell
$subnet = New-AzVirtualNetworkSubnetConfig `
-Name "mySubnet" `
-AddressPrefix "10.0.0.0/24"
```

```azurepowershell
# Create a virtual network
$vnet = New-AzVirtualNetwork `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-Name "myVNET" `
-AddressPrefix "10.0.0.0/16" `
-Subnet $subnet
```

### <a name="create-a-nic"></a>Создание сетевой карты

Create virtual NICs with [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface. В приведенном ниже примере создается виртуальный сетевой адаптер.

```azurepowershell
# Create an IP Config
$ipconfig=New-AzNetworkInterfaceIpConfig `
-Name myIpConfig `
-Subnet $vnet.subnets[0] `
-PrivateIpAddressVersion IPv4 `
-PublicIpAddress  $publicIp

# Create a NIC
$nic = New-AzNetworkInterface `
-Name "mynic" `
-ResourceGroupName $rg.ResourceGroupName `
-Location $rg.Location  `
-NetworkSecurityGroupId $nsg.Id `
-IpConfiguration $ipconfig 
```

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Укажите имя и пароль администратора для виртуальной машины с помощью командлета [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential):

```azurepowershell
 $cred = get-credential -Message "Routing Preference SAMPLE:  Please enter the Administrator credential to log into the VM."
```

Теперь вы можете создать виртуальную машину с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm). В приведенном ниже примере создаются две виртуальные машины и обязательные компоненты виртуальной сети, если их еще нет:

```azurepowershell
 $vmsize = "Standard_A2"
 $ImagePublisher = "MicrosoftWindowsServer"
 $imageOffer = "WindowsServer"
 $imageSKU = "2019-Datacenter"

 $vmName= "myVM"
 $vmconfig = New-AzVMConfig -VMName $vmName -VMSize $vmsize | Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred -ProvisionVMAgent -EnableAutoUpdate | Set-AzVMSourceImage -PublisherName $ImagePublisher -Offer $imageOffer -Skus $imageSKU -Version "latest" | Set-AzVMOSDisk -Name "$vmName.vhd" -CreateOption "FromImage" | Add-AzVMNetworkInterface -Id $nic.Id 
 $VM1 = New-AzVM -ResourceGroupName $rg.ResourceGroupName  -Location $rg.Location  -VM $vmconfig
```

## <a name="allow-network-traffic-to-the-vm"></a>Разрешение сетевого трафика к виртуальной машине

Прежде чем подключиться к общедоступному IP-адресу из Интернета, убедитесь, что в каждой группе безопасности сети, связанной с сетевым интерфейсом, открыты необходимые порты, подсеть, в которой находится сетевой интерфейс, или оба. Действующие правила безопасности для сетевого интерфейса и его подсети можно просмотреть с помощью [портала](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-portal), [CLI](diagnose-network-traffic-filter-problem.md#diagnose-using-azure-cli) или [PowerShell](diagnose-network-traffic-filter-problem.md#diagnose-using-powershell).

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ставшие ненужными группу ресурсов, виртуальную машину и все связанные с ней ресурсы, выполнив команду [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

 ```azurepowershell
 Remove-AzResourceGroup -Name MyResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [предпочтительном варианте маршрутизации в общедоступных IP-адресах](routing-preference-overview.md).
* См. дополнительные сведения об [общедоступных IP-адресах в Azure](./public-ip-addresses.md#public-ip-addresses).
* Дополнительные сведения о [параметрах общедоступного IP-адреса](virtual-network-public-ip-address.md#create-a-public-ip-address).