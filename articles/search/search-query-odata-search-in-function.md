---
title: Справочник по функциям search.in OData
titleSuffix: Azure Cognitive Search
description: Документация по синтаксису и справочной документации по использованию функции search.in в запросах Когнитивный поиск Azure.
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
ms.openlocfilehash: 9ad6f89392846564631b70f0acfb5658a050be80
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88922826"
---
# <a name="odata-searchin-function-in-azure-cognitive-search"></a>`search.in`Функция OData в Azure когнитивный Поиск

Распространенным сценарием в [выражениях фильтра OData](query-odata-filter-orderby-syntax.md) является проверка того, что одно поле в каждом документе равно одному из множества возможных значений. Например, некоторые приложения реализуют [фильтрацию безопасности](search-security-trimming-for-azure-search.md) , проверяя поле, содержащее один или несколько идентификаторов субъектов, по списку идентификаторов субъектов, представляющих пользователя, выполняющего запрос. Один из способов написать запрос, подобный этому, заключается в [`eq`](search-query-odata-comparison-operators.md) использовании [`or`](search-query-odata-logical-operators.md) операторов и:

```odata-filter-expr
    group_ids/any(g: g eq '123' or g eq '456' or g eq '789')
```

Однако существует более короткий способ записи, с помощью `search.in` функции:

```odata-filter-expr
    group_ids/any(g: search.in(g, '123, 456, 789'))
```

> [!IMPORTANT]
> Помимо более короткого и простого чтения, использование `search.in` также обеспечивает [преимущества производительности](#bkmk_performance) и предотвращает определенные [ограничения на размер фильтров](search-query-odata-filter.md#bkmk_limits) , если в фильтре содержится сотни или даже тысячи значений. По этой причине мы настоятельно рекомендуем использовать `search.in` вместо более сложного отсоединения выражений равенства.

> [!NOTE]
> В версии 4,01 стандарта OData недавно появился [ `in` оператор](https://docs.oasis-open.org/odata/odata/v4.01/cs01/part2-url-conventions/odata-v4.01-cs01-part2-url-conventions.html#_Toc505773230), аналогичный поведению `search.in` функции в Azure когнитивный Поиск. Однако Когнитивный поиск Azure не поддерживает этот оператор, поэтому `search.in` вместо него необходимо использовать функцию.

## <a name="syntax"></a>Синтаксис

Следующая EBNF ([Расширенная форма Backus-Naur](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) определяет грамматику `search.in` функции:

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
search_in_call ::=
    'search.in(' variable ',' string_literal(',' string_literal)? ')'
```

Доступна также интерактивная схема синтаксиса:

> [!div class="nextstepaction"]
> [Схема синтаксиса OData для Когнитивный поиск Azure](https://azuresearch.github.io/odata-syntax-diagram/#search_in_call)

> [!NOTE]
> Полный EBNF см. в [справочнике по синтаксису выражений OData для Azure когнитивный Поиск](search-query-odata-syntax-reference.md) .

`search.in`Функция проверяет, равен ли заданное строковое поле или переменная диапазона одному из заданных списков значений. Равенство между переменной и каждым значением в списке определяется с учетом регистра так же, как и для `eq` оператора. Следовательно, выражение, подобное `search.in(myfield, 'a, b, c')`, эквивалентно выражению `myfield eq 'a' or myfield eq 'b' or myfield eq 'c'`, за исключением того, что функция `search.in` выполняется намного быстрее.

Существует две перегрузки `search.in` функции:

- `search.in(variable, valueList)`
- `search.in(variable, valueList, delimiters)`

Параметры определены в следующей таблице.

| Имя параметра | Type | Описание |
| --- | --- | --- |
| `variable` | `Edm.String` | Ссылка на строковое поле (или переменная диапазона для поля коллекции строк в случае, когда `search.in` используется в `any` `all` выражении или). |
| `valueList` | `Edm.String` | Строка, содержащая список значений с разделителями для сопоставления с `variable` параметром. Если `delimiters` параметр не указан, разделителями по умолчанию являются пробел и запятая. |
| `delimiters` | `Edm.String` | Строка, в которой каждый символ обрабатывается как разделитель при синтаксическом анализе `valueList` параметра. Значение этого параметра по умолчанию `' ,'` означает, что все значения с пробелами и (или) запятыми между ними будут разделены. Если необходимо использовать разделители, отличные от пробелов и запятых, так как эти символы содержат значения, можно указать альтернативные разделители, например `'|'` в этом параметре. |

<a name="bkmk_performance"></a>

### <a name="performance-of-searchin"></a>Производительность `search.in`

При использовании `search.in` время отклика будет менее секунды, если второй параметр содержит список из сотен или тысяч значений. Нет явного ограничения на количество элементов, которые можно передать `search.in` , хотя по-прежнему ограничивается максимальным размером запроса. Однако задержка растет по мере увеличения количества значений.

## <a name="examples"></a>Примеры

Найти все гостиницы с именем, равным "Sea View Motel" или "Budget Гостиницы". Фразы содержат пробелы, которые являются разделителем по умолчанию. В качестве третьего строкового параметра можно указать альтернативный разделитель в одинарных кавычках:  

```odata-filter-expr
    search.in(HotelName, 'Sea View motel,Budget hotel', ',')
```

Найти все гостиницы с именем, равным "Sea View Motel" или "Budget Гостиницы", разделенными "|"):

```odata-filter-expr
    search.in(HotelName, 'Sea View motel|Budget hotel', '|')
```

Найти все гостиницы с комнатами с тегом "Wi" или "купание":

```odata-filter-expr
    Rooms/any(room: room/Tags/any(tag: search.in(tag, 'wifi, tub')))
```

Найти совпадение по фразам в коллекции, например "нагревани Товел стоек" или "хаирдрер включен" в тегах.

```odata-filter-expr
    Rooms/any(room: room/Tags/any(tag: search.in(tag, 'heated towel racks,hairdryer included', ','))
```

Найти все гостиницы без тега "Motel" или "кабин':

```odata-filter-expr
    Tags/all(tag: not search.in(tag, 'motel, cabin'))
```

## <a name="next-steps"></a>Дальнейшие действия  

- [Фильтры в Когнитивный поиск Azure](search-filters.md)
- [Общие сведения о языке выражений OData для Azure Когнитивный поиск](query-odata-filter-orderby-syntax.md)
- [Справочник по синтаксису выражений OData для Azure Когнитивный поиск](search-query-odata-syntax-reference.md)
- [Поиск документов &#40;Azure Когнитивный поиск REST API&#41;](/rest/api/searchservice/Search-Documents)