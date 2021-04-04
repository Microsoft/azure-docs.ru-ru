---
title: Руководство по включению проверки подлинности в одностраничном приложении
titleSuffix: Azure AD B2C
description: Из этого учебника вы узнаете, как с помощью Azure Active Directory B2C обеспечить вход пользователя в систему для одностраничного приложения на базе JavaScript (SPA).
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 04/04/2020
ms.custom: mvc, seo-javascript-september2019, devx-track-js
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 6a9f3b864bd8aba2140c7d32d4b5474ff7b95f88
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96171234"
---
# <a name="tutorial-enable-authentication-in-a-single-page-application-with-azure-ad-b2c"></a>Руководство по Включение аутентификации в одностраничном приложении с помощью Azure AD B2C

В этом учебнике показано, как использовать Azure Active Directory B2C (Azure AD B2C) для регистрации и выполнения входа пользователей в одностраничном приложении (SPA) с помощью одного из потоков:
* [Поток кода авторизации OAuth 2.0](./authorization-code-flow.md) (с использованием [MSAL.js 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)).
* [Поток неявного предоставления разрешения OAuth 2.0](./implicit-flow-single-page-application.md) (с использованием [MSAL.js 1.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core)).

В этом руководстве, который представляет собой первую часть серии, состоящей из двух частей, рассматриваются следующие темы:

> [!div class="checklist"]
> * Добавление URL-адреса ответа в приложение, зарегистрированное в клиенте Azure AD B2C.
> * Скачивание примера кода из GitHub.
> * Изменение кода примера приложения для работы с вашим клиентом.
> * Регистрация с помощью потока пользователя для регистрации и выполнения входа.

В [следующем руководстве](tutorial-single-page-app-webapi.md) этой серии описано, как включить веб-API с помощью примера кода.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для продолжения выполнения действий, описанных в этом учебнике, необходимы следующие ресурсы Azure AD B2C:

* [клиент Azure AD B2C](tutorial-create-tenant.md);
* [приложение, зарегистрированное](tutorial-register-spa.md) в клиенте;
* [созданные маршруты пользователей](tutorial-create-user-flows.md) в клиенте.

Кроме того, вам понадобятся следующие компоненты в локальной среде разработки:

