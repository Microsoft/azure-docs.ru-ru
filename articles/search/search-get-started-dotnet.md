---
title: Краткое руководство. Создание индекса поиска в .NET
titleSuffix: Azure Cognitive Search
description: В этом руководстве по C# показано, как создать индекс, загрузить данные и выполнить запросы с помощью клиентской библиотеки Azure.Search.Documents.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 11/20/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: f0d912d5b14932c43d109f8f955d5f16381cf773
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98180104"
---
# <a name="quickstart-create-a-search-index-using-the-azuresearchdocuments-client-library"></a>Краткое руководство. Создание индекса поиска с помощью клиентской библиотеки Azure.Search.Documents

С помощью новой [клиентской библиотеки Azure.Search.Documents (версия 11)](/dotnet/api/overview/azure/search.documents-readme) вы можете создать консольное приложение .NET Core на языке C#, которое создает, загружает и запрашивает индекс поиска.

Вы можете [скачать исходный код](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart/v11), чтобы начать работу с готовым проектом, или выполнить описанные в этой статье действия, чтобы создать собственный проект.

> [!NOTE]
> Вам нужна более ранняя версия? См. статью [Создание индекса поиска с помощью клиентской библиотеки Microsoft.Azure.Search версии 10](search-get-started-dotnet-v10.md).

## <a name="prerequisites"></a>Предварительные требования

Прежде чем начать работу, убедитесь, что у вас есть следующие средства и службы:

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/) бесплатно.

+ Служба "Когнитивный поиск Azure". [Создайте новую](search-create-service-portal.md) или [найдите существующую службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices). Вы можете использовать бесплатную службу для выполнения инструкций, описанных в этом кратком руководстве. 

