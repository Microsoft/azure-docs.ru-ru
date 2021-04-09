---
title: Поиск сущностей с помощью пакета API Поиска сущностей Bing
titleSuffix: Azure Cognitive Services
description: С помощью API Bing для поиска сущностей вы можете извлекать и искать сущности и места на основе поисковых запросов.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: conceptual
ms.date: 02/01/2019
ms.author: aahi
ms.openlocfilehash: 9dabceda17defb24f2a916cd641f625feb551c6a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "96353295"
---
# <a name="searching-for-entities-with-the-bing-entity-api"></a>Поиск сущностей с помощью API Bing для сущностей

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

## <a name="suggest-search-terms-with-the-bing-autosuggest-api"></a>Предложение условий поиска с помощью API автозаполнения Bing

Если вы предоставили окно поиска, в котором пользователь вводит свой поисковый запрос, используйте [API автозаполнения Bing](../../bing-autosuggest/get-suggested-search-terms.md), чтобы оптимизировать работу. API возвращает предложенные строки запроса на основе частичного поиска, как пользовательские типы.

После ввода условия поиска URL-адрес закодирует его перед установкой параметра запроса [q](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#query). Например, если пользователь вводит *Marcus Appel*, задайте `q` как *Marcus+Appel* или *Marcus%20Appel*.

Если условие поиска содержит орфографические ошибки, ответ поиска включает объект [QueryContext](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#querycontext). Объект показывает исходное написание и исправленное написание, которое Bing использует для поиска.

```json
"queryContext": {
    "originalQuery": "hollo wrld",
    "alteredQuery": "hello world",
    "alterationOverrideQuery": "+hollo wrld",
    "adultIntent": false
}
```

## <a name="the-bing-entity-search-api-response"></a>Ответ API Поиска сущностей Bing

Ответ API содержит объект [SearchResponse](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#searchresponse). Обнаружив релевантные сущности или места, объект включает поле `entities` или `places`, или оба. В противном случае объект ответа не включает какое-либо поле.
> [!NOTE]
> Ответы сущности поддерживают разные рынки, но ответ Places поддерживает только расположения организаций в США. 

Поле `entities` является объектом [EntityAnswer](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference), который содержит список объектов [Сущность](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entity) (см. поле `value`). Список может содержать одну главную сущность, несколько объектов, несколько уточняющих сущностей, или и то, и другое. 

Главная сущность возвращается, если Bing считает ее единственной, соответствующей запросу (не возникло неопределенности о том, какие сущности соответствуют запросу). Если несколько объектов могут удовлетворять запросу, список содержит несколько объектов для устранения неоднозначности. Например, если запрос использует универсальный заголовок серии фильмов, список, вероятно, будет содержать уточнение сущностей. Но, если в запросе указывается конкретный заголовок из серии фильмов, список, вероятно, будет содержать одну главную сущность.

Сюда входят известные личности, такие как певцы, актеры, спортсмены, модели и т. д; места и достопримечательности, такие как гора Рейнир или Мемориал Линкольна; и такие вещи, как бананы, золотые украшения, книги или название фильма. Поле [EntityPresentationInfo](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entitypresentationinfo) содержит указания, которые определяют тип сущности. Например, если это лицо, фильм, животное или достопримечательность. Список возможных типов см. в разделе [Entity Types](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entity-types) (Типы сущностей).

```json
"entityPresentationInfo": {
    "entityScenario": "DominantEntity",
    "entityTypeHints": ["Attraction"],
    "entityTypeDisplayHint": "Mountain"
}, ...
```

Ниже показан ответ, который включает главную и уточняющую сущность.

```json
{
    "_type": "SearchResponse",
    "queryContext": {
        "originalQuery": "Mount Rainier"
    },
    "entities": {
        "value": [{
            "contractualRules": [{
                "_type": "ContractualRules/LicenseAttribution",
                "targetPropertyName": "description",
                "mustBeCloseToContent": true,
                "license": {
                    "name": "CC-BY-SA",
                    "url": "https://creativecommons.org/licenses/by-sa/3.0/"
                },
                "licenseNotice": "Text under CC-BY-SA license"
            },
            {
                "_type": "ContractualRules/LinkAttribution",
                "targetPropertyName": "description",
                "mustBeCloseToContent": true,
                "text": "contoso.com",
                "url": "http://contoso.com/mount_rainier"
            },
            {
                "_type": "ContractualRules/MediaAttribution",
                "targetPropertyName": "image",
                "mustBeCloseToContent": true,
                "url": "http://contoso.com/mount-rainier"
            }],
            "webSearchUrl": "https://www.bing.com/search?q=Mount%20Rainier...",
            "name": "Mount Rainier",
            "url": "http://www.northwindtraders.com/",
            "image": {
                "name": "Mount Rainier",
                "thumbnailUrl": "https://www.bing.com/th?id=A4ae343983daa4...",
                "provider": [{
                    "_type": "Organization",
                    "url": "http://contoso.com/mount_rainier"
                }],
                "hostPageUrl": "http://contoso.com/commons/7/72/mount_rain...",
                "width": 110,
                "height": 110
            },
            "description": "Mount Rainier is 14,411 ft tall and the highest mountain...",
            "entityPresentationInfo": {
                "entityScenario": "DominantEntity",
                "entityTypeHints": ["Attraction"]
            },
            "bingId": "38b9431e-cf91-93be-0584-c42a3ecbfdc7"
        },
        {
            "contractualRules": [{
                "_type": "ContractualRules/MediaAttribution",
                "targetPropertyName": "image",
                "mustBeCloseToContent": true,
                "url": "http://contoso.com/mount_rainier_national_park"
            }],
            "webSearchUrl": "https://www.bing.com/search?q=Mount%20Rainier%20National...",
            "name": "Mount Rainier National Park",
            "url": "http://worldwideimporters.com/",
            "image": {
                "name": "Mount Rainier National Park",
                "thumbnailUrl": "https://www.bing.com/th?id=A91bdc5a1b648a695a39...",
                "provider": [{
                    "_type": "Organization",
                    "url": "http://contoso.com/mount_rainier_national_park"
                }],
                "hostPageUrl": "http://contoso.com/en/7/7a...",
                "width": 50,
                "height": 50
            },
            "description": "Mount Rainier National Park is a United States National Park...",
            "entityPresentationInfo": {
                "entityScenario": "DisambiguationItem",
                "entityTypeHints": ["Organization"]
            },
            "bingId": "29d4b681-227a-3924-7bb1-8a54e8666b8c"
        }]
    }
}
```

В сущность включены поля `name`, `description` и `image`. Когда вы показываете эти поля в своем пользовательском интерфейсе, им нужно присвоить атрибуты. Поле `contractualRules` содержит список атрибутов, которые необходимо применить. Правило контракта определяет поле, к которому применяется атрибут. Сведения о применении определения принадлежности см. в [этом разделе](#data-attribution).

```json
"contractualRules": [{
    "_type": "ContractualRules/LicenseAttribution",
    "targetPropertyName": "description",
    "mustBeCloseToContent": true,
    "license": {
        "name": "CC-BY-SA",
        "url": "https://creativecommons.org/licenses/by-sa/3.0/"
    },
    "licenseNotice": "Text under CC-BY-SA license"
},
{
    "_type": "ContractualRules/LinkAttribution",
    "targetPropertyName": "description",
    "mustBeCloseToContent": true,
    "text": "contoso.com",
    "url": "http://contoso.com/wiki/Mount_Rainier"
},
{
    "_type": "ContractualRules/MediaAttribution",
    "targetPropertyName": "image",
    "mustBeCloseToContent": true,
    "url": "http://contoso.com/wiki/Mount_Rainier"
}], ...
```

При отображении информации о сущности (имя, описание и изображение) нужно использовать URL-адрес в поле `webSearchUrl` для ссылки на страницу результатов поиска Bing, содержащую объект.

## <a name="find-places"></a>Поиск мест

Поле `places` является объектом [LocalEntityAnswer](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference), который содержит список объектов [Место](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#place) (дополнительные сведения см. в разделе [Типы сущностей](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entity-types)). Список содержит одну или несколько локальных сущностей, которые соответствуют запросу.

Места включают рестораны, гостиницы или локальные организации. Поле [EntityPresentationInfo](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entitypresentationinfo) содержит указания, которые определяют тип локальной сущности. Список содержит перечисленные указания, например место, локальные организации, ресторан. Каждое последующее указание в массиве сужает тип сущности. Список возможных типов см. в разделе [Entity Types](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#entity-types) (Типы сущностей).

```json
"entityPresentationInfo": {
    "entityScenario": "ListItem",
    "entityTypeHints": ["Place",
    "LocalBusiness",
    "Restaurant"]
}, ...
```
> [!NOTE]
> Ответы сущности поддерживают разные рынки, но ответ Places поддерживает только расположения организаций в США. 

Локальные запросы на объекты, такие как *ресторан рядом со мной*, требуют расположение пользователя, чтобы давать точные результаты. Запросы всегда должны использовать заголовки X-Search-Location и X-MSEdge-ClientIP для указания расположения пользователя. Если Bing считает, что запрос из расположения пользователя будет полезен, он устанавливает полю `askUserForLocation`[QueryContext](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#querycontext) значение **true**. 

```json
{
    "_type": "SearchResponse",
    "queryContext": {
        "originalQuery": "Sinful Bakery and Cafe",
        "askUserForLocation": true
    },
    ...
}
```

Результат запроса места включает имя расположения, адрес, номер телефона и URL-адрес для веб-сайта сущности. При отображении информации о сущности нужно использовать URL-адрес в поле `webSearchUrl` для ссылки на страницу результатов поиска Bing, содержащую объект.

```json
"places": {
    "value": [{
        "_type": "Restaurant",
        "webSearchUrl": "https://www.bing.com/search?q=Sinful%20Bakery...",
        "name": "Liberty's Delightful Sinful Bakery & Cafe",
        "url": "http://libertysdelightfulsinfulbakeryandcafe.com/",
        "entityPresentationInfo": {
            "entityScenario": "ListItem",
            "entityTypeHints": ["Place",
            "LocalBusiness",
            "Restaurant"]
        },
        "address": {
            "addressLocality": "Seattle",
            "addressRegion": "WA",
            "postalCode": "98112",
            "addressCountry": "US",
            "neighborhood": "Madison Park"
        },
        "telephone": "(800) 555-1212"
    }]
}
```

> [!NOTE]
> Вы или третья сторона, действующая от вашего имени, можете не использовать, хранить, кэшировать, распределять данные API сущностей URL-адресов или предоставлять доступ к ним с целью тестирования, разработки, обучения, распределения или предоставления доступа любой сторонней службе или функции (не от корпорации Майкрософт).  

## <a name="data-attribution"></a>Определение принадлежности данных

Ответы API сущности Bing в проекте содержат сведения, принадлежащие третьим лицам. Вы несете ответственность за их должное использование, например за соблюдение условий лицензирования Creative Commons в пользовательском интерфейсе.

Если ответ или результат содержит поля `contractualRules`, `attributions` или `provider`, вы должны определить принадлежность данных. Если эти поля отсутствуют, это делать не нужно. Когда ответ содержит поле `contractualRules` и поле `attributions`, и (или) `provider`, принадлежность данных следует определить с помощью договорных правил.

В приведенном ниже примере показана сущность, содержащая договорное правило MediaAttribution и раздел image с полем `provider`. Правило MediaAttribution определяет раздел image как целевой объект, поэтому вы должны игнорировать поле `provider` этого раздела и вместо этого определить принадлежность данных с помощью правила MediaAttribution.  

```json
"value": [{
    "contractualRules": [
        ...
        {
            "_type": "ContractualRules/MediaAttribution",
            "targetPropertyName": "image",
            "mustBeCloseToContent": true,
            "url": "http://contoso.com/mount_rainier"
        }
    ],
    ...
    "image": {
        "name": "Mount Rainier",
        "thumbnailUrl": "https://www.bing.com/th?id=A46378861201...",
        "provider": [{
            "_type": "Organization",
            "url": "http://contoso.com/mount_rainier"
        }],
        "hostPageUrl": "http://www.graphicdesigninstitute.com/Uploaded...",
        "width": 110,
        "height": 110
    },
    ...
}]
```

Если договорное правило содержит поле `targetPropertyName`, это правило применяется только к целевому полю. В противном случае правило применяется к родительскому объекту, который содержит поле `contractualRules`.

В приведенном ниже примере правило `LinkAttribution` содержит поле `targetPropertyName`, поэтому это правило применяется к полю `description`. Для правил, применяемых к определенным полям, сразу после целевых данных необходимо включить строку, содержащую гиперссылку на веб-сайт поставщика. Например, чтобы определить принадлежность текста описания, сразу после него включите строку с гиперссылкой на веб-сайта поставщика. В этом случае создайте ссылку на сайт contoso.com.

```json
"entities": {
    "value": [{
            ...
            "description": "Marcus Appel is a former American....",
            ...
            "contractualRules": [{
                    "_type": "ContractualRules/LinkAttribution",
                    "targetPropertyName": "description",
                    "mustBeCloseToContent": true,
                    "text": "contoso.com",
                    "url": "http://contoso.com/cr?IG=B8AD73..."
                 },
            ...
  
```

### <a name="license-attribution"></a>Определение принадлежности лицензии

Если список договорных правил содержит правило [LicenseAttribution](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#licenseattribution), сразу после содержимого, к которому применяется лицензия, необходимо добавить строку уведомления. Правило `LicenseAttribution` определяет свойство, к которому применяется лицензия, с помощью поля `targetPropertyName`.

Ниже приведен пример с использованием правила `LicenseAttribution`.

![Определение принадлежности лицензии](../media/cognitive-services-bing-entities-api/licenseattribution.png)

Уведомление о лицензии должно содержать гиперссылку на сайт со сведениями о лицензии. Как правило, имя лицензии делают гиперссылкой. Например, если уведомление содержит описание **Текст лицензии CC-BY-SA**, а CC-BY-SA — это имя лицензии, то CC-BY-SA нужно сделать гиперссылкой.

### <a name="link-and-text-attribution"></a>Определение принадлежности текста или ссылки

Как правило, правила [LinkAttribution](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#linkattribution) и [TextAttribution](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#textattribution) определяют поставщика данных. Поле `targetPropertyName` определяет поле, к которому применяется правило.

Чтобы определить принадлежность поставщиков, сразу после содержимого, к которому применяется определение принадлежности (например, целевое поле), добавьте строку. Строка должна четко указывать, что данные принадлежат этим поставщикам. Например, "Данные с contoso.com". Если используются правила `LinkAttribution`, необходимо создать гиперссылку на веб-сайт поставщика.

Ниже приведен пример с использованием правил `LinkAttribution` и `TextAttribution`.

![Определение принадлежности ссылки](../media/cognitive-services-bing-entities-api/linktextattribution.png)

### <a name="media-attribution"></a>Определение принадлежности медиаданных

Если сущность содержит изображение, необходимо указать ссылку для перехода на веб-сайт поставщика. Если сущность включает правило [MediaAttribution](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#mediaattribution), создайте такую ссылку, указав URL-адрес правила. В противном случае используйте URL-адрес в поле `provider` изображения.

Ниже приведен пример, в котором используется поле `provider` изображения и договорные правила. Так как в этом примере содержится договорное правило, вы должны игнорировать поле `provider` изображения и применить правило `MediaAttribution`.

![Определение принадлежности медиаданных](../media/cognitive-services-bing-entities-api/mediaattribution.png)

### <a name="search-or-search-like-experience"></a>Поиск и подобные функции

Как и при использовании API Bing для поиска в Интернете, API Bing для поиска сущностей может использоваться только в результате прямого запроса пользователя или поиска, или в результате действия в приложении или пользовательском интерфейсе, который логически может быть интерпретирован как запрос на поиск пользователя. Для иллюстрации ниже приведены следующие примеры приемлемых запросов на поиск или подобные функции.

- Пользователь вводит запрос непосредственно в поле поиска в приложении
- Пользователь выбирает определенный текст или изображение и запрашивает "Подробнее" или "Дополнительные сведения".
- Пользователь запрашивает программу-робот про поиск об определенной теме
- Пользователь останавливается на конкретном объекте или объекте в сценарии типа визуального поиска

Если вы не уверены, что вашу среду можно считать обладающей схожими с поиском функциями, рекомендуется проверить ее с помощью Майкрософт.

## <a name="throttling-requests"></a>Запросы на регулирование

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы приступить к поиску сущностей с помощью API Поиска сущностей Bing, ознакомьтесь с [этим кратким руководством](../quickstarts/csharp.md).