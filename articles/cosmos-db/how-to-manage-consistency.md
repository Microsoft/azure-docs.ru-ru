---
title: Управление уровнями согласованности в Azure Cosmos DB
description: Узнайте, как настраивать уровни согласованности и управлять ими в Azure Cosmos DB с помощью портал Azure, пакета SDK для .NET, пакета SDK для Java и различных других пакетов SDK.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/10/2020
ms.author: anfeldma
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: b0c03c2f5313605fbdf288a9262df0852e066efd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93333483"
---
# <a name="manage-consistency-levels-in-azure-cosmos-db"></a>Управление уровнями согласованности в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье рассматривается управление уровнями согласованности в Azure Cosmos DB. Вы узнаете, как настроить уровень согласованности по умолчанию, а также как переопределить ее и управлять маркерами сеанса вручную. Кроме того, здесь приведены общие сведения о метрике вероятностного ограниченного устаревания (PBS).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="configure-the-default-consistency-level"></a>Настройка уровня согласованности по умолчанию

Клиенты по умолчанию используют [стандартный уровень согласованности](consistency-levels.md).

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

Чтобы просмотреть или изменить уровень согласованности по умолчанию, войдите на портал Azure. Найдите учетную запись Azure Cosmos DB и откройте область **Согласованность по умолчанию**. Выберите требуемый уровень согласованности в качестве нового значения по умолчанию, а затем выберите **Сохранить**. На портале Azure также представлена визуализация разных уровней согласованности на основе музыкальных нот. 

:::image type="content" source="./media/how-to-manage-consistency/consistency-settings.png" alt-text="Меню согласованности на портале Azure":::

# <a name="cli"></a>[CLI](#tab/cli)

Создайте учетную запись Cosmos с согласованностью сеанса, а затем обновите согласованность по умолчанию.

```azurecli
# Create a new account with Session consistency
az cosmosdb create --name $accountName --resource-group $resourceGroupName --default-consistency-level Session

# update an existing account's default consistency
az cosmosdb update --name $accountName --resource-group $resourceGroupName --default-consistency-level Strong
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Создайте учетную запись Cosmos с согласованностью сеанса, а затем обновите согласованность по умолчанию.

```azurepowershell-interactive
# Create a new account with Session consistency
New-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
  -Location $locations -Name $accountName -DefaultConsistencyLevel "Session"

# Update an existing account's default consistency
Update-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
  -Name $accountName -DefaultConsistencyLevel "Strong"
```

---

## <a name="override-the-default-consistency-level"></a>Переопределение уровня согласованности по умолчанию

Клиенты могут переопределить уровень согласованности по умолчанию, который задается службой. Уровень согласованности можно задать для каждого запроса. В этом случае будет переопределен уровень согласованности по умолчанию на уровне учетной записи.

> [!TIP]
> Согласованность может быть **ослаблена** только на уровне запроса. Чтобы перейти от более слабого к более надежному согласованию, обновите согласованность по умолчанию для учетной записи Cosmos.

### <a name="net-sdk"></a><a id="override-default-consistency-dotnet"></a>Пакет SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
// Override consistency at the client level
documentClient = new DocumentClient(new Uri(endpoint), authKey, connectionPolicy, ConsistencyLevel.Eventual);

// Override consistency at the request level via request options
RequestOptions requestOptions = new RequestOptions { ConsistencyLevel = ConsistencyLevel.Eventual };

var response = await client.CreateDocumentAsync(collectionUri, document, requestOptions);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

```csharp
// Override consistency at the request level via request options
ItemRequestOptions requestOptions = new ItemRequestOptions { ConsistencyLevel = ConsistencyLevel.Strong };

var response = await client.GetContainer(databaseName, containerName)
    .CreateItemAsync(
        item,
        new PartitionKey(itemPartitionKey),
        requestOptions);
