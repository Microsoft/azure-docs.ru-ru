---
title: Использование простого синтаксиса запроса Lucene
titleSuffix: Azure Cognitive Search
description: Примеры запросов, демонстрирующие простой синтаксис полнотекстового поиска, фильтрацию поиска и географический поиск по индексу Azure Когнитивный поиск.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: 2abe19351c92bf9cea85c85dd55f47b5ee6d1625
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101694042"
---
# <a name="use-the-simple-search-syntax-in-azure-cognitive-search"></a>Использование синтаксиса "простого" поиска в Azure Когнитивный поиск

В Когнитивный поиск Azure [простой синтаксис запроса](query-simple-syntax.md) вызывает средство синтаксического анализа запросов по умолчанию для полнотекстового поиска. Средство синтаксического анализа работает быстро и обрабатывает распространенные сценарии, включая полнотекстовый поиск, фильтрованный и аспектный Поиск, а также Префиксный поиск. В этой статье приводятся примеры, иллюстрирующие простое использование синтаксиса в запросе [поиска документов (REST API)](/rest/api/searchservice/search-documents) .

> [!NOTE]
> Альтернативный синтаксис запросов — это [полная Lucene](query-lucene-syntax.md), поддерживающая более сложные структуры запросов, например поиск нечетких и подстановочных знаков. Дополнительные сведения и примеры см. [в разделе Использование полного синтаксиса Lucene](search-query-lucene-examples.md).

## <a name="hotels-sample-index"></a>Индекс примера гостиниц

Следующие запросы основываются на гостиницах, которые можно создать, следуя инструкциям в этом [кратком руководстве](search-get-started-portal.md).

Примеры запросов обрабатываются с помощью запросов REST API и POST. Их можно вставить и запустить в [POST](search-get-started-rest.md) или в [Visual Studio Code с расширением когнитивный Поиск](search-get-started-vs-code.md).

Заголовки запроса должны иметь следующие значения:

| Ключ | Значение |
|-----|-------|
| Content-Type | приложение/json|
| api-key  | `<your-search-service-api-key>`, либо запрос, либо ключ администратора |

Параметры URI должны включать конечную точку службы поиска с именем индекса, коллекциями документов, командой поиска и версией API, как показано в следующем примере:

```http
https://{{service-name}}.search.windows.net/indexes/hotels-sample-index/docs/search?api-version=2020-06-30
```

Текст запроса должен быть сформирован как допустимый JSON:

```json
{
    "search": "*",
    "queryType": "simple",
    "select": "HotelId, HotelName, Category, Tags, Description",
    "count": true
}
```

+ для параметра "Search" задан `*` неопределенный запрос, эквивалентный null или пустому поиску. Это не особенно полезно, но это самый простой поиск, который можно сделать, и Показать все доступные для получения поля в индексе со всеми значениями.

+ для "queryType" задано значение "Simple" (по умолчанию), и его можно опустить, но он включен для дальнейшего изучения того, что примеры запросов в этой статье выражаются в простом синтаксисе.

+ для композиции результатов поиска используется значение "Select", равное разделенному запятыми списку полей, включая только те поля, которые полезны в контексте результатов поиска.

+ "Count" возвращает количество документов, соответствующих условиям поиска. В пустой строке поиска счетчик будет содержать все документы в индексе (50 в случае использования гостиниц-Sample-index).

## <a name="example-1-full-text-search"></a>Пример 1. полнотекстовый поиск

Полнотекстовый поиск может быть любым числом отдельных терминов или заключенными в кавычки фразами с логическими операторами или без них. 

```http
POST /indexes/hotel-samples-index/docs/search?api-version=2020-06-30
{
    "search": "pool spa +airport",
    "searchMode": any,
    "queryType": "simple",
    "select": "HotelId, HotelName, Category, Description",
    "count": true
}
```

Поиск по ключевым словам, который состоит из важных терминов или фраз, обычно работает лучше всего. Строковые поля пропретерпевают анализ текста во время индексирования и запроса, удаляя неключевые слова, такие как «The», «and», «IT». Чтобы увидеть, как строка запроса размечена в индексе, передайте строку в индексе в текстовом вызове [анализа](/rest/api/searchservice/test-analyzer) .

Параметр "searchMode" контролирует точность и отзыв. Если требуется более удачный отзыв, используйте значение по умолчанию Any, которое возвращает результат при сопоставлении какой-либо части строки запроса. Если вы предпочитаете точность, где все части строки должны быть сопоставлены, измените searchMode на "ALL". Попробуйте выполнить приведенный выше запрос обоими способами, чтобы увидеть, как searchMode изменяет результат.

