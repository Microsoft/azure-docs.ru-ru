---
title: Краткое руководство. Вызов Microsoft Graph из консольного приложения Node.js | Azure
titleSuffix: Microsoft identity platform
description: В этом кратком руководстве описано, как скачать и выполнить пример кода. Вы узнаете, как консольное приложение Node.js может получить маркер доступа и вызвать API, защищенный конечной точкой платформы удостоверений Майкрософт, с помощью собственного удостоверения приложения.
services: active-directory
author: derisen
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.date: 02/17/2021
ms.author: v-doeris
ms.openlocfilehash: 4360810d460c5fc8598ce302ad8b82f65d2d819e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101653751"
---
# <a name="quickstart-acquire-a-token-and-call-microsoft-graph-api-from-a-nodejs-console-app-using-apps-identity"></a>Краткое руководство. Получение маркера и вызов API Microsoft Graph из консольного приложения Node.js с помощью удостоверения приложения

При работе с этим кратким руководством вы скачаете и выполните пример кода, который демонстрирует получение маркера доступа с помощью удостоверения приложения из консольного приложения Node.js для вызова API Microsoft Graph и отображения [списка пользователей](/graph/api/user-list) в каталоге. а также как автоматическое задание или служба Windows могут выполняться с удостоверением приложения, а не пользователя.

В рамках этого краткого руководства используется [библиотека проверки подлинности Майкрософт для Node.js (MSAL Node)](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) с [предоставлением учетных данных клиента](v2-oauth2-client-creds-grant-flow.md).

## <a name="prerequisites"></a>Предварительные требования

* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) или любой другой редактор кода.

> [!div renderon="docs"]
> ## <a name="register-and-download-the-sample-application"></a>Регистрация и скачивание примера приложения
>
> Выполните шаги ниже, чтобы начать.
>
> [!div renderon="docs"]
> #### <a name="step-1-register-the-application"></a>Шаг 1. Регистрация приложения
> Чтобы зарегистрировать приложение и добавить сведения о его регистрации в решение вручную, сделайте следующее:
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Введите **имя** приложения, например `msal-node-cli`. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
> 1. Выберите **Зарегистрировать**.
> 1. В разделе **Управление** выберите **Сертификаты и секреты**.
> 1. В разделе **Секреты клиента** выберите **Новый секрет клиента**, введите имя, а затем нажмите кнопку **Добавить**. Сохраните значение секрета в надежном месте, так как это значение потребуется вам в дальнейшем.
> 1. В разделе **Управление** выберите **Разрешения API** > **Добавить разрешение**. Выберите **Microsoft Graph**.
> 1. Выберите **Разрешения приложений**.
> 1. В узле **Пользователь** выберите **User.Read.All**, а затем щелкните **Добавить разрешения**.

> [!div class="sxs-lookup" renderon="portal"]
> ### <a name="download-and-configure-the-sample-app"></a>Скачивание и настройка примера приложения
>
> #### <a name="step-1-configure-the-application-in-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Для работы примера кода в этом кратком руководстве необходимо создать секрет клиента и добавить разрешение приложения API Graph **User.Read.All**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-netcore-daemon/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-nodejs-sample-project"></a>Шаг 2. Скачивание примера проекта Node.js

> [!div renderon="docs"]
> [Скачивание примера кода](https://github.com/azure-samples/ms-identity-javascript-nodejs-console/archive/main.zip)

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Скачивание примера кода](https://github.com/azure-samples/ms-identity-javascript-nodejs-console/archive/main.zip)

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> #### <a name="step-3-configure-the-nodejs-sample-project"></a>Шаг 3. Настройка примера проекта Node.js
>
> 1. Распакуйте ZIP-файл в локальную папку, расположенную как можно ближе к корню диска (например, *C:\Azure-Samples*).
> 1. Измените файл *.env*, заменив значения полей `TENANT_ID`, `CLIENT_ID` и `CLIENT_SECRET` следующим фрагментом кода:
>
>    ```
>    "TENANT_ID": "Enter_the_Tenant_Id_Here",
>    "CLIENT_ID": "Enter_the_Application_Id_Here",
>    "CLIENT_SECRET": "Enter_the_Client_Secret_Here"
>    ```
>    Где:
>    - `Enter_the_Application_Id_Here` — это **идентификатор** зарегистрированного вами ранее приложения (клиента). Этот идентификатор можно найти в области **Обзор** регистрации приложения на портале Azure.
>    - `Enter_the_Tenant_Id_Here` — замените это значение **идентификатором арендатора** или **именем арендатора** (например, contoso.microsoft.com).  Эти значения можно найти в области **Обзор** регистрации приложения на портале Azure.
>    - `Enter_the_Client_Secret_Here` — замените это значение созданным ранее секретом клиента. Чтобы создать новый ключ, выберите **Сертификаты и секреты** в параметрах регистрации приложения на портале Azure.
>
> > [!WARNING]
> > Использовать секреты в формате открытого текста в исходном коде небезопасно. В этой статье такой секрет клиента применяется только для простоты. Используйте [учетные данные сертификата](active-directory-certificate-credentials.md), а не секреты клиента, в конфиденциальных клиентских приложениях, особенно если вы планируете развертывать такие приложения в рабочей среде.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-admin-consent"></a>Шаг 3. Согласие администратора

> [!div renderon="docs"]
> #### <a name="step-4-admin-consent"></a>Шаг 4. Согласие администратора

Если попытаться запустить приложение на этом этапе, вы получите ошибку *HTTP 403 — Forbidden* (запрещено): `Insufficient privileges to complete the operation`. Эта ошибка возникает, так как *разрешение только для приложения* предоставляется с **согласия администратора**. Следовательно, глобальный администратор каталога должен предоставить такое согласие приложению. Выберите один из следующих вариантов с учетом своей роли:

##### <a name="global-tenant-administrator"></a>Глобальный администратор клиента

> [!div renderon="docs"]
> Если вы являетесь глобальным администратором арендатора, перейдите на страницу **Разрешения API** в разделе регистрации приложения на портале Azure и выберите **Предоставить согласие администратора для {имя арендатора}** (где {имя арендатора} — это название вашего каталога).

> [!div renderon="portal" class="sxs-lookup"]
> Если вы являетесь глобальным администратором, перейдите на страницу **Разрешения API** и выберите **Предоставить согласия администратора для имя_клиента**.
> > [!div id="apipermissionspage"]
> > [Переход на страницу "Разрешения API"]()

##### <a name="standard-user"></a>Обычный пользователь

Если вы являетесь обычным пользователем арендатора, попросите глобального администратора предоставить **согласие администратора** для вашего приложения. Чтобы сделать это, предоставьте следующий URL-адрес администратору:

```url
https://login.microsoftonline.com/Enter_the_Tenant_Id_Here/adminconsent?client_id=Enter_the_Application_Id_Here
```

> [!div renderon="docs"]
>> Где:
>> * `Enter_the_Tenant_Id_Here` — замените это значение на **идентификатор клиента** или **имя клиента** (например, contoso.microsoft.com).
>> * `Enter_the_Application_Id_Here` — это **идентификатор приложения (клиента)** , которое вы зарегистрировали.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-4-run-the-application"></a>Шаг 4. Выполнение приложения

> [!div renderon="docs"]
> #### <a name="step-5-run-the-application"></a>Шаг 5. Выполнение приложения

Найдите корневую папку примера (где располагается файл `package.json`) в командной строке или консоли. Зависимости этого примера необходимо установить один раз:

```console
npm install
```

Затем запустите приложение с помощью командной строки или консоли:

```console
node . --op getUsers
```

В выходных данных консоли вы должны увидеть фрагмент кода JSON, представляющий список пользователей в вашем каталоге Azure AD.

## <a name="about-the-code"></a>Примечания о коде

Ниже рассмотрены некоторые важные аспекты примера приложения.

### <a name="msal-node"></a>MSAL Node

Библиотека [MSAL Node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) нужна для обработки входа пользователей и запросов маркеров, которые используются для доступа к API, защищенному платформой удостоверений Майкрософт. Как описано выше, в рамках этого краткого руководства маркеры запрашиваются по разрешениям приложения (с использованием собственного удостоверения приложения), а не по делегированным разрешениям. Поток аутентификации, используемый в данном случае, называется [потоком учетных данных клиента OAuth 2.0](v2-oauth2-client-creds-grant-flow.md). Дополнительные сведения об использовании MSAL Node с приложениями управляющей программы см. в статье [Сценарий. Приложение управляющей программы, которое вызывает веб-API](scenario-daemon-overview.md).

 Вы можете установить MSAL Node, выполнив следующую команду npm.

```console
npm install @azure/msal-node --save
```

### <a name="msal-initialization"></a>Инициализация MSAL

Добавив следующий код, вы можете добавить ссылку на MSAL.

```javascript
const msal = require('@azure/msal-node');
```

Затем выполните инициализацию MSAL с помощью следующего кода.

```javascript
const msalConfig = {
    auth: {
        clientId: "Enter_the_Application_Id_Here",
        authority: "https://login.microsoftonline.com/Enter_the_Tenant_Id_Here",
        clientSecret: "Enter_the_Client_Secret_Here",
   }
};
const cca = new msal.ConfidentialClientApplication(msalConfig);
```

> | Где: |Описание |
> |---------|---------|
> | `clientId` | **Идентификатор приложения (клиента)** , зарегистрированного на портале Azure. Это значение можно найти на странице приложения **Обзор** на портале Azure. |
> | `authority`    | Конечная точка STS для проверки подлинности пользователей. Обычно `https://login.microsoftonline.com/{tenant}` для общедоступного облака, где {tenant} — имя или идентификатор вашего клиента.|
> | `clientSecret` | Секрет клиента, созданный для приложения на портале Azure. |

Дополнительные сведения см. в [справочной документации по `ConfidentialClientApplication`](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-node/docs/initialize-confidential-client-application.md).

### <a name="requesting-tokens"></a>Запрос маркеров

Чтобы запросить маркер с помощью удостоверение приложения, используйте метод `acquireTokenByClientCredential`.

```javascript
const tokenRequest = {
    scopes: [ 'https://graph.microsoft.com/.default' ],
};

const tokenResponse = await cca.acquireTokenByClientCredential(tokenRequest);
```

> |Где:| Описание |
> |---------|---------|
> | `tokenRequest` | Содержит запрошенные области. Для конфиденциальных клиентов следует использовать формат, аналогичный `{Application ID URI}/.default`, который указывает, что запрашиваемые области — это те, которые статически определены в объекте приложения, заданном на портале Azure (для Microsoft Graph `{Application ID URI}` указывает на `https://graph.microsoft.com`). Для пользовательских веб-API `{Application ID URI}` определяется в разделе **Предоставление API** в разделе регистрации приложения на портале Azure. |
> | `tokenResponse` | Ответ содержит маркер доступа для запрошенных областей. |

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие шаги

Дополнительные сведения о разработке приложений управляющих и консольных программ с помощью MSAL Node см. в учебнике:

> [!div class="nextstepaction"]
> [Создание управляющей программы, которая вызывает веб-API](tutorial-v2-nodejs-console.md)