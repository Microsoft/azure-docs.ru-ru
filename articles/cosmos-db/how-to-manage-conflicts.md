---
title: Управление конфликтами между регионами в Azure Cosmos DB
description: Узнайте, как управлять конфликтами в Azure Cosmos DB путем создания последней или пользовательской политики разрешения конфликтов.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/11/2020
ms.author: anfeldma
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 8f98c2201159350f5774f4d2b05102384f31f3af
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339348"
---
# <a name="manage-conflict-resolution-policies-in-azure-cosmos-db"></a>Управление политиками разрешения конфликтов в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Если операции записи выполняются в нескольких регионах и несколько клиентов обращаются к одному и тому же элементу, может возникнуть конфликт. Такую проблему можно решить с помощью разных политик устранения конфликтов. В этой статье объясняется, как управлять политиками устранения конфликтов.

## <a name="create-a-last-writer-wins-conflict-resolution-policy"></a>Создание политики разрешения конфликтов, реализующей подход "Сохраняются изменения, внесенные последними"

В этих примерах показано, как настроить контейнер с политикой разрешения конфликтов, реализующей подход "Сохраняются изменения, внесенные последними". В пути по умолчанию для политики "Сохраняются изменения, внесенные последними" указывается поле метки времени или свойство `_ts`. Для API SQL также можно указать определяемый пользователем путь с использованием числового типа. При возникновении конфликта сохраняется самое большое значение. Если путь не задан или является недопустимым, по умолчанию используется значение `_ts`. Сведения о конфликтах, устраненных с помощью этой политики, не будут отображаться в веб-канале конфликтов. Эту политику поддерживают все API.

### <a name="net-sdk"></a><a id="create-custom-conflict-resolution-policy-lww-dotnet"></a>Пакет SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
DocumentCollection lwwCollection = await createClient.CreateDocumentCollectionIfNotExistsAsync(
  UriFactory.CreateDatabaseUri(this.databaseName), new DocumentCollection
  {
      Id = this.lwwCollectionName,
      ConflictResolutionPolicy = new ConflictResolutionPolicy
      {
          Mode = ConflictResolutionMode.LastWriterWins,
          ConflictResolutionPath = "/myCustomId",
      },
  });
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

```csharp
Container container = await createClient.GetDatabase(this.databaseName)
    .CreateContainerIfNotExistsAsync(new ContainerProperties(this.lwwCollectionName, "/partitionKey")
    {
        ConflictResolutionPolicy = new ConflictResolutionPolicy()
        {
            Mode = ConflictResolutionMode.LastWriterWins,
            ResolutionPath = "/myCustomId",
        }
    });
```
---

### <a name="java-v4-sdk"></a><a id="create-custom-conflict-resolution-policy-lww-javav4"></a> Пакет SDK для Java версии 4

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

   Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConflictResolutionLWWAsync)]

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

   Синхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConflictResolutionLWWSync)]

--- 

### <a name="java-v2-sdks"></a><a id="create-custom-conflict-resolution-policy-lww-javav2"></a>Пакеты SDK для Java v2

# <a name="async-java-v2-sdk"></a>[Асинхронный пакет SDK для Java v2](#tab/async)

[Асинхронный пакет SDK для Java v2](sql-api-sdk-async-java.md) (Maven [com. Microsoft. Azure:: Azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb))

```java
DocumentCollection collection = new DocumentCollection();
collection.setId(id);
ConflictResolutionPolicy policy = ConflictResolutionPolicy.createLastWriterWinsPolicy("/myCustomId");
collection.setConflictResolutionPolicy(policy);
DocumentCollection createdCollection = client.createCollection(databaseUri, collection, null).toBlocking().value();
```

# <a name="sync-java-v2-sdk"></a>[Синхронизация пакета SDK для Java v2](#tab/sync)

[Синхронизация пакета SDK для Java v2](sql-api-sdk-java.md) (Maven [com. Microsoft. Azure:: Azure-documentdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb))

