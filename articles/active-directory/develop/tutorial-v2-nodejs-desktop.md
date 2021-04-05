---
title: Руководство по реализации входа пользователей и вызова API Microsoft Graph в классическом приложении Electron | Azure
titleSuffix: Microsoft identity platform
description: В этом руководстве описано, как создать классическое приложение Electron, которое поддерживает вход пользователей и может использовать поток кода авторизации для получения маркера доступа с платформы удостоверений Майкрософт и вызова API Microsoft Graph.
services: active-directory
author: derisen
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.date: 02/17/2021
ms.author: v-doeris
ms.openlocfilehash: e9e36a3ed2cab05eb63168452bc8472656ab5efb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101644295"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-in-an-electron-desktop-app"></a>Руководство по реализации входа пользователей и вызова API Microsoft Graph в классическом приложении Electron

В этом руководстве описано, как создать классическое приложение Electron, которое поддерживает вход пользователей и вызовы Microsoft Graph с использованием потока кода авторизации с PKCE. Создаваемое классическое приложение использует [библиотеку проверки подлинности Майкрософт (MSAL) для Node.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node).

Выполните шаги из этого руководства, чтобы сделать следующее:

> [!div class="checklist"]
> - регистрация приложения на портале Azure;
> - создать проект классического приложения Electron;
> - добавить логику аутентификации в приложение;
> - добавить метод для вызова веб-API;
> - добавить сведения о регистрации приложения;
> - Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

