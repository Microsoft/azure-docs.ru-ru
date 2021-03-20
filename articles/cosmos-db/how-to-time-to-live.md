---
title: Настройка срока жизни в Azure Cosmos DB и управление им
description: Узнайте, как настроить срок жизни для контейнера и элемента в Azure Cosmos DB, а затем управлять им.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/11/2020
ms.author: anfeldma
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 2ddba95f9ccc25d536638dbc68c41027d26e71c7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93341014"
---
# <a name="configure-time-to-live-in-azure-cosmos-db"></a>Настройка срока жизни в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В Azure Cosmos DB вы можете настроить срок жизни (TTL) на уровне контейнера или переопределить его на уровне элемента, когда он будет настроен для контейнера. Срок жизни для контейнера можно настроить с помощью портала Azure или пакетов SDK для конкретных языков. Переопределение срока жизни на уровне элемента можно настроить с использованием пакетов SDK.

> Это содержимое связано с Azure Cosmos DB сроком жизни хранилища транзакций. Если вы ищете срок жизни хранилища аналитикал, который включает сценарии Ноетл HTAP с помощью [ссылки Azure синапсе](./synapse-link.md), щелкните [здесь](./analytical-store-introduction.md#analytical-ttl).

## <a name="enable-time-to-live-on-a-container-using-azure-portal"></a>Включение срока жизни для контейнера с помощью портала Azure

Следуйте инструкциям ниже, чтобы включить в контейнере неограниченный срок жизни. Включите этот параметр, чтобы срок жизни можно было переопределить на уровне элемента. Вы также можете задать значение срока жизни, введя ненулевое значение, обозначающее период в секундах.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Создайте новую учетную запись Azure Cosmos или выберите имеющуюся.

3. Откройте область **Data Explorer**.

4. Выберите существующий контейнер, разверните его и измените следующие значения:

   * Откройте окно **Scale & Settings** (Параметры масштабирования).
   * В разделе **Параметры** найдите **Срок жизни**.
   * Выберите **Включен (по умолчанию)** или **Включен** и задайте значение срока жизни.
   * Щелкните **Сохранить** , чтобы сохранить изменения.

   :::image type="content" source="./media/how-to-time-to-live/how-to-time-to-live-portal.png" alt-text="Настройка срока жизни на портале Azure":::

* Если параметр DefaultTimeToLive имеет значение "null", значит параметр срока жизни отключен.
* Если параметр DefaultTimeToLive имеет значение "-1", параметр срока жизни включен (без значения по умолчанию).
* Если параметр DefaultTimeToLive имеет любое другое целочисленное значение (за исключением 0), параметр срока жизни включен.

## <a name="enable-time-to-live-on-a-container-using-azure-cli-or-powershell"></a>Включение срока жизни для контейнера с помощью Azure CLI или PowerShell

Сведения о создании или включении TTL для контейнера см. в разделе.

* [Создание контейнера с TTL с помощью Azure CLI](manage-with-cli.md#create-a-container-with-ttl)
* [Создание контейнера с TTL с помощью PowerShell](manage-with-powershell.md#create-container-unique-key-ttl)

## <a name="enable-time-to-live-on-a-container-using-sdk"></a>Включение срока жизни для контейнера с помощью пакета SDK

### <a name="net-sdk"></a><a id="dotnet-enable-noexpiry"></a> ПАКЕТ SDK ДЛЯ .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

Пакет SDK для .NET версии 2 (Microsoft.Azure.DocУментдб)

```csharp
// Create a new container with TTL enabled and without any expiration value
DocumentCollection collectionDefinition = new DocumentCollection();
collectionDefinition.Id = "myContainer";
collectionDefinition.PartitionKey.Paths.Add("/myPartitionKey");
collectionDefinition.DefaultTimeToLive = -1; //(never expire by default)

DocumentCollection ttlEnabledCollection = await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    collectionDefinition);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

Пакет SDK для .NET v3 (Microsoft. Azure. Cosmos)

```csharp
// Create a new container with TTL enabled and without any expiration value
await client.GetDatabase("database").CreateContainerAsync(new ContainerProperties
{
    Id = "container",
    PartitionKeyPath = "/myPartitionKey",
    DefaultTimeToLive = -1 //(never expire by default)
});
```
---

### <a name="java-sdk"></a><a id="java-enable-noexpiry"></a> Пакет SDK для Java

# <a name="java-sdk-v4"></a>[Пакет SDK для Java v4](#tab/javav4)

Пакет SDK для Java версии 4 (Maven com.azure::azure-cosmos)

```java
CosmosAsyncContainer container;

// Create a new container with TTL enabled and without any expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.setDefaultTimeToLiveInSeconds(-1);
container = database.createContainerIfNotExists(containerProperties, 400).block().getContainer();
```

# <a name="java-sdk-v3"></a>[Пакет SDK для Java v3](#tab/javav3)

Пакет SDK для Java v3 (Maven com. Microsoft. Azure:: Azure-Cosmos)

```java
CosmosContainer container;

// Create a new container with TTL enabled and without any expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.defaultTimeToLive(-1);
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```
---

## <a name="set-time-to-live-on-a-container-using-sdk"></a>Настройка срока жизни для контейнера с помощью пакета SDK

Чтобы задать срок жизни в контейнере, необходимо ввести ненулевое положительное число, указывающее период времени в секундах. Исходя из настроенного значения срока жизни, все элементы в контейнере после последнего изменения метки времени элемента `_ts` удаляются. При необходимости можно задать параметр `TimeToLivePropertyPath` , который будет использовать другое свойство вместо свойства, созданного системой, `_ts` чтобы определить, какие элементы следует удалить в зависимости от срока жизни.

### <a name="net-sdk"></a><a id="dotnet-enable-withexpiry"></a> ПАКЕТ SDK ДЛЯ .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

Пакет SDK для .NET версии 2 (Microsoft.Azure.DocУментдб)

```csharp
// Create a new container with TTL enabled and a 90 day expiration
DocumentCollection collectionDefinition = new DocumentCollection();
collectionDefinition.Id = "myContainer";
collectionDefinition.PartitionKey.Paths.Add("/myPartitionKey");
collectionDefinition.DefaultTimeToLive = 90 * 60 * 60 * 24 // expire all documents after 90 days

DocumentCollection ttlEnabledCollection = await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    collectionDefinition;
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

Пакет SDK для .NET v3 (Microsoft. Azure. Cosmos)

```csharp
// Create a new container with TTL enabled and a 90 day expiration
await client.GetDatabase("database").CreateContainerAsync(new ContainerProperties
{
    Id = "container",
    PartitionKeyPath = "/myPartitionKey",
    DefaultTimeToLive = 90 * 60 * 60 * 24 // expire all documents after 90 days
});
```
---

### <a name="java-sdk"></a><a id="java-enable-defaultexpiry"></a> Пакет SDK для Java

# <a name="java-sdk-v4"></a>[Пакет SDK для Java v4](#tab/javav4)

Пакет SDK для Java версии 4 (Maven com.azure::azure-cosmos)

```java
CosmosAsyncContainer container;

// Create a new container with TTL enabled with default expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.setDefaultTimeToLiveInSeconds(90 * 60 * 60 * 24);
container = database.createContainerIfNotExists(containerProperties, 400).block().getContainer();
```

# <a name="java-sdk-v3"></a>[Пакет SDK для Java v3](#tab/javav3)

Пакет SDK для Java v3 (Maven com. Microsoft. Azure:: Azure-Cosmos)

```java
CosmosContainer container;

// Create a new container with TTL enabled with default expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
containerProperties.defaultTimeToLive(90 * 60 * 60 * 24);
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```
---

### <a name="nodejs-sdk"></a><a id="nodejs-enable-withexpiry"></a>Пакет SDK для NodeJS

```javascript
const containerDefinition = {
          id: "sample container1",
        };

async function createcontainerWithTTL(db: Database, containerDefinition: ContainerDefinition, collId: any, defaultTtl: number) {
      containerDefinition.id = collId;
      containerDefinition.defaultTtl = defaultTtl;
      await db.containers.create(containerDefinition);
}
```

## <a name="set-time-to-live-on-an-item"></a>Настройка значения срока жизни для элемента

Срок жизни можно задать не только для контейнера, но и для элемента. Настройка срока жизни на уровне элемента переопределит значение срока жизни по умолчанию для элемента в этом контейнере.

* Чтобы задать срок жизни для элемента, необходимо ввести ненулевое положительное число, которое будет означать время в секундах, по истечении которого с момента последнего изменения элемента (метка времени `_ts`) этот элемент будет считаться устаревшим.

* Если у элемента нет поля срока жизни, то по умолчанию к элементу будет применяться срок жизни, заданный для контейнера.

* Если свойство TTL отключено на уровне контейнера, поле срока жизни в элементе будет игнорироваться, пока свойство не будет включено для этого контейнера.

### <a name="azure-portal"></a><a id="portal-set-ttl-item"></a>Портал Azure

Следуйте инструкциям ниже, чтобы включить в элементе срок жизни.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Создайте новую учетную запись Azure Cosmos или выберите имеющуюся.

3. Откройте область **Data Explorer**.

4. Выберите существующий контейнер, разверните его и измените следующие значения:

   * Откройте окно **Scale & Settings** (Параметры масштабирования).
   * В разделе **Параметры** найдите **Срок жизни**.
   * Выберите **вкл. (нет значения по умолчанию)** **, а затем** установите значение TTL. 
   * **Сохраните** изменения.

5. Затем перейдите к элементу, для которого нужно установить срок жизни, добавьте свойство `ttl` и выберите **Обновить**. 

   ```json
   {
    "id": "1",
    "_rid": "Jic9ANWdO-EFAAAAAAAAAA==",
    "_self": "dbs/Jic9AA==/colls/Jic9ANWdO-E=/docs/Jic9ANWdO-EFAAAAAAAAAA==/",
    "_etag": "\"0d00b23f-0000-0000-0000-5c7712e80000\"",
    "_attachments": "attachments/",
    "ttl": 10,
    "_ts": 1551307496
   }
   ```

### <a name="net-sdk-any"></a><a id="dotnet-set-ttl-item"></a>Пакет SDK для .NET (любой)

```csharp
// Include a property that serializes to "ttl" in JSON
public class SalesOrder
{
    [JsonProperty(PropertyName = "id")]
    public string Id { get; set; }
    [JsonProperty(PropertyName="cid")]
    public string CustomerId { get; set; }
    // used to set expiration policy
    [JsonProperty(PropertyName = "ttl", NullValueHandling = NullValueHandling.Ignore)]
    public int? ttl { get; set; }

    //...
}
// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder
{
    Id = "SO05",
    CustomerId = "CO18009186470",
    ttl = 60 * 60 * 24 * 30;  // Expire sales orders in 30 days
};
```

### <a name="nodejs-sdk"></a><a id="nodejs-set-ttl-item"></a>Пакет SDK для NodeJS

```javascript
const itemDefinition = {
          id: "doc",
          name: "sample Item",
          key: "value",
          ttl: 2
        };
```

### <a name="java-sdk"></a><a id="java-set-ttl-item"></a> Пакет SDK для Java

# <a name="java-sdk-v4"></a>[Пакет SDK для Java v4](#tab/javav4)

Пакет SDK для Java версии 4 (Maven com.azure::azure-cosmos)

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

    public String getId() {return this.id;}
    public void setId(String new_id) {this.id = new_id;}
    public String getCustomerId() {return this.customerId;}
    public void setCustomerId(String new_cid) {this.customerId = new_cid;}
    public Integer getTtl() {return this.ttl;}
    public void setTtl(Integer new_ttl) {this.ttl = new_ttl;}

    //...
}

// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder(
    "SO05",
    "CO18009186470",
    60 * 60 * 24 * 30  // Expire sales orders in 30 days
);

```

# <a name="java-sdk-v3"></a>[Пакет SDK для Java v3](#tab/javav3)

Пакет SDK для Java v3 (Maven com. Microsoft. Azure:: Azure-Cosmos)

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
    public void id(String new_id) {this.id = new_id;}
    public String customerId() {return this.customerId;}
    public void customerId(String new_cid) {this.customerId = new_cid;}
    public Integer ttl() {return this.ttl;}
    public void ttl(Integer new_ttl) {this.ttl = new_ttl;}

    //...
}

// Set the value to the expiration in seconds
SalesOrder salesOrder = new SalesOrder(
    "SO05",
    "CO18009186470",
    60 * 60 * 24 * 30  // Expire sales orders in 30 days
);

```
---

## <a name="reset-time-to-live"></a>Сброс срока жизни

Вы можете сбросить срок жизни в элементе, выполнив операции записи или обновления в элементе. Операция записи или обновления установит для `_ts` текущий момент времени, и срок жизни для элемента начнет истекать заново. Если вы хотите изменить срок жизни элемента, это поле можно обновить так же, как любое другое.

### <a name="net-sdk"></a><a id="dotnet-extend-ttl-item"></a> ПАКЕТ SDK ДЛЯ .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

Пакет SDK для .NET версии 2 (Microsoft.Azure.DocУментдб)

```csharp
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
response = await client.ReadDocumentAsync(
    "/dbs/salesdb/colls/orders/docs/SO05"),
    new RequestOptions { PartitionKey = new PartitionKey("CO18009186470") });

Document readDocument = response.Resource;
readDocument.ttl = 60 * 30 * 30; // update time to live
response = await client.ReplaceDocumentAsync(readDocument);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

Пакет SDK для .NET v3 (Microsoft. Azure. Cosmos)

```csharp
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
ItemResponse<SalesOrder> itemResponse = await client.GetContainer("database", "container").ReadItemAsync<SalesOrder>("SO05", new PartitionKey("CO18009186470"));

itemResponse.Resource.ttl = 60 * 30 * 30; // update time to live
await client.GetContainer("database", "container").ReplaceItemAsync(itemResponse.Resource, "SO05");
```
---

### <a name="java-sdk"></a><a id="java-enable-modifyitemexpiry"></a> Пакет SDK для Java

# <a name="java-sdk-v4"></a>[Пакет SDK для Java v4](#tab/javav4)

Пакет SDK для Java версии 4 (Maven com.azure::azure-cosmos)

```java
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
CosmosAsyncItemResponse<SalesOrder> itemResponse = container.readItem("SO05", new PartitionKey("CO18009186470"), SalesOrder.class)
        .flatMap(readResponse -> {
            SalesOrder salesOrder = readResponse.getItem();
            salesOrder.setTtl(60 * 30 * 30);
            return container.createItem(salesOrder);
}).block();
```

# <a name="java-sdk-v3"></a>[Пакет SDK для Java v3](#tab/javav3)

Пакет SDK v3 (Maven com. Microsoft. Azure:: Azure-Cosmos)

```java
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
container.getItem("SO05", new PartitionKey("CO18009186470")).read()
        .flatMap(readResponse -> {
            SalesOrder salesOrder = null;
            try {
                salesOrder = readResponse.properties().getObject(SalesOrder.class);
            } catch (Exception err) {

            }
            salesOrder.ttl(60 * 30 * 30);
            return container.createItem(salesOrder);
}).block();
```
---

## <a name="turn-off-time-to-live"></a>Выключение срока жизни

Если для элемента был установлен срок жизни, но вы больше не хотите, чтобы он истек, то вы можете получить элемент, удалить поле срока жизни и заменить элемент на сервере. Когда поле срока жизни удаляется из элемента, к элементу применяется значение срока жизни по умолчанию, назначенное для контейнера. Установите значение срока жизни, равное -1, чтобы предотвратить истечение срока действия элемента и не наследовать значение срока жизни из контейнера.

### <a name="net-sdk"></a><a id="dotnet-turn-off-ttl-item"></a> ПАКЕТ SDK ДЛЯ .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

Пакет SDK для .NET версии 2 (Microsoft.Azure.DocУментдб)

```csharp
// This examples leverages the Sales Order class above.
// Read a document, turn off its override TTL, save it.
response = await client.ReadDocumentAsync(
    "/dbs/salesdb/colls/orders/docs/SO05"),
    new RequestOptions { PartitionKey = new PartitionKey("CO18009186470") });

