---
title: Найти плату за единицу запросов для Azure Cosmos DB API для операций MongoDB
description: Узнайте, как найти плату за единицу запроса (RU) для запросов MongoDB, выполняемых в контейнере Cosmos для Azure. Вы можете использовать драйверы портал Azure, MongoDB .NET, Java Node.js.
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 03/19/2021
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: 6b2944c1d29849ea44b5afd878d5b0e030358cc5
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104801831"
---
# <a name="find-the-request-unit-charge-for-operations-executed-in-azure-cosmos-db-api-for-mongodb"></a>Поиск затрат единиц запросов для операций, выполненных в Azure Cosmos DB API для MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Azure Cosmos DB поддерживает многие интерфейсы API, такие как SQL, MongoDB, Cassandra, Gremlin и Таблицы. У каждого API есть собственный набор операций базы данных, начиная от простого считывания и записи точек и заканчивая сложными запросами. Каждая операция базы данных потребляет системные ресурсы. Потребление зависит от сложности операции.

Стоимость всех операций базы данных нормализуется с помощью Azure Cosmos DB и выражается в единицах запроса (ЕЗ). Плата за запросы — это единицы запросов, потребляемые всеми операциями базы данных. Можно считать, что в качестве валюты производительности разрабатываются системные ресурсы, такие как ЦП, операции ввода-вывода и память, необходимые для выполнения операций с базой данных, поддерживаемых Azure Cosmos DB. Независимо от того, какие API вы используете для взаимодействия с контейнером Azure Cosmos, затраты всегда измеряются в ЕЗ. Является ли операция базы данных записью, чтением точки или запросом, затраты всегда измеряются в версии-получателей. Дополнительные сведения см. в статье [единицы запросов и рекомендации по ИТ](request-units.md) .

В этой статье представлены различные способы, с помощью которых можно найти потребление [единиц запросов](request-units.md) (ru) для любой операции, выполняемой с контейнером в Azure Cosmos DB API для MongoDB. Если вы используете другой API, ознакомьтесь со статьей использование [API SQL](find-request-unit-charge.md), [API Cassandra](find-request-unit-charge-cassandra.md), [API Gremlin](find-request-unit-charge-gremlin.md)и [API ТАБЛИЦ](find-request-unit-charge-table.md) статей, чтобы найти плату за единицу.

Расходы в ЕЗ предоставляются пользовательской [командой базы данных](https://docs.mongodb.com/manual/reference/command/) с именем `getLastRequestStatistics`. Эта команда возвращает документ, содержащий имя последней выполненной операции, ее длительность и количество затраченных ЕЗ. Если вы используете API Azure Cosmos DB для MongoDB, получить данные о расходах в ЕЗ можно несколькими способами.

## <a name="use-the-azure-portal"></a>Использование портала Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. [Создайте учетную запись Azure Cosmos](create-mongodb-dotnet.md#create-a-database-account) и заполните ее данными или выберите существующую учетную запись, которая уже содержит данные.

1. Перейдите на панель **обозревателя данных** и выберите контейнер, с которым собираетесь работать.

1. Нажмите кнопку **...** рядом с именем контейнера и выберите **создать запрос**.

1. Введите допустимый запрос и щелкните **Выполнить запрос**.

1. Выберите **Статистика запросов**, чтобы отобразилось количество ЕЗ, фактически затраченных на выполненный запрос. Этот редактор запросов позволяет выполнять и просматривать плату за единицу запросов только для предикатов запроса. Этот редактор нельзя использовать для команд обработки данных, таких как инструкции INSERT.

   :::image type="content" source="./media/find-request-unit-charge/portal-mongodb-query.png" alt-text="Снимок экрана со сведениями о расходах в ЕЗ для MongoDB на портале Azure":::

1. Чтобы получить плату за запросы для команд обработки данных, выполните `getLastRequestStatistics` команду из пользовательского интерфейса на основе оболочки, например Mongo Shell, [Совместная работа 3T](mongodb-robomongo.md), [MongoDB компаса](mongodb-compass.md)или расширения VS Code с помощью сценариев оболочки.

   `db.runCommand({getLastRequestStatistics: 1})`

## <a name="use-the-mongodb-net-driver"></a>Использование драйвера .NET для MongoDB

При использовании [официального драйвера .NET для MongoDB](https://docs.mongodb.com/ecosystem/drivers/csharp/) можно выполнять команды, вызывая метод `RunCommand` для объекта `IMongoDatabase`. Для работы этого метода необходимо реализовать абстрактный класс `Command<>`.

```csharp
class GetLastRequestStatisticsCommand : Command<Dictionary<string, object>>
{
    public override RenderedCommand<Dictionary<string, object>> Render(IBsonSerializerRegistry serializerRegistry)
    {
        return new RenderedCommand<Dictionary<string, object>>(new BsonDocument("getLastRequestStatistics", 1), serializerRegistry.GetSerializer<Dictionary<string, object>>());
    }
}

Dictionary<string, object> stats = database.RunCommand(new GetLastRequestStatisticsCommand());
double requestCharge = (double)stats["RequestCharge"];
```

Дополнительные сведения см. в разделе [Краткое руководство. Создание веб-приложения .NET с помощью Azure Cosmos DB API для MongoDB](create-mongodb-dotnet.md).

## <a name="use-the-mongodb-java-driver"></a>Использование драйвера Java для MongoDB


При использовании [официального драйвера Java для MongoDB](https://mongodb.github.io/mongo-java-driver/) можно выполнять команды, вызывая метод `runCommand` для объекта `MongoDatabase`.

```java
Document stats = database.runCommand(new Document("getLastRequestStatistics", 1));
Double requestCharge = stats.getDouble("RequestCharge");
```

Дополнительные сведения см. [в разделе Краткое руководство. Создание веб-приложения с помощью Azure Cosmos DB API для MongoDB и пакета SDK для Java](create-mongodb-java.md).

## <a name="use-the-mongodb-nodejs-driver"></a>Использование драйвера Node.js для MongoDB

При использовании [официального драйвера Node.js для MongoDB](https://mongodb.github.io/node-mongodb-native/) можно выполнять команды, вызывая метод `command` для объекта `db`.

```javascript
db.command({ getLastRequestStatistics: 1 }, function(err, result) {
    assert.equal(err, null);
    const requestCharge = result['RequestCharge'];
});
```

Дополнительные сведения см. [в разделе Краткое руководство. Перенос существующего веб-приложения MongoDB Node.js в Azure Cosmos DB](create-mongodb-nodejs.md).

## <a name="next-steps"></a>Следующие шаги

Сведения об оптимизации потребления ЕЗ см. в следующих статьях:

* [Пропускная способность и единицы запросов в Azure Cosmos DB](request-units.md)
* [Optimize provisioned throughput cost in Azure Cosmos DB](optimize-cost-throughput.md) (Оптимизация стоимости подготовленной пропускной способности в Azure Cosmos DB)
* [Optimize query cost in Azure Cosmos DB](./optimize-cost-reads-writes.md) (Оптимизация затрат на отправку запросов в Azure Cosmos DB)