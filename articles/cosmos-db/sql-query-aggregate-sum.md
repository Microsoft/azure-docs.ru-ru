---
title: SUM на языке запросов Azure Cosmos DB
description: Сведения о системной функции SQL Sum (SUM) в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: d5a86cd5af504072480e0cd749caa6c0532d0bc1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96553428"
---
# <a name="sum-azure-cosmos-db"></a>SUM (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Эта агрегатная функция возвращает сумму значений в выражении.
  
## <a name="syntax"></a>Синтаксис
  
```sql
SUM(<numeric_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*numeric_expr*  
   Числовое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
Возвращает числовое выражение.  
  
## <a name="examples"></a>Примеры
  
В следующем примере возвращается сумма `propertyA` :
  
```sql
SELECT SUM(c.propertyA)
FROM c
```  

## <a name="remarks"></a>Remarks

Эта системная функция будет использовать преимущества [индекса диапазона](index-policy.md#includeexclude-strategy). Если какие бы то ни было аргументы в `SUM` строках типа String, Boolean или null, будет возвращена вся агрегатная системная функция `undefined` . Если какой бы то ни было аргумент имеет `undefined` значение, оно не будет влиять на `SUM` вычисление.

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции в Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции в Azure Cosmos DB](sql-query-system-functions.md)
- [Агрегатные функции в Azure Cosmos DB](sql-query-aggregate-functions.md)