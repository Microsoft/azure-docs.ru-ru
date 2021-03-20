---
title: Отправка поисковых запросов в API Bing для поиска видео
titleSuffix: Azure Cognitive Services
description: В этой статье описаны параметры и атрибуты запросов, отправляемых в API Bing для поиска видео, а также возвращаемый объект ответа в формате JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-video-search
ms.topic: conceptual
ms.date: 12/03/2019
ms.author: aahi
ms.openlocfilehash: 78a3c3c2936bfcaf58ea9e46bd3d6a610cdfe436
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96341501"
---
# <a name="sending-search-requests-to-the-bing-video-search-api"></a>Отправка поисковых запросов в API Bing для поиска видео

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

В этой статье описаны параметры и атрибуты запросов, отправляемых в API Bing для поиска видео, а также возвращаемый объект ответа в формате JSON. 

[!INCLUDE [cognitive-services-bing-video-search-signup-requirements](../../../../includes/cognitive-services-bing-video-search-signup-requirements.md)]

## <a name="suggest-search-terms-with-the-bing-autosuggest-api"></a>Предложение условий поиска с помощью API автозаполнения Bing

Если вы предоставили окно поиска, в котором пользователь вводит свой поисковый запрос, используйте [API автозаполнения Bing](../../bing-autosuggest/get-suggested-search-terms.md), чтобы оптимизировать работу. API возвращает предложенные строки запроса на основе частичного поиска, как пользовательские типы.

После ввода условия поиска примените к нему URL-кодирование перед установкой параметра запроса [q](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#query). Например, если пользователь вводит *парусные шлюпки*, установите для параметра `q` значение `sailing+dinghies` или `sailing%20dinghies`.

## <a name="sending-a-request"></a>Отправка запроса

Для получения результатов поиска видео необходимо отправить запрос GET к следующей конечной точке:  
  
```
https://api.cognitive.microsoft.com/bing/v7.0/videos/search
```
   
Для отправки запроса следует использовать протокол HTTPS.

Рекомендуется, чтобы все запросы поступали с сервера. Распространение ключа в рамках клиентского приложения создает больше возможностей для доступа к нему злоумышленников. Осуществление вызовов с сервера также предоставляет единую точку обновления для будущих версий API.

  
В запросе необходимо указать параметр [q](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#query), который содержит условие поиска пользователя. В запросе можно также указать необязательный параметр [mkt](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#mkt), определяющий рынок, для которого будут отображаться результаты. Список необязательных параметров запроса, таких как `pricing`, можно найти в разделе [Параметры запроса](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#query-parameters). Все значения параметров запроса должны быть указаны в формате URL-адреса.  
  
Запрос должен содержать заголовок [Ocp-Apim-Subscription-Key](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#subscriptionkey). Приведенные ниже заголовки являются необязательными, но их также рекомендуется указать:  
  
-   [Агент пользователя](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#useragent)  
-   [X-MSEdge-ClientID](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#clientid)  
-   [X-Search-ClientIP](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#clientip)  
-   [X-Search-расположение](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#location)  

Заголовки IP-адреса и расположения клиента важны для отображения содержимого с учетом расположения.  

Список всех заголовков в запросах и ответах приведен в разделе [Заголовки](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#headers).

## <a name="example-search-request"></a>Пример поискового запроса

Ниже показан поисковый запрос, который включает в себя все рекомендуемые параметры и заголовки запроса. Если вы впервые вызовете любой из интерфейсов API Bing, не включайте заголовок идентификатора клиента. Включите идентификатор клиента, только если вы ранее вызывали API Bing, а Bing возвратил идентификатор клиента для комбинации пользователей и устройств. 
  
```  
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

## <a name="example-json-response"></a>Пример ответа в формате JSON

Ниже показан ответ на предыдущий запрос. В примере также показаны заголовки ответа для Bing.

[!INCLUDE [cognitive-services-bing-url-note](../../../../includes/cognitive-services-bing-url-note.md)]

```json
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D5694...",
    "totalEstimatedMatches" : 1000,
    "value" : [
        {
            "name" : "How to sail - What to Wear for Dinghy Sailing",
            "description" : "An informative video on what to wear when...",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D56...",
            "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OVP.DYWCvh...",
            "datePublished" : "2014-03-04T11:51:53",
            "publisher" : [
                {
                    "name" : "Fabrikam"
                }
            ],
            "creator" : {
                "name" : "Marcus Appel"
            },
            "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjHZ--g",
            "hostPageUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D56944...",
            "encodingFormat" : "h264",
            "hostPageDisplayUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjBZ--g",
            "width" : 1280,
            "height" : 720,
            "duration" : "PT2M47S",
            "motionThumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?id=OM.Y6...",
            "embedHtml" : "<iframe width=\"1280\" height=\"720\" src=\"https:...><\/iframe>",
            "allowHttpsEmbed" : true,
            "viewCount" : 8743,
            "thumbnail" : {
                "width" : 300,
                "height" : 168
            },
            "videoId" : "6DB795E11A6E3CBAAD636DB795E11E3CBAAD63",
            "allowMobileEmbed" : true,
            "isSuperfresh" : false
        },
        . . .
    ],
    "nextOffset" : 0,
    "pivotSuggestions" : [
        {
            "pivot" : "sailing",
            "suggestions" : []
        },
        {
            "pivot" : "dinghies",
            "suggestions" : [
                {
                    "text" : "Sailing Cruising",
                    "displayText" : "Cruising",
                    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF754...",
                    "searchLink" : "https:\/\/api.cognitive.microsoft.com...",
                    "thumbnail" : {
                        "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?q=Sailing..."
                    }
                },
                . . .
            ]
        }
    ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

Проверьте, как работает API. Перейдите на страницу [консоли тестирования API для поиска видео](https://dev.cognitive.microsoft.com/docs/services/56b43f3ccf5ff8098cef3809/operations/58113fe5e31dac0a1ce6b0a8). 

Дополнительные сведения об использовании объектов ответа см. в разделе о [поиске видео в Интернете](../overview.md).

Дополнительные сведения о получении аналитических сведений о видео, таких как связанные поисковые запросы, см. в разделе об [аналитике для видео](../video-insights.md).  
  
Сведения о видео, которое попадает в тренды в социальных сетях, см. в разделе о [видео, набирающих популярность](../trending-videos.md).