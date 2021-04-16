---
title: включить файл
description: включить файл
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: 322f54e4fa2e8096f68d5bbc216032a5b4e53c22
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105726697"
---
## <a name="prerequisites"></a>Предварительные требования
Перед началом работы нужно сделать следующее:

- Создайте учетную запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Установите [Node.js](https://nodejs.org/en/download/) (версии Active LTS и Maintenance LTS).
- Создайте ресурс Служб коммуникации Azure. Дополнительные сведения см. в статье [Краткое руководство по созданию ресурсов Служб коммуникации и управлению ими](../../create-communication-resource.md). Для работы с этим кратким руководством необходимо записать **конечную точку**.
- Создайте *трех* пользователей ACS и выдайте им маркер доступа пользователя (подробнее о [маркере доступа пользователя](../../access-tokens.md)). Обязательно задайте для области значение **chat** и **запишите строку маркера, а также строку userId**. Полная демонстрация создает поток с двумя начальными участниками, а затем добавляет третьего участника в поток.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-web-application"></a>Создание веб-приложения

Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.

```console
mkdir chat-quickstart && cd chat-quickstart
```

Воспользуйтесь командой `npm init -y`, чтобы создать файл **package.json** с параметрами по умолчанию.

```console
npm init -y
```

### <a name="install-the-packages"></a>Установка пакетов

Используйте команду `npm install`, чтобы установить указанные ниже пакеты SDK Служб коммуникации для JavaScript.

```console
npm install @azure/communication-common --save

npm install @azure/communication-identity --save

npm install @azure/communication-signaling --save

npm install @azure/communication-chat --save

```

Параметр `--save` указывает библиотеку как зависимость в файле пакета **package.json**.

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

В этом кратком руководстве для объединения ресурсов приложения используется webpack. Выполните следующую команду, чтобы установить пакеты npm webpack, webpack-cli и webpack-dev-server, а также указать их в качестве зависимостей разработки в файле **package.json**:

```console
npm install webpack webpack-cli webpack-dev-server --save-dev
```

Создайте файл `webpack.config.js` в корневом каталоге. Скопируйте в этот файл следующую конфигурацию.

```
module.exports = {
  entry: "./client.js",
  output: {
    filename: "bundle.js"
  },
  devtool: "inline-source-map",
  mode: "development"
}
```

Добавьте скрипт `start` в `package.json`, который будет использоваться для запуска приложения. Внутри раздела `scripts` файла `package.json` добавьте следующее:

```
"scripts": {
  "start": "webpack serve --config ./webpack.config.js"
}
```

Создайте файл **index.html** в корневом каталоге проекта. Мы будем использовать этот файл в качестве шаблона для добавления возможностей чата с помощью пакета SDK чата Служб коммуникации Azure для JavaScript.

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Communication Client - Chat Sample</title>
  </head>
  <body>
    <h4>Azure Communication Services</h4>
    <h1>Chat Quickstart</h1>
    <script src="./bundle.js"></script>
  </body>
</html>
```

Создайте файл в корневом каталоге проекта с именем **client.js**, чтобы включить логику приложения для этого краткого руководства.

### <a name="create-a-chat-client"></a>Создание клиента чата

Чтобы создать клиент чата в веб-приложении, вы будете использовать **конечную точку** Служб коммуникации и **маркер доступа**, который был создан в рамках подготовки к работе.

Маркеры доступа пользователей позволяют создавать клиентские приложения, которые напрямую проходят проверку подлинности в Службах коммуникации Azure. В этом кратком руководстве не описывается создание уровня служб для управления маркерами для приложения чата. Дополнительные сведения об архитектуре чата и [маркерах доступа пользователей ](../../access-tokens.md) см. в разделе, посвященном [основным понятиям чата](../../../concepts/chat/concepts.md).

В файле **client.js** укажите конечную точку и маркер доступа из приведенного ниже кода, чтобы добавить возможности чата с помощью пакета SDK чата Служб коммуникации Azure для JavaScript.

```JavaScript

import { ChatClient } from '@azure/communication-chat';
import { AzureCommunicationTokenCredential } from '@azure/communication-common';

// Your unique Azure Communication service endpoint
let endpointUrl = 'https://<RESOURCE_NAME>.communication.azure.com';
// The user access token generated as part of the pre-requisites
let userAccessToken = '<USER_ACCESS_TOKEN>';

let chatClient = new ChatClient(endpointUrl, new AzureCommunicationTokenCredential(userAccessToken));
console.log('Azure Communication Chat client created!');
```
- Замените **endpointUrl** конечной точкой ресурса Служб коммуникации (если вы еще этого не сделали, см. статью [Создание ресурса Служб коммуникации Azure](../../create-communication-resource.md)).
- Замените **userAccessToken** выданным маркером.


### <a name="run-the-code"></a>Выполнение кода

Выполните следующую команду, чтобы создать пакет узла приложения на локальном веб-сервере:
```console
npm run start
```
Откройте веб-браузер и перейдите по адресу http://localhost:8080/.
В консоли средств разработчика в браузере должно отобразиться следующее:

```console
Azure Communication Chat client created!
```

## <a name="object-model"></a>Объектная модель
Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK для чата Служб коммуникации для JavaScript.

| Имя                                   | Описание                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ChatClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения пользователей, отправки уведомлений о вводе и уведомления о прочтении и подписки на чат. |


## <a name="start-a-chat-thread"></a>Запуск потока чата

Для создания потока чата используйте метод `createThread`.

`createThreadRequest` используется для описания запроса потока:

- Используйте `topic` для указания темы этого чата. Тему можно обновить после создания беседы чата с помощью функции `UpdateThread`.
- Используйте `participants`, чтобы получить список участников, добавляемых в поток чата.

При разрешении метода `createChatThread` возвращается `CreateChatThreadResult`. Эта модель содержит свойство `chatThread`, в котором можно получить доступ к `id` только что созданного потока. Затем вы можете использовать `id`, чтобы получить экземпляр `ChatThreadClient`. Затем можно использовать `ChatThreadClient` для выполнения операции в потоке, например при отправке сообщений или выводе списка участников.

```JavaScript
async function createChatThread() {
  const createChatThreadRequest = {
    topic: "Hello, World!"
  };
  const createChatThreadOptions = {
    participants: [
      {
        id: '<USER_ID>',
        displayName: '<USER_DISPLAY_NAME>'
      }
    ]
  };
  const createChatTtreadResult = await chatClient.createChatThread(
    createChatThreadRequest,
    createChatThreadOptions
  );
  const threadId = createChatThreadResult.chatThread.id;
  return threadId;
}

createChatThread().then(async threadId => {
  console.log(`Thread created:${threadId}`);
  // PLACEHOLDERS
  // <CREATE CHAT THREAD CLIENT>
  // <RECEIVE A CHAT MESSAGE FROM A CHAT THREAD>
  // <SEND MESSAGE TO A CHAT THREAD>
  // <LIST MESSAGES IN A CHAT THREAD>
  // <ADD NEW PARTICIPANT TO THREAD>
  // <LIST PARTICIPANTS IN A THREAD>
  // <REMOVE PARTICIPANT FROM THREAD>
  });
```

При обновлении вкладки браузера в консоли должны отобразиться следующие сведения:
```console
Thread created: <thread_id>
```

## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата

Метод `getChatThreadClient` возвращает `chatThreadClient` для потока, который уже существует. Его можно использовать для выполнения операций в созданном потоке: добавления участников, отправки сообщения и т. д. Идентификатор thread_id — это уникальный идентификатор имеющегося потока чата.

```JavaScript
let chatThreadClient = chatClient.getChatThreadClient(threadId);
console.log(`Chat Thread client for threadId:${threadId}`);

```
Добавьте этот код вместо комментария `<CREATE CHAT THREAD CLIENT>` в файл **client.js**, обновите вкладку браузера и проверьте консоль, вы увидите:
```console
Chat Thread client for threadId: <threadId>
```

## <a name="list-all-chat-threads"></a>Перечисление всех потоков чата

Метод `listChatThreads` возвращает объект `PagedAsyncIterableIterator` типа `ChatThreadItem`. Его можно использовать для перечисления всех потоков чата.
Итератор `[ChatThreadItem]` является ответом, возвращенным из списка потоков.

```JavaScript
const threads = chatClient.listChatThreads();
for await (const thread of threads) {
   // your code here
}
```

## <a name="send-a-message-to-a-chat-thread"></a>Отправка сообщения в поток чата

Используйте метод `sendMessage` для отправки сообщения в поток, определяемый с помощью threadId.

`sendMessageRequest` используется для описания запроса сообщения:

- Используйте `content`, чтобы указать содержимое сообщения в чате.

`sendMessageOptions` используется для описания необязательных параметров операции:

- Используйте `senderDisplayName`, чтобы указать отображаемое имя отправителя.
- Используйте `type` для определения типа сообщения, например text или html.

`SendChatMessageResult` — ответ, возвращаемый при отправке сообщения. Содержит идентификатор, который является уникальным идентификатором сообщения.

```JavaScript
const sendMessageRequest =
{
  content: 'Hello Geeta! Can you share the deck for the conference?'
};
let sendMessageOptions =
{
  senderDisplayName : 'Jack',
  type: 'text'
};
const sendChatMessageResult = await chatThreadClient.sendMessage(sendMessageRequest, sendMessageOptions);
const messageId = sendChatMessageResult.id;
```

Добавьте этот код вместо комментария `<SEND MESSAGE TO A CHAT THREAD>` в файл **client.js**, обновите вкладку браузера и проверьте консоль.
```console
Message sent!, message id:<number>
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Получение сообщений из потока чата

С помощью оповещений в реальном времени можно подписываться на прослушивание новых входящих сообщений и обновлять текущие сообщения в памяти соответствующим образом. Службы коммуникации Azure поддерживают [список событий, на которые можно подписываться](../../../concepts/chat/concepts.md#real-time-notifications).

```JavaScript
// open notifications channel
await chatClient.startRealtimeNotifications();
// subscribe to new notification
chatClient.on("chatMessageReceived", (e) => {
  console.log("Notification chatMessageReceived!");
  // your code here
});

```
Добавьте этот код вместо комментария `<RECEIVE A CHAT MESSAGE FROM A CHAT THREAD>` в файл **client.js**.
Обновите вкладку браузера, и в консоли отобразится сообщение `Notification chatMessageReceived`.

Кроме того, можно получать сообщения чата, выполняя опрос метода `listMessages` через определенные интервалы.

```JavaScript

const messages = chatThreadClient.listMessages();
for await (const message of messages) {
   // your code here
}

```
Добавьте этот код вместо комментария `<LIST MESSAGES IN A CHAT THREAD>` в файл **client.js**.
Обновите вкладку. В консоли вы должны увидеть список сообщений, отправленных в этом потоке чата.

`listMessages` возвращает различные типы сообщений, которые могут быть идентифицированы с помощью `chatMessage.type`. 

Дополнительные сведения см. в разделе о [типах сообщений](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

После создания потока чата можно добавлять и удалять пользователей. Добавляя пользователей, вы предоставляете им доступ для отправки сообщений в поток чата, а также добавления или удаления других участников.

Перед вызовом метода `addParticipants` убедитесь, что вы получили новый маркер доступа и удостоверение для этого пользователя. Пользователю потребуется маркер доступа для инициализации своего клиента чата.

`addParticipantsRequest` описывает объект запроса, а в `participants` приведены участники, добавляемые в поток чата.
- `id` (требуется) — это идентификатор обмена данными, который должен быть добавлен в поток чата.
- `displayName` (необязательно) — это отображаемое имя для участника потока.
- `shareHistoryTime` (необязательно) — это время, в течение которого участнику предоставляется доступ к журналу чата. Чтобы предоставить общий доступ к журналу с момента запуска потока чата, установите для этого свойства любую дату, равную или меньше времени создания потока. Чтобы в журнале отображались только записи с момента добавления участника, задайте для свойства текущую дату. Чтобы предоставить общий доступ к части журнала, задайте для него дату по своему усмотрению.

```JavaScript

const addParticipantsRequest =
{
  participants: [
    {
      id: { communicationUserId: '<NEW_PARTICIPANT_USER_ID>' },
      displayName: 'Jane'
    }
  ]
};

await chatThreadClient.addParticipants(addParticipantsRequest);

```
Замените **NEW_PARTICIPANT_USER_ID** [новым идентификатором пользователя](../../access-tokens.md) и добавьте этот код вместо комментария `<ADD NEW PARTICIPANT TO THREAD>` в файл **client.js**.

## <a name="list-users-in-a-chat-thread"></a>Вывод списка пользователей в потоке чата
```JavaScript
const participants = chatThreadClient.listParticipants();
for await (const participant of participants) {
   // your code here
}
```
Добавьте этот код вместо комментария `<LIST PARTICIPANTS IN A THREAD>` в файл **client.js**, обновите вкладку браузера и проверьте консоль. Вы увидите сведения о пользователях чата:

## <a name="remove-user-from-a-chat-thread"></a>Удаление пользователя из потока чата

Аналогично тому, как выполняется добавление участников, вы можете удалять их из потока чата. Чтобы удалить участника, отследите идентификаторы добавленных участников.

Используйте метод `removeParticipant`, где `participant` — это пользователь, удаляемый из потока.

```JavaScript

await chatThreadClient.removeParticipant({ communicationUserId: <PARTICIPANT_ID> });
await listParticipants();
```
Замените значение **PARTICIPANT_ID** идентификатором пользователя, который использовался на предыдущем шаге (<NEW_PARTICIPANT_USER_ID>).
Добавьте этот код вместо комментария `<REMOVE PARTICIPANT FROM THREAD>` в файл **client.js**.
