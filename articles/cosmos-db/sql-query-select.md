---
title: Предложение SELECT в Azure Cosmos DB
description: Дополнительные сведения о предложении SQL SELECT для Azure Cosmos DB. Используйте SQL в качестве языка запросов JSON Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: tisande
ms.openlocfilehash: 072e17b1c0ea312b4adfa1687e447fd2cadde233
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93335452"
---
# <a name="select-clause-in-azure-cosmos-db"></a>Предложение SELECT в Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Каждый запрос состоит из `SELECT` предложения и необязательных предложений [from](sql-query-from.md) и [WHERE](sql-query-where.md) для стандартов ANSI SQL. Как правило, источник в `FROM` предложении перечисляется, а `WHERE` предложение применяет фильтр к источнику для получения подмножества элементов JSON. `SELECT`Затем предложение проецирует запрошенные значения JSON в списке выбора.

## <a name="syntax"></a>Синтаксис

```sql
SELECT <select_specification>  

<select_specification> ::=
      '*'
      | [DISTINCT] <object_property_list>
      | [DISTINCT] VALUE <scalar_expression> [[ AS ] value_alias]  
  
<object_property_list> ::=
{ <scalar_expression> [ [ AS ] property_alias ] } [ ,...n ]  
```  
  
## <a name="arguments"></a>Аргументы
  
- `<select_specification>`  

  Свойства или значения, выбираемые для результирующего набора.  
  
- `'*'`  

  Указывает, что значение необходимо извлечь без внесения изменений. В частности, если обработанное значение — это объект, извлекаются все свойства.  
  
- `<object_property_list>`  
  
  Указывает список свойств, которые требуется извлечь. Каждое возвращаемое значение представляет собой объект с указанными свойствами.  
  
- `VALUE`  

  Указывает, что следует извлечь только значение JSON, а не весь объект JSON. Этот аргумент, в отличие от аргумента `<property_list>`, не выделяет прогнозируемое значение в объекте.  

- `DISTINCT`
  
  Указывает, что необходимо удалить дубликаты проецируемых свойств.  

- `<scalar_expression>`  

  Выражение, представляющее вычисляемое значение. Дополнительные сведения см. в разделе [Скалярные выражения](sql-query-scalar-expressions.md).  

## <a name="remarks"></a>Remarks

Если предложение FROM объявило один псевдоним, действителен только синтаксис `SELECT *`. Синтаксис `SELECT *` обеспечивает проекцию удостоверения, что может пригодиться, если проекция не требуется. Кроме того, это * единственный действительный синтаксис, если в предложении FROM указан один источник входных данных.  
  
`SELECT <select_list>` и `SELECT *` — это "синтаксический сахар". При необходимости их можно выразить с помощью простых инструкций SELECT, как показано ниже.  
  
1. `SELECT * FROM ... AS from_alias ...`  
  
   эквивалентно правилу  
  
   `SELECT from_alias FROM ... AS from_alias ...`  
  
2. `SELECT <expr1> AS p1, <expr2> AS p2,..., <exprN> AS pN [other clauses...]`  
  
   эквивалентно правилу  
  
   `SELECT VALUE { p1: <expr1>, p2: <expr2>, ..., pN: <exprN> }[other clauses...]`  
  
## <a name="examples"></a>Примеры

Следующий пример запроса SELECT возвращает `address` из `Families` , чьи `id` соответствия `AndersenFamily` :

```sql
    SELECT f.address
    FROM Families f
    WHERE f.id = "AndersenFamily"
```

Результаты:

```json
    [{
      "address": {
        "state": "WA",
        "county": "King",
        "city": "Seattle"
      }
    }]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Начало работы](sql-query-getting-started.md)
- [Примеры .NET для Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Предложение WHERE](sql-query-where.md)
