---
title: Отправка запросов к API Автозаполнения Bing
titleSuffix: Azure Cognitive Services
description: API "Автозаполнение Bing" возвращает список предлагаемых запросов на основе частично введенной строки запроса в поле поиска. Дополнительные сведения об отправке запросов.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-autosuggest
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: scottwhi
ms.openlocfilehash: dd845c0fb877afa76b84eb5c2d86392f763eccf7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96353397"
---
# <a name="sending-requests-to-the-bing-autosuggest-api"></a>Отправка запросов к API Автозаполнения Bing.

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Если ваше приложение отправляет запросы любому из API "Поиск Bing", вы можете использовать API "Автозаполнение Bing", чтобы улучшить работу пользователей с полем поиска. API "Автозаполнение Bing" возвращает список предлагаемых запросов на основе частично введенной строки запроса в поле поиска. Когда вы вводите символы в поле поиска в приложении, варианты поиска отображаются в раскрывающемся списке. Из этой статьи вы узнаете больше об отправке запросов к этому API. 

## <a name="bing-autosuggest-api-endpoint"></a>Конечная точка API Автозаполнения Bing

**API автозаполнения Bing** включает одну конечную точку, которая возвращает список предлагаемых запросов для частичного поискового запроса.

Чтобы получить предложенные запросы с помощью API Bing, отправьте запрос `GET` на указанную ниже конечную точку. Для определения дополнительных спецификаций используйте заголовки и параметры URL-адреса.

**Конечная точка:** Возвращает варианты поиска в виде результатов JSON, которые относятся к входным данным пользователя, определенным в `?q=""` .

```http
GET https://api.cognitive.microsoft.com/bing/v7.0/Suggestions 
```

Подробные сведения о заголовках, параметрах, кодах рынков, объектах ответов, ошибках и т. п. вы найдете в справочнике [по API автозаполнения Bing версии 7](/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference).

Интерфейсы API **Bing** поддерживают действия поиска, которые возвращают результаты определенного типа. Все конечные точки поиска возвращают результаты в виде объектов ответа JSON.
Все конечные точки поддерживают запросы, которые возвращают результаты с учетом языка и (или) местоположения по значениям долготы, широты и радиуса поиска.

Полные сведения о параметрах, поддерживаемых каждой конечной точкой, приведены в справочной документации по каждому типу.
Примеры основных запросов, созданных с помощью API автозаполнения Bing, см. в [этих кратких руководствах](/azure/cognitive-services/Bing-Autosuggest).

## <a name="bing-autosuggest-api-requests"></a>Запросы к API Автозаполнения Bing

> [!NOTE]
> * Для запросов к API Автозаполнения Bing необходимо использовать протокол HTTPS.

Рекомендуется, чтобы все запросы поступали с сервера. Распространение ключа в рамках клиентского приложения создает больше возможностей для доступа к нему злоумышленников. Кроме этого, осуществление вызовов с сервера также предоставляет единую точку обновления для будущих обновлений.

