---
title: Фильтрация по результатам поиска
titleSuffix: Azure Cognitive Search
description: Фильтрация по идентификатору безопасности пользователя, языку, географическому расположению или числовым значениям для сокращения результатов поиска по запросам в Когнитивный поиск Azure, размещенной облачной службе поиска на Microsoft Azure.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/02/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: ba538f4753c2365406bd88286b6d54cff1a9e9ea
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104800828"
---
# <a name="filters-in-azure-cognitive-search"></a>Фильтры в Когнитивный поиск Azure 

*Фильтр* предоставляет критерии на основе значений для выбора документов, используемых в запросе. Фильтр может быть одиночным значением или [выражением фильтра](search-query-odata-filter.md)OData. В отличие от полнотекстового поиска, значение фильтра или выражение возвращает только полное соответствие.

Некоторые возможности поиска, такие как [многогранная Навигация](search-filters-facets.md), зависят от фильтров в рамках реализации, но вы можете использовать фильтры в любое время, чтобы ограничить область запроса определенными значениями. Если вместо этого требуется ограничить область запроса конкретными полями, существуют альтернативные методы, описанные ниже.

## <a name="when-to-use-a-filter"></a>Когда следует использовать фильтр

Фильтры основаны на нескольких поисковых функциях, включая функцию поиска соседних объектов, навигацию по аспектам и фильтры безопасности, которые показывают только разрешенные документы. При реализации любой из этих функций требуется фильтр. Это фильтр, прикрепленный к поисковому запросу, который предоставляет координаты геолокации, выбранную пользователем категорию аспекта или идентификатор безопасности запрашивающего.

Ниже перечислены распространенные сценарии.

+ Срез результатов поиска на основе содержимого в индексе. Учитывая схему с расположением отеля, категориями и удобствами, вы можете создать фильтр для явного сопоставления по критерию (в Сиэтле, на воду с представлением). 

+ Реализация поиска имеет требования фильтра:

  + [Навигация по аспектам](search-faceted-navigation.md) использует фильтр для возврата категории аспекта, выбранной пользователем.
  + Геопространственный поиск использует фильтр для передачи координат текущего местоположения в приложениях с возможностью поиска соседних объектов. 
  + [Фильтры безопасности](search-security-trimming-for-azure-search.md) передают идентификаторы безопасности как условия фильтра, где соответствие в индексе служит прокси-сервером для прав доступа к документу.

+ Выполните поиск чисел. Числовые поля доступны для извлечения и могут отображаться в результатах поиска, но не доступны для поиска (подчиняются полнотекстовому поиску) по отдельности. Если вам нужны критерии выбора, основанные на числовых данных, используйте фильтр.

### <a name="alternative-methods-for-reducing-scope"></a>Альтернативные методы для сокращения области поиска

Если требуется сузить область поиска в результатах, можно использовать не только фильтры. Эти возможности могут быть более эффективными в зависимости от задачи:

+ `searchFields` параметр запроса позволяет ограничивать поиск определенными полями. Например, если в вашем индексе содержатся отдельные поля для описания на английском и испанском языках, вы можете использовать searchFields, чтобы указать конкретные поля для полнотекстового поиска. 

+ Параметр `$select` используется, чтобы указать поля, которые требуется включить в результирующий набор для эффективной обрезки ответа перед его отправкой вызывающему приложению. Этот параметр не позволяет уточнить запрос или сократить коллекцию документов, но если вы не отвечаете за более маленький ответ, этот параметр можно рассмотреть. 

