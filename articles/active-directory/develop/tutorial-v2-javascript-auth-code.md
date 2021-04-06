---
title: Руководство по Создание одностраничного приложения JavaScript, использующего поток кода авторизации | Azure
titleSuffix: Microsoft identity platform
description: В этом руководстве содержатся инструкции по созданию одностраничного приложения JavaScript для входа пользователей и использованию потока кода авторизации для получения маркера доступа из платформы удостоверений Майкрософт и вызова API Microsoft Graph.
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 07/17/2020
ms.author: hahamil
ms.custom: aaddev, devx-track-js
ms.openlocfilehash: 1ec046ca6b42a5ca8f33b0347c562c85abd42684
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98756173"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-a-javascript-single-page-app-spa-using-auth-code-flow"></a>Руководство по входу пользователей и вызову API Microsoft Graph из одностраничного приложения JavaScript с помощью потока кода авторизации

В этом руководстве показано, как создать одностраничное приложение (SPA) JavaScript, которое поддерживает вход пользователей и вызовы Microsoft Graph с использованием потока кода авторизации с PKCE. Создаваемое одностраничное приложение использует библиотеку проверки подлинности Майкрософт (MSAL) для JavaScript версии 2.0.

В этом руководстве рассматриваются следующие темы:
> [!div class="checklist"]
> * Выполнение потока кода авторизации OAuth 2.0 с PKCE
> * Вход в личные, рабочие и учебные учетные записи Майкрософт
> * Получение маркера доступа
> * Вызов API Microsoft Graph или собственного API, которые требуют маркеры доступа, полученные от платформы удостоверений Майкрософт

MSAL.js 2.0 — это расширенная версия MSAL.js 1.0, поддерживающая поток кода авторизации в браузере вместо потока неявного предоставления разрешения. MSAL.js 2.0 **НЕ** поддерживает неявный поток.

## <a name="prerequisites"></a>Предварительные требования

