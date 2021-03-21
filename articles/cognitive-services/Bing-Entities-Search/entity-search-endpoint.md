---
title: Конечная точка API Bing для поиска сущностей
titleSuffix: Azure Cognitive Services
description: API Bing для поиска сущностей использует одну конечную точку, которая возвращает сущности из Интернета на основе запроса. Результаты поиска возвращаются в формате JSON.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: conceptual
ms.date: 02/01/2019
ms.author: aahi
ms.openlocfilehash: 7fc65b27c3c02d6102210ae277690795d763d91a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96338263"
---
# <a name="bing-entity-search-api-endpoint"></a>Конечная точка API Bing для поиска сущностей

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).


API Bing для поиска сущностей использует одну конечную точку, которая возвращает сущности из Интернета на основе запроса. Результаты поиска возвращаются в формате JSON.

## <a name="get-entity-results-from-the-endpoint"></a>Получение результатов поиска сущностей из конечной точки

Чтобы получить результаты поиска сущностей с помощью **API Bing**, отправьте запрос `GET` на следующую конечную точку. Используйте [заголовки](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#headers) и [параметры запроса](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference#query-parameters) для настройки поискового запроса. Поисковые запросы можно отправлять с помощью параметра `?q=`.

```cURL
 GET https://api.cognitive.microsoft.com/bing/v7.0/entities
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Основные сведения об API Bing для поиска сущностей](overview.md)

## <a name="see-also"></a>См. также раздел 

Дополнительные сведения о заголовках, параметрах, кодах рынков, объектах ответов, ошибках и т. д. приведены в статье [Bing Entity Search API v7 reference](/rest/api/cognitiveservices-bingsearch/bing-entities-api-v7-reference) (Справочник по API Bing для поиска сущностей версии 7).