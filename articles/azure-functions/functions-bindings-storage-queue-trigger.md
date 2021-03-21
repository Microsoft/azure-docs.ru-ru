---
title: Триггер хранилища очередей Azure для функций Azure
description: Узнайте, как выполнять функции Azure в качестве изменений данных хранилища очередей Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/18/2020
ms.author: cshoe
ms.custom: devx-track-csharp, cc996988-fb4f-47, devx-track-python
ms.openlocfilehash: f4477a09f151695b826d0becf28e92ceaf3f9e85
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102453212"
---
# <a name="azure-queue-storage-trigger-for-azure-functions"></a>Триггер хранилища очередей Azure для функций Azure

Триггер хранилища очередей запускает функцию, так как сообщения добавляются в хранилище очередей Azure.

## <a name="encoding"></a>Кодирование

Функции ожидают строку в кодировке *base64*. Любая корректировка типа кодирования (для подготовки данных в виде строки в кодировке *base64*) должна быть реализована в вызывающей службе.

## <a name="example"></a>Пример

Используйте триггер очереди для запуска функции при получении нового элемента очереди. Сообщение очереди предоставляется в качестве входных данных функции.

# <a name="c"></a>[C#](#tab/csharp)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая выполняет опрос очереди `myqueue-items`, а затем делает запись в журнал при каждой обработке элемента очереди.

```csharp
public static class QueueFunctions
{
    [FunctionName("QueueTrigger")]
    public static void QueueTrigger(
        [QueueTrigger("myqueue-items")] string myQueueItem, 
        ILogger log)
    {
        log.LogInformation($"C# function processed: {myQueueItem}");
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показана привязка триггера очереди в файле *function.json* и коде [сценария C# (CSX)](functions-reference-csharp.md), который использует привязку. Эта функция выполняет опрос очереди `myqueue-items`, а затем делает запись в журнал при каждой обработке элемента очереди.

Ниже показан файл *function.json*.

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже приведен код скрипта C#.

```csharp
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.Extensions.Logging;
using Microsoft.WindowsAzure.Storage.Queue;
using System;

public static void Run(CloudQueueMessage myQueueItem, 
    DateTimeOffset expirationTime, 
    DateTimeOffset insertionTime, 
    DateTimeOffset nextVisibleTime,
    string queueTrigger,
    string id,
    string popReceipt,
    int dequeueCount,
    ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem.AsString}\n" +
        $"queueTrigger={queueTrigger}\n" +
        $"expirationTime={expirationTime}\n" +
        $"insertionTime={insertionTime}\n" +
        $"nextVisibleTime={nextVisibleTime}\n" +
        $"id={id}\n" +
        $"popReceipt={popReceipt}\n" + 
        $"dequeueCount={dequeueCount}");
}
```

В этом [разделе](#usage) показано свойство `myQueueItem`, имя которому назначает свойство `name` в function.json.  В разделе [Метаданные сообщений](#message-metadata) показаны все остальные переменные.

# <a name="java"></a>[Java](#tab/java)

В следующем примере Java показана функция триггера очереди хранилища, которая записывает в журнал активируемое сообщение, помещенное в очередь `myqueuename` .

 ```java
 @FunctionName("queueprocessor")
 public void run(
    @QueueTrigger(name = "msg",
                   queueName = "myqueuename",
                   connection = "myconnvarname") String message,
     final ExecutionContext context
 ) {
     context.getLogger().info(message);
 }
 ```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана привязка триггера очереди в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует привязку. Эта функция выполняет опрос очереди `myqueue-items`, а затем делает запись в журнал при каждой обработке элемента очереди.

Ниже показан файл *function.json*.

```json
{
    "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        }
    ]
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

> [!NOTE]
> Параметр name отображается как `context.bindings.<name>` в коде JavaScript, который содержит полезные данные элемента очереди. Эти полезные данные также передаются в качестве второго параметра в функцию.

Ниже показан код JavaScript.

```javascript
module.exports = async function (context, message) {
    context.log('Node.js queue trigger function processed work item', message);
    // OR access using context.bindings.<name>
    // context.log('Node.js queue trigger function processed work item', context.bindings.myQueueItem);
    context.log('expirationTime =', context.bindingData.expirationTime);
    context.log('insertionTime =', context.bindingData.insertionTime);
    context.log('nextVisibleTime =', context.bindingData.nextVisibleTime);
    context.log('id =', context.bindingData.id);
    context.log('popReceipt =', context.bindingData.popReceipt);
    context.log('dequeueCount =', context.bindingData.dequeueCount);
    context.done();
};
```

