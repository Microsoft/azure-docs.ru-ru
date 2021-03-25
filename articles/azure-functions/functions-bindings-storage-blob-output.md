---
title: Выходная привязка хранилища BLOB-объектов Azure для функций Azure
description: Узнайте, как предоставить данные выходной привязки хранилища BLOB-объектов Azure функции Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.custom: devx-track-csharp, devx-track-python
ms.openlocfilehash: 78cbf94bfc19757a4264c7884c3e47b230de5e46
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105044096"
---
# <a name="azure-blob-storage-output-binding-for-azure-functions"></a>Выходная привязка хранилища BLOB-объектов Azure для функций Azure

Выходная привязка позволяет изменять и удалять данные хранилища BLOB-объектов в функции Azure.

Сведения об установке и настройке см. в [этой обзорной статье](./functions-bindings-storage-blob.md).

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

Ниже приведен пример [функции C#](functions-dotnet-class-library.md), которая использует триггер большого двоичного объекта и две выходные привязки больших двоичных объектов. Эта функция активируется путем создания большого двоичного объекта образа в контейнере *sample-images*. Она создает небольшие и средние копии большого двоичного объекта образа.

```csharp
using System.Collections.Generic;
using System.IO;
using Microsoft.Azure.WebJobs;
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Formats;
using SixLabors.ImageSharp.PixelFormats;
using SixLabors.ImageSharp.Processing;

public class ResizeImages
{
    [FunctionName("ResizeImage")]
    public static void Run([BlobTrigger("sample-images/{name}")] Stream image,
        [Blob("sample-images-sm/{name}", FileAccess.Write)] Stream imageSmall,
        [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageMedium)
    {
        IImageFormat format;

        using (Image<Rgba32> input = Image.Load<Rgba32>(image, out format))
        {
            ResizeImage(input, imageSmall, ImageSize.Small, format);
        }

        image.Position = 0;
        using (Image<Rgba32> input = Image.Load<Rgba32>(image, out format))
        {
            ResizeImage(input, imageMedium, ImageSize.Medium, format);
        }
    }

    public static void ResizeImage(Image<Rgba32> input, Stream output, ImageSize size, IImageFormat format)
    {
        var dimensions = imageDimensionsTable[size];

        input.Mutate(x => x.Resize(dimensions.Item1, dimensions.Item2));
        input.Save(output, format);
    }

    public enum ImageSize { ExtraSmall, Small, Medium }

    private static Dictionary<ImageSize, (int, int)> imageDimensionsTable = new Dictionary<ImageSize, (int, int)>() {
        { ImageSize.ExtraSmall, (320, 200) },
        { ImageSize.Small,      (640, 400) },
        { ImageSize.Medium,     (800, 600) }
    };

}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

<!--Same example for input and output. -->

В примере ниже показаны входная и выходная привязки больших двоичных объектов в файле *function.json*, а также код [скрипта C# (CSX)](functions-reference-csharp.md), который использует эти привязки. Функция копирует большой двоичный объект. Она активируется сообщением очереди, содержащим имя копируемого большого двоичного объекта. Новый большой двоичный объект называется *{оригиналблобнаме}-Copy*.

В файле *function.json* свойство метаданных `queueTrigger` используется для указания имени большого двоичного объекта в свойствах `path`:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже приведен код скрипта C#.

```cs
public static void Run(string myQueueItem, string myInputBlob, out string myOutputBlob, ILogger log)
{
    log.LogInformation($"C# Queue trigger function processed: {myQueueItem}");
    myOutputBlob = myInputBlob;
}
```

# <a name="java"></a>[Java](#tab/java)

Этот раздел содержит следующие примеры.

* [Триггер HTTP, использование OutputBinding](#http-trigger-using-outputbinding-java)
* [Триггер очереди, использование возвращаемого значения функции](#queue-trigger-using-function-return-value-java)

#### <a name="http-trigger-using-outputbinding-java"></a>Триггер HTTP, использование OutputBinding (Java)

 В следующем примере показана функция Java, которая использует заметку `HttpTrigger` для получения параметра, содержащего имя файла в контейнере хранилища BLOB-объектов. После этого заметка `BlobInput` считывает файл и передает его содержимое в функцию как `byte[]`. Заметка `BlobOutput` выполняет привязку к `OutputBinding outputItem`, которая затем используется функцией для записи содержимого входного BLOB-объекта в контейнер настраиваемого хранилища.

```java
  @FunctionName("copyBlobHttp")
  @StorageAccount("Storage_Account_Connection_String")
  public HttpResponseMessage copyBlobHttp(
    @HttpTrigger(name = "req", 
      methods = {HttpMethod.GET}, 
      authLevel = AuthorizationLevel.ANONYMOUS) 
    HttpRequestMessage<Optional<String>> request,
    @BlobInput(
      name = "file", 
      dataType = "binary", 
      path = "samples-workitems/{Query.file}") 
    byte[] content,
    @BlobOutput(
      name = "target", 
      path = "myblob/{Query.file}-CopyViaHttp")
    OutputBinding<String> outputItem,
    final ExecutionContext context) {
      // Save blob to outputItem
      outputItem.setValue(new String(content, StandardCharsets.UTF_8));

      // build HTTP response with size of requested blob
      return request.createResponseBuilder(HttpStatus.OK)
        .body("The size of \"" + request.getQueryParameters().get("file") + "\" is: " + content.length + " bytes")
        .build();
  }
```

#### <a name="queue-trigger-using-function-return-value-java"></a>Триггер очереди, использование возвращаемого значения функции (Java)

 В следующем примере показана функция Java, которая использует заметку `QueueTrigger` для получения сообщения, содержащего имя файла в контейнере хранилища BLOB-объектов. После этого заметка `BlobInput` считывает файл и передает его содержимое в функцию как `byte[]`. Заметка `BlobOutput` выполняет привязку к возвращаемому значению функции, которая затем используется средой выполнения для записи содержимого входного BLOB-объекта в контейнер настраиваемого хранилища.

```java
  @FunctionName("copyBlobQueueTrigger")
  @StorageAccount("Storage_Account_Connection_String")
  @BlobOutput(
    name = "target", 
    path = "myblob/{queueTrigger}-Copy")
  public String copyBlobQueue(
    @QueueTrigger(
      name = "filename", 
      dataType = "string",
      queueName = "myqueue-items") 
    String filename,
    @BlobInput(
      name = "file", 
      path = "samples-workitems/{queueTrigger}") 
    String content,
    final ExecutionContext context) {
      context.getLogger().info("The content of \"" + filename + "\" is: " + content);
      return content;
  }
```

 В [библиотеке среды выполнения функций Java](/java/api/overview/azure/functions/runtime) используйте заметку `@BlobOutput` для параметров функции, значение которых будут записываться в хранилище больших двоичных объектов.  Параметр должен быть типа `OutputBinding<T>`, где T — любой собственный тип Java или POJO.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

<!--Same example for input and output. -->

В примере ниже показаны входная и выходная привязки BLOB-объектов в файле *function.json*, а также код [скрипта JavaScript](functions-reference-node.md), который использует эти привязки. Эта функция копирует большой двоичный объект. Она активируется сообщением очереди, содержащим имя копируемого большого двоичного объекта. Новый большой двоичный объект называется *{оригиналблобнаме}-Copy*.

В файле *function.json* свойство метаданных `queueTrigger` используется для указания имени большого двоичного объекта в свойствах `path`:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "myInputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже показан код JavaScript.

```javascript
module.exports = function(context) {
    context.log('Node.js Queue trigger function processed', context.bindings.myQueueItem);
    context.bindings.myOutputBlob = context.bindings.myInputBlob;
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показано, как создать копию входящего большого двоичного объекта в качестве выходных данных [функции PowerShell](functions-reference-powershell.md).

В файле конфигурации функции (*function.json*) `trigger` свойство Metadata используется для указания имени выходного большого двоичного объекта в `path` свойствах.

> [!NOTE]
> Чтобы избежать бесконечных циклов, убедитесь, что входные и выходные пути различаются.

```json
{
  "bindings": [
    {
      "name": "myInputBlob",
      "path": "data/{trigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in",
      "type": "blobTrigger"
    },
    {
      "name": "myOutputBlob",
      "type": "blob",
      "path": "data/copy/{trigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false
}
```

Вот код PowerShell:

```powershell
# Input bindings are passed in via param block.
param([byte[]] $myInputBlob, $TriggerMetadata)
Write-Host "PowerShell Blob trigger function Processed blob Name: $($TriggerMetadata.Name)"
Push-OutputBinding -Name myOutputBlob -Value $myInputBlob
```

# <a name="python"></a>[Python](#tab/python)

<!--Same example for input and output. -->

В приведенном ниже примере показаны входная и выходная привязки больших двоичных объектов в файле *function.json* и [код Python](functions-reference-python.md), который использует эти привязки. Эта функция копирует большой двоичный объект. Она активируется сообщением очереди, содержащим имя копируемого большого двоичного объекта. Новый большой двоичный объект называется *{оригиналблобнаме}-Copy*.

В файле *function.json* свойство метаданных `queueTrigger` используется для указания имени большого двоичного объекта в свойствах `path`:

```json
{
  "bindings": [
    {
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting",
      "name": "queuemsg",
      "type": "queueTrigger",
      "direction": "in"
    },
    {
      "name": "inputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "in"
    },
    {
      "name": "outputblob",
      "type": "blob",
      "path": "samples-workitems/{queueTrigger}-Copy",
      "connection": "MyStorageConnectionAppSetting",
      "direction": "out"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

В разделе [Конфигурация](#configuration) описываются эти свойства.

Ниже приведен код Python.

```python
import logging
import azure.functions as func


def main(queuemsg: func.QueueMessage, inputblob: func.InputStream,
         outputblob: func.Out[func.InputStream]):
    logging.info('Python Queue trigger function processed %s', inputblob.name)
    outputblob.set(inputblob)
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте [BlobAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/dev/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobAttribute.cs).

Конструктор атрибута принимает путь к большому двоичному объекту и параметр `FileAccess`, указывающий на чтение или запись, как показано в следующем примере:

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
{
    ...
}
```

Чтобы указать учетную запись хранения, которую нужно использовать, можно задать свойство `Connection`, как показано в следующем примере.

```csharp
[FunctionName("ResizeImage")]
public static void Run(
    [BlobTrigger("sample-images/{name}")] Stream image,
    [Blob("sample-images-md/{name}", FileAccess.Write, Connection = "StorageConnectionAppSetting")] Stream imageSmall)
{
    ...
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

`@BlobOutput`Атрибут предоставляет доступ к большому двоичному объекту, вызвавшему функцию. Если используется массив байтов с атрибутом, задайте для значение `dataType` `binary` . Дополнительные сведения см. в [примере выходных данных](#example) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

Полный пример см. в разделе [пример выходных данных](#example).

Чтобы указать учетную запись хранения на уровне класса, метода или параметра, можно использовать атрибут `StorageAccount`. Дополнительные сведения см. в разделе [Атрибуты триггера для предкомпилированного кода C#](./functions-bindings-storage-blob-trigger.md#attributes-and-annotations).

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json* и атрибуте `Blob`.

|свойство function.json | Свойство атрибута |Описание|
|---------|---------|----------------------|
|**type** | Недоступно | Нужно задать значение `blob`. |
|**direction** | Недоступно | Нужно задать значение `out` для выходной привязки. Исключения приведены в этом [разделе](#usage). |
|**name** | Недоступно | Имя переменной, представляющей большой двоичный объект в коде функции.  Задайте значение `$return`, ссылающееся на возвращаемое значение функции.|
|**путь** |**BlobPath** | Путь к контейнеру больших двоичных объектов. |
|**connection**; |**Соединение**| Имя параметра приложения, содержащего строку подключения к службе хранилища, используемой для этой привязки. Если имя параметра приложения начинается с AzureWebJobs, можно указать только остальную часть имени. Например, если задать для `connection` значение MyStorage, среда выполнения службы "Функции" будет искать параметр приложения с именем AzureWebJobsMyStorage. Если оставить строку `connection` пустой, среда выполнения службы "Функции" будет использовать строку подключения к службе хранилища по умолчанию для параметра приложения с именем `AzureWebJobsStorage`.<br><br>Строка подключения необходима для учетной записи хранения общего назначения, а не [учетной записи хранения только для больших двоичных объектов](../storage/common/storage-account-overview.md#types-of-storage-accounts).<br><br>Если вы используете [расширение версии 5. x или более поздней](./functions-bindings-storage-blob.md#storage-extension-5x-and-higher), а не строку подключения, можно указать ссылку на раздел конфигурации, который определяет подключение. См. раздел [Подключения](./functions-reference.md#connections).|
|Недоступно | **Доступ** | Указывает, какая операция будет выполняться (запись или чтение). |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-output-usage.md](../../includes/functions-bindings-blob-storage-output-usage.md)]

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-output-usage.md](../../includes/functions-bindings-blob-storage-output-usage.md)]

# <a name="java"></a>[Java](#tab/java)

`@BlobOutput`Атрибут предоставляет доступ к большому двоичному объекту, вызвавшему функцию. Если используется массив байтов с атрибутом, задайте для значение `dataType` `binary` . Дополнительные сведения см. в [примере выходных данных](#example) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Получите доступ к данным большого двоичного объекта с помощью `context.bindings.<BINDING_NAME>` , где имя привязки определяется в _function.js_ в файле.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Получите доступ к данным большого двоичного объекта с помощью параметра, совпадающего с именем, назначенным параметром имени привязки в _function.js_ в файле.

# <a name="python"></a>[Python](#tab/python)

Параметры функции можно объявить как следующие типы для записи в хранилище BLOB-объектов:

* Строки как `func.Out[str]`
* Потоки как `func.Out[func.InputStream]`

Дополнительные сведения см. в [примере выходных данных](#example) .

---

## <a name="exceptions-and-return-codes"></a>Исключения и коды возврата

| Привязка |  Справочник |
|---|---|
| BLOB-объект | [Коды ошибок больших двоичных объектов](/rest/api/storageservices/fileservices/blob-service-error-codes) |
| Большой двоичный объект, таблица, очередь |  [Коды ошибок хранилища](/rest/api/storageservices/fileservices/common-rest-api-error-codes) |
| Большой двоичный объект, таблица, очередь |  [Устранение неполадок](/rest/api/storageservices/fileservices/troubleshooting-api-operations) |

## <a name="next-steps"></a>Следующие шаги

- [Выполнение функции при изменении данных хранилища BLOB-объектов](./functions-bindings-storage-blob-trigger.md)
- [Чтение данных хранилища BLOB-объектов при выполнении функции](./functions-bindings-storage-blob-input.md)
