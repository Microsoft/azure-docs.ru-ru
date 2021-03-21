---
title: ST_DISTANCE языка запросов Azure Cosmos DB
description: Сведения о ST_DISTANCE системных функций SQL в Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 02/17/2021
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: f01f5faf68821fe9f85657c74111efdbb02bd204
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100559937"
---
# <a name="st_distance-azure-cosmos-db"></a>ST_DISTANCE (Azure Cosmos DB)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

 Возвращает расстояние между двумя геоточечными выражениями, многоугольниками, несколькими многоугольниками и LineString. Дополнительные сведения см. в статье [сведения о геопространственном и географическом расположении данных расположения](sql-query-geospatial-intro.md) .
  
## <a name="syntax"></a>Синтаксис
  
```sql
ST_DISTANCE (<spatial_expr>, <spatial_expr>)  
```  
  
## <a name="arguments"></a>Аргументы
  
*spatial_expr*  
   Любое допустимое выражение объекта GeoJSON (точка, многоугольник или LineString).  
  
## <a name="return-types"></a>Типы возвращаемых данных
  
  Возвращает числовое выражение, указывающее расстояние. При использовании эталонной системы по умолчанию это значение указывается в метрах.  
  
## <a name="examples"></a>Примеры
  
  В следующем примере показано, как вернуть все документы семейства, которые находятся в пределах 30 км из указанного расположения, используя `ST_DISTANCE` встроенную функцию. .  
  
```sql
SELECT f.id
FROM Families f
WHERE ST_DISTANCE(f.location, {'type': 'Point', 'coordinates':[31.9, -4.8]}) < 30000  
```  
  
 Результирующий набор:  
  
```json
[{  
  "id": "WakefieldFamily"  
}]  
```

## <a name="remarks"></a>Remarks

Эта системная функция будет использовать [Геопространственное индексирование](index-policy.md#spatial-indexes) , за исключением запросов со статистическими выражениями.

## <a name="next-steps"></a>Дальнейшие действия

- [Пространственные функции Azure Cosmos DB](sql-query-spatial-functions.md)
- [Системные функции Azure Cosmos DB](sql-query-system-functions.md)
- [Знакомство со службой Azure Cosmos DB. API DocumentDB](introduction.md)
