---
title: Начало работы с запросами SQL в Azure Cosmos DB
description: Узнайте, как использовать SQL запросы для запроса данных из Azure Cosmos DB. Образец данных можно передать в контейнер в Azure Cosmos DB и выполнить запрос.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 02/02/2021
ms.author: tisande
ms.openlocfilehash: d5d5bc0a108cd08283ea29ce3bdc2de49310c5aa
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102499566"
---
# <a name="getting-started-with-sql-queries"></a>Getting started with SQL queries (Начало работы с запросами SQL)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

В Azure Cosmos DB учетных записей API SQL есть два способа чтения данных:

**Операции чтения точек** . Поиск ключа и значения можно выполнить по одному *идентификатору элемента* и ключу секции. Сочетание *идентификатора* и ключа секции — это ключ, а сам элемент — это значение. Для документа размером 1 КБ точка считывания обычно имеет 1 [единицу запроса](request-units.md) с задержкой менее 10 мс. Операции чтения точек возвращают один элемент.

Ниже приведены некоторые примеры того, как выполнять **операции чтения** с каждым пакетом SDK.

- [Пакет SDK для .NET](/dotnet/api/microsoft.azure.cosmos.container.readitemasync)
- [пакет SDK для Java](/java/api/com.azure.cosmos.cosmoscontainer.readitem#com_azure_cosmos_CosmosContainer__T_readItem_java_lang_String_com_azure_cosmos_models_PartitionKey_com_azure_cosmos_models_CosmosItemRequestOptions_java_lang_Class_T__)
- [Пакет SDK для Node.js](/javascript/api/@azure/cosmos/item#read-requestoptions-)
- [Пакет SDK для Python](/python/api/azure-cosmos/azure.cosmos.containerproxy#read-item-item--partition-key--populate-query-metrics-none--post-trigger-include-none----kwargs-)

**SQL-запросы** . Вы можете запрашивать данные, создавая запросы с помощью язык SQL (SQL) в качестве языка запросов JSON. Запросы всегда изменяют по крайней мере 2,3 единиц запросов и, как правило, имеют более высокую и более длительные переменные задержки, чем операции чтения точек. Запросы могут возвращать много элементов.

Большинство рабочих нагрузок с интенсивными операциями чтения в Azure Cosmos DB используют сочетание операций чтения с точки зрения и SQL-запросов. Если требуется только чтение одного элемента, то чтение точек выполняется дешевле и быстрее, чем запросы. Операции чтения точек не требуют использования обработчика запросов для доступа к данным и могут считывать данные напрямую. Конечно, все рабочие нагрузки не могут читать данные в монопольном режиме с помощью операций чтения точек, поэтому поддержка SQL в качестве языка запросов и [индексирования без использования схем](index-overview.md) обеспечивает более гибкий способ доступа к данным.

Ниже приведены некоторые примеры выполнения **запросов SQL** с каждым пакетом SDK.

- [Пакет SDK для .NET](./sql-api-dotnet-v3sdk-samples.md#query-examples)
- [пакет SDK для Java](./sql-api-java-sdk-samples.md#query-examples)
- [Пакет SDK для Node.js](./sql-api-nodejs-samples.md#item-examples)
- [Пакет SDK для Python](./sql-api-python-samples.md#item-examples)

Оставшаяся часть документа показывает, как приступить к написанию SQL запросов в Azure Cosmos DB. SQL-запросы можно выполнять с помощью пакета SDK или портал Azure.

## <a name="upload-sample-data"></a>Отправить образец данных

В учетной записи Cosmos DB API SQL откройте [Обозреватель данных](./data-explorer.md) , чтобы создать контейнер с именем `Families` . После создания используйте браузер структуры данных, чтобы найти и открыть его. В `Families` контейнере вы увидите `Items` параметр справа под именем контейнера. Откройте этот параметр, и вы увидите кнопку в строке меню в центре экрана, чтобы создать новый элемент. Эта функция будет использоваться для создания элементов JSON ниже.

### <a name="create-json-items"></a>Создание элементов JSON

Следующие два элемента JSON — это документы о семействах Андерсен и Вейкфилд. К ним относятся родители, дети и их pets, адрес и сведения о регистрации. 

Первый элемент содержит строки, числа, логические значения, массивы и вложенные свойства:

```json
{
  "id": "AndersenFamily",
  "lastName": "Andersen",
  "parents": [
     { "firstName": "Thomas" },
     { "firstName": "Mary Kay"}
  ],
  "children": [
     {
         "firstName": "Henriette Thaulow",
         "gender": "female",
         "grade": 5,
         "pets": [{ "givenName": "Fluffy" }]
     }
  ],
  "address": { "state": "WA", "county": "King", "city": "Seattle" },
  "creationDate": 1431620472,
  "isRegistered": true
}
```

Второй элемент использует `givenName` , а `familyName` вместо `firstName` и `lastName` :

```json
{
  "id": "WakefieldFamily",
  "parents": [
      { "familyName": "Wakefield", "givenName": "Robin" },
      { "familyName": "Miller", "givenName": "Ben" }
  ],
  "children": [
      {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1,
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
        ]
      },
      {
        "familyName": "Miller",
         "givenName": "Lisa",
         "gender": "female",
         "grade": 8 }
  ],
  "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
  "creationDate": 1431620462,
  "isRegistered": false
}
```

### <a name="query-the-json-items"></a>Запрос элементов JSON

Попробуйте выполнить несколько запросов к данным JSON, чтобы понять некоторые ключевые аспекты языка SQL-запросов Azure Cosmos DB.

Следующий запрос возвращает элементы, в которых `id` поле соответствует `AndersenFamily` . Поскольку это `SELECT *` запрос, результатом запроса является полный элемент JSON. Дополнительные сведения о синтаксисе SELECT см. в разделе [инструкция SELECT](sql-query-select.md).

```sql
    SELECT *
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Результаты запроса:

```json
    [{
        "id": "AndersenFamily",
        "lastName": "Andersen",
        "parents": [
           { "firstName": "Thomas" },
           { "firstName": "Mary Kay"}
        ],
        "children": [
           {
               "firstName": "Henriette Thaulow", "gender": "female", "grade": 5,
               "pets": [{ "givenName": "Fluffy" }]
           }
        ],
        "address": { "state": "WA", "county": "King", "city": "Seattle" },
        "creationDate": 1431620472,
        "isRegistered": true
    }]
```

Следующий запрос переформатирует выходные данные JSON в другую форму. Запрос проецирует новый `Family` объект JSON с двумя выбранными полями `Name` и `City` , когда город адреса совпадает с состоянием. «Москва» соответствует этому варианту.

```sql
    SELECT {"Name":f.id, "City":f.address.city} AS Family
    FROM Families f
    WHERE f.address.city = f.address.state
```

Результаты запроса:

```json
    [{
        "Family": {
            "Name": "WakefieldFamily",
            "City": "NY"
        }
    }]
```

Следующий запрос возвращает все заданные имена дочерних элементов в семействе, чьи `id` соответствия `WakefieldFamily` упорядочены по городу.

```sql
    SELECT c.givenName
    FROM Families f
    JOIN c IN f.children
    WHERE f.id = 'WakefieldFamily'
    ORDER BY f.address.city ASC
```

Результаты:

```json
    [
      { "givenName": "Jesse" },
      { "givenName": "Lisa"}
    ]
```

## <a name="remarks"></a>Комментарии

В предыдущих примерах показано несколько аспектов языка запросов Cosmos DB:  

* Так как API SQL работает с значениями JSON, он взаимодействует с сущностями в форме дерева, а не со строками и столбцами. Вы можете ссылаться на узлы дерева в любой произвольной глубине, например `Node1.Node2.Node3…..Nodem` , аналогично ссылке из двух частей `<table>.<column>` в ANSI SQL.

* Поскольку язык запросов работает с данными без схем, система типов должна быть динамически привязана. Одно и то же выражение может возвращать различные типы у разных элементов. Результатом запроса является допустимое значение JSON, но оно не обязательно должно быть фиксированной схемой.  

* Azure Cosmos DB поддерживает только элементы, строго соответствующие JSON. Система типов и выражения могут обрабатывать только типы JSON. Дополнительные сведения см. в [спецификациях JSON](https://www.json.org/).  

* Контейнер Cosmos — это коллекция элементов JSON без схемы. Отношения внутри и между элементами контейнера неявно фиксируются с помощью включения, а не связей первичного и внешнего ключей. Эта функция важна для соединений внутри элемента, описанных в разделе [объединения в Azure Cosmos DB](sql-query-join.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения об Azure Cosmos DB](introduction.md)
- [Примеры .NET для Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Предложение SELECT](sql-query-select.md)