```java
DocumentCollection lwwCollection = new DocumentCollection();
lwwCollection.setId(this.lwwCollectionName);
ConflictResolutionPolicy lwwPolicy = ConflictResolutionPolicy.createLastWriterWinsPolicy("/myCustomId");
lwwCollection.setConflictResolutionPolicy(lwwPolicy);
DocumentCollection createdCollection = this.tryCreateDocumentCollection(createClient, database, lwwCollection);
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="create-custom-conflict-resolution-policy-lww-javascript"></a>Пакет SDK для Node.js, JavaScript и TypeScript

```javascript
const database = client.database(this.databaseName);
const { container: lwwContainer } = await database.containers.createIfNotExists(
  {
    id: this.lwwContainerName,
    conflictResolutionPolicy: {
      mode: "LastWriterWins",
      conflictResolutionPath: "/myCustomId"
    }
  }
);
```

### <a name="python-sdk"></a><a id="create-custom-conflict-resolution-policy-lww-python"></a>Пакет SDK для Python

```python
udp_collection = {
    'id': self.udp_collection_name,
    'conflictResolutionPolicy': {
        'mode': 'LastWriterWins',
        'conflictResolutionPath': '/myCustomId'
    }
}
udp_collection = self.try_create_document_collection(
    create_client, database, udp_collection)
```

## <a name="create-a-custom-conflict-resolution-policy-using-a-stored-procedure"></a>Создание пользовательской политики устранения конфликтов с помощью хранимой процедуры

В этих примерах показано, как настроить контейнер с пользовательской политикой разрешения конфликтов, использующей хранимую процедуру для решения конфликтов. Эти конфликты не будут отображаться в веб-канале конфликтов, если в хранимой процедуре нет ошибки. После создания политики с использованием контейнера необходимо создать хранимую процедуру. Ниже приведен пример пакета SDK для .NET. Эту политику поддерживает только API Core (SQL).

### <a name="sample-custom-conflict-resolution-stored-procedure"></a>Пример пользовательской хранимой процедуры для устранения конфликтов

Пользовательские хранимые процедуры для устранения конфликтов нужно реализовать с помощью сигнатуры функции, представленной ниже. Имя функции не обязательно должно совпадать с именем, используемым при регистрации хранимой процедуры с использованием контейнера, но такой подход упрощает именование. Ниже описаны параметры, которые нужно реализовать для этой хранимой процедуры.

- **инкомингитем**: элемент, вставляемый или обновляемый при фиксации, который создает конфликты. Имеет значение NULL для операций удаления.
- **ексистингитем**: текущий зафиксированный элемент. Значение NULL не устанавливается при обновлении и устанавливается при вставке или удалении.
- a- **захоронение**: логическое значение, указывающее, конфликтует ли инкомингитем с ранее удаленным элементом. При значении true existingItem также имеет значение NULL.
- **конфликтингитемс**: массив зафиксированной версии всех элементов в контейнере, конфликтующих с ИНКОМИНГИТЕМ по идентификатору или любым другим уникальным свойствам индекса.

> [!IMPORTANT]
> Так же, как и любая хранимая процедура, пользовательская процедура разрешения конфликтов может обращаться к данным с одним ключом секции. Процедура позволяет выполнять любые операции вставки, обновления или удаления для устранения конфликтов.

В этом примере представлена хранимая процедура, которая позволяет устранять конфликты, выбирая наименьшее значение из пути `/myCustomId`.

```javascript
function resolver(incomingItem, existingItem, isTombstone, conflictingItems) {
  var collection = getContext().getCollection();

  if (!incomingItem) {
      if (existingItem) {

          collection.deleteDocument(existingItem._self, {}, function (err, responseOptions) {
              if (err) throw err;
          });
      }
  } else if (isTombstone) {
      // delete always wins.
  } else {
      if (existingItem) {
          if (incomingItem.myCustomId > existingItem.myCustomId) {
              return; // existing item wins
          }
      }

      var i;
      for (i = 0; i < conflictingItems.length; i++) {
          if (incomingItem.myCustomId > conflictingItems[i].myCustomId) {
              return; // existing conflict item wins
          }
      }

      // incoming item wins - clear conflicts and replace existing with incoming.
      tryDelete(conflictingItems, incomingItem, existingItem);
  }

  function tryDelete(documents, incoming, existing) {
      if (documents.length > 0) {
          collection.deleteDocument(documents[0]._self, {}, function (err, responseOptions) {
              if (err) throw err;

              documents.shift();
              tryDelete(documents, incoming, existing);
          });
      } else if (existing) {
          collection.replaceDocument(existing._self, incoming,
              function (err, documentCreated) {
                  if (err) throw err;
              });
      } else {
          collection.createDocument(collection.getSelfLink(), incoming,
              function (err, documentCreated) {
                  if (err) throw err;
              });
      }
  }
}
```

### <a name="net-sdk"></a><a id="create-custom-conflict-resolution-policy-stored-proc-dotnet"></a>Пакет SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
DocumentCollection udpCollection = await createClient.CreateDocumentCollectionIfNotExistsAsync(
  UriFactory.CreateDatabaseUri(this.databaseName), new DocumentCollection
  {
      Id = this.udpCollectionName,
      ConflictResolutionPolicy = new ConflictResolutionPolicy
      {
          Mode = ConflictResolutionMode.Custom,
          ConflictResolutionProcedure = string.Format("dbs/{0}/colls/{1}/sprocs/{2}", this.databaseName, this.udpCollectionName, "resolver"),
      },
  });

//Create the stored procedure
await clients[0].CreateStoredProcedureAsync(
UriFactory.CreateStoredProcedureUri(this.databaseName, this.udpCollectionName, "resolver"), new StoredProcedure
{
    Id = "resolver",
    Body = File.ReadAllText(@"resolver.js")
});
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

```csharp
Container container = await createClient.GetDatabase(this.databaseName)
    .CreateContainerIfNotExistsAsync(new ContainerProperties(this.udpCollectionName, "/partitionKey")
    {
        ConflictResolutionPolicy = new ConflictResolutionPolicy()
        {
            Mode = ConflictResolutionMode.Custom,
            ResolutionProcedure = string.Format("dbs/{0}/colls/{1}/sprocs/{2}", this.databaseName, this.udpCollectionName, "resolver")
        }
    });

