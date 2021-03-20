---
title: Использование Azure.Search.Docументс (версии 11) в .NET
titleSuffix: Azure Cognitive Search
description: Узнайте, как создавать объекты поиска и управлять ими в приложении .NET с помощью C# и клиентской библиотеки Azure.Search.Docументс (версии 11). Фрагменты кода демонстрируют подключение к службе, создание индексов и запросов.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.devlang: dotnet
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/27/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 15a878eb863b71a4519e75def2598f013152dfb7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97881639"
---
# <a name="how-to-use-azuresearchdocuments-in-a-c-net-application"></a>Использование Azure.Search.Docументс в приложении .NET на C#

В этой статье объясняется, как создавать объекты поиска и управлять ими с помощью C# и клиентской библиотеки [**Azure.Search.Docументс**](/dotnet/api/overview/azure/search) (версия 11).

## <a name="about-version-11"></a>О версии 11

Пакет Azure SDK для .NET добавляет новую клиентскую библиотеку [**Azure.Search.Docументс**](/dotnet/api/overview/azure/search) из группы Azure SDK, которая функционально эквивалентна [Microsoft. Azure. Search](/dotnet/api/overview/azure/search/client10) Client Libraries, но использует общие подходы и соглашения там, где это применимо. Некоторые примеры включают [`AzureKeyCredential`](/dotnet/api/azure.azurekeycredential) проверку подлинности и [System.Text.Js. Сериализация](/dotnet/api/system.text.json.serialization) для СЕРИАЛИЗАЦИИ JSON.

Как и в предыдущих версиях, эту библиотеку можно использовать в следующих случаях:

+ Создание индексов поиска, источников данных, индексаторов, навыков и сопоставлений синонимов и управление ими
+ Загрузка документов поиска в индекс и управление ими
+ Выполнение запросов без необходимости работы с подробными сведениями о HTTP и JSON

