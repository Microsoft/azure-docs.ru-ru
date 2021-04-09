---
title: Краткое руководство. Вызов Microsoft Graph из классического приложения Node.js | Azure
titleSuffix: Microsoft identity platform
description: В этом кратком руководстве описано, как классическое приложение Node.js Electron может выполнять вход пользователей и получить маркер доступа для вызова API, защищенного конечной точкой платформы удостоверений Майкрософт.
services: active-directory
author: derisen
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.date: 02/17/2021
ms.author: v-doeris
ms.openlocfilehash: beef869b891fe6e3f0ea2f667763cb310008b2fc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101653275"
---
# <a name="quickstart-acquire-an-access-token-and-call-the-microsoft-graph-api-from-an-electron-desktop-app"></a>Краткое руководство. Получение маркера доступа и вызов API Microsoft Graph из классического приложения Electron

В этом кратком руководстве описывается, как скачать и выполнить пример кода. В примере кода показано, как в классическом приложении Electron реализовать вход пользователей и получение маркеров доступа для вызова API Microsoft Graph.

В рамках этого краткого руководства используется [библиотека проверки подлинности Майкрософт для Node.js (MSAL Node)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) с [потоком кода авторизации с использованием PKCE](v2-oauth2-auth-code-flow.md).

## <a name="prerequisites"></a>Предварительные требования

* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) или любой другой редактор кода.

> [!div renderon="docs"]
> ## <a name="register-and-download-the-sample-application"></a>Регистрация и скачивание примера приложения
>
> Выполните шаги ниже, чтобы начать.
>
> #### <a name="step-1-register-the-application"></a>Шаг 1. Регистрация приложения
> Чтобы зарегистрировать приложение и добавить сведения о его регистрации в решение вручную, сделайте следующее:
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Введите **имя** приложения, например `msal-node-desktop`. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
> 1. Выберите **Зарегистрировать**, чтобы создать приложение.
> 1. В разделе **Управление** выберите **Проверка подлинности**.
> 1. Поочередно выберите **Добавить платформу** > **Мобильные и классические приложения**.
> 1. В разделе **URI перенаправления** введите `msal://redirect`.
> 1. Нажмите кнопку **Настроить**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-the-application-in-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Для работы примера кода в этом кратком руководстве необходимо добавить URL-адрес ответа **msal://redirect**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести это изменение для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-windows-desktop/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-electron-sample-project"></a>Шаг 2. Скачивание примера проекта Electron

