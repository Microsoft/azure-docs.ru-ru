---
title: Краткое руководство. Вход пользователей в одностраничных приложениях Angular в Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как приложение Angular может вызывать API, которому требуются маркеры доступа, выданные платформой удостоверений Майкрософт.
services: active-directory
author: jasonnutter
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:JavaScript, devx-track-js
ms.topic: quickstart
ms.workload: identity
ms.date: 03/18/2020
ms.author: janutter
ms.openlocfilehash: bab92a6d7e30f5aefdd28d06b34a006d065cee3c
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105966849"
---
# <a name="quickstart-sign-in-users-and-get-an-access-token-in-an-angular-single-page-application"></a>Краткое руководство. Вход пользователей и получение маркера доступа в одностраничное приложение Angular

Из этого краткого руководства вы узнаете, как скачать и выполнить пример кода, в котором показана реализация входа пользователей в одностраничное приложение (SPA) Angular и вызов Microsoft Graph. В этом примере кода демонстрируется получение маркера доступа для вызова API Microsoft Graph или любого веб-API.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. [Создайте бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Node.js](https://nodejs.org/en/download/).
* [Visual Studio Code](https://code.visualstudio.com/download) для редактирования файлов проекта или [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) для запуска проекта.

> [!div renderon="docs"]
> ## <a name="register-and-download-the-quickstart-app"></a>Регистрация и скачивание приложения быстрого запуска
> Чтобы запустить приложение быстрого запуска, используйте один из следующих вариантов.
>
> ### <a name="option-1-express-register-and-automatically-configure-the-app-and-then-download-the-code-sample"></a>Вариант 1 (экспресс-способ). Регистрация и автоматическая настройка приложения, а затем скачивание примера кода.
>
> 1. Откройте страницу <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType/JavascriptSpaQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Перейдите на панель быстрого запуска и просмотрите сведения по началу работы с Angular. Следуйте инструкциям, чтобы быстро скачать и автоматически настроить новое приложение.
>
> ### <a name="option-2-manual-register-and-manually-configure-the-application-and-code-sample"></a>Вариант 2 (вручную). Регистрация и настройка приложения и примера кода вручную
>
> #### <a name="step-1-register-the-application"></a>Шаг 1. Регистрация приложения
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Следуйте инструкциям, чтобы [зарегистрировать одностраничное приложение](./scenario-spa-app-registration.md) на портале Azure.
> 1. Добавьте новую платформу в область **Проверка подлинности** регистрации приложения и зарегистрируйте URI перенаправления: `http://localhost:4200/`.
> 1. В этом кратком руководстве используется [поток неявного предоставления разрешений](v2-oauth2-implicit-grant-flow.md). В разделе **Неявное предоставление разрешения и гибридные потоки** выберите **Токены идентификации** и **Маркеры доступа**. Маркеры идентификатора и маркеры доступа необходимы, так как это приложение поддерживает вход пользователей и вызов API.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-the-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Чтобы пример кода, приведенный в этом кратком руководстве, работал, добавьте URI перенаправления **http://localhost:4200/** и включите **неявное предоставление разрешений**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-javascript/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-code-sample"></a>Шаг 2. Скачивание примера кода
>[!div renderon="docs"]
>Чтобы запустить проект с веб-сервером с помощью Node.js, [клонируйте пример репозитория](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) или [загрузите основные файлы проекта](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular/archive/master.zip). Откройте файлы, используя редактор, например Visual Studio Code.

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular/archive/master.zip)

> [!div renderon="docs"]
>#### <a name="step-3-configure-the-javascript-app"></a>Шаг 3. Настройка приложения JavaScript
>
> В папке *src/app* измените *app.module.ts* и задайте значения `clientId` и `authority` в разделе `MsalModule.forRoot`.
>
>```javascript
>MsalModule.forRoot({
>    auth: {
>        clientId: 'Enter_the_Application_Id_here', // This is your client ID
>        authority: 'https://login.microsoftonline.com/Enter_the_Tenant_Info_Here', // This is your tenant info
>        redirectUri: 'Enter_the_Redirect_Uri_Here' // This is your redirect URI
>    },
>    cache: {
>        cacheLocation: 'localStorage',
>        storeAuthStateInCookie: isIE, // set to true for IE 11
>    },
>},
> //... )
>```

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > Enter_the_Supported_Account_Info_Here


> [!div renderon="docs"]
>
> Замените следующие значения.
>
>|Имя значения|Описание|
>|---------|---------|
>|Enter_the_Application_Id_Here|Это значение **идентификатора приложения (клиента)** со страницы **обзора** регистрации приложения. |
>|Enter_the_Cloud_Instance_Id_Here|Это экземпляр облака Azure. Для основного или глобального облака Azure введите **https://login.microsoftonline.com** . Сведения для национальных облаков (например, Китая) см. в [этой статье](./authentication-national-cloud.md).|
>|Enter_the_Tenant_Info_Here| Задайте один из следующих параметров: Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение идентификатором каталога (клиента) или именем клиента (например, **contoso.microsoft.com**). Если ваше приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на **organizations**. Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на **common**. Чтобы ограничить поддержку только *личными учетными записями Майкрософт*, замените это значение на **consumers**. |
>|Enter_the_Redirect_Uri_Here|Замените на **http://localhost:4200** .|
>|cacheLocation  | (Необязательно) Этот параметр настраивает хранилище браузера для состояния проверки подлинности. По умолчанию используется **sessionStorage**.   |
>|storeAuthStateInCookieко  | (Необязательно) Проверка подлинности, которая хранит состояние запроса проверки подлинности, необходимое для проверки потоков аутентификации, в файлах cookie браузера. Этот файл cookie установлен для браузеров Internet Explorer и Microsoft Edge, чтобы вы могли использовать оба этих браузера. Дополнительные сведения об известных проблемах см. в [этой записи блога](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues->on-IE-and-Edge-Browser#issues). |
>
> Чтобы найти значения параметров **Идентификатор приложения (клиента)** , **Идентификатор каталога (клиента)** и **Поддерживаемые типы учетных записей**, на портале Azure перейдите на страницу **Обзор**.

> Подробные сведения о доступных настраиваемых параметрах см. в статье [Initialize client applications using MSAL.js](msal-js-initializing-client-applications.md) (Инициализация клиентских приложений с помощью MSAL.js).

> Исходный код библиотеки MSAL.js можно найти в репозитории [AzureAD/microsoft-authentication-library-for-js](https://github.com/AzureAD/microsoft-authentication-library-for-js) на сайте GitHub.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
> Мы настроили проект, указав значения свойств приложения.

> [!div renderon="docs"]
>
> Прокрутите тот же файл вниз и обновите `graphMeEndpoint`. 
> - Замените строку `Enter_the_Graph_Endpoint_Herev1.0/me` строкой `https://graph.microsoft.com/v1.0/me`
> - `Enter_the_Graph_Endpoint_Herev1.0/me` обозначает конечную точку, к которой будут направляться вызовы API. Для основной (глобальной) службы API Microsoft Graph введите значение `https://graph.microsoft.com/` (включая замыкающую косую черту). Дополнительные сведения см. в этой [документации](https://docs.microsoft.com/graph/deployments).
>
>
> ```javascript
>      protectedResourceMap: [
>        ['Enter_the_Graph_Endpoint_Herev1.0/me', ['user.read']]
>      ],
> ```
>
>

>[!div renderon="docs"]
>#### <a name="step-4-run-the-project"></a>Шаг 4. Запуск проекта

Если вы используете Node.js:

1. Запустите сервер, выполнив в каталоге проекта следующие команды:

   ```console
   npm install
   npm start
   ```

1. Перейдите по ссылке **http://localhost:4200/** .
1. Выберите **Имя для входа**.
1. Выберите **Профиль**, чтобы вызвать Microsoft Graph.

Когда приложение загрузится в браузере, нажмите кнопку **Вход**. После первого входа предоставьте приложению разрешение на использование данных вашего профиля для входа. После успешного выполнения входа выберите **Профиль**. Сведения о вашем профиле пользователя будут отображаться на странице.

## <a name="how-the-sample-works"></a>Как работает этот пример

![Схема, на которой показано, как работает пример приложения, созданный в этом кратком руководстве](./media/quickstart-v2-angular/diagram-auth-flow-spa-angular.svg)


## <a name="next-steps"></a>Дальнейшие действия

Далее в учебнике по Angular вы узнаете, как выполнить для пользователя вход в систему и получить маркеры:

> [!div class="nextstepaction"]
> [Учебник по Angular](./tutorial-v2-angular.md)