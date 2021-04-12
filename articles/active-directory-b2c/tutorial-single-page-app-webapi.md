---
title: Руководство по Защита веб-API Node.js с помощью Azure AD B2C и предоставление доступа к одностраничному приложению (SPA)
titleSuffix: Azure AD B2C
description: В этом учебнике показано, как с помощью Active Directory B2C обеспечить защиту веб-API Node.js и выполнить его вызов из одностраничного приложения.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 04/04/2020
ms.custom: mvc, devx-track-js
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: 737810a7d07d0d97b2e42acffa17fdd32986c48b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93421096"
---
# <a name="tutorial-protect-and-grant-access-to-a-nodejs-web-api-from-a-single-page-application-with-azure-ad-b2c"></a>Руководство по Защита веб-API Node.js и предоставление доступа к нему из одностраничного приложения с помощью Azure AD B2C

В этом руководстве показано, как вызывать защищенный с помощью Azure Active Directory B2C (Azure AD B2C) веб-API Node.js из одностраничного приложения.

В этом руководстве, который представляет собой вторую часть серии, состоящей из двух частей, рассматриваются следующие темы:

> [!div class="checklist"]
> * Создание регистрации приложения веб-API в клиенте Azure AD B2C.
> * Настройка областей для веб-API.
> * Предоставление разрешения на использование веб-API.
> * Изменение примера кода веб-API для работы с вашим клиентом.

