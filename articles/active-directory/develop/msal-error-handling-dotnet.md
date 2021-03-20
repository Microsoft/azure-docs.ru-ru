---
title: Обработка ошибок и исключений в MSAL.NET
titleSuffix: Microsoft identity platform
description: Сведения об обработке ошибок и исключений, проблемах условного доступа и повторных попыток в MSAL.NET.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/26/2020
ms.author: marsma
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ms.openlocfilehash: 565acd745ba5d7fdec71f306d3851e599838f7d9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99584050"
---
# <a name="handle-errors-and-exceptions-in-msalnet"></a>Обработка ошибок и исключений в MSAL.NET

[!INCLUDE [Active directory error handling introduction](../../../includes/active-directory-develop-error-handling-introduction.md)]

## <a name="error-handling-in-msalnet"></a>Обработка ошибок в MSAL.NET

При обработке исключений .NET вы можете различать их по типу и элементу `ErrorCode`. Значения `ErrorCode` являются константами типа [MsalError](/dotnet/api/microsoft.identity.client.msalerror).

Вы можете также проверить значения полей [MsalClientException](/dotnet/api/microsoft.identity.client.msalexception), [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception) и [MsalUIRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception).

Если выдается исключение [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception), попробуйте проверить, нет ли в списке [кодов ошибок проверки подлинности и авторизации](reference-aadsts-error-codes.md).

### <a name="common-net-exceptions"></a>Распространенные исключения .NET

Ниже перечислены несколько исключений, которые встречаются чаще других, и предложения по их устранению.  

| Исключение | Код ошибки | Меры по снижению риска|
| --- | --- | --- |
| [MsalUiRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception) | AADSTS65001: The user or administrator has not consented to use the application with ID '{appId}' named '{appName}'. Send an interactive authorization request for this user and resource. (Пользователь или администратор не предоставили разрешение на использование приложения с идентификатором "{идентификатор_приложения}" и именем "{имя_приложения}". Отправьте интерактивный запрос авторизации для этого пользователя и ресурса.)| Сначала получите согласие пользователя. Если вы не используете платформу .NET Core (которая не предоставляет никаких веб-интерфейсов), однократно вызовите `AcquireTokeninteractive`. Если вы используете .NET Core или не хотите выполнять `AcquireTokenInteractive`, пользователь может предоставить согласие по следующему URL-адресу: `https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read`, чтобы вызвать `AcquireTokenInteractive`: `app.AcquireTokenInteractive(scopes).WithAccount(account).WithClaims(ex.Claims).ExecuteAsync();`.|
| [MsalUiRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception) | AADSTS50079: The user is required to use [multi-factor authentication (MFA)](../authentication/concept-mfa-howitworks.md).| There is no mitigation. Если MFA настроено для клиента и Azure Active Directory (AAD) решает его принудительное применение, вернитесь к интерактивному потоку, например `AcquireTokenInteractive` или `AcquireTokenByDeviceCode` .|
| [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception) |AADSTS90010: The grant type isn't supported over the */common* or */consumers* endpoints. Use the */organizations* or tenant-specific endpoint. You used */common*. (Тип предоставления не поддерживается через конечные точки /common или /consumers. Используйте конечную точку /organizations или конечную точку конкретного клиента. Вы использовали /common).| Как описано в сообщении от AAD, центр должен иметь клиента или конечную точку */organizations*.|
| [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception) | AADSTS70002: The request body must contain the following parameter: `client_secret or client_assertion` (Текст запроса должен содержать следующий параметр: client_secret или client_assertion).| Такое исключение может произойти, если приложение не зарегистрировано в качестве общедоступного клиентского приложения в Azure AD. Измените манифест приложения на портале Azure и задайте для параметра `allowPublicClient` значение `true`. |
| [MsalClientException](/dotnet/api/microsoft.identity.client.msalclientexception)| `unknown_user Message`: Could not identify logged in user. (Не удалось идентифицировать пользователя, вошедшего в систему.)| Библиотеке не удалось запросить Текущий вошедший в систему пользователь Windows, или этот пользователь не является AD или присоединен к Azure AD (пользователи, присоединенные к рабочему месту, не поддерживаются). Решение 1. Проверьте на UWP, что приложение имеет следующие возможности: корпоративная аутентификация, частные сети (клиент и сервер), данные учетной записи пользователя. Решение 2. Создайте собственную логику для получения имени пользователя (например, john@contoso.com) и примените форму `AcquireTokenByIntegratedWindowsAuth`, которая принимает имя пользователя.|
| [MsalClientException](/dotnet/api/microsoft.identity.client.msalclientexception)|integrated_windows_auth_not_supported_managed_user| Этот метод использует протокол, предоставляемый Active Directory (AD). Если пользователь был создан в Azure AD без резервного копирования AD (управляемый пользователь), этот метод завершится ошибкой. Пользователи, созданные в AD и поддерживаемые Azure AD ("Федеративные" пользователи), могут воспользоваться этим неинтерактивным методом проверки подлинности. Устранение рисков. Используйте интерактивный режим аутентификации.|