* [Node.js](https://nodejs.org/en/download/) для запуска локального веб-сервера
* [Visual Studio Code](https://code.visualstudio.com/download) или любой другой редактор кода.

## <a name="how-the-tutorial-app-works"></a>Принципы работы приложения из учебника

:::image type="content" source="media/tutorial-v2-javascript-auth-code/diagram-01-auth-code-flow.png" alt-text="Схема потока кода авторизации в одностраничном приложении":::

Приложение, созданное с помощью этого учебника, позволяет одностраничному приложению JavaScript отправлять запросы API Microsoft Graph, принимая маркеры безопасности от платформы удостоверений Майкрософт. В этом сценарии после входа в систему маркер доступа запрашивается и добавляется в HTTP-запросы в заголовок авторизации. Получение маркера и его обновление выполняет библиотека проверки подлинности Майкрософт (MSAL.js).

В этом учебнике используется следующая библиотека:

Библиотека проверки подлинности Майкрософт [msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) для пакета JavaScript версии 2.0 для браузера

## <a name="get-the-completed-code-sample"></a>Получение готового примера кода

Предпочитаете вместо этого загрузить готовый пример проекта для этого учебника? Чтобы запустить проект на локальном веб-сервере, например Node.js, клонируйте репозиторий [ms-identity-javascript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2):

`git clone https://github.com/Azure-Samples/ms-identity-javascript-v2`

Затем перейдите к [настройке](#register-your-application) примера кода перед его выполнением.

Чтобы продолжить работу с учебником и самостоятельно создать приложение, перейдите к следующему разделу, [Предварительные требования](#prerequisites).

## <a name="create-your-project"></a>Создание проекта

Установив [Node.js](https://nodejs.org/en/download/), создайте папку для размещения приложения, например *msal-spa-tutorial*.

Теперь реализуйте небольшой веб-сервер [Express](https://expressjs.com/) для обслуживания файла *index.html*.

1. Сначала перейдите в окне терминала в каталог проекта, а затем выполните следующие команды `npm`:
    ```console
    npm init -y
    npm install @azure/msal-browser
    npm install express
    npm install morgan
    npm install yargs
    ```
2. Далее создайте файл с именем *server.js* и добавьте в него следующий код:

   ```JavaScript
   const express = require('express');
   const morgan = require('morgan');
   const path = require('path');
   const argv = require('yargs')
      .usage('Usage: $0 -p [PORT]')
      .alias('p', 'port')
      .describe('port', '(Optional) Port Number - default is 3000')
      .strict()
      .argv;

   const DEFAULT_PORT = 3000;

   //initialize express.
   const app = express();

   // Initialize variables.
   let port = DEFAULT_PORT; // -p {PORT} || 3000;
   if (argv.p) {
      port = argv.p;
   }

   // Configure morgan module to log all requests.
   app.use(morgan('dev'));

   // Set the front-end folder to serve public assets.
   app.use("/lib", express.static(path.join(__dirname, "../../lib/msal-browser/lib")));

   // Setup app folders
   app.use(express.static('app'));

   // Set up a route for index.html.
   app.get('*', function (req, res) {
      res.sendFile(path.join(__dirname + '/index.html'));
   });

   // Start the server.
   app.listen(port);
   console.log(`Listening on port ${port}...`);
    ```

Теперь у вас есть небольшой веб-сервер для обслуживания одностраничного приложения. После выполнения остальных инструкций в этом учебнике файл и структура папок проекта должны выглядеть следующим образом:

```
msal-spa-tutorial/
├── app
│   ├── authConfig.js
│   ├── authPopup.js
│   ├── authRedirect.js
│   ├── graphConfig.js
│   ├── graph.js
│   ├── index.html
│   └── ui.js
└── server.js
```

## <a name="create-the-spa-ui"></a>Создание пользовательского интерфейса одностраничного приложения

1. Создайте в каталоге проекта папку *app*, а в ней — файл *index.html* для одностраничного приложения JavaScript. Этот файл используется для реализации пользовательского интерфейса, созданного на **платформе Bootstrap 4**, и импорта файлов скриптов для настройки, проверки подлинности и вызовов API.

    В файле *index.html* добавьте следующий код:

    ```html
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, shrink-to-fit=no">
        <title>Tutorial | MSAL.js JavaScript SPA</title>

        <!-- IE support: add promises polyfill before msal.js  -->
        <script type="text/javascript" src="//cdn.jsdelivr.net/npm/bluebird@3.7.2/js/browser/bluebird.min.js"></script>
        <script type="text/javascript" src="https://alcdn.msauth.net/browser/2.0.0-beta.4/js/msal-browser.js" integrity="sha384-7sxY2tN3GMVE5jXH2RL9AdbO6s46vUh9lUid4yNCHJMUzDoj+0N4ve6rLOmR88yN" crossorigin="anonymous"></script>

        <!-- adding Bootstrap 4 for UI components  -->
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
        <link rel="SHORTCUT ICON" href="https://c.s-microsoft.com/favicon.ico?v2" type="image/x-icon">
      </head>
      <body>
        <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
          <a class="navbar-brand" href="/">Microsoft identity platform</a>
          <div class="btn-group ml-auto dropleft">
              <button type="button" id="SignIn" class="btn btn-secondary" onclick="signIn()">
                Sign In
              </button>
          </div>
        </nav>
        <br>
        <h5 class="card-header text-center">JavaScript SPA calling Microsoft Graph API with MSAL.js</h5>
        <br>
        <div class="row" style="margin:auto" >
        <div id="card-div" class="col-md-3" style="display:none">
        <div class="card text-center">
          <div class="card-body">
            <h5 class="card-title" id="WelcomeMessage">Please sign-in to see your profile and read your mails</h5>
            <div id="profile-div"></div>
            <br>
            <br>
            <button class="btn btn-primary" id="seeProfile" onclick="seeProfile()">See Profile</button>
            <br>
            <br>
            <button class="btn btn-primary" id="readMail" onclick="readMail()">Read Mail</button>
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

        <!-- importing bootstrap.js and supporting js libraries -->
        <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
        <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>

        <!-- importing app scripts (load order is important) -->
        <script type="text/javascript" src="./authConfig.js"></script>
        <script type="text/javascript" src="./graphConfig.js"></script>
        <script type="text/javascript" src="./ui.js"></script>

        <!-- <script type="text/javascript" src="./authRedirect.js"></script>   -->
        <!-- uncomment the above line and comment the line below if you would like to use the redirect flow -->
        <script type="text/javascript" src="./authPopup.js"></script>
        <script type="text/javascript" src="./graph.js"></script>
      </body>
    </html>
    ```

2. Затем в папке *app* создайте файл с именем *ui.js* и добавьте следующий код. Этот файл будет обращаться к элементам модели DOM и обновлять их.

    ```JavaScript
    // Select DOM elements to work with
    const welcomeDiv = document.getElementById("WelcomeMessage");
    const signInButton = document.getElementById("SignIn");
    const cardDiv = document.getElementById("card-div");
    const mailButton = document.getElementById("readMail");
    const profileButton = document.getElementById("seeProfile");
    const profileDiv = document.getElementById("profile-div");

    function showWelcomeMessage(account) {
        // Reconfiguring DOM elements
        cardDiv.style.display = 'initial';
        welcomeDiv.innerHTML = `Welcome ${account.username}`;
        signInButton.setAttribute("onclick", "signOut();");
        signInButton.setAttribute('class', "btn btn-success")
        signInButton.innerHTML = "Sign Out";
    }

    function updateUI(data, endpoint) {
        console.log('Graph API responded at: ' + new Date().toString());

        if (endpoint === graphConfig.graphMeEndpoint) {
            const title = document.createElement('p');
            title.innerHTML = "<strong>Title: </strong>" + data.jobTitle;
            const email = document.createElement('p');
            email.innerHTML = "<strong>Mail: </strong>" + data.mail;
            const phone = document.createElement('p');
            phone.innerHTML = "<strong>Phone: </strong>" + data.businessPhones[0];
            const address = document.createElement('p');
            address.innerHTML = "<strong>Location: </strong>" + data.officeLocation;
            profileDiv.appendChild(title);
            profileDiv.appendChild(email);
            profileDiv.appendChild(phone);
            profileDiv.appendChild(address);

        } else if (endpoint === graphConfig.graphMailEndpoint) {
            if (data.value.length < 1) {
                alert("Your mailbox is empty!")
            } else {
                const tabList = document.getElementById("list-tab");
                tabList.innerHTML = ''; // clear tabList at each readMail call
                const tabContent = document.getElementById("nav-tabContent");

                data.value.map((d, i) => {
                    // Keeping it simple
                    if (i < 10) {
                        const listItem = document.createElement("a");
                        listItem.setAttribute("class", "list-group-item list-group-item-action")
                        listItem.setAttribute("id", "list" + i + "list")
                        listItem.setAttribute("data-toggle", "list")
                        listItem.setAttribute("href", "#list" + i)
                        listItem.setAttribute("role", "tab")
                        listItem.setAttribute("aria-controls", i)
                        listItem.innerHTML = d.subject;
                        tabList.appendChild(listItem)

                        const contentItem = document.createElement("div");
                        contentItem.setAttribute("class", "tab-pane fade")
                        contentItem.setAttribute("id", "list" + i)
                        contentItem.setAttribute("role", "tabpanel")
                        contentItem.setAttribute("aria-labelledby", "list" + i + "list")
                        contentItem.innerHTML = "<strong> from: " + d.from.emailAddress.address + "</strong><br><br>" + d.bodyPreview + "...";
                        tabContent.appendChild(contentItem);
                    }
                });
            }
        }
    }
    ```

## <a name="register-your-application"></a>Регистрация приложения

Выполните действия, описанные в статье [Одностраничное приложение: регистрация приложения](scenario-spa-app-registration.md), чтобы создать регистрацию приложения для своего SPA.

На шаге [URI перенаправления: MSAL.js 2.0 с потоком кода авторизации](scenario-spa-app-registration.md#redirect-uri-msaljs-20-with-auth-code-flow) введите `http://localhost:3000`, расположение по умолчанию, в котором выполняется приложение этого учебника.

Если вы хотите использовать другой порт, введите `http://localhost:<port>`, где `<port>` — предпочтительный номер TCP-порта. Если указан номер порта, отличный от `3000`, обновите также *server.js*, указав предпочтительный номер порта.

### <a name="configure-your-javascript-spa"></a>Настройка одностраничного приложения JavaScript

В папке *app* создайте файл с именем *authConfig.js*, который будет содержать параметры конфигурации для проверки подлинности, а затем добавьте в него следующий код:

```javascript
const msalConfig = {
  auth: {
    clientId: "Enter_the_Application_Id_Here",
    authority: "Enter_the_Cloud_Instance_Id_Here/Enter_the_Tenant_Info_Here",
    redirectUri: "Enter_the_Redirect_Uri_Here",
  },
  cache: {
    cacheLocation: "sessionStorage", // This configures where your cache will be stored
    storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
  }
};

// Add scopes here for ID token to be used at Microsoft identity platform endpoints.
const loginRequest = {
 scopes: ["openid", "profile", "User.Read"]
};

// Add scopes here for access token to be used at Microsoft Graph API endpoints.
const tokenRequest = {
 scopes: ["User.Read", "Mail.Read"]
};
```

Измените значения в разделе `msalConfig`, как описано далее.

- `Enter_the_Application_Id_Here`: **идентификатор приложения (клиента)** , которое вы зарегистрировали.
- `Enter_the_Cloud_Instance_Id_Here`: облачный экземпляр Azure, в котором зарегистрировано приложение.
  - Для основного (или *глобального*) облака Azure введите `https://login.microsoftonline.com`.
  - Для **национальных** облаков (например, китайского) соответствующие значения см. в статье [Национальные облака](authentication-national-cloud.md).
- `Enter_the_Tenant_info_here` должно быть одним из следующих:
  - Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение **идентификатором клиента** или **именем клиента**. Например, `contoso.microsoft.com`.
  - Если приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на `organizations`.
  - Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на `common`.
  - Чтобы ограничить поддержку только *личными учетными записями Microsoft*, замените это значение на `consumers`.
- Параметр `Enter_the_Redirect_Uri_Here` равен `http://localhost:3000`.

Если вы используете глобальное облако Azure, значение `authority` в файле *authConfig.js* должно выглядеть примерно так:

```javascript
authority: "https://login.microsoftonline.com/common",
```

В той же папке *app* создайте файл с именем *graphConfig.js*. Добавьте следующий код, чтобы предоставить приложению параметры конфигурации для вызова API Microsoft Graph:

```javascript
// Add the endpoints here for Microsoft Graph API services you'd like to use.
const graphConfig = {
    graphMeEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me",
    graphMailEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me/messages"
};
```

Измените значения в разделе `graphConfig`, как описано далее.

- `Enter_the_Graph_Endpoint_Here` экземпляр API Microsoft Graph, с которым должно взаимодействовать приложение.
  - Для **глобальной** конечной точки API Microsoft Graph замените оба экземпляра этой строки на `https://graph.microsoft.com`.
  - Дополнительные сведения о конечных точках в **национальных** облачных развертываниях см. в статье [Национальные облачные развертывания](/graph/deployments) в документации по Microsoft Graph.

Если вы используете глобальную конечную точку, значения `graphMeEndpoint` и `graphMailEndpoint` в файле *authConfig.js* должно выглядеть примерно так:

```javascript
graphMeEndpoint: "https://graph.microsoft.com/v1.0/me",
graphMailEndpoint: "https://graph.microsoft.com/v1.0/me/messages"
```

## <a name="use-the-microsoft-authentication-library-msal-to-sign-in-user"></a>Использование библиотеки проверки подлинности Майкрософт (MSAL) для входа пользователя

### <a name="pop-up"></a>Всплывающее окно

В папке *app* создайте файл с именем *authPopup.js* и добавьте следующий код для проверки подлинности и получения маркера для всплывающего окна входа:

```JavaScript
// Create the main myMSALObj instance
// configuration parameters are located at authConfig.js
const myMSALObj = new msal.PublicClientApplication(msalConfig);

let username = "";

function loadPage() {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    const currentAccounts = myMSALObj.getAllAccounts();
    if (currentAccounts === null) {
        return;
    } else if (currentAccounts.length > 1) {
        // Add choose account code here
        console.warn("Multiple accounts detected.");
    } else if (currentAccounts.length === 1) {
        username = currentAccounts[0].username;
        showWelcomeMessage(currentAccounts[0]);
    }
}

function handleResponse(resp) {
    if (resp !== null) {
        username = resp.account.username;
        showWelcomeMessage(resp.account);
    } else {
        loadPage();
    }
}

function signIn() {
    myMSALObj.loginPopup(loginRequest).then(handleResponse).catch(error => {
        console.error(error);
    });
}

function signOut() {
    const logoutRequest = {
        account: myMSALObj.getAccountByUsername(username)
    };

    myMSALObj.logout(logoutRequest);
}

function getTokenPopup(request) {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    request.account = myMSALObj.getAccountByUsername(username);
    return myMSALObj.acquireTokenSilent(request).catch(error => {
        console.warn("silent token acquisition fails. acquiring token using redirect");
        if (error instanceof msal.InteractionRequiredAuthError) {
            // fallback to interaction when silent call fails
            return myMSALObj.acquireTokenPopup(request).then(tokenResponse => {
                console.log(tokenResponse);

                return tokenResponse;
            }).catch(error => {
                console.error(error);
            });
        } else {
            console.warn(error);
        }
    });
}

function seeProfile() {
    getTokenPopup(loginRequest).then(response => {
        callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
        profileButton.classList.add('d-none');
        mailButton.classList.remove('d-none');
    }).catch(error => {
        console.error(error);
    });
}

function readMail() {
    getTokenPopup(tokenRequest).then(response => {
        callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
    }).catch(error => {
        console.error(error);
    });
}

loadPage();
```

### <a name="redirect"></a>перенаправление

Создайте файл с именем *authRedirect.js* в папке *app* и добавьте следующий код для проверки подлинности и получения маркера для перенаправления входа:

```javascript
// Create the main myMSALObj instance
// configuration parameters are located at authConfig.js
const myMSALObj = new msal.PublicClientApplication(msalConfig);

let accessToken;
let username = "";

// Redirect: once login is successful and redirects with tokens, call Graph API
myMSALObj.handleRedirectPromise().then(handleResponse).catch(err => {
    console.error(err);
});

function handleResponse(resp) {
    if (resp !== null) {
        username = resp.account.username;
        showWelcomeMessage(resp.account);
    } else {
        /**
         * See here for more info on account retrieval:
         * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
         */
        const currentAccounts = myMSALObj.getAllAccounts();
        if (currentAccounts === null) {
            return;
        } else if (currentAccounts.length > 1) {
            // Add choose account code here
            console.warn("Multiple accounts detected.");
        } else if (currentAccounts.length === 1) {
            username = currentAccounts[0].username;
            showWelcomeMessage(currentAccounts[0]);
        }
    }
}

function signIn() {
    myMSALObj.loginRedirect(loginRequest);
}

function signOut() {
    const logoutRequest = {
        account: myMSALObj.getAccountByUsername(username)
    };

    myMSALObj.logout(logoutRequest);
}

function getTokenRedirect(request) {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    request.account = myMSALObj.getAccountByUsername(username);
    return myMSALObj.acquireTokenSilent(request).catch(error => {
            console.warn("silent token acquisition fails. acquiring token using redirect");
            if (error instanceof msal.InteractionRequiredAuthError) {
                // fallback to interaction when silent call fails
                return myMSALObj.acquireTokenRedirect(request);
            } else {
                console.warn(error);
            }
        });
}

function seeProfile() {
    getTokenRedirect(loginRequest).then(response => {
        callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
        profileButton.classList.add('d-none');
        mailButton.classList.remove('d-none');
    }).catch(error => {
        console.error(error);
    });
}

function readMail() {
    getTokenRedirect(tokenRequest).then(response => {
        callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
    }).catch(error => {
        console.error(error);
    });
}
```

### <a name="how-the-code-works"></a>Принцип работы кода

Когда пользователь впервые нажимает кнопку **Войти**, метод `signIn` вызывает `loginPopup`, чтобы пользователь мог выполнить вход. Метод `loginPopup` открывает всплывающее окно с *конечной точкой платформы удостоверений Майкрософт*, чтобы запросить и проверить учетные данные пользователя. После успешного входа *msal.js* инициирует [поток кода авторизации](v2-oauth2-auth-code-flow.md).

На этом этапе код авторизации, защищенный с помощью PKCE, отправляется в конечную точку маркеров, защищенную с помощью CORS, и обменивается на маркеры. Приложение получает маркер идентификатора, маркер доступа и маркер обновления, *msal.js* их обрабатывает, а информация из этих маркеров кэшируется.

Маркер идентификатора содержит основные сведения о пользователе, например отображаемое имя. Если вы планируете использовать какие-либо данные, предоставляемые этим маркером идентификации, внутренний сервер *должен* его проверить и подтвердить, что маркер выдан допустимому пользователю для вашего приложения.

Срок действия маркера доступа ограничен и истекает через 24 часа. Маркер обновления можно использовать для автоматического получения новый маркеров доступа.

Одностраничное приложение, созданное в соответствии с инструкциями в этом учебнике, вызывает `acquireTokenSilent` и (или) `acquireTokenPopup`, чтобы получить *маркер доступа*, используемый при запросе API Microsoft Graph для получения сведений из профиля пользователя. Если вам нужен пример с проверкой маркера идентификатора, ознакомьтесь с примером приложения [active-directory-javascript-singlepageapp-dotnet-webapi-v2](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2) на сайте GitHub. В нем для проверки маркеров используется веб-API ASP.NET.

#### <a name="get-a-user-token-interactively"></a>Интерактивное получение маркера пользователя

После первоначального входа приложение не должно требовать от пользователей повторно проходить проверку подлинности при каждой попытке доступа к защищенному ресурсу (то есть запрашивать маркер). Чтобы избежать запросов на повторную проверку подлинности, вызовите `acquireTokenSilent`. Но иногда требуется настроить принудительное взаимодействие пользователей с платформой удостоверений Майкрософт. Пример:

- Пользователям нужно повторно вводить учетные данные, когда истекает срок действия пароля.
- Приложение запрашивает доступ к ресурсу, на обращение к которому пользователь должен дать согласие.
- Требуется двухфакторная проверка подлинности.

При вызове `acquireTokenPopup` открывается всплывающее окно (или `acquireTokenRedirect` перенаправляет пользователей на платформу удостоверений Майкрософт). В этом окне пользователям необходимо подтвердить учетные данные, предоставить согласие на требуемый ресурс или выполнить двухфакторную проверку подлинности.

#### <a name="get-a-user-token-silently"></a>Автоматическое получение маркера пользователя

Метод `acquireTokenSilent` отвечает за получение и обновление маркера без участия пользователя. После первого выполнения метода `loginPopup` или `loginRedirect` обычно используется метод `acquireTokenSilent`, чтобы получить маркеры для доступа к защищенным ресурсам для последующих вызовов. (Вызовы для запроса или обновления маркеров не требуют взаимодействия с пользователем.) В некоторых случаях выполнение `acquireTokenSilent` может завершаться сбоем. Например, так происходит, когда истекает срок действия пароля пользователя. Приложение может обработать это исключение двумя способами:

1. Немедленно вызвать `acquireTokenPopup`, чтобы активировать запрос на вход для пользователя. Этот шаблон обычно используется в интерактивных приложениях, где пользователю недоступно не прошедшее проверку подлинности содержимое. Пример, созданный в ходе пошаговой настройки, использует этот шаблон.
1. Визуально уведомить пользователя, что требуется интерактивный вход, чтобы пользователь мог выбрать подходящее время для входа или приложение могло повторить метод `acquireTokenSilent` ​​позднее. Обычно этот способ применим в тех случаях, когда пользователь может использовать другие функции приложения, на которые это не влияет. Например, в приложении есть содержимое, для доступа к аутентификация не требуется. В этой ситуации пользователь может самостоятельно решить, когда выполнять вход для получения доступа к защищенному ресурсу или обновления устаревших данных.

> [!NOTE]
> В этом учебнике по умолчанию используются методы `loginPopup` и `acquireTokenPopup`. Если вы используете Internet Explorer, рекомендуется использовать методы `loginRedirect` и `acquireTokenRedirect` из-за [известной проблемы](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) со всплывающими окнами в этом браузере. Пример достижения такого же результата с помощью методов перенаправления см. на странице [*authRedirect.js*](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/blob/quickstart/JavaScriptSPA/authRedirect.js) на сайте GitHub.

## <a name="call-the-microsoft-graph-api"></a>Вызов API Microsoft Graph

Создайте файл с именем *graph.js* в папке *app* и добавьте следующий код для отправки вызовов REST в API Microsoft Graph.

```javascript
// Helper function to call Microsoft Graph API endpoint
// using authorization bearer token scheme
function callMSGraph(endpoint, token, callback) {
    const headers = new Headers();
    const bearer = `Bearer ${token}`;

    headers.append("Authorization", bearer);

    const options = {
        method: "GET",
        headers: headers
    };

    console.log('request made to Graph API at: ' + new Date().toString());

    fetch(endpoint, options)
        .then(response => response.json())
        .then(response => callback(response, endpoint))
        .catch(error => console.log(error));
}
```

В примере приложения, созданном в соответствии с инструкциями в этом учебнике, для отправки HTTP-запроса `GET` защищенному ресурсу, которому требуется маркер, используется метод `callMSGraph()`. Затем метод возвращает содержимое вызывающему объекту. Этот метод добавляет полученный маркер в *заголовок авторизации HTTP*. Для примера приложения, созданного с помощью этого учебника, защищенным ресурсом является конечная точка *me* из API Microsoft Graph, которая отображает сведения о профиле выполнившего вход пользователя.

## <a name="test-your-application"></a>Тестирование приложения

Вы завершили создание приложения и теперь готовы запустить веб-сервер Node.js и протестировать функциональность приложения.

1. Запустите веб-сервер Node.js, выполнив следующие команды в командной строке из корневой папки проекта:

   ```console
   npm start
   ```
1. В браузере перейдите по адресу `http://localhost:3000` или `http://localhost:<port>`, где `<port>` — это порт, прослушиваемый вашим веб-сервером. Появится содержимое файла *index.html* с кнопкой **Sign In** (Войти).

### <a name="sign-in-to-the-application"></a>Вход в приложение

После загрузки файла *index.html* в браузер нажмите кнопку **Sign In** (Войти). Вам будет предложено войти с помощью платформы удостоверений Майкрософт:

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-01-signin-dialog.png" alt-text="Веб-браузер, в котором отображается диалоговое окно входа":::

### <a name="provide-consent-for-application-access"></a>Предоставление разрешения на доступ к приложению

При первом входе в приложение вам будет предложено предоставить ему доступ к профилю, а также выполнить вход:

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-02-consent-dialog.png" alt-text="Диалоговое окно содержимого, отображаемое в веб-браузере":::

Если вы согласны предоставить запрошенные разрешения, веб-приложения выводят имя пользователя, что означает успешный вход.

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-03-signed-in.png" alt-text="Результаты успешного входа в веб-браузере":::

### <a name="call-the-graph-api"></a>Вызов API Graph

После входа выберите **See Profile** (Просмотреть профиль), чтобы просмотреть сведения о профиле пользователя, возвращенные в ответе на вызов API Microsoft Graph.

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-04-see-profile.png" alt-text="Сведения о профиле из Microsoft Graph, отображаемые в браузере":::

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Дополнительные сведения об областях и делегированных разрешениях

Для чтения профиля пользователя API Microsoft Graph требуется область *user.read*. По умолчанию эта область автоматически добавляется в каждое приложение, зарегистрированное на портале Azure. Для других API Microsoft Graph, а также для пользовательских API вашего внутреннего сервера, могут потребоваться дополнительные области. Например, для отображения сообщений электронной почты пользователя API Microsoft Graph требуется область *Mail.Read*.

При добавлении областей приложение может запросить у пользователей дополнительное согласие на использование таких добавленных областей.

Если API серверной части не требуется область (что не рекомендуется), вы можете использовать `clientId` в качестве области в вызовах для получения маркеров.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите узнать больше о разработке одностраничных приложений JavaScript на платформе удостоверений Майкрософт, см. серию статей:

> [!div class="nextstepaction"]
> [Scenario: Одностраничное приложение](scenario-spa-overview.md)