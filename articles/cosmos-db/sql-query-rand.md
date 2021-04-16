---
title: Использование RAND на языке запросов Azure Cosmos DB
description: Подробнее о системной функции SQL RAND в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 09/16/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: fb3e310970fcc2146ee0d4b790a9744dcd566bad
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93341660"
---
# <a name="rand-azure-cosmos-db"></a>RAND (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает числовое значение в диапазоне [0, 1), сгенерированное случайным образом.
 
## <a name="syntax"></a>Синтаксис
  
```sql
RAND ()  
```  

## <a name="return-types"></a>Типы возвращаемых данных

  Возвращает числовое выражение.

## <a name="remarks"></a>Remarks

  `RAND` является недетерминированной функцией. Результатом повторных вызовов `RAND` будут разные числа. Эта системная функция не использует индексы.


## <a name="examples"></a>Примеры
  
  В следующем примере возвращается числовое значение, сгенерированное случайным образом.
  
```sql
SELECT RAND() AS rand 
```  
  
 Результирующий набор:  
  
```json
[{"rand": 0.87860053195618093}]  
``` 

## <a name="next-steps"></a>Дальнейшие действия

- [Математические функции Azure Cosmos DB](sql-query-mathematical-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
