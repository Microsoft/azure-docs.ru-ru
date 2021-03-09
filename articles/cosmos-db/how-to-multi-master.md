---
title: Настройка записи в несколько регионов в Azure Cosmos DB
description: Узнайте, как настроить запись в несколько регионов для приложений с помощью разных пакетов SDK в Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 01/06/2021
ms.author: mjbrown
ms.custom: devx-track-python, devx-track-js, devx-track-csharp, "seo-nov-2020"
ms.openlocfilehash: 08d50b18605fd833e6b0efca987338d0ca1eef8d
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102488517"
---
# <a name="configure-multi-region-writes-in-your-applications-that-use-azure-cosmos-db"></a>Настройка операций записи в несколько регионов в приложениях, использующих Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

После создания учетной записи с несколькими регионами записи необходимо внести два изменения в приложение в ConnectionPolicy для клиента Cosmos, чтобы включить запись в несколько регионов в Azure Cosmos DB. В ConnectionPolicy задайте для Усемултиплеврителокатионс значение true и передайте имя региона, в котором приложение развертывается в Аппликатионрегион. Это заполнит свойство PreferredLocations на основе географической близости от передаваемого расположения. Позже при добавлении нового региона в учетную запись нет необходимости обновлять или повторно развертывать приложение. В случае инициации регионального события оно автоматически обнаружит ближайший регион и подключится к нему.

> [!Note]
> Учетные записи Cosmos, изначально настроенные с одним регионом записи, можно настроить для нескольких регионов записи с нулевым временем простоя. Дополнительные сведения см. в разделе [Настройка нескольких регионов записи](how-to-manage-database-account.md#configure-multiple-write-regions).

## <a name="azure-portal"></a><a id="portal"></a> портал Azure

Чтобы включить запись в несколько регионов из портал Azure, выполните следующие действия.

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Перейдите к своей учетной записи Azure Cosmos и в меню откройте панель **репликация данных глобально** .

1. В разделе " **записи в нескольких регионах** " выберите **включить**. Он автоматически добавляет существующие регионы для чтения и записи регионов.

1. Можно добавить дополнительные регионы, выбрав значки на карте или нажав кнопку **Добавить регион** . Все добавляемые регионы будут включены как чтение, так и запись.

1. После обновления списка регионов выберите **сохранить** , чтобы применить изменения.

   :::image type="content" source="./media/how-to-multi-master/enable-multi-region-writes.png" alt-text="Снимок экрана: включение записи в несколько регионов с помощью портал Azure" lightbox="./media/how-to-multi-master/enable-multi-region-writes.png":::

## <a name="net-sdk-v2"></a><a id="netv2"></a>Пакет SDK для .NET версии 2

Чтобы включить операции записи в несколько регионов в приложении, задайте `UseMultipleWriteLocations` для значение `true` . Кроме того, задайте для `SetCurrentLocation` регион, в котором развернуто приложение и реплицируется Azure Cosmos DB:

```csharp
ConnectionPolicy policy = new ConnectionPolicy
    {
        ConnectionMode = ConnectionMode.Direct,
        ConnectionProtocol = Protocol.Tcp,
        UseMultipleWriteLocations = true
    };
policy.SetCurrentLocation("West US 2");
```

## <a name="net-sdk-v3"></a><a id="netv3"></a>Пакет SDK версии 3 для .NET

Чтобы включить операции записи в несколько регионов в приложении, задайте `ApplicationRegion` регион, в котором развертывается приложение, а также место, где Cosmos DB реплицируется.

```csharp
CosmosClient cosmosClient = new CosmosClient(
    "<connection-string-from-portal>", 
    new CosmosClientOptions()
    {
        ApplicationRegion = Regions.WestUS2,
    });
```

При необходимости можно использовать `CosmosClientBuilder` и `WithApplicationRegion` для достижения того же результата:

```csharp
CosmosClientBuilder cosmosClientBuilder = new CosmosClientBuilder("<connection-string-from-portal>")
            .WithApplicationRegion(Regions.WestUS2);
CosmosClient client = cosmosClientBuilder.Build();
```

## <a name="java-v4-sdk"></a><a id="java4-multi-region-writes"></a> Пакет SDK для Java версии 4

Чтобы включить операции записи в несколько регионов в приложении, вызовите `.multipleWriteRegionsEnabled(true)` и `.preferredRegions(preferredRegions)` в построителе клиентов, где является элементом, `preferredRegions` `List` содержащим один элемент, то есть регион, в котором развертывается приложение и где Cosmos DB реплицируется:

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

   Асинхронный API [пакета SDK для Java версии 4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos))

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ConfigureMultimasterAsync)]

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

   Синхронный API [пакета SDK для Java версии 4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos))

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ConfigureMultimasterSync)]

