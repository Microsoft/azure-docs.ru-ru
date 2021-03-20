---
title: Выражения и шаблоны привязок функций Azure
description: Научитесь создавать различные выражения привязки функций Azure на основе общих шаблонов.
author: craigshoemaker
ms.topic: reference
ms.custom: devx-track-csharp
ms.date: 02/18/2019
ms.author: cshoe
ms.openlocfilehash: 7245b0c0fb1e96959ef5dca4992cf52a38accb58
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98920296"
---
# <a name="azure-functions-binding-expression-patterns"></a>Шаблоны выражений привязки функций Azure

Одной из самых эффективных функций [триггеров и привязок](./functions-triggers-bindings.md) являются *выражения привязки*. В файле *function.json* и в параметрах функции и коде можно использовать выражения, которые разрешаются в значения из различных источников.

Большинство выражений определяются путем их заключения в фигурные скобки. Например, в функции триггера очереди `{queueTrigger}` разрешается в текст сообщения очереди. Если свойство `path` для выходной привязки большого двоичного объекта — `container/{queueTrigger}`, а функция активируется сообщением очереди `HelloWorld`, создается большой двоичный объект с именем `HelloWorld`.

Типы выражений привязки

* [Параметры приложения](#binding-expressions---app-settings)
* [Имя файла триггера](#trigger-file-name)
* [Метаданные триггера](#trigger-metadata)
* [Полезные данные JSON](#json-payloads)
* [Новый GUID](#create-guids)
* [Текущие дата и время](#current-time)

## <a name="binding-expressions---app-settings"></a>Выражения привязки. Параметры приложения

Для управления секретами и строками подключения рекомендуется использовать параметры приложения, а не файлы конфигурации. Это ограничивает доступ к таким секретам и обеспечивает безопасное хранение файлов, таких как *function.json*, в общедоступном репозитории системы управления версиями.

Параметры приложений также удобно использовать при необходимости изменить конфигурации в соответствии со средой. Например, в тестовой среде можно отслеживать другую очередь или контейнер хранилища BLOB-объектов.

Выражения привязки параметров приложения определяются иначе, чем другие выражения привязки. Они заключены в символы процента, а не в фигурные скобки. Например, если путь выходной привязки большого двоичного объекта — `%Environment%/newblob.txt`, а значение параметра приложения `Environment` — `Development`, в контейнере `Development` будет создан большой двоичный объект.

Если функция выполняется локально, значения параметра приложения поступают из файла *local.settings.json*.

> [!NOTE]
> `connection`Свойство триггеров и привязок является особым случаем и автоматически разрешает значения в виде параметров приложения без знаков процента. 

Следующий пример представляет собой триггер хранилища очередей Azure. Этот триггер использует параметр приложения `%input_queue_name%`, чтобы определить очередь, для которой он должен срабатывать.

```json
{
  "bindings": [
    {
      "name": "order",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "%input_queue_name%",
      "connection": "MY_STORAGE_ACCT_APP_SETTING"
    }
  ]
}
```

Тот же подход можно использовать в библиотеках классов:

```csharp
[FunctionName("QueueTrigger")]
public static void Run(
    [QueueTrigger("%input_queue_name%")]string myQueueItem, 
    ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
}
```

## <a name="trigger-file-name"></a>Имя файла триггера

`path` для триггера большого двоичного объекта может быть шаблоном, который позволяет ссылаться на имя большого двоичного объекта, активирующего триггер, в других привязках и коде функции. Шаблон может также включать критерии фильтрации, которые определяют, какие большие двоичные объекты могут активировать вызов функции.

Например, в следующей привязке триггера большого двоичного объекта шаблон `path` — `sample-images/{filename}`, который создает выражение привязки с именем `filename`:

```json
{
  "bindings": [
    {
      "name": "image",
      "type": "blobTrigger",
      "path": "sample-images/{filename}",
      "direction": "in",
      "connection": "MyStorageConnection"
    },
    ...
```

Затем выражение `filename` можно использовать в выходной привязке для указания имени создаваемого большого двоичного объекта:

```json
    ...
    {
      "name": "imageSmall",
      "type": "blob",
      "path": "sample-images-sm/{filename}",
      "direction": "out",
      "connection": "MyStorageConnection"
    }
  ],
}
```

Код функции получает доступ к тому же значению, используя `filename` в качестве имени параметра:

```csharp
// C# example of binding to {filename}
public static void Run(Stream image, string filename, Stream imageSmall, ILogger log)  
{
    log.LogInformation($"Blob trigger processing: {filename}");
    // ...
} 
```

<!--TODO: add JavaScript example -->
<!-- Blocked by bug https://github.com/Azure/Azure-Functions/issues/248 -->

Возможность применять выражения и шаблоны привязки распространяется и на атрибуты в библиотеке классов. В следующем примере параметры конструктора атрибута совпадают со значениями `path` в предыдущих примерах *function.json*: 

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{filename}")] Stream image,
    [Blob("sample-images-sm/{filename}", FileAccess.Write)] Stream imageSmall,
    string filename,
    ILogger log)
{
    log.LogInformation($"Blob trigger processing: {filename}");
    // ...
}

```

Можно также создать выражения для частей имени файла. В следующем примере функция активируется только для имен файлов, соответствующих шаблону: `anyname-anyfile.csv`

```json
{
    "name": "myBlob",
    "type": "blobTrigger",
    "direction": "in",
    "path": "testContainerName/{date}-{filetype}.csv",
    "connection": "OrderStorageConnection"
}
```

Дополнительные сведения о том, как использовать выражения и шаблоны в строке пути к большому двоичному объекту, см. в статье [Привязки хранилища BLOB-объектов Azure для службы "Функции Azure"](functions-bindings-storage-blob.md).

## <a name="trigger-metadata"></a>Метаданные триггера

Помимо полезных данных, предоставляемых триггером (например, содержимое сообщения очереди, инициирующего вызов функции), многие триггеры предоставляют также дополнительные значения метаданных. Эти значения можно использовать в качестве входных параметров в C# и F# или свойств объекта `context.bindings` в JavaScript. 

Например, триггер хранилища очередей Azure поддерживает следующие свойства:

* QueueTrigger (содержимое активирующего сообщения, если строка допустима)
* DequeueCount
* ExpirationTime
* Идентификатор
* InsertionTime
* NextVisibleTime
* PopReceipt

Значения этих метаданных доступны в свойствах файла *function.json*. Предположим, что вы используете триггер очереди, а сообщение в очереди содержит имя большого двоичного объекта, который вам нужно прочитать. В файле *function.json* укажите свойство `queueTrigger` в метаданных большого двоичного объекта `path`, как показано в следующем примере:

```json
{
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "queueName": "myqueue-items",
      "connection": "MyStorageConnection",
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "direction": "in",
      "connection": "MyStorageConnection"
    }
  ]
}
```

Сведения о свойствах метаданных для каждого триггера приведены в соответствующих статьях документации. Пример см. в разделе [о метаданных триггера очередей](functions-bindings-storage-queue-trigger.md#message-metadata). Документация доступна также на портале на вкладке **Интегрировать** в разделе **Документация** под областью конфигурации привязки.  

## <a name="json-payloads"></a>Полезные данные JSON

Если полезные данные представлены в виде JSON, на его свойства можно ссылаться в конфигурации других привязок в той же функции и ее коде.

В следующем примере показан файл *function.json* для функции веб-перехватчика, который получает имя большого двоичного объекта в JSON: `{"BlobName":"HelloWorld.txt"}`. Входная привязка большого двоичного объекта считывает этот объект, а привязка для вывода HTTP возвращает содержимое большого двоичного объекта в ответе HTTP. Обратите внимание, что входная привязка большого двоичного объекта получает имя такого объекта, обращаясь напрямую к свойству `BlobName` (`"path": "strings/{BlobName}"`).

```json
{
  "bindings": [
    {
      "name": "info",
      "type": "httpTrigger",
      "direction": "in",
      "webHookType": "genericJson"
    },
    {
      "name": "blobContents",
      "type": "blob",
      "direction": "in",
      "path": "strings/{BlobName}",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "res",
      "type": "http",
      "direction": "out"
    }
  ]
}
```

Для этого в C# и F# нужен класс, определяющий поля, которые должны быть десериализованы, как показано в следующем примере:

```csharp
using System.Net;
using Microsoft.Extensions.Logging;

public class BlobInfo
{
    public string BlobName { get; set; }
}
  
public static HttpResponseMessage Run(HttpRequestMessage req, BlobInfo info, string blobContents, ILogger log)
{
    if (blobContents == null) {
        return req.CreateResponse(HttpStatusCode.NotFound);
    } 

    log.LogInformation($"Processing: {info.BlobName}");

    return req.CreateResponse(HttpStatusCode.OK, new {
        data = $"{blobContents}"
    });
}
```

В JavaScript десериализация JSON выполняется автоматически.

```javascript
module.exports = function (context, info) {
    if ('BlobName' in info) {
        context.res = {
            body: { 'data': context.bindings.blobContents }
        }
    }
    else {
        context.res = {
            status: 404
        };
    }
    context.done();
}
```

### <a name="dot-notation"></a>Точечная нотация

Если некоторые свойства в полезных данных JSON являются объектами со свойствами, к ним можно обратиться напрямую с помощью точечной нотации. Например, ваш JSON выглядит следующим образом:

```json
{
  "BlobName": {
    "FileName":"HelloWorld",
    "Extension":"txt"
  }
}
```

На `FileName` можно непосредственно ссылаться, используя формат `BlobName.FileName`. С этим форматом JSON свойство `path` из предыдущего примера будет выглядеть следующим образом:

```json
"path": "strings/{BlobName.FileName}.{BlobName.Extension}",
```

В C# потребовалось бы два класса:

```csharp
public class BlobInfo
{
    public BlobName BlobName { get; set; }
}
public class BlobName
{
    public string FileName { get; set; }
    public string Extension { get; set; }
}
```

## <a name="create-guids"></a>Создание глобальных уникальных идентификаторов

Выражение привязки `{rand-guid}` создает идентификатор GUID. Следующий путь к большому двоичному объекту в файле `function.json` создает большой двоичный объект с именем следующего вида: *50710cb5-84b9-4d87-9d83-a03d6976a682.txt*.

```json
{
  "type": "blob",
  "name": "blobOutput",
  "direction": "out",
  "path": "my-output-container/{rand-guid}.txt"
}
```

## <a name="current-time"></a>Текущее время

Выражение привязки `DateTime` разрешается в `DateTime.UtcNow`. Следующий путь к большому двоичному объекту в файле `function.json` создает большой двоичный объект с именем следующего вида: *2018-02-16T17-59-55Z.txt*.

```json
{
  "type": "blob",
  "name": "blobOutput",
  "direction": "out",
  "path": "my-output-container/{DateTime}.txt"
}
```
## <a name="binding-at-runtime"></a>Привязка во время выполнения

В C# и других языках .NET можно использовать шаблон императивной привязки, которая отличается от декларативной привязки в файле *function.json* или в атрибутах. Императивную привязку удобно использовать, когда параметры привязки должны вычисляться не при проектировании, а во время выполнения. Дополнительные сведения см. в [справочнике разработчика C#](functions-dotnet-class-library.md#binding-at-runtime) и [справочнике разработчика скриптов C#](functions-reference-csharp.md#binding-at-runtime).

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Использование возвращаемого значения функции Azure](./functions-bindings-return-value.md)