Библиотека распространяется как единое [Azure.Search.Docпакет NuGet ументс](https://www.nuget.org/packages/Azure.Search.Documents/), который включает все интерфейсы API, используемые для программного доступа к службе поиска.

Клиентская библиотека определяет такие классы, как `SearchIndex`, `SearchField` и `SearchDocument`, а также такие операции, как `SearchIndexClient.CreateIndex` и `SearchClient.Search` в классах `SearchIndexClient` и `SearchClient`. Эти классы упорядочены в следующие подразделы:

+ [`Azure.Search.Documents`](/dotnet/api/azure.search.documents)
+ [`Azure.Search.Documents.Indexes`](/dotnet/api/azure.search.documents.indexes)
+ [`Azure.Search.Documents.Indexes.Models`](/dotnet/api/azure.search.documents.indexes.models)
+ [`Azure.Search.Documents.Models`](/dotnet/api/azure.search.documents.models)

Azure.Search.Docументс (версия 11) Целевая версия [ `2020-06-30` REST API когнитивный Поиск Azure](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/search/data-plane/Azure.Search/preview/2020-06-30). 

Клиентская библиотека не предоставляет [операций управления службами](/rest/api/searchmanagement/), таких как создание и масштабирование служб поиска и управление ключами API. Если вам нужно управлять ресурсами поиска из приложения .NET, используйте библиотеку [Microsoft. Azure. Management. Search](/dotnet/api/overview/azure/search/management) в пакете Azure SDK для .NET.

## <a name="upgrade-to-v11"></a>Обновление до версии 11

Если вы используете предыдущую версию пакета SDK для .NET и хотите обновить версию до текущей общедоступной версии, см. статью [обновление до Azure когнитивный Поиск .NET SDK версии 11](search-dotnet-sdk-migration-version-11.md) .

## <a name="sdk-requirements"></a>Требования пакета SDK

+ Visual Studio 2019 или более поздней версии.

+ Ваша собственная служба Когнитивный поиск Azure. Чтобы использовать пакет SDK, необходимо указать имя службы и один или несколько ключей API. [Создайте службу на портале](search-create-service-portal.md) , если она отсутствует.

+ Скачайте [пакетAzure.Search.Docументс](https://www.nuget.org/packages/Azure.Search.Documents) с помощью **средств**  >  **Диспетчер пакетов NuGet** Управление пакетами  >  **NuGet для решения** в Visual Studio. Найдите имя пакета `Azure.Search.Documents` .

Пакет SDK Azure для .NET соответствует [.NET Standard 2.0](/dotnet/standard/net-standard#net-implementation-support), что означает .NET Framework 4.6.1 и .NET Core 2.0 соответствуют минимальным требованиям.

## <a name="example-application"></a>Пример приложения

В этой статье "учебные материалы по примеру", полагается на пример кода [дотнесовто](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo) на сайте GitHub, чтобы продемонстрировать фундаментальные понятия в Azure когнитивный Поиск — в частности, как создать, загрузить и запросить индекс поиска.

В оставшейся части этой статьи Предположим, что новый индекс под названием «Гостиницы» заполнен несколькими документами, с несколькими запросами, которые соответствуют результатам.

Ниже приведена основная программа, показывающая общую последовательность:

```csharp
// This sample shows how to delete, create, upload documents and query an index
static void Main(string[] args)
{
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();

    SearchIndexClient indexClient = CreateSearchIndexClient(configuration);

    string indexName = configuration["SearchIndexName"];

    Console.WriteLine("{0}", "Deleting index...\n");
    DeleteIndexIfExists(indexName, indexClient);

    Console.WriteLine("{0}", "Creating index...\n");
    CreateIndex(indexName, indexClient);

    SearchClient searchClient = indexClient.GetSearchClient(indexName);

    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(searchClient);

    SearchClient indexClientForQueries = CreateSearchClientForQueries(indexName, configuration);

    Console.WriteLine("{0}", "Run queries...\n");
    RunQueries(indexClientForQueries);

    Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
    Console.ReadKey();
}
```

Далее следует частичный снимок экрана выходных данных, при условии, что вы запустили это приложение с допустимым именем службы и ключами API:

:::image type="content" source="media/search-howto-dotnet-sdk/console-output.png" alt-text="Выходные данные Console. WriteLine из примера программы":::

### <a name="client-types"></a>Типы клиентов

Клиентская библиотека использует три типа клиентов для различных операций: [`SearchIndexClient`](/dotnet/api/azure.search.documents.indexes.searchindexclient) для создания, обновления и удаления индексов, [`SearchClient`](/dotnet/api/azure.search.documents.searchclient) загрузки или запроса индекса, а [`SearchIndexerClient`](/dotnet/api/azure.search.documents.indexes.searchindexerclient) также для работы с индексаторами и навыков. В этой статье рассматриваются первые две. 

Как минимум, всем клиентам требуется имя службы или конечная точка, а также ключ API. Обычно эти сведения предоставляются в файле конфигурации аналогично тому, что вы найдете в `appsettings.json` файле [примера приложения дотнесовто](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo). Чтобы выполнить чтение из файла конфигурации, добавьте `using Microsoft.Extensions.Configuration;` в программу.

Следующая инструкция создает клиент индекса, используемый для создания, обновления или удаления индексов. Он принимает конечную точку поиска и ключ API администратора.

```csharp
private static SearchIndexClient CreateSearchIndexClient(IConfigurationRoot configuration)
{
    string searchServiceEndPoint = configuration["SearchServiceEndPoint"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];

    SearchIndexClient indexClient = new SearchIndexClient(new Uri(searchServiceEndPoint), new AzureKeyCredential(adminApiKey));
    return indexClient;
}
```

Следующий оператор создает клиент поиска, используемый для загрузки документов или выполнения запросов. `SearchClient` требуется индекс. Для загрузки документов потребуется ключ API администратора, но для выполнения запросов можно использовать ключ API запроса. 

```csharp
string indexName = configuration["SearchIndexName"];

private static SearchClient CreateSearchClientForQueries(string indexName, IConfigurationRoot configuration)
{
    string searchServiceEndPoint = configuration["SearchServiceEndPoint"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchClient searchClient = new SearchClient(new Uri(searchServiceEndPoint), indexName, new AzureKeyCredential(queryApiKey));
    return searchClient;
}
```

> [!NOTE]
> Если вы представили недопустимый ключ для операции импорта (например, ключ запроса, в котором требовался ключ администратора), при `SearchClient` `CloudException` первом вызове метода операции в будет выдано сообщение об ошибке "запрещено". Если это произойдет, дважды проверьте ключ API.
>

### <a name="deleting-the-index"></a>Удаление индекса

На ранних стадиях разработки может потребоваться включить [`DeleteIndex`](/dotnet/api/azure.search.documents.indexes.searchindexclient.deleteindex) инструкцию для удаления индекса выполнения, чтобы его можно было повторно создать с обновленным определением. Пример кода для Azure Когнитивный поиск часто включает шаг удаления, чтобы можно было повторно запустить пример.

Следующие вызовы строк `DeleteIndexIfExists` :

```csharp
Console.WriteLine("{0}", "Deleting index...\n");
DeleteIndexIfExists(indexName, indexClient);
```

Этот метод использует заданный объект `SearchIndexClient` для проверки существования индекса и, если да, удаляет его:

```csharp
private static void DeleteIndexIfExists(string indexName, SearchIndexClient indexClient)
{
    try
    {
        if (indexClient.GetIndex(indexName) != null)
        {
            indexClient.DeleteIndex(indexName);
        }
    }
    catch (RequestFailedException e) when (e.Status == 404)
    {
        // Throw an exception if the index name isn't found
        Console.WriteLine("The index doesn't exist. No deletion occurred.");
```

> [!NOTE]
> В примере кода в этой статье используются синхронные методы для простоты, но в собственных приложениях следует использовать асинхронные методы, чтобы они оставались масштабируемыми и быстрыми. Например, в приведенном выше методе можно использовать [`DeleteIndexAsync`](/dotnet/api/azure.search.documents.indexes.searchindexclient.deleteindexasync) вместо [`DeleteIndex`](/dotnet/api/azure.search.documents.indexes.searchindexclient.deleteindex) .
>

## <a name="create-an-index"></a>Создание индекса

[`SearchIndexClient`](/dotnet/api/azure.search.documents.indexes.searchindexclient)Для создания индекса можно использовать. 

Приведенный ниже метод создает новый [`SearchIndex`](/dotnet/api/azure.search.documents.indexes.models.searchindex) объект со списком [`SearchField`](/dotnet/api/azure.search.documents.indexes.models.searchfield) объектов, определяющих схему нового индекса. Каждое поле имеет имя, тип данных и несколько атрибутов, которые определяют его поведение при поиске. 

Поля можно определить из класса модели с помощью [`FieldBuilder`](/dotnet/api/azure.search.documents.indexes.fieldbuilder) . Класс `FieldBuilder` использует отражение, чтобы создать список объектов `SearchField` для соответствующего индекса. Он проверяет общедоступные свойства и атрибуты полученного класса модели `Hotel`. Класс `Hotel` мы рассмотрим подробно позже.

```csharp
private static void CreateIndex(string indexName, SearchIndexClient indexClient)
{
    FieldBuilder fieldBuilder = new FieldBuilder();
    var searchFields = fieldBuilder.Build(typeof(Hotel));

    var definition = new SearchIndex(indexName, searchFields);

    indexClient.CreateOrUpdateIndex(definition);
}
```

Помимо полей можно также добавить в индекс профили оценки, средства подбора или параметры CORS (эти параметры опущены в образце для краткости). Дополнительные сведения об объекте Сеарчиндекс и его составных частях [`SearchIndex`](/dotnet/api/azure.search.documents.indexes.models.searchindex) см. в списке свойств, а также в [справочнике по REST API](/rest/api/searchservice/).

> [!NOTE]
> При необходимости вы всегда можете создать список объектов `Field` напрямую, а не с помощью `FieldBuilder`. Это полезно, например, если вы не хотите использовать класс модели или используете существующий класс модели, в который не хотите добавлять атрибуты.
>

### <a name="call-createindex-in-main"></a>Вызов CreateIndex в Main ()

`Main` создает новый индекс "отели", вызывая приведенный выше метод:

```csharp
Console.WriteLine("{0}", "Creating index...\n");
CreateIndex(indexName, indexClient);
```

## <a name="use-a-model-class-for-data-representation"></a>Использование класса модели для представления данных

В примере Дотнесовто используются классы моделей для структур данных [отеля](https://github.com/Azure-Samples/search-dotnet-getting-started/blob/master/DotNetHowTo/DotNetHowTo/Hotel.cs), [Address](https://github.com/Azure-Samples/search-dotnet-getting-started/blob/master/DotNetHowTo/DotNetHowTo/Address.cs)и [Room](https://github.com/Azure-Samples/search-dotnet-getting-started/blob/master/DotNetHowTo/DotNetHowTo/Room.cs) . `Hotel` ссылки `Address` , сложный тип с одним уровнем (многокомпонентное поле) и `Room` (коллекция полей с несколькими частями).

Эти типы можно использовать для создания и загрузки индекса, а также для структурирования ответа из запроса:

```csharp
// Use-case: <Hotel> in a field definition
FieldBuilder fieldBuilder = new FieldBuilder();
var searchFields = fieldBuilder.Build(typeof(Hotel));

// Use-case: <Hotel> in a response
private static void WriteDocuments(SearchResults<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.GetResults())
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

Альтернативным подходом является добавление полей непосредственно в индекс. В следующем примере показано лишь несколько полей.

   ```csharp
    SearchIndex index = new SearchIndex(indexName)
    {
        Fields =
            {
                new SimpleField("hotelId", SearchFieldDataType.String) { IsKey = true, IsFilterable = true, IsSortable = true },
                new SearchableField("hotelName") { IsFilterable = true, IsSortable = true },
                new SearchableField("hotelCategory") { IsFilterable = true, IsSortable = true },
                new SimpleField("baseRate", SearchFieldDataType.Int32) { IsFilterable = true, IsSortable = true },
                new SimpleField("lastRenovationDate", SearchFieldDataType.DateTimeOffset) { IsFilterable = true, IsSortable = true }
            }
    };
   ```

### <a name="field-definitions"></a>Определения полей

Ваша модель данных в .NET и соответствующая схема индексов должны поддерживать возможности поиска, которые вы хотите предоставить конечному пользователю. Каждый объект верхнего уровня в .NET, например документ поиска в индексе поиска, соответствует результату поиска, который будет представлен в пользовательском интерфейсе. Например, в приложении поиска отеля конечные пользователи могут захотеть искать по имени Гостиницы, функциям отеля или характеристикам определенной комнаты. 

В каждом классе поле определяется с помощью типа данных и атрибутов, определяющих его использование. Имя каждого общего свойства в каждом классе сопоставляется с полем с тем же именем в определении индекса. 

Рассмотрим следующий фрагмент кода, который извлекает несколько определений полей из класса гостиницы. Обратите внимание, что адрес и комнаты являются типами C# с собственными определениями классов (см. пример кода, если вы хотите их просмотреть). Оба типа являются сложными типами. Дополнительные сведения см. [в разделе как моделировать сложные типы](search-howto-complex-data-types.md).

```csharp
public partial class Hotel
{
    [SimpleField(IsKey = true, IsFilterable = true)]
    public string HotelId { get; set; }

    [SearchableField(IsSortable = true)]
    public string HotelName { get; set; }

    [SearchableField(AnalyzerName = LexicalAnalyzerName.Values.EnLucene)]
    public string Description { get; set; }

    [SearchableField(IsFilterable = true, IsSortable = true, IsFacetable = true)]
    public string Category { get; set; }

    [JsonIgnore]
    public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;

    [SearchableField]
    public Address Address { get; set; }

    public Room[] Rooms { get; set; }
```

#### <a name="choosing-a-field-class"></a>Выбор класса поля

При определении полей можно использовать базовый [`SearchField`](/dotnet/api/azure.search.documents.indexes.models.searchfield) класс или можно использовать производные вспомогательные модели, которые служат шаблонами с предварительно настроенными свойствами.

Только одно поле в индексе должно служить ключом документа ( `IsKey = true` ). Он должен быть строкой и должен уникальным образом идентифицировать каждый документ. Он также должен иметь, что `IsHidden = true` означает, что он не может быть видимым в результатах поиска.

| Тип поля | Описание и использование |
|------------|-----------------------|
| [`SearchField`](/dotnet/api/azure.search.documents.indexes.models.searchfield) | Базовый класс, для большинства свойств которого задано значение null, за исключением того, `Name` что требуется, и `AnalyzerName` значение по умолчанию — Standard Lucene. |
| [`SimpleField`](/dotnet/api/azure.search.documents.indexes.models.simplefield) | Вспомогательная модель. Может быть любым типом данных, всегда является недоступным для поиска (он игнорируется для запросов полнотекстового поиска) и может быть извлечен (он не скрыт). Другие атрибуты отключены по умолчанию, но их можно включить. `SimpleField` можно использовать для идентификаторов документов или полей, используемых только в фильтрах, аспектах или профилях оценки. Если это так, обязательно примените все необходимые для сценария атрибуты, например `IsKey = true` для идентификатора документа. Чтобы узнать больше, см. [SimpleFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SimpleFieldAttribute.cs) в исходном коде. |
| [`SearchableField`](/dotnet/api/azure.search.documents.indexes.models.searchablefield) | Вспомогательная модель. Значение должно быть строкой и всегда доступно для поиска и получения. Другие атрибуты отключены по умолчанию, но их можно включить. Так как этот тип поля доступен для поиска, он поддерживает синонимы и полное дополнение свойств анализатора. Чтобы узнать больше, см. [SearchableFieldAttribute.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/search/Azure.Search.Documents/src/Indexes/SearchableFieldAttribute.cs) в исходном коде. |

Независимо от того, используется ли базовый API `SearchField` или одна из вспомогательных моделей, необходимо явно включить атрибуты фильтров, аспектов и сортировки. Например, [IsFilterable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable), [IsSortable](/dotnet/api/azure.search.documents.indexes.models.searchfield.issortable) и [IsFacetable](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfacetable) должны присваиваться явным образом, как в примере выше.

#### <a name="adding-field-attributes"></a>Добавление атрибутов полей

Обратите внимание, что каждое поле снабжено атрибутами, такими как `IsFilterable` ,, `IsSortable` `IsKey` и `AnalyzerName` . Эти атрибуты сопоставляются непосредственно с [соответствующими атрибутами полей в индексе Azure когнитивный Поиск](/rest/api/searchservice/create-index). `FieldBuilder`Класс использует эти свойства для создания определений полей для индекса.

#### <a name="field-type-mapping"></a>Сопоставление типов полей

Типы .NET свойств сопоставляются с эквивалентными типами полей в определении индекса. Например, свойство строки `Category` сопоставляется с полем `category`, которое имеет тип `Edm.String`. Существуют аналогичные сопоставления типов между,, `bool?` `Edm.Boolean` `DateTimeOffset?` и `Edm.DateTimeOffset` т. д. 

Возможно, вы заметите `SmokingAllowed` свойство?

```csharp
[JsonIgnore]
public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;
```

`JsonIgnore`Атрибут в этом свойстве указывает, что `FieldBuilder` не следует сериализовать его в индекс в качестве поля.  Это отличный способ создания вычисляемых свойств на стороне клиента, которые можно использовать в качестве вспомогательных функций в приложении.  В этом случае `SmokingAllowed` свойство отражает, `Room` `Rooms` разрешено ли в коллекции Курение. Если все имеют значение false, это означает, что вся Гостиницы не допускает Курение.

## <a name="load-an-index"></a>Загрузка индекса.

Следующий шаг в `Main` заполняет только что созданный индекс гостиниц. Это заполнение индекса выполняется в следующем методе: (часть кода заменена на "..." для целей иллюстрации. См. полный пример решения для полного заполнения данных.)

```csharp
private static void UploadDocuments(SearchClient searchClient)
{
    IndexDocumentsBatch<Hotel> batch = IndexDocumentsBatch.Create(
        IndexDocumentsAction.Upload(
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
            }),
        IndexDocumentsAction.Upload(
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
            }),
        IndexDocumentsAction.Upload(
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

    try
    {
        IndexDocumentsResult result = searchClient.IndexDocuments(batch);
    }
    catch (Exception)
    {
        // Sometimes when your Search service is under load, indexing will fail for some of the documents in
        // the batch. Depending on your application, you can take compensating actions like delaying and
        // retrying. For this simple demo, we just log the failed document keys and continue.
        Console.WriteLine("Failed to index some of the documents: {0}");
    }

    Console.WriteLine("Waiting for documents to be indexed...\n");
    Thread.Sleep(2000);
```

Этот метод состоит из четырех частей. Первый создает массив из трех `Hotel` объектов, каждый из которых имеет 3 `Room` объекта, которые будут служить входными данными для отправки в индекс. Эти данные жестко запрограммированы для простоты. В реальных приложениях данные, скорее всего, поступают из внешнего источника данных, такого как база данных SQL.

Во второй части создается объект, [`IndexDocumentsBatch`](/dotnet/api/azure.search.documents.models.indexdocumentsbatch) содержащий документы. Вы указываете операцию, которую необходимо применить к пакету во время ее создания, в данном случае путем вызова [`IndexDocumentsAction.Upload`](/dotnet/api/azure.search.documents.models.indexdocumentsaction.upload) . Затем пакет отправляется в индекс Когнитивный поиск Azure по [`IndexDocuments`](/dotnet/api/azure.search.documents.searchclient.indexdocuments) методу.

> [!NOTE]
> В этом примере мы просто отправляем документы. Если требуется внести изменения в существующие документы или удалить документы, можно создать пакеты, вызвав методы `IndexDocumentsAction.Merge`, `IndexDocumentsAction.MergeOrUpload` или `IndexDocumentsAction.Delete`. Можно также смешивать различные операции в одном пакете, вызывая метод `IndexBatch.New` , который принимает коллекцию `IndexDocumentsAction` объектов, каждый из которых сообщает когнитивный Поиск Azure выполнить определенную операцию над документом. Можно создать каждый `IndexDocumentsAction` с отдельной операцией, вызвав соответствующий метод, например `IndexDocumentsAction.Merge`, `IndexAction.Upload` и т. д.
> 

Третья часть метода — это блок catch, который обрабатывает важные ошибки индексирования. Если службе поиска не удается индексировать некоторые документы в пакете, создается `IndexBatchException` исключение `IndexDocuments` . Это исключение может произойти при индексировании документов, пока служба находится в режиме высокой нагрузки. **Настоятельно рекомендуется явно обрабатывать этот случай в коде.**  Вы можете задержать и повторить попытку индексирования соответствующих документов либо занести ошибку в журнал и продолжить работу, как в нашем примере, а также выполнить другие действия в зависимости от требований вашего приложения к целостности данных.

И наконец, метод `UploadDocuments` вызывает задержку на две секунды. Индексирование выполняется асинхронно в службе поиска, поэтому пример приложения должен подождать некоторое время, чтобы обеспечить доступность документов для поиска. Такие задержки обычно необходимы только в демонстрациях, тестах и примерах приложений.

### <a name="call-uploaddocuments-in-main"></a>Вызов Уплоаддокументс в Main ()

Следующий фрагмент кода настраивает экземпляр [`SearchClient`](/dotnet/api/azure.search.documents.searchclient) с помощью [`GetSearchClient`](/dotnet/api/azure.search.documents.indexes.searchindexclient.getsearchclient) метода индексклиент. Индексклиент использует ключ API администратора для своих запросов, который необходим для загрузки или обновления документов.

Альтернативный подход — прямой вызов `SearchClient` , передающий ключ API администратора в `AzureKeyCredential` .

```csharp
SearchClient searchClient = indexClient.GetSearchClient(indexName);

Console.WriteLine("{0}", "Uploading documents...\n");
UploadDocuments(searchClient);
```

## <a name="run-queries"></a>Выполнение запросов

Сначала настройте объект `SearchClient` , считывающий конечную точку поиска и ключ API запроса из **appsettings.js**:

```csharp
private static SearchClient CreateSearchClientForQueries(string indexName, IConfigurationRoot configuration)
{
    string searchServiceEndPoint = configuration["SearchServiceEndPoint"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchClient searchClient = new SearchClient(new Uri(searchServiceEndPoint), indexName, new AzureKeyCredential(queryApiKey));
    return searchClient;
}
```

Во-вторых, определите метод, отправляющий запрос. 

Каждый раз, когда метод выполняет запрос, он создает новый [`SearchOptions`](/dotnet/api/azure.search.documents.searchoptions) объект. Этот объект используется для указания дополнительных параметров запроса, таких как сортировка, фильтрация, разбиение на страницы и аспекты. В этом методе устанавливается `Filter` `Select` свойство, и `OrderBy` для различных запросов. Дополнительные сведения о синтаксисе выражений запроса поиска см. в [синтаксисе простых запросов](/rest/api/searchservice/Simple-query-syntax-in-Azure-Search).

Следующий этап — фактическое выполнение поискового запроса. Выполнение поиска выполняется с помощью `SearchClient.Search` метода. Для каждого запроса передайте текст поиска для использования в качестве строки (или `"*"` , если текст для поиска отсутствует), а также параметры поиска, созданные ранее. Мы также указываем `Hotel` как параметр типа `SearchClient.Search`, в результате чего пакет SDK будет десериализовать документы в результатах поиска к объектам типа `Hotel`.

```csharp
private static void RunQueries(SearchClient searchClient)
{
    SearchOptions options;
    SearchResults<Hotel> results;

    Console.WriteLine("Query 1: Search for 'motel'. Return only the HotelName in results:\n");

    options = new SearchOptions();
    options.Select.Add("HotelName");

    results = searchClient.Search<Hotel>("motel", options);

    WriteDocuments(results);

    Console.Write("Query 2: Apply a filter to find hotels with rooms cheaper than $100 per night, ");
    Console.WriteLine("returning the HotelId and Description:\n");

    options = new SearchOptions()
    {
        Filter = "Rooms/any(r: r/BaseRate lt 100)"
    };
    options.Select.Add("HotelId");
    options.Select.Add("Description");

    results = searchClient.Search<Hotel>("*", options);

    WriteDocuments(results);

    Console.Write("Query 3: Search the entire index, order by a specific field (lastRenovationDate) ");
    Console.Write("in descending order, take the top two results, and show only hotelName and ");
    Console.WriteLine("lastRenovationDate:\n");

    options =
        new SearchOptions()
        {
            Size = 2
        };
    options.OrderBy.Add("LastRenovationDate desc");
    options.Select.Add("HotelName");
    options.Select.Add("LastRenovationDate");

    results = searchClient.Search<Hotel>("*", options);

    WriteDocuments(results);

    Console.WriteLine("Query 4: Search the HotelName field for the term 'hotel':\n");

    options = new SearchOptions();
    options.SearchFields.Add("HotelName");

    //Adding details to select, because "Location" is not supported yet when deserialize search result to "Hotel"
    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Description");
    options.Select.Add("Category");
    options.Select.Add("Tags");
    options.Select.Add("ParkingIncluded");
    options.Select.Add("LastRenovationDate");
    options.Select.Add("Rating");
    options.Select.Add("Address");
    options.Select.Add("Rooms");

    results = searchClient.Search<Hotel>("hotel", options);

    WriteDocuments(results);
}
```

В-третьих, определите метод, записывающий ответ, напечатая каждый документ на консоль:

```csharp
private static void WriteDocuments(SearchResults<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.GetResults())
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

### <a name="call-runqueries-in-main"></a>Вызов Рункуериес в Main ()

```csharp
SearchClient indexClientForQueries = CreateSearchClientForQueries(indexName, configuration);

Console.WriteLine("{0}", "Running queries...\n");
RunQueries(indexClientForQueries);
```

### <a name="explore-query-constructs"></a>Изучение конструкций запросов

Давайте подробнее рассмотрим каждый из запросов. Ниже приведен код для выполнения первого запроса:

```csharp
options = new SearchOptions();
options.Select.Add("HotelName");

results = searchClient.Search<Hotel>("motel", options);

WriteDocuments(results);
```

В этом случае мы ищем по всему индексу слово «Motel» в любом поле с возможностью поиска, и мы хотим получить только названия гостиниц, как указано в `Select` параметре. Результаты приведены ниже.

```output
Name: Secret Point Motel

Name: Twin Dome Motel
```

Во втором запросе используйте фильтр, чтобы выбрать комнату с ночной частотой менее $100. Возвращает только идентификатор отеля и описание в результатах:

```csharp
options = new SearchOptions()
{
    Filter = "Rooms/any(r: r/BaseRate lt 100)"
};
options.Select.Add("HotelId");
options.Select.Add("Description");

results = searchClient.Search<Hotel>("*", options);
```

Приведенный выше запрос использует `$filter` выражение OData `Rooms/any(r: r/BaseRate lt 100)` для фильтрации документов в индексе. При этом используется [оператор Any](./search-query-odata-collection-operators.md) для применения "басерате LT 100" к каждому элементу в коллекции помещений. Дополнительные сведения см. в разделе [синтаксис фильтра OData](./query-odata-filter-orderby-syntax.md).

В третьем запросе найдите два основных Гостиницы, которые были недавно отремонтированных, и покажите название гостиницы и дату последнего обновления. Ниже приведен код: 

```csharp
options =
    new SearchOptions()
    {
        Size = 2
    };
options.OrderBy.Add("LastRenovationDate desc");
options.Select.Add("HotelName");
options.Select.Add("LastRenovationDate");

results = searchClient.Search<Hotel>("*", options);

WriteDocuments(results);
```

В последнем запросе найдите все названия гостиниц, соответствующие слову «Гостиница»:

```csharp
options.Select.Add("HotelId");
options.Select.Add("HotelName");
options.Select.Add("Description");
options.Select.Add("Category");
options.Select.Add("Tags");
options.Select.Add("ParkingIncluded");
options.Select.Add("LastRenovationDate");
options.Select.Add("Rating");
options.Select.Add("Address");
options.Select.Add("Rooms");

results = searchClient.Search<Hotel>("hotel", options);

WriteDocuments(results);
```

В этом разделе приводится заключение в пакет SDK для .NET, но здесь не стоит останавливаться. В следующем разделе предлагаются дополнительные ресурсы для изучения программирования с помощью Azure Когнитивный поиск.

## <a name="next-steps"></a>Дальнейшие действия

+ Просмотрите справочную документацию по API для [Azure.Search.Docументс](/dotnet/api/azure.search.documents) и [REST API](/rest/api/searchservice/)

+ Просмотрите другие примеры кода на основе Azure.Search.Docументс в [Azure-Search-DotNet-Samples](https://github.com/Azure-Samples/azure-search-dotnet-samples) and [Search-Приступая к работе — DotNet](https://github.com/Azure-Samples/search-dotnet-getting-started)

+ Ознакомьтесь с [соглашениями об именовании](/rest/api/searchservice/Naming-rules) , чтобы узнать правила именования различных объектов.

+ Проверка [поддерживаемых типов данных](/rest/api/searchservice/Supported-data-types)
