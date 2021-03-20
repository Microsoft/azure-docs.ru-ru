---
title: Получение событий из Сетки событий Azure в конечную точку HTTP
description: Описано, как проверить конечную точку HTTP, затем проверить и десериализировать события из Сетки событий Azure
ms.topic: conceptual
ms.date: 11/19/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 75c80fb85d39298f1130537971bc700897c039d0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96023735"
---
# <a name="receive-events-to-an-http-endpoint"></a>Получение событий через конечную точку HTTP

В этой статье описано, как [проверить конечную точку HTTP](webhook-event-delivery.md), чтобы получать события из подписки событий, а затем получать события и десериализировать их. В этой статье в целях демонстрации используется служба "Функции Azure", однако те же принципы применяются независимо от того, где размещается приложение.

> [!NOTE]
> При активации службы "Функции Azure" с помощью Сетки событий **настоятельно** рекомендуется использовать [триггер Сетки событий](../azure-functions/functions-bindings-event-grid.md). Универсальный триггер веб-перехватчика используется для демонстрации.

## <a name="prerequisites"></a>Предварительные условия

Вам потребуется приложение-функция с функцией, активируемой HTTP.

## <a name="add-dependencies"></a>Добавление зависимостей

Если вы разрабатываете в .NET, [добавьте зависимость](../azure-functions/functions-reference-csharp.md#referencing-custom-assemblies) в функцию для `Microsoft.Azure.EventGrid` [пакета NuGet](https://www.nuget.org/packages/Microsoft.Azure.EventGrid). Для работы примеров в этой статье требуется версия 1.4.0 или более поздняя.

Пакеты SDK для других языков доступны по ссылке на [общедоступные пакеты SDK](./sdk-overview.md#data-plane-sdks). Эти пакеты содержат модели для собственных типов событий, таких как `EventGridEvent`, `StorageBlobCreatedEventData` и `EventHubCaptureFileCreatedEventData`.

Щелкните ссылку просмотра файлов в службе "Функции Azure" (правая область на портале службы "Функции Azure") и создайте файл с именем project.json. Добавьте следующее содержимое в файл `project.json` и сохраните его:

 ```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.Azure.EventGrid": "2.0.0"
      }
    }
   }
}
```

![Добавленный пакет NuGet](./media/receive-events/add-dependencies.png)

## <a name="endpoint-validation"></a>Проверка конечной точки

Сначала нужно обработать событие `Microsoft.EventGrid.SubscriptionValidationEvent`. Каждый раз, когда пользователь подписывается на событие, служба "Сетка событий" отправляет событие проверки в конечную точку с `validationCode` в полезных данных. Конечная точка необходима для передачи этих данных обратно в тело ответа, чтобы [подтвердить, что конечная точка допустима и вы являетесь ее владельцем](webhook-event-delivery.md). При использовании [триггера Сетки событий](../azure-functions/functions-bindings-event-grid.md) вместо функции, активируемой веб-перехватчиком, проверка конечной точки выполняется автоматически. При использовании службы API сторонних производителей (например, [Zapier](https://zapier.com/home) или [IFTTT](https://ifttt.com/)), возможно, не удастся вывести на экран код проверки программными средствами. Для этих служб можно вручную проверить подписку с помощью URL-адреса проверки, который отправляется в событии проверки подписки. Скопируйте этот URL-адрес в свойство `validationUrl` и отправьте запрос GET через клиент REST или веб-браузер.

В C# функция `DeserializeEventGridEvents()` десериализует события в сетке. Он десериализует данные события в соответствующий тип, например StorageBlobCreatedEventData. Используйте класс `Microsoft.Azure.EventGrid.EventTypes` для получения поддерживаемых типов и имен событий.

Чтобы вывести на экран код проверки программными средствами, используйте следующий код: Вы можете найти связанные образцы в [примере объекта-получателя Сетки событий](https://github.com/Azure-Samples/event-grid-dotnet-publish-consume-events/tree/master/EventGridConsumer).

```csharp
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.EventGrid;
namespace Function1
{
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");
            string response = string.Empty;
            string requestContent = await new StreamReader(req.Body).ReadToEndAsync();
            log.LogInformation($"Received events: {requestContent}");

            EventGridSubscriber eventGridSubscriber = new EventGridSubscriber();

            EventGridEvent[] eventGridEvents = eventGridSubscriber.DeserializeEventGridEvents(requestContent);

            foreach (EventGridEvent eventGridEvent in eventGridEvents)
            {
                if (eventGridEvent.Data is SubscriptionValidationEventData)
                {
                    var eventData = (SubscriptionValidationEventData)eventGridEvent.Data;
                    log.LogInformation($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");
                    // Do any additional validation (as required) and then return back the below response

                    var responseData = new SubscriptionValidationResponse()
                    {
                        ValidationResponse = eventData.ValidationCode
                    };
                    return new OkObjectResult(responseData);
                }
            }
            return new OkObjectResult(response);
        }
   }
}
```

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }
    }
    context.done();
};
```

### <a name="test-validation-response"></a>Проверка тестового ответа

Проверьте функцию ответа проверки, вставив в примере событие в поле проверки функции:

```json
[{
  "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "subject": "",
  "data": {
    "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
  },
  "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
  "eventTime": "2018-01-25T22:12:19.4556811Z",
  "metadataVersion": "1",
  "dataVersion": "1"
}]
```

При нажатии кнопки запуска выходные данные должны иметь значение "200 — OK" и `{"validationResponse":"512d38b6-c7b8-40c8-89fe-f46f9e9622b6"}` в теле:

:::image type="content" source="./media/receive-events/validation-request.png" alt-text="Запрос на проверку":::

:::image type="content" source="./media/receive-events/validation-output.png" alt-text="Выходные данные проверки":::

## <a name="handle-blob-storage-events"></a>Обработка событий хранилища BLOB-объектов

Теперь можно расширить функцию для обработки `Microsoft.Storage.BlobCreated`:

```cs
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.EventGrid;
using Microsoft.Azure.EventGrid.Models;

namespace FunctionApp1
{
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation($"C# HTTP trigger function begun");
            string response = string.Empty;

            string requestContent = await new StreamReader(req.Body).ReadToEndAsync();
            log.LogInformation($"Received events: {requestContent}");

            EventGridSubscriber eventGridSubscriber = new EventGridSubscriber();

            EventGridEvent[] eventGridEvents = eventGridSubscriber.DeserializeEventGridEvents(requestContent);

            foreach (EventGridEvent eventGridEvent in eventGridEvents)
            {
                if (eventGridEvent.Data is SubscriptionValidationEventData)
                {
                    var eventData = (SubscriptionValidationEventData)eventGridEvent.Data;
                    log.LogInformation($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");
                    // Do any additional validation (as required) and then return back the below response

                    var responseData = new SubscriptionValidationResponse()
                    {
                        ValidationResponse = eventData.ValidationCode
                    };

                    return new OkObjectResult(responseData);
                }
            }

            return new OkObjectResult(response);
        }
    }
}
```

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";
    var storageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type  
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }

        else if (body.data && body.eventType == storageBlobCreatedEvent) {
            var blobCreatedEventData = body.data;
            context.log("Relaying received blob created event payload:" + JSON.stringify(blobCreatedEventData));
        }
    }
    context.done();
};

```

### <a name="test-blob-created-event-handling"></a>Обработка события создания тестового большого двоичного объекта

Проверьте новую возможность функции, поместив [событие хранилища BLOB-объектов](./event-schema-blob-storage.md#example-event) в поле тестирования и выполнив следующий код:

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/Storage/providers/Microsoft.Storage/storageAccounts/xstoretestaccount",
  "subject": "/blobServices/default/containers/testcontainer/blobs/testfile.txt",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2017-06-26T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "0x8D4BCC2E4835CD0",
    "contentType": "text/plain",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://example.blob.core.windows.net/testcontainer/testfile.txt",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

В журнале функции отобразятся выходные данные URL-адреса большого двоичного объекта:

![Журнал выходных данных](./media/receive-events/blob-event-response.png)

Кроме того, вы можете протестировать эти выходные данные, создав учетную запись хранилища BLOB-объектов или учетную запись хранения общего назначения версии 2 (GPv2), [добавив подписку события](../storage/blobs/storage-blob-event-quickstart.md) и задав для конечной точки URL-адрес функции:

![URL-адрес функции](./media/receive-events/function-url.png)

## <a name="handle-custom-events"></a>Обработка пользовательских событий

Наконец, давайте расширим функцию еще раз, чтобы она могла обрабатывать и пользовательские события. 

В C# пакет SDK поддерживает сопоставление типа имени с типом данных события. Используйте функцию `AddOrUpdateCustomEventMapping()` для сопоставления настраиваемого события.

Установите флажок для события `Contoso.Items.ItemReceived`. Итоговый код должен выглядеть следующим образом:

```cs
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.EventGrid;
using Newtonsoft.Json;

namespace FunctionApp1
{
    class ContosoItemReceivedEventData
    {
        [JsonProperty("itemSku")]
        public string ItemSku { get; set; }
    }
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation($"C# HTTP trigger function begun");
            string response = string.Empty;

            string requestContent = await new StreamReader(req.Body).ReadToEndAsync();
            log.LogInformation($"Received events: {requestContent}");

            EventGridSubscriber eventGridSubscriber = new EventGridSubscriber();
            eventGridSubscriber.AddOrUpdateCustomEventMapping("Contoso.Items.ItemReceived", typeof(ContosoItemReceivedEventData));
            EventGridEvent[] eventGridEvents = eventGridSubscriber.DeserializeEventGridEvents(requestContent);

            foreach (EventGridEvent eventGridEvent in eventGridEvents)
            {
                if (eventGridEvent.Data is SubscriptionValidationEventData)
                {
                    var eventData = (SubscriptionValidationEventData)eventGridEvent.Data;
                    log.LogInformation($"Got SubscriptionValidation event data, validation code: {eventData.ValidationCode}, topic: {eventGridEvent.Topic}");
                    // Do any additional validation (as required) and then return back the below response

                    var responseData = new SubscriptionValidationResponse()
                    {
                        ValidationResponse = eventData.ValidationCode
                    };

                    return new OkObjectResult(responseData);
                }
                else if (eventGridEvent.Data is StorageBlobCreatedEventData)
                {
                    var eventData = (StorageBlobCreatedEventData)eventGridEvent.Data;
                    log.LogInformation($"Got BlobCreated event data, blob URI {eventData.Url}");
                }
                else if (eventGridEvent.Data is ContosoItemReceivedEventData)
                {
                    var eventData = (ContosoItemReceivedEventData)eventGridEvent.Data;
                    log.LogInformation($"Got ContosoItemReceived event data, item SKU {eventData.ItemSku}");
                }
            }

            return new OkObjectResult(response);
        }
    }
}
```

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function begun');
    var validationEventType = "Microsoft.EventGrid.SubscriptionValidationEvent";
    var storageBlobCreatedEvent = "Microsoft.Storage.BlobCreated";
    var customEventType = "Contoso.Items.ItemReceived";

    for (var events in req.body) {
        var body = req.body[events];
        // Deserialize the event data into the appropriate type based on event type
        if (body.data && body.eventType == validationEventType) {
            context.log("Got SubscriptionValidation event data, validation code: " + body.data.validationCode + " topic: " + body.topic);

            // Do any additional validation (as required) and then return back the below response
            var code = body.data.validationCode;
            context.res = { status: 200, body: { "ValidationResponse": code } };
        }

        else if (body.data && body.eventType == storageBlobCreatedEvent) {
            var blobCreatedEventData = body.data;
            context.log("Relaying received blob created event payload:" + JSON.stringify(blobCreatedEventData));
        }

        else if (body.data && body.eventType == customEventType) {
            var payload = body.data;
            context.log("Relaying received custom payload:" + JSON.stringify(payload));
        }
    }
    context.done();
};
```

### <a name="test-custom-event-handling"></a>Тестирование обработки пользовательских событий

Наконец, проверьте, может ли функция теперь обрабатывать ваш тип пользовательского события:

```json
[{
    "subject": "Contoso/foo/bar/items",
    "eventType": "Contoso.Items.ItemReceived",
    "eventTime": "2017-08-16T01:57:26.005121Z",
    "id": "602a88ef-0001-00e6-1233-1646070610ea",
    "data": { 
            "itemSku": "Standard"
            },
    "dataVersion": "",
    "metadataVersion": "1"
}]
```

Кроме того, эту функцию можно протестировать в реальном времени, [отправив пользовательское событие с помощью CURL с портала](./custom-event-quickstart-portal.md) или [опубликовав в пользовательском разделе](./post-to-custom-topic.md), используя любую службу или приложение, которое может выполнять оператор POST в конечной точке, например [Postman](https://www.getpostman.com/). Создайте пользовательский раздел и подписку событий с помощью конечной точки, заданной в качестве URL-адреса функции.

[!INCLUDE [event-grid-message-headers](../../includes/event-grid-message-headers.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со статьей [Пакеты SDK для управления службой "Сетка событий Azure" и публикации в ней](./sdk-overview.md).
* Дополнительные сведения см. в статье [Публикация в пользовательском разделе для службы "Сетка событий Azure"](./post-to-custom-topic.md).
* Попробуйте выполнить инструкции одного из подробных руководств по службе "Сетка событий" и "Функции", например [Автоматическое изменение размера переданных изображений с помощью службы "Сетка событий"](resize-images-on-storage-blob-upload-event.md).
