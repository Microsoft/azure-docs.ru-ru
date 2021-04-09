---
title: Отправка поисковых запросов в API Bing для поиска сущностей
titleSuffix: Azure cognitive Services
description: API Bing для поиска сущностей отправляет запрос на поиск в Bing и получает результаты, которые включают сущности и места.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: aahi
ms.openlocfilehash: ad6d89fec9f2c94129e19c09ee3e1e76d5bb6e44
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "96353278"
---
# <a name="sending-search-requests-to-the-bing-entity-search-api"></a>Отправка поисковых запросов в API Bing для поиска сущностей

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

API Bing для поиска сущностей отправляет запрос на поиск в Bing и получает результаты, которые включают сущности и места. Результаты поиска мест включают рестораны, отели или другие местные учреждения. При поиске мест в запросе может указываться имя местного учреждения или запрашиваться список (например, ближайших ресторанов). В результатах поиска сущностей содержатся субъекты, места или вещи. Местами в данном контексте являются достопримечательности, штаты, страны, регионы и т. д.

## <a name="the-endpoint"></a>Конечная точка

Чтобы получить результаты поиска сущностей и мест, отправьте запрос GET на следующую конечную точку:  

```
https://api.cognitive.microsoft.com/bing/v7.0/entities
```

Для запросов должен использоваться протокол HTTPS.

Рекомендуется, чтобы все запросы поступали с сервера. Распространение ключа в рамках клиентского приложения создает больше возможностей для доступа к нему злоумышленников. Осуществление вызовов с сервера также предоставляет единую точку обновления для будущих версий API.

## <a name="specifying-query-parameters-and-headers"></a>Указание параметров запроса и заголовков

В запросе необходимо указать параметр [q](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#query), который содержит условие поиска пользователя. В запросе также необходимо указать параметр [mkt](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#mkt), определяющий рынок, для которого будут отображаться результаты. Список необязательных параметров запроса можно найти в разделе [Параметры запроса](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#query-parameters). Все параметры запроса необходимо закодировать в URL-адресе.  
  
Запрос должен содержать заголовок [Ocp-Apim-Subscription-Key](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#subscriptionkey). Приведенные ниже заголовки являются необязательными, но их также рекомендуется указать:  
  
-   [User-Agent](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#useragent)  
-   [X-MSEdge-ClientID](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#clientid)  
-   [X-MSEdge-ClientIP](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#clientip)  
-   [X-Search-Location](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#location)  

Заголовки IP-адреса и расположения клиента важны для отображения содержимого с учетом расположения.  

Список всех заголовков в запросах и ответах приведен в разделе [Заголовки](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#headers).

## <a name="the-request"></a>Запрос

Ниже показан запрос сущностей, который включает в себя все рекомендуемые параметры и заголовки запроса. 

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/entities?q=mount+rainier&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

Если вы впервые вызовете любой из интерфейсов API Bing, не включайте заголовок идентификатора клиента. Включите идентификатор клиента, только если вы ранее вызывали API Bing, а Bing возвратил идентификатор клиента для комбинации пользователей и устройств.

## <a name="the-response"></a>Ответ

Ниже показан ответ на предыдущий запрос. В примере также показаны заголовки ответа для Bing. Сведения об объекте ответа см. в разделе [SearchResponse](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#searchresponse).

[!INCLUDE [cognitive-services-bing-url-note](../../../../includes/cognitive-services-bing-url-note.md)]

```json
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{
    "_type" : "SearchResponse",
    "queryContext" : {
        "originalQuery" : "mount rainier"
    },
    "entities" : {
        "queryScenario" : "DominantEntity",
        "value" : [{
            "contractualRules" : [{
                "_type" : "ContractualRules\/LicenseAttribution",
                "targetPropertyName" : "description",
                "mustBeCloseToContent" : true,
                "license" : {
                    "name" : "CC-BY-SA",
                    "url" : "http:\/\/creativecommons.org\/licenses\/by-sa\/3.0\/"
                },
                "licenseNotice" : "Text under CC-BY-SA license"
            },
            {
                "_type" : "ContractualRules\/LinkAttribution",
                "targetPropertyName" : "description",
                "mustBeCloseToContent" : true,
                "text" : "en.wikipedia.org",
                "url" : "http:\/\/en.wikipedia.org\/wiki\/Mount_Rainier"
            },
            {
                "_type" : "ContractualRules\/MediaAttribution",
                "targetPropertyName" : "image",
                "mustBeCloseToContent" : true,
                "url" : "http:\/\/en.wikipedia.org\/wiki\/Mount_Rainier"
            }],
            "webSearchUrl" : "https:\/\/www.bing.com\/search?q=Mount%20Rainier...",
            "name" : "Mount Rainier",
            "image" : {
                "name" : "Mount Rainier",
                "thumbnailUrl" : "https:\/\/www.bing.com\/th?id=A21890c0e1f...",
                "provider" : [{
                    "_type" : "Organization",
                    "url" : "http:\/\/en.wikipedia.org\/wiki\/Mount_Rainier"
                }],
                "hostPageUrl" : "http:\/\/upload.wikimedia.org\/wikipedia...",
                "width" : 110,
                "height" : 110
            },
            "description" : "Mount Rainier, Mount Tacoma, or Mount Tahoma is the highest...",
            "entityPresentationInfo" : {
                "entityScenario" : "DominantEntity",
                "entityTypeHints" : ["Attraction"],
                "entityTypeDisplayHint" : "Mountain"
            },
            "bingId" : "9ae3e6ca-81ea-6fa1-ffa0-42e1d78906"
        }]
    }
}
```


## <a name="next-steps"></a>Дальнейшие действия

* [Поиск сущностей с помощью API Bing для поиска сущностей](search-for-entities.md)
* [Требования к отображению данных и использованию API Bing](../../bing-web-search/use-display-requirements.md)