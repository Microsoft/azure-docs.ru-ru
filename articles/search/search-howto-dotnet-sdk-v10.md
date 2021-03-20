---
title: Использование Microsoft. Azure. Search (V10) в .NET
titleSuffix: Azure Cognitive Search
description: Узнайте, как создавать объекты поиска и управлять ими в приложении .NET с помощью C# и Microsoft. Azure. Search (версия 10) пакета SDK для .NET. Фрагменты кода демонстрируют подключение к службе, создание индексов и запросов.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.devlang: dotnet
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/27/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 11102d95bc2aba65e6bc3cba71805a67f195947b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92681718"
---
# <a name="how-to-use-microsoftazuresearch-in-a-c-net-application"></a>Как использовать Microsoft. Azure. Search в приложении C# .NET

В этой статье объясняется, как создавать объекты поиска и управлять ими с помощью C# и устаревшей клиентской библиотеки [**Microsoft. Azure. Search**](/dotnet/api/overview/azure/search/client10) (версия 10) в пакете Azure SDK для .NET. 

Версия 10 — это последняя версия пакета Microsoft. Azure. Search. После перемещения новые функции будут выдаваться в [**Azure.Search.Docументс**](/dotnet/api/overview/azure/search.documents-readme) из группы Azure SDK.

> [!NOTE]
> При наличии существующих или порядковых проектов разработки можно продолжать использовать версию 10. Для новых проектов или для использования новых функций следует перейти к [новой библиотеке](/dotnet/api/overview/azure/search.documents-readme).

## <a name="about-version-10"></a>О версии 10

Пакет SDK содержит клиентские библиотеки, позволяющие управлять индексами, источниками данных, индексаторами и картами синонимов, а также отправлять документы, управлять ими и выполнять запросы, не вникая в детали HTTP и JSON. Эти клиентские библиотеки распределяются как пакеты NuGet.

Главный пакет NuGet — `Microsoft.Azure.Search`. Он представляет собой мета пакет, содержащий все остальные пакеты в виде зависимостей. Используйте этот пакет, если вы только начинаете работу или знаете, что приложение потребует всех функций Когнитивный поиск Azure.

Ниже перечислены другие пакеты NuGet в пакете SDK.
 
  - `Microsoft.Azure.Search.Data`: Используйте этот пакет, если вы разрабатываете приложение .NET с помощью Azure Когнитивный поиск и вам нужно только запрашивать или обновлять документы в индексах. Если требуется также создавать или обновлять индексы, карты синонимов и другие ресурсы уровня службы, используйте пакет `Microsoft.Azure.Search`.
  - `Microsoft.Azure.Search.Service`: Используйте этот пакет, если вы разрабатываете автоматизацию в .NET для управления индексами Azure Когнитивный поиск, сопоставлениями синонимов, индексаторами, источниками данных или другими ресурсами уровня службы. Если требуется только запрашивать и обновлять документы в индексах, используйте пакет `Microsoft.Azure.Search.Data`. Если вам нужны все функции Когнитивный поиск Azure, используйте `Microsoft.Azure.Search` вместо этого пакет.
  - `Microsoft.Azure.Search.Common`: Общие типы, необходимые для библиотек Azure Когнитивный поиск .NET. Этот пакет не требуется использовать непосредственно в приложении. Он предназначен только для использования в качестве зависимости.

Различные клиентские библиотеки определяют такие классы, как `Index`, `Field` и `Document`, а также такие операции, как `Indexes.Create` и `Documents.Search` в классах `SearchServiceClient` и `SearchIndexClient`. Эти классы упорядочены в следующие подразделы:

* [Microsoft.Azure.Search](/dotnet/api/microsoft.azure.search)
* [Microsoft.Azure.Search.Models](/dotnet/api/microsoft.azure.search.models)