Ответ для запроса "защищено пулом" и "аэропорта" должен выглядеть примерно так, как показано в следующем примере, обрезанный для краткости.

```json
"@odata.count": 6,
"value": [
    {
        "@search.score": 7.3617697,
        "HotelId": "21",
        "HotelName": "Nova Hotel & Spa",
        "Description": "1 Mile from the airport.  Free WiFi, Outdoor Pool, Complimentary Airport Shuttle, 6 miles from the beach & 10 miles from downtown.",
        "Category": "Resort and Spa",
        "Tags": [
            "pool",
            "continental breakfast",
            "free parking"
        ]
    },
    {
        "@search.score": 2.5560288,
        "HotelId": "25",
        "HotelName": "Scottish Inn",
        "Description": "Newly Redesigned Rooms & airport shuttle.  Minutes from the airport, enjoy lakeside amenities, a resort-style pool & stylish new guestrooms with Internet TVs.",
        "Category": "Luxury",
        "Tags": [
            "24-hour front desk service",
            "continental breakfast",
            "free wifi"
        ]
    },
    {
        "@search.score": 2.2988036,
        "HotelId": "35",
        "HotelName": "Suites At Bellevue Square",
        "Description": "Luxury at the mall.  Located across the street from the Light Rail to downtown.  Free shuttle to the mall and airport.",
        "Category": "Resort and Spa",
        "Tags": [
            "continental breakfast",
            "air conditioning",
            "24-hour front desk service"
        ]
    }
```

Обратите внимание на оценку поиска в ответе. Это показатель релевантности совпадения. По умолчанию служба поиска вернет первые 50 соответствий на основе этой оценки.

Однородные баллы "1,0" возникают, если нет ранга, так как поиск не является полнотекстовым поиском или не был предоставлен ни один критерий. Например, в пустом поиске (Search = `*` ) строки возвращаются в произвольном порядке. Когда вы добавите условие поиска, вы увидите, как оценки поиска превратятся в понятные значения.

## <a name="example-2-look-up-by-id"></a>Пример 2. Поиск по идентификатору

При возврате результатов поиска в запросе логический следующий шаг — предоставление страницы сведений, содержащей больше полей из документа. В этом примере показано, как вернуть один документ с помощью [документа подстановки](/rest/api/searchservice/lookup-document) , ПЕРЕДАВ ему идентификатор документа.

```http
GET /indexes/hotels-sample-index/docs/41?api-version=2020-06-30
```

Все документы имеют уникальный идентификатор. Если вы используете портал, выберите вкладку Индекс из **индексов** , а затем просмотрите определения полей, чтобы определить, какое поле является ключом. При использовании функции RESTFUL вызов [индекса Get](/rest/api/searchservice/get-index) Возвращает определение индекса в тексте ответа.

Ответ для приведенного выше запроса состоит из документа, ключ которого равен 41. Любое поле, помеченное в определении индекса как "извлекаемое", может быть возвращено в результатах поиска и визуализировано в приложении.

```json
{
    "HotelId": "41",
    "HotelName": "Ocean Air Motel",
    "Description": "Oceanfront hotel overlooking the beach features rooms with a private balcony and 2 indoor and outdoor pools. Various shops and art entertainment are on the boardwalk, just steps away.",
    "Description_fr": "L'hôtel front de mer surplombant la plage dispose de chambres avec balcon privé et 2 piscines intérieures et extérieures. Divers commerces et animations artistiques sont sur la promenade, à quelques pas.",
    "Category": "Budget",
    "Tags": [
        "pool",
        "air conditioning",
        "bar"
    ],
    "ParkingIncluded": true,
    "LastRenovationDate": "1951-05-10T00:00:00Z",
    "Rating": 3.5,
    "Location": {
        "type": "Point",
        "coordinates": [
            -157.846817,
            21.295841
        ],
        "crs": {
            "type": "name",
            "properties": {
                "name": "EPSG:4326"
            }
        }
    },
    "Address": {
        "StreetAddress": "1450 Ala Moana Blvd 2238 Ala Moana Ctr",
        "City": "Honolulu",
        "StateProvince": "HI",
        "PostalCode": "96814",
        "Country": "USA"
    },
```

## <a name="example-3-filter-on-text"></a>Пример 3. Фильтрация по тексту

