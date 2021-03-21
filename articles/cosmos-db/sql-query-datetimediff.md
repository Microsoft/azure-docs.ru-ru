---
title: Датетимедифф на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Датетимедифф в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: aeea2905b6bae094c92bd8b5d46523225c745494
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104595652"
---
# <a name="datetimediff-azure-cosmos-db"></a>Датетимедифф (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]
Возвращает число (как целое число со знаком) указанных границ Датетимепарт, пересекающихся между заданными *StartDate* и *EndDate*.
  
## <a name="syntax"></a>Синтаксис
  
```sql
DateTimeDiff (<DateTimePart> , <StartDate> , <EndDate>)
```

## <a name="arguments"></a>Аргументы
  
*датетимепарт*  
   Часть даты, до которой Датетимеадд добавляет целое число. В этой таблице перечислены все допустимые аргументы Датетимепарт:

| DateTimePart | сокращения;        |
| ------------ | -------------------- |
| Year;         | "Year", "гггг", "гг" |
| Месяц        | "month", "mm", "m"   |
| День          | "Day", "DD", "d"     |
| Час         | "Hour", "HH"         |
| Минута       | "Minute", "MI", "n"  |
| Second       | "Second", "SS", "s"  |
| Миллисекунда  | "миллисекунда", "MS"  |
| Микросекунды  | "микросекунда", "MCS" |
| Потребовавшееся   | "наносекундных", "NS"   |

*/SD*  
    Значение даты и времени в формате UTC 8601 строка в формате, `YYYY-MM-DDThh:mm:ss.fffffffZ` где:
  
|Формат|Описание|
|-|-|
|YYYY|год из четырех цифр|
|ММ|месяц (2 цифры) (01 = Январь и т. д.)|
|DD|четырехзначный день месяца (от 01 до 31)|
|T|обозначает начало элементов времени|
|hh|час с двумя цифрами (от 00 до 23)|
|ММ|двузначное число минут (от 00 до 59)|
|сс|двузначное число секунд (от 00 до 59)|
|. fffffff|семь цифр в долях секунды|
|Z|Обозначение времени в формате UTC (Универсальное координированное время)|
  
  Дополнительные сведения о формате ISO 8601 см. в разделе [ISO_8601](https://en.wikipedia.org/wiki/ISO_8601)

*Предшеству*  
   Строковое значение даты и времени ISO 8601 в формате UTC `YYYY-MM-DDThh:mm:ss.fffffffZ`

## <a name="return-types"></a>Типы возвращаемых данных

Возвращает целочисленное значение со знаком.

## <a name="remarks"></a>Комментарии

Датетимедифф будет возвращать `undefined` по следующим причинам:

- Указано недопустимое значение Датетимепарт
- Дата и время начала или окончания не являются допустимой датой и временем ISO 8601

Датетимедифф всегда возвращает целое число со знаком и представляет собой измерение числа перекрестных границ Датетимепарт, а не измерения интервала времени.

## <a name="examples"></a>Примеры
  
В следующем примере производится вычисление количества границ дня между `2020-01-01T01:02:03.1234527Z` и `2020-01-03T01:02:03.1234567Z` .

```sql
SELECT DateTimeDiff("day", "2020-01-01T01:02:03.1234527Z", "2020-01-03T01:02:03.1234567Z") AS DifferenceInDays
```

```json
[
    {
        "DifferenceInDays": 2
    }
]
```  

В следующем примере производится вычисление числа границ года между `2028-01-01T01:02:03.1234527Z` и `2020-01-03T01:02:03.1234567Z` .

```sql
SELECT DateTimeDiff("yyyy", "2028-01-01T01:02:03.1234527Z", "2020-01-03T01:02:03.1234567Z") AS DifferenceInYears
```

```json
[
    {
        "DifferenceInYears": -8
    }
]
```

В следующем примере производится вычисление числа почасовых границ между `2020-01-01T01:00:00.1234527Z` и `2020-01-01T01:59:59.1234567Z` . Несмотря на то, что эти значения DateTime имеют более 0,99 часов, функция `DateTimeDiff` возвращает 0, так как границы часов не были перепутаны.

```sql
SELECT DateTimeDiff("hh", "2020-01-01T01:00:00.1234527Z", "2020-01-01T01:59:59.1234567Z") AS DifferenceInHours
```

```json
[
    {
        "DifferenceInHours": 0
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Функции даты и времени Azure Cosmos DB](sql-query-date-time-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
