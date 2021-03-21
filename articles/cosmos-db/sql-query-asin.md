---
title: ASIN на языке запросов Azure Cosmos DB
description: Узнайте, как функция арксинус (ASIN) SQL System в Azure Cosmos DB возвращает угол в радианах, синус которого является указанным числовым выражением.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/04/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 69862df5809bffe7c975b0340993b8e42aa47337
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93332480"
---
# <a name="asin-azure-cosmos-db"></a>ASIN (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает угол в радианах, синус которого равен указанному числовому выражению. Также называется арксинусом.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
ASIN(<numeric_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*numeric_expr*  
   Числовое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает числовое выражение.  
  
## <a name="examples"></a>Примеры
  
  В следующем примере возвращается значение, `ASIN` равное-1.  
  
```sql
SELECT ASIN(-1) AS asin  
```  
  
 Результирующий набор:  
  
```json
[{"asin": -1.5707963267948966}]  
```  

## <a name="remarks"></a>Remarks

Эта системная функция не будет использовать индекс.

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
