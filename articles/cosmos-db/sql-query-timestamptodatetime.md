---
title: Тиместамптодатетиме на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Тиместамптодатетиме в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 08/18/2020
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: e9a77ab0ac32d627d59e2cb0fa4a680f174a6833
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104587242"
---
# <a name="timestamptodatetime-azure-cosmos-db"></a>Тиместамптодатетиме (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Преобразует указанное значение метки времени в дату и время.
  
## <a name="syntax"></a>Синтаксис
  
```sql
TimestampToDateTime (<Timestamp>)
```

## <a name="arguments"></a>Аргументы

*Timestamp*  

Числовое значение со знаком, равное текущему количеству миллисекунд, прошедших с момента создания эпохи UNIX. Иными словами, число миллисекунд, прошедших с 00:00:00 четверг, 1 января 1970.

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

Тиместамптодатетиме будет возвращать `undefined` значение, если указана недопустимая метка времени.

## <a name="examples"></a>Примеры
  
В следующем примере метка времени преобразуется в дату и время:

```sql
SELECT TimestampToDateTime(1594227912345) AS DateTime
```

```json
[
    {
        "DateTime": "2020-07-08T17:05:12.3450000Z"
    }
]
```  

## <a name="next-steps"></a>Дальнейшие действия

- [Функции даты и времени Azure Cosmos DB](sql-query-date-time-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
