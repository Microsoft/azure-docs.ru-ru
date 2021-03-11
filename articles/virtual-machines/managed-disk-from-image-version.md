---
title: Создание управляемого диска из версии образа
description: Создание управляемого диска из версии образа в общей коллекции образов.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.topic: how-to
ms.workload: infrastructure
ms.date: 10/27/2020
ms.author: cynthn
ms.reviewer: olayemio
ms.openlocfilehash: 359816bbf3fb93a4ec2bfa3092adbb2236ae2f07
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102562560"
---
# <a name="create-a-managed-disk-from-an-image-version"></a>Создание управляемого диска из версии образа

При необходимости можно экспортировать операционную систему или отдельный диск данных из версии образа в качестве управляемого диска из версии образа, хранящейся в коллекции общих образов.


## <a name="cli"></a>CLI

Выведите список версий образа в галерее, выполнив команду [AZ SIG Image-Version List](/cli/azure/sig/image-version#az_sig_image_version_list). В этом примере мы ищем все версии изображений, которые являются частью определения образа *мимажедефинитион* в коллекции образов *мигаллери* .

```azurecli-interactive
az sig image-version list \
   --resource-group myResourceGroup\
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   -o table
```

Задайте `source` для переменной идентификатор версии образа, а затем выполните команду [AZ Disk Create](/cli/azure/disk#az_disk_create) , чтобы создать управляемый диск. 

В этом примере мы экспортируем диск операционной системы версии образа, чтобы создать управляемый диск с именем *миманажедосдиск* в регионе *EastUS* в группе ресурсов с именем *myResourceGroup*. 

```azurecli-interactive
source="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<galleryImageDefinition>/versions/<imageVersion>"

az disk create --resource-group myResourceGroup --location EastUS --name myManagedOSDisk --gallery-image-reference $source 
```



Если вы хотите экспортировать диск данных из версии образа, добавьте, `--gallery-image-reference-lun` чтобы указать расположение LUN для экспортируемого диска данных. 

В этом примере мы экспортируем диск данных, расположенный в LUN 0 версии образа, чтобы создать управляемый диск с именем *миманажеддатадиск* в регионе *EastUS* в группе ресурсов с именем *myResourceGroup*. 

```azurecli-interactive
source="/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/galleries/<galleryName>/images/<galleryImageDefinition>/versions/<imageVersion>"

az disk create --resource-group myResourceGroup --location EastUS --name myManagedDataDisk --gallery-image-reference $source --gallery-image-reference-lun 0
``` 

## <a name="powershell"></a>PowerShell

Выведите список версий образа в галерее с помощью командлета [Get-азресаурце](/powershell/module/az.resources/get-azresource). 

```azurepowershell-interactive
Get-AzResource `
   -ResourceType Microsoft.Compute/galleries/images/versions | `
   Format-Table -Property Name,ResourceId,ResourceGroupName
```

После получения всех необходимых сведений можно использовать [Get-азгаллеримажеверсион](/powershell/module/az.compute/get-azgalleryimageversion) , чтобы получить версию исходного образа, которую вы хотите использовать, и назначить ее переменной. В этом примере мы получаем `1.0.0` версию образа `myImageDefinition` определения в `myGallery` коллекции исходных файлов в `myResourceGroup` группе ресурсов.

```azurepowershell-interactive
$sourceImgVer = Get-AzGalleryImageVersion `
   -GalleryImageDefinitionName myImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name 1.0.0
```

После присвоения `source` переменной идентификатора версии образа используйте [New-аздискконфиг](/powershell/module/az.compute/new-azdiskconfig) для создания конфигурации диска и [New-аздиск](/powershell/module/az.compute/new-azdisk) , чтобы создать диск. 

В этом примере мы экспортируем диск операционной системы версии образа, чтобы создать управляемый диск с именем *миманажедосдиск* в регионе *EastUS* в группе ресурсов с именем *myResourceGroup*. 

Создайте конфигурацию диска.
```azurepowershell-interactive
$diskConfig = New-AzDiskConfig `
   -Location EastUS `
   -CreateOption FromImage `
   -GalleryImageReference @{Id = $sourceImgVer.Id}
```

Создайте диск.

```azurepowershell-interactive
New-AzDisk -Disk $diskConfig `
   -ResourceGroupName myResourceGroup `
   -DiskName myManagedOSDisk
```

Если вы хотите экспортировать диск данных в версию образа, добавьте идентификатор LUN в конфигурацию диска, чтобы указать расположение LUN для экспортируемого диска данных. 

В этом примере мы экспортируем диск данных, расположенный в LUN 0 версии образа, чтобы создать управляемый диск с именем *миманажеддатадиск* в регионе *EastUS* в группе ресурсов с именем *myResourceGroup*. 

Создайте конфигурацию диска.
```azurepowershell-interactive
$diskConfig = New-AzDiskConfig `
   -Location EastUS `
   -CreateOption FromImage `
   -GalleryImageReference @{Id = $sourceImgVer.Id; Lun=0}
```

Создайте диск.

```azurepowershell-interactive
New-AzDisk -Disk $diskConfig `
   -ResourceGroupName myResourceGroup `
   -DiskName myManagedDataDisk
```

## <a name="next-steps"></a>Дальнейшие действия

Вы также можете создать версию образа на основе управляемого диска с помощью [Azure CLI](image-version-managed-image-cli.md) или [PowerShell](image-version-managed-image-powershell.md).


