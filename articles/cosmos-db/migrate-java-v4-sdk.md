---
title: Перевод приложения на использование пакета средств разработки Java для Azure Cosmos DB версии 4 (com.azure.cosmos)
description: Узнайте, как обновить существующее приложение Java, в котором используется старая версия пакета средств разработки Java для Azure Cosmos DB, до нового пакета средств разработки Java версии 4.0 (пакет com.azure.cosmos), работающего с API Core (SQL).
author: anfeldma-ms
ms.custom: devx-track-java
ms.author: anfeldma
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/11/2020
ms.reviewer: sngun
ms.openlocfilehash: 92a9abec36bd75c594c67843286bf8fa067d7dba
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101658543"
---
# <a name="migrate-your-application-to-use-the-azure-cosmos-db-java-sdk-v4"></a>Перевод приложения на использование пакета средств разработки Java для Azure Cosmos DB версии 4
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!IMPORTANT]  
> Дополнительные сведения об этом пакете SDK см. в [заметках о выпуске](sql-api-sdk-java-v4.md) пакета SDK Java Azure Cosmos DB версии 4, [репозитории Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos), [рекомендациях по повышению производительности](performance-tips-java-sdk-v4-sql.md) для пакета SDK Java Azure Cosmos DB версии 4, а также в [руководстве по устранению неполадок](troubleshoot-java-sdk-v4-sql.md) для пакета SDK Java Azure Cosmos DB версии 4.
>

В этой статье объясняется, как обновить существующее приложение Java, в котором используется старая версия пакета средств разработки Java для Azure Cosmos DB, до нового пакета средств разработки Java для Azure Cosmos DB версии 4.0, работающего с API Core (SQL). Пакет средств разработки Java для Azure Cosmos DB версии 4 соответствует пакету `com.azure.cosmos`. Инструкции из этого документа можно использовать при переводе приложения с любого из следующих пакетов средств разработки Java для Azure Cosmos DB: 

* пакет средств разработки Sync Java версии 2.x.x;
* пакет средств разработки Async Java версии 2.x.x;
* пакет средств разработки Java версии 3.x.x.

## <a name="azure-cosmos-db-java-sdks-and-package-mappings"></a>Сопоставления пакета средств разработки Java для Azure Cosmos DB и пакетов

В следующей таблице перечислены разные пакеты средств разработки Java для Azure Cosmos DB, имена пакетов и сведения о выпуске.