В этом [разделе](#usage) показано свойство `myQueueItem`, имя которому назначает свойство `name` в function.json.  В разделе [Метаданные сообщений](#message-metadata) показаны все остальные переменные.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показано, как прочитать сообщение очереди, переданное в функцию через триггер.

Триггер очереди хранилища определяется в *function.jsв* файле `type` , где имеет значение `queueTrigger` .

```json
{
  "bindings": [
    {
      "name": "QueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "messages",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

Код в файле *Run.ps1* объявляет параметр как `$QueueItem` , который позволяет считывать сообщение очереди в функции.

```powershell
# Input bindings are passed in via param block.
param([string] $QueueItem, $TriggerMetadata)

# Write out the queue message and metadata to the information log.
Write-Host "PowerShell queue trigger function processed work item: $QueueItem"
Write-Host "Queue item expiration time: $($TriggerMetadata.ExpirationTime)"
Write-Host "Queue item insertion time: $($TriggerMetadata.InsertionTime)"
Write-Host "Queue item next visible time: $($TriggerMetadata.NextVisibleTime)"
Write-Host "ID: $($TriggerMetadata.Id)"
Write-Host "Pop receipt: $($TriggerMetadata.PopReceipt)"
Write-Host "Dequeue count: $($TriggerMetadata.DequeueCount)"
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показано, как прочитать сообщение очереди, переданное в функцию через триггер.

Триггер очереди хранилища определяется в *function.js* , где *тип* имеет значение `queueTrigger` .

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "name": "msg",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "messages",
      "connection": "AzureStorageQueuesConnectionString"
    }
  ]
}
```

В коде *_\_ init_ \_ . Корректировка* объявляется параметр `func.QueueMessage` , который позволяет считывать сообщение очереди в функции.

```python
import logging
import json

import azure.functions as func

def main(msg: func.QueueMessage):
    logging.info('Python queue trigger function processed a queue item.')

    result = json.dumps({
        'id': msg.id,
        'body': msg.get_body().decode('utf-8'),
        'expiration_time': (msg.expiration_time.isoformat()
                            if msg.expiration_time else None),
        'insertion_time': (msg.insertion_time.isoformat()
                           if msg.insertion_time else None),
        'time_next_visible': (msg.time_next_visible.isoformat()
                              if msg.time_next_visible else None),
        'pop_receipt': msg.pop_receipt,
        'dequeue_count': msg.dequeue_count
    })

    logging.info(result)
```

 ---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте следующие атрибуты для настройки триггера очереди:

* [QueueTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Queues/QueueTriggerAttribute.cs)

  Конструктор атрибута принимает имя очереди для отслеживания, как показано в следующем примере:

  ```csharp
  [FunctionName("QueueTrigger")]
  public static void Run(
      [QueueTrigger("myqueue-items")] string myQueueItem, 
      ILogger log)
  {
      ...
  }
  ```

  Можно задать `Connection` свойство, чтобы указать параметр приложения, который содержит используемую строку подключения к учетной записи хранения, как показано в следующем примере:

  ```csharp
  [FunctionName("QueueTrigger")]
  public static void Run(
      [QueueTrigger("myqueue-items", Connection = "StorageConnectionAppSetting")] string myQueueItem, 
      ILogger log)
  {
      ....
  }
  ```

  Полный пример см. [здесь](#example).

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  Предоставляет еще один способ указать используемую учетную запись хранения. Конструктор принимает имя параметра приложения, содержащего строку подключения к службе хранилища. Атрибут может применяться на уровне класса, метода или параметра. Ниже показан пример уровня класса и метода.

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("QueueTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ...
  }
  ```

Используемая учетная запись хранения определяется в следующем порядке:

* Свойство `QueueTrigger` атрибута `Connection`.
* Атрибут `StorageAccount`, примененный к тому же параметру, что и `QueueTrigger`.
* Атрибут `StorageAccount`, примененный к функции.
* Атрибут `StorageAccount`, примененный к классу.
* Параметр приложения AzureWebJobsStorage.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

`QueueTrigger`Заметка предоставляет доступ к очереди, которая запускает функцию. В следующем примере сообщение очереди становится доступным для функции через `message` параметр.

```java
package com.function;
import com.microsoft.azure.functions.annotation.*;
import java.util.Queue;
import com.microsoft.azure.functions.*;

public class QueueTriggerDemo {
    @FunctionName("QueueTriggerDemo")
    public void run(
        @QueueTrigger(name = "message", queueName = "messages", connection = "MyStorageConnectionAppSetting") String message,
        final ExecutionContext context
    ) {
        context.getLogger().info("Queue message: " + message);
    }
}
```

