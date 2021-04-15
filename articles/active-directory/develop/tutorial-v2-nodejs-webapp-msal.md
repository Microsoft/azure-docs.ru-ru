---
title: Руководство. Добавление возможности входа пользователей в веб-приложение Node.js & Express | Azure
titleSuffix: Microsoft identity platform
description: В этом руководстве показано, как включить возможность входа пользователей в веб-приложение.
services: active-directory
author: derisen
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.date: 02/17/2021
ms.author: v-doeris
ms.openlocfilehash: 58132b6d038abe0db02d1e58f8c8a8f64962ff49
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106279720"
---
# <a name="tutorial-sign-in-users-in-a-nodejs--express-web-app"></a>Руководство. Добавление возможности входа пользователей в веб-приложение Node.js & Express

В этом руководстве показано, как создать веб-приложение, которое отвечает за вход пользователей. Создаваемое веб-приложение использует [библиотеку проверки подлинности Майкрософт (MSAL) для Node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node).

Выполните инструкции из этого руководства, чтобы:

> [!div class="checklist"]
> - регистрация приложения на портале Azure;
> - создать проект веб-приложения Express;
> - установить пакеты библиотеки аутентификации;
> - добавить сведения о регистрации приложения;
> - добавить код для выполнения входа пользователя;
> - Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

