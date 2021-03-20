---
title: Создание масштабируемого набора из обобщенного образа с помощью Azure PowerShell
description: Создание масштабируемого набора с помощью обобщенного образа в коллекции общих образов с помощью PowerShell.
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 9edade1aa54d6f4f8160a107f84e6da2e6cf316e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98878025"
---
# <a name="create-a-scale-set-from-a-generalized-image-using-powershell"></a>Создание масштабируемого набора из обобщенного образа с помощью PowerShell 

Создайте виртуальную машину из универсальной версии образа, хранящейся в [коллекции общих образов](../virtual-machines/shared-image-galleries.md). Если требуется создать масштабируемый набор с помощью специализированного образа, см. раздел [Создание экземпляров масштабируемых наборов на основе специализированного образа](instance-specialized-image-version-powershell.md).

После создания обобщенного образа можно создать масштабируемый набор виртуальных машин с помощью командлета [New-азвмсс](/powershell/module/az.compute/new-azvmss) . 

В этом примере мы используем идентификатор определения образа, чтобы убедиться, что новая виртуальная машина будет использовать самую последнюю версию образа. Вы также можете использовать определенную версию с помощью идентификатора версии образа для `-ImageReferenceId` . Например, чтобы использовать тип Image версии *1.0.0* : `-ImageReferenceId "/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"` . 

Имейте в виду, что использование конкретной версии образа означает, что автоматизация может завершиться ошибкой, если эта версия изображения недоступна, так как она была удалена или удалена из региона. Рекомендуется использовать идентификатор определения образа для создания новой виртуальной машины, если только не требуется определенная версия образа.


В следующих примерах создается масштабируемый набор с именем *myScaleSet* в группе ресурсов *мивмссрг* в расположении *SouthCentralUS* . Масштабируемый набор будет создан из образа *мимажедефинитион* в коллекции образов *мигаллери* в группе ресурсов *мигаллерирг* . При появлении запроса задайте собственные учетные данные администратора для экземпляров виртуальных машин в масштабируемом наборе.


## <a name="simplified-parameter-set"></a>Упрощенный набор параметров

Чтобы быстро создать масштабируемый набор с минимальными сведениями, используйте упрощенный набор параметров, чтобы создать масштабируемый набор из образа из коллекции общих образов.

```azurepowershell-interactive
$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myGalleryRG `
   -Name myImageDefinition

# Create user object

$cred = Get-Credential `
   -Message "Enter a username and password for the virtual machine."

# Create the resource group and scale set
New-AzResourceGroup -ResourceGroupName myVMSSRG -Location SouthCentralUS
New-AzVmss `
   -Credential $cred `
   -VMScaleSetName myScaleSet `
   -ImageName $imageDefinition.Id `
   -UpgradePolicyMode Automatic `
   -ResourceGroupName myVMSSRG
```

Создание и настройка всех ресурсов и виртуальных машин масштабируемого набора занимает несколько минут.

## <a name="extended-parameter-set"></a>Набор расширенных параметров

Для полного контроля над всеми ресурсами, включая именование, используйте полный набор параметров для создания масштабируемого набора с помощью общего образа коллекции образов. 

```azurepowershell-interactive
# Get the image definition

$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myGalleryRG `
   -Name myImageDefinition

# Create user object

$cred = Get-Credential `
   -Message "Enter a username and password for the virtual machine."
   
# Define variables for the scale set
$resourceGroupName = "myVMSSRG"
$scaleSetName = "myScaleSet"
$location = "South Central US"

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
  -ImageReferenceId $imageDefinition.Id

# Complete the configuration
Set-AzVmssOsProfile $vmssConfig `
  -AdminUsername $cred.UserName `
  -AdminPassword $cred.Password `
  -ComputerNamePrefix "myVM"
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

## <a name="next-steps"></a>Дальнейшие действия
С помощью [построителя образов Azure (Предварительная версия)](../virtual-machines/image-builder-overview.md) можно автоматизировать создание версий изображений, а также использовать его для обновления и [создания новой версии образа из существующей версии образа](../virtual-machines/linux/image-builder-gallery-update-image-version.md). 

Вы также можете создать ресурс коллекции общих образов с помощью шаблонов. Существует несколько шаблонов быстрого запуска Azure: 

- [Создание коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Создание определения образа в коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Создание версии образа в коллекции общих образов](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)

Дополнительные сведения о коллекциях общих образов см. в [обзорной статье](../virtual-machines/shared-image-galleries.md). Если вы столкнетесь с проблемами, обратитесь к статье об [устранении неполадок c коллекциями общих образов](../virtual-machines/troubleshooting-shared-images.md).