---
title: Выходные привязки RabbitMQ для функций Azure
description: Узнайте, как отправить сообщения RabbitMQ из функций Azure.
author: cachai2
ms.assetid: ''
ms.topic: reference
ms.date: 12/17/2020
ms.author: cachai
ms.custom: ''
ms.openlocfilehash: 1664656f82492e664b7574339893cd688f0a061d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100097319"
---
# <a name="rabbitmq-output-binding-for-azure-functions-overview"></a>Обзор выходной привязки RabbitMQ для функций Azure

> [!NOTE]
> Привязки RabbitMQ полностью поддерживаются только для планов уровня " **Премиум" и "выделен** ". Использование не поддерживается.

Используйте выходную привязку RabbitMQ для отправки сообщений в очередь RabbitMQ.

Сведения об установке и настройке см. в [этой обзорной статье](functions-bindings-rabbitmq-output.md).

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md) , которая отправляет сообщение RabbitMQ при активации TimerTrigger каждые 5 минут, используя возвращаемое значение метода в качестве выходных данных:

```cs
[FunctionName("RabbitMQOutput")]
[return: RabbitMQ(QueueName = "outputQueue", ConnectionStringSetting = "rabbitMQConnectionAppSetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    log.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
    return $"{DateTime.Now}";
}
```

В следующем примере показано, как использовать интерфейс IAsyncCollector для отправки сообщений.

```cs
[FunctionName("RabbitMQOutput")]
public static async Task Run(
[RabbitMQTrigger("sourceQueue", ConnectionStringSetting = "rabbitMQConnectionAppSetting")] string rabbitMQEvent,
[RabbitMQ(QueueName = "destinationQueue", ConnectionStringSetting = "rabbitMQConnectionAppSetting")]IAsyncCollector<string> outputEvents,
ILogger log)
{
     // send the message
    await outputEvents.AddAsync(JsonConvert.SerializeObject(rabbitMQEvent));
}
```

В следующем примере показано, как отправить сообщения в виде POCO.

```cs
namespace Company.Function
{
    public class TestClass
    {
        public string x { get; set; }
    }
    public static class RabbitMQOutput{
        [FunctionName("RabbitMQOutput")]
        public static async Task Run(
        [RabbitMQTrigger("sourceQueue", ConnectionStringSetting = "rabbitMQConnectionAppSetting")] TestClass rabbitMQEvent,
        [RabbitMQ(QueueName = "destinationQueue", ConnectionStringSetting = "rabbitMQConnectionAppSetting")]IAsyncCollector<TestClass> outputPocObj,
        ILogger log)
        {
            // send the message
            await outputPocObj.AddAsync(rabbitMQEvent);
        }
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показана выходная привязка RabbitMQ в *function.jsдля* файла и [функция сценария C#](functions-reference-csharp.md) , которая использует эту привязку. Функция считывает сообщение из триггера HTTP и выводит его в очередь RabbitMQ.

Данные привязки в файле *function.json*:

```json
{
    "bindings": [
        {
            "type": "httpTrigger",
            "direction": "in",
            "authLevel": "function",
            "name": "input",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "type": "rabbitMQ",
            "name": "outputMessage",
            "queueName": "outputQueue",
            "connectionStringSetting": "rabbitMQConnectionAppSetting",
            "direction": "out"
        }
    ]
}
```

Ниже приведен код скрипта C#.

```C#
using System;
using Microsoft.Extensions.Logging;

