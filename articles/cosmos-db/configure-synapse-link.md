---
title: Настройка и использование Azure Synapse Link для Azure Cosmos DB
description: Узнайте, как включить ссылку синапсе для учетных записей Azure Cosmos DB, создать контейнер с включенным аналитическим хранилищем, подключить базу данных Cosmos для Azure к рабочей области синапсе и выполнить запросы.
author: Rodrigossz
ms.service: cosmos-db
ms.topic: how-to
ms.date: 11/30/2020
ms.author: rosouz
ms.custom: references_regions, synapse-cosmos-db
ms.openlocfilehash: 60b720f3f5d91570e32ecf3d03aa7065f93990c5
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104868211"
---
# <a name="configure-and-use-azure-synapse-link-for-azure-cosmos-db"></a>Настройка и использование Azure Synapse Link для Azure Cosmos DB
[!INCLUDE[appliesto-sql-mongodb-api](includes/appliesto-sql-mongodb-api.md)]

[Ссылка Azure синапсе для Azure Cosmos DB](synapse-link.md) — это собственная Гибридная функция для работы с транзакциями и аналитической обработки (HTAP), которая позволяет запускать аналитические данные практически в реальном времени по рабочим данным в Azure Cosmos DB. Synapse Link обеспечивает тесную эффективную интеграцию между Azure Cosmos DB и Azure Synapse Analytics.

Ссылка Azure синапсе доступна для Azure Cosmos DB контейнеров API SQL или для Azure Cosmos DB API для коллекций Mongo DB. Выполните следующие действия, чтобы выполнить аналитические запросы с помощью ссылки Azure синапсе для Azure Cosmos DB:

