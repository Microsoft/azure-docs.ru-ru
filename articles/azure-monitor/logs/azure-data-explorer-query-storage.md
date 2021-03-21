---
title: Запрос экспортированных данных из Azure Monitor с помощью обозреватель данных Azure (Предварительная версия)
description: Используйте обозреватель данных Azure для запроса данных, экспортированных из рабочей области Log Analytics в учетную запись хранения Azure.
author: osalzberg
ms.author: bwren
ms.reviewer: bwren
ms.topic: conceptual
ms.date: 10/13/2020
ms.openlocfilehash: 5eff593075db118b23d74147e33b40eb4402193c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102031163"
---
# <a name="query-exported-data-from-azure-monitor-using-azure-data-explorer-preview"></a>Запрос экспортированных данных из Azure Monitor с помощью обозреватель данных Azure (Предварительная версия)
Экспорт данных из Azure Monitor в учетную запись хранения Azure обеспечивает недорогое хранение и возможность перераспределения журналов в разные регионы. Используйте обозреватель данных Azure для запроса данных, экспортированных из рабочих областей Log Analytics. После настройки поддерживаемые таблицы, отправленные из рабочих областей в учетную запись хранения Azure, будут доступны в качестве источника данных для Azure обозреватель данных.

Поток процесса выглядит следующим образом: 

1.  Экспорт данных из рабочей области Log Analytics в учетную запись хранения Azure.
2.  Создайте внешнюю таблицу в кластере Azure обозреватель данных и сопоставлении для типов данных.
3.  Запрос данных из обозреватель данных Azure.

:::image type="content" source="media/azure-data-explorer-query-storage/exported-data-query.png" alt-text="Поток запросов экспортированных данных обозреватель данных Azure.":::



## <a name="send-data-to-azure-storage"></a>Отправка данных в службу хранилища Azure
Журналы Azure Monitor можно экспортировать в учетную запись хранения Azure с помощью любого из следующих параметров.

