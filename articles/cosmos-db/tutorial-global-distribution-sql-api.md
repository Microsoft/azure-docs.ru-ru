---
title: Руководство по Руководство по настройке глобального распределения Azure Cosmos DB с помощью API SQL
description: Руководство по настройке глобального распределения Azure Cosmos DB с помощью API SQL с .NET, Java, Python и другими пакетами SDK
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: tutorial
ms.date: 11/05/2019
ms.reviewer: sngun
ms.custom: devx-track-python, devx-track-js, devx-track-csharp
ms.openlocfilehash: aacb8d4ffb98d553b17aa52e4c4b11a4837dc1c6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93337903"
---
# <a name="tutorial-set-up-azure-cosmos-db-global-distribution-using-the-sql-api"></a>Руководство по Настройка глобального распределения Azure Cosmos DB с помощью API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье показано, как настроить глобальное распределение базы данных Azure Cosmos DB и подключиться к ней с помощью API SQL на портале Azure.

В этой статье рассматриваются следующие задачи: 

> [!div class="checklist"]
> * настроили глобальное распределение на портале Azure;
> * настроили глобальное распределение с помощью [API SQL](./introduction.md).

<a id="portal"></a>
[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]

## <a name="connecting-to-a-preferred-region-using-the-sql-api"></a><a id="preferred-locations"></a> Подключение к предпочтительному региону с помощью API SQL

Чтобы воспользоваться преимуществами [глобального распределения](distribute-data-globally.md), клиентские приложения могут указать упорядоченный список предпочитаемых регионов, который будет использоваться для операций с документами. Для операций записи и чтения с помощью пакета SDK для SQL выбирается наиболее оптимальная конечная точка на основании текущих данных о региональной доступности и списка предпочтений, указанного в конфигурации учетной записи Azure Cosmos DB.

Этот список предпочтений указывается при инициализации подключения с помощью пакетов SDK для SQL. Пакеты SDK принимают необязательный параметр `PreferredLocations`, представляющий собой упорядоченный список регионов Azure.

Пакет SDK будет автоматически отправлять все операции записи в текущий регион записи. Все операции чтения будут отправляться в первый доступный регион из списка предпочтительных расположений. Если запрос завершится неудачей, клиент перейдет к следующему региону дальше по списку.

Пакеты SDK будут пытаться выполнять чтение данных из регионов, указанных в списке предпочтительных расположений. Например, если учетная запись Azure Cosmos доступна в четырех регионах, но клиент указывает в `PreferredLocations` только два региона для чтения (не для записи), то операции чтения не будут обслуживаться для региона, который не указан в `PreferredLocations`. Если указанные в `PreferredLocations` регионы не доступны, операции чтения обслуживаются в регионе записи.

Приложение может проверить текущие конечную точку записи и конечную точку чтения, выбранные пакетом SDK, просмотрев два свойства, `WriteEndpoint` и `ReadEndpoint`, доступные в SDK 1.8 и более поздней версии. Если свойство `PreferredLocations` не задано, будут обслуживаться все запросы из текущего региона записи.

Если не указать предпочтительные расположения, используя метод `setCurrentLocation`, пакет SDK автоматически заполняет предпочтительные расположения в зависимости от текущего региона, в котором работает клиент. Пакет SDK упорядочивает регионы на основе близости региона к текущему региону.

## <a name="net-sdk"></a>Пакет SDK для .NET

Пакет SDK можно использовать без изменения кода. В этом случае пакет SDK автоматически направляет операции чтения и записи в текущий регион записи.

В пакете SDK для .NET 1.8 и более поздней версии параметр ConnectionPolicy конструктора DocumentClient имеет свойство Microsoft.Azure.Documents.ConnectionPolicy.PreferredLocations. Это свойство имеет тип коллекции `<string>` и должно содержать список имен регионов. Строковые значения форматируются по столбцу "Имя региона" на странице [Регионы Azure][regions] без пробелов до или после первого и последнего знака соответственно.

Текущие конечные точки записи и чтения доступны в DocumentClient.WriteEndpoint и DocumentClient.ReadEndpoint, соответственно.

> [!NOTE]
> Не следует считать URL-адреса конечных точек долговременными константами. Служба может обновить их в любой момент. Пакет SDK обрабатывает это изменение автоматически.
>

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

Если вы используете пакет SDK для .NET версии 2, примените свойство `PreferredLocations`, чтобы задать предпочтительный регион.

```csharp
// Getting endpoints from application settings or other configuration location
Uri accountEndPoint = new Uri(Properties.Settings.Default.GlobalDatabaseUri);
string accountKey = Properties.Settings.Default.GlobalDatabaseKey;
  
ConnectionPolicy connectionPolicy = new ConnectionPolicy();

//Setting read region selection preference
connectionPolicy.PreferredLocations.Add(LocationNames.WestUS); // first preference
connectionPolicy.PreferredLocations.Add(LocationNames.EastUS); // second preference
connectionPolicy.PreferredLocations.Add(LocationNames.NorthEurope); // third preference

// initialize connection
DocumentClient docClient = new DocumentClient(
    accountEndPoint,
    accountKey,
    connectionPolicy);

// connect to DocDB
await docClient.OpenAsync().ConfigureAwait(false);
```

Кроме того, можно использовать свойство `SetCurrentLocation` и позволить пакету SDK выбрать предпочтительное расположение на основе сходства.

