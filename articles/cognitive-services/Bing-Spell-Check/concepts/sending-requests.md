---
title: Отправка запросов к API Проверки орфографии Bing
titleSuffix: Azure Cognitive Services
description: В этой статье описаны режимы проверки орфографии Bing, параметры и другие сведения, относящиеся к API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-spell-check
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: aahi
ms.openlocfilehash: 761cc3908b677b129e85be442b7a593a38a367f9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96341569"
---
# <a name="sending-requests-to-the-bing-spell-check-api"></a>Отправка запросов к API Проверки орфографии Bing

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Чтобы проверить текстовую строку на наличие орфографических и грамматических ошибок, необходимо отправить запрос GET на следующую конечную точку:  

```
https://api.cognitive.microsoft.com/bing/v7.0/spellcheck
```  
  
Для отправки запроса следует использовать протокол HTTPS.

Рекомендуется, чтобы все запросы поступали с сервера. Распространение ключа в рамках клиентского приложения создает больше возможностей для доступа к нему злоумышленников. Сервер также предоставляет единую точку обновления для будущих версий API.

В запросе необходимо указать параметр [text](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#text), который содержит текстовую строку для проверки. В запросе можно также указать необязательный параметр [mkt](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#mkt), определяющий рынок, для которого будут отображаться результаты. Список необязательных параметров запроса, таких как `mode`, можно найти в разделе [Параметры запроса](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#query-parameters). Все значения параметров запроса должны быть указаны в формате URL-адреса.  
  
Запрос должен содержать заголовок [Ocp-Apim-Subscription-Key](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#subscriptionkey). Приведенные ниже заголовки являются необязательными, но их также рекомендуется указать. Эти заголовки позволяют API "Проверка орфографии Bing" возвращать более точные результаты:  
  
-   [Агент пользователя](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#useragent)  
-   [X-MSEdge-ClientID](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#clientid)  
-   [X-Search-ClientIP](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#clientip)  
-   [X-Search-расположение](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#location)  

Список всех заголовков в запросах и ответах приведен в разделе [Заголовки](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v5-reference#headers).

При вызове API "Проверка орфографии Bing" с помощью JavaScript встроенные средства обеспечения безопасности браузера могут блокировать доступ к значениям этих заголовков.

Для решения этой проблемы запрос API Bing для проверки орфографии можно выполнить через прокси-сервер CORS. Ответ такого прокси-сервера имеет `Access-Control-Expose-Headers` заголовок, который фильтрует заголовки ответа и делает их доступными для JavaScript.

Установить прокси-сервер CORS довольно просто. Это позволит [учебному приложению](../tutorials/spellcheck.md) получить доступ к необязательным заголовкам клиента. Для начала [установите платформу Node.js](https://nodejs.org/en/download/), если она еще не установлена. В командной строке введите следующую команду:

```console
npm install -g cors-proxy-server
```

Затем измените конечную точку API Bing для проверки орфографии в HTML-файле на: \
`http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/spellcheck/`

И наконец, запустите прокси-сервер CORS с помощью следующей команды:

```console
cors-proxy-server
```

Не закрывайте командное окно, пока используете учебное приложение. Это приведет к остановке прокси-сервера. Теперь в развертываемом разделе заголовков HTTP под результатами поиска можно увидеть заголовок `X-MSEdge-ClientID` (среди прочих) и убедиться, что он одинаковый для всех запросов.

## <a name="example-api-request"></a>Пример запроса к API

Ниже показан запрос, который включает в себя все рекомендуемые параметры и заголовки запроса. Если вы впервые вызовете любой из интерфейсов API Bing, не включайте заголовок идентификатора клиента. Включите идентификатор клиента, только если вы ранее вызывали API Bing, а Bing возвратил идентификатор клиента для комбинации пользователей и устройств. 
  
```http
GET https://api.cognitive.microsoft.com/bing/v7.0/spellcheck?text=when+its+your+turn+turn,+john,+come+runing&mkt=en-us HTTP/1.1
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com  
```

Ниже показан ответ на предыдущий запрос. В примере также показаны заголовки ответа для Bing.

[!INCLUDE [cognitive-services-bing-url-note](../../../../includes/cognitive-services-bing-url-note.md)]

```json
BingAPIs-TraceId: 76DD2C2549B94F9FB55B4BD6FEB6AC
X-MSEdge-ClientID: 1C3352B306E669780D58D607B96869
BingAPIs-Market: en-US

{  
    "_type" : "SpellCheck",  
    "flaggedTokens" : [{  
        "offset" : 5,  
        "token" : "its",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "it's",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 25,  
        "token" : "john",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "John",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 19,  
        "token" : "turn",  
        "type" : "RepeatedToken",  
        "suggestions" : [{  
            "suggestion" : "",  
            "score" : 1  
        }]  
    },  
    {  
        "offset" : 35,  
        "token" : "runing",  
        "type" : "UnknownToken",  
        "suggestions" : [{  
            "suggestion" : "running",  
            "score" : 1  
        }]  
    }]  
}  
```  

## <a name="next-steps"></a>Дальнейшие действия

- [Что такое API проверки орфографии Bing?](../overview.md)
- [Справочник по API Проверки орфографии Bing версии 7](/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference)