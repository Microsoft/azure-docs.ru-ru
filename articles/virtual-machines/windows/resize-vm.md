---
title: Изменение размера виртуальной машины Windows в Azure
description: Измените размер ВИРТУАЛЬНОЙ машины, используемой для виртуальной машины Azure.
author: cynthn
ms.service: virtual-machines
ms.collection: windows
ms.workload: infrastructure
ms.topic: how-to
ms.date: 01/13/2020
ms.author: cynthn
ms.openlocfilehash: f7f41c0fb2bf949dd47491761fde8025a2d35155
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102560656"
---
# <a name="resize-a-windows-vm"></a>Изменение размера виртуальной машины Windows

В этой статье показано, как переместить виртуальную машину на другой [Размер виртуальной машины](../sizes.md).

После того как вы создали виртуальную машину, ее можно масштабировать, изменяя размер. В некоторых случаях сначала необходимо освободить виртуальную машину. Это может случиться, если новый размер недоступен в кластере оборудования, в котором она сейчас размещена.

Если виртуальная машина использует хранилище класса Premium, выберите версию размера **s**, чтобы получить поддержку этого хранилища. Например, выберите Standard_E4 **s** _v3 вместо Standard_E4_v3.

## <a name="use-the-portal"></a>Использование портала

1. Откройте [портал Azure](https://portal.azure.com).
1. Откройте страницу виртуальной машины.
1. В меню слева выберите **Размер**.
1. Выберите новый размер из списка доступных размеров и нажмите кнопку **изменить размер**.


Если виртуальная машина выполняется в данный момент, изменение ее размера приведет к ее перезапуску. Остановка виртуальной машины может привести к раскрытию дополнительных размеров.

## <a name="use-powershell-to-resize-a-vm-not-in-an-availability-set"></a>Использование PowerShell для изменения размера виртуальной машины, не наданной в группе доступности

Задайте несколько переменных. Подставьте собственные значения.

```powershell
$resourceGroup = "myResourceGroup"
$vmName = "myVM"
```

Выведите список размеров виртуальной машины, доступных в кластере оборудования, в котором она размещена. 
   
```powershell
Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName 
```

Если в списке есть нужный размер, выполните следующие команды, чтобы изменить размер виртуальной машины. Если нужный размер отсутствует в списке, перейдите к шагу 3.
   
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$vm.HardwareProfile.VmSize = "<newVMsize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

Если нужного размера нет в списке, выполните следующие команды, чтобы освободить виртуальную машину, изменить ее размер и перезагрузить. Замените **\<newVMsize>** на нужный размер.
   
```powershell
Stop-AzVM -ResourceGroupName $resourceGroup -Name $vmName -Force
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName
$vm.HardwareProfile.VmSize = "<newVMSize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
Start-AzVM -ResourceGroupName $resourceGroup -Name $vmName
```

> [!WARNING]
> При освобождении виртуальной машины освобождаются все назначенные ей динамические IP-адреса. Это не влияет на диски ОС и данных. 
> 
> 

## <a name="use-powershell-to-resize-a-vm-in-an-availability-set"></a>Изменение размера виртуальной машины в группе доступности с помощью PowerShell

Если новый размер виртуальной машины, входящей в группу доступности, недоступен в кластере оборудования, в котором она сейчас размещена, то для изменения размера этой виртуальной машины потребуется освободить все виртуальные машины в данной группе доступности. После изменения размера одной виртуальной машины может также потребоваться обновить размер остальных виртуальных машин в группе доступности. Чтобы изменить размер виртуальной машины в группе доступности, выполните следующие действия.

```powershell
$resourceGroup = "myResourceGroup"
$vmName = "myVM"
```

Выведите список размеров виртуальной машины, доступных в кластере оборудования, в котором она размещена. 
   
```powershell
Get-AzVMSize -ResourceGroupName $resourceGroup -VMName $vmName 
```

Если в списке есть нужный размер, выполните следующие команды, чтобы изменить размер виртуальной машины. Если его нет в списке, перейдите к следующему разделу.
   
```powershell
$vm = Get-AzVM -ResourceGroupName $resourceGroup -VMName $vmName 
$vm.HardwareProfile.VmSize = "<newVmSize>"
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```
    
Если нужный размер отсутствует в списке, выполните приведенные ниже действия, чтобы освободить все виртуальные машины в группе доступности, изменить их размер, а затем перезапустить.

Остановите все виртуальные машины в группе доступности.
   
```powershell
$availabilitySetName = "<availabilitySetName>"
$as = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $availabilitySetName
$virtualMachines = $as.VirtualMachinesReferences |  Get-AzResource | Get-AzVM
$virtualMachines |  Stop-AzVM -Force -NoWait  
```

Измените размер виртуальных машин в группе доступности и перезапустите их.
   
```powershell
$availabilitySetName = "<availabilitySetName>"
$newSize = "<newVmSize>"
$as = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup -Name $availabilitySetName
$virtualMachines = $as.VirtualMachinesReferences |  Get-AzResource | Get-AzVM
$virtualMachines | Foreach-Object { $_.HardwareProfile.VmSize = $newSize }
$virtualMachines | Update-AzVM
$virtualMachines | Start-AzVM
```

## <a name="next-steps"></a>Дальнейшие действия

Для повышения масштабируемости запустите несколько экземпляров виртуальных машин и выполните горизонтальное масштабирование. Дополнительные сведения см. [в статье Автоматическое масштабирование компьютеров Windows в масштабируемом наборе виртуальных машин](../../virtual-machine-scale-sets/tutorial-autoscale-powershell.md).