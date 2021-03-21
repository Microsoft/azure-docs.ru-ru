---
title: Поиск новостей с помощью API Bing для поиска новостей
titleSuffix: Azure Cognitive Services
description: Узнайте, как отправлять запросы для поиска общих новостей, а также наиболее популярных тем и заголовков.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: conceptual
ms.date: 12/18/2019
ms.author: scottwhi
ms.openlocfilehash: ecf01720126a9bf5da4aabb08653e62b42265d36
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96351356"
---
# <a name="search-for-news-with-the-bing-news-search-api"></a>Поиск новостей с помощью API Bing для поиска новостей

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

API Bing для поиска изображений позволяет легко интегрировать возможности когнитивного поиска новости Bing в приложения.

API Bing для поиска новостей в основном находит и возвращает соответствующие статьи, а также предоставляет несколько функций для интеллектуального и тематического поиска новостей в Интернете.

## <a name="suggest-and-use-search-terms"></a>Использование и предложение терминов для поиска

Если вы предоставили окно поиска, в котором пользователь вводит свой поисковый запрос, используйте [API автозаполнения Bing](../../bing-autosuggest/get-suggested-search-terms.md), чтобы оптимизировать работу. API возвращает предложенные строки запроса на основе частичного поиска, как пользовательские типы.

