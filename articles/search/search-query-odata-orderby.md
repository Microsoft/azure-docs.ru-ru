---
title: Порядок сортировки по Справочнику OData
titleSuffix: Azure Cognitive Search
description: Справочная документация по синтаксису и языку для использования предложения ORDER-BY в запросах Когнитивный поиск Azure.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/05/2020
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
ms.openlocfilehash: 83ab2c6b97435ace0d2bc508cbf522600391b60b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88926836"
---
# <a name="odata-orderby-syntax-in-azure-cognitive-search"></a>Синтаксис $orderby OData в Azure Когнитивный поиск

 С помощью [параметра **$OrderBy** OData](query-odata-filter-orderby-syntax.md) можно применить пользовательский порядок сортировки для результатов поиска в Azure когнитивный Поиск. В этой статье подробно описывается синтаксис **$OrderBy** . Дополнительные общие сведения об использовании **$OrderBy** при представлении результатов поиска см. [в статье работа с результатами поиска в Azure когнитивный Поиск](search-pagination-page-layout.md).

## <a name="syntax"></a>Синтаксис

Параметр **$OrderBy** принимает разделенный запятыми список из 32 **предложений упорядочения**. Синтаксис предложения ORDER-BY описывается следующим EBNF ([Расширенная форма Backus-Naur](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)):

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
order_by_clause ::= (field_path | sortable_function) ('asc' | 'desc')?

sortable_function ::= geo_distance_call | 'search.score()'
```

Доступна также интерактивная схема синтаксиса:

> [!div class="nextstepaction"]
> [Схема синтаксиса OData для Когнитивный поиск Azure](https://azuresearch.github.io/odata-syntax-diagram/#order_by_clause)

> [!NOTE]
> Полный EBNF см. в [справочнике по синтаксису выражений OData для Azure когнитивный Поиск](search-query-odata-syntax-reference.md) .

Каждое предложение имеет критерий сортировки, при необходимости за которым следует направление сортировки ( `asc` для по возрастанию или `desc` по убыванию). Если не указать направление, по умолчанию используется значение по возрастанию. Если в поле есть значения NULL, то значения NULL отображаются первыми, если сортировка имеет тип `asc` и Last, если сортировка имеет значение `desc` .

Критерий сортировки может быть либо путем к полю, `sortable` либо вызовом либо [`geo.distance`](search-query-odata-geo-spatial-functions.md) функций, либо [`search.score`](search-query-odata-search-score-function.md) .

Если несколько документов имеют одинаковые условия сортировки и `search.score` функция не используется (например, если сортировка выполняется по числовому `Rating` полю, а все три документа имеют оценку 4), то в убывающем порядке они будут разорваны. Если оценки документа одинаковы (например, если в запросе не указан запрос полнотекстового поиска), относительный порядок связанных документов является неопределенным.

Вы можете определить несколько условий сортировки. Порядок выражений определяет окончательный порядок сортировки. Например, для сортировки по убыванию по показателям, за которыми следует оценка, синтаксис будет выглядеть так `$orderby=search.score() desc,Rating desc` :.

Синтаксис для `geo.distance` в **$orderby** такой же, как и в **$filter**. При использовании функции `geo.distance` в **$orderby** поле, к которому она применяется, должно быть сортируемым (`sortable`) и иметь тип `Edm.GeographyPoint`.

Синтаксис для `search.score` в **$orderby** — `search.score()`. Функция `search.score` не принимает никаких параметров.

## <a name="examples"></a>Примеры

Сортировать отели по возрастанию базового тарифа:

```odata-filter-expr
    $orderby=BaseRate asc
```

Сортировать отели по убыванию рейтинга, а затем по возрастанию базового тарифа (помните, что по умолчанию сортировка происходит по возрастанию):

```odata-filter-expr
    $orderby=Rating desc,BaseRate
```

Сортировка гостиниц по убыванию по рейтингу, затем по возрастанию по расстоянию от заданных координат:

```odata-filter-expr
    $orderby=Rating desc,geo.distance(Location, geography'POINT(-122.131577 47.678581)') asc
```

Сортировка гостиниц в порядке убывания по поиску. Оценка и оценка, а затем в порядке возрастания по расстоянию от заданных координат. Между двумя гостиницами с одинаковыми показателями релевантности и рейтингами в первую очередь указывается ближайшее значение:

```odata-filter-expr
    $orderby=search.score() desc,Rating desc,geo.distance(Location, geography'POINT(-122.131577 47.678581)') asc
```

## <a name="next-steps"></a>Дальнейшие действия  

- [Работа с результатами поиска в Azure Когнитивный поиск](search-pagination-page-layout.md)
- [Общие сведения о языке выражений OData для Azure Когнитивный поиск](query-odata-filter-orderby-syntax.md)
- [Справочник по синтаксису выражений OData для Azure Когнитивный поиск](search-query-odata-syntax-reference.md)
- [Поиск документов &#40;Azure Когнитивный поиск REST API&#41;](/rest/api/searchservice/Search-Documents)