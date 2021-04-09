---
title: Руководство по C#. Индексирование нескольких источников данных Azure
titleSuffix: Azure Cognitive Search
description: Узнайте, как импортировать данные из нескольких источников данных в один индекс службы "Когнитивный поиск Azure" с помощью индексаторов. В этом учебнике и примере кода используется язык C#.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 01/23/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: df7dcf9859b6942662ae447af836f59985e2d11a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99509508"
---
# <a name="tutorial-index-from-multiple-data-sources-using-the-net-sdk"></a>Руководство по Индексирование из нескольких источников данных с помощью пакета SDK для .NET

Когнитивный поиск Azure позволяет импортировать, анализировать и индексировать данные из нескольких источников данных в единый объединенный поисковой индекс. 

При прохождении этого учебника для индексации примеров данных отеля из Azure Cosmos DB и их объединения с данными о гостиничных номерах из документов Хранилища BLOB-объектов Azure в пакете Azure SDK для .NET используется C# и клиентская библиотека [Azure.Search.Documents](/dotnet/api/overview/azure/search). В результате будет сформирован комбинированный индекс поиска отелей, содержащий документы об отелях, в котором гостиничные номера представляют из себя сложные типы данных.

Вот какие шаги выполняются в этом учебнике:

> [!div class="checklist"]
> * отправка примера данных и создание источников данных;
> * Определение ключа документа
> * определение и создание индекса;
> * индексирование данных об отелях из Azure Cosmos DB;
> * объединение данных о номерах отелей из хранилища BLOB-объектов.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="overview"></a>Обзор

