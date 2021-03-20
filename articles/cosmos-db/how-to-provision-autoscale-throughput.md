---
title: Подготавливает пропускную способность автомасштабирования в API Azure Cosmos DB SQL
description: Узнайте, как подготавливать пропускную способность автомасштабирования на уровне контейнера и базы данных в Azure Cosmos DB API SQL с помощью портал Azure, CLI, PowerShell и различных других пакетов SDK.
author: deborahc
ms.author: dech
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/15/2020
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 52904296df77d9097a6180345388e8e702e2bca0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97357637"
---
# <a name="provision-autoscale-throughput-on-database-or-container-in-azure-cosmos-db---sql-api"></a>Подготавливает пропускную способность автомасштабирования для базы данных или контейнера в Azure Cosmos DB API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье объясняется, как подготавливать пропускную способность автомасштабирования для базы данных или контейнера (коллекции, графа или таблицы) в Azure Cosmos DB API SQL. Вы можете включить подготовку пропускной способности для одного контейнера или подготовку автомасштабируемой пропускной способности для целой базы данных, распределив ее между контейнерами в этой базе данных.

Если вы используете другой API, см. статью [API for MongoDB](how-to-provision-throughput-mongodb.md), [API Cassandra](how-to-provision-throughput-cassandra.md), [Gremlin API](how-to-provision-throughput-gremlin.md) для обеспечения пропускной способности.

## <a name="azure-portal"></a>Портал Azure

### <a name="create-new-database-or-container-with-autoscale"></a>Создание новой базы данных или контейнера с автомасштабированием

