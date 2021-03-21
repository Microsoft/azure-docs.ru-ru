---
title: Жеткурренттиккс на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Жеткурренттиккс в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 02/03/2021
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 16004e6e471094c99229c32a63396ac3b0490905
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99524313"
---
# <a name="getcurrentticks-azure-cosmos-db"></a>Жеткурренттиккс (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Возвращает число 100-наносекундных тактов, истекших с 00:00:00 четверг, 1 января 1970.
  
## <a name="syntax"></a>Синтаксис
  
```sql
GetCurrentTicks ()
```

## <a name="return-types"></a>Типы возвращаемых данных

Возвращает числовое значение со знаком, текущее число 100-наносекундных тактов, прошедшее с момента создания эпохи UNIX. Иными словами, Жеткурренттиккс возвращает число импульсов 100 наносекунд, истекших с 00:00:00 четверг, 1 января 1970.

## <a name="remarks"></a>Remarks

Жеткурренттиккс () является недетерминированной функцией. Возвращаемый результат — UTC (всемирное время).

> [!NOTE]
> Эта системная функция не будет использовать индекс. Если необходимо сравнить значения с текущим временем, получите текущее время перед выполнением запроса и используйте это постоянное строковое значение в `WHERE` предложении.

## <a name="examples"></a>Примеры

Ниже приведен пример, который возвращает текущее время, измеряемое в тактах:

```sql
SELECT GetCurrentTicks() AS CurrentTimeInTicks
```

```json
[
    {
        "CurrentTimeInTicks": 15973607943002652
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия

- [Функции даты и времени Azure Cosmos DB](sql-query-date-time-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
