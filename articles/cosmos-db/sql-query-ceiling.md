---
title: Azure Cosmos DB языка запросов
description: Узнайте о том, как функция CEILING SQL System в Azure Cosmos DB возвращает наименьшее целочисленное значение, большее или равное указанному числовому выражению.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 8c5fdda416aca698b9ad0a68ef050957f32aef31
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93332412"
---
# <a name="ceiling-azure-cosmos-db"></a>CEILING (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает наименьшее целочисленное значение, которое больше или равно указанному числовому выражению.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
CEILING (<numeric_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*numeric_expr*  
   Числовое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает числовое выражение.  
  
## <a name="examples"></a>Примеры
  
  В следующем примере показаны положительные числовые, отрицательные и нулевые значения с `CEILING` функцией.  
  
```sql
SELECT CEILING(123.45) AS c1, CEILING(-123.45) AS c2, CEILING(0.0) AS c3  
```  
  
 Результирующий набор:  
  
```json
[{c1: 124, c2: -123, c3: 0}]  
```  

## <a name="remarks"></a>Remarks

Эта системная функция воспользуется преимуществами [индекса диапазона](index-policy.md#includeexclude-strategy).

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
