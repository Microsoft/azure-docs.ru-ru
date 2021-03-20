---
title: Регистрация и использование хранимых процедур, триггеров и определяемых пользователем функций в Azure Cosmos DB SDK
description: Узнайте, как зарегистрировать и вызывать хранимые процедуры, триггеры и определяемые пользователем функции с помощью пакетов SDK для Azure Cosmos DB
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tisande
ms.custom: devx-track-python, devx-track-js, devx-track-csharp
ms.openlocfilehash: 022a45199cfc2d467b1d0d408e86cb5d621070d9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339858"
---
# <a name="how-to-register-and-use-stored-procedures-triggers-and-user-defined-functions-in-azure-cosmos-db"></a>Как зарегистрировать и использовать хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

API SQL в Azure Cosmos DB поддерживает регистрацию и вызов хранимых процедур, триггеров и определяемых пользователем функций (UDF), написанных на языке JavaScript. Для регистрации и вызова хранимых процедур можно использовать пакеты SDK для [.NET](sql-api-sdk-dotnet.md), [.NET Core](sql-api-sdk-dotnet-core.md), [Java](sql-api-sdk-java.md), [JavaScript](sql-api-sdk-node.md), [Node.js](sql-api-sdk-node.md)или [Python](sql-api-sdk-python.md) . После определения одной или нескольких хранимых процедур, триггеров и определяемых пользователем функций их можно загрузить и просмотреть в [портал Azure](https://portal.azure.com/) с помощью обозреватель данных.

## <a name="how-to-run-stored-procedures"></a><a id="stored-procedures"></a>Выполнение хранимых процедур

Хранимые процедуры написаны с использованием JavaScript. Они могут создавать, обновлять, читать, запрашивать и удалять элементы в контейнере Azure Cosmos. Дополнительные сведения о том, как написать хранимую процедуру в Azure Cosmos DB см. в статье [Как писать хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB](how-to-write-stored-procedures-triggers-udfs.md#stored-procedures).

В следующих примерах показано, как регистрировать и вызывать хранимую процедуру с помощью пакетов SDK для Azure Cosmos DB. Используйте раздел [Создание документа](how-to-write-stored-procedures-triggers-udfs.md#create-an-item) как источник для этой хранимой процедуры, которая сохраняется в виде `spCreateToDoItem.js`.

> [!NOTE]
> Для секционированных контейнеров при выполнении хранимой процедуры в параметрах запроса необходимо указать значение ключа секции. Хранимые процедуры всегда ограничиваются ключом секции. Элементы с другим значением ключа секции не будут видны хранимой процедуре. Это также применяется к триггерам.

### <a name="stored-procedures---net-sdk-v2"></a>Хранимые процедуры — пакет SDK для .NET v2

В следующем примере показано, как зарегистрировать хранимую процедуру с помощью пакета SDK для .NET версии 2.

```csharp
string storedProcedureId = "spCreateToDoItems";
StoredProcedure newStoredProcedure = new StoredProcedure
   {
       Id = storedProcedureId,
       Body = File.ReadAllText($@"..\js\{storedProcedureId}.js")
   };
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
var response = await client.CreateStoredProcedureAsync(containerUri, newStoredProcedure);
StoredProcedure createdStoredProcedure = response.Resource;
```

В следующем коде показано, как вызвать хранимую процедуру с помощью пакета SDK для .NET версии 2:

```csharp
dynamic[] newItems = new dynamic[]
{
    new {
        category = "Personal",
        name = "Groceries",
        description = "Pick up strawberries",
        isComplete = false
    },
    new {
        category = "Personal",
        name = "Doctor",
        description = "Make appointment for check up",
        isComplete = false
    }
};

Uri uri = UriFactory.CreateStoredProcedureUri("myDatabase", "myContainer", "spCreateToDoItem");
RequestOptions options = new RequestOptions { PartitionKey = new PartitionKey("Personal") };
var result = await client.ExecuteStoredProcedureAsync<string>(uri, options, new[] { newItems });
```

### <a name="stored-procedures---net-sdk-v3"></a>Хранимые процедуры — пакет SDK для .NET v3

В следующем примере показано, как зарегистрировать хранимую процедуру с помощью пакета SDK для .NET v3.

```csharp
string storedProcedureId = "spCreateToDoItems";
StoredProcedureResponse storedProcedureResponse = await client.GetContainer("myDatabase", "myContainer").Scripts.CreateStoredProcedureAsync(new StoredProcedureProperties
{
    Id = storedProcedureId,
    Body = File.ReadAllText($@"..\js\{storedProcedureId}.js")
});
```

В следующем коде показано, как вызвать хранимую процедуру с помощью пакета SDK версии 3 для .NET:

```csharp
dynamic[] newItems = new dynamic[]
{
    new {
        category = "Personal",
        name = "Groceries",
        description = "Pick up strawberries",
        isComplete = false
    },
    new {
        category = "Personal",
        name = "Doctor",
        description = "Make appointment for check up",
        isComplete = false
    }
};

var result = await client.GetContainer("database", "container").Scripts.ExecuteStoredProcedureAsync<string>("spCreateToDoItem", new PartitionKey("Personal"), new[] { newItems });
```

### <a name="stored-procedures---java-sdk"></a>Хранимые процедуры — пакет SDK для Java

Ниже показано, как правильно зарегистрировать хранимую процедуру с помощью пакета SDK для Java:

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
StoredProcedure newStoredProcedure = new StoredProcedure(
    "{" +
        "  'id':'spCreateToDoItems'," +
        "  'body':" + new String(Files.readAllBytes(Paths.get("..\\js\\spCreateToDoItems.js"))) +
    "}");
//toBlocking() blocks the thread until the operation is complete and is used only for demo.  
StoredProcedure createdStoredProcedure = asyncClient.createStoredProcedure(containerLink, newStoredProcedure, null)
    .toBlocking().single().getResource();
```

В коде ниже показано, как правильно вызывать хранимую процедуру с помощью пакета SDK для Java:

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String sprocLink = String.format("%s/sprocs/%s", containerLink, "spCreateToDoItems");
final CountDownLatch successfulCompletionLatch = new CountDownLatch(1);

List<ToDoItem> ToDoItems = new ArrayList<ToDoItem>();

class ToDoItem {
    public String category;
    public String name;
    public String description;
    public boolean isComplete;
}

ToDoItem newItem = new ToDoItem();
newItem.category = "Personal";
newItem.name = "Groceries";
newItem.description = "Pick up strawberries";
newItem.isComplete = false;

ToDoItems.add(newItem)

newItem.category = "Personal";
newItem.name = "Doctor";
newItem.description = "Make appointment for check up";
newItem.isComplete = false;

ToDoItems.add(newItem)

RequestOptions requestOptions = new RequestOptions();
requestOptions.setPartitionKey(new PartitionKey("Personal"));

Object[] storedProcedureArgs = new Object[] { ToDoItems };
asyncClient.executeStoredProcedure(sprocLink, requestOptions, storedProcedureArgs)
    .subscribe(storedProcedureResponse -> {
        String storedProcResultAsString = storedProcedureResponse.getResponseAsString();
        successfulCompletionLatch.countDown();
        System.out.println(storedProcedureResponse.getActivityId());
    }, error -> {
        successfulCompletionLatch.countDown();
        System.err.println("an error occurred while executing the stored procedure: actual cause: "
                + error.getMessage());
    });

successfulCompletionLatch.await();
```

### <a name="stored-procedures---javascript-sdk"></a>Хранимые процедуры — пакет SDK для JavaScript

Ниже показано, как правильно зарегистрировать хранимую процедуру с помощью пакета SDK для JavaScript

```javascript
const container = client.database("myDatabase").container("myContainer");
const sprocId = "spCreateToDoItems";
await container.scripts.storedProcedures.create({
    id: sprocId,
    body: require(`../js/${sprocId}`)
});
```

В коде ниже показано, как правильно вызывать хранимую процедуру с помощью пакета SDK для JavaScript:

```javascript
const newItem = [{
    category: "Personal",
    name: "Groceries",
    description: "Pick up strawberries",
    isComplete: false
}];
const container = client.database("myDatabase").container("myContainer");
const sprocId = "spCreateToDoItems";
const {body: result} = await container.scripts.storedProcedure(sprocId).execute(newItem, {partitionKey: newItem[0].category});
```

### <a name="stored-procedures---python-sdk"></a>Хранимые процедуры — пакет SDK для Python

В следующем примере показано, как зарегистрировать хранимую процедуру с помощью пакета SDK для Python:

```python
import azure.cosmos.cosmos_client as cosmos_client

url = "your_cosmos_db_account_URI"
key = "your_cosmos_db_account_key"
database_name = 'your_cosmos_db_database_name'
container_name = 'your_cosmos_db_container_name'

with open('../js/spCreateToDoItems.js') as file:
    file_contents = file.read()

sproc = {
    'id': 'spCreateToDoItem',
    'serverScript': file_contents,
}
client = cosmos_client.CosmosClient(url, key)
database = client.get_database_client(database_name)
container = database.get_container_client(container_name)
created_sproc = container.scripts.create_stored_procedure(body=sproc) 
```

В следующем коде показано, как вызвать хранимую процедуру с помощью пакета SDK для Python:

```python
import uuid

new_id= str(uuid.uuid4())

# Creating a document for a container with "id" as a partition key.

new_item =   {
      "id": new_id, 
      "category":"Personal",
      "name":"Groceries",
      "description":"Pick up strawberries",
      "isComplete":False
   }
result = container.scripts.execute_stored_procedure(sproc=created_sproc,params=[[new_item]], partition_key=new_id) 
```

## <a name="how-to-run-pre-triggers"></a><a id="pre-triggers"></a>Как выполнять предварительные триггеры

В следующих примерах показано, как регистрировать и вызывать предварительный триггер с помощью пакетов SDK для Azure Cosmos DB. Используйте раздел [Пример предварительного триггера](how-to-write-stored-procedures-triggers-udfs.md#pre-triggers) как источник для этого предварительного триггера, который сохраняется как `trgPreValidateToDoItemTimestamp.js`.

При выполнении предварительные триггеры передаются в объекте RequestOptions. Для этого нужно указать `PreTriggerInclude` и передать имя триггера в объект List.

> [!NOTE]
> Несмотря на то что имя триггера передается в виде списка, по-прежнему можно выполнить только один триггер на каждую операцию.

### <a name="pre-triggers---net-sdk-v2"></a>Предварительные триггеры — пакет SDK для .NET v2

В следующем коде показано, как зарегистрировать предварительный триггер с помощью пакета SDK для .NET версии 2.

```csharp
string triggerId = "trgPreValidateToDoItemTimestamp";
Trigger trigger = new Trigger
{
    Id =  triggerId,
    Body = File.ReadAllText($@"..\js\{triggerId}.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Pre
};
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateTriggerAsync(containerUri, trigger);
```

В следующем коде показано, как вызвать предварительный триггер с помощью пакета SDK для .NET версии 2:

```csharp
dynamic newItem = new
{
    category = "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
};

Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
RequestOptions requestOptions = new RequestOptions { PreTriggerInclude = new List<string> { "trgPreValidateToDoItemTimestamp" } };
await client.CreateDocumentAsync(containerUri, newItem, requestOptions);
```

### <a name="pre-triggers---net-sdk-v3"></a>Предварительные триггеры — пакет SDK для .NET v3

В следующем коде показано, как зарегистрировать предварительный триггер с помощью пакета SDK для .NET v3.

```csharp
await client.GetContainer("database", "container").Scripts.CreateTriggerAsync(new TriggerProperties
{
    Id = "trgPreValidateToDoItemTimestamp",
    Body = File.ReadAllText("@..\js\trgPreValidateToDoItemTimestamp.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Pre
});
```

В следующем коде показано, как вызвать предварительный триггер с помощью пакета SDK для .NET v3:

```csharp
dynamic newItem = new
{
    category = "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
};

await client.GetContainer("database", "container").CreateItemAsync(newItem, null, new ItemRequestOptions { PreTriggers = new List<string> { "trgPreValidateToDoItemTimestamp" } });
```

### <a name="pre-triggers---java-sdk"></a>Предварительные триггеры — пакет SDK для Java

Ниже показано, как зарегистрировать предварительный триггер с помощью пакета SDK для Java:

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String triggerId = "trgPreValidateToDoItemTimestamp";
Trigger trigger = new Trigger();
trigger.setId(triggerId);
trigger.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", triggerId)));
trigger.setTriggerOperation(TriggerOperation.Create);
trigger.setTriggerType(TriggerType.Pre);
//toBlocking() blocks the thread until the operation is complete and is used only for demo. 
Trigger createdTrigger = asyncClient.createTrigger(containerLink, trigger, new RequestOptions()).toBlocking().single().getResource();
```

Ниже показано, как вызывать предварительный триггер с помощью пакета SDK для Java:

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
    Document item = new Document("{ "
            + "\"category\": \"Personal\", "
            + "\"name\": \"Groceries\", "
            + "\"description\": \"Pick up strawberries\", "
            + "\"isComplete\": false, "
            + "}"
            );
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPreTriggerInclude(Arrays.asList("trgPreValidateToDoItemTimestamp"));
//toBlocking() blocks the thread until the operation is complete and is used only for demo. 
asyncClient.createDocument(containerLink, item, requestOptions, false).toBlocking();
```

### <a name="pre-triggers---javascript-sdk"></a>Предварительные триггеры — пакет SDK для JavaScript

Ниже показано, как зарегистрировать предварительный триггер с помощью пакета SDK для JavaScript:

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPreValidateToDoItemTimestamp";
await container.triggers.create({
    id: triggerId,
    body: require(`../js/${triggerId}`),
    triggerOperation: "create",
    triggerType: "pre"
});
```

Ниже показано, как вызвать предварительный триггер с помощью пакета SDK для JavaScript:

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPreValidateToDoItemTimestamp";
await container.items.create({
    category: "Personal",
    name = "Groceries",
    description = "Pick up strawberries",
    isComplete = false
}, {preTriggerInclude: [triggerId]});
```

### <a name="pre-triggers---python-sdk"></a>Предварительные триггеры — пакет SDK для Python

Ниже показано, как зарегистрировать предварительный триггер с помощью пакета SDK для Python:

```python
import azure.cosmos.cosmos_client as cosmos_client

url = "your_cosmos_db_account_URI"
key = "your_cosmos_db_account_key"
database_name = 'your_cosmos_db_database_name'
container_name = 'your_cosmos_db_container_name'

with open('../js/trgPreValidateToDoItemTimestamp.js') as file:
    file_contents = file.read()

trigger_definition = {
    'id': 'trgPreValidateToDoItemTimestamp',
    'serverScript': file_contents,
    'triggerType': documents.TriggerType.Pre,
    'triggerOperation': documents.TriggerOperation.All
}
client = cosmos_client.CosmosClient(url, key)
database = client.get_database_client(database_name)
container = database.get_container_client(container_name)
trigger = container.scripts.create_trigger(trigger_definition)
```

Ниже показано, как вызвать предварительный триггер с помощью пакета SDK для Python:

```python
item = {'category': 'Personal', 'name': 'Groceries',
        'description': 'Pick up strawberries', 'isComplete': False}
container.create_item(item, {'pre_trigger_include': 'trgPreValidateToDoItemTimestamp'})
```

## <a name="how-to-run-post-triggers"></a><a id="post-triggers"></a>Как выполнять триггеры после операции

В следующих примерах показано, как регистрировать триггер после операции с помощью пакетов SDK для Azure Cosmos DB. Используйте раздел [Пример триггера после операции](how-to-write-stored-procedures-triggers-udfs.md#post-triggers) как источник для этого триггера после операции, который сохраняется как `trgPostUpdateMetadata.js`.

### <a name="post-triggers---net-sdk-v2"></a>После триггеров — пакет SDK для .NET v2

В следующем коде показано, как зарегистрировать после триггера с помощью пакета SDK для .NET версии 2.

```csharp
string triggerId = "trgPostUpdateMetadata";
Trigger trigger = new Trigger
{
    Id = triggerId,
    Body = File.ReadAllText($@"..\js\{triggerId}.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Post
};
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateTriggerAsync(containerUri, trigger);
```

В следующем коде показано, как вызвать событие после триггера с помощью пакета SDK для .NET версии 2:

```csharp
var newItem = { 
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};

RequestOptions options = new RequestOptions { PostTriggerInclude = new List<string> { "trgPostUpdateMetadata" } };
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.createDocumentAsync(containerUri, newItem, options);
```

### <a name="post-triggers---net-sdk-v3"></a>После триггеров — пакет SDK для .NET v3

В следующем коде показано, как зарегистрировать событие после триггера с помощью пакета SDK для .NET v3.

```csharp
await client.GetContainer("database", "container").Scripts.CreateTriggerAsync(new TriggerProperties
{
    Id = "trgPostUpdateMetadata",
    Body = File.ReadAllText(@"..\js\trgPostUpdateMetadata.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Post
});
```

В следующем коде показано, как вызвать событие после триггера с помощью пакета SDK для .NET v3:

```csharp
var newItem = { 
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};

await client.GetContainer("database", "container").CreateItemAsync(newItem, null, new ItemRequestOptions { PostTriggers = new List<string> { "trgPostUpdateMetadata" } });
```

### <a name="post-triggers---java-sdk"></a>Триггеры после операции — пакет SDK для Java

Ниже показано, как зарегистрировать триггер после операции с помощью пакета SDK для Java:

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String triggerId = "trgPostUpdateMetadata";
Trigger trigger = new Trigger();
trigger.setId(triggerId);
trigger.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", triggerId)))));
trigger.setTriggerOperation(TriggerOperation.Create);
trigger.setTriggerType(TriggerType.Post);
Trigger createdTrigger = asyncClient.createTrigger(containerLink, trigger, new RequestOptions()).toBlocking().single().getResource();
```

Ниже показано, как вызывать триггер после операции с помощью пакета SDK для Java:

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
Document item = new Document(String.format("{ "
    + "\"name\": \"artist_profile_1023\", "
    + "\"artist\": \"The Band\", "
    + "\"albums\": [\"Hellujah\", \"Rotators\", \"Spinning Top\"]"
    + "}"
));
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPostTriggerInclude(Arrays.asList("trgPostUpdateMetadata"));
//toBlocking() blocks the thread until the operation is complete, and is used only for demo.
asyncClient.createDocument(containerLink, item, requestOptions, false).toBlocking();
```

### <a name="post-triggers---javascript-sdk"></a>Триггеры после операции — пакет SDK для JavaScript

Ниже показано, как зарегистрировать триггер после операции с помощью пакета SDK для JavaScript:

```javascript
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPostUpdateMetadata";
await container.triggers.create({
    id: triggerId,
    body: require(`../js/${triggerId}`),
    triggerOperation: "create",
    triggerType: "post"
});
```

Ниже показано, как вызывать триггер после операции с помощью пакета SDK для JavaScript:

```javascript
const item = {
    name: "artist_profile_1023",
    artist: "The Band",
    albums: ["Hellujah", "Rotators", "Spinning Top"]
};
const container = client.database("myDatabase").container("myContainer");
const triggerId = "trgPostUpdateMetadata";
await container.items.create(item, {postTriggerInclude: [triggerId]});
```

### <a name="post-triggers---python-sdk"></a>Триггеры после операции — пакет SDK для Python

Ниже показано, как зарегистрировать триггер после операции с помощью пакета SDK для Python:

```python
import azure.cosmos.cosmos_client as cosmos_client

url = "your_cosmos_db_account_URI"
key = "your_cosmos_db_account_key"
database_name = 'your_cosmos_db_database_name'
container_name = 'your_cosmos_db_container_name'

with open('../js/trgPostValidateToDoItemTimestamp.js') as file:
    file_contents = file.read()

trigger_definition = {
    'id': 'trgPostValidateToDoItemTimestamp',
    'serverScript': file_contents,
    'triggerType': documents.TriggerType.Post,
    'triggerOperation': documents.TriggerOperation.All
}
client = cosmos_client.CosmosClient(url, key)
database = client.get_database_client(database_name)
container = database.get_container_client(container_name)
trigger = container.scripts.create_trigger(trigger_definition)
```

Ниже показано, как вызывать триггер после операции с помощью пакета SDK для Python:

```python
item = {'category': 'Personal', 'name': 'Groceries',
        'description': 'Pick up strawberries', 'isComplete': False}
container.create_item(item, {'post_trigger_include': 'trgPreValidateToDoItemTimestamp'})
```

## <a name="how-to-work-with-user-defined-functions"></a><a id="udfs"></a>Работа с определяемыми пользователем функциями

В следующих примерах показано, как зарегистрировать определяемую пользователем функцию с помощью пакетов SDK для Azure Cosmos DB. Используйте раздел [Пример определяемой пользователем функции](how-to-write-stored-procedures-triggers-udfs.md#udfs) как источник для этого триггера после операции, который сохраняется как `udfTax.js`.

### <a name="user-defined-functions---net-sdk-v2"></a>Определяемые пользователем функции — пакет SDK для .NET v2

В следующем коде показано, как зарегистрировать определяемую пользователем функцию с помощью пакета SDK для .NET версии 2.

```csharp
string udfId = "Tax";
var udfTax = new UserDefinedFunction
{
    Id = udfId,
    Body = File.ReadAllText($@"..\js\{udfId}.js")
};

Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
await client.CreateUserDefinedFunctionAsync(containerUri, udfTax);

```

В следующем коде показано, как вызвать определяемую пользователем функцию с помощью пакета SDK для .NET версии 2:

```csharp
Uri containerUri = UriFactory.CreateDocumentCollectionUri("myDatabase", "myContainer");
var results = client.CreateDocumentQuery<dynamic>(containerUri, "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000"));

foreach (var result in results)
{
    //iterate over results
}
```

### <a name="user-defined-functions---net-sdk-v3"></a>Определяемые пользователем функции — пакет SDK для .NET v3

В следующем коде показано, как зарегистрировать определяемую пользователем функцию с помощью пакета SDK для .NET v3.

```csharp
await client.GetContainer("database", "container").Scripts.CreateUserDefinedFunctionAsync(new UserDefinedFunctionProperties
{
    Id = "Tax",
    Body = File.ReadAllText(@"..\js\Tax.js")
});
```

В следующем коде показано, как вызвать определяемую пользователем функцию с помощью пакета SDK для .NET v3:

```csharp
var iterator = client.GetContainer("database", "container").GetItemQueryIterator<dynamic>("SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000");
while (iterator.HasMoreResults)
{
    var results = await iterator.ReadNextAsync();
    foreach (var result in results)
    {
        //iterate over results
    }
}
```

### <a name="user-defined-functions---java-sdk"></a>Определяемые пользователем функции — пакет SDK для Java

Ниже показано, как зарегистрировать определяемую пользователем функцию с помощью пакета SDK для Java:

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
String udfId = "Tax";
UserDefinedFunction udf = new UserDefinedFunction();
udf.setId(udfId);
udf.setBody(new String(Files.readAllBytes(Paths.get(String.format("..\\js\\%s.js", udfId)))));
//toBlocking() blocks the thread until the operation is complete and is used only for demo.
UserDefinedFunction createdUDF = client.createUserDefinedFunction(containerLink, udf, new RequestOptions()).toBlocking().single().getResource();
```

Ниже показано, как вызывать определяемую пользователем функцию с помощью пакета SDK для Java:

```java
String containerLink = String.format("/dbs/%s/colls/%s", "myDatabase", "myContainer");
Observable<FeedResponse<Document>> queryObservable = client.queryDocuments(containerLink, "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000", new FeedOptions());
final CountDownLatch completionLatch = new CountDownLatch(1);
queryObservable.subscribe(
        queryResultPage -> {
            System.out.println("Got a page of query result with " +
                    queryResultPage.getResults().size());
        },
        // terminal error signal
        e -> {
            e.printStackTrace();
            completionLatch.countDown();
        },

        // terminal completion signal
        () -> {
            completionLatch.countDown();
        });
completionLatch.await();
```

### <a name="user-defined-functions---javascript-sdk"></a>Определяемые пользователем функции — пакет SDK для JavaScript

Ниже показано, как зарегистрировать определяемую пользователем функцию с помощью пакета SDK для JavaScript:

```javascript
const container = client.database("myDatabase").container("myContainer");
const udfId = "Tax";
await container.userDefinedFunctions.create({
    id: udfId,
    body: require(`../js/${udfId}`)
```

Ниже показано, как вызывать определяемую пользователем функцию с помощью пакета SDK для JavaScript:

```javascript
const container = client.database("myDatabase").container("myContainer");
const sql = "SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000";
const {result} = await container.items.query(sql).toArray();
```

### <a name="user-defined-functions---python-sdk"></a>Определяемые пользователем функции — пакет SDK для Python

Ниже показано, как зарегистрировать определяемую пользователем функцию с помощью пакета SDK для Python:

```python
import azure.cosmos.cosmos_client as cosmos_client

url = "your_cosmos_db_account_URI"
key = "your_cosmos_db_account_key"
database_name = 'your_cosmos_db_database_name'
container_name = 'your_cosmos_db_container_name'

with open('../js/udfTax.js') as file:
    file_contents = file.read()
udf_definition = {
    'id': 'Tax',
    'serverScript': file_contents,
}
client = cosmos_client.CosmosClient(url, key)
database = client.get_database_client(database_name)
container = database.get_container_client(container_name)
udf = container.scripts.create_user_defined_function(udf_definition)
```

Ниже показано, как вызывать определяемую пользователем функцию с помощью пакета SDK для Python:

```python
results = list(container.query_items(
    'query': 'SELECT * FROM Incomes t WHERE udf.Tax(t.income) > 20000'))
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как записать или использовать хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB, см. в статьях ниже:

- [Working with Azure Cosmos DB stored procedures, triggers, and user-defined functions (Работа с хранимыми процедурами, триггерами и определяемыми пользователем функциями в Azure Cosmos DB)](stored-procedures-triggers-udfs.md)
- [API для выполнения запросов JavaScript в Azure Cosmos DB](javascript-query-api.md)
- [Как писать хранимые процедуры, триггеры и определяемые пользователем функции в Azure Cosmos DB](how-to-write-stored-procedures-triggers-udfs.md)
- [Как записывать хранимые процедуры и триггеры в Azure Cosmos DB с помощью API запросов JavaScript](how-to-write-javascript-query-api.md)
