---
title: Включить файл
description: Включить файл
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 03/18/2020
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 807cbd283cf7971bf4256451028ffa16a0911266
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96027946"
---
## <a name="create-an-image-gallery"></a>Создание коллекции образов 

Коллекция образов является основным ресурсом, который позволяет обмен изображениями. Допустимыми знаками для имени коллекции являются прописные или строчные буквы, цифры и точки. Имя коллекции не должно содержать дефисы. Имена коллекций должны быть уникальным в пределах вашей подписки. 

Создайте коллекцию образов, используя команду [New-AzGallery](/powershell/module/az.compute/new-azgallery). В следующем примере показано, как создать коллекцию с именем *myGallery* в группе ресурсов *myGalleryRG*.

```azurepowershell-interactive
$resourceGroup = New-AzResourceGroup `
   -Name 'myGalleryRG' `
   -Location 'West Central US'  
$gallery = New-AzGallery `
   -GalleryName 'myGallery' `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -Description 'Shared Image Gallery for my organization'  
```
   

## <a name="share-the-gallery"></a>Предоставление общего доступа к коллекции

Мы рекомендуем предоставлять общий доступ на уровне коллекции образов. Используйте адрес электронной почты и командлет [Get-AzADUser](/powershell/module/az.resources/get-azaduser), чтобы получить идентификатор объекта для пользователя, а затем используйте командлет [New-AzRoleAssignment](/powershell/module/Az.Resources/New-AzRoleAssignment), чтобы предоставить этому пользователю доступ к коллекции. Замените пример адреса электронной почты alinne_montes@contoso.com в этом примере своим значением адреса.

```azurepowershell-interactive
# Get the object ID for the user
$user = Get-AzADUser -StartsWith alinne_montes@contoso.com
# Grant access to the user for our gallery
New-AzRoleAssignment `
   -ObjectId $user.Id `
   -RoleDefinitionName Reader `
   -ResourceName $gallery.Name `
   -ResourceType Microsoft.Compute/galleries `
   -ResourceGroupName $resourceGroup.ResourceGroupName

```