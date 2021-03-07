---
title: Триггер служебной шины Azure для функций Azure
description: Научитесь запускать функцию Azure при создании сообщений служебной шины Azure.
author: craigshoemaker
ms.assetid: daedacf0-6546-4355-a65c-50873e74f66b
ms.topic: reference
ms.date: 02/19/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 4b95c25400317b2baac694f4ba2b1b1dc1eae098
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102435160"
---
# <a name="azure-service-bus-trigger-for-azure-functions"></a>Триггер служебной шины Azure для функций Azure

Используйте триггер служебной шины для ответа на сообщения из очереди или раздела служебной шины. Начиная с версии Extension 3.1.0 можно активировать в очереди или разделе с поддержкой сеанса.

Сведения об установке и настройке см. в [этой обзорной статье](functions-bindings-service-bus.md).

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая считывает [метаданные сообщения](#message-metadata) и заносит в журнал сообщение из очереди службы "Служебная шина".

```cs
[FunctionName("ServiceBusQueueTriggerCSharp")]                    
public static void Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] 
    string myQueueItem,
    Int32 deliveryCount,
    DateTime enqueuedTimeUtc,
    string messageId,
    ILogger log)
{
    log.LogInformation($"C# ServiceBus queue trigger function processed message: {myQueueItem}");
    log.LogInformation($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.LogInformation($"DeliveryCount={deliveryCount}");
    log.LogInformation($"MessageId={messageId}");
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показаны привязка триггера служебной шины в файле *function.json* и [функция сценария C#](functions-reference-csharp.md), которая использует эту привязку. Функция считывает [метаданные сообщения](#message-metadata) и заносит в журнал сообщение из очереди службы "Служебная шина".

Данные привязки в файле *function.json*:

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

Ниже приведен код скрипта C#.

```cs
using System;

public static void Run(string myQueueItem,
    Int32 deliveryCount,
    DateTime enqueuedTimeUtc,
    string messageId,
    TraceWriter log)
{
    log.Info($"C# ServiceBus queue trigger function processed message: {myQueueItem}");

    log.Info($"EnqueuedTimeUtc={enqueuedTimeUtc}");
    log.Info($"DeliveryCount={deliveryCount}");
    log.Info($"MessageId={messageId}");
}
```

# <a name="java"></a>[Java](#tab/java)

Следующая функция Java использует `@ServiceBusQueueTrigger` аннотацию из [библиотеки среды выполнения функций Java](/java/api/overview/azure/functions/runtime) для описания конфигурации триггера очереди служебной шины. Функция извлекает сообщение, помещенное в очередь, и добавляет его в журналы.

```java
@FunctionName("sbprocessor")
 public void serviceBusProcess(
    @ServiceBusQueueTrigger(name = "msg",
                             queueName = "myqueuename",
                             connection = "myconnvarname") String message,
   final ExecutionContext context
 ) {
     context.getLogger().info(message);
 }
```

Функции Java также могут быть активированы при добавлении сообщения в раздел служебной шины. В следующем примере `@ServiceBusTopicTrigger` заметка используется для описания конфигурации триггера.

```java
@FunctionName("sbtopicprocessor")
    public void run(
        @ServiceBusTopicTrigger(
            name = "message",
            topicName = "mytopicname",
            subscriptionName = "mysubscription",
            connection = "ServiceBusConnection"
        ) String message,
        final ExecutionContext context
    ) {
        context.getLogger().info(message);
    }
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показаны привязка триггера служебной шины в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует эту привязку. Функция считывает [метаданные сообщения](#message-metadata) и заносит в журнал сообщение из очереди службы "Служебная шина".

Данные привязки в файле *function.json*:

```json
{
"bindings": [
    {
    "queueName": "testqueue",
    "connection": "MyServiceBusConnection",
    "name": "myQueueItem",
    "type": "serviceBusTrigger",
    "direction": "in"
    }
],
"disabled": false
}
```

Ниже показан код сценария JavaScript.

```javascript
module.exports = function(context, myQueueItem) {
    context.log('Node.js ServiceBus queue trigger function processed message', myQueueItem);
    context.log('EnqueuedTimeUtc =', context.bindingData.enqueuedTimeUtc);
    context.log('DeliveryCount =', context.bindingData.deliveryCount);
    context.log('MessageId =', context.bindingData.messageId);
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показана привязка триггера служебной шины в *function.jsв* файле и [функция PowerShell](functions-reference-powershell.md) , которая использует эту привязку. 

Данные привязки в файле *function.json*:

```json
{
  "bindings": [
    {
      "name": "mySbMsg",
      "type": "serviceBusTrigger",
      "direction": "in",
      "topicName": "mytopic",
      "subscriptionName": "mysubscription",
      "connection": "AzureServiceBusConnectionString"
    }
  ]
}
```

Ниже приведена функция, которая выполняется при отправке сообщения служебной шины.

```powershell
param([string] $mySbMsg, $TriggerMetadata)

Write-Host "PowerShell ServiceBus queue trigger function processed message: $mySbMsg"
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показано, как считать сообщение очереди служебной шины с помощью триггера.

Привязка служебной шины определяется в *function.js* , где *тип* имеет значение `serviceBusTrigger` .

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "msg",
      "type": "serviceBusTrigger",
      "direction": "in",
      "queueName": "inputqueue",
      "connection": "AzureServiceBusConnectionString"
    }
  ]
}
```

Код в *_\_ init_ \_ . Корректировка* объявляет параметр как `func.ServiceBusMessage` , который позволяет считывать сообщение очереди в функции.

```python
import azure.functions as func

