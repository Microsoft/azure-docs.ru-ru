---
title: Руководство по настройке глобального распределения с помощью API Azure Cosmos DB для MongoDB
description: Сведения о настройке глобального распределения с помощью API Azure Cosmos DB для MongoDB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: tutorial
ms.date: 12/26/2018
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 89826eab7b1686ae695a2716a03b2f5d03da277f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93099262"
---
# <a name="set-up-global-distributed-database-using-azure-cosmos-dbs-api-for-mongodb"></a>Настройка глобальной распределенной базы данных с помощью API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

В этой статье показано, как на портале Azure настроить глобальную распределенную базу данных и подключиться к ней с помощью API Azure Cosmos DB для MongoDB.

В этой статье рассматриваются следующие задачи: 

> [!div class="checklist"]
> * настроили глобальное распределение на портале Azure;
> * Настройка глобального распределения с помощью [API Azure Cosmos DB для MongoDB](mongodb-introduction.md).

[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]

## <a name="verifying-your-regional-setup"></a>Проверка региональных настроек 
Самый простой способ проверить глобальную конфигурацию в API Cosmos DB для MongoDB — выполнить команду *isMaster()* из оболочки Mongo.

Из оболочки Mongo:

   ```
      db.isMaster()
   ```
   
Пример результатов:

   ```JSON
      {
         "_t": "IsMasterResponse",
         "ok": 1,
         "ismaster": true,
         "maxMessageSizeBytes": 4194304,
         "maxWriteBatchSize": 1000,
         "minWireVersion": 0,
         "maxWireVersion": 2,
         "tags": {
            "region": "South India"
         },
         "hosts": [
            "vishi-api-for-mongodb-southcentralus.documents.azure.com:10255",
            "vishi-api-for-mongodb-westeurope.documents.azure.com:10255",
            "vishi-api-for-mongodb-southindia.documents.azure.com:10255"
         ],
         "setName": "globaldb",
         "setVersion": 1,
         "primary": "vishi-api-for-mongodb-southindia.documents.azure.com:10255",
         "me": "vishi-api-for-mongodb-southindia.documents.azure.com:10255"
      }
   ```

## <a name="connecting-to-a-preferred-region"></a>Подключение к предпочтительному региону 

API Azure Cosmos DB для MongoDB позволяет указать параметры чтения коллекции для глобально распределенной базы данных. Для выполнения операций чтения с низкой задержкой и обеспечения глобальной высокой доступности рекомендуется задать в параметрах чтения коллекции значение *Nearest* (Ближайший). Параметр *Nearest* означает, что чтение будет выполняться из ближайшего региона.

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.Nearest));
```

Для приложений, использующих сценарии с основным регионом для операций чтения и записи и дополнительным регионом для аварийного восстановления, рекомендуется задать в параметрах чтения коллекции значение *SecondaryPreferred* (Предпочтительно дополнительный). Параметр *SecondaryPreferred* означает, что чтение будет выполняться из дополнительного региона при недоступности основного региона.

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.SecondaryPreferred));
```

Наконец, вы можете вручную указать регион для чтения. Для этого задайте в параметрах чтения тег региона.

```csharp
var collection = database.GetCollection<BsonDocument>(collectionName);
var tag = new Tag("region", "Southeast Asia");
collection = collection.WithReadPreference(new ReadPreference(ReadPreferenceMode.Secondary, new[] { new TagSet(new[] { tag }) }));
```

На этом руководство завершено. Сведения об управлении согласованностью глобально реплицируемой учетной записи Azure Cosmos DB см. в [этой статье](consistency-levels.md). Сведения о том, как функционирует репликация глобальной базы данных в Azure Cosmos DB, см. в статье о [глобальном распределении данных в Azure Cosmos DB](distribute-data-globally.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы выполнили следующее:

> [!div class="checklist"]
> * настроили глобальное распределение на портале Azure;
> * Настройка глобального распределения с помощью API Cosmos DB для MongoDB

Перейдите к следующему руководству, чтобы узнать о разработке в локальной среде с помощью локального эмулятора Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Разработка с помощью эмулятора Azure Cosmos DB в локальной среде](local-emulator.md)
