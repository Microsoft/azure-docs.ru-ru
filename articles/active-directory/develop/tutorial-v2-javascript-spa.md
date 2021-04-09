---
title: Руководство по созданию одностраничного приложения JavaScript, использующего платформу удостоверений Майкрософт для аутентификации | Azure
titleSuffix: Microsoft identity platform
description: В этом руководстве показано, как создать одностраничное приложение (SPA) JavaScript, которое использует платформу удостоверений Майкрософт для реализации входа пользователей, и как получить маркер доступа для вызова API Microsoft Graph от имени пользователей.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 08/06/2020
ms.author: nacanuma
ms.custom: aaddev, identityplatformtop40, devx-track-js
ms.openlocfilehash: dab4ac22a8b0da927f05376755463885ce32c343
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100103011"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-a-javascript-single-page-application-spa"></a>Руководство по Вход пользователей и вызов API Microsoft Graph из одностраничного приложения JavaScript (SPA)

В этом руководстве показано, как создать одностраничное приложение (SPA) JavaScript, которое поддерживает вход пользователей и вызовы Microsoft Graph с помощью неявного потока. Создаваемое одностраничное приложение использует библиотеку проверки подлинности Майкрософт (MSAL) для JavaScript версии 1.0.

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * создание проекта JavaScript с помощью `npm`;
> * регистрация приложения на портале Azure;
> * добавление кода для поддержки входа и выхода пользователей;
> * добавление кода для вызова API Microsoft Graph;
> * Тестирование приложения

>[!TIP]
> В этом учебнике используется MSAL.js версии 1.x, в которой можно использовать только поток с неявным предоставлением разрешения для одностраничных приложений. Мы рекомендуем использовать для всех новых приложений [MSAL.js версии 2.x и поток кода авторизации с поддержкой PKCE и CORS](tutorial-v2-javascript-auth-code.md).

## <a name="prerequisites"></a>Предварительные требования

