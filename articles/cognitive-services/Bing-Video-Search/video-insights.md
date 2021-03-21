---
title: Получение полезных сведений о видео с помощью API Bing для поиска видео
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать API Bing для поиска видео для получения дополнительных сведений о видео, в том числе связанных видео.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-video-search
ms.topic: conceptual
ms.date: 01/31/2019
ms.author: scottwhi
ms.openlocfilehash: cbbde354c1bc68a2b1017c3ccba61b846fa62916
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94380765"
---
# <a name="get-insights-about-a-video"></a>Получение полезных сведений о видео

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Каждое видео, возвращаемое API Bing для поиска видео, содержит идентификатор, с помощью которого можно получить дополнительные сведения об этом видео, например найти связанные видео. Чтобы получить полезные сведения о видео, получите токен [videoId](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video-videoid), указанный в ответе API. 

```json
    "value" : [
        {
            . . .
            "name" : "How to sail - What to Wear for Dinghy Sailing",
            "description" : "An informative video on what to wear...",
            "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjHBZ--g",
            "videoId" : "6DB795E11A6E3CBAAD636DB795E11A6E3CBAAD63",
            . . .
        }
    ],
```

Затем отправьте запрос GET к конечной точке сведений о видео, указав этот идентификатор. Укажите для параметра запроса [id](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#id) значение токена `videoId`. Чтобы указать полезные сведения, которые необходимо получить, задайте значение параметра запроса [modules](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#modulesrequested). Чтобы получить все сведения, задайте для `modules` значение All. Ответ будет содержать все сведения, которые вы запросили, если они доступны.

```cURL
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/details?q=sailiing+dinghies&id=6DB795E11A6E3CBAAD636DB795E11A6E3CBAAD63&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
``` 

## <a name="getting-related-videos-insights"></a>Получение полезных сведений о связанных видео  

Чтобы найти видео, связанные с указанным видео, задайте для параметра запроса [modules](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#modulesrequested) значение `RelatedVideos`.
  
```cURL  
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/details?q=sailiing+dinghies&id=6DB795E11A6E3CBAAD636DB795E11A6E3CBAAD63&modules=RelatedVideos&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-Search-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```  

Ответ на этот запрос будет содержать объект верхнего уровня [VideoDetails](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#videodetails) вместо объекта [Videos](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#videos).  
  
```json
{
    "_type" : "Api.VideoDetails.VideoDetails",
    "relatedVideos" : {
        "value" : [
            {
                "name" : "How to sail - Reefing a Sail",
                "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=7284B07...",
                "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?id=OVP.zt...",
                "datePublished" : "2014-03-04T16:11:09",
                "publisher" : [
                    {
                        "name" : "Fabrikam"
                    }
                ],
                "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=...",
                "hostPageUrl" : "https:\/\/www.bing.com\/cr?IG=7284B07...",
                "hostPageDisplayUrl" : "https:\/\/www.fabrikam.com\/watch?...",
                "duration" : "PT4M56S",
                "motionThumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OM...",
                "allowHttpsEmbed" : true,
                "viewCount" : 21756,
                "videoId" : "AC1A157A4DDB571D03D6AC157A4DDB571D03D6",
                "allowMobileEmbed" : false,
                "isSuperfresh" : false
            },
            . . .
        ]
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Поиск видеороликов по тенденциям](trending-videos.md)