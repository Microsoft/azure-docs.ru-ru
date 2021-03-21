---
title: Триггер службы "Сетка событий Azure" для Функций Azure
description: Сведения о том, как выполнять код при диспетчеризации событий службы "Сетка событий" в Функциях Azure.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: devx-track-csharp, fasttrack-edit, devx-track-python
ms.openlocfilehash: 886db905008af94b66a902cc551e4d55b36572a8
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98250134"
---
# <a name="azure-event-grid-trigger-for-azure-functions"></a>Триггер службы "Сетка событий Azure" для Функций Azure

Используйте триггер функции для реагирования на событие, отправленное в раздел Сетки событий.

Сведения об установке и настройке см. в [этой обзорной статье](./functions-bindings-event-grid.md).

## <a name="example"></a>Пример

# <a name="c"></a>[C#](#tab/csharp)

Пример триггера HTTP можно изучить в статье [Получение событий через конечную точку HTTP](../event-grid/receive-events.md).

### <a name="c-2x-and-higher"></a>C# (версия 2.x и более поздние)

В следующем примере показана [функция C#](functions-dotnet-class-library.md), которая выполняет привязку к `EventGridEvent`:

```cs
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace Company.Function
{
    public static class EventGridTriggerCSharp
    {
        [FunctionName("EventGridTest")]
        public static void EventGridTest([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.Data.ToString());
        }
    }
}
```

Ознакомьтесь с дополнительными сведениями о [пакетах](#attributes-and-annotations), [конфигурации](#configuration) и [использовании](#usage).

### <a name="version-1x"></a>Версия 1.x

В следующем примере показана [функция C#](functions-dotnet-class-library.md) службы "Функции" 1.х, которая выполняет привязку к `JObject`.

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Host;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using Microsoft.Extensions.Logging;

namespace Company.Function
{
    public static class EventGridTriggerCSharp
    {
        [FunctionName("EventGridTriggerCSharp")]
        public static void Run([EventGridTrigger]JObject eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.ToString(Formatting.Indented));
        }
    }
}
```

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В следующем примере показаны привязка триггера в файле *function.json* и [функция сценария C#](functions-reference-csharp.md), которая использует эту привязку.

Данные привязки в файле *function.json*:

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

### <a name="version-2x-and-higher"></a>Версия 2.x и более поздние

В следующем примере используется привязка к `EventGridEvent`.

```csharp
#r "Microsoft.Azure.EventGrid"
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(EventGridEvent eventGridEvent, ILogger log)
{
    log.LogInformation(eventGridEvent.Data.ToString());
}
```

Ознакомьтесь с дополнительными сведениями о [пакетах](#attributes-and-annotations), [конфигурации](#configuration) и [использовании](#usage).

### <a name="version-1x"></a>Версия 1.x

Ниже приведен код сценария C# службы "Функции" 1.х, который выполняет привязку к `JObject`:

```cs
#r "Newtonsoft.Json"

using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public static void Run(JObject eventGridEvent, TraceWriter log)
{
    log.Info(eventGridEvent.ToString(Formatting.Indented));
}
```

# <a name="java"></a>[Java](#tab/java)

Этот раздел содержит следующие примеры.

* [Триггер сетки событий, строковый параметр](#event-grid-trigger-string-parameter)
* [Триггер сетки событий, параметр POJO](#event-grid-trigger-pojo-parameter)

В следующих примерах демонстрируется привязка триггера в [Java](functions-reference-java.md) с использованием привязки, которая выводит сведения о событии. Здесь событие сначала принимается как `String`, затем — как POJO.

### <a name="event-grid-trigger-string-parameter"></a>Триггер Сетки событий, параметр String

```java
  @FunctionName("eventGridMonitorString")
  public void logEvent(
    @EventGridTrigger(
      name = "event"
    ) 
    String content, 
    final ExecutionContext context) {
      context.getLogger().info("Event content: " + content);      
  }
```

### <a name="event-grid-trigger-pojo-parameter"></a>Триггер Сетки событий, параметр POJO

В этом примере используется следующий POJO, представляющий свойства верхнего уровня события сетки событий.

```java
import java.util.Date;
import java.util.Map;

public class EventSchema {

  public String topic;
  public String subject;
  public String eventType;
  public Date eventTime;
  public String id;
  public String dataVersion;
  public String metadataVersion;
  public Map<String, Object> data;

}
```

По прибытии полезные данные JSON события десериализуются в ```EventSchema``` POJO для использования функцией. Этот процесс позволяет функции использовать свойства событий в объектно-ориентированном стиле.

```java
  @FunctionName("eventGridMonitor")
  public void logEvent(
    @EventGridTrigger(
      name = "event"
    ) 
    EventSchema event, 
    final ExecutionContext context) {
      context.getLogger().info("Event content: ");
      context.getLogger().info("Subject: " + event.subject);
      context.getLogger().info("Time: " + event.eventTime); // automatically converted to Date by the runtime
      context.getLogger().info("Id: " + event.id);
      context.getLogger().info("Data: " + event.data);
  }
```

В [библиотеке выполнения функций Java](/java/api/overview/azure/functions/runtime) используйте заметку `EventGridTrigger` для параметров, значение которых должно быть получено от EventGrid. Параметры с этими заметками запускают функцию, когда происходит событие.  Эта заметка может использоваться с собственными типами Java, объектами POJO или значениями, допускающими значения NULL, используя `Optional<T>`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В следующем примере показана привязка триггера в файле *function.json* и [функция JavaScript](functions-reference-node.md), которая использует привязку.

Данные привязки в файле *function.json*:

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

Ниже показан код JavaScript.

```javascript
module.exports = function (context, eventGridEvent) {
    context.log("JavaScript Event Grid function processed a request.");
    context.log("Subject: " + eventGridEvent.subject);
    context.log("Time: " + eventGridEvent.eventTime);
    context.log("Data: " + JSON.stringify(eventGridEvent.data));
    context.done();
};
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В следующем примере показано, как настроить привязку триггера для сетки событий в *function.jsв* файле.

```powershell
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ]
}
```

Событие сетки событий становится доступным для функции через параметр с именем `eventGridEvent` , как показано в следующем примере PowerShell.

```powershell
param($eventGridEvent, $TriggerMetadata)

# Make sure to pass hashtables to Out-String so they're logged correctly
$eventGridEvent | Out-String | Write-Host
```

# <a name="python"></a>[Python](#tab/python)

В следующем примере показаны привязка триггера в файле *function.json* и [функция Python](functions-reference-python.md), использующая эту привязку.

Данные привязки в файле *function.json*:

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "event",
      "direction": "in"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

Ниже приведен код Python.

```python
import json
import logging

import azure.functions as func

def main(event: func.EventGridEvent):

    result = json.dumps({
        'id': event.id,
        'data': event.get_json(),
        'topic': event.topic,
        'subject': event.subject,
        'event_type': event.event_type,
    })

    logging.info('Python EventGrid trigger processed an event: %s', result)
```

---

## <a name="attributes-and-annotations"></a>Атрибуты и заметки

# <a name="c"></a>[C#](#tab/csharp)

В [библиотеках классов C#](functions-dotnet-class-library.md) используйте атрибут [EventGridTrigger](https://github.com/Azure/azure-functions-eventgrid-extension/blob/master/src/EventGridExtension/TriggerBinding/EventGridTriggerAttribute.cs).

Вот как выглядит атрибут `EventGridTrigger` в сигнатуре метода:

```csharp
[FunctionName("EventGridTest")]
public static void EventGridTest([EventGridTrigger] JObject eventGridEvent, ILogger log)
{
    ...
}
```

Полный пример см. в разделе "Пример C#".

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В скрипте C# атрибуты не поддерживаются.

# <a name="java"></a>[Java](#tab/java)

Аннотация [EventGridTrigger](https://github.com/Azure/azure-functions-java-library/blob/master/src/main/java/com/microsoft/azure/functions/annotation/EventGridTrigger.java) позволяет декларативно настроить привязку Сетки событий, предоставив значения конфигурации. Дополнительные сведения см. в разделах [примера](#example) и [конфигурации](#configuration).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

В JavaScript атрибуты не поддерживаются.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

В PowerShell не поддерживаются атрибуты.

# <a name="python"></a>[Python](#tab/python)

В Python атрибуты не поддерживаются.

---

## <a name="configuration"></a>Конфигурация

В следующей таблице описываются свойства конфигурации привязки, которые задаются в файле *function.json*. В атрибуте `EventGridTrigger` не задаются параметры или свойства конструкции.

|свойство function.json |Описание|
|---------|---------|
| **type** | Обязательное. Необходимо задать значение `eventGridTrigger`. |
| **direction** | Обязательное. Необходимо задать значение `in`. |
| **name** | Обязательное. Имя переменной, используемой в коде функции, для параметра, получающего данные события. |

## <a name="usage"></a>Использование

# <a name="c"></a>[C#](#tab/csharp)

В Функциях Azure 1.x для триггера Сетки событий можно использовать следующие типы параметров:

* `JObject`
* `string`

В Функциях Azure 2.x и более поздних версий вы также можете использовать для триггера Сетки событий следующие типы параметров:

* `Microsoft.Azure.EventGrid.Models.EventGridEvent` — определяет свойства для полей, которые являются общими для всех типов событий.

> [!NOTE]
> В службе "Функции" 1.х при попытке сделать привязку к `Microsoft.Azure.WebJobs.Extensions.EventGrid.EventGridEvent` в компиляторе появится сообщение о том, что это не рекомендуется, и совет использовать `Microsoft.Azure.EventGrid.Models.EventGridEvent`. Чтобы использовать более новый тип, ссылайтесь на пакет NuGet [Microsoft.Azure.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.EventGrid) и полностью определите имя типа `EventGridEvent`, добавив префикс `Microsoft.Azure.EventGrid.Models`.

# <a name="c-script"></a>[Скрипт C#](#tab/csharp-script)

В Функциях Azure 1.x для триггера Сетки событий можно использовать следующие типы параметров:

* `JObject`
* `string`

В Функциях Azure 2.x и более поздних версий вы также можете использовать для триггера Сетки событий следующие типы параметров:

* `Microsoft.Azure.EventGrid.Models.EventGridEvent` — определяет свойства для полей, которые являются общими для всех типов событий.

> [!NOTE]
> В службе "Функции" 1.х при попытке сделать привязку к `Microsoft.Azure.WebJobs.Extensions.EventGrid.EventGridEvent` в компиляторе появится сообщение о том, что это не рекомендуется, и совет использовать `Microsoft.Azure.EventGrid.Models.EventGridEvent`. Чтобы использовать более новый тип, ссылайтесь на пакет NuGet [Microsoft.Azure.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.EventGrid) и полностью определите имя типа `EventGridEvent`, добавив префикс `Microsoft.Azure.EventGrid.Models`. Сведения о том, как ссылаться на пакеты NuGet в функции сценария C#, см. в разделе [об использовании пакетов NuGet](functions-reference-csharp.md#using-nuget-packages).

# <a name="java"></a>[Java](#tab/java)

Экземпляр Сетки событий доступен через параметр, связанный с атрибутом `EventGridTrigger`, который имеет тип `EventSchema`. Дополнительные сведения см. в [этом примере](#example).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Экземпляр Сетки событий доступен через параметр, который настраивается в свойстве `name` файла *function.json*.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Экземпляр Сетки событий доступен через параметр, который настраивается в свойстве `name` файла *function.json*.

# <a name="python"></a>[Python](#tab/python)

Экземпляр Сетки событий доступен через параметр, который настраивается в свойстве `name` с типом `func.EventGridEvent` в файле *function.json*.

---

## <a name="event-schema"></a>Схема событий

Данные для события службы "Сетка событий" получаются в качестве объекта JSON в теле HTTP-запроса. JSON должен быть аналогичным приведенному ниже:

```json
[{
  "topic": "/subscriptions/{subscriptionid}/resourceGroups/eg0122/providers/Microsoft.Storage/storageAccounts/egblobstore",
  "subject": "/blobServices/default/containers/{containername}/blobs/blobname.jpg",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2018-01-23T17:02:19.6069787Z",
  "id": "{guid}",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "{guid}",
    "requestId": "{guid}",
    "eTag": "0x8D562831044DDD0",
    "contentType": "application/octet-stream",
    "contentLength": 2248,
    "blobType": "BlockBlob",
    "url": "https://egblobstore.blob.core.windows.net/{containername}/blobname.jpg",
    "sequencer": "000000000000272D000000000003D60F",
    "storageDiagnostics": {
      "batchId": "{guid}"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

Показанный пример является массивом из одного элемента. Сетка событий всегда отправляет массив, который может содержать более одного события. Среда выполнения вызывает функцию один раз для каждого элемента массива.

Свойства верхнего уровня в данных JSON события одинаковы для всех типов событий, тогда как содержимое свойства `data` определено для каждого типа события. Пример показан для события хранилища BLOB-объектов.

Описания распространенных и связанных с событием свойств см. в разделе [Свойства события](../event-grid/event-schema.md#event-properties) в документации по службе "Сетка событий".

Тип `EventGridEvent` определяет только свойства верхнего уровня. Свойство `Data` имеет значение `JObject`.

## <a name="create-a-subscription"></a>Создание подписки

Чтобы начать получать HTTP-запросы Сетки событий, создайте подписку службы "Сетка событий", которая указывает URL-адрес конечной точки, вызывающей функцию.

### <a name="azure-portal"></a>Портал Azure

Для функций, разрабатываемых на портале Azure с использованием триггера Сетки событий, выберите **Интеграция**, **Триггер Сетки событий** и **Создание подписки для Сетки событий**.

:::image type="content" source="media/functions-bindings-event-grid/portal-sub-create.png" alt-text="Подключите новую подписку на событие к триггеру с помощью портала.":::

Когда вы щелкните эту ссылку, на портале откроется страница **Создать подписку на события**, где автоматически определена конечная точка текущего триггера.

:::image type="content" source="media/functions-bindings-event-grid/endpoint-url.png" alt-text="Страница Создать подписку на события, где определена конечная точка функции" :::

Дополнительные сведения о создании подписок с помощью портала Azure см. в разделе [Создание и перенаправление пользовательских событий с помощью портала Azure и службы "Сетка событий"](../event-grid/custom-event-quickstart-portal.md) документации по службе "Сетка событий".

### <a name="azure-cli"></a>Azure CLI

Чтобы создать подписку с помощью [Azure CLI](/cli/azure/get-started-with-azure-cli), используйте команду [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-create).

Для команды нужен URL-адрес конечной точки, который вызывает функцию. В следующем примере показан шаблон URL-адреса в зависимости от версии:

#### <a name="version-2x-and-higher-runtime"></a>Среда выполнения версии 2.x и более поздних

```http
https://{functionappname}.azurewebsites.net/runtime/webhooks/eventgrid?functionName={functionname}&code={systemkey}
```

#### <a name="version-1x-runtime"></a>Среда выполнения версии 1.x

```http
https://{functionappname}.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName={functionname}&code={systemkey}
```

Системный ключ — это ключ авторизации, который необходимо включить в URL-адрес конечной точки для триггера службы "Сетка событий". В следующем разделе показано, как получить системный ключ.

Ниже приведен пример, который подписывается на учетную запись хранения больших двоичных объектов (с заполнителем для системного ключа):

#### <a name="version-2x-and-higher-runtime"></a>Среда выполнения версии 2.x и более поздних

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup \
    --provider-namespace Microsoft.Storage --resource-type storageAccounts \
    --resource-name myblobstorage12345 --name myFuncSub \
    --included-event-types Microsoft.Storage.BlobCreated \
    --subject-begins-with /blobServices/default/containers/images/blobs/ \
    --endpoint https://mystoragetriggeredfunction.azurewebsites.net/runtime/webhooks/eventgrid?functionName=imageresizefunc&code=<key>
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup ^
    --provider-namespace Microsoft.Storage --resource-type storageAccounts ^
    --resource-name myblobstorage12345 --name myFuncSub ^
    --included-event-types Microsoft.Storage.BlobCreated ^
    --subject-begins-with /blobServices/default/containers/images/blobs/ ^
    --endpoint https://mystoragetriggeredfunction.azurewebsites.net/runtime/webhooks/eventgrid?functionName=imageresizefunc&code=<key>
```

---

#### <a name="version-1x-runtime"></a>Среда выполнения версии 1.x

# <a name="bash"></a>[Bash](#tab/bash)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup \
    --provider-namespace Microsoft.Storage --resource-type storageAccounts \
    --resource-name myblobstorage12345 --name myFuncSub \
    --included-event-types Microsoft.Storage.BlobCreated \
    --subject-begins-with /blobServices/default/containers/images/blobs/ \
    --endpoint https://mystoragetriggeredfunction.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName=imageresizefunc&code=<key>
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup ^
    --provider-namespace Microsoft.Storage --resource-type storageAccounts ^
    --resource-name myblobstorage12345 --name myFuncSub ^
    --included-event-types Microsoft.Storage.BlobCreated ^
    --subject-begins-with /blobServices/default/containers/images/blobs/ ^
    --endpoint https://mystoragetriggeredfunction.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName=imageresizefunc&code=<key>
```

---

Дополнительные сведения о создании подписки см. в разделе [Примеры использования хранилища больших двоичных объектов](../storage/blobs/storage-blob-event-quickstart.md#subscribe-to-your-storage-account) или других коротких руководствах по использованию службы "Сетка событий".

### <a name="get-the-system-key"></a>Получение системного ключа

Системный ключ можно получить с помощью следующего API-интерфейса (HTTP GET):

#### <a name="version-2x-and-higher-runtime"></a>Среда выполнения версии 2.x и более поздних

```
http://{functionappname}.azurewebsites.net/admin/host/systemkeys/eventgrid_extension?code={masterkey}
```

#### <a name="version-1x-runtime"></a>Среда выполнения версии 1.x

```
http://{functionappname}.azurewebsites.net/admin/host/systemkeys/eventgridextensionconfig_extension?code={masterkey}
```

Это API администратора, поэтому для него нужен [главный ключ](functions-bindings-http-webhook-trigger.md#authorization-keys) приложения-функции. Не путайте системный ключ (для вызова функции триггера службы "Сетка событий") с главным ключом (для выполнения административных задач в приложении-функции). Во время оформления подписки на раздел службы "Сетка событий" необходимо использовать системный ключ.

Ниже приведен пример ответа, который предоставляет системный ключ:

```
{
  "name": "eventgridextensionconfig_extension",
  "value": "{the system key for the function}",
  "links": [
    {
      "rel": "self",
      "href": "{the URL for the function, without the system key}"
    }
  ]
}
```

Можно получить главный ключ для приложения-функции на вкладке **Параметры приложения-функции** на портале.

> [!IMPORTANT]
> Главный ключ предоставляет административный доступ к приложению-функции. Не передавайте этот ключ третьим лицам и не распространяйте его в собственных клиентских приложениях.

Дополнительные сведения см. в разделе [Ключи авторизации](functions-bindings-http-webhook-trigger.md#authorization-keys) в справочнике по триггерам HTTP.

Кроме того, вы можете отправить HTTP-запрос PUT, чтобы самостоятельно указать значение ключа.

## <a name="local-testing-with-viewer-web-app"></a>Локальное тестирование с помощью веб-приложения средства просмотра

Чтобы протестировать триггер службы "Сетка событий" локально, необходимо отправить HTTP-запросы службы "Сетка событий", переданные из своего источника в облаке, на локальный компьютер. Для этого можно записать запросы в сети и вручную отправить их на локальный компьютер:

1. [Создайте веб-приложение средства просмотра](#create-a-viewer-web-app), которое собирает сообщения о событиях.
1. [Создайте подписку службы "Сетка событий"](#create-an-event-grid-subscription), которая отправляет события в приложение средства просмотра.
1. [Создайте запрос](#generate-a-request) и скопируйте текст запроса из приложения средства просмотра.
1. [Вручную вставьте запрос](#manually-post-the-request) в URL-адрес localhost функции триггера службы "Сетка событий".

Закончив тестирование, можно использовать ту же подписку для рабочей среды, обновив конечную точку. Используйте команду Azure CLI [az eventgrid event-subscription update](/cli/azure/eventgrid/event-subscription#az-eventgrid-event-subscription-update).

### <a name="create-a-viewer-web-app"></a>Создание веб-приложения средства просмотра

Чтобы упростить запись сообщений о событиях, разверните [готовое веб-приложение](https://github.com/Azure-Samples/azure-event-grid-viewer), которое отображает сообщения о событиях. Развернутое решение содержит план службы приложений, веб-приложение службы приложений и исходный код из GitHub.

Выберите **Развернуть в Azure**, чтобы развернуть решение в своей подписке. На портале Azure укажите значения остальных параметров.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png" alt="Button to Deploy to Azure." /></a>

Завершение развертывания может занять несколько минут. Когда развертывание успешно завершится, откройте веб-приложение и убедитесь, что оно работает. Откройте браузер и перейдите по адресу `https://<your-site-name>.azurewebsites.net`.

Вы увидите сайт без опубликованных событий.

![Представление нового сайта](media/functions-bindings-event-grid/view-site.png)

### <a name="create-an-event-grid-subscription"></a>Создание подписки Сетки событий

Создайте подписку "Сетка событий" того типа, который требуется протестировать, и предоставьте URL-адрес из веб-приложения как конечную точку для уведомления о событиях. Конечная точка веб-приложения должна содержать суффикс `/api/updates/`. Таким образом, полный URL-адрес будет выглядеть следующим образом: `https://<your-site-name>.azurewebsites.net/api/updates`

Дополнительные сведения о создании подписок с помощью портала Azure см. в разделе [Создание пользовательского события с помощью портала Azure](../event-grid/custom-event-quickstart-portal.md) в документации по службе "Сетка событий".

### <a name="generate-a-request"></a>Создание запроса

Активируйте событие, которое будет создавать HTTP-трафик в конечную точку веб-приложения.  Например, если вы создали подписку хранилища BLOB-объектов, отправьте или удалите большой двоичный объект. Скопируйте текст запроса, когда запрос отобразится в веб-приложении.

Сначала будет получен запрос на подтверждение подписки. Игнорируйте такие запросы и скопируйте запрос события.

![Копирование текста запроса из веб-приложения](media/functions-bindings-event-grid/view-results.png)

### <a name="manually-post-the-request"></a>Публикация запроса вручную

Запустите функцию службы "Сетка событий" локально. `Content-Type`Заголовки и должны `aeg-event-type` быть заданы вручную, тогда как все остальные значения можно оставить по умолчанию.

Используйте такие средства, как [Postman](https://www.getpostman.com/) или [cURL](https://curl.haxx.se/docs/httpscripting.html), чтобы создать HTTP-запрос POST:

* Задайте заголовок `Content-Type: application/json`.
* Задайте заголовок `aeg-event-type: Notification`.
* Вставьте данные RequestBin в текст запроса.
* Опубликуйте по URL-адресу функции триггера Сетки событий.
  * Для версии 2.x и более поздних используйте следующий шаблон:

    ```
    http://localhost:7071/runtime/webhooks/eventgrid?functionName={FUNCTION_NAME}
    ```

  * Для версии 1.x используйте этот:

    ```
    http://localhost:7071/admin/extensions/EventGridExtensionConfig?functionName={FUNCTION_NAME}
    ```

Для параметра `functionName` нужно указать имя, заданное в атрибуте `FunctionName`.

На следующих снимках экрана показаны заголовки и текст запроса в Postman:

![Заголовки в Postman](media/functions-bindings-event-grid/postman2.png)

![Текст запроса в Postman](media/functions-bindings-event-grid/postman.png)

Функция триггера службы "Сетка событий" выполняет и показывает журналы, похожие на приведенные ниже:

![Пример журналов функций триггера службы "Сетка событий"](media/functions-bindings-event-grid/eg-output.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Отправка события Сетки событий](./functions-bindings-event-grid-output.md)