| Пакет SDK для Java| Дата выпуска | Пакетные API-интерфейсы   | JAR-файл Maven  | Имя пакета Java  |Справочник по API   | Заметки о выпуске  |
|-------|------|-----------|-----------|--------------|-------------|---------------------------|
| Async 2.x.x  | Июнь 2018 г.    | Async(RxJava)  | `com.microsoft.azure::azure-cosmosdb` | `com.microsoft.azure.cosmosdb.rx` | [API](https://azure.github.io/azure-cosmosdb-java/2.0.0/) | [Заметки о выпуске](sql-api-sdk-async-java.md) |
| Sync 2.x.x     | Сентябрь 2018 г.    | Sync   | `com.microsoft.azure::azure-documentdb` | `com.microsoft.azure.cosmosdb` | [API](https://azure.github.io/azure-cosmosdb-java/2.0.0/) | [Заметки о выпуске](sql-api-sdk-java.md)  |
| 3.x.x    | Июль 2019 г.    | Async(Reactor)/Sync  | `com.microsoft.azure::azure-cosmos`  | `com.azure.data.cosmos` | [API](https://azure.github.io/azure-cosmosdb-java/3.0.0/) | - |
| 4,0   | Июнь 2020 г.   | Async(Reactor)/Sync  | `com.azure::azure-cosmos` | `com.azure.cosmos`   | -  | [API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-cosmos/4.0.1/index.html)  |

## <a name="sdk-level-implementation-changes"></a>Изменения реализации на уровне пакета SDK

Ниже приведены основные различия в реализации между разными пакетами SDK.

### <a name="rxjava-is-replaced-with-reactor-in-azure-cosmos-db-java-sdk-versions-3xx-and-40"></a>RxJava заменяется на Reactor в пакетах средств разработки Java для Azure Cosmos DB версий 3.x.x и 4.0

Если вы еще не знакомы с асинхронным и (или) реактивным программированием, изучите [руководство по шаблону реактора](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-pattern-guide.md), в котором приведены общие сведения об асинхронном программировании и проекте Reactor. Это руководство может оказаться полезным, если вы ранее использовали пакет средств разработки Sync Java для Azure Cosmos DB версии 2.x.x или пакет средств разработки Sync Java для Azure Cosmos DB версии 3.x.x.

Если вы используете пакет средств разработки Async Java для Azure Cosmos DB версии 2.x.x и планируете перейти на пакет SDK 4.0, ознакомьтесь со статьей [Сравнительная характеристика Reactor и RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-rxjava-guide.md), где представлены инструкции по преобразованию кода для RxJava в код для Reactor.

### <a name="azure-cosmos-db-java-sdk-v4-has-direct-connectivity-mode-in-both-async-and-sync-apis"></a>Пакет средств разработки Java для Azure Cosmos DB версии 4 имеет режим прямого подключения к API-интерфейсам Sync и Async.

Если вы используете пакет средств разработки Sync Java для Azure Cosmos DB версии 2.x.x, обратите внимание, что режим прямого подключения по протоколу TCP (а не HTTP) реализован в пакете средств разработки Java для Azure Cosmos DB версии 4.0 для API-интерфейсов Async и Sync.

## <a name="api-level-changes"></a>Изменения на уровне API

Ниже перечислены изменения на уровне API в пакете средств разработки Java для Azure Cosmos DB версии 4.x.x по сравнению с предыдущими пакетами SDK (Java SDK 3.x.x, Async Java SDK 2.x.x и Sync Java SDK 2.x.x):

:::image type="content" source="./media/migrate-java-v4-sdk/java-sdk-naming-conventions.png" alt-text="Конвенции именования в пакете средства разработки Sync Java для Azure Cosmos DB":::

* Пакеты средств разработки Java для Azure Cosmos DB версий 3.x.x и 4.0 указывают ресурсы клиента как `Cosmos<resourceName>`. Например `CosmosClient`, `CosmosDatabase`, `CosmosContainer`. В то же время пакет средств разработки Java для Azure Cosmos DB версии 2.x.x не имеет единообразной схемы именования.

* Пакеты средств разработки Java для Azure Cosmos DB версий 3.x.x и 4.0 предоставляют API-интерфейсы Sync и Async.

  * **Пакет средств разработки Java версии 4.0.** Все классы принадлежат API Sync, если имя класса не имеет постфикса `Async` после `Cosmos`.

  * **Пакет средств разработки Java версии 3.x.x.** Все классы принадлежат API Async, если имя класса не имеет постфикса `Async` после `Cosmos`.

  * **Пакет средств разработки Async Java версии 2.x.x.** Имена классов похожи на имена в пакете средств разработки Java версии 2.x.x, но всегда начинаются с префикса *Async*.

### <a name="hierarchical-api-structure"></a>Иерархическая структура API

Пакеты средств разработки Java для Azure Cosmos DB версий 4.0 и 3.x.x представляют иерархическую структуру API, которая поддерживает вложенность клиентов, баз данных и контейнеров, как показано в следующем фрагменте кода для пакета средств разработки версии 4.0:

```java
CosmosContainer container = client.getDatabase("MyDatabaseName").getContainer("MyContainerName");

```

В пакете средств разработки Java для Azure Cosmos DB версии 2.x.x все операции с ресурсами и документами выполняются через экземпляр клиента.

### <a name="representing-documents"></a>Представление документов

В пакете средств разработки Java для Azure Cosmos DB версии 4.0 есть два варианта для чтения и записи документов из Azure Cosmos DB — пользовательские POJO и `JsonNodes`.

В пакете средств разработки Java для Azure Cosmos DB версии 3.x.x объект `CosmosItemProperties` предоставляется через общедоступные API-интерфейсы в формате представления документа. Этот класс больше не является общедоступным в версии 4.0.

### <a name="imports"></a>Импорт

* Все пакеты средств разработки Java для Azure Cosmos DB версии 4.0 начинаются с инструкций `com.azure.cosmos`
* Пакеты средств разработки Java для Azure Cosmos DB версии 3.x.x начинаются с инструкций `com.azure.data.cosmos`
* Пакеты Sync API-интерфейсов SDK для Java версии 2.x.x в Azure Cosmos DB начинаются с `com.microsoft.azure.documentdb`

* Пакет средств разработки Java для Azure Cosmos DB версии 4.0 размещает несколько классов во вложенном пакете `com.azure.cosmos.models`. Сюда относятся следующие пакеты.

  * `CosmosContainerResponse`
  * `CosmosDatabaseResponse`
  * `CosmosItemResponse`
  * Аналоги API Async для всех перечисленных выше пакетов:
  * `CosmosContainerProperties`
  * `FeedOptions`
  * `PartitionKey`
  * `IndexingPolicy`
  * `IndexingMode`... и т. д.

### <a name="accessors"></a>Методы доступа

Пакет средств разработки Java для Azure Cosmos DB версии 4.0 предоставляет методы `get` и `set` для доступа к членам экземпляра. Например, экземпляр `CosmosContainer` содержит методы `container.getId()` и `container.setId()`.

В отличие от него, пакет средств разработки Java для Azure Cosmos DB версии 3.x.x предоставляет текучий интерфейс. Например, экземпляр `CosmosSyncContainer` имеет `container.id()` с перегрузками для получения или установки значения `id`.

## <a name="code-snippet-comparisons"></a>Сравнение фрагментов кода

### <a name="create-resources"></a>Создание ресурсов

В следующем фрагменте кода показаны различия по созданию ресурсов между Async API-интерфейсами версий 4.0, 3.x.x и Sync API-интерфейсами версий 2.x.x:

# <a name="java-sdk-40-async-api"></a>[API Async пакета средств разработки Java версии 4.0](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateJavaSDKv4ResourceAsync)]

# <a name="java-sdk-3xx-async-api"></a>[API Async пакета средств разработки Java версии 3.x.x](#tab/java-v3-async)

```java
ConnectionPolicy defaultPolicy = ConnectionPolicy.defaultPolicy();
//  Setting the preferred location to Cosmos DB Account region
defaultPolicy.preferredLocations(Lists.newArrayList("Your Account Location"));

//  Create async client
//  <CreateAsyncClient>
client = new CosmosClientBuilder()
        .endpoint("your.hostname")
        .key("yourmasterkey")
        .connectionPolicy(defaultPolicy)
        .consistencyLevel(ConsistencyLevel.EVENTUAL)
        .build();

// Create database with specified name
client.createDatabaseIfNotExists("YourDatabaseName")
      .flatMap(databaseResponse -> {
        database = databaseResponse.database();
        // Container properties - name and partition key
        CosmosContainerProperties containerProperties = 
            new CosmosContainerProperties("YourContainerName", "/id");
        // Create container with specified properties & provisioned throughput
        return database.createContainerIfNotExists(containerProperties, 400);
    }).flatMap(containerResponse -> {
        container = containerResponse.container();
        return Mono.empty();
}).subscribe();
```

# <a name="java-sdk-2xx-sync-api"></a>[Sync API-интерфейс Java SDK 2.x.x](#tab/java-v2-sync)

```java
ConnectionPolicy defaultPolicy = ConnectionPolicy.GetDefault();
//  Setting the preferred location to Cosmos DB Account region
defaultPolicy.setPreferredLocations(Lists.newArrayList("Your Account Location"));

//  Create document client
//  <CreateDocumentClient>
client = new DocumentClient("your.hostname", "your.masterkey", defaultPolicy, ConsistencyLevel.Eventual)

// Create database with specified name
Database databaseDefinition = new Database();
databaseDefinition.setId("YourDatabaseName");
ResourceResponse<Database> databaseResourceResponse = client.createDatabase(databaseDefinition, new RequestOptions());

// Read database with specified name
String databaseLink = "dbs/YourDatabaseName";
databaseResourceResponse = client.readDatabase(databaseLink, new RequestOptions());
Database database = databaseResourceResponse.getResource();

// Create container with specified name
DocumentCollection documentCollection = new DocumentCollection();
documentCollection.setId("YourContainerName");
documentCollection = client.createCollection(database.getSelfLink(), documentCollection, new RequestOptions()).getResource();
```
---

### <a name="item-operations"></a>Операции с элементами

В следующем фрагменте кода показаны различия по выполнению операций с элементами между Async API-интерфейсами версий 4.0, 3.x.x и Sync API-интерфейсами версий 2.x.x:

# <a name="java-sdk-40-async-api"></a>[API Async пакета средств разработки Java версии 4.0](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateItemOpsAsync)]

# <a name="java-sdk-3xx-async-api"></a>[API Async пакета средств разработки Java версии 3.x.x](#tab/java-v3-async)

```java
// Container is created. Generate many docs to insert.
int number_of_docs = 50000;
ArrayList<JsonNode> docs = generateManyDocs(number_of_docs);

// Insert many docs into container...
Flux.fromIterable(docs)
    .flatMap(doc -> container.createItem(doc))
    .subscribe(); // ...Subscribing triggers stream execution.
```

# <a name="java-sdk-2xx-sync-api"></a>[Sync API-интерфейс Java SDK 2.x.x](#tab/java-v2-sync)

```java
//  Container is created. Generate documents to insert.
Document document = new Document();
document.setId("YourDocumentId");
ResourceResponse<Document> documentResourceResponse = client.createDocument(documentCollection.getSelfLink(), document,
    new RequestOptions(), true);
Document responseDocument = documentResourceResponse.getResource();
```
---

### <a name="indexing"></a>Индексация

В следующем фрагменте кода показаны различия по выполнению индексирования между Async API-интерфейсами версий 4.0, 3.x.x и Sync API-интерфейсами версий 2.x.x:

# <a name="java-sdk-40-async-api"></a>[API Async пакета средств разработки Java версии 4.0](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateIndexingAsync)]

# <a name="java-sdk-3xx-async-api"></a>[API Async пакета средств разработки Java версии 3.x.x](#tab/java-v3-async)

```java
CosmosContainerProperties containerProperties = new CosmosContainerProperties(containerName, "/lastName");

// Custom indexing policy
IndexingPolicy indexingPolicy = new IndexingPolicy();
indexingPolicy.setIndexingMode(IndexingMode.CONSISTENT); //To turn indexing off set IndexingMode.NONE

// Included paths
List<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.path("/*");
includedPaths.add(includedPath);
indexingPolicy.includedPaths(includedPaths);

// Excluded paths
List<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.path("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.excludedPaths(excludedPaths);

containerProperties.indexingPolicy(indexingPolicy);

CosmosContainer containerIfNotExists = database.createContainerIfNotExists(containerProperties, 400)
                                               .block()
                                               .container();
```

# <a name="java-sdk-2xx-sync-api"></a>[Sync API-интерфейс Java SDK 2.x.x](#tab/java-v2-sync)

```java
// Custom indexing policy
IndexingPolicy indexingPolicy = new IndexingPolicy();
indexingPolicy.setIndexingMode(IndexingMode.Consistent); //To turn indexing off set IndexingMode.NONE

// Included paths
List<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.setPath("/*");
includedPaths.add(includedPath);
indexingPolicy.setIncludedPaths(includedPaths);

// Excluded paths
List<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.setPath("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.setExcludedPaths(excludedPaths);

// Create container with specified name and indexing policy
DocumentCollection documentCollection = new DocumentCollection();
documentCollection.setId("YourContainerName");
documentCollection.setIndexingPolicy(indexingPolicy);
documentCollection = client.createCollection(database.getSelfLink(), documentCollection, new RequestOptions()).getResource();
```
---

### <a name="stored-procedures"></a>Хранимые процедуры

В следующем фрагменте кода показаны различия по созданию хранимых процедур между Async API-интерфейсами версий 4.0, 3.x.x и Sync API-интерфейсами версий 2.x.x:

# <a name="java-sdk-40-async-api"></a>[API Async пакета средств разработки Java версии 4.0](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateSprocAsync)]

# <a name="java-sdk-3xx-async-api"></a>[API Async пакета средств разработки Java версии 3.x.x](#tab/java-v3-async)

```java
logger.info("Creating stored procedure...\n");

sprocId = "createMyDocument";
String sprocBody = "function createMyDocument() {\n" +
        "var documentToCreate = {\"id\":\"test_doc\"}\n" +
        "var context = getContext();\n" +
        "var collection = context.getCollection();\n" +
        "var accepted = collection.createDocument(collection.getSelfLink(), documentToCreate,\n" +
        "    function (err, documentCreated) {\n" +
        "if (err) throw new Error('Error' + err.message);\n" +
        "context.getResponse().setBody(documentCreated.id)\n" +
        "});\n" +
        "if (!accepted) return;\n" +
        "}";
CosmosStoredProcedureProperties storedProcedureDef = new CosmosStoredProcedureProperties(sprocId, sprocBody);
container.getScripts()
        .createStoredProcedure(storedProcedureDef,
                new CosmosStoredProcedureRequestOptions()).block();

// ...

logger.info(String.format("Executing stored procedure %s...\n\n", sprocId));

CosmosStoredProcedureRequestOptions options = new CosmosStoredProcedureRequestOptions();
options.partitionKey(new PartitionKey("test_doc"));

container.getScripts()
        .getStoredProcedure(sprocId)
        .execute(null, options)
        .flatMap(executeResponse -> {
            logger.info(String.format("Stored procedure %s returned %s (HTTP %d), at cost %.3f RU.\n",
                    sprocId,
                    executeResponse.responseAsString(),
                    executeResponse.statusCode(),
                    executeResponse.requestCharge()));
            return Mono.empty();
        }).block();
```

# <a name="java-sdk-2xx-sync-api"></a>[Sync API-интерфейс Java SDK 2.x.x](#tab/java-v2-sync)

```java
logger.info("Creating stored procedure...\n");

String sprocId = "createMyDocument";
String sprocBody = "function createMyDocument() {\n" +
    "var documentToCreate = {\"id\":\"test_doc\"}\n" +
    "var context = getContext();\n" +
    "var collection = context.getCollection();\n" +
    "var accepted = collection.createDocument(collection.getSelfLink(), documentToCreate,\n" +
    "    function (err, documentCreated) {\n" +
    "if (err) throw new Error('Error' + err.message);\n" +
    "context.getResponse().setBody(documentCreated.id)\n" +
    "});\n" +
    "if (!accepted) return;\n" +
    "}";
StoredProcedure storedProcedureDef = new StoredProcedure();
storedProcedureDef.setId(sprocId);
storedProcedureDef.setBody(sprocBody);
StoredProcedure storedProcedure = client.createStoredProcedure(documentCollection.getSelfLink(), storedProcedureDef, new RequestOptions())
                                        .getResource();

// ...

logger.info(String.format("Executing stored procedure %s...\n\n", sprocId));

RequestOptions options = new RequestOptions();
options.setPartitionKey(new PartitionKey("test_doc"));

StoredProcedureResponse storedProcedureResponse =
    client.executeStoredProcedure(storedProcedure.getSelfLink(), options, null);
logger.info(String.format("Stored procedure %s returned %s (HTTP %d), at cost %.3f RU.\n",
    sprocId,
    storedProcedureResponse.getResponseAsString(),
    storedProcedureResponse.getStatusCode(),
    storedProcedureResponse.getRequestCharge()));
```
---

### <a name="change-feed"></a>Канал изменений

В следующем фрагменте кода показаны различия между выполнением операций с каналом изменений между API-интерфейсами Async версий 4.0 и 3.x.x:

# <a name="java-sdk-40-async-api"></a>[API Async пакета средств разработки Java версии 4.0](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateCFAsync)]

# <a name="java-sdk-3xx-async-api"></a>[API Async пакета средств разработки Java версии 3.x.x](#tab/java-v3-async)

```java
ChangeFeedProcessor changeFeedProcessorInstance = 
ChangeFeedProcessor.Builder()
        .hostName(hostName)
        .feedContainer(feedContainer)
        .leaseContainer(leaseContainer)
        .handleChanges((List<CosmosItemProperties> docs) -> {
            logger.info("--->setHandleChanges() START");

            for (CosmosItemProperties document : docs) {
                try {

                    // You are given the document as a CosmosItemProperties instance which you may
                    // cast to the desired type.
                    CustomPOJO pojo_doc = document.getObject(CustomPOJO.class);
                    logger.info("----=>id: " + pojo_doc.id());

                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            logger.info("--->handleChanges() END");

        })
        .build();

// ...

 changeFeedProcessorInstance.start()
                            .subscribeOn(Schedulers.elastic())
                            .subscribe();
```

# <a name="java-sdk-2xx-sync-api"></a>[Sync API-интерфейс Java SDK 2.x.x](#tab/java-v2-sync)

* Эта функция не поддерживается пакетом SDK для Sync Java начиная с версии 2. 
---

### <a name="container-level-time-to-livettl"></a>Срок жизни на уровне контейнера

В следующем фрагменте кода показаны различия при создании срока жизни данных в контейнере с помощью Async API-интерфейсов версий 4.0, 3.x.x и Sync API-интерфейсов версий 2.x.x:

# <a name="java-sdk-40-async-api"></a>[API Async пакета средств разработки Java версии 4.0](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateContainerTTLAsync)]

# <a name="java-sdk-3xx-async-api"></a>[API Async пакета средств разработки Java версии 3.x.x](#tab/java-v3-async)

```java
CosmosContainer container;

// Create a new container with TTL enabled with default expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.defaultTimeToLive(90 * 60 * 60 * 24);
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```

# <a name="java-sdk-2xx-sync-api"></a>[Sync API-интерфейс Java SDK 2.x.x](#tab/java-v2-sync)

```java
DocumentCollection documentCollection;

// Create a new container with TTL enabled with default expiration value
documentCollection.setDefaultTimeToLive(90 * 60 * 60 * 24);
documentCollection = client.createCollection(database.getSelfLink(), documentCollection, new RequestOptions()).getResource();
```
---

### <a name="item-level-time-to-livettl"></a>Срок жизни на уровне элемента

В следующем фрагменте кода показаны различия при создании срока жизни для элемента с помощью Async API-интерфейсов версий 4.0, 3.x.x и Sync API-интерфейсов версий 2.x.x:

# <a name="java-sdk-40-async-api"></a>[API Async пакета средств разработки Java версии 4.0](#tab/java-v4-async)

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateItemTTLClassAsync)]

[!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=MigrateItemTTLAsync)]

