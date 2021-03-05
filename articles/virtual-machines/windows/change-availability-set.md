---
title: Изменение группы доступности виртуальных машин
description: Узнайте, как изменить группу доступности для виртуальной машины с помощью Azure PowerShell.
ms.service: virtual-machines
author: cynthn
ms.topic: how-to
ms.date: 01/31/2020
ms.author: cynthn
ms.openlocfilehash: 65bf2dea6f3bc2e33ec10dc75b1678466401c10b
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102184480"
---
# <a name="change-the-availability-set-for-a-vm"></a>Изменение группы доступности для виртуальной машины
Ниже приведены инструкции по изменению группы доступности виртуальной машины с помощью Azure PowerShell. Виртуальную машину можно добавить в группу доступности только при ее создании. Чтобы изменить группу доступности, необходимо удалить и повторно создать виртуальную машину. 

Эта статья относится к виртуальным машинам Linux и Windows.

Последнее тестирование этой статьи выполнялось 12.02.2019 г. с помощью [Azure Cloud Shell](https://shell.azure.com/powershell) и [модуля Az PowerShell](/powershell/azure/install-az-ps) версии 1.2.0.

В этом примере не проверяется, подключена ли виртуальная машина к подсистеме балансировки нагрузки. Если виртуальная машина подключена к подсистеме балансировки нагрузки, необходимо обновить скрипт для обработки этого случая. 


## <a name="change-the-availability-set"></a>Изменение группы доступности 

Следующий сценарий иллюстрирует сбор необходимых сведений, удаление исходной виртуальной машины и ее повторное создание в новой группе доступности.

```powershell
# Set variables
    $resourceGroup = "myResourceGroup"
    $vmName = "myVM"
    $newAvailSetName = "myAvailabilitySet"

# Get the details of the VM to be moved to the Availability Set
    $originalVM = Get-AzVM `
       -ResourceGroupName $resourceGroup `
       -Name $vmName

# Create new availability set if it does not exist
    $availSet = Get-AzAvailabilitySet `
       -ResourceGroupName $resourceGroup `
       -Name $newAvailSetName `
       -ErrorAction Ignore
    if (-Not $availSet) {
    $availSet = New-AzAvailabilitySet `
       -Location $originalVM.Location `
       -Name $newAvailSetName `
       -ResourceGroupName $resourceGroup `
       -PlatformFaultDomainCount 2 `
       -PlatformUpdateDomainCount 2 `
       -Sku Aligned
    }
    
# Remove the original VM
    Remove-AzVM -ResourceGroupName $resourceGroup -Name $vmName    

# Create the basic configuration for the replacement VM. 
    $newVM = New-AzVMConfig `
       -VMName $originalVM.Name `
       -VMSize $originalVM.HardwareProfile.VmSize `
       -AvailabilitySetId $availSet.Id
 
# For a Linux VM, change the last parameter from -Windows to -Linux 
    Set-AzVMOSDisk `
       -VM $newVM -CreateOption Attach `
       -ManagedDiskId $originalVM.StorageProfile.OsDisk.ManagedDisk.Id `
       -Name $originalVM.StorageProfile.OsDisk.Name `
       -Windows

# Add Data Disks
    foreach ($disk in $originalVM.StorageProfile.DataDisks) { 
    Add-AzVMDataDisk -VM $newVM `
       -Name $disk.Name `
       -ManagedDiskId $disk.ManagedDisk.Id `
       -Caching $disk.Caching `
       -Lun $disk.Lun `
       -DiskSizeInGB $disk.DiskSizeGB `
       -CreateOption Attach
    }
    
# Add NIC(s) and keep the same NIC as primary; keep the Private IP too, if it exists. 
    foreach ($nic in $originalVM.NetworkProfile.NetworkInterfaces) {    
    if ($nic.Primary -eq "True")
    {
            Add-AzVMNetworkInterface `
               -VM $newVM `
               -Id $nic.Id -Primary
               }
           else
               {
                 Add-AzVMNetworkInterface `
                -VM $newVM `
                 -Id $nic.Id 
                }
      }

# Recreate the VM
    New-AzVM `
       -ResourceGroupName $resourceGroup `
       -Location $originalVM.Location `
       -VM $newVM `
       -DisableBginfoExtension
```

## <a name="next-steps"></a>Дальнейшие действия

Увеличьте емкость хранилища для виртуальной машины, добавив дополнительный [диск данных](attach-managed-disk-portal.md).
