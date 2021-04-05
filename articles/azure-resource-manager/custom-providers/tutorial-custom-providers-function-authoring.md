---
title: Создание конечной точки RESTful
description: В этом руководстве демонстрируется, как создать конечную точку RESTful для настраиваемых поставщиков. Здесь подробно описано, как обрабатывать запросы и ответы для поддерживаемых методов HTTP REST.
author: jjbfour
ms.topic: tutorial
ms.date: 01/13/2021
ms.author: jobreen
ms.openlocfilehash: 54d0df287865d5d92403bf68227a2d4c5faa8bb4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98200215"
---
# <a name="author-a-restful-endpoint-for-custom-providers"></a>Создание конечной точки RESTful для настраиваемых поставщиков

Настраиваемым поставщиком называется контракт между Azure и конечной точкой. Настраиваемые поставщики позволяют настраивать рабочие процессы в Azure. В руководстве показано, как создать конечную точку RESTful для настраиваемого поставщика. Если вы еще не знакомы с настраиваемыми поставщиками Azure, перейдите к этой [обзорной статье](overview.md).

> [!NOTE]
> Это руководство является продолжением статьи [Настройка Функций Azure для настраиваемых поставщиков Azure](./tutorial-custom-providers-function-setup.md). Для выполнения некоторых инструкций необходимо настроить поддержку настраиваемых поставщиков в приложении-функции Azure.

## <a name="work-with-custom-actions-and-custom-resources"></a>Работа с настраиваемыми действиями и настраиваемыми ресурсами

При работе с этим руководством вы обновите приложение-функцию, чтобы оно работало в качестве конечной точки RESTful для настраиваемого поставщика. Все ресурсы и действия Azure соблюдают базовую спецификацию REST:

- **PUT**: создание нового ресурса.
- **GET (экземпляр)** : получение существующего ресурса.
- **DELETE**: удаление существующего ресурса.
- **POST**: активация действия.
- **GET (коллекция)** : перечисление всех существующих ресурсов.

 В этом руководстве используется табличное хранилище Azure. Но подойдет и любая другая база данных или служба хранилища.

## <a name="partition-custom-resources-in-storage"></a>Секционирование настраиваемых ресурсов в хранилище

Так как мы создаем службу RESTful, нам нужно хранить созданные ресурсы. В табличном хранилище Azure нам нужно создать ключи секций и строк для наших данных. Для пользовательских поставщиков данные должны быть секционированы. Когда входящий запрос отправляется настраиваемому поставщику, этот поставщик добавляет заголовок `x-ms-customproviders-requestpath` к исходящему запросу к конечной точке.

В следующем примере показан заголовок `x-ms-customproviders-requestpath` для настраиваемого ресурса:

```
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/{myResourceType}/{myResourceName}
```

Основываясь на приведенном выше примере заголовка `x-ms-customproviders-requestpath`, вы можете создать параметры *partitionKey* и *rowKey* для используемого хранилища, как показано в следующей таблице.

Параметр | Шаблон | Описание
---|---|---
*partitionKey* | `{subscriptionId}:{resourceGroupName}:{resourceProviderName}` | Параметр *partitionKey* определяет способ секционирования данных. Обычно данные секционируются по экземплярам настраиваемого поставщика.
*rowKey* | `{myResourceType}:{myResourceName}` | Параметр *rowKey* содержит индивидуальный идентификатор данных. Обычно идентификатором служит имя ресурса.

Вам также нужно создать новый класс для моделирования настраиваемого ресурса. Далее вы добавите в приложение-функцию класс **CustomResource**.

```csharp
// Custom Resource Table Entity
public class CustomResource : TableEntity
{
    public string Data { get; set; }
}
```
**CustomResource** — это простой и универсальный класс, который принимает любые входные данные. Он основан на классе **TableEntity**, который используется для хранения данных. Класс **CustomResource** наследует от **TableEntity** два свойства: **partitionKey** и **rowKey**.

## <a name="support-custom-provider-restful-methods"></a>Поддержка методов пользовательских поставщиков RESTful

> [!NOTE]
> Если вы не копируете код непосредственно из руководства, предоставьте в ответе допустимый объект JSON, который устанавливает для заголовка `Content-Type` значение `application/json`.

Итак, вы завершили настройку секционирования данных, и теперь переходите к созданию методов CRUD для настраиваемых ресурсов и триггеров для настраиваемых действий. Так как настраиваемые поставщики выполняют функцию посредников, распознавание и обработка запроса и ответа возлагаются на конечную точку RESTful. В приведенном ниже фрагменте кода показано, как выполнять базовые операции RESTful.

### <a name="trigger-a-custom-action"></a>Активация настраиваемого действия

