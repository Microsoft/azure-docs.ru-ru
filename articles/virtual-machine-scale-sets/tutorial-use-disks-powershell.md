---
title: Руководство. Создание и использование дисков для масштабируемых наборов с помощью Azure PowerShell
description: Узнайте, как с помощью Azure PowerShell создавать и использовать Управляемые диски с масштабируемыми наборами виртуальных машин, в том числе добавлять, подготавливать, перечислять и отсоединять диски.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 9e995e88b80bf14f9c7784f465bcd3d89d0bed65
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92367964"
---
# <a name="tutorial-create-and-use-disks-with-virtual-machine-scale-set-with-azure-powershell"></a>Руководство. Создание и использование дисков с масштабируемым набором виртуальных машин с помощью Azure PowerShell

В масштабируемых наборах виртуальных машин на дисках хранится операционная система, приложения и данные. Для создания масштабируемого набора и управления им важно выбрать размер диска и конфигурацию в соответствии с ожидаемой рабочей нагрузкой. В этом руководстве объясняется, как создать диски виртуальной машины и управлять ими. Из этого руководства вы узнаете, как выполнить следующие задачи:

> [!div class="checklist"]
> * дисках ОС и временных дисках;
> * Диски данных
> * дисками уровня "Стандартный" и "Премиум";
> * производительностью дисков;
> * подключение и подготовка дисков данных.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="default-azure-disks"></a>Диски Azure по умолчанию
При создании или масштабировании масштабируемого набора к каждому экземпляру виртуальной машины автоматически подключаются два диска. 

**Диск операционной системы**. Размер дисков операционной системы может составлять до 2 ТБ. Это диски, содержащие операционную систему экземпляра виртуальной машины. По умолчанию диск ОС помечается как */dev/sda*. Конфигурация кэширования диска ОС оптимизирована для производительности операционной системы. Ввиду этой конфигурации на дисках ОС **не должны** размещаться приложения или данные. Для приложений и данных используйте диски данных, которые описаны далее в этой статье. 

**Временный диск**. Временные диски используют твердотельные накопители, расположенные на том же узле Azure, что и экземпляр виртуальной машины. Эти диски обладают высокой производительностью и могут быть использованы для таких операций, как обработка временных данных. Но если экземпляр виртуальной машины перемещается на новый узел, удаляются все данные, хранящиеся на временном диске. Размер временного диска определяется размером экземпляра виртуальной машины. Временные диски помечаются как */dev/sdb* и используют точку подключения */mnt*.

### <a name="temporary-disk-sizes"></a>Размеры временных дисков
| Тип | Распространенные размеры | Максимальный размер временного диска (ГБ) |
|----|----|----|
| [Универсальные](../virtual-machines/sizes-general.md) | Серии A, B и D | 1600 |
| [Оптимизированные для вычислений](../virtual-machines/sizes-compute.md) | Серия F | 576 |
| [Оптимизированные для памяти](../virtual-machines/sizes-memory.md) | Серии D, E, G и M | 6144 |
| [Оптимизированные для хранилища](../virtual-machines/sizes-storage.md) | Серия L | 5630 |
| [GPU](../virtual-machines/sizes-gpu.md) | Серия N | 1440 |
| [Высокопроизводительные](../virtual-machines/sizes-hpc.md) | Серии A и H | 2000 |


## <a name="azure-data-disks"></a>Диски данных Azure
Вы можете добавить дополнительные диски данных, чтобы установить приложения и хранить данные. Диски данных следует использовать в любой ситуации, где требуется надежное хранилище данных, обеспечивающее высокую скорость реагирования. Максимальная емкость каждого диска составляет 4 ТБ. От размера экземпляра виртуальной машины зависит, сколько дисков данных можно к ней подключить. Для каждого виртуального ЦП виртуальной машины можно подключить два диска данных.

