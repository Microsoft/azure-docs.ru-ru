---
title: включить файл
description: включить файл
services: azure-communication-services
author: tomaschladek
manager: nmurav
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: tchladek
ms.openlocfilehash: 68b5fc7c958f611c43ba3f38bf30ceb63608886a
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106113365"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Node.js](https://nodejs.org/), активная версия LTS и версия Maintenance LTS (рекомендуются версии 8.11.1 и 10.14.1).
- Активный ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../create-communication-resource.md)

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>Создание нового приложения Node.js

Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.

```console
mkdir access-tokens-quickstart && cd access-tokens-quickstart
```

Воспользуйтесь командой `npm init -y`, чтобы создать файл **package.json** с параметрами по умолчанию.

```console
npm init -y
```

### <a name="install-the-package"></a>Установка пакета

Используйте команду `npm install`, чтобы установить пакет SDK для Служб коммуникации Azure для удостоверений для JavaScript.

```console

npm install @azure/communication-identity --save

```

Параметр `--save` указывает библиотеку как зависимость в файле пакета **package.json**.

## <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Откройте новый текстовый файл в редакторе кода.
1. Добавьте вызов `require`, чтобы загрузить `CommunicationIdentityClient`.
1. Создайте структуру программы, включая простую обработку исключений.

Используйте следующий код:

```javascript
const { CommunicationIdentityClient } = require('@azure/communication-identity');

const main = async () => {
  console.log("Azure Communication Services - Access Tokens Quickstart")

  // Quickstart code goes here
};

main().catch((error) => {
  console.log("Encountered an error");
  console.log(error);
})
```

1. Сохраните новый файл как **issue-access-token.js** в каталоге *access-tokens-quickstart*.

## <a name="authenticate-the-client"></a>Аутентификация клиента

Создайте экземпляр `CommunicationIdentityClient` с использованием строки подключения. Приведенный ниже код извлекает строку подключения для ресурса из переменной среды с именем `COMMUNICATION_SERVICES_CONNECTION_STRING`. См. сведения о том, как [управлять строкой подключения ресурса](../create-communication-resource.md#store-your-connection-string).

Добавьте следующий код в метод `main`:

```javascript
// This code demonstrates how to fetch your connection string
// from an environment variable.
const connectionString = process.env['COMMUNICATION_SERVICES_CONNECTION_STRING'];

// Instantiate the identity client
const identityClient = new CommunicationIdentityClient(connectionString);
```

Кроме того, можно разделить конечную точку и ключ доступа.
```javascript
// This code demonstrates how to fetch your endpoint and access key
// from an environment variable.
const endpoint = process.env["COMMUNICATION_SERVICES_ENDPOINT"];
const accessKey = process.env["COMMUNICATION_SERVICES_ACCESSKEY"];
const tokenCredential = new AzureKeyCredential(accessKey);
// Instantiate the identity client
const identityClient = new CommunicationIdentityClient(endpoint, tokenCredential)
```

Если вы настроили управляемое удостоверение, то из статьи [Использование управляемых удостоверений](../managed-identity.md) можете узнать, как использовать его для аутентификации.
```javascript
const endpoint = process.env["COMMUNICATION_SERVICES_ENDPOINT"];
const tokenCredential = new DefaultAzureCredential();
const identityClient = new CommunicationIdentityClient(endpoint, tokenCredential);
```

## <a name="create-an-identity"></a>Создание удостоверения

Службы коммуникации Azure позволяют использовать упрощенный каталог удостоверений. Чтобы создать новую запись в каталоге с уникальным идентификатором (`Id`), используйте метод `createUser`. Сохраните полученное удостоверение с сопоставлением с пользователями вашего приложения. Например, сохраните их в базе данных сервера приложений. Удостоверение потребуется позже для выдачи маркеров доступа.

```javascript
let identityResponse = await identityClient.createUser();
console.log(`\nCreated an identity with ID: ${identityResponse.communicationUserId}`);
```

## <a name="issue-access-tokens"></a>Выпуск маркеров доступа

Чтобы выдать маркер доступа для имеющегося удостоверения Служб коммуникации, используйте метод `getToken`. Параметр `scopes` определяет набор базовых функций, которые будут авторизовать этот маркер доступа. Ознакомьтесь со [списком поддерживаемых действий](../../concepts/authentication.md). Новый экземпляр параметра `communicationUser` можно создать на основе строкового представления удостоверения Служб коммуникации Azure.

```javascript
// Issue an access token with the "voip" scope for an identity
let tokenResponse = await identityClient.getToken(identityResponse, ["voip"]);
const { token, expiresOn } = tokenResponse;
console.log(`\nIssued an access token with 'voip' scope that expires at ${expiresOn}:`);
console.log(token);
```

Маркеры доступа — это недолговечные учетные данные, которые необходимо выдавать повторно. Если этого не сделать, это может привести к нарушению работы пользователей приложения. Свойство ответа `expiresOn` указывает время существования маркера доступа.

## <a name="create-an-identity-and-issue-an-access-token-within-the-same-request"></a>Создание удостоверения и выдача маркера доступа в рамках одного запроса

Чтобы создать удостоверение Служб коммуникации и выдать для него маркер доступа, используйте метод `createUserAndToken`. Параметр `scopes` определяет набор базовых функций, которые будут авторизовать этот маркер доступа. Ознакомьтесь со [списком поддерживаемых действий](../../concepts/authentication.md).

```javascript
// Issue an identity and an access token with the "voip" scope for the new identity
let identityTokenResponse = await identityClient.createUserAndToken(["voip"]);
const { token, expiresOn, user } = identityTokenResponse;
console.log(`\nCreated an identity with ID: ${user.communicationUserId}`);
console.log(`\nIssued an access token with 'voip' scope that expires at ${expiresOn}:`);
console.log(token);
```

## <a name="refresh-access-tokens"></a>Обновление токенов доступа

Чтобы обновить токены доступа, нужно всего лишь вызвать `getToken` с тем же удостоверением, которое использовалось для выпуска токенов. Для обновляемых токенов также необходимо указать `scopes`.

```javascript
// // Value of identityResponse represents the Azure Communication Services identity stored during identity creation and then used to issue the tokens being refreshed
let refreshedTokenResponse = await identityClient.getToken(identityResponse, ["voip"]);
```


## <a name="revoke-access-tokens"></a>Отмена маркеров доступа

В некоторых случаях вы можете явно отменить маркеры доступа. Например, когда пользователь приложения меняет пароль, который он использует для проверки подлинности в службе. Метод `revokeTokens` отменяет все активные маркеры доступа, выданные идентификатору.

```javascript
await identityClient.revokeTokens(identityResponse);
console.log(`\nSuccessfully revoked all access tokens for identity with ID: ${identityResponse.communicationUserId}`);
```

## <a name="delete-an-identity"></a>Удаление удостоверения

При удалении удостоверения отзываются все активные маркеры доступа и запрещается выдача последующих маркеров для удостоверения. Вместе с этим удаляется и все хранимое содержимое, связанное с удостоверением.

```javascript
await identityClient.deleteUser(identityResponse);
console.log(`\nDeleted the identity with ID: ${identityResponse.communicationUserId}`);
```

## <a name="run-the-code"></a>Выполнение кода

В окне консоли перейдите в каталог, содержащий файл *issue-access-token.js*, а затем выполните команду `node`, чтобы запустить приложение.

```console
node ./issue-access-token.js
```
