---
title: Включение общих дисков для управляемых дисков Azure
description: Настройка управляемого диска Azure с общими дисками для совместного использования на нескольких виртуальных машинах
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 09/30/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: e8bb97196fcceea0c86f58fe4a63146e681c427e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96500739"
---
# <a name="enable-shared-disk"></a>Включить общий диск

В этой статье описывается, как включить функцию "Общие диски" для управляемых дисков Azure. Общие диски Azure — это новая функция для управляемых дисков Azure, которая позволяет подключать управляемые диски одновременно к нескольким виртуальным машинам. Подключение управляемого диска к нескольким виртуальным машинам позволяет разворачивать новые или переносить существующие кластеризованные приложения в Azure. 

Если вы ищете основные сведения об управляемых дисках с включенными общими дисками, см. статью [Общие диски Azure](disks-shared.md).

## <a name="limitations"></a>Ограничения

[!INCLUDE [virtual-machines-disks-shared-limitations](../../includes/virtual-machines-disks-shared-limitations.md)]

## <a name="supported-operating-systems"></a>Поддерживаемые операционные системы

Общие диски поддерживают несколько операционных систем. Поддерживаемые операционные системы см. в разделах, посвященных [ОС Windows](./disks-shared.md#windows) и [Linux](./disks-shared.md#linux) .

## <a name="disk-sizes"></a>Размеры диска

[!INCLUDE [virtual-machines-disks-shared-sizes](../../includes/virtual-machines-disks-shared-sizes.md)]

## <a name="deploy-shared-disks"></a>Развертывание общих дисков

### <a name="deploy-a-premium-ssd-as-a-shared-disk"></a>Развертывание SSD уровня "Премиум" как общего диска

Чтобы развернуть управляемый диск с включенной функцией "общий диск", используйте новое свойство `maxShares` и определите значение больше 1. Это сделает диск общим для нескольких виртуальных машин.

> [!IMPORTANT]
> Значение параметра `maxShares` может быть задано или изменено только при отключении диска от всех виртуальных машин. Допустимые значения для см. в разделе [размеры дисков](#disk-sizes) `maxShares` .

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az disk create -g myResourceGroup -n mySharedDisk --size-gb 1024 -l westcentralus --sku Premium_LRS --max-shares 2
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$dataDiskConfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType Premium_LRS -CreateOption Empty -MaxSharesCount 2

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $dataDiskConfig
```

# <a name="resource-manager-template"></a>[Шаблон Resource Manager](#tab/azure-resource-manager)

Перед использованием следующего шаблона замените,, `[parameters('dataDiskName')]` `[resourceGroup().location]` `[parameters('dataDiskSizeGB')]` и `[parameters('maxShares')]` собственными значениями.

[Шаблон общего диска SSD (цен. категория "Премиум")](https://aka.ms/SharedPremiumDiskARMtemplate)

---

### <a name="deploy-an-ultra-disk-as-a-shared-disk"></a>Развертывание Ultra Disk в качестве общего диска

Чтобы развернуть управляемый диск с включенным компонентом "общий диск", измените `maxShares` параметр на значение больше 1. Это сделает диск общим для нескольких виртуальных машин.

> [!IMPORTANT]
> Значение параметра `maxShares` может быть задано или изменено только при отключении диска от всех виртуальных машин. Допустимые значения для см. в разделе [размеры дисков](#disk-sizes) `maxShares` .


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

##### <a name="regional-disk-example"></a>Пример для регионального диска

```azurecli
#Creating an Ultra shared Disk 
az disk create -g rg1 -n clidisk --size-gb 1024 -l westus --sku UltraSSD_LRS --max-shares 5 --disk-iops-read-write 2000 --disk-mbps-read-write 200 --disk-iops-read-only 100 --disk-mbps-read-only 1

#Updating an Ultra shared Disk 
az disk update -g rg1 -n clidisk --disk-iops-read-write 3000 --disk-mbps-read-write 300 --set diskIopsReadOnly=100 --set diskMbpsReadOnly=1

#Show shared disk properties:
az disk show -g rg1 -n clidisk
```

##### <a name="zonal-disk-example"></a>Пример диска зональные

Этот пример практически аналогичен предыдущему, за исключением того, что он создает диск в зоне доступности 1.

```azurecli
#Creating an Ultra shared Disk 
az disk create -g rg1 -n clidisk --size-gb 1024 -l westus --sku UltraSSD_LRS --max-shares 5 --disk-iops-read-write 2000 --disk-mbps-read-write 200 --disk-iops-read-only 100 --disk-mbps-read-only 1 --zone 1

#Updating an Ultra shared Disk 
az disk update -g rg1 -n clidisk --disk-iops-read-write 3000 --disk-mbps-read-write 300 --set diskIopsReadOnly=100 --set diskMbpsReadOnly=1

#Show shared disk properties:
az disk show -g rg1 -n clidisk
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

##### <a name="regional-disk-example"></a>Пример для регионального диска

```azurepowershell-interactive
$datadiskconfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType UltraSSD_LRS -CreateOption Empty -DiskIOPSReadWrite 2000 -DiskMBpsReadWrite 200 -DiskIOPSReadOnly 100 -DiskMBpsReadOnly 1 -MaxSharesCount 5

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

##### <a name="zonal-disk-example"></a>Пример диска зональные

Этот пример практически аналогичен предыдущему, за исключением того, что он создает диск в зоне доступности 1.

```azurepowershell-interactive
$datadiskconfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType UltraSSD_LRS -CreateOption Empty -DiskIOPSReadWrite 2000 -DiskMBpsReadWrite 200 -DiskIOPSReadOnly 100 -DiskMBpsReadOnly 1 -MaxSharesCount 5 -Zone 1

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

# <a name="resource-manager-template"></a>[Шаблон Resource Manager](#tab/azure-resource-manager)

##### <a name="regional-disk-example"></a>Пример для регионального диска

Перед использованием следующего шаблона замените,, `[parameters('dataDiskName')]` `[resourceGroup().location]` `[parameters('dataDiskSizeGB')]` , `[parameters('maxShares')]` , `[parameters('diskIOPSReadWrite')]` , `[parameters('diskMBpsReadWrite')]` , `[parameters('diskIOPSReadOnly')]` и `[parameters('diskMBpsReadOnly')]` собственными значениями.

[Шаблон региональных общих Ultra дисков](https://aka.ms/SharedUltraDiskARMtemplateRegional)

##### <a name="zonal-disk-example"></a>Пример диска зональные

Перед использованием следующего шаблона замените,, `[parameters('dataDiskName')]` `[resourceGroup().location]` `[parameters('dataDiskSizeGB')]` , `[parameters('maxShares')]` , `[parameters('diskIOPSReadWrite')]` , `[parameters('diskMBpsReadWrite')]` , `[parameters('diskIOPSReadOnly')]` и `[parameters('diskMBpsReadOnly')]` собственными значениями.

[Шаблон общих Ultra дисков зональные](https://aka.ms/SharedUltraDiskARMtemplateZonal)

---

## <a name="using-azure-shared-disks-with-your-vms"></a>Использование общих дисков Azure с виртуальными машинами

После развертывания общего диска с помощью его `maxShares>1` можно подключить к одной или нескольким виртуальным машинам.

> [!NOTE]
> При развертывании Ultra Disk убедитесь, что он соответствует необходимым требованиям. Дополнительные сведения см. [в статье Использование Ultra Disks для Azure](disks-enable-ultra-ssd.md) .

```azurepowershell-interactive

$resourceGroup = "myResourceGroup"
$location = "WestCentralUS"

$vm = New-AzVm -ResourceGroupName $resourceGroup -Name "myVM" -Location $location -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress"

$dataDisk = Get-AzDisk -ResourceGroupName $resourceGroup -DiskName "mySharedDisk"

$vm = Add-AzVMDataDisk -VM $vm -Name "mySharedDisk" -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0

update-AzVm -VM $vm -ResourceGroupName $resourceGroup
```

## <a name="supported-scsi-pr-commands"></a>Поддерживаемые команды для запросов на SCSI

После подключения общего диска к виртуальным машинам в кластере можно установить кворум и выполнить чтение и запись на диск с помощью SCSI-запроса. При использовании общих дисков Azure доступны следующие команды.

Чтобы взаимодействовать с диском, начните со списка "постоянное — резервирование — действие":

```
PR_REGISTER_KEY 

PR_REGISTER_AND_IGNORE 

PR_GET_CONFIGURATION 

PR_RESERVE 

PR_PREEMPT_RESERVATION 

PR_CLEAR_RESERVATION 

PR_RELEASE_RESERVATION 
```

При использовании PR_RESERVE, PR_PREEMPT_RESERVATION или PR_RELEASE_RESERVATION укажите один из следующих типов постоянного резервирования:

```
PR_NONE 

PR_WRITE_EXCLUSIVE 

PR_EXCLUSIVE_ACCESS 

PR_WRITE_EXCLUSIVE_REGISTRANTS_ONLY 

PR_EXCLUSIVE_ACCESS_REGISTRANTS_ONLY 

PR_WRITE_EXCLUSIVE_ALL_REGISTRANTS 

PR_EXCLUSIVE_ACCESS_ALL_REGISTRANTS 
```

Кроме того, при использовании PR_RESERVE, PR_REGISTER_AND_IGNORE, PR_REGISTER_KEY, PR_PREEMPT_RESERVATION, PR_CLEAR_RESERVATION или PR_RELEASE-RESERVATION необходимо указать постоянный ключ-резервирование.


## <a name="next-steps"></a>Дальнейшие действия

Если вы предпочитаете использовать шаблоны Azure Resource Manager для развертывания диска, доступны следующие примеры шаблонов:
- [Диск SSD (цен. категория "Премиум")](https://aka.ms/SharedPremiumDiskARMtemplate)
- [Региональные диски Ultra](https://aka.ms/SharedUltraDiskARMtemplateRegional)
- [Зональные Ultra Disks](https://aka.ms/SharedUltraDiskARMtemplateZonal)