- [Node.js](https://nodejs.org/en/download/)
- [Visual Studio Code](https://code.visualstudio.com/download) или любой другой редактор кода.

## <a name="register-the-application"></a>Регистрация приложения

Сначала выполните действия, описанные в статье [Краткое руководство. Регистрация приложения с помощью платформы удостоверений Майкрософт](quickstart-register-app.md), чтобы зарегистрировать приложение.

Используйте следующие параметры для регистрации приложения:

- Имя: `ExpressWebApp` (рекомендуется).
- Поддерживаемые типы учетных записей: **учетные записи в любом каталоге организации (любой каталог Azure AD — мультитенантный) и личные учетные записи Майкрософт (например, Skype, Xbox)** .
- Тип платформы: **Web**
- URI перенаправления: `http://localhost:3000/redirect`.
- Секрет клиента: `*********` (запишите это значение для использования на последующих этапах; оно отображается только один раз).

## <a name="create-the-project"></a>Создание проекта

Создайте папку для размещения приложения, например *ExpressWebApp*.

1. Сначала перейдите в окне терминала в каталог проекта, а затем выполните следующие команды `npm`:

```console
    npm init -y
    npm install --save express
```

2. Затем создайте файл с именем *index.js* и добавьте следующий код:

```JavaScript
    const express = require("express");
    const msal = require('@azure/msal-node');

    const SERVER_PORT = process.env.PORT || 3000;

    // Create Express App and Routes
    const app = express();

    app.listen(SERVER_PORT, () => console.log(`Msal Node Auth Code Sample app listening on port ${SERVER_PORT}!`))
```

Теперь у вас есть простой веб-сервер, работающий на порте 3000. Файл и структура папок проекта должны выглядеть следующим образом:

```
ExpressWebApp/
├── index.js
└── package.json
```

## <a name="install-the-auth-library"></a>Установка библиотеки аутентификации

Найдите корневой каталог проекта в терминале и установите пакет MSAL Node с помощью npm.

```console
    npm install --save @azure/msal-node
```

## <a name="add-app-registration-details"></a>Добавление сведений о регистрации приложения

В созданном ранее файле *index.js* добавьте следующий код:

```JavaScript
    // Before running the sample, you will need to replace the values in the config,
    // including the clientSecret
    const config = {
        auth: {
            clientId: "Enter_the_Application_Id",
            authority: "Enter_the_Cloud_Instance_Id_Here/Enter_the_Tenant_Id_here",
            clientSecret: "Enter_the_Client_secret"
        },
        system: {
            loggerOptions: {
                loggerCallback(loglevel, message, containsPii) {
                    console.log(message);
                },
                piiLoggingEnabled: false,
                logLevel: msal.LogLevel.Verbose,
            }
        }
    };
```

Приведите следующие сведения, указав значения, которые вы получили на портале регистрации приложений Azure:

- `Enter_the_Tenant_Id_here` должно быть одним из следующих:
  - Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение **идентификатором клиента** или **именем клиента**. Например, `contoso.microsoft.com`.
  - Если приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на `organizations`.
  - Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на `common`.
  - Чтобы ограничить поддержку только *личными учетными записями Microsoft*, замените это значение на `consumers`.
- `Enter_the_Application_Id_Here`: **идентификатор приложения (клиента)** , которое вы зарегистрировали.
- `Enter_the_Cloud_Instance_Id_Here`: облачный экземпляр Azure, в котором зарегистрировано приложение.
  - Для основного (или *глобального*) облака Azure введите `https://login.microsoftonline.com`.
  - Для **национальных** облаков (например, китайского) соответствующие значения см. в статье [Национальные облака](authentication-national-cloud.md).
- `Enter_the_Client_secret`. Замените это значение созданным ранее секретом клиента. Чтобы создать новый ключ, выберите элемент **Сертификаты и секреты** в параметрах регистрации приложения на портале Azure.

> [!WARNING]
> Использовать секреты в формате открытого текста в исходном коде небезопасно. В этой статье такой секрет клиента применяется только для простоты. Используйте [учетные данные сертификата](active-directory-certificate-credentials.md), а не секреты клиента, в конфиденциальных клиентских приложениях, особенно если вы планируете развертывать такие приложения в рабочей среде.

## <a name="add-code-for-user-login"></a>Добавление кода для выполнения входа пользователя

В созданном ранее файле *index.js* добавьте следующий код:

```JavaScript
    // Create msal application object
    const cca = new msal.ConfidentialClientApplication(config);

    app.get('/', (req, res) => {
        const authCodeUrlParameters = {
            scopes: ["user.read"],
            redirectUri: "http://localhost:3000/redirect",
        };

        // get url to sign user in and consent to scopes needed for application
        cca.getAuthCodeUrl(authCodeUrlParameters).then((response) => {
            res.redirect(response);
        }).catch((error) => console.log(JSON.stringify(error)));
    });

    app.get('/redirect', (req, res) => {
        const tokenRequest = {
            code: req.query.code,
            scopes: ["user.read"],
            redirectUri: "http://localhost:3000/redirect",
        };

        cca.acquireTokenByCode(tokenRequest).then((response) => {
            console.log("\nResponse: \n:", response);
            res.sendStatus(200);
        }).catch((error) => {
            console.log(error);
            res.status(500).send(error);
        });
    });
```

## <a name="test-sign-in"></a>Проверка входа

Вы создали приложение и теперь готовы протестировать его функциональность.

1. Запустите консольное приложение Node.js, выполнив следующую команду из корневой папки проекта:

```console
   node index.js
```

2. Запустите окно браузера и перейдите по адресу `http://localhost:3000`. Вы увидите экран входа:

:::image type="content" source="media/tutorial-v2-nodejs-webapp-msal/sign-in-screen.png" alt-text="Экран входа в Azure AD":::

3. После ввода учетных данных появится экран для предоставления согласия с предложением подтвердить разрешения для приложения.

:::image type="content" source="media/tutorial-v2-nodejs-webapp-msal/consent-screen.png" alt-text="Экран для предоставления согласия в Azure AD":::

## <a name="how-the-application-works"></a>Принцип работы приложения

При работе с этим руководством вы инициализировали объект [ConfidentialClientApplication](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/initialize-confidential-client-application.md) MSAL Node, передав ему объект конфигурации (*msalConfig*), который содержит параметры, полученные из регистрации приложения Azure AD на портале Azure. Созданное веб-приложение использует [поток предоставления кода авторизации OAuth 2.0](./v2-oauth2-auth-code-flow.md) для выполнения входа пользователей, а также для получения идентификатора и маркеров доступа.

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите узнать больше о разработке консольных приложений Node.js и Express на платформе удостоверений Майкрософт, см. серию статей:

> [!div class="nextstepaction"]
> [Scenario: Веб-приложение, выполняющее вход пользователей](scenario-web-app-sign-user-overview.md)
