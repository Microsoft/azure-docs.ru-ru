---
title: Справочник по геопространственной функции OData
titleSuffix: Azure Cognitive Search
description: Документация по синтаксису и справочной документации по использованию геопространственных функций OData, Geo. Distance и Geo. intersects в запросах Когнитивный поиск Azure.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: 376cece922ca424ec78011224852b1fa5499da16
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88934843"
---
# <a name="odata-geo-spatial-functions-in-azure-cognitive-search---geodistance-and-geointersects"></a>Геопространственные функции OData в Azure Когнитивный поиск `geo.distance` и `geo.intersects`

Azure Когнитивный поиск поддерживает геопространственные запросы в [выражениях фильтра OData](query-odata-filter-orderby-syntax.md) с помощью `geo.distance` `geo.intersects` функций и. `geo.distance`Функция возвращает расстояние в километрах между двумя точками, одно из которых является переменной поля или диапазона, а второй — константой, передаваемой в качестве части фильтра. `geo.intersects`Функция возвращает `true` , если заданная точка находится внутри заданного многоугольника, где точка — это поле или переменная диапазона, а многоугольник — как константа, передаваемая как часть фильтра.

`geo.distance`Функция также может использоваться в [параметре **$OrderBy**](search-query-odata-orderby.md) для сортировки результатов поиска по расстоянию от заданной точки. Синтаксис для `geo.distance` в **$orderby** такой же, как и в **$filter**. При использовании `geo.distance` в **$OrderBy** поле, к которому оно применяется, должно иметь тип `Edm.GeographyPoint` , а также быть **сортируемый**.

> [!NOTE]
> При использовании `geo.distance` в параметре **$OrderBy** поле, передаваемое в функцию, должно содержать только одну географическую точку. Иными словами, он должен иметь тип `Edm.GeographyPoint` , а не `Collection(Edm.GeographyPoint)` . Невозможно выполнить сортировку по полям коллекции в Когнитивный поиск Azure.

## <a name="syntax"></a>Синтаксис

