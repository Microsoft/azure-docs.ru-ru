---
author: craigshoemaker
ms.service: azure-functions
ms.topic: include
ms.date: 02/21/2020
ms.author: cshoe
ms.openlocfilehash: bc2bec364f8d752b7416ecccf0b00d0fbec4c8e8
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105729874"
---
Используйте выходную привязку Центров событий для записи событий в поток событий. Чтобы записывать события в центр событий, необходимо иметь разрешение на отправку в него событий.

Прежде чем реализовывать привязку к выходным данным, убедитесь, что указаны ссылки на требуемые пакеты.

<a id="example" name="example"></a>

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](../articles/azure-functions/functions-dotnet-class-library.md), которая записывает сообщение в концентратор событий, используя возвращаемое значение метода в качестве выходных данных.

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    return $"{DateTime.Now}";
}
```

В следующем примере показано, как использовать интерфейс `IAsyncCollector` для отправки пакета сообщений: Этот сценарий характерен, если вы обрабатываете сообщения, поступающие из одного концентратора событий, и отправляете результат в другой концентратор событий.

```csharp
[FunctionName("EH2EH")]
public static async Task Run(
    [EventHubTrigger("source", Connection = "EventHubConnectionAppSetting")] EventData[] events,
    [EventHub("dest", Connection = "EventHubConnectionAppSetting")]IAsyncCollector<string> outputEvents,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        // do some processing:
        var myProcessedEvent = DoSomething(eventData);

        // then send the message
        await outputEvents.AddAsync(JsonConvert.SerializeObject(myProcessedEvent));
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показаны привязка триггера концентратора событий в файле *function.json* и [функция сценария C#](../articles/azure-functions/functions-reference-csharp.md), которая использует эту привязку. Эта функция записывает сообщение в концентратор событий.

В приведенных ниже примерах показаны данные привязки Центров событий в файле *function.json*. В первом примере приведены данные для службы "Функции" версии 2.x и более поздних, а во втором — для версии 1.x. 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Ниже приведен код сценария C#, который создает одно сообщение.

```cs
using System;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, out string outputEventHubMessage, ILogger log)
{
    String msg = $"TimerTriggerCSharp1 executed at: {DateTime.Now}";
    log.LogInformation(msg);   
    outputEventHubMessage = msg;
}
```

Ниже приведен код сценария C#, который создает несколько сообщений.

```cs
public static void Run(TimerInfo myTimer, ICollector<string> outputEventHubMessage, ILogger log)
{
    string message = $"Message created at: {DateTime.Now}";
    log.LogInformation(message);
    outputEventHubMessage.Add("1 " + message);
    outputEventHubMessage.Add("2 " + message);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показаны привязка триггера концентратора событий в файле *function.json* и [функция JavaScript](../articles/azure-functions/functions-reference-node.md), которая использует эту привязку. Эта функция записывает сообщение в концентратор событий.

В приведенных ниже примерах показаны данные привязки Центров событий в файле *function.json*. В первом примере приведены данные для службы "Функции" версии 2.x и более поздних, а во втором — для версии 1.x. 

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

```json
{
    "type": "eventHub",
    "name": "outputEventHubMessage",
    "path": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Ниже приведен код JavaScript, который отправляет отдельное сообщение.

```javascript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();
    context.log('Message created at: ', timeStamp);   
    context.bindings.outputEventHubMessage = "Message created at: " + timeStamp;
    context.done();
};
```

Ниже приведен код JavaScript, который отправляет несколько сообщений.

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();
    var message = 'Message created at: ' + timeStamp;

    context.bindings.outputEventHubMessage = [];

    context.bindings.outputEventHubMessage.push("1 " + message);
    context.bindings.outputEventHubMessage.push("2 " + message);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показаны привязка триггера концентратора событий в файле *function.json* и [функция Python](../articles/azure-functions/functions-reference-python.md), которая использует эту привязку. Эта функция записывает сообщение в концентратор событий.

В приведенных ниже примерах показаны данные привязки Центров событий в файле *function.json*.

```json
{
    "type": "eventHub",
    "name": "$return",
    "eventHubName": "myeventhub",
    "connection": "MyEventHubSendAppSetting",
    "direction": "out"
}
```

Ниже приведен код Python, который отправляет отдельное сообщение.

```python
import datetime
import logging
import azure.functions as func


def main(timer: func.TimerRequest) -> str:
    timestamp = datetime.datetime.utcnow()
    logging.info('Message created at: %s', timestamp)
    return 'Message created at: {}'.format(timestamp)
```

# <a name="java"></a>[Java](#tab/java)

В следующем примере показана функция Java, которая записывает в концентратор событий сообщение, содержащее текущее время.

```java
@FunctionName("sendTime")
@EventHubOutput(name = "event", eventHubName = "samples-workitems", connection = "AzureEventHubConnection")
public String sendTime(
   @TimerTrigger(name = "sendTimeTrigger", schedule = "0 */5 * * * *") String timerInfo)  {
     return LocalDateTime.now().toString();
 }
```

В [библиотеке среды выполнения функций Java](/java/api/overview/azure/functions/runtime) используйте заметку `@EventHubOutput` для параметров, значения которых будут опубликованы в концентраторе событий.  Параметр должен быть типа `OutputBinding<T>`, где T — POJO или любой собственный тип Java.

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](../articles/azure-functions/functions-dotnet-class-library.md) используйте атрибут [EventHubAttribute](https://github.com/Azure/azure-functions-eventhubs-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.EventHubs/EventHubAttribute.cs).

Конструктор атрибута принимает имя концентратора событий и имя параметра приложения, содержащего строку подключения. Дополнительные сведения об этих параметрах см. в разделе [Привязки концентраторов событий функций Azure](#configuration). Ниже приведен пример атрибута `EventHub`.

```csharp
[FunctionName("EventHubOutput")]
[return: EventHub("outputEventHubMessage", Connection = "EventHubConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    ...
}
```

Полный пример см. в разделе [Пример выходных данных C#](#example).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

В [библиотеке среды выполнения функций Java](/java/api/overview/azure/functions/runtime) используйте аннотацию [EventHubOutput](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput) для параметров, значения которых будут опубликованы в концентраторе событий. Параметр должен быть типа `OutputBinding<T>`, где `T` — это POJO или любой собственный тип Java.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `EventHub`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Для этого свойства необходимо задать значение "eventHub" |
|**direction** | Недоступно | Для этого свойства необходимо задать значение out. Этот параметр задается автоматически при создании привязки на портале Azure. |
|**name** | Недоступно | Имя переменной, используемое в коде функции, которая представляет событие. |
|**путь** |**EventHubName** | Только служба "Функции" версии 1.x. Имя концентратора событий. Если имя концентратора событий указано также в строке подключения, такое значение переопределяет это свойство во время выполнения. |
|**eventHubName** |**EventHubName** | Функции 2.x и более поздних версий. Имя концентратора событий. Если имя концентратора событий указано также в строке подключения, такое значение переопределяет это свойство во время выполнения. |
|**connection**; |**Соединение** | Имя параметра приложения, содержащего строку подключения к пространству имен концентратора событий. Скопируйте эту строку подключения, нажав кнопку **Сведения о подключении** для [пространства имен](../articles/event-hubs/event-hubs-create.md#create-an-event-hubs-namespace), а не сам концентратор событий. Чтобы отправлять сообщения в поток событий, эта строка подключения должна иметь разрешения на отправку. <br><br>Если вы используете [расширение версии 5. x или более поздней версии](../articles/azure-functions/functions-bindings-event-hubs.md#event-hubs-extension-5x-and-higher), а не строку подключения, можно указать ссылку на раздел конфигурации, который определяет подключение. См. раздел [Подключения](../articles/azure-functions/functions-reference.md#connections).|

[!INCLUDE [app settings to local.settings.json](../articles/azure-functions/../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

### <a name="default"></a>По умолчанию

Для привязки к выходным данным концентратора событий можно использовать параметры следующих типов:

* `string`
* `byte[]`
* `POCO`
* `EventData` — свойства EventData по умолчанию предоставляются для [пространства имен Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs.eventdata).

Для отправки сообщений используйте параметр метода, например `out string paramName`. В скрипте C# `paramName` — это значение, заданное в свойстве `name` файла *function.json*. Для записи нескольких сообщений можно использовать `ICollector<string>` или `IAsyncCollector<string>` вместо `out string`.

### <a name="additional-types"></a>Дополнительные типы 
Приложения, использующие расширение Центра событий 5.0.0 или более поздней версии, используют тип `EventData` в пространстве имен [Azure.Messaging.EventHubs](/dotnet/api/azure.messaging.eventhubs.eventdata), а не [Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs.eventdata). В этой версии прекращена поддержка устаревшего типа `Body`. Вместо него теперь поддерживаются следующие типы:

- [EventBody](/dotnet/api/azure.messaging.eventhubs.eventdata.eventbody);

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

### <a name="default"></a>По умолчанию

Для привязки к выходным данным концентратора событий можно использовать параметры следующих типов:

* `string`
* `byte[]`
* `POCO`
* `EventData` — свойства EventData по умолчанию предоставляются для [пространства имен Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs.eventdata).

Для отправки сообщений используйте параметр метода, например `out string paramName`. В скрипте C# `paramName` — это значение, заданное в свойстве `name` файла *function.json*. Для записи нескольких сообщений можно использовать `ICollector<string>` или `IAsyncCollector<string>` вместо `out string`.

### <a name="additional-types"></a>Дополнительные типы 
Приложения, использующие расширение Центра событий 5.0.0 или более поздней версии, используют тип `EventData` в пространстве имен [Azure.Messaging.EventHubs](/dotnet/api/azure.messaging.eventhubs.eventdata), а не [Microsoft.Azure.EventHubs](/dotnet/api/microsoft.azure.eventhubs.eventdata). В этой версии прекращена поддержка устаревшего типа `Body`. Вместо него теперь поддерживаются следующие типы:

- [EventBody](/dotnet/api/azure.messaging.eventhubs.eventdata.eventbody);

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Доступ к выходному событию с помощью `context.bindings.<name>`, где `<name>` — это значение, указанное в свойстве `name` файла *function.json*.

# <a name="python"></a>[Python](#tab/python)

Существует два варианта для вывода сообщения концентратора событий из функции:

- **Возвращаемое значение** — задайте для свойства `name` в файле *function.json* значение `$return`. В этой конфигурации возвращаемое значение функции сохраняется как сообщение концентратора событий.

- **Императив** — передайте значение методу [set](/python/api/azure-functions/azure.functions.out#set-val--t-----none) параметра, объявленного с типом [Out](/python/api/azure-functions/azure.functions.out). Значение, переданное `set`, сохраняется как сообщение концентратора событий.

# <a name="java"></a>[Java](#tab/java)

Существует два варианта для вывода сообщения концентратора событий из функции с помощью заметки [EventHubOutput](/java/api/com.microsoft.azure.functions.annotation.eventhuboutput):

- **Возвращаемое значение** — вследствие применения аннотации к самой функции возвращаемое значение функции сохраняется как сообщение концентратора событий.

- **Императив** — чтобы явно задать значение сообщения, примените аннотацию к определенному параметру с типом [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.OutputBinding), где `T` — это POJO или любой собственный тип Java. В такой конфигурации передача значения методу `setValue` приводит к сохранению значения как сообщения концентратора событий.

---

## <a name="exceptions-and-return-codes"></a>Исключения и коды возврата

| Привязка | Справочник |
|---|---|
| Концентратор событий | [Руководство по операциям](/rest/api/eventhub/publisher-policy-operations) |