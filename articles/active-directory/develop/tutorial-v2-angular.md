---
title: Руководство по Создание приложения Angular, которое использует платформу удостоверений Майкрософт для аутентификации | Azure
titleSuffix: Microsoft identity platform
description: В этом руководстве показано, как создать одностраничное приложение (SPA) Angular, которое использует платформу удостоверений Майкрософт для реализации входа пользователей, и как получить маркер доступа для вызова API Microsoft Graph от имени пользователей.
services: active-directory
author: hamiltonha
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 03/05/2020
ms.author: hahamil
ms.custom: aaddev, identityplatformtop40, devx-track-js
ms.openlocfilehash: 105353598a2af60c407bacf02b4527b2de84e450
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98756148"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-an-angular-single-page-application"></a>Руководство по входу пользователей в систему и вызову API Microsoft Graph из одностраничного приложения Angular

В этом руководстве показано, как создать одностраничное приложение (SPA) Angular, которое поддерживает вход пользователей и вызовы API Microsoft Graph.

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * создание проекта Angular с помощью `npm`;
> * регистрация приложения на портале Azure;
> * добавление кода для поддержки входа и выхода пользователей;
> * добавление кода для вызова API Microsoft Graph;
> * Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

* [Node.js](https://nodejs.org/en/download/) для запуска локального веб-сервера.
* [Visual Studio Code](https://code.visualstudio.com/download) или другой редактор для изменения файлов проекта.

## <a name="how-the-sample-app-works"></a>Как работает пример приложения

![Схема работы примера приложения, создаваемого в этом учебнике](./media/tutorial-v2-angular/diagram-auth-flow-spa-angular.svg)

Пример приложения, созданный по инструкциям из этого руководства, позволяет одностраничному приложению Angular выполнять запрос к API Microsoft Graph или веб-API, принимающему маркеры, выпущенные платформой удостоверений Майкрософт. В нем используется библиотека проверки подлинности Майкрософт (MSAL) для Angular, программа-оболочка для основной библиотеки MSAL.js. MSAL Angular позволяет приложениям Angular 6 и более поздних версий выполнять проверку подлинности корпоративных пользователей с помощью Azure Active Directory (AAD), пользователей учетных записей Майкрософт и пользователей социальных сетей, таких как Facebook, Google, LinkedIn и т. п. Библиотека также позволяет приложениям получать доступ к облачным службам Майкрософт или Microsoft Graph.

В этом сценарии после входа пользователя в систему маркер доступа запрашивается и добавляется в HTTP-запросы с использованием заголовка авторизации. Получение и обновление маркера выполняются MSAL.

### <a name="libraries"></a>Библиотеки

В этом учебнике используется следующая библиотека:

|Библиотека|Описание|
|---|---|
|[msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)|Библиотека проверки подлинности Майкрософт для программы-оболочки JavaScript Angular|

Исходный код библиотеки msal.js можно найти в репозитории [AzureAD/microsoft-authentication-library-for-js](https://github.com/AzureAD/microsoft-authentication-library-for-js) на сайте GitHub.

## <a name="create-your-project"></a>Создание проекта

Создайте приложение Angular, выполнив следующие команды npm:

```bash
npm install -g @angular/cli@8                    # Install the Angular CLI
ng new my-application --routing=true --style=css # Generate a new Angular app
cd my-application                                # Change to the app directory
npm install @angular/material@8 @angular/cdk@8   # Install the Angular Material component library (optional, for UI)
npm install msal @azure/msal-angular             # Install MSAL and MSAL Angular in your application
ng generate component page-name                  # To add a new page (such as a home or profile page)
```

## <a name="register-your-application"></a>Регистрация приложения

Выполните эти [инструкции, чтобы зарегистрировать одностраничное приложение](./scenario-spa-app-registration.md) на портале Azure.

На странице **Обзор** регистрации приложения запишите **идентификатор приложения (клиента)** для использования в будущем.

Зарегистрируйте **URI перенаправления** со значением **http://localhost:4200/** и включите параметры неявного предоставления разрешения.

## <a name="configure-the-application"></a>Настройка приложения

1. В папке *src/app* измените *app.module.ts* и добавьте `MSALModule` в `imports`, а также константу `isIE`:

    ```javascript
    const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;
    @NgModule({
      declarations: [
        AppComponent
      ],
      imports: [
        BrowserModule,
        AppRoutingModule,
        MsalModule.forRoot({
          auth: {
            clientId: 'Enter_the_Application_Id_here', // This is your client ID
            authority: 'Enter_the_Cloud_Instance_Id_Here'/'Enter_the_Tenant_Info_Here', // This is your tenant ID
            redirectUri: 'Enter_the_Redirect_Uri_Here'// This is your redirect URI
          },
          cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE, // Set to true for Internet Explorer 11
          },
        }, {
          popUp: !isIE,
          consentScopes: [
            'user.read',
            'openid',
            'profile',
          ],
          unprotectedResources: [],
          protectedResourceMap: [
            ['https://graph.microsoft.com/v1.0/me', ['user.read']]
          ],
          extraQueryParameters: {}
        })
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    ```

    Замените следующие значения.

    |Имя значения|Сведения|
    |---------|---------|
    |Enter_the_Application_Id_Here|Это значение **идентификатора приложения (клиента)** со страницы **обзора** регистрации приложения. |
    |Enter_the_Cloud_Instance_Id_Here|Это экземпляр облака Azure. Для основного или глобального облака Azure введите **https://login.microsoftonline.com** . Сведения для национальных облаков (например, Китая) см. в [этой статье](./authentication-national-cloud.md).|
    |Enter_the_Tenant_Info_Here| Задайте один из следующих параметров: Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение идентификатором каталога (клиента) или именем клиента (например, **contoso.microsoft.com**). Если ваше приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на **organizations**. Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на **common**. Чтобы ограничить поддержку только *личными учетными записями Майкрософт*, замените это значение на **consumers**. |
    |Enter_the_Redirect_Uri_Here|Замените на **http://localhost:4200** .|

    Подробные сведения о доступных настраиваемых параметрах см. в статье [Initialize client applications using MSAL.js](msal-js-initializing-client-applications.md) (Инициализация клиентских приложений с помощью MSAL.js).

2. В начало того же файла добавьте следующий оператор импорта:

    ```javascript
    import { MsalModule, MsalInterceptor } from '@azure/msal-angular';
    ```

3. Добавьте в начало файла `src/app/app.component.ts` следующие операторы импорта:

    ```javascript
    import { MsalService, BroadcastService } from '@azure/msal-angular';
    import { Component, OnInit } from '@angular/core';
    ```
## <a name="sign-in-a-user"></a>Вход пользователя

Добавьте в `AppComponent` следующий код, чтобы выполнить вход пользователя в систему:

```javascript
export class AppComponent implements OnInit {
    constructor(private broadcastService: BroadcastService, private authService: MsalService) { }

    ngOnInit() { }

    login() {
        const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

        if (isIE) {
          this.authService.loginRedirect({
            extraScopesToConsent: ["user.read", "openid", "profile"]
          });
        } else {
          this.authService.loginPopup({
            extraScopesToConsent: ["user.read", "openid", "profile"]
          });
        }
    }
}
```

> [!TIP]
> Для пользователей браузера Internet Explorer мы рекомендуем использовать `loginRedirect`.

## <a name="acquire-a-token"></a>Получение маркера

### <a name="angular-interceptor"></a>Перехватчик Angular

MSAL Angular предоставляет класс `Interceptor`, который автоматически получает маркеры для исходящих запросов, которые используют клиент Angular `http` для известных защищенных ресурсов.

Во-первых, включите класс `Interceptor` в качестве поставщика для вашего приложения:

```javascript
import { MsalInterceptor, MsalModule } from "@azure/msal-angular";
import { HTTP_INTERCEPTORS, HttpClientModule } from "@angular/common/http";

@NgModule({
    // ...
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: MsalInterceptor,
            multi: true
        }
    ]
}
```

Затем предоставьте `MsalModule.forRoot()` карту защищенных ресурсов в виде `protectedResourceMap` и включите эти области в `consentScopes`. Для URL-адресов, которые вы указываете в коллекции `protectedResourceMap`, учитывается регистр.

```javascript
@NgModule({
  // ...
  imports: [
    // ...
    MsalModule.forRoot({
      auth: {
        clientId: 'Enter_the_Application_Id_here', // This is your client ID
        authority: 'https://login.microsoftonline.com/Enter_the_Tenant_Info_Here', // This is your tenant info
        redirectUri: 'Enter_the_Redirect_Uri_Here' // This is your redirect URI
      },
      cache: {
        cacheLocation: 'localStorage',
        storeAuthStateInCookie: isIE, // Set to true for Internet Explorer 11
      },
    },
    {
      popUp: !isIE,
      consentScopes: [
        'user.read',
        'openid',
        'profile',
      ],
      unprotectedResources: [],
      protectedResourceMap: [
        ['https://graph.microsoft.com/v1.0/me', ['user.read']]
      ],
      extraQueryParameters: {}
    })
  ],
});
```

Наконец, извлеките профиль пользователя с помощью HTTP-запроса:

```JavaScript
const graphMeEndpoint = "https://graph.microsoft.com/v1.0/me";

getProfile() {
  this.http.get(graphMeEndpoint).toPromise()
    .then(profile => {
      this.profile = profile;
    });
}
```

### <a name="acquiretokensilent-acquiretokenpopup-acquiretokenredirect"></a>seekTokenSilent, acquTokenPopup, acquTokenRedirect
В MSAL есть три метода получения маркеров: `acquireTokenRedirect`, `acquireTokenPopup` и `acquireTokenSilent`. Но для приложений Angular мы рекомендуем использовать класс `MsalInterceptor`, как показано в предыдущем разделе.

#### <a name="get-a-user-token-silently"></a>Автоматическое получение маркера пользователя

Метод `acquireTokenSilent` обрабатывает получение и обновление маркера без участия пользователя. После первого выполнения метода `loginRedirect` или `loginPopup` обычно используется метод `acquireTokenSilent` для получения маркеров для доступа к защищенным ресурсам в последующих вызовах. Вызовы для запроса или обновления маркеров выполняются автоматически.

```javascript
const requestObj = {
    scopes: ["user.read"]
};

this.authService.acquireTokenSilent(requestObj).then(function (tokenResponse) {
    // Callback code here
    console.log(tokenResponse.accessToken);
}).catch(function (error) {
    console.log(error);
});
```

В этом коде `scopes` содержит запрашиваемые области, которые должны быть возвращены в маркере доступа для API.

Пример:

* `["user.read"]` для Microsoft Graph;
* `["<Application ID URL>/scope"]` для настраиваемых веб-API (т. е. `api://<Application ID>/access_as_user`).

#### <a name="get-a-user-token-interactively"></a>Интерактивное получение маркера пользователя

Иногда необходимо, чтобы пользователь взаимодействовал платформой удостоверений Майкрософт. Пример:

* Пользователям может потребоваться повторно ввести учетные данные, так как истек срок действия пароля.
* Ваше приложение запрашивает доступ к дополнительным областям ресурса, на обращение к которым пользователь должен дать согласие.
* Требуется двухфакторная проверка подлинности.

Рекомендуемый шаблон для большинства приложений — вызов `acquireTokenSilent`, перехват исключения, а затем вызов `acquireTokenPopup` (или `acquireTokenRedirect`) для запуска интерактивного запроса.

Вызов `acquireTokenPopup` приводит к появлению всплывающего окна входа в систему. Кроме того, можно использовать `acquireTokenRedirect`, что приведет к перенаправлению пользователей на платформу удостоверений Майкрософт. В этом окне пользователям необходимо подтвердить учетные данные, предоставить согласие на требуемый ресурс или выполнить двухфакторную проверку подлинности.

```javascript
  const requestObj = {
      scopes: ["user.read"]
  };

  this.authService.acquireTokenPopup(requestObj).then(function (tokenResponse) {
      // Callback code here
      console.log(tokenResponse.accessToken);
  }).catch(function (error) {
      console.log(error);
  });
```

> [!NOTE]
> В этом кратком руководстве используются методы `loginRedirect` и `acquireTokenRedirect` для Microsoft Internet Explorer из-за [известной проблемы](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) с обработкой всплывающих окон в Internet Explorer.

## <a name="log-out"></a>выход из системы;

Добавьте указанный ниже код, чтобы выполнить выход пользователя из системы.

```javascript
logout() {
  this.authService.logout();
}
```

## <a name="add-ui"></a>Добавление пользовательского интерфейса
Изучите [пример приложения](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular), чтобы понять, как добавить пользовательский интерфейс с помощью библиотеки компонентов Angular Material.

## <a name="test-your-code"></a>Тестирование кода

1.  Запустите веб-сервер для прослушивания порта, выполнив приведенные ниже команды в командной строке из папки приложения:

    ```bash
    npm install
    npm start
    ```
1. В браузере введите **http://localhost:4200** или **http://localhost:{port}** (*port* — это порт, прослушивающий ваш веб-сервер).


### <a name="provide-consent-for-application-access"></a>Предоставление разрешения на доступ к приложению

При первом входе в приложение вам будет предложено предоставить ему доступ к профилю, а также выполнить вход:

![Окно "Запрошенные разрешения"](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspaconsent.png)

## <a name="add-scopes-and-delegated-permissions"></a>Добавление областей и делегированных разрешений

Для чтения профиля пользователя API Microsoft Graph требуется область *user.read*. По умолчанию эта область автоматически добавляется в каждое приложение, зарегистрированное на портале регистрации. Для других API Microsoft Graph, а также для пользовательских API вашего внутреннего сервера, могут потребоваться дополнительные области. Например, для отображения списка календарей пользователя для API Microsoft Graph требуется область *Calendars.Read*.

Чтобы из контекста приложения получить доступ к календарям пользователя, добавьте делегированное разрешение *Calendars.Read* в сведения о регистрации приложения. Затем добавьте область *Calendars.Read* в вызов `acquireTokenSilent`.

>[!NOTE]
>При увеличении количества областей от пользователя могут потребоваться дополнительные согласия.

Если серверному API не требуется область (мы не рекомендуем использовать такую конфигурацию), вы можете применять в вызовах *clientId* в качестве области для получения маркеров.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать больше о разработке одностраничных приложений (SPA) на платформе удостоверений Майкрософт, ознакомьтесь с нашей серией из нескольких статей.

> [!div class="nextstepaction"]
> [Scenario: Одностраничное приложение](scenario-spa-overview.md)
