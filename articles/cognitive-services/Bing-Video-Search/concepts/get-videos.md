---
title: Поиск видео с помощью API Bing для поиска видео
titleSuffix: Azure Cognitive Services
description: Поиск видео Bing Апифиндс и возвращает соответствующие видеоматериалы из Интернета, она предоставляет несколько функций для интеллектуального и специализированного извлечения видео в Интернете.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-video-search
ms.topic: conceptual
ms.date: 06/24/2019
ms.author: aahi
ms.openlocfilehash: 10277efe1f06de3633b2d614e2ee5ec0cc351c76
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96351935"
---
# <a name="search-for-videos-with-the-bing-video-search-api"></a>Поиск видео с помощью API Поиска видео Bing

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

API Поиска видео Bing позволяет легко интегрировать возможности когнитивного поиска новостей Bing в приложениях. API в основном находит и возвращает соответствующие видео из Интернета, а также предоставляет несколько функций для интеллектуального и тематического поиска видео в Интернете.

## <a name="getting-videos"></a>Получение видео

Чтобы получить видео, связанные с условием поиска пользователя в Интернете, отправьте следующий запрос GET.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-Search-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

Все запросы должны быть выполнены на сервере.

Если вы впервые вызовете любой из интерфейсов API Bing, не включайте заголовок идентификатора клиента. Включите идентификатор клиента, только если вы ранее вызывали API Bing, а Bing возвратил идентификатор клиента для комбинации пользователей и устройств.

