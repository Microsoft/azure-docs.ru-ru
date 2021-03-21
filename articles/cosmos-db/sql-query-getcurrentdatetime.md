---
title: Жеткуррентдатетиме на языке запросов Azure Cosmos DB
description: Дополнительные сведения о функции SQL System Жеткуррентдатетиме в Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 02/03/2021
ms.author: tisande
ms.custom: query-reference
ms.openlocfilehash: 12ce8beab082674cd7672713325d4b3f4322aeae
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104587310"
---
# <a name="getcurrentdatetime-azure-cosmos-db"></a>Жеткуррентдатетиме (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Возвращает текущее значение даты и времени (в формате UTC) в виде строки ISO 8601.
  
## <a name="syntax"></a>Синтаксис
  
```sql
GetCurrentDateTime ()
```

## <a name="return-types"></a>Типы возвращаемых данных
  
Возвращает строковое значение даты и времени в формате UTC 8601 в формате, `YYYY-MM-DDThh:mm:ss.fffffffZ` где:
  
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

Жеткуррентдатетиме () является недетерминированной функцией. Возвращаемый результат — UTC. Точность составляет 7 цифр с точностью 100 наносекунд.

> [!NOTE]
> Эта системная функция не будет использовать индекс. Если необходимо сравнить значения с текущим временем, получите текущее время перед выполнением запроса и используйте это постоянное строковое значение в `WHERE` предложении.

## <a name="examples"></a>Примеры
  
В следующем примере показано, как получить текущую дату и время в формате UTC с помощью встроенной функции Жеткуррентдатетиме ().
  
```sql
SELECT GetCurrentDateTime() AS currentUtcDateTime
```  
  
 Ниже приведен пример результирующего набора.
  
```json
[{
  "currentUtcDateTime": "2019-05-03T20:36:17.1234567Z"
}]  
```  

## <a name="next-steps"></a>Дальнейшие действия

- [Функции даты и времени Azure Cosmos DB](sql-query-date-time-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
