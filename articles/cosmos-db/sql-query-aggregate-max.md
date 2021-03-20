---
title: MAX в языке запросов Azure Cosmos DB
description: Сведения о функции Max (MAX) SQL System в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 12/02/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: a3d8e3f2687a492461bdbbdd761e26cdb58f9e96
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96553409"
---
# <a name="max-azure-cosmos-db"></a>MAX (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Эта агрегатная функция возвращает максимум значений в выражении.
  
## <a name="syntax"></a>Синтаксис
  
```sql
MAX(<scalar_expr>)  
```  
  
## <a name="arguments"></a>Аргументы

*scalar_expr*  
   Скалярное выражение. 
  
## <a name="return-types"></a>Типы возвращаемых данных
  
Возвращает скалярное выражение.  
  
## <a name="examples"></a>Примеры
  
В следующем примере возвращается максимальное значение `propertyA` :
  
```sql
SELECT MAX(c.propertyA)
FROM c
```  

## <a name="remarks"></a>Remarks

Эта системная функция будет использовать преимущества [индекса диапазона](index-policy.md#includeexclude-strategy). Аргументами в `MAX` может быть число, строка, логическое значение или значение null. Все неопределенные значения будут игнорироваться.

При сравнении данных разных типов используется следующий порядок приоритета (в порядке убывания):

- строка
- число
- Логическое
- null

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции в Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции в Azure Cosmos DB](sql-query-system-functions.md)
- [Агрегатные функции в Azure Cosmos DB](sql-query-aggregate-functions.md)