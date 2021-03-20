---
title: Azure Cosmos DB язык запросов в среднем
description: Сведения о средней (AVG) системной функции SQL в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 2cfbca798a7b404e4ee12b93396b2a5b08d7d5bb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96555421"
---
# <a name="avg-azure-cosmos-db"></a>AVG (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Эта агрегатная функция возвращает среднее значение в выражении.
  
## <a name="syntax"></a>Синтаксис
  
```sql
AVG(<numeric_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*numeric_expr*  
   Числовое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
Возвращает числовое выражение.  
  
## <a name="examples"></a>Примеры
  
В следующем примере возвращается среднее значение `propertyA` :
  
```sql
SELECT AVG(c.propertyA)
FROM c
```  

## <a name="remarks"></a>Remarks

Эта системная функция будет использовать преимущества [индекса диапазона](index-policy.md#includeexclude-strategy). Если какие бы то ни было аргументы в `AVG` строках типа String, Boolean или null, будет возвращена вся агрегатная системная функция `undefined` . Если какой бы то ни было аргумент имеет `undefined` значение, это не повлияет на `AVG` вычисление.

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции в Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции в Azure Cosmos DB](sql-query-system-functions.md)
- [Агрегатные функции в Azure Cosmos DB](sql-query-aggregate-functions.md)