# <a name="java-sdk-3xx-async-api"></a>[API Async пакета средств разработки Java версии 3.x.x](#tab/java-v3-async)

```java
// Include a property that serializes to "ttl" in JSON
public class SalesOrder
{
    private String id;
    private String customerId;
    private Integer ttl;

    public SalesOrder(String id, String customerId, Integer ttl) {
        this.id = id;
        this.customerId = customerId;
        this.ttl = ttl;
    }

    public String id() {return this.id;}
    public SalesOrder id(String new_id) {this.id = new_id; return this;}
    public String customerId() {return this.customerId; return this;}
    public SalesOrder customerId(String new_cid) {this.customerId = new_cid;}
    public Integer ttl() {return this.ttl;}
    public SalesOrder ttl(Integer new_ttl) {this.ttl = new_ttl; return this;}

    //...
}

// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder(
    "SO05",
    "CO18009186470",
    60 * 60 * 24 * 30  // Expire sales orders in 30 days
);
```

# <a name="java-sdk-2xx-sync-api"></a>[Sync API-интерфейс Java SDK 2.x.x](#tab/java-v2-sync)

```java
Document document = new Document();
document.setId("YourDocumentId");
document.setTimeToLive(60 * 60 * 24 * 30 ); // Expire document in 30 days
ResourceResponse<Document> documentResourceResponse = client.createDocument(documentCollection.getSelfLink(), document,
    new RequestOptions(), true);
Document responseDocument = documentResourceResponse.getResource();
```
---

## <a name="next-steps"></a>Дальнейшие действия

* [Создайте приложение Java](create-sql-api-java.md) для управления данными API SQL для Azure Cosmos DB с помощью пакета средств разработки версии 4.
* Узнайте о [пакетах средств разработки Java на основе Reactor](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-pattern-guide.md).
* Узнайте о преобразовании асинхронного кода RxJava в асинхронный код Reactor см. в статье [Сравнительная характеристика Reactor и RxJava](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/reactor-rxjava-guide.md).