- Чтобы экспортировать все данные из рабочей области Log Analytics в учетную запись хранения Azure или концентратор событий, используйте функцию экспорта данных рабочей области Log Analytics в журналах Azure Monitor. См. раздел [Экспорт данных рабочей области log Analytics в Azure Monitor (Предварительная версия)](./logs-data-export.md) .
- Запланированный экспорт из запроса журнала с помощью приложения логики. Это похоже на функцию экспорта данных, но позволяет отсылать отфильтрованные или агрегированные данные в службу хранилища Azure. Несмотря на то, что для этого метода действуют [ограничения на запросы журнала](../service-limits.md#log-analytics-workspaces)  , см. раздел [архивные данные из рабочей области log Analytics в хранилище Azure с помощью приложения логики](./logs-export-logic-app.md).
- Однократный экспорт с помощью приложения логики. [Logic Apps и автоматизация питания см. в разделе Azure Monitor журналов](./logicapp-flow-connector.md).
- Один раз экспортируйте на локальный компьютер с помощью скрипта PowerShell. См. раздел [Invoke-азоператионалинсигхтскуерекспорт](https://www.powershellgallery.com/packages/Invoke-AzOperationalInsightsQueryExport).

> [!TIP]
> Вы можете использовать существующий кластер Azure обозреватель данных или создать новый выделенный кластер с необходимыми конфигурациями.

## <a name="create-an-external-table-located-in-azure-blob-storage"></a>Создание внешней таблицы, расположенной в хранилище BLOB-объектов Azure
Используйте [внешние таблицы](/azure/data-explorer/kusto/query/schema-entities/externaltables) , чтобы связать обозреватель данных Azure с учетной записью хранения Azure. Внешняя таблица — это сущность схемы Kusto, которая ссылается на данные, хранящиеся вне базы данных Kusto. Как и таблицы, внешняя таблица имеет четко определенную схему. В отличие от таблиц, данные хранятся и управляются за пределами кластера Kusto. Экспортированные данные из предыдущего раздела сохраняются в строках JSON.

Чтобы создать ссылку, требуется схема экспортированной таблицы. Используйте оператор [GetSchema](/azure/data-explorer/kusto/query/getschemaoperator) из log Analytics, чтобы получить эти сведения, включая столбцы таблицы и их типы данных.

:::image type="content" source="media\azure-data-explorer-query-storage\exported-data-map-schema.jpg" alt-text="Схема таблицы Log Analytics.":::

Теперь можно использовать выходные данные для создания запроса Kusto для создания внешней таблицы.
Следуя указаниям в статье [Создание и изменение внешних таблиц в службе хранилища Azure или Azure Data Lake](/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake), создайте внешнюю таблицу в формате JSON, а затем выполните запрос из базы данных Обозреватель данных Azure.

>[!NOTE]
>Создание внешней таблицы основано на двух процессах. Первый — это создание внешней таблицы, а вторая — создание сопоставления.

Следующий скрипт PowerShell создаст команды [CREATE](/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake#create-external-table-mapping) для таблицы и сопоставления.

```powershell
PARAM(
    $resourcegroupname, #The name of the Azure resource group
    $TableName, # The Log Analytics table you wish to convert to external table
    $MapName, # The name of the map
    $subscriptionId, #The ID of the subscription
    $WorkspaceId, # The Log Analytics WorkspaceId
    $WorkspaceName, # The Log Analytics workspace name
    $BlobURL, # The Blob URL where the data is saved
    $ContainerAccessKey, # The blob container Access Key (Option to add a SAS url)
    $ExternalTableName = $null # The External Table name, null to use the same name
)

if($null -eq $ExternalTableName) {
    $ExternalTableName = $TableName
}

$query = $TableName + ' | getschema | project ColumnName, DataType'

$output = (Invoke-AzOperationalInsightsQuery -WorkspaceId $WorkspaceId -Query $query).Results

$FirstCommand = @()
$SecondCommand = @()

foreach ($record in $output) {
    if ($record.DataType -eq 'System.DateTime') {
        $dataType = 'datetime'
    } else {
        $dataType = 'string'
    }
    $FirstCommand += $record.ColumnName + ":" + "$dataType" + ","
    $SecondCommand += "{`"column`":" + "`"" + $record.ColumnName + "`"," + "`"datatype`":`"$dataType`",`"path`":`"$." + $record.ColumnName + "`"},"
}
$schema = ($FirstCommand -join '') -replace ',$'
$mapping = ($SecondCommand -join '') -replace ',$'

$CreateExternal = @'
.create external table {0} ({1})
kind=blob
partition by (TimeGeneratedPartition:datetime = bin(TimeGenerated, 1min))
pathformat = (datetime_pattern("'y='yyyy'/m='MM'/d='dd'/h='HH'/m='mm", TimeGeneratedPartition))
dataformat=multijson
(
   h@'{2}/WorkspaceResourceId=/subscriptions/{4}/resourcegroups/{6}/providers/microsoft.operationalinsights/workspaces/{5};{3}'
)
with
(
   docstring = "Docs",
   folder = "ExternalTables"
)
'@ -f $TableName, $schema, $BlobURL, $ContainerAccessKey, $subscriptionId, $WorkspaceName, $resourcegroupname,$WorkspaceId

$createMapping = @'
.create external table {0} json mapping "{1}" '[{2}]'
'@ -f $ExternalTableName, $MapName, $mapping

Write-Host -ForegroundColor Red 'Copy and run the following commands (one by one), on your Azure Data Explorer cluster query window to create the external table and mappings:'
write-host -ForegroundColor Green $CreateExternal
Write-Host -ForegroundColor Green $createMapping
```

На следующем рисунке показаны и примеры выходных данных.

:::image type="content" source="media/azure-data-explorer-query-storage/external-table-create-command-output.png" alt-text="Екстерналтабле создание выходных данных команды.":::

[![Пример выходных данных](media/azure-data-explorer-query-storage/external-table-create-command-output.png)](media/azure-data-explorer-query-storage/external-table-create-command-output.png#lightbox)

>[!TIP]
>* Скопируйте, вставьте и запустите выходные данные скрипта в клиентском средстве Azure обозреватель данных, чтобы создать таблицу и сопоставление.
>* Для использования всех данных в контейнере измените скрипт и измените URL-адрес на " https://your.blob.core.windows.net/containername ; Секкэй "

## <a name="query-the-exported-data-from-azure-data-explorer"></a>Запрос экспортированных данных из Azure обозреватель данных 

После настройки сопоставления можно запросить экспортированные данные из обозреватель данных Azure. Для запроса требуется функция [external_table](/azure/data-explorer/kusto/query/externaltablefunction) , как показано в следующем примере.

```kusto
external_table("HBTest","map") | take 10000
```

[![Запрос Log Analytics экспортированных данных](media/azure-data-explorer-query-storage/external-table-query.png)](media/azure-data-explorer-query-storage/external-table-query.png#lightbox)

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о [написании запросов в Azure обозреватель данных](/azure/data-explorer/write-queries)