Если вы хотите оставить отзыв о будущем обновлении пакета SDK, ознакомьтесь со [страницей отзывов](https://feedback.azure.com/forums/263029-azure-search/) или создайте вопрос на сайте [GitHub](https://github.com/azure/azure-sdk-for-net/issues) и приведем название "Azure когнитивный Поиск" в названии проблемы.

Версия пакета SDK для .NET предназначена [ `2019-05-06` для REST API когнитивный Поиск Azure](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/search/data-plane/Microsoft.Azure.Search.Data/stable/2019-05-06). Эта версия включает поддержку [сложных типов](search-howto-complex-data-types.md), [обогащения AI](cognitive-search-concept-intro.md), [автозаполнения](/rest/api/searchservice/autocomplete)и [режима синтаксического анализа Жсонлинес](search-howto-index-json-blobs.md) при индексировании больших двоичных объектов Azure. 

Этот пакет SDK не поддерживает такие [операции управления](/rest/api/searchmanagement/), как создание и масштабирование служб поиска или управление ключами API. Если вам нужно управлять ресурсами поиска из приложения .NET, можно использовать [пакет SDK для управления Azure когнитивный Поиск .NET](/dotnet/api/overview/azure/search/management).

## <a name="upgrade-to-v10"></a>Обновление до v10
Если вы уже используете более раннюю версию пакета SDK для Azure Когнитивный поиск .NET и хотите перейти на последнюю версию общедоступной версии, в [этой статье](search-dotnet-sdk-migration-version-9.md) объясняется, как это делать.

## <a name="sdk-requirements"></a>Требования пакета SDK
1. Visual Studio 2017 или более поздней версии.
2. Ваша собственная служба Когнитивный поиск Azure. Чтобы использовать пакет SDK, необходимо указать имя службы и один или несколько ключей API. [Создание службы на портале](search-create-service-portal.md) поможет вам это сделать.
3. Скачайте [пакет NuGet пакета](https://www.nuget.org/packages/Microsoft.Azure.Search) SDK для Azure когнитивный Поиск .NET с помощью команды "Управление пакетами NuGet" в Visual Studio. Достаточно найти имя пакета `Microsoft.Azure.Search` на сайте NuGet.org (или одно из указанных выше имен пакета, если требуется только подмножество функциональных возможностей).

Пакет SDK Azure Когнитивный поиск .NET поддерживает приложения, предназначенные для платформа .NET Framework 4.5.2 и более поздних версий, а также .NET Core 2,0 и более поздних версий.

## <a name="core-scenarios"></a>Основные сценарии
В приложении поиска необходимо выполнить несколько действий. В данном руководстве мы обсудим эти основные сценарии:

* Создание индекса
* Заполнение индекса документами
* Поиск документов с помощью полнотекстового поиска и фильтров

В следующем примере кода показан каждый из этих сценариев. Вы можете использовать фрагменты кода в собственном приложении.

### <a name="overview"></a>Обзор
Образец приложения, который мы будем рассматривать, создает новый индекс с именем hotels, заполняет его несколькими документами, а затем выполняет некоторые поисковые запросы. Вот основная программа, показывающая общий поток.

```csharp
// This sample shows how to delete, create, upload documents and query an index
static void Main(string[] args)
{
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();

    SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

    string indexName = configuration["SearchIndexName"];

    Console.WriteLine("{0}", "Deleting index...\n");
    DeleteIndexIfExists(indexName, serviceClient);

    Console.WriteLine("{0}", "Creating index...\n");
    CreateIndex(indexName, serviceClient);

    ISearchIndexClient indexClient = serviceClient.Indexes.GetClient(indexName);

    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(indexClient);

    ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

    RunQueries(indexClientForQueries);

    Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
    Console.ReadKey();
}
```

> [!NOTE]
> Полный исходный код образца приложения, используемого в этом пошаговом руководстве, см. в репозитории [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).
> 
>

Мы рассмотрим ее шаг за шагом. Для начала необходимо создать `SearchServiceClient`. Этот объект позволяет управлять индексами. Чтобы создать его, необходимо указать имя службы Когнитивный поиск Azure, а также ключ API администратора. Эту информацию можно указать в файле `appsettings.json` из [примера приложения](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).

```csharp
private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];

    SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
    return serviceClient;
}
```

> [!NOTE]
> Если указать неправильный ключ (например, ключ запроса вместо ключа администратора), `SearchServiceClient` вызовет `CloudException` с сообщением об ошибке "Доступ запрещен" при первом вызове метода операции, например `Indexes.Create`. В этом случае проверьте ключ API еще раз.
> 
> 

Следующие несколько строк вызывают методы для создания индекса под названием hotels, предварительно удалив его, если он уже существует. Мы рассмотрим эти методы немного позже.

```csharp
Console.WriteLine("{0}", "Deleting index...\n");
DeleteIndexIfExists(indexName, serviceClient);

Console.WriteLine("{0}", "Creating index...\n");
CreateIndex(indexName, serviceClient);
```

Затем необходимо заполнить индекс. Чтобы заполнить индекс, нам потребуется `SearchIndexClient` . Его можно получить двумя способами: создать его самостоятельно или вызвать метод `Indexes.GetClient` в `SearchServiceClient`. Для удобства мы используем второй способ.

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient(indexName);
```

> [!NOTE]
> В типичном приложении поиска Управление индексами и заполнение может обрабатываться отдельным компонентом из поисковых запросов. `Indexes.GetClient` удобно использовать для заполнения индекса, так как он позволяет избежать дополнительных проблем `SearchCredentials` . Это достигается благодаря передачи ключа администратора, который использовался для создания `SearchServiceClient`, в новый элемент `SearchIndexClient`. Однако в части приложения, выполняющей запросы, лучше создать `SearchIndexClient` напрямую, чтобы можно было передать ключ запроса, который позволяет только считывать данные вместо ключа администратора. Это согласуется с принципом наименьших прав доступа и поможет сделать приложение более безопасным. Дополнительные сведения о ключах администраторов и запросов см. [здесь](/rest/api/searchservice/#authentication-and-authorization).
> 
> 

Теперь, когда у нас есть `SearchIndexClient`, мы можем заполнить индекс. Заполнение индекса выполняется другим методом, который мы рассмотрим позже.

```csharp
Console.WriteLine("{0}", "Uploading documents...\n");
UploadDocuments(indexClient);
```

Наконец, мы выполним несколько поисковых запросов и отобразим результаты. На этот раз мы используем другой метод `SearchIndexClient`:

```csharp
ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(indexName, configuration);

RunQueries(indexClientForQueries);
```

Метод `RunQueries` мы позже рассмотрим подробно. Это код для создания нового объекта `SearchIndexClient`:

```csharp
private static SearchIndexClient CreateSearchIndexClient(string indexName, IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, indexName, new SearchCredentials(queryApiKey));
    return indexClient;
}
```

На этот раз мы воспользуемся ключом запроса, так как доступ на запись к индексу не требуется. Эту информацию можно указать в файле `appsettings.json` из [примера приложения](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).

Если запустить это приложение с допустимым именем службы и ключами API, выходные данные должны выглядеть, как в следующем примере: (некоторые выходные данные консоли были заменены на "..." в целях иллюстрации.)

```output

Deleting index...

Creating index...

Uploading documents...

Waiting for documents to be indexed...

Search the entire index for the term 'motel' and return only the HotelName field:

Name: Secret Point Motel

Name: Twin Dome Motel


Apply a filter to the index to find hotels with a room cheaper than $100 per night, and return the hotelId and description:

HotelId: 1
Description: The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Times Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.

HotelId: 2
Description: The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.


Search the entire index, order by a specific field (lastRenovationDate) in descending order, take the top two results, and show only hotelName and lastRenovationDate:

Name: Triple Landscape Hotel
Last renovated on: 9/20/2015 12:00:00 AM +00:00

Name: Twin Dome Motel
Last renovated on: 2/18/1979 12:00:00 AM +00:00


Search the hotel names for the term 'hotel':

HotelId: 3
Name: Triple Landscape Hotel
...

Complete.  Press any key to end application... 
```

Полный исходный код приложения представлен в конце этой статьи.

Далее мы подробнее рассмотрим каждый из методов, вызываемых элементом `Main`.

### <a name="creating-an-index"></a>Создание индекса
После создания `SearchServiceClient` `Main` удаляет индекс гостиниц, если он уже существует. Удаление выполняется с помощью следующего метода:

```csharp
private static void DeleteIndexIfExists(string indexName, SearchServiceClient serviceClient)
{
    if (serviceClient.Indexes.Exists(indexName))
    {
        serviceClient.Indexes.Delete(indexName);
    }
}
```

Этот метод использует соответствующий `SearchServiceClient` , чтобы проверить наличие индекса, и если он существует, удалить его.

> [!NOTE]
> В примере кода в этой статье для простоты используются синхронные методы пакета SDK Azure Когнитивный поиск .NET. Рекомендуем использовать в ваших приложениях асинхронные методы, чтобы сохранить масштабируемость и скорость отклика. Например, в приведенном выше методе можно использовать `ExistsAsync` и `DeleteAsync` вместо `Exists` и `Delete`.
> 
> 

Затем `Main` создает новый индекс hotels, вызывая следующий метод:

```csharp
private static void CreateIndex(string indexName, SearchServiceClient serviceClient)
{
    var definition = new Index()
    {
        Name = indexName,
        Fields = FieldBuilder.BuildForType<Hotel>()
    };
    
    serviceClient.Indexes.Create(definition);
}
```

Этот метод создает новый объект `Index` со списком объектов `Field`, который определяет схему нового индекса. Каждое поле имеет имя, тип данных и несколько атрибутов, которые определяют его поведение при поиске. Класс `FieldBuilder` использует отражение, чтобы создать список объектов `Field` для соответствующего индекса. Он проверяет общедоступные свойства и атрибуты полученного класса модели `Hotel`. Класс `Hotel` мы рассмотрим подробно позже.

> [!NOTE]
> При необходимости вы всегда можете создать список объектов `Field` напрямую, а не с помощью `FieldBuilder`. Это полезно, например, если вы не хотите использовать класс модели или используете существующий класс модели, в который не хотите добавлять атрибуты.
>
> 

Помимо полей, в индекс можно добавить профили оценки, средства подбора или параметры CORS (эти параметры опущены в образце для краткости). Дополнительные сведения об объекте index и его составных частях см. в [справочнике по пакету SDK](/dotnet/api/microsoft.azure.search.models.index), а также в [справочнике по Azure когнитивный Поиск REST API](/rest/api/searchservice/).

### <a name="populating-the-index"></a>Заполнение индекса
Следующий шаг в `Main` заполняет только что созданный индекс. Это заполнение индекса выполняется в следующем методе: (часть кода заменена на "..." для целей иллюстрации.  См. полный пример решения для полного заполнения данных.)

```csharp
private static void UploadDocuments(ISearchIndexClient indexClient)
{
    var hotels = new Hotel[]
    {
        new Hotel()
        {
            HotelId = "1",
            HotelName = "Secret Point Motel",
            ...
            Address = new Address()
            {
                StreetAddress = "677 5th Ave",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Budget Room, 1 Queen Bed (Cityside)",
                    ...
                },
                new Room()
                {
                    Description = "Budget Room, 1 King Bed (Mountain View)",
                    ...
                },
                new Room()
                {
                    Description = "Deluxe Room, 2 Double Beds (City View)",
                    ...
                }
            }
        },
        new Hotel()
        {
            HotelId = "2",
            HotelName = "Twin Dome Motel",
            ...
            {
                StreetAddress = "140 University Town Center Dr",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Suite, 2 Double Beds (Mountain View)",
                    ...
                },
                new Room()
                {
                    Description = "Standard Room, 1 Queen Bed (City View)",
                    ...
                },
                new Room()
                {
                    Description = "Budget Room, 1 King Bed (Waterfront View)",
                    ...
                }
            }
        },
        new Hotel()
        {
            HotelId = "3",
            HotelName = "Triple Landscape Hotel",
            ...
            Address = new Address()
            {
                StreetAddress = "3393 Peachtree Rd",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Standard Room, 2 Queen Beds (Amenities)",
                    ...
                },
                new Room ()
                {
                    Description = "Standard Room, 2 Double Beds (Waterfront View)",
                    ...
                },
                new Room()
                {
                    Description = "Deluxe Room, 2 Double Beds (Cityside)",
                    ...
                }
            }
        }
    };

    var batch = IndexBatch.Upload(hotels);

    try
    {
        indexClient.Documents.Index(batch);
    }
    catch (IndexBatchException e)
    {
        // Sometimes when your Search service is under load, indexing will fail for some of the documents in
        // the batch. Depending on your application, you can take compensating actions like delaying and
        // retrying. For this simple demo, we just log the failed document keys and continue.
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

    Console.WriteLine("Waiting for documents to be indexed...\n");
    Thread.Sleep(2000);
}
```

Этот метод состоит из четырех частей. Первый создает массив из трех `Hotel` объектов, каждый из которых имеет 3 `Room` объекта, которые будут служить входными данными для отправки в индекс. Эти данные жестко запрограммированы для простоты. В вашем приложении данные скорее всего будут поступать из внешнего источника, например базы данных SQL.

Вторая часть создает `IndexBatch` , содержащий документы. Вы указываете операцию, которую необходимо применить к пакету во время его создания, в данном случае путем вызова метода `IndexBatch.Upload`. Затем пакет отправляется в индекс Когнитивный поиск Azure по `Documents.Index` методу.

> [!NOTE]
> В этом примере мы просто отправляем документы. Если требуется внести изменения в существующие документы или удалить документы, можно создать пакеты, вызвав методы `IndexBatch.Merge`, `IndexBatch.MergeOrUpload` или `IndexBatch.Delete`. Можно также смешивать различные операции в одном пакете, вызывая метод `IndexBatch.New` , который принимает коллекцию `IndexAction` объектов, каждый из которых сообщает когнитивный Поиск Azure выполнить определенную операцию над документом. Можно создать каждый `IndexAction` с отдельной операцией, вызвав соответствующий метод, например `IndexAction.Merge`, `IndexAction.Upload` и т. д.
> 
> 

Третья часть метода — это блок catch, который обрабатывает важные ошибки индексирования. Если службе Когнитивный поиск Azure не удается индексировать некоторые документы в пакете, создается `IndexBatchException` исключение `Documents.Index` . Это исключение может произойти при индексировании документов, пока служба находится в режиме высокой нагрузки. **Настоятельно рекомендуется явно обрабатывать этот случай в коде.**  Вы можете задержать и повторить попытку индексирования соответствующих документов либо занести ошибку в журнал и продолжить работу, как в нашем примере, а также выполнить другие действия в зависимости от требований вашего приложения к целостности данных.

> [!NOTE]
> Метод можно использовать [`FindFailedActionsToRetry`](/dotnet/api/microsoft.azure.search.indexbatchexception.findfailedactionstoretry) для создания нового пакета, содержащего только действия, которые завершились сбоем в предыдущем вызове метода `Index` . На сайте [StackOverflow](https://stackoverflow.com/questions/40012885/azure-search-net-sdk-how-to-use-findfailedactionstoretry) опубликовано обсуждение правил его использования.
>
>

И наконец, метод `UploadDocuments` вызывает задержку на две секунды. Индексирование выполняется асинхронно в службе Когнитивный поиск Azure, поэтому пример приложения должен подождать некоторое время, чтобы обеспечить доступность документов для поиска. Такие задержки обычно необходимы только в демонстрациях, тестах и примерах приложений.

<a name="how-dotnet-handles-documents"></a>

#### <a name="how-the-net-sdk-handles-documents"></a>Обработка документов пакетом SDK для .NET
Возможно, вас интересует, как пакет SDK Azure Когнитивный поиск для .NET может отправлять экземпляры определяемого пользователем класса, например в `Hotel` индекс. Чтобы ответить на этот вопрос, рассмотрим класс `Hotel` :

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Microsoft.Spatial;
using Newtonsoft.Json;

public partial class Hotel
{
    [System.ComponentModel.DataAnnotations.Key]
    [IsFilterable]
    public string HotelId { get; set; }

    [IsSearchable, IsSortable]
    public string HotelName { get; set; }

    [IsSearchable]
    [Analyzer(AnalyzerName.AsString.EnLucene)]
    public string Description { get; set; }

    [IsSearchable]
    [Analyzer(AnalyzerName.AsString.FrLucene)]
    [JsonProperty("Description_fr")]
    public string DescriptionFr { get; set; }

    [IsSearchable, IsFilterable, IsSortable, IsFacetable]
    public string Category { get; set; }

    [IsSearchable, IsFilterable, IsFacetable]
    public string[] Tags { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public bool? ParkingIncluded { get; set; }

    // SmokingAllowed reflects whether any room in the hotel allows smoking.
    // The JsonIgnore attribute indicates that a field should not be created 
    // in the index for this property and it will only be used by code in the client.
    [JsonIgnore]
    public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;

    [IsFilterable, IsSortable, IsFacetable]
    public DateTimeOffset? LastRenovationDate { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public double? Rating { get; set; }

    public Address Address { get; set; }

    [IsFilterable, IsSortable]
    public GeographyPoint Location { get; set; }

    public Room[] Rooms { get; set; }
}
```