* [Node.js](https://nodejs.org/en/download/) для запуска локального веб-сервера.
* [Visual Studio Code](https://code.visualstudio.com/download) или другой редактор для изменения файлов проекта.
* Современный браузер. **Internet Explorer** **не поддерживается** приложением, которое вы создадите в этом руководстве, так как оно использует соглашения [ES6](http://www.ecma-international.org/ecma-262/6.0/).

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Как работает пример приложения, созданный в этом руководстве

![Схема работы примера приложения, создаваемого в этом кратком руководстве](media/active-directory-develop-guidedsetup-javascriptspa-introduction/javascriptspa-intro.svg)

Пример приложения, созданный с помощью этого руководства, позволяет одностраничному приложению JavaScript выполнять запрос к API Microsoft Graph или веб-API, принимающему маркеры от платформы удостоверений Майкрософт. В этом сценарии после входа пользователя в систему маркер доступа запрашивается и добавляется в HTTP-запросы с использованием заголовка авторизации. Этот маркер будет использоваться для получения профиля пользователя и почтовых сообщений через **API Microsoft Graph**.

Получение маркера и его обновление выполняет [библиотека проверки подлинности Майкрософт](https://github.com/AzureAD/microsoft-authentication-library-for-js) (MSAL).

## <a name="set-up-your-web-server-or-project"></a>Настройка веб-сервера или проекта

> Хотите скачать этот пример проекта вместо указанного выше проекта? [Скачайте файлы проекта](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip).
>
> Перейдите к [настройке](#register-your-application), чтобы настроить пример кода перед выполнением.

## <a name="create-your-project"></a>Создание проекта

Установите [Node.js](https://nodejs.org/en/download/) и создайте папку для размещения приложения. В ней будет реализован простой веб-сервер [Express](https://expressjs.com/) для обслуживания файла `index.html`.

1. С помощью терминала, например встроенного терминала Visual Studio Code, найдите папку проекта и введите:

   ```console
   npm init
   ```

2. Затем установите необходимые зависимости:

   ```console
   npm install express --save
   npm install morgan --save
   ```

1. Теперь создайте JS-файл с именем `server.js` и добавьте следующий код:

   ```JavaScript
   const express = require('express');
   const morgan = require('morgan');
   const path = require('path');

   //initialize express.
   const app = express();

   // Initialize variables.
   const port = 3000; // process.env.PORT || 3000;

   // Configure morgan module to log all requests.
   app.use(morgan('dev'));

   // Set the front-end folder to serve public assets.
   app.use(express.static('JavaScriptSPA'))

   // Set up a route for index.html.
   app.get('*', function (req, res) {
       res.sendFile(path.join(__dirname + '/index.html'));
   });

   // Start the server.
   app.listen(port);
   console.log('Listening on port ' + port + '...');
   ```

Теперь у вас есть простой сервер для обслуживания одностраничного приложения (SPA). Ниже приведена предполагаемая структура папок после выполнения инструкций этого учебника.

![изображение с текстом предполагаемой структуры папок SPA](./media/tutorial-v2-javascript-spa/single-page-application-folder-structure.png)

## <a name="create-the-spa-ui"></a>Создание пользовательского интерфейса одностраничного приложения

1. Создайте файл `index.html` для своего одностраничного приложения JavaScript. Этот файл реализует пользовательский интерфейс, созданный на **платформе Bootstrap 4**, и импортирует файлы скриптов для настройки, проверки подлинности и вызова API.

   Добавьте в файл `index.html` указанный ниже код:

   ```html
   <!DOCTYPE html>
   <html lang="en">
     <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0, shrink-to-fit=no">
       <title>Quickstart | MSAL.JS Vanilla JavaScript SPA</title>

       <!-- msal.js with a fallback to backup CDN -->
       <script type="text/javascript" src="https://alcdn.msauth.net/lib/1.2.1/js/msal.js" integrity="sha384-9TV1245fz+BaI+VvCjMYL0YDMElLBwNS84v3mY57pXNOt6xcUYch2QLImaTahcOP" crossorigin="anonymous"></script>
       <script type="text/javascript">
         if(typeof Msal === 'undefined')document.write(unescape("%3Cscript src='https://alcdn.msftauth.net/lib/1.2.1/js/msal.js' type='text/javascript' integrity='sha384-m/3NDUcz4krpIIiHgpeO0O8uxSghb+lfBTngquAo2Zuy2fEF+YgFeP08PWFo5FiJ' crossorigin='anonymous'%3E%3C/script%3E"));
       </script>

       <!-- adding Bootstrap 4 for UI components  -->
       <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
     </head>
     <body>
       <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
         <a class="navbar-brand" href="/">MS Identity Platform</a>
         <div class="btn-group ml-auto dropleft">
           <button type="button" id="signIn" class="btn btn-secondary" onclick="signIn()">Sign In</button>
           <button type="button" id="signOut" class="btn btn-success d-none" onclick="signOut()">Sign Out</button>
       </div>
       </nav>
       <br>
       <h5 class="card-header text-center">Vanilla JavaScript SPA calling MS Graph API with MSAL.JS</h5>
       <br>
       <div class="row" style="margin:auto" >
       <div id="card-div" class="col-md-3 d-none">
       <div class="card text-center">
         <div class="card-body">
           <h5 class="card-title" id="welcomeMessage">Please sign-in to see your profile and read your mails</h5>
           <div id="profile-div"></div>
           <br>
           <br>
           <button class="btn btn-primary" id="seeProfile" onclick="seeProfile()">See Profile</button>
           <br>
           <br>
           <button class="btn btn-primary d-none" id="readMail" onclick="readMail()">Read Mails</button>
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

       <!-- replace next line with authRedirect.js if you would like to use the redirect flow -->
       <!-- <script type="text/javascript" src="./authRedirect.js"></script>   -->
       <script type="text/javascript" src="./authPopup.js"></script>
       <script type="text/javascript" src="./graph.js"></script>
     </body>
   </html>
   ```

   > [!TIP]
   > Приведенную в сценарии выше версию MSAL.js можно заменить последней выпущенной версией в разделе с [выпусками MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases).

2. Теперь создайте JS-файл с именем `ui.js` для получения доступа к элементам DOM и их обновления и добавьте следующий код:

   ```JavaScript
   // Select DOM elements to work with
   const welcomeDiv = document.getElementById("welcomeMessage");
   const signInButton = document.getElementById("signIn");
   const signOutButton = document.getElementById('signOut');
   const cardDiv = document.getElementById("card-div");
   const mailButton = document.getElementById("readMail");
   const profileButton = document.getElementById("seeProfile");
   const profileDiv = document.getElementById("profile-div");

   function showWelcomeMessage(account) {
     // Reconfiguring DOM elements
     cardDiv.classList.remove('d-none');
     welcomeDiv.innerHTML = `Welcome ${account.name}`;
     signInButton.classList.add('d-none');
     signOutButton.classList.remove('d-none');
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

Прежде чем настроить проверку подлинности, зарегистрируйте приложение в **Azure Active Directory**.

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
1. Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
1. Введите значение **Name** (Имя) для приложения. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт).
1. В разделе **URI перенаправления** выберите в раскрывающемся списке **Веб-платформа**, а затем задайте значение URL-адреса приложения, размещенного на вашем веб-сервере.
1. Выберите **Зарегистрировать**.
1. На странице приложения **Обзор** запишите **идентификатор приложения (клиента)** для использования в будущем.
1. В разделе **Управление** выберите **Проверка подлинности**.
1. В разделе **Implicit grant and hybrid flows** (Неявное предоставление разрешения и гибридные потоки) выберите элементы **Токен идентификатора** и **Маркеры доступа**. Токены идентификатора и маркеры доступа необходимы, так как это приложение должно обрабатывать вход пользователей и вызов API.
1. Щелкните **Сохранить**.

> ### <a name="set-a-redirect-url-for-nodejs"></a>Настройка URL-адреса перенаправления для Node.js
>
> При использовании Node.js порт веб-сервера можно задать в файле *server.js*. В этом учебнике используется порт 3000, но вы можете указать любой другой доступный порт.
>
> Чтобы настроить URL-адрес перенаправления в сведениях о регистрации приложения, снова перейдите на панель **Регистрация приложения** и выполните одно из следующих действий:
>
> - Установите значение *`http://localhost:3000/`* в качестве **URL-адреса перенаправления**.
> - Если вы используете настраиваемый TCP-порт, используйте *`http://localhost:<port>/`* (где *\<port>*  — это номер пользовательского TCP-порта).
>   1. Скопируйте значение **URL-адреса**.
>   1. Вернитесь на панель **Регистрация приложения** и вставьте скопированное значение как **URL-адрес перенаправления**.
>

### <a name="configure-your-javascript-spa"></a>Настройка одностраничного приложения JavaScript

Создайте JS-файл с именем `authConfig.js`, который будет содержать параметры конфигурации для проверки подлинности, и добавьте следующий код:

```javascript
  const msalConfig = {
    auth: {
      clientId: "Enter_the_Application_Id_Here",
      authority: "Enter_the_Cloud_Instance_Id_HereEnter_the_Tenant_Info_Here",
      redirectUri: "Enter_the_Redirect_Uri_Here",
    },
    cache: {
      cacheLocation: "sessionStorage", // This configures where your cache will be stored
      storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
    }
  };

  // Add here scopes for id token to be used at MS Identity Platform endpoints.
  const loginRequest = {
    scopes: ["openid", "profile", "User.Read"]
  };

  // Add here scopes for access token to be used at MS Graph API endpoints.
  const tokenRequest = {
    scopes: ["Mail.Read"]
  };
```

 Где:
 - *\<Enter_the_Application_Id_Here>*  — это **идентификатор приложения (клиента)** для зарегистрированного приложения.
 - *\<Enter_the_Cloud_Instance_Id_Here>* представляет экземпляр облака Azure. Для основного или глобального облака Azure просто введите *https://login.microsoftonline.com* . Сведения для **национальных** облаков (например, Китая) см. в [этой статье](./authentication-national-cloud.md).
 - *\<Enter_the_Tenant_info_here>* может иметь несколько значений:
   - Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение **идентификатором клиента** или **именем клиента** (например, *contoso.microsoft.com*).
   - Если ваше приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на **organizations**.
   - Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на **common**. Чтобы ограничить поддержку только *личными учетными записями Майкрософт*, замените это значение на **consumers**.


## <a name="use-the-microsoft-authentication-library-msal-to-sign-in-the-user"></a>Использование библиотеки аутентификации Майкрософт (MSAL) для входа пользователя

Создайте JS-файл с именем `authPopup.js`, который будет содержать логику проверки подлинности и получения маркера, и добавьте следующий код:

   ```JavaScript
   const myMSALObj = new Msal.UserAgentApplication(msalConfig);

   function signIn() {
     myMSALObj.loginPopup(loginRequest)
       .then(loginResponse => {
         console.log('id_token acquired at: ' + new Date().toString());
         console.log(loginResponse);

         if (myMSALObj.getAccount()) {
           showWelcomeMessage(myMSALObj.getAccount());
         }
       }).catch(error => {
         console.log(error);
       });
   }

   function signOut() {
     myMSALObj.logout();
   }

   function callMSGraph(theUrl, accessToken, callback) {
       var xmlHttp = new XMLHttpRequest();
       xmlHttp.onreadystatechange = function () {
           if (this.readyState == 4 && this.status == 200) {
              callback(JSON.parse(this.responseText));
           }
       }
       xmlHttp.open("GET", theUrl, true); // true for asynchronous
       xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
       xmlHttp.send();
   }

   function getTokenPopup(request) {
     return myMSALObj.acquireTokenSilent(request)
       .catch(error => {
         console.log(error);
         console.log("silent token acquisition fails. acquiring token using popup");

         // fallback to interaction when silent call fails
           return myMSALObj.acquireTokenPopup(request)
             .then(tokenResponse => {
               return tokenResponse;
             }).catch(error => {
               console.log(error);
             });
       });
   }

   function seeProfile() {
     if (myMSALObj.getAccount()) {
       getTokenPopup(loginRequest)
         .then(response => {
           callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
           profileButton.classList.add('d-none');
           mailButton.classList.remove('d-none');
         }).catch(error => {
           console.log(error);
         });
     }
   }

   function readMail() {
     if (myMSALObj.getAccount()) {
       getTokenPopup(tokenRequest)
         .then(response => {
           callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
         }).catch(error => {
           console.log(error);
         });
     }
   }
   ```

### <a name="more-information"></a>Дополнительные сведения

Когда пользователь впервые нажимает кнопку **Войти**, метод `signIn` вызывает `loginPopup`, чтобы пользователь мог выполнить вход. Этот метод открывает всплывающее окно с *конечной точкой платформы удостоверений Майкрософт*, чтобы запросить и проверить учетные данные пользователя. После успешного входа пользователь перенаправляется обратно на исходную страницу *index.html*. Полученный маркер обрабатывается в `msal.js`, а сведения из него кэшируются. Этот маркер известен как *маркер идентификатора*. Он содержит основные сведения о пользователе, такие как отображаемое имя пользователя. Если вы планируете использовать какие-либо данные, предоставляемые этим маркером, убедитесь, что маркер проверен внутренним сервером. Это позволит гарантировать, что маркер был выдан допустимому пользователю для вашего приложения.

Одностраничное приложение, создаваемое в рамках этого руководства вызывает `acquireTokenSilent` и (или) `acquireTokenPopup`, чтобы получить *маркер доступа*, используемый при запросе API Microsoft Graph для получения сведений из профиля пользователя. Если вам нужен пример, проверяющий маркер идентификатора, см. [пример приложения на GitHub](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2 "Пример на GitHub: active-directory-javascript-singlepageapp-dotnet-webapi-v2"). В нем для проверки маркеров используется веб-API ASP.NET.

#### <a name="get-a-user-token-interactively"></a>Интерактивное получение маркера пользователя

После первоначального входа вам не нужно требовать от пользователей повторно выполнять аутентификацию при каждом запросе маркера для доступа к ресурсу. Это означает, что в большинстве случаев для получения маркеров следует использовать *acquireTokenSilent*. Но иногда требуется настроить принудительное взаимодействие пользователей с платформой удостоверений Майкрософт. Примеры приведены ниже.

- Пользователям нужно повторно вводить учетные данные, когда истекает срок действия пароля.
- Приложение запрашивает доступ к ресурсу, на обращение к которому пользователь должен дать согласие.
- Требуется двухфакторная проверка подлинности.

При вызове *acquireTokenPopup* открывается всплывающее окно (или же *acquireTokenRedirect*; при этом пользователи перенаправляются на платформу удостоверений Майкрософт). В этом окне пользователям необходимо подтвердить учетные данные, предоставить согласие на требуемый ресурс или выполнить двухфакторную проверку подлинности.

#### <a name="get-a-user-token-silently"></a>Автоматическое получение маркера пользователя

Метод `acquireTokenSilent` отвечает за получение и обновление маркера без участия пользователя. После первого выполнения метода `loginPopup` или `loginRedirect` обычно используется метод `acquireTokenSilent`, чтобы получить маркеры для доступа к защищенным ресурсам для последующих вызовов. (Вызовы для запроса или обновления маркеров не требуют взаимодействия с пользователем.) В некоторых случаях выполнение `acquireTokenSilent` может завершаться сбоем. Например, так происходит, когда истекает срок действия пароля пользователя. Приложение может обработать это исключение двумя способами:

1. Немедленно вызвать `acquireTokenPopup`, чтобы отобразить запрос на вход для пользователя. Этот шаблон обычно используется в интерактивных приложениях, где пользователю недоступно не прошедшее проверку подлинности содержимое. Пример, созданный в ходе пошаговой настройки, использует этот шаблон.

1. Приложения также могут визуально уведомить пользователя, что требуется интерактивный вход, чтобы пользователь мог выбрать подходящее время для входа или приложение могло повторить метод `acquireTokenSilent` ​​позднее. Обычно это применимо в тех случаях, когда пользователь может использовать другие функции приложения, на которые это не влияет. Например, в приложении есть содержимое, для доступа к аутентификация не требуется. В этой ситуации пользователь может самостоятельно решить, когда выполнять вход для получения доступа к защищенному ресурсу или обновления устаревших данных.

> [!NOTE]
> В этом кратком руководстве по умолчанию используются методы `loginPopup` и `acquireTokenPopup`. Если в качестве браузера вы используете Internet Explorer, мы рекомендуем использовать методы `loginRedirect` и `acquireTokenRedirect` в связи с [известной проблемой](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) со способом работы всплывающих окон Internet Explorer. Если вы хотите узнать, как добиться того же результата с помощью `Redirect methods`, см. [эту страницу](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/blob/quickstart/JavaScriptSPA/authRedirect.js).

## <a name="call-the-microsoft-graph-api-by-using-the-token-you-just-acquired"></a>Вызов API Microsoft Graph с помощью полученного маркера

1. Сначала создайте JS-файл с именем `graphConfig.js`, в котором будут храниться конечные точки REST. Добавьте следующий код:

   ```JavaScript
      const graphConfig = {
        graphMeEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me",
        graphMailEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me/messages"
      };
   ```

   Где:
   - *\<Enter_the_Graph_Endpoint_Here>*  — экземпляр API MS Graph. Для глобальной конечной точки API Microsoft Graph просто замените эту строку на `https://graph.microsoft.com`. Сведения о национальных облачных развертываниях см. [здесь](/graph/deployments).

1. Затем создайте JS-файл с именем `graph.js` для вызова REST к API Microsoft Graph и добавьте следующий код:

   ```javascript
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
       .catch(error => console.log(error))
   }
   ```

### <a name="more-information-about-making-a-rest-call-against-a-protected-api"></a>Дополнительные сведения о вызове REST через защищенный API

В примере приложения, созданном с помощью этого руководства, метод `callMSGraph()` выполняет HTTP-запрос `GET` к защищенному ресурсу, которому требуется маркер. Затем метод возвращает содержимое вызывающему объекту. Этот метод добавляет полученный маркер в *заголовок авторизации HTTP*. Для примера приложения, созданного с помощью этого руководства, ресурсом является конечная точка *me* из API Microsoft Graph, которая отображает сведения о профиле пользователя.

## <a name="test-your-code"></a>Тестирование кода

1. Настройте для сервера ожидание передачи данных через TCP-порт на основе расположения файла *index.html*. Чтобы запустить для Node.js веб-сервер для прослушивания порта, выполните следующие команды в командной строке из папки приложения:

   ```bash
   npm install
   npm start
   ```
1. В браузере введите **http://localhost:3000** или **http://localhost:{port}** , где *port* — это порт, который прослушивает ваш веб-сервер. Появится содержимое файла *index.html* с кнопкой **Sign In** (Войти).

После загрузки файла *index.html* в браузер нажмите кнопку **Sign In** (Войти). Вам будет предложено войти с помощью платформы удостоверений Майкрософт:

![Окно входа учетной записи JavaScript SPA](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspascreenshot1.png)

### <a name="provide-consent-for-application-access"></a>Предоставление разрешения на доступ к приложению

При первом входе в приложение вам будет предложено предоставить ему доступ к профилю, а также выполнить вход:

![Окно "Запрошенные разрешения"](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspaconsent.png)

### <a name="view-application-results"></a>Просмотр результатов приложения

После входа сведения о профиле пользователя будут получены в отображаемом ответе API Microsoft Graph:

![Результаты после вызова Microsoft API Graph](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptsparesults.png)

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Дополнительные сведения об областях и делегированных разрешениях

Для чтения профиля пользователя API Microsoft Graph требуется область *user.read*. По умолчанию эта область автоматически добавляется в каждое приложение, зарегистрированное на портале регистрации. Для других API Microsoft Graph, а также для пользовательских API вашего внутреннего сервера, могут потребоваться дополнительные области. Например, для отображения почтового списка пользователя API Microsoft Graph требуется область *Mail.Read*.

> [!NOTE]
> При увеличении количества областей от пользователя могут потребоваться дополнительные согласия.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать больше о разработке одностраничных приложений (SPA) на платформе удостоверений Майкрософт, ознакомьтесь с нашим циклом из нескольких статей.

> [!div class="nextstepaction"]
> [Scenario: одностраничное приложение](scenario-spa-overview.md)