Выполнив инструкции из [первого руководства](tutorial-single-page-app.md) этой серии, вы скачали пример кода и изменили его для выполнения входа пользователей с помощью потока пользователя в своем арендаторе Azure AD B2C.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* Выполните действия и необходимые условия в статье [Руководство. Включение аутентификации в одностраничном приложении с помощью Azure AD B2C](tutorial-single-page-app.md)
* [Visual Studio Code](https://code.visualstudio.com/) или любой другой редактор кода.
* [Node.js](https://nodejs.org/en/download/)

## <a name="add-a-web-api-application"></a>Добавление приложения веб-API

[!INCLUDE [active-directory-b2c-appreg-webapi](../../includes/active-directory-b2c-appreg-webapi.md)]

## <a name="configure-scopes"></a>Настройка областей

Области предоставляют способ контроля доступа к защищенным ресурсам. Области используются веб-API для реализации управления доступом на уровне области. Например, некоторые пользователи могут иметь доступ на чтение и запись, тогда как другие пользователи могут иметь разрешения только на чтение. В этом учебнике вы определяете разрешения на чтение и запись для веб-API.

[!INCLUDE [active-directory-b2c-scopes](../../includes/active-directory-b2c-scopes.md)]

Запишите значение в разделе **Области**, которое будет использоваться в области `demo.read` на следующих шагах при настройке одностраничного приложения. Значение полной области аналогично `https://contosob2c.onmicrosoft.com/api/demo.read`.

## <a name="grant-permissions"></a>Предоставить разрешения

Чтобы вызвать защищенный веб-API из другого приложения, необходимо предоставить приложению разрешения на доступ к веб-API.

Выполняя предварительные требования, вы создали одностраничное приложение с именем *spaapp1*. При работе с этим учебником вы настроите это приложение для вызова веб-API *spaapp1*, созданного в предыдущем разделе.

[!INCLUDE [active-directory-b2c-permissions-api](../../includes/active-directory-b2c-permissions-api.md)]

Вашему одностраничному приложению теперь предоставлены разрешения на доступ к защищенному веб-API для указанных областей. Пользователь выполняет проверку подлинности в Azure AD B2C для использования одностраничного приложения. Одностраничное приложение получает маркер доступа из Azure AD B2C для доступа к защищенному веб-API.

## <a name="configure-the-sample"></a>Настройка примера

Теперь, когда веб-API зарегистрирован и вы определили области, настройте код веб-API для работы с арендатором Azure AD B2C. В этом руководстве показано, как настроить пример веб-API Node.js, который можно скачать из репозитория GitHub.

[Скачайте \*архив ZIP](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi/archive/master.zip) или клонируйте пример проекта веб-API с GitHub. Вы также можете перейти непосредственно к проекту [Azure-Samples/active-directory-b2c-javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) на сайте GitHub.

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi.git
```

### <a name="configure-the-web-api"></a>Настройка веб-API

1. Откройте файл *config.json* в редакторе кода.
1. Измените значения переменных, указав значения из созданной ранее регистрации приложения. Также обновите `policyName`, указав поток пользователя, созданный в рамках предварительных требований (например, *B2C_1_signupsignin1*).
    
    ```json
    "credentials": {
        "tenantName": "<your-tenant-name>",
        "clientID": "<your-webapi-application-ID>"
    },
    "policies": {
        "policyName": "B2C_1_signupsignin1"
    },
    "resource": {
        "scope": ["demo.read"] 
    },
    ```

#### <a name="enable-cors"></a>Включение CORS

Чтобы разрешить одностраничному приложению вызывать веб-API Node.js, необходимо включить [CORS](https://expressjs.com/en/resources/middleware/cors.html) в веб-API. В рабочем приложении следует проявлять осторожность в отношении того, какой домен отправляет запрос, но при работе с этим руководством вы можете разрешить запросы от любого домена.

Чтобы включить CORS, используйте следующее ПО промежуточного слоя. В примере кода веб-API Node.js в этом руководстве оно уже добавлено в файл *index.js*.

```javascript
app.use((req, res, next) => {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Authorization, Origin, X-Requested-With, Content-Type, Accept");
    next();
});
```

### <a name="configure-the-single-page-application"></a>Настройка одностраничного приложения

Одностраничное приложение (SPA) из [предыдущего руководства](tutorial-single-page-app.md) этой серии использует Azure AD B2C для регистрации и выполнения входа пользователя, а также по умолчанию вызывает веб-API Node.js, защищенный примером арендатора *fabrikamb2c*.

В этом разделе показано, как обновить одностраничное приложение для вызова веб-API Node.js, защищенного *вашим* арендатором Azure AD B2C и запущенного на локальном компьютере.

Чтобы изменить параметры в SPA, выполните приведенные ниже действия.

1. В проекте [active-directory-b2c-javascript-msal-singlepageapp][github-js-spa], который вы скачали или клонировали при выполнении инструкций из предыдущего руководства, откройте файл *apiConfig.js* в папке *JavaScriptSPA*.
1. Настройте пример с помощью URI для области *demo.read*, созданной ранее, а также URL-адрес веб-API.
    1. В определении `apiConfig` замените значение `b2cScopes` полным URI для области *demo.read* (значение **Область**, записанное ранее).
    1. Измените домен в значении `webApi` на URI перенаправления, добавленный при регистрации приложения веб-API на предыдущем шаге.

    Так как API доступен на конечной точке `/hello`, не удаляйте */hello* из URI.

    Определение `apiConfig` должно выглядеть как следующий блок кода, но с именем арендатора B2C вместо `<your-tenant-name>`:

    ```javascript
    // The current application coordinates were pre-registered in a B2C tenant.
    const apiConfig = {
      b2cScopes: ["https://<your-tenant-name>.onmicrosoft.com/api/demo.read"],
      webApi: "http://localhost:5000/hello" // '/hello' should remain in the URI
    };
    ```

## <a name="run-the-spa-and-web-api"></a>Запуск SPA и веб-API

Теперь вы готовы протестировать ограниченный областью доступ одностраничного приложения к API. Запустите веб-API Node.js и пример одностраничного приложения JavaScript на локальном компьютере. Затем войдите в одностраничное приложение и нажмите кнопку **Вызов API**, чтобы инициировать запрос к защищенному API.

Хотя в нашем примере оба приложения запущены локально, вы настроили их для использования Azure AD B2C для безопасных операций регистрации и входа, а также для предоставления доступа к защищенному веб-API.

### <a name="run-the-nodejs-web-api"></a>Запуск веб-API для Node.js

1. Откройте окно консоли и перейдите в каталог, содержащий пример веб-API Node.js. Пример:

    ```console
    cd active-directory-b2c-javascript-nodejs-webapi
    ```

1. Выполните следующие команды:

    ```console
    npm install && npm update
    node index.js
    ```

    В окне консоли отобразится номер порта, на котором размещено приложение.

    ```console
    Listening on port 5000...
    ```

### <a name="run-the-single-page-app"></a>Запуск одностраничного приложения

1. Откройте еще одно окно консоли и перейдите в каталог, содержащий пример одностраничного приложения JavaScript. Пример:

    ```console
    cd ms-identity-b2c-javascript-spa
    ```

1. Выполните следующие команды:

    ```console
    npm install && npm update
    npm start
    ```

    В окне консоли отобразится номер порта, связанный с размещаемым приложением.

    ```console
    Listening on port 6420...
    ```

1. Перейдите к `http://localhost:6420` в браузере для просмотра приложения.

    ![Пример открытого в браузере одностраничного приложения](./media/tutorial-single-page-app-webapi/tutorial-01-sample-app-browser.png)

1. Выполните вход с помощью адреса электронной почты и пароля, которые вы использовали в [предыдущем учебнике](tutorial-single-page-app.md). После успешного входа отобразится сообщение `User 'Your Username' logged-in`.
1. Нажмите кнопку **вызова API**. Одностраничное приложение получает предоставление авторизации из Azure AD B2C, а затем — доступ к защищенному веб-API для отображения имени выполнившего вход пользователя.

    ![Одностраничное приложение в браузере с отображением результата JSON с именем пользователя, возвращаемого API](./media/tutorial-single-page-app-webapi/tutorial-02-call-api.png)

## <a name="next-steps"></a>Дальнейшие действия

Изучив это руководство, вы:

> [!div class="checklist"]
> * создали регистрацию приложения веб-API в арендаторе Azure AD B2C;
> * настроили области для веб-API;
> * предоставили разрешения на использование веб-API;
> * изменили пример кода веб-API для работы с вашим арендатором.

Теперь, когда вы узнали, как SPA выполняет запрос ресурса из защищенного веб-API, вы можете получить более глубокое представление о том, как эти типы приложений взаимодействуют друг с другом и с Azure AD B2C.

> [!div class="nextstepaction"]
> [Application types that can be used in Active Directory B2C](application-types.md) (Типы приложений, используемые в Azure Active Directory B2C)

<!-- Links - EXTERNAL -->
[github-js-spa]: https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp
