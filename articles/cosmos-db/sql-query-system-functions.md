---
title: Системные функции на языке запросов Azure Cosmos DB
description: Сведения о встроенных и определяемых пользователем системных функциях SQL в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 02/03/2021
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: 36ebe80671c77cc83ddba770e3259f6542472e58
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99526845"
---
# <a name="system-functions-azure-cosmos-db"></a>Системные функции (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Cosmos DB предоставляет множество встроенных функций SQL. Ниже перечислены их категории.  
  
|Группа функций|Описание|Операции|  
|--------------|-----------------|-----------------| 
|[Функции массива](sql-query-array-functions.md)|Функции массивов выполняют операцию над входным массивом и возвращают числовое, логическое значение либо массив. | [ARRAY_CONCAT](sql-query-array-concat.md), [ARRAY_CONTAINS](sql-query-array-contains.md), [ARRAY_LENGTH](sql-query-array-length.md), [ARRAY_SLICE](sql-query-array-slice.md) |
|[Функции даты и времени](sql-query-date-time-functions.md)|Функции даты и времени позволяют получить текущую дату и время в формате UTC в двух формах. числовая метка времени, значение которой является эпохой UNIX в миллисекундах или в виде строки, которая соответствует формату ISO 8601. | [Жеткуррентдатетиме](sql-query-getcurrentdatetime.md), [жеткурренттиместамп](sql-query-getcurrenttimestamp.md), [жеткурренттиккс](sql-query-getcurrentticks.md) |
|[Математические функции](sql-query-mathematical-functions.md)|Математические функции выполняют вычисление, которое обычно основано на входных значениях, предоставляемых в форме аргументов, и возвращают числовое значение. | [ABS](sql-query-abs.md), [Acos](sql-query-acos.md), [ASIN](sql-query-asin.md), [ATAN](sql-query-atan.md), [ATN2](sql-query-atn2.md), [Ceiling](sql-query-ceiling.md), [COS](sql-query-cos.md), [COT](sql-query-cot.md), [градусы](sql-query-degrees.md), [exp](sql-query-exp.md), [этаж](sql-query-floor.md), [Журнал](sql-query-log.md), [LOG10](sql-query-log10.md), [Pi](sql-query-pi.md), [Power](sql-query-power.md), [радианы](sql-query-radians.md), [Rand](sql-query-rand.md), [круглый](sql-query-round.md), [знак](sql-query-sign.md), [sin](sql-query-sin.md), [sqrt](sql-query-sqrt.md), [квадратный](sql-query-square.md), [Tan](sql-query-tan.md), [TRUNC](sql-query-trunc.md) |
|[Пространственные функции](sql-query-spatial-functions.md)|Пространственные функции выполняют операцию над входным пространственным объектом и возвращают числовое или логическое значение. | [ST_DISTANCE](sql-query-st-distance.md), [ST_INTERSECTS](sql-query-st-intersects.md), [ST_ISVALID](sql-query-st-isvalid.md), [ST_ISVALIDDETAILED](sql-query-st-isvaliddetailed.md), [ST_WITHIN](sql-query-st-within.md) |
|[Строковые функции](sql-query-string-functions.md)|Cтроковые функции выполняют операцию над входным строковым значением и возвращают строковое, числовое или логическое значение. | [Concat](sql-query-concat.md), [Contains](sql-query-contains.md), [ENDSWITH](sql-query-endswith.md), [INDEX_OF](sql-query-index-of.md), [ЛЕВСИМВ](sql-query-left.md), [length](sql-query-length.md), [Lower](sql-query-lower.md), [LTRIM](sql-query-ltrim.md), [режексматч](sql-query-regexmatch.md)[Replace](sql-query-replace.md), [replicate](sql-query-replicate.md), [Reverse](sql-query-reverse.md), [right](sql-query-right.md), [RTRIM](sql-query-rtrim.md), [STARTSWITH](sql-query-startswith.md), [стрингтоаррай](sql-query-stringtoarray.md), [стрингтобулеан](sql-query-stringtoboolean.md), [стрингтонулл](sql-query-stringtonull.md), [стрингтонумбер](sql-query-stringtonumber.md), [стрингтубжект](sql-query-stringtoobject.md), [substring](sql-query-substring.md), [ToString](sql-query-tostring.md), [Trim](sql-query-trim.md), [Upper](sql-query-upper.md) |
|[Функции проверки типа](sql-query-type-checking-functions.md)|Функции проверки типа позволяют проверять тип выражения в запросах SQL. | [IS_ARRAY](sql-query-is-array.md), [IS_BOOL](sql-query-is-bool.md), [IS_DEFINED](sql-query-is-defined.md), [IS_NULL](sql-query-is-null.md), [IS_NUMBER](sql-query-is-number.md), [IS_OBJECT](sql-query-is-object.md), [IS_PRIMITIVE](sql-query-is-primitive.md), [IS_STRING](sql-query-is-string.md) |

## <a name="built-in-versus-user-defined-functions-udfs"></a>Встроенные и определяемые пользователем функции (UDF)

Если в настоящее время используется определяемая пользователем функция (UDF), для которой теперь доступна встроенная функция, то соответствующая встроенная функция будет быстрее запускаться и эффективнее.

## <a name="built-in-versus-ansi-sql-functions"></a>Встроенные функции SQL и ANSI

Основное различие между функциями Cosmos DB и ANSI SQL-функциями заключается в том, что функции Cosmos DB предназначены для работы с данными без схем и смешанными схемами. Например, если свойство отсутствует или имеет нечисловое значение, такое как `undefined` , элемент пропускается вместо возврата ошибки.

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения об Azure Cosmos DB](introduction.md)
- [Функции массива](sql-query-array-functions.md)
- [Функции даты и времени](sql-query-date-time-functions.md)
- [Математические функции](sql-query-mathematical-functions.md)
- [Пространственные функции](sql-query-spatial-functions.md)
- [Строковые функции](sql-query-string-functions.md)
- [Функции проверки типа](sql-query-type-checking-functions.md)
- [Определенные пользователем функции](sql-query-udfs.md)
- [Статистические выражения](sql-query-aggregate-functions.md)
