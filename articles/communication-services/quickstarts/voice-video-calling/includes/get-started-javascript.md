---
title: Краткое руководство. Добавление вызова VOIP к приложению с помощью Служб коммуникации Azure
description: Из этого руководства вы узнаете, как использовать клиентскую библиотеку Служб коммуникации Azure для реализации вызовов на JavaScript.
author: ddematheu
ms.author: nimag
ms.date: 08/11/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: d27a79e180a0219773a3094fb85f842773d75183
ms.sourcegitcommit: b4647f06c0953435af3cb24baaf6d15a5a761a9c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/02/2021
ms.locfileid: "101656622"
---
Из этого краткого руководства вы узнаете, как начать вызов с помощью клиентской библиотеки Служб коммуникации Azure для реализации вызовов на JavaScript.
В этом документе используются типы, определенные в библиотеке вызовов версии 1.0.0-beta.5.

> [!NOTE]
> В этом документе используется клиентская библиотека вызовов версии 1.0.0-beta.6.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Node.js](https://nodejs.org/), активная версия LTS и версия Maintenance LTS (рекомендуются версии 8.11.1 и 10.14.1).
- Активный ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- Маркер доступа пользователя для создания экземпляра клиента вызова. Узнайте, как [создать маркер доступа пользователя и обеспечить управление им](../../access-tokens.md).


[!INCLUDE [Calling with JavaScript](./get-started-javascript-setup.md)]

Вот этот код:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Communication Client - Calling Sample</title>
  </head>
  <body>
    <h4>Azure Communication Services</h4>
    <h1>Calling Quickstart</h1>
    <input 
      id="callee-id-input"
      type="text"
      placeholder="Who would you like to call?"
      style="margin-bottom:1em; width: 200px;"
    />
    <div>
      <button id="call-button" type="button" disabled="true">
        Start Call
      </button>
      &nbsp;
      <button id="hang-up-button" type="button" disabled="true">
        Hang Up
      </button>
    </div>
    <script src="./bundle.js"></script>
  </body>
</html>
```

Создайте файл в корневом каталоге проекта с именем **client.js**, чтобы включить логику приложения для этого краткого руководства. Добавьте следующий код, чтобы импортировать клиент вызова и получить ссылки на элементы модели DOM, чтобы мы могли присоединить нашу бизнес-логику. 

```javascript
import { CallClient, CallAgent } from "@azure/communication-calling";
import { AzureCommunicationTokenCredential } from '@azure/communication-common';

let call;
let callAgent;
const calleeInput = document.getElementById("callee-id-input");
const callButton = document.getElementById("call-button");
const hangUpButton = document.getElementById("hang-up-button");
```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки Служб коммуникации Azure для вызовов:

| Имя                             | Описание                                                                                                                                 |
| ---------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------- |
| CallClient                       | CallClient — это основная точка входа в клиентскую библиотеку для вызовов.                                                                       |
| CallAgent                        | CallAgent используется для инициирования вызовов и управления ими.                                                                                            |
| AzureCommunicationTokenCredential | Класс AzureCommunicationTokenCredential реализует интерфейс CommunicationTokenCredential, который используется для создания экземпляра CallAgent. |


## <a name="authenticate-the-client"></a>Аутентификация клиента

Вам необходимо заменить `<USER_ACCESS_TOKEN>` допустимым маркером доступа пользователя для вашего ресурса. Если у вас еще нет доступного маркера, см. документацию по [маркеру доступа пользователя](../../access-tokens.md). С помощью `CallClient` инициализируйте экземпляр `CallAgent` с `CommunicationTokenCredential`, который позволит нам выполнять и принимать вызовы. Добавьте следующий код в файл **client.js**:

```javascript
async function init() {
    const callClient = new CallClient();
    const tokenCredential = new AzureCommunicationTokenCredential("<USER ACCESS TOKEN>");
    callAgent = await callClient.createCallAgent(tokenCredential);
    callButton.disabled = false;
}
init();
```

## <a name="start-a-call"></a>Инициирование вызова

Добавьте обработчик событий для инициации вызова при нажатии `callButton`:

```javascript
callButton.addEventListener("click", () => {
    // start a call
    const userToCall = calleeInput.value;
    call = callAgent.startCall(
        [{ communicationUserId: userToCall }],
        {}
    );
    // toggle button states
    hangUpButton.disabled = false;
    callButton.disabled = true;
});
```

## <a name="end-a-call"></a>Завершение вызова

Добавьте прослушиватель событий для завершения текущего вызова при нажатии `hangUpButton`:

```javascript
hangUpButton.addEventListener("click", () => {
  // end the current call
  call.hangUp({ forEveryone: true });

  // toggle button states
  hangUpButton.disabled = true;
  callButton.disabled = false;
});
```

Свойство `forEveryone` позволяет завершить вызов для всех его участников.

## <a name="run-the-code"></a>Выполнение кода

Чтобы создать и запустить приложение, используйте `webpack-dev-server`. Выполните следующую команду, чтобы создать пакет узла приложения на локальном веб-сервере:

```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```

Откройте веб-браузер и перейдите по адресу http://localhost:8080/. Вы увидите следующее:

:::image type="content" source="../media/javascript/calling-javascript-app.png" alt-text="Снимок экрана: готовое приложение JavaScript.":::

Вы можете выполнить исходящий VOIP-вызов, указав идентификатор пользователя в текстовом поле и нажав кнопку **Начать вызов**. При вызове `8:echo123` вы будете подключены к эхо-боту, что позволит начать работу и проверить работоспособность ваших аудиоустройств.
