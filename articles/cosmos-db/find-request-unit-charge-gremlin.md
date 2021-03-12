---
title: Найти плату за единицу запроса (RU) для запросов API Gremlin в Azure Cosmos DB
description: Узнайте, как найти плату за единицу запроса (RU) для запросов Gremlin, выполняемых в контейнере Cosmos для Azure. Для поиска оплаты за ЕДИНИЦу можно использовать драйверы портал Azure, .NET, Java.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-graph
ms.topic: how-to
ms.date: 10/14/2020
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: 295ed3eb2312a5f614a4214b5b627f5657fba1eb
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103201346"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-gremlin-api"></a>Поиск затрат единиц запросов для операций, выполненных в Azure Cosmos DB API Gremlin
[!INCLUDE[appliesto-gremlin-api](includes/appliesto-gremlin-api.md)]

Azure Cosmos DB поддерживает многие интерфейсы API, такие как SQL, MongoDB, Cassandra, Gremlin и Таблицы. У каждого API есть собственный набор операций базы данных, начиная от простого считывания и записи точек и заканчивая сложными запросами. Каждая операция базы данных потребляет системные ресурсы. Потребление зависит от сложности операции.

Стоимость всех операций базы данных нормализуется с помощью Azure Cosmos DB и выражается в единицах запроса (ЕЗ). Плата за запросы — это единицы запросов, потребляемые всеми операциями базы данных. Можно считать, что в качестве валюты производительности разрабатываются системные ресурсы, такие как ЦП, операции ввода-вывода и память, необходимые для выполнения операций с базой данных, поддерживаемых Azure Cosmos DB. Независимо от того, какие API вы используете для взаимодействия с контейнером Azure Cosmos, затраты всегда измеряются в ЕЗ. Является ли операция базы данных записью, чтением точки или запросом, затраты всегда измеряются в версии-получателей. Дополнительные сведения см. в статье [единицы запросов и рекомендации по ИТ](request-units.md) .

В этой статье представлены различные способы, с помощью которых можно найти потребление [единиц запросов](request-units.md) (ru) для любой операции, выполняемой для контейнера в Azure Cosmos DB API Gremlin. Если вы используете другой API, ознакомьтесь со статьей [API для MongoDB](find-request-unit-charge-mongodb.md), [API Cassandra](find-request-unit-charge-cassandra.md), [API SQL](find-request-unit-charge.md)и [API ТАБЛИЦ](find-request-unit-charge-table.md) статей, чтобы найти плату за единицу.

Заголовки, возвращаемые API Gremlin, сопоставляются с пользовательскими атрибутами состояния, которые сейчас предоставляются через пакеты SDK Gremlin для .NET и Java. Сведения о стоимости запроса доступны в ключе `x-ms-request-charge`. При использовании API Gremlin оценить потребление в ЕЗ для операций в контейнере Azure Cosmos можно несколькими способами.

## <a name="use-the-azure-portal"></a>Использование портала Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте учетную запись Azure Cosmos](create-graph-gremlin-console.md#create-a-database-account) и заполните ее данными или выберите существующую учетную запись, которая уже содержит данные.

1. Перейдите на панель **обозревателя данных** и выберите контейнер, с которым собираетесь работать.

1. Введите допустимый запрос и нажмите кнопку **выполнить запрос Gremlin**.

1. Выберите **Статистика запросов**, чтобы отобразилось количество ЕЗ, фактически затраченных на выполненный запрос.

:::image type="content" source="./media/find-request-unit-charge/portal-gremlin-query.png" alt-text="Снимок экрана для получения Gremlin запроса на запрос в портал Azure":::

## <a name="use-the-net-sdk-driver"></a>Использование драйвера пакета SDK для .NET

При использовании [пакета SDK Gremlin для .NET](https://www.nuget.org/packages/Gremlin.Net/) атрибуты состояния доступны в свойстве `StatusAttributes` объекта `ResultSet<>`.

```csharp
ResultSet<dynamic> results = client.SubmitAsync<dynamic>("g.V().count()").Result;
double requestCharge = (double)results.StatusAttributes["x-ms-request-charge"];
```

Дополнительные сведения см. в разделе [Краткое руководство. создание платформа .NET Framework или основного приложения с помощью учетной записи API Azure Cosmos DB Gremlin](create-graph-dotnet.md).

## <a name="use-the-java-sdk-driver"></a>Использование драйвера пакета SDK для Java

При использовании [пакета SDK Gremlin для Java](https://mvnrepository.com/artifact/org.apache.tinkerpop/gremlin-driver) атрибуты состояния можно получить, вызвав метод `statusAttributes()` для объекта `ResultSet`.

```java
ResultSet results = client.submit("g.V().count()");
Double requestCharge = (Double)results.statusAttributes().get().get("x-ms-request-charge");
```

Дополнительные сведения см. [в разделе Краткое руководство. Создание базы данных Graph в Azure Cosmos DB с помощью пакета SDK для Java](create-graph-java.md).

## <a name="next-steps"></a>Дальнейшие действия

Сведения об оптимизации потребления ЕЗ см. в следующих статьях:

* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)
* [Optimize provisioned throughput cost in Azure Cosmos DB](optimize-cost-throughput.md) (Оптимизация стоимости подготовленной пропускной способности в Azure Cosmos DB)
* [Optimize query cost in Azure Cosmos DB](./optimize-cost-reads-writes.md) (Оптимизация затрат на отправку запросов в Azure Cosmos DB)