```csharp
// Getting endpoints from application settings or other configuration location
Uri accountEndPoint = new Uri(Properties.Settings.Default.GlobalDatabaseUri);
string accountKey = Properties.Settings.Default.GlobalDatabaseKey;
  
ConnectionPolicy connectionPolicy = new ConnectionPolicy();

connectionPolicy.SetCurrentLocation("West US 2"); /

// initialize connection
DocumentClient docClient = new DocumentClient(
    accountEndPoint,
    accountKey,
    connectionPolicy);

// connect to DocDB
await docClient.OpenAsync().ConfigureAwait(false);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

Если вы используете пакет SDK для .NET версии 3, примените свойство `ApplicationPreferredRegions`, чтобы задать предпочтительный регион.

```csharp

CosmosClientOptions options = new CosmosClientOptions();
options.ApplicationName = "MyApp";
options.ApplicationPreferredRegions = new List<string> {Regions.WestUS, Regions.WestUS2};

CosmosClient client = new CosmosClient(connectionString, options);

```

Кроме того, можно использовать свойство `ApplicationRegion` и позволить пакету SDK выбрать предпочтительное расположение на основе сходства.

```csharp
CosmosClientOptions options = new CosmosClientOptions();
options.ApplicationName = "MyApp";
// If the application is running in West US
options.ApplicationRegion = Regions.WestUS;

CosmosClient client = new CosmosClient(connectionString, options);
```

---

## <a name="nodejsjavascript"></a>Node. js и JavaScript

> [!NOTE]
> Не следует считать URL-адреса конечных точек долговременными константами. Служба может обновить их в любой момент. Пакет SDK обработает это изменение автоматически.
>
>

Ниже приведен пример кода для Node.js и JavaScript.

```JavaScript
// Setting read region selection preference, in the following order -
// 1 - West US
// 2 - East US
// 3 - North Europe
const preferredLocations = ['West US', 'East US', 'North Europe'];

// initialize the connection
const client = new CosmosClient{ endpoint, key, connectionPolicy: { preferredLocations } });
```

## <a name="python-sdk"></a>Пакет SDK для Python

В коде ниже показано, как задать требуемое расположение с помощью пакета SDK для Python.

```python
connectionPolicy = documents.ConnectionPolicy()
connectionPolicy.PreferredLocations = ['West US', 'East US', 'North Europe']
client = cosmos_client.CosmosClient(ENDPOINT, {'masterKey': MASTER_KEY}, connectionPolicy)

```

## <a name="java-v4-sdk"></a><a id="java4-preferred-locations"></a> Пакет SDK для Java версии 4

В коде ниже показано, как задать требуемое расположение с помощью пакета SDK для Java.

# <a name="async"></a>[Асинхронный режим](#tab/api-async)

   Асинхронный API [пакета SDK для Java версии 4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos))

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TutorialGlobalDistributionPreferredLocationAsync)]

# <a name="sync"></a>[Синхронизация](#tab/api-sync)

   Синхронный API [пакета SDK для Java версии 4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos))

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=TutorialGlobalDistributionPreferredLocationSync)]

--- 

## <a name="rest"></a>REST

После того, как учетная запись базы данных станет доступной в нескольких регионах, клиенты смогут запрашивать ее доступность с помощью запроса GET к универсальному коду ресурса (URI) `https://{databaseaccount}.documents.azure.com/`.

Служба вернет список регионов и соответствующих универсальных кодов ресурса (URI) конечных точек Azure Cosmos DB для реплик. В ответе будет указан текущий регион записи. Затем клиент сможет выбрать подходящую конечную точку для всех последующих запросов REST API следующим образом.

Пример ответа

```json
{
    "_dbs": "//dbs/",
    "media": "//media/",
    "writableLocations": [
        {
            "Name": "West US",
            "DatabaseAccountEndpoint": "https://globaldbexample-westus.documents.azure.com:443/"
        }
    ],
    "readableLocations": [
        {
            "Name": "East US",
            "DatabaseAccountEndpoint": "https://globaldbexample-eastus.documents.azure.com:443/"
        }
    ],
    "MaxMediaStorageUsageInMB": 2048,
    "MediaStorageUsageInMB": 0,
    "ConsistencyPolicy": {
        "defaultConsistencyLevel": "Session",
        "maxStalenessPrefix": 100,
        "maxIntervalInSeconds": 5
    },
    "addresses": "//addresses/",
    "id": "globaldbexample",
    "_rid": "globaldbexample.documents.azure.com",
    "_self": "",
    "_ts": 0,
    "_etag": null
}
```

* Все запросы PUT, POST и DELETE должны направляться на указанный универсальный код ресурса (URI) записи.
* Все запросы GET и другие запросы только для чтения могут направляться к любой конечной точке на выбор клиента.

Запросы на запись в регионы, доступные только для чтения, вернут код ошибки HTTP 403 (Forbidden).

В случае изменения региона записи после того, как клиент выполнил этап начального обнаружения, все последующие операции записи в предыдущий регион записи завершатся ошибкой с кодом HTTP 403 (Forbidden). Клиенту следует еще раз получить список регионов (запрос GET), чтобы получить обновленный регион записи.

На этом руководство завершено. Сведения об управлении согласованностью глобально реплицируемой учетной записи Azure Cosmos DB см. в [этой статье](consistency-levels.md). Сведения о том, как функционирует репликация глобальной базы данных в Azure Cosmos DB, см. в статье о [глобальном распределении данных в Azure Cosmos DB](distribute-data-globally.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы выполнили следующее:

> [!div class="checklist"]
> * настроили глобальное распределение на портале Azure;
> * настроили глобальное распределение с помощью API SQL.

Перейдите к следующему руководству, чтобы узнать о разработке в локальной среде с помощью локального эмулятора Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Разработка в локальной среде с помощью эмулятора](local-emulator.md)

[regions]: https://azure.microsoft.com/regions/