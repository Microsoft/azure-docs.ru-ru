---
title: Работа с датами в Azure Cosmos DB
description: Узнайте, как хранить, индексировать и запрашивать объекты времени в Azure Cosmos DB
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
author: SnehaGunda
ms.author: sngun
ms.topic: conceptual
ms.date: 04/03/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: ba4d4e63bdd1e795bc1c599d0eae8a595aa0d643
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359529"
---
# <a name="working-with-dates-in-azure-cosmos-db"></a>Работа с датами в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Azure Cosmos DB обеспечивает гибкость схемы данных и обширные возможности индексирования благодаря использованию собственной модели данных на основе [JSON](https://www.json.org). Все ресурсы Azure Cosmos DB, включая базы данных, контейнеры, документы и хранимые процедуры, моделируются и хранятся в виде документов JSON. Чтобы гарантировать свойство переносимости, JSON (и Azure Cosmos DB) поддерживает только небольшой набор основных типов: строка, число, логическое значение, массив, объект и Null. Но JSON благодаря своей гибкости позволяет разработчикам и платформам моделировать и более сложные типы, составляя их из поддерживаемых примитивов в виде объектов или массивов.

Помимо основных типов многим приложениям требуется формат DateTime для представления дат и отметок времени. В этой статье описывается, как разработчики могут хранить, извлекать и искать даты в Azure Cosmos DB с помощью пакета SDK для .NET.

## <a name="storing-datetimes"></a>Хранение значений даты и времени

Azure Cosmos DB поддерживает типы JSON, такие как строка, число, логическое значение, null, массив, объект. Он не поддерживает непосредственно тип DateTime. В настоящее время Azure Cosmos DB не поддерживает локализацию дат. Поэтому необходимо хранить значения DateTime в виде строк. Рекомендуемый формат для строк DateTime в Azure Cosmos DB соответствует `yyyy-MM-ddTHH:mm:ss.fffffffZ` стандарту ISO 8601 UTC. Рекомендуется хранить все даты в Azure Cosmos DB как время в формате UTC. Преобразование строк даты в этот формат позволит сортировать даты лексикографически. Если хранятся даты, отличные от времени в формате UTC, логика должна обрабатываться на стороне клиента. Чтобы преобразовать локальную дату и время в формат UTC, смещение должно быть известно или сохранено как свойство в JSON, а клиент может использовать смещение для расчета значения DateTime в формате UTC.

Запросы Range с строками DateTime в виде фильтров поддерживаются только в том случае, если строки DateTime находятся в формате UTC и имеют одинаковую длину. В Azure Cosmos DB системная функция [жеткуррентдатетиме](sql-query-getcurrentdatetime.md) будет возвращать текущее строковое значение даты и времени в формате UTC 8601 в формате: `yyyy-MM-ddTHH:mm:ss.fffffffZ` .

Многим приложениям вполне достаточно используемого по умолчанию строкового представления дат и времени, поскольку оно:

* позволяет сравнивать строки, а следовательно и сохранять относительный порядок значений даты и времени, после преобразования в строки;
* не требует создавать пользовательский код или специальные атрибуты для преобразования JSON;
* сохраняет даты в понятном для человека формате JSON;
* Позволяет воспользоваться преимуществами индексов Azure Cosmos DB для быстрой обработки запросов.

Например, следующий фрагмент кода сохраняет объект `Order`, содержащий два свойства в формате даты и времени (`ShipDate` и `OrderDate`) в виде документа, использующего пакет SDK для .NET.

```csharp
    public class Order
    {
        [JsonProperty(PropertyName="id")]
        public string Id { get; set; }
        public DateTime OrderDate { get; set; }
        public DateTime ShipDate { get; set; }
        public double Total { get; set; }
    }

    await container.CreateItemAsync(
        new Order
        {
            Id = "09152014101",
            OrderDate = DateTime.UtcNow.AddDays(-30),
            ShipDate = DateTime.UtcNow.AddDays(-14),
            Total = 113.39
        });
```

Этот документ хранится в Azure Cosmos DB в следующем виде:

```json
    {
        "id": "09152014101",
        "OrderDate": "2014-09-15T23:14:25.7251173Z",
        "ShipDate": "2014-09-30T23:14:25.7251173Z",
        "Total": 113.39
    }
```  

Кроме того, вы можете хранить даты как отметки времени Unix, то есть как число, представляющее количество секунд, прошедших с 1 января 1970 года. Внутреннее свойство Azure Cosmos DB для отметки времени (`_ts`) использует именно этот подход. Для сериализации значений даты и времени в числа используйте класс [UnixDateTimeConverter](/dotnet/api/microsoft.azure.documents.unixdatetimeconverter).

## <a name="querying-datetimes-in-linq"></a>Запрос по датам и времени в LINQ

Пакет SDK .NET для SQL автоматически поддерживает запросы с использованием LINQ к данным, хранящимся в Azure Cosmos DB. Например, в следующем фрагменте кода показан запрос LINQ, фильтрующий заказы, которые были отгружены за последние три дня:

```csharp
    IQueryable<Order> orders = container.GetItemLinqQueryable<Order>(allowSynchronousQueryExecution: true).Where(o => o.ShipDate >= DateTime.UtcNow.AddDays(-3));
```

Преобразуется в следующую инструкцию SQL и выполняется в Azure Cosmos DB:

```sql
    SELECT * FROM root WHERE (root["ShipDate"] >= "2014-09-30T23:14:25.7251173Z")
```

Дополнительные сведения о языке запросов SQL Azure Cosmos DB и поставщике LINQ см. в статье [запрос Cosmos DB в LINQ](sql-query-linq-to-sql.md).

## <a name="indexing-datetimes-for-range-queries"></a>Индексирование значений даты и времени для запросов в диапазоне

Запросы обычно имеют значения типа DateTime. Для эффективного выполнения этих запросов необходимо иметь индекс, определенный для всех свойств в фильтре запроса.

Дополнительные сведения о настройке политик индексирования см. в статье [Как работает индексирование данных в Azure Cosmos DB?](index-policy.md). 

## <a name="next-steps"></a>Дальнейшие действия

* Скачайте и запустите [примеры кода, хранящиеся на GitHub](https://github.com/Azure/azure-cosmos-dotnet-v2/tree/master/samples/code-samples).
* Дополнительные сведения об [SQL-запросах](sql-query-getting-started.md)
* Дополнительные сведения о политиках индексации в Azure Cosmos DB см. в [этой статье](index-policy.md).