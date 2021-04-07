---
title: Учебник. Использование пользовательского образа виртуальной машины в масштабируемом наборе с помощью Azure PowerShell
description: Сведения о создании пользовательского образа виртуальной машины, который можно использовать для развертывания масштабируемого набора виртуальных машин, с помощью Azure PowerShell
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.topic: tutorial
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: akjosh
ms.openlocfilehash: 803bbf485d783fc160d1f2644a56e1339f73690e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98678310"
---
# <a name="tutorial-create-and-use-a-custom-image-for-virtual-machine-scale-sets-with-azure-powershell"></a>Руководство по Создание и использование пользовательского образа для масштабируемых наборов виртуальных машин с помощью Azure PowerShell

Создавая масштабируемый набор, вы указываете образ для использования при развертывании экземпляров виртуальных машин. Чтобы сократить количество задач после развертывания экземпляров виртуальных машин, можно использовать пользовательский образ виртуальной машины. Этот образ содержит все необходимые установки или конфигурации приложения. Все экземпляры виртуальных машин, созданные в масштабируемом наборе, используют пользовательский образ виртуальной машины и готовы обслуживать трафик приложения. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * Создание Общей коллекции образов.
> * Создание определения образа
> * Создание версии образа
> * Создание масштабируемого набора на основе образа 
> * Предоставление общего доступа к коллекции образов

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="before-you-begin"></a>Перед началом

Ниже подробно описано, как преобразовать существующую виртуальную машину в многократно используемый пользовательский образ, на основе которого можно создавать масштабируемый набор.

Для выполнения примера в этом руководстве требуется виртуальная машина. При необходимости просмотрите [краткое руководство по PowerShell](quick-create-powershell.md), чтобы создать виртуальную машину для работы с этим учебником. При работе с учебником заменяйте имена ресурсов по мере необходимости.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

Azure Cloud Shell — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью. 

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Cloud Shell можно также запустить в отдельной вкладке браузера, перейдя на страницу [https://shell.azure.com/powershell](https://shell.azure.com/powershell). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу "ВВОД", чтобы выполнить его.


## <a name="get-the-vm"></a>Получение виртуальной машины

Список виртуальных машин, доступных в группе ресурсов, можно просмотреть с помощью командлета [Get-AzVM](/powershell/module/az.compute/get-azvm). Зная имя виртуальной машины и группу ресурсов, в которой она находится, можно снова выполнить командлет `Get-AzVM`, чтобы получить объект виртуальной машины и сохранить его в переменной для последующего использования. Этот пример получает виртуальную машину с именем *sourceVM* из группы ресурсов myResourceGroup и присваивает его переменной *$vm*. 

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

Образы можно объединять в логические группы с помощью определений образов. Определения образов используются для управления сведениями о версиях созданных в них образов. В имени определения образа можно использовать прописные и строчные буквы, цифры, точки и дефисы. Дополнительные сведения о значениях, которые можно указать для определения образа, см. в разделе [Определения образов](../virtual-machines/shared-image-galleries.md#image-definitions).

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

## <a name="create-a-scale-set-from-the-image"></a>Создание масштабируемого набора на основе образа
Теперь создайте масштабируемый набор с помощью командлета [New-AzVmss](/powershell/module/az.compute/new-azvmss), который использует параметр `-ImageName` для определения пользовательского образа виртуальной машины, созданного на предыдущем шаге. Чтобы распределить трафик между отдельными экземплярами виртуальных машин, создается еще и подсистема балансировки нагрузки. Подсистема балансировки нагрузки определяет правила передачи трафика на TCP-порт 80, а также разрешает подключение удаленного рабочего стола трафик через TCP-порт 3389 и удаленное взаимодействие PowerShell через TCP-порт 5985. При появлении запроса введите учетные данные администратора для экземпляров виртуальных машин в масштабируемом наборе:

```azurepowershell-interactive
# Define variables for the scale set
$resourceGroupName = "myVMSSRG3"
$scaleSetName = "myScaleSet3"
$location = "East US"

# Create a resource group
New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location

# Create a networking pieces
$subnet = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.0.0/24
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName $resourceGroupName `
  -Name "myVnet" `
  -Location $location `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnet
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName $resourceGroupName `
  -Location $location `
  -AllocationMethod Static `
  -Name "myPublicIP"
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "myFrontEndPool" `
  -PublicIpAddress $publicIP
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name "myBackEndPool"
$inboundNATPool = New-AzLoadBalancerInboundNatPoolConfig `
  -Name "myRDPRule" `
  -FrontendIpConfigurationId $frontendIP.Id `
  -Protocol TCP `
  -FrontendPortRangeStart 50001 `
  -FrontendPortRangeEnd 50010 `
  -BackendPort 3389
# Create the load balancer and health probe
$lb = New-AzLoadBalancer `
  -ResourceGroupName $resourceGroupName `
  -Name "myLoadBalancer" `
  -Location $location `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -InboundNatPool $inboundNATPool
Add-AzLoadBalancerProbeConfig -Name "myHealthProbe" `
  -LoadBalancer $lb `
  -Protocol TCP `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2
Add-AzLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol TCP `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe (Get-AzLoadBalancerProbeConfig -Name "myHealthProbe" -LoadBalancer $lb)
Set-AzLoadBalancer -LoadBalancer $lb

# Create IP address configurations
$ipConfig = New-AzVmssIpConfig `
  -Name "myIPConfig" `
  -LoadBalancerBackendAddressPoolsId $lb.BackendAddressPools[0].Id `
  -LoadBalancerInboundNatPoolsId $inboundNATPool.Id `
  -SubnetId $vnet.Subnets[0].Id

# Create a configuration 
$vmssConfig = New-AzVmssConfig `
    -Location $location `
    -SkuCapacity 2 `
    -SkuName "Standard_DS2" `
    -UpgradePolicyMode "Automatic"

# Reference the image version
Set-AzVmssStorageProfile $vmssConfig `
  -OsDiskCreateOption "FromImage" `
  -ImageReferenceId $galleryImage.Id

# Complete the configuration
 
Add-AzVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name "network-config" `
  -Primary $true `
  -IPConfiguration $ipConfig 

# Create the scale set 
New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Name $scaleSetName `
  -VirtualMachineScaleSet $vmssConfig
```

Создание и настройка всех ресурсов и виртуальных машин масштабируемого набора занимает несколько минут.


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

# Delete the scale set resource group
Remove-AzResourceGroup -Name myResoureceGroup
```

## <a name="azure-image-builder"></a>Средство создания образов Azure

Azure также предлагает службу на основе Packer — [Конструктор образов виртуальных машин Azure](../virtual-machines/image-builder-overview.md). Просто опишите настройки в шаблоне, и эта служба автоматически создаст образ. 

## <a name="next-steps"></a>Дальнейшие действия
Из этого руководства вы узнали, как создавать и использовать пользовательский образ виртуальной машины для масштабируемых наборов с помощью Azure PowerShell, в частности, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание Общей коллекции образов.
> * Создание определения образа
> * Создание версии образа
> * Создание масштабируемого набора на основе образа 
> * Предоставление общего доступа к коллекции образов

Перейдите к следующему руководству, чтобы узнать, как развертывать приложения в масштабируемый набор.

> [!div class="nextstepaction"]
> [Руководство по развертыванию приложений в масштабируемых наборах](tutorial-install-apps-powershell.md)