---
title: Управление масштабируемыми наборами виртуальных машин с помощью Azure PowerShell
description: Стандартные командлеты Azure PowerShell для управления масштабируемыми наборами виртуальных машин, в том числе команды для запуска и остановки экземпляра, а также для изменения емкости масштабируемого набора.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 05/29/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: eee4dd7fae872f6b3ddd01f60aba732edc170766
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91570577"
---
# <a name="manage-a-virtual-machine-scale-set-with-azure-powershell"></a>Управление масштабируемым набором виртуальных машин с помощью Azure PowerShell

На протяжении жизненного цикла масштабируемого набора виртуальных машин может возникнуть необходимость выполнить одну или несколько задач управления. Кроме того, можно создавать сценарии для автоматизации различных задач жизненного цикла. В этой статье подробно рассматриваются некоторые стандартные командлеты Azure PowerShell, которые позволяют выполнять эти задачи.

Если необходимо создать масштабируемый набор виртуальных машин, это можно [сделать с помощью Azure PowerShell](quick-create-powershell.md).

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

## <a name="view-information-about-a-scale-set"></a>Просмотр информации о масштабируемом наборе
С помощью командлета [Get-AzVmss](/powershell/module/az.compute/get-azvmss) просмотрите общие сведения о масштабируемом наборе. Следующий пример возвращает сведения о масштабируемом наборе *myScaleSet* в группе ресурсов *myResourceGroup*. Введите свои имена следующим образом.

```powershell
Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```


## <a name="view-vms-in-a-scale-set"></a>Просмотр виртуальных машин в масштабируемом наборе
С помощью командлета [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) просмотрите список экземпляров виртуальных машин в масштабируемом наборе. Следующий пример выводит списки всех экземпляров виртуальных машин в масштабируемом наборе *myScaleSet* в группе ресурсов *myResourceGroup*. Введите собственные значения для имен.

```powershell
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

Чтобы просмотреть дополнительные сведения о конкретном экземпляре виртуальной машины, добавьте параметр `-InstanceId` в командлет [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) и укажите экземпляр для просмотра. Следующий пример возвращает сведения об экземпляре виртуальной машины *0* в масштабируемом наборе *myScaleSet* в группе ресурсов *myResourceGroup*. Введите свои имена следующим образом.

```powershell
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```

Вы также можете получить подробные сведения о *instanceView* для всех экземпляров в одном вызове API, что поможет избежать регулирования API для больших установок.

```powershell
Get-AzVmssVM -InstanceView -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

```rest
GET "https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<resourceGroupName>/providers/Microsoft.Compute/virtualMachineScaleSets/<VMSSName>/virtualMachines?api-version=2019-03-01&%24expand=instanceView"
```

## <a name="change-the-capacity-of-a-scale-set"></a>Изменение емкости масштабируемого набора
Указанные выше команды показывали сведения о масштабируемом наборе и экземплярах виртуальных машин. Чтобы увеличить или уменьшить число экземпляров в масштабируемом наборе, можно изменить его емкость. Масштабируемый набор автоматически создает или удаляет необходимое количество виртуальных машин, а затем настраивает виртуальные машины для приема трафика приложения.

Сначала создайте объект масштабируемого набора с помощью командлета [Get-AzVmss](/powershell/module/az.compute/get-azvmss), затем укажите новое значение для `sku.capacity`. С помощью командлета [Update-AzVmss](/powershell/module/az.compute/update-azvmss) примените изменение емкости. В следующем примере емкость масштабируемого набора *myScaleSet* в группе ресурсов *myResourceGroup* изменяется до *5* экземпляров. Укажите свои значения следующим образом.

```powershell
# Get current scale set
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$vmss.sku.capacity = 5
Update-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -VirtualMachineScaleSet $vmss
```

На обновление емкости масштабируемого набора требуется несколько минут. При уменьшении емкости масштабируемого набора первыми удаляются виртуальные машины с наибольшим значением идентификатора экземпляра.


