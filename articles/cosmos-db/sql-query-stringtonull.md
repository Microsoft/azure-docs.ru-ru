---
title: Стрингтонулл на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Стрингтонулл в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: ee91ca3a32bacbf6590877f49bf74499fc00a8f9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339593"
---
# <a name="stringtonull-azure-cosmos-db"></a>Стрингтонулл (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает выражение, преобразованное в значение null. Если выражение не может быть преобразовано, возвращает значение undefine.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
StringToNull(<str_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*str_expr*  
   Строковое выражение, анализируемое как выражение null.
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает пустое или неопределенное выражение.  
  
## <a name="examples"></a>Примеры
  
  В следующем примере показано `StringToNull` , как ведет себя в разных типах. 

Ниже приведены примеры с допустимыми входными данными.

 Пробелы допускаются только до или после "null".

```sql
SELECT 
    StringToNull("null") AS n1, 
    StringToNull("  null ") AS n2,
    IS_NULL(StringToNull("null   ")) AS n3
```  
  
 Результирующий набор:  
  
```json
[{"n1": null, "n2": null, "n3": true}]
```  

Ниже приведены примеры с недопустимыми входными данными.

NULL учитывается с учетом регистра и должен быть записан со всеми символами нижнего регистра, например "null".

```sql
SELECT    
    StringToNull("NULL"),
    StringToNull("Null")
```  
  
 Результирующий набор:  
  
```json
[{}]
```  

Переданное выражение будет проанализировано как выражение null; Эти входные данные не вычисляют значение NULL и, таким же, возвращают неопределенный тип.

```sql
SELECT    
    StringToNull("true"), 
    StringToNull(false), 
    StringToNull(undefined),
    StringToNull(NaN) 
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
