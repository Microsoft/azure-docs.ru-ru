---
title: Язык запросов ROUND Azure Cosmos DB
description: Дополнительные сведения о функции SQL System ROUND в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 7dc4d78f7af1086f9a4de9aa7392acb388df966e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104590676"
---
# <a name="round-azure-cosmos-db"></a>ROUND (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает числовое значение, округленное до ближайшего целого значения в большую сторону.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
ROUND(<numeric_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*numeric_expr*  
   Числовое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает числовое выражение.  
  
## <a name="remarks"></a>Комментарии
  
Операция округления выполняется после среднего значения, округленного от нуля. Если входное значение является числовым выражением, которое находится в точности между двумя целыми числами, результатом будет ближайшее целое число от нуля. Эта системная функция будет использовать преимущества [индекса диапазона](index-policy.md#includeexclude-strategy).
  
|<numeric_expr>|Округляется|
|-|-|
|— 6,5000|-7|
|— 0,5|-1|
|0,5|1|
|6,5000|7|
  
## <a name="examples"></a>Примеры
  
В примере ниже положительные и отрицательные числа округляются до ближайшего целого.  
  
```sql
SELECT ROUND(2.4) AS r1, ROUND(2.6) AS r2, ROUND(2.5) AS r3, ROUND(-2.4) AS r4, ROUND(-2.6) AS r5  
```  
  
Результирующий набор:  
  
```json
[{r1: 2, r2: 3, r3: 3, r4: -2, r5: -3}]  
```  

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
