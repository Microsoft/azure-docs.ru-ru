---
title: Вход в одностраничное приложение & выходом
titleSuffix: Microsoft identity platform
description: Узнайте, как создать одностраничное приложение (вход)
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/11/2020
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: bdfffbf8d2e416c87dd5abb8f6383c58ad270231
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99584369"
---
# <a name="single-page-application-sign-in-and-sign-out"></a>Одностраничное приложение: вход и выход

Узнайте, как добавить вход в код для приложения с одной страницей.

Прежде чем можно будет получить маркеры для доступа к API в приложении, необходим контекст пользователя, прошедшего проверку подлинности. Вход пользователей в приложение можно выполнять в MSAL.js двумя способами:

* [Всплывающее окно](#sign-in-with-a-pop-up-window)с помощью `loginPopup` метода
* [Перенаправление](#sign-in-with-redirect)с помощью `loginRedirect` метода

При необходимости можно также передать области API, для которых требуется согласие пользователя во время входа в систему.

> [!NOTE]
> Если у приложения уже есть доступ к контексту пользователя, прошедшему проверку подлинности, или маркеру идентификации, можно пропустить шаг входа и получить токены напрямую. Дополнительные сведения см. в разделе [SSO без MSAL.js login](msal-js-sso.md#sso-without-msaljs-login).

## <a name="choosing-between-a-pop-up-or-redirect-experience"></a>Выбор между всплывающим окном или перенаправлением

В приложении нельзя использовать методы всплывающего окна и перенаправления. Выбор между всплывающим окном или перенаправлением зависит от потока приложения:

* Если вы не хотите, чтобы пользователи переходят со страницы основного приложения во время проверки подлинности, рекомендуется использовать метод всплывающего окна. Так как перенаправление проверки подлинности происходит во всплывающем окне, состояние основного приложения сохраняется.

* Если у пользователей есть ограничения браузера или политики, в которых отключены всплывающие окна, можно использовать метод перенаправления. Используйте метод Redirect в браузере Internet Explorer, так как существуют [Известные проблемы с всплывающими окнами в Internet Explorer](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser).

## <a name="sign-in-with-a-pop-up-window"></a>Вход с помощью всплывающего окна


# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2. x)](#tab/javascript2)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

let username = "";

const myMsal = new PublicClientApplication(config);

myMsal.loginPopup(loginRequest)
    .then(function (loginResponse) {
        //login success

        // In case multiple accounts exist, you can select
        const currentAccounts = myMsal.getAllAccounts();
    
        if (currentAccounts === null) {
            // no accounts detected
        } else if (currentAccounts.length > 1) {
            // Add choose account code here
        } else if (currentAccounts.length === 1) {
            username = currentAccounts[0].username;
        }
    
    }).catch(function (error) {
        //login failure
        console.log(error);
    });
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1. x)](#tab/javascript1)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

const myMsal = new UserAgentApplication(config);

myMsal.loginPopup(loginRequest)
    .then(function (loginResponse) {
        //login success
    }).catch(function (error) {
        //login failure
        console.log(error);
    });
```

# <a name="angular"></a>[Angular](#tab/angular)

Угловая оболочка MSAL позволяет защищать определенные маршруты в приложении, добавляя `MsalGuard` к определению маршрута. Это условие вызывает метод для входа при доступе к этому маршруту.

```javascript
// In app-routing.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { ProfileComponent } from './profile/profile.component';
import { MsalGuard } from '@azure/msal-angular';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  {
    path: 'profile',
    component: ProfileComponent,
    canActivate: [
      MsalGuard
    ]
  },
  {
    path: '',
    component: HomeComponent
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes, { useHash: false })],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

Для интерфейса всплывающего окна включите `popUp` параметр конфигурации. Вы также можете передать области, требующие согласия, следующим образом:

```javascript
// In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id',
            }
        }, {
            popUp: true,
            consentScopes: ["User.ReadWrite"]
        })
    ]
})
```
---

## <a name="sign-in-with-redirect"></a>Вход с перенаправлением

# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2. x)](#tab/javascript2)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

let username = "";

const myMsal = new PublicClientApplication(config);

function handleResponse(response) {
    //handle redirect response

    // In case multiple accounts exist, you can select
    const currentAccounts = myMsal.getAllAccounts();

    if (currentAccounts === null) {
        // no accounts detected
    } else if (currentAccounts.length > 1) {
        // Add choose account code here
    } else if (currentAccounts.length === 1) {
        username = currentAccounts[0].username;
    }
}

myMsal.handleRedirectPromise(handleResponse);

myMsal.loginRedirect(loginRequest);
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1. x)](#tab/javascript1)

Методы перенаправления не возвращают обещание из-за перемещения из основного приложения. Чтобы обработать возвращенные маркеры и получить к ним доступ, перед вызовом методов перенаправления Зарегистрируйте успешные и ошибочные обратные вызовы.

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

const myMsal = new UserAgentApplication(config);

function authCallback(error, response) {
    //handle redirect response
}

myMsal.handleRedirectCallback(authCallback);

myMsal.loginRedirect(loginRequest);
```

# <a name="angular"></a>[Angular](#tab/angular)

Код описан выше, как описано выше в разделе Вход с помощью всплывающего окна. Поток по умолчанию — Redirect.

---

## <a name="sign-out"></a>Функция выхода

Библиотека MSAL предоставляет `logout` метод, который очищает кэш в хранилище браузера и отправляет запрос на выход в Azure Active Directory (Azure AD). После выхода библиотека по умолчанию перенаправляется обратно на начальную страницу приложения.

Вы можете настроить универсальный код ресурса (URI), на который будет осуществляться перенаправление после выхода, задав параметр `postLogoutRedirectUri` . Этот URI также должен быть зарегистрирован в качестве URI выхода при регистрации приложения.

# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2. x)](#tab/javascript2)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new PublicClientApplication(config);

// you can select which account application should sign out
const logoutRequest = {
    account: myMsal.getAccountByUsername(username)
}

myMsal.logout(logoutRequest);
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1. x)](#tab/javascript1)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new UserAgentApplication(config);

myMsal.logout();
```

# <a name="angular"></a>[Angular](#tab/angular)

```javascript
//In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id',
                postLogoutRedirectUri: "your_app_logout_redirect_uri"
            }
        })
    ]
})

// In app.component.ts
this.authService.logout();
```

---

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [получая маркер для приложения](scenario-spa-acquire-token.md).