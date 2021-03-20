---
title: Справочник по синтаксису выражений OData
titleSuffix: Azure Cognitive Search
description: Формальное описание грамматики и синтаксиса для выражений OData в запросах Когнитивный поиск Azure.
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
ms.openlocfilehash: 83e9ae4aa68bc0c819d02a0cc6c39758549811cb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88928859"
---
# <a name="odata-expression-syntax-reference-for-azure-cognitive-search"></a>Справочник по синтаксису выражений OData для Azure Когнитивный поиск

Azure Когнитивный поиск использует [выражения OData](https://docs.oasis-open.org/odata/odata/v4.01/odata-v4.01-part2-url-conventions.html) в качестве параметров во всех API. Чаще всего для параметров и используются выражения OData `$orderby` `$filter` . Эти выражения могут быть сложными, содержащими несколько предложений, функций и операторов. Однако даже простые выражения OData, такие как пути к свойствам, используются во многих частях REST API Когнитивный поиск Azure. Например, выражения пути используются для ссылки на вложенные поля сложных полей везде в API, например при перечислении вложенных полей в [предложении](index-add-suggesters.md), [функции оценки](index-add-scoring-profiles.md), `$select` параметре или даже [при поиске по полям в запросах Lucene](query-lucene-syntax.md).

В этой статье описываются все формы выражений OData с использованием формальной грамматики. Также есть [Интерактивная схема](#syntax-diagram) , позволяющая визуально изучить грамматику.

## <a name="formal-grammar"></a>Формальное грамматика

Мы можем описать подмножество языка OData, поддерживаемого Когнитивный поиском Azure, с помощью грамматики EBNF ([Расширенная форма Backus-Naur](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)). Правила перечислены "сверху вниз", начиная с наиболее сложных выражений и разбивая их на более простые выражения. В верхней части представлены правила грамматики, соответствующие определенным параметрам REST API Когнитивный поиск Azure:

- [`$filter`](search-query-odata-filter.md), определяемое `filter_expression` правилом.
- [`$orderby`](search-query-odata-orderby.md), определяемое `order_by_expression` правилом.
- [`$select`](search-query-odata-select.md), определяемое `select_expression` правилом.
- Пути к полям, определяемые `field_path` правилом. Пути к полям используются через API. Они могут ссылаться либо на поля верхнего уровня индекса, либо на подполя с одним или несколькими предками [поля](search-howto-complex-data-types.md) .

После того как EBNF представляет собой отображаемую [схему синтаксиса](https://en.wikipedia.org/wiki/Syntax_diagram) , которая позволяет интерактивно исследовать грамматику и связи между ее правилами.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
/* Top-level rules */

filter_expression ::= boolean_expression

order_by_expression ::= order_by_clause(',' order_by_clause)*

select_expression ::= '*' | field_path(',' field_path)*

field_path ::= identifier('/'identifier)*


/* Shared base rules */

identifier ::= [a-zA-Z_][a-zA-Z_0-9]*


/* Rules for $orderby */

order_by_clause ::= (field_path | sortable_function) ('asc' | 'desc')?

sortable_function ::= geo_distance_call | 'search.score()'


/* Rules for $filter */

boolean_expression ::=
    collection_filter_expression
    | logical_expression
    | comparison_expression
    | boolean_literal
    | boolean_function_call
    | '(' boolean_expression ')'
    | variable

/* This can be a range variable in the case of a lambda, or a field path. */
variable ::= identifier | field_path

collection_filter_expression ::=
    field_path'/all(' lambda_expression ')'
    | field_path'/any(' lambda_expression ')'
    | field_path'/any()'

lambda_expression ::= identifier ':' boolean_expression

logical_expression ::=
    boolean_expression ('and' | 'or') boolean_expression
    | 'not' boolean_expression

comparison_expression ::= 
    variable_or_function comparison_operator constant | 
    constant comparison_operator variable_or_function

variable_or_function ::= variable | function_call

comparison_operator ::= 'gt' | 'lt' | 'ge' | 'le' | 'eq' | 'ne'


/* Rules for constants and literals */

constant ::=
    string_literal
    | date_time_offset_literal
    | integer_literal
    | float_literal
    | boolean_literal
    | 'null'

string_literal ::= "'"([^'] | "''")*"'"

date_time_offset_literal ::= date_part'T'time_part time_zone

date_part ::= year'-'month'-'day

time_part ::= hour':'minute(':'second('.'fractional_seconds)?)?

zero_to_fifty_nine ::= [0-5]digit

digit ::= [0-9]

year ::= digit digit digit digit

month ::= '0'[1-9] | '1'[0-2]

day ::= '0'[1-9] | [1-2]digit | '3'[0-1]

hour ::= [0-1]digit | '2'[0-3]

minute ::= zero_to_fifty_nine

second ::= zero_to_fifty_nine

fractional_seconds ::= integer_literal

time_zone ::= 'Z' | sign hour':'minute

sign ::= '+' | '-'

/* In practice integer literals are limited in length to the precision of
the corresponding EDM data type. */
integer_literal ::= digit+

float_literal ::=
    sign? whole_part fractional_part? exponent?
    | 'NaN'
    | '-INF'
    | 'INF'

whole_part ::= integer_literal

fractional_part ::= '.'integer_literal

exponent ::= 'e' sign? integer_literal

boolean_literal ::= 'true' | 'false'


/* Rules for functions */

function_call ::=
    geo_distance_call |
    boolean_function_call

geo_distance_call ::=
    'geo.distance(' variable ',' geo_point ')'
    | 'geo.distance(' geo_point ',' variable ')'

geo_point ::= "geography'POINT(" lon_lat ")'"

lon_lat ::= float_literal ' ' float_literal

boolean_function_call ::=
    geo_intersects_call |
    search_in_call |
    search_is_match_call

geo_intersects_call ::=
    'geo.intersects(' variable ',' geo_polygon ')'

/* You need at least four points to form a polygon, where the first and
last points are the same. */
geo_polygon ::=
    "geography'POLYGON((" lon_lat ',' lon_lat ',' lon_lat ',' lon_lat_list "))'"

lon_lat_list ::= lon_lat(',' lon_lat)*

search_in_call ::=
    'search.in(' variable ',' string_literal(',' string_literal)? ')'

/* Note that it is illegal to call search.ismatch or search.ismatchscoring
from inside a lambda expression. */
search_is_match_call ::=
    'search.ismatch'('scoring')?'(' search_is_match_parameters ')'

search_is_match_parameters ::=
    string_literal(',' string_literal(',' query_type ',' search_mode)?)?

query_type ::= "'full'" | "'simple'"

search_mode ::= "'any'" | "'all'"
```

## <a name="syntax-diagram"></a>Схема синтаксиса

Чтобы визуально исследовать грамматику языка OData, поддерживаемую Когнитивный поиск Azure, воспользуйтесь интерактивной схемой синтаксиса:

> [!div class="nextstepaction"]
> [Схема синтаксиса OData для Когнитивный поиск Azure](https://azuresearch.github.io/odata-syntax-diagram/)

## <a name="see-also"></a>См. также раздел  

- [Фильтры в Когнитивный поиск Azure](search-filters.md)
- [Поиск документов &#40;Azure Когнитивный поиск REST API&#41;](/rest/api/searchservice/Search-Documents)
- [Синтаксис запросов Lucene](query-lucene-syntax.md)
- [Простой синтаксис запросов в Azure Когнитивный поиск](query-simple-syntax.md)