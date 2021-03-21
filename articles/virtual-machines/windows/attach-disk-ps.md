---
title: Подключение диска данных к виртуальной машине Windows в Azure с помощью PowerShell
description: Подключение нового или существующего диска данных к виртуальной машине Windows с помощью PowerShell в модели развертывания Resource Manager.
author: roygara
ms.service: virtual-machines
ms.collection: windows
ms.topic: how-to
ms.date: 10/16/2018
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: d5c638a63e4e8dc1a55c07f4bdf713fd36ca6b3d
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102550881"
---
# <a name="attach-a-data-disk-to-a-windows-vm-with-powershell"></a>Подключение диска данных к виртуальной машине Windows с помощью PowerShell

В этой статье демонстрируется подключение нового и существующего дисков к виртуальной машине Windows с помощью PowerShell. 

Во-первых, ознакомьтесь со следующими советами:

* Размер виртуальной машины определяет, сколько дисков данных к ней можно подключить. Дополнительные сведения см. в статье [размеры виртуальных машин](../sizes.md).
* Для использования хранилища категории "Премиум" (SSD) необходима [виртуальная машина соответствующего типа](../sizes-memory.md), например серии DS или GS.

В этой статье используется PowerShell в [Azure Cloud Shell](../../cloud-shell/overview.md), который постоянно обновляется до последней версии. Чтобы открыть Cloud Shell, выберите **Попробовать** в верхнем углу любого блока кода.

## <a name="add-an-empty-data-disk-to-a-virtual-machine"></a>Добавление пустого диска данных в виртуальную машину

В этом примере показано, как добавить пустой диск данных в существующую виртуальную машину.

### <a name="using-managed-disks"></a>Использование управляемых дисков

```azurepowershell-interactive
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'East US' 
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

### <a name="using-managed-disks-in-an-availability-zone"></a>Использование управляемых дисков в зоне доступности

Чтобы создать диск в зоне доступности, запустите командлет [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) с параметром `-Zone`. В следующем примере создается диск в зоне *1*.

```powershell
$rgName = 'myResourceGroup'
$vmName = 'myVM'
$location = 'East US 2'
$storageType = 'Premium_LRS'
$dataDiskName = $vmName + '_datadisk1'

$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $location -CreateOption Empty -DiskSizeGB 128 -Zone 1
$dataDisk1 = New-AzDisk -DiskName $dataDiskName -Disk $diskConfig -ResourceGroupName $rgName

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 
$vm = Add-AzVMDataDisk -VM $vm -Name $dataDiskName -CreateOption Attach -ManagedDiskId $dataDisk1.Id -Lun 1

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

### <a name="initialize-the-disk"></a>Инициализировать диск

После добавления пустого диска его необходимо инициализировать. Чтобы инициализировать этот диск, можно войти в содержащую его виртуальную машину и использовать средство управления дисками. Если при создании виртуальной машины вы установили на нее [WinRM](/windows/desktop/winrm/portal) и сертификат, то вы можете инициализировать диск удаленно с помощью PowerShell. Можно также использовать расширение пользовательского сценария.

```azurepowershell-interactive
    $location = "location-name"
    $scriptName = "script-name"
    $fileName = "script-file-name"
    Set-AzVMCustomScriptExtension -ResourceGroupName $rgName -Location $locName -VMName $vmName -Name $scriptName -TypeHandlerVersion "1.4" -StorageAccountName "mystore1" -StorageAccountKey "primary-key" -FileName $fileName -ContainerName "scripts"
```

Файл сценария может содержать код для инициализации дисков, например:

```azurepowershell-interactive
    $disks = Get-Disk | Where partitionstyle -eq 'raw' | sort number

    $letters = 70..89 | ForEach-Object { [char]$_ }
    $count = 0
    $labels = "data1","data2"

    foreach ($disk in $disks) {
        $driveLetter = $letters[$count].ToString()
        $disk | 
        Initialize-Disk -PartitionStyle MBR -PassThru |
        New-Partition -UseMaximumSize -DriveLetter $driveLetter |
        Format-Volume -FileSystem NTFS -NewFileSystemLabel $labels[$count] -Confirm:$false -Force
    $count++
    }
```

## <a name="attach-an-existing-data-disk-to-a-vm"></a>Подключение существующего диска данных к виртуальной машине

Можно подключить существующий управляемый диск к виртуальной машине как диск данных.

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$location = "East US" 
$dataDiskName = "myDisk"
$disk = Get-AzDisk -ResourceGroupName $rgName -DiskName $dataDiskName 

$vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName 

$vm = Add-AzVMDataDisk -CreateOption Attach -Lun 0 -VM $vm -ManagedDiskId $disk.Id

Update-AzVM -VM $vm -ResourceGroupName $rgName
```

## <a name="next-steps"></a>Дальнейшие действия

Вы также можете развернуть управляемые диски с помощью шаблонов. Дополнительные сведения см. в статьях [Использование управляемых дисков в](../using-managed-disks-template-deployments.md) шаблонах Azure Resource Manager или [шаблон](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-multiple-data-disk) быстрого запуска для развертывания нескольких дисков данных.