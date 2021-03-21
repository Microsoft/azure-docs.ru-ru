---
title: Выходная привязка Сетки событий Azure для функций Azure
description: Узнайте, как отправить событие Сетки событий в функции Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: devx-track-csharp, fasttrack-edit, devx-track-python
ms.openlocfilehash: 888afdc2764fed9f0b2c8b548c3e2b1c48e9a31e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97094682"
---
# <a name="azure-event-grid-output-binding-for-azure-functions"></a>Выходная привязка Сетки событий Azure для функций Azure

Используйте выходную привязку Сетки событий для записи событий в пользовательский раздел. Необходимо иметь допустимый [ключ доступа для пользовательского раздела](../event-grid/security-authenticate-publishing-clients.md).

Сведения об установке и настройке см. в [обзорной статье](./functions-bindings-event-grid.md).

> [!NOTE]
> Выходная привязка Сетки событий не поддерживает подписанные URL-адреса (маркеры SAS). Необходимо использовать ключ доступа раздела.

> [!IMPORTANT]
> Выходная привязка Сетки событий доступна только для функций 2.x и более поздних версий.

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая записывает сообщение в пользовательский раздел Сетки событий, используя возвращаемое значение метода в качестве выходных данных:

```csharp
[FunctionName("EventGridOutput")]
[return: EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
public static EventGridEvent Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    return new EventGridEvent("message-id", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
}
```

В следующем примере показано, как использовать интерфейс `IAsyncCollector` для отправки пакета сообщений:

