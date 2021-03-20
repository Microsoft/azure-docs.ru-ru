---
title: Константы SQL в Azure Cosmos DB
description: Сведения о том, как константы SQL-запросов в Azure Cosmos DB используются для представления определенного значения данных
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/31/2019
ms.author: tisande
ms.openlocfilehash: b0973dca980b65b85ac66cc1b3c9ed45f0f9c779
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339212"
---
# <a name="azure-cosmos-db-sql-query-constants"></a>Константы запроса SQL Azure Cosmos DB  
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Константа, также называемая литералом или скалярным значением, представляет собой символ, означающий указанное значение типа данных. Формат константы зависит от типа данных значения, который она означает.  
  
 **Поддерживаемые скалярные типы данных**  
  
|**Тип**|**Порядок значений**|  
|-|-|  
|**Не определено.**|Доступное значение: **undefined**|  
|**Null**|Доступное значение: **Null**|  
|**Boolean**|Доступные значения: **false**, **true**|  
|**Число**|Число с плавающей запятой двойной точности, стандарт IEEE 754.|  
|**String**|Последовательности из нуля или более знаков Юникода. Строки необходимо заключить в одинарные или двойные кавычки.|  
|**массив**.|Последовательность из нуля или более элементов. Каждый элемент может быть значением любого скалярного типа данных, за исключением **неопределенного**.|  
|**Объект**|Неупорядоченный набор из нуля или более пар "имя — значение". Имя является строкой Юникода. Значение может иметь любой скалярный тип данных (за исключением типа **Неопределенное**).|  
  
## <a name="syntax"></a><a name="bk_syntax"></a>Синтаксис
  
```sql  
<constant> ::=  
   <undefined_constant>  
     | <null_constant>   
     | <boolean_constant>   
     | <number_constant>   
     | <string_constant>   
     | <array_constant>   
     | <object_constant>   
  
<undefined_constant> ::= undefined  
  
<null_constant> ::= null  
  
<boolean_constant> ::= false | true  
  
<number_constant> ::= decimal_literal | hexadecimal_literal  
  
<string_constant> ::= string_literal  
  
<array_constant> ::=  
    '[' [<constant>][,...n] ']'  
  
<object_constant> ::=   
   '{' [{property_name | "property_name"} : <constant>][,...n] '}'  
  
```  
  
##  <a name="arguments"></a><a name="bk_arguments"></a> Аргументы
  
* `<undefined_constant>; Undefined`  
  
  Представляет неопределенное значение типа "Неопределенное".  
  
* `<null_constant>; null`  
  
  Представляет значение **Null** типа **Null**.  
  
* `<boolean_constant>`  
  
  Представляет константу типа "Логический".  
  
* `false`  
  
  Представляет значение **false** типа "Логический".  
  
* `true`  
  
  Представляет значение **true** типа "Логический".  
  
* `<number_constant>`  
  
  Представляет константу.  
  
* `decimal_literal`  
  
  Десятичные литералы — это числа, представленные в десятичном или экспоненциальном представлении.  
  
* `hexadecimal_literal`  
  
  Шестнадцатеричные литералы — это числа, представленные с помощью префикса 0x, за которым следуют одна или несколько шестнадцатеричных цифр.  
  
* `<string_constant>`  
  
  Представляет константу типа "Строка".  
  
* `string _literal`  
  
  Строковые литералы — это строки Юникода, представленные в виде последовательности из нуля или более знаков Юникода или escape-последовательностей. Строковые литералы заключаются в одинарные кавычки (апостроф — ') или двойные кавычки (кавычки — ").  
  
  Допускаются следующие escape-последовательности:  
  
|**Escape-последовательность**|**Описание**|**Символ Юникода**|  
|-|-|-|  
|\\'|Апостроф (')|U+0027|  
|\\"|Кавычки (")|U+0022|  
|\\\ |Обратная косая черта (\\)|U+005C|  
|\\/|Косая черта (/)|U+002F|  
|\b|BACKSPACE|U+0008|  
|\f|Смена страницы|U+000C|  
|\n|перевод строки|U+000A|  
|\r|Возврат каретки|U+000D|  
|\t|TAB|U+0009|  
|\uXXXX|Символ Юникода, определяемый 4 шестнадцатеричными цифрами.|U+XXXX|  

## <a name="next-steps"></a>Дальнейшие действия

- [Примеры .NET для Azure Cosmos DB](https://github.com/Azure/azure-cosmos-dotnet-v3)
- [Данные документов модели](modeling-data.md)
