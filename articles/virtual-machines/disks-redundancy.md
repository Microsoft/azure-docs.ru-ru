---
title: Варианты избыточности для управляемых дисков Azure
description: Сведения о хранилище, избыточном в зонах, и локально избыточном хранилище для управляемых дисков Azure.
author: roygara
ms.author: rogarana
ms.date: 03/02/2021
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: f0f3baf1bf56f958408f789961812c0555f289f1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102043649"
---
# <a name="redundancy-options-for-managed-disks"></a>Параметры избыточности для управляемых дисков

Управляемые диски Azure предлагают два варианта избыточности хранилища, хранилище, избыточное в виде зоны (ZRS), в качестве предварительной версии и локально избыточное хранилище. ZRS обеспечивает более высокую доступность для управляемых дисков, чем локально избыточное хранилище (LRS). Однако задержка записи для дисков LRS лучше, чем ZRS дисков, так как LRS диски синхронно записывают данные в три копии в одном центре обработки данных.

## <a name="locally-redundant-storage-for-managed-disks"></a>Локально избыточное хранилище для управляемых дисков

Локально избыточное хранилище (LRS) реплицирует данные три раза в пределах одного центра обработки данных в выбранном регионе. LRS защищает ваши данные от сбоев в стойках сервера и на дисках. 

Существует несколько способов защиты приложения с помощью дисков LRS из-за сбоя зоны, которая может возникнуть в результате стихийных аварий или проблем с оборудованием.
- Используйте приложение, например SQL Server AlwaysOn, которое может синхронно записывать данные в две зоны и автоматически отработка отказа в другую зону во время аварии.
- Регулярно задавайте резервные копии дисков LRS с помощью моментальных снимков ZRS.
- Включение аварийного восстановления между зонами для дисков LRS с помощью [Azure Site Recovery](../site-recovery/azure-to-azure-how-to-enable-zone-to-zone-disaster-recovery.md). Однако при аварийном восстановлении между зонами не предоставляется нулевая цель точки восстановления (RPO).

Если рабочий процесс не поддерживает синхронные операции записи на уровне приложения в зонах или приложение должно отвечать за нулевое значение, то диски ZRS идеальны.

## <a name="zone-redundant-storage-for-managed-disks-preview"></a>Хранилище, избыточное в виде зоны для управляемых дисков (Предварительная версия)

Хранилище, избыточное в пределах зоны (ZRS), синхронно реплицирует управляемый диск Azure по трем зонам доступности Azure в выбранном регионе. Каждая зона доступности — это отдельное физическое расположение с независимым питанием, охлаждением и сетью. 

Диски ZRS позволяют восстанавливаться после сбоев в зонах доступности. Если вся зона остановлена, диск ZRS можно подключить к виртуальной машине в другой зоне. Вы также можете использовать диски ZRS в качестве общего диска, чтобы обеспечить более высокую доступность для кластерных или распределенных приложений, таких как SQL FCI, SAP ASCS/SCS или GFS2. Вы можете подключить общий диск ZRS к первичным и вторичным виртуальным машинам в разных зонах, чтобы воспользоваться преимуществами ZRS и [зоны доступности](../availability-zones/az-overview.md). В случае сбоя основной зоны можно быстро выполнить отработку отказа на вторичную виртуальную машину, используя [постоянное резервирование SCSI](disks-shared-enable.md#supported-scsi-pr-commands).

### <a name="limitations"></a>Ограничения

Во время действия предварительной версии ZRS для управляемых дисков имеют следующие ограничения.

- Поддерживается только для твердотельных накопителей уровня "Премиум" и "Стандартный SSD".
- В настоящее время доступно только в регионе EastUS2EUAP.
- Диски ZRS можно создавать только с помощью шаблонов Azure Resource Manager, использующих `2020-12-01` API.

Зарегистрируйтесь для просмотра [здесь](https://aka.ms/ZRSDisksPreviewSignUp).

### <a name="billing-implications"></a>Процесс выставления счетов

Дополнительные сведения см. на [странице цен на Azure](https://azure.microsoft.com/pricing/details/managed-disks/).

### <a name="comparison-with-other-disk-types"></a>Сравнение с другими типами дисков

За исключением дополнительной задержки записи, диски, использующие ZRS, идентичны дискам, использующим LRS. Они имеют одинаковые целевые показатели производительности.

### <a name="create-zrs-managed-disks"></a>Создание управляемых дисков ZRS

Используйте `2020-12-01` API с шаблоном Azure Resource Manager, чтобы создать диск ZRS.

#### <a name="create-a-vm-with-zrs-disks"></a>Создание виртуальной машины с ZRS дисками

```
$vmName = "yourVMName" 
$adminUsername = "yourAdminUsername"
$adminPassword = ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$osDiskType = "StandardSSD_ZRS"
$dataDiskType = "Premium_ZRS"
$region = "eastus2euap"
$resourceGroupName = "yourResourceGroupName"

New-AzResourceGroup -Name $resourceGroupName -Location $region
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMWithZRSDataDisks.json" `
-resourceName $vmName `
-adminUsername $adminUsername `
-adminPassword $adminPassword `
-region $region `
-osDiskType $osDiskType `
-dataDiskType $dataDiskType
```

#### <a name="create-vms-with-a-shared-zrs-disk-attached-to-the-vms-in-different-zones"></a>Создание виртуальных машин с общим диском ZRS, подключенным к виртуальным машинам в разных зонах

```
$vmNamePrefix = "yourVMNamePrefix"
$adminUsername = "yourAdminUserName"
$adminPassword = ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$osDiskType = "StandardSSD_LRS"
$sharedDataDiskType = "Premium_ZRS"
$region = "eastus2euap"
$resourceGroupName = "zrstesting1"

New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMsWithASharedDisk.json" `
-vmNamePrefix $vmNamePrefix `
-adminUsername $adminUsername `
-adminPassword $adminPassword `
-region $region `
-osDiskType $osDiskType `
-dataDiskType $sharedDataDiskType
```

#### <a name="create-a-virtual-machine-scale-set-with-zrs-disks"></a>Создание масштабируемого набора виртуальных машин с ZRS дисками

```
$vmssName="yourVMSSName"
$adminUsername="yourAdminName"
$adminPassword=ConvertTo-SecureString "yourAdminPassword" -AsPlainText -Force
$region="eastus2euap"
$osDiskType="StandardSSD_LRS"
$dataDiskType="Premium_ZRS"

New-AzResourceGroupDeployment -ResourceGroupName zrstesting `
-TemplateUri "https://raw.githubusercontent.com/Azure-Samples/managed-disks-powershell-getting-started/master/ZRSDisks/CreateVMSSWithZRSDisks.json" `
-vmssName "yourVMSSName" `
-adminUsername "yourAdminName" `
-adminPassword $password `
-region "eastus2euap" `
-osDiskType "StandardSSD_LRS" `
-dataDiskType "Premium_ZRS" `
```

## <a name="next-steps"></a>Дальнейшие действия

- Используйте эти примеры [Azure Resource Manager шаблонов, чтобы создать виртуальную машину с дисками ZRS](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/ZRSDisks).