---
title: Включение ускорения дисков по запросу
description: Включите на управляемом диске возможность ускорения использования дисков по требованию.
author: albecker1
ms.author: albecker
ms.date: 03/02/2021
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 733d441705c7c77f0667f88151e96f76975ee0b2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104596405"
---
# <a name="enable-on-demand-bursting"></a>Включение ускорения по запросу

Твердотельные накопители уровня "Премиум" имеют две доступные модели нагрузки. Разбивка на запросы на кредитование и выполнение по требованию. В этой статье объясняется, как переключаться на пакет по требованию. Диски, использующие модель по запросу, могут работать дольше, чем исходные подготовленные целевые объекты. Выполнение по запросу происходит так часто, как это требуется для рабочей нагрузки, вплоть до максимальной нагрузки. Для ускорения по запросу требуется дополнительная плата.

Дополнительные сведения о ускоренном обращении к дискам см. в разделе нагрузка [управляемого диска](disk-bursting.md).

> [!IMPORTANT]
> Вам не нужно выполнять действия, описанные в этой статье, чтобы использовать разбивку на пакеты на основе кредита. По умолчанию разбивка по кредиту включена на всех соответствующих дисках.

Перед включением ускорения по запросу необходимо учитывать следующее.

[!INCLUDE [managed-disk-bursting-regions-limitations](../../includes/managed-disk-bursting-regions-limitations.md)]

### <a name="regional-availability"></a>Доступность по регионам

[!INCLUDE [managed-disk-bursting-availability](../../includes/managed-disk-bursting-availability.md)]

## <a name="get-started"></a>Начало работы

Для включения поддержки по требованию можно использовать модуль Azure PowerShell, Azure CLI или шаблоны Azure Resource Manager. В следующих примерах показано, как создать новый диск с включенной поддержкой ускорения по запросу и включить для него возможность ускорения работы на существующих дисках.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Командлеты для ускорения по запросу доступны в версии 5.5.0 и более поздней версии модуля AZ. Кроме того, вы можете использовать [Azure Cloud Shell](https://shell.azure.com/).
### <a name="create-an-empty-data-disk-with-on-demand-bursting"></a>Создание пустого диска данных с помощью ускорения по запросу

Управляемый диск должен быть больше 512 гиб для включения ускорения по требованию. Замените `<myResourceGroupDisk>` Параметры и, `<myDataDisk>` а затем выполните следующий скрипт, чтобы создать SSD уровня "Премиум" с помощью ускорения по запросу:

```azurepowershell
Set-AzContext -SubscriptionName <yourSubscriptionName>

$diskConfig = New-AzDiskConfig -Location 'WestCentralUS' -CreateOption Empty -DiskSizeGB 1024 -SkuName Premium_LRS -BurstingEnabled $true

$dataDisk = New-AzDisk -ResourceGroupName <myResourceGroupDisk> -DiskName <myDataDisk> -Disk $diskConfig
```

### <a name="enable-on-demand-bursting-on-an-existing-disk"></a>Включение ускорения по запросу на существующем диске

Управляемый диск должен быть больше 512 гиб для включения ускорения по требованию. Замените `<myResourceGroupDisk>` , `<myDataDisk>` Параметры и выполните следующую команду, чтобы включить ускоренное выполнение по запросу на существующем диске:

```azurepowershell
New-AzDiskUpdateConfig -BurstingEnabled $true | Update-AzDisk -ResourceGroupName <myResourceGroupDisk> -DiskName <myDataDisk> //Set the flag to $false to disable on-demand bursting
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Командлеты для ускорения по запросу доступны в версии 2.19.0 и более новой версии [модуля Azure CLI](/cli/azure/install-azure-cli). Кроме того, вы можете использовать [Azure Cloud Shell](https://shell.azure.com/).

### <a name="create-and-attach-a-on-demand-bursting-data-disk"></a>Создание и присоединение диска данных для ускорения по запросу

Управляемый диск должен быть больше 512 гиб для включения ускорения по требованию. Замените `<yourDiskName>` Параметры, `<yourResourceGroup>` и `<yourVMName>` , а затем выполните следующие команды, чтобы создать SSD уровня "Премиум" с разбивкой по запросу.

```azurecli
az disk create -g <yourResourceGroup> -n <yourDiskName> --size-gb 1024 --sku Premium_LRS -l westcentralus --enable-bursting true

az vm disk attach --vm-name <yourVMName> --name <yourDiskName> --resource-group <yourResourceGroup>
```

### <a name="enable-on-demand-bursting-on-an-existing-disk---cli"></a>Включение ускорения по запросу на существующем диске — CLI

Управляемый диск должен быть больше 512 гиб для включения ускорения по требованию. Замените `<myResourceGroupDisk>` Параметры и `<yourDiskName>` , а затем выполните следующую команду, чтобы включить ускоренное выполнение по запросу на существующем диске:

```azurecli
az disk update --name <yourDiskName> --resource-group <yourResourceGroup> --enable-bursting true //Set the flag to false to disable on-demand bursting
```

# <a name="azure-resource-manager"></a>[Azure Resource Manager](#tab/azure-resource-manager)

С помощью `2020-09-30` API диска можно включить пакет по требованию для вновь созданных или существующих твердотельных накопителей уровня "Премиум", превышающих 512 гиб. `2020-09-30`API представил новое свойство, `burstingEnabled` . По умолчанию для этого свойства задано значение false. В следующем примере шаблона создается SSD 1TiB Premium в западной центральной части США с включенной обрезкой дисков:

```
{
  "$schema": "http://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskSkuName": {
        "type": "string",
        "defaultValue": "Premium_LRS" //Supported on premium SSDs only
},
    "dataDiskSizeInGb": {
      "type": "string",
      "defaultValue": "1024" //Supported on disk size > 512 GiB
},
    "location": {
      "type": "string",
      "defaultValue": "westcentralus" //Preview regions: West Central US
},
"diskApiVersion": {
      "type": "string",
      "defaultValue": "2020-09-30" //Preview supported version: 2020-09-30 or above
    }
  },
  "resources": [
    {
      "apiVersion": "[parameters('diskApiVersion')]",
      "type": "Microsoft.Compute/disks",
      "name": "[parameters('diskName')]",
      "location": "[parameters(location)]",
      "properties": {
        "creationData": {
          "createOption": "Empty"
        },
        "diskSizeGB": "[parameters('dataDiskSizeInGb')]",
        "burstingEnabled": "true" //Feature flag to enable disk bursting on disks > 512 GiB
      },
      "sku": {
        "name": "[parameters('diskSkuName')]"
      }
    ]
}
```
---
 
## <a name="next-steps"></a>Дальнейшие действия

Сведения о том, как получить представление о пакетных ресурсах, см. в разделе [метрики ускорения работы диска](disks-metrics.md).