--- 

## <a name="async-java-v2-sdk"></a><a id="java2-multi-region-writes"></a> Асинхронный пакет SDK для Java v2

Пакет SDK для Java v2 использовал Maven [com. Microsoft. Azure:: Azure-cosmosdb](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb). Чтобы включить запись в несколько регионов в приложении, задайте `policy.setUsingMultipleWriteLocations(true)` и задайте `policy.setPreferredLocations` регион, в котором развертывается приложение, а также место, где Cosmos DB реплицируется.

```java
ConnectionPolicy policy = new ConnectionPolicy();
policy.setUsingMultipleWriteLocations(true);
policy.setPreferredLocations(Collections.singletonList(region));

AsyncDocumentClient client =
    new AsyncDocumentClient.Builder()
        .withMasterKeyOrResourceToken(this.accountKey)
        .withServiceEndpoint(this.accountEndpoint)
        .withConsistencyLevel(ConsistencyLevel.Eventual)
        .withConnectionPolicy(policy).build();
```

## <a name="nodejs-javascript-and-typescript-sdks"></a><a id="javascript"></a>Пакеты SDK для Node.js, JavaScript и TypeScript

Чтобы включить операции записи в несколько регионов в приложении, задайте `connectionPolicy.UseMultipleWriteLocations` для значение `true` . Кроме того, задайте для параметра `connectionPolicy.PreferredLocations` регион, в котором развернуто приложение и реплицируется Cosmos DB:

```javascript
const connectionPolicy: ConnectionPolicy = new ConnectionPolicy();
connectionPolicy.UseMultipleWriteLocations = true;
connectionPolicy.PreferredLocations = [region];

const client = new CosmosClient({
  endpoint: config.endpoint,
  auth: { masterKey: config.key },
  connectionPolicy,
  consistencyLevel: ConsistencyLevel.Eventual
});
```

## <a name="python-sdk"></a><a id="python"></a>Пакет SDK для Python

Чтобы включить операции записи в несколько регионов в приложении, задайте `connection_policy.UseMultipleWriteLocations` для значение `true` . Кроме того, задайте для параметра `connection_policy.PreferredLocations` регион, в котором развернуто приложение и реплицируется Cosmos DB.

```python
connection_policy = documents.ConnectionPolicy()
connection_policy.UseMultipleWriteLocations = True
connection_policy.PreferredLocations = [region]

client = cosmos_client.CosmosClient(self.account_endpoint, {
                                    'masterKey': self.account_key}, connection_policy, documents.ConsistencyLevel.Session)
```

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь со следующими статьями:

* [Использование маркеров сеанса](how-to-manage-consistency.md#utilize-session-tokens)
* [Типы конфликтов и политики их устранения](conflict-resolution-policies.md)
* [Высокий уровень доступности в Azure Cosmos DB](high-availability.md)
* [Уровни согласованности для Azure Cosmos DB](consistency-levels.md)
* [Choose the right consistency level](./consistency-levels.md) (Выбор правильного уровня согласованности)
* [Недостатки согласованности, доступности и производительности в Azure Cosmos DB](./consistency-levels.md)
* [Достижение компромисса между доступностью и быстродействием для разных уровней согласованности](./consistency-levels.md)
* [Глобальное масштабирование подготовленной пропускной способности](./request-units.md)
* [Глобальное распределение. внутри](global-dist-under-the-hood.md)