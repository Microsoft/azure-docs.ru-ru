---
title: Подготавливает пропускную Azure Cosmos DB способность для ресурсов MongoDB API
description: Узнайте, как подготавливать пропускную способность контейнера, базы данных и автомасштабирования в Azure Cosmos DB API для ресурсов MongoDB. Вы будете использовать портал Azure, CLI, PowerShell и различные другие пакеты SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mjbrown
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 25eaa4a66fb4a73f976edbd30e6f82015ce84f6f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93086131"
---
# <a name="provision-database-container-or-autoscale-throughput-on-azure-cosmos-db-api-for-mongodb-resources"></a>Подготавливает пропускную способность для базы данных, контейнера или автомасштабирования в Azure Cosmos DB API для ресурсов MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

В этой статье объясняется, как подготавливать пропускную способность в Azure Cosmos DB API для MongoDB. Можно подготавливать стандартные (ручные) или автомасштабированные пропускные способности в контейнере или базе данных и совместно использовать их в контейнерах базы данных. Пропускную способность можно подготавливать с помощью портал Azure, Azure CLI или пакетов SDK для Azure Cosmos DB.

Если вы используете другой API, см. статью [SQL API](how-to-provision-container-throughput.md), [API Cassandra](how-to-provision-throughput-cassandra.md), [Gremlin API](how-to-provision-throughput-gremlin.md) , чтобы обеспечить пропускную способность.

## <a name="azure-portal"></a><a id="portal-mongodb"></a> портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) или выберите существующую.

1. Откройте панель **Обозреватель данных** и выберите **Новая коллекция**. После этого предоставьте следующие сведения.

   * Укажите, создаете ли вы новую базу данных или используете существующую. Выберите параметр настроить **пропускную способность базы данных** , если требуется обеспечить пропускную способность на уровне базы данных.
   * Введите идентификатор коллекции.
   * Введите значение ключа секции (например, `/ItemID`).
   * Укажите подготавливаемую пропускную способность (например, 1000 ЕЗ/с).
   * Щелкните **ОК**.

    :::image type="content" source="./media/how-to-provision-throughput-mongodb/provision-database-throughput-portal-mongodb-api.png" alt-text="Снимок экрана обозреватель данных при создании новой коллекции с пропускной способностью уровня базы данных":::

> [!Note]
> При подготовке пропускной способности для учетной записи Azure Cosmos, настроенной с помощью API Azure Cosmos DB для MongoDB, используйте `/myShardKey` для пути к ключу секции.

## <a name="net-sdk"></a><a id="dotnet-mongodb"></a> ПАКЕТ SDK ДЛЯ .NET

```csharp
// refer to MongoDB .NET Driver
// https://docs.mongodb.com/drivers/csharp

// Create a new Client
String mongoConnectionString = "mongodb://DBAccountName:Password@DBAccountName.documents.azure.com:10255/?ssl=true&replicaSet=globaldb";
mongoUrl = new MongoUrl(mongoConnectionString);
mongoClientSettings = MongoClientSettings.FromUrl(mongoUrl);
mongoClient = new MongoClient(mongoClientSettings);

// Change the database name
mongoDatabase = mongoClient.GetDatabase("testdb");

// Change the collection name, throughput value then update via MongoDB extension commands
// https://docs.microsoft.com/en-us/azure/cosmos-db/mongodb-custom-commands#update-collection

var result = mongoDatabase.RunCommand<BsonDocument>(@"{customAction: ""UpdateCollection"", collection: ""testcollection"", offerThroughput: 400}");
```

## <a name="azure-resource-manager"></a>Azure Resource Manager

Azure Resource Manager шаблоны можно использовать для инициализации пропускной способности автомасштабирования для ресурсов базы данных или уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. [в разделе Azure Resource Manager Templates for Azure Cosmos DB](templates-samples-mongodb.md) .

## <a name="azure-cli"></a>Azure CLI

Azure CLI можно использовать для инициализации пропускной способности автомасштабирования в базе данных или ресурсах уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. в разделе [Azure CLI Samples for Azure Cosmos DB](cli-samples-mongodb.md).

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell можно использовать для инициализации пропускной способности автомасштабирования в базе данных или ресурсах уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. в разделе [Azure PowerShell Samples for Azure Cosmos DB](powershell-samples-mongodb.md).

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать о подготовке пропускной способности в Cosmos DB, обратитесь к следующим статьям:

* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)