---
title: Чтение журналов потоков NSG | Документация Майкрософт
description: Узнайте, как использовать Azure PowerShell для анализа журналов потоков для групп безопасности сети, которые создаются ежечасно и обновляются каждые несколько минут в наблюдателе за сетями Azure.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/04/2021
ms.author: damendo
ms.openlocfilehash: 27f94c43266fe324016a73e2e6d31e8488457416
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100593058"
---
# <a name="read-nsg-flow-logs"></a>Чтение журналов потоков NSG

Узнайте, как читать записи журналов потоков групп безопасности сети (NSG) с помощью PowerShell.

Журналы потоков NSG хранятся в учетной записи хранения в [блочных BLOB-объектах](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs). Блочные BLOB-объекты состоят из небольших блоков. Каждый журнал — это отдельный блочный BLOB-объект, который создается один раз в час. Каждый час создается по журналу, а новые записи добавляются в журнал каждые несколько минут, используя самые актуальные данные. В этой статье показано, как читать части этих журналов потоков.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="scenario"></a>Сценарий

В следующем сценарии используется пример журнала потоков, хранящийся в учетной записи хранения. Вы узнаете, как выборочно просматривать последние события в журналах потоков NSG. В этой статье вы будете использовать PowerShell, однако описанные здесь концепции не ограничиваются одним языком программирования и применимы для всех языков, которые поддерживаются интерфейсами API службы хранилища Azure.

## <a name="setup"></a>Настройка

Прежде чем мы начнем работу, вам необходимо включить журналы потоков для одной или нескольких групп безопасности сети в вашей учетной записи. Инструкции по включению журналов потоков для групп безопасности сети есть в статье [Introduction to flow logging for Network Security Groups](network-watcher-nsg-flow-logging-overview.md) (Введение в журналы потоков для групп безопасности сети).

## <a name="retrieve-the-block-list"></a>Получение списка блоков

Следующий сценарий PowerShell задает переменные, которые необходимы для запроса BLOB-объекта журнала потоков NSG и отображения списка блоков внутри блочного BLOB-объекта [CloudBlockBlob](/dotnet/api/microsoft.azure.storage.blob.cloudblockblob). Обновите сценарий, указав допустимые значения для своей среды.

```powershell
function Get-NSGFlowLogCloudBlockBlob {
    [CmdletBinding()]
    param (
        [string] [Parameter(Mandatory=$true)] $subscriptionId,
        [string] [Parameter(Mandatory=$true)] $NSGResourceGroupName,
        [string] [Parameter(Mandatory=$true)] $NSGName,
        [string] [Parameter(Mandatory=$true)] $storageAccountName,
        [string] [Parameter(Mandatory=$true)] $storageAccountResourceGroup,
        [string] [Parameter(Mandatory=$true)] $macAddress,
        [datetime] [Parameter(Mandatory=$true)] $logTime
    )

    process {
        # Retrieve the primary storage account key to access the NSG logs
        $StorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $storageAccountResourceGroup -Name $storageAccountName).Value[0]

        # Setup a new storage context to be used to query the logs
        $ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

        # Container name used by NSG flow logs
        $ContainerName = "insights-logs-networksecuritygroupflowevent"

        # Name of the blob that contains the NSG flow log
        $BlobName = "resourceId=/SUBSCRIPTIONS/${subscriptionId}/RESOURCEGROUPS/${NSGResourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/${NSGName}/y=$($logTime.Year)/m=$(($logTime).ToString("MM"))/d=$(($logTime).ToString("dd"))/h=$(($logTime).ToString("HH"))/m=00/macAddress=$($macAddress)/PT1H.json"

        # Gets the storage blog
        $Blob = Get-AzStorageBlob -Context $ctx -Container $ContainerName -Blob $BlobName

        # Gets the block blog of type 'Microsoft.Azure.Storage.Blob.CloudBlob' from the storage blob
        $CloudBlockBlob = [Microsoft.Azure.Storage.Blob.CloudBlockBlob] $Blob.ICloudBlob

        #Return the Cloud Block Blob
        $CloudBlockBlob
    }
}

function Get-NSGFlowLogBlockList  {
    [CmdletBinding()]
    param (
        [Microsoft.Azure.Storage.Blob.CloudBlockBlob] [Parameter(Mandatory=$true)] $CloudBlockBlob
    )
    process {
        # Stores the block list in a variable from the block blob.
        $blockList = $CloudBlockBlob.DownloadBlockListAsync()

        # Return the Block List
        $blockList
    }
}


$CloudBlockBlob = Get-NSGFlowLogCloudBlockBlob -subscriptionId "yourSubscriptionId" -NSGResourceGroupName "FLOWLOGSVALIDATIONWESTCENTRALUS" -NSGName "V2VALIDATIONVM-NSG" -storageAccountName "yourStorageAccountName" -storageAccountResourceGroup "ml-rg" -macAddress "000D3AF87856" -logTime "11/11/2018 03:00" 

$blockList = Get-NSGFlowLogBlockList -CloudBlockBlob $CloudBlockBlob
```

