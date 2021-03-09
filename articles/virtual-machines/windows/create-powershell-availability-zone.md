---
title: Создание виртуальной машины с зоной с помощью Azure PowerShell
description: Создание виртуальной машины в зоне доступности с Azure PowerShell
author: cynthn
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: ''
ms.openlocfilehash: 02897add4f3ca51fa7e172bc200af2c7b9065ed4
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102502643"
---
# <a name="create-a-virtual-machine-in-an-availability-zone-using-azure-powershell"></a>Создание виртуальной машины в зоне доступности с помощью Azure PowerShell

В этой статье описывается использование Azure PowerShell для создания виртуальной машины Azure под управлением Windows Server 2016 в зоне доступности Azure. [Зона доступности](../../availability-zones/az-overview.md) — это физически отдельная зона в регионе Azure. Зоны доступности позволяют защитить приложения и данные от маловероятных сбоев и потери всего центра обработки данных.

Чтобы использовать зону доступности, создайте виртуальную машину в [поддерживаемом регионе Azure](../../availability-zones/az-region.md).

 

## <a name="sign-in-to-azure"></a>Вход в Azure

Войдите в подписку Azure с помощью команды `Connect-AzAccount` и следуйте инструкциям на экране.

```powershell
Connect-AzAccount
```

## <a name="check-vm-sku-availability"></a>Проверка доступности SKU виртуальной машины
Доступность размеров или номеров SKU виртуальных машин зависит от региона и зоны. Чтобы спланировать использование с учетом зон доступности, можно составить список доступных SKU виртуальных машин по регионам и зонам Azure. Так вы сможете выбрать нужный размер виртуальной машины и получить требуемую степень устойчивости в разных зонах. См. дополнительные сведения о разных типах и [размерах виртуальных машин](../sizes.md).

Доступные номера SKU виртуальных машин можно просмотреть с помощью командлета [Get-AzComputeResourceSku](/powershell/module/az.compute/get-azcomputeresourcesku). В следующем примере перечислены доступные номера SKU виртуальных машин в регионе *eastus2*:

```powershell
Get-AzComputeResourceSku | where {$_.Locations.Contains("eastus2")};
```

Вывод команды будет примерно таким, как в следующем сокращенном примере с отображением размеров виртуальных машин, используемых в пределах разных зон доступности:

```powershell
ResourceType                Name  Location      Zones   [...]
------------                ----  --------      -----
virtualMachines  Standard_DS1_v2   eastus2  {1, 2, 3}
virtualMachines  Standard_DS2_v2   eastus2  {1, 2, 3}
[...]
virtualMachines     Standard_F1s   eastus2  {1, 2, 3}
virtualMachines     Standard_F2s   eastus2  {1, 2, 3}
[...]
virtualMachines  Standard_D2s_v3   eastus2  {1, 2, 3}
virtualMachines  Standard_D4s_v3   eastus2  {1, 2, 3}
[...]
virtualMachines   Standard_E2_v3   eastus2  {1, 2, 3}
virtualMachines   Standard_E4_v3   eastus2  {1, 2, 3}
```


## <a name="create-resource-group"></a>Создать группу ресурсов

Создайте группу ресурсов Azure с помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. В этом примере создается группа ресурсов с именем *myResourceGroup* в регионе *eastus2*. 

```powershell
New-AzResourceGroup -Name myResourceGroup -Location EastUS2
```

## <a name="create-networking-resources"></a>Создание сетевых ресурсов

### <a name="create-a-virtual-network-subnet-and-a-public-ip-address"></a>Создайте виртуальную сеть, подсеть и общедоступный IP-адрес. 
Эти ресурсы используются для того, чтобы установить сетевое подключение к виртуальной машине и подключить ее к Интернету. Создайте IP-адрес в зоне доступности (*2* в этом примере). Позднее вы создадите виртуальную машину в той же зоне доступности, которая использовалась для создания IP-адреса.

```powershell
# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzVirtualNetwork -ResourceGroupName myResourceGroup -Location eastus2 `
    -Name myVNet -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig

