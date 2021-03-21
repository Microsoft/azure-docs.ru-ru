---
title: Фильтрация результатов поиска — API Bing для поиска в Интернете
titleSuffix: Azure Cognitive Services
description: Можно отфильтровать типы ответов, включаемых Bing в ответ (например, изображения, видео и новости), с помощью параметра запроса "Респонсефилтер".
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: 8B837DC2-70F1-41C7-9496-11EDFD1A888D
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 07/08/2019
ms.author: scottwhi
ms.openlocfilehash: 571314009b6f58e5c2ab6aac02cfebc82c53f42f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96351867"
---
# <a name="filtering-the-answers-that-the-search-response-includes"></a>Фильтрация результатов, возвращаемых в ответе на запрос поиска  

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Когда вы делаете запрос в Интернете, Bing возвращает все релевантное содержимое для поиска. Например, если поисковый запрос содержит условие "кораблевождение+шлюпки", то ответ может содержать приведенные ниже результаты.

```json
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43C...",
        "totalEstimatedMatches" : 262000,
        "value" : [...]
    },
    "images" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=sail...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43CA5CA6464E5D...",
        "isFamilyFriendly" : true,
        "value" : [...]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [...]
        }
    }
}    
```

## <a name="query-parameters"></a>Параметры запроса

Чтобы отфильтровать ответы, возвращаемые Bing, используйте приведенные ниже параметры запроса при вызове API.  

### <a name="responsefilter"></a>респонсефилтер