Чтобы получить видео с определенного домена, используйте оператор запроса [site:](/previous-versions/bing/search/ff795613(v=msdn.10)).

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us HTTP/1.1
```

Ответ типа [Videos](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#videos) содержит список видео, которые поиск Bing посчитал соответствующими запросу. Каждый объект [Video](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video) в списке содержит URL-адрес видео, его продолжительность, размеры и формат кодировки среди других атрибутов. Объект видео также содержит URL-адрес эскиза видео и его размеры.

```json
{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545...",
    "totalEstimatedMatches" : 1000,
    "value" : [
        {
            "name" : "How to sail - What to Wear for Dinghy Sailing",
            "description" : "An informative video on what to wear when...",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7...",
            "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?id=OVP.DYW...",
            "datePublished" : "2014-03-04T11:51:53",
            "publisher" : [
                {
                    "name" : "Fabrikam"
                }
            ],
            "creator" : 
            {
                "name" : "Marcus Appel"
            },
            "contentUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjZ--g",
            "hostPageUrl" : "https:\/\/www.bing.com\/cr?IG=81EF7545D569...",
            "encodingFormat" : "h264",
            "hostPageDisplayUrl" : "https:\/\/www.fabrikam.com\/watch?v=vzmPjZ--g",
            "width" : 1280,
            "height" : 720,
            "duration" : "PT2M47S",
            "motionThumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?id=OM.Y62...",
            "embedHtml" : "<iframe width=\"1280\" height=\"720\" src=\"https:...><\/iframe>",
            "allowHttpsEmbed" : true,
            "viewCount" : 8743,
            "thumbnail" : 
            {
                "width" : 300,
                "height" : 168
            },
            "videoId" : "6DB795E11A6E3CBAAD636DB795E113CBAAD63",
            "allowMobileEmbed" : true,
            "isSuperfresh" : false
        },
        ...
    ],
    "queryExpansions" : [...],
    "nextOffsetAddCount" : 0,
    "pivotSuggestions" : [...]
}
```

## <a name="video-thumbnails"></a>Эскизы видео

Вы можете отобразить все или подмножество эскизов видео, возвращенных API Поиска видео Bing. Если вы отобразите подмножество, предоставьте пользователю возможность просмотра оставшихся видеороликов. В соответствии с [требованиями API Bing к использованию и отображению](../../bing-web-search/use-display-requirements.md) все видео должны отображаться в порядке, предоставленном в ответе. Сведения об изменении размера эскизов см. в [этой статье](../../bing-web-search/resize-and-crop-thumbnails.md). 

Когда пользователь наводит указатель на эскиз, можно использовать [motionThumbnailUrl](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video-motionthumbnailurl) для воспроизведения видео в режиме эскиза. Обязательно назначьте видеоролику эскиз при его отображении.

<!-- Removing until the images can be sanitized.
![Motion thumbnail of a video](../bing-web-search/media/cognitive-services-bing-web-api/bing-web-video-motion-thumbnail.PNG)
-->

Существует три варианта просмотра видео при щелчке эскиза:

- [hostPageUrl](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video-hostpageurl) для просмотра видео на исходном веб-сайте (например, YouTube);
- [webSearchUrl](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video-websearchurl) для просмотра видео в браузере видео Bing;
- [embdedHtml](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video-embedhtml) для внедрения видео в собственный интерфейс. 

Обязательно укажите издателя и автора для атрибуции видео во время его воспроизведения.

Подробнее об использовании [videoId](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#video-videoid) для получения полезных сведений о видео см. в разделе [Получение полезных сведений о видео](../video-insights.md).

## <a name="filtering-videos"></a>Фильтрация видео

По умолчанию API для поиска видео возвращает все видеоролики, относящиеся к запросу. Если вам нужны только бесплатные видеоролики или длиной менее пяти минут, используйте следующие параметры запроса фильтра:

- [цены](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#pricing) &mdash; на Отфильтруйте видео по ценам (например, к бесплатным видеоматериалам или к оплате).
- [решение](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#resolution) &mdash; Фильтрация видео по разрешению (например, видео с 720p или более высоким разрешением)
- [видеоленгс](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#videolength) &mdash; Отфильтруйте видео по длине видео (например, в видеороликах длиной менее пяти минут).
- [актуальность](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#freshness) &mdash; Фильтрация видео по возрасту (например, видео, обнаруженных Bing за прошлую неделю)

Чтобы получить видео из определенного домена, добавьте оператор запроса [site:](/previous-versions/bing/search/ff795613(v=msdn.10)) в строку запроса.

> [!NOTE]
> Если вы используете оператор запроса `site:`, в зависимости от запроса есть вероятность, что ответ включает материалы для взрослых независимо от параметра [safeSearch](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#safesearch). Вы должны использовать `site:`, только если вам известно о содержимом на сайте, и ваш сценарий поддерживает возможность использования содержимого для взрослых.

В следующем примере показано, как получить бесплатные видео с ContosoSailing.com с разрешением 720p или выше, которые Bing обнаружил в прошлом месяце.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/videos/search?q=sailing+dinghies+site:contososailing.com&pricing=free&freshness=month&resolution=720p&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-MSEdge-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

## <a name="expanding-the-query"></a>Расширение запроса

Если Bing может расширить запрос, чтобы сузить исходный поисковый запрос, объект [Videos](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#videos) будет содержать поле `queryExpansions`. Например, если запрос был *очистка водостоков*, расширенные запросы могут быть такими: **средства** для очистки водостоков, очистка водостоков **с земли**, **машина** для очистки водостоков и **легкая** очистка водостока.

В следующем примере показаны расширенные запросы для *очистки водостоков*.

```json
{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52FBC5...",
    "totalEstimatedMatches" : 1000,
    "value" : [...],
    "nextOffsetAddCount" : 4,
    "queryExpansions" : [
        {
            "text" : "Gutter Cleaning Tools",
            "displayText" : "Tools",
            "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52FB....",
            "searchLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v5...",
            "thumbnail" : {
                "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?q=Gutter..."
            }
        },
        ...
    ]
    "pivotSuggestions" : [...],
}
```

Поле `queryExpansions` содержит список объектов [Query](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#query_obj). Поле `text` содержит расширенный запрос, а `displayText` — условие расширения. Вы можете использовать поля текста и эскиза, чтобы отображать строки расширенных запросов для пользователя на случай, если он ищет именно их. Создайте эскиз и текст с помощью URL-адреса `webSearchUrl` или `searchLink`. Используйте `webSearchUrl`, чтобы отправить пользователя в результаты поиска Bing, или `searchLink`, если вы предоставите свою собственную страницу результатов.

## <a name="pivoting-the-query"></a>Сведение запроса

Если Bing может сегментировать исходный поисковый запрос, объект [Videos](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#videos) содержит поле `pivotSuggestions`. Например, если исходный запрос был *очистка водостоков*, Bing может сегментировать запрос на *очистка* и *водостоки*.

В следующем примере показаны сводные предложения для *очистки водостоков*.

```json
{
    "_type" : "Videos",
    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52FBC...",
    "totalEstimatedMatches" : 1000,
    "value" : [...],
    "nextOffsetAddCount" : 0,
    "queryExpansions" : [...],
    "pivotSuggestions" : [
        {
            "pivot" : "cleaning",
            "suggestions" : [
                {
                    "text" : "Gutter Repair",
                    "displayText" : "Repair",
                    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52...",
                    "searchLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v5\/videos...",
                    "thumbnail" : {
                        "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?q=Gutter..."
                    }
                },
                ...
            ]
        },
        {
            "pivot" : "gutters",
            "suggestions" : [
                {
                    "text" : "Window Cleaning",
                    "displayText" : "Window",
                    "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=B52FBC59...",
                    "searchLink" : "https:\/\/api.cognitive.microsoft.com\/api\/v5...",
                    "thumbnail" : {
                        "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?q=Window..."
                    }
                },
                ...
            ]
        }
    ]
}
```

Для каждой сводки ответ содержит список объектов [Query](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference#query_obj), содержащих предлагаемые запросы. Поле `text` содержит предлагаемый запрос, а `displayText` — условие, заменяющее сводку в исходном запросе. Например, очистка окна.

Вы можете использовать поля `text` и `thumbnail`, чтобы отображать строки расширенных запросов для пользователя, если они ищут именно такие строки. Создайте эскиз и текст с помощью URL-адреса `webSearchUrl` или `searchLink`. Используйте `webSearchUrl`, чтобы отправить пользователя в результаты поиска Bing, или `searchLink`, если вы предоставите свою собственную страницу результатов.

## <a name="throttling-requests"></a>Запросы на регулирование

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../../includes/cognitive-services-bing-throttling-requests.md)]