```
---

### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> Пакет SDK для Java версии 4

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

   Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConsistencyAsync)]

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

   Синхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConsistencySync)]

--- 

### <a name="java-v2-sdks"></a><a id="override-default-consistency-javav2"></a> Пакеты SDK для Java v2

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

Асинхронный пакет SDK для Java v2 (Maven com. Microsoft. Azure:: Azure-cosmosdb)

```java
// Override consistency at the client level
ConnectionPolicy policy = new ConnectionPolicy();

AsyncDocumentClient client =
        new AsyncDocumentClient.Builder()
                .withMasterKey(this.accountKey)
                .withServiceEndpoint(this.accountEndpoint)
                .withConsistencyLevel(ConsistencyLevel.Eventual)
                .withConnectionPolicy(policy).build();
```

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

Синхронизация пакета SDK для Java v2 (Maven com. Microsoft. Azure:: Azure-documentdb)

```java
// Override consistency at the client level
ConnectionPolicy connectionPolicy = new ConnectionPolicy();
DocumentClient client = new DocumentClient(accountEndpoint, accountKey, connectionPolicy, ConsistencyLevel.Eventual);
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="override-default-consistency-javascript"></a>Пакет SDK для Node.js, JavaScript и TypeScript

```javascript
// Override consistency at the client level
const client = new CosmosClient({
  /* other config... */
  consistencyLevel: ConsistencyLevel.Eventual
});

// Override consistency at the request level via request options
const { body } = await item.read({ consistencyLevel: ConsistencyLevel.Eventual });
```

### <a name="python-sdk"></a><a id="override-default-consistency-python"></a>Пакет SDK для Python

```python
# Override consistency at the client level
connection_policy = documents.ConnectionPolicy()
client = cosmos_client.CosmosClient(self.account_endpoint, {
                                    'masterKey': self.account_key}, connection_policy, documents.ConsistencyLevel.Eventual)
```

## <a name="utilize-session-tokens"></a>Использование маркеров сеанса

Один из этих уровней согласованности в Azure Cosmos DB — согласованность *сеансов*. Этот уровень по умолчанию применяются к учетным записям Cosmos. Если задана согласованность *сеансов*, то клиент будет использовать внутренний маркер сеанса для каждой операции чтения или запроса, чтобы обеспечить соблюдение заданного уровня согласованности.

Чтобы управлять токенами сеанса вручную, получите токен сеанса из ответа и установите их для каждого запроса. Если у вас нет необходимости управлять токенами сеанса вручную, то вам не нужно использовать эти примеры. Пакет SDK отслеживает токены сеанса автоматически. Если токен сеанса не задан вручную, по умолчанию в пакете SDK используется последний.

### <a name="net-sdk"></a><a id="utilize-session-tokens-dotnet"></a>Пакет SDK для .NET

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
var response = await client.ReadDocumentAsync(
                UriFactory.CreateDocumentUri(databaseName, collectionName, "SalesOrder1"));
string sessionToken = response.SessionToken;

RequestOptions options = new RequestOptions();
options.SessionToken = sessionToken;
var response = await client.ReadDocumentAsync(
                UriFactory.CreateDocumentUri(databaseName, collectionName, "SalesOrder1"), options);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

```csharp
Container container = client.GetContainer(databaseName, collectionName);
ItemResponse<SalesOrder> response = await container.CreateItemAsync<SalesOrder>(salesOrder);
string sessionToken = response.Headers.Session;

ItemRequestOptions options = new ItemRequestOptions();
options.SessionToken = sessionToken;
ItemResponse<SalesOrder> response = await container.ReadItemAsync<SalesOrder>(salesOrder.Id, new PartitionKey(salesOrder.PartitionKey), options);
```
---

### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> Пакет SDK для Java версии 4

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

   Асинхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConsistencySessionAsync)]

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

   Синхронный API пакета SDK для Java версии 4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConsistencySessionSync)]

--- 