Дополнительные сведения о параметрах запроса см. в подразделе ["Параметры запроса" раздела "Запрос" статьи Search Documents (Azure Search Service REST API)](/rest/api/searchservice/search-documents#query-parameters) (Поиск документов (REST API службы "Поиск" Azure)).

## <a name="how-filters-are-executed"></a>Как выполняются фильтры

Во время запроса средство синтаксического анализа фильтров принимает критерии в качестве входных данных, преобразует выражение в атомарные логические выражения, представленные в виде дерева, а затем вычисляет дерево фильтров по фильтруемым полям в индексе.

Фильтрация выполняется в сочетании с поиском, определяющим, какие документы следует включить в подчиненную обработку для получения документа и оценки релевантности. При связывании со строкой поиска фильтр эффективно сокращает набор отзывов последующей операции поиска. При использовании отдельно (например, когда строка запроса пуста, где `search=*`), критерием фильтра являются только входные данные. 

## <a name="defining-filters"></a>Определение фильтров

Фильтры — это выражения OData, которые выводятся в [синтаксисе фильтров](search-query-odata-filter.md) , поддерживаемом когнитивный Поиск.

Можно указать один фильтр для каждой операции **поиска** , но сам фильтр может включать несколько полей, несколько условий, а при использовании функции **Match** — несколько выражений полнотекстового поиска. В выражении фильтра из нескольких частей можно указать предикаты в любом порядке (в соответствии с правилами приоритета операторов). Попытка поставить предикаты в определенной последовательности не даст значительного прироста производительности.

Одним из ограничений критерия фильтра является максимальный размер запроса. Весь запрос, включая функции фильтра, не может содержать более 16 МБ данных для POST или 8 КБ для GET. Кроме того, существует ограничение на количество предложений в выражении фильтра. Хорошее проверенное правило: если у вас есть сотни предложений, вы рискуете превысить лимит. Мы рекомендуем разрабатывать приложение таким образом, чтобы оно не создавало фильтры неограниченного размера.

Следующие примеры представляют собой прототипные определения фильтров в нескольких API.

```http
# Option 1:  Use $filter for GET
GET https://[service name].search.windows.net/indexes/hotels/docs?api-version=2020-06-30&search=*&$filter=Rooms/any(room: room/BaseRate lt 150.0)&$select=HotelId, HotelName, Rooms/Description, Rooms/BaseRate

# Option 2: Use filter for POST and pass it in the request body
POST https://[service name].search.windows.net/indexes/hotels/docs/search?api-version=2020-06-30
{
    "search": "*",
    "filter": "Rooms/any(room: room/BaseRate lt 150.0)",
    "select": "HotelId, HotelName, Rooms/Description, Rooms/BaseRate"
}
```

```csharp
    parameters =
        new SearchParameters()
        {
            Filter = "Rooms/any(room: room/BaseRate lt 150.0)",
            Select = new[] { "HotelId", "HotelName", "Rooms/Description" ,"Rooms/BaseRate"}
        };

    var results = searchIndexClient.Documents.Search("*", parameters);
```

## <a name="filter-usage-patterns"></a>Шаблоны использования фильтров

В следующих примерах показаны несколько шаблонов использования для сценариев фильтрации. Дополнительные примеры можно найти в разделе [Примеры OData](./search-query-odata-filter.md#examples).

+ Автономный параметр **$filter** без строки запроса полезен, когда выражение фильтра может полностью определить интересующие документы. Без строки запроса не выполняется лексический или лингвистический анализ, нет оценки и рейтинга. Обратите внимание, что строка поиска — это просто звездочка, что означает "сопоставить все документы".

  ```http
  {
    "search": "*",
    "filter": "Rooms/any(room: room/BaseRate ge 60 and room/BaseRate lt 300) and Address/City eq 'Honolulu"
  }
  ```

+ Сочетание строки запроса и **$filter**, где фильтр создает подмножество, а строка запроса предоставляет термины для полнотекстового поиска по отфильтрованному подмножеству. Добавление терминов («театров расстояний») приводит к получению оценок в результатах поиска, где документы, которые лучше соответствуют условиям, ранжированы выше. Использование фильтра с строкой запроса является наиболее распространенным шаблоном использования.

  ```http
  {
    "search": "walking distance theaters",
    "filter": "Rooms/any(room: room/BaseRate ge 60 and room/BaseRate lt 300) and Address/City eq 'Seattle'"
  }

+ Compound queries, separated by "or", each with its own filter criteria (for example, 'beagles' in 'dog' or 'siamese' in 'cat'). Expressions combined with `or` are evaluated individually, with the union of documents matching each expression sent back in the response. This usage pattern is achieved through the `search.ismatchscoring` function. You can also use the non-scoring version, `search.ismatch`.

   ```
   # <a name="match-on-hostels-rated-higher-than-4-or-5-star-motels"></a>Совпадение с Хостелс, номинальным выше 4 или 5-звездой мотелей.
   $filter = Search. исматчскоринг (' общежития ') и оценка GE 4 или Search. исматчскоринг (' Motel ') и рейтинг EQ 5

   # <a name="match-on-luxury-or-high-end-in-the-description-field-or-on-category-exactly-equal-to-luxury"></a>Совпадение с "люкс" или "High-End" в поле описания или в категории в точности равным "люкс".
   $filter = Search. исматчскоринг ("люкс | High-End", "Description") или Category EQ "люкс" &$count = true
   ```

  It is also possible to combine full-text search via `search.ismatchscoring` with filters using `and` instead of `or`, but this is functionally equivalent to using the `search` and `$filter` parameters in a search request. For example, the following two queries produce the same result:

  ```
  $filter = Search. исматчскоринг ("пул") и оценка GE 4

  Поиск = пул&$filter = Оценка GE 4
  ```

Follow up with these articles for comprehensive guidance on specific use cases:

+ [Facet filters](search-filters-facets.md)
+ [Security trimming](search-security-trimming-for-azure-search.md) 

## Field requirements for filtering

In the REST API, filterable is *on* by default for simple fields. Filterable fields increase index size; be sure to set `"filterable": false` for fields that you don't plan to actually use in a filter. For more information about settings for field definitions, see [Create Index](/rest/api/searchservice/create-index).

In the .NET SDK, the filterable is *off* by default. You can make a field filterable by setting the [IsFilterable property](/dotnet/api/azure.search.documents.indexes.models.searchfield.isfilterable) of the corresponding [SearchField](/dotnet/api/azure.search.documents.indexes.models.searchfield) object to `true`. In the example below, the attribute is set on the `BaseRate` property of a model class that maps to the index definition.

```csharp
[IsFilterable, IsSortable, IsFacetable]
public double? BaseRate { get; set; }
```

### <a name="making-an-existing-field-filterable"></a>Обеспечение фильтрации существующего поля

Нельзя изменить существующие поля, чтобы сделать их фильтруемыми. Вместо этого необходимо добавить новое поле или перестроить индекс. Дополнительные сведения о перестроении индекса или повторном заполнении полей см. [в статье перестроение индекса Azure когнитивный Поиск](search-howto-reindex.md).

## <a name="text-filter-fundamentals"></a>Принципы работы текстовых фильтров

Текстовые фильтры соответствуют строковым полям строк литералов, которые вы задаюте в фильтре: `$filter=Category eq 'Resort and Spa'`

В отличие от полнотекстового поиска, для текстовых фильтров не существует лексического анализа или разбиения слов, поэтому сравнения предназначены только для точных совпадений. Например, предположим, что поле *f* содержит "Sunny Day", `$filter=f eq 'Sunny'` не соответствует, но `$filter=f eq 'sunny day'` будет. 

Текстовые строки учитывают регистр. Отсутствует нижний регистр слов с прописными буквами: `$filter=f eq 'Sunny day'` не находит "Sunny Day".

### <a name="approaches-for-filtering-on-text"></a>Подходы к фильтрации по тексту

| Метод | Описание | Назначение |
|----------|-------------|-------------|
| [`search.in`](search-query-odata-search-in-function.md) | Функция, которая сопоставляет поле со списком строк с разделителями. | Рекомендуется для [фильтров безопасности](search-security-trimming-for-azure-search.md) и для любых фильтров, в которых несколько необработанных текстовых значений должны сопоставляться с строковым полем. Функция **Search.in** разработана для ускорения и намного быстрее, чем явно сравнивать поле с каждой строкой с помощью `eq` и `or` . | 
| [`search.ismatch`](search-query-odata-full-text-search-functions.md) | Функция, которая позволяет совместно использовать операции полнотекстового поиска вместе с операциями строго логического фильтра в одном выражении фильтра. | Используйте **Поиск. Match** (или его эквивалент, **Search. исматчскоринг**), если требуется несколько сочетаний фильтра поиска в одном запросе. Вы также можете использовать ее для фильтра *contains* (для фильтрации в частичной строке в контексте большей строки). |
| [`$filter=field operator string`](search-query-odata-comparison-operators.md) | Определенное пользователем выражение, состоящее из поля, операторов и значений. | Используйте этот параметр, если нужно найти точные соответствия между строковым и строковым значениями. |

## <a name="numeric-filter-fundamentals"></a>Основные компоненты числового фильтра

Числовые поля недоступны для поиска (`searchable`) в контексте полнотекстового поиска. Только строки подлежат полному текстовому поиску. Например, при вводе 99,99 в качестве поискового запроса вы не сможете найти товары по цене 99,99 доллара США. Вместо этого отображаются элементы, которые содержат строку поля документа с номером 99. Таким образом, если у вас есть числовые данные, предполагается, что вы будете использовать их для фильтров, включая диапазоны, аспекты, группы и т. д. 

Документы, содержащие числовые поля (цена, размер, SKU, идентификатор), предоставляют эти значения в результатах поиска, если поле отмечено `retrievable`. Суть в том, что полнотекстовый поиск не применим к числовым типам полей.

## <a name="next-steps"></a>Следующие шаги

Сначала попробуйте использовать **обозреватель поиска** на портале, чтобы отправить запросы с параметрами **$filter**. [Пример индекса выборки недвижимости](search-get-started-portal.md) предоставляет полезные результаты для следующих отфильтрованных запросов при их вставке в строку поиска:

```
# Geo-filter returning documents within 5 kilometers of Redmond, Washington state
# Use $count=true to get a number of hits returned by the query
# Use $select to trim results, showing values for named fields only
# Use search=* for an empty query string. The filter is the sole input

search=*&$count=true&$select=description,city,postCode&$filter=geo.distance(location,geography'POINT(-122.121513 47.673988)') le 5

# Numeric filters use comparison like greater than (gt), less than (lt), not equal (ne)
# Include "and" to filter on multiple fields (baths and bed)
# Full text search is on John Leclerc, matching on John or Leclerc

search=John Leclerc&$count=true&$select=source,city,postCode,baths,beds&$filter=baths gt 3 and beds gt 4

# Text filters can also use comparison operators
# Wrap text in single or double quotes and use the correct case
# Full text search is on John Leclerc, matching on John or Leclerc

search=John Leclerc&$count=true&$select=source,city,postCode,baths,beds&$filter=city gt 'Seattle'
```

Дополнительные примеры можно найти в разделе [Примеры OData](./search-query-odata-filter.md#examples).

## <a name="see-also"></a>См. также раздел

+ [How full text search works in Azure Cognitive Search](search-lucene-query-architecture.md) (Как выполняется полнотекстовый поиск в Когнитивном поиске Azure)
+ [Search Documents (Azure Search Service REST API)](/rest/api/searchservice/search-documents) (Поиск по документам (REST API службы поиска Azure))
+ [Синтаксис простых запросов](/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [Синтаксис запросов Lucene](/rest/api/searchservice/lucene-query-syntax-in-azure-search)
+ [Поддерживаемые типы данных](/rest/api/searchservice/supported-data-types)