Следующая EBNF ([Расширенная форма Backus-Naur](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) определяет грамматику `geo.distance` `geo.intersects` функций и, а также геопространственные значения, по которым они работают:

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
geo_distance_call ::=
    'geo.distance(' variable ',' geo_point ')'
    | 'geo.distance(' geo_point ',' variable ')'

geo_point ::= "geography'POINT(" lon_lat ")'"

lon_lat ::= float_literal ' ' float_literal

geo_intersects_call ::=
    'geo.intersects(' variable ',' geo_polygon ')'

/* You need at least four points to form a polygon, where the first and
last points are the same. */
geo_polygon ::=
    "geography'POLYGON((" lon_lat ',' lon_lat ',' lon_lat ',' lon_lat_list "))'"

lon_lat_list ::= lon_lat(',' lon_lat)*
```

Доступна также интерактивная схема синтаксиса:

> [!div class="nextstepaction"]
> [Схема синтаксиса OData для Когнитивный поиск Azure](https://azuresearch.github.io/odata-syntax-diagram/#geo_distance_call)

> [!NOTE]
> Полный EBNF см. в [справочнике по синтаксису выражений OData для Azure когнитивный Поиск](search-query-odata-syntax-reference.md) .

### <a name="geodistance"></a>географическое расстояние

`geo.distance`Функция принимает два параметра типа `Edm.GeographyPoint` и возвращает `Edm.Double` значение, которое является расстоянием между ними в километрах. Это отличается от других служб, поддерживающих геопространственные операции OData, которые обычно возвращают расстояния в метрах.

Один из параметров `geo.distance` должен быть константой географического объекта, а другой — путем к полю (или переменной диапазона в случае итерации по полю типа `Collection(Edm.GeographyPoint)` ). Порядок этих параметров не имеет значения.

Константа географической точки имеет форму `geography'POINT(<longitude> <latitude>)'` , где Долгота и Широта являются числовыми константами.

> [!NOTE]
> При использовании `geo.distance` в фильтре необходимо сравнить расстояние, возвращаемое функцией, с константой `lt` , используя, `le` , `gt` или `ge` . Операторы `eq` и `ne` не предназначены для сравнения расстояний. Например, это правильное использование `geo.distance` : `$filter=geo.distance(location, geography'POINT(-122.131577 47.678581)') le 5` .

### <a name="geointersects"></a>географические пересечения

`geo.intersects`Функция принимает переменную типа `Edm.GeographyPoint` и константу `Edm.GeographyPolygon` и возвращает, `Edm.Boolean`  --  `true` Если точка находится внутри границ многоугольника, `false` в противном случае — значение.

Многоугольник — это двухмерная поверхность, которая хранится в виде последовательности точек, определяющих ограничивающее кольцо (см. [примеры](#examples) ниже). Многоугольник должен быть замкнут, то есть первая и конечная точки должны совпадать. [Точки многоугольника должны наноситься в порядке против часовой стрелки](/rest/api/searchservice/supported-data-types#Anchor_1).

### <a name="geo-spatial-queries-and-polygons-spanning-the-180th-meridian"></a>Геопространственные запросы и многоугольники, охватывающие 180TH меридианов

Для многих библиотек геопространственных запросов, составления запрос, включающий 180TH меридианов (вблизи дателине), не ограничивается или требует обходного пути, например разделение многоугольника на два, один с обеих сторон в меридианов.

В Когнитивный поиск Azure геопространственные запросы, включающие долготу в 180 градусов, будут работать должным образом, если фигура запроса прямоугольная, а координаты совпадают с макетом сетки вдоль долготы и широты (например, `geo.intersects(location, geography'POLYGON((179 65, 179 66, -179 66, -179 65, 179 65))'` ). В противном случае для непрямоугольных или невыровненных фигур используйте подход с разделенным многоугольником.  

### <a name="geo-spatial-functions-and-null"></a>Геопространственные функции и `null`

Как и в случае с другими полями, не являющимися коллекциями в Когнитивный поиск Azure, поля типа `Edm.GeographyPoint` могут содержать `null` значения. Когда Когнитивный поиск Azure вычисляет значение `geo.intersects` поля `null` , то результатом всегда будет `false` . Поведение `geo.distance` в этом случае зависит от контекста:

- В фильтрах `geo.distance` `null` поле приводит к `null` . Это означает, что документ не будет совпадать, так как по `null` сравнению с любым значением, отличным от NULL, вычисляется значение `false` .
- При сортировке результатов с помощью **$OrderBy** `geo.distance` `null` поле получает максимально возможное расстояние. Документы с таким полем будут сортироваться ниже остальных, если используется направление сортировки `asc` (по умолчанию) и выше, чем все остальные, если направление имеет значение `desc` .

## <a name="examples"></a>Примеры

### <a name="filter-examples"></a>Примеры фильтрации

Найти все гостиницы в 10 километрах заданной контрольной точки (где Location является полем типа `Edm.GeographyPoint` ):

```odata-filter-expr
    geo.distance(location, geography'POINT(-122.131577 47.678581)') le 10
```

Найти все гостиницы в заданном окне просмотра, описанные в виде многоугольника (где Location — поле типа `Edm.GeographyPoint` ). Обратите внимание, что многоугольник замкнут (первая и последняя точки должны совпадать, и [точки должны быть нанесены в порядке против часовой стрелки](/rest/api/searchservice/supported-data-types#Anchor_1)).

```odata-filter-expr
    geo.intersects(location, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))')
```

### <a name="order-by-examples"></a>Примеры выражений упорядочивания

Сортировка гостиниц по убыванию `rating` , затем по возрастанию по расстоянию от заданных координат:

```odata-filter-expr
    rating desc,geo.distance(location, geography'POINT(-122.131577 47.678581)') asc
```

Сортировать Гостиницы в убывающем порядке по `search.score` и `rating` , а затем в возрастающем порядке по расстоянию от заданных координат, чтобы между двумя гостиницами с одинаковыми рейтингами первым был указан ближайший вариант:

```odata-filter-expr
    search.score() desc,rating desc,geo.distance(location, geography'POINT(-122.131577 47.678581)') asc
```

## <a name="next-steps"></a>Дальнейшие действия  

- [Фильтры в Когнитивный поиск Azure](search-filters.md)
- [Общие сведения о языке выражений OData для Azure Когнитивный поиск](query-odata-filter-orderby-syntax.md)
- [Справочник по синтаксису выражений OData для Azure Когнитивный поиск](search-query-odata-syntax-reference.md)
- [Поиск документов &#40;Azure Когнитивный поиск REST API&#41;](/rest/api/searchservice/Search-Documents)