---
title: Обновление API с версии 5 до версии 7 — API Bing для поиска в Интернете
titleSuffix: Azure Cognitive Services
description: Определите, какие части вашего приложения требуют обновления для использования API Bing для поиска в Интернете версии 7.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: E8827BEB-4379-47CE-B67B-6C81AD7DAEB1
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 02/12/2019
ms.author: scottwhi
ms.openlocfilehash: d930543671a5328d76a38aa7e1b421c111e89e39
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96349526"
---
# <a name="upgrade-from-bing-web-search-api-v5-to-v7"></a>Обновление API Bing для поиска в Интернете с версии 5 до версии 7

> [!WARNING]
> API Поиска Bing будут перенесены из Cognitive Services в службы Поиска Bing. С **30 октября 2020 г.** подготовку всех новых экземпляров Поиска Bing необходимо будет выполнять в соответствии с процедурой, описанной [здесь](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> API-интерфейсы Поиска Bing, подготовленные с помощью Cognitive Services, будут поддерживаться в течение следующих трех лет или до завершения срока действия вашего Соглашения Enterprise (в зависимости от того, какой период окончится раньше).
> Инструкции по миграции см. в статье о [службах Поиска Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

В этом руководстве по обновлению определены изменения между версиями 5 и 7 API Bing для поиска в Интернете. Руководство поможет определить компоненты приложения, которые необходимо обновить для использования версии 7.

## <a name="breaking-changes"></a>Критические изменения

### <a name="endpoints"></a>Конечные точки

- Номер версии конечной точки изменен с 5-го на 7-й. Например, https:\/\/api.cognitive.microsoft.com/bing/**v7.0**/search.

### <a name="error-response-objects-and-error-codes"></a>Объекты ответов на ошибки и коды ошибок

- Все невыполненные запросы теперь должны включать в тело ответа объект `ErrorResponse`.

- В объект `Error` добавлены следующие поля:  
  - `subCode`&mdash;Разделяет код ошибки на дискретные сегменты, если это возможно
  - `moreDetails`&mdash;Дополнительные сведения об ошибке, описанной в поле `message`


- Коды ошибок версии 5 заменены следующими возможными значениями `code` и `subCode`.

|Код|SubCode (дополнительный код)|Описание
|-|-|-
|ServerError|UnexpectedError<br/>ResourceError<br/>NotImplemented|Bing возвращает ServerError (ошибку сервера) каждый раз при возникновении любого из условий дополнительного кода. Ответ будет включать в себя ошибки, если код состояния HTTP — 500.
|InvalidRequest|ParameterMissing<br/>ParameterInvalidValue<br/>HttpNotAllowed<br/>Блокировано|Bing возвращает ошибку InvalidRequest (недопустимый запрос) всякий раз, когда любая часть запроса недопустима. Например, отсутствует обязательный параметр или значение параметра недопустимо.<br/><br/>В случае ошибки ParameterMissing или ParameterInvalidValue возвращается код состояния HTTP 400.<br/><br/>При ошибке HttpNotAllowed (HTTP запрещен) будет наблюдаться код состояния HTTP 410.
|RateLimitExceeded||Bing возвращает ошибку RateLimitExceeded всякий раз при превышении квоты запросов в секунду (QPS) или запросов в месяц (QPM).<br/><br/>Bing возвращает код состояния HTTP 429 при превышении квоты QPS и 403 при превышении QPM.
|InvalidAuthorization|AuthorizationMissing<br/>AuthorizationRedundancy|Bing возвращает InvalidAuthorization, когда Bing не может проверить подлинность вызывающего объекта. Например, когда заголовок `Ocp-Apim-Subscription-Key` отсутствует или при недопустимом ключе подписки.<br/><br/>Избыточность возникает, если указать более одного способа проверки подлинности.<br/><br/>При ошибке InvalidAuthorization кодом состояния HTTP будет 401.
|InsufficientAuthorization|AuthorizationDisabled<br/>AuthorizationExpired|Bing возвращает InsufficientAuthorization, когда вызывающая сторона не имеет разрешений на доступ к ресурсу. Эта ошибка может произойти, если ключ подписки отключен или срок его действия истек. <br/><br/>При ошибке InsufficientAuthorization возвращается код состояния HTTP 403.

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


## <a name="non-breaking-changes"></a>Некритические изменения  

### <a name="headers"></a>Заголовки

- Добавлен необязательный заголовок запроса [Pragma](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#pragma). По умолчанию Bing возвращает кэшированное содержимое, если оно доступно. Чтобы Bing не возвращал кэшированное содержимое, установите для заголовка Pragma значение no-cache (например, Pragma:no-cache).

### <a name="query-parameters"></a>Параметры запроса

- Добавлен параметр запроса [answerCount](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#answercount). С помощью этого параметра можно указать число ответов, которое должен включать ответ. Ответы выбираются на основе ранжирования. Например, если задать этот параметр равным трем (3), ответ будет включать три ответа с самым высоким приоритетом.  

- Добавлен параметр запроса [promote](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#promote). Используя этот параметр вместе с `answerCount`, можно явно включить один или несколько типов ответов независимо от их ранга. Например, чтобы распространить видео и изображения в ответ, вы можете задать для параметра повысить уровень значение *видео, изображения*. Список ответов, который требуется включить, не учитывается в предельном количестве `answerCount`. Например, если `answerCount` параметр имеет значение 2 и `promote` для него заданы *видео, изображения*, ответ может включать веб-страницы, Новости, видео и изображения.

### <a name="object-changes"></a>Изменения объектов

- Поле `someResultsRemoved` добавлено в объект [WebAnswer](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#webanswer). Поле содержит логическое значение, которое указывает, были ли исключены из ответа некоторые результаты ответа при поиске в Интернете.