Первое, что следует заметить: имя каждого открытого свойства в `Hotel` классе будет сопоставляться с полем с тем же именем в определении индекса. Если требуется, чтобы каждое поле начинались с строчной буквы («Camel»), пакет SDK может автоматически сопоставлять имена свойств с `[SerializePropertyNamesAsCamelCase]` атрибутом в стиле «Camel». Этот сценарий часто встречается в приложениях .NET, которые выполняют привязку данных, когда целевая схема находится вне контроля разработчика приложения без необходимости нарушать рекомендации по именованию стиля "Pascal" в .NET.

> [!NOTE]
> Пакет SDK Azure Когнитивный поиск .NET использует библиотеку [NewtonSoft JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm) для сериализации и десериализации пользовательских объектов модели в JSON и обратно. При необходимости эту сериализацию можно настроить. Дополнительные сведения см. [в разделе настраиваемая сериализация с помощью JSON.NET](#JsonDotNet).
> 
> 

Второе, что нужно отметить, — это каждое свойство, дополненное атрибутами, такими как `IsFilterable` ,, `IsSearchable` `Key` и `Analyzer` . Эти атрибуты сопоставляются непосредственно с [соответствующими атрибутами полей в индексе Azure когнитивный Поиск](/rest/api/searchservice/create-index). `FieldBuilder`Класс использует эти свойства для создания определений полей для индекса.

Третьим важным моментом для `Hotel` класса являются типы данных общих свойств. Типы .NET этих свойств сопоставляются с эквивалентными типами полей в определении индекса. Например, свойство строки `Category` сопоставляется с полем `category`, которое имеет тип `Edm.String`. Существуют аналогичные сопоставления типов между,, `bool?` `Edm.Boolean` `DateTimeOffset?` и `Edm.DateTimeOffset` т. д. Конкретные правила сопоставления типов описаны в описании `Documents.Get` метода в [справочнике по пакету SDK для Azure когнитивный Поиск .NET](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.get). Хотя класс `FieldBuilder` выполняет это сопоставление, этот механизм полезно понимать на случай, если вам понадобится устранять неполадки, связанные с сериализацией.

Возможно, вы заметите `SmokingAllowed` свойство?

```csharp
[JsonIgnore]
public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;
```

`JsonIgnore`Атрибут в этом свойстве указывает, что `FieldBuilder` не следует сериализовать его в индекс в качестве поля.  Это отличный способ создания вычисляемых свойств на стороне клиента, которые можно использовать в качестве вспомогательных функций в приложении.  В этом случае `SmokingAllowed` свойство отражает, `Room` `Rooms` разрешено ли в коллекции Курение.  Если все имеют значение false, это означает, что вся Гостиницы не допускает Курение.

Некоторые свойства, такие как `Address` и `Rooms` , являются экземплярами классов .NET.  Эти свойства представляют более сложные структуры данных, и в результате для них требуются поля со [сложным типом данных](./search-howto-complex-data-types.md) в индексе.

`Address`Свойство представляет набор из нескольких значений в `Address` классе, определенный ниже:

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Newtonsoft.Json;

namespace AzureSearch.SDKHowTo
{
    public partial class Address
    {
        [IsSearchable]
        public string StreetAddress { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string City { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string StateProvince { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string PostalCode { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string Country { get; set; }
    }
}
```

Этот класс содержит стандартные значения, используемые для описания адресов в США или Канаде. Такие типы можно использовать для группирования логических полей в индексе.

`Rooms`Свойство представляет массив `Room` объектов:

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Newtonsoft.Json;

namespace AzureSearch.SDKHowTo
{
    public partial class Room
    {
        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.EnMicrosoft)]
        public string Description { get; set; }

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.FrMicrosoft)]
        [JsonProperty("Description_fr")]
        public string DescriptionFr { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string Type { get; set; }

        [IsFilterable, IsFacetable]
        public double? BaseRate { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string BedOptions { get; set; }

        [IsFilterable, IsFacetable]
        public int SleepsCount { get; set; }

        [IsFilterable, IsFacetable]
        public bool? SmokingAllowed { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string[] Tags { get; set; }
    }
}
```

Модель данных в .NET и соответствующую ей схему индексов должны быть спроектированы для поддержки возможностей поиска, которые вы хотели бы предоставить конечному пользователю. Каждый объект верхнего уровня в .NET, документ IE в индексе, соответствует результату поиска, который будет представлен в пользовательском интерфейсе. Например, в приложении поиска отеля конечные пользователи могут захотеть искать по имени Гостиницы, функциям отеля или характеристикам определенной комнаты. Некоторые примеры запросов будут рассмотрены чуть позже.

Эта возможность использовать собственные классы для взаимодействия с документами в индексе работает в обоих направлениях. Кроме того, можно получить результаты поиска, чтобы пакет SDK автоматически десериализовать их по своему усмотрению, как будет показано в следующем разделе.

> [!NOTE]
> Пакет SDK Azure Когнитивный поиск .NET также поддерживает динамически типизированные документы с помощью `Document` класса, который представляет собой сопоставление ключей и значений имен полей со значениями полей. Это полезно в тех случаях, когда во время разработки вам неизвестна схема индекса или привязка к конкретным классам моделей нецелесообразна. Все методы пакета SDK, связанные с документами, имеют перегрузки, работающие с классом `Document`, а также строго типизированные перегрузки, принимающие параметр универсального типа. Пример кода в данном руководстве содержит только перегрузки второго типа. [ `Document` Класс](/dotnet/api/microsoft.azure.search.models.document) наследует от `Dictionary<string, object>` .
> 
>

**Почему следует использовать типы данных, допускающие значение NULL**

При проектировании собственных классов модели для соотнесения с индексом Когнитивный поиск Azure рекомендуется объявлять свойства типов значений, таких как `bool` и, `int` для допускающих значения NULL (например, `bool?` вместо `bool` ). При использовании ненулевого свойства необходимо **гарантировать** , что документы в индексе не будут содержать значение NULL для соответствующего поля. Ни пакет SDK, ни служба Когнитивный поиск Azure не помогут обеспечить это.

Это не просто гипотетическое соображение. Представьте себе ситуацию, когда вы добавляете новое поле в существующий индекс с типом `Edm.Int32`. После обновления определения индекса все документы будут иметь значение NULL для этого нового поля (так как все типы допускают значения NULL в Azure Когнитивный поиск). Если затем для этого поля вы используете класс модели со свойством `int`, не допускающим нулевое значение, при попытке получения документов вы получите `JsonSerializationException` следующим образом:

```output
Error converting value {null} to type 'System.Int32'. Path 'IntValue'.
```

По этой причине по-прежнему рекомендуется использовать типы, допускающие значения NULL, в классах модели.

<a name="JsonDotNet"></a>

#### <a name="custom-serialization-with-jsonnet"></a>Пользовательская сериализация с помощью JSON.NET
Пакет SDK использует JSON.NET для сериализации и десериализации документов. При необходимости можно настроить сериализацию и десериализацию, определив собственные `JsonConverter` или `IContractResolver` . Дополнительные сведения см. в [документации по JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm). Это может быть полезно, если вы хотите адаптировать существующий класс модели из приложения для использования с Azure Когнитивный поиск и другими более сложными сценариями. Например, с помощью пользовательской сериализации можно делать следующее.

* Включить или исключить определенные свойства класса модели с сохранением в виде поля документа.
* Реализовать сопоставление между именами свойств в коде и именами полей в индексе.
* Создать настраиваемые атрибуты, которые можно использовать для сопоставления свойств с полями документов.

Примеры реализации пользовательской сериализации можно найти в модульных тестах для пакета SDK Azure Когнитивный поиск .NET на сайте GitHub. Хорошей отправной точкой будет [эта папка](https://github.com/Azure/azure-sdk-for-net/tree/4f6f4e4c90200c1b0621c4cead302a91e89f2aba/sdk/search/Microsoft.Azure.Search/tests/Tests/Models). Она содержит классы, используемые в тестах пользовательской сериализации.

### <a name="searching-for-documents-in-the-index"></a>Поиск документов в индексе
Последний шаг в примере приложения — поиск некоторых документов в индексе:

```csharp
private static void RunQueries(ISearchIndexClient indexClient)
{
    SearchParameters parameters;
    DocumentSearchResult<Hotel> results;

    Console.WriteLine("Search the entire index for the term 'motel' and return only the HotelName field:\n");

    parameters =
        new SearchParameters()
        {
            Select = new[] { "HotelName" }
        };

    results = indexClient.Documents.Search<Hotel>("motel", parameters);

    WriteDocuments(results);

    Console.Write("Apply a filter to the index to find hotels with a room cheaper than $100 per night, ");
    Console.WriteLine("and return the hotelId and description:\n");

    parameters =
        new SearchParameters()
        {
            Filter = "Rooms/any(r: r/BaseRate lt 100)",
            Select = new[] { "HotelId", "Description" }
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.Write("Search the entire index, order by a specific field (lastRenovationDate) ");
    Console.Write("in descending order, take the top two results, and show only hotelName and ");
    Console.WriteLine("lastRenovationDate:\n");

    parameters =
        new SearchParameters()
        {
            OrderBy = new[] { "LastRenovationDate desc" },
            Select = new[] { "HotelName", "LastRenovationDate" },
            Top = 2
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.WriteLine("Search the entire index for the term 'hotel':\n");

    parameters = new SearchParameters();
    results = indexClient.Documents.Search<Hotel>("hotel", parameters);

    WriteDocuments(results);
}
```

При каждом выполнении запроса этот метод прежде всего создает объект `SearchParameters`. Этот объект используется для указания дополнительных параметров запроса, таких как сортировка, фильтрация, разбиение на страницы и аспекты. В этом методе мы задаем свойства `Filter`, `Select`, `OrderBy` и `Top` для разных запросов. Все свойства `SearchParameters` описаны [здесь](/dotnet/api/microsoft.azure.search.models.searchparameters).

Следующий этап — фактическое выполнение поискового запроса. Выполнение поиска выполняется с помощью `Documents.Search` метода. Для каждого запроса мы передаем текст для поиска в строковом формате (или `"*"`, если текст для поиска отсутствует) и созданные ранее параметры поиска. Мы также указываем `Hotel` как параметр типа `Documents.Search`, в результате чего пакет SDK будет десериализовать документы в результатах поиска к объектам типа `Hotel`.

> [!NOTE]
> Дополнительные сведения о синтаксисе выражений запросов см. [здесь](/rest/api/searchservice/Simple-query-syntax-in-Azure-Search).
> 
> 

А теперь, после завершения каждого запроса (поиска совпадений в результатах поиска), этот метод последовательно выводит в консоль все удовлетворяющие условиям документы.

```csharp
private static void WriteDocuments(DocumentSearchResult<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.Results)
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

Давайте подробнее рассмотрим каждый из запросов. Ниже приведен код для выполнения первого запроса:

```csharp
parameters =
    new SearchParameters()
    {
        Select = new[] { "HotelName" }
    };

results = indexClient.Documents.Search<Hotel>("motel", parameters);

WriteDocuments(results);
```

В этом случае мы ищем по всему индексу слово «Motel» в любом поле с возможностью поиска, и мы хотим получить только названия гостиниц, как указано в `Select` параметре. Результаты приведены ниже.

```output
Name: Secret Point Motel

Name: Twin Dome Motel
```

Следующий запрос является чуть более интересным.  Мы хотим найти все гостиницы, у которых есть комната со ставкой менее $100 и возвращающие только идентификатор отеля и описание:

```csharp
parameters =
    new SearchParameters()
    {
        Filter = "Rooms/any(r: r/BaseRate lt 100)",
        Select = new[] { "HotelId", "Description" }
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

Этот запрос использует выражение OData `$filter` (по значению `Rooms/any(r: r/BaseRate lt 100)`), чтобы отфильтровать в индексе нужные документы. При этом используется [оператор Any](./search-query-odata-collection-operators.md) для применения "басерате LT 100" к каждому элементу в коллекции помещений. Дополнительные сведения о синтаксисе OData, который поддерживает Azure Когнитивный поиск, можно найти [здесь](./query-odata-filter-orderby-syntax.md).

Вот результат выполнения этого запроса:

```output
HotelId: 1
Description: The hotel is ideally located on the main commercial artery of the city in the heart of New York...

HotelId: 2
Description: The hotel is situated in a nineteenth century plaza, which has been expanded and renovated to...
```

Теперь нам нужно найти две гостиницы с самым свежим ремонтом и отобразить для них название и дату ремонта. Ниже приведен код: 

```csharp
parameters =
    new SearchParameters()
    {
        OrderBy = new[] { "LastRenovationDate desc" },
        Select = new[] { "HotelName", "LastRenovationDate" },
        Top = 2
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

В этом случае мы снова используем синтаксис OData, присвоив параметру `OrderBy` значение `lastRenovationDate desc`. Мы также указываем значение 2 для параметра `Top`, чтобы получить только два первых документа. Как и ранее, параметр `Select` содержит поля, которые мы хотим получить.

Результаты приведены ниже.

```output
Name: Fancy Stay        Last renovated on: 6/27/2010 12:00:00 AM +00:00
Name: Roach Motel       Last renovated on: 4/28/1982 12:00:00 AM +00:00
```

Наконец, мы хотим найти все названия гостиниц, соответствующие слову «Гостиница»:

```csharp
parameters = new SearchParameters()
{
    SearchFields = new[] { "HotelName" }
};
results = indexClient.Documents.Search<Hotel>("hotel", parameters);

WriteDocuments(results);
```

Ниже представлены результаты этого запроса. Они включают все поля, так как мы не указали свойство `Select`:

```output
    HotelId: 3
    Name: Triple Landscape Hotel
    ...
```

На этом руководство заканчивается, но вам не стоит останавливаться. * * Дальнейшие действия предоставляют дополнительные ресурсы для изучения Когнитивный поиск Azure.

## <a name="next-steps"></a>Дальнейшие действия
* Изучите справочную информацию о [пакете SDK для .NET](/dotnet/api/microsoft.azure.search) и [REST API](/rest/api/searchservice/).
* Изучите [соглашения о наименовании](/rest/api/searchservice/Naming-rules) , чтобы узнать правила именования различных объектов.
* Проверьте [Поддерживаемые типы данных](/rest/api/searchservice/Supported-data-types) в Azure когнитивный Поиск.