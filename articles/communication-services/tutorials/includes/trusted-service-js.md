---
title: Доверенная служба (JavaScript)
description: Это версия JavaScript для создания доверенной службы для Служб коммуникации.
author: dademath
manager: nimag
services: azure-communication-services
ms.author: ddematheu2
ms.date: 03/10/2021
ms.topic: include
ms.service: azure-communication-services
ms.openlocfilehash: 41d959468e3183af00d2ab514e7c1bf0a134a1f8
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103490486"
---
## <a name="download-code"></a>Скачивание кода

Итоговый код для этого краткого руководства можно найти на сайте [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/trusted-authentication-service).

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Visual Studio Code](https://code.visualstudio.com/) на одной из [поддерживаемых платформ](https://code.visualstudio.com/docs/supporting/requirements#_platforms).
- [Node.js](https://nodejs.org/), активная версия LTS и версия Maintenance LTS (рекомендуется 10.14.1). Используйте команду `node --version`, чтобы проверить установленную версию.
- [Расширение "Функции Azure"](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) для Visual Studio Code.
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../quickstarts/create-communication-resource.md)

## <a name="overview"></a>Обзор

:::image type="content" source="../media/trusted-service-architecture.png" alt-text="Схема архитектуры доверенной службы":::

В рамках этого учебника мы создадим функцию Azure, которая будет использоваться в качестве доверенной службы подготовки маркеров. С помощью сведений из этого учебника вы можете выполнить начальную загрузку собственной службы подготовки маркеров.

Эта служба отвечает за проверку подлинности пользователей в Службах коммуникации Azure. Пользователям ваших приложений Служб коммуникации потребуется `Access Token`, чтобы участвовать в беседах и выполнять вызовы по протоколу VoIP. Функция Azure будет работать как доверенный посредник между пользователем и Службами коммуникации. Это позволяет подготавливать маркеры доступа, не предоставляя пользователям строку подключения к ресурсам.

Дополнительные сведения см. в основной документации по [архитектуре "клиент-сервер"](../../concepts/client-and-server-architecture.md) и [проверке подлинности и авторизации](../../concepts/authentication.md).

## <a name="setting-up"></a>Настройка

### <a name="azure-functions-set-up"></a>Настройка функций Azure

Сначала создадим базовую структуру для нашей функции Azure. Пошаговые инструкции по настройке можно найти на следующей странице: [Краткое руководство. Создание функции в Azure с помощью Visual Studio Code](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript)

Для функции Azure потребуется следующая конфигурация:

- Язык: JavaScript
- Шаблон: Триггер HTTP
- Уровень авторизации: Анонимный (можно изменить позже, если вы предпочитаете другую модель авторизации)
- Имя функции: Определяется пользователем

После выполнения [инструкций из этого краткого руководства](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript) с использованием приведенной выше конфигурации у вас должен быть проект в Visual Studio Code для функции Azure с файлом `index.js`, содержащим саму функцию. Код внутри этого файла должен быть следующим:

```javascript

module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    const name = (req.query.name || (req.body && req.body.name));
    const responseMessage = name
        ? "Hello, " + name + ". This HTTP triggered function executed successfully."
        : "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.";

    context.res = {
        // status: 200, /* Defaults to 200 */
        body: responseMessage
    };
}

```

Теперь мы приступим к установке библиотек Служб коммуникации Azure.

### <a name="install-communication-services-libraries"></a>Установка библиотек Служб коммуникации

Для создания `User Access Tokens` мы будем использовать библиотеку `Identity`.

Используйте команду `npm install`, чтобы установить клиентскую библиотеку удостоверений Служб коммуникации для JavaScript.

```console

npm install @azure/communication-identity --save

```

Параметр `--save` указывает библиотеку как зависимость в файле пакета **package.json**.

В верхней части файла `index.js` импортируйте интерфейс для `CommunicationIdentityClient`.

```javascript
const { CommunicationIdentityClient } = require('@azure/communication-identity');
```

## <a name="access-token-generation"></a>Создание маркера доступа

Чтобы функция Azure создавала `User Access Tokens`, сначала необходимо использовать строку подключения для нашего ресурса Служб коммуникации.

Дополнительные сведения о получении строки подключения см. в [кратком руководстве по подготовке ресурсов](../../quickstarts/create-communication-resource.md).

``` javascript
const connectionString = 'INSERT YOUR RESOURCE CONNECTION STRING'
```

Далее мы изменим исходную функцию для создания `User Access Tokens`.

`User Access Tokens` формируются путем создания пользователя из метода `createUser`. После создания пользователя можно использовать метод `issueToken` для создания маркера для этого пользователя, который возвращает функцию Azure.

В этом примере мы укажем в качестве области маркера `voip`. Для вашего приложения могут потребоваться другие области. См. дополнительные сведения об [областях](../../quickstarts/access-tokens.md).

```javascript
module.exports = async function (context, req) {
    let tokenClient = new CommunicationIdentityClient(connectionString);

    const user = await tokenClient.createUser();

    const userToken = await tokenClient.issueToken(user, ["voip"]);

    context.res = {
        body: userToken
    };
}
```

Для имеющихся `CommunicationUser` Служб коммуникации можно пропустить шаг создания и просто сгенерировать маркер доступа. Дополнительные сведения см. в [кратком руководстве по созданию маркеров доступа пользователей](../../quickstarts/access-tokens.md).

## <a name="test-the-azure-function"></a>Тестирование функции Azure

Запустите функцию Azure локально, нажав клавишу `F5`. Это позволит локально инициализировать функцию Azure и сделать ее доступной через: `http://localhost:7071/api/FUNCTION_NAME`. Ознакомьтесь с дополнительной документацией по [локальному выполнению](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript#run-the-function-locally).

Откройте URL-адрес в браузере, и вы увидите текст ответа с идентификатором пользователя коммуникации, маркером и его сроком действия.

:::image type="content" source="../media/trusted-service-sample-response.png" alt-text="Снимок экрана с примером ответа для созданной функции Azure.":::

## <a name="deploy-the-function-to-azure"></a>Развертывание функции в Azure

Чтобы развернуть функцию Azure, выполните эти [пошаговые инструкции](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript#sign-in-to-azure).

Таким образом, вам потребуется сделать следующее:
1. Войдите в Azure из Visual Studio.
2. Опубликуйте проект в учетной записи Azure. Выберите имеющуюся подписку.
3. Создайте ресурс функции Azure с помощью мастера Visual Studio или используйте имеющийся ресурс. Новый ресурс необходимо настроить для нужного региона, среды выполнения и уникального идентификатора.
4. Дождитесь завершения развертывания.
5. Выполните функцию 🎉.

## <a name="run-azure-function"></a>Запуск функции Azure

Запустите функцию Azure локально с помощью URL-адреса `http://<function-appn-ame>.azurewebsites.net/api/<function-name>`.

Чтобы найти URL-адрес, щелкните правой кнопкой мыши функцию в Visual Studio Code и скопируйте URL-адрес функции.

Дополнительные сведения см. в разделе о [запуске функции в Azure](../../../azure-functions/create-first-function-vs-code-csharp.md?pivots=programming-language-javascript#run-the-function-in-azure).
