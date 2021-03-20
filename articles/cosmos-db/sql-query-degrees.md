---
title: ГРАДУСы на языке запросов Azure Cosmos DB
description: Сведения о функции SQL System в ГРАДУСАХ в Azure Cosmos DB, чтобы получить соответствующий угол в градусах для угла, указанного в радианах
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 5c7aa5496533a42ad726f9ed8efd8ed7429375fc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93338906"
---
# <a name="degrees-azure-cosmos-db"></a>ГРАДУСы (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает соответствующее значение угла в градусах для угла, указанного в радианах.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
DEGREES (<numeric_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*numeric_expr*  
   Числовое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает числовое выражение.  
  
## <a name="examples"></a>Примеры
  
  Следующий пример возвращает число градусов в угле, равном PI/2 радиан.  
  
```sql
SELECT DEGREES(PI()/2) AS degrees  
```  
  
 Результирующий набор:  
  
```json
[{"degrees": 90}]  
```  

## <a name="remarks"></a>Remarks

Эта системная функция не будет использовать индекс.

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