Document readDocument = response.Resource;
readDocument.ttl = null; // inherit the default TTL of the container

response = await client.ReplaceDocumentAsync(readDocument);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

Пакет SDK для .NET v3 (Microsoft. Azure. Cosmos)

```csharp
// This examples leverages the Sales Order class above.
// Read a document, turn off its override TTL, save it.
ItemResponse<SalesOrder> itemResponse = await client.GetContainer("database", "container").ReadItemAsync<SalesOrder>("SO05", new PartitionKey("CO18009186470"));

itemResponse.Resource.ttl = null; // inherit the default TTL of the container
await client.GetContainer("database", "container").ReplaceItemAsync(itemResponse.Resource, "SO05");
```
---

### <a name="java-sdk"></a><a id="java-enable-itemdefaultexpiry"></a> Пакет SDK для Java

# <a name="java-sdk-v4"></a>[Пакет SDK для Java v4](#tab/javav4)

Пакет SDK для Java версии 4 (Maven com.azure::azure-cosmos)

```java
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
CosmosAsyncItemResponse<SalesOrder> itemResponse = container.readItem("SO05", new PartitionKey("CO18009186470"), SalesOrder.class)
        .flatMap(readResponse -> {
            SalesOrder salesOrder = readResponse.getItem();
            salesOrder.setTtl(null);
            return container.createItem(salesOrder);
}).block();
```

