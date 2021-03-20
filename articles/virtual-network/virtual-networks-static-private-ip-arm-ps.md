---
title: Создание виртуальной машины со статическим частным IP-адресом — Azure PowerShell
description: Узнайте, как создать виртуальную машину с частным IP-адресом с помощью PowerShell.
services: virtual-network
documentationcenter: na
author: KumudD
editor: ''
tags: azure-resource-manager
ms.assetid: d5f18929-15e3-40a2-9ee3-8188bc248ed8
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/07/2019
ms.author: kumud
ms.custom: ''
ms.openlocfilehash: 43c2e2c8905ca60e59caed3fdf1b4920f509fd88
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98783790"
---
# <a name="create-a-virtual-machine-with-a-static-private-ip-address-using-powershell"></a>Создание виртуальной машины со статическим частным IP-адресом с помощью PowerShell

Вы можете создать виртуальную машину со статическим частным IP-адресом. Назначьте статический частный IP-адрес, а не динамический адрес, если вы хотите выбрать адрес из подсети, назначенный виртуальной машине. Дополнительные сведения о [статических частных IP-адресах](./public-ip-addresses.md#allocation-method). Чтобы изменить частный IP-адрес, назначенный существующей виртуальной машине из динамического в статический, или для работы с общедоступными IP-адресами, см. раздел [Добавление, изменение или удаление IP-адресов](virtual-network-network-interface-addresses.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины

Указанные ниже действия можно выполнить с помощью локального компьютера или Azure Cloud Shell. Чтобы использовать локальный компьютер, на нем должна быть установлена служба [Azure PowerShell](/powershell/azure/install-az-ps?toc=%2fazure%2fvirtual-network%2ftoc.json). Чтобы использовать Azure Cloud Shell, выберите функцию **Попробовать** в правом верхнем углу любого окна со следующими командами. Через Cloud Shell будет выполнен вход в Azure.

1. При использовании Cloud Shell перейдите к шагу 2. Откройте сеанс командной строки и войдите в Azure с помощью команды `Connect-AzAccount`.
2. Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). В следующем примере создается группа ресурсов в регионе Azure "Восточная часть США".

   ```azurepowershell-interactive
   $RgName = "myResourceGroup"
   $Location = "eastus"
   New-AzResourceGroup -Name $RgName -Location $Location
   ```

3. Создайте конфигурацию подсети и виртуальную сеть с помощью команд [New-азвиртуалнетворксубнетконфиг](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) и [New-азвиртуалнетворк](/powershell/module/az.network/new-azvirtualnetwork) :

   ```azurepowershell-interactive
   # Create a subnet configuration
   $SubnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name MySubnet `
   -AddressPrefix 10.0.0.0/24

   # Create a virtual network
   $VNet = New-AzVirtualNetwork `
   -ResourceGroupName $RgName `
   -Location $Location `
   -Name MyVNet `
   -AddressPrefix 10.0.0.0/16 `
   -Subnet $subnetConfig

   # Get the subnet object for use in a later step.
   $Subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetConfig.Name -VirtualNetwork $VNet
   ```

4. Создайте сетевой интерфейс в виртуальной сети и назначьте ему частный IP-адрес из подсети к сетевому интерфейсу с помощью команд [New-азнетворкинтерфацеипконфиг](/powershell/module/Az.Network/New-AzNetworkInterfaceIpConfig) и [New-азнетворкинтерфаце](/powershell/module/az.network/new-aznetworkinterface) .

   ```azurepowershell-interactive
   $IpConfigName1 = "IPConfig-1"
   $IpConfig1     = New-AzNetworkInterfaceIpConfig `
     -Name $IpConfigName1 `
     -Subnet $Subnet `
     -PrivateIpAddress 10.0.0.4 `
     -Primary

   $NIC = New-AzNetworkInterface `
     -Name MyNIC `
     -ResourceGroupName $RgName `
     -Location $Location `
     -IpConfiguration $IpConfig1
   ```

5. Создайте конфигурацию виртуальной машины с помощью [New-азвмконфиг](/powershell/module/Az.Compute/New-AzVMConfig), а затем создайте виртуальную машину с помощью [New-AzVM](/powershell/module/az.Compute/New-azVM). При появлении запроса введите имя пользователя и пароль, которые будут использоваться в качестве учетных данных для входа для виртуальной машины.

   ```azurepowershell-interactive
   $VirtualMachine = New-AzVMConfig -VMName MyVM -VMSize "Standard_DS3"
   $VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName MyServerVM -ProvisionVMAgent -EnableAutoUpdate
   $VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
   $VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest
   New-AzVM -ResourceGroupName $RgName -Location $Location -VM $VirtualMachine -Verbose
   ```

> [!WARNING]
> Несмотря на то, что в операционную систему можно добавить параметры частного IP-адреса, мы рекомендуем не делать это до тех пор, пока не прочтите [Добавление частного IP-адреса в операционную систему](virtual-network-network-interface-addresses.md#private).
> 
> 
> <a name = "change-the-allocation-method-for-a-private-ip-address-assigned-to-a-network-interface"></a>
> 
> [!IMPORTANT]
> Чтобы получить доступ к виртуальной машине из Интернета, необходимо назначить виртуальной машине общедоступный IP-адрес. Можно также изменить назначение динамического частного IP-адреса на статическое назначение. Дополнительные сведения см. в разделе [Добавление или изменение IP-адресов](virtual-network-network-interface-addresses.md). Кроме того, рекомендуется ограничить сетевой трафик к виртуальной машине, связав сетевую группу безопасности с сетевым интерфейсом, подсетью, в которой был создан сетевой интерфейс, или и то, и другое. Дополнительные сведения см. в разделе [Управление группами безопасности сети](manage-network-security-group.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все содержащиеся в ней ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup):

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше о [частных IP-](./private-ip-addresses.md) адресах и назначении [статического частного IP-адреса](virtual-network-network-interface-addresses.md#add-ip-addresses) виртуальной машине Azure.
- Дополнительные сведения о создании виртуальных машин [Linux](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) и [Windows](../virtual-machines/windows/tutorial-manage-vm.md?toc=%2fazure%2fvirtual-network%2ftoc.json) .
