---
title: Обработка ошибок и исключений в MSAL.js
titleSuffix: Microsoft identity platform
description: Сведения об обработке ошибок и исключений, проблемах условного доступа и повторных попыток в MSAL.js приложениях.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/26/2020
ms.author: marsma
ms.reviewer: saeeda, hahamil
ms.custom: aaddev
ms.openlocfilehash: 20d276aba2ee3260911748cbee0a16020270059a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98761339"
---
# <a name="handle-errors-and-exceptions-in-msaljs"></a>Обработка ошибок и исключений в MSAL.js

[!INCLUDE [Active directory error handling introduction](../../../includes/active-directory-develop-error-handling-introduction.md)]

## <a name="error-handling-in-msaljs"></a>Обработка ошибок в MSAL.js

MSAL.js предоставляет объекты ошибок, которые являются абстрактными и используются для классификации различных типов распространенных ошибок. Он также предоставляет интерфейс для доступа к конкретным сведениям об ошибках, таким как сообщения об ошибке для их правильной работы.

### <a name="error-object"></a>Объект ошибки

```javascript
export class AuthError extends Error {
    // This is a short code describing the error
    errorCode: string;
    // This is a descriptive string of the error,
    // and may also contain the mitigation strategy
    errorMessage: string;
    // Name of the error class
    this.name = "AuthError";
}
```

Расширяя класс Error, вы получаете доступ к следующим свойствам.
- `AuthError.message`:  эквивалентно `errorMessage`.
- `AuthError.stack`: Трассировка стека для возникающих ошибок.

### <a name="error-types"></a>Типы ошибок

Ниже приведены доступные типы ошибок.

- `AuthError`: Базовый класс ошибки в библиотеке MSAL.js, который также используется для непредвиденных ошибок.

- `ClientAuthError`: класс ошибок, который обозначает проблему с проверкой подлинности клиента. К типу ClientAuthErrors относятся почти все ошибки, возникающие в этой библиотеке. Эти ошибки происходят из-за вызова метода входа, когда вход уже выполняется, отмены входа пользователем и т. п.

- `ClientConfigurationError`: класс ошибок, расширяющий `ClientAuthError`, который возникает перед выполнением запросов, если отсутствуют или неправильно указаны параметры конфигурации для пользователя.

- `ServerError`: класс ошибок для представления строк ошибок, отправленных сервером проверки подлинности. Это могут быть такие ошибки, как недопустимый формат запроса или недопустимые параметры, а также любые другие ошибки, не позволяющие серверу выполнить аутентификацию или авторизацию пользователя.

- `InteractionRequiredAuthError`: класс ошибок, расширяющий `ServerError`, для представления ошибок сервера, которые требуют вызвать интерактивный запрос. Такую ошибку вызывает `acquireTokenSilent`, если пользователю необходимо связаться с сервером и предоставить учетные данные или согласие для целей проверки подлинности и (или) авторизации. Коды ошибок: `"interaction_required"`, `"login_required"` и `"consent_required"`.

Для обработки ошибок в потоках проверки подлинности с методами перенаправления (`loginRedirect`, `acquireTokenRedirect`) следует зарегистрировать после перенаправления обратный вызов, который будет вызываться при успехе или неудаче, используя метод `handleRedirectCallback()` следующим образом.

```javascript
function authCallback(error, response) {
    //handle redirect response
}

var myMSALObj = new Msal.UserAgentApplication(msalConfig);

// Register Callbacks for redirect flow
myMSALObj.handleRedirectCallback(authCallback);
myMSALObj.acquireTokenRedirect(request);
```

Методы для всплывающих взаимодействий (`loginPopup`, `acquireTokenPopup`) возвращают обещания, поэтому вы можете использовать для их обработки шаблон обещания (.then и .catch), как показано далее:

```javascript
myMSALObj.acquireTokenPopup(request).then(
    function (response) {
        // success response
    }).catch(function (error) {
        console.log(error);
    });
```

### <a name="errors-that-require-interaction"></a>Ошибки, требующие взаимодействия

Ошибка возвращается при попытке использовать неинтерактивный метод получения токена, например `acquireTokenSilent`, если MSAL не может получить его автоматически.

Возможны следующие причины:

- требуется вход в систему;
- требуется предоставить согласие;
- нужно пройти многофакторную проверку подлинности.

Чтобы избавиться от этой ошибки, нужно вызывать интерактивный метод, например `acquireTokenPopup` или `acquireTokenRedirect`:

```javascript
// Request for Access Token
myMSALObj.acquireTokenSilent(request).then(function (response) {
    // call API
}).catch( function (error) {
    // call acquireTokenPopup in case of acquireTokenSilent failure
    // due to consent or interaction required
    if (error.errorCode === "consent_required"
    || error.errorCode === "interaction_required"
    || error.errorCode === "login_required") {
        myMSALObj.acquireTokenPopup(request).then(
            function (response) {
                // call API
            }).catch(function (error) {
                console.log(error);
            });
    }
});
```

[!INCLUDE [Active directory error handling claims challenges](../../../includes/active-directory-develop-error-handling-claims-challenges.md)]

Если вы получаете токены автоматически (с помощью `acquireTokenSilent`) через MSAL.js, приложение может получать ошибки, если для используемого API применяются [запросы утверждений условного доступа](../azuread-dev/conditional-access-dev-guide.md), например политики MFA.

Для обработки этой ошибки следует выполнить интерактивный вызов для получения токена через MSAL.js, например `acquireTokenPopup` или `acquireTokenRedirect`, как в следующем примере:

```javascript
myMSALObj.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // call API
}).catch(function(error) {
    if (error instanceof InteractionRequiredAuthError) {
    
        // extract, if exists, claims from error message
        if (error.ErrorMessage.claims) {
            accessTokenRequest.claimsRequest = JSON.stringify(error.ErrorMessage.claims);
        }
        
        // call acquireTokenPopup in case of InteractionRequiredAuthError failure
        myMSALObj.acquireTokenPopup(accessTokenRequest).then(function(accessTokenResponse) {
            // call API
        }).catch(function(error) {
            console.log(error);
        });
    }
});
```

Интерактивное получение токена передает запрос пользователю и предоставляет ему возможность выполнить требования политики условного доступа.

При вызове API, требующих условного доступа, вы можете получать запрос утверждений прямо в полученной от API ошибке. В этом случае можно передать утверждения, возвращенные в сообщении об ошибке, в поле `claimsRequest` класса `AuthenticationParameters.ts`, чтобы выполнить условия соответствующей политики. 

Дополнительные сведения см. в статье о [запросе дополнительных утверждений](active-directory-optional-claims.md).


[!INCLUDE [Active directory error handling retries](../../../includes/active-directory-develop-error-handling-retries.md)]

## <a name="next-steps"></a>Дальнейшие действия

Рекомендуется включить [ведение журнала в MSAL.js](msal-logging-js.md) для диагностики и отладки проблем.