Можно отфильтровать типы ответов, включаемых Bing в ответ (например, изображения, видео и новости), с помощью параметра запроса [респонсефилтер](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#responsefilter) , который представляет собой список ответов с разделителями-запятыми. Ответ будет добавлен в ответ, если Bing найдет соответствующее для него содержимое. 

Чтобы исключить конкретные ответы из ответа, например изображения, добавьте `-` символ к типу ответа. Пример:

```
&responseFilter=-images,-videos
```

Ниже показано, как использовать `responseFilter` для запроса изображений, видео и новостей по вождению шлюпок. При кодировании строки запроса запятые меняются на %2C.  

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&responseFilter=images%2Cvideos%2Cnews&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

Ниже показан ответ на предыдущий запрос. Так как служба Bing не нашла подходящих видео и новостей, их не будет в ответе.

```json
{
    "_type" : "SearchResponse",
    "images" : {
        "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images",
        "readLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/images\/search?q=sail...",
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3AD78B183C56456C...",
        "isFamilyFriendly" : true,
        "value" : [...]
    },
    "rankingResponse" : {
        "mainline" : {
            "items" : [{
                "answerType" : "Images",
                "value" : {
                    "id" : "https:\/\/api.cognitive.microsoft.com\/api\/v7\/#Images"
                }
            }]
        }
    }
}
```

Хотя служба Bing не вернула видео и новости в предыдущем ответе, это не означает, что таких видео и новостей не существует. Это лишь означает, что их нет на странице. Тем не менее, если вы [пролистаете страницы](./paging-search-results.md) с дополнительными результатами, скорее всего, они будут содержать видео или новости. Кроме того, если вызвать конечные точки [API для поиска видео](../bing-video-search/overview.md) и [API для поиска новостей](../bing-news-search/search-the-web.md) напрямую, скорее всего, ответ будет содержать эти результаты.

Не рекомендуется использовать `responseFilter` для получения результатов из одного API. Если требуется содержимое из одного API Bing, вызывайте этот API напрямую. Например, чтобы получить только изображения, отправьте запрос к конечной точке API для поиска изображений (`https://api.cognitive.microsoft.com/bing/v7.0/images/search`) или другой конечной точке [службы "Поиск изображений"](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#endpoints). Вызов одного API важен не только из соображений производительности, но потому, что интерфейсы API конкретного содержимого предоставляют более подробные результаты. Например, можно использовать фильтры результатов, которые недоступны в API для поиска в Интернете.  

### <a name="site"></a>Сайт

Чтобы получить результаты поиска из определенного домена, включите `site:` параметр запроса в строку запроса.  

```
https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us
```

> [!NOTE]
> Если вы используете оператор `site:`, в зависимости от запроса есть вероятность, что ответ включает содержимое для взрослых независимо от параметра [safeSearch](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#safesearch). Вы должны использовать `site:`, только если вам известно о содержимом на сайте, и ваш сценарий поддерживает возможность использования содержимого для взрослых.

### <a name="freshness"></a>Актуальность

Чтобы ограничить результаты Интернета веб-страницами, обнаруженными Bing в течение определенного периода времени, задайте для параметра запроса [свежести](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#freshness) одно из следующих значений без учета регистра:

* `Day` — Возврат веб-страниц, обнаруженных Bing за последние 24 часа
* `Week` — Возврат веб-страниц, обнаруженных Bing за последние семь дней
* `Month` — Возврат веб-страниц, обнаруженных в течение последних 30 дней

Можно также задать для этого параметра настраиваемый диапазон дат в форме `YYYY-MM-DD..YYYY-MM-DD` . 

`https://<host>/bing/v7.0/search?q=ipad+updates&freshness=2019-02-01..2019-05-30`

Чтобы ограничить результаты одной датой, задайте для параметра свежести определенную дату:

`https://<host>/bing/v7.0/search?q=ipad+updates&freshness=2019-02-04`

Результаты могут включать веб-страницы, находящиеся за пределами указанного периода, если число веб-страниц, найденных Bing в критерии фильтра, меньше числа запрошенных веб-страниц (или числа по умолчанию, возвращаемого Bing).

## <a name="limiting-the-number-of-answers-in-the-response"></a>Ограничение числа результатов в ответе

Bing может возвращать несколько типов ответов в ответе JSON. Например, при запросе *гоночные + дингхиес* Bing может возвращать `webpages` , `images` , `videos` и `relatedSearches` .

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "relatedSearches" : {...},
    "videos" : {...},
    "rankingResponse" : {...}
}
```

Чтобы ограничить количество результатов, которые возвращает служба Bing, двумя наиболее популярными ответами (веб-страницы и изображения), задайте для параметра запроса [answerCount](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#answercount) значение 2.

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&answerCount=2&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

Ответ содержит только `webPages` и `images`.

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "rankingResponse" : {...}
}
```

Если в предыдущий запрос добавить параметр `responseFilter` и задать для него веб-страницы и новости, то ответ будет содержать только веб-страницы, так как новости не ранжируются.

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailing dinghies"
    },
    "webPages" : {...},
    "rankingResponse" : {...}
}
```

## <a name="promoting-answers-that-are-not-ranked"></a>Повышение уровня результатов, которые не ранжированы

Если наиболее подходящими ранжированными результатами запроса являются веб-страницы, изображения, видео и сведения relatedSearches, то служба Bing возвращает их. Если задать для параметра [answerCount](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#answercount) значение 2, служба Bing вернет два наиболее подходящих ранжированных результата: веб-страницы и изображения. Если требуется, чтобы служба Bing включила в ответ изображения и видео, укажите параметр запроса [promote](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#promote) и задайте для него изображения и видео.

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&answerCount=2&promote=images%2Cvideos&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location:  47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

Ниже показан ответ на представленный выше запрос. Служба Bing возвращает два наиболее подходящих результата, веб-страницы и изображения, и повышает уровень видео в ответе.

```json
{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "sailiing dinghies"
    },
    "webPages" : {...},
    "images" : {...},
    "videos" : {...},
    "rankingResponse" : {...}
}
```

Если задать для параметра `promote` новости, ответ не будет содержать новостей, так как они не ранжируются&mdash;, а повысить уровень можно только для ранжированных результатов.

Результаты, уровень которых требуется повысить, не учитываются в предельном количестве `answerCount`. Например, если к ранжированным результатам относятся новости, изображения и видео, для параметра `answerCount` задано значение 1 и для параметра `promote` заданы новости, то ответ содержит новости и изображения. Если же к ранжированным результатам относятся видео, изображения и новости, то ответ содержит видео и новости.

Вы можете использовать `promote` только в том случае, если указан параметр запроса `answerCount`.