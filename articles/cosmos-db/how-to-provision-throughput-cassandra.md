---
title: Подготавливает пропускную способность для ресурсов Azure Cosmos DB API Cassandra
description: Узнайте, как подготавливать пропускную способность контейнера, базы данных и автомасштабирования в Azure Cosmos DB API Cassandraных ресурсах. Вы будете использовать портал Azure, CLI, PowerShell и различные другие пакеты SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mjbrown
ms.custom: devx-track-js, devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 63b633ed67c03a006a154bc69a1aafb4cb4aa6d0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93086284"
---
# <a name="provision-database-container-or-autoscale-throughput-on-azure-cosmos-db-cassandra-api-resources"></a>Подготавливает пропускную способность для базы данных, контейнера или автомасштабирования в ресурсах Azure Cosmos DB API Cassandra
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

В этой статье объясняется, как подготавливать пропускную способность в Azure Cosmos DB API Cassandra. Можно подготавливать стандартные (ручные) или автомасштабированные пропускные способности в контейнере или базе данных и совместно использовать их в контейнерах базы данных. Пропускную способность можно подготавливать с помощью портал Azure, Azure CLI или пакетов SDK для Azure Cosmos DB.

Если вы используете другой API, см. статью [SQL API](how-to-provision-container-throughput.md), [API для MONGODB](how-to-provision-throughput-mongodb.md), [Gremlin API](how-to-provision-throughput-gremlin.md) для предоставления пропускной способности.

## <a name="azure-portal"></a><a id="portal-cassandra"></a> портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) или выберите существующую.

1. Откройте панель **Обозреватель данных** и выберите **создать таблицу**. После этого предоставьте следующие сведения.

   * Укажите, следует ли создать новый пространства ключей или использовать существующий. Выберите параметр обеспечить **пропускную способность базы данных** , если вы хотите обеспечить пропускную способность на уровне пространства ключей.
   * Введите идентификатор таблицы в команде CQL.
   * Введите значение первичного ключа (например, `/userrID` ).
   * Укажите подготавливаемую пропускную способность (например, 1000 ЕЗ/с).
   * Щелкните **ОК**.

    :::image type="content" source="./media/how-to-provision-throughput-cassandra/provision-table-throughput-portal-cassandra-api.png" alt-text="Снимок экрана обозреватель данных при создании новой коллекции с пропускной способностью уровня базы данных":::

> [!Note]
> При подготовке пропускной способности для контейнера в учетной записи Azure Cosmos, настроенной с помощью API Cassandra, используйте `/myPrimaryKey` для пути к ключу секции.

## <a name="net-sdk"></a><a id="dotnet-cassandra"></a> ПАКЕТ SDK ДЛЯ .NET

### <a name="provision-throughput-for-a-cassandra-table"></a>Подготавливает пропускную способность для таблицы Cassandra

```csharp
// Create a Cassandra table with a partition (primary) key and provision throughput of 400 RU/s
session.Execute("CREATE TABLE myKeySpace.myTable(
    user_id int PRIMARY KEY,
    firstName text,
    lastName text) WITH cosmosdb_provisioned_throughput=400");

```
Аналогичные команды могут выдаваться любым драйвером, совместимым с CQL.

### <a name="alter-or-change-throughput-for-a-cassandra-table"></a>Изменение или изменение пропускной способности для таблицы Cassandra

```csharp
// Altering the throughput too can be done through code by issuing following command
session.Execute("ALTER TABLE myKeySpace.myTable WITH cosmosdb_provisioned_throughput=5000");
```

Аналогичную команду можно выполнить с помощью любого драйвера, совместимого с CQL.

```csharp
// Create a Cassandra keyspace and provision throughput of 400 RU/s
session.Execute("CREATE KEYSPACE IF NOT EXISTS myKeySpace WITH cosmosdb_provisioned_throughput=400");
```

## <a name="azure-resource-manager"></a>Azure Resource Manager

Azure Resource Manager шаблоны можно использовать для инициализации пропускной способности автомасштабирования для ресурсов базы данных или уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. [в разделе Azure Resource Manager Templates for Azure Cosmos DB](templates-samples-cassandra.md) .

## <a name="azure-cli"></a>Azure CLI

Azure CLI можно использовать для инициализации пропускной способности автомасштабирования в базе данных или ресурсах уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. в разделе [Azure CLI Samples for Azure Cosmos DB](cli-samples-cassandra.md).

## <a name="azure-powershell"></a>Azure PowerShell

Azure PowerShell можно использовать для инициализации пропускной способности автомасштабирования в базе данных или ресурсах уровня контейнера для всех API-интерфейсов Azure Cosmos DB. Примеры см. в разделе [Azure PowerShell Samples for Azure Cosmos DB](powershell-samples-cassandra.md).

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать о подготовке пропускной способности в Cosmos DB, обратитесь к следующим статьям:

* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)