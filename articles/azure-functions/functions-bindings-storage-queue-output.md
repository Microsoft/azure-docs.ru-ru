---
title: Выходная привязка хранилища очередей Azure для функций Azure
description: Узнайте, как создавать сообщения хранилища очередей Azure в функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/18/2020
ms.author: cshoe
ms.custom: devx-track-csharp, cc996988-fb4f-47, devx-track-python
ms.openlocfilehash: 5d94625e3eb121e556b28038cf59626be1332966
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102455811"
---
# <a name="azure-queue-storage-output-bindings-for-azure-functions"></a>Выходные привязки хранилища очередей Azure для функций Azure

Функции Azure могут создавать новые сообщения в хранилище очередей Azure, настроив выходную привязку.

Сведения об установке и настройке см. в [этой обзорной статье](./functions-bindings-storage-queue.md).

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая создает сообщения очереди для каждого полученного HTTP-запроса.

```csharp
[StorageAccount("MyStorageConnectionAppSetting")]
public static class QueueFunctions
{
    [FunctionName("QueueOutput")]
    [return: Queue("myqueue-items")]
    public static string QueueOutput([HttpTrigger] dynamic input,  ILogger log)
    {
        log.LogInformation($"C# function processed: {input.Text}");
        return input.Text;
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показана привязка триггера HTTP в файле *function.json* и коде [сценария C# (CSX)](functions-reference-csharp.md), который использует привязку. Функция создает элемент очереди с полезными данными объекта **CustomQueueMessage** для каждого полученного HTTP-запроса.

Ниже показан файл *function.json*.

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function",
      "name": "input"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "$return",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже приведен код скрипта C#, который создает одно сообщение очереди.

```cs
public class CustomQueueMessage
{
    public string PersonName { get; set; }
    public string Title { get; set; }
}

public static CustomQueueMessage Run(CustomQueueMessage input, ILogger log)
{
    return input;
}
```

За один раз можно отправить несколько сообщений с помощью параметра `ICollector` или `IAsyncCollector`. Ниже приведен код скрипта C#, который отправляет несколько сообщений (одно — с данными HTTP-запроса, а другое — с кодовыми значениями).

```cs
public static void Run(
    CustomQueueMessage input, 
    ICollector<CustomQueueMessage> myQueueItems, 
    ILogger log)
{
    myQueueItems.Add(input);
    myQueueItems.Add(new CustomQueueMessage { PersonName = "You", Title = "None" });
}
```

# <a name="java"></a>[Java](#tab/java)

 В следующем примере показана функция Java, которая создает сообщение очереди для события, вызванного HTTP-запросом.

```java
@FunctionName("httpToQueue")
@QueueOutput(name = "item", queueName = "myqueue-items", connection = "MyStorageConnectionAppSetting")
 public String pushToQueue(
     @HttpTrigger(name = "request", methods = {HttpMethod.POST}, authLevel = AuthorizationLevel.ANONYMOUS)
     final String message,
     @HttpOutput(name = "response") final OutputBinding<String> result) {
       result.setValue(message + " has been added.");
       return message;
 }
