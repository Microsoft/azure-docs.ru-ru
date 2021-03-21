---
title: Создание виртуальной машины из специализированного образа
description: Создание виртуальной машины с помощью специализированного образа в коллекции общих образов.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 6b728591aeb330c7b2af8ca5dc4770147761b5b0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102552292"
---
# <a name="create-a-vm-using-a-specialized-image"></a>Создание виртуальной машины с помощью специализированного образа 

Создайте виртуальную машину из специализированной версии образа, хранящейся в коллекции общих образов. Если вы хотите создать виртуальную машину с помощью обобщенной версии образа, см. статью [Создание виртуальной машины с помощью обобщенного образа](vm-generalized-image-version-powershell.md).

После создания специализированной версии образа можно создать одну или несколько новых виртуальных машин с помощью командлета [New-AzVM](/powershell/module/az.compute/new-azvm) . 

В этом примере мы используем идентификатор определения образа, чтобы убедиться, что новая виртуальная машина будет использовать самую последнюю версию образа. Вы также можете использовать определенную версию с помощью идентификатора версии образа для `Set-AzVMSourceImage -Id` . Например, чтобы использовать тип Image версии *1.0.0* : `Set-AzVMSourceImage -Id "/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"` . 

Имейте в виду, что использование конкретной версии образа означает, что автоматизация может завершиться ошибкой, если эта версия изображения недоступна, так как она была удалена или удалена из региона. Рекомендуется использовать идентификатор определения образа для создания новой виртуальной машины, если только не требуется определенная версия образа.

При необходимости замените имена ресурсов в этом примере. 


```azurepowershell-interactive

# Create some variables for the new VM.

$resourceGroup = "mySIGSpecializedRG"
$location = "South Central US"
$vmName = "mySpecializedVM"

# Get the image. Replace the name of your resource group, gallery, and image definition. This will create the VM from the latest image version available.

$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition


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

# Create a virtual machine configuration using Set-AzVMSourceImage -Id $imageDefinition.Id to use the latest available image version.

$vmConfig = New-AzVMConfig `
   -VMName $vmName `
   -VMSize Standard_D1_v2 | `
   Set-AzVMSourceImage -Id $imageDefinition.Id | `
   Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM `
   -ResourceGroupName $resourceGroup `
   -Location $location `
   -VM $vmConfig

```

## <a name="attach-the-data-disk"></a>Подключение диска данных
Если образ содержал диск данных, необходимо подключить диск данных к виртуальной машине.

```azurepowershell-interactive

$vm = Get-AzVM -Name $vmName -ResourceGroupName $resourceGroup 

$lun = $imageVersion.StorageProfile.DataDiskImages.Lun

Add-AzVMDataDisk `
   -CreateOption FromImage `
   -SourceImageUri $imageversion.Id `
   -Lun $lun `
   -Caching $imageVersion.StorageProfile.DataDiskImages.HostCaching `
   -DiskSizeInGB $imageVersion.StorageProfile.DataDiskImages.SizeInGB `
   -VM $vm

```


## <a name="next-steps"></a>Дальнейшие действия
С помощью [построителя образов Azure (Предварительная версия)](./image-builder-overview.md) можно автоматизировать создание версий изображений, а также использовать его для обновления и [создания новой версии образа из существующей версии образа](./linux/image-builder-gallery-update-image-version.md). 

Вы также можете создать ресурс коллекции общих образов с помощью шаблонов. Существует несколько шаблонов быстрого запуска Azure: 

- [Создание коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Создание определения образа в коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Создание версии образа в коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [Создание виртуальной машины из версии образа](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

Дополнительные сведения о коллекциях общих образов см. в [обзорной статье](./shared-image-galleries.md). Если вы столкнетесь с проблемами, обратитесь к статье об [устранении неполадок c коллекциями общих образов](troubleshooting-shared-images.md).