# Create a public IP address in an availability zone and specify a DNS name
$pip = New-AzPublicIpAddress -ResourceGroupName myResourceGroup -Location eastus2 -Zone 2 `
    -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)" -Sku Standard
```

### <a name="create-a-network-security-group-and-a-network-security-group-rule"></a>Создайте группу безопасности сети и правило группы безопасности сети. 
Группа безопасности сети обеспечивает защиту виртуальной машины с помощью правил для входящего и исходящего трафика. В нашем случае создается правило для входящего трафика порта 3389, которое разрешает входящие подключения к удаленному рабочему столу. Мы также создадим правило входящего трафика для порта 80, разрешающее входящий веб-трафик.

```powershell
# Create an inbound network security group rule for port 3389
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleWWW  -Protocol Tcp `
    -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location eastus2 `
    -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP,$nsgRuleWeb
```

### <a name="create-a-network-card-for-the-virtual-machine"></a>Создание сетевой карты для виртуальной машины 
Создайте сетевую карту для виртуальной машины с помощью командлета [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). Сетевая карта подключает виртуальную машину к подсети, группе безопасности сети и общедоступному IP-адресу.

```powershell
# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzNetworkInterface -Name myNic -ResourceGroupName myResourceGroup -Location eastus2 `
    -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-virtual-machine"></a>Создание виртуальной машины

Создайте конфигурацию виртуальной машины. Эта конфигурация содержит параметры, которые используются при развертывании виртуальной машины, в том числе образ виртуальной машины, ее размер и настройки аутентификации. В этом примере в зонах доступности поддерживается размер *Standard_DS1_v2*. Эта конфигурация также указывает зону доступности, заданную при создании IP-адреса. При выполнении этого шага будут запрошены учетные данные. В качестве вводимых значений указываются имя пользователя и пароль для виртуальной машины.

```powershell
# Define a credential object
$cred = Get-Credential

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName myVM -VMSize Standard_DS1_v2 -Zone 2 | `
    Set-AzVMOperatingSystem -Windows -ComputerName myVM -Credential $cred | `
    Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer `
    -Skus 2016-Datacenter -Version latest | Add-AzVMNetworkInterface -Id $nic.Id
```

Создайте виртуальную машину с помощью команды [New-AzVM](/powershell/module/az.compute/new-azvm).

```powershell
New-AzVM -ResourceGroupName myResourceGroup -Location eastus2 -VM $vmConfig
```

## <a name="confirm-zone-for-managed-disk"></a>Подтверждение зоны для управляемого диска

Вы создали ресурс IP-адреса виртуальной машины в той же зоне доступности, где находится виртуальная машина. В той же зоне доступности создается управляемый диск для виртуальной машины. Это можно проверить с помощью командлета [Get-AzDisk](/powershell/module/az.compute/get-azdisk):

```powershell
Get-AzDisk -ResourceGroupName myResourceGroup
```

В выходных данных видно, что управляемый диск находится в той же зоне доступности, что и виртуальная машина:

```powershell
ResourceGroupName  : myResourceGroup
AccountType        : PremiumLRS
OwnerId            : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/virtualMachines/myVM
ManagedBy          : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx//resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/virtualMachines/myVM
Sku                : Microsoft.Azure.Management.Compute.Models.DiskSku
Zones              : {2}
TimeCreated        : 9/7/2017 6:57:26 PM
OsType             : Windows
CreationData       : Microsoft.Azure.Management.Compute.Models.CreationData
DiskSizeGB         : 127
EncryptionSettings :
ProvisioningState  : Succeeded
Id                 : /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.
                     Compute/disks/myVM_OsDisk_1_bd921920bb0a4650becfc2d830000000
Name               : myVM_OsDisk_1_bd921920bb0a4650becfc2d830000000
Type               : Microsoft.Compute/disks
Location           : eastus2
Tags               : {}
```


## <a name="next-steps"></a>Дальнейшие действия

Из этой статье вы узнали, как создать виртуальную машину в зоне доступности. Дополнительные сведения о [доступности](../availability.md) для виртуальных машин Azure.