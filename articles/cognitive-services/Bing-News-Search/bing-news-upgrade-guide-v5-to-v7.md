---
title: Обновление API Bing для поиска новостей с версии 5 до версии 7
titleSuffix: Azure Cognitive Services
description: Определяет части приложения Поиск новостей Bing, которые необходимо обновить для использования версии 7.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: conceptual
ms.date: 01/10/2019
ms.author: scottwhi
ms.openlocfilehash: a114cb24d79189f9e370fae1962f60ca97241d90
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96351373"
---
# <a name="news-search-api-upgrade-guide"></a>Руководство по обновлению API для поиска новостей

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

В этом руководстве по обновлению определены изменения между версиями 5 и 7 API Bing для поиска новостей. Руководство поможет определить компоненты приложения, которые необходимо обновить для использования версии 7.

## <a name="breaking-changes"></a>Критические изменения

### <a name="endpoints"></a>Конечные точки

- Номер версии конечной точки изменен с 5 на 7. Например, `https://api.cognitive.microsoft.com/bing/v7.0/news/search`.

### <a name="error-response-objects-and-error-codes"></a>Объекты ответов на ошибки и коды ошибок

- Все невыполненные запросы теперь должны включать в тело ответа объект `ErrorResponse`.

- В объект `Error` добавлены следующие поля:  
  - `subCode`&mdash;Разделяет код ошибки на дискретные сегменты, если это возможно
  - `moreDetails`&mdash;Дополнительные сведения об ошибке, описанной в поле `message`

- Коды ошибок версии 5 заменены следующими возможными значениями `code` и `subCode`.

|Код|SubCode (дополнительный код)|Описание
|-|-|-
|ServerError|UnexpectedError<br/>ResourceError<br/>NotImplemented|Bing возвращает ServerError (ошибку сервера) каждый раз при возникновении любого из условий вложенного кода. Ответ включает в себя ошибки, если код состояния HTTP — 500.
|InvalidRequest|ParameterMissing<br/>ParameterInvalidValue<br/>HttpNotAllowed<br/>Блокировано|Bing возвращает ошибку InvalidRequest (недопустимый запрос) всякий раз, когда любая часть запроса недопустима. Например, отсутствует обязательный параметр или значение параметра недопустимо.<br/><br/>В случае ошибки ParameterMissing или ParameterInvalidValue возвращается код состояния HTTP 400.<br/><br/>При ошибке HttpNotAllowed (HTTP запрещен) будет наблюдаться код состояния HTTP 410.
|RateLimitExceeded||Bing возвращает ошибку RateLimitExceeded всякий раз при превышении квоты запросов в секунду (QPS) или запросов в месяц (QPM).<br/><br/>Bing возвращает код состояния HTTP 429 при превышении квоты QPS и 403 при превышении QPM.
|InvalidAuthorization|AuthorizationMissing<br/>AuthorizationRedundancy|Bing возвращает InvalidAuthorization, когда Bing не может проверить подлинность вызывающего объекта. Например, когда заголовок `Ocp-Apim-Subscription-Key` отсутствует или при недопустимом ключе подписки.<br/><br/>Избыточность возникает, если указать более одного способа проверки подлинности.<br/><br/>При ошибке InvalidAuthorization кодом состояния HTTP будет 401.
|InsufficientAuthorization|AuthorizationDisabled<br/>AuthorizationExpired|Bing возвращает InsufficientAuthorization, когда вызывающая сторона не имеет разрешений на доступ к ресурсу. Это может произойти, если ключ подписки отключен или срок его действия истек. <br/><br/>При ошибке InsufficientAuthorization возвращается код состояния HTTP 403.

- Ниже представлено сопоставление предыдущих кодов ошибок с новыми. При наличии зависимостей от кодов ошибок версии 5 обновите свой код соответствующим образом.

|Код версии 5|Код версии 7.subCode
|-|-
|RequestParameterMissing|InvalidRequest.ParameterMissing
RequestParameterInvalidValue|InvalidRequest.ParameterInvalidValue
ResourceAccessDenied|InsufficientAuthorization
ExceededVolume|RateLimitExceeded
ExceededQpsLimit|RateLimitExceeded
Выключено|InsufficientAuthorization.AuthorizationDisabled
UnexpectedError|ServerError.UnexpectedError
DataSourceErrors|ServerError.ResourceError
AuthorizationMissing|InvalidAuthorization.AuthorizationMissing
HttpNotAllowed|InvalidRequest.HttpNotAllowed
UserAgentMissing|InvalidRequest.ParameterMissing
NotImplemented|ServerError.NotImplemented
InvalidAuthorization|InvalidAuthorization
InvalidAuthorizationMethod|InvalidAuthorization
MultipleAuthorizationMethod|InvalidAuthorization.AuthorizationRedundancy
ExpiredAuthorizationToken|InsufficientAuthorization.AuthorizationExpired
InsufficientScope|InsufficientAuthorization
Блокировано|InvalidRequest.Blocked

### <a name="object-changes"></a>Изменения объектов

- В объект [NewsArticle](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle) добавлено поле `contractualRules`. Поле `contractualRules` содержит список правил, которым необходимо следовать (например, определение принадлежности статьи). Необходимо применить правила принадлежности, указанные в поле `contractualRules`, а не в `provider`. Статья включает поле `contractualRules`, только если в ответе [API для поиска в Интернете](../bing-web-search/overview.md) содержится ответ по новостям.

## <a name="non-breaking-changes"></a>Некритические изменения

### <a name="query-parameters"></a>Параметры запроса

- Добавлены продукты, которые можно задать в качестве значения параметра запроса [category](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#category). Категории по рынкам приводятся в [этой статье](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference).

- Добавлен параметр запроса [SortBy](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#sortby), который возвращает темы, набирающие популярность (начиная с самых последних).

- Добавлен параметр запроса [Since](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#since), который возвращает темы, набирающие популярность. Это темы, обнаруженные Bing во время или после указанной метки времени (формат времени UNIX EPOCH).

### <a name="object-changes"></a>Изменения объектов

- В объект [NewsArticle](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle) добавлено поле `mentions`. Поле `mentions` содержит список сущностей (людей или мест), найденных в статье.

- В объект [NewsArticle](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#newsarticle) добавлено поле `video`. Поле `video` содержит видео, связанное с новостной статьей. Видео является либо \<iframe\> внедренным, либо эскизом движения.

- В объект [News](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#news) добавлено поле `sort`. Поле `sort` показывает порядок сортировки статей. Например, статьи сортируются по релевантности (по умолчанию) или дате.

- Добавлен объект [SortValue](/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#sortvalue), который определяет порядок сортировки. Поле `isSelected` указывает, используется ли в ответе порядок сортировки. Значение **true** означает, что в ответе использовался порядок сортировки. А если `isSelected` имеет значение **false**, то вы можете использовать URL-адрес в поле `url`, чтобы запросить другой порядок сортировки.