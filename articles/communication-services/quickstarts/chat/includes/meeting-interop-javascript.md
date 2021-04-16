---
title: Краткое руководство о том, как присоединиться к собранию Teams
author: askaur
ms.author: askaur
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 773bca81694534346019e30e9d55190af6f51e74
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105106805"
---
## <a name="joining-the-meeting-chat"></a>Присоединение к чату собрания 

Включив взаимодействие с Teams, пользователь Служб коммуникации может присоединиться в качестве внешнего пользователя к вызову в Teams с помощью пакета SDK для вызовов. При присоединении к вызову пользователь будет также добавлен в чат собрания как его участник, где сможет обмениваться сообщениями с другими участниками вызова. Пользователь не будет видеть сообщения в чате, отправленные до его присоединения к вызову. Чтобы присоединиться к собранию и начать беседу, можно выполнить следующие действия.

## <a name="install-the-chat-packages"></a>Установка пакетов чата

Используйте команду `npm install`, чтобы установить необходимые пакеты SDK Служб коммуникации для JavaScript.

```console
npm install @azure/communication-common --save

npm install @azure/communication-identity --save

npm install @azure/communication-signaling --save

npm install @azure/communication-chat --save

npm install @azure/communication-calling --save
```

Параметр `--save` указывает библиотеку как зависимость в файле пакета **package.json**.

## <a name="add-the-teams-ui-controls"></a>Добавление элементов управления пользовательского интерфейса Teams

Замените код в файле index.html приведенным ниже фрагментом кода.
В текстовые поля в верхней части страницы будут вводиться контекст собрания группы и идентификатор потока собрания. Для присоединения к указанному собранию будет использоваться кнопка "Join Teams Meeting" (Присоединиться к собранию в Teams).
В нижней части страницы появится всплывающее окно чата. Его можно использовать для отправки сообщений в поток собрания. Кроме того, в нем в режиме реального времени будут отображаться все сообщения, отправленные в поток, участником которого является пользователь ACS.

```html
<!DOCTYPE html>
<html>
   <head>
      <title>Communication Client - Calling and Chat Sample</title>
      <style>
         body {box-sizing: border-box;}
         /* The popup chat - hidden by default */
         .chat-popup {
         display: none;
         position: fixed;
         bottom: 0;
         left: 15px;
         border: 3px solid #f1f1f1;
         z-index: 9;
         }
         .message-box {
         display: none;
         position: fixed;
         bottom: 0;
         left: 15px;
         border: 3px solid #FFFACD;
         z-index: 9;
         }
         .form-container {
         max-width: 300px;
         padding: 10px;
         background-color: white;
         }
         .form-container textarea {
         width: 90%;
         padding: 15px;
         margin: 5px 0 22px 0;
         border: none;
         background: #e1e1e1;
         resize: none;
         min-height: 50px;
         }
         .form-container .btn {
         background-color: #4CAF40;
         color: white;
         padding: 14px 18px;
         margin-bottom:10px;
         opacity: 0.6;
         border: none;
         cursor: pointer;
         width: 100%;
         }
         .container {
         border: 1px solid #dedede;
         background-color: #F1F1F1;
         border-radius: 3px;
         padding: 8px;
         margin: 8px 0;
         }
         .darker {
         border-color: #ccc;
         background-color: #ffdab9;
         margin-left: 25px;
         margin-right: 3px;
         }
         .lighter {
         margin-right: 20px;
         margin-left: 3px;
         }
         .container::after {
         content: "";
         clear: both;
         display: table;
         }
      </style>
   </head>
   <body>
      <h4>Azure Communication Services</h4>
      <h1>Calling and Chat Quickstart</h1>
          <input id="teams-link-input" type="text" placeholder="Teams meeting link"
        style="margin-bottom:1em; width: 300px;" />
          <input id="thread-id-input" type="text" placeholder="Chat thread id"
        style="margin-bottom:1em; width: 300px;" />
        <p>Call state <span style="font-weight: bold" id="call-state">-</span></p>
      <div>
        <button id="join-meeting-button" type="button">
            Join Teams Meeting
        </button>
        <button id="hang-up-button" type="button" disabled="true">
            Hang Up
        </button>
      </div>
      <div class="chat-popup" id="chat-box">
         <div id="messages-container"></div>
         <form class="form-container">
            <textarea placeholder="Type message.." name="msg" id="message-box" required></textarea>
            <button type="button" class="btn" id="send-message">Send</button>
         </form>
      </div>
      <script src="./bundle.js"></script>
   </body>
</html>
```

## <a name="enable-the-teams-ui-controls"></a>Включение элементов управления пользовательского интерфейса Teams

Замените содержимое файла client.js приведенным ниже фрагментом кода.

Во фрагменте кода замените 
- `SECRET CONNECTION STRING` строкой подключения Служб коммуникации, 
- `ENDPOINT URL` URL-адресом конечной точки Служб коммуникации.

