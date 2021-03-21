---
title: Триггер RabbitMQ для функций Azure
description: Узнайте, как запустить функцию Azure при создании сообщения RabbitMQ.
author: cachai2
ms.assetid: ''
ms.topic: reference
ms.date: 12/17/2020
ms.author: cachai
ms.custom: ''
ms.openlocfilehash: be3c5bc2d178171aaebd322e13b23b3a6f79c442
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100388997"
---
# <a name="rabbitmq-trigger-for-azure-functions-overview"></a>Общие сведения о триггере RabbitMQ для функций Azure

> [!NOTE]
> Привязки RabbitMQ полностью поддерживаются только для планов уровня " **Премиум" и "выделен** ". Использование не поддерживается.

Используйте триггер RabbitMQ для реагирования на сообщения из очереди RabbitMQ.

Сведения об установке и настройке см. в [этой обзорной статье](functions-bindings-rabbitmq.md).

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md) , которая считывает и регистрирует сообщение RabbitMQ как [событие RabbitMQ](https://rabbitmq.github.io/rabbitmq-dotnet-client/api/RabbitMQ.Client.Events.BasicDeliverEventArgs.html):

```cs
[FunctionName("RabbitMQTriggerCSharp")]
public static void RabbitMQTrigger_BasicDeliverEventArgs(
    [RabbitMQTrigger("queue", ConnectionStringSetting = "rabbitMQConnectionAppSetting")] BasicDeliverEventArgs args,
    ILogger logger
    )
{
    logger.LogInformation($"C# RabbitMQ queue trigger function processed message: {Encoding.UTF8.GetString(args.Body)}");
}
```

В следующем примере показано, как прочитать сообщение в виде POCO.

```cs
namespace Company.Function
{
    public class TestClass
    {
        public string x { get; set; }
    }

    public class RabbitMQTriggerCSharp{
        [FunctionName("RabbitMQTriggerCSharp")]
        public static void RabbitMQTrigger_BasicDeliverEventArgs(
            [RabbitMQTrigger("queue", ConnectionStringSetting = "rabbitMQConnectionAppSetting")] TestClass pocObj,
            ILogger logger
            )
        {
            logger.LogInformation($"C# RabbitMQ queue trigger function processed message: {pocObj}");
        }
    }
}
```

Как и в случае с объектами JSON, при неправильном форматировании сообщения в качестве объекта C# возникнет ошибка. Если это так, он привязывается к переменной Покобж, которую можно использовать для того, для чего она нужна.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показана привязка триггера RabbitMQ в *function.jsдля* файла и [функция сценария C#](functions-reference-csharp.md) , которая использует эту привязку. Функция считывает и регистрирует сообщение RabbitMQ.

Данные привязки в файле *function.json*:

```json
{
    "bindings": [
        {
            "name": "myQueueItem",
            "type": "rabbitMQTrigger",
            "direction": "in",
            "queueName": "queue",
            "connectionStringSetting": "rabbitMQConnectionAppSetting"
        }
    ]
}
```

Ниже приведен код скрипта C#.

```C#
using System;

public static void Run(string myQueueItem, ILogger log)
{
    log.LogInformation($"C# Script RabbitMQ trigger function processed: {myQueueItem}");
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана привязка триггера RabbitMQ в *function.jsдля* файла и [функция JavaScript](functions-reference-node.md) , которая использует эту привязку. Функция считывает и записывает в журнал сообщение RabbitMQ.

Данные привязки в файле *function.json*:

```json
{
    "bindings": [
        {
            "name": "myQueueItem",
            "type": "rabbitMQTrigger",
            "direction": "in",
            "queueName": "queue",
            "connectionStringSetting": "rabbitMQConnectionAppSetting"
        }
    ]
}
```

Ниже показан код сценария JavaScript.

```javascript
module.exports = async function (context, myQueueItem) {
    context.log('JavaScript RabbitMQ trigger function processed work item', myQueueItem);
};
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показано, как прочитать сообщение очереди RabbitMQ с помощью триггера.

Привязка RabbitMQ определяется в *function.js* , где *тип* имеет значение `RabbitMQTrigger` .

```json
{
    "scriptFile": "__init__.py",
    "bindings": [
        {
            "name": "myQueueItem",
            "type": "rabbitMQTrigger",
            "direction": "in",
            "queueName": "queue",
            "connectionStringSetting": "rabbitMQConnectionAppSetting"
        }
    ]
}
```

```python
import logging
import azure.functions as func

def main(myQueueItem) -> None:
    logging.info('Python RabbitMQ trigger function processed a queue item: %s', myQueueItem)
```

# <a name="java"></a>[Java](#tab/java)

Следующая функция Java использует `@RabbitMQTrigger` аннотацию из [типов RabbitMQ Java](https://mvnrepository.com/artifact/com.microsoft.azure.functions/azure-functions-java-library-rabbitmq) для описания конфигурации триггера очереди RabbitMQ. Функция извлекает сообщение, помещенное в очередь, и добавляет его в журналы.

```java
@FunctionName("RabbitMQTriggerExample")
public void run(
    @RabbitMQTrigger(connectionStringSetting = "rabbitMQConnectionAppSetting", queueName = "queue") String input,
    final ExecutionContext context)
{
    context.getLogger().info("Java HTTP trigger processed a request." + input);
}
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md)используйте атрибут [раббитмктригжер](https://github.com/Azure/azure-functions-rabbitmq-extension/blob/dev/src/Trigger/RabbitMQTriggerAttribute.cs) .

Вот `RabbitMQTrigger` атрибут в сигнатуре метода:

```csharp
[FunctionName("RabbitMQTest")]
public static void RabbitMQTest([RabbitMQTrigger("queue")] string message, ILogger log)
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

`RabbitMQTrigger`Заметка позволяет создать функцию, которая выполняется при создании сообщения RabbitMQ. Доступные параметры конфигурации включают имя очереди и имя строки подключения. Дополнительные сведения о параметрах см. в [заметках Java раббитмктригжер](https://github.com/Azure/azure-functions-rabbitmq-extension/blob/dev/binding-library/java/src/main/java/com/microsoft/azure/functions/rabbitmq/annotation/RabbitMQTrigger.java).

Дополнительные сведения см. в [примере](#example) триггера.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `RabbitMQTrigger`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Необходимо задать значение "Раббитмктригжер".|
|**direction** | Недоступно | Для этого свойства необходимо задать значение "in".|
|**name** | Недоступно | Имя переменной, представляющей очередь в коде функции. |
|**queueName**|**QueueName**| Имя очереди, из которой должны быть получены сообщения. |
|**Имя узла**|**HostName**|(пропускается при использовании Коннектстрингсеттинг) <br>Имя узла очереди (например: 10.26.45.210)|
|**усернамесеттинг**|**усернамесеттинг**|(пропускается при использовании Коннектионстрингсеттинг) <br>Имя параметра приложения, содержащего имя пользователя для доступа к очереди. Например: Усернамесеттинг: "% < Усернамефромсеттингс >%"|
|**пассвордсеттинг**|**пассвордсеттинг**|(пропускается при использовании Коннектионстрингсеттинг) <br>Имя параметра приложения, который содержит пароль для доступа к очереди. Например: Пассвордсеттинг: "% < Пассвордфромсеттингс >%"|
|**коннектионстрингсеттинг**|**ConnectionStringSetting**|Имя параметра приложения, содержащего строку подключения очереди сообщений RabbitMQ. Обратите внимание, что если вы укажете строку подключения напрямую, а не через параметр приложения в local.settings.js, триггер не будет работать. (Пример: в *function.json*: коннектионстрингсеттинг: "раббитмкконнектион" <br> В *local.settings.js*: "раббитмкконнектион": "< актуалконнектионстринг >")|
|**port**|**порт**.|(пропускается при использовании Коннектионстрингсеттинг) Возвращает или задает используемый порт. По умолчанию принимает значение 0, которое указывает на порт клиента rabbitmq по умолчанию: 5672.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

Типом сообщений по умолчанию является [событие RabbitMQ](https://rabbitmq.github.io/rabbitmq-dotnet-client/api/RabbitMQ.Client.Events.BasicDeliverEventArgs.html), а `Body` свойство события RabbitMQ может быть считано так, как показано ниже.

* `An object serializable as JSON` — Сообщение доставляется в виде допустимой строки JSON.
* `string`
* `byte[]`
* `POCO` — Сообщение отформатировано как объект C#. Полный пример см. в разделе [Пример](#example)на C#.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

Типом сообщений по умолчанию является [событие RabbitMQ](https://rabbitmq.github.io/rabbitmq-dotnet-client/api/RabbitMQ.Client.Events.BasicDeliverEventArgs.html), а `Body` свойство события RabbitMQ может быть считано так, как показано ниже.

* `An object serializable as JSON` — Сообщение доставляется в виде допустимой строки JSON.
* `string`
* `byte[]`
* `POCO` — Сообщение отформатировано как объект C#. Полный пример см. в разделе [Пример](#example)скрипта C#.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Сообщение очереди доступно через контекст. Bindings.<NAME> где <NAME> соответствует имени, определенному в function.js. Если полезная нагрузка — JSON, значение десериализуется в объект.

# <a name="python"></a>[Python](#tab/python)

См. [Пример](#example)на Python.

# <a name="java"></a>[Java](#tab/java)

См. [атрибуты и заметки](#attributes-and-annotations)Java.

---

## <a name="dead-letter-queues"></a>Очереди недоставленных сообщений
Нельзя управлять и настраивать очереди недоставленных сообщений и обмена с помощью триггера RabbitMQ.  Чтобы использовать очереди недоставленных сообщений, предварительно Настройте очередь, используемую триггером в RabbitMQ. См. [документацию по RabbitMQ](https://www.rabbitmq.com/dlx.html).

## <a name="hostjson-settings"></a>Параметры файла host.json

В этом разделе описаны глобальные параметры конфигурации, доступные для этой привязки в версиях 2. x и более поздних. Пример *host.jsв* файле ниже содержит только параметры для этой привязки. Дополнительные сведения о глобальных параметрах конфигурации см. [ в разделеhost.jsпо Справочнику по функциям Azure](functions-host-json.md).

```json
{
    "version": "2.0",
    "extensions": {
        "rabbitMQ": {
            "prefetchCount": 100,
            "queueName": "queue",
            "connectionString": "amqp://user:password@url:port",
            "port": 10
        }
    }
}
```

|Свойство  |По умолчанию | Описание |
|---------|---------|---------|
|prefetchCount|30|Возвращает или задает количество сообщений, которые получатель сообщения может одновременно запрашивать и кэшировать.|
|queueName|Н/Д| Имя очереди, из которой должны быть получены сообщения.|
|connectionString|Н/Д|Строка подключения очереди сообщений RabbitMQ. Обратите внимание, что строка подключения указывается непосредственно здесь, а не с помощью параметра приложения.|
|порт|0|(пропускается при использовании connectionString) Возвращает или задает используемый порт. По умолчанию принимает значение 0, которое указывает на порт клиента rabbitmq по умолчанию: 5672.|

## <a name="local-testing"></a>Локальное тестирование

> [!NOTE]
> Строка подключения имеет приоритет над "имя_узла", "userName" и "Password". Если все они заданы, то строка подключения будет переопределять два других.

Если выполняется локальное тестирование без строки подключения, следует задать параметр hostName и "userName" и "Password", если это применимо в разделе "rabbitMQ" *host.json*:

```json
{
    "version": "2.0",
    "extensions": {
        "rabbitMQ": {
            ...
            "hostName": "localhost",
            "username": "userNameSetting",
            "password": "passwordSetting"
        }
    }
}
```

|Свойство  |По умолчанию | Описание |
|---------|---------|---------|
|hostName|Н/Д|(пропускается при использовании connectionString) <br>Имя узла очереди (например: 10.26.45.210)|
|userName|Н/Д|(пропускается при использовании connectionString) <br>Имя для доступа к очереди |
|password|Н/Д|(пропускается при использовании connectionString) <br>Пароль для доступа к очереди|


## <a name="enable-runtime-scaling"></a>Включить масштабирование среды выполнения

Чтобы триггер RabbitMQ можно было масштабировать до нескольких экземпляров, необходимо включить параметр **наблюдения за масштабом среды выполнения** . 

На портале этот параметр можно найти в разделе   >  **Параметры среды выполнения функции** конфигурации для приложения функции.

:::image type="content" source="media/functions-networking-options/virtual-network-trigger-toggle.png" alt-text="VNETToggle":::

В интерфейсе командной строки можно включить **мониторинг масштабирования во время выполнения** с помощью следующей команды:

```azurecli-interactive
az resource update -g <resource_group> -n <function_app_name>/config/web --set properties.functionsRuntimeScaleMonitoringEnabled=1 --resource-type Microsoft.Web/sites
```

## <a name="monitoring-rabbitmq-endpoint"></a>Мониторинг конечной точки RabbitMQ
Для мониторинга очередей и обмена для определенной конечной точки RabbitMQ:

* Включение [подключаемого модуля управления RabbitMQ](https://www.rabbitmq.com/management.html)
* Перейдите по http://{node-имя_узла}: 15672 и войдите в систему, используя имя пользователя и пароль.

## <a name="next-steps"></a>Дальнейшие действия

- [Отправка сообщений RabbitMQ из функций Azure (Выходная привязка)](./functions-bindings-rabbitmq-output.md)