public static void Run(string input, out string outputMessage, ILogger log)
{
    log.LogInformation(input);
    outputMessage = input;
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана выходная привязка RabbitMQ в *function.jsдля* файла и [функция JavaScript](functions-reference-node.md) , которая использует эту привязку. Функция считывает сообщение из триггера HTTP и выводит его в очередь RabbitMQ.

Данные привязки в файле *function.json*:

```json
{
    "bindings": [
        {
            "type": "httpTrigger",
            "direction": "in",
            "authLevel": "function",
            "name": "input",
            "methods": [
                "get",
                "post"
            ]
        },
        {
            "type": "rabbitMQ",
            "name": "outputMessage",
            "queueName": "outputQueue",
            "connectionStringSetting": "rabbitMQConnectionAppSetting",
            "direction": "out"
        }
    ]
}
```

Вот код JavaScript:

```javascript
module.exports = function (context, input) {
    context.bindings.myQueueItem = input.body;
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показана выходная привязка RabbitMQ в *function.jsдля* файла и функция Python, использующая привязку. Функция считывает сообщение из триггера HTTP и выводит его в очередь RabbitMQ.

Данные привязки в файле *function.json*:

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
            "type": "rabbitMQ",
            "name": "outputMessage",
            "queueName": "outputQueue",
            "connectionStringSetting": "rabbitMQConnectionAppSetting",
            "direction": "out"
        }
    ]
}
```

В *_\_ init_ \_ . Корректировка*:

```python
import azure.functions as func

def main(req: func.HttpRequest, outputMessage: func.Out[str]) -> func.HttpResponse:
    input_msg = req.params.get('message')
    outputMessage.set(input_msg)
    return 'OK'
```

# <a name="java"></a>[Java](#tab/java)

Следующая функция Java использует `@RabbitMQOutput` аннотацию из [типов Java RabbitMQ](https://mvnrepository.com/artifact/com.microsoft.azure.functions/azure-functions-java-library-rabbitmq) для описания конфигурации для выходной привязки очереди RabbitMQ. Функция отправляет сообщение в очередь RabbitMQ при активации TimerTrigger каждые 5 минут.

```java
@FunctionName("RabbitMQOutputExample")
public void run(
@TimerTrigger(name = "keepAliveTrigger", schedule = "0 */5 * * * *") String timerInfo,
@RabbitMQOutput(connectionStringSetting = "rabbitMQConnectionAppSetting", queueName = "hello") OutputBinding<String> output,
final ExecutionContext context) {
    output.setValue("Some string");
}
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md)используйте [раббитмкаттрибуте](https://github.com/Azure/azure-functions-rabbitmq-extension/blob/dev/src/RabbitMQAttribute.cs).

Вот `RabbitMQAttribute` атрибут в сигнатуре метода:

```csharp
[FunctionName("RabbitMQOutput")]
public static async Task Run(
[RabbitMQTrigger("SourceQueue", ConnectionStringSetting = "TriggerConnectionString")] string rabbitMQEvent,
[RabbitMQ("DestinationQueue", ConnectionStringSetting = "OutputConnectionString")]IAsyncCollector<string> outputEvents,
ILogger log)
{
    ...
}
```

Полный пример см. в разделе [Пример](#example)на C#.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

`RabbitMQOutput`Заметка позволяет создать функцию, которая выполняется при отправке сообщения RabbitMQ. Доступные параметры конфигурации включают имя очереди и имя строки подключения. Дополнительные сведения о параметрах см. в [заметках Java раббитмкаутпут](https://github.com/Azure/azure-functions-rabbitmq-extension/blob/dev/binding-library/java/src/main/java/com/microsoft/azure/functions/rabbitmq/annotation/RabbitMQOutput.java).

Дополнительные сведения см. в [примере](#example) выходной привязки.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `RabbitMQ`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Необходимо задать значение "RabbitMQ".|
|**direction** | Недоступно | Для этого свойства необходимо задать значение out. |
|**name** | Недоступно | Имя переменной, представляющей очередь в коде функции. |
|**queueName**|**QueueName**| Имя очереди, в которую отправляются сообщения. |
|**Имя узла**|**HostName**|(пропускается при использовании Коннектстрингсеттинг) <br>Имя узла очереди (например: 10.26.45.210)|
|**userName**|**UserName**|(пропускается при использовании Коннектионстрингсеттинг) <br>Имя параметра приложения, содержащего имя пользователя для доступа к очереди. Например: Усернамесеттинг: "< Усернамефромсеттингс >"|
|**password**|**Пароль**|(пропускается при использовании Коннектионстрингсеттинг) <br>Имя параметра приложения, который содержит пароль для доступа к очереди. Например: Усернамесеттинг: "< Усернамефромсеттингс >"|
|**коннектионстрингсеттинг**|**ConnectionStringSetting**|Имя параметра приложения, содержащего строку подключения очереди сообщений RabbitMQ. Обратите внимание, что если вы укажете строку подключения напрямую, а не через параметр приложения в local.settings.js, триггер не будет работать. (Пример: в *function.json*: коннектионстрингсеттинг: "раббитмкконнектион" <br> В *local.settings.js*: "раббитмкконнектион": "< актуалконнектионстринг >")|
|**port**|**порт**.|(пропускается при использовании Коннектионстрингсеттинг) Возвращает или задает используемый порт. По умолчанию принимает значение 0, которое указывает на порт клиента rabbitmq по умолчанию: 5672.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

Используйте следующие типы параметров для выходной привязки:

* `byte[]`. Если при выходе из функции параметр имеет значение NULL, то служба "Функции" не создает сообщение.
* `string`. Если при выходе из функции параметр имеет значение NULL, то служба "Функции" не создает сообщение.
* `POCO` — Если значение параметра не отформатировано как объект C#, будет получена ошибка. Полный пример см. в разделе [Пример](#example)на C#.

При работе с функциями C#:

* Асинхронным функциям требуется возвращаемое значение или `IAsyncCollector` вместо `out` параметра.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

Используйте следующие типы параметров для выходной привязки:

* `byte[]`. Если при выходе из функции параметр имеет значение NULL, то служба "Функции" не создает сообщение.
* `string`. Если при выходе из функции параметр имеет значение NULL, то служба "Функции" не создает сообщение.
* `POCO` — Если значение параметра не отформатировано как объект C#, будет получена ошибка. Полный пример см. в разделе [Пример](#example)скрипта C#.

При работе с функциями скриптов C#:

* Асинхронным функциям требуется возвращаемое значение или `IAsyncCollector` вместо `out` параметра.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Сообщение очереди доступно через контекст. Bindings.<NAME> где <NAME> соответствует имени, определенному в function.js. Если полезная нагрузка — JSON, значение десериализуется в объект.

# <a name="python"></a>[Python](#tab/python)

См. [Пример](#example)на Python.

# <a name="java"></a>[Java](#tab/java)

Используйте следующие типы параметров для выходной привязки:

* `byte[]`. Если при выходе из функции параметр имеет значение NULL, то служба "Функции" не создает сообщение.
* `string`. Если при выходе из функции параметр имеет значение NULL, то служба "Функции" не создает сообщение.
* `POJO` — Если значение параметра не отформатировано как объект Java, будет получено сообщение об ошибке.

---

## <a name="next-steps"></a>Дальнейшие действия

- [Выполнение функции при создании сообщения RabbitMQ (триггер)](./functions-bindings-rabbitmq-trigger.md)