```csharp
[FunctionName("EventGridAsyncOutput")]
public static async Task Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer,
    [EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]IAsyncCollector<EventGridEvent> outputEvents,
    ILogger log)
{
    for (var i = 0; i < 3; i++)
    {
        var myEvent = new EventGridEvent("message-id-" + i, "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
        await outputEvents.AddAsync(myEvent);
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В приведенных ниже примерах показаны данные выходной привязки Сетки событий в файле *function.json*.

```json
{
    "type": "eventGrid",
    "name": "outputEvent",
    "topicEndpointUri": "MyEventGridTopicUriSetting",
    "topicKeySetting": "MyEventGridTopicKeySetting",
    "direction": "out"
}
```

Ниже приведен код скрипта C#, который создает одно событие:

```cs
#r "Microsoft.Azure.EventGrid"
using System;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, out EventGridEvent outputEvent, ILogger log)
{
    outputEvent = new EventGridEvent("message-id", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
}
```

Ниже приведен код скрипта C#, который создает несколько событий:

```cs
#r "Microsoft.Azure.EventGrid"
using System;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, ICollector<EventGridEvent> outputEvent, ILogger log)
{
    outputEvent.Add(new EventGridEvent("message-id-1", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0"));
    outputEvent.Add(new EventGridEvent("message-id-2", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0"));
}
```

# <a name="java"></a>[Java](#tab/java)

Выходная привязка Сетки событий недоступна для Java.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В приведенных ниже примерах показаны данные выходной привязки Сетки событий в файле *function.json*.

```json
{
    "type": "eventGrid",
    "name": "outputEvent",
    "topicEndpointUri": "MyEventGridTopicUriSetting",
    "topicKeySetting": "MyEventGridTopicKeySetting",
    "direction": "out"
}
```

Ниже приведен код скрипта JavaScript, который создает одно событие:

```javascript
module.exports = async function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    context.bindings.outputEvent = {
        id: 'message-id',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    };
    context.done();
};
```

Ниже приведен код скрипта JavaScript, который создает несколько событий:

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();

    context.bindings.outputEvent = [];

    context.bindings.outputEvent.push({
        id: 'message-id-1',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    });
    context.bindings.outputEvent.push({
        id: 'message-id-2',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    });
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показано, как настроить функцию для вывода сообщения о событии сетки событий. В разделе, где `type` задается `eventGrid` Настройка значений, необходимых для создания выходной привязки сетки событий.

```powershell
{
  "bindings": [
    {
      "type": "eventGrid",
      "name": "outputEvent",
      "topicEndpointUri": "MyEventGridTopicUriSetting",
      "topicKeySetting": "MyEventGridTopicKeySetting",
      "direction": "out"
    },
    {
      "authLevel": "anonymous",
      "type": "httpTrigger",
      "direction": "in",
      "name": "Request",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "Response"
    }
  ]
}
```

В функции используйте `Push-OutputBinding` для отправки события в пользовательский раздел через выходную привязку сетки событий.

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$message = $Request.Query.Message

Push-OutputBinding -Name outputEvent -Value  @{
    id = "1"
    EventType = "testEvent"
    Subject = "testapp/testPublish"
    EventTime = "2020-08-27T21:03:07+00:00"
    Data = @{
        Message = $message
    }
    DataVersion = "1.0"
}

Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = 200
    Body = "OK"
})
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показаны привязка триггера в файле *function.json* и [функция Python](functions-reference-python.md), использующая эту привязку. Затем он отправляет событие в пользовательский раздел, как указано в `topicEndpointUri` .

Данные привязки в файле *function.json*:

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    },
    {
      "type": "eventGrid",
      "name": "outputEvent",
      "topicEndpointUri": "MyEventGridTopicUriSetting",
      "topicKeySetting": "MyEventGridTopicKeySetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Ниже приведен пример Python для отправки события в пользовательский раздел путем установки `EventGridOutputEvent` :

```python
import logging
import azure.functions as func
import datetime

def main(eventGridEvent: func.EventGridEvent, 
         outputEvent: func.Out[func.EventGridOutputEvent]) -> None:

    logging.log("eventGridEvent: ", eventGridEvent)

    outputEvent.set(
        func.EventGridOutputEvent(
            id="test-id",
            data={"tag1": "value1", "tag2": "value2"},
            subject="test-subject",
            event_type="test-event-1",
            event_time=datetime.datetime.utcnow(),
            data_version="1.0"))
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте атрибут [EventGridAttribute](https://github.com/Azure/azure-functions-eventgrid-extension/blob/dev/src/EventGridExtension/OutputBinding/EventGridAttribute.cs).

Конструктор атрибута принимает имя параметра приложения, содержащего имя настраиваемого раздела, и имя параметра приложения, содержащего ключ раздела. Дополнительные сведения об этих параметрах см. в разделе [Привязки концентраторов событий функций Azure](#configuration). Ниже приведен пример атрибута `EventGrid`.

```csharp
[FunctionName("EventGridOutput")]
[return: EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    ...
}
```

Полный пример см. [здесь](#example).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

Выходная привязка Сетки событий недоступна для Java.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

Выходная привязка Сетки событий недоступна для Python.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `EventGrid`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Для этого свойства необходимо задать значение "eventGrid" |
|**direction** | Недоступно | Для этого свойства необходимо задать значение out. Этот параметр задается автоматически при создании привязки на портале Azure. |
|**name** | Недоступно | Имя переменной, используемое в коде функции, которая представляет событие. |
|**topicEndpointUri** |**TopicEndpointUri** | Имя параметра приложения, содержащего URI для пользовательского раздела, например `MyTopicEndpointUri`. |
|**topicKeySetting** |**TopicKeySetting** | Имя параметра приложения, содержащего ключ доступа для пользовательского раздела. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> Убедитесь, что в качестве значения свойства конфигурации `TopicEndpointUri` задано имя параметра приложения, содержащего URI пользовательского раздела. Не указывайте URI пользовательского раздела непосредственно в этом свойстве.

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

Для отправки сообщений используйте параметр метода, например `out EventGridEvent paramName`. Для записи нескольких сообщений можно использовать `ICollector<EventGridEvent>` или `IAsyncCollector<EventGridEvent>` вместо `out EventGridEvent`.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

Для отправки сообщений используйте параметр метода, например `out EventGridEvent paramName`. В скрипте C# `paramName` — это значение, заданное в свойстве `name` файла *function.json*. Для записи нескольких сообщений можно использовать `ICollector<EventGridEvent>` или `IAsyncCollector<EventGridEvent>` вместо `out EventGridEvent`.

# <a name="java"></a>[Java](#tab/java)

Выходная привязка Сетки событий недоступна для Java.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Доступ к выходному событию с помощью `context.bindings.<name>`, где `<name>` — это значение, указанное в свойстве `name` файла *function.json*.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Получите доступ к выходному событию с помощью `Push-OutputBinding` командлет, чтобы отправить событие в выходную привязку сетки событий.

# <a name="python"></a>[Python](#tab/python)

Выходная привязка Сетки событий недоступна для Python.

---

## <a name="next-steps"></a>Дальнейшие действия

* [Отправка события Сетки событий](./functions-bindings-event-grid-trigger.md)