# <a name="java-sdk-v3"></a>[Пакет SDK для Java v3](#tab/javav3)

Пакет SDK для Java v3 (Maven com. Microsoft. Azure:: Azure-Cosmos)

```java
// This examples leverages the Sales Order class above.
// Read a document, update its TTL, save it.
container.getItem("SO05", new PartitionKey("CO18009186470")).read()
        .flatMap(readResponse -> {
            SalesOrder salesOrder = null;
            try {
                salesOrder = readResponse.properties().getObject(SalesOrder.class);
            } catch (Exception err) {

            }
            salesOrder.ttl(null);
            return container.createItem(salesOrder);
}).block();
```
---

## <a name="disable-time-to-live"></a>Отключение срока жизни

Чтобы отключить срок жизни в контейнере и остановить фоновый процесс проверки на наличие просроченных элементов, нужно удалить свойство `DefaultTimeToLive` контейнера. Удаление этого свойства и выбор значения -1 имеют разный эффект. Если задано значение -1, срок действия новых элементов, добавляемых в контейнер, не будет истекать, однако это значение можно переопределить для определенных элементов в контейнере. При удалении свойства срока жизни из контейнера элементы будут устаревать, даже если их значение явно переопределило предыдущее значение срока жизни по умолчанию.

### <a name="net-sdk"></a><a id="dotnet-disable-ttl"></a> ПАКЕТ SDK ДЛЯ .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