В этом руководстве для создания и запуска нескольких индексаторов используется новая клиентская библиотека, [Azure.Search.Documents](/dotnet/api/overview/azure/search), версии 11.x. При прохождении этого учебника вы настроите два источника данных Azure таким образом, чтобы создать индексатор, который будет извлекать данные из обоих источников данных для заполнения одного индекса поиска. Для поддержки слияния оба набора данных должны иметь общее значение. В этом примере таким полем является идентификатор. Пока есть поле для поддержки сопоставления, индексатор может объединять данные из разрозненных ресурсов: структурированные данные из Azure SQL, неструктурированные данные из Хранилища BLOB-объектов или любую комбинацию [поддерживаемых источников данных](search-indexer-overview.md#supported-data-sources) в Azure.

Готовую версию кода для этого руководства можно найти в следующем проекте:

* [multiple-data-sources/v11 (GitHub)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources/v11)

Это руководство было обновлено для использования пакета Azure.Search.Documents (версия 11). Если у вас более ранняя версия пакета SDK для .NET, просмотрите [пример кода Microsoft.Azure.Search (версия 10)](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources/v10) на GitHub.

## <a name="prerequisites"></a>Предварительные требования

+ [Azure Cosmos DB](../cosmos-db/create-cosmosdb-resources-portal.md)
+ [Хранилище Azure](../storage/common/storage-account-create.md)
+ [Visual Studio](https://visualstudio.microsoft.com/)
+ [Пакет NuGet для Когнитивного поиска (версия 11.x)](https://www.nuget.org/packages/Azure.Search.Documents/)
+ [Создайте службу поиска](search-create-service-portal.md) или [найдите существующую службу](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Для выполнения инструкций из этого руководства вы можете использовать бесплатную версию службы. В бесплатной версии вы можете использовать не более трех индексов, трех индексаторов и трех источников данных. В этом руководстве создается по одному объекту из каждой категории. Перед началом работы убедитесь, что у службы есть достаточно места, чтобы принять новые ресурсы.

## <a name="1---create-services"></a>1\. Создание служб

В этом учебнике используется служба "Когнитивный поиск Azure" для индексирования и запросов, Azure Cosmos DB — для первого набора данных и хранилище BLOB-объектов Azure — для второго. 

Желательно создать все службы в одном регионе и в одной группе ресурсов, чтобы упростить взаимодействие и управление. На практике службы могут находиться в любом регионе.

В этом примере используются два небольших набора данных, которые описывают семь вымышленных отелей. Один набор, который описывает сами отели, мы загрузим в базу данных Azure Cosmos DB. Другой набор с подробными сведениями о номерах в этих отелях предоставляется в виде семи отдельных файлов формата JSON, и его мы отправим в хранилище BLOB-объектов Azure.

### <a name="start-with-cosmos-db"></a>Начало работы с Cosmos DB

1. Войдите на [портал Azure](https://portal.azure.com) и перейдите к странице обзора для учетной записи Azure Cosmos DB.

1. Выберите раздел **Обозреватель данных**, а затем — команду **Новая база данных**.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-newdb.png" alt-text="Создание базы данных" border="false":::

1. Введите имя **hotel-rooms-db**. Примите значения по умолчанию для остальных параметров.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-dbname.png" alt-text="Настройка базы данных" border="false":::

1. Создайте новый контейнер. Используйте имеющуюся базу данных, которую вы только что создали. В качестве имени контейнера введите **hotels**, а имени ключа секции — **/HotelId**.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-add-container.png" alt-text="Добавление контейнера" border="false":::

1. В разделе **hotels** выберите **Элементы**, а затем на панели команд щелкните **Upload Item** (Отправить элемент). Перейдите к файлу **cosmosdb/HotelsDataSubset_CosmosDb.json** в папке проекта и выберите его.

   :::image type="content" source="media/tutorial-multiple-data-sources/cosmos-upload.png" alt-text="Отправка в коллекцию Azure Cosmos DB" border="false":::

1. С помощью кнопки "Обновить" освежите представление элементов в коллекции hotels. Вы увидите в списке семь новых документов базы данных.

1. Скопируйте строку подключения со страницы **Ключи** в блокнот. Она понадобится для файла **appsettings.json** при выполнении шагов позже. Если вы не используете предложенное название для базы данных hotel-rooms-db, тогда скопируйте также название базы данных.

### <a name="azure-blob-storage"></a>Хранилище BLOB-объектов Azure

1. Войдите на [портал Azure](https://portal.azure.com), перейдите к учетной записи хранения Azure, щелкните **Большие двоичные объекты**, а затем — **+ Container** (+ Контейнер).

1. [Создайте контейнер больших двоичных объектов](../storage/blobs/storage-quickstart-blobs-portal.md) с именем **hotel-rooms**, в котором будут храниться JSON-файлы с примерами данных о номерах отелей. Можно задать любое из допустимых значений уровня общего доступа.

   :::image type="content" source="media/tutorial-multiple-data-sources/blob-add-container.png" alt-text="Создание контейнера больших двоичных объектов" border="false":::

1. Откройте контейнер после создания и на панели команд выберите **Загрузить**. Перейдите к папке, содержащей примеры файлов. Выберите их, а затем щелкните **Загрузить**.

   :::image type="content" source="media/tutorial-multiple-data-sources/blob-upload.png" alt-text="Передача файлов" border="false":::

1. Скопируйте название учетной записи хранения и строку подключения со страницы **Ключи доступа** в блокнот. Позднее вам понадобятся оба эти значения для файла **appsettings.json**.

### <a name="azure-cognitive-search"></a>Когнитивный поиск Azure

Третий компонент — Когнитивный поиск Azure, который вы можете [создать на портале](search-create-service-portal.md). 

### <a name="copy-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Копирование ключа API и URL-адреса конечной точки для администрирования Когнитивного поиска Azure

Чтобы пройти проверку подлинности для службы поиска, понадобится URL-адрес и ключ доступа службы.

1. [Войдите на портал Azure](https://portal.azure.com/) и на странице **обзора** службы поиска получите URL-адрес. Пример конечной точки может выглядеть так: `https://mydemo.search.windows.net`.

1. В разделе **Параметры** > **Ключи** получите ключ администратора, чтобы обрести полные права на службу. Существуют два взаимозаменяемых ключа администратора, предназначенных для обеспечения непрерывности бизнес-процессов на случай, если вам потребуется сменить один из них. Вы можете использовать первичный или вторичный ключ для выполнения запросов на добавление, изменение и удаление объектов.

   :::image type="content" source="media/search-get-started-javascript/service-name-and-keys.png" alt-text="Получение имени службы, ключей запросов и администратора" border="false":::

Если есть действительный ключ, для каждого запроса устанавливаются отношения доверия между приложением, которое отправляет запрос, и службой, которая его обрабатывает.

## <a name="2---set-up-your-environment"></a>2\. Настройка среды

1. Запустите Visual Studio. В меню **Средства** выберите **Диспетчер пакетов NuGet**, а затем — **Управление пакетами NuGet для решения...** . 

1. Во вкладке **Обзор** найдите и установите **Azure.Search.Documents** (версия 11.0 или более поздняя). Для завершения установки нужно пройти серию дополнительных диалоговых окон.

    :::image type="content" source="media/tutorial-csharp-create-first-app/azure-search-nuget-azure.png" alt-text="Добавление библиотек Azure с помощью NuGet" border="false":::

1. Найдите и установите пакеты NuGet **Microsoft.Extensions.Configuration** и **Microsoft.Extensions.Configuration.Json**.

1. Откройте файл решения **/v11/AzureSearchMultipleDataSources.sln**.

1. В обозревателе решений измените файл **appsettings.json**, добавив сведения о подключении.  

    ```json
    {
      "SearchServiceUri": "<YourSearchServiceURL>",
      "SearchServiceAdminApiKey": "<YourSearchServiceAdminApiKey>",
      "BlobStorageAccountName": "<YourBlobStorageAccountName>",
      "BlobStorageConnectionString": "<YourBlobStorageConnectionString>",
      "CosmosDBConnectionString": "<YourCosmosDBConnectionString>",
      "CosmosDBDatabaseName": "hotel-rooms-db"
    }
    ```

Первые две записи содержат URL-адрес и ключи администратора службы поиска. Используйте полную конечную точку, например: `https://mydemo.search.windows.net`.

Следующие записи содержат имена учетных записей и строки подключения для хранилища BLOB-объектов Azure и источников данных Azure Cosmos DB.

## <a name="3---map-key-fields"></a>3\. Сопоставление полей ключей

Для объединения содержимого требуется, чтобы оба потока данных использовали одни и те же документы в индексе поиска. 

В службе "Когнитивный поиск Azure" поле ключа однозначно определяет каждый документ. Каждый индекс поиска должен содержать только одно поле ключа типа `Edm.String`. Это поле ключа должно присутствовать в источнике данных для каждого документа, который добавляется к индексу. (Фактически, это единственное обязательное для заполнения поле.)

При индексировании данных из нескольких источников данных убедитесь, что каждая входная строка или входной документ содержит общий ключ документа для объединения данных из двух физически различных исходных документов в новый документ поиска в комбинированном индексе. 

Зачастую приходится потратить некоторое время, чтобы заранее составить разумный ключ документа для создаваемого индекса и убедиться, что он существует в обоих источниках данных. В этом демонстрационном примере ключ `HotelId` для каждого отеля в Cosmos DB также присутствует в больших двоичных объектах номеров в формате JSON в хранилище BLOB-объектов.

Индексаторы службы "Когнитивный поиск Azure" умеют использовать сопоставления полей для изменения имен и форматов полей данных в процессе индексирования, чтобы правильно сопоставить данные из источника с нужным полем индекса. Например, в Cosmos DB идентификатор отеля называется **`HotelId`** . Но в файлах JSON больших двоичных объектов с информацией о номерах отелей этот идентификатор именуется **`Id`** . Программа решает эту проблему, сопоставляя поле **`Id`** из больших двоичных объектов c полем ключа **`HotelId`** в индексаторе.

> [!NOTE]
> В большинстве случаев автоматически сформированные ключи документов, например стандартные ключи некоторых индексаторов, плохо подходят для объединенных индексов. Обычно вам нужны содержательные и уникальные значения ключей, которые уже существуют в источниках данных и (или) легко могут быть добавлены к ним.

## <a name="4---explore-the-code"></a>4\. Обзор кода

Когда вы завершите отправку данных и настройку параметров конфигурации, пример программы **/v11/AzureSearchMultipleDataSources.sln** будет готов к сборке и запуску.

Это простое консольное приложение для C#/.NET выполняет следующие задачи:

* создает индекс на основе структуры данных в классе C# Hotel (который также ссылается на классы Address и Room);
* создает источник данных и индексатор, который сопоставляет данные в Azure Cosmos DB с полями индекса (это объекты в службе "Когнитивный поиск Azure");
* запускает индексатор для загрузки данных Hotel из Cosmos DB;
* создает второй источник данных и индексатор, который сопоставляет поля индекса и данные больших двоичных объектов в формате JSON;
* запускает второй индексатор для загрузки данных Rooms из хранилища BLOB-объектов.

 Перед выполнением этой программы вам стоит изучить ее код и определения индекса и индексатора, настроенные для нашего примера. Соответствующий код находится в двух файлах:

  + **Hotel.cs** содержит схему, которая определяет индекс;
  + **Program.cs** содержит функции, которые создают индекс службы "Когнитивный поиск Azure", источники данных и индексаторы, а также отправляют в индекс объединенные результаты.

### <a name="create-an-index"></a>Создание индекса

Этот пример программы использует [CreateIndexAsync](/dotnet/api/azure.search.documents.indexes.searchindexclient.createindexasync), чтобы определить и создать индекс службы "Когнитивный поиск Azure". С помощью [FieldBuilder](/dotnet/api/azure.search.documents.indexes.fieldbuilder) из класса C# модели данных программа создает структуру индекса.

Модель данных определяется в классе Hotel, который содержит также ссылки на классы Address и Room. FieldBuilder углубляется в структуру определений нескольких классов, создавая на их основе сложную структуру данных для индекса. Теги метаданных применяются для определения атрибутов каждого поля, например сведений о поддержке поиска или сортировки.

Перед созданием индекса программа также удаляет существующий индекс с тем же именем на случай, если вы будете выполнять этот пример более одного раза.

В следующих фрагментах кода из файла **Hotel.cs** показаны отдельные поля, а затем ссылка на другой класс модели данных, Room[], которая определена в файле **Room.cs** (не отображается).

```csharp
. . .
[SimpleField(IsFilterable = true, IsKey = true)]
public string HotelId { get; set; }

[SearchableField(IsFilterable = true, IsSortable = true)]
public string HotelName { get; set; }
. . .
public Room[] Rooms { get; set; }
. . .
```

В файле **Program.cs** для [SearchIndex](/dotnet/api/azure.search.documents.indexes.models.searchindex) определяется имя и коллекция полей, созданных методом `FieldBuilder.Build`, а затем этот индекс создается, как показано ниже:

```csharp
private static async Task CreateIndexAsync(string indexName, SearchIndexClient indexClient)
{
    // Create a new search index structure that matches the properties of the Hotel class.
    // The Address and Room classes are referenced from the Hotel class. The FieldBuilder
    // will enumerate these to create a complex data structure for the index.
    FieldBuilder builder = new FieldBuilder();
    var definition = new SearchIndex(indexName, builder.Build(typeof(Hotel)));

    await indexClient.CreateIndexAsync(definition);
}
```

### <a name="create-azure-cosmos-db-data-source-and-indexer"></a>Создание источника данных и индексатора Azure Cosmos DB

Далее основая программа включает логику для создания источника данных Azure Cosmos DB, где будут храниться данные об отелях.

Сначала она объединяет имя базы данных Azure Cosmos DB и строку подключения. Затем определяет объект [SearchIndexerDataSourceConnection](/dotnet/api/azure.search.documents.indexes.models.searchindexerdatasourceconnection).

```csharp
private static async Task CreateAndRunCosmosDbIndexerAsync(string indexName, SearchIndexerClient indexerClient)
{
    // Append the database name to the connection string
    string cosmosConnectString =
        configuration["CosmosDBConnectionString"]
        + ";Database="
        + configuration["CosmosDBDatabaseName"];

    SearchIndexerDataSourceConnection cosmosDbDataSource = new SearchIndexerDataSourceConnection(
        name: configuration["CosmosDBDatabaseName"],
        type: SearchIndexerDataSourceType.CosmosDb,
        connectionString: cosmosConnectString,
        container: new SearchIndexerDataContainer("hotels"));

    // The Cosmos DB data source does not need to be deleted if it already exists,
    // but the connection string might need to be updated if it has changed.
    await indexerClient.CreateOrUpdateDataSourceConnectionAsync(cosmosDbDataSource);
```

После создания источника данных программа настраивает индексатор Azure Cosmos DB с именем **hotel-rooms-cosmos-indexer**.

Программа обновляет все существующие индексаторы с одинаковыми названиями, перезаписывая существующий индексатор содержимым приведенного выше кода. Она также содержит команды сброса и выполнения, если вы хотите несколько раз запустить пример.

Следующий пример определяет для индексатора расписание, согласно которому он будет выполняться один раз в день. Вы можете удалить из вызова свойство расписания, если вам не нужно автоматически выполнять индексатор в будущем.

```csharp
SearchIndexer cosmosDbIndexer = new SearchIndexer(
    name: "hotel-rooms-cosmos-indexer",
    dataSourceName: cosmosDbDataSource.Name,
    targetIndexName: indexName)
{
    Schedule = new IndexingSchedule(TimeSpan.FromDays(1))
};

// Indexers keep metadata about how much they have already indexed.
// If we already ran the indexer, it "remembers" and does not run again.
// To avoid this, reset the indexer if it exists.
try
{
    await indexerClient.GetIndexerAsync(cosmosDbIndexer.Name);
    // Reset the indexer if it exists.
    await indexerClient.ResetIndexerAsync(cosmosDbIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 404)
{
    // If the indexer does not exist, 404 will be thrown.
}

await indexerClient.CreateOrUpdateIndexerAsync(cosmosDbIndexer);

Console.WriteLine("Running Cosmos DB indexer...\n");

try
{
    // Run the indexer.
    await indexerClient.RunIndexerAsync(cosmosDbIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 429)
{
    Console.WriteLine("Failed to run indexer: {0}", ex.Message);
}
```

Этот пример содержит простой блок try-catch, чтобы информировать о любых ошибках в процессе выполнения.

После выполнения индексатора Azure Cosmos DB полученный индекс поиска будет содержать полный набор документов с примерами данных об отелях. Но при этом поле rooms для каждого отеля будет содержать пустой массив, поскольку источник данных Azure Cosmos DB не содержит сведения о номерах. После этого программа выполняет запрос к хранилищу BLOB-объектов на загрузку данных о номерах для объединения с остальными данными.

### <a name="create-blob-storage-data-source-and-indexer"></a>Создание источника данных и индексатора хранилища BLOB-объектов

Чтобы получить сведения о номерах, программа прежде всего настраивает Хранилище BLOB-объектов в качестве источника данных, из которого будет использовать набор JSON-файлов с большими двоичными объектами.

```csharp
private static async Task CreateAndRunBlobIndexerAsync(string indexName, SearchIndexerClient indexerClient)
{
    SearchIndexerDataSourceConnection blobDataSource = new SearchIndexerDataSourceConnection(
        name: configuration["BlobStorageAccountName"],
        type: SearchIndexerDataSourceType.AzureBlob,
        connectionString: configuration["BlobStorageConnectionString"],
        container: new SearchIndexerDataContainer("hotel-rooms"));

    // The blob data source does not need to be deleted if it already exists,
    // but the connection string might need to be updated if it has changed.
    await indexerClient.CreateOrUpdateDataSourceConnectionAsync(blobDataSource);
```

После создания источника данных программа настраивает для него индексатор с именем **hotel-rooms-blob-indexer**, как показано ниже.

Большие двоичные объекты в формате JSON содержат поля ключа с именем **`Id`** вместо **`HotelId`** . В коде используется класс `FieldMapping`, сообщающий индексатору о необходимости направить значения поля **`Id`** в ключ документа **`HotelId`** в индексе.

Индексаторы Хранилища BLOB-объектов могут использовать [IndexingParameters](/dotnet/api/azure.search.documents.indexes.models.indexingparameters) для определения режима анализа. Вам понадобится задать различные режимы анализа в зависимости от того, представляет большой двоичный объект один документ или несколько документов. В нашем примере каждый большой двоичный объект представляет один документ JSON, поэтому в коде используется режим анализа `json`. Дополнительные сведения о параметрах индексатора для анализа больших двоичных объектов JSON см. в [этой статье](search-howto-index-json-blobs.md).

Этот пример определяет для индексатора расписание, согласно которому он будет выполняться один раз в день. Вы можете удалить из вызова свойство расписания, если вам не нужно автоматически выполнять индексатор в будущем.

```csharp
// Map the Id field in the Room documents to the HotelId key field in the index
List<FieldMapping> map = new List<FieldMapping> {
    new FieldMapping("Id")
    {
        TargetFieldName =  "HotelId"
    }
};

IndexingParameters parameters = new IndexingParameters();
parameters.Configuration.Add("parsingMode", "json");

SearchIndexer blobIndexer = new SearchIndexer(
    name: "hotel-rooms-blob-indexer",
    dataSourceName: blobDataSource.Name,
    targetIndexName: indexName)
{
    Parameters = parameters,
    Schedule = new IndexingSchedule(TimeSpan.FromDays(1))
};

blobIndexer.FieldMappings.Add(new FieldMapping("Id") { TargetFieldName = "HotelId" });

// Reset the indexer if it already exists
try
{
    await indexerClient.GetIndexerAsync(blobIndexer.Name);
    await indexerClient.ResetIndexerAsync(blobIndexer.Name);
}
catch (RequestFailedException ex) when (ex.Status == 404) { }

await indexerClient.CreateOrUpdateIndexerAsync(blobIndexer);

try
{
    // Run the indexer.
    await searchService.Indexers.RunAsync(blobIndexer.Name);
}
catch (CloudException e) when (e.Response.StatusCode == (HttpStatusCode)429)
{
    Console.WriteLine("Failed to run indexer: {0}", e.Response.Content);
}
```

Так как индекс уже был заполнен данными об отелях из базы данных Azure Cosmos DB, индексатор больших двоичных объектов просто обновляет существующие документы в индексе, добавляя в них сведения о номерах.

> [!NOTE]
> Если в обоих источниках данных есть одинаковые поля, не являющиеся ключевыми, и данные в этих полях не совпадают, то итоговый индекс будет содержать значения от того индексатора, который выполнялся последним. В нашем примере оба источники данных содержат поля **HotelName**. Если по какой-либо причине данные в этом поле различаются, то для документов с одинаковым значением ключа будет сохранено значение **HotelName** из того источника данных, который был проиндексирован позднее.

## <a name="5---search"></a>5\. Поиск

Вы можете изучить заполненный индекс поиска на портале после выполнения программы с помощью компонента [**Обозреватель поиска**](search-explorer.md).

На портале Azure откройте страницу **Обзор** для службы поиска и найдите в списке **Индексы** созданный индекс **hotel-rooms-sample**.

  :::image type="content" source="media/tutorial-multiple-data-sources/index-list.png" alt-text="Список индексов службы Когнитивный поиск Azure" border="false":::

Щелкните индекс hotel-rooms-sample в этом списке. Вы увидите интерфейс обозревателя поиска для этого индекса. Введите запрос для поиска термина, например Luxury. Вы увидите результат поиска с хотя бы одним документом, в котором отображается список объектов номеров из массива rooms.

## <a name="reset-and-rerun"></a>Сброс и повторный запуск

На ранних экспериментальных этапах разработки самый практичный подход к итерации схемы — удалить все объекты из службы "Когнитивный поиск Azure" и восстановить их с помощью кода. Имена ресурсов являются уникальными. Удаление объекта позволяет воссоздать его с использованием того же имени.

Пример кода проверяет наличие объектов и удаляет или обновляет их, так чтобы вы могли повторно выполнить программу.

Для удаления индексов, индексаторов и источников данных вы также можете использовать портал.

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы работаете в своей подписке, после завершения проекта целесообразно удалить созданные ресурсы, которые вам больше не потребуются. Работающие ресурсы могут означать лишние затраты. Можно удалить отдельные ресурсы или удалить группу ресурсов, что позволит удалить весь набор ресурсов.

Просматривать ресурсы и управлять ими можно на портале с помощью ссылок "Все ресурсы" или "Группы ресурсов" в области навигации слева.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы ознакомились с процессом приема данных из нескольких источников, давайте подробнее рассмотрим конфигурацию индексатора, начав с Cosmos DB.

> [!div class="nextstepaction"]
> [Индексирование данных Cosmos DB с помощью индексатора в службе "Когнитивный поиск Azure"](search-howto-index-cosmosdb.md)