```

В [библиотеке среды выполнения функций Java](/java/api/overview/azure/functions/runtime) используйте заметку `@QueueOutput` для параметров, значения которых будут записываться в хранилище очередей.  Тип параметра должен быть `OutputBinding<T>` , где `T` — любой собственный тип Java для POJO.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана привязка триггера HTTP в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует привязку. Эта функция создает элемент очереди для каждого полученного HTTP-запроса.

Ниже показан файл *function.json*.

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "authLevel": "function",
      "name": "input"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "myQueueItem",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже показан код JavaScript.

```javascript
module.exports = function (context, input) {
    context.bindings.myQueueItem = input.body;
    context.done();
};
```

Вы можете отправить несколько сообщений одновременно, определив массив сообщений для выходной привязки `myQueueItem`. Следующий код JavaScript отправляет два сообщения очереди с четко фиксированными значениями для каждого полученного HTTP-запроса.

```javascript
module.exports = function(context) {
    context.bindings.myQueueItem = ["message 1","message 2"];
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующих примерах кода показано, как вывести сообщение очереди из функции, активируемой HTTP. Раздел конфигурации с параметром `type` класса `queue` определяет выходную привязку.

```json
{
  "bindings": [
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
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "Msg",
      "queueName": "outqueue",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

С помощью этой конфигурации привязки функция PowerShell может создать сообщение очереди с помощью `Push-OutputBinding` . В этом примере создается сообщение на основе строки запроса или параметра Body.

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$message = $Request.Query.Message
Push-OutputBinding -Name Msg -Value $message
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = 200
    Body = "OK"
})
```

Чтобы отправить несколько сообщений одновременно, определите массив сообщений и используйте `Push-OutputBinding` для отправки сообщений в выходную привязку очереди.

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

# Write to the Azure Functions log stream.
Write-Host "PowerShell HTTP trigger function processed a request."

# Interact with query parameters or the body of the request.
$message = @("message1", "message2")
Push-OutputBinding -Name Msg -Value $message
Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
    StatusCode = 200
    Body = "OK"
})
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показано, как вывести одно и несколько значений в очереди хранилища. Настройка, необходимая для *function.js* , совпадает в любом случае.

Привязка очереди хранилища определяется в *function.js* , где *тип* имеет значение `queue` .

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": [
        "get",
        "post"
      ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "$return"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "msg",
      "queueName": "outqueue",
      "connection": "AzureStorageQueuesConnectionString"
    }
  ]
}
```

Чтобы задать отдельное сообщение в очереди, в метод необходимо передать одно значение `set` .

```python
import azure.functions as func

def main(req: func.HttpRequest, msg: func.Out[str]) -> func.HttpResponse:

    input_msg = req.params.get('message')

    msg.set(input_msg)

    return 'OK'
```

Чтобы создать несколько сообщений в очереди, объявите параметр в качестве соответствующего типа списка и передайте в метод массив значений (которые соответствуют типу списка) `set` .

```python
import azure.functions as func
import typing

def main(req: func.HttpRequest, msg: func.Out[typing.List[str]]) -> func.HttpResponse:

    msg.set(['one', 'two'])

    return 'OK'
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте [QueueAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Queues/QueueAttribute.cs).

Этот атрибут применяется к параметру `out` или возвращаемому значению функции. Конструктор атрибута принимает имя очереди, как показано в следующем примере:

```csharp
[FunctionName("QueueOutput")]
[return: Queue("myqueue-items")]
public static string Run([HttpTrigger] dynamic input,  ILogger log)
{
    ...
}
```

Чтобы указать учетную запись хранения, которую нужно использовать, можно задать свойство `Connection`, как показано в следующем примере.

```csharp
[FunctionName("QueueOutput")]
[return: Queue("myqueue-items", Connection = "StorageConnectionAppSetting")]
public static string Run([HttpTrigger] dynamic input,  ILogger log)
{
    ...
}
```

