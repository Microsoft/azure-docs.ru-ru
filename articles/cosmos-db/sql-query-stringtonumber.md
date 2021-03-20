---
title: Стрингтонумбер на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Стрингтонумбер в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 03/03/2020
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: f2622958a2315458ccd01da4aea7c6b75941e755
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339569"
---
# <a name="stringtonumber-azure-cosmos-db"></a>Стрингтонумбер (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает выражение, преобразованное в число. Если выражение не может быть преобразовано, возвращает значение undefine.  
  
## <a name="syntax"></a>Синтаксис
  
```sql
StringToNumber(<str_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*str_expr*  
   Строковое выражение, анализируемое как выражение числа JSON. Числа в JSON должны быть целым числом или плавающей запятой. Дополнительные сведения о формате JSON см. в разделе [JSON.org](https://json.org/)  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает числовое или неопределенное выражение.  
  
## <a name="examples"></a>Примеры
  
  В следующем примере показано `StringToNumber` , как ведет себя в разных типах. 

Пробелы допускаются только до или после числа.

```sql
SELECT 
    StringToNumber("1.000000") AS num1, 
    StringToNumber("3.14") AS num2,
    StringToNumber("   60   ") AS num3, 
    StringToNumber("-1.79769e+308") AS num4
```  
  
 Результирующий набор:  
  
```json
{{"num1": 1, "num2": 3.14, "num3": 60, "num4": -1.79769e+308}}
```  

В JSON допустимый номер должен быть целым числом или числом с плавающей запятой.

```sql
SELECT   
    StringToNumber("0xF")
```  
  
 Результирующий набор:  
  
```json
{{}}
```  

Переданное выражение будет проанализировано как числовое выражение; Эти входные данные не имеют тип Number и, таким же, возвращают неопределенное значение. 

```sql
SELECT 
    StringToNumber("99     54"),   
    StringToNumber(undefined),
    StringToNumber("false"),
    StringToNumber(false),
    StringToNumber(" "),
    StringToNumber(NaN)
```  
  
 Результирующий набор:  
  
```json
{{}}
```  

## <a name="remarks"></a>Remarks

Эта системная функция не будет использовать индекс.

## <a name="next-steps"></a>Дальнейшие действия

- [Строковые функции Azure Cosmos DB](sql-query-string-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
