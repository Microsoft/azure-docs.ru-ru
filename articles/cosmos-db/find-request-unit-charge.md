---
title: Найти плату за единицу запроса (RU) для SQL-запроса в Azure Cosmos DB
description: Узнайте, как найти плату за единицу запроса (RU) для запросов SQL, выполняемых в контейнере Azure Cosmos. Для поиска оплаты за ЕДИНИЦу можно использовать языки портал Azure, .NET, Java, Python и Node.js.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: b3e61ca2cee6dd5a2c279b4297e84668729f4ccb
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102178751"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-sql-api"></a>Поиск затрат единиц запросов для операций, выполненных в Azure Cosmos DB API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB поддерживает многие интерфейсы API, такие как SQL, MongoDB, Cassandra, Gremlin и Таблицы. У каждого API есть собственный набор операций базы данных, начиная от простого считывания и записи точек и заканчивая сложными запросами. Каждая операция базы данных потребляет системные ресурсы. Потребление зависит от сложности операции.

Стоимость всех операций базы данных нормализуется с помощью Azure Cosmos DB и выражается в единицах запроса (ЕЗ). Можно считать, что в качестве валюты производительности разрабатываются системные ресурсы, такие как ЦП, операции ввода-вывода и память, необходимые для выполнения операций с базой данных, поддерживаемых Azure Cosmos DB. Независимо от того, какие API вы используете для взаимодействия с контейнером Azure Cosmos, затраты всегда измеряются в ЕЗ. Является ли операция базы данных записью, чтением точки или запросом, затраты всегда измеряются в версии-получателей. Дополнительные сведения см. в статье [единицы запросов и рекомендации](request-units.md) .

В этой статье представлены различные способы, с помощью которых можно найти потребление [единиц запросов](request-units.md) (ru) для любой операции, выполняемой для контейнера в Azure Cosmos DB API SQL. Если вы используете другой API, ознакомьтесь со статьей [API для MongoDB](find-request-unit-charge-mongodb.md), [API Cassandra](find-request-unit-charge-cassandra.md), [Gremlin API](find-request-unit-charge-gremlin.md)и [API ТАБЛИЦ](find-request-unit-charge-table.md) статей, чтобы найти плату за единицу.

Сейчас потребление можно оценить только с помощью портала Azure или сведений из ответа, который Azure Cosmos DB возвращает через пакеты SDK. Если вы используете API SQL, оценить потребление ЕЗ для операций в контейнере Azure Cosmos можно несколькими способами.

## <a name="use-the-azure-portal"></a>Использование портала Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте учетную запись Azure Cosmos](create-sql-api-dotnet.md#create-account) и заполните ее данными или выберите существующую учетную запись Azure Cosmos, которая уже содержит данные.

1. Перейдите на панель **обозревателя данных** и выберите контейнер, с которым собираетесь работать.

1. Выберите **Создать SQL-запрос**.

1. Введите допустимый запрос и щелкните **Выполнить запрос**.

1. Выберите **Статистика запросов**, чтобы отобразилось количество ЕЗ, фактически затраченных на выполненный запрос.

:::image type="content" source="./media/find-request-unit-charge/portal-sql-query.png" alt-text="Снимок экрана со сведениями о расходах в ЕЗ на SQL-запрос на портале Azure":::

## <a name="use-the-net-sdk"></a>Использование пакета SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

Объекты, возвращаемые из [пакета SDK для .NET версии 2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) предоставляют свойство `RequestCharge`.

```csharp
ResourceResponse<Document> fetchDocumentResponse = await client.ReadDocumentAsync(
    UriFactory.CreateDocumentUri("database", "container", "itemId"),
    new RequestOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    });
var requestCharge = fetchDocumentResponse.RequestCharge;

StoredProcedureResponse<string> storedProcedureCallResponse = await client.ExecuteStoredProcedureAsync<string>(
    UriFactory.CreateStoredProcedureUri("database", "container", "storedProcedureId"),
    new RequestOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    });
requestCharge = storedProcedureCallResponse.RequestCharge;

IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
    UriFactory.CreateDocumentCollectionUri("database", "container"),
    "SELECT * FROM c",
    new FeedOptions
    {
        PartitionKey = new PartitionKey("partitionKey")
    }).AsDocumentQuery();
while (query.HasMoreResults)
{
    FeedResponse<dynamic> queryResponse = await query.ExecuteNextAsync<dynamic>();
    requestCharge = queryResponse.RequestCharge;
}
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

Объекты, возвращаемые из [пакета SDK для .NET версии 3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) предоставляют свойство `RequestCharge`:

[!code-csharp[](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos/tests/Microsoft.Azure.Cosmos.Tests/SampleCodeForDocs/CustomDocsSampleCode.cs?name=GetRequestCharge)]

Дополнительные сведения см. [в разделе Краткое руководство. Создание веб-приложения .NET с помощью учетной записи API SQL в Azure Cosmos DB](create-sql-api-dotnet.md).

---

## <a name="use-the-java-sdk"></a>Использование пакета SDK для Java

Объекты, возвращаемые из [пакета SDK для Java](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) предоставляют метод `getRequestCharge()`.

```java
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPartitionKey(new PartitionKey("partitionKey"));

