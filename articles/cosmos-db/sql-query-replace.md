---
title: ЗАМЕНИТЬ на языке запросов Azure Cosmos DB
description: Сведения о функции "замена системных функций SQL" в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 8c9b7ffdf8395944cf75dabbbf4c42cea0e0c9b3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93341626"
---
# <a name="replace-azure-cosmos-db"></a>Replace (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Заменяет все вхождения указанного строкового значения другим строковым значением.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
REPLACE(<str_expr1>, <str_expr2>, <str_expr3>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*str_expr1*  
   Строковое выражение, в котором выполняется поиск.  
  
*str_expr2*  
   Строковое выражение, которое должно быть найдено.  
  
*str_expr3*  
   Строковое выражение для замены вхождений *str_expr2* в *str_expr1*.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает строковое выражение.  
  
## <a name="examples"></a>Примеры
  
  В следующем примере показано, как использовать `REPLACE` в запросе.  
  
```sql
SELECT REPLACE("This is a Test", "Test", "desk") AS replace
```  
  
 Результирующий набор:  
  
```json
[{"replace": "This is a desk"}]  
```  

## <a name="remarks"></a>Remarks

Эта системная функция не будет использовать индекс.

## <a name="next-steps"></a>Дальнейшие действия

- [Строковые функции Azure Cosmos DB](sql-query-string-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