После ввода условия поиска URL-адрес закодирует его перед установкой параметра запроса [q](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#query). Например, если пользователь вводит *парусные шлюпки*, установите для параметра `q` значение `sailing+dinghies` или `sailing%20dinghies`.

## <a name="get-general-news"></a>Получение общих новостей

Чтобы получить общие новостные статьи, связанные с условием поиска пользователя, из Интернета, отправьте следующий запрос GET.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/news/search?q=sailing+dinghies&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-Search-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

Если вы впервые вызовете любой из интерфейсов API Bing, не включайте заголовок идентификатора клиента. Включите идентификатор клиента, только если вы ранее вызывали API Bing, а Bing возвратил идентификатор клиента для комбинации пользователей и устройств.

Чтобы получить новости с определенного домена, используйте оператор запроса [site:](/previous-versions/bing/search/ff795613(v=msdn.10)).

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/news/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us HTTP/1.1
```

В следующем примере JSON показан ответ на предыдущий запрос. В соответствии с [требованиями к использованию и отображению](../../bing-web-search/use-display-requirements.md) для API поиска Bing все новостные статьи должны отображаться в порядке, указанном в ответе. Если статья содержит сгруппированные статьи, необходимо указать, что существуют связанные статьи, и отображать их по запросу.

```json
{
    "_type" : "News",
    "readLink" : "https:\/\/api.cognitive.microsoft.com\/bing\/v5\/news\/search?q=sailing+dinghies",
    "totalEstimatedMatches" : 88400,
    "value" : [{
        "name" : "Sailing Vies for Four Trophies",
        "url" : "http:\/\/www.bing.com\/cr?IG=CCE2F06CA750455891FE99A72...",
        "image" : {
            "thumbnail" : {
                "contentUrl" : "https:\/\/www.bing.com\/th?id=ON.9C23AA5...",
                "width" : 650,
                "height" : 341
            }
        },
        "description" : "College Rankings, presented by Zim...",
        "provider" : [{
            "_type" : "Organization",
            "name" : "contoso.com"
        }],
        "datePublished" : "2017-04-14T15:28:00"
    },

    ...

    {
        "name" : "Fabrikam Sailing Club to host Mirror Dinghy...",
        "url" : "http:\/\/www.bing.com\/cr?IG=CCE2F06CA750455891F...",
        "image" : {
            "thumbnail" : {
                "contentUrl" : "https:\/\/www.bing.com\/th?id=ON.36...",
                "width" : 448,
                "height" : 300
            }
        },
        "description" : "The sailing club that trained Olympian Ben...",
        "provider" : [{
            "_type" : "Organization",
            "name" : "Contoso"
        }],
        "datePublished" : "2017-04-04T11:02:00",
        "category" : "Sports"
    }]
}
```

В ответе [Новости](/rest/api/cognitiveservices-bingsearch/bing-news-api-v5-reference#news) перечислены новостные статьи, которые система Bing посчитала соответствующими запросу. Поле `totalEstimatedMatches` содержит оценку количества статей, доступных для просмотра. Сведения о разбиении статей на страницы см. в разделе [Разбиение новостей по страницам](../../bing-web-search/paging-search-results.md).

Каждый объект [новостной статьи](/rest/api/cognitiveservices-bingsearch/bing-news-api-v5-reference#newsarticle) в списке содержит название, описание и URL-адрес статьи на исходном веб-сайте. Если статья содержит изображение, то объект включает эскиз этого изображения. Используйте свойства `name` и `url`, чтобы создать гиперссылку, направляющую пользователя к новостной статье на исходном сайте. Если статья содержит изображение, также сделайте его интерактивным с помощью свойства `url`. Обязательно используйте свойство `provider`, чтобы указать источник статьи.

Если Bing может определить категорию новостной статьи, значит в ней содержится поле `category`.

## <a name="get-todays-top-news"></a>Получение главных новостей за сегодня

Чтобы получить главные новостные статьи за сегодня, можно отправить тот же общий запрос новостей что и раньше, не задавая параметр `q`.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/news/search?q=&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-Search-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

Ответ на получение главных новостей почти такой же, как и на получение общих новостей. Однако ответ `news` не включает поле `totalEstimatedMatches`, потому что есть определенное количество результатов. Количество главных новостных статей может различаться в зависимости от цикла новостей. Обязательно используйте поле `provider`, чтобы указать источник статьи.

## <a name="get-news-by-category"></a>Получение новостей по категориям

Чтобы получать новости по категориям, например лучшие спортивные или развлекательные статьи, отправьте следующий запрос GET в Bing.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/news?category=sports&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-Search-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

Используйте параметр запроса [Категория](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#category), чтобы указать категорию статей для получения. Список возможных категорий новостей, которые можно указать, см. в разделе [News Categories by Market](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#news-categories-by-market) (Категории новостей по рынку).

Ответ на получение новостей по категориям почти такой же, как и на получение общих новостей. Однако все статьи принадлежат к указанной категории.

## <a name="get-headline-news"></a>Получение краткой сводки новостей

Чтобы запрашивать краткую сводку новостей и получать статьи из всех категорий, отправьте следующий запрос GET в Bing.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/news?mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-MSEdge-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

Не включайте параметр запроса [Категория](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#category).

Ответ на получение краткой сводки новостей почти такой же, как и на получение главных новостей за сегодняшний день. Если статья является новостной сводкой, полю `headline` присваивается значение **true**.

По умолчанию ответ включает до 12 новостных сводок. Чтобы изменить количество новостных сводок для возврата, укажите параметр запроса [headlineCount](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#headlinecount). Ответ также включает до четырех информационных статей по каждой категории новостей.

Ответ считает сгруппированные статьи как одну. Поскольку в группе несколько статей, ответ может включать более 12 новостных сводок и более четырех информационных статей по каждой категории.

## <a name="get-trending-news"></a>Получение популярных новостей

Чтобы получить темы новостей, популярные в социальных сетях, отправьте следующий запрос GET в Bing.

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/news/trendingtopics?mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)
X-Search-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
X-MSAPI-UserState: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

> [!NOTE]
> Популярные темы доступны только на рынках en-US и zh-CN.

Следующий JSON — это ответ на предыдущий запрос. Каждая популярная новостная статья включает связанное изображение, пометку "последние новости" и URL-адрес результатов поиска Bing для статьи. Используйте URL-адрес из поля `webSearchUrl`, чтобы переадресовать пользователя на страницу результатов поиска Bing. Или используйте текст запроса для вызова [API для Поиска в Интернете](../../bing-web-search/overview.md), чтобы отобразить результаты самостоятельно.

```json
{
    "_type" : "TrendingTopics",
    "value" : [{
        "name" : "Canada pot measure",
        "image" : {
            "url" : "https:\/\/www.bing.com\/th?id=OPN.RTNews_hHD...",
            "provider" : [{
                "_type" : "Organization",
                "name" : "Contoso Images"
            }]
        },
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=070292D8CEDD...",
        "isBreakingNews" : false,
        "query" : {
            "text" : "Canada marijuana"
        }
    },
    {
        "name" : "Down on Vegas move",
        "image" : {
            "url" : "https:\/\/www.bing.com\/th?id=OPN.RTNews_Bfbmg8h...",
            "provider" : [{
                "_type" : "Organization",
                "name" : "Contoso"
            }]
        },
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=070292D8CEDD...",
        "isBreakingNews" : false,
        "query" : {
            "text" : "Marcus Appel Las Vegas"
        }
    },

    ...

    ]
}
```

## <a name="getting-related-news"></a>Получение похожих новостей

Если есть другие статьи, связанные с новостной статьей, то она может включать в себя поле [clusteredArticles](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle-clusteredarticles). Ниже показаны сгруппированные статьи.

```json
    {
        "name" : "Playoffs 2017: Betting lines, point spreads...",
        "url" : "http:\/\/www.bing.com\/cr?IG=4B7056CEC271408997D115...",
        "image" : {
            "thumbnail" : {
                "contentUrl" : "https:\/\/www.bing.com\/th?id=ON.D7B1...",
                "width" : 700,
                "height" : 393
            }
        },
        "description" : "April 14, 2017 3:37pm EDT April 14, 2017 3:34pm...",
        "provider" : [{
            "_type" : "Organization",
            "name" : "Contoso"
        }],
        "datePublished" : "2017-04-14T19:43:00",
        "category" : "Sports",
        "clusteredArticles" : [{
            "name" : "Playoffs 2017: Betting odds, favorites to win...",
            "url" : "http:\/\/www.bing.com\/cr?IG=4B7056CEC271408997D1159E...",
            "description" : "April 14, 2017 3:30pm EDT April 14, 2017 3:27pm...",
            "provider" : [{
                "_type" : "Organization",
                "name" : "Contoso"
            }],
            "datePublished" : "2017-04-14T19:37:00",
            "category" : "Sports"
        }]
    },
```

## <a name="throttling-requests"></a>Запросы на регулирование

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Разбивка на страницы результатов, возвращаемых Поиском новостей Bing](../../bing-web-search/paging-search-results.md)