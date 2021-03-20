---
title: Создание контейнера в Azure Cosmos DB API Gremlin
description: Узнайте, как создать контейнер в Azure Cosmos DB API Gremlin с помощью портал Azure, .NET и других пакетов SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: f7e9de1f23ec46af08fe96b5db3170fac9a7eb2e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93101635"
---
# <a name="create-a-container-in-azure-cosmos-db-gremlin-api"></a>Создание контейнера в Azure Cosmos DB API Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

В этой статье объясняются различные способы создания контейнера в Azure Cosmos DB API Gremlin. В нем показано, как создать контейнер с помощью портал Azure, Azure CLI, PowerShell или поддерживаемых пакетов SDK. В этой статье показано, как создать контейнер, указать ключ секции и подготовить пропускную способность.

В этой статье объясняются различные способы создания контейнера в Azure Cosmos DB API Gremlin. Если вы используете другой API, см. статью [API for MongoDB](how-to-create-container-mongodb.md), [API Cassandra](how-to-create-container-cassandra.md), [API таблиц](how-to-create-container-table.md)и [API SQL](how-to-create-container.md) для создания контейнера.

> [!NOTE]
> При создании контейнеров следите за тем, чтобы не создать два контейнера с одинаковыми именами, но в разных регистрах. Причина в том, что некоторые компоненты платформы Azure не учитывают регистр. Это может привести к путанице или конфликтам в данных телеметрии и действиях с контейнерами с такими именами.

## <a name="create-using-azure-portal"></a><a id="portal-gremlin"></a>Создание с помощью портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure Cosmos](create-graph-dotnet.md#create-a-database-account)или выберите существующую.

1. Откройте панель **Обозреватель данных** и выберите **создать граф**. После этого предоставьте следующие сведения.

   * Укажите, создаете ли вы новую базу данных или используете существующую.
   * Укажите идентификатор графа.
   * Выберите емкость **Без ограничений**.
   * Введите ключ секции для вершин.
   * Укажите подготавливаемую пропускную способность (например, 1000 ЕЗ/с).
   * Щелкните **ОК**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-gremlin.png" alt-text="Снимок экрана API Gremlin, диалоговое окно добавления графа":::

## <a name="create-using-net-sdk"></a><a id="dotnet-sql-graph"></a>Создание с помощью пакета SDK для .NET

При возникновении исключения timeout при создании коллекции выполните операцию чтения, чтобы проверить, успешно ли создана коллекция. Операция чтения создает исключение до тех пор, пока операция создания коллекции не будет выполнена успешно. Список кодов состояния, поддерживаемых операцией Create, см. в статье [коды состояния HTTP для Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) .

```csharp
// Create a container with a partition key and provision 1000 RU/s throughput.
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "myContainerName";
myCollection.PartitionKey.Paths.Add("/myPartitionKey");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("myDatabaseName"),
    myCollection,
    new RequestOptions { OfferThroughput = 1000 });
```

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Создание с помощью Azure CLI

[Создание графа Gremlin с Azure CLI](./scripts/cli/gremlin/create.md). Список всех Azure CLI примеров для всех Azure Cosmos DB API см. в [Azure CLI примерах для Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Создание с помощью PowerShell

[Создание графа Gremlin с помощью PowerShell](./scripts/powershell/gremlin/create.md). Список всех примеров PowerShell для всех Azure Cosmos DB API см. в статье [примеры PowerShell](powershell-samples.md) .

## <a name="next-steps"></a>Дальнейшие действия

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Секционирование в Azure Cosmos DB)
* [Единицы запросов в Azure Cosmos DB](request-units.md)
* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md)
* [Работа с учетной записью Azure Cosmos](./account-databases-containers-items.md)