### <a name="java-v2-sdks"></a><a id="utilize-session-tokens-javav2"></a>Пакеты SDK для Java v2

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

Асинхронный пакет SDK для Java v2 (Maven com. Microsoft. Azure:: Azure-cosmosdb)

```java
// Get session token from response
RequestOptions options = new RequestOptions();
options.setPartitionKey(new PartitionKey(document.get("mypk")));
Observable<ResourceResponse<Document>> readObservable = client.readDocument(document.getSelfLink(), options);
readObservable.single()           // we know there will be one response
  .subscribe(
      documentResourceResponse -> {
          System.out.println(documentResourceResponse.getSessionToken());
      },
      error -> {
          System.err.println("an error happened: " + error.getMessage());
      });

// Resume the session by setting the session token on RequestOptions
RequestOptions options = new RequestOptions();
requestOptions.setSessionToken(sessionToken);
Observable<ResourceResponse<Document>> readObservable = client.readDocument(document.getSelfLink(), options);
```

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

Синхронизация пакета SDK для Java v2 (Maven com. Microsoft. Azure:: Azure-documentdb)

```java
// Get session token from response
ResourceResponse<Document> response = client.readDocument(documentLink, null);
String sessionToken = response.getSessionToken();

// Resume the session by setting the session token on the RequestOptions
RequestOptions options = new RequestOptions();
options.setSessionToken(sessionToken);
ResourceResponse<Document> response = client.readDocument(documentLink, options);
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="utilize-session-tokens-javascript"></a>Пакет SDK для Node.js, JavaScript и TypeScript

```javascript
// Get session token from response
const { headers, item } = await container.items.create({ id: "meaningful-id" });
const sessionToken = headers["x-ms-session-token"];

// Immediately or later, you can use that sessionToken from the header to resume that session.
const { body } = await item.read({ sessionToken });
```

### <a name="python-sdk"></a><a id="utilize-session-tokens-python"></a>Пакет SDK для Python

```python
// Get the session token from the last response headers
item = client.ReadItem(item_link)
session_token = client.last_response_headers["x-ms-session-token"]

// Resume the session by setting the session token on the options for the request
options = {
    "sessionToken": session_token
}
item = client.ReadItem(doc_link, options)
```

## <a name="monitor-probabilistically-bounded-staleness-pbs-metric"></a>Мониторинг метрики вероятностного ограниченного устаревания (PBS)

Насколько итоговая согласованность является итоговой? В среднем мы можем предложить границы устаревания с учетом журнала версий и времени. Метрика [**Probabilistically Bounded Staleness (PBS)**](https://pbs.cs.berkeley.edu/) (Вероятностное ограниченное устаревание) пытается количественно оценить вероятность устаревания и отображает полученные результаты. Чтобы просмотреть метрику PBS, перейдите к учетной записи Azure Cosmos DB на портале Azure. Откройте панель **метрики** и перейдите на вкладку **согласованность** . Взгляните на граф с именем **вероятность строго стабильного чтения на основе рабочей нагрузки (см. PBS)**.

:::image type="content" source="./media/how-to-manage-consistency/pbs-metric.png" alt-text="График PBS на портале Azure":::

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о том, как управлять конфликтами данных, или перейдите к следующей ключевой концепции в Azure Cosmos DB. См. следующие статьи:

* [Уровни согласованности в Azure Cosmos DB](consistency-levels.md)
* [Секционирование и масштабирование в Azure Cosmos DB](./partitioning-overview.md)
* [Управление конфликтами между регионами](how-to-manage-conflicts.md)
* [Секционирование и масштабирование в Azure Cosmos DB](partitioning-overview.md)
* [Компромиссы согласованности в современных распределенных системах баз данных](https://www.computer.org/csdl/magazine/co/2012/02/mco2012020037/13rRUxjyX7k)
* [Высокая доступность](high-availability.md)
* [Соглашение об уровне обслуживания для Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_2/)