await container.Scripts.CreateStoredProcedureAsync(
    new StoredProcedureProperties("resolver", File.ReadAllText(@"resolver.js"))
);
```
---

### <a name="java-v4-sdk"></a><a id="create-custom-conflict-resolution-policy-stored-proc-javav4"></a> Пакет SDK для Java версии 4

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

   Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConflictResolutionSprocAsync)]

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

   Синхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConflictResolutionSprocSync)]

--- 

### <a name="java-v2-sdks"></a><a id="create-custom-conflict-resolution-policy-stored-proc-javav2"></a>Пакеты SDK для Java v2

# <a name="async-java-v2-sdk"></a>[Асинхронный пакет SDK для Java v2](#tab/async)

[Асинхронный пакет SDK для Java v2](sql-api-sdk-async-java.md) (Maven [com. Microsoft. Azure:: Azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb))

```java
DocumentCollection collection = new DocumentCollection();
collection.setId(id);
ConflictResolutionPolicy policy = ConflictResolutionPolicy.createCustomPolicy("resolver");
collection.setConflictResolutionPolicy(policy);
DocumentCollection createdCollection = client.createCollection(databaseUri, collection, null).toBlocking().value();
```

# <a name="sync-java-v2-sdk"></a>[Синхронизация пакета SDK для Java v2](#tab/sync)

[Синхронизация пакета SDK для Java v2](sql-api-sdk-java.md) (Maven [com. Microsoft. Azure:: Azure-documentdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb))

```java
DocumentCollection udpCollection = new DocumentCollection();
udpCollection.setId(this.udpCollectionName);
ConflictResolutionPolicy udpPolicy = ConflictResolutionPolicy.createCustomPolicy(
        String.format("dbs/%s/colls/%s/sprocs/%s", this.databaseName, this.udpCollectionName, "resolver"));
