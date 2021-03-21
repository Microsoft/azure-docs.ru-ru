---
title: Инициализация клиентских приложений MSAL.js | Службы
titleSuffix: Microsoft identity platform
description: Сведения об инициализации клиентских приложений с помощью библиотеки проверки подлинности Майкрософт для JavaScript (MSAL.js).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/17/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev, devx-track-js
ms.openlocfilehash: a6d7b760ffd1931fa5dcdb3a67dd02f2798957a9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100365843"
---
# <a name="initialize-client-applications-using-msaljs"></a>Инициализация клиентских приложений с помощью MSAL.js

В этой статье описывается инициализация библиотеки проверки подлинности Майкрософт для JavaScript (MSAL.js) с помощью экземпляра приложения агента пользователя.

Приложение агента пользователя — это форма общедоступного клиентского приложения, в котором клиентский код выполняется в агенте пользователя, например в веб-браузере. Такие клиенты не сохраняют секреты, так как контекст браузера открыто доступен.

Дополнительные сведения о типах клиентских приложений и параметрах конфигурации приложений см. [в разделе общедоступные и конфиденциальные клиентские приложения в MSAL](msal-client-applications.md).

## <a name="prerequisites"></a>Предварительные условия

Перед инициализацией приложения сначала необходимо [зарегистрировать его в портал Azure](scenario-spa-app-registration.md), установив отношения доверия между приложением и платформой Microsoft Identity.

После регистрации приложения потребуются некоторые или все следующие значения, которые можно найти в портал Azure.

| Значение | Обязательно | Описание |
|:----- | :------: | :---------- |
| Идентификатор приложения (клиента) | Обязательно | Идентификатор GUID, который однозначно идентифицирует ваше приложение на платформе Microsoft Identity. |
| Authority | Необязательно | URL-адрес поставщика удостоверений ( *экземпляр*) и *аудитория входа* для приложения. Аудитория экземпляра и входа, если они объединены, составляют *центр*. |
| Идентификатор каталога (клиента) | Необязательно | Укажите это значение, если вы создаете бизнес-приложение исключительно для вашей организации, которое часто называют *приложением с одним клиентом*. |
| URI перенаправления | Необязательно | При создании веб-приложения компонент `redirectUri` указывает, где поставщик удостоверений (платформа идентификации Майкрософт) должен возвращать маркеры безопасности, выданные им. |

## <a name="initialize-msaljs-2x-apps"></a>Инициализация приложений MSAL.js 2. x

Инициализируйте контекст проверки подлинности MSAL, создав экземпляр [публикклиентаппликатион][msal-js-publicclientapplication] с объектом [конфигурации][msal-js-configuration] . Минимальное требуемое свойство конфигурации — это компонент `clientID` приложения, показанный в качестве **идентификатора приложения (клиента)** на странице **обзора** регистрации приложения в портал Azure.

Ниже приведен пример объекта конфигурации и экземпляра класса `PublicClientApplication` .

```javascript
const msalConfig = {
    auth: {
        clientId: "11111111-1111-1111-111111111111",
        authority: "https://login.microsoftonline.com/common",
        knownAuthorities: [],
        redirectUri: "https://localhost:3001",
        postLogoutRedirectUri: "https://localhost:3001/logout",
        navigateToLoginRequestUrl: true
    },
    cache: {
        cacheLocation: "sessionStorage",
        storeAuthStateInCookie: false
    },
    system: {
        loggerOptions: {
            loggerCallback: (level: LogLevel, message: string, containsPii: boolean): void => {
                if (containsPii) {
                    return;
                }
                switch (level) {
                    case LogLevel.Error:
                        console.error(message);
                        return;
                    case LogLevel.Info:
                        console.info(message);
                        return;
                    case LogLevel.Verbose:
                        console.debug(message);
                        return;
                    case LogLevel.Warning:
                        console.warn(message);
                        return;
                }
            },
            piiLoggingEnabled: false
        },
        windowHashTimeout: 60000,
        iframeHashTimeout: 6000,
        loadFrameTimeout: 0
    }
};

// Create an instance of PublicClientApplication
const msalInstance = new PublicClientApplication(msalConfig);

// Handle the redirect flows
msalInstance.handleRedirectPromise().then((tokenResponse) => {
    // Handle redirect response
}).catch((error) => {
    // Handle redirect error
});
```

