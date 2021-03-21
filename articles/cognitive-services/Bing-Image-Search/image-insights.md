---
title: Получений аналитических сведений об изображениях — API Bing для поиска изображений
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать API Bing для поиска изображений для получения дополнительных сведений об изображениях.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: 0BCD936E-D4C0-472D-AE40-F4B2AB6912D5
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: conceptual
ms.date: 03/04/2019
ms.author: scottwhi
ms.openlocfilehash: 9dffc16a1d1171343fa10fce10fddba2a12c2167
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96348251"
---
# <a name="get-image-insights-with-the-bing-image-search-api"></a>Получение аналитических сведений об изображениях с помощью API Bing для поиска изображений

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

> [!IMPORTANT]
> В случае использования конечной точки /images/details для получения аналитических сведений об изображении вы должны использовать [Визуальный поиск](../bing-visual-search/overview.md), так как он предоставляет более подробные аналитические сведения.


Каждое изображение включает в себя токен аналитики, который вы можете использовать для получения сведений об изображении. Например, вы можете получить коллекцию связанных изображений, веб-страниц, содержащих изображение, или список продавцов, у которых вы можете приобрести продукт, показанный на изображении.  

Для получения сведений об изображении запишите токен [imageInsightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#image-imageinsightstoken), указанный в ответе.

```json
"value" : [{
        . . .
        "name":"sailing dinghy.jpg",
        "imageInsightsToken" : "mid_D6426898706EC7..."
        "insightsSourcesSummary" : {
            "shoppingSourcesCount" : 9,
            "recipeSourcesCount" : 0
        },
        . . .
}],
```

Затем вызовите конечную точку "Сведения об образе" и установите в качестве значения параметра запроса [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) токен `imageInsightsToken`.  

Чтобы указать полезные сведения, которые необходимо получить, задайте значение параметра запроса `modules`. Чтобы получить все сведения, задайте для `modules` значение `All`. Чтобы получить только сведения о заголовках и коллекциях, установите для `modules` значение `Caption%2CCollection`. Полный список возможных аналитических сведений см. в разделе [модулей](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested). Не все аналитические сведения доступны для всех изображений. Ответ будет содержать все сведения, которые вы запросили, если они доступны.

В следующем примере запрашиваются все доступные аналитические сведения для предыдущего изображения.

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&insightsToken=mid_D6426898706EC7...&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

## <a name="getting-insights-of-a-known-image"></a>Получение полезных сведений об известном изображении

Если у вас есть URL-адрес изображения, о котором вы хотите получить аналитические сведения, используйте параметр запроса [imgUrl](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imgurl) вместо параметра [insightsToke ](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken), чтобы указать изображение. Или, если у вас есть изображения, вы можете отправить двоичный файл изображения в теле запроса POST. Если вы используете запрос POST, для заголовка `Content-Type` следует задать значение `multipart/data-form`. В обоих случаях размер изображения не может превышать 1 МБ.  

Если у вас есть URL-адрес изображения, в следующем примере показано, как запросить аналитические сведения об изображении.

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&imgUrl=https%3A%2F%2Fwww.mydomain.com%2Fimages%2Fsunflower.png&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

## <a name="getting-all-image-insights"></a>Получение всех аналитических сведений для изображения  

Чтобы запросить все аналитические сведения для изображения, установите для параметра запроса [modules](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested) значение `All`. Чтобы получить связанные запросы, запрос должен содержать строку запроса пользователя. В этом примере показано использование [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) для указания изображения.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=sailing+dinghy&insightsToken=mid_68364D764J...&modules=All&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

Объект верхнего уровня представляет собой объект [ImageInsightsResponse](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imageinsightsresponse) вместо объекта [Images](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#images).  

```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "bcid_3297E6A54E4787A5F51C49D9BA342B9A*ccid_Fe2Hx...",
    "bestRepresentativeQuery" : {
        "text" : "Sailing Dinghy",
        "displayText" : "Sailing Dinghy",
        "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=Sailing+Dinghy...",
    },
    "pagesIncluding" : {
        "value" : [
            {
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=...",
                "name" : "Powerboating Dublin, Dinghy Sailing Courses...",
                "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP....",
                "datePublished" : "2017-01-20T00:41:00.0000000Z",
                "contentUrl" : "http:\/\/www.contoso.ie\/content...",
                "hostPageUrl" : "http:\/\/www.contoso.ie\/powerboating...",
                "contentSize" : "59063 B",
                "encodingFormat" : "jpeg",
                "hostPageDisplayUrl" : "www.contoso.ie\/powerboating...",
                "width" : 800,
                "height" : 600,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 225
                },
                "imageInsightsToken" : "ccid_pHjQIA0x*mid_17F61B1316A39C92214...",
                "imageId" : "17F61B1316A39C922143FFDE9DFB5B0FB41171",
                "accentColor" : "0997C2"
            },
            . . .
        ]
    },
    "relatedSearches" : {
        "value" : [
            {
                "text" : "Sailing Fun",
                "displayText" : "Sailing Fun",
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?q=Sailing...",
                "thumbnail" : {
                    "url" : "https:\/\/tse1.mm.bing.net\/th?q=Sailing+Fun..."
                }
            },
            . . .
        ]
    },
    "visuallySimilarImages" : {
        "value" : [
            {
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=...",
                "name" : "Weekend On the Water",
                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?id=OIP...",
                "datePublished" : "2010-09-05T12:00:00.0000000Z",
                "contentUrl" : "http:\/\/1.bp.contoso.com\/_dc_6...",
                "hostPageUrl" : "http:\/\/contoso.com\/2010...",
                "contentSize" : "203806 B",
                "encodingFormat" : "jpeg",
                "hostPageDisplayUrl" : "contoso.com\/2010...",
                "width" : 1600,
                "height" : 1249,
                "thumbnail" : {
                    "width" : 300,
                    "height" : 234
                },
                "imageInsightsToken" : "ccid_Jg1Kwuc4*mid_5B7DA43976D3A422...",
                "imageId" : "5B7DA43976D3A422BA679A3AB019BB52C08DBC",
                "accentColor" : "0B2543"
            },
            . . .
        ]
    },
    "imageTags" : {
        "value" : [
            {
                "name" : "sail boat"
            },
            . . .
        ]
    }
}
```

## <a name="recognizing-entities-in-an-image"></a>Распознавание сущностей на изображении  

Функция распознавания сущности идентифицирует сущности на изображении, в настоящее время только людей. Чтобы идентифицировать сущности на изображении, установите параметр запроса [modules](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested) для `RecognizedEntities`.  

> [!NOTE]
> Этот модуль нельзя использовать с любым другим модулем. Если вы укажете этот модуль с другими модулями, ответ не будет включать распознанные сущности.  

Ниже показано, как указать изображение с помощью параметра [imgUrl](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imgurl). Не забывайте, что параметры запроса нужно закодировать в URL-адресе.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=faith+hill&insightsToken=mid_68364D764J...&modules=RecognizedEntities&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```  

Ниже показан ответ на предыдущий запрос. Так как изображение содержит двух человек, ответ определяет область для каждого. В этом случае люди были распознаны в группах CelebrityAnnotations и CelebRecognitionAnnotations. Bing перечисляет людей в каждой группе, исходя из вероятности соответствия лица исходного изображения. Список составляется по убыванию достоверности. Группа CelebRecognitionAnnotations предоставляет наивысший уровень достоверности в правильности совпадения.  

```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "recognizedEntityGroups" : {
        "value" : [
            {
                "recognizedEntityRegions" : [...],
                "name" : "CelebRecognitionAnnotations"
            },
            {
                "recognizedEntityRegions" : [...],
                "name" : "CelebrityAnnotations"
            }
        ]
    }
}
```

Поле `region` определяет область изображения, где Bing распознал сущности. Для людей область — это лицо человека.  

Значения прямоугольника относительны ширине и высоте исходного изображения и находятся в диапазоне от 0,0 до 1,0. Например, если изображение 300x200, верхний левый угол области находится в точке (10; 20), а нижний правый угол находится в точке (290; 150), тогда стандартный прямоугольник:  

-   По левому краю: 10 / 300 = 0,03333...  
-   Сверху: 20 / 200 = 0,1  
-   По правому краю: 290 / 300 = 0,9667...  
-   Снизу: 150 / 200 = 0,75  

Вы можете использовать область, которую Bing возвращает при последующих вызовах аналитических сведений. Например, чтобы получить визуально похожие изображения распознанной сущности. Дополнительные сведения см. в разделе "Обрезка изображений для использования с визуально похожими модулями распознавания сущности". Ниже показано сопоставление полей области и параметров запроса, которые вы использовали для обрезки изображений.  

-   Сопоставление по левому краю [cal](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cal)  
-   Сопоставление по верхнему краю [cat](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cat)  
-   Сопоставление по правому краю [car](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#car)  
-   Сопоставление по нижнему краю[cab](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cab)  

## <a name="finding-visually-similar-images"></a>Поиск визуально похожих изображений  

Чтобы найти изображения, визуально похожие на исходное, установите параметр запроса [modules](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested) в категорию SimilarImages.  

Следующий запрос показывает способ получения визуально похожих изображений. Запрос использует параметр запроса [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) для идентификации исходного изображения. Чтобы повысить релевантность, следует включить строку запроса пользователя.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?insightsToken=mid_68364D764J...&modules=SimilarImages&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```


Ниже показан ответ на предыдущий запрос.  

```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "visuallySimilarImages" : {
        "value" : [
            {
                "name" : "typical Hawaiian Sunset! :) | Scenes of Hawaii",
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=detailv2...",
                "thumbnailUrl" : "https:\/\/tse1.mm.bing.net\/th?id=OIP.Mda2a86...",
                 . . .
            }
        ]
    }
```

## <a name="cropping-images-to-use-with-visually-similar-and-entity-recognition-modules"></a>Обрезка изображений для использования с визуально похожими модулями распознавания сущности  

Чтобы указать область изображения, которую Bing использует для определения визуального сходства изображений или для распознавания сущностей, используйте параметры запроса [cal](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cal), [cat](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cat), [cab](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#cab) и [car](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#car). По умолчанию Bing использует изображение целиком.  

Параметры определяют верхний левый и нижний правый угол области, которую Bing использует для сравнения. Укажите значения долей ширины и высоты исходного изображения. Дробные значения начинаются с (0,0; 0,0) в верхнем левом углу и заканчиваются (1,0; 1,0) в нижнем правом углу. Например, чтобы указать, что верхний левый угол начинается на четверть вниз от верха и четверть с левой стороны, задайте для `cal` и `cat` значение 0,25.  

Следующая последовательность вызовов показывает результат указания области обрезки. Первый вызов не включает обрезку, и Bing распознает двух людей, стоящих рядом друг с другом в середине изображения.  

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?modules=RecognizedEntities&imgurl=https%3A%2F%2Ftse1.mm.bing.net%2Fth%3Fid%3DOIP.M0cbee6fadb43f35b2344e53da7a23ec1o0%26pid%3DApi&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```  

Ответ содержит две распознанные сущности.  

```json
{  
    "_type" : "ImageInsights",  
    "recognizedEntityGroups" : {
        "value": [  
            . . .  
            {  
                "recognizedEntityRegions" : [{  
                    "region" : {  
                        "left" : 0.5066667,  
                        "top" : 0.1955556,  
                        "right" : 0.75,  
                        "bottom" : 0.52  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Charlene Whitney",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                },  
                {  
                    "region" : {  
                        "left" : 0.25,  
                        "top" : 0.2488889,  
                        "right" : 0.4466667,  
                        "bottom" : 0.5111111  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Marcus Appel",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                }],  
            "name" : "CelebRecognitionAnnotations"
        }]
    }  
}  
```  

Второй вызов обрезает изображение по вертикали вниз, пополам, и Bing распознает одного человека с правой стороны изображения.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?cal=0.5&cat=0.0&car=1.0&cab=1.0&modules=RecognizedEntities&imgurl=https%3A%2F%2Ftse1.mm.bing.net%2Fth%3Fid%3DOIP.M0cbee6fadb43f35b2344e53da7a23ec1o0%26pid%3DApi&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

Ответ содержит одну распознанную сущность.  

```json  
{  
    "_type" : "ImageInsights",  
    "recognizedEntityGroups" : {
        "value" : [  
            . . .  
            {  
                "recognizedEntityRegions" : [{  
                    "region" : {  
                        "left" : 0.5066667,  
                        "top" : 0.1955556,  
                        "right" : 0.75,  
                        "bottom" : 0.52  
                    },  
                    "matchingEntities" : [{  
                        "entity" : {  
                            "_type" : "Person",  
                            "name" : "Charlene Whitney",  
                            . . .  
                        },  
                        "matchConfidence" : 0.9961388  
                    }]  
                }],  
                "name" : "CelebRecognitionAnnotations"  
            }
        ]
    }
}  
```  

## <a name="finding-visually-similar-products"></a>Поиск визуально похожих продуктов  

Чтобы найти изображения, содержащие продукты, которые визуально похожи на продукты, найденные на исходном изображении, задайте для параметра запроса [модулей](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#modulesrequested) значение симиларпродуктс.  

Следующий запрос показывает способ получения изображений визуально похожих продуктов. Запрос использует параметр запроса [insightsToken](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#insightstoken) для идентификации исходного изображения, которое было возвращено в предыдущем запросе. Чтобы повысить релевантность, следует включить строку запроса пользователя.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?q=anne+klein+dresses&modules=SimilarProducts&insightsToken=ccid_WOeyfoSp*mid_4B0A357&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

Ниже показан ответ на предыдущий запрос. Ответ содержит изображение аналогичного продукта и указывает, сколько продавцов предлагает продукт онлайн, есть ли рейтинги продуктов, а также самую низкую цену (см. поле `aggregateOffer`).  

```json
{
    "_type" : "ImageInsights",
    "imageInsightsToken" : "ccid_ldi5nX38*mid_29470780DE0E6F969...",
    "visuallySimilarProducts" : {
        "value" : [
            {
                "name" : "Sequin One-Shoulder Twist-Drape Dress",  
                "webSearchUrl" : "https:\/\/www.bing.com\/images\/search?view=de...",  
                "thumbnailUrl" : "https:\/\/tse2.mm.bing.net\/th?id=OIP.M85bdee...",  
                . . .
            },
            . . .
        ]
    }
}
```

Чтобы получить список продавцов, предлагающих продукт в Интернете (см. поле [offerCount](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)), снова вызовите API и задайте `modules` для ShoppingSources. Затем задайте параметр запроса `insightsToken` в токен, найденный в сводном изображении продукта.  

```
GET https://api.cognitive.microsoft.com/bing/v7.0/images/details?modules=ShoppingSources&insightsToken=ccid_hb3uRvUk*mid_BF5C252A47F2C765...&mkt=en-us HTTP/1.1    
Ocp-Apim-Subscription-Key: 123456789ABCDE  
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com
```

Ниже показан ответ на предыдущий запрос.  

```json  
{  
    "_type" : "ImageInsights",  
    "shoppingSources" : {  
        "offers" : [{  
            "url" : "http:\/\/www.contoso.com\/dp\/B00O...",  
            "seller" : {  
                "name" : "Contoso",  
                "image" : {  
                    "url" : "https:\/\/tse3.mm.bing.net\/th?id=A10d50fe..."  
                }  
            },  
            "price" : 126.87,  
            "priceCurrency" : "USD",  
            "availability" : "InStock"  
        },  
        {  
            "url" : "http:\/\/www.adatum.com\/product\/heritage...\/",  
            "seller" : {  
                "name" : "fabrikam.com"  
            },  
            "price" : 495,  
            "priceCurrency" : "USD",  
            "availability" : "InStock"  
        }]  
    }  
}  
```