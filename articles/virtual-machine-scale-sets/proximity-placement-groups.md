---
title: Предварительная версия групп размещения близкого взаимодействия для масштабируемых наборов виртуальных машин
description: Сведения о создании и использовании групп размещения близкого взаимодействия для масштабируемых наборов виртуальных машин Windows в Azure.
author: cynthn
ms.author: cynthn
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: proximity-placement-groups
ms.date: 07/01/2019
ms.reviewer: zivr
ms.custom: mimckitt
ms.openlocfilehash: eeb0dd6cd5ce838c0f3a68763c40612038ee82f0
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105933497"
---
# <a name="preview-creating-and-using-proximity-placement-groups-using-powershell"></a>Предварительный просмотр: Создание и использование групп размещения близкого взаимодействия с помощью PowerShell

Чтобы виртуальные машины были максимально близки для обеспечения наименьшей задержки, необходимо развернуть масштабируемый набор в [группе размещения близкого взаимодействия](../virtual-machines/co-location.md#proximity-placement-groups).

Группа размещения близкого взаимодействия — это логическая группировка, используемая для того, чтобы ресурсы вычислений Azure физически размещались близко друг к другу. Группа размещения близкого взаимодействия удобны для рабочих нагрузок, где требуется низкая задержка.

> [!IMPORTANT]
> В настоящее время доступна общедоступная предварительная версия группы размещения близкого взаимодействия.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>
> Предварительная версия групп размещения близкого взаимодействия недоступна в следующих регионах: **Восточная Япония**, **Восточная Австралия** и **Центральная Индия**.


## <a name="create-a-proximity-placement-group"></a>Создание группы размещения близкого взаимодействия
Создайте группу размещения близкого взаимодействия с помощью командлета [New-AzProximityPlacementGroup](/powershell/module/az.compute/new-azproximityplacementgroup). 

```azurepowershell-interactive
$resourceGroup = "myPPGResourceGroup"
$location = "East US"
$ppgName = "myPPG"
New-AzResourceGroup -Name $resourceGroup -Location $location
$ppg = New-AzProximityPlacementGroup `
   -Location $location `
   -Name $ppgName `
   -ResourceGroupName $resourceGroup `
   -ProximityPlacementGroupType Standard
```

## <a name="list-proximity-placement-groups"></a>Перечисление групп размещения близкого взаимодействия

Список всех групп размещения с учетом расположения можно получить с помощью командлета [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup).

```azurepowershell-interactive
Get-AzProximityPlacementGroup
```


## <a name="create-a-scale-set"></a>Создание масштабируемого набора

Создайте масштаб в группе размещения близкого взаимодействия с помощью `-ProximityPlacementGroup $ppg.Id` для ссылки на идентификатор группы размещения близкого взаимодействия при использовании [New-AzVMSS](/powershell/module/az.compute/new-azvmss) для создания масштабируемого набора.

```azurepowershell-interactive
$scalesetName = "myVM"

New-AzVmss `
  -ResourceGroupName $resourceGroup `
  -Location $location `
  -VMScaleSetName $scalesetName `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic" `
  -ProximityPlacementGroup $ppg.Id
```

Экземпляр можно просмотреть в группе размещения с помощью [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup).

```azurepowershell-interactive
  Get-AzProximityPlacementGroup `
   -ResourceId $ppg.Id | Format-Table `
   -Wrap `
   -Property VirtualMachineScaleSets
```

## <a name="next-steps"></a>Дальнейшие действия

Можно также использовать [Azure CLI](../virtual-machines/linux/proximity-placement-groups.md) для создания групп размещения близкого взаимодействия.
