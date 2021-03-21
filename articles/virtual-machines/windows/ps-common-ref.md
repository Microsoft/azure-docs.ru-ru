---
title: Общие команды PowerShell для виртуальных машин Azure
description: Распространенные команды PowerShell, позволяющие приступить к созданию виртуальных машин и управлению ими в Azure.
author: cynthn
ms.service: virtual-machines
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/01/2018
ms.author: cynthn
ms.openlocfilehash: ecc3f5934e4dd37cc27a1715ea39f86619d581ff
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98730212"
---
# <a name="common-powershell-commands-for-creating-and-managing-azure-virtual-machines"></a>Общие команды PowerShell для создания виртуальных машин Azure и управления ими

В этой статье рассматриваются некоторые из команд Azure PowerShell, которые можно использовать для создания виртуальных машин в подписке Azure и управления ими.  Для получения более подробных сведений о параметрах и параметрах командной строки можно использовать команду **Get-Help** .

 

При выполнении нескольких команд, описываемых в этой статье, могут пригодиться приведенные ниже переменные.

- $location — расположение виртуальной машины. Вы можете использовать командлет [Get-AzLocation](/powershell/module/az.resources/get-azlocation), чтобы найти используемый [географический регион](https://azure.microsoft.com/regions/).
- $myResourceGroup — имя группы ресурсов, содержащей виртуальную машину.
- $myVM — имя виртуальной машины.

## <a name="create-a-vm---simplified"></a>Создание упрощенной виртуальной машины

| Задача | Get-Help |
| ---- | ------- |
| Создание простой виртуальной машины | [New-AzVM](/powershell/module/az.compute/new-azvm) -Name $myVM <BR></BR><BR></BR> Команда New-AzVM имеет набор *упрощенных* параметров, в которых нужно указать только одно имя. Значения параметра -Name будет использоваться как имя для всех ресурсов, необходимых для создания виртуальной машины. Можно указать дополнительные значения, но только это является обязательным.|
| Создание виртуальной машины из пользовательского образа | New-AzVm -ResourceGroupName $myResourceGroup -Name $myVM ImageName "myImage" -Location $location  <BR></BR><BR></BR>У вас уже должен быть собственный [управляемый образ](capture-image-resource.md). Его можно использовать для создания нескольких одинаковых виртуальных машин. |



## <a name="create-a-vm-configuration"></a>Создание конфигурации виртуальной машины

| Задача | Get-Help |
| ---- | ------- |
| Создание конфигурации виртуальной машины |$vm = [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig) -VMName $myVM -VMSize "Standard_D1_v1"<BR></BR><BR></BR>Конфигурация виртуальной машины используется для определения или обновления параметров виртуальной машины. Для инициализации конфигурации используется имя виртуальной машины и ее [размер](../sizes.md). |
| Добавление параметров конфигурации |$vm = [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) -VM $vm -Windows -ComputerName $myVM -Credential $cred -ProvisionVMAgent -EnableAutoUpdate<BR></BR><BR></BR>Параметры операционной системы, в том числе [учетные данные](/powershell/module/microsoft.powershell.security/get-credential), добавляются в объект конфигурации, созданный ранее с помощью командлета New-AzVMConfig. |
| Добавление сетевого интерфейса |$vm = [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) -VM $vm -Id $nic.Id<BR></BR><BR></BR>У виртуальной машины должен быть [сетевой интерфейс](./quick-create-powershell.md?toc=/azure/virtual-machines/windows/toc.json) для обмена данными в виртуальной сети. Вы можете также использовать командлет [Get-AzNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) для получения существующего объекта сетевого интерфейса. |
| Указание образа платформы |$vm = [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage) -VM $vm -PublisherName "publisher_name" -Offer "publisher_offer" -Skus "product_sku" -Version "latest"<BR></BR><BR></BR>В объект конфигурации, созданный ранее с помощью командлета New-AzVMConfig, добавляется [информация об образе](cli-ps-findimage.md). Объект, возвращенный этой командой, используется только в том случае, если диск операционной системы настроен для использования образа платформы. |
| Создание виртуальной машины |[New-AzVM](/powershell/module/az.compute/new-azvm) -ResourceGroupName $myResourceGroup -Location $location -VM $vm<BR></BR><BR></BR>Все ресурсы создаются в [группе ресурсов](../../azure-resource-manager/management/manage-resource-groups-powershell.md). Прежде чем выполнить эту команду, выполните командлеты New-AzVMConfig, Set-AzVMOperatingSystem, Set-AzVMSourceImage, Add-AzVMNetworkInterface и Set-AzVMOSDisk. |
| Обновление виртуальной машины |[Update-AzVM](/powershell/module/az.compute/update-azvm) -ResourceGroupName $myResourceGroup -VM $vm<BR></BR><BR></BR>Получите текущую конфигурацию виртуальной машины с помощью командлета Get-AzVM, измените параметры конфигурации в объекте виртуальной машины, после чего выполните данную команду. |

## <a name="get-information-about-vms"></a>Получение информации о виртуальных машинах

| Задача | Get-Help |
| ---- | ------- |
| Вывод списка виртуальных машин в подписке |[Get-AzVM](/powershell/module/az.compute/get-azvm) |
| Вывод списка виртуальных машин в группе ресурсов |Get-AzVM -ResourceGroupName $myResourceGroup<BR></BR><BR></BR>Для получения списка групп ресурсов в подписке используйте командлет [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup). |
| Получение информации о виртуальной машине |Get-AzVM -ResourceGroupName $myResourceGroup -Name $myVM |

## <a name="manage-vms"></a>Управление виртуальными машинами
| Задача | Get-Help |
| --- | --- |
| Запуск виртуальной машины |[Start-AzVM](/powershell/module/az.compute/start-azvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| Остановка виртуальной машины |[Stop-AzVM](/powershell/module/az.compute/stop-azvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| Перезапуск выполняющейся виртуальной машины |[Restart-AzVM](/powershell/module/az.compute/restart-azvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| Удаление виртуальной машины |[Remove-AzVM](/powershell/module/az.compute/remove-azvm) -ResourceGroupName $myResourceGroup -Name $myVM |


## <a name="next-steps"></a>Дальнейшие действия
* Ознакомьтесь с основными шагами по созданию виртуальной машины в разделе [Создание виртуальной машины Windows с помощью Resource Manager и PowerShell](./quick-create-powershell.md?toc=/azure/virtual-machines/windows/toc.json).
