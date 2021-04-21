---
title: Краткое руководство. Присоединение к конференции Teams из веб-приложения
description: В этом руководстве описывается, как присоединиться к конференции Teams с помощью пакета SDK для вызовов Служб коммуникации Azure для JavaScript.
author: chpalm
ms.author: mikben
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 6747d1d3cfba1c9e2bee7a8a7a48d67d6bed9f8e
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107564547"
---
В этом кратком руководстве описывается, как присоединиться к конференции Teams с помощью пакета SDK для вызовов Служб коммуникации Azure для JavaScript.

## <a name="prerequisites"></a>Предварительные требования

- Рабочее веб-приложение, [вызывающее Службы коммуникации Azure](../getting-started-with-calling.md).
- [Развертывание Teams](/deployoffice/teams-install).


## <a name="add-the-teams-ui-controls"></a>Добавление элементов управления пользовательского интерфейса Teams

Замените код в index.html приведенным ниже фрагментом кода.
Текстовое поле будет использоваться для ввода контекста собрания Teams, а кнопка — для присоединения к указанному собранию:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Communication Client - Calling Sample</title>
</head>
<body>
    <h4>Azure Communication Services</h4>
    <h1>Teams meeting join quickstart</h1>
    <input id="teams-link-input" type="text" placeholder="Teams meeting link"
        style="margin-bottom:1em; width: 300px;" />
        <p>Call state <span style="font-weight: bold" id="call-state">-</span></p>
        <p><span style="font-weight: bold" id="recording-state"></span></p>
    <div>
        <button id="join-meeting-button" type="button" disabled="false">
            Join Teams Meeting
        </button>
        <button id="hang-up-button" type="button" disabled="true">
            Hang Up
        </button>
    </div>
    <script src="./bundle.js"></script>
</body>

</html>
```

## <a name="enable-the-teams-ui-controls"></a>Включение элементов управления пользовательского интерфейса Teams

Замените содержимое файла client.js приведенным ниже фрагментом кода.

```javascript
import { CallClient } from "@azure/communication-calling";
import { Features } from "@azure/communication-calling";
import { AzureCommunicationTokenCredential } from '@azure/communication-common';

let call;
let callAgent;
const meetingLinkInput = document.getElementById('teams-link-input');
const hangUpButton = document.getElementById('hang-up-button');
const teamsMeetingJoinButton = document.getElementById('join-meeting-button');
const callStateElement = document.getElementById('call-state');
const recordingStateElement = document.getElementById('recording-state');

async function init() {
    const callClient = new CallClient();
    const tokenCredential = new AzureCommunicationTokenCredential("<USER ACCESS TOKEN>");
    callAgent = await callClient.createCallAgent(tokenCredential, {displayName: 'ACS user'});
    teamsMeetingJoinButton.disabled = false;
}
init();

hangUpButton.addEventListener("click", async () => {
    // end the current call
    await call.hangUp();
  
    // toggle button states
    hangUpButton.disabled = true;
    teamsMeetingJoinButton.disabled = false;
    callStateElement.innerText = '-';
  });

teamsMeetingJoinButton.addEventListener("click", () => {    
    // join with meeting link
    call = callAgent.join({meetingLink: meetingLinkInput.value}, {});
    
    call.on('stateChanged', () => {
        callStateElement.innerText = call.state;
    })

    call.api(Features.Recording).on('isRecordingActiveChanged', () => {
        if (call.api(Features.Recording).isRecordingActive) {
            recordingStateElement.innerText = "This call is being recorded";
        }
        else {
            recordingStateElement.innerText = "";
        }
    });
    // toggle button states
    hangUpButton.disabled = false;
    teamsMeetingJoinButton.disabled = true;
});
```

## <a name="get-the-teams-meeting-link"></a>Получение ссылки на собрание Teams

Ссылку на собрание Teams можно получить с помощью интерфейсов API Graph. Это действие подробно описано в [документации по Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true).
Пакет SDK вызовов Служб коммуникации принимает полную ссылку на собрание Teams. Эта ссылка возвращается как часть ресурса `onlineMeeting`, доступного в [свойстве `joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta&preserve-view=true). Кроме того, вы можете получить необходимые сведения о собрании, воспользовавшись URL-адресом для **участия в собрании** в самом приглашении на собрание Teams.

## <a name="run-the-code"></a>Выполнение кода

Пользователи Webpack могут использовать `webpack-dev-server` для сборки и запуска приложения. Выполните следующую команду, чтобы создать пакет узла приложения на локальном веб-сервере:

```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```

Откройте веб-браузер и перейдите по адресу http://localhost:8080/. Вы увидите следующее:

:::image type="content" source="../media/javascript/acs-join-teams-meeting-quickstart.PNG" alt-text="Снимок экрана: готовое приложение JavaScript.":::

Вставьте контекст Teams в текстовое поле и нажмите *Присоединиться к собранию Teams*, чтобы присоединиться к собранию Teams из приложения Служб коммуникации Azure.
