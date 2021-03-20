---
title: Обновление до пакета SDK .NET для поиска Azure версии 1,1
titleSuffix: Azure Cognitive Search
description: Перенос кода в пакет SDK .NET для службы "Поиск Azure" версии 1.1 с предыдущих версий API. Сведения о новых возможностях и необходимых изменениях кода.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 4860918fc4f995ad267fc35b142d3fcb460ce177
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89002833"
---
# <a name="upgrade-to-azure-search-net-sdk-version-11"></a>Обновление до пакета SDK .NET для поиска Azure версии 1,1

Если вы используете версию 1.0.2-preview [пакета SDK .NET для службы поиска Azure](/dotnet/api/overview/azure/search) или более раннюю версию, то эта статья поможет вам обновить приложение для использования версии 1.1.

Более общее пошаговое руководство по пакету SDK, включая примеры, см. в разделе [Использование службы поиска Azure в приложении .NET](search-howto-dotnet-sdk.md).

> [!NOTE]
> После обновления до версии 1.1, а также если вы уже используете версии с 1.1 по 2.0-preview, следует обновить пакет SDK до версии 3. Инструкции приведены в разделе [Обновление пакета SDK службы поиска Azure для .NET до версии 3](search-dotnet-sdk-migration.md).
>

Прежде всего обновите справочник NuGet для `Microsoft.Azure.Search` , воспользовавшись консолью диспетчера пакетов NuGet или щелкнув правой кнопкой мыши ссылки проекта и выбрав "Управление пакетами NuGet" в Visual Studio.

После того как NuGet загрузит новые пакеты и их зависимости, перестройте проект.

Если ранее вы пользовались предварительной версией 1.0.0, 1.0.1 или 1.0.2, построение будет выполнено успешно.

Если ранее вы пользовались предварительной версией 0.13.0 или более ранней, вы должны увидеть примерно такие ошибки построения:

```output
Program.cs(137,56,137,62): error CS0117: 'Microsoft.Azure.Search.Models.IndexBatch' does not contain a definition for 'Create'
Program.cs(137,99,137,105): error CS0117: 'Microsoft.Azure.Search.Models.IndexAction' does not contain a definition for 'Create'
Program.cs(146,41,146,54): error CS1061: 'Microsoft.Azure.Search.IndexBatchException' does not contain a definition for 'IndexResponse' and no extension method 'IndexResponse' accepting a first argument of type 'Microsoft.Azure.Search.IndexBatchException' could be found (are you missing a using directive or an assembly reference?)
Program.cs(163,13,163,42): error CS0246: The type or namespace name 'DocumentSearchResponse' could not be found (are you missing a using directive or an assembly reference?)
```