Полный пример см. в разделе [пример выходных данных](#example).

Чтобы указать учетную запись хранения на уровне класса, метода или параметра, можно использовать атрибут `StorageAccount`. Дополнительные сведения см. в разделе "Атрибуты триггера".

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

`QueueOutput`Заметка позволяет записывать сообщение в качестве выходных данных функции. В следующем примере показана функция, активируемая по протоколу HTTP, которая создает сообщение очереди.

```java
package com.function;
import java.util.*;
import com.microsoft.azure.functions.annotation.*;
import com.microsoft.azure.functions.*;

public class HttpTriggerQueueOutput {
    @FunctionName("HttpTriggerQueueOutput")
    public HttpResponseMessage run(
            @HttpTrigger(name = "req", methods = {HttpMethod.GET, HttpMethod.POST}, authLevel = AuthorizationLevel.FUNCTION) HttpRequestMessage<Optional<String>> request,
            @QueueOutput(name = "message", queueName = "messages", connection = "MyStorageConnectionAppSetting") OutputBinding<String> message,
            final ExecutionContext context) {

        message.setValue(request.getQueryParameters().get("name"));
        return request.createResponseBuilder(HttpStatus.OK).body("Done").build();
    }
}
```

| Свойство.    | Описание |
|-------------|-----------------------------|
|`name`       | Объявляет имя параметра в сигнатуре функции. При активации функции значение этого параметра будет иметь содержимое сообщения очереди. |
|`queueName`  | Объявляет имя очереди в учетной записи хранения. |
|`connection` | Указывает строку подключения к учетной записи хранения. |

Параметр, связанный с `QueueOutput` заметкой, типизирован как [экземпляр \<T\> аутпутбиндинг](https://github.com/Azure/azure-functions-java-library/blob/master/src/main/java/com/microsoft/azure/functions/OutputBinding.java) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `Queue`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Нужно задать значение `queue`. Это свойство задается автоматически при создании триггера на портале Azure.|
|**direction** | Недоступно | Нужно задать значение `out`. Это свойство задается автоматически при создании триггера на портале Azure. |
|**name** | Недоступно | Имя переменной, представляющей очередь в коде функции. Задайте значение `$return`, ссылающееся на возвращаемое значение функции.|
|**queueName** |**QueueName** | Имя очереди. |
|**connection**; | **Соединение** |Имя параметра приложения, содержащего строку подключения к службе хранилища, используемой для этой привязки. Если имя параметра приложения начинается с AzureWebJobs, можно указать только остальную часть имени.<br><br>Например, если задано значение `connection` "MyStorage", среда выполнения функций ищет параметр приложения с именем "MyStorage". Если оставить строку `connection` пустой, среда выполнения службы "Функции" будет использовать строку подключения к службе хранилища по умолчанию для параметра приложения с именем `AzureWebJobsStorage`.<br><br>Если используется [расширение версии 5. x или более поздней](./functions-bindings-storage-queue.md#storage-extension-5x-and-higher)версии, а не строка подключения, можно указать ссылку на раздел конфигурации, который определяет соединение. См. раздел [подключения](./functions-reference.md#connections).|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

### <a name="default"></a>По умолчанию

Напишите одно сообщение очереди с помощью параметра метода, например `out T paramName` . Вы можете использовать этот метод типа возвращаемого значения вместо параметра `out`. `T` может быть любого из следующих типов:

* Объект, сериализуемый как JSON
* `string`
* `byte[]`
* [CloudQueueMessage] 

Если при попытке привязать к `CloudQueueMessage` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](functions-bindings-storage-queue.md#azure-storage-sdk-version-in-functions-1x).

В коде и скрипте C# запишите несколько сообщений очереди с помощью одного из следующих типов: 

* `ICollector<T>` или `IAsyncCollector<T>`
* [CloudQueue](/dotnet/api/microsoft.azure.storage.queue.cloudqueue).

### <a name="additional-types"></a>Дополнительные типы

В приложениях, в которых используется[расширение службы хранилища версии 5.0.0 или выше](./functions-bindings-storage-queue.md#storage-extension-5x-and-higher), также могут использоваться типы из [пакета Azure SDK для .NET](/dotnet/api/overview/azure/storage.queues-readme). Эта версия отменяет поддержку устаревших `CloudQueue` `CloudQueueMessage` типов и в пользу следующих типов:

- [куеуемессаже](/dotnet/api/azure.storage.queues.models.queuemessage)
- [QueueClient](/dotnet/api/azure.storage.queues.queueclient) для записи нескольких сообщений в очереди

Примеры использования этих типов см. в разделе [репозитория GitHub для расширения](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Queues#examples).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

### <a name="default"></a>По умолчанию

Напишите одно сообщение очереди с помощью параметра метода, например `out T paramName` . `paramName`— Это значение, указанное в `name` свойстве *function.json*. Вы можете использовать этот метод типа возвращаемого значения вместо параметра `out`. `T` может быть любого из следующих типов:

* Объект, сериализуемый как JSON
* `string`
* `byte[]`
* [CloudQueueMessage] 

Если при попытке привязать к `CloudQueueMessage` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](functions-bindings-storage-queue.md#azure-storage-sdk-version-in-functions-1x).

В коде и скрипте C# запишите несколько сообщений очереди с помощью одного из следующих типов: 

* `ICollector<T>` или `IAsyncCollector<T>`
* [CloudQueue](/dotnet/api/microsoft.azure.storage.queue.cloudqueue).

### <a name="additional-types"></a>Дополнительные типы

В приложениях, в которых используется[расширение службы хранилища версии 5.0.0 или выше](./functions-bindings-storage-queue.md#storage-extension-5x-and-higher), также могут использоваться типы из [пакета Azure SDK для .NET](/dotnet/api/overview/azure/storage.queues-readme). Эта версия отменяет поддержку устаревших `CloudQueue` `CloudQueueMessage` типов и в пользу следующих типов:

- [куеуемессаже](/dotnet/api/azure.storage.queues.models.queuemessage)
- [QueueClient](/dotnet/api/azure.storage.queues.queueclient) для записи нескольких сообщений в очереди

Примеры использования этих типов см. в разделе [репозитория GitHub для расширения](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Queues#examples).

# <a name="java"></a>[Java](#tab/java)

Существует два варианта вывода сообщения очереди из функции с помощью аннотации [куеуеаутпут](/java/api/com.microsoft.azure.functions.annotation.queueoutput) :

- **Возвращаемое значение**: применяя заметку к самой функции, возвращаемое значение функции сохраняется как сообщение очереди.

- **Императив** — чтобы явно задать значение сообщения, примените аннотацию к определенному параметру с типом [`OutputBinding<T>`](/java/api/com.microsoft.azure.functions.outputbinding), где `T` — это POJO или любой собственный тип Java. При такой конфигурации передача значения `setValue` методу сохраняет значение в виде сообщения очереди.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Элемент очереди вывода доступен по адресу `context.bindings.<NAME>` , в котором `<NAME>` совпадает с именем, определенным в *function.json*. Строку или сериализуемый объект JSON можно использовать для полезных данных элемента очереди.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Выходные данные в сообщение очереди доступны через `Push-OutputBinding` место передачи аргументов, совпадающих с именем, назначенным `name` параметром binding в *function.js* в файле.

# <a name="python"></a>[Python](#tab/python)

Существует два варианта вывода сообщения очереди из функции:

- **Возвращаемое значение** — задайте для свойства `name` в файле *function.json* значение `$return`. В этой конфигурации возвращаемое значение функции сохраняется как сообщение хранилища очереди.

- **Императив** — передайте значение методу [set](/python/api/azure-functions/azure.functions.out#set-val--t-----none) параметра, объявленного с типом [Out](/python/api/azure-functions/azure.functions.out). Значение, передаваемое в `set` , сохраняется как сообщение хранилища очереди.

---

## <a name="exceptions-and-return-codes"></a>Исключения и коды возврата

| Привязка |  Справочник |
|---|---|
| Очередь | [Коды ошибок очередей](/rest/api/storageservices/queue-service-error-codes) |
| Большой двоичный объект, таблица, очередь | [Коды ошибок хранилища](/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Большой двоичный объект, таблица, очередь |  [Устранение неполадок](/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>Следующие шаги

- [Выполнение функции как изменения данных хранилища очередей (триггер)](./functions-bindings-storage-queue-trigger.md)

<!-- LINKS -->

[CloudQueueMessage]: /dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage
