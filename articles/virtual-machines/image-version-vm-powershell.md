---
title: Создание образа из виртуальной машины
description: Узнайте, как использовать Azure PowerShell для создания образа в общей коллекции образов на основе существующей виртуальной машины в Azure.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: f7afc671dbb3d4ef8aa30222c4f235c9f7869d02
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102556848"
---
# <a name="create-an-image-from-a-vm"></a>Создание образа из виртуальной машины

Если у вас уже есть виртуальная машина, которую вы хотите использовать для создания нескольких идентичных виртуальных машин, вы можете использовать эту виртуальную машину, чтобы создать образ в общей коллекции образов с помощью Azure PowerShell. Вы также можете создать образ из виртуальной машины с помощью [Azure CLI](image-version-vm-cli.md).

Вы можете записать образ из [специализированных и обобщенных](./shared-image-galleries.md#generalized-and-specialized-images) виртуальных машин с помощью Azure PowerShell. 

Изображения в коллекции образов имеют два компонента, которые будут созданы в этом примере:
- **Определение образа** содержит сведения о образе и требования для его использования. Сюда входит, является ли образ Windows или Linux, специализированным или обобщенным, заметками о выпуске, а также минимальным и максимальным требованиями к памяти. Это определение типа образа. 
- **Версия образа** — это то, что используется для создания виртуальной машины при использовании общей коллекции образов. В зависимости от требований для вашей среды, у вас может быть несколько версий образа. При создании виртуальной машины используется версия образа для создания новых дисков для виртуальной машины. Версии образов можно использовать несколько раз.


## <a name="before-you-begin"></a>Подготовка к работе

Для работы с этой статьей необходимо иметь имеющуюся коллекцию общих образов и существующую виртуальную машину в Azure для использования в качестве источника. 

Если к виртуальной машине подключен диск данных, размер диска данных не может превышать 1 ТБ.

При работе с этой статьей при необходимости замените имена ресурсов.


## <a name="get-the-gallery"></a>Получение коллекции

Вы можете вывести список всех коллекций и определений изображений по имени. Результаты представлены в формате `gallery\image definition\image version` .

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Compute/galleries | Format-Table
```

После нахождения нужной коллекции и определений изображений создайте переменные для них, которые будут использоваться позже. Этот пример получает коллекцию с именем *мигаллери* в группе ресурсов *myResourceGroup* .

```azurepowershell-interactive
$gallery = Get-AzGallery `
   -Name myGallery `
   -ResourceGroupName myResourceGroup
```

## <a name="get-the-vm"></a>Получение виртуальной машины

Список виртуальных машин, доступных в группе ресурсов, можно просмотреть с помощью командлета [Get-AzVM](/powershell/module/az.compute/get-azvm). Зная имя виртуальной машины и группу ресурсов, в которой она находится, можно `Get-AzVM` снова использовать для получения объекта виртуальной машины и сохранить его в переменной для последующего использования. Этот пример получает виртуальную машину с именем *саурцевм* из группы ресурсов "myResourceGroup" и назначает ее переменной *$sourceVm*. 

```azurepowershell-interactive
$sourceVm = Get-AzVM `
   -Name sourceVM `
   -ResourceGroupName myResourceGroup
```

Рекомендуется стоп\деаллокате виртуальную машину перед созданием образа с помощью команды " [останавливается-AzVM](/powershell/module/az.compute/stop-azvm)".

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName $sourceVm.ResourceGroupName `
   -Name $sourceVm.Name `
   -Force
```

## <a name="create-an-image-definition"></a>Создание определения образа 

Образы можно объединять в логические группы с помощью определений образов. Они используются для управления сведениями об образе. В имени определения образа можно использовать прописные и строчные буквы, цифры, точки и дефисы. 

При создании определения образа убедитесь, что содержит все правильные сведения. Если вы обобщенно используете виртуальную машину (с помощью Sysprep для Windows или waagent-unготовить для Linux), то следует создать определение образа с помощью `-OsState generalized` . Если виртуальная машина не была обобщена, создайте определение образа с помощью `-OsState specialized` .

Дополнительные сведения о значениях, которые можно указать для определения образа, см. в разделе [Определения образов](./shared-image-galleries.md#image-definitions).

Создайте определение образа с помощью командлета [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). 

В этом примере определение образа называется *мимажедефинитион* и предназначено для специализированной виртуальной машины под Windows. Чтобы создать определение для образов с помощью Linux, используйте `-OsType Linux` . 

```azurepowershell-interactive
$imageDefinition = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-an-image-version"></a>Создание версии образа

Создайте версию образа с помощью команды [New-азгаллеримажеверсион](/powershell/module/az.compute/new-azgalleryimageversion). 

Допустимыми знаками для имени версии образа являются цифры и точки. Числа должны быть в диапазоне 32-битного целого числа. Формат: *основной номер версии*.*дополнительный номер версии*.*исправление*.

В этом примере используется версия образа *1.0.0*, которая реплицируется в центры обработки данных в регионах *центрально-западная часть США* и *центрально-южная часть США*. При выборе целевых регионов для репликации Помните, что также необходимо включить *Исходный* регион в качестве цели для репликации.

Чтобы создать версию образа на основе виртуальной машины, используйте для параметра `-SourceImageId` значение `$vm.Id.ToString()`.

```azurepowershell-interactive
   $region1 = @{Name='South Central US';ReplicaCount=1}
   $region2 = @{Name='East US';ReplicaCount=2}
   $targetRegions = @($region1,$region2)

$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -TargetRegion $targetRegions  `
   -SourceImageId $sourceVm.Id.ToString() `
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
> Вы также можете сохранить образ в хранилище класса Premium, добавив `-StorageAccountType Premium_LRS` [хранилище или избыточное в зону](../storage/common/storage-redundancy.md) , добавив `-StorageAccountType Standard_ZRS` его при создании версии образа.
>

## <a name="next-steps"></a>Дальнейшие действия

Убедившись, что новая версия образа работает правильно, можно создать виртуальную машину. Создайте виртуальную машину на основе [специализированной версии образа](vm-specialized-image-version-powershell.md) или [обобщенной версии образа](vm-generalized-image-version-powershell.md).

Сведения о том, как предоставить сведения о плане покупки, см. в разделе [предоставление сведений о плане покупки Azure Marketplace при создании образов](marketplace-images.md).