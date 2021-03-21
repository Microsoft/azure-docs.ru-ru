---
title: Ultra Disks для виртуальных машин с управляемыми дисками Azure
description: Дополнительные сведения о Ultra Disks для виртуальных машин Azure
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 03/16/2021
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: 43dac1692dd6ee4ed1ab67a9b18ca69738e0a0f0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104580611"
---
# <a name="using-azure-ultra-disks"></a>Использование Ultra дисков Azure

В этой статье объясняется, как развернуть и использовать Ultra Disk. для получения концептуальных сведений о Ultra Disks см. раздел [какие типы дисков доступны в Azure?](disks-types.md#ultra-disk).

Высокопроизводительные диски Azure обеспечивают высокую пропускную способность, высокую скорость операций ввода-вывода и постоянную задержку на диске для виртуальных машин Azure IaaS. Это новое предложение обеспечивает первоклассную производительность с тем же уровнем доступности, что и имеющиеся предложения дисков. Одним из основных преимуществ использования Ultra Disks является возможность динамического изменения производительности SSD вместе с рабочими нагрузками без необходимости перезапуска виртуальных машин. Диски категории "Ультра" подходят для рабочих нагрузок, предполагающих интенсивную работу с данными, например SAP HANA, базы данных верхнего уровня и рабочие нагрузки с большим количеством транзакций.

## <a name="ga-scope-and-limitations"></a>Область и ограничения общедоступного уровня

[!INCLUDE [managed-disks-ultra-disks-GA-scope-and-limitations](../../includes/managed-disks-ultra-disks-GA-scope-and-limitations.md)]

## <a name="determine-vm-size-and-region-availability"></a>Определение размера виртуальной машины и доступности регионов

### <a name="vms-using-availability-zones"></a>Виртуальные машины, использующие зоны доступности

Чтобы использовать диски Ultra, необходимо определить, в какой зоне доступности вы используете. Не каждый регион поддерживает каждый размер виртуальной машины с Ultra Disks. Чтобы определить, поддерживает ли ваш регион, зону и размер виртуальной машины, выполните одну из следующих команд, обязательно замените значения **Region**, **vmSize** и **Subscription** .

#### <a name="cli"></a>CLI

```azurecli
subscription="<yourSubID>"
# example value is southeastasia
region="<yourLocation>"
# example value is Standard_E64s_v3
vmSize="<yourVMSize>"

az vm list-skus --resource-type virtualMachines  --location $region --query "[?name=='$vmSize'].locationInfo[0].zoneDetails[0].Name" --subscription $subscription
```

#### <a name="powershell"></a>PowerShell

```powershell
$region = "southeastasia"
$vmSize = "Standard_E64s_v3"
$sku = (Get-AzComputeResourceSku | where {$_.Locations.Contains($region) -and ($_.Name -eq $vmSize) -and $_.LocationInfo[0].ZoneDetails.Count -gt 0})
if($sku){$sku[0].LocationInfo[0].ZoneDetails} Else {Write-host "$vmSize is not supported with Ultra Disk in $region region"}
```

Ответ будет похож на приведенный ниже формат, где X — это зона, используемая для развертывания в выбранном регионе. Значением X может иметь 1, 2 или 3.

Сохраните значение **зон** , оно представляет вашу зону доступности и потребуется для развертывания Ultra Disk.

|ResourceType  |Имя  |Расположение  |Зоны  |Ограничение  |Функция  |Значение  |
|---------|---------|---------|---------|---------|---------|---------|
|disks     |UltraSSD_LRS         |eastus2         |X         |         |         |         |

> [!NOTE]
> Если команда не ответила, выбранный размер виртуальной машины не поддерживается для Ultra Disks в выбранном регионе.

Теперь, когда вы узнали, какая зона будет развертываться, выполните действия по развертыванию, описанные в этой статье, чтобы развернуть виртуальную машину с подключенным Ultra Disk или подключить к существующей виртуальной машине Ultra Disk.

### <a name="vms-with-no-redundancy-options"></a>Виртуальные машины без параметров избыточности

В настоящее же отсутствие параметров избыточности, развернутых в выбранных регионах, должно быть развернуто. Однако в этом регионе могут находиться не все диски, поддерживающие Ultra Disks. Чтобы определить, какие размеры дисков поддерживают Ultra, можно использовать один из следующих фрагментов кода. Обязательно замените `vmSize` `subscription` значения и первыми:

```azurecli
subscription="<yourSubID>"
region="westus"
# example value is Standard_E64s_v3
vmSize="<yourVMSize>"

az vm list-skus --resource-type virtualMachines  --location $region --query "[?name=='$vmSize'].capabilities" --subscription $subscription
```

```azurepowershell
$region = "westus"
$vmSize = "Standard_E64s_v3"
(Get-AzComputeResourceSku | where {$_.Locations.Contains($region) -and ($_.Name -eq $vmSize) })[0].Capabilities
```

Ответ будет похож на следующий вид: `UltraSSDAvailable   True` указывает, поддерживает ли размер виртуальной машины Ultra Disks в этом регионе.

```
Name                                         Value
----                                         -----
MaxResourceVolumeMB                          884736
OSVhdSizeMB                                  1047552
vCPUs                                        64
HyperVGenerations                            V1,V2
MemoryGB                                     432
MaxDataDiskCount                             32
LowPriorityCapable                           True
PremiumIO                                    True
VMDeploymentTypes                            IaaS
vCPUsAvailable                               64
ACUs                                         160
vCPUsPerCore                                 2
CombinedTempDiskAndCachedIOPS                128000
CombinedTempDiskAndCachedReadBytesPerSecond  1073741824
CombinedTempDiskAndCachedWriteBytesPerSecond 1073741824
CachedDiskBytes                              1717986918400
UncachedDiskIOPS                             80000
UncachedDiskBytesPerSecond                   1258291200
EphemeralOSDiskSupported                     True
AcceleratedNetworkingEnabled                 True
RdmaEnabled                                  False
MaxNetworkInterfaces                         8
UltraSSDAvailable                            True
```

## <a name="deploy-an-ultra-disk-using-azure-resource-manager"></a>Развертывание Ultra Disk с помощью Azure Resource Manager

Сначала определите размер виртуальной машины для развертывания. Список поддерживаемых размеров виртуальных машин см. в разделе общедоступная [область и ограничения](#ga-scope-and-limitations) .

Если вы хотите создать виртуальную машину с несколькими Ultra-дисками, ознакомьтесь с примером [Создание виртуальной машины с несколькими Ultra дисками](https://aka.ms/ultradiskArmTemplate).

Если вы планируете использовать собственный шаблон, убедитесь, что **apiVersion** для `Microsoft.Compute/virtualMachines` и `Microsoft.Compute/Disks` задан как `2018-06-01` (или более поздней версии).

Задайте для номера SKU диска значение **UltraSSD_LRS**, а затем установите емкость диска, число операций ввода-вывода, зону доступности и пропускную способность в Мбит/с для создания диска Ultra.

После подготовки виртуальной машины можно будет секционировать и отформатировать ее диски данных, а затем настроить их для рабочих нагрузок.


## <a name="deploy-an-ultra-disk"></a>Развертывание Ultra Disk

# <a name="portal"></a>[Портал](#tab/azure-portal)

В этом разделе описывается развертывание виртуальной машины, оснащенной диском Ultra, в качестве диска данных. Предполагается, что вы знакомы с развертыванием виртуальной машины. Если этого не сделать, ознакомьтесь с нашим [кратким руководством по созданию виртуальной машины Windows на портал Azure](./windows/quick-create-portal.md).

1. Войдите в [портал Azure](https://portal.azure.com/) и перейдите к разделу развертывание виртуальной машины (ВМ).
1. Обязательно выберите [поддерживаемый размер и регион виртуальной машины](#ga-scope-and-limitations).
1. Выберите **зону доступности** в области **параметры доступности**.
1. Заполните оставшиеся записи, выбрав нужный вариант.
1. Выберите **Диски**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-create.png" alt-text="Снимок экрана: последовательность создания виртуальной машины, колонка &quot;основные&quot;." lightbox="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-create.png":::

1. В колонке диски выберите **Да** , чтобы **включить совместимость с Ultra Disk**.
1. Выберите **создать и подключите новый диск** , чтобы подключить к нему диск.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-vm-disk-enable.png" alt-text="Снимок экрана: поток создания виртуальной машины, колонка диска, Ultra включен, создание и подключение нового диска выделено." :::

1. В колонке **Создание нового диска** введите имя, а затем выберите **изменить размер**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-create-disk.png" alt-text="Снимок экрана: создание колонки нового диска с выделенным изменением размера.":::


1. Измените **номер SKU диска** на **Ultra Disk**.
1. Измените значения параметра **Пользовательский размер диска (гиб)**, **дисковые операции ввода-вывода** и **пропускную способность диска** на выбранные.
1. Нажмите кнопку **ОК** в обоих колонках.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-select-ultra-disk-size.png" alt-text="Снимок экрана: колонка &quot;Выбор размера диска&quot;, выбран параметр &quot;Ultra Disk&quot; для типа хранилища, другие выделенные значения.":::

1. Продолжайте развертывание виртуальной машины, оно будет таким же, как и при развертывании любой другой виртуальной машины.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Сначала определите размер виртуальной машины для развертывания. Список поддерживаемых размеров виртуальных машин см. в разделе общедоступная [область и ограничения](#ga-scope-and-limitations) .

Для подключения Ultra Disk необходимо создать виртуальную машину, которая может использовать диски Ultra.

Замените или установите **$vmname**, **$rgname**, **$diskname**, **$Location**, **$Password**, **$User** переменные собственными значениями. Задайте **$Zone**  в качестве значения вашей зоны доступности, полученной в [начале этой статьи](#determine-vm-size-and-region-availability). Затем выполните следующую команду CLI, чтобы создать виртуальную машину с Ultra Enabled.

```azurecli-interactive
az disk create --subscription $subscription -n $diskname -g $rgname --size-gb 1024 --location $location --sku UltraSSD_LRS --disk-iops-read-write 8192 --disk-mbps-read-write 400
az vm create --subscription $subscription -n $vmname -g $rgname --image Win2016Datacenter --ultra-ssd-enabled true --zone $zone --authentication-type password --admin-password $password --admin-username $user --size Standard_D4s_v3 --location $location --attach-data-disks $diskname
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Сначала определите размер виртуальной машины для развертывания. Список поддерживаемых размеров виртуальных машин см. в разделе общедоступная [область и ограничения](#ga-scope-and-limitations) .

Чтобы использовать диски Ultra, необходимо создать виртуальную машину, которая может использовать диски Ultra. Замените или задайте переменные **$resourcegroup** и **$vmName** собственными значениями. Задайте **$Zone** в качестве значения вашей зоны доступности, полученной в [начале этой статьи](#determine-vm-size-and-region-availability). Затем выполните следующую команду [New-AzVm](/powershell/module/az.compute/new-azvm) , чтобы создать виртуальную машину с Ultra Enabled.

```powershell
New-AzVm `
    -ResourceGroupName $resourcegroup `
    -Name $vmName `
    -Location "eastus2" `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -size "Standard_D4s_v3" `
    -zone $zone
```

### <a name="create-and-attach-the-disk"></a>Создание и присоединение диска

После развертывания виртуальной машины можно создать и подключить к ней Ultra Disk, используя следующий скрипт:

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

---

## <a name="deploy-an-ultra-disk---512-byte-sector-size"></a>Развертывание Ultra Disk-512 байтового сектора

# <a name="portal"></a>[Портал](#tab/azure-portal)

1. Войдите в [портал Azure](https://portal.azure.com/), а затем найдите и выберите **диски**.
1. Выберите **+ создать** , чтобы создать новый диск.
1. Выберите регион, который поддерживает Ultra Disks и выберите зону доступности, заполните остальные значения по своему желанию.
1. Выберите **Изменить размер**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/create-managed-disk-basics-workflow.png" alt-text="Снимок экрана: колонка создания диска, регион, зона доступности и выделенный размер изменения.":::

1. В поле **номер SKU диска** выберите значение **Ultra Disk**, а затем укажите нужные значения производительности и нажмите кнопку **ОК**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-disk-size-ultra.png" alt-text="Снимок экрана создания Ultra Disk.":::

1. В колонке **Основные сведения** перейдите на вкладку **Дополнительно** .
1. Выберите **512** для **размера логического сектора**, а затем щелкните **проверить и создать**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-different-sector-size-ultra.png" alt-text="Снимок экрана выбора для изменения размера логического сектора Ultra Disk до 512.":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Сначала определите размер виртуальной машины для развертывания. Список поддерживаемых размеров виртуальных машин см. в разделе общедоступная [область и ограничения](#ga-scope-and-limitations) .

Для подключения Ultra Disk необходимо создать виртуальную машину, которая может использовать диски Ultra.

Замените или установите **$vmname**, **$rgname**, **$diskname**, **$Location**, **$Password**, **$User** переменные собственными значениями. Задайте **$Zone**  в качестве значения вашей зоны доступности, полученной в [начале этой статьи](#determine-vm-size-and-region-availability). Затем выполните следующую команду CLI, чтобы создать виртуальную машину с Ultra Disk размером 512 байтового сектора:

```azurecli
#create an ultra disk with 512 sector size
az disk create --subscription $subscription -n $diskname -g $rgname --size-gb 1024 --location $location --sku UltraSSD_LRS --disk-iops-read-write 8192 --disk-mbps-read-write 400 --logical-sector-size 512
az vm create --subscription $subscription -n $vmname -g $rgname --image Win2016Datacenter --ultra-ssd-enabled true --zone $zone --authentication-type password --admin-password $password --admin-username $user --size Standard_D4s_v3 --location $location --attach-data-disks $diskname
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Сначала определите размер виртуальной машины для развертывания. Список поддерживаемых размеров виртуальных машин см. в разделе общедоступная [область и ограничения](#ga-scope-and-limitations) .

Чтобы использовать диски Ultra, необходимо создать виртуальную машину, которая может использовать диски Ultra. Замените или задайте переменные **$resourcegroup** и **$vmName** собственными значениями. Задайте **$Zone** в качестве значения вашей зоны доступности, полученной в [начале этой статьи](#determine-vm-size-and-region-availability). Затем выполните следующую команду [New-AzVm](/powershell/module/az.compute/new-azvm) , чтобы создать виртуальную машину с Ultra Enabled.

```powershell
New-AzVm `
    -ResourceGroupName $resourcegroup `
    -Name $vmName `
    -Location "eastus2" `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -size "Standard_D4s_v3" `
    -zone $zone
```

Чтобы создать и подключить Ultra Disk с размером 512 байтового сектора, можно использовать следующий скрипт:

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-LogicalSectorSize 512 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```
---
## <a name="attach-an-ultra-disk"></a>Подключение к Ultra Disk

# <a name="portal"></a>[Портал](#tab/azure-portal)

Кроме того, если существующая виртуальная машина находится в зоне региона или доступности, которая поддерживает использование Ultra Disks, можно использовать Ultra Disks без необходимости создавать новую виртуальную машину. Включив диски Ultra на существующей виртуальной машине, присоединив их в качестве дисков данных. Чтобы включить совместимость с Ultra Disk, необходимо отключить виртуальную машину. После того как вы завершите работу виртуальной машины, вы можете включить совместимость, а затем перезапустить виртуальную машину. После включения совместимости можно подключить Ultra Disk:

1. Перейдите к виртуальной машине и прервите ее, дождитесь ее освобождения.
1. После освобождения виртуальной машины выберите **диски**.
1. выберите **Дополнительные параметры**;

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-ultra-disk-additional-settings.png" alt-text="Снимок экрана: колонка &quot;диск&quot;, дополнительные параметры выделены.":::

1. Выберите **Да** , чтобы **включить совместимость с Ultra Disk**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/enable-ultra-disks-existing-vm.png" alt-text="Снимок экрана: включение обеспечения совместимости с Ultra Disk.":::

1. Щелкните **Сохранить**.
1. Выберите **создать и подключите новый диск** и введите имя нового диска.
1. В качестве **типа хранилища** выберите **Ultra Disk**.
1. Измените значения **размера (гиб)**, **максимального числа операций ввода-вывода в секунду** и **максимальной пропускной способности** на выбранные.
1. После возврата в колонку диска нажмите кнопку **сохранить**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/new-create-ultra-disk-existing-vm.png" alt-text="Снимок экрана колонки диска с добавлением нового Ultra Disk.":::

1. Снова запустите виртуальную машину.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Кроме того, если существующая виртуальная машина находится в зоне региона или доступности, которая поддерживает использование Ultra Disks, можно использовать Ultra Disks без необходимости создавать новую виртуальную машину.

### <a name="enable-ultra-disk-compatibility-on-an-existing-vm---cli"></a>Включение обеспечения совместимости с Ultra Disk на существующей виртуальной машине — CLI

Если ваша виртуальная машина соответствует требованиям, изложенным в области общедоступного [уровня и ограничениях](#ga-scope-and-limitations) , и находится в [соответствующей зоне для вашей учетной записи](#determine-vm-size-and-region-availability), можно включить поддержку Ultra Disk на виртуальной машине.

Чтобы включить совместимость с Ultra Disk, необходимо отключить виртуальную машину. После того как вы завершите работу виртуальной машины, вы можете включить совместимость, а затем перезапустить виртуальную машину. После включения совместимости можно подключить Ultra Disk:

```azurecli
az vm deallocate -n $vmName -g $rgName
az vm update -n $vmName -g $rgName --ultra-ssd-enabled true
az vm start -n $vmName -g $rgName
```

### <a name="create-an-ultra-disk---cli"></a>Создание Ultra Disk-CLI

Теперь, когда у вас есть виртуальная машина, которая поддерживает подключение Ultra Disks, можно создать и подключить к ней Ultra Disk.

```azurecli-interactive
location="eastus2"
subscription="xxx"
rgname="ultraRG"
diskname="ssd1"
vmname="ultravm1"
zone=123

#create an ultra disk
az disk create `
--subscription $subscription `
-n $diskname `
-g $rgname `
--size-gb 4 `
--location $location `
--zone $zone `
--sku UltraSSD_LRS `
--disk-iops-read-write 1000 `
--disk-mbps-read-write 50
```

### <a name="attach-the-disk---cli"></a>Подключение диска-CLI

```azurecli
rgName="<yourResourceGroupName>"
vmName="<yourVMName>"
diskName="<yourDiskName>"
subscriptionId="<yourSubscriptionID>"

az vm disk attach -g $rgName --vm-name $vmName --disk $diskName --subscription $subscriptionId
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Кроме того, если существующая виртуальная машина находится в зоне региона или доступности, которая поддерживает использование Ultra Disks, можно использовать Ultra Disks без необходимости создавать новую виртуальную машину.

### <a name="enable-ultra-disk-compatibility-on-an-existing-vm---powershell"></a>Включение обеспечения совместимости с Ultra Disk на существующей виртуальной машине с помощью PowerShell

Если ваша виртуальная машина соответствует требованиям, изложенным в области общедоступного [уровня и ограничениях](#ga-scope-and-limitations) , и находится в [соответствующей зоне для вашей учетной записи](#determine-vm-size-and-region-availability), можно включить поддержку Ultra Disk на виртуальной машине.

Чтобы включить совместимость с Ultra Disk, необходимо отключить виртуальную машину. После того как вы завершите работу виртуальной машины, вы можете включить совместимость, а затем перезапустить виртуальную машину. После включения совместимости можно подключить Ultra Disk:

```azurepowershell
#Stop the VM
Stop-AzVM -Name $vmName -ResourceGroupName $rgName
#Enable ultra disk compatibility
$vm1 = Get-AzVM -name $vmName -ResourceGroupName $rgName
Update-AzVM -ResourceGroupName $rgName -VM $vm1 -UltraSSDEnabled $True
#Start the VM
Start-AzVM -Name $vmName -ResourceGroupName $rgName
```

### <a name="create-and-attach-an-ultra-disk---powershell"></a>Создание и подключение Ultra Disk-PowerShell

Теперь, когда у вас есть виртуальная машина, способная использовать диски Ultra, вы можете создать и подключить к ней Ultra Disk:

```powershell
# Set parameters and select subscription
$subscription = "<yourSubscriptionID>"
$resourceGroup = "<yourResourceGroup>"
$vmName = "<yourVMName>"
$diskName = "<yourDiskName>"
$lun = 1
Connect-AzAccount -SubscriptionId $subscription

# Create the disk
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName $diskName `
-Disk $diskconfig;

# add disk to VM
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $vmName
$disk = Get-AzDisk -ResourceGroupName $resourceGroup -Name $diskName
$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Attach -ManagedDiskId $disk.Id -Lun $lun
Update-AzVM -VM $vm -ResourceGroupName $resourceGroup
```

---
## <a name="adjust-the-performance-of-an-ultra-disk"></a>Настройка производительности Ultra Disk

# <a name="portal"></a>[Портал](#tab/azure-portal)

Ultra Disks предлагает уникальную возможность, позволяющую настраивать их производительность. Эти корректировки можно выполнить из портал Azure на самих дисках.

1. Перейдите к виртуальной машине и выберите **диски**.
1. Выберите Ultra Disk, для которого требуется изменить производительность.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/select-ultra-disk-to-modify.png" alt-text="Снимок экрана: колонка &quot;диски&quot; на виртуальной машине, выделена дискета.":::

1. Выберите **размер и производительность** , а затем внесите необходимые изменения.
1. Щелкните **Сохранить**.

    :::image type="content" source="media/virtual-machines-disks-getting-started-ultra-ssd/modify-ultra-disk-performance.png" alt-text="Снимок экрана: колонка настройки на Ultra Disk, размер диска, число операций ввода-вывода в секунду и пропускная способность выделены, сохранение выделено.":::

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Ultra Disks предлагает уникальную возможность, позволяющую настроить их производительность. в следующей команде показано, как использовать эту функцию:

```azurecli-interactive
az disk update `
--subscription $subscription `
--resource-group $rgname `
--name $diskName `
--set diskIopsReadWrite=80000 `
--set diskMbpsReadWrite=800
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

## <a name="adjust-the-performance-of-an-ultra-disk-using-powershell"></a>Настройка производительности Ultra Disk с помощью PowerShell

У дисков Ultra есть уникальные возможности, позволяющие настроить их производительность. ниже приведен пример, в котором выполняется настройка производительности без отключения диска.

```powershell
$diskupdateconfig = New-AzDiskUpdateConfig -DiskMBpsReadWrite 2000
Update-AzDisk -ResourceGroupName $resourceGroup -DiskName $diskName -DiskUpdate $diskupdateconfig
```
---

## <a name="next-steps"></a>Дальнейшие действия

- [Используйте Azure Ultra Disks в службе Kubernetes Azure (Предварительная версия)](../aks/use-ultra-disks.md).
- [Перенесите диск журнала на диск Ultra](../azure-sql/virtual-machines/windows/storage-migrate-to-ultradisk.md).
