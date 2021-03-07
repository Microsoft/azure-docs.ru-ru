---
title: Использование библиотеки массового исполнителя .NET с API Gremlin в Azure Cosmos DB
description: Узнайте, как использовать библиотеку массового исполнителя, чтобы массово импортировать данные графа в контейнер API Gremlin в Azure Cosmos DB.
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 05/28/2019
ms.author: chrande
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 15e94dac02770bf28aae4cbfc4e337cb68b8be40
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102425330"
---
# <a name="using-the-graph-bulk-executor-net-library-to-perform-bulk-operations-in-azure-cosmos-db-gremlin-api"></a>Использование библиотеки массового исполнителя .NET для выполнения массовых операций с графами в API Gremlin в Azure Cosmos DB
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

В этом учебнике содержатся инструкции по использованию библиотеки массового исполнителя .NET в Azure CosmosDB для импорта и обновления объектов графа в контейнере API Gremlin в Azure Cosmos DB. В этом процессе используется класс Graph в [библиотеке массового исполнителя](./bulk-executor-overview.md), чтобы программными средствами создавать объекты Vertex (вершина) и Edge (ребро), а затем вставлять несколько из них в один сетевой запрос. Такое поведение можно настроить в библиотеке массового исполнителя, чтобы оптимизировать использование как базы данных, так и локальной памяти.

В отличие от отправки запросов Gremlin в базу данных, когда команды оцениваются и выполняются поочередно, при использовании библиотеки массового исполнителя объекты создаются и проверяются локально. После создания объектов можно последовательно отправлять объекты графа в службу базы данных. Этот метод позволяет повысить скорость приема данных практически в 100 раз, что очень полезно при первоначальном переносе или периодическом перемещении данных. Дополнительные сведения см. на странице GitHub с [примером приложения с использованием библиотеки массового исполнителя для работы с графами в Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started).

## <a name="bulk-operations-with-graph-data"></a>Массовые операции с данными графа

[Библиотека массового исполнителя](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph) содержит пространство имен `Microsoft.Azure.CosmosDB.BulkExecutor.Graph`, которое предоставляет функциональные возможности для создания и импорта объектов графа. 

Ниже описан процесс переноса данных для контейнера API Gremlin:
1. Извлеките записи из источника данных.
2. Создайте объекты `GremlinVertex` и `GremlinEdge` на основе полученных записей и добавьте объекты в структуру данных `IEnumerable`. В этой части приложения необходимо реализовать логику распознавания и добавления связей на случай, если источник данных не является базой данных графа.
3. Вставьте объекты графа в коллекцию с помощью [метода Graph BulkImportAsync](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph.graphbulkexecutor.bulkimportasync).

Такой механизм повышает эффективность переноса данных по сравнению с использованием Gremlin-клиента. При вставке данных c помощью Gremlin приложение отправляет единоразовый запрос на создание данных, который нужно проверить, оценить и затем выполнить. Если используется библиотека массового исполнителя, проверка выполняется в приложении и для каждого сетевого запроса отправляется несколько объектов графа за раз.

### <a name="creating-vertices-and-edges"></a>Создание ребер и вершин

`GraphBulkExecutor` предоставляет метод `BulkImportAsync`, которому необходим список `IEnumerable` объектов `GremlinVertex` или `GremlinEdge`, определенных в пространстве имен `Microsoft.Azure.CosmosDB.BulkExecutor.Graph.Element`. В приведенном примере мы разделили создание ребер и вершин на две отдельные задачи импорта в BulkExecutor. См. пример ниже.

```csharp

IBulkExecutor graphbulkExecutor = new GraphBulkExecutor(documentClient, targetCollection);

BulkImportResponse vResponse = null;
BulkImportResponse eResponse = null;

try
{
    // Import a list of GremlinVertex objects
    vResponse = await graphbulkExecutor.BulkImportAsync(
            Utils.GenerateVertices(numberOfDocumentsToGenerate),
            enableUpsert: true,
            disableAutomaticIdGeneration: true,
            maxConcurrencyPerPartitionKeyRange: null,
            maxInMemorySortingBatchSize: null,
            cancellationToken: token);

    // Import a list of GremlinEdge objects
    eResponse = await graphbulkExecutor.BulkImportAsync(
            Utils.GenerateEdges(numberOfDocumentsToGenerate),
            enableUpsert: true,
            disableAutomaticIdGeneration: true,
            maxConcurrencyPerPartitionKeyRange: null,
            maxInMemorySortingBatchSize: null,
            cancellationToken: token);
}
catch (DocumentClientException de)
{
    Trace.TraceError("Document client exception: {0}", de);
}
catch (Exception e)
{
    Trace.TraceError("Exception: {0}", e);
}
```