В настраиваемых поставщиках настраиваемые действия активируются запросами POST. Пользовательское действие может дополнительно принимать текст запроса, который содержит набор входных параметров. Затем это действие возвращает ответ с информацией об успешности и результате действия.

Добавьте в приложение-функцию следующий метод **TriggerCustomAction**:

```csharp
/// <summary>
/// Triggers a custom action with some side effects.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <returns>The HTTP response result of the custom action.</returns>
public static async Task<HttpResponseMessage> TriggerCustomAction(HttpRequestMessage requestMessage)
{
    var myCustomActionRequest = await requestMessage.Content.ReadAsStringAsync();

    var actionResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    actionResponse.Content = myCustomActionRequest != string.Empty ? 
        new StringContent(JObject.Parse(myCustomActionRequest).ToString(), System.Text.Encoding.UTF8, "application/json") :
        null;
    return actionResponse;
}
```

Метод **TriggerCustomAction** принимает входящий запрос и просто возвращает в ответе текст запроса с кодом состояния.

### <a name="create-a-custom-resource"></a>Создание настраиваемого ресурса

В настраиваемых поставщиках настаиваемые ресурсы создаются с помощью запросов PUT. Настраиваемый поставщик принимает текст запроса в формате JSON с набором свойств для настраиваемого ресурса. Все ресурсы в Azure соответствуют модели RESTful. Для создания, извлечения или удаления ресурсов можно использовать один и тот же URL-адрес запроса.

Добавьте метод **CreateCustomResource** для создания ресурсов.

```csharp
/// <summary>
/// Creates a custom resource and saves it to table storage.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="azureResourceId">The parsed Azure resource ID.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the created custom resource.</returns>
public static async Task<HttpResponseMessage> CreateCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, ResourceId azureResourceId, string partitionKey, string rowKey)
{
    // Adds the Azure top-level properties.
    var myCustomResource = JObject.Parse(await requestMessage.Content.ReadAsStringAsync());
    myCustomResource["name"] = azureResourceId.Name;
    myCustomResource["type"] = azureResourceId.FullResourceType;
    myCustomResource["id"] = azureResourceId.Id;

    // Save the resource into storage.
    var insertOperation = TableOperation.InsertOrReplace(
        new CustomResource
        {
            PartitionKey = partitionKey,
            RowKey = rowKey,
            Data = myCustomResource.ToString(),
        });
    await tableStorage.ExecuteAsync(insertOperation);

    var createResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    createResponse.Content = new StringContent(myCustomResource.ToString(), System.Text.Encoding.UTF8, "application/json");
    return createResponse;
}
```

Метод **CreateCustomResource** изменяет входящий запрос, добавляя поля **id** (идентификатор), **name** (имя) и **type** (тип), которые понимает платформа Azure. Эти поля содержат свойства верхнего уровня, которые используются службами в Azure. Они позволят интегрировать настраиваемый поставщик с другими службами, такими как Политика Azure, шаблоны Azure Resource Manager и журналы действий Azure.

Свойство | Пример | Описание
---|---|---
**name** | {имя_настраиваемого_ресурса} | Имя настраиваемого ресурса
**type** | Microsoft.CustomProviders/resourceProviders/{имя_типа_ресурса} | Пространство имен для типа ресурса
**id** | /subscriptions/{subscriptionId}/resourceGroups/{имя_группы_ресурсов}/<br>providers/Microsoft.CustomProviders/resourceProviders/{имя поставщика ресурсов}/<br>{имя_типа_ресурса}/{имя_настраиваемого_ресурса} | Идентификатор ресурса

Вы не только добавили свойства, но и сохранили документ JSON в табличное хранилище Azure.

### <a name="retrieve-a-custom-resource"></a>Извлечение настраиваемого ресурса

В настраиваемых поставщиках настаиваемый ресурс извлекается с помощью запросов GET. Настраиваемый поставщик *не принимает* текст запроса в формате JSON. По запросу GET конечная точка возвращает уже созданный ресурс, указанный в заголовке `x-ms-customproviders-requestpath`.

Добавьте метод **RetrieveCustomResource**, который будет извлекать существующие ресурсы:

```csharp
/// <summary>
/// Retrieves a custom resource.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the existing custom resource.</returns>
public static async Task<HttpResponseMessage> RetrieveCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string rowKey)
{
    // Attempt to retrieve the Existing Stored Value
    var tableQuery = TableOperation.Retrieve<CustomResource>(partitionKey, rowKey);
    var existingCustomResource = (CustomResource)(await tableStorage.ExecuteAsync(tableQuery)).Result;

    var retrieveResponse = requestMessage.CreateResponse(
        existingCustomResource != null ? HttpStatusCode.OK : HttpStatusCode.NotFound);

    retrieveResponse.Content = existingCustomResource != null ?
            new StringContent(existingCustomResource.Data, System.Text.Encoding.UTF8, "application/json"):
            null;
    return retrieveResponse;
}
```