| Свойство.    | Описание |
|-------------|-----------------------------|
|`name`       | Объявляет имя параметра в сигнатуре функции. При активации функции значение этого параметра будет иметь содержимое сообщения очереди. |
|`queueName`  | Объявляет имя очереди в учетной записи хранения. |
|`connection` | Указывает строку подключения к учетной записи хранения. |

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `QueueTrigger`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно| Нужно задать значение `queueTrigger`. Это свойство задается автоматически при создании триггера на портале Azure.|
|**direction**| Недоступно | Только в файле *function.json*. Нужно задать значение `in`. Это свойство задается автоматически при создании триггера на портале Azure. |
|**name** | Недоступно |Имя переменной, содержащей полезные данные элемента очереди в коде функции.  |
|**queueName** | **QueueName**| Имя очереди для опроса. |
|**connection**; | **Соединение** |Имя параметра приложения, содержащего строку подключения к службе хранилища, используемой для этой привязки. Если имя параметра приложения начинается с AzureWebJobs, можно указать только остальную часть имени.<br><br>Например, если задано значение `connection` "MyStorage", среда выполнения функций ищет параметр приложения с именем "MyStorage". Если оставить строку `connection` пустой, среда выполнения службы "Функции" будет использовать строку подключения к службе хранилища по умолчанию для параметра приложения с именем `AzureWebJobsStorage`.<br><br>Если вы используете [расширение версии 5. x или более поздней](./functions-bindings-storage-queue.md#storage-extension-5x-and-higher), а не строку подключения, можно указать ссылку на раздел конфигурации, который определяет подключение. См. раздел [Подключения](./functions-reference.md#connections).|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

### <a name="default"></a>По умолчанию

Получите доступ к данным сообщения с помощью параметра метода, например `string paramName` . Вы можете выполнить привязку к одному из следующих типов:

* Объект. Среда выполнения службы "Функции" десериализует полезные данные JSON в экземпляр произвольного класса, определенного в коде.
* `string`
* `byte[]`
* [CloudQueueMessage]

Если при попытке привязать к `CloudQueueMessage` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](functions-bindings-storage-queue.md#azure-storage-sdk-version-in-functions-1x).

### <a name="additional-types"></a>Дополнительные типы

В приложениях, в которых используется[расширение службы хранилища версии 5.0.0 или выше](./functions-bindings-storage-queue.md#storage-extension-5x-and-higher), также могут использоваться типы из [пакета Azure SDK для .NET](/dotnet/api/overview/azure/storage.queues-readme). В этой версии прекращена поддержка устаревшего типа `CloudQueueMessage`. Вместо него теперь поддерживаются следующие типы:

- [куеуемессаже](/dotnet/api/azure.storage.queues.models.queuemessage)
 
Примеры использования этих типов см. в разделе [репозитория GitHub для расширения](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Queues#examples).

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

### <a name="default"></a>По умолчанию

Получите доступ к данным сообщения с помощью параметра метода, например `string paramName` . `paramName`— Это значение, указанное в `name` свойстве *function.json*. Вы можете выполнить привязку к одному из следующих типов:

* Объект. Среда выполнения службы "Функции" десериализует полезные данные JSON в экземпляр произвольного класса, определенного в коде. 
* `string`
* `byte[]`
* [CloudQueueMessage]

Если при попытке привязать к `CloudQueueMessage` вы получаете сообщение об ошибке, убедитесь, что у вас есть ссылка на [правильную версию пакета SDK для службы хранилища](functions-bindings-storage-queue.md#azure-storage-sdk-version-in-functions-1x).

### <a name="additional-types"></a>Дополнительные типы

В приложениях, в которых используется[расширение службы хранилища версии 5.0.0 или выше](./functions-bindings-storage-queue.md#storage-extension-5x-and-higher), также могут использоваться типы из [пакета Azure SDK для .NET](/dotnet/api/overview/azure/storage.queues-readme). В этой версии прекращена поддержка устаревшего типа `CloudQueueMessage`. Вместо него теперь поддерживаются следующие типы:

- [куеуемессаже](/dotnet/api/azure.storage.queues.models.queuemessage)

Примеры использования этих типов см. в разделе [репозитория GitHub для расширения](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Microsoft.Azure.WebJobs.Extensions.Storage.Queues#examples).

# <a name="java"></a>[Java](#tab/java)

Заметка [QueueTrigger](/java/api/com.microsoft.azure.functions.annotation.queuetrigger) предоставляет доступ к сообщению очереди, вызвавшему функцию.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Полезные данные элемента очереди доступны через `context.bindings.<NAME>` , где `<NAME>` совпадает с именем, определенным в *function.json*. Если полезная нагрузка — JSON, значение десериализуется в объект.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Доступ к сообщению очереди через параметр String, который соответствует имени, назначенному `name` параметром привязки в *function.js* в файле.

# <a name="python"></a>[Python](#tab/python)

Доступ к сообщению очереди с помощью параметра, типизированного как [куеуемессаже](/python/api/azure-functions/azure.functions.queuemessage).

---

## <a name="message-metadata"></a>Метаданные сообщения

Триггер очереди предоставляет несколько [свойств метаданных](./functions-bindings-expressions-patterns.md#trigger-metadata). Эти свойства можно использовать как часть выражений привязки в других привязках или как параметры в коде. Свойства являются членами класса [CloudQueueMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage) .

|Свойство.|Тип|Описание|
|--------|----|-----------|
|`QueueTrigger`|`string`|Полезные данные очереди (если это допустимая строка). Если полезная нагрузка сообщения очереди является строкой, `QueueTrigger` то она имеет то же значение, что и переменная, именованная `name` свойством в *function.json*.|
|`DequeueCount`|`int`|Количество раз, когда сообщение было выведено из очереди.|
|`ExpirationTime`|`DateTimeOffset`|Время истечения срока действия сообщения.|
|`Id`|`string`|Идентификатор сообщения в очереди.|
|`InsertionTime`|`DateTimeOffset`|Время, когда сообщение было добавлено в очередь.|
|`NextVisibleTime`|`DateTimeOffset`|Время, когда сообщение станет видимым в следующий раз.|
|`PopReceipt`|`string`|Уведомление о получении сообщения.|

## <a name="poison-messages"></a>Подозрительные сообщения

При сбое функции триггера очереди по умолчанию служба "Функции Azure" выполняет ее для заданного сообщения очереди еще пять раз (включая первую попытку). Если все пять попыток завершаются ошибкой, среда выполнения функций добавляет сообщение в очередь с именем *&lt; originalqueuename>-подозрительным*. Можно написать функции для обработки сообщений из очереди подозрительных сообщений путем внесения их в журнал или отправки уведомления о необходимости ручного вмешательства.

Для обработки сообщений о сбоях вручную проверьте значение [dequeueCount](#message-metadata) сообщения очереди.

## <a name="polling-algorithm"></a>Алгоритм опроса

В триггере очереди реализован алгоритм случайной экспоненциальной отсрочки, который позволяет уменьшить влияние опроса очереди ожидающих задач на затраты на транзакции хранилища.

Алгоритм использует следующую логику:

- При обнаружении сообщения среда выполнения ожидает две секунды, а затем проверяет наличие другого сообщения.
- Если сообщение не найдено, оно ждет около четырех секунд, прежде чем повторять попытку.
- После последующих неудачных попыток получения сообщения очереди время ожидания продолжает увеличиваться, пока не достигнет максимального времени ожидания, по умолчанию — одна минута.
- Максимальное время ожидания настраивается при помощи свойства `maxPollingInterval` в [файле host.json](functions-host-json-v1.md#queues).

Для локальной разработки максимальный интервал опроса по умолчанию равен двум секундам.

В отношении выставления счетов время, затраченное на опрос среды выполнения, является бесплатным и не учитывается в вашей учетной записи.

## <a name="concurrency"></a>Параллелизм

При наличии нескольких сообщений, ожидающих в очереди, триггер очереди извлекает пакет сообщений и в параллельном режиме вызывает экземпляры функций для обработки. По умолчанию в пакете содержится 16 сообщений. Когда число обрабатываемых сообщений снижается до 8, среда выполнения получает следующий пакет и начинает обработку содержащихся в нем сообщений. Поэтому максимальное количество сообщений, одновременно обрабатываемых каждой функцией на одной виртуальной машине, равно 24. Это ограничение применяется отдельно к каждой функции, активируемой с помощью очереди, на каждой виртуальной машине. Если приложение-функция масштабируется на несколько виртуальных машин, каждая машина будет ожидать триггеров и пытаться выполнить функции. Например, если приложение-функция масштабируется на 3 виртуальные машины, то максимальное количество параллельных экземпляров одной функции, активируемой с помощью очереди, равно 72.

Размер пакета и порог для получения нового пакета настраиваются в [файле host.json](functions-host-json.md#queues). Если в приложении-функции требуется свести к минимуму параллельное выполнение функций, активируемых с помощью очереди, вы можете задать для размера пакета значение 1. Этот параметр позволяет исключить параллелизм только при условии, что приложение-функция выполняется на одной виртуальной машине. 

Триггер очереди автоматически предотвращает многократную обработку сообщения очереди функцией одновременно.

## <a name="hostjson-properties"></a>Свойства host.json

В файле [host.json](functions-host-json.md#queues) содержатся параметры, управляющие поведением очереди триггера. Дополнительные сведения о доступных параметрах см. в разделе [host.jsпо параметрам](functions-bindings-storage-queue.md#hostjson-settings) .

## <a name="next-steps"></a>Следующие шаги

- [Сообщения хранилища очереди записи (Выходная привязка)](./functions-bindings-storage-queue-output.md)

<!-- LINKS -->

[CloudQueueMessage]: /dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage
