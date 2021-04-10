---
title: Краткое руководство. Вход пользователей в одностраничных приложениях JavaScript | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как приложение JavaScript может вызывать API, которому требуются маркеры доступа, выданные платформой удостоверений Майкрософт.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 04/11/2019
ms.author: nacanuma
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:JavaScript, devx-track-js
ms.openlocfilehash: 8e35342bd704f662d41f676f58e2cc14b54f29a8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105023390"
---
# <a name="quickstart-sign-in-users-and-get-an-access-token-in-a-javascript-spa"></a>Краткое руководство. Вход пользователей и получение маркера доступа в SPA JavaScript

При работе с этим кратким руководством вы скачаете и выполните пример кода. Такой пример кода, демонстрирует, как в одностраничном приложении (SPA) JavaScript реализовать вход пользователей и вызов Microsoft Graph, а также как получить маркер доступа для вызова API Microsoft Graph или любого веб-API.

Иллюстрацию см. в разделе [Как работает этот пример](#how-the-sample-works).

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) (для внесения правок в файлы проекта).

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>Регистрация и скачивание приложения, используемого в этом кратком руководстве
> Чтобы запустить приложение, используемое в этом кратком руководстве, выберите любой из следующих вариантов.
>
> ### <a name="option-1-express-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Вариант 1 (экспресс-способ). Регистрация и автоматическая настройка приложения, а затем скачивание примера кода
>
> 1. Откройте страницу <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType/JavascriptSpaQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
> 1. Введите имя приложения.
> 1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт).
> 1. Выберите **Зарегистрировать**.
> 1. Следуйте инструкциям, чтобы быстро скачать и автоматически настроить новое приложение.
>
> ### <a name="option-2-manual-register-and-manually-configure-your-application-and-code-sample"></a>Вариант 2 (вручную). Регистрация и настройка приложения и примера кода вручную
>
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Введите значение **Name** (Имя) для приложения. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
> 1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт).
> 1. Выберите **Зарегистрировать**. На странице приложения **Обзор** запишите **идентификатор приложения (клиента)** для использования в будущем.
> 1. Для этого краткого руководства должно быть включено [неявное предоставление разрешения потока](v2-oauth2-implicit-grant-flow.md). В разделе **Управление** выберите **Проверка подлинности**.
> 1. Выберите в разделе **Конфигурации платформ** > **Добавить платформу**. Выберите **Интернет**.
> 1. В поле **URI перенаправления** укажите значение `http://localhost:3000/`. 
> 1. Выберите **Маркеры доступа** и **Маркеры идентификации** в разделе **Неявное предоставление разрешения и гибридные потоки**.
> 1. Нажмите кнопку **Настроить**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Чтобы пример кода, приведенный в этом кратком руководстве, работал, добавьте **URI перенаправления** `http://localhost:3000/` и включите **неявное предоставление разрешений**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-javascript/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-project"></a>Шаг 2. Скачивание проекта

> [!div renderon="docs"]
> [Скачайте основные файлы проекта](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip), чтобы запустить проект с помощью веб-сервера, используя Node.js.

> [!div renderon="portal" class="sxs-lookup"]
> Запустите проект с помощью веб-сервера, используя Node.js.

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip)

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-javascript-app"></a>Шаг 3. Настройка приложения JavaScript
>
> В папке *JavaScriptSPA* измените файл *authConfig.js* и установите значения `clientID`, `authority` и `redirectUri` в разделе `msalConfig`.
>
> ```javascript
>
>  // Config object to be passed to Msal on creation
>  const msalConfig = {
>    auth: {
>      clientId: "Enter_the_Application_Id_Here",
>      authority: "Enter_the_Cloud_Instance_Id_Here/Enter_the_Tenant_Info_Here",
>      redirectUri: "Enter_the_Redirect_Uri_Here",
>    },
>    cache: {
>      cacheLocation: "sessionStorage", // This configures where your cache will be stored
>      storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
>    }
>  };
>
>```

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
>
> Где:
> - `Enter_the_Application_Id_Here` содержит **идентификатор приложения (клиента)** для зарегистрированного приложения.
>
>    Чтобы найти значение параметра **Идентификатор приложения (клиента)** , на портале Azure перейдите на страницу **Обзор** приложения.
> - `Enter_the_Cloud_Instance_Id_Here` представляет экземпляр облака Azure. Для основного или глобального облака Azure просто введите `https://login.microsoftonline.com/` . Сведения для **национальных** облаков (например, Китая) см. в [этой статье](./authentication-national-cloud.md).
> - `Enter_the_Tenant_info_here` может иметь несколько значений:
>    - Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение **идентификатором клиента** или **именем клиента** (например, `contoso.microsoft.com`).
>
>    Чтобы найти значение параметра **Идентификатор каталога (арендатора)** , на портале Azure перейдите на страницу **Обзор** регистрации приложения.
>    - Если приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на `organizations`.
>    - Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на `common`. Чтобы ограничить поддержку только *личными учетными записями Microsoft*, замените это значение на `consumers`.
>
>    Чтобы найти значение параметра **Поддерживаемые типы учетных записей**, на портале Azure перейдите на страницу **Обзор** регистрации приложения.
> - Параметр `Enter_the_Redirect_Uri_Here` равен `http://localhost:3000/`.
>
> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
> Мы настроили проект, указав значения свойств приложения.

> [!div renderon="docs"]
>
> Затем в той же папке измените файл *graphConfig.js*, задав значения `graphMeEndpoint` и `graphMeEndpoint` для объекта `apiConfig`.
> ```javascript
>   // Add here the endpoints for MS Graph API services you would like to use.
>   const graphConfig = {
>     graphMeEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me",
>     graphMailEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me/messages"
>   };
>
>   // Add here scopes for access token to be used at MS Graph API endpoints.
>   const tokenRequest = {
>       scopes: ["Mail.Read"]
>   };
> ```
>

> [!div renderon="docs"]
>
> Где:
> - *\<Enter_the_Graph_Endpoint_Here>* обозначает конечную точку, к которой будут направляться вызовы API. Для основной или глобальной службы API Microsoft Graph просто введите `https://graph.microsoft.com/`. Дополнительные сведения см. в статье [Национальные облачные развертывания](/graph/deployments).
>
> #### <a name="step-4-run-the-project"></a>Шаг 4. Запуск проекта

Запустите проект с помощью веб-сервера, используя [Node.js](https://nodejs.org/en/download/):

1. Чтобы запустить сервер, выполните в каталоге проекта следующую команду.
    ```cmd
    npm install
    npm start
    ```
1. Откройте веб-браузер и перейдите на страницу `http://localhost:3000/`.

1. Нажмите кнопку **Войти**, чтобы войти в систему, а затем вызовите API Microsoft Graph.

Когда приложение загрузится в браузере, нажмите кнопку **Войти**. После первого входа предоставьте приложению разрешение на использование данных вашего профиля для входа. После выполнения входа сведения о вашем профиле пользователя должны отображаться на странице.

## <a name="more-information"></a>Дополнительные сведения

### <a name="how-the-sample-works"></a>Как работает этот пример

![Принцип работы примера JavaScript SPA: 1. SPA инициирует вход. 2. SPA получает токен идентификатора от платформы удостоверений Майкрософт. 3. SPA вызывает получение токена. 4. Платформа удостоверений Майкрософт возвращает токен доступа в SPA. 5. SPA принимает и HTTP-запрос GET с токеном доступа к Microsoft API Graph. 6. API Graph возвращает ответ HTTP в SPA.](media/quickstart-v2-javascript/javascriptspa-intro.svg)

### <a name="msaljs"></a>msal.js

MSAL — это библиотека, используемая для выполнения входа пользователей и запросов маркеров, которые нужны для доступа к API, защищенному платформой удостоверений Майкрософт. В файле *index.html*, используемом в кратком руководстве, содержится ссылка на библиотеку:

```html
<script type="text/javascript" src="https://alcdn.msftauth.net/lib/1.2.1/js/msal.js" integrity="sha384-9TV1245fz+BaI+VvCjMYL0YDMElLBwNS84v3mY57pXNOt6xcUYch2QLImaTahcOP" crossorigin="anonymous"></script>
```

Приведенную выше версию можно заменить последней выпущенной версией в разделе с [выпусками MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases).

Кроме того, если вы установили Node.js, последнюю версию можно скачать с помощью диспетчера пакетов Node.js (NPM):

```cmd
npm install msal
```

### <a name="msal-initialization"></a>Инициализация MSAL

В примере кода ниже показано, как инициализировать библиотеку MSAL:

```javascript
  // Config object to be passed to Msal on creation
  const msalConfig = {
    auth: {
      clientId: "75d84e7a-40bx-f0a2-91b9-0c82d4c556aa", // this is a fake id
      authority: "https://login.microsoftonline.com/common",
      redirectUri: "http://localhost:3000/",
    },
    cache: {
      cacheLocation: "sessionStorage", // This configures where your cache will be stored
      storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
    }
  };

const myMSALObj = new Msal.UserAgentApplication(msalConfig);
```

|Where  | Описание |
|---------|---------|
|`clientId`     | Идентификатор приложения, зарегистрированного на портале Azure.|
|`authority`    | (Необязательно.) Это URL-адрес центра сертификации, предназначенный для поддержки типов учетных записей, как описано выше в разделе конфигурации. По умолчанию центром является `https://login.microsoftonline.com/common`. |
|`redirectUri`     | URI-перенаправления и URL-адрес ответа регистрации приложения. В этом случае — `http://localhost:3000/`. |
|`cacheLocation`  | (Необязательно.) Этот параметр настраивает хранилище браузера для состояния проверки подлинности. По умолчанию используется sessionStorage.   |
|`storeAuthStateInCookie`  | (Необязательно.) Библиотека, которая хранит состояние запроса проверки подлинности, необходимое для проверки потоков аутентификации, в файлах cookie браузера. Параметр файла cookie установлен для браузеров IE и Edge, чтобы уменьшить определенные [известные проблемы](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues). |

Подробные сведения о доступных настраиваемых параметрах см. в статье [Initialize client applications using MSAL.js](msal-js-initializing-client-applications.md) (Инициализация клиентских приложений с помощью MSAL.js).

### <a name="sign-in-users"></a>Выполнение входа пользователей

В приведенном ниже фрагменте кода показано, как выполнить вход пользователей:

```javascript
// Add scopes for the id token to be used at Microsoft identity platform endpoints.
const loginRequest = {
    scopes: ["openid", "profile", "User.Read"],
};

myMSALObj.loginPopup(loginRequest)
    .then((loginResponse) => {
    //Login Success callback code here
}).catch(function (error) {
    console.log(error);
});
```

|Where  | Описание |
|---------|---------|
| `scopes`   | (Необязательно.) Содержит области, для которых запрашивается согласие пользователя во время входа в систему. Например, `[ "user.read" ]` для Microsoft Graph или `[ "<Application ID URL>/scope" ]` для пользовательских веб-API (то есть `api://<Application ID>/access_as_user`). |

Можно также использовать метод `loginRedirect`, чтобы с текущей страницы выполнялось перенаправление на страницу входа вместо отображения всплывающего окна.

### <a name="request-tokens"></a>Запрос маркеров

В MSAL есть три метода получения маркеров: `acquireTokenRedirect`, `acquireTokenPopup` и `acquireTokenSilent`.

#### <a name="get-a-user-token-silently"></a>Автоматическое получение маркера пользователя

Метод `acquireTokenSilent` обрабатывает получение и обновление маркера без участия пользователя. После первого выполнения метода `loginRedirect` или `loginPopup` обычно используется метод `acquireTokenSilent`, чтобы получить маркеры для доступа к защищенным ресурсам для последующих вызовов. Вызовы для запроса или обновления маркеров выполняются автоматически.

```javascript

const tokenRequest = {
    scopes: ["Mail.Read"]
};

myMSALObj.acquireTokenSilent(tokenRequest)
    .then((tokenResponse) => {
        // Callback code here
        console.log(tokenResponse.accessToken);
    }).catch((error) => {
        console.log(error);
    });
```

|Where  | Описание |
|---------|---------|
| `scopes`   | Содержит запрашиваемые области, которые должны быть возвращены в маркере доступа для API. Например, `[ "mail.read" ]` для Microsoft Graph или `[ "<Application ID URL>/scope" ]` для пользовательских веб-API (то есть `api://<Application ID>/access_as_user`).|

#### <a name="get-a-user-token-interactively"></a>Интерактивное получение маркера пользователя

В некоторых ситуациях требуется настроить принудительное взаимодействие пользователей с платформой удостоверений Майкрософт. Пример:
* Пользователям может потребоваться повторно ввести учетные данные, так как истек срок действия пароля.
* Ваше приложение запрашивает доступ к дополнительным областям ресурса, на обращение к которым пользователь должен дать согласие.
* Требуется двухфакторная проверка подлинности.

Обычный рекомендуемый шаблон для большинства приложений — вызов `acquireTokenSilent`, перехват исключения, а затем вызов `acquireTokenPopup` (или `acquireTokenRedirect`) для запуска интерактивного запроса.

Вызов `acquireTokenPopup` приводит к появлению всплывающего окна для входа. (Или `acquireTokenRedirect` приводит к перенаправлению пользователей на платформу удостоверений Майкрософт.) В этом окне пользователям необходимо подтверждать свои учетные данные, давать согласие на требуемый ресурс и выполнять двухфакторную аутентификацию.

```javascript
// Add here scopes for access token to be used at MS Graph API endpoints.
const tokenRequest = {
    scopes: ["Mail.Read"]
};

myMSALObj.acquireTokenPopup(requestObj)
    .then((tokenResponse) => {
        // Callback code here
        console.log(tokenResponse.accessToken);
    }).catch((error) => {
        console.log(error);
    });
```

> [!NOTE]
> В этом кратком руководстве для Microsoft Internet Explorer используются методы `loginRedirect` и `acquireTokenRedirect` из-за [известной проблемы](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) с обработкой всплывающих окон в Internet Explorer.

## <a name="next-steps"></a>Дальнейшие действия

Более подробное пошаговое руководство по созданию приложения для этого краткого руководства см. в следующей статье:

> [!div class="nextstepaction"]
> [Руководство. Вход пользователей и вызов API Microsoft Graph из одностраничного приложения JavaScript (SPA)](tutorial-v2-javascript-spa.md)
