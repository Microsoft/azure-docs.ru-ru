---
title: API SQL для Azure Cosmos DB. Примеры для пакета SDK Java версии 4
description: Поиск примеров Java на GitHub для типичных задач с помощью API SQL для Azure Cosmos DB, включая операции CRUD.
author: anfeldma-ms
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 09/23/2020
ms.custom: devx-track-java
ms.author: anfeldma
ms.openlocfilehash: 0b0bdd204b2aa322adeb7373911b90e742c71999
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98019006"
---
# <a name="azure-cosmos-db-sql-api-java-sdk-v4-examples"></a>API SQL для Azure Cosmos DB. Примеры для пакета SDK Java версии 4
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

> [!div class="op_single_selector"]
> * [Примеры пакета SDK для .NET версии 2](sql-api-dotnet-samples.md)
> * [Примеры пакета SDK для .NET версии 3](sql-api-dotnet-v3sdk-samples.md)
> * [Примеры для пакета SDK Java версии 4](sql-api-java-sdk-samples.md)
> * [Примеры для пакета SDK для Spring Data версии 3](sql-api-spring-data-sdk-samples.md)
> * [Примеры Node.js](sql-api-nodejs-samples.md)
> * [Примеры Python](sql-api-python-samples.md)
> * [Коллекция примеров кода Azure](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
> 
> 

> [!IMPORTANT]  
> Дополнительные сведения о пакете SDK Java версии 4 см. в [заметках о выпуске](sql-api-sdk-java-v4.md) для пакета SDK Java для Azure Cosmos DB версии 4, [репозитории Maven](https://mvnrepository.com/artifact/com.azure/azure-cosmos), [рекомендациях по повышению производительности для пакета SDK Java для Azure Cosmos DB версии 4](performance-tips-java-sdk-v4-sql.md), а также в [руководстве по устранению неполадок для пакета SDK Java версии 4 для Azure Cosmos DB версии 4](troubleshoot-java-sdk-v4-sql.md). Если в настоящее время используется более ранняя версия, чем версия 4, ознакомьтесь с руководством [Перевод приложения на использование пакета средств разработки Java для Azure Cosmos DB версии 4](migrate-java-v4-sdk.md), чтобы получить сведения об обновлении до версии 4.
>

> [!IMPORTANT]  
>[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
>  
>- Вы можете [активировать преимущества подписчика Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio): в вашей подписке Visual Studio каждый месяц зачисляются деньги на счет, которые можно использовать для оплаты служб Azure.
>
>[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]
>

Новейшие примеры приложений, которые выполняют операции CRUD и другие распространенные операции с ресурсами Azure Cosmos DB, содержатся в репозитории GitHub [azure-cosmos-java-sql-api-samples](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples). Эта статья содержит:

* Ссылки на задачи в каждом из примеров файлов проектов Java. 
* Ссылки на соответствующие справочные материалы по API.

**Предварительные требования**

Чтобы запустить пример приложения, необходимы следующие компоненты:

* комплект разработчика Java 8;
* Пакет SDK Java версии 4 для Azure Cosmos DB

При необходимости можно использовать Maven, чтобы получить последние двоичные файлы пакета SDK Java версии 4 для Azure Cosmos DB для использования в этом проекте. Maven автоматически добавляет все необходимые зависимости. В противном случае можно непосредственно скачать зависимости, которые перечислены в файле pom.xml, и добавить их в путь сборки.

```bash
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-cosmos</artifactId>
    <version>LATEST</version>
</dependency>
```

**Запуск примеров приложений**

Клонирование примера репозитория:
```bash
$ git clone https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples.git

$ cd azure-cosmos-java-sql-api-samples
```

Примеры можно запустить, используя интегрированною среду разработки (Eclipse, IntelliJ или VSCODE), или с помощью командной строки в Maven.

Необходимо задать эти переменные среды:

```
ACCOUNT_HOST=your account hostname;ACCOUNT_KEY=your account primary key
```

Это предоставит примерам доступ на чтение и запись для вашей учетной записи.

Чтобы запустить пример, укажите его основной класс: 

```
com.azure.cosmos.examples.sample.synchronicity.MainClass
```

В качестве *sample.synchronicity.MainClass* может использоваться:
* crudquickstart.sync.SampleCRUDQuickstart
* crudquickstart.async.SampleCRUDQuickstartAsync
* indexmanagement.sync.SampleIndexManagement
* indexmanagement.async.SampleIndexManagementAsync
* storedprocedure.sync.SampleStoredProcedure
* storedprocedure.async.SampleStoredProcedureAsync
* changefeed.SampleChangeFeedProcesso *(Для канала изменений доступен только пример асинхронности, пример синхронности отсутствует.)* и т. д…

> [!NOTE]
> Каждый пример является самодостаточным, он устанавливается самостоятельно и выполняет необходимую очистку после удаления. В примерах выполняется несколько вызовов для создания `CosmosContainer`. Каждый раз, когда это происходит, вам выставляется счет за 1 час использования каждого уровня производительности создаваемой коллекции. 
> 
> 

## <a name="database-examples"></a>Примеры баз данных
В файле [Database CRUD Samples](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java) показано, как выполнять следующие задачи. Чтобы узнать больше о базах данных Azure Cosmos перед выполнением приведенных ниже примеров, ознакомьтесь с тематической статьей [Работа с базами данных, контейнерами и элементами](account-databases-containers-items.md). 

| Задача | Справочник по API |
| --- | --- |
| [Создание базы данных](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java#L77-L85) | CosmosClient.createDatabaseIfNotExists |
| [Чтение базы данных по идентификатору](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java#L88-L95) | CosmosClient.getDatabase |
| [Чтение всех баз данных](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java#L98-L112) | CosmosClient.readAllDatabases |
| [Удаление базы данных](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/databasecrud/sync/DatabaseCRUDQuickstart.java#L115-L123) | CosmosDatabase.delete |

## <a name="collection-examples"></a>Примеры коллекций
В файле [Collection CRUD Samples](https://github.com/Azure/azure-documentdb-java/blob/master/documentdb-examples/src/test/java/com/microsoft/azure/documentdb/examples/CollectionCrudSamples.java) показано, как выполнять следующие задачи. Чтобы узнать больше о коллекциях Azure Cosmos перед выполнением приведенных ниже примеров, ознакомьтесь с тематической статьей [Работа с базами данных, контейнерами и элементами](account-databases-containers-items.md).

| Задача | Справочник по API |
| --- | --- |
| [Создание коллекции](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L97-L112) | CosmosDatabase.createContainerIfNotExists |
| [Изменение настроенной производительности коллекции](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L115-L123) | CosmosContainer.replaceProvisionedThroughput |
| [Получение коллекции по идентификатору](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L126-L133) | CosmosDatabase.getContainer |
| [Чтение всех коллекций в базе данных](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L136-L150) | CosmosDatabase.readAllContainers |
| [Удаление коллекции](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/containercrud/sync/ContainerCRUDQuickstart.java#L153-L161) | CosmosContainer.delete |

## <a name="autoscale-collection-examples"></a>Примеры автомасштабирования коллекций

Чтобы получить дополнительные сведений об автомасштабировании перед выполнением этих примеров, ознакомьтесь с инструкциями по включению автомасштабирования в [учетной записи](https://azure.microsoft.com/resources/templates/101-cosmosdb-sql-autoscale/) и в [базах данных и контейнерах](./provision-throughput-autoscale.md).

В файле [autoscale Database CRUD Samples](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/autoscaledatabasecrud/sync/AutoscaleDatabaseCRUDQuickstart.java) показано, как выполнять следующие задачи.

| Задача | Справочник по API |
| --- | --- |
| [Создание базы данных с указанной максимальной автомасштабируемой пропускной способностью](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/autoscaledatabasecrud/sync/AutoscaleDatabaseCRUDQuickstart.java#L78-L89) | CosmosClient.CreateDatabase<br>ThroughputProperties.createAutoscaledThroughput |

В файле [autoscale Collection CRUD Samples](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/autoscalecontainercrud/sync/AutoscaleContainerCRUDQuickstart.java) показано, как выполнять следующие задачи. 

| Задача | Справочник по API |
| --- | --- |
| [Создание коллекции с указанной максимальной автомасштабируемой пропускной способностью](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/autoscalecontainercrud/sync/AutoscaleContainerCRUDQuickstart.java#L97-L110) | CosmosDatabase.createContainerIfNotExists |
| [Изменение настроенной максимальной автомасштабируемой пропускной способности коллекции](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/autoscalecontainercrud/sync/AutoscaleContainerCRUDQuickstart.java#L113-L120) | CosmosContainer.replaceThroughput |
| [Чтение конфигурации автомасштабируемой пропускной способности коллекции](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/autoscalecontainercrud/sync/AutoscaleContainerCRUDQuickstart.java#L122-L133) | CosmosContainer.readThroughput |

## <a name="analytical-storage-collection-examples"></a>Примеры коллекции аналитического хранилища

В файле [Analytical storage Collection CRUD Samples](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/analyticalcontainercrud/sync/AnalyticalContainerCRUDQuickstart.java) показано, как выполнять следующие задачи. Чтобы узнать о коллекциях Azure Cosmos перед выполнением приведенных ниже примеров, ознакомьтесь со сведениями об Azure Cosmos DB Synapse и аналитическом хранилище.

| Задача | Справочник по API |
| --- | --- |
| [Создание коллекции](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/analyticalcontainercrud/sync/AnalyticalContainerCRUDQuickstart.java#L93-L108) | CosmosDatabase.createContainerIfNotExists |

## <a name="document-examples"></a>Примеры документов
В файле [Document CRUD Samples](https://github.com/Azure/azure-documentdb-java/blob/master/documentdb-examples/src/test/java/com/microsoft/azure/documentdb/examples/DocumentCrudSamples.java) показано, как выполнять следующие задачи. Чтобы узнать больше о документах Azure Cosmos перед выполнением приведенных ниже примеров, ознакомьтесь с тематической статьей [Работа с базами данных, контейнерами и элементами](account-databases-containers-items.md).

| Задача | Справочник по API |
| --- | --- |
| [Создание документа](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L133-L147) | CosmosContainer.createItem |
| [Чтение документа по идентификатору](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L179-L193) | CosmosContainer.readItem |
| [Запрос документов](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L162-L176) | CosmosContainer.queryItems |
| [Замена документа](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L195-L210) | CosmosContainer.replaceItem |
| [Выполнение операции Upsert для документа](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L212-L2225) | CosmosContainer.upsertItem |
| [Удаление документа](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L303-L310) | CosmosContainer.deleteItem |
| [Замена документа с помощью условной проверки ETag](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L227-L264) | AccessCondition.setType<br>AccessCondition.setCondition |
| [Чтение документа только в том случае, если он был изменен](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/documentcrud/sync/DocumentCRUDQuickstart.java#L266-L300) | AccessCondition.setType<br>AccessCondition.setCondition |

## <a name="indexing-examples"></a>Примеры индексирования
В файле [Collection CRUD Samples](https://github.com/Azure/azure-documentdb-java/blob/master/documentdb-examples/src/test/java/com/microsoft/azure/documentdb/examples/CollectionCrudSamples.java) показано, как выполнять следующие задачи. Чтобы узнать больше об индексировании в Azure Cosmos DB перед выполнением приведенных ниже примеров, ознакомьтесь с тематическими статьями, посвященными [политикам](index-policy.md), [типам](index-overview.md#index-types) и [путям индексирования](index-policy.md#include-exclude-paths). 

| Задача | Справочник по API |
| --- | --- |
| Исключение документа из индекса | ExcludedIndex<br>IndexingPolicy |
| Использование отложенного индексирования | IndexingPolicy.IndexingMode |
| [Включение указанных путей к документам в индексе](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/indexmanagement/sync/SampleIndexManagement.java#L145-L148) | IndexingPolicy.IncludedPaths |
| [Исключение указанных путей к документам из индекса](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/indexmanagement/sync/SampleIndexManagement.java#L150-L153) | IndexingPolicy.ExcludedPaths |
| [Создание составного индекса](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/indexmanagement/sync/SampleIndexManagement.java#L171-L186) | IndexingPolicy.setCompositeIndexes<br>CompositePath |
| Принудительное выполнение операции проверки диапазона в хэш-индексированном пути | FeedOptions.EnableScanInQuery |
| Использование диапазонных индексов в строках | IndexingPolicy.IncludedPaths<br>RangeIndex |
| Преобразование индекса | - |
| [Создание геопространственного индекса](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/indexmanagement/sync/SampleIndexManagement.java#L157-L166) | IndexingPolicy.setSpatialIndexes<br>SpatialSpec<br>SpatialType |

Дополнительные сведения об индексации Azure Cosmos DB см. в [этой статье](index-policy.md).

## <a name="query-examples"></a>Примеры запросов
В файле [Query Samples](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java) показано, как выполнять следующие задачи с помощью грамматики SQL-запроса. Чтобы узнать больше об SQL-запросах в Azure Cosmos DB перед выполнением приведенных ниже примеров, ознакомьтесь со статьей о [примерах SQL-запросов для Azure Cosmos DB](./sql-query-getting-started.md). 

| Задача | Справочник по API |
| --- | --- |
| [Запрос всех документов](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L210-L214) | CosmosContainer.queryItems |
| [Выполнение запроса на соответствие условию с использованием оператора равенства ==](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L291-L295) | CosmosContainer.queryItems |
| [Выполнение запроса на несоответствие условию с использованием операторов != и NOT](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L297-L305) | CosmosContainer.queryItems |
| [Выполнение запроса с использованием операторов диапазона, таких как >, <, >=, <=](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L307-L312) | CosmosContainer.queryItems |
| [Выполнение запроса к строкам с использованием операторов диапазона](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L314-L319) | CosmosContainer.queryItems |
| [Выполнение запроса с использованием предложения ORDER BY](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L321-L326) | CosmosContainer.queryItems |
| [Создание запросов с помощью DISTINCT](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L328-L333) | CosmosContainer.queryItems |
| [Выполнение запроса с использованием агрегатных функций](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L335-L343) | CosmosContainer.queryItems |
| [Работа с вложенными документами](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L345-L353) | CosmosContainer.queryItems |
| [Выполнение запроса с использованием внутридокументных соединений](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L355-L377) | CosmosContainer.queryItems |
| [Выполнение запроса с использованием операторов string, math и array](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L379-L390) | CosmosContainer.queryItems |
| [Выполнение параметризованного SQL-запроса с использованием SqlQuerySpec](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L392-L421) |CosmosContainer.queryItems |
| [Выполнение запроса с применением явного разбиения на страницы](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L216-L266) | CosmosContainer.queryItems |
| [Выполнение запроса к секционированным коллекциям в параллельном режиме](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/queries/sync/QueriesQuickstart.java#L268-L289) | CosmosContainer.queryItems |
| Выполнение запроса к секционированным коллекциям с использованием предложения ORDER BY | CosmosContainer.queryItems |

## <a name="change-feed-examples"></a>Примеры веб-канала изменений 
В файле [Change Feed Processor Sample](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/changefeed/SampleChangeFeedProcessor.java) показано, как выполнять следующие задачи. Чтобы узнать больше о канале изменений в Azure Cosmos DB перед выполнением приведенных ниже примеров, ознакомьтесь со статьями [Чтение канала изменений Azure Cosmos DB](read-change-feed.md) и [Обработчик канала изменений в Azure Cosmos DB](change-feed-processor.md).

| Задача | Справочник по API |
| --- | --- |
| [Базовая функциональность канала изменений](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/changefeed/SampleChangeFeedProcessor.java#L124-L154) |ChangeFeedProcessor.changeFeedProcessorBuilder |
| Чтение канала изменений за определенное время | ChangeFeedProcessor.changeFeedProcessorBuilder |
| [Чтение канала изменений с начала](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/changefeed/SampleChangeFeedProcessor.java#L124-L154) | - |

## <a name="server-side-programming-examples"></a>Примеры программирования на стороне сервера

В файле [Stored Procedure Sample](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/storedprocedure/sync/SampleStoredProcedure.java) показано, как выполнять следующие задачи. Чтобы узнать больше о программировании на стороне сервера в Azure Cosmos DB перед выполнением приведенных ниже примеров, ознакомьтесь со статьей о [хранимых процедурах, триггерах и определяемых пользователем функциях](stored-procedures-triggers-udfs.md).

| Задача | Справочник по API |
| --- | --- |
| [Создание хранимой процедуры](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/storedprocedure/sync/SampleStoredProcedure.java#L132-L151) | CosmosScripts.createStoredProcedure |
| [Выполнение хранимой процедуры](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/storedprocedure/sync/SampleStoredProcedure.java#L167-L181) | CosmosStoredProcedure.execute |
| [Удаление хранимой процедуры](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/main/src/main/java/com/azure/cosmos/examples/storedprocedure/sync/SampleStoredProcedure.java#L183-L193) | CosmosStoredProcedure.delete |

## <a name="user-management-examples"></a>Примеры управления пользователями
В файле User Management Sample показано, как выполнять следующие задачи.

| Задача | Справочник по API |
| --- | --- |
| Создание пользователя | - |
| Установка разрешений на коллекцию или документ | - |
| Получение списка разрешений пользователя |- |
