---
title: Подготавливает пропускную способность контейнера в Azure Cosmos DB API SQL
description: Узнайте, как подготавливать пропускную способность на уровне контейнера в Azure Cosmos DB API SQL с помощью портал Azure, CLI, PowerShell и различных других пакетов SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/14/2020
ms.author: mjbrown
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 4caf43cb972b44dd1482b9e6e467e41cae294708
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93100105"
---
# <a name="provision-standard-manual-throughput-on-an-azure-cosmos-container---sql-api"></a>Предоставление стандартной (ручной) пропускной способности в контейнере Azure Cosmos — API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье объясняется, как подготавливать стандартную (ручную) пропускную способность контейнера в Azure Cosmos DB API SQL. Вы можете подготовить пропускную способность для одного контейнера или [базы данных](how-to-provision-database-throughput.md) и предоставить к ней общий доступ для контейнеров в пределах этой базы данных. Пропускную способность для контейнера можно подготовить с помощью портала Azure, Azure CLI или пакетов SDK Azure Cosmos DB.

Если вы используете другой API, см. статью [API for MongoDB](how-to-provision-throughput-mongodb.md), [API Cassandra](how-to-provision-throughput-cassandra.md), [Gremlin API](how-to-provision-throughput-gremlin.md) для обеспечения пропускной способности.

## <a name="azure-portal"></a>Портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure Cosmos](create-sql-api-dotnet.md#create-account) или выберите существующую.

1. Откройте панель **Обозреватель данных** и выберите **создать контейнер**. После этого предоставьте следующие сведения.

   * Укажите, создаете ли вы новую базу данных или используете существующую.
   * Введите идентификатор контейнера.
   * Введите значение ключа секции (например, `/ItemID`).
   * Укажите подготавливаемую пропускную способность (например, 1000 ЕЗ/с).
   * Щелкните **ОК**.

    :::image type="content" source="./media/how-to-provision-container-throughput/provision-container-throughput-portal-sql-api.png" alt-text="Снимок экрана обозревателя данных с выделенным элементом &quot;Новая коллекция&quot;":::

## <a name="azure-cli-or-powershell"></a>Azure CLI или Azure PowerShell

Чтобы создать контейнер с выделенной пропускной способностью, см. разделы:

* [Создание контейнера с помощью Azure CLI](manage-with-cli.md#create-a-container)
* [Создание контейнера с помощью PowerShell](manage-with-powershell.md#create-container)

## <a name="net-sdk"></a>Пакет SDK для .NET

> [!Note]
> Используйте пакеты SDK Cosmos для API SQL, чтобы подготавливать пропускную способность для всех Cosmos DB API, за исключением API Cassandra и MongoDB.

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
// Create a container with a partition key and provision throughput of 400 RU/s
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "myContainerName";
myCollection.PartitionKey.Paths.Add("/myPartitionKey");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    myCollection,
    new RequestOptions { OfferThroughput = 400 });
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

[!code-csharp[](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos/tests/Microsoft.Azure.Cosmos.Tests/SampleCodeForDocs/ContainerDocsSampleCode.cs?name=ContainerCreateWithThroughput)]

---

## <a name="javascript-sdk"></a>Пакет SDK для JavaScript

```javascript
// Create a new Client
const client = new CosmosClient({ endpoint, key });

// Create a database
const { database } = await client.databases.createIfNotExists({ id: "databaseId" });

// Create a container with the specified throughput
const { resource } = await database.containers.createIfNotExists({
id: "containerId",
throughput: 1000
});

// To update an existing container or databases throughput, you need to user the offers API
// Get all the offers
const { resources: offers } = await client.offers.readAll().fetchAll();

// Find the offer associated with your container or the database
const offer = offers.find((_offer) => _offer.offerResourceId === resource._rid);

// Change the throughput value
offer.content.offerThroughput = 2000;

// Replace the offer.
await client.offer(offer.id).replace(offer);
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать о подготовке пропускной способности в Cosmos DB, обратитесь к следующим статьям:

* [Подготовка стандартной (масштабируемой вручную) пропускной способности в базе данных](how-to-provision-database-throughput.md)
* [Подготовка автомасштабируемой пропускной способности в базе данных](how-to-provision-autoscale-throughput.md)
* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)
