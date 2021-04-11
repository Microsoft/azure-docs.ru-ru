---
title: Учебник. Создание пользовательских образов виртуальных машин с помощью Azure PowerShell
description: В этом руководстве описано, как с помощью Azure PowerShell создать пользовательский образ виртуальной машины Windows, который будет храниться в Общей коллекции образов Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: shared-image-gallery
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 05/01/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 145b7300272243099cfccad0c99d8f69e83e6762
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102555828"
---
# <a name="tutorial-create-windows-vm-images-with-azure-powershell"></a>Руководство по Создание образов виртуальных машин Windows с помощью Azure PowerShell

Образы можно использовать для начальной загрузки развертываний и обеспечения согласованности между несколькими виртуальными машинами. С помощью этого руководства вы создадите собственный пользовательский образ виртуальной машины Azure с помощью PowerShell и сохраните его в Общей коллекции образов. Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание Общей коллекции образов.
> * Создание определения образа
> * Создание версии образа
> * Создание виртуальной машины из образа 
> * Предоставление общего доступа к коллекции образов.



## <a name="before-you-begin"></a>Перед началом

Ниже подробно описано, как преобразовать существующую виртуальную машину в многократно используемый пользовательский образ, на основе которого можно создавать виртуальные машины.

Для выполнения примера в этом руководстве требуется виртуальная машина. При необходимости просмотрите [краткое руководство по PowerShell](quick-create-powershell.md), чтобы создать виртуальную машину для работы с этим учебником. При работе с учебником заменяйте имена ресурсов по мере необходимости.

## <a name="overview"></a>Обзор

[Коллекция общих образов](../shared-image-galleries.md) упрощает обмен пользовательскими образами в вашей организации. Пользовательские образы похожи на образы магазина, однако их можно создавать самостоятельно. Пользовательские образы можно использовать для начальной загрузки конфигураций, например при предварительной загрузке приложений, конфигураций приложений и других конфигураций операционной системы. 

Общая коллекция образов позволяет предоставить другим пользователям общий доступ к пользовательским образам. Выберите образы, к которым нужно предоставить общий доступ, регионы, где они будут доступны, и пользователей, которым будет доступно совместное использование. 

Функция коллекции общих образов имеет несколько типов ресурсов.

[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]


## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.

## <a name="get-the-vm"></a>Получение виртуальной машины

Список виртуальных машин, доступных в группе ресурсов, можно просмотреть с помощью командлета [Get-AzVM](/powershell/module/az.compute/get-azvm). Зная имя виртуальной машины и группу ресурсов, в которой она находится, можно снова выполнить командлет `Get-AzVM`, чтобы получить объект виртуальной машины и сохранить его в переменной для последующего использования. В этом примере мы получаем виртуальную машину с именем *sourceVM* из группы ресурсов myResourceGroup и присваиваем ее переменной *$sourceVM*. 

```azurepowershell-interactive
$sourceVM = Get-AzVM `
   -Name sourceVM `
   -ResourceGroupName myResourceGroup
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте группу ресурсов с помощью команды [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *myGalleryRG* в регионе *EastUS*.

```azurepowershell-interactive
$resourceGroup = New-AzResourceGroup `
   -Name 'myGalleryRG' `
   -Location 'EastUS'
```

## <a name="create-an-image-gallery"></a>Создание коллекции образов 

Коллекция образов является основным ресурсом, который позволяет обмен изображениями. Имя коллекции может содержать прописные и строчные буквы, а также цифры и точки, но не дефисы. Имена коллекций должны быть уникальным в пределах вашей подписки. 

Создайте коллекцию образов, используя команду [New-AzGallery](/powershell/module/az.compute/new-azgallery). В следующем примере показано, как создать коллекцию с именем *myGallery* в группе ресурсов *myGalleryRG*.

```azurepowershell-interactive
$gallery = New-AzGallery `
   -GalleryName 'myGallery' `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -Description 'Shared Image Gallery for my organization'  
```


## <a name="create-an-image-definition"></a>Создание определения образа 

Образы можно объединять в логические группы с помощью определений образов. Определения образов используются для управления сведениями о версиях созданных в них образов. В имени определения образа можно использовать прописные и строчные буквы, цифры, точки и дефисы. Дополнительные сведения о значениях, которые можно указать для определения образа, см. в разделе [Определения образов](../shared-image-galleries.md#image-definitions).

Создайте определение образа с помощью командлета [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). В этом примере образ коллекции называется *myGalleryImage* и создается для специализированного образа. 

```azurepowershell-interactive
$galleryImage = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-an-image-version"></a>Создание версии образа

Создайте версию образа из виртуальной машины с помощью командлета [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Допустимыми знаками для имени версии образа являются цифры и точки. Числа должны быть в диапазоне 32-битного целого числа. Формат: *основной номер версии*.*дополнительный номер версии*.*исправление*.

В этом примере используется версия образа *1.0.0*, которая реплицируется в центры обработки данных в регионах *Восточная часть США* и *Центрально-южная часть США*. При выборе целевых регионов для репликации необходимо включить *исходный* регион в качестве целевого для репликации.

Чтобы создать версию образа на основе виртуальной машины, используйте для параметра `-Source` значение `$vm.Id.ToString()`.

```azurepowershell-interactive
   $region1 = @{Name='South Central US';ReplicaCount=1}
   $region2 = @{Name='East US';ReplicaCount=2}
   $targetRegions = @($region1,$region2)

New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $galleryImage.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -TargetRegion $targetRegions  `
   -Source $sourceVM.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-01'
```

Репликация образа во все целевые регионы может занять некоторое время.


## <a name="create-a-vm"></a>Создание виртуальной машины 

Получив специализированный образ, вы можете создать одну или несколько виртуальных машин. Используйте командлет [New-AzVM](/powershell/module/az.compute/new-azvm). Чтобы использовать образ, используйте `Set-AzVMSourceImage`, указав `-Id` в качестве идентификатора определения образа (в нашем примере — $galleryImage.Id). Тогда будет использоваться самая новая версия образа. 

При необходимости замените имена ресурсов в этом примере. 

```azurepowershell-interactive
# Create some variables for the new VM.
$resourceGroup = "myResourceGroup"
$location = "South Central US"
$vmName = "mySpecializedVM"

# Create a resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create the network resources.
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name $vmName -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using $imageVersion.Id to specify the image version.
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1_v2 | `
Set-AzVMSourceImage -Id $galleryImage.Id | `
Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
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
   
## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить ненужную группу ресурсов и все связанные с ней ресурсы, выполнив командлет [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```azurepowershell-interactive
# Delete the gallery 
Remove-AzResourceGroup -Name myGalleryRG

# Delete the VM
Remove-AzResourceGroup -Name myResoureceGroup
```

## <a name="azure-image-builder"></a>Средство создания образов Azure

Azure также предлагает службу на основе Packer — [Конструктор образов виртуальных машин Azure](../image-builder-overview.md). Просто опишите настройки в шаблоне, и эта служба автоматически создаст образ. 

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого руководства вы создали специализированный образ виртуальной машины. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * Создание Общей коллекции образов.
> * Создание определения образа
> * Создание версии образа
> * Создание виртуальной машины из образа 
> * Предоставление общего доступа к коллекции образов.

В следующем руководстве описано, как создавать виртуальные машины высокого уровня доступности.

> [!div class="nextstepaction"]
> [Создание высокодоступных виртуальных машин](tutorial-availability-sets.md)