Дополнительные сведения о параметрах библиотеки массового исполнителя см. в разделе [Массовый импорт данных в Azure Cosmos DB](bulk-executor-dot-net.md#bulk-import-data-to-an-azure-cosmos-account).

Необходимо создать экземпляры полезных данных в объектах `GremlinVertex` и `GremlinEdge`. Ниже показан способ их создания:

**Вершины**:
```csharp
// Creating a vertex
GremlinVertex v = new GremlinVertex(
    "vertexId",
    "vertexLabel");

// Adding custom properties to the vertex
v.AddProperty("customProperty", "value");

// Partitioning keys must be specified for all vertices
v.AddProperty("partitioningKey", "value");
```

**Ребра**:
```csharp
// Creating an edge
GremlinEdge e = new GremlinEdge(
    "edgeId",
    "edgeLabel",
    "targetVertexId",
    "sourceVertexId",
    "targetVertexLabel",
    "sourceVertexLabel",
    "targetVertexPartitioningKey",
    "sourceVertexPartitioningKey");

// Adding custom properties to the edge
e.AddProperty("customProperty", "value");
```

> [!NOTE]
> Массовый исполнитель не выполняет автоматическую проверку существующих вершин, пока не будут добавлены ребра. Перед выполнением задач BulkImport нужно проверить вершины в приложении.

## <a name="sample-application"></a>Пример приложения

### <a name="prerequisites"></a>Предварительные требования
* Visual Studio 2019 с рабочей нагрузкой разработки Azure. Вы можете бесплатно начать работу в [выпуске Visual Studio 2019 Community](https://visualstudio.microsoft.com/downloads/).
* Подписка Azure. Вы можете создать [бесплатную учетную запись Azure здесь](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cosmos-db). Кроме того, можно создать учетную запись базы данных Cosmos в [бесплатной пробной версии Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure.
* База данных API Gremlin в Azure Cosmos DB с **неограниченной коллекцией**. В этом руководстве объясняется, как начать работу с [Gremlin API в Azure Cosmos DB в .NET](./create-graph-dotnet.md).
* Git. Дополнительные сведения см. на [странице скачивания Git](https://git-scm.com/downloads).

### <a name="clone-the-sample-application"></a>Клонирование примера приложения
Здесь приведены пошаговые инструкции по началу работы на [примере библиотеки массового исполнителя для работы с графами в Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started), размещенном в GitHub. Приложение представляет собой .NET-решение, которое случайным образом создает объекты вершины и ребра, а затем выполняет массовую вставку данных в определенную учетную запись базы данных графа. Чтобы запустить приложение, выполните следующую команду `git clone`:

```bash
git clone https://github.com/Azure-Samples/azure-cosmosdb-graph-bulkexecutor-dotnet-getting-started.git
```

Этот репозиторий содержит пример GraphBulkExecutor со следующими файлами:

Файл|Описание
---|---
`App.config`|Здесь указаны параметры, относящиеся к приложению и базе данных. Сначала нужно изменить файл, чтобы подключить целевую базу данных и коллекции.
`Program.cs`| Этот файл содержит логику создания коллекции `DocumentClient`, управления очистками и отправки запросов массового исполнителя.
`Util.cs`| Файл содержит вспомогательный класс с логикой генерации тестовых данных, а также проверки наличия базы данных и коллекции.

В файле `App.config` приведены возможные значения конфигурации:

Параметр|Описание
---|---
`EndPointUrl`|**Конечная точка .NET SDK**, указанная в колонке "Обзор" вашей учетной записи в базе данных API Gremlin в Azure Cosmos DB в следующем формате: `https://your-graph-database-account.documents.azure.com:443/`
`AuthorizationKey`|Первичный или вторичный ключ из вашей учетной записи Azure Cosmos DB. Дополнительные сведения см. в статье [Защита доступа к данным Azure Cosmos DB](./secure-access-to-data.md#primary-keys).
`DatabaseName`, `CollectionName`|**Названия целевой базы данных и коллекции**. Если `ShouldCleanupOnStart` присвоено значение `true`, эти значения, наряду с `CollectionThroughput`, используются для очистки и создания новой базы данных и коллекции. Аналогично, если `ShouldCleanupOnFinish` присвоено значение `true`, база данных будет удалена по окончании приема данных. Обратите внимание: целевая коллекция должна быть **неограниченной**.
`CollectionThroughput`|Применяется для создания новой коллекции, если параметру `ShouldCleanupOnStart` присвоено значение `true`.
`ShouldCleanupOnStart`|Сбрасывает учетную запись базы данных и коллекции до запуска программы, а затем создает новые со значениями `DatabaseName`, `CollectionName` и `CollectionThroughput`.
`ShouldCleanupOnFinish`|Сбрасывает учетную запись базы данных и коллекции с определенными значениями `DatabaseName` и `CollectionName` после запуска программы.
`NumberOfDocumentsToImport`|Определяет количество тестовых вершин и ребер, которые будут созданы в примере. Это количество относится и к вершинам, и к ребрам.
`NumberOfBatches`|Определяет количество тестовых вершин и ребер, которые будут созданы в примере. Это количество относится и к вершинам, и к ребрам.
`CollectionPartitionKey`|Применяется для создания тестовых вершин и ребер, если это свойство присвоено автоматически. Также используется при повторном создании базы данных и коллекций, если `ShouldCleanupOnStart` присвоено значение `true`.

### <a name="run-the-sample-application"></a>Запуск примера приложения

1. Добавьте параметры конкретной базы данных в `App.config`, чтобы создать экземпляр DocumentClient. Если базы данных и контейнера еще нет, они будут созданы автоматически.
2. Запустите приложение. `BulkImportAsync` будет вызываться дважды: при первом вызове импортируются вершины, при втором — ребра. Если вставка каких-либо объектов приводит к ошибке, они будут добавлены в `.\BadVertices.txt` или `.\BadEdges.txt`.
3. Оцените результаты, отправив запрос в базу данных графа. Если `ShouldCleanupOnFinish` имеет значение true, база данных будет автоматически удалена.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о пакете NuGet и заметках о выпуске библиотеки NET выполнителя .NET см. в разделе [Подробности пакета SDK](sql-api-sdk-bulk-executor-dot-net.md)для пакетного исполнителя. 
* Ознакомьтесь с [советами по повышению производительности](./bulk-executor-dot-net.md#performance-tips) при использовании библиотеки массового исполнителя.
* Прочитайте [справочную статью о BulkExecutor.Graph](/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.graph) с дополнительными сведениями о классах и методах, указанных в этом пространстве имен.