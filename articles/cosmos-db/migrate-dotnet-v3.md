---
title: Перенос приложения для использования пакета SDK для Azure Cosmos DB .NET 3,0 (com. Azure. Cosmos)
description: Узнайте, как обновить существующее приложение .NET с пакета SDK v2 до нового пакета SDK .NET v3 (com. Azure. Cosmos) для Core (SQL) API.
author: stefArroyo
ms.author: esarroyo
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 09/23/2020
ms.openlocfilehash: 514a6c6daccfe63865ae0b2b9f5bf29c5cbedc29
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93334027"
---
# <a name="migrate-your-application-to-use-the-azure-cosmos-db-net-sdk-v3"></a>Перенос приложения для использования пакета SDK для Azure Cosmos DB .NET v3
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!IMPORTANT]
> Дополнительные сведения о пакете SDK версии 3 для .NET Azure Cosmos DB см. в [заметках о выпуске](sql-api-sdk-dotnet-standard.md), [репозитории .NET GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3), [Советы по повышению производительности](performance-tips-dotnet-sdk-v3-sql.md).NET SDK v3 и [руководство по устранению неполадок](troubleshoot-dot-net-sdk.md).
>

В этой статье рассматриваются некоторые вопросы обновления существующего приложения .NET до более нового Azure Cosmos DB API .NET v3 для Core (SQL). Azure Cosmos DB пакета SDK для .NET v3 соответствует пространству имен Microsoft. Azure. Cosmos. Сведения, приведенные в этом документе, можно использовать при переносе приложения из любого из следующих Azure Cosmos DB пакетов SDK для .NET:

* Azure Cosmos DB платформа .NET Framework SDK v2 для API SQL
* Azure Cosmos DB пакет SDK для .NET Core v2 для API SQL

Инструкции, приведенные в этой статье, также помогут перенести следующие внешние библиотеки, которые теперь входят в состав API-интерфейса Azure Cosmos DB .NET SDK v3 для Core (SQL):

* Библиотека обработчика веб-канала изменений .NET 2,0
* Библиотека .NET NET выполнителя 1,1 или более поздней версии

## <a name="whats-new-in-the-net-v3-sdk"></a>Новые возможности пакета SDK для .NET v3

Пакет SDK V3 содержит множество улучшений удобства использования и производительности, в том числе:

* Понятное Именование модели программирования
* .NET Standard 2,0 * *
* Повышение производительности за счет поддержки API потоков
* Иерархия Fluent, которая заменяет потребность в фабрике URI
* Встроенная поддержка библиотеки обработчика веб-канала изменений
* Встроенная поддержка для операций с массовыми операциями
* Интерфейсы API моккабале для упрощения модульного тестирования
* Транзакционная поддержка пакетной службы и Блазор
* Подключаемые сериализаторы
* Масштабировать контейнеры без секционирования и автомасштабирования

* * Пакет SDK предназначен для .NET Standard 2,0, который объединяет существующие Azure Cosmos DB платформа .NET Framework и пакеты SDK для .NET Core в один пакет SDK для .NET. Пакет SDK для .NET можно использовать на любой платформе, реализующей .NET Standard 2,0, включая приложения платформа .NET Framework 4.6.1 + и .NET Core 2.0 +.

Большинство сетей, логика повторных попыток и более низкие уровни пакета SDK остаются в значительной степени без изменений.

