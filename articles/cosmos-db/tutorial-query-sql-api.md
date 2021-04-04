---
title: Руководство по Как выполнять запросы с помощью SQL в базе данных Azure Cosmos DB
description: Руководство по выполнению запросов SQL в Azure Cosmos DB с помощью Площадки для тестирования запросов
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.custom: tutorial-develop, mvc
ms.topic: tutorial
ms.date: 11/05/2019
ms.reviewer: sngun
ms.openlocfilehash: c206b68495b09988e49f37433ccd5616acd29cb7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93337784"
---
# <a name="tutorial-query-azure-cosmos-db-by-using-the-sql-api"></a>Руководство по Выполнение запросов в Azure Cosmos DB с использованием API SQL
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

[API SQL](./introduction.md) базы данных Azure Cosmos DB позволяет выполнять запросы к документам с помощью SQL. В этой статье содержится пример документа и два примера SQL-запросов и результатов.

В этой статье рассматриваются следующие задачи: 

> [!div class="checklist"]
> * Выполнение запросов к данным с помощью SQL.

## <a name="sample-document"></a>Пример документа

Запросы SQL в этой статье используют следующий пример документа.

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
        "gender": "female", "grade": 1,
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

## <a name="where-can-i-run-sql-queries"></a>Где могут выполняться SQL-запросы

Вы можете выполнять запросы с помощью обозревателя данных на портале Azure, через [REST API и пакеты SDK](sql-api-sdk-dotnet.md) и даже через [площадку для запросов](https://www.documentdb.com/sql/demo), которая выполняет запросы с использованием существующего набора примера данных.

Дополнительные сведения об SQL-запросах см. в статье:
* [SQL-запросы и синтаксис SQL в DocumentDB](sql-query-getting-started.md)

## <a name="prerequisites"></a>Предварительные требования

В этом руководстве предполагается, что у вас есть учетная запись и коллекция базы данных Azure Cosmos DB. Если у вас их нет, выполните указанные ниже действия. Выполните задачи [5-минутного краткого руководства](create-cosmosdb-resources-portal.md).

## <a name="example-query-1"></a>Пример запроса 1

С учетом приведенного выше образца документа со сведениями о семье следующий SQL-запрос возвращает документы, в которых поле идентификатора соответствует `WakefieldFamily`. Так как это инструкция `SELECT *`, результатом выполнения запроса является сформированный документ JSON:

**Запрос**

```sql
    SELECT * 
    FROM Families f 
    WHERE f.id = "WakefieldFamily"
```

**Результаты**

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
        "gender": "female", "grade": 1,
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

## <a name="example-query-2"></a>Пример запроса 2

Следующий запрос возвращает все заданные имена детей в семье, идентификатор которых соответствует `WakefieldFamily`.

**Запрос**

```sql
    SELECT c.givenName 
    FROM Families f 
    JOIN c IN f.children 
    WHERE f.id = 'WakefieldFamily'
```

**Результаты**

```
[
    {
        "givenName": "Jesse"
    },
    {
        "givenName": "Lisa"
    }
]
```


## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике были выполнены следующие задачи:

> [!div class="checklist"]
> * Вы научились выполнять запросы с помощью SQL.  

Теперь вы можете приступать к следующему руководству, чтобы узнать, как глобально распределять данные.

> [!div class="nextstepaction"]
> [Глобальное распределение данных](tutorial-global-distribution-sql-api.md)