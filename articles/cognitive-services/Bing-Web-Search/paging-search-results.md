---
title: Просмотр результатов поиска с помощью API-интерфейсы поиска Bing
titleSuffix: Azure Cognitive Services
description: Сведения о просмотре результатов поиска из API-интерфейсы поиска Bing.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: 26CA595B-0866-43E8-93A2-F2B5E09D1F3B
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 10/31/2019
ms.author: aahi
ms.openlocfilehash: 670460759a9495de735da35ae9f3d8388e59e0e5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96350625"
---
# <a name="how-to-page-through-results-from-the-bing-search-apis"></a>Как пролистывать результаты из API-интерфейсы поиска Bing

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

При отправке вызова в API Bing для веб-приложений, пользовательских интерфейсов, изображений, новостей или Поиск видеоов Bing возвращает подмножество общего количества результатов, которые могут быть актуальны для запроса. Чтобы получить оценочное общее число доступных результатов, получите доступ к полю объекта ответа `totalEstimatedMatches` . 

Пример: 

```json
{
    "_type" : "SearchResponse",
    "webPages" : {
        "webSearchUrl" : "https:\/\/www.bing.com\/cr?IG=3A43CA...",
        "totalEstimatedMatches" : 262000,
        "value" : [...]
    }
}  
```

## <a name="paging-through-search-results"></a>Разбиение по страницам результатов поиска

Чтобы просмотреть доступные результаты, используйте `count` `offset` Параметры запроса и при отправке запроса.  

> [!NOTE]
>
> * Подкачка интерфейсов API Bing Video, Image и News применяется только к основным операциям `/video/search` поиска видео (), новостей ( `/news/search` ) и Image ( `/image/search` ). Разбиение по страницам разделов и категорий тенденций не поддерживается.  
> * `TotalEstimatedMatches`Поле представляет собой оценку общего количества результатов поиска для текущего запроса. При задании `count` параметров и `offset` Эта оценка может измениться.

| Параметр | Описание                                                                                                                                                                |
|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `count`   | Задает количество результатов, возвращаемых в ответе. Обратите внимание, что значение по умолчанию `count` и максимальное количество результатов, которые можно запросить, зависят от API. Эти значения можно найти в справочной документации в разделе [дальнейшие действия](#next-steps). |
| `offset`  | Задает количество пропускаемых результатов. Значение `offset` начинается с нуля и должно быть меньше (`totalEstimatedMatches` - `count`).                                           |

Например, если вы хотите отобразить 15 результатов на странице, для `count` получения первой страницы результатов необходимо задать значение 15 и значение `offset` 0. Для каждого последующего вызова API значение будет увеличено `offset` на 15. В следующем примере запрашиваются 15 веб-страниц с начальным смещением в 45 результатов.

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&count=15&offset=45&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```

Если используется значение по умолчанию `count` , необходимо только указать `offset` параметр запроса в вызовах API.  

```  
GET https://api.cognitive.microsoft.com/bing/v7.0/search?q=sailing+dinghies&offset=45&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
Host: api.cognitive.microsoft.com  
```

При использовании API-интерфейсов изображений и видео Bing можно использовать `nextOffset` значение, чтобы избежать дублирования результатов поиска. Получите значение из `Images` `Videos` объектов ответов или и используйте его в запросах с `offset` параметром.  

> [!NOTE]
> API Bing для поиска в Интернете возвращает результаты поиска, которые могут включать веб-страницы, изображения, видео и новости. При просмотре результатов поиска из API Bing для поиска в Интернете отображаются только [веб-страницы](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#webpage), а не другие типы ответов, такие как изображения или Новости. Результаты поиска в `WebPage` объектах могут содержать результаты, которые также отображаются в других типах ответов.
>
> Если вы используете `responseFilter` параметр запроса без указания каких бы то ни было значений фильтра, не используйте `count` `offset` Параметры и. 

## <a name="next-steps"></a>Дальнейшие действия

* [Что такое Поиск в Интернете Bing API?](bing-api-comparison.md)
* [Справка по API Bing для поиска в Интернете версии 7](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference)
* [Справочник по API пользовательского поиска Bing версии 7](/rest/api/cognitiveservices-bingsearch/bing-custom-search-api-v7-reference)
* [Справочник по API Bing для поиска новостей версии 7](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference)
* [Справочник по API Bing для поиска видео версии 7](/rest/api/cognitiveservices-bingsearch/bing-video-api-v7-reference)
* [Справочник по API Bing для поиска изображений версии 7](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference)