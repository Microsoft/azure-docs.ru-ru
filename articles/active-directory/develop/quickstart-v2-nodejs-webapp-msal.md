---
title: Краткое руководство. Добавление проверки подлинности в веб-приложение Node с помощью MSAL Node | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как реализовать проверку подлинности с помощью веб-приложения Node.js и библиотеки проверки подлинности Майкрософт (MSAL) для Node.js.
services: active-directory
author: mmacy
manager: celested
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/22/2020
ms.author: marsma
ms.custom: aaddev, scenarios:getting-started, languages:js, devx-track-js
ms.openlocfilehash: 72eb6e77cfbcae662181f642393085185514eed6
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106550954"
---
# <a name="quickstart-sign-in-users-and-get-an-access-token-in-a-node-web-app-using-the-auth-code-flow"></a>Краткое руководство. Вход пользователей и получение маркера доступа в веб-приложении Node с помощью потока кода авторизации

При работе с этим кратким руководством вы скачаете и запустите пример кода. Такой пример кода демонстрирует, как в веб-приложении Node.js реализовать вход пользователей с помощью потока кода авторизации, а также как получить маркер доступа для вызова API Microsoft Graph. 

Иллюстрацию см. в разделе [Как работает этот пример](#how-the-sample-works).

В рамках этого краткого руководства используется библиотека проверки подлинности Майкрософт для Node.js (MSAL Node) с потоком кода авторизации.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. [Создать подписку Azure бесплатно](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) или любой другой редактор кода.

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>Регистрация и скачивание приложения, используемого в этом кратком руководстве
>
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Введите значение **Name** (Имя) для приложения. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
> 1. В разделе **Поддерживаемые типы учетных записей** выберите **Accounts in any organizational directory and personal Microsoft accounts** (Учетные записи в любом каталоге организации и личные учетные записи Майкрософт).
> 1. В поле **URI перенаправления** укажите значение `http://localhost:3000/redirect`.
> 1. Выберите **Зарегистрировать**. 
> 1. На странице приложения **Обзор** запишите **идентификатор приложения (клиента)** для использования в будущем.
> 1. В разделе **Управление** выберите **Сертификаты и секреты** > **Создать секрет клиента**.  Оставьте поле для описания пустым, а параметр срока действия — по умолчанию. Затем выберите **Добавить**.
> 1. Запишите **значение** параметра **Секрет клиента** для последующего использования.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-the-application-in-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
> Чтобы пример кода из этого краткого руководства заработал, создайте секрет клиента и добавьте URL-адрес ответа как **http://localhost:3000/redirect** .
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести это изменение для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-windows-desktop/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-project"></a>Шаг 2. Скачивание проекта

> [!div renderon="docs"]
> [Скачайте основные файлы проекта](https://github.com/Azure-Samples/ms-identity-node/archive/main.zip), чтобы запустить проект с помощью веб-сервера, используя Node.js.

> [!div renderon="portal" class="sxs-lookup"]
> Запустите проект с помощью веб-сервера, используя Node.js.

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/ms-identity-node/archive/main.zip)

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-node-app"></a>Шаг 3. Настройка приложения Node
>
> Извлеките проект и откройте папку *ms-identity-node-main*, а затем — файл *index.js*.
> Задайте `clientID`, используя параметр **Идентификатор приложения (клиент)** .
> Задайте `clientSecret`, используя **значение** параметра **Секрет клиента**.
>
>```javascript
>const config = {
>    auth: {
>        clientId: "Enter_the_Application_Id_Here",
>        authority: "https://login.microsoftonline.com/common",
>        clientSecret: "Enter_the_Client_Secret_Here"
>    },
>    system: {
>        loggerOptions: {
>            loggerCallback(loglevel, message, containsPii) {
>                console.log(message);
>            },
>            piiLoggingEnabled: false,
>            logLevel: msal.LogLevel.Verbose,
>        }
>    }
>};
> ```

> [!div renderon="docs"]
>
> Измените значения в разделе `config`, как описано далее.
>
> - `Enter_the_Application_Id_Here` содержит **идентификатор приложения (клиента)** для зарегистрированного приложения.
>
>    Чтобы найти значение параметра **Идентификатор приложения (клиента)** , на портале Azure перейдите на страницу **Обзор** регистрации приложения.
> - `Enter_the_Client_Secret_Here` — это **значение** параметра **Секрет клиента** для зарегистрированного приложения.
>
>    Чтобы получить или создать новый **секрет клиента**, в разделе **Управление** выберите **Сертификаты и секреты**.
>
> Значение `authority` по умолчанию представляет основное (глобальное) облако Azure:
>
> ```javascript
> authority: "https://login.microsoftonline.com/common",
> ```
>
> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Шаг 3. Приложение настроено и готово к запуску
>
> [!div renderon="docs"]
>
> #### <a name="step-4-run-the-project"></a>Шаг 4. Запуск проекта

Запустите проект с помощью Node.js, выполнив приведенные ниже действия.

1. Чтобы запустить сервер, выполните в каталоге проекта следующую команду.
    ```console
    npm install
    npm start
    ```
1. Перейдите по адресу `http://localhost:3000/`.

1. Выберите **Войти**, чтобы начать процесс входа в систему.

    После первого входа предоставьте приложению разрешение на использование данных вашего профиля для входа. После успешного входа вы увидите сообщение журнала в командной строке.

## <a name="more-information"></a>Дополнительные сведения

### <a name="how-the-sample-works"></a>Как работает этот пример

При запуске примера веб-сервер размещается на localhost (порт 3000). Когда веб-браузер получает доступ к этому сайту, этот пример немедленно перенаправляет пользователя на страницу проверки подлинности Майкрософт. В связи с этим пример не содержит *HTML* или отображаемые элементы. При успешной проверке подлинности отображается сообщение "ОК".

### <a name="msal-node"></a>MSAL Node

Библиотека MSAL Node — это библиотека, используемая для выполнения входа пользователей и запросов маркеров, которые нужны для доступа к API, защищенному платформой удостоверений Майкрософт. Последнюю версию можно скачать с помощью диспетчера пакетов Node.js (NPM):

```console
npm install @azure/msal-node
```

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление проверки подлинности в имеющееся веб-приложение — пример кода GitHub >](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/samples/msal-node-samples/auth-code)
