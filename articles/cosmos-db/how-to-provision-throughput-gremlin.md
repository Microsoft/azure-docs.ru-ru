---
title: Подготавливает пропускную способность для ресурсов API Azure Cosmos DB Gremlin
description: Узнайте, как подготавливать пропускную способность контейнера, базы данных и автомасштабирования в Azure Cosmos DB ресурсов API Gremlin. Вы будете использовать портал Azure, CLI, PowerShell и различные другие пакеты SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mjbrown
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 3c2af7f33135a8c6621db233451231ffa89c2d64
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93086165"
---
# <a name="provision-database-container-or-autoscale-throughput-on-azure-cosmos-db-gremlin-api-resources"></a>Подготавливает пропускную способность для базы данных, контейнера или автомасштабирования в Azure Cosmos DB ресурсов API Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

В этой статье объясняется, как подготавливать пропускную способность в Azure Cosmos DB API Gremlin. Можно подготавливать стандартные (ручные) или автомасштабированные пропускные способности в контейнере или базе данных и совместно использовать их в контейнерах базы данных. Пропускную способность можно подготавливать с помощью портал Azure, Azure CLI или пакетов SDK для Azure Cosmos DB.

Если вы используете другой API, см. статью [SQL API](how-to-provision-container-throughput.md), [API Cassandra](how-to-provision-throughput-cassandra.md), [API для MongoDB](how-to-provision-throughput-mongodb.md) , чтобы обеспечить пропускную способность.

## <a name="azure-portal"></a><a id="portal-gremlin"></a> портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) или выберите существующую.

1. Откройте панель **Обозреватель данных** и выберите **создать граф**. После этого предоставьте следующие сведения.

   * Укажите, создаете ли вы новую базу данных или используете существующую. Выберите параметр настроить **пропускную способность базы данных** , если требуется обеспечить пропускную способность на уровне базы данных.
   * Введите идентификатор графа.
   * Введите значение ключа секции (например, `/ItemID`).
   * Укажите подготавливаемую пропускную способность (например, 1000 ЕЗ/с).
   * Щелкните **ОК**.

    :::image type="content" source="./media/how-to-provision-throughput-gremlin/provision-database-throughput-portal-gremlin-api.png" alt-text="Снимок экрана обозреватель данных при создании нового графа с пропускной способностью уровня базы данных":::

## <a name="net-sdk"></a>Пакет SDK для .NET

> [!Note]
> Используйте пакеты SDK Cosmos для API SQL, чтобы подготавливать пропускную способность для всех Azure Cosmos DB API, за исключением API Cassandra и MongoDB.

### <a name="provision-container-level-throughput"></a>Подготавливает пропускную способность на уровне контейнера

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

### <a name="provision-database-level-throughput"></a>Подготавливает пропускную способность на уровне базы данных

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
//set the throughput for the database
RequestOptions options = new RequestOptions
{
    OfferThroughput = 500
};

//create the database
await client.CreateDatabaseIfNotExistsAsync(
    new Database {Id = databaseName},  
    options);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

[!code-csharp[](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos/tests/Microsoft.Azure.Cosmos.Tests/SampleCodeForDocs/DatabaseDocsSampleCode.cs?name=DatabaseCreateWithThroughput)]

---

## <a name="azure-resource-manager"></a>Azure Resource Manager

Azure Resource Manager шаблоны можно использовать для инициализации пропускной способности автомасштабирования для ресурсов базы данных или уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. [в разделе Azure Resource Manager Templates for Azure Cosmos DB](templates-samples-gremlin.md) .

## <a name="azure-cli"></a>Azure CLI

Azure CLI можно использовать для инициализации пропускной способности автомасштабирования в базе данных или ресурсах уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. в разделе [Azure CLI Samples for Azure Cosmos DB](cli-samples-gremlin.md).

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell можно использовать для инициализации пропускной способности автомасштабирования в базе данных или ресурсах уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. в разделе [Azure PowerShell Samples for Azure Cosmos DB](powershell-samples-gremlin.md).

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать о подготовке пропускной способности в Cosmos DB, обратитесь к следующим статьям:

* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)