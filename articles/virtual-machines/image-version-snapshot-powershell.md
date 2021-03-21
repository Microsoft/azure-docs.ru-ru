---
title: PowerShell. Создание образа из моментального снимка или управляемого диска в общей коллекции образов
description: Узнайте, как создать образ из моментального снимка или управляемого диска в общей коллекции образов с помощью PowerShell.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.workload: infrastructure
ms.date: 06/30/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 70ef194ab7f7403f31ad431eba25db5f71dfb580
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102556909"
---
# <a name="create-an-image-from-a-managed-disk-or-snapshot-in-a-shared-image-gallery-using-powershell"></a>Создание образа из управляемого диска или моментального снимка в коллекции общих образов с помощью PowerShell

Если у вас есть моментальный снимок или управляемый диск, который вы хотите перенести в коллекцию общих образов, можно создать образ из коллекции общих образов непосредственно с управляемого диска или из моментального снимка. После тестирования нового образа можно удалить исходный управляемый диск или моментальный снимок. Вы также можете создать образ из управляемого диска или моментального снимка в коллекции общих образов с помощью [Azure CLI](image-version-snapshot-cli.md).

Изображения в коллекции образов имеют два компонента, которые будут созданы в этом примере:
- **Определение образа** содержит сведения о образе и требования для его использования. Сюда входит, является ли образ Windows или Linux, специализированным или обобщенным, заметками о выпуске, а также минимальным и максимальным требованиями к памяти. Это определение типа образа. 
- **Версия образа** — это то, что используется для создания виртуальной машины при использовании общей коллекции образов. В зависимости от требований для вашей среды, у вас может быть несколько версий образа. При создании виртуальной машины используется версия образа для создания новых дисков для виртуальной машины. Версии образов можно использовать несколько раз.


## <a name="before-you-begin"></a>Перед началом

Для работы с этой статьей необходим моментальный снимок или управляемый диск. 

Если вы хотите включить диск данных, размер диска данных не может превышать 1 ТБ.

При работе с этой статьей при необходимости замените имена ресурсов.


## <a name="get-the-snapshot-or-managed-disk"></a>Получение моментального снимка или управляемого диска

Список моментальных снимков, доступных в группе ресурсов, можно просмотреть с помощью команды [Get-азснапшот](/powershell/module/az.compute/get-azsnapshot). 

```azurepowershell-interactive
get-azsnapshot | Format-Table -Property Name,ResourceGroupName
```

Зная имя моментального снимка и группу ресурсов, в которой он находится, можно `Get-AzSnapshot` снова использовать для получения объекта моментального снимка и сохранить его в переменной для последующего использования. Этот пример получает моментальный снимок с именем *миснапшот* из группы ресурсов "myResourceGroup" и присваивает его переменной *$Source*. 

```azurepowershell-interactive
$source = Get-AzSnapshot `
   -SnapshotName mySnapshot `
   -ResourceGroupName myResourceGroup
```

Вместо моментального снимка можно также использовать управляемый диск. Чтобы получить управляемый диск, используйте [Get-аздиск](/powershell/module/az.compute/get-azdisk). 

```azurepowershell-interactive
Get-AzDisk | Format-Table -Property Name,ResourceGroupName
```

Затем получите управляемый диск и присвойте его `$source` переменной.

```azurepowershell-interactive
$source = Get-AzDisk `
   -Name myDisk
   -ResourceGroupName myResourceGroup
```

Те же командлеты можно использовать для получения любых дисков данных, которые необходимо включить в образ. Назначьте их переменным, а затем используйте эти переменные позже при создании версии образа.


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

При создании определения образа убедитесь, что содержит все правильные сведения. В этом примере предполагается, что моментальный снимок или управляемый диск относятся к используемой виртуальной машине и не были обобщены. Если управляемый диск или моментальный снимок был создан в обобщенной ОС (после запуска Sysprep для Windows или [waagent](https://github.com/Azure/WALinuxAgent) `-deprovision` или `-deprovision+user` для Linux), измените `-OsState` на `generalized` . 

Дополнительные сведения о значениях, которые можно указать для определения образа, см. в разделе [Определения образов](./shared-image-galleries.md#image-definitions).

Создайте определение образа с помощью командлета [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). В этом примере определение образа называется *мимажедефинитион* и предназначено для специализированной ОС Windows. Чтобы создать определение для образов с помощью ОС Linux, используйте `-OsType Linux` . 

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

### <a name="purchase-plan-information"></a>Сведения о плане покупки

В некоторых случаях при создании виртуальной машины из образа, созданного на основе образа Azure Marketplace, необходимо передать сведения о плане покупки в. В этих случаях мы рекомендуем включить сведения о плане покупки в определение образа. В этом случае см. статью [предоставление сведений о плане покупки Azure Marketplace при создании образов](marketplace-images.md).


## <a name="create-an-image-version"></a>Создание версии образа

Создайте версию образа из моментального снимка, используя [New-азгаллеримажеверсион](/powershell/module/az.compute/new-azgalleryimageversion). 

Допустимыми знаками для имени версии образа являются цифры и точки. Числа должны быть в диапазоне 32-битного целого числа. Формат: *основной номер версии*.*дополнительный номер версии*.*исправление*.

Если вы хотите, чтобы образ содержал диск данных, помимо диска операционной системы, добавьте `-DataDiskImage` параметр и задайте для него идентификатор моментального снимка диска данных или управляемого диска.

В этом примере используется версия образа *1.0.0*, которая реплицируется в центры обработки данных в регионах *центрально-западная часть США* и *центрально-южная часть США*. При выборе целевых регионов для репликации Помните, что также необходимо включить *Исходный* регион в качестве цели для репликации.


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West Central US';ReplicaCount=2}
$targetRegions = @($region1,$region2)
$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name `
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -TargetRegion $targetRegions  `
   -OSDiskImage @{Source = @{Id=$source.Id}; HostCaching = "ReadOnly" } `
   -PublishingProfileEndOfLifeDate '2025-01-01' `
   -asJob 
```

Репликация образа во все целевые регионы может занять некоторое время, поэтому мы создали задание, чтобы отслеживать ход выполнения. Чтобы просмотреть ход выполнения задания, введите `$job.State`.

```azurepowershell-interactive
$job.State
```

> [!NOTE]
> Перед тем как использовать тот же моментальный снимок для создания другой версии образа, необходимо дождаться, пока завершится сборка и репликация версии образа. 
>
> Можно также сохранить версию образа в хранилище, [избыточном](../storage/common/storage-redundancy.md) в виде зоны, путем добавления `-StorageAccountType Standard_ZRS` при создании версии образа.
>

## <a name="delete-the-source"></a>Удаление источника

Убедившись, что новая версия образа работает правильно, можно удалить источник образа с помощью [Remove-азснапшот](/powershell/module/Az.Compute/Remove-AzSnapshot) или [Remove-аздиск](/powershell/module/az.compute/remove-azdisk).


## <a name="next-steps"></a>Дальнейшие действия

Убедившись, что репликация завершена, можно создать виртуальную машину из [специализированного образа](vm-specialized-image-version-powershell.md).

Сведения о том, как предоставить сведения о плане покупки, см. в разделе [предоставление сведений о плане покупки Azure Marketplace при создании образов](marketplace-images.md).