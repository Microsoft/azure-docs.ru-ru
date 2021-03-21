---
title: Переключение диска ОС для виртуальной машины Azure с помощью PowerShell
description: Изменение диска операционной системы, используемого виртуальной машиной Azure, с помощью PowerShell.
author: cynthn
ms.service: virtual-machines
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/24/2018
ms.author: cynthn
ms.openlocfilehash: 8e928944a7508cc2a0ed35e89189fa2dd8c50665
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102550388"
---
# <a name="change-the-os-disk-used-by-an-azure-vm-using-powershell"></a>Изменение диска ОС виртуальной машины Azure с помощью PowerShell

Если у вас имеется виртуальная машина, но нужно заменить ее диск на резервный диск или другой диск ОС, для этого можно использовать Azure PowerShell. Нет необходимости удалять и повторно создавать виртуальную машину. Можно даже использовать управляемый диск в другой группе ресурсов, если он еще не используется.

 

Не нужно останавливать виртуальную машину или отменять ее распределение. Идентификатор ресурса управляемого диска может быть заменен идентификатором ресурса другого управляемого диска.

Убедитесь, что тип и размер виртуальной машины совместимы с диском, который необходимо подключить. Например, если диск, который вы хотите использовать, размещен в хранилище уровня "Премиум", то виртуальная машина должна поддерживать это хранилище (например, она должна относиться к серии DS). Оба диска должны также иметь одинаковый размер.
И убедитесь, что вы не используете незашифрованную виртуальную машину с зашифрованным диском ОС, это не поддерживается. Если виртуальная машина не использует шифрование дисков Azure, диск ОС, для которого выполняется переключение, не должен использовать шифрование дисков Azure.

Получите список дисков в группе ресурсов с помощью команды [Get-AzDisk](/powershell/module/az.compute/get-azdisk).

```azurepowershell-interactive
Get-AzDisk -ResourceGroupName myResourceGroup | Format-Table -Property Name
```
 
Зная имя диска, который вы хотите использовать, задайте его в качестве диска ОС для виртуальной машины. Этот пример останавливает виртуальную машину *myVM* и отменяет ее выделение, а затем назначает диск *newDisk* в качестве нового диска ОС. 
 
```azurepowershell-interactive 
# Get the VM 
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM 

# Make sure the VM is stopped\deallocated
Stop-AzVM -ResourceGroupName myResourceGroup -Name $vm.Name -Force

# Get the new disk that you want to swap in
$disk = Get-AzDisk -ResourceGroupName myResourceGroup -Name newDisk

# Set the VM configuration to point to the new disk  
Set-AzVMOSDisk -VM $vm -ManagedDiskId $disk.Id -Name $disk.Name 

# Update the VM with the new OS disk
Update-AzVM -ResourceGroupName myResourceGroup -VM $vm 

# Start the VM
Start-AzVM -Name $vm.Name -ResourceGroupName myResourceGroup

```

**Дальнейшие действия**

Создание копии диска описывается в разделе [Моментальный снимок диска](snapshot-copy-managed-disk.md).