**Azure Cosmos DB .NET SDK версии 3 теперь является открытым исходным кодом.** Мы рады всем запросам на вытягивание и будем регистрировать проблемы и отслеживать Отзывы на [GitHub.](https://github.com/Azure/azure-cosmos-dotnet-v3/) Мы будем работать над всеми функциями, которые улучшат работу клиентов.

## <a name="why-migrate-to-the-net-v3-sdk"></a>Зачем переходить на пакет SDK для .NET v3

В дополнение к многочисленным улучшениям в области удобства использования и производительности новые функции, внесенные в последний пакет SDK, не будут перенесены в более старые версии.

Хотя нет непосредственных планов, позволяющих [снять поддержку пакетов sdk 2,0](sql-api-sdk-dotnet.md), пакеты SDK будут заменены более новыми версиями в будущем, и пакет SDK перейдет в режим обслуживания. Для лучшего удобства разработки рекомендуется всегда начинать с последней поддерживаемой версии пакета SDK.

## <a name="major-name-changes-from-v2-sdk-to-v3-sdk"></a>Изменение основного имени с пакета SDK v2 на v3 SDK

Следующие изменения имен были применены в пакете SDK для .NET 3,0, чтобы согласовать с соглашениями об именовании API для ядра API (SQL):

* `DocumentClient` переименовывается в `CosmosClient`
* `Collection` переименовывается в `Container`
* `Document` переименовывается в `Item`

Все объекты ресурсов переименовываются с помощью дополнительных свойств, которые включают имя ресурса для ясности.

Ниже приведены некоторые изменения имени основного класса:

| Пакет SDK для .NET v2 | Пакет SDK для .NET v3 |
|-------------|-------------|
|`Microsoft.Azure.Documents.Client.DocumentClient`|`Microsoft.Azure.CosmosClient`|
|`Microsoft.Azure.Documents.Client.ConnectionPolicy`|`Microsoft.Azure.Cosmos.CosmosClientOptions`|
|`Microsoft.Azure.Documents.Client.DocumentClientException` |`Microsoft.Azure.Cosmos.CosmosException`|
|`Microsoft.Azure.Documents.Client.Database`|`Microsoft.Azure.Cosmos.DatabaseProperties`|
|`Microsoft.Azure.Documents.Client.DocumentCollection`|`Microsoft.Azure.Cosmos.ContainerProperties`|
|`Microsoft.Azure.Documents.Client.RequestOptions`|`Microsoft.Azure.Cosmos.ItemRequestOptions`|
|`Microsoft.Azure.Documents.Client.FeedOptions`|`Microsoft.Azure.Cosmos.QueryRequestOptions`|
|`Microsoft.Azure.Documents.Client.StoredProcedure`|`Microsoft.Azure.Cosmos.StoredProcedureProperties`|
|`Microsoft.Azure.Documents.Client.Trigger`|`Microsoft.Azure.Cosmos.TriggerProperties`|

### <a name="classes-replaced-on-net-v3-sdk"></a>Классы, замененные на пакете SDK для .NET v3

В пакете SDK 3,0 были заменены следующие классы:

* `Microsoft.Azure.Documents.UriFactory`

* `Microsoft.Azure.Documents.Document`

* `Microsoft.Azure.Documents.Resource`

Microsoft.Azure.Docументс. Класс UriFactory был заменен конструкцией Fluent. Дизайнер Fluent внутренне создает URL-адреса и позволяет `Container` передавать один объект вместо `DocumentClient` , `DatabaseName` , и `DocumentCollection` .

### <a name="changes-to-item-id-generation"></a>Изменения при создании идентификатора элемента

Идентификатор элемента больше не заполняется автоматически в пакете SDK для .NET v3. Поэтому идентификатор элемента должен содержать сформированный идентификатор. Просмотрите следующий пример:

```csharp
[JsonProperty(PropertyName = "id")]
public Guid Id { get; set; }
```

### <a name="changed-default-behavior-for-connection-mode"></a>Изменено поведение по умолчанию для режима соединения

По умолчанию пакет SDK v3 имеет значение Direct + TCP-подключения по сравнению с предыдущим пакетом SDK v2, который по умолчанию используется в режимах подключения шлюза и HTTPS. Это изменение обеспечивает повышенную производительность и масштабируемость.

### <a name="changes-to-feedoptions-queryrequestoptions-in-v30-sdk"></a>Изменения в FeedOptions (Куерирекуестоптионс в пакете SDK для версии 3.0)

`FeedOptions`Класс в пакете SDK версии 2 теперь был переименован в `QueryRequestOptions` пакете SDK v3 и в классе. Некоторые свойства имели изменения в имени и (или) значении по умолчанию или полностью удалены.  

`FeedOptions.MaxDegreeOfParallelism` переименовано в, `QueryRequestOptions.MaxConcurrency` и значение по умолчанию и связанное поведение остаются неизменными. на стороне клиента операции запуска во время параллельного выполнения запроса будут выполняться последовательно с непараллелизмом.

`FeedOptions.EnableCrossPartitionQuery` было удалено, и поведение по умолчанию в пакете SDK 3,0 заключается в том, что запросы между секциями будут выполняться без необходимости включать свойство специально.

`FeedOptions.PopulateQueryMetrics` параметр включен по умолчанию с результатами, присутствующими в свойстве Diagnostics ответа.

`FeedOptions.RequestContinuation` Теперь был выдвинут на сами методы запросов.

Удалены следующие свойства:

* `FeedOptions.DisableRUPerMinuteUsage`

* `FeedOptions.EnableCrossPartitionQuery`

* `FeedOptions.JsonSerializerSettings`

* `FeedOptions.PartitionKeyRangeId`

* `FeedOptions.PopulateQueryMetrics`

### <a name="constructing-a-client"></a>Создание клиента

Пакет SDK для .NET v3 предоставляет `CosmosClientBuilder` класс Fluent, который заменяет потребность в фабрике URI пакета SDK v2.

В следующем примере создается новый объект `CosmosClientBuilder` со строгим ConsistencyLevel и списком предпочтительных расположений:

```csharp
CosmosClientBuilder cosmosClientBuilder = new CosmosClientBuilder(
    accountEndpoint: "https://testcosmos.documents.azure.com:443/",
    authKeyOrResourceToken: "SuperSecretKey")
.WithConsistencyLevel(ConsistencyLevel.Strong)
.WithApplicationRegion(Regions.EastUS);
CosmosClient client = cosmosClientBuilder.Build();
```

### <a name="using-the-change-feed-processor-apis-directly-from-the-v3-sdk"></a>Использование API-интерфейсов обработчика веб-канала изменений непосредственно из пакета SDK v3

Пакет SDK v3 имеет встроенную поддержку API-интерфейсов обработчика веб-канала изменений, позволяя использовать один и тот же пакет SDK для создания приложения и реализации обработчика веб-канала изменений. Ранее было необходимо использовать отдельную библиотеку обработчика канала изменений.

Дополнительные сведения см. в разделе [как выполнить миграцию из библиотеки обработчика веб-канала изменений в Azure Cosmos DB пакет SDK для .NET v3](how-to-migrate-from-change-feed-library.md) .

### <a name="using-the-bulk-executor-library-directly-from-the-v3-sdk"></a>Использование библиотеки пакетного исполнителя непосредственно из пакета SDK v3

Пакет SDK v3 имеет встроенную поддержку библиотеки для небольшого выполнителя, что позволяет использовать один и тот же пакет SDK для создания приложения и выполнения групповой работы. Ранее было необходимо использовать отдельную библиотеку небольшого выполнителя.

Дополнительные сведения см. [в разделе как выполнить миграцию из библиотеки небольшого исполнителя в пакетную поддержку в Azure Cosmos DB пакете SDK для .NET v3](how-to-migrate-from-bulk-executor-library.md) .

## <a name="code-snippet-comparisons"></a>Сравнение фрагментов кода

В следующем фрагменте кода показаны различия между созданием ресурсов между пакетами SDK для .NET v2 и v3.

## <a name="database-operations"></a>Операции с базой данных

### <a name="create-a-database"></a>Создание базы данных

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
// Create database with no shared provisioned throughput
DatabaseResponse databaseResponse = await client.CreateDatabaseIfNotExistsAsync(DatabaseName);
Database database = databaseResponse;
DatabaseProperties databaseProperties = databaseResponse;

// Create a database with a shared manual provisioned throughput
string databaseIdManual = new string(DatabaseName + "_SharedManualThroughput");
database = await client.CreateDatabaseIfNotExistsAsync(databaseIdManual, ThroughputProperties.CreateManualThroughput(400));

// Create a database with shared autoscale provisioned throughput
string databaseIdAutoscale = new string(DatabaseName + "_SharedAutoscaleThroughput");
database = await client.CreateDatabaseIfNotExistsAsync(databaseIdAutoscale, ThroughputProperties.CreateAutoscaleThroughput(4000));
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
// Create database
ResourceResponse<Database> databaseResponse = await client.CreateDatabaseIfNotExistsAsync(new Database { Id = DatabaseName });
Database database = databaseResponse;

// Create a database with shared standard provisioned throughput
database = await client.CreateDatabaseIfNotExistsAsync(new Database{ Id = databaseIdStandard }, new RequestOptions { OfferThroughput = 400 });

// Creating a database with shared autoscale provisioned throughput from v2 SDK is not supported use v3 SDK
```
---

### <a name="read-a-database-by-id"></a>Чтение базы данных по идентификатору

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
// Read a database
Console.WriteLine($"{Environment.NewLine} Read database resource: {DatabaseName}");
database = client.GetDatabase(DatabaseName);
Console.WriteLine($"{Environment.NewLine} database { database.Id.ToString()}");

// Read all databases
string findQueryText = "SELECT * FROM c";
using (FeedIterator<DatabaseProperties> feedIterator = client.GetDatabaseQueryIterator<DatabaseProperties>(findQueryText))
{
    while (feedIterator.HasMoreResults)
    {
        FeedResponse<DatabaseProperties> databaseResponses = await feedIterator.ReadNextAsync();
        foreach (DatabaseProperties _database in databaseResponses)
        {
            Console.WriteLine($"{ Environment.NewLine} database {_database.Id.ToString()}");
        }
    }
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
// Read a database
database = await client.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(DatabaseName));
Console.WriteLine("\n database {0}", database.Id.ToString());

// Read all databases
Console.WriteLine("\n1.1 Reading all databases resources");
foreach (Database _database in await client.ReadDatabaseFeedAsync())
{
    Console.WriteLine("\n database {0} \n {1}", _database.Id.ToString(), _database.ToString());
}
```
---

### <a name="delete-a-database"></a>удаление базы данных.

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
// Delete a database
await client.GetDatabase(DatabaseName).DeleteAsync();
Console.WriteLine($"{ Environment.NewLine} database {DatabaseName} deleted.");

// Delete all databases in an account
string deleteQueryText = "SELECT * FROM c";
using (FeedIterator<DatabaseProperties> feedIterator = client.GetDatabaseQueryIterator<DatabaseProperties>(deleteQueryText))
{
    while (feedIterator.HasMoreResults)
    {
        FeedResponse<DatabaseProperties> databaseResponses = await feedIterator.ReadNextAsync();
        foreach (DatabaseProperties _database in databaseResponses)
        {
            await client.GetDatabase(_database.Id).DeleteAsync();
            Console.WriteLine($"{ Environment.NewLine} database {_database.Id} deleted");
        }
    }
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
// Delete a database
database = await client.DeleteDatabaseAsync(UriFactory.CreateDatabaseUri(DatabaseName));
Console.WriteLine(" database {0} deleted.", DatabaseName);

// Delete all databases in an account
foreach (Database _database in await client.ReadDatabaseFeedAsync())
{
    await client.DeleteDatabaseAsync(UriFactory.CreateDatabaseUri(_database.Id));
    Console.WriteLine("\n database {0} deleted", _database.Id);
}
```
---

## <a name="container-operations"></a>Операции с контейнерами

### <a name="create-a-container-autoscale--time-to-live-with-expiration"></a>Создание контейнера (Автомасштабирование + время жизни с истекающим сроком действия)

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
private static async Task CreateManualThroughputContainer(Database database)
{
    // Set throughput to the minimum value of 400 RU/s manually configured throughput
    string containerIdManual = ContainerName + "_Manual";
    ContainerResponse container = await database.CreateContainerIfNotExistsAsync(
        id: containerIdManual,
        partitionKeyPath: partitionKeyPath,
        throughput: 400);
}

// Create container with autoscale
private static async Task CreateAutoscaleThroughputContainer(Database database)
{
    string autoscaleContainerId = ContainerName + "_Autoscale";
    ContainerProperties containerProperties = new ContainerProperties(autoscaleContainerId, partitionKeyPath);

    Container container = await database.CreateContainerIfNotExistsAsync(
        containerProperties: containerProperties,
        throughputProperties: ThroughputProperties.CreateAutoscaleThroughput(autoscaleMaxThroughput: 4000);
}

// Create a container with TTL Expiration
private static async Task CreateContainerWithTtlExpiration(Database database)
{
    string containerIdManualwithTTL = ContainerName + "_ManualTTL";

    ContainerProperties properties = new ContainerProperties
        (id: containerIdManualwithTTL,
        partitionKeyPath: partitionKeyPath);

    properties.DefaultTimeToLive = (int)TimeSpan.FromDays(1).TotalSeconds; //expire in 1 day

    ContainerResponse containerResponse = await database.CreateContainerIfNotExistsAsync(containerProperties: properties);
    ContainerProperties returnedProperties = containerResponse;
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
// Create a collection
private static async Task CreateManualThroughputContainer(DocumentClient client)
{
    string containerIdManual = ContainerName + "_Manual";

    // Set throughput to the minimum value of 400 RU/s manually configured throughput

    DocumentCollection collectionDefinition = new DocumentCollection();
    collectionDefinition.Id = containerIdManual;
    collectionDefinition.PartitionKey.Paths.Add(partitionKeyPath);

    DocumentCollection partitionedCollection = await client.CreateDocumentCollectionIfNotExistsAsync(
        UriFactory.CreateDatabaseUri(DatabaseName),
        collectionDefinition,
        new RequestOptions { OfferThroughput = 400 });
}

private static async Task CreateAutoscaleThroughputContainer(DocumentClient client)
{
        // .NET v2 SDK does not support the creation of provisioned autoscale throughput containers
}

 private static async Task CreateContainerWithTtlExpiration(DocumentClient client)
{
    string containerIdManualwithTTL = ContainerName + "_ManualTTL";

    DocumentCollection collectionDefinition = new DocumentCollection();
    collectionDefinition.Id = containerIdManualwithTTL;
    collectionDefinition.DefaultTimeToLive = (int)TimeSpan.FromDays(1).TotalSeconds; //expire in 1 day
    collectionDefinition.PartitionKey.Paths.Add(partitionKeyPath);

    DocumentCollection partitionedCollection = await client.CreateDocumentCollectionIfNotExistsAsync(
        UriFactory.CreateDatabaseUri(DatabaseName),
        collectionDefinition,
        new RequestOptions { OfferThroughput = 400 });

}
```
---

### <a name="read-container-properties"></a>Чтение свойств контейнера

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
private static async Task ReadContainerProperties(Database database)
{
    string containerIdManual = ContainerName + "_Manual";
    Container container = database.GetContainer(containerIdManual);
    ContainerProperties containerProperties = await container.ReadContainerAsync();
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
private static async Task ReadContainerProperties(DocumentClient client)
{
    string containerIdManual = ContainerName + "_Manual";
    DocumentCollection collection = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(DatabaseName, containerIdManual));
}
```
---

### <a name="delete-a-container"></a>Удаление контейнера

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
private static async Task DeleteContainers(Database database)
{
    string containerIdManual = ContainerName + "_Manual";

    // Delete a container
    await database.GetContainer(containerIdManual).DeleteContainerAsync();

    // Delete all CosmosContainer resources for a database
    using (FeedIterator<ContainerProperties> feedIterator = database.GetContainerQueryIterator<ContainerProperties>())
    {
        while (feedIterator.HasMoreResults)
        {
            foreach (ContainerProperties _container in await feedIterator.ReadNextAsync())
            {
                await database.GetContainer(_container.Id).DeleteContainerAsync();
                Console.WriteLine($"{Environment.NewLine}  deleted container {_container.Id}");
            }
        }
    }
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
private static async Task DeleteContainers(DocumentClient client)
{
    // Delete a collection
    string containerIdManual = ContainerName + "_Manual";
    await client.DeleteDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(DatabaseName, containerIdManual));

    // Delete all containers for a database
    foreach (var collection in await client.ReadDocumentCollectionFeedAsync(UriFactory.CreateDatabaseUri(DatabaseName)))
    {
        await client.DeleteDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri(DatabaseName, collection.Id));
    }
}
```
---

## <a name="item-and-query-operations"></a>Операции с элементами и запросами

### <a name="create-an-item"></a>Создание элемента

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
private static async Task CreateItemAsync(Container container)
{
    // Create a SalesOrder POCO object
    SalesOrder salesOrder1 = GetSalesOrderSample("Account1", "SalesOrder1");
    ItemResponse<SalesOrder> response = await container.CreateItemAsync(salesOrder1,
        new PartitionKey(salesOrder1.AccountNumber));
}

private static async Task RunBasicOperationsOnDynamicObjects(Container container)
{
    // Dynamic Object
    dynamic salesOrder = new
    {
        id = "SalesOrder5",
        AccountNumber = "Account1",
        PurchaseOrderNumber = "PO18009186470",
        OrderDate = DateTime.UtcNow,
        Total = 5.95,
    };
    Console.WriteLine("\nCreating item");
    ItemResponse<dynamic> response = await container.CreateItemAsync<dynamic>(
        salesOrder, new PartitionKey(salesOrder.AccountNumber));
    dynamic createdSalesOrder = response.Resource;
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
private static async Task CreateItemAsync(DocumentClient client)
{
    // Create a SalesOrder POCO object
    SalesOrder salesOrder1 = GetSalesOrderSample("Account1", "SalesOrder1");
    await client.CreateDocumentAsync(
        UriFactory.CreateDocumentCollectionUri(DatabaseName, ContainerName),
        salesOrder1,
        new RequestOptions { PartitionKey = new PartitionKey("Account1")});
}

private static async Task RunBasicOperationsOnDynamicObjects(DocumentClient client)
{
    // Create a dynamic object
    dynamic salesOrder = new
    {
        id= "SalesOrder5",
        AccountNumber = "Account1",
        PurchaseOrderNumber = "PO18009186470",
        OrderDate = DateTime.UtcNow,
        Total = 5.95,
    };
    ResourceResponse<Document> response = await client.CreateDocumentAsync(
        UriFactory.CreateDocumentCollectionUri(DatabaseName, ContainerName),
        salesOrder,
        new RequestOptions { PartitionKey = new PartitionKey(salesOrder.AccountNumber)});

    dynamic createdSalesOrder = response.Resource;
    }
```
---

### <a name="read-all-the-items-in-a-container"></a>Считывание всех элементов в контейнере

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
private static async Task ReadAllItems(Container container)
{
    // Read all items in a container
    List<SalesOrder> allSalesForAccount1 = new List<SalesOrder>();

    using (FeedIterator<SalesOrder> resultSet = container.GetItemQueryIterator<SalesOrder>(
        queryDefinition: null,
        requestOptions: new QueryRequestOptions()
        {
            PartitionKey = new PartitionKey("Account1"),
            MaxItemCount = 5
        }))
    {
        while (resultSet.HasMoreResults)
        {
            FeedResponse<SalesOrder> response = await resultSet.ReadNextAsync();
            SalesOrder salesOrder = response.First();
            Console.WriteLine($"\n1.3.1 Account Number: {salesOrder.AccountNumber}; Id: {salesOrder.Id}");
            allSalesForAccount1.AddRange(response);
        }
    }
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
private static async Task ReadAllItems(DocumentClient client)
{
    // Read all items in a collection
    List<SalesOrder> allSalesForAccount1 = new List<SalesOrder>();

    string continuationToken = null;
    do
    {
        var feed = await client.ReadDocumentFeedAsync(
            UriFactory.CreateDocumentCollectionUri(DatabaseName, ContainerName),
            new FeedOptions { MaxItemCount = 5, RequestContinuation = continuationToken });
        continuationToken = feed.ResponseContinuation;
        foreach (Document document in feed)
        {
            SalesOrder salesOrder = (SalesOrder)(dynamic)document;
            Console.WriteLine($"\n1.3.1 Account Number: {salesOrder.AccountNumber}; Id: {salesOrder.Id}");
            allSalesForAccount1.Add(salesOrder);

        }
    } while (continuationToken != null);
}
```
---

### <a name="query-items"></a>Элементы запроса

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
private static async Task QueryItems(Container container)
{
    // Query for items by a property other than Id
    QueryDefinition queryDefinition = new QueryDefinition(
        "select * from sales s where s.AccountNumber = @AccountInput")
        .WithParameter("@AccountInput", "Account1");

    List<SalesOrder> allSalesForAccount1 = new List<SalesOrder>();
    using (FeedIterator<SalesOrder> resultSet = container.GetItemQueryIterator<SalesOrder>(
        queryDefinition,
        requestOptions: new QueryRequestOptions()
        {
            PartitionKey = new PartitionKey("Account1"),
            MaxItemCount = 1
        }))
    {
        while (resultSet.HasMoreResults)
        {
            FeedResponse<SalesOrder> response = await resultSet.ReadNextAsync();
            SalesOrder sale = response.First();
            Console.WriteLine($"\n Account Number: {sale.AccountNumber}; Id: {sale.Id};");
            allSalesForAccount1.AddRange(response);
        }
    }
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
private static async Task QueryItems(DocumentClient client)
{
    // Query for items by a property other than Id
    SqlQuerySpec querySpec = new SqlQuerySpec()
    {
        QueryText = "select * from sales s where s.AccountNumber = @AccountInput",
        Parameters = new SqlParameterCollection()
            {
                new SqlParameter("@AccountInput", "Account1")
            }
    };
    var query = client.CreateDocumentQuery<SalesOrder>(
        UriFactory.CreateDocumentCollectionUri(DatabaseName, ContainerName),
        querySpec,
        new FeedOptions {EnableCrossPartitionQuery = true});

    var allSalesForAccount1 = query.ToList();

    Console.WriteLine($"\n1.4.2 Query found {allSalesForAccount1.Count} items.");
}
```
---

### <a name="delete-an-item"></a>Удаление элемента

# <a name="net-sdk-v3"></a>[Пакет SDK версии 3 для .NET](#tab/dotnet-v3)

```csharp
private static async Task DeleteItemAsync(Container container)
{
    ItemResponse<SalesOrder> response = await container.DeleteItemAsync<SalesOrder>(
        partitionKey: new PartitionKey("Account1"), id: "SalesOrder3");
}
```

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnet-v2)

```csharp
private static async Task DeleteItemAsync(DocumentClient client)
{
    ResourceResponse<Document> response = await client.DeleteDocumentAsync(
        UriFactory.CreateDocumentUri(DatabaseName, ContainerName, "SalesOrder3"),
        new RequestOptions { PartitionKey = new PartitionKey("Account1") });
}
```
---

## <a name="next-steps"></a>Дальнейшие действия

* [Создание консольного приложения](sql-api-get-started.md) для управления Azure Cosmos DB данных API SQL с помощью пакета SDK v3
* Дополнительные сведения о том, [что можно сделать с помощью пакета SDK v3](sql-api-dotnet-v3sdk-samples.md)
