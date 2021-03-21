---
title: Создание контейнера в Azure Cosmos DB API для MongoDB
description: Узнайте, как создать контейнер в Azure Cosmos DB API для MongoDB с помощью портал Azure, .NET, Java, Node.js и других пакетов SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 10/16/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: a5669b15c041f663605a62ef8d02b206928d0c14
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93101601"
---
# <a name="create-a-container-in-azure-cosmos-db-api-for-mongodb"></a>Создание контейнера в Azure Cosmos DB API для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

В этой статье объясняются различные способы создания контейнера в Azure Cosmos DB API для MongoDB. В нем показано, как создать контейнер с помощью портал Azure, Azure CLI, PowerShell или поддерживаемых пакетов SDK. В этой статье показано, как создать контейнер, указать ключ секции и подготовить пропускную способность.

В этой статье объясняются различные способы создания контейнера в Azure Cosmos DB API для MongoDB. Если вы используете другой API, см. статью создание контейнера с помощью [API SQL](how-to-create-container.md), [API Cassandra](how-to-create-container-cassandra.md), [API Gremlin](how-to-create-container-gremlin.md)и [API таблиц](how-to-create-container-table.md) .

> [!NOTE]
> При создании контейнеров следите за тем, чтобы не создать два контейнера с одинаковыми именами, но в разных регистрах. Причина в том, что некоторые компоненты платформы Azure не учитывают регистр. Это может привести к путанице или конфликтам в данных телеметрии и действиях с контейнерами с такими именами.

## <a name="create-using-azure-portal"></a><a id="portal-mongodb"></a>Создание с помощью портал Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account)или выберите существующую.

1. Откройте панель **Обозреватель данных** и выберите **создать контейнер**. После этого предоставьте следующие сведения.

   * Укажите, создаете ли вы новую базу данных или используете существующую.
   * Введите идентификатор контейнера.
   * Введите ключ сегмента.
   * Укажите подготавливаемую пропускную способность (например, 1000 ЕЗ/с).
   * Щелкните **ОК**.

    :::image type="content" source="./media/how-to-create-container/partitioned-collection-create-mongodb.png" alt-text="Снимок экрана: Azure Cosmos DB API для MongoDB, диалоговое окно &quot;Добавление контейнера&quot;":::

## <a name="create-using-net-sdk"></a><a id="dotnet-mongodb"></a>Создание с помощью пакета SDK для .NET

```csharp
// Create a collection with a partition key by using Mongo Shell:
db.runCommand( { shardCollection: "myDatabase.myCollection", key: { myShardKey: "hashed" } } )
```

> [!Note]
> Сетевой протокол MongoDB не поддерживает понятие [единицы запроса](request-units.md). Чтобы создать коллекцию с подготовленной пропускной способностью, используйте портал Azure или пакеты SDK Cosmos DB для API SQL.

При возникновении исключения timeout при создании коллекции выполните операцию чтения, чтобы проверить, успешно ли создана коллекция. Операция чтения создает исключение до тех пор, пока операция создания коллекции не будет выполнена успешно. Список кодов состояния, поддерживаемых операцией Create, см. в статье [коды состояния HTTP для Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb) .

## <a name="create-using-azure-cli"></a><a id="cli-mongodb"></a>Создание с помощью Azure CLI

[Создайте коллекцию для Azure Cosmos DB API MongoDB с Azure CLI](./scripts/cli/mongodb/create.md). Список всех Azure CLI примеров для всех Azure Cosmos DB API см. в [Azure CLI примерах для Azure Cosmos DB](cli-samples.md).

## <a name="create-using-powershell"></a>Создание с помощью PowerShell

[Создайте коллекцию для Azure Cosmos DB API MongoDB с помощью PowerShell](./scripts/powershell/mongodb/create.md). Список всех примеров PowerShell для всех Azure Cosmos DB API см. в статье [примеры PowerShell](powershell-samples.md) .

## <a name="create-a-container-using-azure-resource-manager-templates"></a>Создание контейнера с помощью шаблонов Azure Resource Manager

[Создайте коллекцию для Azure Cosmos DB для API-интерфейса MongoDB с помощью шаблона диспетчер ресурсов](./manage-with-templates.md#azure-cosmos-account-with-standard-provisioned-throughput).

## <a name="next-steps"></a>Дальнейшие действия

* [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Секционирование в Azure Cosmos DB)
* [Единицы запросов в Azure Cosmos DB](request-units.md)
* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md)
* [Работа с учетной записью Azure Cosmos](./account-databases-containers-items.md)