> [!div renderon="docs"]
> [Скачивание примера кода](https://github.com/azure-samples/ms-identity-javascript-nodejs-desktop/archive/main.zip)

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Скачивание примера кода](https://github.com/azure-samples/ms-identity-javascript-nodejs-desktop/archive/main.zip)

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> #### <a name="step-3-configure-the-electron-sample-project"></a>Шаг 3. Настройка примера проекта Electron
>
> 1. Распакуйте ZIP-файл в локальную папку, расположенную как можно ближе к корню диска (например, *C:\Azure-Samples*).
> 1. Измените файл *.env*, заменив значения полей `TENANT_ID` и `CLIENT_ID` следующим фрагментом кода:
>
>    ```
>    "TENANT_ID": "Enter_the_Tenant_Id_Here",
>    "CLIENT_ID": "Enter_the_Application_Id_Here"
>    ```
>    Где:
>    - `Enter_the_Application_Id_Here` — это **идентификатор приложения (клиента)** , которое вы зарегистрировали.
>    - `Enter_the_Tenant_Id_Here` — замените это значение на **идентификатор клиента** или **имя клиента** (например, contoso.microsoft.com).
>
> > [!TIP]
> > Чтобы найти значения параметров **Идентификатор приложения (клиента)** и **Идентификатор каталога (клиента)** , на портале Azure перейдите на страницу приложения **Обзор**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-4-run-the-application"></a>Шаг 4. Выполнение приложения

> [!div renderon="docs"]
> #### <a name="step-4-run-the-application"></a>Шаг 4. Выполнение приложения

Зависимости этого примера необходимо установить один раз:

```console
npm install
```

Затем запустите приложение с помощью командной строки или консоли:

```console
npm start
```

Вы должны увидеть пользовательский интерфейс приложения с кнопкой **Sign in** (Войти).

## <a name="about-the-code"></a>Примечания о коде

Ниже рассмотрены некоторые важные аспекты примера приложения.

### <a name="msal-node"></a>MSAL Node

Библиотека [MSAL Node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) нужна для обработки входа пользователей и запросов маркеров, которые используются для доступа к API, защищенному платформой удостоверений Майкрософт. Дополнительные сведения об использовании MSAL Node с классическими приложениями см. в [этой статье](scenario-desktop-overview.md).

Вы можете установить MSAL Node, выполнив следующую команду npm.

```console
npm install @azure/msal-node --save
```

### <a name="msal-initialization"></a>Инициализация MSAL

Добавив следующий код, вы можете добавить ссылку на MSAL Node:

```javascript
const { PublicClientApplication } = require('@azure/msal-node');
```

Затем выполните инициализацию MSAL с помощью следующего кода.

```javascript
const MSAL_CONFIG = {
    auth: {
        clientId: "Enter_the_Application_Id_Here",
        authority: "https://login.microsoftonline.com/Enter_the_Tenant_Id_Here",
    },
};

const pca = new PublicClientApplication(MSAL_CONFIG);
```

> | Где: |Описание |
> |---------|---------|
> | `clientId` | **Идентификатор приложения (клиента)** , зарегистрированного на портале Azure. Это значение можно найти на странице приложения **Обзор** на портале Azure. |
> | `authority`    | Конечная точка STS для проверки подлинности пользователей. Обычно `https://login.microsoftonline.com/{tenant}` для общедоступного облака, где {tenant} — имя или идентификатор вашего клиента.|

### <a name="requesting-tokens"></a>Запрос маркеров

На первом этапе потока кода авторизации с использованием PKCE выполните подготовку и отправку запроса кода авторизации с соответствующими параметрами. Затем, на втором этапе потока, обеспечьте прослушивание ответа кода авторизации. После получения кода обменяйте его для получения маркера.

```javascript
// The redirect URI you setup during app registration with a custom file protocol "msal"
const redirectUri = "msal://redirect";

const cryptoProvider = new CryptoProvider();

const pkceCodes = {
    challengeMethod: "S256", // Use SHA256 Algorithm
    verifier: "", // Generate a code verifier for the Auth Code Request first
    challenge: "" // Generate a code challenge from the previously generated code verifier
};

/**
 * Starts an interactive token request
 * @param {object} authWindow: Electron window object
 * @param {object} tokenRequest: token request object with scopes
 */
async function getTokenInteractive(authWindow, tokenRequest) {

    /**
     * Proof Key for Code Exchange (PKCE) Setup
     *
     * MSAL enables PKCE in the Authorization Code Grant Flow by including the codeChallenge and codeChallengeMethod
     * parameters in the request passed into getAuthCodeUrl() API, as well as the codeVerifier parameter in the
     * second leg (acquireTokenByCode() API).
     */

    const {verifier, challenge} = await cryptoProvider.generatePkceCodes();

    pkceCodes.verifier = verifier;
    pkceCodes.challenge = challenge;

    const authCodeUrlParams = {
        redirectUri: redirectUri
        scopes: tokenRequest.scopes,
        codeChallenge: pkceCodes.challenge, // PKCE Code Challenge
        codeChallengeMethod: pkceCodes.challengeMethod // PKCE Code Challenge Method
    };

    const authCodeUrl = await pca.getAuthCodeUrl(authCodeUrlParams);

    // register the custom file protocol in redirect URI
    protocol.registerFileProtocol(redirectUri.split(":")[0], (req, callback) => {
        const requestUrl = url.parse(req.url, true);
        callback(path.normalize(`${__dirname}/${requestUrl.path}`));
    });

    const authCode = await listenForAuthCode(authCodeUrl, authWindow); // see below

    const authResponse = await pca.acquireTokenByCode({
        redirectUri: redirectUri,
        scopes: tokenRequest.scopes,
        code: authCode,
        codeVerifier: pkceCodes.verifier // PKCE Code Verifier
    });

    return authResponse;
}

/**
 * Listens for auth code response from Azure AD
 * @param {string} navigateUrl: URL where auth code response is parsed
 * @param {object} authWindow: Electron window object
 */
async function listenForAuthCode(navigateUrl, authWindow) {

    authWindow.loadURL(navigateUrl);

    return new Promise((resolve, reject) => {
        authWindow.webContents.on('will-redirect', (event, responseUrl) => {
            try {
                const parsedUrl = new URL(responseUrl);
                const authCode = parsedUrl.searchParams.get('code');
                resolve(authCode);
            } catch (err) {
                reject(err);
            }
        });
    });
}
```

> |Где:| Описание |
> |---------|---------|
> | `authWindow` | Текущее окно Electron в процессе. |
> | `tokenRequest` | Содержит запрашиваемые области, такие как `"User.Read"` для Microsoft Graph или `"api://<Application ID>/access_as_user"` для пользовательских веб-API. |

## <a name="next-steps"></a>Дальнейшие шаги

Дополнительные сведения о разработке классических приложений Electron с помощью MSAL Node см. в учебнике:

> [!div class="nextstepaction"]
> [Учебник. Вход пользователей и вызов API Microsoft Graph в классическом приложении Electron](tutorial-v2-nodejs-desktop.md)