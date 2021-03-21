---
title: Избегайте перегрузок страниц (MSAL.js) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как избежать перегрузок страниц при автоматическом получении и обновлении маркеров с помощью библиотеки проверки подлинности Майкрософт для JavaScript (MSAL.js).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 05/29/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 0c96d161e55261af1bbe04eae6ead1d245158d02
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98064833"
---
# <a name="avoid-page-reloads-when-acquiring-and-renewing-tokens-silently-using-msaljs"></a>Избегайте перезагрузки страниц при автоматическом получении и обновлении маркеров с помощью MSAL.js
Библиотека проверки подлинности Microsoft для JavaScript (MSAL.js) использует скрытые `iframe` элементы для получения и обновления маркеров в фоновом режиме. Azure AD возвращает маркер обратно в зарегистрированную redirect_uri, указанную в запросе маркера (по умолчанию это корневая страница приложения). Поскольку ответ равен 302, он приводит к возникновению HTML, соответствующему `redirect_uri` загружаемому элементу в `iframe` . Обычно это корневая страница приложения, которая `redirect_uri` приводит к ее перезагрузке.

В других случаях, если переход к корневой странице приложения требует проверки подлинности, это может привести к вложенным `iframe` элементам или `X-Frame-Options: deny` ошибкам.

Поскольку MSAL.js не может отклонить 302, выданный Azure AD, и требуется для обработки возвращенного маркера, он не может предотвратить `redirect_uri` загрузку из в `iframe` .

Чтобы избежать повторной загрузки приложения или других ошибок, вызванных этой ошибкой, используйте эти обходные пути.

## <a name="specify-different-html-for-the-iframe"></a>Укажите другой HTML-код для IFRAME

Задайте `redirect_uri` для свойства конфигурации значение простая страница, для которой не требуется проверка подлинности. Необходимо убедиться, что он совпадает с `redirect_uri` зарегистрированным в портал Azure. Это не повлияет на взаимодействие с пользователем при входе в систему, так как MSAL сохраняет начальную страницу, когда пользователь начинает процесс входа в систему и перенаправляет его в точное расположение после завершения входа.

## <a name="initialization-in-your-main-app-file"></a>Инициализация в главном файле приложения

Если ваше приложение структурировано таким, что существует один центральный файл JavaScript, определяющий инициализацию, маршрутизацию и другие компоненты приложения, можно условно загрузить модули приложения в зависимости от того, загружается ли приложение в `iframe` или нет. Пример:

В AngularJS: app.js

```javascript
// Check that the window is an iframe and not popup
if (window !== window.parent && !window.opener) {
angular.module('todoApp', ['ui.router', 'MsalAngular'])
    .config(['$httpProvider', 'msalAuthenticationServiceProvider','$locationProvider', function ($httpProvider, msalProvider,$locationProvider) {
        msalProvider.init(
            // msal configuration
        );

        $locationProvider.html5Mode(false).hashPrefix('');
    }]);
}
else {
    angular.module('todoApp', ['ui.router', 'MsalAngular'])
        .config(['$stateProvider', '$httpProvider', 'msalAuthenticationServiceProvider', '$locationProvider', function ($stateProvider, $httpProvider, msalProvider, $locationProvider) {
            $stateProvider.state("Home", {
                url: '/Home',
                controller: "homeCtrl",
                templateUrl: "/App/Views/Home.html",
            }).state("TodoList", {
                url: '/TodoList',
                controller: "todoListCtrl",
                templateUrl: "/App/Views/TodoList.html",
                requireLogin: true
            })

            $locationProvider.html5Mode(false).hashPrefix('');

            msalProvider.init(
                // msal configuration
            );
        }]);
}
```

В угловом приложении: App. Module. TS

```javascript
// Imports...
@NgModule({
  declarations: [
    AppComponent,
    MsalComponent,
    MainMenuComponent,
    AccountMenuComponent,
    OsNavComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    HttpClientModule,
    ServiceWorkerModule.register('ngsw-worker.js', { enabled: environment.production }),
    MsalModule.forRoot(environment.MsalConfig),
    SuiModule,
    PagesModule
  ],
  providers: [
    HttpServiceHelper,
    {provide: HTTP_INTERCEPTORS, useClass: MsalInterceptor, multi: true},
    AuthService
  ],
  entryComponents: [
    AppComponent,
    MsalComponent
  ]
})
export class AppModule {
  constructor() {
    console.log('APP Module Constructor!');
  }

  ngDoBootstrap(ref: ApplicationRef) {
    if (window !== window.parent && !window.opener)
    {
      console.log("Bootstrap: MSAL");
      ref.bootstrap(MsalComponent);
    }
    else
    {
    //this.router.resetConfig(RouterModule);
      console.log("Bootstrap: App");
      ref.bootstrap(AppComponent);
    }
  }
}
```

Мсалкомпонент:

```javascript
import { Component} from '@angular/core';
import { MsalService } from '@azure/msal-angular';

// This component is used only to avoid Angular reload
// when doing acquireTokenSilent()

@Component({
  selector: 'app-root',
  template: '',
})
export class MsalComponent {
  constructor(private Msal: MsalService) {
  }
}
```

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше о [создании одностраничного приложения (SPA)](scenario-spa-overview.md) с помощью MSAL.js.