### `MsalUiRequiredException`

Один из распространенных кодов состояния, возвращаемых из MSAL.NET при вызове `AcquireTokenSilent()`, — `MsalError.InvalidGrantError`. Этот код состояния означает, что приложение должно снова вызвать библиотеку проверки подлинности, но в интерактивном режиме (Аккуиретокенинтерактиве или Аккуиретокенбидевицекодефлов для общедоступных клиентских приложений у вас есть проблема в веб-приложениях). Это связано с тем, что перед выдачей токена проверки подлинности требуется дополнительное взаимодействие с пользователем.

В большинстве случаев происходит сбой `AcquireTokenSilent`, так как кэш токенов не содержит токены, соответствующие запросу. Срок действия маркеров доступа истекает через 1 час, а `AcquireTokenSilent` попытается получить новый на основе токена обновления (в терминах OAuth2 это поток "Обновление токена"). Этот поток также может завершиться сбоем по различным причинам, например, если администратор клиента настроит более строгие политики входа в систему. 

Цель взаимодействия — обеспечить выполнение действия пользователем. С некоторыми из этих ситуаций пользователи могут легко справиться (например, принять условия использования одним щелчком), в то время как в других решить проблему с текущей конфигурацией невозможно (например, если компьютер должен подключаться к определенной корпоративной сети). При определенных обстоятельствах пользователю может быть оказана помощь в настройке многофакторной проверки подлинности или установке Microsoft Authenticator на устройстве.

### <a name="msaluirequiredexception-classification-enumeration"></a>Перечисление классификации `MsalUiRequiredException`

MSAL предоставляет `Classification` поле, которое можно прочитать, чтобы обеспечить лучшую работу пользователя. Например, чтобы сообщить пользователю о том, что срок действия пароля истек или ему необходимо предоставить согласие на использование некоторых ресурсов. Поддерживаемые значения являются частью перечисления `UiRequiredExceptionClassification`.

| Классификация    | Значение           | Рекомендуемая обработка |
|-------------------|-------------------|----------------------|
| BasicAction | Ситуацию можно разрешить путем взаимодействия с пользователем во время интерактивного потока проверки подлинности. | Вызов AcquireTokenInteractively(). |
| AdditionalAction | Ситуацию можно разрешить с помощью дополнительного корректирующего взаимодействия с системой за пределами интерактивного потока проверки подлинности. | Вызов AcquireTokenInteractively() для отображения сообщения с объяснением корректирующего действия. Вызывающее приложение может скрыть потоки, требующие additional_action, если пользователь, скорее всего, не станет выполнять корректирующее действие. |
| MessageOnly      | В данный момент разрешить ситуацию невозможно. При запуске интерактивного потока проверки подлинности отобразится сообщение с объяснением сложившейся ситуации. | Вызов AcquireTokenInteractively() для отображения сообщения с объяснением сложившейся ситуации. AcquireTokenInteractively() возвращает ошибку UserCanceled после того, как пользователь прочтет сообщение и закроет окно. Вызывающее приложение может скрыть потоки, которые приводят к message_only, если сообщение, скорее всего, будет бесполезным для пользователя.|
| ConsentRequired  | Согласие пользователя отсутствует или отменено. | Вызовите AcquireTokenInteractively() для получения согласия пользователя. |
| UserPasswordExpired | Срок действия пароля пользователя истек. | Вызов AcquireTokenInteractively() для предоставления пользователю возможности сброса пароля. |
| PromptNeverFailed| Интерактивная проверка подлинности вызвана с параметром prompt=never, что вынудило MSAL использовать файлы cookie браузера, а не отображать запрос в браузере. Сделать это не удалось. | Вызов AcquireTokenInteractively() без Prompt.None |
| AcquireTokenSilentFailed | Пакет SDK для MSAL не содержит достаточно сведений для получения токена из кэша. Это может быть вызвано тем, что в кэше отсутствуют токены или учетная запись не найдена. Дополнительные сведения приводятся в сообщении об ошибке.  | Вызов AcquireTokenInteractively(). |
| None    | Дополнительные сведения не предоставляются. Ситуацию можно разрешить путем взаимодействия с пользователем во время интерактивного потока проверки подлинности. | Вызов AcquireTokenInteractively(). |

