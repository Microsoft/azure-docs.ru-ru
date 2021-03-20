---
title: Стрингтубжект на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Стрингтубжект в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 369c86c8fc2ef4d125c3f13db4eb3af88b7e726e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93337852"
---
# <a name="stringtoobject-azure-cosmos-db"></a>Стрингтубжект (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает выражение, преобразованное в объект. Если выражение не может быть преобразовано, возвращает значение undefine.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
StringToObject(<str_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*str_expr*  
   Строковое выражение, анализируемое как выражение объекта JSON. Обратите внимание, что вложенные строковые значения должны быть написаны с помощью двойных кавычек. Дополнительные сведения о формате JSON см. в разделе [JSON.org](https://json.org/)  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает выражение объекта или значение undefine.  
  
## <a name="examples"></a>Примеры
  
  В следующем примере показано `StringToObject` , как ведет себя в разных типах. 
  
 Ниже приведены примеры с допустимыми входными данными.

```sql
SELECT 
    StringToObject("{}") AS obj1, 
    StringToObject('{"A":[1,2,3]}') AS obj2,
    StringToObject('{"B":[{"b1":[5,6,7]},{"b2":8},{"b3":9}]}') AS obj3, 
    StringToObject("{\"C\":[{\"c1\":[5,6,7]},{\"c2\":8},{\"c3\":9}]}") AS obj4
``` 

Результирующий набор:

```json
[{"obj1": {}, 
  "obj2": {"A": [1,2,3]}, 
  "obj3": {"B":[{"b1":[5,6,7]},{"b2":8},{"b3":9}]},
  "obj4": {"C":[{"c1":[5,6,7]},{"c2":8},{"c3":9}]}}]
```

 Ниже приведены примеры с недопустимыми входными данными.
Несмотря на то, что они являются допустимыми в запросе, они не будут анализироваться в допустимые объекты. Строки в строке объекта должны либо быть экранированы "{ \\ " a \\ ": \\ " str \\ "}", либо окружающая кавычка должна быть единственной "{" a ":" str "}".

Одинарные кавычки, относящиеся к именам свойств, не являются допустимыми JSON.

```sql
SELECT 
    StringToObject("{'a':[1,2,3]}")
```

Результирующий набор:

```json
[{}]
```  

Имена свойств без окружающих кавычек не являются допустимыми JSON.

```sql
SELECT 
    StringToObject("{a:[1,2,3]}")
```

Результирующий набор:

```json
[{}]
``` 

Ниже приведены примеры с недопустимыми входными данными.

 Переданное выражение будет анализироваться как объект JSON; Эти входные данные не вычисляют тип Object и, таким же, возвращают undefine.

```sql
SELECT 
    StringToObject("}"),
    StringToObject("{"),
    StringToObject("1"),
    StringToObject(NaN), 
    StringToObject(false), 
    StringToObject(undefined)
``` 
 
 Результирующий набор:

```json
[{}]
```

## <a name="remarks"></a>Remarks

Эта системная функция не будет использовать индекс.

## <a name="next-steps"></a>Дальнейшие действия

- [Строковые функции Azure Cosmos DB](sql-query-string-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
