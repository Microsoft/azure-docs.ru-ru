---
title: Клонирование управляемого образа в коллекцию общих образов
description: Узнайте, как использовать Azure PowerShell для клонирования управляемого образа в версию образа в общей коллекции образов.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: cec4f258cfaa0584c24f2cfc92ec1a536f6277cc
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102556986"
---
# <a name="clone-a-managed-image-to-a-shared-image-gallery-image"></a>Клонирование управляемого образа в общий образ коллекции образов

Если у вас есть управляемый образ, который вы хотите клонировать и переместить в коллекцию общих образов, можно создать образ из коллекции образов напрямую из управляемого образа. После тестирования нового образа можно удалить исходный управляемый образ. Можно также выполнить миграцию из управляемого образа в общую коллекцию образов с помощью [Azure CLI](image-version-managed-image-cli.md).

Изображения в коллекции образов имеют два компонента, которые будут созданы в этом примере:
- **Определение образа** содержит сведения о образе и требования для его использования. Сюда входит, является ли образ Windows или Linux, специализированным или обобщенным, заметками о выпуске, а также минимальным и максимальным требованиями к памяти. Это определение типа образа. 
- **Версия образа** — это то, что используется для создания виртуальной машины при использовании общей коллекции образов. В зависимости от требований для вашей среды, у вас может быть несколько версий образа. При создании виртуальной машины используется версия образа для создания новых дисков для виртуальной машины. Версии образов можно использовать несколько раз.


## <a name="before-you-begin"></a>Подготовка к работе

Для работы с этой статьей необходим существующий управляемый образ. Если управляемый образ содержит диск данных, размер диска данных не может превышать 1 ТБ.

При работе со статьей по мере необходимости заменяйте имена групп ресурсов и виртуальных машин.

## <a name="get-the-gallery"></a>Получение коллекции

Вы можете вывести список всех коллекций и определений изображений по имени. Результаты представлены в формате `gallery\image definition\image version` .

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Compute/galleries | Format-Table
```

Найдя нужную галерею, создайте переменную для последующего использования. Этот пример получает коллекцию с именем *мигаллери* в группе ресурсов *myResourceGroup* .

```azurepowershell-interactive
$gallery = Get-AzGallery `
   -Name myGallery `
   -ResourceGroupName myResourceGroup
```


## <a name="create-an-image-definition"></a>Создание определения образа 

Образы можно объединять в логические группы с помощью определений образов. Они используются для управления сведениями об образе. В имени определения образа можно использовать прописные и строчные буквы, цифры, точки и дефисы. 

При создании определения образа убедитесь, что содержит все правильные сведения. Так как управляемые образы всегда обобщены, следует задать `-OsState generalized` . 

Дополнительные сведения о значениях, которые можно указать для определения образа, см. в разделе [Определения образов](./shared-image-galleries.md#image-definitions).

Создайте определение образа с помощью командлета [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). В этом примере определение образа называется *мимажедефинитион* и предназначено для универсальной ОС Windows. Чтобы создать определение для образов с помощью ОС Linux, используйте `-OsType Linux` . 

```azurepowershell-interactive
$imageDefinition = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState generalized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```

## <a name="get-the-managed-image"></a>Получение управляемого образа

Список образов, доступных в группе ресурсов, можно просмотреть с помощью команды [Get-AzImage](/powershell/module/az.compute/get-azimage). Зная имя образа и группу ресурсов, в которой он находится, можно снова выполнить команду `Get-AzImage`, чтобы получить объект образа и сохранить его в переменной для последующего использования. Этот пример получает образ с именем *myImage* из группы ресурсов myResourceGroup и присваивает его переменной *$managedImage*. 

```azurepowershell-interactive
$managedImage = Get-AzImage `
   -ImageName myImage `
   -ResourceGroupName myResourceGroup
```


## <a name="create-an-image-version"></a>Создание версии образа

Создайте версию образа из управляемого образа с помощью команды [New-азгаллеримажеверсион](/powershell/module/az.compute/new-azgalleryimageversion). 

Допустимыми знаками для имени версии образа являются цифры и точки. Числа должны быть в диапазоне 32-битного целого числа. Формат: *основной номер версии*.*дополнительный номер версии*.*исправление*.

В этом примере используется версия образа *1.0.0*, которая реплицируется в центры обработки данных в регионах *центрально-западная часть США* и *центрально-южная часть США*. При выборе целевых регионов для репликации Помните, что также необходимо включить *Исходный* регион в качестве цели для репликации. 


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West Central US';ReplicaCount=2}
$targetRegions = @($region1,$region2)
$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name `
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $imageDefinition.ResourceGroupName `
   -Location $imageDefinition.Location `
   -TargetRegion $targetRegions  `
   -SourceImageId $managedImage.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-31' `
   -asJob 
```

Репликация образа во все целевые регионы может занять некоторое время, поэтому мы создали задание, чтобы отслеживать ход выполнения. Чтобы просмотреть ход выполнения, введите `$job.State` .

```azurepowershell-interactive
$job.State
```


> [!NOTE]
> Прежде чем использовать тот же управляемый образ для создания другой версии образа, необходимо дождаться завершения сборки и репликации версии образа. 
>
> Вы также можете сохранить образ в хранилище класса Premium, добавив `-StorageAccountType Premium_LRS` [хранилище или избыточное в зону](../storage/common/storage-redundancy.md) , добавив `-StorageAccountType Standard_ZRS` его при создании версии образа.
>

## <a name="delete-the-managed-image"></a>Удаление управляемого образа

Убедившись, что новая версия образа работает правильно, можно удалить управляемый образ.

```azurepowershell-interactive
Remove-AzImage `
   -ImageName $managedImage.Name `
   -ResourceGroupName $managedImage.ResourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

Убедившись, что репликация завершена, можно создать виртуальную машину из [обобщенного образа](vm-generalized-image-version-powershell.md).

Сведения о том, как предоставить сведения о плане покупки, см. в разделе [предоставление сведений о плане покупки Azure Marketplace при создании образов](marketplace-images.md).