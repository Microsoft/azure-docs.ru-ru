---
title: Фильтрация событий для Сетки событий Azure
description: В этой статье показано, как отфильтровать события (по типу события, по темам, по операторам и данным и т. д.) при создании подписки на сетку событий.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 99fb00f99a055033ccfcd99e32a52d423878fb44
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86105580"
---
# <a name="filter-events-for-event-grid"></a>Фильтрация событий для Сетки событий

В этой статье показано, как фильтровать события при создании подписки на Сетку событий. Дополнительные сведения о параметрах фильтрации событий см. в статье [Understand event filtering for Event Grid subscriptions](event-filtering.md) (Общие сведения о фильтрации событий для подписок на Сетку событий).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="filter-by-event-type"></a>Фильтрация по типу события

При создании подписки на Сетку событий можно указать, какие [типы событий](event-schema.md) будут отправляться в конечную точку. В приведенных в этом разделе примерах создаются подписки на события для группы ресурсов, но в `Microsoft.Resources.ResourceWriteFailure` и `Microsoft.Resources.ResourceWriteSuccess` отправляется ограниченное количество событий. Если вам нужно больше возможностей фильтрации событий по типу, см. сведения о фильтрации по дополнительным операторам и полям данных.

В PowerShell при создании подписки используйте параметр `-IncludedEventType`.

```powershell
$includedEventTypes = "Microsoft.Resources.ResourceWriteFailure", "Microsoft.Resources.ResourceWriteSuccess"

New-AzEventGridSubscription `
  -EventSubscriptionName demoSubToResourceGroup `
  -ResourceGroupName myResourceGroup `
  -Endpoint <endpoint-URL> `
  -IncludedEventType $includedEventTypes
```

В Azure CLI используйте параметр `--included-event-types`. В приведенном ниже примере используется Azure CLI в оболочке bash.

```azurecli
includedEventTypes="Microsoft.Resources.ResourceWriteFailure Microsoft.Resources.ResourceWriteSuccess"

az eventgrid event-subscription create \
  --name demoSubToResourceGroup \
  --resource-group myResourceGroup \
  --endpoint <endpoint-URL> \
  --included-event-types $includedEventTypes
```

В шаблоне Resource Manager используйте свойство `includedEventTypes`.

```json
"resources": [
  {
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "name": "[parameters('eventSubName')]",
    "apiVersion": "2018-09-15-preview",
    "properties": {
      "destination": {
        "endpointType": "WebHook",
        "properties": {
          "endpointUrl": "[parameters('endpoint')]"
        }
      },
      "filter": {
        "subjectBeginsWith": "",
        "subjectEndsWith": "",
        "isSubjectCaseSensitive": false,
        "includedEventTypes": [
          "Microsoft.Resources.ResourceWriteFailure",
          "Microsoft.Resources.ResourceWriteSuccess"
        ]
      }
    }
  }
]
```

## <a name="filter-by-subject"></a>Фильтрация по теме

Вы можете фильтровать события по теме данных. Можно указать значение, которое сопоставляется с началом или концом темы. Если вам нужно больше возможностей фильтрации событий по теме, см. сведения о фильтрации по дополнительным операторам и полям данных.

В приведенном ниже примере PowerShell создается подписка, которая позволяет фильтровать события по началу темы. Чтобы ограничить события, предназначенные для конкретного ресурса, используйте параметр `-SubjectBeginsWith`. Можно передать идентификатор ресурса группы безопасности сети.

```powershell
$resourceId = (Get-AzResource -ResourceName demoSecurityGroup -ResourceGroupName myResourceGroup).ResourceId

New-AzEventGridSubscription `
  -Endpoint <endpoint-URL> `
  -EventSubscriptionName demoSubscriptionToResourceGroup `
  -ResourceGroupName myResourceGroup `
  -SubjectBeginsWith $resourceId
```