### `handleRedirectPromise`

Вызовите [хандлередиректпромисе][msal-js-handleredirectpromise] , когда приложение использует потоки перенаправления. При использовании потоков перенаправления `handleRedirectPromise` следует выполнять при каждой загрузке страницы.

Существует три возможных результата из обещания:

- `.then` вызывается и `tokenResponse` является истинным: приложение возвращается из операции перенаправления, которая была успешной.
- `.then` вызывается метод и `tokenResponse` является false ( `null` ): приложение не возвращается из операции перенаправления.
- `.catch` вызывается: приложение возвращается из операции перенаправления, и произошла ошибка.

## <a name="initialize-msaljs-1x-apps"></a>Инициализация приложений MSAL.js 1. x

Инициализируйте контекст проверки подлинности MSAL 1. x, создав экземпляр [усеражентаппликатион][msal-js-useragentapplication] с объектом конфигурации. Минимальное требуемое свойство конфигурации — это компонент `clientID` приложения, показанный в качестве **идентификатора приложения (клиента)** на странице **обзора** регистрации приложения в портал Azure.

Для методов проверки подлинности с последовательностями перенаправления ([логинредирект][msal-js-loginredirect] и [acquireTokenRedirect][msal-js-acquiretokenredirect]) в MSAL.js 1.2. x или более ранней версии необходимо явно зарегистрировать обратный вызов для успешного выполнения или ошибки с помощью `handleRedirectCallback()` метода. Явная регистрация обратного вызова необходима в MSAL.js 1.2. x и более ранних версиях, так как потоки перенаправления не возвращают обещания, таких как методы со всплывающим окном. Регистрация обратного вызова *необязательна* в MSAL.js версии 1.3. x и более поздних версий.

```javascript
// Configuration object constructed
const msalConfig = {
    auth: {
        clientId: "11111111-1111-1111-111111111111"
    }
}

// Create UserAgentApplication instance
const msalInstance = new UserAgentApplication(msalConfig);

function authCallback(error, response) {
    // Handle redirect response
}

// Register a redirect callback for Success or Error (when using redirect methods)
// **REQUIRED** in MSAL.js 1.2.x and earlier
// **OPTIONAL** in MSAL.js 1.3.x and later
msalInstance.handleRedirectCallback(authCallback);
```

## <a name="single-instance-and-configuration"></a>Одиночный экземпляр и конфигурация

Как MSAL.js 1. x, так и 2. x, предназначены для одного экземпляра и конфигурации, `UserAgentApplication` `PublicClientApplication` соответственно, для представления одного контекста проверки подлинности.

`UserAgentApplication` `PublicClientApplication` Не рекомендуется использовать несколько экземпляров или, так как они вызывают конфликтующие записи кэша и поведение в браузере.

## <a name="next-steps"></a>Дальнейшие действия

Этот пример кода MSAL.js 2. x в GitHub демонстрирует создание экземпляра [публикклиентаппликатион][msal-js-publicclientapplication] с помощью объекта [конфигурации][msal-js-configuration] :

[Azure-Samples/MS-Identity-JavaScript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2)

<!-- LINKS - External -->
[msal-browser]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/
[msal-core]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/
[msal-js-acquiretokenredirect]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal.useragentapplication.html#acquiretokenredirect
[msal-js-configuration]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/modules/_azure_msal.html#configuration
[msal-js-handleredirectpromise]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal_browser.publicclientapplication.html#handleredirectpromise
[msal-js-loginredirect]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal.useragentapplication.html#loginredirect
[msal-js-publicclientapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal_browser.publicclientapplication.html
[msal-js-useragentapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/classes/_azure_msal.useragentapplication.html