### <a name="max-data-disks-per-vm"></a>Максимальное число дисков данных на виртуальную машину
| Тип | Распространенные размеры | Максимальное число дисков данных на виртуальную машину |
|----|----|----|
| [Универсальные](../virtual-machines/sizes-general.md) | Серии A, B и D | 64 |
| [Оптимизированные для вычислений](../virtual-machines/sizes-compute.md) | Серия F | 64 |
| [Оптимизированные для памяти](../virtual-machines/sizes-memory.md) | Серии D, E, G и M | 64 |
| [Оптимизированные для хранилища](../virtual-machines/sizes-storage.md) | Серия L | 64 |
| [GPU](../virtual-machines/sizes-gpu.md) | Серия N | 64 |
| [Высокопроизводительные](../virtual-machines/sizes-hpc.md) | Серии A и H | 64 |


## <a name="vm-disk-types"></a>Типы дисков виртуальной машины
В Azure предоставляются диски двух типов.

### <a name="standard-disk"></a>Диск уровня "Стандартный"
Служба хранилища ценовой категории "Стандартный" использует HDD-диски и обеспечивает экономичное хранилище с достаточной производительностью. Эти диски идеально подходят для экономных рабочих нагрузок разработки и тестирования.

### <a name="premium-disk"></a>Диск уровня "Премиум"
Для хранилища класса Premium используются высокопроизводительные твердотельные накопители с низкой задержкой. Эти диски рекомендуется использовать на виртуальных машинах, на которых выполняются рабочие нагрузки рабочей среды. Хранилище уровня "Премиум" поддерживает виртуальные машины серий DS, DSv2, GS и FS. При выборе размер диска округляется в большую сторону до следующего типа. Например, если размер диска составляет менее 128 ГБ, то типом диска является P10. Если размер диска составляет от 129 до 512 ГБ, то его размер — P20. Диски более 512 ГБ относятся к типу P30.

### <a name="premium-disk-performance"></a>Производительность диска уровня "Премиум"
|Тип диска хранилища уровня "Премиум" | P4 | P6 | P10 | P20 | P30 | P40 | P50 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Размер диска (округленный в большую сторону) | 32 ГБ | 64 ГБ | 128 ГБ | 512 ГБ | 1024 ГБ (1 ТБ) | 2048 ГБ (2 ТБ) | 4095 ГБ (4 ТБ) |
| Макс. количество операций ввода-вывода в секунду на диск | 120 | 240 | 500 | 2300 | 5 000 | 7500 | 7500 |
Пропускная способность на диск | 25 МБ/с | 50 МБ/с | 100 МБ/с | 150 МБ/с | 200 МБ/с | 250 МБ/с | 250 МБ/с |

Хотя в таблице выше указано максимальное число операций ввода-вывода в секунду на диск, можно обеспечить более высокий уровень производительности, применив чередование нескольких дисков данных. Например, виртуальная машина Standard_GS5 может достичь 80 000 операций ввода-вывода в секунду. Дополнительные сведения о максимальных количествах операций ввода-вывода в секунду для виртуальных машин см. в статье [Размеры виртуальных машин Windows в Azure](../virtual-machines/sizes.md).


## <a name="create-and-attach-disks"></a>Создание и подключение дисков
Диски можно создать и присоединить к создаваемому или существующему масштабируемому набору.