1. Войдите на [портал Azure](https://portal.azure.com) или в [обозреватель Azure Cosmos DB](https://cosmos.azure.com/).

1. Перейдите к своей учетной записи Azure Cosmos DB и откройте вкладку **Data Explorer**.

1. Выберите **Новый контейнер**. Введите имя базы данных, контейнера и ключа раздела. В разделе **Пропускная способность** выберите параметр **Автомасштабирование** и задайте значение [максимальной пропускной способности (единиц запросов в секунду)](provision-throughput-autoscale.md#how-autoscale-provisioned-throughput-works), которое будет использоваться для масштабирования базы данных или контейнера.

   :::image type="content" source="./media/how-to-provision-autoscale-throughput/create-new-autoscale-container.png" alt-text="Создание контейнера и настройка пропускной способности, подготовленной с использованием автомасштабирования":::

1. Щелкните **ОК**.

Чтобы подготовить автомасштабирование для базы данных с совместно используемой пропускной способностью, выберите параметр **Подготовка пропускной способности базы данных** при создании новой базы данных. 

### <a name="enable-autoscale-on-existing-database-or-container"></a>Включение автомасштабирования в уже существующей базе данных или контейнере

> [!IMPORTANT]
> В текущем выпуске портал Azure является единственным способом перехода от автомасштабирования к стандартной (вручную) подготовке пропускной способности. 

1. Войдите на [портал Azure](https://portal.azure.com) или в [обозреватель Azure Cosmos DB](https://cosmos.azure.com/).

1. Перейдите к своей учетной записи Azure Cosmos DB и откройте вкладку **Data Explorer**.

1. Выберите **Масштаб и параметры** для контейнера или **Масштаб** для базы данных.

1. В разделе **Масштаб** выберите параметр **Автомасштабирование** и нажмите **Сохранить**.

   :::image type="content" source="./media/how-to-provision-autoscale-throughput/autoscale-scale-and-settings.png" alt-text="Включение автомасштабирования для существующего контейнера":::

> [!NOTE]
> При включении автомасштабирования для существующей базы данных или контейнера начальное значение максимального числа единиц запросов в секунду определяется системой с учетом текущих параметров пропускной способности и хранилища, подготовленных вручную. По завершении операции при необходимости можно изменить значение максимального числа единиц запросов в секунду. [Подробнее.](autoscale-faq.md#how-does-the-migration-between-autoscale-and-standard-manual-provisioned-throughput-work) 

## <a name="azure-cosmos-db-net-v3-sdk"></a>Пакет SDK для .NET v3 Azure Cosmos DB

Для управления ресурсами автомасштабирования используется пакет SDK .NET Azure Cosmos DB для API SQL [версии 3.9 или более поздней](https://www.nuget.org/packages/Microsoft.Azure.Cosmos). 

> [!IMPORTANT]
> Пакет SDK .NET можно использовать для создания новых ресурсов автомасштабирования. Пакет SDK не поддерживает миграцию между автомасштабируемой и стандартной (вручную) пропускной способностью. Сейчас этот сценарий миграции поддерживается только на портале Azure. 

### <a name="create-database-with-shared-throughput"></a>Создание базы данных с совместно используемой пропускной способностью

```csharp
// Create instance of CosmosClient
CosmosClient cosmosClient = new CosmosClient(Endpoint, PrimaryKey);
 
// Autoscale throughput settings
ThroughputProperties autoscaleThroughputProperties = ThroughputProperties.CreateAutoscaleThroughput(4000); //Set autoscale max RU/s

//Create the database with autoscale enabled
database = await cosmosClient.CreateDatabaseAsync(DatabaseName, throughputProperties: autoscaleThroughputProperties);
```

### <a name="create-container-with-dedicated-throughput"></a>Создание контейнеров с выделенной пропускной способностью

```csharp
// Get reference to database that container will be created in
Database database = await cosmosClient.GetDatabase("DatabaseName");

// Container and autoscale throughput settings
ContainerProperties autoscaleContainerProperties = new ContainerProperties("ContainerName", "/partitionKey");
ThroughputProperties autoscaleThroughputProperties = ThroughputProperties.CreateAutoscaleThroughput(4000); //Set autoscale max RU/s

// Create the container with autoscale enabled
container = await database.CreateContainerAsync(autoscaleContainerProperties, autoscaleThroughputProperties);
```

### <a name="read-the-current-throughput-rus"></a>Чтение текущей пропускной способности (единиц запросов в секунду)

```csharp
// Get a reference to the resource
Container container = cosmosClient.GetDatabase("DatabaseName").GetContainer("ContainerName");

// Read the throughput on a resource
ThroughputProperties autoscaleContainerThroughput = await container.ReadThroughputAsync(requestOptions: null); 

// The autoscale max throughput (RU/s) of the resource
int? autoscaleMaxThroughput = autoscaleContainerThroughput.AutoscaleMaxThroughput;

// The throughput (RU/s) the resource is currently scaled to
int? currentThroughput = autoscaleContainerThroughput.Throughput;
```

### <a name="change-the-autoscale-max-throughput-rus"></a>Изменение максимальной автомасштабируемой пропускной способности (единиц запросов в секунду)

```csharp
// Change the autoscale max throughput (RU/s)
await container.ReplaceThroughputAsync(ThroughputProperties.CreateAutoscaleThroughput(newAutoscaleMaxThroughput));
```

## <a name="azure-cosmos-db-java-v4-sdk"></a>Пакет SDK для Java версии 4 Azure Cosmos DB

Для управления ресурсами автомасштабирования используется пакет SDK Java Azure Cosmos DB для API SQL [версии 4.0 или более поздней](https://mvnrepository.com/artifact/com.azure/azure-cosmos).

> [!IMPORTANT]
> Пакет SDK Java можно использовать для создания новых ресурсов автомасштабирования. Пакет SDK не поддерживает миграцию между автомасштабируемой и стандартной (вручную) пропускной способностью. Сейчас этот сценарий миграции поддерживается только на портале Azure.

### <a name="create-database-with-shared-throughput"></a>Создание базы данных с совместно используемой пропускной способностью

#### <a name="async"></a>[Асинхронный режим](#tab/api-async)

```java
// Create instance of CosmosClient
CosmosAsyncClient client = new CosmosClientBuilder()
    .setEndpoint(HOST)
    .setKey(PRIMARYKEY)
    .setConnectionPolicy(CONNECTIONPOLICY)
    .buildAsyncClient();

// Autoscale throughput settings
ThroughputProperties autoscaleThroughputProperties = ThroughputProperties.createAutoscaledThroughput(4000); //Set autoscale max RU/s

//Create the database with autoscale enabled
CosmosAsyncDatabase database = client.createDatabase(databaseName, autoscaleThroughputProperties).block().getDatabase();
```

#### <a name="sync"></a>[Синхронизация](#tab/api-sync)

```java
// Create instance of CosmosClient
CosmosClient client = new CosmosClientBuilder()
    .setEndpoint(HOST)
    .setKey(PRIMARYKEY)
    .setConnectionPolicy(CONNECTIONPOLICY)
    .buildClient();

// Autoscale throughput settings
ThroughputProperties autoscaleThroughputProperties = ThroughputProperties.createAutoscaledThroughput(4000); //Set autoscale max RU/s

//Create the database with autoscale enabled
CosmosDatabase database = client.createDatabase(databaseName, autoscaleThroughputProperties).getDatabase();
```

--- 

### <a name="create-container-with-dedicated-throughput"></a>Создание контейнеров с выделенной пропускной способностью

#### <a name="async"></a>[Асинхронный режим](#tab/api-async)

```java
// Get reference to database that container will be created in
CosmosAsyncDatabase database = client.createDatabase("DatabaseName").block().getDatabase();

// Container and autoscale throughput settings
CosmosContainerProperties autoscaleContainerProperties = new CosmosContainerProperties("ContainerName", "/partitionKey");
ThroughputProperties autoscaleThroughputProperties = ThroughputProperties.createAutoscaledThroughput(4000); //Set autoscale max RU/s

// Create the container with autoscale enabled
CosmosAsyncContainer container = database.createContainer(autoscaleContainerProperties, autoscaleThroughputProperties, new CosmosContainerRequestOptions())
                                .block()
                                .getContainer();
```

#### <a name="sync"></a>[Синхронизация](#tab/api-sync)

```java
// Get reference to database that container will be created in
CosmosDatabase database = client.createDatabase("DatabaseName").getDatabase();

// Container and autoscale throughput settings
CosmosContainerProperties autoscaleContainerProperties = new CosmosContainerProperties("ContainerName", "/partitionKey");
ThroughputProperties autoscaleThroughputProperties = ThroughputProperties.createAutoscaledThroughput(4000); //Set autoscale max RU/s

// Create the container with autoscale enabled
CosmosContainer container = database.createContainer(autoscaleContainerProperties, autoscaleThroughputProperties, new CosmosContainerRequestOptions())
                                .getContainer();
```

--- 

### <a name="read-the-current-throughput-rus"></a>Чтение текущей пропускной способности (единиц запросов в секунду)

#### <a name="async"></a>[Асинхронный режим](#tab/api-async)

```java
// Get a reference to the resource
CosmosAsyncContainer container = client.getDatabase("DatabaseName").getContainer("ContainerName");

// Read the throughput on a resource
ThroughputProperties autoscaleContainerThroughput = container.readThroughput().block().getProperties();

// The autoscale max throughput (RU/s) of the resource
int autoscaleMaxThroughput = autoscaleContainerThroughput.getAutoscaleMaxThroughput();

// The throughput (RU/s) the resource is currently scaled to
int currentThroughput = autoscaleContainerThroughput.Throughput;
```

#### <a name="sync"></a>[Синхронизация](#tab/api-sync)

```java
// Get a reference to the resource
CosmosContainer container = client.getDatabase("DatabaseName").getContainer("ContainerName");

// Read the throughput on a resource
ThroughputProperties autoscaleContainerThroughput = container.readThroughput().getProperties();

// The autoscale max throughput (RU/s) of the resource
int autoscaleMaxThroughput = autoscaleContainerThroughput.getAutoscaleMaxThroughput();

// The throughput (RU/s) the resource is currently scaled to
int currentThroughput = autoscaleContainerThroughput.Throughput;
```

--- 

### <a name="change-the-autoscale-max-throughput-rus"></a>Изменение максимальной автомасштабируемой пропускной способности (единиц запросов в секунду)

#### <a name="async"></a>[Асинхронный режим](#tab/api-async)

```java
// Change the autoscale max throughput (RU/s)
container.replaceThroughput(ThroughputProperties.createAutoscaledThroughput(newAutoscaleMaxThroughput)).block();
```

#### <a name="sync"></a>[Синхронизация](#tab/api-sync)

```java
// Change the autoscale max throughput (RU/s)
container.replaceThroughput(ThroughputProperties.createAutoscaledThroughput(newAutoscaleMaxThroughput));
```

---

## <a name="azure-resource-manager"></a>Azure Resource Manager

Azure Resource Manager шаблоны можно использовать для инициализации пропускной способности автомасштабирования для ресурсов базы данных или уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. [в разделе Azure Resource Manager Templates for Azure Cosmos DB](./templates-samples-sql.md) .

## <a name="azure-cli"></a>Azure CLI

Azure CLI можно использовать для инициализации пропускной способности автомасштабирования в базе данных или ресурсах уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. в разделе [Azure CLI Samples for Azure Cosmos DB](cli-samples.md).

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell можно использовать для инициализации пропускной способности автомасштабирования в базе данных или ресурсах уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. в разделе [Azure PowerShell Samples for Azure Cosmos DB](powershell-samples.md).

## <a name="next-steps"></a>Дальнейшие действия

* Подробнее о [преимуществах подготовленной пропускной способности с поддержкой автомасштабирования](provision-throughput-autoscale.md#benefits-of-autoscale).
* Узнайте, как [выбрать автомасштабируемую или масштабируемую вручную пропускную способность](how-to-choose-offer.md).
* Изучите[часто задаваемые вопросы об автомасштабировании](autoscale-faq.md).