В Azure ресурсы соответствуют модели RESTful. URL-адрес запроса, который создает ресурс, также возвращает этот ресурс, если выполняется запрос GET.

### <a name="remove-a-custom-resource"></a>Удаление настраиваемого ресурса

В настраиваемых поставщиках настаиваемый ресурс удаляется с помощью запросов DELETE. Настраиваемый поставщик *не принимает* текст запроса в формате JSON. По запросу DELETE конечная точка удаляет уже созданный ресурс, указанный в заголовке `x-ms-customproviders-requestpath`.

Добавьте метод **RemoveCustomResource**, который будет удалять существующие ресурсы:

```csharp
/// <summary>
/// Removes an existing custom resource.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure storage account table.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="rowKey">The row key for storage. This is '{resourceType}:{customResourceName}'.</param>
/// <returns>The HTTP response containing the result of the deletion.</returns>
public static async Task<HttpResponseMessage> RemoveCustomResource(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string rowKey)
{
    // Attempt to retrieve the Existing Stored Value
    var tableQuery = TableOperation.Retrieve<CustomResource>(partitionKey, rowKey);
    var existingCustomResource = (CustomResource)(await tableStorage.ExecuteAsync(tableQuery)).Result;

    if (existingCustomResource != null) {
        var deleteOperation = TableOperation.Delete(existingCustomResource);
        await tableStorage.ExecuteAsync(deleteOperation);
    }

    return requestMessage.CreateResponse(
        existingCustomResource != null ? HttpStatusCode.OK : HttpStatusCode.NoContent);
}
```

В Azure ресурсы соответствуют модели RESTful. URL-адрес запроса, который создает ресурс, также удаляет этот ресурс, если выполняется запрос DELETE.

### <a name="list-all-custom-resources"></a>Список всех настраиваемых ресурсов

В настраиваемых поставщиках можно получить список существующих настраиваемых ресурсов с помощью коллекции запросов GET. Настраиваемый поставщик *не принимает* текст запроса в формате JSON. По коллекции запросов GET конечная точка перечисляет уже созданные ресурсы, указанные в заголовке `x-ms-customproviders-requestpath`.

Добавьте следующий метод **EnumerateAllCustomResources**, который будет перечислять существующие ресурсы:

```csharp
/// <summary>
/// Enumerates all the stored custom resources for a given type.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <param name="partitionKey">The partition key for storage. This is the custom provider ID.</param>
/// <param name="resourceType">The resource type of the enumeration.</param>
/// <returns>The HTTP response containing a list of resources stored under 'value'.</returns>
public static async Task<HttpResponseMessage> EnumerateAllCustomResources(HttpRequestMessage requestMessage, CloudTable tableStorage, string partitionKey, string resourceType)
{
    // Generate upper bound of the query.
    var rowKeyUpperBound = new StringBuilder(resourceType);
    rowKeyUpperBound[rowKeyUpperBound.Length - 1]++;

    // Create the enumeration query.
    var enumerationQuery = new TableQuery<CustomResource>().Where(
        TableQuery.CombineFilters(
            TableQuery.GenerateFilterCondition("PartitionKey", QueryComparisons.Equal, partitionKey),
            TableOperators.And,
            TableQuery.CombineFilters(
                TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.GreaterThan, resourceType),
                TableOperators.And,
                TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, rowKeyUpperBound.ToString()))));
    
    var customResources = (await tableStorage.ExecuteQuerySegmentedAsync(enumerationQuery, null))
        .ToList().Select(customResource => JToken.Parse(customResource.Data));

    var enumerationResponse = requestMessage.CreateResponse(HttpStatusCode.OK);
    enumerationResponse.Content = new StringContent(new JObject(new JProperty("value", customResources)).ToString(), System.Text.Encoding.UTF8, "application/json");
    return enumerationResponse;
}
```

> [!NOTE]
> Синтаксис RowKey QueryComparisons.GreaterThan и QueryComparisons.LessThan используется в табличном хранилище Azure для выполнения запроса строк startswith.

Чтобы перечислить все существующие ресурсы, создайте запрос к табличному хранилищу Azure, который проверяет существование ресурсов в секции настраиваемого поставщика. Затем этот запрос проверяет, начинается ли ключ строки с того же значения `{myResourceType}`.

## <a name="integrate-restful-operations"></a>Интеграция операций RESTful

