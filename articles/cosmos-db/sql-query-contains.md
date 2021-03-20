---
title: Функция Contains в языке запросов Azure Cosmos DB
description: Сведения о том, как системная функция SQL CONTAINS в Azure Cosmos DB возвращает значение логического типа, указывающее, содержит ли первое строковое выражение второе
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 06/02/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 4b6835b22e5cfa4ca703b95d70e20112b8723def
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339178"
---
# <a name="contains-azure-cosmos-db"></a>CONTAINS (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Возвращает значение логического типа, указывающее, содержит ли первое строковое выражение второе.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
CONTAINS(<str_expr1>, <str_expr2> [, <bool_expr>])  
```  
  
## <a name="arguments"></a>Аргументы
  
*str_expr1*  
   Строковое выражение, в котором выполняется поиск.  
  
*str_expr2*  
   Искомое строковое выражение.  

*bool_expr* Необязательное значение для игнорирования регистра. Если задано значение true, CONTAINS будет выполнять поиск без учета регистра. Если значение не указано, оно равно false.
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает логическое выражение.  
  
## <a name="examples"></a>Примеры
  
  Пример ниже проверяет, содержит ли строка abc вхождения ab и вхождения A.  
  
```sql
SELECT CONTAINS("abc", "ab", false) AS c1, CONTAINS("abc", "A", false) AS c2, CONTAINS("abc", "A", true) AS c3
```  
  
 Результирующий набор:  
  
```json
[
    {
        "c1": true,
        "c2": false,
        "c3": true
    }
]
```  

## <a name="remarks"></a>Remarks

Эта системная функция будет использовать преимущества [индекса диапазона](index-policy.md#includeexclude-strategy).

При увеличении количества элементов свойства в системной функции Contains будет увеличиваться потребление единиц запросов. Иными словами, если проверяется, содержит ли значение свойства определенную строку, стоимость запроса в единицах запросов будет зависеть от количества возможных значений этого свойства.

Например, рассмотрим два свойства: город и страна. Количество элементов города — 5000, а количество элементов страны — 200. Ниже приведены два примера запросов.

```sql
    SELECT * FROM c WHERE CONTAINS(c.town, "Red", false)
```

```sql
    SELECT * FROM c WHERE CONTAINS(c.country, "States", false)
```

Первый запрос, скорее всего, будет использовать больше единиц запросов, чем второй, поскольку количество элементов города выше, чем страны.

Если размер свойства в параметре Contains превышает 1 КБ для некоторых документов, то обработчику запросов потребуется загрузить эти документы. В этом случае обработчик запросов не сможет полностью оценивать Contains с помощью индекса. При наличии большого количества документов с размерами свойств, превышающими 1 КБ, плата за ЕДИНИЦу для Contains будет высока.

## <a name="next-steps"></a>Дальнейшие действия

- [Строковые функции Azure Cosmos DB](sql-query-string-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