Далее необходимо устранить ошибки построения одну за другой. Большинство требует изменения некоторых имен классов и методов, которые были переименованы в пакете SDK. [Список критических изменений в версии 1.1](#ListOfChangesV1) содержит список этих изменений имен.

Если вы используете настраиваемые классы для моделирования документов и эти классы имеют свойства примитивных типов, не допускающие нулевого значения (например `int` или `bool` в C#), в версии пакета SDK 1.1 существует исправление ошибки, которое следует иметь в виду. Дополнительные сведения см. в разделе [Исправления в версии 1.1](#BugFixesV1).

Наконец, после устранения ошибки сборки при необходимости можно внести изменения в приложение, чтобы воспользоваться преимуществами новых функциональных возможностей.

<a name="ListOfChangesV1"></a>

## <a name="list-of-breaking-changes-in-version-11"></a>Список критических изменений в версии 1.1
Следующий список упорядочен по вероятности того, повлияет ли изменение на код приложения.

### <a name="indexbatch-and-indexaction-changes"></a>Изменения IndexBatch и IndexAction
`IndexBatch.Create` был переименован в `IndexBatch.New` и больше не имеет аргумента `params`. Можно использовать `IndexBatch.New` для пакетов, сочетающих в себе различные типы действий (слияние, удаление и т. д.). Кроме того, существуют новые статические методы для создания пакетов, в которых все действия одинаковы: `Delete`, `Merge`, `MergeOrUpload` и `Upload`.

`IndexAction` больше не имеет открытых конструкторов, и его свойства теперь являются неизменяемыми. Следует использовать новые статические методы для создания действий для различных целей: `Delete`, `Merge`, `MergeOrUpload` и `Upload`. `IndexAction.Create` был удален. Если используется перегрузка, которая принимает только документ, обязательно используйте вместо нее `Upload` .

#### <a name="example"></a>Пример
Если код выглядит следующим образом:

```csharp
var batch = IndexBatch.Create(documents.Select(doc => IndexAction.Create(doc)));
indexClient.Documents.Index(batch);
```

Можно изменить его на следующий для устранения ошибок сборки:

```csharp
var batch = IndexBatch.New(documents.Select(doc => IndexAction.Upload(doc)));
indexClient.Documents.Index(batch);
```

Если требуется, можно упростить его так:

```csharp
var batch = IndexBatch.Upload(documents);
indexClient.Documents.Index(batch);
```

### <a name="indexbatchexception-changes"></a>Изменения IndexBatchException
Свойство `IndexBatchException.IndexResponse` было переименовано в `IndexingResults`, и теперь его тип `IList<IndexingResult>`.

#### <a name="example"></a>Пример
Если код выглядит следующим образом:

```csharp
catch (IndexBatchException e)
{
    Console.WriteLine(
        "Failed to index some of the documents: {0}",
        String.Join(", ", e.IndexResponse.Results.Where(r => !r.Succeeded).Select(r => r.Key)));
}
```

Можно изменить его на следующий для устранения ошибок сборки:

```csharp
catch (IndexBatchException e)
{
    Console.WriteLine(
        "Failed to index some of the documents: {0}",
        String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
}
```

<a name="OperationMethodChanges"></a>

### <a name="operation-method-changes"></a>Изменение метода операции
Каждая операция в пакете SDK для .NET для поиска Azure представляется как набор перегрузок метода для синхронных и асинхронных вызовов. В версии 1.1 были изменены подписи и разложение этих перегрузок метода.

Например, операция "Получить статистику индекса" в более старых версиях пакета SDK предоставляла эти подписи:

В `IIndexOperations`:

```csharp
// Asynchronous operation with all parameters
Task<IndexGetStatisticsResponse> GetStatisticsAsync(
    string indexName,
    CancellationToken cancellationToken);
```

В `IndexOperationsExtensions`:

```csharp
// Asynchronous operation with only required parameters
public static Task<IndexGetStatisticsResponse> GetStatisticsAsync(
    this IIndexOperations operations,
    string indexName);

// Synchronous operation with only required parameters
public static IndexGetStatisticsResponse GetStatistics(
    this IIndexOperations operations,
    string indexName);
```

Подписи метода для той же операции в версии 1.1 выглядят следующим образом.

В `IIndexesOperations`:

```csharp
// Asynchronous operation with lower-level HTTP features exposed
Task<AzureOperationResponse<IndexGetStatisticsResult>> GetStatisticsWithHttpMessagesAsync(
    string indexName,
    SearchRequestOptions searchRequestOptions = default(SearchRequestOptions),
    Dictionary<string, List<string>> customHeaders = null,
    CancellationToken cancellationToken = default(CancellationToken));
```

В `IndexesOperationsExtensions`:

```csharp
// Simplified asynchronous operation
public static Task<IndexGetStatisticsResult> GetStatisticsAsync(
    this IIndexesOperations operations,
    string indexName,
    SearchRequestOptions searchRequestOptions = default(SearchRequestOptions),
    CancellationToken cancellationToken = default(CancellationToken));

// Simplified synchronous operation
public static IndexGetStatisticsResult GetStatistics(
    this IIndexesOperations operations,
    string indexName,
    SearchRequestOptions searchRequestOptions = default(SearchRequestOptions));
```

Начиная с версии 1.1, пакет SDK для .NET для поиска Azure распределяет методы операции иначе:

* Необязательные параметры теперь моделируются как параметры по умолчанию, а не как дополнительные перегрузки метода. Это уменьшает количество перегрузок метода, порой радикально.
* Методы расширения теперь скрывают много лишних сведений HTTP от вызывающего объекта. Например, более старые версии пакета SDK возвращали объект ответа с кодом состояния HTTP, который часто не нужно проверять, так как методы операции создают исключение `CloudException` для любого кода состояния, который указывает на ошибку. Новые методы расширения просто возвращают объекты модели, что устраняет необходимость развертывать их в коде.
* И наоборот, основные интерфейсы теперь демонстрируют методы, которые предоставляют более полный контроль на уровне HTTP при необходимости. Теперь можно передавать пользовательские заголовки HTTP для добавления в запросы, и новый возвращаемый тип. Теперь можно передавать пользовательские заголовки HTTP для добавления в запросы, и новый возвращаемый тип `AzureOperationResponse<T>` предоставляет прямой доступ к `HttpRequestMessage` и `HttpResponseMessage` для операции. `AzureOperationResponse` определяется в пространстве имен `Microsoft.Rest.Azure` и заменяет `Hyak.Common.OperationResponse`.

### <a name="scoringparameters-changes"></a>Изменения в параметрах оценки
В последнюю версию пакета SDK был добавлен класс `ScoringParameter` для упрощения передачи параметров профилям оценки в поисковом запросе. Ранее свойство `ScoringProfiles` класса `SearchParameters` имело тип `IList<string>`. Теперь оно имеет тип `IList<ScoringParameter>`.

#### <a name="example"></a>Пример
Если код выглядит следующим образом:

```csharp
var sp = new SearchParameters();
sp.ScoringProfile = "jobsScoringFeatured";      // Use a scoring profile
sp.ScoringParameters = new[] { "featuredParam-featured", "mapCenterParam-" + lon + "," + lat };
```

Можно изменить его на следующий для устранения ошибок сборки: 

```csharp
var sp = new SearchParameters();
sp.ScoringProfile = "jobsScoringFeatured";      // Use a scoring profile
sp.ScoringParameters =
    new[]
    {
        new ScoringParameter("featuredParam", new[] { "featured" }),
        new ScoringParameter("mapCenterParam", GeographyPoint.Create(lat, lon))
    };
```

### <a name="model-class-changes"></a>Изменение классов модели
Из-за изменений подписи, описанных в разделе [Изменение метода операции](#OperationMethodChanges), многие классы в пространстве имен `Microsoft.Azure.Search.Models` были переименованы или удалены. Пример:

* `IndexDefinitionResponse` был заменен на `AzureOperationResponse<Index>`.
* `DocumentSearchResponse` был переименован в `DocumentSearchResult`.
* `IndexResult` был переименован в `IndexingResult`.
* `Documents.Count()` теперь возвращает `long` с количеством документов вместо `DocumentCountResponse`.
* `IndexGetStatisticsResponse` был переименован в `IndexGetStatisticsResult`.
* `IndexListResponse` был переименован в `IndexListResult`.

Подводя итог, классы, производные от `OperationResponse`, которые существовали только для оборачивания объектов модели, были удалены. В остальных классах суффикс изменен с `Response` на `Result`.

#### <a name="example"></a>Пример
Если код выглядит следующим образом:

```csharp
IndexerGetStatusResponse statusResponse = null;

try
{
    statusResponse = _searchClient.Indexers.GetStatus(indexer.Name);
}
catch (Exception ex)
{
    Console.WriteLine("Error polling for indexer status: {0}", ex.Message);
    return;
}

IndexerExecutionResult lastResult = statusResponse.ExecutionInfo.LastResult;
```

Можно изменить его на следующий для устранения ошибок сборки:

```csharp
IndexerExecutionInfo status = null;

try
{
    status = _searchClient.Indexers.GetStatus(indexer.Name);
}
catch (Exception ex)
{
    Console.WriteLine("Error polling for indexer status: {0}", ex.Message);
    return;
}

IndexerExecutionResult lastResult = status.LastResult;
```

#### <a name="response-classes-and-ienumerable"></a>Классы ответа и IEnumerable
Дополнительное изменение, которое может повлиять на код, состоит в том, что классы ответов, содержащие коллекции, больше не реализуют `IEnumerable<T>`. Вместо этого вы можете получать прямой доступ к свойству коллекции. Например, если код выглядит следующим образом:

```csharp
DocumentSearchResponse<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
foreach (SearchResult<Hotel> result in response)
{
    Console.WriteLine(result.Document);
}
```

Можно изменить его на следующий для устранения ошибок сборки:

```csharp
DocumentSearchResult<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
foreach (SearchResult<Hotel> result in response.Results)
{
    Console.WriteLine(result.Document);
}
```

#### <a name="special-case-for-web-applications"></a>Исключение, касающееся веб-приложений
Если у вас есть веб-приложение, которое сериализует `DocumentSearchResponse` непосредственно для отправки результатов поиска в браузер, необходимо будет внести изменения в код, иначе результаты не будут сериализованы правильно. Например, если код выглядит следующим образом:

```csharp
public ActionResult Search(string q = "")
{
    // If blank search, assume they want to search everything
    if (string.IsNullOrWhiteSpace(q))
        q = "*";

    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = _featuresSearch.Search(q)
    };
}
```

Его можно изменить путем получения свойства `.Results` для ответа поиска, чтобы исправить отрисовку результата поиска.

```csharp
public ActionResult Search(string q = "")
{
    // If blank search, assume they want to search everything
    if (string.IsNullOrWhiteSpace(q))
        q = "*";

    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = _featuresSearch.Search(q).Results
    };
}
```

Вам придется найти такие случаи в коде самостоятельно. **Компилятор не предупредит вас**, так как `JsonResult.Data` имеет тип `object`.

### <a name="cloudexception-changes"></a>Изменения CloudException
Класс `CloudException` перемещен из пространства имен `Hyak.Common` в пространство имен `Microsoft.Rest.Azure`. Кроме того, его свойство `Error` было изменено на `Body`.

### <a name="searchserviceclient-and-searchindexclient-changes"></a>Изменения SearchServiceClient и SearchIndexClient
Тип свойства `Credentials` был изменен с `SearchCredentials` на базовый класс `ServiceClientCredentials`. Если необходимо получить доступ к `SearchCredentials` из `SearchIndexClient` или `SearchServiceClient`, используйте новое свойство `SearchCredentials`.

В более старых версиях пакета SDK у `SearchServiceClient` и `SearchIndexClient` были конструкторы, которые принимали параметр `HttpClient`. Они были заменены конструкторами, принимающими `HttpClientHandler` и массив объектов `DelegatingHandler`. При необходимости это упрощает установку пользовательских обработчиков для предварительной обработки HTTP-запросов.

Наконец, изменились конструкторы, которые принимали `Uri` и `SearchCredentials`. Например, если у вас есть код, который выглядит следующим образом:

```csharp
var client =
    new SearchServiceClient(
        new SearchCredentials("abc123"),
        new Uri("http://myservice.search.windows.net"));
```

Можно изменить его на следующий для устранения ошибок сборки:

```csharp
var client =
    new SearchServiceClient(
        new Uri("http://myservice.search.windows.net"),
        new SearchCredentials("abc123"));
```

Также обратите внимание, что тип параметра учетных данных изменился на `ServiceClientCredentials`. Это вряд ли повлияет на код, так как `SearchCredentials` является производным от `ServiceClientCredentials`.

### <a name="passing-a-request-id"></a>Передача идентификатора запроса
В более старых версиях пакета SDK можно было задать идентификатор запроса в `SearchServiceClient` или `SearchIndexClient`, и он включался в каждый запрос к REST API. Это полезно для устранения неполадок в службе поиска, если необходимо обратиться в службу поддержки. Однако более полезно присвоить уникальный идентификатор запроса для каждой операции, а не использовать тот же идентификатор для всех операций. По этой причине методы `SetClientRequestId` из `SearchServiceClient` и `SearchIndexClient` были удалены. Вместо этого вы можете передать идентификатор запроса для каждого метода операции через необязательный параметр `SearchRequestOptions` .

> [!NOTE]
> В будущем выпуске пакета SDK мы добавим новый механизм для глобальной настройки идентификатора запроса на объектах клиента, который согласован с подходом, используемым другим пакетом SDK Azure.
> 
> 

### <a name="example"></a>Пример
Если у вас есть код, который выглядит следующим образом:

```csharp
client.SetClientRequestId(Guid.NewGuid());
...
long count = client.Documents.Count();
```

Можно изменить его на следующий для устранения ошибок сборки:

```csharp
long count = client.Documents.Count(new SearchRequestOptions(requestId: Guid.NewGuid()));
```

### <a name="interface-name-changes"></a>Изменение имени интерфейса
Имена интерфейсов группы операций изменены, чтобы согласовать их с соответствующими именами свойств:

* Тип `ISearchServiceClient.Indexes` был переименован из `IIndexOperations` в `IIndexesOperations`.
* Тип `ISearchServiceClient.Indexers` был переименован из `IIndexerOperations` в `IIndexersOperations`.
* Тип `ISearchServiceClient.DataSources` был переименован из `IDataSourceOperations` в `IDataSourcesOperations`.
* Тип `ISearchIndexClient.Documents` был переименован из `IDocumentOperations` в `IDocumentsOperations`.

Это изменение вряд ли повлияет на код, если вы не создали макеты этих интерфейсов для тестирования.

<a name="BugFixesV1"></a>

## <a name="bug-fixes-in-version-11"></a>Исправления в версии 1.1
В более старых версиях пакета SDK для .NET для поиска Azure была ошибка, связанная с сериализацией классов пользовательской модели. Ошибка могла произойти, если вы создавали класс пользовательской модели со свойством типа значения, не допускающего нуля.

### <a name="steps-to-reproduce"></a>Шаги для воспроизведения
Создайте класс пользовательской модели со свойством типа значения, не допускающего нуля. Например, добавьте открытое свойство `UnitCount` типа `int` вместо `int?`.

При индексировании документа со значением по умолчанию этого типа (например 0 для `int`) поле будет иметь значение NULL в Поиске Azure. Если впоследствии выполняется поиск этого документа, вызов `Search` выдаст `JsonSerializationException` с сообщением о том, что невозможно преобразовать `null` в `int`.

Кроме того, фильтры могут не работать, как ожидалось, поскольку в индекс был записано значение null вместо ожидаемого значения.

### <a name="fix-details"></a>Сведения об исправлении
Эта проблема устранена в пакете SDK версии 1.1. Теперь, если имеется такой класс модели:

```csharp
public class Model
{
    public string Key { get; set; }

    public int IntValue { get; set; }
}
```

и вы устанавливаете для `IntValue` значение 0, это значение теперь правильно сериализуется как 0 по сети и сохраняется как 0 в индексе. Циклы обработки также работают ожидаемым образом.

Существует одна потенциальная проблема, которую следует учитывать при таком подходе: при использовании типа модели со свойством, не допускающим значения NULL, необходимо **гарантировать** , что документы в индексе не будут содержать значение NULL для соответствующего поля. Ни пакет SDK, ни API REST поиска Azure не поможет вам это обеспечить.

Это не просто гипотетическое соображение. Представьте себе ситуацию, когда вы добавляете новое поле в существующий индекс с типом `Edm.Int32`. После обновления определения индекса все документы будут иметь значение null для этого нового поля (поскольку все типы допускают значение NULL в службе поиска Azure). Если затем для этого поля вы используете класс модели со свойством `int`, не допускающим нулевое значение, при попытке получения документов вы получите `JsonSerializationException` следующим образом:

```output
Error converting value {null} to type 'System.Int32'. Path 'IntValue'.
```

По этой причине по-прежнему рекомендуется использовать типы, допускающие значения NULL, в классах модели.

Дополнительные сведения по этой ошибке и исправлению см. в разделе об [этой проблеме на GitHub](https://github.com/Azure/azure-sdk-for-net/issues/1063).