udpCollection.setConflictResolutionPolicy(udpPolicy);
DocumentCollection createdCollection = this.tryCreateDocumentCollection(createClient, database, udpCollection);
```
---

После создания контейнера создайте хранимую процедуру `resolver`.

### <a name="nodejsjavascripttypescript-sdk"></a><a id="create-custom-conflict-resolution-policy-stored-proc-javascript"></a>Пакет SDK для Node.js, JavaScript и TypeScript

```javascript
const database = client.database(this.databaseName);
const { container: udpContainer } = await database.containers.createIfNotExists(
  {
    id: this.udpContainerName,
    conflictResolutionPolicy: {
      mode: "Custom",
      conflictResolutionProcedure: `dbs/${this.databaseName}/colls/${
        this.udpContainerName
      }/sprocs/resolver`
    }
  }
);
```

После создания контейнера создайте хранимую процедуру `resolver`.

### <a name="python-sdk"></a><a id="create-custom-conflict-resolution-policy-stored-proc-python"></a>Пакет SDK для Python

```python
udp_collection = {
    'id': self.udp_collection_name,
    'conflictResolutionPolicy': {
        'mode': 'Custom',
        'conflictResolutionProcedure': 'dbs/' + self.database_name + "/colls/" + self.udp_collection_name + '/sprocs/resolver'
    }
}
udp_collection = self.try_create_document_collection(
    create_client, database, udp_collection)
```

После создания контейнера создайте хранимую процедуру `resolver`.

## <a name="create-a-custom-conflict-resolution-policy"></a>Создание политики разрешения конфликтов

В этих примерах показано, как настроить контейнер с пользовательской политикой разрешения конфликтов. Эти конфликты отображаются в веб-канале конфликтов.

### <a name="net-sdk"></a><a id="create-custom-conflict-resolution-policy-dotnet"></a>Пакет SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
DocumentCollection manualCollection = await createClient.CreateDocumentCollectionIfNotExistsAsync(
  UriFactory.CreateDatabaseUri(this.databaseName), new DocumentCollection
  {
      Id = this.manualCollectionName,
      ConflictResolutionPolicy = new ConflictResolutionPolicy
      {
          Mode = ConflictResolutionMode.Custom,
      },
  });
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

```csharp
Container container = await createClient.GetDatabase(this.databaseName)
    .CreateContainerIfNotExistsAsync(new ContainerProperties(this.manualCollectionName, "/partitionKey")
    {
        ConflictResolutionPolicy = new ConflictResolutionPolicy()
        {
            Mode = ConflictResolutionMode.Custom
        }
    });
```
---

### <a name="java-v4-sdk"></a><a id="create-custom-conflict-resolution-policy-javav4"></a> Пакет SDK для Java версии 4

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

   Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConflictResolutionCustomAsync)]

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

   Синхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConflictResolutionCustomSync)]

--- 

### <a name="java-v2-sdks"></a><a id="create-custom-conflict-resolution-policy-javav2"></a>Пакеты SDK для Java v2

# <a name="async-java-v2-sdk"></a>[Асинхронный пакет SDK для Java v2](#tab/async)

[Асинхронный пакет SDK для Java v2](sql-api-sdk-async-java.md) (Maven [com. Microsoft. Azure:: Azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb))

```java
DocumentCollection collection = new DocumentCollection();
collection.setId(id);
ConflictResolutionPolicy policy = ConflictResolutionPolicy.createCustomPolicy();
collection.setConflictResolutionPolicy(policy);
DocumentCollection createdCollection = client.createCollection(databaseUri, collection, null).toBlocking().value();
```

# <a name="sync-java-v2-sdk"></a>[Синхронизация пакета SDK для Java v2](#tab/sync)

[Синхронизация пакета SDK для Java v2](sql-api-sdk-java.md) (Maven [com. Microsoft. Azure:: Azure-documentdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb))

```java
DocumentCollection manualCollection = new DocumentCollection();
manualCollection.setId(this.manualCollectionName);
ConflictResolutionPolicy customPolicy = ConflictResolutionPolicy.createCustomPolicy(null);
manualCollection.setConflictResolutionPolicy(customPolicy);
DocumentCollection createdCollection = client.createCollection(database.getSelfLink(), collection, null).getResource();
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="create-custom-conflict-resolution-policy-javascript"></a>Пакет SDK для Node.js, JavaScript и TypeScript

```javascript
const database = client.database(this.databaseName);
const {
  container: manualContainer
} = await database.containers.createIfNotExists({
  id: this.manualContainerName,
  conflictResolutionPolicy: {
    mode: "Custom"
  }
});
```

### <a name="python-sdk"></a><a id="create-custom-conflict-resolution-policy-python"></a>Пакет SDK для Python