- [Node.js](https://nodejs.org/en/download/)
- [Electron](https://www.electronjs.org/)
- [Visual Studio Code](https://code.visualstudio.com/download) или любой другой редактор кода.

## <a name="register-the-application"></a>Регистрация приложения

Сначала выполните действия, описанные в статье [Краткое руководство. Регистрация приложения с помощью платформы удостоверений Майкрософт](quickstart-register-app.md), чтобы зарегистрировать приложение.

Используйте следующие параметры для регистрации приложения:

- Имя: `ElectronDesktopApp` (рекомендуется).
- Поддерживаемые типы учетных записей: **учетные записи в любом каталоге организации (любой каталог Azure AD — мультитенантный) и личные учетные записи Майкрософт (например, Skype, Xbox)** .
- Тип платформы: **Мобильные и классические приложения**
- URI перенаправления: `msal://redirect`.

## <a name="create-the-project"></a>Создание проекта

Создайте папку для размещения приложения, например *ElectronDesktopApp*.

1. Сначала перейдите в окне терминала в каталог проекта, а затем выполните следующие команды `npm`:

    ```console
    npm init -y
    npm install --save @azure/msal-node axios bootstrap dotenv jquery popper.js
    npm install --save-dev babel electron@10.1.6 webpack
    ```

2. Затем создайте папку с именем *App*. В этой папке создайте файл с именем *index.html*, который будет использоваться в качестве пользовательского интерфейса. Добавьте в него следующий код:

    ```html
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, shrink-to-fit=no">
        <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />
        <title>MSAL Node Electron Sample App</title>

        <!-- adding Bootstrap 4 for UI components  -->
        <link rel="stylesheet" href="../node_modules/bootstrap/dist/css/bootstrap.min.css">

        <link rel="SHORTCUT ICON" href="https://c.s-microsoft.com/favicon.ico?v2" type="image/x-icon">
    </head>

    <body>
        <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
            <a class="navbar-brand">Microsoft identity platform</a>
            <div class="btn-group ml-auto dropleft">
                <button type="button" id="signIn" class="btn btn-secondary" aria-expanded="false">
                    Sign in
                </button>
                <button type="button" id="signOut" class="btn btn-success" hidden aria-expanded="false">
                    Sign out
                </button>
            </div>
        </nav>
        <br>
        <h5 class="card-header text-center">Electron sample app calling MS Graph API using MSAL Node</h5>
        <br>
        <div class="row" style="margin:auto">
            <div id="cardDiv" class="col-md-3" style="display:none">
                <div class="card text-center">
                    <div class="card-body">
                        <h5 class="card-title" id="WelcomeMessage">Please sign-in to see your profile and read your mails
                        </h5>
                        <div id="profileDiv"></div>
                        <br>
                        <br>
                        <button class="btn btn-primary" id="seeProfile">See Profile</button>
                        <br>
                        <br>
                        <button class="btn btn-primary" id="readMail">Read Mails</button>
                    </div>
                </div>
            </div>
            <br>
            <br>
            <div class="col-md-4">
                <div class="list-group" id="list-tab" role="tablist">
                </div>
            </div>
            <div class="col-md-5">
                <div class="tab-content" id="nav-tabContent">
                </div>
            </div>
        </div>
        <br>
        <br>

        <script>
            window.jQuery = window.$ = require('jquery');
            require("./renderer.js");
        </script>

        <!-- importing bootstrap.js and supporting js libraries -->
        <script src="../node_modules/jquery/dist/jquery.js"></script>
        <script src="../node_modules/popper.js/dist/umd/popper.js"></script>
        <script src="../node_modules/bootstrap/dist/js/bootstrap.js"></script>
    </body>

    </html>
    ```

3. Затем создайте файл с именем *main.js* и добавьте в него следующий код:

    ```JavaScript
    require('dotenv').config()

    const path = require('path');
    const { app, ipcMain, BrowserWindow } = require('electron');
    const { IPC_MESSAGES } = require('./constants');

    const { callEndpointWithToken } = require('./fetch');
    const AuthProvider = require('./AuthProvider');

    const authProvider = new AuthProvider();
    let mainWindow;

    function createWindow () {
        mainWindow = new BrowserWindow({
            width: 800,
            height: 600,
            webPreferences: {
            nodeIntegration: true
            }
        });

        mainWindow.loadFile(path.join(__dirname, './index.html'));
        };

    app.on('ready', () => {
        createWindow();
    });

    app.on('window-all-closed', () => {
        app.quit();
    });


    // Event handlers
    ipcMain.on(IPC_MESSAGES.LOGIN, async() => {
        const account = await authProvider.login(mainWindow);

        await mainWindow.loadFile(path.join(__dirname, './index.html'));

        mainWindow.webContents.send(IPC_MESSAGES.SHOW_WELCOME_MESSAGE, account);
    });

    ipcMain.on(IPC_MESSAGES.LOGOUT, async() => {
        await authProvider.logout();
        await mainWindow.loadFile(path.join(__dirname, './index.html'));
    });

    ipcMain.on(IPC_MESSAGES.GET_PROFILE, async() => {

        const tokenRequest = {
            scopes: ['User.Read'],
        };

        const token = await authProvider.getToken(mainWindow, tokenRequest);
        const account = authProvider.account

        await mainWindow.loadFile(path.join(__dirname, './index.html'));

        const graphResponse = await callEndpointWithToken(`${process.env.GRAPH_ENDPOINT_HOST}${process.env.GRAPH_ME_ENDPOINT}`, token);

        mainWindow.webContents.send(IPC_MESSAGES.SHOW_WELCOME_MESSAGE, account);
        mainWindow.webContents.send(IPC_MESSAGES.SET_PROFILE, graphResponse);
    });

    ipcMain.on(IPC_MESSAGES.GET_MAIL, async() => {

        const tokenRequest = {
            scopes: ['Mail.Read'],
        };

        const token = await authProvider.getToken(mainWindow, tokenRequest);
        const account = authProvider.account;

        await mainWindow.loadFile(path.join(__dirname, './index.html'));

        const graphResponse = await callEndpointWithToken(`${process.env.GRAPH_ENDPOINT_HOST}${process.env.GRAPH_MAIL_ENDPOINT}`, token);

        mainWindow.webContents.send(IPC_MESSAGES.SHOW_WELCOME_MESSAGE, account);
        mainWindow.webContents.send(IPC_MESSAGES.SET_MAIL, graphResponse);
    });
    ```

В приведенном выше фрагменте кода мы инициализируем объект основного окна Electron и создаем некоторые обработчики событий для взаимодействия с окном Electron. Мы также импортируем параметры конфигурации, создаем экземпляр класса *authProvider* для обработки входа, выхода и получения маркера, а также вызываем API Microsoft Graph.

4. В той же папке (*App*) создайте файл с именем *renderer.js* и добавьте следующий код:

    ```JavaScript
    const { ipcRenderer } = require('electron');
    const { IPC_MESSAGES } = require('./constants');

    // UI event handlers
    document.querySelector('#signIn').addEventListener('click', () => {
        ipcRenderer.send(IPC_MESSAGES.LOGIN);
    });

    document.querySelector('#signOut').addEventListener('click', () => {
        ipcRenderer.send(IPC_MESSAGES.LOGOUT);
    });

    document.querySelector('#seeProfile').addEventListener('click', () => {
        ipcRenderer.send(IPC_MESSAGES.GET_PROFILE);
    });

    document.querySelector('#readMail').addEventListener('click', () => {
        ipcRenderer.send(IPC_MESSAGES.GET_MAIL);
    });

    // Main process message subscribers
    ipcRenderer.on(IPC_MESSAGES.SHOW_WELCOME_MESSAGE, (event, account) => {
        showWelcomeMessage(account);
    });

    ipcRenderer.on(IPC_MESSAGES.SET_PROFILE, (event, graphResponse) => {
        updateUI(graphResponse, `${process.env.GRAPH_ENDPOINT_HOST}${process.env.GRAPH_ME_ENDPOINT}`);
    });

    ipcRenderer.on(IPC_MESSAGES.SET_MAIL, (event, graphResponse) => {
        updateUI(graphResponse, `${process.env.GRAPH_ENDPOINT_HOST}${process.env.GRAPH_MAIL_ENDPOINT}`);
    });

    // DOM elements to work with
    const welcomeDiv = document.getElementById("WelcomeMessage");
    const signInButton = document.getElementById("signIn");
    const signOutButton = document.getElementById("signOut");
    const cardDiv = document.getElementById("cardDiv");
    const profileDiv = document.getElementById("profileDiv");
    const tabList = document.getElementById("list-tab");
    const tabContent = document.getElementById("nav-tabContent");

    function showWelcomeMessage(account) {
        cardDiv.style.display = "initial";
        welcomeDiv.innerHTML = `Welcome ${account.name}`;
        signInButton.hidden = true;
        signOutButton.hidden = false;
    }

    function clearTabs() {
        tabList.innerHTML = "";
        tabContent.innerHTML = "";
    }

    function updateUI(data, endpoint) {

        console.log(`Graph API responded at: ${new Date().toString()}`);

        if (endpoint === `${process.env.GRAPH_ENDPOINT_HOST}${process.env.GRAPH_ME_ENDPOINT}`) {
            setProfile(data);
        } else if (endpoint === `${process.env.GRAPH_ENDPOINT_HOST}${process.env.GRAPH_MAIL_ENDPOINT}`) {
            setMail(data);
        }
    }

    function setProfile(data) {
        profileDiv.innerHTML = ''

        const title = document.createElement('p');
        const email = document.createElement('p');
        const phone = document.createElement('p');
        const address = document.createElement('p');

        title.innerHTML = "<strong>Title: </strong>" + data.jobTitle;
        email.innerHTML = "<strong>Mail: </strong>" + data.mail;
        phone.innerHTML = "<strong>Phone: </strong>" + data.businessPhones[0];
        address.innerHTML = "<strong>Location: </strong>" + data.officeLocation;

        profileDiv.appendChild(title);
        profileDiv.appendChild(email);
        profileDiv.appendChild(phone);
        profileDiv.appendChild(address);
    }

    function setMail(data) {
        const mailInfo = data;
        if (mailInfo.value.length < 1) {
            alert("Your mailbox is empty!")
        } else {
            clearTabs();
            mailInfo.value.slice(0, 10).forEach((d, i) => {
                    createAndAppendListItem(d, i);
                    createAndAppendContentItem(d, i);
            });
        }
    }

    function createAndAppendListItem(d, i) {
        const listItem = document.createElement("a");
        listItem.setAttribute("class", "list-group-item list-group-item-action")
        listItem.setAttribute("id", "list" + i + "list")
        listItem.setAttribute("data-toggle", "list")
        listItem.setAttribute("href", "#list" + i)
        listItem.setAttribute("role", "tab")
        listItem.setAttribute("aria-controls", i)
        listItem.innerHTML = d.subject;
        tabList.appendChild(listItem);
    }

    function createAndAppendContentItem(d, i) {
        const contentItem = document.createElement("div");
        contentItem.setAttribute("class", "tab-pane fade")
        contentItem.setAttribute("id", "list" + i)
        contentItem.setAttribute("role", "tabpanel")
        contentItem.setAttribute("aria-labelledby", "list" + i + "list")

        if (d.from) {
            contentItem.innerHTML = "<strong> from: " + d.from.emailAddress.address + "</strong><br><br>" + d.bodyPreview + "...";
            tabContent.appendChild(contentItem);
        }
    }
    ```

5. Наконец, создайте файл с именем *constants.js*, в котором будут храниться строковые константы для описания **событий** приложения:

    ```JavaScript
    const IPC_MESSAGES = {
        SHOW_WELCOME_MESSAGE: 'SHOW_WELCOME_MESSAGE',
        LOGIN: 'LOGIN',
        LOGOUT: 'LOGOUT',
        GET_PROFILE: 'GET_PROFILE',
        SET_PROFILE: 'SET_PROFILE',
        GET_MAIL: 'GET_MAIL',
        SET_MAIL: 'SET_MAIL'
    }

    module.exports = {
        IPC_MESSAGES: IPC_MESSAGES,
    }
    ```

Вы реализовали простой графический пользовательский интерфейс и взаимодействия для вашего приложения Electron. После выполнения остальных инструкций в этом учебнике файл и структура папок проекта должны выглядеть следующим образом:

```
ElectronDesktopApp/
├── App
│   ├── authProvider.js
│   ├── constants.js
│   ├── fetch.js
│   ├── main.js
│   ├── renderer.js
│   ├── index.html
├── package.json
└── .env
```

## <a name="add-authentication-logic-to-your-app"></a>Добавление логики аутентификации в приложение

В папке *App* создайте файл с именем *AuthProvider.js*. Он будет содержать класс поставщика аутентификации, который будет обрабатывать операции входа, выхода, получения маркера, выбора учетной записи и связанных задач аутентификации с помощью MSAL Node. Добавьте в файл следующий код:

```JavaScript
const { PublicClientApplication, LogLevel, CryptoProvider } = require('@azure/msal-node');
const { protocol } = require('electron');
const path = require('path');
const url = require('url');

/**
 * To demonstrate best security practices, this Electron sample application makes use of
 * a custom file protocol instead of a regular web (https://) redirect URI in order to
 * handle the redirection step of the authorization flow, as suggested in the OAuth2.0 specification for Native Apps.
 */
const CUSTOM_FILE_PROTOCOL_NAME = process.env.REDIRECT_URI.split(':')[0]; // e.g. msal://redirect

/**
 * Configuration object to be passed to MSAL instance on creation.
 * For a full list of MSAL Node configuration parameters, visit:
 * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md
 */
const MSAL_CONFIG = {
    auth: {
        clientId: process.env.CLIENT_ID,
        authority: `${process.env.AAD_ENDPOINT_HOST}${process.env.TENANT_ID}`,
        redirectUri: process.env.REDIRECT_URI,
    },
    system: {
        loggerOptions: {
            loggerCallback(loglevel, message, containsPii) {
                console.log(message);
            },
            piiLoggingEnabled: false,
            logLevel: LogLevel.Verbose,
        }
    }
};

class AuthProvider {

    clientApplication;
    cryptoProvider;
    authCodeUrlParams;
    authCodeRequest;
    pkceCodes;
    account;

    constructor() {
        /**
         * Initialize a public client application. For more information, visit:
         * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/initialize-public-client-application.md
         */
        this.clientApplication = new PublicClientApplication(MSAL_CONFIG);
        this.account = null;

        // Initialize CryptoProvider instance
        this.cryptoProvider = new CryptoProvider();

        this.setRequestObjects();
    }

    /**
     * Initialize request objects used by this AuthModule.
     */
    setRequestObjects() {
        const requestScopes =  ['openid', 'profile', 'User.Read'];
        const redirectUri = process.env.REDIRECT_URI;

        this.authCodeUrlParams = {
            scopes: requestScopes,
            redirectUri: redirectUri
        };

        this.authCodeRequest = {
            scopes: requestScopes,
            redirectUri: redirectUri,
            code: null
        }

        this.pkceCodes = {
            challengeMethod: "S256", // Use SHA256 Algorithm
            verifier: "", // Generate a code verifier for the Auth Code Request first
            challenge: "" // Generate a code challenge from the previously generated code verifier
        };
    }

    async login(authWindow) {
        const authResult = await this.getTokenInteractive(authWindow, this.authCodeUrlParams);
        return this.handleResponse(authResult);
    }

    async logout() {
        if (this.account) {
            await this.clientApplication.getTokenCache().removeAccount(this.account);
            this.account = null;
        }
    }

    async getToken(authWindow, tokenRequest) {
        let authResponse;

        authResponse = await this.getTokenInteractive(authWindow, tokenRequest);

        return authResponse.accessToken || null;
    }

    // This method contains an implementation of access token acquisition in authorization code flow
    async getTokenInteractive(authWindow, tokenRequest) {

        /**
         * Proof Key for Code Exchange (PKCE) Setup
         *
         * MSAL enables PKCE in the Authorization Code Grant Flow by including the codeChallenge and codeChallengeMethod parameters
         * in the request passed into getAuthCodeUrl() API, as well as the codeVerifier parameter in the
         * second leg (acquireTokenByCode() API).
         *
         * MSAL Node provides PKCE Generation tools through the CryptoProvider class, which exposes
         * the generatePkceCodes() asynchronous API. As illustrated in the example below, the verifier
         * and challenge values should be generated previous to the authorization flow initiation.
         *
         * For details on PKCE code generation logic, consult the
         * PKCE specification https://tools.ietf.org/html/rfc7636#section-4
         */

        const {verifier, challenge} = await this.cryptoProvider.generatePkceCodes();

        this.pkceCodes.verifier = verifier;
        this.pkceCodes.challenge = challenge;

        const authCodeUrlParams = {
            ...this.authCodeUrlParams,
            scopes: tokenRequest.scopes,
            codeChallenge: this.pkceCodes.challenge, // PKCE Code Challenge
            codeChallengeMethod: this.pkceCodes.challengeMethod // PKCE Code Challenge Method
        };

        const authCodeUrl = await this.clientApplication.getAuthCodeUrl(authCodeUrlParams);

        protocol.registerFileProtocol(CUSTOM_FILE_PROTOCOL_NAME, (req, callback) => {
            const requestUrl = url.parse(req.url, true);
            callback(path.normalize(`${__dirname}/${requestUrl.path}`));
        });

        const authCode = await this.listenForAuthCode(authCodeUrl, authWindow);

        const authResponse = await this.clientApplication.acquireTokenByCode({
            ...this.authCodeRequest,
            scopes: tokenRequest.scopes,
            code: authCode,
            codeVerifier: this.pkceCodes.verifier // PKCE Code Verifier
        });

        return authResponse;
    }

    // Listen for authorization code response from Azure AD
    async listenForAuthCode(navigateUrl, authWindow) {

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

    /**
     * Handles the response from a popup or redirect. If response is null, will check if we have any accounts and attempt to sign in.
     * @param response
     */
    async handleResponse(response) {
        if (response !== null) {
            this.account = response.account;
        } else {
            this.account = await this.getAccount();
        }

        return this.account;
    }

    /**
     * Calls getAllAccounts and determines the correct account to sign into, currently defaults to first account found in cache.
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    async getAccount() {
        const cache = this.clientApplication.getTokenCache();
        const currentAccounts = await cache.getAllAccounts();

        if (currentAccounts === null) {
            console.log('No accounts detected');
            return null;
        }

        if (currentAccounts.length > 1) {
            // Add choose account code here
            console.log('Multiple accounts detected, need to add choose account code.');
            return currentAccounts[0];
        } else if (currentAccounts.length === 1) {
            return currentAccounts[0];
        } else {
            return null;
        }
    }
}

module.exports = AuthProvider;
```

В приведенном выше фрагменте кода мы сначала инициализировали класс `PublicClientApplication` MSAL Node, передав объект конфигурации (`msalConfig`). Затем мы предоставили методы `login`, `logout` и `getToken` для вызова модулем main (*main.js*). В `login` и `getToken` мы получили идентификатор и маркеры доступа соответственно, сначала запросив код авторизации, а затем обменяв его на маркер с помощью общедоступного API `acquireTokenByCode` MSAL Node.

## <a name="add-a-method-to-call-a-web-api"></a>Добавление метода для вызова веб-API

Создайте файл с именем *fetch.js*. Этот файл будет содержать HTTP-клиент Axios для вызовов REST к API Microsoft Graph.

```JavaScript
const axios = require('axios');

/**
 * Makes an Authorization 'Bearer' request with the given accessToken to the given endpoint.
 * @param endpoint
 * @param accessToken
 */
async function callEndpointWithToken(endpoint, accessToken) {
    const options = {
        headers: {
            Authorization: `Bearer ${accessToken}`
        }
    };

    console.log('Request made at: ' + new Date().toString());

    const response = await axios.default.get(endpoint, options);

    return response.data;
}

module.exports = {
    callEndpointWithToken: callEndpointWithToken,
};
```

## <a name="add-app-registration-details"></a>Добавление сведений о регистрации приложения

Наконец, создайте файл среды для хранения сведений о регистрации приложения, которые будут использоваться при получении маркеров. Для этого создайте файл с расширением *.env* в корневой папке примера (*ElectronDesktopApp*) и добавьте следующий код:

```
# Credentials
CLIENT_ID=Enter_the_Application_Id_Here
TENANT_ID=Enter_the_Tenant_Id_Here

# Configuration
REDIRECT_URI=msal://redirect

# Endpoints
AAD_ENDPOINT_HOST=Enter_the_Cloud_Instance_Id_Here
GRAPH_ENDPOINT_HOST=Enter_the_Graph_Endpoint_Here

# RESOURCES
GRAPH_ME_ENDPOINT=v1.0/me
GRAPH_MAIL_ENDPOINT=v1.0/me/messages

# SCOPES
GRAPH_SCOPES=User.Read Mail.Read
```

Приведите следующие сведения, указав значения, которые вы получили на портале регистрации приложений Azure:

- `Enter_the_Tenant_Id_here` должно быть одним из следующих:
  - Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение **идентификатором клиента** или **именем клиента**. Например, `contoso.microsoft.com`.
  - Если приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на `organizations`.
  - Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на `common`.
  - Чтобы ограничить поддержку только *личными учетными записями Microsoft*, замените это значение на `consumers`.
- `Enter_the_Application_Id_Here`: **идентификатор приложения (клиента)** , которое вы зарегистрировали.
- `Enter_the_Cloud_Instance_Id_Here`: облачный экземпляр Azure, в котором зарегистрировано приложение.
  - Для основного (или *глобального*) облака Azure введите `https://login.microsoftonline.com/`.
  - Для **национальных** облаков (например, китайского) соответствующие значения см. в статье [Национальные облака](authentication-national-cloud.md).
- `Enter_the_Graph_Endpoint_Here` экземпляр API Microsoft Graph, с которым должно взаимодействовать приложение.
  - Для **глобальной** конечной точки API Microsoft Graph замените оба экземпляра этой строки на `https://graph.microsoft.com/`.
  - Дополнительные сведения о конечных точках в **национальных** облачных развертываниях см. в статье [Национальные облачные развертывания](/graph/deployments) в документации по Microsoft Graph.

## <a name="test-the-app"></a>Тестирование приложения

Вы завершили создание приложения и теперь готовы запустить классическое приложение Electron и протестировать его функциональность.

1. Запустите приложение, выполнив следующую команду из корневой папки проекта:

```console
electron App/main.js
```

2. В главном окне приложения отобразится содержимое файла *index.html* и кнопка **Sign In** (Войти).

## <a name="test-sign-in-and-sign-out"></a>Проверка входа и выхода

После загрузки файла *index.html* выберите **Sign In** (Войти). Вам будет предложено войти с помощью платформы удостоверений Майкрософт:

:::image type="content" source="media/tutorial-v2-nodejs-desktop/desktop-01-signin-prompt.png" alt-text="Запрос на вход":::

Если вы согласны предоставить запрошенные разрешения, веб-приложения выводят имя пользователя, что означает успешный вход.

:::image type="content" source="media/tutorial-v2-nodejs-desktop/desktop-03-after-signin.png" alt-text="Успешный вход":::

## <a name="test-web-api-call"></a>Проверка вызова веб-API

После входа выберите **See Profile** (Просмотреть профиль), чтобы просмотреть сведения о профиле пользователя, возвращенные в ответе на вызов API Microsoft Graph.

:::image type="content" source="media/tutorial-v2-nodejs-desktop/desktop-04-profile.png" alt-text="Сведения профиля из Microsoft Graph":::

Выберите **Read Mails** (Прочитать почту), чтобы просмотреть сообщения в учетной записи пользователя. Отобразится экран для предоставления согласия:

:::image type="content" source="media/tutorial-v2-nodejs-desktop/desktop-05-consent-mail.png" alt-text="Экран предоставления согласия для разрешения read.mail":::

После предоставления согласия вы сможете просмотреть сообщения, возвращенные в ответ на вызов к API Microsoft Graph:

:::image type="content" source="media/tutorial-v2-nodejs-desktop/desktop-06-mails.png" alt-text="Сведения о просмотре сообщений из Microsoft Graph":::

## <a name="how-the-application-works"></a>Принцип работы приложения

Когда пользователь впервые нажимает кнопку **Sign In** (Войти), вызывается метод получения (`getTokenInteractive`) из *AuthProvider.js*. Этот метод перенаправляет пользователя для выполнения входа с помощью *конечной точки платформы удостоверений Майкрософт* и проверяет учетные данные пользователя, а затем получает **код авторизации**. Этот код затем обменивается на маркер доступа с помощью общедоступного API `acquireTokenByCode` MSAL Node.

На этом этапе код авторизации, защищенный с помощью PKCE, отправляется в конечную точку маркеров, защищенную с помощью CORS, и обменивается на маркеры. Приложение получает маркер идентификатора, маркер доступа и маркер обновления, MSAL Node их обрабатывает, а информация из этих маркеров кэшируется.

Маркер идентификатора содержит основные сведения о пользователе, например отображаемое имя. Срок действия маркера доступа ограничен и истекает через 24 часа. Если вы планируете использовать эти маркеры для доступа к защищенному ресурсу, внутренний сервер *должен* его проверить и подтвердить, что маркер выдан допустимому пользователю для вашего приложения.

Классическое приложение, созданное при работе с этим руководством, выполняет вызов REST к API Microsoft Graph, используя маркер доступа в качестве маркера носителя в заголовке запроса ([RFC 6750](https://tools.ietf.org/html/rfc6750)).

Для чтения профиля пользователя API Microsoft Graph требуется область *user.read*. По умолчанию эта область автоматически добавляется в каждое приложение, зарегистрированное на портале Azure. Для других API Microsoft Graph, а также для пользовательских API вашего внутреннего сервера, могут потребоваться дополнительные области. Например, для отображения сообщений электронной почты пользователя API Microsoft Graph требуется область *Mail.Read*.

При добавлении областей приложение может запросить у пользователей дополнительное согласие на использование таких добавленных областей.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите узнать больше о Node.js и разработке классических приложений Electron на платформе удостоверений Майкрософт, см. серию статей:

> [!div class="nextstepaction"]
> [Сценарий: Классическое приложение, которое вызывает веб-API](scenario-desktop-overview.md)