Пакет SDK для .NET версии 2 (Microsoft.Azure.DocУментдб)

```csharp
// Get the container, update DefaultTimeToLive to null
DocumentCollection collection = await client.ReadDocumentCollectionAsync("/dbs/salesdb/colls/orders");
// Disable TTL
collection.DefaultTimeToLive = null;
await client.ReplaceDocumentCollectionAsync(collection);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

Пакет SDK для .NET v3 (Microsoft. Azure. Cosmos)

```csharp
// Get the container, update DefaultTimeToLive to null
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Disable TTL
containerResponse.Resource.DefaultTimeToLive = null;
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```
---

### <a name="java-sdk"></a><a id="java-enable-disableexpiry"></a> Пакет SDK для Java

# <a name="java-sdk-v4"></a>[Пакет SDK для Java v4](#tab/javav4)

Пакет SDK для Java версии 4 (Maven com.azure::azure-cosmos)

```java
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
// Disable TTL
containerProperties.setDefaultTimeToLiveInSeconds(null);
// Update container settings
container.replace(containerProperties).block();
```

# <a name="java-sdk-v3"></a>[Пакет SDK для Java v3](#tab/javav3)

Пакет SDK для Java v3 (Maven com. Microsoft. Azure:: Azure-Cosmos)

```java
CosmosContainer container;

// Create a new container with TTL enabled and without any expiration value
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");
// Disable TTL
containerProperties.defaultTimeToLive(null);
// Update container settings
container = database.createContainerIfNotExists(containerProperties, 400).block().container();
```
---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о сроке жизни см. в следующей статье:

* [Срок жизни](time-to-live.md)