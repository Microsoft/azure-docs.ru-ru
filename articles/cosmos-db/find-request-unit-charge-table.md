---
title: Найти плату за единицу запроса (RU) для запросов API таблиц в Azure Cosmos DB
description: Узнайте, как найти плату за единицу запроса (RU) для API таблиц запросов, выполняемых в контейнере Azure Cosmos. Для поиска оплаты за ЕДИНИЦу можно использовать языки портал Azure, .NET, Java, Python и Node.js.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: 0c3f2495630358f48900a9cf27cbc0feb4192e1d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103201310"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-table-api"></a>Найдите плату за единицу запросов для операций, выполненных в Azure Cosmos DB API таблиц
[!INCLUDE[appliesto-table-api](includes/appliesto-table-api.md)]

Azure Cosmos DB поддерживает многие интерфейсы API, такие как SQL, MongoDB, Cassandra, Gremlin и Таблицы. У каждого API есть собственный набор операций базы данных, начиная от простого считывания и записи точек и заканчивая сложными запросами. Каждая операция базы данных потребляет системные ресурсы. Потребление зависит от сложности операции.

Стоимость всех операций базы данных нормализуется с помощью Azure Cosmos DB и выражается в единицах запроса (ЕЗ). Плата за запросы — это единицы запросов, потребляемые всеми операциями базы данных. Можно считать, что в качестве валюты производительности разрабатываются системные ресурсы, такие как ЦП, операции ввода-вывода и память, необходимые для выполнения операций с базой данных, поддерживаемых Azure Cosmos DB. Независимо от того, какие API вы используете для взаимодействия с контейнером Azure Cosmos, затраты всегда измеряются в ЕЗ. Является ли операция базы данных записью, чтением точки или запросом, затраты всегда измеряются в версии-получателей. Дополнительные сведения см. в статье [единицы запросов и рекомендации по ИТ](request-units.md) .

В этой статье представлены различные способы, с помощью которых можно найти потребление [единиц запросов](request-units.md) (ru) для любой операции, выполняемой для контейнера в Azure Cosmos DB API таблиц. Если вы используете другой API, ознакомьтесь со статьей [API для MongoDB](find-request-unit-charge-mongodb.md), [API Cassandra](find-request-unit-charge-cassandra.md), [API Gremlin](find-request-unit-charge-gremlin.md)и [SQL API](find-request-unit-charge.md) , чтобы найти единицы оплаты за единицу.

## <a name="use-the-net-sdk"></a>Использование пакета SDK для .NET

Единственный пакет SDK, который сейчас возвращает данные о расходах в ЕЗ для операций с таблицами, — это [пакет SDK для .NET Standard](https://www.nuget.org/packages/Microsoft.Azure.Cosmos.Table). Объект `TableResult` предоставляет свойство `RequestCharge`, которое указывается пакетом SDK, при использовании с API таблиц Azure Cosmos DB.

```csharp
CloudTable tableReference = client.GetTableReference("table");
TableResult tableResult = tableReference.Execute(TableOperation.Insert(new DynamicTableEntity("partitionKey", "rowKey")));
if (tableResult.RequestCharge.HasValue) // would be false when using Azure Storage Tables
{
    double requestCharge = tableResult.RequestCharge.Value;
}
```

Дополнительные сведения см. [в разделе Краткое руководство. создание API таблиц приложения с помощью пакета SDK для .NET и Azure Cosmos DB](create-table-dotnet.md).

## <a name="next-steps"></a>Следующие шаги

Сведения об оптимизации потребления ЕЗ см. в следующих статьях:

* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)
* [Optimize provisioned throughput cost in Azure Cosmos DB](optimize-cost-throughput.md) (Оптимизация стоимости подготовленной пропускной способности в Azure Cosmos DB)
* [Optimize query cost in Azure Cosmos DB](./optimize-cost-reads-writes.md) (Оптимизация затрат на отправку запросов в Azure Cosmos DB)