[Синтаксис фильтра](search-query-odata-filter.md) — это выражение OData, которое можно использовать самостоятельно или с помощью поиска. Используется вместе, "Filter" сначала применяется ко всему индексу, а затем выполняется поиск по результатам фильтра. Поэтому фильтры могут повысить производительность запросов, так как они уменьшают количество документов, которые поисковый запрос должен обработать.

Фильтры могут быть определены для любого поля, помеченного как "фильтруемое" в определении индекса. Для гостиниц-выборка — индекс, фильтруемые поля: Категория, теги, Паркингинклудед, оценка и большинство полей адресов.

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "search": "art tours",
    "queryType": "simple",
    "filter": "Category eq 'Resort and Spa'",
    "select": "HotelId,HotelName,Description,Category",
    "count": true
}
```

Ответ для приведенного выше запроса ограничивается только теми гостиницами, которые отнесены к категории "отчет и SPA", включая термины "искусство" или "обзоры". В этом случае существует только одно совпадение.

```json
{
    "@search.score": 2.8576312,
    "HotelId": "31",
    "HotelName": "Santa Fe Stay",
    "Description": "Nestled on six beautifully landscaped acres, located 2 blocks from the Plaza. Unwind at the spa and indulge in art tours on site.",
    "Category": "Resort and Spa"
}
```

## <a name="example-4-filter-functions"></a>Пример 4. функции фильтров

К выражениям фильтра могут относиться [функции "Search. исматчскоринг" и "Search."](search-query-odata-full-text-search-functions.md), позволяющие создавать поисковый запрос в фильтре. Это выражение фильтра использует подстановочный знак *для выбора* удобствами, включая бесплатный WiFi, бесплатную стоянку и т. д.

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30
  {
    "search": "",
    "filter": "search.ismatch('free*', 'Tags', 'full', 'any')",
    "select": "HotelId, HotelName, Category, Description",
    "count": true
  }
```

Ответ для приведенного выше запроса соответствует в 19 гостиницам, предлагающим бесплатную удобствами. Обратите внимание, что оценка поиска — это единое значение "1,0" во всех результатах. Это обусловлено тем, что выражение поиска имеет значение null или является пустым, что приводит к совпадению точных фильтров, но не содержит полнотекстовый поиск. Оценки релевантности возвращаются только при полнотекстовом поиске. Если вы используете фильтры без "поиска", убедитесь, что у вас есть достаточное количество полей для сортировки, чтобы вы могли управлять рангом поиска.

```json
"@odata.count": 19,
"value": [
    {
        "@search.score": 1.0,
        "HotelId": "31",
        "HotelName": "Santa Fe Stay",
        "Tags": [
            "view",
            "restaurant",
            "free parking"
        ]
    },
    {
        "@search.score": 1.0,
        "HotelId": "27",
        "HotelName": "Super Deluxe Inn & Suites",
        "Tags": [
            "bar",
            "free wifi"
        ]
    },
    {
        "@search.score": 1.0,
        "HotelId": "39",
        "HotelName": "Whitefish Lodge & Suites",
        "Tags": [
            "continental breakfast",
            "free parking",
            "free wifi"
        ]
    },
    {
        "@search.score": 1.0,
        "HotelId": "11",
        "HotelName": "Regal Orb Resort & Spa",
        "Tags": [
            "free wifi",
            "restaurant",
            "24-hour front desk service"
        ]
    },
```

## <a name="example-5-range-filters"></a>Пример 5. фильтры диапазона

Фильтрация диапазона поддерживается через выражения фильтров для любого типа данных. В следующих примерах иллюстрируются числовые и строковые диапазоны. Типы данных важны в фильтрах диапазона и лучше всего работают, когда числовые данные находятся в числовых полях, а строковые данные — в строковых. Числовые данные в строковых полях не подходят для диапазонов, так как числовые строки несравнимы.

