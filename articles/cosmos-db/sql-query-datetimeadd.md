---
title: DateTimeAdd в языке запросов Azure Cosmos DB
description: Дополнительные сведения о системной функции SQL DateTimeAdd в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: c04c63a5ec72f08807b1702f74db39e00662656f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104597663"
---
# <a name="datetimeadd-azure-cosmos-db"></a>DateTimeAdd (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Возвращает строковое значение даты и времени, полученное при добавлении указанного числового значения (в виде целого числа со знаком) в указанную строку даты и времени  
  
## <a name="syntax"></a>Синтаксис
  
```sql
DateTimeAdd (<DateTimePart> , <numeric_expr> ,<DateTime>)
```

## <a name="arguments"></a>Аргументы
  
*DateTimePart*  
   Компонент даты, к которому DateTimeAdd добавляет целое число. В приведенной ниже таблице перечислены все допустимые аргументы DateTimePart:

| DateTimePart | сокращения;        |
| ------------ | -------------------- |
| Year;         | "year", "yyyy", "yy" |
| Месяц        | "month", "mm", "m"   |
| День          | "day", "dd", "d"     |
| Час         | "hour", "hh"         |
| Минута       | "minute", "mi", "n"  |
| Секунда       | "second", "ss", "s"  |
| Миллисекунда  | "millisecond", "ms"  |
| Микросекунда  | "microsecond", "mcs" |
| Наносекунда   | "nanosecond", "ns"   |

*numeric_expr*  
   Целое число со знаком, которое будет добавлено к DateTimePart указанного значения даты и времени

*DateTime*  
   Значение строки даты и времени в формате UTC ISO 8601 в формате `YYYY-MM-DDThh:mm:ss.fffffffZ`, где:
  
|Формат|Описание|
|-|-|
|YYYY|Год из четырех цифр|
|ММ|Месяц из двух цифр (01 = январь и т. д.)|
|DD|Двузначный день месяца (от 01 до 31)|
|T|Обозначает начало элементов времени|
|hh|Час из двух цифр (от 00 до 23)|
|ММ|Минута из двух цифр (от 00 до 59)|
|сс|Секунда из двух цифр (от 00 до 59)|
|.fffffff|Доли секунды из семи цифр|
|Z|Обозначение времени в формате UTC|
  
  Дополнительные сведения о формате ISO 8601 см. в разделе [ISO_8601](https://en.wikipedia.org/wiki/ISO_8601)

## <a name="return-types"></a>Типы возвращаемых данных

Возвращает значение строки даты и времени в формате UTC ISO 8601 в формате `YYYY-MM-DDThh:mm:ss.fffffffZ`, где:
  
|Формат|Описание|
|-|-|
|YYYY|Год из четырех цифр|
|ММ|Месяц из двух цифр (01 = январь и т. д.)|
|DD|Двузначный день месяца (от 01 до 31)|
|T|Обозначает начало элементов времени|
|hh|Час из двух цифр (от 00 до 23)|
|ММ|Минута из двух цифр (от 00 до 59)|
|сс|Секунда из двух цифр (от 00 до 59)|
|.fffffff|Доли секунды из семи цифр|
|Z|Обозначение времени в формате UTC|

## <a name="remarks"></a>Комментарии

DateTimeAdd возвращает `undefined` по следующим причинам:

- Указанное значение DateTimePart недопустимо.
- Указанный numeric_expr не является допустимым целым числом.
- Значение даты и времени в аргументе или результате не является допустимым значением даты и времени по стандарту ISO 8601.

## <a name="examples"></a>Примеры
  
В следующем примере к значению даты и времени добавляется 1 месяц: `2020-07-09T23:20:13.4575530Z`

```sql
SELECT DateTimeAdd("mm", 1, "2020-07-09T23:20:13.4575530Z") AS OneMonthLater
```

```json
[
    {
        "OneMonthLater": "2020-08-09T23:20:13.4575530Z"
    }
]
```  

В следующем примере из значения даты и времени вычитается 2 часа: `2020-07-09T23:20:13.4575530Z`

```sql
SELECT DateTimeAdd("hh", -2, "2020-07-09T23:20:13.4575530Z") AS TwoHoursEarlier
```

```json
[
    {
        "TwoHoursEarlier": "2020-07-09T21:20:13.4575530Z"
    }
]
```  

## <a name="next-steps"></a>Дальнейшие действия

- [Функции даты и времени в Azure Cosmos DB](sql-query-date-time-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
