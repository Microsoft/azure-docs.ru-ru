---
title: ОБРАТная Azure Cosmos DB языка запросов
description: Дополнительные сведения о функции SQL System в ОБРАТную Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 7d70d65609211ea18f566dbae42aca5231ed2eb7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93341589"
---
# <a name="reverse-azure-cosmos-db"></a>REVERSE (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает строковое значение, где символы переставлены в обратном порядке справа налево.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
REVERSE(<str_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*str_expr*  
   Строковое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает строковое выражение.  
  
## <a name="examples"></a>Примеры
  
  В следующем примере показано, как использовать `REVERSE` в запросе.  
  
```sql
SELECT REVERSE("Abc") AS reverse  
```  
  
 Результирующий набор:  
  
```json
[{"reverse": "cbA"}]  
```  

## <a name="remarks"></a>Remarks

Эта системная функция не будет использовать индекс.

## <a name="next-steps"></a>Дальнейшие действия

- [Строковые функции Azure Cosmos DB](sql-query-string-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