import logging
import json

def main(msg: func.ServiceBusMessage):
    logging.info('Python ServiceBus queue trigger processed message.')

    result = json.dumps({
        'message_id': msg.message_id,
        'body': msg.get_body().decode('utf-8'),
        'content_type': msg.content_type,
        'expiration_time': msg.expiration_time,
        'label': msg.label,
        'partition_key': msg.partition_key,
        'reply_to': msg.reply_to,
        'reply_to_session_id': msg.reply_to_session_id,
        'scheduled_enqueue_time': msg.scheduled_enqueue_time,
        'session_id': msg.session_id,
        'time_to_live': msg.time_to_live,
        'to': msg.to,
        'user_properties': msg.user_properties,
        'metadata' : msg.metadata
    })

    logging.info(result)
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте следующие атрибуты для настройки триггера службы "Служебная шина":

* [ServiceBusTriggerAttribute](https://github.com/Azure/azure-functions-servicebus-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusTriggerAttribute.cs)

  Конструктор этого атрибута принимает имя очереди или раздела и подписки. В Функциях Azure версии 1.х можно также указать права доступа для подключения. Если права доступа не указаны, то используется значение по умолчанию, `Manage`. Дополнительные сведения см. в разделе [Конфигурация триггера](#configuration).

  Ниже приведен пример, в котором показано, как атрибут используется со строковым параметром.

  ```csharp
  [FunctionName("ServiceBusQueueTriggerCSharp")]                    
  public static void Run(
      [ServiceBusTrigger("myqueue")] string myQueueItem, ILogger log)
  {
      ...
  }
  ```

  Так как `Connection` свойство не определено, функции ищет параметр приложения с именем `AzureWebJobsServiceBus` , который является именем по умолчанию для строки подключения служебной шины. Можно также задать свойство, `Connection` чтобы указать имя параметра приложения, содержащего строку подключения служебной шины, которая будет использоваться, как показано в следующем примере:

  ```csharp
  [FunctionName("ServiceBusQueueTriggerCSharp")]                    
  public static void Run(
      [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] 
      string myQueueItem, ILogger log)
  {
      ...
  }
  ```

  Полный пример см. в разделе [пример триггера](#example).

* [ServiceBusAccountAttribute](https://github.com/Azure/azure-functions-servicebus-extension/blob/master/src/Microsoft.Azure.WebJobs.Extensions.ServiceBus/ServiceBusAccountAttribute.cs)

  Предоставляет еще один способ указать используемую учетную запись служебной шины. Конструктор принимает имя параметра приложения, содержащего строку подключения к служебной шине. Атрибут может применяться на уровне класса, метода или параметра. Ниже показан пример уровня класса и метода.

  ```csharp
  [ServiceBusAccount("ClassLevelServiceBusAppSetting")]
  public static class AzureFunctions
  {
      [ServiceBusAccount("MethodLevelServiceBusAppSetting")]
      [FunctionName("ServiceBusQueueTriggerCSharp")]
      public static void Run(
          [ServiceBusTrigger("myqueue", AccessRights.Manage)] 
          string myQueueItem, ILogger log)
  {
      ...
  }
  ```

Используемая учетная запись служебной шины определяется в следующем порядке:

* Свойство `ServiceBusTrigger` атрибута `Connection`.
* Атрибут `ServiceBusAccount`, примененный к тому же параметру, что и `ServiceBusTrigger`.
* Атрибут `ServiceBusAccount`, примененный к функции.
* Атрибут `ServiceBusAccount`, примененный к классу.
* Параметр приложения AzureWebJobsServiceBus.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

`ServiceBusQueueTrigger`Заметка позволяет создать функцию, которая выполняется при создании сообщения очереди служебной шины. Доступные параметры конфигурации включают имя очереди и имя строки подключения.

`ServiceBusTopicTrigger`Заметка позволяет назначить раздел и подписку, чтобы указать, какие данные активирует функция.

Дополнительные сведения см. в [примере](#example) триггера.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `ServiceBusTrigger`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Для этого свойства нужно задать значение "serviceBusTrigger". Это свойство задается автоматически при создании триггера на портале Azure.|
|**direction** | Недоступно | Для этого свойства необходимо задать значение "in". Это свойство задается автоматически при создании триггера на портале Azure. |
|**name** | Недоступно | Имя переменной, представляющей сообщение очереди или раздела в коде функции. |
|**queueName**|**QueueName**|Имя очереди для отслеживания.  Задается только в том случае, если отслеживается очередь, но не раздел.
|**topicName**|**топикнаме**|Имя отслеживаемого раздела. Задается только в том случае, если отслеживается раздел, но не очередь.|
|**subscriptionName**|**SubscriptionName**|Имя отслеживаемой подписки. Задается только в том случае, если отслеживается раздел, но не очередь.|
|**connection**;|**Соединение**|Имя параметра приложения, содержащего строку подключения к служебной шине, используемую для этой привязки. Если имя параметра приложения начинается с AzureWebJobs, можно указать только остальную часть имени. Например, если задано значение `connection` "мисервицебус", среда выполнения функций ищет параметр приложения с именем "азуревебжобсмисервицебус". Если оставить строку `connection` пустой, то среда выполнения службы "Функции" будет использовать строку подключения к служебной шине по умолчанию для параметра приложения AzureWebJobsServiceBus.<br><br>Чтобы получить строку подключения, следуйте инструкциям, указанным в разделе [Получение учетных данных управления](../service-bus-messaging/service-bus-quickstart-portal.md#get-the-connection-string). Строка подключения указывается для пространства имен служебной шины, и она не должна ограничиваться определенной очередью или разделом. |
|**accessRights**|**Доступ**|Права доступа для строки подключения. Доступные значения: `manage` и `listen`. Значение по умолчанию — `manage`. Это означает, что у свойства `connection` есть разрешение на **управление**. При использовании строки подключения без разрешения на **управление** задайте для свойства `accessRights` значение "listen". В противном случае выполнение операций, для которых требуются права на управление, в среде выполнения Функций Azure может завершиться ошибкой. В функциях Azure версии 2. x и более поздних это свойство недоступно, поскольку последняя версия пакета SDK служебной шины не поддерживает операции управления.|
|**иссессионсенаблед**|**иссессионсенаблед**|`true` При подключении к очереди или подписке с [поддержкой сеансов](../service-bus-messaging/message-sessions.md) . `false` в противном случае — значение по умолчанию.|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

Для сообщения очереди или раздела доступны следующие типы параметров:

* `string`. Если сообщение является текстом.
* `byte[]`. Используется для двоичных данных.
* Пользовательский тип. Если сообщение содержит JSON, то служба "Функции Azure" пытается выполнить десериализацию данных JSON.
* `BrokeredMessage` — Возвращает десериализованное сообщение с помощью метода [BrokeredMessage. @ Body \<T> ()](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.getbody#Microsoft_ServiceBus_Messaging_BrokeredMessage_GetBody__1) .
* [`MessageReceiver`](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver) — Используется для получения и подтверждения сообщений из контейнера сообщений (требуется, если параметр [`autoComplete`](functions-bindings-service-bus-output.md#hostjson-settings) имеет значение `false` ).

Эти типы параметров предназначены для функций Azure версии 1. x; для 2. x и более поздних версий используйте [`Message`](/dotnet/api/microsoft.azure.servicebus.message) вместо `BrokeredMessage` .

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

Для сообщения очереди или раздела доступны следующие типы параметров:

* `string`. Если сообщение является текстом.
* `byte[]`. Используется для двоичных данных.
* Пользовательский тип. Если сообщение содержит JSON, то служба "Функции Azure" пытается выполнить десериализацию данных JSON.
* `BrokeredMessage` — Возвращает десериализованное сообщение с помощью метода [BrokeredMessage. @ Body \<T> ()](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.getbody#Microsoft_ServiceBus_Messaging_BrokeredMessage_GetBody__1) .

Эти параметры предназначены для функций Azure версии 1. x; для 2. x и более поздних версий используйте [`Message`](/dotnet/api/microsoft.azure.servicebus.message) вместо `BrokeredMessage` .

# <a name="java"></a>[Java](#tab/java)

Входящее сообщение служебной шины доступно через `ServiceBusQueueMessage` `ServiceBusTopicMessage` параметр или.

Дополнительные [сведения см. в примере](#example).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Доступ к сообщению очереди или раздела с помощью `context.bindings.<name from function.json>` . Сообщение служебной шины передается в функцию в качестве строки или объекта JSON.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Экземпляр служебной шины доступен через параметр, настроенный в *function.js* свойству имени файла.

# <a name="python"></a>[Python](#tab/python)

Сообщение очереди доступно функции через параметр, типизированный как `func.ServiceBusMessage` . Сообщение служебной шины передается в функцию в качестве строки или объекта JSON.

---

## <a name="poison-messages"></a>Подозрительные сообщения

В службе "Функции Azure" невозможно управление обработкой сообщений о сбое или настройка этой обработки. Служебная шина сама обрабатывает сообщения о сбое.

## <a name="peeklock-behavior"></a> Поведение PeekLock

Среда выполнения службы "Функции" получает сообщение в [режиме PeekLock](../service-bus-messaging/service-bus-performance-improvements.md#receive-mode). Она вызывает `Complete` для сообщения, если функция выполнена успешно, или `Abandon` в случае сбоя. Если функция выполняется дольше времени ожидания `PeekLock`, блокировка возобновляется автоматически до тех пор, пока выполняется функция.

Параметр `maxAutoRenewDuration` можно задать в файле *host.json*. Этот параметр сопоставляется с параметром [OnMessageOptions.MaxAutoRenewDuration](/dotnet/api/microsoft.azure.servicebus.messagehandleroptions.maxautorenewduration). В соответствии с документацией по служебной шине максимально допустимое значение для этого параметра равно 5 минутам, тогда как ограничение времени выполнения функций можно увеличить с 5 минут по умолчанию до 10 минут. Этого не стоит делать для функций служебной шины, потому что лимит продления служебной шины будет превышен.

## <a name="message-metadata"></a>Метаданные сообщения

Триггер служебной шины предоставляет несколько [свойств метаданных](./functions-bindings-expressions-patterns.md#trigger-metadata). Эти свойства можно использовать как часть выражений привязки в других привязках или как параметры в коде. Эти свойства являются членами класса [Message](/dotnet/api/microsoft.azure.servicebus.message) .

|Свойство.|Type|Описание|
|--------|----|-----------|
|`ContentType`|`string`|Идентификатор типа содержимого, используемый отправителем и получателем для логики конкретного приложения.|
|`CorrelationId`|`string`|Идентификатор корреляции.|
|`DeadLetterSource`|`string`|Источник недоставленных сообщений.|
|`DeliveryCount`|`Int32`|Число доставок.|
|`EnqueuedTimeUtc`|`DateTime`|Время попадания в очередь в формате UTC.|
|`ExpiresAtUtc`|`DateTime`|Время окончания срока действия в формате UTC.|
|`Label`|`string`|Метка для конкретного приложения.|
|`MessageId`|`string`|Определяемое пользователем значение, используемое служебной шиной для выявления повторяющихся сообщений.|
|`MessageReceiver`|`MessageReceiver`|Получатель сообщений служебной шины. Может использоваться для отмены, завершения или недоставленного сообщения.|
|`MessageSession`|`MessageSession`|Получатель сообщений специально предназначен для очередей и разделов, использующих сеансы.|
|`ReplyTo`|`string`|Адрес очереди для ответа.|
|`SequenceNumber`|`long`|Уникальный номер, назначенный сообщению службой "Служебная шина".|
|`To`|`string`|Адрес для отправки.|
|`UserProperties`|`IDictionary<string, object>`|Свойства, заданные отправителем.|

См. [примеры кода](#example), в которых используются эти свойства, в предыдущих разделах этой статьи.

## <a name="next-steps"></a>Дальнейшие действия

- [Отправка сообщений служебной шины Azure из функций Azure (Выходная привязка)](./functions-bindings-service-bus-output.md)
