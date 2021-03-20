---
title: Скалярные выражения в запросах SQL Azure Cosmos DB
description: Сведения о скалярном выражении синтаксиса SQL для Azure Cosmos DB. В этой статье также описывается объединение скалярных выражений в сложные выражения с помощью операторов.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/17/2019
ms.author: mjbrown
ms.openlocfilehash: 34df2d2f4a2209cca93c3f7ac12dcd203bf4a089
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339637"
---
# <a name="scalar-expressions-in-azure-cosmos-db-sql-queries"></a>Скалярные выражения в запросах SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

[Предложение SELECT](sql-query-select.md) поддерживает скалярные выражения. Скалярное выражение — это сочетание символов и операторов, в результате вычисления которых возвращается одно значение. Примерами скалярных выражений могут быть константы, ссылки на свойства, ссылки на элементы массива, ссылки на псевдонимы или вызовы функций. Скалярные выражения можно объединять в сложные выражения с помощью операторов.

## <a name="syntax"></a>Синтаксис
  
```sql  
<scalar_expression> ::=  
       <constant>
     | input_alias
     | parameter_name  
     | <scalar_expression>.property_name  
     | <scalar_expression>'['"property_name"|array_index']'  
     | unary_operator <scalar_expression>  
     | <scalar_expression> binary_operator <scalar_expression>    
     | <scalar_expression> ? <scalar_expression> : <scalar_expression>  
     | <scalar_function_expression>  
     | <create_object_expression>
     | <create_array_expression>  
     | (<scalar_expression>)
  
<scalar_function_expression> ::=  
        'udf.' Udf_scalar_function([<scalar_expression>][,…n])  
        | builtin_scalar_function([<scalar_expression>][,…n])  
  
<create_object_expression> ::=  
   '{' [{property_name | "property_name"} : <scalar_expression>][,…n] '}'  
  
<create_array_expression> ::=  
   '[' [<scalar_expression>][,…n] ']'  
  
```

## <a name="arguments"></a>Аргументы
  
- `<constant>`  
  
   Представляет значение константы. Дополнительные сведения см. в подразделе [Константы](sql-query-constants.md).  
  
- `input_alias`  
  
   Представляет значение, определяемое аргументом `input_alias` в предложении `FROM`.  
  Выражение гарантировано не может принять значение **undefined**. Значения **undefined** пропускаются.  
  
- `<scalar_expression>.property_name`  
  
   Представляет значение свойства объекта. Если свойство не существует или на свойство ссылается значение, которое не является объектом, результатом вычисления выражения является **неопределенное** значение.  
  
- `<scalar_expression>'['"property_name"|array_index']'`  
  
   Представляет значение свойства с именем `property_name` или элементом массива с индексом `array_index` массива. Если объект или массив не существует или ссылается на значение, которое не является объектом или массивом, выражение принимает значение undefined.  
  
- `unary_operator <scalar_expression>`  
  
   Представляет оператор, применяемый к одному значению. Дополнительные сведения см. в подразделе [Операторы](sql-query-operators.md).  
  
- `<scalar_expression> binary_operator <scalar_expression>`  
  
   Представляет оператор, применяемый к двум значениям. Дополнительные сведения см. в подразделе [Операторы](sql-query-operators.md).  
  
- `<scalar_function_expression>`  
  
   Представляет значение, определяемое результатом вызова функции.  
  
- `udf_scalar_function`  
  
   Имя определяемой пользователем скалярной функции.  
  
- `builtin_scalar_function`  
  
   Имя встроенной скалярной функции.  
  
- `<create_object_expression>`  
  
   Представляет значение, полученное в процессе создания объекта с заданными свойствами и их значениями.  
  
- `<create_array_expression>`  
  
   Представляет значение, полученное в процессе создания массива с заданными значениями в качестве элементов.  
  
- `parameter_name`  
  
   Представляет значение указанного имени параметра. Имена параметров должны начинаться с одного символа \@.  
  
## <a name="remarks"></a>Remarks
  
  При вызове встроенной или определяемой пользователем скалярной функции необходимо определить все аргументы. Если любой из аргументов не определен, функция не будет вызвана, и значение не будет определено.  
  
  При создании объекта все свойства со значением undefined пропускаются и не включаются в созданный объект.  
  
  При создании массива все элементы со значением **undefined** пропускаются и не включаются в созданный объект. В этом случае место неопределенного элемента занимает следующий определенный элемент, таким образом в созданном массиве не будет пропущенных индексов.  

## <a name="examples"></a>Примеры

```sql
    SELECT ((2 + 11 % 7)-2)/3
```

Результаты:

```json
    [{
      "$1": 1.33333
    }]
```

В следующем запросе результат скалярного выражения является логическим:

```sql
    SELECT f.address.city = f.address.state AS AreFromSameCityState
    FROM Families f
```

Результаты:

```json
    [
      {
        "AreFromSameCityState": false
      },
      {
        "AreFromSameCityState": true
      }
    ]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения об Azure Cosmos DB](introduction.md)
- [Примеры .NET для Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Подзапросы](sql-query-subquery.md)