* [Включение ссылки синапсе для учетных записей Azure Cosmos DB](#enable-synapse-link)
* [Создание контейнера Azure Cosmos DB с включенным аналитическим хранилищем](#create-analytical-ttl)
* [Необязательно. обновление срока жизни аналитического хранилища для контейнера Azure Cosmos DB](#update-analytical-ttl)
* [Подключение базы данных Azure Cosmos DB к рабочей области синапсе](#connect-to-cosmos-database)
* [выполните запрос к аналитическому хранилищу с помощью Synapse Spark](#query-analytical-store-spark).
* [Запрос к аналитическому хранилищу с помощью бессерверного пула SQL](#query-analytical-store-sql-on-demand)
* [Использование несерверного пула SQL для анализа и визуализации данных в Power BI](#analyze-with-powerbi)

## <a name="enable-azure-synapse-link-for-azure-cosmos-db-accounts"></a><a id="enable-synapse-link"></a>Включение ссылки Azure синапсе для учетных записей Azure Cosmos DB

### <a name="azure-portal"></a>Портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure](create-sql-api-dotnet.md#create-account)или выберите существующую учетную запись Azure Cosmos DB.

1. Перейдите к своей учетной записи Azure Cosmos DB и откройте панель **функции** .

1. Выберите **Synapse Link** из списка функций.

   :::image type="content" source="./media/configure-synapse-link/find-synapse-link-feature.png" alt-text="Функция поиска ссылок синапсе":::

1. Далее вам будет предложено включить Synapse Link в вашей учетной записи. Нажмите кнопку **Включить**. Выполнение этого процесса может занять от 1 до 5 минут.

   :::image type="content" source="./media/configure-synapse-link/enable-synapse-link-feature.png" alt-text="Включение функции Synapse Link":::

1. Теперь ваша учетная запись включена для использования Synapse Link. Далее ознакомьтесь со сведениями о создании контейнеров с поддержкой аналитического хранилища для автоматического запуска репликации операционных данных из хранилища транзакций в аналитическое хранилище.

> [!NOTE]
> Включение ссылки синапсе не приводит к автоматическому включению аналитического хранилища. После включения ссылки синапсе в учетной записи Cosmos DB включите аналитическое хранилище в контейнерах при их создании, чтобы начать репликацию данных операций в аналитическое хранилище. 

### <a name="azure-cli"></a>Azure CLI

Следующие ссылки показывают, как включить синапсе Link с помощью Azure CLI:

* [Создать новую учетную запись Azure Cosmos DB с включенной ссылкой синапсе](https://docs.microsoft.com/cli/azure/cosmosdb?view=azure-cli-latest#az_cosmosdb_create-optional-parameters&preserve-view=true)
* [Обновление существующей учетной записи Azure Cosmos DB для включения ссылки синапсе](https://docs.microsoft.com/cli/azure/cosmosdb?view=azure-cli-latest#az_cosmosdb_update-optional-parameters&preserve-view=true)

### <a name="powershell"></a>PowerShell

* [Создать новую учетную запись Azure Cosmos DB с включенной ссылкой синапсе](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbaccount?view=azps-5.5.0#description&preserve-view=true)
* [Обновление существующей учетной записи Azure Cosmos DB для включения ссылки синапсе](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbaccount?view=azps-5.5.0&preserve-view=true)


Следующие ссылки показывают, как включить синапсе Link с помощью PowerShell:

## <a name="create-an-azure-cosmos-container-with-analytical-store"></a><a id="create-analytical-ttl"></a> Создание контейнера Azure Cosmos с помощью аналитического хранилища

Вы можете включить аналитическое хранилище в контейнере Azure Cosmos при создании контейнера. Вы можете использовать портал Azure или настроить свойство `analyticalTTL` во время создания контейнера с помощью пакетов SDK Azure Cosmos DB.

> [!NOTE]
> В настоящее время аналитическое хранилище можно включить для **новых** контейнеров (как в новых, так и в существующих учетных записях). Данные из контейнеров существующего можно перенести в новые контейнеры с помощью [Azure Cosmos DB средств миграции.](cosmosdb-migrationchoices.md)

### <a name="azure-portal"></a>портал Azure;

1. Войдите в [портал Azure](https://portal.azure.com/) или [Обозреватель Azure Cosmos DB](https://cosmos.azure.com/).

1. Перейдите к своей учетной записи Azure Cosmos DB и откройте вкладку **Data Explorer**.

1. Выберите **Создать контейнер** и введите имя базы данных, контейнера, ключ секции и сведения о пропускной способности. Активируйте параметр **Analytical store** (Аналитическое хранилище). После включения аналитического хранилища создается контейнер со свойством `AnalyicalTTL`, для которого задано значение по умолчанию -1 (неограниченный срок хранения). В этом аналитическом хранилище содержатся все исторические версии записей.

   :::image type="content" source="./media/configure-synapse-link/create-container-analytical-store.png" alt-text="Включение аналитического хранилища для контейнера Azure Cosmos":::

1. Если вы ранее не включили Synapse Link в этой учетной записи, вам будет предложено сделать это, так как создание контейнера с включенным аналитическим хранилищем является необходимым условием. При появлении запроса выберите **Включить Synapse Link**. Выполнение этого процесса может занять от 1 до 5 минут.

1. Нажмите кнопку **OK**, чтобы создать контейнер Azure Cosmos с включенным аналитическим хранилищем.

1. После создания контейнера убедитесь, что аналитическое хранилище включено, щелкнув **Параметры**, расположенные ниже документы в обозреватель данных и убедитесь, что параметр **аналитический период времени жизни** включен.

### <a name="net-sdk"></a>Пакет SDK для .NET

Следующий код создает контейнер с аналитическим хранилищем с помощью пакета SDK для .NET. Задайте требуемое значение для свойства аналитического срока жизни. Список допустимых значений см. в статье о [поддерживаемых значениях аналитического срока жизни](analytical-store-introduction.md#analytical-ttl):

```csharp
// Create a container with a partition key, and analytical TTL configured to -1 (infinite retention)
ContainerProperties properties = new ContainerProperties()
{
    Id = "myContainerId",
    PartitionKeyPath = "/id",
    AnalyticalStoreTimeToLiveInSeconds = -1,
};
CosmosClient cosmosClient = new CosmosClient("myConnectionString");
await cosmosClient.GetDatabase("myDatabase").CreateContainerAsync(properties);
```

### <a name="java-v4-sdk"></a>Пакет SDK для Java версии 4

Следующий код создает контейнер с аналитическим хранилищем с помощью пакета SDK для Java версии 4. Задайте для свойства `AnalyticalStoreTimeToLiveInSeconds` необходимое значение:

```java
// Create a container with a partition key and  analytical TTL configured to  -1 (infinite retention) 
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");

containerProperties.setAnalyticalStoreTimeToLiveInSeconds(-1);

container = database.createContainerIfNotExists(containerProperties, 400).block().getContainer();
```

### <a name="python-v4-sdk"></a>Пакет SDK для Python v4

Для Python 2,7 и Azure Cosmos DB SDK 4.1.0 требуются минимальные версии, а пакет SDK совместим только с API SQL.

Первый шаг — убедиться, что используется по меньшей мере версия 4.1.0 [пакета SDK для Python версии Azure Cosmos DB](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos):

```python
import azure.cosmos as cosmos

print (cosmos.__version__)
```
Следующий шаг создает контейнер с аналитическим хранилищем с помощью пакета SDK для Azure Cosmos DB Python:

```python
# Azure Cosmos DB Python SDK, for SQL API only.
# Creating an analytical store enabled container.

import azure.cosmos.cosmos_client as cosmos_client
import azure.cosmos.exceptions as exceptions
from azure.cosmos.partition_key import PartitionKey

HOST = 'your-cosmos-db-account-URI'
KEY = 'your-cosmos-db-account-key'
DATABASE = 'your-cosmos-db-database-name'
CONTAINER = 'your-cosmos-db-container-name'

client = cosmos_client.CosmosClient(HOST,  KEY )
# setup database for this sample. 
# If doesn't exist, creates a new one with the name informed above.
try:
    db = client.create_database(DATABASE)

except exceptions.CosmosResourceExistsError:
    db = client.get_database_client(DATABASE)

# Creating the container with analytical store enabled, using the name informed above.
# If a container with the same name exists, an error is returned.
#
# The 3 options for the analytical_storage_ttl parameter are:
# 1) 0 or Null or not informed (Not enabled).
# 2) -1 (The data will be stored in analytical store infinitely).
# 3) Any other number is the actual ttl, in seconds.

try:
    container = db.create_container(
        id=CONTAINER,
        partition_key=PartitionKey(path='/id', kind='Hash'),analytical_storage_ttl=-1
    )
    properties = container.read()
    print('Container with id \'{0}\' created'.format(container.id))
    print('Partition Key - \'{0}\''.format(properties['partitionKey']))

except exceptions.CosmosResourceExistsError:
    print('A container with already exists')
```

### <a name="azure-cli"></a>Azure CLI

Следующие ссылки показывают, как создать контейнеры с поддержкой аналитического хранилища с помощью Azure CLI.

* [API Azure Cosmos DB для Mongo DB](https://docs.microsoft.com/cli/azure/cosmosdb/mongodb/collection?view=azure-cli-latest#az_cosmosdb_mongodb_collection_create-examples&preserve-view=true)
* [Azure Cosmos DB API SQL](https://docs.microsoft.com/cli/azure/cosmosdb/sql/container?view=azure-cli-latest#az_cosmosdb_sql_container_create&preserve-view=true)

### <a name="powershell"></a>PowerShell

Следующие ссылки показывают, как создать контейнеры с поддержкой аналитического хранилища с помощью PowerShell:

* [API Azure Cosmos DB для Mongo DB](https://docs.microsoft.com/powershell/module/az.cosmosdb/new-azcosmosdbmongodbcollection?view=azps-5.5.0#description&preserve-view=true)
* [Azure Cosmos DB API SQL](https://docs.microsoft.com/cli/azure/cosmosdb/sql/container?view=azure-cli-latest#az_cosmosdb_sql_container_create&preserve-view=true)


## <a name="optional---update-the-analytical-store-time-to-live"></a><a id="update-analytical-ttl"></a> Необязательно. Обновите срок жизни аналитического хранилища.

После включения аналитического хранилища с определенным значением TTL вы можете позже обновить его до другого допустимого значения. Это значение можно обновить с помощью портал Azure, Azure CLI, PowerShell или Cosmos DB пакетов SDK. Сведения о различных параметрах конфигурации аналитического срока жизни см. в статье о [поддерживаемых значениях аналитического срока жизни](analytical-store-introduction.md#analytical-ttl).


### <a name="azure-portal"></a>Портал Azure

Если контейнер с включенным аналитическим хранилищем был создан на портале Azure, он содержит стандартное значение аналитического срока жизни, которое равно -1. Чтобы обновить это значение, выполните следующие действия:

1. Войдите в [портал Azure](https://portal.azure.com/) или [Обозреватель Azure Cosmos DB](https://cosmos.azure.com/).

1. Перейдите к своей учетной записи Azure Cosmos DB и откройте вкладку **Data Explorer**.

1. Выберите существующий контейнер с включенным аналитическим хранилищем. Разверните его и измените следующие значения:

  * Откройте окно **Scale & Settings** (Параметры масштабирования).
  * В разделе **Параметр** найдите ** Analytical Storage Time to Live** (Срок жизни аналитического хранилища).
  * Выберите **Включен (по умолчанию)** или **Включен** и задайте значение срока жизни.
  * Щелкните **Сохранить** , чтобы сохранить изменения.

### <a name="net-sdk"></a>Пакет SDK для .NET

В следующем коде показано, как обновить срок жизни для аналитического хранилища с помощью пакета SDK для .NET:

```csharp
// Get the container, update AnalyticalStorageTimeToLiveInSeconds 
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Update analytical store TTL
containerResponse.Resource. AnalyticalStorageTimeToLiveInSeconds = 60 * 60 * 24 * 180  // Expire analytical store data in 6 months;
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```

### <a name="java-v4-sdk"></a>Пакет SDK для Java версии 4

В следующем коде показано, как обновить срок жизни для аналитического хранилища с помощью пакета SDK для Java версии 4:

```java
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");

// Update analytical store TTL to expire analytical store data in 6 months;
containerProperties.setAnalyticalStoreTimeToLiveInSeconds (60 * 60 * 24 * 180 );  
 
// Update container settings
container.replace(containerProperties).block();
```

### <a name="python-v4-sdk"></a>Пакет SDK для Python v4

В настоящий момент не поддерживается.


### <a name="azure-cli"></a>Azure CLI

Следующие ссылки показывают, как обновить Контейнеры с аналитическим сроком жизни с помощью Azure CLI.

* [API Azure Cosmos DB для Mongo DB](https://docs.microsoft.com/cli/azure/cosmosdb/mongodb/collection?view=azure-cli-latest#az_cosmosdb_mongodb_collection_update&preserve-view=true)
* [Azure Cosmos DB API SQL](https://docs.microsoft.com/cli/azure/cosmosdb/sql/container?view=azure-cli-latest#az_cosmosdb_sql_container_update&preserve-view=true)

### <a name="powershell"></a>PowerShell

Следующие ссылки показывают, как обновить Контейнеры с аналитическим сроком жизни с помощью PowerShell:

* [API Azure Cosmos DB для Mongo DB](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbmongodbcollection?view=azps-5.5.0&preserve-view=true)
* [Azure Cosmos DB API SQL](https://docs.microsoft.com/powershell/module/az.cosmosdb/update-azcosmosdbsqlcontainer?view=azps-5.5.0&preserve-view=true)


## <a name="connect-to-a-synapse-workspace"></a><a id="connect-to-cosmos-database"></a> Подключение к рабочей области Synapse

Выполните инструкции из статьи о [подключении к Azure Synapse Link](../synapse-analytics/synapse-link/how-to-connect-synapse-link-cosmos-db.md), чтобы получить доступ к базе данных Azure Cosmos DB из Azure Synapse Analytics Studio с помощью Azure Synapse Link.

## <a name="query-analytical-store-using-apache-spark-for-azure-synapse-analytics"></a><a id="query-analytical-store-spark"></a> Запрос к аналитическому хранилищу с помощью Apache Spark для Azure синапсе Analytics

Инструкции по выполнению запросов с помощью Synapse Spark см. в статье о [запросах в аналитическое хранилище Azure Cosmos DB](../synapse-analytics/synapse-link/how-to-query-analytical-store-spark.md). В этой статье приводятся некоторые примеры того, как можно взаимодействовать с аналитическим хранилищем с помощью жестов Synapse. Чтобы просмотреть эти жесты, щелкните правой кнопкой мыши контейнер. С помощью жестов можно быстро создать код и скорректировать его в соответствии с потребностями. Они также идеально подходят для обнаружения данных одним щелчком мыши.

## <a name="query-the-analytical-store-using-serverless-sql-pool-in-azure-synapse-analytics"></a><a id="query-analytical-store-sql-on-demand"></a> Запрос к аналитическому хранилищу с помощью бессерверного пула SQL в Azure синапсе Analytics

Бессерверный пул SQL позволяет запрашивать и анализировать данные в контейнерах Azure Cosmos DB, которые включены с помощью ссылки Azure синапсе. Данные можно анализировать практически в реальном времени, не влияя на производительность транзакционных рабочих нагрузок. Он предлагает знакомый синтаксис T-SQL для запроса данных из аналитического хранилища и интегрированного подключения к широкому спектру средств BI и специальных запросов через интерфейс T-SQL. Дополнительные сведения см. в статье [запрос аналитического хранилища с использованием несерверного пула SQL](../synapse-analytics/sql/query-cosmos-db-analytical-store.md) .

## <a name="use-serverless-sql-pool-to-analyze-and-visualize-data-in-power-bi"></a><a id="analyze-with-powerbi"></a>Использование несерверного пула SQL для анализа и визуализации данных в Power BI

Для Azure Cosmos DB можно создать бессерверную базу данных пула SQL и представления по ссылке синапсе. Позже можно будет выполнить запрос к контейнерам Azure Cosmos, а затем создать модель с Power BI для этих представлений, чтобы отразить этот запрос. Дополнительные сведения см. в статье Использование [Бессерверного пула SQL для анализа Azure Cosmos DB данных с помощью ссылки на синапсе](synapse-link-power-bi.md) .

## <a name="azure-resource-manager-template"></a>Шаблон Azure Resource Manager

[Шаблон Azure Resource Manager](./manage-with-templates.md#azure-cosmos-account-with-analytical-store) создает Azure Cosmos DB учетную запись с поддержкой синапсе Link для API SQL. Этот шаблон позволяет создать учетную запись API (SQL) Core в одном регионе с настроенным контейнером, для которого включена поддержка аналитического срока жизни и возможность выбрать для пропускной способности настройку вручную или с помощью автомасштабирования. Чтобы развернуть этот шаблон, щелкните **Развертывание в Azure** на странице файла сведений.

## <a name="getting-started-with-azure-synapse-link---samples"></a><a id="cosmosdb-synapse-link-samples"></a> Приступая к работе с Azure синапсе Link-Samples

Примеры для начала работы с ссылкой на Azure синапсе можно найти на сайте [GitHub](https://aka.ms/cosmosdb-synapselink-samples). Эти презентации представляют комплексные решения с помощью Интернета вещей и сценариев розничной торговли. Вы также можете найти образцы, соответствующие Azure Cosmos DB API для MongoDB в том же репозитории в папке [MongoDB](https://github.com/Azure-Samples/Synapse/tree/main/Notebooks/PySpark/Synapse%20Link%20for%20Cosmos%20DB%20samples) . 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих документах:

* [Сведения об Azure Synapse Link для Azure Cosmos DB (предварительная версия)](synapse-link.md).

* [Общие сведения об аналитическом хранилище Azure Cosmos DB (предварительная версия)](analytical-store-introduction.md).

* [Часто задаваемые вопросы о Azure Synapse Link для Azure Cosmos DB](synapse-link-frequently-asked-questions.md).

* [Ключевые концепции Apache Spark в Azure Synapse Analytics](../synapse-analytics/spark/apache-spark-concepts.md).

* [Поддержка среды выполнения пула SQL на сервере в Azure синапсе Analytics](../synapse-analytics/sql/on-demand-workspace-overview.md).
