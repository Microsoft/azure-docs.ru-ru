---
title: Краткое руководство. Добавление в приложение функции видеовызова (JavaScript)
titleSuffix: An Azure Communication Services quickstart
description: В этом кратком руководстве показано, как добавить возможности видеовызова в приложение с помощью Служб коммуникации Azure.
author: xumo-95
ms.author: mikben
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 82f4d9028fa94d4df0ff089fda213d64e13d56ec
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103487876"
---
# <a name="quickstart-add-11-video-calling-to-your-app-javascript"></a>Краткое руководство. Добавление в приложение функции персонального видеовызова (JavaScript)

## <a name="download-code"></a>Скачивание кода

Итоговый код для этого краткого руководства можно найти на сайте [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/add-1-on-1-video-calling).

## <a name="prerequisites"></a>Предварительные требования
- Получите учетную запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [Node.js](https://nodejs.org/en/): версии Active LTS и Maintenance LTS (8.11.1 и 10.14.1).
- Создайте активный ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](https://docs.microsoft.com/azure/communication-services/quickstarts/create-communication-resource?tabs=windows&pivots=platform-azp)
- Создайте маркер доступа пользователя для создания экземпляра клиента вызова. Узнайте, как [создать маркер доступа пользователя и обеспечить управление им](https://docs.microsoft.com/azure/communication-services/quickstarts/access-tokens?pivots=programming-language-csharp).

## <a name="setting-up"></a>Настройка
### <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;
Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.
```console
mkdir calling-quickstart && cd calling-quickstart
```
### <a name="install-the-package"></a>Установка пакета
Используйте команду `npm install`, чтобы установить клиентскую библиотеку Служб коммуникации для реализации вызовов на JavaScript.

В этом кратком руководстве используется клиентская библиотека для вызовов Служб коммуникации Azure `1.0.0.beta-6`. 

```console
npm install @azure/communication-common --save
npm install @azure/communication-calling --save
```
### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

В этом кратком руководстве для объединения ресурсов приложения используется webpack. Выполните следующую команду, чтобы установить пакеты npm `webpack`, `webpack-cli` и `webpack-dev-server`, а также указать их в качестве зависимостей разработки в `package.json`:

```console
npm install webpack@4.42.0 webpack-cli@3.3.11 webpack-dev-server@3.10.3 --save-dev
```
Создайте файл `index.html` в корневом каталоге проекта. Мы будем использовать этот файл для настройки базового макета, с помощью которого пользователь сможет осуществить персональный видеовызов.

Вот этот код:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Communication Client - 1:1 Video Calling Sample</title>
</head>

<body>
    <h4>Azure Communication Services</h4>
    <h1>1:1 Video Calling Quickstart</h1>
    <input 
    id="callee-id-input"
    type="text"
    placeholder="Who would you like to call?"
    style="margin-bottom:1em; width: 200px;"
    />

    <div>
    <button id="call-button" type="button" disabled="true">
        start call
    </button>
        &nbsp;
    <button id="hang-up-button" type="button" disabled="true">
        hang up
    </button>
        &nbsp;
    <button id="start-Video" type="button" disabled="true">
        start video
    </button>
        &nbsp;
    <button id="stop-Video" type="button" disabled="true">
        stop video
    </button>     
    </div>

    <div>Local Video</div>
    <div style="height:200px; width:300px; background-color:black; position:relative;">
      <div id="myVideo" style="background-color: black; position:absolute; top:50%; transform: translateY(-50%);">
      </div>     
    </div>
    <div>Remote Video</div>
    <div style="height:200px; width:300px; background-color:black; position:relative;"> 
      <div id="remoteVideo" style="background-color: black; position:absolute; top:50%; transform: translateY(-50%);">
      </div>
    </div>

    <script src="./bundle.js"></script>
</body>
</html>
```

Создайте файл в корневом каталоге проекта с именем `client.js`, чтобы включить логику приложения для этого краткого руководства. Добавьте следующий код, чтобы импортировать клиент вызова и получить ссылки на элементы модели DOM.

```JavaScript
import { CallClient, CallAgent, Renderer, LocalVideoStream } from "@azure/communication-calling";
import { AzureCommunicationTokenCredential } from '@azure/communication-common';

let call;
let callAgent;
const calleeInput = document.getElementById("callee-id-input");
const callButton = document.getElementById("call-button");
const hangUpButton = document.getElementById("hang-up-button");
const stopVideoButton = document.getElementById("stop-Video");
const startVideoButton = document.getElementById("start-Video");

let placeCallOptions;
let deviceManager;
let localVideoStream;
let rendererLocal;
let rendererRemote;
```
## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки Служб коммуникации Azure для вызовов:

| Имя      | Описание | 
| :---        |    :----   |
| CallClient  | CallClient — это основная точка входа в клиентскую библиотеку для вызовов.      |
| CallAgent  | CallAgent используется для инициирования вызовов и управления ими.        |
| DeviceManager | DeviceManager используется для управления устройствами мультимедиа.    |
| AzureCommunicationTokenCredential | Класс AzureCommunicationTokenCredential реализует интерфейс CommunicationTokenCredential, который используется для создания экземпляра CallAgent.        |

## <a name="authenticate-the-client-and-access-devicemanager"></a>Проверка подлинности клиента и получение доступа к DeviceManager

Вам необходимо заменить <USER_ACCESS_TOKEN> допустимым маркером доступа пользователя для вашего ресурса. Если у вас еще нет доступного маркера, см. документацию по маркеру доступа пользователя. С помощью CallClient инициализируйте экземпляр CallAgent с CommunicationUserCredential, и мы сможем осуществлять и принимать вызовы. Для получения доступа к DeviceManager необходимо сначала создать экземпляр callAgent. Затем можно применить метод `getDeviceManager` для экземпляра `CallClient`, чтобы получить `DeviceManager`.

Добавьте в `client.js` следующий код:

```JavaScript
async function init() {
    const callClient = new CallClient();
    const tokenCredential = new AzureCommunicationTokenCredential("<USER ACCESS TOKEN>");
    callAgent = await callClient.createCallAgent(tokenCredential, { displayName: 'optional ACS user name' });
    
    deviceManager = await callClient.getDeviceManager();
    callButton.disabled = false;
}
init();
```
## <a name="place-a-11-outgoing-video-call-to-a-user"></a>Осуществление персонального исходящего видеовызова к пользователю

Добавьте прослушиватель событий для инициации вызова при выборе `callButton`:

Сначала вам нужно перечислить локальные камеры с помощью API getCameraList deviceManager. В этом кратком руководстве используется первая камера в коллекции. Когда нужная камера будет выбрана, экземпляр LocalVideoStream будет создан и передан в videoOptions как элемент в массиве localVideoStream в метод вызова. Когда вызов будет осуществлен, автоматически начнется отправка видеопотока другому участнику. 

```JavaScript
callButton.addEventListener("click", async () => {
    const videoDevices = await deviceManager.getCameras();
    const videoDeviceInfo = videoDevices[0];
    localVideoStream = new LocalVideoStream(videoDeviceInfo);
    placeCallOptions = {videoOptions: {localVideoStreams:[localVideoStream]}};

    localVideoView();
    stopVideoButton.disabled = false;
    startVideoButton.disabled = true;

    const userToCall = calleeInput.value;
    call = callAgent.startCall(
        [{ communicationUserId: userToCall }],
        placeCallOptions
    );

    subscribeToRemoteParticipantInCall(call);

    hangUpButton.disabled = false;
    callButton.disabled = true;
});
```  
Чтобы отобразить `LocalVideoStream`, создайте новый экземпляр `Renderer`, а затем создать новый экземпляр RendererView с помощью асинхронного метода `createView`. Затем вы сможете присоединить `view.target` к любому элементу пользовательского интерфейса. 

```JavaScript
async function localVideoView() {
    rendererLocal = new Renderer(localVideoStream);
    const view = await rendererLocal.createView();
    document.getElementById("myVideo").appendChild(view.target);
}
```
Все удаленные участники доступны через коллекцию `remoteParticipants` в экземпляре вызова. Вам нужно подписаться на удаленных участников текущего вызова и прослушивать событие `remoteParticipantsUpdated`, чтобы подписаться на добавленных удаленных участников.

```JavaScript
function subscribeToRemoteParticipantInCall(callInstance) {
    callInstance.remoteParticipants.forEach( p => {
        subscribeToRemoteParticipant(p);
    })
    callInstance.on('remoteParticipantsUpdated', e => {
        e.added.forEach( p => {
            subscribeToRemoteParticipant(p);
        })
    });   
}
```
Вы можете подписаться на коллекцию `remoteParticipants` текущего вызова и проверить коллекции `videoStreams`, чтобы перечислить потоки каждого участника. Также необходимо подписаться на событие remoteParticipantsUpdated для управления добавленными удаленными участниками. 

```JavaScript
function subscribeToRemoteParticipant(remoteParticipant) {
    remoteParticipant.videoStreams.forEach(v => {
        handleVideoStream(v);
    });
    remoteParticipant.on('videoStreamsUpdated', e => {
        e.added.forEach(v => {
            handleVideoStream(v);
        })
    });
}
```
Чтобы отобразить `isAvailableChanged`, нужно подписаться на событие `remoteVideoStream`. Если свойство `isAvailable` изменит значение на `true`, значит этот удаленный участник отправляет поток. При каждом изменении состояния доступности удаленного потока вы можете удалить `Renderer` или определенное представление (`RendererView`) либо оставить все как есть, но это приведет к отображению пустого видеокадра.
```JavaScript
function handleVideoStream(remoteVideoStream) {
    remoteVideoStream.on('availabilityChanged', async () => {
        if (remoteVideoStream.isAvailable) {
            remoteVideoView(remoteVideoStream);
        } else {
            rendererRemote.dispose();
        }
    });
    if (remoteVideoStream.isAvailable) {
        remoteVideoView(remoteVideoStream);
    }
}
```
Чтобы отобразить `RemoteVideoStream`, создайте новый экземпляр `Renderer`, а затем создайте новый экземпляр `RendererView` с помощью асинхронного метода `createView`. Затем вы сможете присоединить `view.target` к любому элементу пользовательского интерфейса. 

```JavaScript
async function remoteVideoView(remoteVideoStream) {
    rendererRemote = new Renderer(remoteVideoStream);
    const view = await rendererRemote.createView();
    document.getElementById("remoteVideo").appendChild(view.target);
}
```
## <a name="receive-an-incoming-call"></a>Прием входящего вызова
Для обработки входящих вызовов необходимо прослушивать событие `incomingCall` `callAgent`. После приема входящего вызова необходимо перечислить локальные камеры и создать экземпляр `LocalVideoStream` для отправки видеопотока другому участнику. Кроме того, нужно подписаться на `remoteParticipants` для обработки удаленных видеопотоков. Вы можете принять или отклонить вызов через экземпляр `incomingCall`. 

Разместите реализацию в `init()` для обработки входящих вызовов. 

```JavaScript
callAgent.on('incomingCall', async e => {
    const videoDevices = await deviceManager.getCameras();
    const videoDeviceInfo = videoDevices[0];
    localVideoStream = new LocalVideoStream(videoDeviceInfo);
    localVideoView();

    stopVideoButton.disabled = false;
    callButton.disabled = true;
    hangUpButton.disabled = false;

    const addedCall = await e.incomingCall.accept({videoOptions: {localVideoStreams:[localVideoStream]}});
    call = addedCall;

    subscribeToRemoteParticipantInCall(addedCall);   
});
```
## <a name="end-the-current-call"></a>Завершение текущего вызова
Добавьте прослушиватель событий для завершения текущего вызова при нажатии `hangUpButton`:
```JavaScript
hangUpButton.addEventListener("click", async () => {
    // dispose of the renderers
    rendererLocal.dispose();
    rendererRemote.dispose();
    // end the current call
    await call.hangUp();
    // toggle button states
    hangUpButton.disabled = true;
    callButton.disabled = false;
    stopVideoButton.disabled = true;
});
```
## <a name="subscribe-to-call-updates"></a>Создание подписки на обновления вызова
Вам нужно подписаться на событие, когда удаленный участник завершает вызов для удаления отрисовщиков видео и переключения состояния выключателей. 

Разместите реализацию в init(), чтобы подписаться на событие `callsUpdated`. 
 ```JavaScript 
callAgent.on('callsUpdated', e => {
    e.removed.forEach(removedCall => {
        // dispose of video renders
        rendererLocal.dispose();
        rendererRemote.dispose();
        // toggle button states
        hangUpButton.disabled = true;
        callButton.disabled = false;
        stopVideoButton.disabled = true;
    })
})
```

## <a name="start-and-end-video-during-the-call"></a>Запуск и остановка видео во время вызова
Вы можете остановить видео во время текущего вызова, добавив прослушиватель событий к кнопке "Остановить видео", чтобы удалить отрисовщик `localVideoStream`. 
 ```JavaScript       
stopVideoButton.addEventListener("click", async () => {
    await call.stopVideo(localVideoStream);
    rendererLocal.dispose();
    startVideoButton.disabled = false;
    stopVideoButton.disabled = true;
});
```
Вы можете добавить прослушиватель событий к кнопке "Запустить видео", чтобы снова включить его после остановки во время текущего вызова. 
```JavaScript
startVideoButton.addEventListener("click", async () => {
    await call.startVideo(localVideoStream);
    localVideoView();
    stopVideoButton.disabled = false;
    startVideoButton.disabled = true;
});
```
## <a name="run-the-code"></a>Выполнение кода
Чтобы создать и запустить приложение, используйте `webpack-dev-server`. Выполните следующую команду, чтобы создать пакет узла приложения на локальном веб-сервере:

```console
npx webpack-dev-server --entry ./client.js --output bundle.js --debug --devtool inline-source-map
```
Откройте веб-браузер и перейдите по адресу http://localhost:8080/. Вы увидите следующее:

:::image type="content" source="./media/javascript/1-on-1-video-calling.png" alt-text="Страница персонального видеовызова":::

Вы можете выполнить исходящий персональный видеовызов, указав идентификатор пользователя в текстовом поле и нажав кнопку "Начать вызов". 

## <a name="sample-code"></a>Пример кода
Пример приложения можно загрузить в репозитории [GitHub](https://github.com/Azure-Samples/communication-services-javascript-quickstarts/tree/main/add-1-on-1-video-calling).

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](https://docs.microsoft.com/azure/communication-services/quickstarts/create-communication-resource?tabs=windows&pivots=platform-azp#clean-up-resources).

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения см. в следующих статьях:
- Ознакомьтесь с нашим [примером функции веб-вызовов](https://docs.microsoft.com/azure/communication-services/samples/web-calling-sample).
- Изучите [возможности клиентской библиотеки для вызовов](https://docs.microsoft.com/azure/communication-services/quickstarts/voice-video-calling/calling-client-samples?pivots=platform-web).
- Узнайте больше о [принципе работы функции вызовов](https://docs.microsoft.com/azure/communication-services/concepts/voice-video-calling/about-call-types).
