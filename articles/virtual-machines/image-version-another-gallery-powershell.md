---
title: Копирование изображения из другой коллекции с помощью PowerShell
description: Копирование изображения из другой коллекции с помощью Azure PowerShell.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: d9bbe40e35bdad6fac5c5ccb0b15b909e77b938c
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102564022"
---
# <a name="copy-an-image-from-another-gallery-using-powershell"></a>Копирование изображения из другой коллекции с помощью PowerShell

При наличии нескольких коллекций в Организации можно создавать образы из образов, хранящихся в других галереях. Например, у вас может быть коллекция разработки и тестирования для создания и тестирования новых образов. Когда они готовы к использованию в рабочей среде, их можно скопировать в рабочую коллекцию, используя этот пример. Вы также можете создать образ из изображения в другой коллекции, используя [Azure CLI](image-version-another-gallery-cli.md).


## <a name="before-you-begin"></a>Подготовка к работе

Для работы с этой статьей необходимо наличие существующей коллекции источников, определения образа и версии образа. Кроме того, у вас должна быть конечная коллекция. 

Исходная версия образа должна быть реплицирована в регион, где находится конечная коллекция. 

Мы создадим новое определение образа и версию образа в коллекции назначения.


При работе с этой статьей при необходимости замените имена ресурсов.


## <a name="get-the-source-image"></a>Получение исходного изображения 

Вам понадобятся сведения из определения исходного образа, чтобы можно было создать его копию в коллекции назначения.

Перечислите сведения о существующих галереях, определениях образов и версиях изображений с помощью командлета [Get-азресаурце](/powershell/module/az.resources/get-azresource) .

Результаты представлены в формате `gallery\image definition\image version` .

```azurepowershell-interactive
Get-AzResource `
   -ResourceType Microsoft.Compute/galleries/images/versions | `
   Format-Table -Property Name,ResourceGroupName
```

После получения всех необходимых сведений можно получить идентификатор исходной версии образа с помощью команды [Get-азгаллеримажеверсион](/powershell/module/az.compute/get-azgalleryimageversion). В этом примере мы получаем `1.0.0` версию образа `myImageDefinition` определения в `myGallery` коллекции исходных файлов в `myResourceGroup` группе ресурсов.

```azurepowershell-interactive
$sourceImgVer = Get-AzGalleryImageVersion `
   -GalleryImageDefinitionName myImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name 1.0.0
```


## <a name="create-the-image-definition"></a>Создание определения образа 

Необходимо создать новое определение образа, соответствующее определению образа источника. Вы можете просмотреть все сведения, необходимые для повторного создания определения образа с помощью команды [Get-азгаллеримажедефинитион](/powershell/module/az.compute/get-azgalleryimagedefinition).

```azurepowershell-interactive
Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition
```


Результат должен выглядеть следующим образом.

```output
{
  "description": null,
  "disallowed": null,
  "endOfLifeDate": null,
  "eula": null,
  "hyperVgeneration": "V1",
  "id": "/subscriptions/1111abcd-1a23-4b45-67g7-1234567de123/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition",
  "identifier": {
    "offer": "myOffer",
    "publisher": "myPublisher",
    "sku": "mySKU"
  },
  "location": "eastus",
  "name": "myImageDefinition",
  "osState": "Specialized",
  "osType": "Windows",
  "privacyStatementUri": null,
  "provisioningState": "Succeeded",
  "purchasePlan": null,
  "recommended": null,
  "releaseNoteUri": null,
  "resourceGroup": "myGalleryRG",
  "tags": null,
  "type": "Microsoft.Compute/galleries/images"
}
```

Создайте новое определение образа в коллекции назначения с помощью командлета [New-азгаллеримажедефинитион](/powershell/module/az.compute/new-azgalleryimageversion) и сведений из выходных данных выше.


В этом примере определение образа называется *мидестинатионимгдеф* в коллекции с именем *мидестинатионгаллери*.


```azurepowershell-interactive
$destinationImgDef  = New-AzGalleryImageDefinition `
   -GalleryName myDestinationGallery `
   -ResourceGroupName myDestinationRG `
   -Location WestUS `
   -Name 'myDestinationImgDef' `
   -OsState specialized `
   -OsType Windows `
   -HyperVGeneration v1 `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-the-image-version"></a>Создание версии образа

Создайте версию образа с помощью команды [New-азгаллеримажеверсион](/powershell/module/az.compute/new-azgalleryimageversion). Необходимо передать идентификатор исходного образа в `--managed-image` параметре для создания версии образа в коллекции назначения. 

Допустимыми знаками для имени версии образа являются цифры и точки. Числа должны быть в диапазоне 32-битного целого числа. Формат: *основной номер версии*.*дополнительный номер версии*.*исправление*.

В этом примере конечная коллекция называется *мидестинатионгаллери* в группе ресурсов *Мидестинатионрг* в регионе *Западная часть США* . Версия нашего образа — *1.0.0* , и мы создадим 1 реплику в *юго-центральном регионе США* и 2 реплики в регионе " *Западная часть США* ". 


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West US';ReplicaCount=2}
$targetRegions = @($region1,$region2)

$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $destinationImgDef.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName myDestinationGallery `
   -ResourceGroupName myDestinationRG `
   -Location WestUS `
   -TargetRegion $targetRegions  `
   -Source $sourceImgVer.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-01' `
   -asJob 
```

Репликация образа во все целевые регионы может занять некоторое время, поэтому мы создали задание, чтобы отслеживать ход выполнения. Чтобы просмотреть ход выполнения задания, введите `$job.State`.

```azurepowershell-interactive
$job.State
```

> [!NOTE]
> Прежде чем использовать тот же управляемый образ для создания другой версии образа, необходимо дождаться завершения сборки и репликации версии образа.
>
> Вы также можете сохранить образ в хранилище класса Premium, добавив `-StorageAccountType Premium_LRS`, или [хранилище, избыточное между зонами](../storage/common/storage-redundancy.md), добавив `-StorageAccountType Standard_ZRS` при создании версии образа.
>


## <a name="next-steps"></a>Дальнейшие действия

Создайте виртуальную машину на основе [обобщенной](vm-generalized-image-version-powershell.md) или [специализированной](vm-specialized-image-version-powershell.md) версии образа.

С помощью [построителя образов Azure (Предварительная версия)](./image-builder-overview.md) можно автоматизировать создание версий изображений, а также использовать его для обновления и [создания новой версии образа из существующей версии образа](./linux/image-builder-gallery-update-image-version.md). 

Сведения о том, как предоставить сведения о плане покупки, см. в разделе [предоставление сведений о плане покупки Azure Marketplace при создании образов](marketplace-images.md).