Добавив все методы REST в приложение-функцию, обновите основной метод **Run**, чтобы вызвать функции для обработки разных запросов REST.

```csharp
/// <summary>
/// Entry point for the Azure function app webhook that acts as the service behind a custom provider.
/// </summary>
/// <param name="requestMessage">The HTTP request message.</param>
/// <param name="log">The logger.</param>
/// <param name="tableStorage">The Azure Table storage account.</param>
/// <returns>The HTTP response for the custom Azure API.</returns>
public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, ILogger log, CloudTable tableStorage)
{
    // Get the unique Azure request path from request headers.
    var requestPath = req.Headers.GetValues("x-ms-customproviders-requestpath").FirstOrDefault();

    if (requestPath == null)
    {
        var missingHeaderResponse = req.CreateResponse(HttpStatusCode.BadRequest);
        missingHeaderResponse.Content = new StringContent(
            new JObject(new JProperty("error", "missing 'x-ms-customproviders-requestpath' header")).ToString(),
            System.Text.Encoding.UTF8, 
            "application/json");
    }

    log.LogInformation($"The Custom Provider Function received a request '{req.Method}' for resource '{requestPath}'.");

    // Determines if it is a collection level call or action.
    var isResourceRequest = requestPath.Split('/').Length % 2 == 1;
    var azureResourceId = isResourceRequest ? 
        ResourceId.FromString(requestPath) :
        ResourceId.FromString($"{requestPath}/");

    // Create the Partition Key and Row Key
    var partitionKey = $"{azureResourceId.SubscriptionId}:{azureResourceId.ResourceGroupName}:{azureResourceId.Parent.Name}";
    var rowKey = $"{azureResourceId.FullResourceType.Replace('/', ':')}:{azureResourceId.Name}";

    switch (req.Method)
    {
        // Action request for a custom action.
        case HttpMethod m when m == HttpMethod.Post && !isResourceRequest:
            return await TriggerCustomAction(
                requestMessage: req);

        // Enumerate request for all custom resources.
        case HttpMethod m when m == HttpMethod.Get && !isResourceRequest:
            return await EnumerateAllCustomResources(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                resourceType: rowKey);

        // Retrieve request for a custom resource.
        case HttpMethod m when m == HttpMethod.Get && isResourceRequest:
            return await RetrieveCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Create request for a custom resource.
        case HttpMethod m when m == HttpMethod.Put && isResourceRequest:
            return await CreateCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                azureResourceId: azureResourceId,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Remove request for a custom resource.
        case HttpMethod m when m == HttpMethod.Delete && isResourceRequest:
            return await RemoveCustomResource(
                requestMessage: req,
                tableStorage: tableStorage,
                partitionKey: partitionKey,
                rowKey: rowKey);

        // Invalid request received.
        default:
            return req.CreateResponse(HttpStatusCode.BadRequest);
    }
}
```

Обновленный метод **Run** теперь содержит входную привязку *tableStorage*, которую вы добавили для табличного хранилища Azure. Первая часть метода считывает заголовок `x-ms-customproviders-requestpath` и применяет библиотеку `Microsoft.Azure.Management.ResourceManager.Fluent` для анализа значения в качестве идентификатора ресурса. Настраиваемый поставщик отправляет заголовок `x-ms-customproviders-requestpath`, который определяет путь входящего запроса.

Используя полученный идентификатор ресурса, вы можете создать значения **partitionKey** и **rowKey** для данных, чтобы искать или сохранять настраиваемые ресурсы.

После добавления методов и классов обновите методы **using** в приложении-функции. Добавьте следующий код в начало файла на C#:

```csharp
#r "Newtonsoft.Json"
#r "Microsoft.WindowsAzure.Storage"
#r "../bin/Microsoft.Azure.Management.ResourceManager.Fluent"

using System;
using System.Net;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Configuration;
using System.Text;
using System.Threading;
using System.Globalization;
using System.Collections.Generic;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.WindowsAzure.Storage.Table;
using Microsoft.Azure.Management.ResourceManager.Fluent.Core;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
```

Если при работе с руководством у вас возникнут трудности, изучите полный готовый пример кода в [справочнике по конечной точке C# RESTful настраиваемого поставщика](./reference-custom-providers-csharp-endpoint.md). Завершив работу с приложением-функцией, сохраните URL-адрес приложения-функции. Он потребуется для активации приложения-функции в следующих руководствах.

## <a name="next-steps"></a>Дальнейшие действия

Выполнив инструкции из этой статьи, вы создали конечную точку RESTful для работы с конечной точкой настраиваемого поставщика Azure. Сведения о том, как создать настраиваемый поставщик, см. в статье [Создание и использование настраиваемого поставщика](./tutorial-custom-providers-create.md).
