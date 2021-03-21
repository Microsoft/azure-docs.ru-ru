---
title: Удаление больших двоичных объектов хранилища для журналов потоков для групп безопасности сети в наблюдателе за сетями Azure | Документация Майкрософт
description: В этой статье объясняется, как удалить большие двоичные объекты хранилища журналов потоков для групп безопасности сети, которые выходят за пределы политики хранения в наблюдателе за сетями Azure.
services: network-watcher
documentationcenter: na
author: damendo
manager: ''
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/07/2021
ms.author: damendo
ms.openlocfilehash: 1b73b8fa14271619d07fca63b01e4197f4f7c8cb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98017856"
---
# <a name="delete-network-security-group-flow-log-storage-blobs-in-network-watcher"></a>Удаление больших двоичных объектов хранилища журналов потоков для групп безопасности сети в наблюдателе за сетями

Если вам когда-либо потребуется вручную удалить журналы последовательностей из учетной записи хранения, можно использовать приведенный ниже сценарий PowerShell.
Этот скрипт удаляет только большие двоичные объекты хранилища, которые старше существующей политики хранения, указанной пользователем.

## <a name="run-powershell-script-to-delete-nsg-flow-logs"></a>Запуск скрипта PowerShell для удаления журналов потоков NSG
 
Скопируйте и сохраните следующий скрипт в расположении, например в текущем рабочем каталоге. 

```powershell
# This powershell script deletes all NSG flow log blobs that should not be retained anymore as per configured retention policy.
# While configuring NSG flow logs on Azure portal, the user configures the retention period of NSG flow log blobs in
# their storage account (in days).
# This script reads all blobs and deletes blobs that are not to be retained (outside retention window)
# if the retention days are zero; all blobs are retained forever and hence no blobs are deleted.
#
#

param (
        [string] [Parameter(Mandatory=$true)]  $SubscriptionId,
        [string] [Parameter(Mandatory=$true)]  $Location,
        [switch] [Parameter(Mandatory=$false)] $Confirm
    )

Login-AzAccount

$SubId = Get-AzSubscription| Where-Object {$_.Id.contains($SubscriptionId.ToLower())}

if ($SubId.Count -eq 0)
{
    Write-Error 'The SubscriptionId does not exist' -ErrorAction Stop
}

Set-AzContext -SubscriptionId $SubscriptionId

$NsgList = Get-AzNetworkSecurityGroup | Where-Object {$_.Location -eq $Location}
$NW = Get-AzNetworkWatcher | Where-Object {$_.Location -eq $Location}

$FlowLogsList = @()
foreach ($Nsg in $NsgList)
{
    # Query Flow Log Status which are enabled
    $NsgFlowLog = Get-AzNetworkWatcherFlowLogStatus -NetworkWatcher $NW -TargetResourceId $Nsg.Id | Where-Object {$_.Enabled -eq "True"}
    if ($NsgFlowLog.Count -gt 0)
    {
        $FlowLogsList +=  $NsgFlowLog
        Write-Output ('Enabled NSG found: ' +  $NsgFlowLog.TargetResourceId)
    }
}

foreach ($Psflowlog in $FlowLogsList)
{
    $RetentionDays = $Psflowlog.RetentionPolicy.Days
    if ($RetentionDays -le 0)
    {
        continue
    }

    $Strings = $Psflowlog.StorageId -split '/'
    $RGName = $Strings[4]
    $StorageAccountName = $Strings[-1]

    $Key = (Get-AzStorageAccountKey -ResourceGroupName $RGName -Name $StorageAccountName).Value[1]
    $StorageAccount = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $Key

    $ContainerName = 'insights-logs-networksecuritygroupflowevent'  
    $BLobsList = Get-AzStorageBlob -Container $ContainerName -Context $StorageAccount.Context

    $TargetBLobsList = $BLobsList | Where-Object {$_.Name.contains($Psflowlog.TargetResourceId.ToUpper())}

    $RetentionDate = Get-Date
    $RetentionDate = $RetentionDate.AddDays(-1*$RetentionDays)
    $RetentionDateInUTC = $RetentionDate.ToUniversalTime()

    foreach ($Blob in $TargetBLobsList)
    {
        $BlobLastModifietedDTinUTC = [datetime]$Blob.LastModified.UtcDateTime

        if ($BlobLastModifietedDTinUTC -ge  $RetentionDateInUTC)
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> RETAINED')
            continue
        }

        if ($Confirm)
        {
            Write-Output (Blob to be deleted: $Blob.Name)
            $Confirmation = Read-Host "Are you sure you want to remove this blob (Y/N)?"
        }

        if ((-not $Confirm) -or ($Confirmation -eq 'Y'))
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> DELETED')
            Remove-AzStorageBlob -Container $ContainerName -Context $StorageAccount.Context -Blob $Blob.Name
        }
        else
        {
            Write-Output ($Blob.Name + '===>' + $BlobLastModifietedDTinUTC  + ' ===> RETAINED')
        }
    }
}

Write-Output ('Retention policy for all NSGs evaluated and completed successfully')
```

1. При необходимости введите следующие параметры в скрипте:
   - **SubscriptionId** [обязательный]: идентификатор подписки, откуда вы хотите удалить BLOB-объекты журнала потоков NSG.
   - **Location** [обязательный]: _строка расположения_ области группы безопасности сети, для которой необходимо удалить большие двоичные объекты журнала потоков NSG. Эти сведения можно просмотреть на портал Azure или на сайте [GitHub](https://github.com/Azure/azure-extensions-cli/blob/beb3d3fe984cfa9c7798cb11a274c5337968cbc5/regions.go#L23).
   - **Подтверждение** [необязательно]: передайте флаг подтверждения, если вы хотите вручную подтвердить удаление каждого BLOB-объекта хранилища.

1. Запустите сохраненный скрипт, как показано в следующем примере, где файл скрипта был сохранен как **Delete-NsgFlowLogsBlobs.ps1**:
   ```
   .\Delete-NsgFlowLogsBlobs.ps1 -SubscriptionId <subscriptionId> -Location  <location> -Confirm
   ```
    
## <a name="next-steps"></a>Дальнейшие действия
- Клиенты могут автоматизировать выполнение скрипта с помощью [Azure Logic Apps](../logic-apps/logic-apps-overview.md) или [службы автоматизации Azure](https://azure.microsoft.com/services/automation/) .
- Дополнительные сведения о ведении журнала NSG см. в разделе [журналы Azure Monitor для групп безопасности сети (группы безопасности сети)](../virtual-network/virtual-network-nsg-manage-log.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