```javascript
// run using
// npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
import { CallClient, CallAgent } from "@azure/communication-calling";
import { AzureCommunicationUserCredential } from "@azure/communication-common";
import { CommunicationIdentityClient } from "@azure/communication-identity";
import { ChatClient } from "@azure/communication-chat";

let call;
let callAgent;
let chatClient;
let chatThreadClient;

const meetingLinkInput = document.getElementById('teams-link-input');
const threadIdInput = document.getElementById('thread-id-input');
const callButton = document.getElementById("join-meeting-button");
const hangUpButton = document.getElementById("hang-up-button");
const callStateElement = document.getElementById('call-state');

const messagesContainer = document.getElementById("messages-container");
const chatBox = document.getElementById("chat-box");
const sendMessageButton = document.getElementById("send-message");
const messagebox = document.getElementById("message-box");

var userId = '';
var messages = '';

async function init() {
  const connectionString = "<SECRET CONNECTION STRING>";
  const endpointUrl = "<ENDPOINT URL>";

  const identityClient = new CommunicationIdentityClient(connectionString);

  let identityResponse = await identityClient.createUser();
  userId = identityResponse.communicationUserId;
  console.log(
    `\nCreated an identity with ID: ${identityResponse.communicationUserId}`
  );

  let tokenResponse = await identityClient.issueToken(identityResponse, [
    "voip",
    "chat",
  ]);
  const { token, expiresOn } = tokenResponse;
  console.log(
    `\nIssued an access token that expires at ${expiresOn}:`
  );
  console.log(token);

  const callClient = new CallClient();
  const tokenCredential = new AzureCommunicationUserCredential(token);
  callAgent = await callClient.createCallAgent(tokenCredential);
  callButton.disabled = false;

  chatClient = new ChatClient(
    endpointUrl,
    new AzureCommunicationUserCredential(token)
  );

  console.log('Azure Communication Chat client created!');
}

init();

callButton.addEventListener("click", async () => {
  // join with meeting link
  call = callAgent.join({meetingLink: meetingLinkInput.value}, {});
    
  call.on('callStateChanged', () => {
        callStateElement.innerText = call.state;
  })
  // toggle button and chat box states
  chatBox.style.display = "block";
  hangUpButton.disabled = false;
  callButton.disabled = true;

  messagesContainer.innerHTML = messages;
  
  console.log(call);

  // open notifications channel
  await chatClient.startRealtimeNotifications();

  // subscribe to new message notifications
  chatClient.on("chatMessageReceived", (e) => {
    console.log("Notification chatMessageReceived!");
    
    if (e.sender.communicationUserId != userId) {
       renderReceivedMessage(e.content);
    }
    else {
       renderSentMessage(e.content);
    }
  });
  chatThreadClient = await chatClient.getChatThreadClient(threadIdInput.value);
});

async function renderReceivedMessage(message) {
   messages += '<div class="container lighter">' + message + '</div>';
   messagesContainer.innerHTML = messages;
}

async function renderSentMessage(message) {
   messages += '<div class="container darker">' + message + '</div>';
   messagesContainer.innerHTML = messages;
}

hangUpButton.addEventListener("click", async () => 
  {
    // end the current call
    await call.hangUp();

    // toggle button states
    hangUpButton.disabled = true;
    callButton.disabled = false;
    callStateElement.innerText = '-';

    // toggle chat states
    chatBox.style.display = "none";
    messages = "";
  });

sendMessageButton.addEventListener("click", async () =>
  {
      let message = messagebox.value;

      let sendMessageRequest = { content: message };
      let sendMessageOptions = { senderDisplayName : 'Jack' };
      let sendChatMessageResult = await chatThreadClient.sendMessage(sendMessageRequest, sendMessageOptions);
      let messageId = sendChatMessageResult.id;

      messagebox.value = '';
      console.log(`Message sent!, message id:${messageId}`);
  });
```

## <a name="get-a-teams-meeting-chat-thread-for-a-communication-services-user"></a>Получение данных о беседе в чате для пользователя Служб коммуникации

Ссылку на собрание и чат в Teams можно получить с помощью API-интерфейсов Graph. Подробности см. в [документации по Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta). Пакет SDK вызовов Служб коммуникации принимает полную ссылку на собрание Teams. Эта ссылка возвращается как часть `onlineMeeting` ресурса, доступного в [свойстве `joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta). С помощью [API-интерфейсов Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta) можно также получить `threadId`. В ответе будет указан объект `chatInfo`, содержащий `threadID`. 

Вы также можете получить необходимую информацию о собрании и идентификатор потока по URL-адресу **Присоединиться к собранию** в самом приглашении на собрание в Teams.
Ссылка на собрание в Teams выглядит следующим образом: `https://teams.microsoft.com/l/meetup-join/meeting_chat_thread_id/1606337455313?context=some_context_here`. `threadId` — это расположение `meeting_chat_thread_id` в ссылке. Перед использованием убедитесь, что объект `meeting_chat_thread_id` не экранирован. Он должен иметь следующий формат: `19:meeting_ZWRhZDY4ZGUtYmRlNS00OWZaLTlkZTgtZWRiYjIxOWI2NTQ4@thread.v2`.


## <a name="run-the-code"></a>Выполнение кода

Пользователи Webpack могут использовать `webpack-dev-server` для сборки и запуска приложения. Выполните следующую команду, чтобы создать пакет узла приложения на локальном веб-сервере:

```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```

Откройте веб-браузер и перейдите по адресу http://localhost:8080/. Вы увидите следующее:

:::image type="content" source="../acs-join-teams-meeting-chat-quickstart.png" alt-text="Снимок экрана: готовое приложение JavaScript.":::

В текстовые поля вставьте ссылку на собрание в Teams и идентификатор потока. Чтобы присоединиться к собранию в Teams, нажмите кнопку *Join Teams Meeting* (Присоединиться к собранию в Teams). После того как пользователь ACS будет допущен на собрание, вы сможете общаться в приложении Служб коммуникации. Чтобы начать беседу, перейдите к полю в нижней части страницы.

> [!NOTE] 
> В настоящее время для сценариев взаимодействия с Teams поддерживается только отправка, получение и редактирование сообщений. Другие функции, такие как индикаторы ввода и возможность пользователей Служб коммуникации добавлять и удалять других пользователей в собрании в Teams, пока не поддерживаются.  