Следующий запрос является числовым диапазоном. В гостиницах — пример — index — Оценка является единственным фильтруемым числовым полем.

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "search": "*",
    "filter": "Rating ge 2 and Rating lt 4",
    "select": "HotelId, HotelName, Rating",
    "orderby": "Rating desc",
    "count": true
}
```

Ответ для этого запроса должен выглядеть примерно так, как показано в следующем примере, с укороченным для краткости.

```json
"@odata.count": 27,
"value": [
    {
        "@search.score": 1.0,
        "HotelId": "22",
        "HotelName": "Stone Lion Inn",
        "Rating": 3.9
    },
    {
        "@search.score": 1.0,
        "HotelId": "25",
        "HotelName": "Scottish Inn",
        "Rating": 3.8
    },
    {
        "@search.score": 1.0,
        "HotelId": "2",
        "HotelName": "Twin Dome Motel",
        "Rating": 3.6
    }
```

Следующий запрос является фильтром диапазона для строкового поля (Address/StateProvince):

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "search": "*",
    "filter": "Address/StateProvince ge 'A*' and Address/StateProvince lt 'D*'",
    "select": "HotelId, HotelName, Address/StateProvince",
    "count": true
}
```

Ответ для этого запроса должен выглядеть, как в приведенном ниже примере, обрезанном для краткости. В этом примере невозможно выполнить сортировку по StateProvince, так как поле не является атрибутом "Sorted" в определении индекса.

```json
"@odata.count": 9,
"value": [
    {
        "@search.score": 1.0,
        "HotelId": "9",
        "HotelName": "Smile Hotel",
        "Address": {
            "StateProvince": "CA "
        }
    },
    {
        "@search.score": 1.0,
        "HotelId": "39",
        "HotelName": "Whitefish Lodge & Suites",
        "Address": {
            "StateProvince": "CO"
        }
    },
    {
        "@search.score": 1.0,
        "HotelId": "7",
        "HotelName": "Countryside Resort",
        "Address": {
            "StateProvince": "CA "
        }
    },
```

## <a name="example-6-geo-search"></a>Пример 6. Географический поиск

Индекс гостиниц включает в себя geo_location поле с координатами широты и долготы. В этом примере используется [функция geo.distance](search-query-odata-geo-spatial-functions.md#examples), которая фильтрует документы в пределах окружности начальной точки до произвольного расстояния (в километрах), которое вы предоставляете. Вы можете настроить Последнее значение в запросе (10), чтобы уменьшить или увеличить контактную зону запроса.

```http
POST /indexes/v/docs/search?api-version=2020-06-30
{
    "search": "*",
    "filter": "geo.distance(Location, geography'POINT(-122.335114 47.612839)') le 10",
    "select": "HotelId, HotelName, Address/City, Address/StateProvince",
    "count": true
}
```

Ответ для этого запроса возвращает все гостиницы в 10-километрном расстоянии от указанных координат:

```json
{
    "@odata.count": 3,
    "value": [
        {
            "@search.score": 1.0,
            "HotelId": "45",
            "HotelName": "Arcadia Resort & Restaurant",
            "Address": {
                "City": "Seattle",
                "StateProvince": "WA"
            }
        },
        {
            "@search.score": 1.0,
            "HotelId": "24",
            "HotelName": "Gacc Capital",
            "Address": {
                "City": "Seattle",
                "StateProvince": "WA"
            }
        },
        {
            "@search.score": 1.0,
            "HotelId": "16",
            "HotelName": "Double Sanctuary Resort",
            "Address": {
                "City": "Seattle",
                "StateProvince": "WA"
            }
        }
    ]
}
```

## <a name="example-7-booleans-with-searchmode"></a>Пример 7. Логические операторы с параметром searchMode

Простой синтаксис поддерживает логические операторы в виде символов ( `+, -, |` ) для поддержки логики запросов and, OR и not. Логический поиск ведет себя так, как вы можете ожидать, с помощью нескольких значимых исключений. 

В предыдущих примерах параметр "searchMode" появился в качестве механизма для возможного влияния на точность и отзыв, при этом "searchMode = Any" отпользующий отзыв (документ, удовлетворяющий любому критерию, считается совпадением) и "searchMode = ALL" (все условия должны быть сопоставлены в документе). 

В контексте логического поиска по умолчанию «searchMode = Any» может быть запутанным, если вы накладываете запрос с несколькими операторами и получаете более широкие вместо более узких результатов. Это особенно справедливо в случае ОТСУТСТВия, где результаты включают все документы "не содержащие" определенного термина или фразы.

Ниже приведен пример. При выполнении следующего запроса с searchMode (Any) возвращаются документы 42: те, которые содержат термин "Ресторан", а также все документы, у которых нет фразы "кондиционирование воздуха". 

Обратите внимание, что между логическим оператором ( `-` ) и фразой «кондиционирование воздуха» нет пробелов.

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "search": "restaurant -\"air conditioning\"",
    "searchMode": "any",
    "searchFields": "Tags",
    "select": "HotelId, HotelName, Tags",
    "count": true
}
```

Изменение на "searchMode = ALL" обеспечивает совокупное воздействие на критерии и возвращает меньший результирующий набор (7 совпадений), состоящий из документов, содержащих термин "Ресторан", минус те, которые содержат фразу "кондиционирование воздуха".

Ответ для этого запроса теперь будет выглядеть примерно так, как в следующем примере, обрезанном для краткости.

```json
"@odata.count": 7,
"value": [
    {
        "@search.score": 2.5460577,
        "HotelId": "11",
        "HotelName": "Regal Orb Resort & Spa",
        "Tags": [
            "free wifi",
            "restaurant",
            "24-hour front desk service"
        ]
    },
    {
        "@search.score": 2.166792,
        "HotelId": "10",
        "HotelName": "Countryside Hotel",
        "Tags": [
            "24-hour front desk service",
            "coffee in lobby",
            "restaurant"
        ]
    },
```

## <a name="example-8-paging-results"></a>Пример 8. Результаты разбиения на страницы

В предыдущих примерах вы узнали о параметрах, влияющих на композицию результатов поиска, включая "Select", который определяет, какие поля в результатах, порядок сортировки и как включить число совпадений. Этот пример является продолжением композиции результатов поиска в виде параметров подкачки, которые позволяют пакетировать количество результатов, отображаемых на данной странице. 

По умолчанию служба поиска возвращает первые 50 соответствий. Чтобы управлять количеством совпадений на каждой странице, используйте "Top", чтобы определить размер пакета, а затем используйте "пропустить", чтобы выбрать последующие пакеты.

В следующем примере используется фильтр и порядок сортировки для поля рейтинг (оценка является как фильтруемым, так и сортируемый), так как это позволяет увидеть результаты разбиения по страницам отсортированных результатов. В обычном поисковом запросе верхние соответствия ранжированы и размещаются на странице " @search.score ".

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "search": "*",
    "filter": "Rating gt 4",
    "select": "HotelName, Rating",
    "orderby": "Rating desc",
    "top": "5",
    "count": true
}
```

Запрос находит 21 соответствующий документ, но, так как вы указали "Top", ответ возвращает только первые пять совпадений с оценками, начиная с 4,9 и заканчивая 4,7 с "хозяйкой & B". 

Чтобы получить следующие 5 документов, пропустите первый пакет.

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "search": "*",
    "filter": "Rating gt 4",
    "select": "HotelName, Rating",
    "orderby": "Rating desc",
    "top": "5",
    "skip": "5",
    "count": true
}
```

Ответ для второго пакета пропускает первые пять совпадений, возвращая следующие пять, начиная с "Пулл'р Инн Motel". Чтобы продолжить работу с дополнительными пакетами, можно сохранить "Top" в 5, а затем увеличить "Skip" на 5 для каждого нового запроса (Skip = 5, SKIP = 10, Skip = 15 и т. д.).

```json
"value": [
    {
        "@search.score": 1.0,
        "HotelName": "Pull'r Inn Motel",
        "Rating": 4.7
    },
    {
        "@search.score": 1.0,
        "HotelName": "Sublime Cliff Hotel",
        "Rating": 4.6
    },
    {
        "@search.score": 1.0,
        "HotelName": "Antiquity Hotel",
        "Rating": 4.5
    },
    {
        "@search.score": 1.0,
        "HotelName": "Nordick's Motel",
        "Rating": 4.5
    },
    {
        "@search.score": 1.0,
        "HotelName": "Winter Panorama Resort",
        "Rating": 4.5
    }
]
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда у вас есть опыт использования базового синтаксиса запросов, попробуйте указать запросы в коде. Следующие ссылки содержат сведения о настройке поисковых запросов с помощью пакетов SDK для Azure.

+ [Запрос индекса с помощью пакета SDK для .NET](search-get-started-dotnet.md)
+ [Запрос индекса с помощью пакета SDK для Python](search-get-started-python.md)
+ [Запрос индекса с помощью пакета SDK для JavaScript](search-get-started-javascript.md)

Дополнительные справочные материалы по синтаксису, архитектуре запросов и примеры можно найти по следующим ссылкам:

+ [Примеры синтаксиса запросов Lucene для создания расширенных запросов](search-query-lucene-examples.md)
+ [How full text search works in Azure Cognitive Search](search-lucene-query-architecture.md) (Как выполняется полнотекстовый поиск в Когнитивном поиске Azure)
+ [Синтаксис простых запросов](query-simple-syntax.md)
+ [Lucene query syntax in Azure Search](query-lucene-syntax.md) (Синтаксис запросов Lucene в службе поиска Azure)
+ [Синтаксис фильтра](search-query-odata-filter.md)