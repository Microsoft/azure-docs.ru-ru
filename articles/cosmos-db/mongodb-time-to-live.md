---
title: Функция срока жизни на уровне документа MongoDB в Azure Cosmos DB
description: Узнайте, как задать срок жизни для документов с помощью API Azure Cosmos DB для MongoDB, чтобы система автоматически удаляла их по истечении определенного периода.
author: sivethe
ms.author: sivethe
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: javascript
ms.topic: how-to
ms.date: 12/26/2018
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: f044e03fa3f8aece677e3918b2add8190041c7ac
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2021
ms.locfileid: "107589522"
---
# <a name="expire-data-with-azure-cosmos-dbs-api-for-mongodb"></a>Завершение срока действия данных с помощью API Azure Cosmos DB для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Функция срока жизни позволяет автоматически прекращать хранение данных в базе данных. API Azure Cosmos DB для MongoDB использует основные возможности срока жизни Cosmos DB. Поддерживаются два режима: установка срока жизни по умолчанию для всей коллекции и выбор отдельных значений срока жизни для каждого документа. В API Cosmos DB для MongoDB индексы срока жизни и значения срока жизни для документов управляются с помощью [той же логики, что и в Cosmos DB](../cosmos-db/mongodb-indexing.md).

## <a name="ttl-indexes"></a>Индексы срока жизни
Чтобы установить срок жизни для всей коллекции, нужно создать [индекс срока жизни](../cosmos-db/mongodb-indexing.md). Это индекс для поля _ts со значением expireAfterSeconds.

Пример
```JavaScript
globaldb:PRIMARY> db.coll.createIndex({"_ts":1}, {expireAfterSeconds: 10})
{
        "_t" : "CreateIndexesResponse",
        "ok" : 1,
        "createdCollectionAutomatically" : true,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 4
}
```

С помощью приведенной выше команды создается индекс с функцией срока жизни. После создания индекса в базе данных будут автоматически удаляться все документы в этой коллекции, которые не были изменены за последние 10 секунд. 

> [!NOTE]
> **_ts** является специфическим полем Cosmos DB и не доступно клиентам MongoDB. Это зарезервированное (системное) свойство, содержащее метку времени последнего изменения документа.
>
    
Дополнительный пример на C#: 

```csharp
var options = new CreateIndexOptions {ExpireAfter = TimeSpan.FromSeconds(10)}; 
var field = new StringFieldDefinition<BsonDocument>("_ts"); 
var indexDefinition = new IndexKeysDefinitionBuilder<BsonDocument>().Ascending(field); 
await collection.Indexes.CreateOneAsync(indexDefinition, options); 
``` 

## <a name="set-time-to-live-value-for-a-document"></a>Установка срока жизни для документа 
Также есть возможность установить срок жизни для документа. Документы должны содержать свойство ttl (в нижнем регистре) корневого уровня. Кроме того, для коллекции нужно создать индекс срока жизни, как описано выше. Значения срока жизни, заданные для отдельных документов, переопределяют значение срока жизни для коллекции.

Значение срока жизни должно быть типа int32, значением типа int64, которое помещается в значение типа int32 либо значением double без десятичной части, которое помещается в значение типа int32. Значения срока жизни, не соответствующие этим характеристикам, допускаются, но не рассматриваются как значимые.

Значение срока жизни для документа является необязательным. В коллекцию можно вставлять и документы без значения срока жизни.  В этом случае будет применяться значение срока жизни для коллекции. 

Для указанных ниже документов заданы допустимые значения срока жизни. После вставки документов значения их срока жизни переопределяют значение срока жизни для коллекции. Следовательно, документы будут удалены через 20 секунд.   

```JavaScript 
globaldb:PRIMARY> db.coll.insert({id:1, location: "Paris", ttl: 20.0}) 
globaldb:PRIMARY> db.coll.insert({id:1, location: "Paris", ttl: NumberInt(20)}) 
globaldb:PRIMARY> db.coll.insert({id:1, location: "Paris", ttl: NumberLong(20)}) 
```

Для указанных ниже документов заданы недопустимые значения срока жизни. Документы будут вставлены, но значения их срока жизни не будут применяться. Следовательно, документы будут удалены через 10 секунд из-за истечения срока жизни, заданного для коллекции. 

```JavaScript 
globaldb:PRIMARY> db.coll.insert({id:1, location: "Paris", ttl: 20.5}) //TTL value contains non-zero decimal part. 
globaldb:PRIMARY> db.coll.insert({id:1, location: "Paris", ttl: NumberLong(2147483649)}) //TTL value is greater than Int32.MaxValue (2,147,483,648). 
``` 

## <a name="next-steps"></a>Дальнейшие действия
* [Срок жизни для данных Azure Cosmos DB](../cosmos-db/time-to-live.md)
* [Индексирование в Azure Cosmos DB. API MongoDB](../cosmos-db/mongodb-indexing.md)