Переменная `$blockList` возвращает список блоков в BLOB-объекте. Каждый блочный BLOB-объект содержит не менее двух блоков.  Длина первого блока — `12` байт. Этот блок содержит открывающие скобки журнала JSON. Последний блок содержит закрывающие скобки, а его длина — `2` байт.  Как видите, следующий пример журнала содержит семь записей, каждая из которых является отдельной записью. Все новые записи добавляются в конец журнала, непосредственно перед последним блоком.

```
Name                                         Length Committed
----                                         ------ ---------
ZDk5MTk5N2FkNGE0MmY5MTk5ZWViYjA0YmZhODRhYzY=     12      True
NzQxNDA5MTRhNDUzMGI2M2Y1MDMyOWZlN2QwNDZiYzQ=   2685      True
ODdjM2UyMWY3NzFhZTU3MmVlMmU5MDNlOWEwNWE3YWY=   2586      True
ZDU2MjA3OGQ2ZDU3MjczMWQ4MTRmYWNhYjAzOGJkMTg=   2688      True
ZmM3ZWJjMGQ0ZDA1ODJlOWMyODhlOWE3MDI1MGJhMTc=   2775      True
ZGVkYTc4MzQzNjEyMzlmZWE5MmRiNjc1OWE5OTc0OTQ=   2676      True
ZmY2MjUzYTIwYWIyOGU1OTA2ZDY1OWYzNmY2NmU4ZTY=   2777      True
Mzk1YzQwM2U0ZWY1ZDRhOWFlMTNhYjQ3OGVhYmUzNjk=   2675      True
ZjAyZTliYWE3OTI1YWZmYjFmMWI0MjJhNzMxZTI4MDM=      2      True
```

## <a name="read-the-block-blob"></a>Чтение блочного BLOB-объекта

Чтобы получить данные, необходимо прочитать переменную `$blocklist`. В этом примере мы выполняем итерацию по списку блоков, считываем байты из каждого блока и сохраняем их в массиве. Для извлечения данных используйте метод [DownloadRangeToByteArray](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadrangetobytearray).

```powershell
function Get-NSGFlowLogReadBlock  {
    [CmdletBinding()]
    param (
        [System.Array] [Parameter(Mandatory=$true)] $blockList,
        [Microsoft.Azure.Storage.Blob.CloudBlockBlob] [Parameter(Mandatory=$true)] $CloudBlockBlob

    )
    # Set the size of the byte array to the largest block
    $maxvalue = ($blocklist | measure Length -Maximum).Maximum

    # Create an array to store values in
    $valuearray = @()

    # Define the starting index to track the current block being read
    $index = 0

    # Loop through each block in the block list
    for($i=0; $i -lt $blocklist.count; $i++)
    {
        # Create a byte array object to story the bytes from the block
        $downloadArray = New-Object -TypeName byte[] -ArgumentList $maxvalue

        # Download the data into the ByteArray, starting with the current index, for the number of bytes in the current block. Index is increased by 3 when reading to remove preceding comma.
        $CloudBlockBlob.DownloadRangeToByteArray($downloadArray,0,$index, $($blockList[$i].Length)) | Out-Null

        # Increment the index by adding the current block length to the previous index
        $index = $index + $blockList[$i].Length

        # Retrieve the string from the byte array

        $value = [System.Text.Encoding]::ASCII.GetString($downloadArray)

        # Add the log entry to the value array
        $valuearray += $value
    }
    #Return the Array
    $valuearray
}
$valuearray = Get-NSGFlowLogReadBlock -blockList $blockList -CloudBlockBlob $CloudBlockBlob
```

Теперь массив `$valuearray` содержит строковое значение каждого блока. Чтобы проверить запись, получите предпоследнее значение из массива, выполнив команду `$valuearray[$valuearray.Length-2]`. Последнее значение не потребуется, так как оно содержит закрывающую скобку.

В следующем примере показано полученное значение:

