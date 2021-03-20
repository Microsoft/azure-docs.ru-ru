---
title: Использование интерфейсов API с поддержкой оценки непрерывного доступа в приложениях | Службы
titleSuffix: Microsoft identity platform
description: Как повысить безопасность и устойчивость приложений, добавив поддержку оценки непрерывного доступа, включив долгосрочные маркеры доступа, которые можно отозвать на основе критических событий и оценки политики.
services: active-directory
author: knicholasa
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/06/2020
ms.author: nichola
ms.reviewer: ''
ms.openlocfilehash: f6ce792b3db0100d7356884bbc6ee2696580df10
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97652064"
---
# <a name="how-to-use-continuous-access-evaluation-enabled-apis-in-your-applications"></a>Использование интерфейсов API с поддержкой оценки непрерывного доступа в приложениях

[Оценка непрерывного доступа](../conditional-access/concept-continuous-access-evaluation.md) (автоматизированного конструирования) — это развивающийся промышленный стандарт, который позволяет отзывать маркеры доступа на основе [критических событий](../conditional-access/concept-continuous-access-evaluation.md#critical-event-evaluation) и [оценки политики](../conditional-access/concept-continuous-access-evaluation.md#conditional-access-policy-evaluation-preview) , а не исключать истечение срока действия маркера на основе времени существования. Для некоторых API-интерфейсов ресурсов, поскольку риск и политика оцениваются в режиме реального времени, это может увеличить время существования маркера до 28 часов. Эти долгосрочные токены будут заблаговременно обновляться библиотекой проверки подлинности Майкрософт (MSAL), что повышает устойчивость приложений.

В этой статье показано, как использовать интерфейсы API с поддержкой автоматизированного конструирования в приложениях.

## <a name="implementation-considerations"></a>Рекомендации по реализации

Чтобы использовать оценку непрерывного доступа, как приложение, так и API ресурсов, к которому он обращается, необходимо включить автоматизированного конструирования. Однако подготовка кода к использованию ресурса с включенным автоматизированного конструирования не позволит вам использовать интерфейсы API, которые не включены в автоматизированного конструирования.

Если API ресурсов реализует автоматизированного конструирования, и ваше приложение объявляет, что оно может обслуживать автоматизированного конструирования, приложение будет получать маркеры автоматизированного конструирования для этого ресурса. По этой причине, если вы объявили, что приложение автоматизированного конструирования готово, приложение должно обрабатывать запрос на утверждение автоматизированного конструирования для всех API-интерфейсов ресурсов, которые принимают маркеры доступа Microsoft Identity. Если вы не обрабатываете ответы автоматизированного конструирования в этих вызовах API, ваше приложение может выполнить цикл повторного вызова API с токеном, который еще находится в возвращенном сроке действия маркера, но был отозван из-за автоматизированного конструирования.

## <a name="the-code"></a>Код

Первым шагом является добавление кода, обрабатывающего ответ от API ресурсов, который отклоняет вызов из-за автоматизированного конструирования. При использовании автоматизированного конструирования API возвращает состояние 401 и заголовок WWW-Authenticate, когда маркер доступа был отозван, или API обнаруживает изменение в используемом IP-адресе. Заголовок WWW-Authenticate содержит запрос на утверждения, который приложение может использовать для получения нового маркера доступа.

Пример:

```console
HTTP 401; Unauthorized
WWW-Authenticate=Bearer
  authorization_uri="https://login.windows.net/common/oauth2/authorize",
  error="insufficient_claims",
  claims="eyJhY2Nlc3NfdG9rZW4iOnsibmJmIjp7ImVzc2VudGlhbCI6dHJ1ZSwgInZhbHVlIjoiMTYwNDEwNjY1MSJ9fX0="
```

Приложение будет проверять следующее:

- вызов API возвращает состояние 401
- существование заголовка WWW-Authenticate, содержащего:
  - параметр "Error" со значением "insufficient_claims"
  - параметр "Claims"

При соблюдении этих условий приложение может извлекать и декодировать запрос на утверждения.

```csharp
if (APIresponse.IsSuccessStatusCode)
{
    // ...
}
else
{
    if (APIresponse.StatusCode == System.Net.HttpStatusCode.Unauthorized
        && APIresponse.Headers.WwwAuthenticate.Any())
    {
        AuthenticationHeaderValue bearer = APIresponse.Headers.WwwAuthenticate.First
            (v => v.Scheme == "Bearer");
        IEnumerable<string> parameters = bearer.Parameter.Split(',').Select(v => v.Trim()).ToList();
        var error = GetParameter(parameters, "error");

        if (null != error && "insufficient_claims" == error)
        {
            var claimChallengeParameter = GetParameter(parameters, "claims");
            if (null != claimChallengeParameter)
            {
                var claimChallengebase64Bytes = System.Convert.FromBase64String(claimChallengeParameter);
                var claimChallenge = System.Text.Encoding.UTF8.GetString(claimChallengebase64Bytes);
                var newAccessToken = await GetAccessTokenWithClaimChallenge(scopes, claimChallenge);
```

Приложение будет использовать запрос утверждений для получения нового маркера доступа для ресурса.

```csharp
try
{
    authResult = await _clientApp.AcquireTokenSilent(scopes, firstAccount)
        .WithClaims(claimChallenge)
        .ExecuteAsync()
        .ConfigureAwait(false);
}
catch (MsalUiRequiredException)
{
    try
    {
        authResult = await _clientApp.AcquireTokenInteractive(scopes)
            .WithClaims(claimChallenge)
            .WithAccount(firstAccount)
            .ExecuteAsync()
            .ConfigureAwait(false);
    }
    // ...
```

Когда ваше приложение будет готово к обработке запроса на утверждение, возвращенного ресурсом с включенным автоматизированного конструирования, вы можете сообщить Microsoft Identity, что приложение автоматизированного конструирования готово. Чтобы сделать это в приложении MSAL, создайте общедоступный клиент, используя возможности клиента "CP1".

```csharp
_clientApp = PublicClientApplicationBuilder.Create(App.ClientId)
    .WithDefaultRedirectUri()
    .WithAuthority(authority)
    .WithClientCapabilities(new [] {"cp1"})
    .Build();
```

Вы можете протестировать приложение, войдя пользователя в приложение, а затем используя портал Azure для отзыва сеансов пользователя. При следующем вызове приложением API с поддержкой автоматизированного конструирования пользователю будет предложено повторно пройти проверку подлинности.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в разделе [Оценка непрерывного доступа](../conditional-access/concept-continuous-access-evaluation.md).
