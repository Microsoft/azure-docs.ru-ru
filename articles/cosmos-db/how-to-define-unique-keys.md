---
title: Определение уникальных ключей для контейнера Azure Cosmos
description: Узнайте, как определить уникальные ключи для контейнера Azure Cosmos с помощью портал Azure, PowerShell, .NET, Java и различных пакетов SDK.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 12/02/2019
ms.author: thweiss
ms.custom: devx-track-python, devx-track-js, devx-track-csharp
ms.openlocfilehash: 55fc5222c1c245c56ba0a26caa816c5c845147c1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93336628"
---
# <a name="define-unique-keys-for-an-azure-cosmos-container"></a>Определение уникальных ключей для контейнера Azure Cosmos
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье описаны различные способы определения [уникальных ключей](unique-keys.md) при создании контейнера Azure Cosmos. Сейчас выполнить эту операцию можно с помощью портала Azure или одного из пакетов SDK.

## <a name="use-the-azure-portal"></a>Использование портала Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте учетную запись Azure Cosmos](create-sql-api-dotnet.md#create-account) или выберите имеющуюся.

1. Откройте панель **обозревателя данных** и выберите контейнер, с которым собираетесь работать.

1. Щелкните **Создать контейнер**.

1. В диалоговом окне **Добавить контейнер** щелкните **+ Add unique key** (+ Добавить уникальный ключ), чтобы добавить запись уникального ключа.

1. Введите пути к ограничению уникального ключа.

1. При необходимости добавьте больше записей уникального ключа. Для этого щелкните **+ Add unique key** (+ Добавить уникальный ключ).

    :::image type="content" source="./media/how-to-define-unique-keys/unique-keys-portal.png" alt-text="Снимок экрана записи ограничения уникального ключа на портале Azure":::

## <a name="use-powershell"></a>Использование PowerShell

Чтобы создать контейнер с уникальными ключами, см. раздел [Создание контейнера Azure Cosmos с уникальным ключом и TTL](manage-with-powershell.md#create-container-unique-key-ttl)

## <a name="use-the-net-sdk"></a>Использование пакета SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

При создании контейнера с помощью [пакета SDK для .NET версии 2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) объект `UniqueKeyPolicy` можно использовать для определения ограничений уникального ключа.

```csharp
client.CreateDocumentCollectionAsync(UriFactory.CreateDatabaseUri("database"), new DocumentCollection
{
    Id = "container",
    PartitionKey = new PartitionKeyDefinition { Paths = new Collection<string>(new List<string> { "/myPartitionKey" }) },
    UniqueKeyPolicy = new UniqueKeyPolicy
    {
        UniqueKeys = new Collection<UniqueKey>(new List<UniqueKey>
        {
            new UniqueKey { Paths = new Collection<string>(new List<string> { "/firstName", "/lastName", "/emailAddress" }) },
            new UniqueKey { Paths = new Collection<string>(new List<string> { "/address/zipCode" }) }
        })
    }
});
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

При создании нового контейнера с помощью [пакета SDK для .NET v3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/)используйте API Fluent пакета SDK, чтобы объявить уникальные ключи в кратком и удобочитаемом виде.

```csharp
await client.GetDatabase("database").DefineContainer(name: "container", partitionKeyPath: "/myPartitionKey")
    .WithUniqueKey()
        .Path("/firstName")
        .Path("/lastName")
        .Path("/emailAddress")
    .Attach()
    .WithUniqueKey()
        .Path("/address/zipCode")
    .Attach()
    .CreateIfNotExistsAsync();
```
---

## <a name="use-the-java-sdk"></a>Использование пакета SDK для Java

При создании контейнера с помощью [пакета SDK для Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) объект `UniqueKeyPolicy` можно использовать для определения ограничений уникального ключа.

```java
// create a new DocumentCollection object
DocumentCollection container = new DocumentCollection();
container.setId("container");

// create array of strings and populate them with the unique key paths
Collection<String> uniqueKey1Paths = new ArrayList<String>();
uniqueKey1Paths.add("/firstName");
uniqueKey1Paths.add("/lastName");
uniqueKey1Paths.add("/emailAddress");
Collection<String> uniqueKey2Paths = new ArrayList<String>();
uniqueKey2Paths.add("/address/zipCode");

// create UniqueKey objects and set their paths
UniqueKey uniqueKey1 = new UniqueKey();
UniqueKey uniqueKey2 = new UniqueKey();
uniqueKey1.setPaths(uniqueKey1Paths);
uniqueKey2.setPaths(uniqueKey2Paths);

// create a new UniqueKeyPolicy object and set its unique keys
UniqueKeyPolicy uniqueKeyPolicy = new UniqueKeyPolicy();
Collection<UniqueKey> uniqueKeys = new ArrayList<UniqueKey>();
uniqueKeys.add(uniqueKey1);
uniqueKeys.add(uniqueKey2);
uniqueKeyPolicy.setUniqueKeys(uniqueKeys);

// set the unique key policy
container.setUniqueKeyPolicy(uniqueKeyPolicy);

// create the container
client.createCollection(String.format("/dbs/%s", "database"), container, null);
```

## <a name="use-the-nodejs-sdk"></a>Использование пакета SDK для Node.js

При создании контейнера с помощью [пакета SDK для Node.js](https://www.npmjs.com/package/@azure/cosmos) объект `UniqueKeyPolicy` можно использовать для определения ограничений уникального ключа.

```javascript
client.database('database').containers.create({
    id: 'container',
    uniqueKeyPolicy: {
        uniqueKeys: [
            { paths: ['/firstName', '/lastName', '/emailAddress'] },
            { paths: ['/address/zipCode'] }
        ]
    }
});
```

## <a name="use-the-python-sdk"></a>Использование пакета SDK для Python

При создании контейнера с помощью [пакета SDK для Python](https://pypi.org/project/azure-cosmos/) ограничения уникального ключа могут быть указаны как часть словаря в виде параметра.

```python
client.CreateContainer('dbs/' + config['DATABASE'], {
    'id': 'container',
    'uniqueKeyPolicy': {
        'uniqueKeys': [
            {'paths': ['/firstName', '/lastName', '/emailAddress']},
            {'paths': ['/address/zipCode']}
        ]
    }
})
```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [секционировании](partitioning-overview.md)
- [Как работает индексация](index-overview.md)