## <a name="stop-and-start-vms-in-a-scale-set"></a>Остановка и запуск виртуальных машин в масштабируемом наборе
С помощью командлета [Stop-AzVmss](/powershell/module/az.compute/stop-azvmss) остановите одну или несколько виртуальных машин в масштабируемом наборе. С помощью параметра `-InstanceId` можно указать одну или несколько виртуальных машин, которые нужно остановить. Если не указать идентификатор экземпляра, останавливаются все виртуальные машины в масштабируемом наборе. Чтобы остановить несколько виртуальных машин, разделите идентификаторы экземпляров запятыми.

Следующий пример останавливает экземпляр *0* в масштабируемом наборе *myScaleSet* в группе ресурсов *myResourceGroup*. Укажите свои значения следующим образом.

```powershell
Stop-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```

По умолчанию остановленные виртуальные машины освобождаются, и за них не взимается плата за вычислительные операции. Если требуется, чтобы остановленная виртуальная машина осталась в подготовленном состоянии, добавьте параметр `-StayProvisioned` в предыдущую команду. За остановленные виртуальные машины, которые остаются в подготовленном состоянии, взимается плата за вычислительные операции по стандартной ставке.


### <a name="start-vms-in-a-scale-set"></a>Запуск виртуальных машин в масштабируемом наборе
С помощью командлета [Start-AzVmss](/powershell/module/az.compute/start-azvmss) запустите одну или несколько виртуальных машин в масштабируемом наборе. С помощью параметра `-InstanceId` можно указать одну или несколько виртуальных машин, которые нужно запустить. Если не указать идентификатор экземпляра, запускаются все виртуальные машины в масштабируемом наборе. Чтобы запустить несколько виртуальных машин, разделите идентификаторы экземпляров запятыми.

Следующий пример запускает экземпляр *0* в масштабируемом наборе *myScaleSet* в группе ресурсов *myResourceGroup*. Укажите свои значения следующим образом.

```powershell
Start-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="restart-vms-in-a-scale-set"></a>Перезапуск виртуальных машин в масштабируемом наборе
Чтобы перезапустить одну или несколько виртуальных машин в масштабируемом наборе, используйте командлет [Restart-AzVmss](/powershell/module/az.compute/restart-azvmss). С помощью параметра `-InstanceId` можно указать одну или несколько виртуальных машин, которые нужно перезапустить. Если не указать идентификатор экземпляра, перезапускаются все виртуальные машины в масштабируемом наборе. Чтобы перезапустить несколько виртуальных машин, разделите идентификаторы экземпляров запятыми.

Следующий пример перезапускает экземпляр *0* в масштабируемом наборе *myScaleSet* в группе ресурсов *myResourceGroup*. Укажите свои значения следующим образом.

```powershell
Restart-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="remove-vms-from-a-scale-set"></a>Удаление виртуальных машин из масштабируемого набора
Чтобы удалить одну или несколько виртуальных машин из масштабируемого набора, используйте командлет [Remove-AzVmss](/powershell/module/az.compute/remove-azvmss). С помощью параметра `-InstanceId` можно указать одну или несколько виртуальных машин, которые нужно удалить. Если не указать идентификатор экземпляра, удаляются все виртуальные машины в масштабируемом наборе. Чтобы удалить несколько виртуальных машин, разделите идентификаторы экземпляров запятыми.

Следующий пример удаляет экземпляр *0* в масштабируемом наборе *myScaleSet* в группе ресурсов *myResourceGroup*. Укажите свои значения следующим образом.

```powershell
Remove-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "0"
```


## <a name="next-steps"></a>Дальнейшие действия
К другим типичным задачам управления масштабируемым набором относятся [развертывание приложения](virtual-machine-scale-sets-deploy-app.md) и [обновление экземпляров виртуальных машин](virtual-machine-scale-sets-upgrade-scale-set.md). Можно также использовать Azure PowerShell для [настройки правил автомасштабирования](virtual-machine-scale-sets-autoscale-overview.md).
