---
title: Многоязыковая индексация для поисковых запросов, отличных от английского
titleSuffix: Azure Cognitive Search
description: Создайте индекс, который поддерживает многоязычное содержимое, а затем создайте запросы, областью которых является это содержимое.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/22/2021
ms.openlocfilehash: 627ec77af4e492b4f22404972729cecdb1c40f06
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104801610"
---
# <a name="how-to-create-an-index-for-multiple-languages-in-azure-cognitive-search"></a>Создание индекса для нескольких языков в Azure Когнитивный поиск

Основным требованием к многоязычному приложению поиска является возможность выполнять поиск и извлекать результаты на языке пользователя. В Когнитивный поиск Azure одним из способов удовлетворения языковых требований многоязыкового приложения является создание выделенных полей для хранения строк на определенном языке, а затем ограничение полнотекстового поиска только этими полями во время запроса.

+ В определениях полей задайте языковой анализатор, который вызывает лингвистические правила целевого языка. Полный список поддерживаемых анализаторов см. в разделе [Добавление анализаторов языка](index-add-language-analyzers.md).

+ В запросе запроса задайте для параметров значение область полнотекстового поиска для конкретных полей, а затем обрежьте результаты всех полей, не предоставляющих содержимое, совместимое с поисковым интерфейсом, который вы хотите доставить.

Успех этого метода зависит от целостности содержимого поля. Когнитивный поиск Azure не преобразует строки или не выполняет определение языка в ходе выполнения запроса. Вы должны убедиться, что поля содержат ожидаемые строки.

## <a name="define-fields-for-content-in-different-languages"></a>Определение полей для содержимого на разных языках

В Когнитивный поиск Azure запросы предназначены для одного индекса. Разработчики, желающие предоставить языковые строки в одном поисковом интерфейсе, обычно задают выделенные поля для хранения значений: одно поле для строк на английском, одно для строк на французском и т. д.

Свойство "Analyzer" в определении поля используется для установки [анализатора языка](index-add-language-analyzers.md). Он будет использоваться как для индексирования, так и для выполнения запросов.

```JSON
{
  "name": "hotels-sample-index",
  "fields": [
    {
      "name": "Description",
      "type": "Edm.String",
      "retrievable": true,
      "searchable": true,
      "analyzer": "en.microsoft"
    },
    {
      "name": "Description_fr",
      "type": "Edm.String",
      "retrievable": true,
      "searchable": true,
      "analyzer": "fr.microsoft"
    },
```

## <a name="build-and-load-an-index"></a>Построение и загрузка индекса

Промежуточным (и, возможно, очевидным) шагом является [сборка и заполнение индекса](search-get-started-dotnet.md) перед формулировкой запроса. Здесь мы упомянем этот шаг для полноты картины. Одним из способов определения доступности индекса является проверка списка индексов на [портале](https://portal.azure.com).

> [!TIP]
> Обнаружение языка и преобразование текста поддерживаются во время приема данных с помощью обогащения и [навыкови](cognitive-search-working-with-skillsets.md) [искусственного интеллекта](cognitive-search-concept-intro.md) . Если у вас есть источник данных Azure с содержимым на разных языках, вы можете испытать функции определения языка и перевода с помощью [мастера импорта данных](cognitive-search-quickstart-blob.md).

## <a name="constrain-the-query-and-trim-results"></a>Ограничение запроса и усечение результатов

Параметры запроса используются для ограничения поиска конкретных полей, а затем усечения результатов всех полей, которые не полезны для сценария. 

| Параметры | Цель |
|-----------|--------------|
| **searchFields** | Ограничивает полнотекстовый поиск до списка именованных полей. |
| **$select** | Обрезает ответ, включая только указанные поля. По умолчанию возвращаются все извлекаемые поля. Параметр **$Select** позволяет выбрать, какие из них следует вернуть. |

Учитывая цель ограничения поиска по полям, содержащим французские строки, нужно использовать **searchFields** для целевого запроса в полях, содержащих строки на этом языке.

Указывать анализатор для запроса запроса не обязательно. Анализатор языка для определения поля всегда будет использоваться во время обработки запроса. Для запросов, которые задают несколько полей для вызова различных языковых анализаторов, термины или фразы будут обрабатываться независимо от назначенных анализаторов для каждого поля.

По умолчанию поиск возвращает все поля, которые помечены как извлекаемые. Таким образом может потребоваться исключить поля, которые не соответствуют языковому интерфейсу поиска, который необходимо предоставить. В частности при ограничении поиска до полей со строками на французском языке, может потребоваться исключить из результатов поля со строками на английском языке. С помощью параметра запроса **$select** можно контролировать какие поля возвращаются вызывающему приложению.

#### <a name="example-in-rest"></a>Пример в ОСТАВШЕЙся

```http
POST https://[service name].search.windows.net/indexes/hotels-sample-index/docs/search?api-version=2020-06-30
{
    "search": "animaux acceptés",
    "searchFields": "Tags, Description_fr",
    "select": "HotelName, Description_fr, Address/City, Address/StateProvince, Tags",
    "count": "true"
}
```

#### <a name="example-in-c"></a>Пример в C #

```csharp
private static void RunQueries(SearchClient srchclient)
{
    SearchOptions options;
    SearchResults<Hotel> response;

    options = new SearchOptions()
    {
        IncludeTotalCount = true,
        Filter = "",
        OrderBy = { "" }
    };

    options.Select.Add("HotelId");
    options.Select.Add("HotelName");
    options.Select.Add("Description_fr");
    options.SearchFields.Add("Tags");
    options.SearchFields.Add("Description_fr");

    response = srchclient.Search<Hotel>("*", options);
    WriteDocuments(response);
}
```

## <a name="boost-language-specific-fields"></a>Повышение уровня полей для конкретного языка

Если язык агента, отправившего запрос, неизвестен, запрос можно применить ко всем полям одновременно. Предпочтения IA для результатов на определенном языке можно определить с помощью [профилей оценки](index-add-scoring-profiles.md). В приведенном ниже примере совпадения, найденные в описании на английском языке, будут оцениваться выше относительно совпадений на других языках:

```JSON
  "scoringProfiles": [
    {
      "name": "englishFirst",
      "text": {
        "weights": { "description": 2 }
      }
    }
  ]
```

Затем вы включите профиль оценки в поисковый запрос:

```http
POST /indexes/hotels/docs/search?api-version=2020-06-30
{
  "search": "pets allowed",
  "searchFields": "Tags, Description",
  "select": "HotelName, Tags, Description",
  "scoringProfile": "englishFirst",
  "count": "true"
}
```

## <a name="next-steps"></a>Следующие шаги

+ [Языковые анализаторы](index-add-language-analyzers.md)
+ [How full text search works in Azure Cognitive Search](search-lucene-query-architecture.md) (Как выполняется полнотекстовый поиск в Когнитивном поиске Azure)
+ [Search Documents (Azure Search Service REST API)](/rest/api/searchservice/search-documents) (Поиск по документам (REST API службы поиска Azure))
+ [Обзор обогащения искусственного интеллекта](cognitive-search-concept-intro.md)
+ [Обзор навыков](cognitive-search-working-with-skillsets.md)