## <a name="net-code-example"></a>Пример кода .NET

```csharp
AuthenticationResult res;
try
{
 res = await application.AcquireTokenSilent(scopes, account)
        .ExecuteAsync();
}
catch (MsalUiRequiredException ex) when (ex.ErrorCode == MsalError.InvalidGrantError)
{
 switch (ex.Classification)
 {
  case UiRequiredExceptionClassification.None:
   break;
  case UiRequiredExceptionClassification.MessageOnly:
  // You might want to call AcquireTokenInteractive(). Azure AD will show a message
  // that explains the condition. AcquireTokenInteractively() will return UserCanceled error
  // after the user reads the message and closes the window. The calling application may choose
  // to hide features or data that result in message_only if the user is unlikely to benefit 
  // from the message
  try
  {
      res = await application.AcquireTokenInteractive(scopes).ExecuteAsync();
  }
  catch (MsalClientException ex2) when (ex2.ErrorCode == MsalError.AuthenticationCanceledError)
  {
   // Do nothing. The user has seen the message
  }
  break;

  case UiRequiredExceptionClassification.BasicAction:
  // Call AcquireTokenInteractive() so that the user can, for instance accept terms
  // and conditions

  case UiRequiredExceptionClassification.AdditionalAction:
  // You might want to call AcquireTokenInteractive() to show a message that explains the remedial action. 
  // The calling application may choose to hide flows that require additional_action if the user 
  // is unlikely to complete the remedial action (even if this means a degraded experience)

  case UiRequiredExceptionClassification.ConsentRequired:
  // Call AcquireTokenInteractive() for user to give consent.
  
  case UiRequiredExceptionClassification.UserPasswordExpired:
  // Call AcquireTokenInteractive() so that user can reset their password
  
  case UiRequiredExceptionClassification.PromptNeverFailed:
  // You used WithPrompt(Prompt.Never) and this failed
  
  case UiRequiredExceptionClassification.AcquireTokenSilentFailed:
  default:
  // May be resolved by user interaction during the interactive authentication flow.
  res = await application.AcquireTokenInteractive(scopes)
                         .ExecuteAsync(); break;
 }
}
```
[!INCLUDE [Active directory error handling claims challenges](../../../includes/active-directory-develop-error-handling-claims-challenges.md)]

При вызове API, который требует условного доступа через MSAL.NET, приложение должно обрабатывать исключения, связанные с запросом утверждений. Они передаются в виде [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception) с непустым свойством [Claims](/dotnet/api/microsoft.identity.client.msalserviceexception.claims).

Чтобы обработать такой запрос утверждений, следует использовать метод `.WithClaim()` из класса `PublicClientApplicationBuilder`.

[!INCLUDE [Active directory error handling retries](../../../includes/active-directory-develop-error-handling-retries.md)]

### <a name="http-error-codes-500-600"></a>Коды ошибок HTTP 500–600

MSAL.NET реализует простой механизм однократного повтора для всех ошибок с кодами HTTP 500–600.

Исключение [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception) предоставляет `System.Net.Http.Headers.HttpResponseHeaders` в качестве свойства `namedHeaders`. Вы можете использовать дополнительные сведения из кода ошибки для повышения надежности приложений. В описанном выше случае можно использовать `RetryAfterproperty` (с типом `RetryConditionHeaderValue`) для вычисления времени повторной попытки.

Ниже приведен пример для управляющего приложения, использующего поток учетных данных клиента. Его можно адаптировать к любому из методов получения токена.

```csharp

bool retry = false;
do
{
    TimeSpan? delay;
    try
    {
         result = await publicClientApplication.AcquireTokenForClient(scopes, account).ExecuteAsync();
    }
    catch (MsalServiceException serviceException)
    {
         if (serviceException.ErrorCode == "temporarily_unavailable")
         {
             RetryConditionHeaderValue retryAfter = serviceException.Headers.RetryAfter;
             if (retryAfter.Delta.HasValue)
             {
                 delay = retryAfter.Delta;
             }
             else if (retryAfter.Date.HasValue)
             {
                 delay = retryAfter.Date.Value.Offset;
             }
         }
    }
    // . . .
    if (delay.HasValue)
    {
        Thread.Sleep((int)delay.Value.TotalMilliseconds); // sleep or other
        retry = true;
    }
} while (retry);
```

## <a name="next-steps"></a>Дальнейшие действия

Рекомендуется включить [ведение журнала в MSAL.NET](msal-logging-dotnet.md) , чтобы помочь в диагностике и отладке проблем.