Observable<ResourceResponse<Document>> readDocumentResponse = client.readDocument(String.format("/dbs/%s/colls/%s/docs/%s", "database", "container", "itemId"), requestOptions);
readDocumentResponse.subscribe(result -> {
    double requestCharge = result.getRequestCharge();
});

Observable<StoredProcedureResponse> storedProcedureResponse = client.executeStoredProcedure(String.format("/dbs/%s/colls/%s/sprocs/%s", "database", "container", "storedProcedureId"), requestOptions, null);
storedProcedureResponse.subscribe(result -> {
    double requestCharge = result.getRequestCharge();
});

FeedOptions feedOptions = new FeedOptions();
feedOptions.setPartitionKey(new PartitionKey("partitionKey"));

Observable<FeedResponse<Document>> feedResponse = client
    .queryDocuments(String.format("/dbs/%s/colls/%s", "database", "container"), "SELECT * FROM c", feedOptions);
feedResponse.forEach(result -> {
    double requestCharge = result.getRequestCharge();
});
```

Дополнительные сведения см. в разделе [Краткое руководство. Создание приложения Java с помощью учетной записи API Azure Cosmos DB SQL](create-sql-api-java.md).

## <a name="use-the-nodejs-sdk"></a>Использование пакета SDK для Node.js

Объекты, возвращаемые из [пакета SDK для Node.js](https://www.npmjs.com/package/@azure/cosmos) предоставляют вложенный объект `headers`, который позволяет сопоставить все заголовки, возвращаемые базовым интерфейсом API для HTTP. Сведения о расходах в ЕЗ доступны в ключе `x-ms-request-charge`.

```javascript
const item = await client
    .database('database')
    .container('container')
    .item('itemId', 'partitionKey')
    .read();
var requestCharge = item.headers['x-ms-request-charge'];

const storedProcedureResult = await client
    .database('database')
    .container('container')
    .storedProcedure('storedProcedureId')
    .execute({
        partitionKey: 'partitionKey'
    });
requestCharge = storedProcedureResult.headers['x-ms-request-charge'];

const query = client.database('database')
    .container('container')
    .items
    .query('SELECT * FROM c', {
        partitionKey: 'partitionKey'
    });
while (query.hasMoreResults()) {
    var result = await query.executeNext();
    requestCharge = result.headers['x-ms-request-charge'];
}
```

Дополнительные сведения см. в разделе [Краткое руководство. создание Node.js приложения с помощью учетной записи API Azure Cosmos DB SQL](create-sql-api-nodejs.md). 

## <a name="use-the-python-sdk"></a>Использование пакета SDK для Python

Объект `CosmosClient`, возвращаемый из [пакета SDK для Python](https://pypi.org/project/azure-cosmos/) предоставляет словарь `last_response_headers`, который позволяет сопоставить все заголовки для последней выполненной операции, возвращаемые базовым API HTTP. Сведения о расходах в ЕЗ доступны в ключе `x-ms-request-charge`.

```python
response = client.ReadItem(
    'dbs/database/colls/container/docs/itemId', {'partitionKey': 'partitionKey'})
request_charge = client.last_response_headers['x-ms-request-charge']

response = client.ExecuteStoredProcedure(
    'dbs/database/colls/container/sprocs/storedProcedureId', None, {'partitionKey': 'partitionKey'})
request_charge = client.last_response_headers['x-ms-request-charge']
```

Дополнительные сведения см. в разделе [Краткое руководство. Создание приложения Python с помощью учетной записи API Azure Cosmos DB SQL](create-sql-api-python.md). 

## <a name="next-steps"></a>Дальнейшие действия

Сведения об оптимизации потребления ЕЗ см. в следующих статьях:

* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)
* [Optimize provisioned throughput cost in Azure Cosmos DB](optimize-cost-throughput.md) (Оптимизация стоимости подготовленной пропускной способности в Azure Cosmos DB)
* [Optimize query cost in Azure Cosmos DB](./optimize-cost-reads-writes.md) (Оптимизация затрат на отправку запросов в Azure Cosmos DB)
* [Globally scale provisioned throughput](./request-units.md) (Глобальное масштабирование подготовленной пропускной способности)
* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md)
* [Подготовка пропускной способности для контейнера](how-to-provision-container-throughput.md)
* [Monitor and debug with metrics in Azure Cosmos DB](use-metrics.md) (Мониторинг и отладка с помощью метрик в Azure Cosmos DB)
