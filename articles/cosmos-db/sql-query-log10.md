---
title: LOG10 на языке запросов Azure Cosmos DB
description: Сведения о системной функции LOG10 SQL в Azure Cosmos DB для возврата десятичного логарифма указанного числового выражения
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 27f7d916c1a3f84f26674fea6b04597fd9e546bc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93338443"
---
# <a name="log10-azure-cosmos-db"></a>LOG10 (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает десятичный логарифм от указанного числового выражения.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
LOG10 (<numeric_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*numeric_expression*  
   Числовое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает числовое выражение.  
  
## <a name="remarks"></a>Комментарии
  
  Функции LOG10 и POWER находятся в обратной зависимости друг от друга. Например, 10 ^ LOG10(n) = n. Эта системная функция не будет использовать индекс.
  
## <a name="examples"></a>Примеры
  
  В примере ниже объявляется переменная и возвращается значение LOG10 указанной переменной (100).  
  
```sql
SELECT LOG10(100) AS log10 
```  
  
 Результирующий набор:  
  
```json
[{log10: 2}]  
```  

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