В следующем примере PowerShell создается подписка для хранилища BLOB-объектов. Она ограничивает события темой, которая заканчивается на `.jpg`.

```powershell
$storageId = (Get-AzStorageAccount -ResourceGroupName myResourceGroup -AccountName $storageName).Id

New-AzEventGridSubscription `
  -EventSubscriptionName demoSubToStorage `
  -Endpoint <endpoint-URL> `
  -ResourceId $storageId `
  -SubjectEndsWith ".jpg"
```

В приведенном ниже примере Azure CLI создается подписка, которая позволяет фильтровать события по началу темы. Чтобы ограничить события, предназначенные для конкретного ресурса, используйте параметр `--subject-begins-with`. Можно передать идентификатор ресурса группы безопасности сети.

```azurecli
resourceId=$(az resource show --name demoSecurityGroup --resource-group myResourceGroup --resource-type Microsoft.Network/networkSecurityGroups --query id --output tsv)

az eventgrid event-subscription create \
  --name demoSubscriptionToResourceGroup \
  --resource-group myResourceGroup \
  --endpoint <endpoint-URL> \
  --subject-begins-with $resourceId
```

В следующем примере Azure CLI создается подписка для хранилища BLOB-объектов. Она ограничивает события темой, которая заканчивается на `.jpg`.

```azurecli
storageid=$(az storage account show --name $storageName --resource-group myResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --resource-id $storageid \
  --name demoSubToStorage \
  --endpoint <endpoint-URL> \
  --subject-ends-with ".jpg"
```

В приведенном ниже примере шаблона Resource Manager создается подписка, которая позволяет фильтровать события по началу темы. Чтобы ограничить события, предназначенные для конкретного ресурса, используйте свойство `subjectBeginsWith`. Можно передать идентификатор ресурса группы безопасности сети.

```json
"resources": [
  {
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "name": "[parameters('eventSubName')]",
    "apiVersion": "2018-09-15-preview",
    "properties": {
      "destination": {
        "endpointType": "WebHook",
        "properties": {
          "endpointUrl": "[parameters('endpoint')]"
        }
      },
      "filter": {
        "subjectBeginsWith": "[resourceId('Microsoft.Network/networkSecurityGroups','demoSecurityGroup')]",
        "subjectEndsWith": "",
        "isSubjectCaseSensitive": false,
        "includedEventTypes": [ "All" ]
      }
    }
  }
]
```

В следующем примере шаблона Resource Manager создается подписка для хранилища BLOB-объектов. Она ограничивает события темой, которая заканчивается на `.jpg`.

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts/providers/eventSubscriptions",
    "name": "[concat(parameters('storageName'), '/Microsoft.EventGrid/', parameters('eventSubName'))]",
    "apiVersion": "2018-09-15-preview",
    "properties": {
      "destination": {
        "endpointType": "WebHook",
        "properties": {
          "endpointUrl": "[parameters('endpoint')]"
        }
      },
      "filter": {
        "subjectEndsWith": ".jpg",
        "subjectBeginsWith": "",
        "isSubjectCaseSensitive": false,
        "includedEventTypes": [ "All" ]
      }
    }
  }
]
```

## <a name="filter-by-operators-and-data"></a>Фильтрация по операторам и данным

Для большей гибкости при фильтрации используйте операторы и свойства данных для фильтрации событий.

### <a name="subscribe-with-advanced-filters"></a>Подписка с дополнительными фильтрами

Дополнительные сведения об операторах и ключах для расширенной фильтрации см. в разделе [Advanced filtering](event-filtering.md#advanced-filtering) (Расширенная фильтрация).

В этих примерах создается пользовательский раздел. Затем формируется подписка на этот раздел и выполняется фильтрация по значению в объекте данных. События со свойством, выделенным синим, красным или зеленым цветом, отправляются в подписку.

Для интерфейса командной строки Azure:

```azurecli
topicName=<your-topic-name>
endpointURL=<endpoint-URL>

