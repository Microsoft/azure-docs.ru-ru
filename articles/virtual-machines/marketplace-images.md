---
title: Указание сведений о плане покупки Marketplace с помощью Azure PowerShell
description: Узнайте, как указать сведения о плане покупки Azure Marketplace при создании образов в общей коллекции образов.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.topic: how-to
ms.workload: infrastructure
ms.date: 07/07/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 9df1f6b02e6572c8f2153016c0142912e24fcfe8
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102562543"
---
# <a name="supply-azure-marketplace-purchase-plan-information-when-creating-images"></a>Предоставление сведений о плане покупки Azure Marketplace при создании образов

Если вы создаете образ в общей коллекции, используя источник, который изначально был создан из образа Azure Marketplace, может потребоваться отслеживание сведений о плане покупки. В этой статье показано, как найти сведения о плане покупки для виртуальной машины, а затем использовать эти сведения при создании определения образа. Мы также расскажем об использовании информации из определения образа, чтобы упростить предоставление сведений о плане покупки при создании виртуальной машины для образа.

Дополнительные сведения о поиске и использовании образов Marketplace см. в статье [Поиск и использование образов Azure Marketplace](./windows/cli-ps-findimage.md).


## <a name="get-the-source-vm-information"></a>Получение сведений об исходной виртуальной машине
Если у вас по-прежнему есть исходная виртуальная машина, можно получить имя плана, издателя и сведения о продукте с помощью команды Get-AzVM. Этот пример получает виртуальную машину с именем *myVM* в группе ресурсов *myResourceGroup* , а затем отображает сведения о плане покупки для виртуальной машины.

```azurepowershell-interactive
$vm = Get-azvm `
   -ResourceGroupName myResourceGroup `
   -Name myVM
$vm.Plan
```

## <a name="create-the-image-definition"></a>Создание определения образа

Получите коллекцию образов, которая будет использоваться для хранения образа. Сначала можно вывести список всех галерей.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Compute/galleries | Format-Table
```

Затем создайте переменные для коллекции, которую вы хотите использовать. В этом примере мы создаем переменную с именем `$gallery` для *мигаллери* в группе ресурсов *мигаллерирг* .

```azurepowershell-interactive
$gallery = Get-AzGallery `
   -Name myGallery `
   -ResourceGroupName myGalleryRG
```

Создайте определение образа с помощью  `-PurchasePlanPublisher` `-PurchasePlanProduct` параметров, и `-PurchasePlanName` .

```azurepowershell-interactive
 $imageDefinition = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Linux `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU' `
   -PurchasePlanPublisher $vm.Plan.Publisher `
   -PurchasePlanProduct $vm.Plan.Product `
   -PurchasePlanName  $vm.Plan.Name
```

Затем создайте версию образа с помощью команды [New-азгаллеримажеверсион](/powershell/module/az.compute/new-azgalleryimageversion). Вы можете создать версию образа из [виртуальной машины](image-version-vm-powershell.md#create-an-image-version), [управляемого образа](image-version-managed-image-powershell.md#create-an-image-version), [вхд\снапшот](image-version-snapshot-powershell.md#create-an-image-version)или [другой версии образа](image-version-another-gallery-powershell.md#create-the-image-version). 


## <a name="create-the-vm"></a>Создание виртуальной машины

При переходе к созданию виртуальной машины из образа можно использовать сведения из определения образа для передачи сведений об издателе с помощью [Set-азвмплан](/powershell/module/az.compute/set-azvmplan).


```azurepowershell-interactive
# Create some variables for the new VM.
$resourceGroup = "mySIGPubVM"
$location = "West Central US"
$vmName = "mySIGPubVM"

# Create a resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create the network resources.
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name mySubnet `
   -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -Name MYvNET `
   -AddressPrefix 192.168.0.0/16 `
   -Subnet $subnetConfig
$pip = New-AzPublicIpAddress `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -Name "mypublicdns$(Get-Random)" `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig `
   -Name myNetworkSecurityGroupRuleRDP  `
   -Protocol Tcp `
   -Direction Inbound `
   -Priority 1000 `
   -SourceAddressPrefix * `
   -SourcePortRange * `
   -DestinationAddressPrefix * `
   -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -Name myNetworkSecurityGroup `
   -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface `
   -Name $vmName `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using Set-AzVMSourceImage -Id $imageDefinition.Id to use the latest available image version. Set-AZVMPlan is used to pass the plan information in for the VM.

$vmConfig = New-AzVMConfig `
   -VMName $vmName `
   -VMSize Standard_D1_v2   | `
   Set-AzVMSourceImage -Id $imageDefinition.Id | `
   Set-AzVMPlan `
     -Publisher $imageDefinition.PurchasePlan.Publisher `
     -Product $imageDefinition.PurchasePlan.Product `
     -Name $imageDefinition.PurchasePlan.Name | `
   Add-AzVMNetworkInterface -Id $nic.Id

# Create the virtual machine
New-AzVM `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -VM $vmConfig
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о поиске и использовании образов Marketplace см. в статье [Поиск и использование образов Azure Marketplace](./windows/cli-ps-findimage.md).
