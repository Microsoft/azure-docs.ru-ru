---
title: Отправка и использование API-запросов и ответов. локальный поиск в Bing для бизнеса
titleSuffix: Azure Cognitive Services
description: В этой статье вы узнаете, как с помощью API Bing Local Business Search отправлять и использовать запросы поиска.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-local-business
ms.topic: conceptual
ms.date: 06/26/2018
ms.author: rosh
ms.openlocfilehash: 70a33774ac82312660d887fb86f7e2a482c30a0c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96487173"
---
# <a name="sending-and-using-bing-local-business-search-api-queries-and-responses"></a>Отправка и использование запросов и ответов API Bing Local Business Search

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Локальные результаты можно получить из API Bing Local Business Search, отправляя запрос поиска на конечную точку и включая обязательный заголовок `Ocp-Apim-Subscription-Key`. Вместе с доступными [заголовками](local-search-reference.md#headers) и [параметрами](local-search-reference.md#query-parameters) можно настроить Поиск Azure, указав [географические границы](specify-geographic-search.md) для поиска области и [категории](local-search-query-response.md) возвращаемых знаков.

## <a name="creating-a-request"></a>Создание запроса

Чтобы отправить запрос к API Bing Local Business Search, добавьте условие поиска для параметра `q=`, прежде чем добавлять его в конечную точку API и включать в заголовок `Ocp-Apim-Subscription-Key`. Пример:

`https://api.cognitive.microsoft.com/bing/localbusinesses/v7.0/search?q=restaurant+in+Bellevue`

Ниже приведен синтаксис полного запроса URL-адреса. Дополнительные сведения об отправке запросов см. в [Quickstart: Send a query to the Bing Local Business Search API in C#](quickstarts/local-quickstart.md) (Краткое руководство. Отправка запроса к API Bing Local Business Search) и справочные материалы в разделах [Заголовки](local-search-reference.md#headers) и [Параметры](local-search-reference.md#query-parameters). 

Сведения о категории локального поиска см. в статье [Поиск категорий для API Bing Local Business Search](local-categories.md).

```
https://api.cognitive.microsoft.com/bing/v7.0/localbusinesses/search[?q][&localCategories][&cc][&mkt][&safesearch][&setlang][&count][&first][&localCircularView][&localMapView]
```

## <a name="using-responses"></a>Использование ответов

Ответы JSON из API Bing Local Business Search содержат объект `SearchResponse`. API вернет релевантные результаты поиска в поле `places`. Если результаты не найдены, поле `places` не будет включено в ответ.

[!INCLUDE [cognitive-services-bing-url-note](../../../includes/cognitive-services-bing-url-note.md)]

```
{
   "_type": "SearchResponse",
   "queryContext": {
      "originalQuery": "restaurant in Bellevue"
   },
   "places": {
      "totalEstimatedMatches": 10,
. . . 
```

### <a name="search-result-attributes"></a>Поиск атрибутов результатов

Результаты JSON, возвращенные API, включают в себя следующие атрибуты:

* _type
* address
* entityPresentationInfo
* геообъект
* идентификатор
* name
* routeablePoint
* telephone
* url

Дополнительные сведения о заголовках, параметрах, кодах рынков, объектах ответов, ошибках и т. д. см. в статье [Справочник по API Bing Local Business Search версии 7](local-search-reference.md).

> [!NOTE]
> Вы или третья сторона, действующая от вашего имени, можете не использовать, хранить, кэшировать, распределять данные из API локального поиска, чтобы предоставить доступ к ним с целью тестирования, разработки, обучения, распределения или предоставления доступа любой сторонней службе или функции (не от корпорации Майкрософт). 


## <a name="example-json-response"></a>Пример ответа в формате JSON

Следующий ответ JSON включает результаты поиска, указанные в запросе `?q=restaurant+in+Bellevue`.

```json
Vary: Accept-Encoding
BingAPIs-TraceId: 5376FFEB65294E24BB9F91AD70545826
BingAPIs-SessionId: 06ED7CEC80F746AA892EDAAC97CB0CB4
X-MSEdge-ClientID: 112C391E72C0624204153594738C63DE
X-MSAPI-UserState: aeab
BingAPIs-Market: en-US
X-Search-ResponseInfo: InternalResponseTime=659,MSDatacenter=CO4
X-MSEdge-Ref: Ref A: 5376FFEB65294E24BB9F91AD70545826 Ref B: BY3EDGE0306 Ref C: 2018-10-16T16:26:15Z
apim-request-id: fe54f585-7c54-4bf5-8b92-b9bede2b710a
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
x-content-type-options: nosniff
Cache-Control: max-age=0, private
Date: Tue, 16 Oct 2018 16:26:15 GMT
P3P: CP="NON UNI COM NAV STA LOC CURa DEVa PSAa PSDa OUR IND"
Content-Length: 978
Content-Type: application/json; charset=utf-8
Expires: Tue, 16 Oct 2018 16:25:15 GMT

{
  "_type": "SearchResponse",
  "queryContext": {
    "originalQuery": "restaurant Bellevue"
  },
  "places": {
    "totalEstimatedMatches": 50,
    "value": [{
      "_type": "LocalBusiness",
      "id": "https:\/\/cognitivegblppe.azure-api.net\/api\/v7\/#Places.0",
      "name": "Facing East Taiwanese Restaurant",
      "url": "http:\/\/litadesign.wix.com\/facingeastrestaurant",
      "entityPresentationInfo": {
        "entityScenario": "ListItem",
        "entityTypeHints": ["Place", "LocalBusiness", "Restaurant"]
      },
      "geo": {
        "latitude": 47.6199188232422,
        "longitude": -122.202796936035
      },
      "routablePoint": {
        "latitude": 47.6199188232422,
        "longitude": -122.201713562012
      },
      "address": {
        "streetAddress": "1075 Bellevue Way NE Ste B2",
        "addressLocality": "Bellevue",
        "addressRegion": "WA",
        "postalCode": "98004",
        "addressCountry": "US",
        "neighborhood": "Bellevue",
        "text": "1075 Bellevue Way NE Ste B2, Bellevue, WA 98004"
      },
      "telephone": "(425) 688-2986"
    }],
    "searchAction": {
      "location": [{
        "name": "Bellevue, Washington"
      }],
      "query": "restaurant"
    }
  }
}
 
```

## <a name="throttling-requests"></a>Запросы на регулирование

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../includes/cognitive-services-bing-throttling-requests.md)]


## <a name="next-steps"></a>Дальнейшие действия
- [Quickstart: Send a query to the Bing Local Business Search API in C#](quickstarts/local-quickstart.md) (Краткое руководство. Отправка запроса в API Bing Local Business Search с помощью C#)
- [Краткое руководство. Отправка запроса в API Bing для поиска местных компаний с помощью Java](quickstarts/local-search-java-quickstart.md)
- [Краткое руководство. Использование Local Business Search с помощью Node](quickstarts/local-search-node-quickstart.md)
- [Краткое руководство. Отправка запроса в API Bing Local Business Search с помощью Python](quickstarts/local-search-python-quickstart.md)