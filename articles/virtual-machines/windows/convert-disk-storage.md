---
title: Преобразование хранилища управляемых дисков между разными типами дисков с помощью Azure PowerShell
description: Как преобразовать управляемые диски Azure между различными типами дисков с помощью Azure PowerShell.
author: roygara
ms.service: virtual-machines
ms.subservice: disks
ms.topic: how-to
ms.date: 02/13/2021
ms.author: albecker
ms.openlocfilehash: e2541607b116159e4f6ec4028c83ecc9a45eded8
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102550745"
---
# <a name="update-the-storage-type-of-a-managed-disk"></a>Обновление типа хранилища управляемого диска

Существует четыре типа дисков, управляемых с Azure: диски Azure Ultra, SSD уровня "Премиум", "Стандартный SSD" и "Стандартный HDD". В зависимости от потребностей производительности можно переключаться между твердотельным накопителем уровня "Премиум", стандартным SSD и стандартным HDD. Вы еще не можете переключиться с Ultra Disk или на него. необходимо развернуть новый.

Эта функция не поддерживается для неуправляемых дисков. Но вы можете легко [преобразовать неуправляемый диск в управляемый](convert-unmanaged-to-managed-disks.md) , чтобы иметь возможность переключаться между типами дисков.



## <a name="before-you-begin"></a>Подготовка к работе

* Так как для преобразования требуется перезагрузка виртуальной машины, необходимо запланировать миграцию дискового накопителя во время работы предварительно существующего периода обслуживания.
* Если диск не управляется, сначала [преобразуйте его в управляемый диск](convert-unmanaged-to-managed-disks.md) , чтобы можно было переключаться между вариантами хранения.

## <a name="switch-all-managed-disks-of-a-vm-between-from-one-account-to-another"></a>Переключение всех управляемых дисков виртуальной машины с одной учетной записи на другую

В этом примере показано, как преобразовать все диски виртуальной машины в хранилище класса Premium. Однако, изменив переменную $storageType в этом примере, можно преобразовать диски виртуальной машины в стандартный SSD или стандартный HDD. Чтобы использовать управляемые диски уровня "Премиум", виртуальная машина должна использовать [Размер виртуальной машины](../sizes.md) , поддерживающий хранилище класса Premium. В этом примере также выполняется переключение на размер, поддерживающий хранилище уровня "Премиум".

```azurepowershell-interactive
# Name of the resource group that contains the VM
$rgName = 'yourResourceGroup'

# Name of the your virtual machine
$vmName = 'yourVM'

# Choose between Standard_LRS, StandardSDD_LRS and Premium_LRS based on your scenario
$storageType = 'Premium_LRS'

# Premium capable size
# Required only if converting storage from Standard to Premium
$size = 'Standard_DS2_v2'

# Stop and deallocate the VM before changing the size
Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force

$vm = Get-AzVM -Name $vmName -resourceGroupName $rgName

# Change the VM size to a size that supports Premium storage
# Skip this step if converting storage from Premium to Standard
$vm.HardwareProfile.VmSize = $size
Update-AzVM -VM $vm -ResourceGroupName $rgName

# Get all disks in the resource group of the VM
$vmDisks = Get-AzDisk -ResourceGroupName $rgName 

# For disks that belong to the selected VM, convert to Premium storage
foreach ($disk in $vmDisks)
{
    if ($disk.ManagedBy -eq $vm.Id)
    {
        $disk.Sku = [Microsoft.Azure.Management.Compute.Models.DiskSku]::new($storageType)
        $disk | Update-AzDisk
    }
}

Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

## <a name="switch-individual-managed-disks-between-standard-and-premium"></a>Переключение отдельных управляемых дисков между уровнями "Стандартный" и "Премиум"

Для рабочей нагрузки разработки и тестирования может потребоваться сочетание дисков уровня "Стандартный" и "Премиум" для снижения затрат. Можно выбрать обновление только тех дисков, для которых требуется более высокая производительность. В этом примере показано, как преобразовать один диск виртуальной машины из уровня "Стандартный" в хранилище "Премиум". Однако, изменив переменную $storageType в этом примере, можно преобразовать диски виртуальной машины в стандартный SSD или стандартный HDD. Чтобы использовать управляемые диски уровня "Премиум", виртуальная машина должна использовать [Размер виртуальной машины](../sizes.md) , поддерживающий хранилище класса Premium. В этом примере также показано, как переключиться на размер, поддерживающий хранилище класса Premium:

```azurepowershell-interactive

$diskName = 'yourDiskName'
# resource group that contains the managed disk
$rgName = 'yourResourceGroupName'
# Choose between Standard_LRS, StandardSSD_LRS and Premium_LRS based on your scenario
$storageType = 'Premium_LRS'
# Premium capable size 
$size = 'Standard_DS2_v2'

$disk = Get-AzDisk -DiskName $diskName -ResourceGroupName $rgName

# Get parent VM resource
$vmResource = Get-AzResource -ResourceId $disk.ManagedBy

# Stop and deallocate the VM before changing the storage type
Stop-AzVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name -Force

$vm = Get-AzVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name 

# Change the VM size to a size that supports Premium storage
# Skip this step if converting storage from Premium to Standard
$vm.HardwareProfile.VmSize = $size
Update-AzVM -VM $vm -ResourceGroupName $rgName

# Update the storage type
$disk.Sku = [Microsoft.Azure.Management.Compute.Models.DiskSku]::new($storageType)
$disk | Update-AzDisk

Start-AzVM -ResourceGroupName $vm.ResourceGroupName -Name $vm.Name
```

## <a name="switch-managed-disks-from-one-disk-type-to-another"></a>Переключение управляемых дисков с одного типа на другой

Выполните указанные ниже действия:

1. Войдите на [портал Microsoft Azure](https://portal.azure.com).
2. Выберите виртуальную машину из списка **виртуальных машин**.
3. Если виртуальная машина не остановлена, выберите **остановить** в верхней части панели **Обзор** виртуальной машины и дождитесь остановки виртуальной машины.
4. В области виртуальной машины в меню выберите **диски** .
5. Выберите диск, который требуется преобразовать.
6. В меню выберите пункт **Конфигурация** .
7. Измените **тип учетной записи** с диска исходного типа на нужный тип диска.
8. Нажмите кнопку **сохранить** и закройте область диск.

Преобразование типа диска происходит мгновенно. После преобразования можно запустить виртуальную машину.

## <a name="next-steps"></a>Дальнейшие действия

Создайте копию виртуальной машины, доступную только для чтения, с помощью [моментальных снимков](snapshot-copy-managed-disk.md).