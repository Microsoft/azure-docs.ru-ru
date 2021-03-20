---
title: Стрингтоаррай на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Стрингтоаррай в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 7ae1f69e92e890daae528eb1f4dfb95f76560043
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93337988"
---
# <a name="stringtoarray-azure-cosmos-db"></a>Стрингтоаррай (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает выражение, преобразованное в массив. Если выражение не может быть преобразовано, возвращает значение undefine.  
  
## <a name="syntax"></a>Синтаксис
  
```sql  
StringToArray(<str_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*str_expr*  
   Строковое выражение, анализируемое как выражение массива JSON. 
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает выражение массива или значение undefine. 
  
## <a name="remarks"></a>Remarks
  Вложенные строковые значения должны быть записаны с двойными кавычками, чтобы быть допустимыми JSON. Дополнительные сведения о формате JSON см. в разделе [JSON.org](https://json.org/). Эта системная функция не будет использовать индекс.
  
## <a name="examples"></a>Примеры
  
  В следующем примере показано `StringToArray` , как ведет себя в разных типах. 
  
 Ниже приведены примеры с допустимыми входными данными.

```sql
SELECT 
    StringToArray('[]') AS a1, 
    StringToArray("[1,2,3]") AS a2,
    StringToArray("[\"str\",2,3]") AS a3,
    StringToArray('[["5","6","7"],["8"],["9"]]') AS a4,
    StringToArray('[1,2,3, "[4,5,6]",[7,8]]') AS a5
```

Результирующий набор:

```json
[{"a1": [], "a2": [1,2,3], "a3": ["str",2,3], "a4": [["5","6","7"],["8"],["9"]], "a5": [1,2,3,"[4,5,6]",[7,8]]}]
```

Ниже приведен пример недопустимых входных данных. 
   
 Одинарные кавычки в массиве не являются допустимыми JSON.
Несмотря на то, что они являются допустимыми в запросе, они не будут анализироваться в допустимые массивы. Строки в строке массива должны либо быть экранированы "[ \\ " \\ "]", либо окружающая кавычка должна быть единственной "[" "]".

```sql
SELECT
    StringToArray("['5','6','7']")
```

Результирующий набор:

```json
[{}]
```

Ниже приведены примеры недопустимых входных данных.
   
 Переданное выражение будет проанализировано как массив JSON. Следующий тип не вычисляется как массив типа и, таким же, возвращает значение undefine.
   
```sql
SELECT
    StringToArray("["),
    StringToArray("1"),
    StringToArray(NaN),
    StringToArray(false),
    StringToArray(undefined)
```

Результирующий набор:

```json
[{}]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Строковые функции Azure Cosmos DB](sql-query-string-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
