---
title: Получение маркера для вызова веб-API (одностраничные приложения) | Службы
titleSuffix: Microsoft identity platform
description: Сведения о создании одностраничного приложения (получение маркера для вызова API)
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/20/2019
ms.author: negoe
ms.custom: aaddev
ms.openlocfilehash: 83896b2599f03961b2dcaf34ea9b55fe16c13b9e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98756444"
---
# <a name="single-page-application-acquire-a-token-to-call-an-api"></a>Одностраничное приложение: получение маркера для вызова API

Шаблон для получения маркеров для API-интерфейсов с MSAL.jsом — сначала попытайтесь выполнить запрос маркера без уведомления с помощью `acquireTokenSilent` метода. При вызове этого метода библиотека сначала проверяет кэш в хранилище браузера, чтобы узнать, существует ли допустимый маркер, и возвращает его. Если в кэше нет допустимого маркера, он отправляет запрос на автоматическую лексему в Azure Active Directory (Azure AD) из скрытого IFRAME. Этот метод также позволяет библиотеке обновлять маркеры. Дополнительные сведения о сеансах единого входа и значениях времени существования маркеров в Azure AD см. в разделе [время существования маркеров](active-directory-configurable-token-lifetimes.md).

Запросы токенов без уведомления к Azure AD могут завершиться сбоем по причинам, например просроченному сеансу Azure AD или изменению пароля. В этом случае можно вызвать один из интерактивных методов (который будет предлагать пользователю) получить маркеры:

* [Всплывающее окно](#acquire-a-token-with-a-pop-up-window)с помощью `acquireTokenPopup`
* [Перенаправление](#acquire-a-token-with-a-redirect)с помощью `acquireTokenRedirect`

## <a name="choose-between-a-pop-up-or-redirect-experience"></a>Выбор между всплывающим окном или перенаправлением

 В приложении нельзя использовать методы всплывающего окна и перенаправления. Выбор между всплывающим окном или перенаправлением зависит от потока приложения:

* Если вы не хотите, чтобы пользователи перейдут с основной страницы приложения во время проверки подлинности, мы рекомендуем использовать всплывающий метод. Так как перенаправление проверки подлинности происходит во всплывающем окне, состояние основного приложения сохраняется.

* Если у пользователей есть ограничения браузера или политики, в которых отключены всплывающие окна, можно использовать метод перенаправления. Используйте метод Redirect в браузере Internet Explorer, так как существуют [Известные проблемы с всплывающими окнами в Internet Explorer](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser).

Можно задать области API, которые должен включить маркер доступа при создании запроса маркера доступа. Обратите внимание, что в маркере доступа могут быть не предоставлены все запрошенные области. Это зависит от согласия пользователя.

## <a name="acquire-a-token-with-a-pop-up-window"></a>Получение маркера с помощью всплывающего окна

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Следующий код сочетает ранее описанный шаблон с методами для всплывающего окна:

```javascript
const accessTokenRequest = {
    scopes: ["user.read"]
}

userAgentApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    // Call API with token
    let accessToken = accessTokenResponse.accessToken;
}).catch(function (error) {
    //Acquire token silent failure, and send an interactive request
    if (error.errorMessage.indexOf("interaction_required") !== -1) {
        userAgentApplication.acquireTokenPopup(accessTokenRequest).then(function(accessTokenResponse) {
            // Acquire token interactive success
        }).catch(function(error) {
            // Acquire token interactive failure
            console.log(error);
        });
    }
    console.log(error);
});
```

# <a name="angular"></a>[Angular](#tab/angular)

MSAL угловая оболочка предоставляет перехватчик HTTP, который автоматически получает маркеры доступа и прикрепляет их к HTTP-запросам к API.

Области для API-интерфейсов можно указать в `protectedResourceMap` параметре конфигурации. `MsalInterceptor` запрашивает эти области при автоматическом получении маркеров.

```javascript
// app.module.ts
@NgModule({
  declarations: [
    // ...
  ],
  imports: [
    // ...
    MsalModule.forRoot({
      auth: {
        clientId: 'Enter_the_Application_Id_Here',
      }
    },
    {
      popUp: !isIE,
      consentScopes: [
        'user.read',
        'openid',
        'profile',
      ],
      protectedResourceMap: [
        ['https://graph.microsoft.com/v1.0/me', ['user.read']]
      ]
    })
  ],
  providers: [
    {
      provide: HTTP_INTERCEPTORS,
      useClass: MsalInterceptor,
      multi: true
    }
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

В случае успеха и сбоя при получении автоматического получения маркера MSALный угловой предоставляет обратные вызовы, на которые можно подписываться. Также важно отменить подписывание.

```javascript
// In app.component.ts
 ngOnInit() {
    this.subscription=  this.broadcastService.subscribe("msal:acquireTokenFailure", (payload) => {
    });
}

ngOnDestroy() {
   this.broadcastService.getMSALSubject().next(1);
   if (this.subscription) {
     this.subscription.unsubscribe();
   }
 }
```

Кроме того, можно явно получить токены с помощью методов получения маркера, как описано в основной библиотеке MSAL.js.

---

## <a name="acquire-a-token-with-a-redirect"></a>Получение маркера с перенаправлением

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Следующий шаблон описан ранее, но показан с помощью метода Redirect для получения маркеров в интерактивном режиме. Вам потребуется зарегистрировать обратный вызов перенаправления, как упоминалось ранее.

```javascript
function authCallback(error, response) {
    // Handle redirect response
}

userAgentApplication.handleRedirectCallback(authCallback);

const accessTokenRequest: AuthenticationParameters = {
    scopes: ["user.read"]
}

userAgentApplication.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // Acquire token silent success
    // Call API with token
    let accessToken = accessTokenResponse.accessToken;
}).catch(function (error) {
    //Acquire token silent failure, and send an interactive request
    console.log(error);
    if (error.errorMessage.indexOf("interaction_required") !== -1) {
        userAgentApplication.acquireTokenRedirect(accessTokenRequest);
    }
});
```

## <a name="request-optional-claims"></a>Запрос необязательных утверждений

Необязательные утверждения можно использовать в следующих целях:

- Включите дополнительные утверждения в токены для вашего приложения.
- изменить поведение определенных утверждений в токенах, возвращаемых Azure AD;
- добавлять пользовательские утверждения для приложения и обращаться к ним.

Чтобы запросить необязательные утверждения в `IdToken` , можно отправить объект утверждений переведенные в `claimsRequest` поле `AuthenticationParameters.ts` класса.

```javascript
"optionalClaims":
   {
      "idToken": [
            {
                  "name": "auth_time",
                  "essential": true
             }
      ],

var request = {
    scopes: ["user.read"],
    claimsRequest: JSON.stringify(claims)
};

myMSALObj.acquireTokenPopup(request);
```

Дополнительные сведения см. в разделе [необязательные утверждения](active-directory-optional-claims.md).

# <a name="angular"></a>[Angular](#tab/angular)

Этот код аналогичен описанному выше.

---

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [вызвав веб-API](scenario-spa-call-api.md).