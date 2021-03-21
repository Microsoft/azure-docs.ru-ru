---
title: MIN в языке запросов Azure Cosmos DB
description: Сведения о функции min (MIN) SQL System в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 679814822cca5a72bd261d3c8944863715e31f70
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96555409"
---
# <a name="min-azure-cosmos-db"></a>MIN (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Эта агрегатная функция возвращает минимум значений в выражении.
  
## <a name="syntax"></a>Синтаксис
  
```sql
MIN(<scalar_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*scalar_expr*  
   Скалярное выражение. 
  
## <a name="return-types"></a>Типы возвращаемых данных
  
Возвращает скалярное выражение.  
  
## <a name="examples"></a>Примеры
  
В следующем примере возвращается минимальное значение `propertyA` :
  
```sql
SELECT MIN(c.propertyA)
FROM c
```  

## <a name="remarks"></a>Remarks

Эта системная функция будет использовать преимущества [индекса диапазона](index-policy.md#includeexclude-strategy). Аргументами в `MIN` может быть число, строка, логическое значение или значение null. Все неопределенные значения будут игнорироваться.

При сравнении данных разных типов используется следующий порядок приоритета (в порядке возрастания):

- null
- Логическое
- Число
- строка

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции в Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции в Azure Cosmos DB](sql-query-system-functions.md)
- [Агрегатные функции в Azure Cosmos DB](sql-query-aggregate-functions.md)