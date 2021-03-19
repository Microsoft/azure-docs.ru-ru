---
title: EXP на языке запросов Azure Cosmos DB
description: Сведения о функции SQL System экспоненты (EXP) в Azure Cosmos DB, возвращающей экспоненциальное значение указанного числового выражения
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: adf80d36edbe8f9a5535e8fa04501918a62ea7ca
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93335659"
---
# <a name="exp-azure-cosmos-db"></a>EXP (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает значение экспоненты для указанного числового выражения.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
EXP (<numeric_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*numeric_expr*  
   Числовое выражение.  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает числовое выражение.  
  
## <a name="remarks"></a>Remarks
  
  Константа **e** (2,718281...) является основанием натуральных логарифмов.  
  
  Экспонента числа является константой **e**, возведенной в степень числа. Например, EXP(1.0) = e^1.0 = 2,71828182845905 и EXP(10) = e^10 = 22026,4657948067.  
  
  Экспонента натурального логарифма числа — это само число. Например, EXP (LOG (n)) = n. Натуральный логарифм экспоненты числа также является числом. Например, LOG (EXP (n)) = n.  
  
## <a name="examples"></a>Примеры
  
  В примере ниже объявляется переменная и возвращается значение экспоненты указанной переменной (10).  
  
```sql
SELECT EXP(10) AS exp  
```  
  
 Результирующий набор:  
  
```json
[{exp: 22026.465794806718}]  
```  
  
 Пример ниже возвращает значение экспоненты от натурального логарифма 20 и натуральный логарифм экспоненты 20. Так как эти функции обратные, в обоих случаях возвращается значение 20 (после округления до плавающей запятой).  
  
```sql
SELECT EXP(LOG(20)) AS exp1, LOG(EXP(20)) AS exp2  
```  
  
 Результирующий набор:  
  
```json
[{exp1: 19.999999999999996, exp2: 20}]  
```  

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
