---
title: Найти плату за единицу запроса (RU) для API Cassandra запроса в Azure Cosmos DB
description: Узнайте, как найти плату за единицу запроса (RU) для запросов Cassandra, выполняемых в контейнере Cosmos для Azure. Вы можете использовать драйверы портал Azure, .NET и Java для поиска оплаты за ЕДИНИЦу.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: b504702ad0c74ae9728c0a8b34fa94df26184f70
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103200512"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-cassandra-api"></a>Найдите плату за единицу запросов для операций, выполненных в Azure Cosmos DB API Cassandra
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

Azure Cosmos DB поддерживает многие интерфейсы API, такие как SQL, MongoDB, Cassandra, Gremlin и Таблицы. У каждого API есть собственный набор операций базы данных, начиная от простого считывания и записи точек и заканчивая сложными запросами. Каждая операция базы данных потребляет системные ресурсы. Потребление зависит от сложности операции.

Стоимость всех операций базы данных нормализуется с помощью Azure Cosmos DB и выражается в единицах запроса (ЕЗ). Плата за запросы — это единицы запросов, потребляемые всеми операциями базы данных. Можно считать, что в качестве валюты производительности разрабатываются системные ресурсы, такие как ЦП, операции ввода-вывода и память, необходимые для выполнения операций с базой данных, поддерживаемых Azure Cosmos DB. Независимо от того, какие API вы используете для взаимодействия с контейнером Azure Cosmos, затраты всегда измеряются в ЕЗ. Является ли операция базы данных записью, чтением точки или запросом, затраты всегда измеряются в версии-получателей. Дополнительные сведения см. в статье [единицы запросов и рекомендации по ИТ](request-units.md) .

В этой статье представлены различные способы, с помощью которых можно найти потребление [единиц запросов](request-units.md) (ru) для любой операции, выполняемой для контейнера в Azure Cosmos DB API Cassandra. Если вы используете другой API, ознакомьтесь со статьей [API для MongoDB](find-request-unit-charge-mongodb.md), [API SQL](find-request-unit-charge.md), API [Gremlin](find-request-unit-charge-gremlin.md)и [API ТАБЛИЦ](find-request-unit-charge-table.md) статей, чтобы найти плату за единицу.

Если вы выполняете операции с API Cassandra для Azure Cosmos DB, сведения о расходах в ЕЗ возвращаются во входящих полезных данных в поле с именем `RequestCharge`. Есть несколько способов получения данных о расходах в ЕЗ.

## <a name="use-the-net-sdk"></a>Использование пакета SDK для .NET

При использовании [пакета SDK для .NET](https://www.nuget.org/packages/CassandraCSharpDriver/) вы можете получить входящие полезные данные в свойстве `Info` объекта `RowSet`.

```csharp
RowSet rowSet = session.Execute("SELECT table_name FROM system_schema.tables;");
double requestCharge = BitConverter.ToDouble(rowSet.Info.IncomingPayload["RequestCharge"].Reverse().ToArray(), 0);
```

Дополнительные сведения см. [в разделе Краткое руководство. Создание приложения Cassandra с помощью пакета SDK для .NET и Azure Cosmos DB](create-cassandra-dotnet.md).

## <a name="use-the-java-sdk"></a>Использование пакета SDK для Java

При использовании [пакета SDK для Java](https://mvnrepository.com/artifact/com.datastax.cassandra/cassandra-driver-core) входные полезные данные можно получить, вызвав метод `getExecutionInfo()` для объекта `ResultSet`.

```java
ResultSet resultSet = session.execute("SELECT table_name FROM system_schema.tables;");
Double requestCharge = resultSet.getExecutionInfo().getIncomingPayload().get("RequestCharge").getDouble();
```

Дополнительные сведения см. [в разделе Краткое руководство. Создание приложения Cassandra с помощью пакета SDK для Java и Azure Cosmos DB](create-cassandra-java.md).

## <a name="next-steps"></a>Дальнейшие действия

Сведения об оптимизации потребления ЕЗ см. в следующих статьях:

* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)
* [Optimize provisioned throughput cost in Azure Cosmos DB](optimize-cost-throughput.md) (Оптимизация стоимости подготовленной пропускной способности в Azure Cosmos DB)
* [Optimize query cost in Azure Cosmos DB](./optimize-cost-reads-writes.md) (Оптимизация затрат на отправку запросов в Azure Cosmos DB)