В запросе необходимо указать параметр [q](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#query), который содержит частичный поисковый запрос пользователя. В запросе можно также указать необязательный параметр [mkt](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#mkt), определяющий рынок, для которого будут отображаться результаты. Список необязательных параметров запроса можно найти в разделе [Параметры запроса](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#query-parameters). Все значения параметров запроса должны быть указаны в формате URL-адреса.

Запрос должен содержать заголовок [Ocp-Apim-Subscription-Key](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#subscriptionkey). Приведенные ниже заголовки являются необязательными, но их также рекомендуется указать:

- [Агент пользователя](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#useragent)
- [X-MSEdge-ClientID](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#clientid)
- [X-Search-ClientIP](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#clientip)
- [X-Search-расположение](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#location)

Заголовки IP-адреса и расположения клиента важны для отображения содержимого с учетом расположения.

Список всех заголовков в запросах и ответах приведен в разделе [Заголовки](/rest/api/cognitiveservices/bing-autosuggest-api-v5-reference#headers).

> [!NOTE]
> При вызове API Автозаполнения Bing из JavaScript встроенные средства обеспечения безопасности браузера могут блокировать доступ к значениям этих заголовков.

Для устранения этой проблемы запрос API автозаполнения Bing можно выполнить через прокси-сервер CORS. Ответ такого прокси-сервера имеет `Access-Control-Expose-Headers` заголовок, который фильтрует заголовки ответа и делает их доступными для JavaScript.

Установить прокси-сервер CORS довольно просто. Это позволит нашему [учебному приложению](../tutorials/autosuggest.md) иметь доступ к необязательным заголовкам клиента. Для начала [установите платформу Node.js](https://nodejs.org/en/download/), если она еще не установлена. В командной строке введите следующую команду:

```console
npm install -g cors-proxy-server
```

Затем в HTML-файле измените конечную точку API автозаполнения Bing на следующую:

```http
http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/Suggestions
```

И наконец, запустите прокси-сервер CORS с помощью следующей команды:

```console
cors-proxy-server
```

Не закрывайте командное окно, пока используете учебное приложение. Это приведет к остановке прокси-сервера. Теперь в развертываемом разделе заголовков HTTP под результатами поиска можно увидеть заголовок `X-MSEdge-ClientID` (среди прочих) и убедиться, что он одинаковый для всех запросов.

Запрос должен содержать все параметры и заголовки для предложений запроса. 

В следующем примере показано, как отправить запрос, который возвращает предлагаемые строки запроса для слова *sail*.

> ```http
> GET https://api.cognitive.microsoft.com/bing/v7.0/suggestions?q=sail&mkt=en-us HTTP/1.1
> Ocp-Apim-Subscription-Key: 123456789ABCDE
> X-MSEdge-ClientIP: 999.999.999.999
> X-Search-Location: lat:47.60357;long:-122.3295;re:100
> X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>
> Host: api.cognitive.microsoft.com
> ```

Если вы впервые вызовете любой из интерфейсов API Bing, не включайте заголовок идентификатора клиента. Передавайте заголовок с идентификатором клиента только в том случае, если вы уже вызывали API Bing и получили от него идентификатор клиента для конкретного сочетания пользователя и устройства.

Следующая группа веб-предложений является ответом на запрос выше. Группа содержит список предложений запроса поиска, а каждое предложение включает поле `displayText`, `query` и `url`.

Поле `displayText` содержит предлагаемые запросы, которые вы можете поместить в раскрывающийся список для поля поиска. Должны отображаться все предоставленные предложения и строго в указанном порядке.  

Если пользователь выбирает запрос из раскрывающегося списка, вы можете использовать его для вызова одного из [API поиска Bing](../../bing-web-search/bing-api-comparison.md?bc=%2fen-us%2fazure%2fbread%2ftoc.json&toc=%2fen-us%2fazure%2fcognitive-services%2fbing-autosuggest%2ftoc.json) и самостоятельного отображения результатов или направить пользователя на страницу результатов Bing с помощью возвращенного поля `url`.

[!INCLUDE [cognitive-services-bing-url-note](../../../../includes/cognitive-services-bing-url-note.md)]

```json
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{
    "_type" : "Suggestions",
    "queryContext" : {
        "originalQuery" : "sail"
    },
    "suggestionGroups" : [{
        "name" : "Web",
        "searchSuggestions" : [{
            "url" : "https:\/\/www.bing.com\/search?q=sailing+lessons+seattle&FORM=USBAPI",
            "displayText" : "sailing lessons seattle",
            "query" : "sailing lessons seattle",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailor+moon+news&FORM=USBAPI",
            "displayText" : "sailor moon news",
            "query" : "sailor moon news",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailor+jack%27s+lincoln+city&FORM=USBAPI",
            "displayText" : "sailor jack's lincoln city",
            "query" : "sailor jack's lincoln city",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailing+anarchy&FORM=USBAPI",
            "displayText" : "sailing anarchy",
            "query" : "sailing anarchy",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailboats+for+sale&FORM=USBAPI",
            "displayText" : "sailboats for sale",
            "query" : "sailboats for sale",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailstn.mylabsplus.com&FORM=USBAPI",
            "displayText" : "sailstn.mylabsplus.com",
            "query" : "sailstn.mylabsplus.com",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailusfood&FORM=USBAPI",
            "displayText" : "sailusfood",
            "query" : "sailusfood",
            "searchKind" : "WebSearch"
        },
        {
            "url" : "https:\/\/www.bing.com\/search?q=sailboats+for+sale+seattle&FORM=USBAPI",
            "displayText" : "sailboats for sale seattle",
            "query" : "sailboats for sale seattle",
            "searchKind" : "WebSearch"
        }]
    }]
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Что такое API автозаполнения Bing?](../get-suggested-search-terms.md)
- [Справочник по API автозаполнения Bing версии 7](/rest/api/cognitiveservices-bingsearch/bing-autosuggest-api-v7-reference)
- [Предложение терминов запроса](get-suggestions.md)