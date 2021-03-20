---
title: Датетимепарт на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Датетимепарт в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/14/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 97eaff405086190b60279ac7d5cf8bf441c8f840
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93336373"
---
# <a name="datetimepart-azure-cosmos-db"></a>Датетимепарт (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Возвращает значение указанного Датетимепарта между заданным значением DateTime.
  
## <a name="syntax"></a>Синтаксис
  
```sql
DateTimePart (<DateTimePart> , <DateTime>)
```

## <a name="arguments"></a>Аргументы
  
*датетимепарт*  
   Часть даты, для которой Датетимепарт будет возвращать значение. В этой таблице перечислены все допустимые аргументы Датетимепарт:

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

*DateTime*  
   Строковое значение даты и времени ISO 8601 в формате UTC `YYYY-MM-DDThh:mm:ss.fffffffZ`

## <a name="return-types"></a>Типы возвращаемых данных

Возвращает положительное целое значение.

## <a name="remarks"></a>Remarks

Датетимепарт будет возвращать `undefined` по следующим причинам:

- Указано недопустимое значение Датетимепарт
- DateTime не является допустимым значением DateTime ISO 8601

Эта системная функция не будет использовать индекс.

## <a name="examples"></a>Примеры

Ниже приведен пример, который возвращает целочисленное значение месяца.

```sql
SELECT DateTimePart("m", "2020-01-02T03:04:05.6789123Z") AS MonthValue
```

```json
[
    {
        "MonthValue": 1
    }
]
```

Ниже приведен пример, возвращающий число микросекунд:

```sql
SELECT DateTimePart("mcs", "2020-01-02T03:04:05.6789123Z") AS MicrosecondsValue
```

```json
[
    {
        "MicrosecondsValue": 678912
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Функции даты и времени Azure Cosmos DB](sql-query-date-time-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
