---
title: Краткое руководство. Вход пользователей в одностраничных приложениях (SPA) JavaScript React с помощью кода проверки подлинности и вызова Microsoft Graph | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как в одностраничном приложении (SPA) JavaScript React реализовать вход пользователей с личными, рабочими и учебными учетными записями с помощью потока кода авторизации и вызова Microsoft Graph.
services: active-directory
author: j-mantu
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 01/14/2021
ms.author: jamesmantu
ms.custom: aaddev, scenarios:getting-started, languages:JavaScript, devx-track-js
ms.openlocfilehash: 0bf08c45e82dc6f36d4e179e95e1b58e655b14db
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103224373"
---
# <a name="quickstart-sign-in-and-get-an-access-token-in-a-react-spa-using-the-auth-code-flow"></a>Краткое руководство. Вход и получение маркера доступа в React SPA с помощью потока кода авторизации

При работе с этим кратким руководством вы скачаете и выполните пример кода. Такой пример кода демонстрирует, как в одностраничном приложении (SPA) JavaScript React реализовать вход пользователей и вызов Microsoft Graph с помощью потока кода авторизации. В этом примере кода демонстрируется получение маркера доступа для вызова API Microsoft Graph или любого веб-API. 

Иллюстрацию см. в разделе [Как работает этот пример](#how-the-sample-works).

В этом кратком руководстве используется MSAL React с потоком кода авторизации. Есть аналогичное руководство для MSAL.js с неявным потоком: [Краткое руководство. Вход пользователей в одностраничных приложениях JavaScript](./quickstart-v2-javascript.md).

> [!IMPORTANT]
> MSAL React [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. [Создать подписку Azure бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) или любой другой редактор кода.

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>Регистрация и скачивание приложения, используемого в этом кратком руководстве
> Чтобы запустить приложение, используемое в этом кратком руководстве, выберите любой из следующих вариантов.
>
>
> ### <a name="option-1-express-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Вариант 1 (экспресс-способ). Регистрация и автоматическая настройка приложения, а затем скачивание примера кода
>
> 1. Откройте страницу <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType/JavascriptSpaQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
> 1. Введите имя приложения.
> 1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт).
> 1. Выберите **Зарегистрировать**.
> 1. Перейдите на панель быстрого запуска и следуйте инструкциям по скачиванию и автоматической настройке нового приложения.
>
> ### <a name="option-2-manual-register-and-manually-configure-your-application-and-code-sample"></a>Вариант 2 (вручную). Регистрация и настройка приложения и примера кода вручную
>
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Когда откроется страница **Register an application** (Регистрация приложения), введите имя приложения.
> 1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт).
> 1. Выберите **Зарегистрировать**. На странице приложения **Обзор** запишите **идентификатор приложения (клиента)** для использования в будущем.
> 1. В разделе **Управление** выберите **Проверка подлинности**.
> 1. В разделе **Конфигурации платформ** щелкните **Добавить платформу**. На открывшейся панели выберите **Одностраничное приложение**.
> 1. В поле **URI перенаправления** укажите значение `http://localhost:3000/`. Это порт по умолчанию, который Node.js будет прослушивать на вашем локальном компьютере. Когда пользователь пройдет проверку подлинности, мы вернем ответ на этот URI. 
> 1. Выберите **Настроить**, чтобы применить изменения.
> 1. В разделе **Конфигурации платформ** разверните **Одностраничное приложение**.
> 1. Убедитесь, что в разделе **Типы грантов** ![Значок "Уже настроено"](media/quickstart-v2-javascript/green-check.png) ваш URI перенаправления подходит для потока кода авторизации с использованием PKCE.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Чтобы пример кода, приведенный в этом кратком руководстве, работал, добавьте **URI перенаправления** `http://localhost:3000/`.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-javascript/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-project"></a>Шаг 2. Скачивание проекта

> [!div renderon="docs"]
> [Скачайте основные файлы проекта](https://github.com/Azure-Samples/ms-identity-javascript-react-spa/archive/main.zip), чтобы запустить проект с помощью веб-сервера, используя Node.js.

> [!div renderon="portal" class="sxs-lookup"]
> Запустите проект с помощью веб-сервера, используя Node.js.

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/ms-identity-javascript-react-spa/archive/main.zip)

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-javascript-app"></a>Шаг 3. Настройка приложения JavaScript
>
> В папке *src* откройте файл *authConfig.js* и измените в нем значения `clientID`, `authority` и `redirectUri` для объекта `msalConfig`.
>
> ```javascript
> /**
> * Configuration object to be passed to MSAL instance on creation. 
> * For a full list of MSAL.js configuration parameters, visit:
> * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/configuration.md 
> */
> export const msalConfig = {
>    auth: {
>        clientId: "Enter_the_Application_Id_Here",
>        authority: "Enter_the_Cloud_Instance_Id_HereEnter_the_Tenant_Info_Here",
>        redirectUri: "Enter_the_Redirect_Uri_Here"
>    },
>    cache: {
>        cacheLocation: "sessionStorage", // This configures where your cache will be stored
>        storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
>    },
> ```

