---
title: Тиккстодатетиме на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Тиккстодатетиме в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/18/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 9adf525f4edb4104978bae32bbbef15ffb131f68
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104587259"
---
# <a name="tickstodatetime-azure-cosmos-db"></a>Тиккстодатетиме (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Преобразует указанное значение тактов в тип DateTime.
  
## <a name="syntax"></a>Синтаксис
  
```sql
TicksToDateTime (<Ticks>)
```

## <a name="arguments"></a>Аргументы

*Тактов*  

Числовое значение со знаком, текущее число 100 наносекундных тактов, прошедшее с момента создания эпохи UNIX. Иными словами, это число 100 наносекундных тактов, истекших с 00:00:00 четверг, 1 января 1970.

## <a name="return-types"></a>Типы возвращаемых данных

Возвращает строковое значение даты и времени ISO 8601 по ГРИНВИЧу в формате, `YYYY-MM-DDThh:mm:ss.fffffffZ` где:
  
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

## <a name="remarks"></a>Комментарии

Тиккстодатетиме будет возвращать, `undefined` Если указанное значение тактов недопустимо.

## <a name="examples"></a>Примеры
  
В следующем примере кванты преобразуются в DateTime:

```sql
SELECT TicksToDateTime(15943368134575530) AS DateTime
```

```json
[
    {
        "DateTime": "2020-07-09T23:20:13.4575530Z"
    }
]
```  

## <a name="next-steps"></a>Дальнейшие действия

- [Функции даты и времени Azure Cosmos DB](sql-query-date-time-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