Начиная с версии API `2019-07-01`, можно задать размер диска операционной системы в масштабируемом наборе виртуальных машин с помощью свойства [storageProfile.osDisk.diskSizeGb](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosdisk). После подготовки может потребоваться расширить или повторно секционировать диск, чтобы использовать все его пространство. Дополнительные сведения см. в разделе [Расширение тома в ОС](../virtual-machines/windows/expand-os-disk.md#expand-the-volume-within-the-os).

### <a name="attach-disks-at-scale-set-creation"></a>Присоединение дисков при создании масштабируемого набора
Создайте масштабируемый набор виртуальных машин с помощью командлета [New-AzVmss](/powershell/module/az.compute/new-azvmss). Когда отобразится запрос, введите имя и пароль для экземпляров виртуальных машин. Чтобы распределить трафик между отдельными экземплярами виртуальных машин, создается еще и подсистема балансировки нагрузки. Подсистема балансировки нагрузки определяет правила передачи трафика на TCP-порт 80, а также разрешает подключение удаленного рабочего стола трафик через TCP-порт 3389 и удаленное взаимодействие PowerShell через TCP-порт 5985.

Создаются два диска с параметром `-DataDiskSizeGb`. Размер первого диска — *64* ГБ, а второго — *128* ГБ. При появлении запроса введите учетные данные администратора для экземпляров виртуальных машин в масштабируемом наборе:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic" `
  -DataDiskSizeInGb 64,128
```

Создание и настройка всех ресурсов и экземпляров виртуальных машин масштабируемого набора занимает несколько минут.

### <a name="attach-a-disk-to-existing-scale-set"></a>Присоединение диска к существующему масштабируемому набору
Вы также можете присоединить диски к существующему масштабируемому набору. Используйте созданный на предыдущем шаге масштабируемый набор, чтобы добавить другой диск, выполнив командлет [Add-AzVmssDataDisk](/powershell/module/az.compute/add-azvmssdatadisk). В следующем примере к имеющемуся масштабируемому набору подключается дополнительный диск размером *128* ГБ.

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Attach a 128 GB data disk to LUN 2
Add-AzVmssDataDisk `
  -VirtualMachineScaleSet $vmss `
  -CreateOption Empty `
  -Lun 2 `
  -DiskSizeGB 128

# Update the scale set to apply the change
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```


## <a name="prepare-the-data-disks"></a>Подготовка дисков данных
Диски, которые создаются и подключаются к экземплярам виртуальных машин в масштабируемом наборе, являются дисками в формате RAW. Прежде чем использовать диски для данных и приложений, их необходимо подготовить. Чтобы подготовить диски, создайте раздел и файловую систему и подключите их.

Чтобы автоматизировать процесс на нескольких экземплярах виртуальных машин в масштабируемом наборе, можно использовать расширение пользовательских скриптов Azure. Это расширение может выполнять скрипты локально на каждом экземпляре виртуальной машины, например, чтобы подготовить подключенные диски данных. Дополнительные сведения см. в статье [Расширение Custom Script в ОС Windows](../virtual-machines/extensions/custom-script-windows.md).


В следующем примере на каждом экземпляре виртуальной машины выполняется скрипт из примера репозитория GitHub с помощью командлета [Add-AzVmssExtension](/powershell/module/az.compute/Add-AzVmssExtension), который подготавливает все подключенные диски данных в формате RAW.


```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Define the script for your Custom Script Extension to run
$publicSettings = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File prepare_vm_disks.ps1"
}

# Use Custom Script Extension to prepare the attached data disks
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Чтобы убедиться, что диски подготовлены правильно, подключитесь с помощью RDP к одному из экземпляров виртуальных машин. 

Сначала получите объект подсистемы балансировки нагрузки с помощью командлета [Get-AzLoadBalancer](/powershell/module/az.network/Get-AzLoadBalancer). Затем просмотрите правила NAT для входящего трафика с помощью командлета [Get-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/Get-AzLoadBalancerInboundNatRuleConfig). Эти правила содержат сведения о портах *FrontendPort* каждого экземпляра виртуальной машины, которые прослушивает RDP. Наконец, получите общедоступный IP-адрес подсистемы балансировки нагрузки с помощью командлета [Get-AzPublicIpAddress](/powershell/module/az.network/Get-AzPublicIpAddress).


```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort

# View the public IP address of the load balancer
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" -Name myPublicIPAddress | Select IpAddress
```

Чтобы подключиться к виртуальной машине, укажите собственный общедоступный IP-адрес и номер порта требуемого экземпляра виртуальной машины, как показано в предыдущих командах. При появлении запроса введите учетные данные, использованные при создании масштабируемого набора. Если вы используете Azure Cloud Shell, выполните этот шаг с помощью локальной командной строки PowerShell или клиента удаленного рабочего стола. В следующем примере устанавливается подключение к экземпляру виртуальной машины *1*.

```powershell
mstsc /v 52.168.121.216:50001
```

Откройте локальный сеанс PowerShell на экземпляре виртуальной машины и просмотрите подключенные диски с помощью командлета [Get-Disk](/powershell/module/storage/get-disk).

```powershell
Get-Disk
```

В следующем примере выходных данных показано, что к экземпляру виртуальной машины подключены три диска.

```powershell
Number  Friendly Name      HealthStatus  OperationalStatus  Total Size  Partition Style
------  -------------      ------------  -----------------  ----------  ---------------
0       Virtual HD         Healthy       Online                 127 GB  MBR
1       Virtual HD         Healthy       Online                  14 GB  MBR
2       Msft Virtual Disk  Healthy       Online                  64 GB  MBR
3       Msft Virtual Disk  Healthy       Online                 128 GB  MBR
4       Msft Virtual Disk  Healthy       Online                 128 GB  MBR
```

Просмотрите файловые системы и точки подключения на экземпляре виртуальной машины, как показано ниже:

```powershell
Get-Partition
```

В следующем примере выходных данных показано, что для трех дисков данных назначены буквы.

```powershell
   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000001

PartitionNumber  DriveLetter  Offset   Size  Type
---------------  -----------  ------   ----  ----
1                F            1048576  64 GB  IFS

   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000002

PartitionNumber  DriveLetter  Offset   Size   Type
---------------  -----------  ------   ----   ----
1                G            1048576  128 GB  IFS

   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000003

PartitionNumber  DriveLetter  Offset   Size   Type
---------------  -----------  ------   ----   ----
1                H            1048576  128 GB  IFS
```

Диски на каждом экземпляре виртуальной машины в масштабируемом наборе автоматически подготавливаются таким же образом. По мере масштабирования масштабируемого набора к новым экземплярам виртуальных машин подключаются необходимые диски данных. Расширение пользовательских скриптов также запускается для автоматической подготовки дисков.

Закройте сеанс подключения к экземпляру виртуальной машины по протоколу удаленного рабочего стола.


## <a name="list-attached-disks"></a>Отображение списка подключенных дисков
Чтобы просмотреть информацию о дисках, подключенных к масштабируемому набору, выполните командлет [Get-AzVmss](/powershell/module/az.compute/get-azvmss), как показано ниже.

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet"
```

В разделе свойства *VirtualMachineProfile.StorageProfile* отображается список дисков данных (*DataDisks*). Отобразится информация о размере диска, уровне хранилища и LUN (логическом номере устройства). В следующем примере выходных данных содержатся сведения о трех дисках данных, подключенных к масштабируемому набору.

```powershell
DataDisks[0]                            :
  Lun                                   : 0
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 64
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
DataDisks[1]                            :
  Lun                                   : 1
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 128
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
DataDisks[2]                            :
  Lun                                   : 2
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 128
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
```


## <a name="detach-a-disk"></a>Отсоединение диска
Если конкретный диск больше не нужен, его можно отключить от масштабируемого набора. Диск удаляется из всех экземпляров виртуальных машин в масштабируемом наборе. Чтобы отключить диск от масштабируемого набора, используйте командлет [Remove-AzVmssDataDisk](/powershell/module/az.compute/remove-azvmssdatadisk) и укажите номер LUN диска. Эти номера отображаются в выходных данных, полученных с помощью командлета [Get-AzVmss](/powershell/module/az.compute/get-azvmss) в предыдущем разделе. В следующем примере диск с LUN *3* отключается от масштабируемого набора.

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Detach a disk from the scale set
Remove-AzVmssDataDisk `
  -VirtualMachineScaleSet $vmss `
  -Lun 2

# Update the scale set and detach the disk from the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```


## <a name="clean-up-resources"></a>Очистка ресурсов
Чтобы удалить масштабируемый набор и диски, удалите группу ресурсов и все ее ресурсы с помощью командлета [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). Параметр `-Force` подтверждает, что вы хотите удалить ресурсы без дополнительного запроса. При использовании параметра `-AsJob` управление возвращается в командную строку без ожидания завершения операции.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Дальнейшие действия
Из этого руководства вы узнали, как создавать и использовать диски с масштабируемыми наборами с помощью Azure PowerShell:

> [!div class="checklist"]
> * дисках ОС и временных дисках;
> * Диски данных
> * дисками уровня "Стандартный" и "Премиум";
> * производительностью дисков;
> * подключение и подготовка дисков данных.

Перейдите к следующему руководству, чтобы узнать, как использовать пользовательский образ для экземпляров виртуальных машин масштабируемого набора.

> [!div class="nextstepaction"]
> [Использование пользовательского образа для экземпляров виртуальных машин в масштабируемом наборе](tutorial-use-custom-image-powershell.md)