az group create -n gridResourceGroup -l eastus2
az eventgrid topic create --name $topicName -l eastus2 -g gridResourceGroup

topicid=$(az eventgrid topic show --name $topicName -g gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  -n demoAdvancedSub \
  --advanced-filter data.color stringin blue red green \
  --endpoint $endpointURL \
  --expiration-date "<yyyy-mm-dd>"
```

Обратите внимание, что задана [дата окончания срока действия](concepts.md#event-subscription-expiration) подписки.

Для PowerShell используйте команду:

```powershell
$topicName = <your-topic-name>
$endpointURL = <endpoint-URL>

New-AzResourceGroup -Name gridResourceGroup -Location eastus2
New-AzEventGridTopic -ResourceGroupName gridResourceGroup -Location eastus2 -Name $topicName

$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name $topicName).Id

$expDate = '<mm/dd/yyyy hh:mm:ss>' | Get-Date
$AdvFilter1=@{operator="StringIn"; key="Data.color"; Values=@('blue', 'red', 'green')}

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint $endpointURL `
  -ExpirationDate $expDate `
  -AdvancedFilter @($AdvFilter1)
```

### <a name="test-filter"></a>Тестирование фильтрации

Чтобы протестировать фильтрацию, отправьте событие, выделенное зеленым цветом. Так как зеленым цветом выделено одно из значений в фильтре, событие отправляется в конечную точку.

Для интерфейса командной строки Azure:

```azurecli
topicEndpoint=$(az eventgrid topic show --name $topicName -g gridResourceGroup --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name $topicName -g gridResourceGroup --query "key1" --output tsv)

event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/cars", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "model": "SUV", "color": "green"},"dataVersion": "1.0"} ]'

curl -X POST -H "aeg-sas-key: $key" -d "$event" $topicEndpoint
```

Для PowerShell используйте команду:

```powershell
$endpoint = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name $topicName).Endpoint
$keys = Get-AzEventGridTopicKey -ResourceGroupName gridResourceGroup -Name $topicName

$eventID = Get-Random 99999
$eventDate = Get-Date -Format s

$htbody = @{
    id= $eventID
    eventType="recordInserted"
    subject="myapp/vehicles/cars"
    eventTime= $eventDate
    data= @{
        model="SUV"
        color="green"
    }
    dataVersion="1.0"
}

$body = "["+(ConvertTo-Json $htbody)+"]"

Invoke-WebRequest -Uri $endpoint -Method POST -Body $body -Headers @{"aeg-sas-key" = $keys.Key1}
```

Чтобы протестировать сценарий, когда событие не отправлено, отправьте событие, выделенное желтым цветом. Среди допустимых значений подписки желтый цвет не указан, поэтому событие не будет отправлено.

Для интерфейса командной строки Azure:

```azurecli
event='[ {"id": "'"$RANDOM"'", "eventType": "recordInserted", "subject": "myapp/vehicles/cars", "eventTime": "'`date +%Y-%m-%dT%H:%M:%S%z`'", "data":{ "model": "SUV", "color": "yellow"},"dataVersion": "1.0"} ]'

curl -X POST -H "aeg-sas-key: $key" -d "$event" $topicEndpoint
```
Для PowerShell используйте команду:

```powershell
$htbody = @{
    id= $eventID
    eventType="recordInserted"
    subject="myapp/vehicles/cars"
    eventTime= $eventDate
    data= @{
        model="SUV"
        color="yellow"
    }
    dataVersion="1.0"
}

$body = "["+(ConvertTo-Json $htbody)+"]"

Invoke-WebRequest -Uri $endpoint -Method POST -Body $body -Headers @{"aeg-sas-key" = $keys.Key1}
```

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со сведениями о [мониторинге доставки сообщений в службе "Сетка событий"](monitor-event-delivery.md).
* Дополнительные сведения о ключе аутентификации см. в статье [Сетка событий: безопасность и проверка подлинности](security-authentication.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).
