---
title: Создание контейнера в Azure Cosmos DB API Cassandra
description: Узнайте, как создать контейнер в Azure Cosmos DB API Cassandra с помощью портал Azure, .NET, Java, Python, Node.js и других пакетов SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 01030a563c15eee7786058c2eae30d23803dfe42
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93101652"
---
# <a name="create-a-container-in-azure-cosmos-db-cassandra-api"></a>Создание контейнера в Azure Cosmos DB API Cassandra
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

В этой статье описываются различные способы создания контейнера в Azure Cosmos DB API Cassandra. В нем показано, как создать контейнер с помощью портал Azure, Azure CLI, PowerShell или поддерживаемых пакетов SDK. В этой статье показано, как создать контейнер, указать ключ секции и подготовить пропускную способность.

В этой статье описываются различные способы создания контейнера в Azure Cosmos DB API Cassandra. Если вы используете другой API, см. статью [API for MongoDB](how-to-create-container-mongodb.md), [api Gremlin](how-to-create-container-gremlin.md), [API таблиц](how-to-create-container-table.md)и [API SQL](how-to-create-container.md) , чтобы создать контейнер.

> [!NOTE]
> При создании контейнеров следите за тем, чтобы не создать два контейнера с одинаковыми именами, но в разных регистрах. Причина в том, что некоторые компоненты платформы Azure не учитывают регистр. Это может привести к путанице или конфликтам в данных телеметрии и действиях с контейнерами с такими именами.

## <a name="create-using-azure-portal"></a><a id="portal-cassandra"></a>Создание с помощью портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure Cosmos](create-cassandra-dotnet.md#create-a-database-account)или выберите существующую.

1. Откройте панель **Обозреватель данных** и выберите **создать таблицу**. После этого предоставьте следующие сведения.

   * Укажите, создаете ли вы новое пространство ключей или используете существующее.
   * Введите имя таблицы.
   * Укажите свойства и первичный ключ.
   * Укажите подготавливаемую пропускную способность (например, 1000 ЕЗ/с).
   * Щелкните **ОК**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-cassandra.png" alt-text="Снимок экрана API Cassandra, диалоговое окно добавления таблицы":::

> [!NOTE]
> Для API Cassandra первичный ключ используется в качестве ключа секции.

## <a name="create-using-net-sdk"></a><a id="dotnet-cassandra"></a>Создание с помощью пакета SDK для .NET

```csharp
// Create a Cassandra table with a partition/primary key and provision 1000 RU/s throughput.
session.Execute(CREATE TABLE myKeySpace.myTable(
    user_id int PRIMARY KEY,
    firstName text,
    lastName text) WITH cosmosdb_provisioned_throughput=1000);
```

При возникновении исключения timeout при создании коллекции выполните операцию чтения, чтобы проверить, успешно ли создана коллекция. Операция чтения создает исключение до тех пор, пока операция создания коллекции не будет выполнена успешно. Список кодов состояния, поддерживаемых операцией Create, см. в статье [коды состояния HTTP для Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) .

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Создание с помощью Azure CLI

[Создайте таблицу Cassandra с Azure CLI](./scripts/cli/cassandra/create.md). Список всех Azure CLI примеров для всех Azure Cosmos DB API см. в [Azure CLI примерах для Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Создание с помощью PowerShell

[Создайте таблицу Cassandra с помощью PowerShell](./scripts/powershell/cassandra/create.md). Список всех примеров PowerShell для всех Azure Cosmos DB API см. в статье [примеры PowerShell](powershell-samples.md) .

## <a name="next-steps"></a>Дальнейшие действия

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Секционирование в Azure Cosmos DB)
* [Единицы запросов в Azure Cosmos DB](request-units.md)
* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md)
* [Работа с учетной записью Azure Cosmos](./account-databases-containers-items.md)