```json
        {
             "time": "2017-06-16T20:59:43.7340000Z",
             "systemId": "5f4d02d3-a7d0-4ed4-9ce8-c0ae9377951c",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/CONTOSORG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/CONTOSONSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_AllowInternetOutBound","flows":[{"mac":"000D3A18077E","flowTuples":["1497646722,10.0.0.4,168.62.32.14,44904,443,T,O,A","1497646722,10.0.0.4,52.240.48.24,45218,443,T,O,A","1497646725,10.
0.0.4,168.62.32.14,44910,443,T,O,A","1497646725,10.0.0.4,52.240.48.24,45224,443,T,O,A","1497646728,10.0.0.4,168.62.32.14,44916,443,T,O,A","1497646728,10.0.0.4,52.240.48.24,45230,443,T,O,A","1497646732,10.0.0.4,168.62.32.14,44922,443,T,O,A","14976
46732,10.0.0.4,52.240.48.24,45236,443,T,O,A","1497646735,10.0.0.4,168.62.32.14,44928,443,T,O,A","1497646735,10.0.0.4,52.240.48.24,45242,443,T,O,A","1497646738,10.0.0.4,168.62.32.14,44934,443,T,O,A","1497646738,10.0.0.4,52.240.48.24,45248,443,T,O,
A","1497646742,10.0.0.4,168.62.32.14,44942,443,T,O,A","1497646742,10.0.0.4,52.240.48.24,45256,443,T,O,A","1497646745,10.0.0.4,168.62.32.14,44948,443,T,O,A","1497646745,10.0.0.4,52.240.48.24,45262,443,T,O,A","1497646749,10.0.0.4,168.62.32.14,44954
,443,T,O,A","1497646749,10.0.0.4,52.240.48.24,45268,443,T,O,A","1497646753,10.0.0.4,168.62.32.14,44960,443,T,O,A","1497646753,10.0.0.4,52.240.48.24,45274,443,T,O,A","1497646756,10.0.0.4,168.62.32.14,44966,443,T,O,A","1497646756,10.0.0.4,52.240.48
.24,45280,443,T,O,A","1497646759,10.0.0.4,168.62.32.14,44972,443,T,O,A","1497646759,10.0.0.4,52.240.48.24,45286,443,T,O,A","1497646763,10.0.0.4,168.62.32.14,44978,443,T,O,A","1497646763,10.0.0.4,52.240.48.24,45292,443,T,O,A","1497646766,10.0.0.4,
168.62.32.14,44984,443,T,O,A","1497646766,10.0.0.4,52.240.48.24,45298,443,T,O,A","1497646769,10.0.0.4,168.62.32.14,44990,443,T,O,A","1497646769,10.0.0.4,52.240.48.24,45304,443,T,O,A","1497646773,10.0.0.4,168.62.32.14,44996,443,T,O,A","1497646773,
10.0.0.4,52.240.48.24,45310,443,T,O,A","1497646776,10.0.0.4,168.62.32.14,45002,443,T,O,A","1497646776,10.0.0.4,52.240.48.24,45316,443,T,O,A","1497646779,10.0.0.4,168.62.32.14,45008,443,T,O,A","1497646779,10.0.0.4,52.240.48.24,45322,443,T,O,A"]}]}
,{"rule":"DefaultRule_DenyAllInBound","flows":[]},{"rule":"UserRule_ssh-rule","flows":[]},{"rule":"UserRule_web-rule","flows":[{"mac":"000D3A18077E","flowTuples":["1497646738,13.82.225.93,10.0.0.4,1180,80,T,I,A","1497646750,13.82.225.93,10.0.0.4,
1184,80,T,I,A","1497646768,13.82.225.93,10.0.0.4,1181,80,T,I,A","1497646780,13.82.225.93,10.0.0.4,1336,80,T,I,A"]}]}]}
        }
```

Этот сценарий является примером того, как прочитать записи в журналах потоков NSG, не анализируя весь журнал. Вы можете считывать новые записи в журнале по мере их записывания, используя идентификатор блока или отслеживая длину блоков, сохраненных в блочном BLOB-объекте. Это позволяет считывать только новые записи.

## <a name="next-steps"></a>Дальнейшие действия


Для дополнительных сведений о способах просмотра журналов потоков NSG см. статьи [Визуализация журнала потоков для групп безопасности сети Наблюдателя за сетями Azure с помощью инструментов с открытым кодом](network-watcher-visualize-nsg-flow-logs-open-source-tools.md), [Управление журналами потоков для групп безопасности сети и их анализ с помощью наблюдателя за сетями и Grafana](network-watcher-nsg-grafana.md), и [Анализ журналов потоков для групп безопасности сети и управление ими в Azure с помощью Наблюдателя за сетями и Graylog](network-watcher-analyze-nsg-flow-logs-graylog.md). Функция Azure с открытым исходным кодом для использования больших двоичных объектов, которые можно использовать непосредственно и передать различным потребителям log Analytics, находится здесь: [соединитель журналов потоков NSG для наблюдателя за сетями Azure](https://github.com/Microsoft/AzureNetworkWatcherNSGFlowLogsConnector).

[Аналитика трафика Azure](./traffic-analytics.md) можно использовать для получения ценной информации о потоках трафика. Аналитика трафика использует [log Analytics](../azure-monitor/logs/log-analytics-tutorial.md) , чтобы сделать запрос трафика необработанным.

Дополнительные сведения о больших двоичных объектах хранилища см. в статье [Привязки хранилища BLOB-объектов для Функций Azure](../azure-functions/functions-bindings-storage-blob.md).