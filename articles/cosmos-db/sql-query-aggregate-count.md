---
title: ЧИСЛО Azure Cosmos DB языке запросов
description: Сведения о системной функции SQL Count (COUNT) в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 5228558f4bcb146ec08ee5fff45fb1bdf4d56f01
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96553404"
---
# <a name="count-azure-cosmos-db"></a>Количество (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Эта системная функция возвращает количество значений в выражении.
  
## <a name="syntax"></a>Синтаксис
  
```sql
COUNT(<scalar_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*scalar_expr*  
   Любое скалярное выражение
  
## <a name="return-types"></a>Типы возвращаемых данных
  
Возвращает числовое выражение.  
  
## <a name="examples"></a>Примеры
  
В следующем примере возвращается общее число элементов в контейнере:
  
```sql
SELECT COUNT(1)
FROM c
``` 
ЧИСЛО может принимать любое скалярное выражение в качестве входных данных. В приведенном ниже запросе будут получены эквивалентные результаты.

```sql
SELECT COUNT(2)
FROM c
```

## <a name="remarks"></a>Remarks

Эта системная функция будет использовать [индекс диапазона](index-policy.md#includeexclude-strategy) для всех свойств в фильтре запроса.

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции в Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции в Azure Cosmos DB](sql-query-system-functions.md)
- [Агрегатные функции в Azure Cosmos DB](sql-query-aggregate-functions.md)