+ [Visual Studio](https://visualstudio.microsoft.com/downloads/) (любой выпуск). Пример кода был протестирован в Visual Studio 2019 (бесплатный выпуск Community).

При настройке проекта вы скачиваете [пакет NuGet Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/).

Пакет SDK Azure для .NET соответствует [.NET Standard 2.0](/dotnet/standard/net-standard#net-implementation-support), что означает .NET Framework 4.6.1 и .NET Core 2.0 соответствуют минимальным требованиям.

## <a name="set-up-your-project"></a>Настройка проекта

Соберите сведения о подключении к службе, а затем откройте Visual Studio, чтобы создать новый проект консольного приложения, который можно запустить в .NET Core.

<a name="get-service-info"></a>

### <a name="copy-a-key-and-endpoint"></a>Копирование ключа и конечной точки

Вызовы к службе требуют конечную точку URL-адреса и ключ доступа при каждом запросе. В первую очередь найдите ключ API и URL-адрес для добавления в проект. При создании клиента на более позднем этапе необходимо указать оба значения.

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

2. В разделе **Параметры** > **Ключи** получите ключ администратора с полным доступом к службе, требуемый для создания и удаления объектов. Существует две взаимозаменяемых пары первичного и вторичного ключей. Вы можете использовать любую из этих пар.

   ![Получение конечной точки HTTP и ключа доступа](media/search-get-started-rest/get-url-key.png "Получение конечной точки HTTP и ключа доступа")

Для выполнения любого запроса к службе требуется использование ключа API. Если есть действительный ключ, для каждого запроса устанавливаются отношения доверия между приложением, которое отправляет запрос, и службой, которая его обрабатывает.

### <a name="install-the-nuget-package"></a>Установка пакета NuGet

После создания проекта добавьте клиентскую библиотеку. [Пакет Azure.Search.Documents](https://www.nuget.org/packages/Azure.Search.Documents/) состоит из одной клиентской библиотеки, которая предоставляет все API для работы со службой поиска в .NET.

1. Запустите Visual Studio и создайте консольное приложение .NET Core.

1. В разделе **Инструменты** > **Диспетчер пакетов NuGet** выберите  **Управление пакетами NugGet для решения...** . 

1. Нажмите кнопку **Обзор**.

1. Найдите `Azure.Search.Documents` и выберите версию 11.0 или более позднюю.

1. Щелкните **Установить** справа, чтобы добавить сборку в проект и решение.

### <a name="create-a-search-client"></a>Создание клиента для поиска

1. В файле **Program.cs** измените пространство имен на `AzureSearch.SDK.Quickstart.v11`, а затем добавьте следующие директивы `using`.

   ```csharp
   using Azure;
   using Azure.Search.Documents;
   using Azure.Search.Documents.Indexes;
   using Azure.Search.Documents.Indexes.Models;
   using Azure.Search.Documents.Models;
   ```

1. Создайте два клиента: [SearchIndexClient](/dotnet/api/azure.search.documents.indexes.searchindexclient) создает индекс, а [SearchClient](/dotnet/api/azure.search.documents.searchclient) загружает и запрашивает существующий индекс. Обоим клиентам нужны конечная точка службы и ключ API администрирования, чтобы пройти проверку подлинности и получить права на создание и удаление.

   ```csharp
   static void Main(string[] args)
   {
       string serviceName = "<YOUR-SERVICE-NAME>";
       string indexName = "hotels-quickstart";
       string apiKey = "<YOUR-ADMIN-API-KEY>";

        // Create a SearchIndexClient to send create/delete index commands
        Uri serviceEndpoint = new Uri($"https://{serviceName}.search.windows.net/");
        AzureKeyCredential credential = new AzureKeyCredential(apiKey);
        SearchIndexClient adminClient = new SearchIndexClient(serviceEndpoint, credential);

        // Create a SearchClient to load and query documents
        SearchClient srchclient = new SearchClient(serviceEndpoint, indexName, credential);
    ```

## <a name="1---create-an-index"></a>1\. Создание индекса

В этом кратком руководстве показано, как создать индекс Hotels, в который вы позже загрузите данные об отелях и к которому будете выполнять запросы. На этом шаге следует определить поля в этом индексе. Каждое определение поля содержит имя, тип данных и атрибуты, которые определяют способ использования этого поля.

В нашем примере для простоты и удобочитаемости используются синхронные методы библиотеки Azure.Search.Documents. Но для рабочих сценариев лучше использовать асинхронные методы, чтобы приложение было масштабируемым и отзывчивым. Например, следует использовать [CreateIndexAsync](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindexasync) вместо [CreateIndex](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindex).

1. Добавьте в проект два пустых определения классов: **Hotel.cs**

1. Скопируйте следующий код в **Hotel.cs**, чтобы определить структуру документа hotel. Атрибуты поля определяют его использование в приложении. Например, атрибут `IsFilterable` должен быть присвоен каждому полю, которое поддерживает выражение фильтра.

    ```csharp
    using System;
    using System.Text.Json.Serialization;
    using Azure.Search.Documents.Indexes;
    using Azure.Search.Documents.Indexes.Models;

    namespace AzureSearch.Quickstart
    {
        public partial class Hotel
        {
            [SimpleField(IsKey = true, IsFilterable = true)]
            public string HotelId { get; set; }

            [SearchableField(IsSortable = true)]
            public string HotelName { get; set; }

            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnLucene)]
            public string Description { get; set; }

            [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.FrLucene)]
            [JsonPropertyName("Description_fr")]
            public string DescriptionFr { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string Category { get; set; }

            [SearchableField(IsFilterable = true, IsFacetable = true)]
            public string[] Tags { get; set; }

            [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public bool? ParkingIncluded { get; set; }

            [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public DateTimeOffset? LastRenovationDate { get; set; }

            [SimpleField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public double? Rating { get; set; }

            [SearchableField]
            public Address Address { get; set; }
        }
    }
    ```

   В клиентской библиотеке Azure.Search.Documents можно использовать [SearchableField](/dotnet/api/azure.search.documents.indexes.models.searchablefield) и [SimpleField](/dotnet/api/azure.search.documents.indexes.models.simplefield) для упрощения определения полей. Оба метода являются производными от [SearchField](/dotnet/api/azure.search.documents.indexes.models.searchfield) и потенциально могут упростить код:

   + `SimpleField` может быть любым типом данных, он всегда недоступен для поиска (он игнорируется для запросов полнотекстового поиска) и его можно извлечь (он не скрыт). Другие атрибуты отключены по умолчанию, но их можно включить. `SimpleField` можно использовать для идентификаторов документов или полей, используемых только в фильтрах, аспектах или профилях оценки. Если это так, обязательно примените все необходимые для сценария атрибуты, например `IsKey = true` для идентификатора документа. Чтобы узнать больше, см. [SimpleFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SimpleFieldAttribute.cs) в исходном коде.

   + `SearchableField` должен быть строкой и всегда доступен для поиска и извлечения. Другие атрибуты отключены по умолчанию, но их можно включить. Так как этот тип поля доступен для поиска, он поддерживает синонимы и полное дополнение свойств анализатора. Чтобы узнать больше, см. [SearchableFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SearchableFieldAttribute.cs) в исходном коде.

   Независимо от того, используется ли базовый API `SearchField` или одна из вспомогательных моделей, необходимо явно включить атрибуты фильтров, аспектов и сортировки. Например, [IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable), [IsSortable](/dotnet/api/azure.search.documents.indexes.models.searchfield.issortable) и [IsFacetable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfacetable) должны присваиваться явным образом, как в примере выше. 

1. Добавьте в проект второе пустое определение класса: **Address.cs**.  Скопируйте приведенный ниже код и вставьте его в класс.

   ```csharp
   using Azure.Search.Documents.Indexes;

    namespace AzureSearch.Quickstart
    {
        public partial class Address
        {
            [SearchableField(IsFilterable = true)]
            public string StreetAddress { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string City { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string StateProvince { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string PostalCode { get; set; }

            [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
            public string Country { get; set; }
        }
    }
   ```

1. Создайте еще два класса: **Hotel.Methods.cs** и **Address.Methods.cs** для переопределений ToString(). Эти классы используются для отображения результатов поиска в выходных данных консоли.  Содержимое этих классов не указано в этой статье, однако вы можете скопировать код из [файлов на сайте GitHub](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/quickstart/v11/AzureSearchQuickstart-v11).

1. В **Program.cs** создайте объект [SearchClient](/dotnet/api/azure.search.documents.indexes.models.searchindex), а затем вызовите метод [CreateIndex](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindex), чтобы отразить индекс в службе поиска. Индекс также содержит [SearchSuggester](/dotnet/api/azure.search.documents.indexes.models.searchsuggester), чтобы применить автозаполнение для указанных полей.

   ```csharp
    // Create hotels-quickstart index
    private static void CreateIndex(string indexName, SearchIndexClient adminClient)
    {
        FieldBuilder fieldBuilder = new FieldBuilder();
        var searchFields = fieldBuilder.Build(typeof(Hotel));

        var definition = new SearchIndex(indexName, searchFields);

        var suggester = new SearchSuggester("sg", new[] { "HotelName", "Category", "Address/City", "Address/StateProvince" });
        definition.Suggesters.Add(suggester);

        adminClient.CreateOrUpdateIndex(definition);
    }
   ```

<a name="load-documents"></a>

## <a name="2---load-documents"></a>2\. Загрузка документов

Когнитивный поиск Azure выполняет поиск по содержимому, которое хранится в этой службе. На этом шаге вы отправите документы JSON, которые соответствуют формату созданного индекса отелей.

Искомые документы в службе "Когнитивный поиск Azure" представляют собой структуры данных, которые служат входными данными для индексирования и (или) результатами запросов. Полученные из внешнего источника данных входные документы могут содержать строки базы данных, большие двоичные объекты из хранилища BLOB-объектов или сохраненные на диске документы JSON. В нашем примере мы выбрали самый простой путь, внедрив прямо в код документы JSON с информацией о четырех отелях. 

При отправке документов необходимо использовать объект [IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1). Объект `IndexDocumentsBatch` содержит коллекцию [действий](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1.actions), каждое из которых содержит документ и свойство с описанием действия, которое должен выполнить Когнитивный поиск Azure ([отправка, объединение, удаление и mergeOrUpload](search-what-is-data-import.md#indexing-actions)).

1. В файле **Program.cs** создайте массив документов и действий индексирования, а затем передайте этот массив в `IndexDocumentsBatch`. Приведенные ниже документы соответствуют индексу hotels-quickstart, как определено классом hotel.

    ```csharp
    // Upload documents in a single Upload request.
    private static void UploadDocuments(SearchClient searchClient)
    {
        IndexDocumentsBatch<Hotel> batch = IndexDocumentsBatch.Create(
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "1",
                    HotelName = "Secret Point Motel",
                    Description = "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
                    DescriptionFr = "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
                    Category = "Boutique",
                    Tags = new[] { "pool", "air conditioning", "concierge" },
                    ParkingIncluded = false,
                    LastRenovationDate = new DateTimeOffset(1970, 1, 18, 0, 0, 0, TimeSpan.Zero),
                    Rating = 3.6,
                    Address = new Address()
                    {
                        StreetAddress = "677 5th Ave",
                        City = "New York",
                        StateProvince = "NY",
                        PostalCode = "10022",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "2",
                    HotelName = "Twin Dome Motel",
                    Description = "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
                    DescriptionFr = "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
                    Category = "Boutique",
                    Tags = new[] { "pool", "free wifi", "concierge" },
                    ParkingIncluded = false,
                    LastRenovationDate = new DateTimeOffset(1979, 2, 18, 0, 0, 0, TimeSpan.Zero),
                    Rating = 3.60,
                    Address = new Address()
                    {
                        StreetAddress = "140 University Town Center Dr",
                        City = "Sarasota",
                        StateProvince = "FL",
                        PostalCode = "34243",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "3",
                    HotelName = "Triple Landscape Hotel",
                    Description = "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
                    DescriptionFr = "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
                    Category = "Resort and Spa",
                    Tags = new[] { "air conditioning", "bar", "continental breakfast" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(2015, 9, 20, 0, 0, 0, TimeSpan.Zero),
                    Rating = 4.80,
                    Address = new Address()
                    {
                        StreetAddress = "3393 Peachtree Rd",
                        City = "Atlanta",
                        StateProvince = "GA",
                        PostalCode = "30326",
                        Country = "USA"
                    }
                }),
            IndexDocumentsAction.Upload(
                new Hotel()
                {
                    HotelId = "4",
                    HotelName = "Sublime Cliff Hotel",
                    Description = "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
                    DescriptionFr = "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
                    Category = "Boutique",
                    Tags = new[] { "concierge", "view", "24-hour front desk service" },
                    ParkingIncluded = true,
                    LastRenovationDate = new DateTimeOffset(1960, 2, 06, 0, 0, 0, TimeSpan.Zero),
                    Rating = 4.60,
                    Address = new Address()
                    {
                        StreetAddress = "7400 San Pedro Ave",
                        City = "San Antonio",
                        StateProvince = "TX",
                        PostalCode = "78216",
                        Country = "USA"
                    }
                })
            );

        try
        {
            IndexDocumentsResult result = searchClient.IndexDocuments(batch);
        }
        catch (Exception)
        {
            // If for some reason any documents are dropped during indexing, you can compensate by delaying and
            // retrying. This simple demo just logs the failed document keys and continues.
            Console.WriteLine("Failed to index some of the documents: {0}");
        }
    }
    ```

    Создав экземпляр объекта [IndexDocumentsBatch](/dotnet/api/azure.search.documents.models.indexdocumentsbatch-1), вы сможете отправить его в индекс, вызвав [IndexDocuments](/dotnet/api/azure.search.documents.searchclient.indexdocuments) из объекта [SearchClient](/dotnet/api/azure.search.documents.searchclient).

1. Добавьте следующие строки в Main(). Загрузка документов выполняется с помощью SearchClient, однако для операции также понадобятся права администратора для службы, которая обычно связана с SearchIndexClient. Одним из способов настройки этой операции является получение SearchClient через SearchIndexClient (в этом примере — adminClient).

   ```csharp
    SearchClient ingesterClient = adminClient.GetSearchClient(indexName);

    // Load documents
    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(ingesterClient);
   ```

1. Так как это консольное приложение выполняет все команды последовательно, включите 2-секундные паузы между операциями индексирования и запросов.

    ```csharp
    // Wait 2 seconds for indexing to complete before starting queries (for demo and console-app purposes only)
    Console.WriteLine("Waiting for indexing...\n");
    System.Threading.Thread.Sleep(2000);
    ```

    2-секундная задержка дает достаточно времени для асинхронного индексирования, чтобы все документы уже были проиндексированы перед выполнением запросов. Задержки в коде обычно используются только в демонстрациях, тестах и примерах приложений.

## <a name="3---search-an-index"></a>3\. Поиск в индексе

Результаты запросов можно получить сразу по завершении индексирования первого документа, но для полноценного тестирования индекса придется подождать, пока закончится индексирование всех документов.

В этом разделе мы добавим две новые функции: логику запроса и результаты. Для запросов примените метод [Search](/dotnet/api/azure.search.documents.searchclient.search). Этот метод принимает текст для поиска (строку запроса) и другие [параметры](/dotnet/api/azure.search.documents.searchoptions).

Класс [SearchResults](/dotnet/api/azure.search.documents.models.searchresults-1) представляет результаты запроса.

1. В файле **Program.cs** создайте метод **WriteDocuments**, который выводит на консоль результаты поиска.

    ```csharp
    // Write search results to console
    private static void WriteDocuments(SearchResults<Hotel> searchResults)
    {
        foreach (SearchResult<Hotel> result in searchResults.GetResults())
        {
            Console.WriteLine(result.Document);
        }

        Console.WriteLine();
    }
    ```

1. Создайте метод **RunQueries** для выполнения запросов и возврата результатов. Результаты имеют формат объектов Hotel. В этом примере показана сигнатура метода и первый запрос. В этом запросе представлен параметр Select, позволяющий составить результат, используя выбранные поля документа.

    ```csharp
    // Run queries, use WriteDocuments to print output
    private static void RunQueries(SearchClient srchclient)
    {
        SearchOptions options;
        SearchResults<Hotel> response;

        Console.WriteLine("Query #1: Search on empty term '*' to return all documents, showing a subset of fields...\n");

        options = new SearchOptions()
        {
            IncludeTotalCount = true,
            Filter = "",
            OrderBy = { "" }
        };

        options.Select.Add("HotelId");
        options.Select.Add("HotelName");
        options.Select.Add("Address/City");

        response = srchclient.Search<Hotel>("*", options);
        WriteDocuments(response);
    ```

1. Во втором запросе выполните поиск по термину, добавьте фильтр для выбора документов с оценкой больше 4, а затем выполните сортировку по оценке в порядке убывания. Фильтром называется логическое выражение, которое оценивается для всех полей в индексе с атрибутом [IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable). Фильтрующие запросы могут включать или исключать указанные значения. Поэтому для фильтрующего запроса не существует оценки релевантности. 

    ```csharp
    Console.WriteLine("Query #2: Search on 'hotels', filter on 'Rating gt 4', sort by Rating in descending order...\n");

    options = new SearchOptions()
    {
        Filter = "Rating gt 4",
        OrderBy = { "Rating desc" }
    };

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Rating");

    response = srchclient.Search<Hotel>("hotels", options);
    WriteDocuments(response);
    ```

1. Третий запрос демонстрирует свойство searchFields, используемое для ограничения операции полнотекстового поиска по конкретным полям.

    ```csharp
    Console.WriteLine("Query #3: Limit search to specific fields (pool in Tags field)...\n");

    options = new SearchOptions()
    {
        SearchFields = { "Tags" }
    };

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Tags");

    response = srchclient.Search<Hotel>("pool", options);
    WriteDocuments(response);
    ```

1. Четвертый запрос демонстрирует аспекты, которые можно использовать для составления структуры фасетной навигации. 

   ```csharp
    Console.WriteLine("Query #4: Facet on 'Category'...\n");

    options = new SearchOptions()
    {
        Filter = ""
    };

    options.Facets.Add("Category");

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Category");

    response = srchclient.Search<Hotel>("*", options);
    WriteDocuments(response);
   ```

1. В пятом запросе возвращается конкретный документ. Поиск документа — это типичный ответ на событие OnClick в результирующем наборе.

   ```csharp
    Console.WriteLine("Query #5: Look up a specific document...\n");

    Response<Hotel> lookupResponse;
    lookupResponse = srchclient.GetDocument<Hotel>("3");

    Console.WriteLine(lookupResponse.Value.HotelId);
   ```

1. Последний запрос представляет синтаксис средства автозавершения, моделирующего частичный ввод пользователем слова на "sa", при котором в свойстве sourceFields, связанном со средством подбора, определенным в индексе, распознается два возможных совпадения.

   ```csharp
    Console.WriteLine("Query #6: Call Autocomplete on HotelName that starts with 'sa'...\n");

    var autoresponse = srchclient.Autocomplete("sa", "sg");
    WriteDocuments(autoresponse);
   ```

1. Добавьте **RunQueries** в Main().

    ```csharp
    // Call the RunQueries method to invoke a series of queries
    Console.WriteLine("Starting queries...\n");
    RunQueries(srchclient);

    // End the program
    Console.WriteLine("{0}", "Complete. Press any key to end this program...\n");
    Console.ReadKey();
    ```

В предыдущих запросах показано несколько [способов сопоставления условий в запросе](search-query-overview.md#types-of-queries): полнотекстовый поиск, фильтры и автозавершение.

Полнотекстовый поиск и фильтрация выполняются с помощью метода [SearchClient.Search](/dotnet/api/azure.search.documents.searchclient.search). Строку поиска можно передать в параметре `searchText`, а выражение фильтра — свойстве [Filter](/dotnet/api/azure.search.documents.searchoptions.filter) класса [SearchOptions](/dotnet/api/azure.search.documents.searchoptions). Чтобы выполнить фильтрацию без поиска, передайте `"*"` в качестве значения параметра `searchText` в метод [Search](/dotnet/api/azure.search.documents.searchclient.search). Чтобы выполнить поиск без фильтрации, не задавайте значение свойству `Filter` или вообще не передавайте экземпляр `SearchOptions`.

## <a name="run-the-program"></a>Запуск программы

Нажмите клавишу F5, чтобы перестроить приложение и запустить полнофункциональную программу. 

Выходные данные содержат сообщения из [Console.WriteLine](/dotnet/api/system.console.writeline), а также сведения о запросе и результаты.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, по окончании проекта рекомендуем решить, нужны ли вам созданные ресурсы. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок **Все ресурсы** или **Группы ресурсов** на панели навигации слева.

При работе с бесплатной версией службы помните о том, что вам доступно максимум три индекса, индексатора и источника данных. Вы можете удалить отдельные элементы на портале, чтобы не превысить лимит. 

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве для C# вы отработали задачи по созданию индекса, загрузке документов в него и выполнению запросов. Мы несколько упростили решение, чтобы код было проще читать и понимать. Если основные понятия вы усвоили хорошо, мы рекомендуем перейти к следующей статье для изучения альтернативных подходов и основных понятий, которые дополнят и расширят ваши знания. 

> [!div class="nextstepaction"]
> [Разработка в .NET](search-howto-dotnet-sdk.md)

Хотите оптимизировать и сократить ваши расходы на облако?

> [!div class="nextstepaction"]
> [Начните анализировать затраты с помощью службы "Управление затратами"](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)