```python
database = client.ReadDatabase("dbs/" + self.database_name)
manual_collection = {
    'id': self.manual_collection_name,
    'conflictResolutionPolicy': {
        'mode': 'Custom'
    }
}
manual_collection = client.CreateContainer(database['_self'], collection)
```

## <a name="read-from-conflict-feed"></a>Чтение из веб-канала конфликтов

В этих примерах показано, как выполнять чтение из веб-канала конфликтов контейнера. Сведения о конфликтах отображаются в веб-каналах конфликтов, только если они не устранены автоматически или если используется пользовательская политика устранения конфликтов.

### <a name="net-sdk"></a><a id="read-from-conflict-feed-dotnet"></a>Пакет SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
FeedResponse<Conflict> conflicts = await delClient.ReadConflictFeedAsync(this.collectionUri);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

```csharp
FeedIterator<ConflictProperties> conflictFeed = container.Conflicts.GetConflictQueryIterator();
while (conflictFeed.HasMoreResults)
{
    FeedResponse<ConflictProperties> conflicts = await conflictFeed.ReadNextAsync();
    foreach (ConflictProperties conflict in conflicts)
    {
        // Read the conflicted content
        MyClass intendedChanges = container.Conflicts.ReadConflictContent<MyClass>(conflict);
        MyClass currentState = await container.Conflicts.ReadCurrentAsync<MyClass>(conflict, new PartitionKey(intendedChanges.MyPartitionKey));

        // Do manual merge among documents
        await container.ReplaceItemAsync<MyClass>(intendedChanges, intendedChanges.Id, new PartitionKey(intendedChanges.MyPartitionKey));

        // Delete the conflict
        await container.Conflicts.DeleteAsync(conflict, new PartitionKey(intendedChanges.MyPartitionKey));
    }
}
```
---

### <a name="java-v2-sdks"></a><a id="read-from-conflict-feed-javav2"></a>Пакеты SDK для Java v2

# <a name="async-java-v2-sdk"></a>[Асинхронный пакет SDK для Java v2](#tab/async)

[Асинхронный пакет SDK для Java v2](sql-api-sdk-async-java.md) (Maven [com. Microsoft. Azure:: Azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb))

```java
FeedResponse<Conflict> response = client.readConflicts(this.manualCollectionUri, null)
                    .first().toBlocking().single();
for (Conflict conflict : response.getResults()) {
    /* Do something with conflict */
}
```
# <a name="sync-java-v2-sdk"></a>[Синхронизация пакета SDK для Java v2](#tab/sync)

[Синхронизация пакета SDK для Java v2](sql-api-sdk-java.md) (Maven [com. Microsoft. Azure:: Azure-documentdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-documentdb))

```java
Iterator<Conflict> conflictsIterator = client.readConflicts(this.collectionLink, null).getQueryIterator();
while (conflictsIterator.hasNext()) {
    Conflict conflict = conflictsIterator.next();
    /* Do something with conflict */
}
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="read-from-conflict-feed-javascript"></a>Пакет SDK для Node.js, JavaScript и TypeScript

```javascript
const container = client
  .database(this.databaseName)
  .container(this.lwwContainerName);

const { result: conflicts } = await container.conflicts.readAll().toArray();
```

### <a name="python"></a><a id="read-from-conflict-feed-python"></a>Python

```python
conflicts_iterator = iter(client.ReadConflicts(self.manual_collection_link))
conflict = next(conflicts_iterator, None)
while conflict:
    # Do something with conflict
    conflict = next(conflicts_iterator, None)
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о следующих понятиях Azure Cosmos DB.

- [Глобальное распределение (взгляд изнутри)](global-dist-under-the-hood.md)
- [Настройка операций записи в несколько регионов в приложениях](how-to-multi-master.md)
- [Настройка клиентов для поддержки нескольких веб-сайтов](how-to-manage-database-account.md#configure-multiple-write-regions)
- [Добавление и удаление регионов из учетной записи Azure Cosmos DB](how-to-manage-database-account.md#addremove-regions-from-your-database-account)
- [Как конфигуремулти запись в приложения](how-to-multi-master.md).
- [Секционирование и масштабирование в Azure Cosmos DB](partitioning-overview.md)
- [Индексирование в Azure Cosmos DB](index-policy.md)