> [!div renderon="portal" class="sxs-lookup"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
>
> Измените значения в разделе `msalConfig`, как описано далее.
>
> - `Enter_the_Application_Id_Here` содержит **идентификатор приложения (клиента)** для зарегистрированного приложения.
>
>    Чтобы найти значение параметра **Идентификатор приложения (клиента)** , на портале Azure перейдите на страницу **Обзор** регистрации приложения.
> - `Enter_the_Cloud_Instance_Id_Here` представляет экземпляр облака Azure. Для основного или глобального облака Azure введите `https://login.microsoftonline.com/`. Сведения для **национальных** облаков (например, Китая) см. в [этой статье](authentication-national-cloud.md).
> - `Enter_the_Tenant_info_here` может иметь одно из следующих значений.
>   - Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение **идентификатором клиента** или **именем клиента**. Например, `contoso.microsoft.com`.
>
>    Чтобы найти значение параметра **Идентификатор каталога (арендатора)** , на портале Azure перейдите на страницу **Обзор** регистрации приложения.
>   - Если приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на `organizations`.
>   - Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на `common`. В примерах **этого краткого руководства** укажите `common`.
>   - Чтобы ограничить поддержку только *личными учетными записями Microsoft*, замените это значение на `consumers`.
>
>    Чтобы найти значение параметра **Поддерживаемые типы учетных записей**, на портале Azure перейдите на страницу **Обзор** регистрации приложения.
> - Параметр `Enter_the_Redirect_Uri_Here` равен `http://localhost:3000/`.
>
> Если вы используете основное (глобальное) облако Azure, значение `authority` в файле *authConfig.js* должно выглядеть примерно так:
>
> ```javascript
> authority: "https://login.microsoftonline.com/common",
> ```
>
> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
> Мы настроили проект, указав значения свойств приложения.

> [!div renderon="docs"]
>
> Прокрутите тот же файл вниз и обновите `graphMeEndpoint`. 
> - Замените строку `Enter_the_Graph_Endpoint_Herev1.0/me` строкой `https://graph.microsoft.com/v1.0/me`
> - `Enter_the_Graph_Endpoint_Herev1.0/me` обозначает конечную точку, к которой будут направляться вызовы API. Для основной (глобальной) службы API Microsoft Graph введите значение `https://graph.microsoft.com/` (включая замыкающую косую черту). Дополнительные сведения см. в этой [документации](/graph/deployments).
>
>
>
> ```javascript
>   // Add here the endpoints for MS Graph API services you would like to use.
>    export const graphConfig = {
>        graphMeEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me"
>    };
> ```
>
>
#### <a name="step-4-run-the-project"></a>Шаг 4. Запуск проекта

Запустите проект на веб-сервере с помощью Node.js:

1. Чтобы запустить сервер, выполните в каталоге проекта следующую команду.
    ```console
    npm install
    npm start
    ```
1. Перейдите по адресу `http://localhost:3000/`.

1. Нажмите кнопку **Войти**, чтобы начать процесс входа в систему, а затем вызовите API Microsoft Graph.

    После первого входа предоставьте приложению разрешение на использование данных вашего профиля для входа. После успешного входа нажмите **Request Profile Information** (Запросить сведения о профиле), чтобы получить сведения о профиле на странице.

## <a name="more-information"></a>Дополнительные сведения

### <a name="how-the-sample-works"></a>Как работает этот пример

![Схема, демонстрирующая поток кода авторизации для одностраничного приложения](media/quickstart-v2-javascript-auth-code/diagram-01-auth-code-flow.png)

### <a name="msaljs"></a>msal.js

MSAL.js — это библиотека, используемая для выполнения входа пользователей и запросов маркеров, которые нужны для доступа к API, защищенному платформой удостоверений Майкрософт.

Если у вас установлен Node.js, последнюю версию можно скачать с помощью диспетчера пакетов npm из Node.js:

```console
npm install @azure/msal-browser @azure/msal-react
```

## <a name="next-steps"></a>Дальнейшие действия

Подробное пошаговое руководство по созданию приложения потока кода для проверки подлинности с помощью vanilla JavaScript см. в следующем учебнике:

> [!div class="nextstepaction"]
> [Tutorial to sign in and call MS Graph](./tutorial-v2-javascript-auth-code.md) (Руководство по вызову API Microsoft Graph)