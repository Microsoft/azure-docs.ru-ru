---
title: Создание добавочного моментального снимка
description: Дополнительные сведения о добавочных моментальных снимках для управляемых дисков, в том числе о создании их с помощью портал Azure, Azure PowerShell модуля и Azure Resource Manager.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 01/15/2021
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 52e491c88d3483f21aa74f1a9f176246033bee3c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98735798"
---
# <a name="create-an-incremental-snapshot-for-managed-disks"></a>Создание добавочного моментального снимка для управляемых дисков

[!INCLUDE [virtual-machines-disks-incremental-snapshots-description](../../includes/virtual-machines-disks-incremental-snapshots-description.md)]

## <a name="restrictions"></a>Ограничения

[!INCLUDE [virtual-machines-disks-incremental-snapshots-restrictions](../../includes/virtual-machines-disks-incremental-snapshots-restrictions.md)]


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Для создания добавочного моментального снимка можно использовать Azure PowerShell. Вам потребуется последняя версия Azure PowerShell. Следующая команда либо установит ее, либо обновит существующую установку до последней версии:

```PowerShell
Install-Module -Name Az -AllowClobber -Scope CurrentUser
```

После установки войдите в сеанс PowerShell с помощью `Connect-AzAccount` .

Чтобы создать добавочный моментальный снимок с Azure PowerShell, настройте конфигурацию с помощью командлета [New-азснапшотконфиг](/powershell/module/az.compute/new-azsnapshotconfig) с `-Incremental` параметром, а затем передайте эту переменную в командлет [New-азснапшот](/powershell/module/az.compute/new-azsnapshot) с помощью `-Snapshot` параметра.

```PowerShell
$diskName = "yourDiskNameHere>"
$resourceGroupName = "yourResourceGroupNameHere"
$snapshotName = "yourDesiredSnapshotNameHere"

# Get the disk that you need to backup by creating an incremental snapshot
$yourDisk = Get-AzDisk -DiskName $diskName -ResourceGroupName $resourceGroupName

# Create an incremental snapshot by setting the SourceUri property with the value of the Id property of the disk
$snapshotConfig=New-AzSnapshotConfig -SourceUri $yourDisk.Id -Location $yourDisk.Location -CreateOption Copy -Incremental 
New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName -Snapshot $snapshotConfig 
```

Можно выявление добавочных моментальных снимков с того же диска со `SourceResourceId` `SourceUniqueId` свойствами и моментальными снимками. `SourceResourceId` Azure Resource Manager идентификатор ресурса родительского диска. `SourceUniqueId` значение, унаследованное от `UniqueId` Свойства диска. Если вы удалили диск, а затем создаете новый диск с тем же именем, значение `UniqueId` Свойства изменится.

`SourceResourceId` `SourceUniqueId` Для создания списка всех моментальных снимков, связанных с определенным диском, можно использовать и. Замените на `<yourResourceGroupNameHere>` свое значение, а затем можно использовать следующий пример для создания списка существующих добавочных моментальных снимков:

```PowerShell
$snapshots = Get-AzSnapshot -ResourceGroupName $resourceGroupName

$incrementalSnapshots = New-Object System.Collections.ArrayList
foreach ($snapshot in $snapshots)
{
    
    if($snapshot.Incremental -and $snapshot.CreationData.SourceResourceId -eq $yourDisk.Id -and $snapshot.CreationData.SourceUniqueId -eq $yourDisk.UniqueId){

        $incrementalSnapshots.Add($snapshot)
    }
}

$incrementalSnapshots
```

# <a name="portal"></a>[Портал](#tab/azure-portal)
[!INCLUDE [virtual-machines-disks-incremental-snapshots-portal](../../includes/virtual-machines-disks-incremental-snapshots-portal.md)]

# <a name="resource-manager-template"></a>[Шаблон Resource Manager](#tab/azure-resource-manager)

Можно также использовать шаблоны Azure Resource Manager для создания добавочного моментального снимка. Необходимо убедиться, что для apiVersion задано значение **2019-03-01** и для свойства инкремента задано значение true. В следующем фрагменте кода приведен пример создания добавочного моментального снимка с помощью шаблонов диспетчер ресурсов.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskName": {
      "type": "string",
      "defaultValue": "contosodisk1"
    },
  "diskResourceId": {
    "defaultValue": "<your_managed_disk_resource_ID>",
    "type": "String"
  }
  }, 
  "resources": [
  {
    "type": "Microsoft.Compute/snapshots",
    "name": "[concat( parameters('diskName'),'_snapshot1')]",
    "location": "[resourceGroup().location]",
    "apiVersion": "2019-03-01",
    "properties": {
      "creationData": {
        "createOption": "Copy",
        "sourceResourceId": "[parameters('diskResourceId')]"
      },
      "incremental": true
    }
  }
  ]
}
```
---

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите просмотреть пример кода, демонстрирующий возможность создания добавочных моментальных снимков с помощью .NET, см. статью [копирование резервных копий управляемых дисков Azure в другой регион с помощью разностных моментальных снимков](https://github.com/Azure-Samples/managed-disks-dotnet-backup-with-incremental-snapshots).