* [Visual Studio Code](https://code.visualstudio.com/) или любой другой редактор кода.
* [Node.js](https://nodejs.org/en/download/)

## <a name="update-the-application"></a>Обновление приложения

При прохождении [второго учебника](./tutorial-register-spa.md) для выполнения предварительных условий вы зарегистрировали одностраничное приложение в Azure AD B2C. Для обеспечения взаимодействия с примером из этого руководства необходимо добавить URL ответа (также называется URI перенаправления) в регистрацию приложения.

Чтобы обновить приложение в клиенте Azure AD B2C, можно использовать новый унифицированный интерфейс **Регистрация приложений** или устаревший интерфейс **Приложения (прежняя версия)** . [См. дополнительные сведения о новом интерфейсе](./app-registrations-training-guide.md).

#### <a name="app-registrations-auth-code-flow"></a>[Регистрация приложений (поток кода авторизации)](#tab/app-reg-auth/)

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. В меню слева выберите **Azure AD B2C**. Либо щелкните **Все службы**, а затем найдите и выберите **Azure AD B2C**
1. Выберите **Регистрация приложений**, откройте вкладку **Собственные приложения**, а затем выберите приложение *spaapp1*.
1. В разделе **Одностраничное приложение** щелкните ссылку **Добавить URI**, а затем введите `http://localhost:6420`.
1. Нажмите кнопку **Сохранить**.
1. Щелкните **Обзор**.
1. Запишите значение параметра **Идентификатор приложения (клиент)** для использования на более позднем этапе при обновлении кода в одностраничном веб-приложении.

#### <a name="app-registrations-implicit-flow"></a>[Регистрация приложений (неявный поток)](#tab/app-reg-implicit/)

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. В меню слева выберите **Azure AD B2C**. Либо щелкните **Все службы**, а затем найдите и выберите **Azure AD B2C**
1. Выберите **Регистрация приложений**, откройте вкладку **Собственные приложения**, а затем выберите приложение *spaapp1*.
1. В разделе **Одностраничное приложение** щелкните ссылку **Добавить URI**, а затем введите `http://localhost:6420`.
1. В разделе **Неявное предоставление** установите флажки для пунктов **Маркеры доступа** и **Маркеры идентификации** (если они не установлены) и нажмите кнопку **Сохранить**.
1. Щелкните **Обзор**.
1. Запишите значение параметра **Идентификатор приложения (клиент)** для использования на более позднем этапе при обновлении кода в одностраничном веб-приложении.

#### <a name="applications-legacy"></a>[Приложения (прежняя версия)](#tab/applications-legacy/)

1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. Щелкните **Applications (Legacy)** (Приложения (прежняя версия)), а затем выберите приложение *spaapp1*.
1. В разделе **URL-адрес ответа** добавьте `http://localhost:6420`.
1. Щелкните **Сохранить**.
1. На странице свойств запишите **идентификатор приложения**. Идентификатор приложения будет использован на более позднем этапе, когда вы обновите код в одностраничном веб-приложении.

* * *

## <a name="get-the-sample-code"></a>Получение кода примера

С помощью этого руководства вы настроите пример кода, который можно скачать из репозитория GitHub, для работы с клиентом B2C. В этом примере показано, как использовать Azure AD B2C в одностраничном приложении для регистрации и выполнения входа пользователя, а также вызова защищенного веб-API (вы сможете включить веб-API, выполнив инструкции из следующего руководства в этой серии).

* Пример потока кода авторизации MSAL.js 2.x:

    [Скачайте ZIP-файл](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa/archive/main.zip) или клонируйте пример с GitHub:

    ```
    git clone https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa.git
    ```
* Пример неявного потока MSAL.js 1:

    [Скачайте ZIP-файл](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp/archive/master.zip) или клонируйте пример с GitHub:

    ```
    git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git
    ```

## <a name="update-the-sample"></a>Обновление примера

Теперь, когда вы получили пример, обновите код, указав имя своего клиента Azure AD B2C и идентификатор приложения, который был записан на предыдущем шаге.

#### <a name="auth-code-flow-sample"></a>[Пример потока кода авторизации](#tab/config-auth/)

1. Откройте файл *authConfig.js* в папке *App*.
1. В объекте `msalConfig` найдите назначение для `clientId` и замените его **идентификатором приложения (клиента)** , который вы записали ранее.
1. Откройте файл `policies.js`.
1. Найдите записи в разделе `names` и замените их назначение именем пользовательских потоков, созданных ранее, например `B2C_1_signupsignin1`.
1. Найдите записи в разделе `authorities` и замените их соответствующими именами пользовательских потоков, созданных ранее, например `https://<your-tenant-name>.b2clogin.com/<your-tenant-name>.onmicrosoft.com/<your-sign-in-sign-up-policy>`.
1. Найдите назначение для `authorityDomain` и замените его на `<your-tenant-name>.b2clogin.com`.
1. Откройте файл `apiConfig.js`.
1. Найдите назначение для `b2cScopes` и замените URL-адрес соответствующим URL-адресом области, который вы создали для веб-API, например `b2cScopes: ["https://<your-tenant-name>.onmicrosoft.com/helloapi/demo.read"]`.
1. Найдите назначение для `webApi` и замените текущий URL-адрес соответствующим URL-адресом, по которому вы развернули свой API на шаге 4, например `webApi: http://localhost:5000/hello`.

#### <a name="implicit-flow-sample"></a>[Пример неявного потока](#tab/config-implicit/)

1. Откройте файл *authConfig.js* в папке *JavaScriptSPA*.
1. В объекте `msalConfig` найдите назначение для `clientId` и замените его **идентификатором приложения (клиента)** , который вы записали ранее.
1. Откройте файл `policies.js`.
1. Найдите записи в разделе `names` и замените их назначение именем пользовательских потоков, созданных ранее, например `B2C_1_signupsignin1`.
1. Найдите записи в разделе `authorities` и замените их соответствующими именами пользовательских потоков, созданных ранее, например `https://<your-tenant-name>.b2clogin.com/<your-tenant-name>.onmicrosoft.com/<your-sign-in-sign-up-policy>`.
1. Откройте файл `apiConfig.js`.
1. Найдите назначение для `b2cScopes` и замените URL-адрес соответствующим URL-адресом области, который вы создали для веб-API, например `b2cScopes: ["https://<your-tenant-name>.onmicrosoft.com/helloapi/demo.read"]`.
1. Найдите назначение для `webApi` и замените текущий URL-адрес соответствующим URL-адресом, по которому вы развернули свой API на шаге 4, например `webApi: http://localhost:5000/hello`.

* * *

Итоговый код должен выглядеть следующим образом:

#### <a name="auth-code-flow-sample"></a>[Пример потока кода авторизации](#tab/review-auth/)

*authConfig.js*:

```javascript
const msalConfig = {
  auth: {
    clientId: "e760cab2-b9a1-4c0d-86fb-ff7084abd902",
    authority: b2cPolicies.authorities.signUpSignIn.authority,
    knownAuthorities: [b2cPolicies.authorityDomain],
  },
  cache: {
    cacheLocation: "localStorage",
    storeAuthStateInCookie: true
  }
};

const loginRequest = {
  scopes: ["openid", "profile"],
};

const tokenRequest = {
  scopes: apiConfig.b2cScopes // i.e. ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"]
};
```

*policies.js*:

```javascript
const b2cPolicies = {
    names: {
        signUpSignIn: "b2c_1_susi",
        forgotPassword: "b2c_1_reset",
        editProfile: "b2c_1_edit_profile"
    },
    authorities: {
        signUpSignIn: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_susi",
        },
        forgotPassword: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_reset",
        },
        editProfile: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_edit_profile"
        }
    },
    authorityDomain: "fabrikamb2c.b2clogin.com"
}
```

*apiConfig.js*:

```javascript
const apiConfig = {
  b2cScopes: ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"],
  webApi: "https://fabrikamb2chello.azurewebsites.net/hello"
};
```

#### <a name="implicit-flow-sample"></a>[Пример неявного потока](#tab/review-implicit/)

*authConfig.js*:

```javascript
const msalConfig = {
  auth: {
    clientId: "e760cab2-b9a1-4c0d-86fb-ff7084abd902",
    authority: b2cPolicies.authorities.signUpSignIn.authority,
    validateAuthority: false
  },
  cache: {
    cacheLocation: "localStorage",
    storeAuthStateInCookie: true
  }
};

const loginRequest = {
  scopes: ["openid", "profile"],
};

const tokenRequest = {
  scopes: apiConfig.b2cScopes // i.e. ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"]
};
```

*policies.js*:

```javascript
const b2cPolicies = {
    names: {
        signUpSignIn: "b2c_1_susi",
        forgotPassword: "b2c_1_reset",
        editProfile: "b2c_1_edit_profile"
    },
    authorities: {
        signUpSignIn: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_susi",
        },
        forgotPassword: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_reset",
        },
        editProfile: {
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_edit_profile"
        }
    },
}
```

*apiConfig.js*:

```javascript
const apiConfig = {
  b2cScopes: ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"],
  webApi: "https://fabrikamb2chello.azurewebsites.net/hello"
};
```

* * *


## <a name="run-the-sample"></a>Запуск примера

1. Откройте окно консоли и перейдите в каталог, содержащий этот пример. 

    - Для примера потока кода авторизации MSAL.js 2.x выполните следующую команду:

        ```console
        cd ms-identity-b2c-javascript-spa
        ```
    - Для примера неявного потока MSAL.js 1 выполните следующую команду: 

        ```console
        cd active-directory-b2c-javascript-msal-singlepageapp
        ```

1. Выполните следующие команды:

    ```console
    npm install && npm update
    npm start
    ```

    В окне консоли отображается номер порта локально работающего сервера Node.js:

    ```console
    Listening on port 6420...
    ```
1. Перейдите по адресу `http://localhost:6420`, чтобы просмотреть веб-приложение, выполняемое на локальном компьютере.

    :::image type="content" source="media/tutorial-single-page-app/web-app-spa-01-not-logged-in.png" alt-text="Веб-браузер с отображением выполняющегося локально одностраничного приложения":::

### <a name="sign-up-using-an-email-address"></a>Регистрация с помощью адреса электронной почты

Этот пример приложения поддерживает регистрацию, вход и сброс пароля. При работе с этим руководством вы зарегистрируетесь в приложении с помощью адреса электронной почты.

1. Щелкните **Войти**, чтобы инициировать поток пользователя *B2C_1_signupsignin1*, указанный на предыдущем шаге.
1. Отобразится страница входа в Azure AD B2C со ссылкой для регистрации. Щелкните ссылку **Зарегистрироваться сейчас**, так как у вас пока нет учетной записи.
1. В рамках рабочего процесса регистрации отобразится страница для сбора данных и проверки личности пользователя с использованием адреса электронной почты. Рабочий процесс регистрации также собирает пароль пользователя и запрашиваемые атрибуты, определенные в потоке пользователя.

    Используйте действительный адрес электронной почты и подтвердите его с помощью кода проверки. Задайте пароль. Введите значения запрашиваемых атрибутов.

    :::image type="content" source="media/tutorial-single-page-app/user-flow-sign-up-workflow-01.png" alt-text="Страница регистрации, отображаемая потоком пользователя Azure AD B2C":::

1. Щелкните **Создать**, чтобы создать локальную учетную запись в каталоге Azure AD B2C.

При нажатии кнопки **Создать** приложение покажет имя выполнившего вход пользователя.

:::image type="content" source="media/tutorial-single-page-app/web-app-spa-02-logged-in.png" alt-text="Веб-браузер с отображением одностраничного приложения с выполнившим вход пользователем":::

Если вы хотите протестировать возможность входа, щелкните **Выйти** и **Войти** и войдите с адресом электронной почты и паролем, которые вы указали при регистрации.

### <a name="what-about-calling-the-api"></a>Как насчет вызова API?

Если вы нажмете кнопку **Вызов API** после выполнения входа, отобразится страница потока пользователя для регистрации и выполнения входа, а не результаты вызова API. Это стандартное поведение, так как вы еще не настроили отвечающий за API компонент приложения для обмена данными с приложением веб-API, зарегистрированным в *вашем* арендаторе Azure AD B2C.

На этом этапе приложение по-прежнему пытается связаться с API, зарегистрированным в демонстрационном клиенте (fabrikamb2c.onmicrosoft.com), но так как вы не выполнили аутентификацию в этом клиенте, отображается страница регистрации и входа.

Перейдите к следующему руководству в серии, чтобы включить защищенный API (см. раздел [Дальнейшие действия](#next-steps)).

## <a name="next-steps"></a>Дальнейшие действия

С помощью инструкций из этого руководства вы настроили одностраничное приложение для работы с потоком пользователя в клиенте Azure AD B2C, обеспечив таким образом возможность регистрации и входа. Вы сделали следующее:

> [!div class="checklist"]
> * добавили URL-адрес ответа в приложение, зарегистрированное в клиенте Azure AD B2C;
> * скачали пример кода из GitHub;
> * изменили код примера приложения для работы с вашим арендатором;
> * выполнили регистрацию с помощью своего потока пользователя для регистрации и выполнения входа.

Теперь перейдите к следующему руководству из этой серии, чтобы предоставить доступ к защищенному веб-API из SPA:

> [!div class="nextstepaction"]
> [Руководство. Предоставление доступа к веб-API из одностраничного приложения](tutorial-single-page-app-webapi.md)