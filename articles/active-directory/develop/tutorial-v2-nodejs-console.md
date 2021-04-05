---
title: Руководство. Вызов Microsoft Graph в консольном приложении Node.js | Azure
titleSuffix: Microsoft identity platform
description: В этом руководстве описано, как создать консольное приложение для вызова Microsoft Graph в консольном приложении Node.js.
services: active-directory
author: derisen
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.date: 02/17/2021
ms.author: v-doeris
ms.openlocfilehash: 33d3712e25a06419e0ccc5914cdddfae7d85a371
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101645795"
---
# <a name="tutorial-call-the-microsoft-graph-api-in-a-nodejs-console-app"></a>Руководство. Вызов API Microsoft Graph в консольном приложении Node.js

В этом руководстве описано, как создать консольное приложение, которое вызывает API Microsoft Graph, используя собственное удостоверение. Создаваемое консольное приложение использует [библиотеку проверки подлинности Майкрософт (MSAL) для Node.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node).

Выполните шаги из этого руководства, чтобы:

> [!div class="checklist"]
> - регистрация приложения на портале Azure;
> - создать проект консольного приложения Node.js;
> - добавить логику аутентификации в приложение;
> - добавить сведения о регистрации приложения;
> - добавить метод для вызова веб-API;
> - Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

- [Node.js](https://nodejs.org/en/download/)
- [Visual Studio Code](https://code.visualstudio.com/download) или любой другой редактор кода.

## <a name="register-the-application"></a>Регистрация приложения

Сначала выполните действия, описанные в статье [Краткое руководство. Регистрация приложения с помощью платформы удостоверений Майкрософт](quickstart-register-app.md), чтобы зарегистрировать приложение.

Используйте следующие параметры для регистрации приложения:

- Имя: `NodeConsoleApp` (рекомендуется).
- Поддерживаемые типы учетных записей: **Accounts in this organizational directory only** (Учетные записи только в этом каталоге организации)
- Разрешения API: **API Майкрософт** > **Microsoft Graph** > **Разрешения приложения** > `User.Read.All`.
- Секрет клиента: `*********` (запишите это значение для использования на последующих этапах; оно отображается только один раз).

## <a name="create-the-project"></a>Создание проекта

Создайте папку для размещения приложения, например *NodeConsoleApp*.

1. Сначала перейдите в окне терминала в каталог проекта, а затем выполните следующие команды npm:

```console
    npm init -y
    npm install --save dotenv yargs axios @azure/msal-node
```

2. Затем создайте папку с именем *bin*. Затем в этой папке создайте файл с именем *index.js* и добавьте следующий код:

```JavaScript
#!/usr/bin/env node

// read in env settings
require('dotenv').config();

const yargs = require('yargs');

const fetch = require('./fetch');
const auth = require('./auth');

const options = yargs
    .usage('Usage: --op <operation_name>')
    .option('op', { alias: 'operation', describe: 'operation name', type: 'string', demandOption: true })
    .argv;

async function main() {
    console.log(`You have selected: ${options.op}`);

    switch (yargs.argv['op']) {
        case 'getUsers':

            try {
                // here we get an access token
                const authResponse = await auth.getToken(auth.tokenRequest);

                // call the web API with the access token
                const users = await fetch.callApi(auth.apiConfig.uri, authResponse.accessToken);

                // display result
                console.log(users);
            } catch (error) {
                console.log(error);
            }

            break;
        default:
            console.log('Select a Graph operation first');
            break;
    }
};

main();
```

Этот файл ссылается на два других модуля Node: *auth.js*, содержащий реализацию узла MSAL Node для получения маркеров доступа, и *fetch.js*, содержащий метод для выполнения HTTP-запроса к API Microsoft Graph с маркером доступа. После выполнения остальных инструкций в этом учебнике файл и структура папок проекта должны выглядеть следующим образом:

```
NodeConsoleApp/
├── bin
│   ├── auth.js
│   ├── fetch.js
│   ├── index.js
├── package.json
└── .env
```

## <a name="add-authentication-logic"></a>Добавление логики аутентификации

В папке *bin* создайте другой файл с именем *auth.js* и добавьте следующий код для получения маркера доступа, который будет представлен при вызове API Microsoft Graph.

```JavaScript
const msal = require('@azure/msal-node');

/**
 * Configuration object to be passed to MSAL instance on creation.
 * For a full list of MSAL Node configuration parameters, visit:
 * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/configuration.md
 */
const msalConfig = {
    auth: {
        clientId: process.env.CLIENT_ID,
        authority: process.env.AAD_ENDPOINT + process.env.TENANT_ID,
        clientSecret: process.env.CLIENT_SECRET,
    }
};

/**
 * With client credentials flows permissions need to be granted in the portal by a tenant administrator.
 * The scope is always in the format '<resource>/.default'. For more, visit:
 * https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-client-creds-grant-flow
 */
const tokenRequest = {
    scopes: [process.env.GRAPH_ENDPOINT + '.default'],
};

const apiConfig = {
    uri: process.env.GRAPH_ENDPOINT + 'v1.0/users',
};

/**
 * Initialize a confidential client application. For more info, visit:
 * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/initialize-confidential-client-application.md
 */
const cca = new msal.ConfidentialClientApplication(msalConfig);

/**
 * Acquires token with client credentials.
 * @param {object} tokenRequest
 */
async function getToken(tokenRequest) {
    return await cca.acquireTokenByClientCredential(tokenRequest);
}

module.exports = {
    apiConfig: apiConfig,
    tokenRequest: tokenRequest,
    getToken: getToken
};
```

В приведенном выше фрагменте кода сначала создается объект конфигурации (*msalConfig*), который передается для инициализации объекта MSAL [ConfidentialClientApplication](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/initialize-confidential-client-application.md). Затем создается метод для получения маркеров через **учетные данные клиента** и файлу *main.js* предоставляется доступ к этому модулю. Параметры конфигурации в этом модуле копируются из файла среды, который будет создан на следующем шаге.

## <a name="add-app-registration-details"></a>Добавление сведений о регистрации приложения

Создайте файл среды для хранения сведений о регистрации приложения, которые будут использоваться при получении маркеров. Для этого создайте файл с именем *.env* в корневой папке примера (*NodeConsoleApp*) и добавьте следующий код:

```
# Credentials
TENANT_ID=Enter_the_Tenant_Id_Here
CLIENT_ID=Enter_the_Application_Id_Here
CLIENT_SECRET=Enter_the_Client_Secret_Here

# Endpoints
AAD_ENDPOINT=Enter_the_Cloud_Instance_Id_Here
GRAPH_ENDPOINT=Enter_the_Graph_Endpoint_Here
```

Приведите следующие сведения, указав значения, которые вы получили на портале регистрации приложений Azure:

- `Enter_the_Tenant_Id_here` должно быть одним из следующих:
  - Если приложение поддерживает *учетные записи только в этом каталоге организации*, замените это значение **идентификатором клиента** или **именем клиента**. Например, `contoso.microsoft.com`.
  - Если приложение поддерживает *учетные записи в любом каталоге организации*, замените это значение на `organizations`.
  - Если приложение поддерживает *учетные записи в любом каталоге организации и личные учетные записи Майкрософт*, замените это значение на `common`.
  - Чтобы ограничить поддержку только *личными учетными записями Microsoft*, замените это значение на `consumers`.
- `Enter_the_Application_Id_Here`: **идентификатор приложения (клиента)** , которое вы зарегистрировали.
- `Enter_the_Cloud_Instance_Id_Here`: облачный экземпляр Azure, в котором зарегистрировано приложение.
  - Для основного (или *глобального*) облака Azure введите `https://login.microsoftonline.com`.
  - Для **национальных** облаков (например, китайского) соответствующие значения см. в статье [Национальные облака](authentication-national-cloud.md).
- `Enter_the_Graph_Endpoint_Here` экземпляр API Microsoft Graph, с которым должно взаимодействовать приложение.
  - Для **глобальной** конечной точки API Microsoft Graph замените оба экземпляра этой строки на `https://graph.microsoft.com`.
  - Дополнительные сведения о конечных точках в **национальных** облачных развертываниях см. в статье [Национальные облачные развертывания](/graph/deployments) в документации по Microsoft Graph.

## <a name="add-a-method-to-call-a-web-api"></a>Добавление метода для вызова веб-API

В папке *bin* создайте еще один файл с именем *fetch.js* и добавьте следующий код для выполнения вызовов REST к API Microsoft Graph:

```javascript
const axios = require('axios');

/**
 * Calls the endpoint with authorization bearer token.
 * @param {string} endpoint
 * @param {string} accessToken
 */
async function callApi(endpoint, accessToken) {

    const options = {
        headers: {
            Authorization: `Bearer ${accessToken}`
        }
    };

    console.log('request made to web API at: ' + new Date().toString());

    try {
        const response = await axios.default.get(endpoint, options);
        return response.data;
    } catch (error) {
        console.log(error)
        return error;
    }
};

module.exports = {
    callApi: callApi
};
```

Здесь метод `callApi` используется для выполнения HTTP-запроса `GET` к защищенному ресурсу, которому требуется маркер доступа. Затем метод возвращает содержимое вызывающему объекту. Этот метод добавляет полученный маркер в *заголовок авторизации HTTP*. Защищенный ресурс здесь — это [конечная точка пользователей](/graph/api/user-list) API Microsoft Graph, в которой отображаются пользователи в арендаторе, где зарегистрировано это приложение.

## <a name="test-the-app"></a>Тестирование приложения

Вы создали приложение и теперь готовы протестировать его функциональность.

Запустите консольное приложение Node.js, выполнив следующую команду из корневой папки проекта:

```console
node . --op getUsers
```

В результате вы получите некоторый ответ JSON от API Microsoft Graph и сможете просмотреть массив пользовательских объектов в консоли:

```console
You have selected: getUsers
request made to web API at: Fri Jan 22 2021 09:31:52 GMT-0800 (Pacific Standard Time)
{
    '@odata.context': 'https://graph.microsoft.com/v1.0/$metadata#users',
    value: [
        {
            displayName: 'Adele Vance'
            givenName: 'Adele',
            jobTitle: 'Retail Manager',
            mail: 'AdeleV@msaltestingjs.onmicrosoft.com',
            mobilePhone: null,
            officeLocation: '18/2111',
            preferredLanguage: 'en-US',
            surname: 'Vance',
            userPrincipalName: 'AdeleV@msaltestingjs.onmicrosoft.com',
            id: 'a6a218a5-f5ae-462a-acd3-581af4bcca00'
        }
    ]
}
```
:::image type="content" source="media/tutorial-v2-nodejs-console/screenshot.png" alt-text="Интерфейс командной строки с отображением ответа Graph":::

## <a name="how-the-application-works"></a>Принцип работы приложения

Это приложение использует [предоставление учетных данных клиента OAuth 2.0](./v2-oauth2-client-creds-grant-flow.md). Этот тип предоставления разрешения часто используется для взаимодействия между серверами, которое должно выполняться в фоновом режиме без непосредственного взаимодействия с пользователем. Процесс предоставления учетных данных позволяет веб-службе (конфиденциальный клиент) вместо олицетворения пользователя использовать свои собственные учетные данные для аутентификации при вызове другой веб-службы. Приложения, поддерживаемые этой моделью аутентификации, обычно представляют собой **управляющие программы** или **учетные записи служб**.

Область запросов для потока учетных данных клиента — это имя ресурса с добавлением `/.default`. Такая запись указывает Azure Active Directory (Azure AD) использовать разрешения уровня приложения, объявленные статически во время регистрации приложения. Кроме того, эти разрешения API должны быть предоставлены **администратором арендатора**.

## <a name="next-steps"></a>Дальнейшие действия

Если вы хотите узнать больше о разработке консольных приложений Node.js на платформе удостоверений Майкрософт, см. серию статей:

> [!div class="nextstepaction"]
> [Scenario: Управляющая программа](scenario-daemon-overview.md)