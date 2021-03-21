---
title: Создание виртуальной машины на основе обобщенного образа
description: Создание виртуальной машины с помощью обобщенного образа в коллекции общих образов.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: dbe1e31a9b56f2f60edbebd8286bf0bfdeaa327e
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102560976"
---
# <a name="create-a-vm-using-a-generalized-image"></a>Создание виртуальной машины с помощью обобщенного образа 

Создайте виртуальную машину из обобщенного образа, хранящегося в коллекции общих образов. Если вы хотите создать виртуальную машину с помощью специализированного образа, см. раздел [Создание виртуальной машины на основе специализированного образа](vm-specialized-image-version-powershell.md).

После создания обобщенной версии образа можно создать одну или несколько новых виртуальных машин. Используйте командлет [New-AzVM](/powershell/module/az.compute/new-azvm). 

В этом примере мы используем идентификатор определения образа, чтобы убедиться, что новая виртуальная машина будет использовать самую последнюю версию образа. Вы также можете использовать определенную версию с помощью идентификатора версии образа для `Set-AzVMSourceImage -Id` . Например, чтобы использовать тип Image версии *1.0.0* : `Set-AzVMSourceImage -Id "/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"` . 

Имейте в виду, что использование конкретной версии образа означает, что автоматизация может завершиться ошибкой, если эта версия изображения недоступна, так как она была удалена или удалена из региона. Рекомендуется использовать идентификатор определения образа для создания новой виртуальной машины, если только не требуется определенная версия образа.

При необходимости замените имена ресурсов в этих примерах. 

## <a name="simplified-parameter-set"></a>Упрощенный набор параметров

С помощью упрощенного набора параметров можно быстро создать виртуальную машину из образа. Упрощенный набор параметров использует имя виртуальной машины для автоматического создания некоторых необходимых ресурсов, таких как виртуальная сеть и общедоступный IP-адрес. 

```azurepowershell-interactive
# Create some variables for the new VM 
$resourceGroup = "myResourceGroup"
$location = "South Central US"
$vmName = "myVMfromImage"

# Get the image. Replace the name of your resource group, gallery, and image definition. This will create the VM from the latest image version available.

$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition

# Create user object
$cred = Get-Credential `
   -Message "Enter a username and password for the virtual machine."

# Create a resource group
New-AzResourceGroup `
   -Name $resourceGroup `
   -Location $location

New-AzVM `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -Name $vmName `
   -Image $imageDefinition.Id
   -Credential $cred
```



## <a name="full-parameter-set"></a>Полный набор параметров

Вы можете создать виртуальную машину с использованием конкретных ресурсов с помощью полного набора параметров.

```azurepowershell-interactive
# Create some variables for the new VM 
$resourceGroup = "myResourceGroup"
$location = "South Central US"
$vmName = "myVMfromImage"

# Get the image. Replace the name of your resource group, gallery, and image definition. This will create the VM from the latest image version available.

$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition

# Create user object
$cred = Get-Credential `
   -Message "Enter a username and password for the virtual machine."

# Create a resource group
New-AzResourceGroup `
   -Name $resourceGroup `
   -Location $location

# Network pieces
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
   -DestinationPortRange 3389 `
   -Access Allow
$nsg = New-AzNetworkSecurityGroup `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -Name myNetworkSecurityGroup `
  -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface `
   -Name myNic `
   -ResourceGroupName $resourceGroup `
   -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using $imageDefinition.Id to use the latest image version.
$vmConfig = New-AzVMConfig `
   -VMName $vmName `
   -VMSize Standard_D1_v2 | `
   Set-AzVMOperatingSystem -Windows -ComputerName $vmName -Credential $cred | `
   Set-AzVMSourceImage -Id $imageDefinition.Id | `
   Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -VM $vmConfig
```


## <a name="next-steps"></a>Дальнейшие действия
С помощью [построителя образов Azure (Предварительная версия)](./image-builder-overview.md) можно автоматизировать создание версий изображений, а также использовать его для обновления и [создания новой версии образа из существующей версии образа](./windows/image-builder-gallery-update-image-version.md). 

Вы также можете создать ресурс коллекции общих образов с помощью шаблонов. Существует несколько шаблонов быстрого запуска Azure: 

- [Создание коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Создание определения образа в коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Создание версии образа в коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [Создание виртуальной машины из версии образа](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

Дополнительные сведения о коллекциях общих образов см. в [обзорной статье](./shared-image-galleries.md). Если вы столкнетесь с проблемами, обратитесь к статье об [устранении неполадок c коллекциями общих образов](troubleshooting-shared-images.md).