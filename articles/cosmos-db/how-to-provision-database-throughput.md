---
title: Инициализация пропускной способности базы данных в Azure Cosmos DB API SQL
description: Узнайте, как подготавливать пропускную способность на уровне базы данных в Azure Cosmos DB API SQL с помощью портал Azure, CLI, PowerShell и других пакетов SDK.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 4ecbee2260da735cd6ba74d3b9ffb55b4a652e9e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93342006"
---
# <a name="provision-standard-manual-throughput-on-a-database-in-azure-cosmos-db---sql-api"></a>Предоставление стандартной (ручной) пропускной способности в базе данных в Azure Cosmos DB API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В этой статье объясняется, как подготавливать стандартную (ручную) пропускную способность для базы данных в Azure Cosmos DB API SQL. Вы можете подготовить пропускную способность для [одного](how-to-provision-container-throughput.md) контейнера или для базы данных и разделить ее между контейнерами внутри базы данных. Сведения о том, когда следует использовать пропускную способность уровня контейнера и уровня базы данных, см. в статье [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md). Вы можете подготовить пропускную способность уровня базы данных с помощью портала Azure или пакетов SDK для Azure Cosmos DB.

Если вы используете другой API, см. статью [API for MongoDB](how-to-provision-throughput-mongodb.md), [API Cassandra](how-to-provision-throughput-cassandra.md), [Gremlin API](how-to-provision-throughput-gremlin.md) для обеспечения пропускной способности.

## <a name="provision-throughput-using-azure-portal"></a>Подготовка пропускной способности с помощью портала Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте новую учетную запись Azure Cosmos](create-sql-api-dotnet.md#create-account) или выберите существующую.

1. Откройте панель **обозревателя данных** и выберите **Новая база данных**. Укажите следующие сведения:

   * Введите идентификатор базы данных.
   * Выберите параметр **готовить к пропускной способности базы данных** .
   * Укажите пропускную способность (например, 1000 ЕЗ/с).
   * Щелкните **ОК**.

    :::image type="content" source="./media/how-to-provision-database-throughput/provision-database-throughput-portal-sql-api.png" alt-text="Снимок экрана: диалоговое окно &quot;Новая база данных&quot;":::

## <a name="provision-throughput-using-azure-cli-or-powershell"></a>Подготовка пропускной способности с помощью портала Azure CLI или PowerShell

Чтобы получить сведения о создании базы данных с совместно используемой пропускной способностью см.:

* [Создание базы данных с помощью Azure CLI](manage-with-cli.md#create-a-database-with-shared-throughput)
* [Создание базы данных с помощью PowerShell](manage-with-powershell.md#create-db-ru)

## <a name="provision-throughput-using-net-sdk"></a>Подготовка пропускной способности с помощью пакета SDK для .NET

> [!Note]
> Для обеспечения пропускной способности для всех API-интерфейсов можно использовать пакеты SDK Azure Cosmos для API SQL. При необходимости также можно использовать указанный ниже пример для API Cassandra.

# <a name="net-sdk-v2"></a>[Пакет SDK для .NET версии 2](#tab/dotnetv2)

```csharp
//set the throughput for the database
RequestOptions options = new RequestOptions
{
    OfferThroughput = 500
};

//create the database
await client.CreateDatabaseIfNotExistsAsync(
    new Database {Id = databaseName},  
    options);
```

# <a name="net-sdk-v3"></a>[Пакет SDK для .NET версии 3](#tab/dotnetv3)

[!code-csharp[](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos/tests/Microsoft.Azure.Cosmos.Tests/SampleCodeForDocs/DatabaseDocsSampleCode.cs?name=DatabaseCreateWithThroughput)]

---

## <a name="next-steps"></a>Дальнейшие действия

См. подробнее о подготовке пропускной способности в Azure Cosmos DB:

* [Globally scale provisioned throughput](./request-units.md) (Глобальное масштабирование подготовленной пропускной способности)
* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md)
* [Подготовка стандартной (масштабируемой вручную) пропускной способности для контейнера](how-to-provision-container-throughput.md)
* [Подготовка автомасштабируемой пропускной способности для контейнера Azure Cosmos DB](how-to-provision-autoscale-throughput.md)
* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)