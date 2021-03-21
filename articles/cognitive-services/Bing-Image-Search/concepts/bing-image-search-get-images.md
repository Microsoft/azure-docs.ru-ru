---
title: Получение изображений из Интернета — API Bing для поиска изображений
titleSuffix: Azure Cognitive Services
description: Используйте API Bing для поиска изображений и получайте соответствующие изображения из Интернета.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: AB1B9898-C94A-4B59-91A1-8623C94BA3D4
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: conceptual
ms.date: 03/04/2019
ms.author: aahi
ms.openlocfilehash: 988a1332d03bf2c9563ab0576f7a20ee6b0615aa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96342062"
---
# <a name="get-images-from-the-web-with-the-bing-image-search-api"></a>Получение изображений из Интернета с помощью API Bing для поиска изображений

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

При использовании REST API Bing для поиска изображений можно получать изображения из Интернета, которые связаны с условием вашего поиска путем отправки следующего GET-запроса:

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE
X-MSEdge-ClientIP: 999.999.999.999
X-Search-Location: lat:47.60357;long:-122.3295;re:100
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
Host: api.cognitive.microsoft.com
```

Используйте параметр запроса [q](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#query) для выражения поиска, закодированного с помощью URL-адреса. Например, если вы вводите *sailing dinghies*, установите для параметра `q` значение `sailing+dinghies` или `sailing%20dinghies`.

> [!IMPORTANT]
> * Все запросы нужно выполнить на сервере, а не в клиенте.
> * Если вы впервые вызываете любой из поисковых API-интерфейсов Bing, не включайте заголовок идентификатора клиента. Включите идентификатор клиента только в том случае, если вы ранее вызывали API Bing, который возвратил идентификатор клиента для определенного сочетания пользователя и устройства.

## <a name="get-images-from-a-specific-web-domain"></a>Получение изображений из определенного веб-домена

Чтобы получить изображения с определенного домена, используйте оператор запроса [site:](/previous-versions/bing/search/ff795613(v=msdn.10)).

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies+site:contososailing.com&mkt=en-us HTTP/1.1
```

> [!NOTE]
> Ответы на запросы с помощью оператора `site:` могут относиться к содержимому для взрослых вне зависимости от параметров [SafeSearch](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#safesearch). Используйте `site:` только тогда, когда вам известно содержимое домена.

## <a name="filter-images"></a>Фильтрация изображений

 По умолчанию API для поиска изображений возвращает все изображения, относящиеся к запросу. Если необходимо использовать фильтры для изображений, которые возвращает Bing (например, только изображения с прозрачным фоном или определенного размера), используйте следующие параметры запроса:

* [аспект](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#aspect)— фильтрация изображений по пропорциям (например, к стандартным или широким экранным изображениям).
* [Цвет](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#color)— фильтрация изображений по основному или черно-белому цвету.
* [актуальность](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#freshness)— фильтрация изображений по возрасту (например, изображения, обнаруженные Bing за прошлую неделю).
* [Высота](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#height), [Ширина](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#width)— фильтрация изображений по ширине и высоте.
* [имажеконтент](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imagecontent)— фильтрация изображений по содержимому (например, изображения, отображающие только лицо человека).
* [imageType](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imagetype)— фильтрация изображений по типу (например, коллекции картинок, анимированных GIF-изображений или прозрачных фоновых рисунков).
* [Лицензия](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#license)— фильтрация изображений по типу лицензии, связанной с сайтом.
* [Размер](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#size)— фильтрация изображений по размеру, например мелким изображениям размером до 200x200 пикселей.

Чтобы получить изображения с определенного домена, используйте оператор запроса [site:](/previous-versions/bing/search/ff795613(v=msdn.10)).

В следующем примере показано, как получить небольшие изображения с сайта ContosoSailing.com, обнаруженные Bing на прошлой неделе.  

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=sailing+dinghies+site:contososailing.com&size=small&freshness=week&mkt=en-us HTTP/1.1  
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```

## <a name="bing-image-search-response-format"></a>Формат ответа для API Bing для поиска изображений

Ответное сообщение от Bing содержит [Изображения](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#images), список которых определен в Cognitive Services как соответствующий запросу. Каждый объект [Изображения](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#image) в списке содержит следующие сведения об изображении: URL-адрес, размер, измерения, формат кодирования, URL-адрес эскиза изображения, а также размеры эскизов.

> [!NOTE]
> * Изображения должны отображаться в порядке, указанном в ответе.
> * Так как формат и параметры URL-адресов могут измениться в любой момент без предварительного уведомления, используйте все URL-адреса как есть. Формат и параметры URL-адресов не следует учитывать, если это явно не обозначено.

```json
{
    "name": "Rich Passage Sailing Dinghy",
    "webSearchUrl": "https:\/\/www.bing.com\/cr?IG=73118C8B4E3...",
    "thumbnailUrl": "https:\/\/tse1.mm.bing.net\/th?id=OIP.GNarK7m...",
    "datePublished": "2011-10-29T11:26:00",
    "contentUrl": "http:\/\/www.bing.com\/cr?IG=73118C8B4E3D4C3...",
    "hostPageUrl": "http:\/\/www.bing.com\/cr?IG=73118C8B4E3D4C3687...",
    "contentSize": "79239 B",
    "encodingFormat": "jpeg",
    "hostPageDisplayUrl": "en.contoso.org\/wiki\/File:Rich_Passage...",
    "width": 526,
    "height": 688,
    "thumbnail": {
        "width": 229,
        "height": 300
    },
    "imageInsightsToken": "ccid_GNarK7ma*mid_CCF85447ADA6...",
    "insightsSourcesSummary": {
        "shoppingSourcesCount": 0,
        "recipeSourcesCount": 0
    },
    "imageId": "CCF85447ADA6FFF9E96E7DF0B796F7A86E34593",
    "accentColor": "376094"
},
```

Когда вы вызываете API Bing для поиска изображений, Bing возвращает список результатов. Список — это подмножество общего количества результатов, относящихся к запросу. Поле ответа `totalEstimatedMatches` содержит оценку количества изображений, доступных для просмотра. Подробнее о том, как просматривать оставшиеся изображения, см. в статье [Разбиение результатов по страницам](../../bing-web-search/paging-search-results.md).

## <a name="next-steps"></a>Дальнейшие действия

Если вы еще ни разу не использовали API Bing для поиска изображений, ознакомьтесь со следующей [статьей](../quickstarts/csharp.md). Если вы ищете что-нибудь посложнее, ознакомьтесь с руководством по созданию [одностраничного веб-приложения](../tutorial-bing-image-search-single-page-app.md).