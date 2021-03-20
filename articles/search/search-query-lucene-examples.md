---
title: Использовать полный синтаксис запроса Lucene
titleSuffix: Azure Cognitive Search
description: Примеры запросов, демонстрирующие синтаксис запросов Lucene для поиска нечетких результатов, поиска с учетом расположения, повышения терминов, поиска регулярных выражений и поиска с использованием подстановочных знаков в индексе Azure Когнитивный поиск.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: 6213efb6ba14052c6f957a6d999f48f55f65186c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101693566"
---
# <a name="use-the-full-lucene-search-syntax-advanced-queries-in-azure-cognitive-search"></a>Использование полного синтаксиса поиска Lucene (расширенные запросы в Azure Когнитивный поиск)

При создании запросов для Когнитивный поиск Azure можно заменить [простой анализатор запросов](query-simple-syntax.md) по умолчанию на более мощный [анализатор запросов Lucene](query-lucene-syntax.md) , чтобы сформулировать специализированные и расширенные выражения запросов.

Средство синтаксического анализа Lucene поддерживает сложные форматы запросов, такие как запросы с областью видимости полей, Поиск нечетких запросов, инфиксные и подстановочный знак суффикса, поиск по сходству, повышение термина и поиск регулярных выражений. Дополнительные возможности поставляются с дополнительными требованиями к обработке, поэтому следует предусмотреть немного больше времени для выполнения. В этой статье вы можете пошагово пройти примеры, демонстрирующие операции запросов на основе полного синтаксиса.

> [!Note]
> Многие специализированные конструкции запросов, обеспечиваемые благодаря полному синтаксису запросов Lucene, не поддерживают [анализ текста](search-lucene-query-architecture.md#stage-2-lexical-analysis), что может оказаться неожиданным, если вы хотите использовать выделение корней слов или лемматизацию. Лексический анализ выполняется только для полными терминами (запрос термина или запрос фразы). Типы запросов с неполными терминами (запрос с префиксом, запрос с подстановочными знаками, запрос с регулярными выражениями, нечеткий запрос) добавляются непосредственно к дереву запроса в обход этапа анализа. Единственным преобразованием, выполняемым в условиях частичного запроса, является нижний регистр. 
>

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
    "queryType": "full",
    "select": "HotelId, HotelName, Category, Tags, Description",
    "count": true
}
```

+ для параметра "Search" задан `*` неопределенный запрос, эквивалентный null или пустому поиску. Это не особенно полезно, но это самый простой поиск, который можно сделать, и Показать все доступные для получения поля в индексе со всеми значениями.

+ для параметра "queryType" задано значение Full, вызывается полное средство синтаксического анализа запросов Lucene, которое требуется для этого синтаксиса.

+ для композиции результатов поиска используется значение "Select", равное разделенному запятыми списку полей, включая только те поля, которые полезны в контексте результатов поиска.

+ "Count" возвращает количество документов, соответствующих условиям поиска. В пустой строке поиска счетчик будет содержать все документы в индексе (50 в случае использования гостиниц-Sample-index).

## <a name="example-1-fielded-search"></a>Пример 1. Поиск по полям

Область поиска по полям отдельные внедренные выражения поиска для определенного поля. В этом примере выполняется поиск названий отеля с термином «Гостиницы», но не «Motel». Можно указать несколько полей с помощью и. 

При использовании этого синтаксиса запроса параметр "searchFields" можно опустить, если поля, к которым необходимо выполнить запрос, находятся в самом выражении поиска. Если включить «searchFields» с поиском по полям, то `fieldName:searchExpression` всегда имеет приоритет над «searchFields».

```http
POST /indexes/hotel-samples-index/docs/search?api-version=2020-06-30
{
    "search": "HotelName:(hotel NOT motel) AND Category:'Resort and Spa'",
    "queryType": "full",
    "select": "HotelName, Category",
    "count": true
}
```

Ответ для этого запроса должен выглядеть, как в следующем примере, с фильтрацией по "курорту и SPA", возвращая Гостиницы, которые включают в себя "Гостиницы" или "Motel".

```json
"@odata.count": 4,
"value": [
    {
        "@search.score": 4.481559,
        "HotelName": "Nova Hotel & Spa",
        "Category": "Resort and Spa"
    },
    {
        "@search.score": 2.4524608,
        "HotelName": "King's Palace Hotel",
        "Category": "Resort and Spa"
    },
    {
        "@search.score": 2.3970203,
        "HotelName": "Triple Landscape Hotel",
        "Category": "Resort and Spa"
    },
    {
        "@search.score": 2.2953436,
        "HotelName": "Peaceful Market Hotel & Spa",
        "Category": "Resort and Spa"
    }
]
```

Выражение поиска может представлять собой один термин или фразу или более сложное выражение в круглых скобках, при необходимости с логическими операторами. Вот несколько примеров.

+ `HotelName:(hotel NOT motel)`
+ `Address/StateProvince:("WA" OR "CA")`
+ `Tags:("free wifi" NOT "free parking") AND "coffee in lobby"`

Обязательно заключите фразу в кавычки, если хотите, чтобы обе строки вычислить как единую сущность, как в этом случае поиск двух различных расположений в поле Address/StateProvince. В зависимости от клиента может потребоваться экранирование ( `\` ) кавычек.

Поле, указанное в `fieldName:searchExpression` , должно быть полем с возможностью поиска. Дополнительные сведения об атрибутах определений полей см. в разделе [CREATE INDEX (REST API)](/rest/api/searchservice/create-index) .

## <a name="example-2-fuzzy-search"></a>Пример 2. нечеткий поиск

Поиск нечетких совпадений по похожим терминам, включая слова с ошибками. Чтобы выполнить поиск нечетких соответствий, необходимо добавить символ тильды `~` в конце слова. Дополнительно можно поставить цифру от 0 до 2, указывающую расстояние редактирования. Например, `blue~` или `blue~1` вернет результаты с blue, blues и glue.

```http
POST /indexes/hotel-samples-index/docs/search?api-version=2020-06-30
{
    "search": "Tags:conserge~",
    "queryType": "full",
    "select": "HotelName, Category, Tags",
    "searchFields": "HotelName, Category, Tags",
    "count": true
}
```

Ответ для этого запроса разрешается в "Concierge" в соответствующих документах, обрезанных для краткости:

```json
"@odata.count": 12,
"value": [
    {
        "@search.score": 1.1832147,
        "HotelName": "Secret Point Motel",
        "Category": "Boutique",
        "Tags": [
            "pool",
            "air conditioning",
            "concierge"
        ]
    },
    {
        "@search.score": 1.1819803,
        "HotelName": "Twin Dome Motel",
        "Category": "Boutique",
        "Tags": [
            "pool",
            "free wifi",
            "concierge"
        ]
    },
    {
        "@search.score": 1.1773309,
        "HotelName": "Smile Hotel",
        "Category": "Suite",
        "Tags": [
            "view",
            "concierge",
            "laundry service"
        ]
    },
```

Фразы не поддерживаются напрямую, но можно указать нечеткое соответствие для каждого термина многокомпонентной фразы, например `search=Tags:landy~ AND sevic~` .  Это выражение запроса находит 15 совпадений в "белье Service".

> [!Note]
> Запросы с нечетким соответствием не [анализируются](search-lucene-query-architecture.md#stage-2-lexical-analysis). Типы запросов с неполными терминами (запрос с префиксом, запрос с подстановочными знаками, запрос с регулярными выражениями, нечеткий запрос) добавляются непосредственно к дереву запроса в обход этапа анализа. Единственное преобразование, выполняемое в условиях частичного запроса, имеет более низкий регистр.
>

## <a name="example-3-proximity-search"></a>Пример 3. Поиск по сходству

Поиск по сходству находит термины, близкие друг к другу в документе. Вставьте символ тильды "~" в конце фразы, а затем — цифру, обозначающую количество слов, определяющее границу близости.

Этот запрос выполняет поиск терминов "Гостиница" и "Аэропорт" в нескольких пяти словах в документе. Кавычки экранированы ( `\"` ) для сохранения фразы:

```http
POST /indexes/hotel-samples-index/docs/search?api-version=2020-06-30
{
    "search": "Description: \"hotel airport\"~5",
    "queryType": "full",
    "select": "HotelName, Description",
    "searchFields": "HotelName, Description",
    "count": true
}
```

Ответ для этого запроса должен выглядеть следующим образом:

```json
"@odata.count": 2,
"value": [
    {
        "@search.score": 0.6331726,
        "HotelName": "Trails End Motel",
        "Description": "Only 8 miles from Downtown.  On-site bar/restaurant, Free hot breakfast buffet, Free wireless internet, All non-smoking hotel. Only 15 miles from airport."
    },
    {
        "@search.score": 0.43032226,
        "HotelName": "Catfish Creek Fishing Cabins",
        "Description": "Brand new mattresses and pillows.  Free airport shuttle. Great hotel for your business needs. Comp WIFI, atrium lounge & restaurant, 1 mile from light rail."
    }
]
```

## <a name="example-4-term-boosting"></a>Пример 4. повышение термина

При повышении приоритета слов документы сортируются по приоритету, т. е. документы, в которых содержится условие поиска, имеют высший приоритет по отношению к документам, в которых его нет. Чтобы увеличить термин, используйте знак крышки, `^` символ с коэффициентом увеличения (число) в конце искомого термина. Коэффициент увеличения по умолчанию равен 1, и хотя он должен быть положительным, он может быть меньше 1 (например, 0,2). Повышение приоритета терминов отличается от профилей повышения, так как они повышают приоритет определенных полей, а не определенных терминов.

В этом запросе выполните поиск по запросу «пляже Access» и обратите внимание, что существует семь документов, соответствующих одному или обоим терминам.

```http
POST /indexes/hotel-samples-index/docs/search?api-version=2020-06-30
{
    "search": "beach access",
    "queryType": "full",
    "select": "HotelName, Description, Tags",
    "searchFields": "HotelName, Description, Tags",
    "count": true
}
```

На самом деле, существует только один документ, совпадающий с "доступом", и поскольку он является единственным, его размещение велико (вторая позиция), даже если в документе отсутствует термин "пляже".

```json
"@odata.count": 7,
"value": [
    {
        "@search.score": 2.2723424,
        "HotelName": "Nova Hotel & Spa",
        "Description": "1 Mile from the airport.  Free WiFi, Outdoor Pool, Complimentary Airport Shuttle, 6 miles from the beach & 10 miles from downtown."
    },
    {
        "@search.score": 1.5507699,
        "HotelName": "Old Carrabelle Hotel",
        "Description": "Spacious rooms, glamorous suites and residences, rooftop pool, walking access to shopping, dining, entertainment and the city center."
    },
    {
        "@search.score": 1.5358944,
        "HotelName": "Whitefish Lodge & Suites",
        "Description": "Located on in the heart of the forest. Enjoy Warm Weather, Beach Club Services, Natural Hot Springs, Airport Shuttle."
    },
    {
        "@search.score": 1.3433652,
        "HotelName": "Ocean Air Motel",
        "Description": "Oceanfront hotel overlooking the beach features rooms with a private balcony and 2 indoor and outdoor pools. Various shops and art entertainment are on the boardwalk, just steps away."
    },
```

В запросе "после" повторите поиск, на этот раз Повысьте результаты с термином "пляже" в термине "доступ". Понятная для человека версия запроса — `search=Description:beach^2 access` . В зависимости от клиента может потребоваться выразить его `^2` как `%5E2` .

После увеличения термина "пляже" совпадение со старым Каррабелле отеля перемещается в шестое место.

<!-- Consider a scoring profile that boosts matches in a certain field, such as "genre" in a music app. Term boosting could be used to further boost certain search terms higher than others. For example, "rock^2 electronic" will boost documents that contain the search terms in the "genre" field higher than other searchable fields in the index. Furthermore, documents that contain the search term "rock" will be ranked higher than the other search term "electronic" as a result of the term boost value (2). -->

## <a name="example-5-regex"></a>Пример 5. регулярное выражение

Операция поиска по регулярным выражениям позволяет найти совпадение в зависимости от содержимого между косыми чертами "/", как указано в документации [класса RegExp](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/util/automaton/RegExp.html).

```http
POST /indexes/hotel-samples-index/docs/search?api-version=2020-06-30
{
    "search": "HotelName:/(Mo|Ho)tel/",
    "queryType": "full",
    "select": "HotelName",
    "count": true
}
```

Ответ для этого запроса должен выглядеть следующим образом:

```json
    "@odata.count": 22,
    "value": [
        {
            "@search.score": 1.0,
            "HotelName": "Days Hotel"
        },
        {
            "@search.score": 1.0,
            "HotelName": "Triple Landscape Hotel"
        },
        {
            "@search.score": 1.0,
            "HotelName": "Smile Hotel"
        },
        {
            "@search.score": 1.0,
            "HotelName": "Pelham Hotel"
        },
        {
            "@search.score": 1.0,
            "HotelName": "Sublime Cliff Hotel"
        },
        {
            "@search.score": 1.0,
            "HotelName": "Twin Dome Motel"
        },
        {
            "@search.score": 1.0,
            "HotelName": "Nova Hotel & Spa"
        },
        {
            "@search.score": 1.0,
            "HotelName": "Scarlet Harbor Hotel"
        },
```

> [!Note]
> Запросы с регулярными выражениями не [анализируются](./search-lucene-query-architecture.md#stage-2-lexical-analysis). Единственное преобразование, выполняемое в условиях частичного запроса, имеет более низкий регистр.
>

## <a name="example-6-wildcard-search"></a>Пример 6. Поиск с подстановочными знаками

В целом можно использовать распознаваемый синтаксис для `*` поиска с подстановочными знаками в нескольких () или одиночных ( `?` ) символах. Обратите внимание, что средство синтаксического анализа запросов Lucene поддерживает использование этих символов для поиска одного слова, а не фразы.

В этом запросе найдите имена Гостиницы, содержащие префикс "SC". Нельзя использовать `*` символ или в `?` качестве первого символа поиска.

```http
POST /indexes/hotel-samples-index/docs/search?api-version=2020-06-30
{
    "search": "HotelName:sc*",
    "queryType": "full",
    "select": "HotelName",
    "count": true
}
```

Ответ для этого запроса должен выглядеть следующим образом:

```json
    "@odata.count": 2,
    "value": [
        {
            "@search.score": 1.0,
            "HotelName": "Scarlet Harbor Hotel"
        },
        {
            "@search.score": 1.0,
            "HotelName": "Scottish Inn"
        }
    ]
```

> [!Note]
> Запросы с подстановочными знаками не [анализируются](./search-lucene-query-architecture.md#stage-2-lexical-analysis). Единственное преобразование, выполняемое в условиях частичного запроса, имеет более низкий регистр.
>

## <a name="next-steps"></a>Дальнейшие действия

Попробуйте указать запросы в коде. Следующие ссылки содержат сведения о настройке поисковых запросов с помощью пакетов SDK для Azure.

+ [Запрос индекса с помощью пакета SDK для .NET](search-get-started-dotnet.md)
+ [Запрос индекса с помощью пакета SDK для Python](search-get-started-python.md)
+ [Запрос индекса с помощью пакета SDK для JavaScript](search-get-started-javascript.md)

Дополнительные справочные материалы по синтаксису, архитектуре запросов и примеры можно найти по следующим ссылкам:

+ [Примеры синтаксиса запросов Lucene для создания расширенных запросов](search-query-lucene-examples.md)
+ [How full text search works in Azure Cognitive Search](search-lucene-query-architecture.md) (Как выполняется полнотекстовый поиск в Когнитивном поиске Azure)
+ [Синтаксис простых запросов](query-simple-syntax.md)
+ [Lucene query syntax in Azure Search](query-lucene-syntax.md) (Синтаксис запросов Lucene в службе поиска Azure)
+ [Синтаксис фильтра](search-query-odata-filter.md)