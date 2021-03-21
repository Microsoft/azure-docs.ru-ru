---
title: Работа с массивами и объектами в Azure Cosmos DB
description: Сведения о синтаксисе SQL для создания массивов и объектов в Azure Cosmos DB. В этой статье также приведены некоторые примеры выполнения операций с объектами Array.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 02/02/2021
ms.author: tisande
ms.openlocfilehash: 1dccb8e51fbc578f8f218fe1582f95f7bcaf42d7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99493796"
---
# <a name="working-with-arrays-and-objects-in-azure-cosmos-db"></a>Работа с массивами и объектами в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Ключевой функцией API Azure Cosmos DB SQL является создание массивов и объектов. В этом документе используются примеры, которые можно создать повторно с помощью [набора данных Family](sql-query-getting-started.md#upload-sample-data).

Ниже приведен пример элемента в этом наборе данных:

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

## <a name="arrays"></a>Массивы

Можно создавать массивы, как показано в следующем примере:

```sql
SELECT [f.address.city, f.address.state] AS CityState
FROM Families f
```

Результаты:

```json
[
  {
    "CityState": [
      "Seattle",
      "WA"
    ]
  },
  {
    "CityState": [
      "NY", 
      "NY"
    ]
  }
]
```

Можно также использовать [выражение массива](sql-query-subquery.md#array-expression) для создания массива из результатов [вложенного запроса](sql-query-subquery.md) . Этот запрос получает все уникальные заданные имена дочерних элементов в массиве.

```sql
SELECT f.id, ARRAY(SELECT DISTINCT VALUE c.givenName FROM c IN f.children) as ChildNames
FROM f
```

Результаты:

```json
[
    {
        "id": "AndersenFamily",
        "ChildNames": []
    },
    {
        "id": "WakefieldFamily",
        "ChildNames": [
            "Jesse",
            "Lisa"
        ]
    }
]
```

## <a name="iteration"></a><a id="Iteration"></a>Итерация

API SQL обеспечивает поддержку итерации по массивам JSON с [ключевым словом in](sql-query-keywords.md#in) в источнике from. В следующем примере:

```sql
SELECT *
FROM Families.children
```

Результаты:

```json
[
  [
    {
      "firstName": "Henriette Thaulow",
      "gender": "female",
      "grade": 5,
      "pets": [{ "givenName": "Fluffy"}]
    }
  ], 
  [
    {
        "familyName": "Merriam",
        "givenName": "Jesse",
        "gender": "female",
        "grade": 1
    }, 
    {
        "familyName": "Miller",
        "givenName": "Lisa",
        "gender": "female",
        "grade": 8
    }
  ]
]
```

Следующий запрос выполняет итерацию `children` в `Families` контейнере. Выходной массив отличается от предыдущего запроса. В этом примере показано разбиение `children` и сведение результатов в один массив:  

```sql
SELECT *
FROM c IN Families.children
```

Результаты:

```json
[
  {
      "firstName": "Henriette Thaulow",
      "gender": "female",
      "grade": 5,
      "pets": [{ "givenName": "Fluffy" }]
  },
  {
      "familyName": "Merriam",
      "givenName": "Jesse",
      "gender": "female",
      "grade": 1
  },
  {
      "familyName": "Miller",
      "givenName": "Lisa",
      "gender": "female",
      "grade": 8
  }
]
```

Можно выполнить фильтрацию для каждой отдельной записи массива, как показано в следующем примере:

```sql
SELECT c.givenName
FROM c IN Families.children
WHERE c.grade = 8
```

Результаты:

```json
[{
  "givenName": "Lisa"
}]
```

Можно также выполнить статистическую обработку по результату итерации массива. Например, следующий запрос подсчитывает количество дочерних элементов во всех семействах:

```sql
SELECT COUNT(1) AS Count
FROM child IN Families.children
```

Результаты:

```json
[
  {
    "Count": 3
  }
]
```

> [!NOTE]
> При использовании ключевого слова IN для итерации нельзя фильтровать или проецировать любые свойства за пределами массива. Вместо этого следует использовать [объединения](sql-query-join.md).

Дополнительные примеры см. [в записи блога о работе с массивами в Azure Cosmos DB](https://devblogs.microsoft.com/cosmosdb/understanding-how-to-query-arrays-in-azure-cosmos-db/).

## <a name="next-steps"></a>Дальнейшие действия

- [Начало работы](sql-query-getting-started.md)
- [Примеры .NET для Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Joins](sql-query-join.md)
