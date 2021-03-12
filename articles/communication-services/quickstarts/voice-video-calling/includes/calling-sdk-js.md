---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 9/1/2020
ms.author: mikben
ms.openlocfilehash: e013765579fd560952172166b24f898b354c1d17
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103021181"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- Развернутый ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- `User Access Token` для включения клиента вызова. Дополнительные сведения о том, [как получить `User Access Token`](../../access-tokens.md)
- (Необязательно) Выполните инструкции из краткого руководства [Добавление функции голосового вызова в приложение](../getting-started-with-calling.md).

## <a name="setting-up"></a>Настройка

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

> [!NOTE]
> В этом документе используется клиентская библиотека вызовов версии 1.0.0-beta.6.

С помощью команды `npm install` вы можете установить клиентские библиотеки JavaScript Служб коммуникации Azure для вызовов и типичных операций.
В этом документе используются типы, определенные в библиотеке вызовов версии 1.0.0-beta.5.

```console
npm install @azure/communication-common --save
npm install @azure/communication-calling --save

```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки Служб коммуникации Azure для вызовов:

| Имя                             | Описание                                                                                                                                 |
| ---------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------- |
| CallClient                       | CallClient — это основная точка входа в клиентскую библиотеку для вызовов.                                                                       |
| CallAgent                        | CallAgent используется для инициирования вызовов и управления ими.                                                                                            |
| DeviceManager                    | DeviceManager используется для управления устройствами мультимедиа.                                                                                           |
| AzureCommunicationTokenCredential | Класс AzureCommunicationTokenCredential реализует интерфейс CommunicationTokenCredential, который используется для создания экземпляра CallAgent. |


## <a name="initialize-the-callclient-create-callagent-and-access-devicemanager"></a>Инициализация CallClient, создание CallAgent и использование DeviceManager

Создайте новый экземпляр `CallClient`. Для настройки доступны специальные параметры, например экземпляр Logger.
После создания экземпляра `CallClient` вы можете создать экземпляр `CallAgent`, вызвав метод `createCallAgent` для экземпляра `CallClient`. Это действие асинхронно возвращает объект экземпляра `CallAgent`.
Метод `createCallAgent` принимает в качестве аргумента `CommunicationTokenCredential`, который принимает [маркер доступа пользователя](../../access-tokens.md).
Для доступа к `DeviceManager` необходимо сначала создать экземпляр callAgent. Затем можно применить метод `getDeviceManager` для экземпляра `CallClient`, чтобы получить DeviceManager.

```js
const userToken = '<user token>';
callClient = new CallClient(options);
const tokenCredential = new AzureCommunicationTokenCredential(userToken);
const callAgent = await callClient.createCallAgent(tokenCredential, {displayName: 'optional ACS user name'});
const deviceManager = await callClient.getDeviceManager()
```

## <a name="place-an-outgoing-call"></a>Осуществление исходящего вызова

Чтобы создать и запустить вызов, необходимо использовать один из API-интерфейсов на Каллажент и предоставить пользователя, созданного с помощью клиентской библиотеки удостоверений служб связи.

Создание и запуск вызова активируются синхронно. Экземпляр Call позволяет подписываться на события вызова.

## <a name="place-a-call"></a>Осуществление вызовов

### <a name="place-a-11-call-to-a-user-or-pstn"></a>Осуществление персонального вызова к пользователю или ТСОП
Чтобы поместить вызов другого пользователя служб связи, вызовите `startCall` метод для `callAgent` и передайте вызываемый коммуникатионусеридентифиер, [созданный с помощью библиотеки удостоверений служб связи](https://docs.microsoft.com/azure/communication-services/quickstarts/access-tokens).

```js
const userCallee = { communicationUserId: '<ACS_USER_ID>' }
const oneToOneCall = callAgent.startCall([userCallee]);
```

Чтобы осуществить вызов к ТСОП, вызовите метод `startCall` для `callAgent` и передайте PhoneNumberIdentifier вызываемого объекта.
Для осуществления вызовов к ТСОП необходимо настроить ресурс Служб коммуникации.
При вызове номера ТСОП необходимо указать альтернативный идентификатор вызывающего. Альтернативный идентификатор вызывающего обозначает номер телефона (по стандарту E.164), который определяет вызывающую сторону в вызове ТСОП. Например, переданный в вызове ТСОП альтернативный идентификатор абонента будет отображаться вызываемой стороне.ю когда вызов будет входящим.

> [!WARNING]
> Вызовы ТСОП сейчас доступны в закрытой предварительной версии. Чтобы использовать эту возможность, [зарегистрируйтесь в программе ранних последователей](https://aka.ms/ACS-EarlyAdopter).
```js
const pstnCalee = { phoneNumber: '<ACS_USER_ID>' }
const alternateCallerId = {alternateCallerId: '<Alternate caller Id>'};
const oneToOneCall = callAgent.startCall([pstnCallee], {alternateCallerId});
```

### <a name="place-a-1n-call-with-users-and-pstn"></a>Осуществление группового вызова к пользователям и ТСОП
```js
const userCallee = { communicationUserId: <ACS_USER_ID> }
const pstnCallee = { phoneNumber: <PHONE_NUMBER>};
const alternateCallerId = {alternateCallerId: '<Alternate caller Id>'};
const groupCall = callAgent.startCall([userCallee, pstnCallee], {alternateCallerId});

```

### <a name="place-a-11-call-with-video-camera"></a>Осуществление персонального вызова с использованием видеокамеры
> [!WARNING]
> Сейчас допускается не более одного исходящего локального видеопотока в один момент времени.
Чтобы осуществить видеовызов, необходимо перечислить локальные камеры с помощью API deviceManager `getCameras()`.
Выбрав нужную камеру, используйте ее для создания экземпляра `LocalVideoStream`, который затем передайте в составе `videoOptions` в качестве элемента массива `localVideoStream` в метод `startCall`.
Когда вызов будет осуществлен, автоматически начнется отправка видеопотока с выбранной камеры другим участникам вызова. Это же относится к параметрам видео в Call.Accept() и CallAgent.join().
```js
const deviceManager = await callClient.getDeviceManager();
const cameras = await deviceManager.getCameras();
videoDeviceInfo = cameras[0];
localVideoStream = new LocalVideoStream(videoDeviceInfo);
const placeCallOptions = {videoOptions: {localVideoStreams:[localVideoStream]}};
const call = callAgent.startCall(['acsUserId'], placeCallOptions);

```

### <a name="join-a-group-call"></a>Присоединение к групповому вызову
Чтобы начать новый групповой вызов или присоединиться к текущему групповому вызову, примените метод join и передайте ему объект со свойством `groupId`. В качестве значения должен быть указан идентификатор GUID.
```js

const context = { groupId: <GUID>}
const call = callAgent.join(context);

```
### <a name="join-a-teams-meeting"></a>Присоединение к конференции Teams
Чтобы присоединиться к конференции Teams, выполните метод join и передайте ему ссылку на собрание или координаты собрания.
```js
// Join using meeting link
const locator = { meetingLink: <meeting link>}
const call = callAgent.join(locator);

// Join using meeting coordinates
const locator = {
    threadId: <thread id>,
    organizerId: <organizer id>,
    tenantId: <tenant id>,
    messageId: <message id>
}
const call = callAgent.join(locator);
```

## <a name="receiving-an-incoming-call"></a>Прием входящего вызова

Экземпляр `CallAgent` создает событие `incomingCall`, когда зарегистрированный в системе идентификатор принимает входящий вызов. Чтобы прослушать это событие, подпишитесь на него следующим образом:

```js
const incomingCallHander = async (args: { incomingCall: IncomingCall }) => {
    //Get information about caller
    var callerInfo = incomingCall.callerInfo
    
    //accept the call
    var call = await incomingCall.accept();

    //reject the call
    incomingCall.reject();
};
callAgentInstance.on('incomingCall', incomingCallHander);
```

Событие `incomingCall` предоставит экземпляр `IncomingCall`, который позволяет принять или отклонить вызов.


## <a name="call-management"></a>Управление вызовами

Во время вызова вы можете получать доступ ко свойствам вызова и выполнять различные операции для управления параметрами, связанными с видео и аудио.

### <a name="call-properties"></a>Свойства вызова
* Получите уникальный идентификатор (в строковом формате) этого вызова.
```js

const callId: string = call.id;

```

* Чтобы получить данные о других участниках вызова, просмотрите коллекцию `remoteParticipant` в экземпляре `call`. Массив содержит объекты списка `RemoteParticipant`
```js
const remoteParticipants = call.remoteParticipants;
```

* Для входящих вызовов это идентификатор вызывающего объекта. Идентификатор имеет один из типов `CommunicationIdentifier`
```js

const callerIdentity = call.callerInfo.identifier;

* Get the state of the Call.
```js

const callState = call.state;

```
Возвращает строку, которая представляет текущее состояние вызова:
* None — исходное состояние любого вызова;
* Incoming — указывает, что вызов является входящим и должен быть принят или отклонен;
* Connecting — исходное переходное состояние после осуществления или приема вызова;
* Ringing — для исходящего вызова означает, что на стороне удаленных участников звучит сигнал вызова, то есть этот вызов для них является входящим (состояние Incoming);
* EarlyMedia — обозначает состояние, при котором перед вызовом воспроизводится объявление;
* Connected — вызов осуществлен;
* LocalHold — вызов переведен локальным участником в режим ожидания, и потоки мультимедиа не передаются между локальной конечной точкой и удаленными участниками;
* RemoteHold — вызов переведен удаленным участником в режим ожидания, и потоки мультимедиа не передаются между локальной конечной точкой и удаленными участниками;
* Disconnecting — переходное состояние перед тем, как вызов перейдет в состояние Disconnected;
* Disconnected — завершающее состояние вызова.
  * При потере сетевого подключения вызов через 2 минуты автоматически переходит в состояние Disconnected.

* Чтобы увидеть причину завершения конкретного вызова, проверьте свойство `callEndReason`.
```js

const callEndReason = call.callEndReason;
// callEndReason.code (number) code associated with the reason
// callEndReason.subCode (number) subCode associated with the reason
```

* Чтобы выяснить, является ли текущий вызов входящим или исходящим, проверьте свойство `direction`, которое возвращает значение `CallDirection`.
```js
const isIncoming = call.direction == 'Incoming';
const isOutgoing = call.direction == 'Outgoing';
```

*  Чтобы проверить, отключен ли выбранный микрофон, проверьте свойство `muted`, которое возвращает значение `Boolean`.
```js

const muted = call.isMicrophoneMuted;

```

* Чтобы узнать, отправляется ли поток общего доступа к экрану из конкретной конечной точки, проверьте свойство `isScreenSharingOn`, которое возвращает значение `Boolean`.
```js

const isScreenSharingOn = call.isScreenSharingOn;

```

* Чтобы проверить активные видеопотоки, проверьте коллекцию `localVideoStreams` с объектами `LocalVideoStream`.
```js

const localVideoStreams = call.localVideoStreams;

```

### <a name="call-ended-event"></a>Событие завершения вызова

Экземпляр `Call` при завершении вызова создает событие `callEnded`. Чтобы прослушать это событие, подпишитесь на него следующим образом:

```js
const callEndHander = async (args: { callEndReason: CallEndReason }) => {
    console.log(args.callEndReason)
};

call.on('callEnded', callEndHander);
```

### <a name="mute-and-unmute"></a>Отключение и включение звука

Чтобы отключить или включить звук для локальной конечной точки, можно использовать асинхронные вызовы API `mute` и `unmute`:

```js

//mute local device
await call.mute();

//unmute local device
await call.unmute();

```

### <a name="start-and-stop-sending-local-video"></a>Запуск и остановка отправки локального видео


Чтобы запустить видео, необходимо перечислить камеры с помощью метода `getCameras` для объекта `deviceManager`. Затем создайте новый экземпляр `LocalVideoStream`, передав нужную камеру в качестве аргумента в метод `startVideo`:


```js
const localVideoStream = new LocalVideoStream(videoDeviceInfo);
await call.startVideo(localVideoStream);

```

После успешного запуска отправки видео экземпляр `LocalVideoStream` будет добавлен в коллекцию `localVideoStreams` в экземпляре вызова.

```js

call.localVideoStreams[0] === localVideoStream;

```

Чтобы остановить отправку локального видео, передайте экземпляр `localVideoStream`, доступный в коллекции `localVideoStreams`:

```js

await call.stopVideo(localVideoStream);

```

Вы можете переключиться на другое устройство камеры, не прекращая передачу видео, вызвав `switchSource` для экземпляра `localVideoStream`:

```js
const cameras = await callClient.getDeviceManager().getCameras();
localVideoStream.switchSource(cameras[1]);

```

## <a name="remote-participants-management"></a>Управление удаленными участниками

Все удаленные участники представлены типом `RemoteParticipant` и доступны через коллекцию `remoteParticipants` в экземпляре вызова.

### <a name="list-participants-in-a-call"></a>Вывод списка участников в вызове
Коллекция `remoteParticipants` возвращает список удаленных участников в указанном вызове:

```js

call.remoteParticipants; // [remoteParticipant, remoteParticipant....]

```

### <a name="remote-participant-properties"></a>Свойства удаленного участника
С каждым удаленным участником связан набор свойств и коллекций
#### <a name="communicationidentifier"></a>CommunicationIdentifier
Получите идентификатор для этого удаленного участника.
Идентификатор имеет один из типов CommunicationIdentifier:
```js
const identifier = remoteParticipant.identifier;
```
Это может быть один из типов CommunicationIdentifier:
  * { communicationUserId: '<ACS_USER_ID'> } — объект, представляющий пользователя ACS;
  * { phoneNumber: '<E.164>' } — объект, представляющий номер телефона в формате E.164;
  * { microsoftTeamsUserId: '<TEAMS_USER_ID>', isAnonymous?: boolean; cloud?: "public" | "dod" | "gcch" } — объект, представляющий пользователя Teams.

#### <a name="state"></a>Состояние
Получите состояние удаленного участника.
```js

const state = remoteParticipant.state;
```
Состояние может иметь одно из следующих значений:
* Idle — исходное состояние;
* Connected — переходное состояние во время подключения участника к вызову;
* Ringing — для участника звучит сигнал вызова;
* Connected — участник подключен к вызову;
* Hold — участник находится в режиме ожидания;
* EarlyMedia — перед подключением к вызову для участника воспроизводится объявление;
* Disconnected — окончательное состояние, которое обозначает завершение вызова для участника.
  * Если у удаленного участника происходит потеря сетевого подключения, он через 2 минуты автоматически переходит в состояние Disconnected.

#### <a name="call-end-reason"></a>Причина окончания вызова
Чтобы узнать, почему участник покинул вызов, проверьте свойство `callEndReason`:
```js
const callEndReason = remoteParticipant.callEndReason;
// callEndReason.code (number) code associated with the reason
// callEndReason.subCode (number) subCode associated with the reason
```
#### <a name="is-muted"></a>Звук отключен
Чтобы проверить, отключен ли звук для удаленного участника, проверьте свойство `isMuted`, которое возвращает значение `Boolean`.
```js
const isMuted = remoteParticipant.isMuted;
```
#### <a name="is-speaking"></a>Участник говорит
Чтобы проверить, говорит ли в настоящий момент удаленный участник, проверьте свойство `isSpeaking`, которое возвращает значение `Boolean`.
```js
const isSpeaking = remoteParticipant.isSpeaking;
```

#### <a name="video-streams"></a>Video Streams (Потоки видео)
Чтобы проверить все потоки видео, отправляемые определенным участником в вызове, проверьте коллекцию `videoStreams` с объектами `RemoteVideoStream`.
```js

const videoStreams = remoteParticipant.videoStreams; // [RemoteVideoStream, ...]

```


### <a name="add-a-participant-to-a-call"></a>Добавление участника в вызов

Чтобы добавить в вызов участника (пользователя или номер телефона), можно вызвать `addParticipant`.
Укажите в этом вызове один из типов Identifier.
Этот вызов в синхронном режиме возвращает экземпляр удаленного участника.

```js
const userIdentifier = { communicationUserId: <ACS_USER_ID> };
const pstnIdentifier = { phoneNumber: <PHONE_NUMBER>}
const remoteParticipant = call.addParticipant(userIdentifier);
const remoteParticipant = call.addParticipant(pstnIdentifier, {alternateCallerId: '<Alternate Caller ID>'});
```

### <a name="remove-participant-from-a-call"></a>Удаление участника из вызова

Чтобы удалить участника (пользователя или номер телефона) из вызова, можно вызвать `removeParticipant`.
Нужно передать один из типов Identifier, который будет асинхронно разрешаться после удаления участника из вызова.
Этот же участник будет удален из коллекции `remoteParticipants`.

```js
const userIdentifier = { communicationUserId: <ACS_USER_ID> };
const pstnIdentifier = { phoneNumber: <PHONE_NUMBER>}
await call.removeParticipant(userIdentifier);
await call.removeParticipant(pstnIdentifier);
```

## <a name="render-remote-participant-video-streams"></a>Отрисовка видеопотоков удаленных участников

Чтобы получить список видеопотоков и потоков демонстрации экрана, получаемых от удаленных участников, проверьте коллекции `videoStreams`:

```js
const remoteVideoStream: RemoteVideoStream = call.remoteParticipants[0].videoStreams[0];
const streamType: MediaStreamType = remoteVideoStream.mediaStreamType;
```

Чтобы отрисовывать `RemoteVideoStream`, нужно оформить подписку на событие `isAvailableChanged`.
Если свойство `isAvailable` изменит значение на `true`, значит этот удаленный участник отправляет поток.
Когда это произойдет, создайте новый экземпляр `Renderer`, а затем новый экземпляр `RendererView` с помощью асинхронного метода `createView`.  Затем вы сможете присоединить `view.target` к любому элементу пользовательского интерфейса.
При каждом изменении состояния доступности удаленного потока вы можете удалить отрисовщик или определенное представление `RendererView` либо оставить все как есть, но это приведет к отрисовке пустого видеокадра.

```js
function subscribeToRemoteVideoStream(remoteVideoStream: RemoteVideoStream) {
    let renderer: Renderer = new Renderer(remoteVideoStream);
    const displayVideo = () => {
        const view = await renderer.createView();
        htmlElement.appendChild(view.target);
    }
    remoteVideoStream.on('availabilityChanged', async () => {
        if (remoteVideoStream.isAvailable) {
            displayVideo();
        } else {
            renderer.dispose();
        }
    });
    if (remoteVideoStream.isAvailable) {
        displayVideo();
    }
}
```

### <a name="remote-video-stream-properties"></a>Свойства удаленного видеопотока
Удаленные потоки видео имеют следующие свойства:

* `Id` — идентификатор удаленного видеопотока;
```js
const id: number = remoteVideoStream.id;
```

* `StreamSize` — размер (ширина и высота) удаленного потока видео;
```js
const size: {width: number; height: number} = remoteVideoStream.size;
```

* `MediaStreamType` — принимает значения Video или ScreenSharing;
```js
const type: MediaStreamType = remoteVideoStream.mediaStreamType;
```
* `isAvailable` — указывает на то, что конечная точка этого удаленного участника активно отправляет поток данных.
```js
const type: boolean = remoteVideoStream.isAvailable;
```

### <a name="renderer-methods-and-properties"></a>Методы и свойства отрисовщика

* Создайте экземпляр `RendererView`, который позже можно будет подключить к пользовательскому интерфейсу приложения для отрисовки удаленного видеопотока.
```js
renderer.createView()
```

* Удаляет отрисовщик и все связанные с ним экземпляры `RendererView`.
```js
renderer.dispose()
```


### <a name="rendererview-methods-and-properties"></a>Методы и свойства RendererView
При создании `RendererView` вы можете указать свойства `scalingMode` и `isMirrored`.
Режим масштабирования может принимать значения Stretch, Crop или Fit. Если указан параметр `isMirrored`, поток будет отрисовываться в зеркальном отображении (по вертикали).

```js
const rendererView: RendererView = renderer.createView({ scalingMode, isMirrored });
```
Каждый экземпляр `RendererView` имеет свойство `target`, которое представляет область отрисовки. Он должен быть присоединен к пользовательскому интерфейсу приложения:
```js
document.body.appendChild(rendererView.target);
```

Вы можете позже изменить режим масштабирования, вызвав метод `updateScalingMode`.
```js
view.updateScalingMode('Crop')
```

## <a name="device-management"></a>Управление устройствами

`DeviceManager` позволяет получить список локальных устройств, которые можно использовать в вызове для передачи аудио- и видеопотоков. Вы также можете запрашивать у пользователя разрешение на доступ к его микрофону и камере через собственный API браузера.

Доступ к `deviceManager` осуществляется путем вызова метода `callClient.getDeviceManager()`.
> [!WARNING]
> Сейчас для получения доступа к DeviceManager необходимо сначала создать экземпляр объекта `callAgent`.

```js

const deviceManager = await callClient.getDeviceManager();

```

### <a name="enumerate-local-devices"></a>Перечисление локальных устройств

Для доступа к локальным устройствам можно использовать методы перечисления из Диспетчера устройств. Действие перечисления выполняется асинхронно.

```js

//  Get a list of available video devices for use.
const localCameras = await deviceManager.getCameras(); // [VideoDeviceInfo, VideoDeviceInfo...]

// Get a list of available microphone devices for use.
const localMicrophones = await deviceManager.getMicrophones(); // [AudioDeviceInfo, AudioDeviceInfo...]

// Get a list of available speaker devices for use.
const localSpeakers = await deviceManager.getSpeakers(); // [AudioDeviceInfo, AudioDeviceInfo...]

```

### <a name="set-default-microphonespeaker"></a>Назначение микрофона или динамика по умолчанию

Диспетчер устройств позволяет задать устройство по умолчанию, которое будет использоваться при осуществлении вызова.
Если значения по умолчанию для клиента не заданы, Службы коммуникации будут использовать значения по умолчанию для ОС.

```js

// Get the microphone device that is being used.
const defaultMicrophone = deviceManager.selectedMicrophone;

// Set the microphone device to use.
await deviceManager.selectMicrophone(AudioDeviceInfo);

// Get the speaker device that is being used.
const defaultSpeaker = deviceManager.selectedSpeaker;

// Set the speaker device to use.
await deviceManager.selectSpeaker(AudioDeviceInfo);

```

### <a name="local-camera-preview"></a>Предварительный просмотр изображения с локальной камеры

С помощью `DeviceManager` и `Renderer` вы можете реализовать отрисовку потоков из локальной камеры. Этот поток не отправляется другим участникам, а используется как локальный канал предварительного просмотра. Это действие выполняется асинхронно.

```js
const cameras = await deviceManager.getCameras();
const localVideoDevice = cameras[0];
const localCameraStream = new LocalVideoStream(localVideoDevice);
const renderer = new Renderer(localCameraStream);
const view = await renderer.createView();
document.body.appendChild(view.target);

```

### <a name="request-permission-to-cameramicrophone"></a>Запрос разрешений для камеры или микрофона

Запросите у пользователя разрешение на доступ к камере и микрофону следующим образом:

```js
const result = await deviceManager.askDevicePermission({audio: true, video: true});
```
Это значение асинхронно разрешается в объект, который обозначает предоставление разрешений `audio` и `video`:
```js
console.log(result.audio);
console.log(result.video);
```


## <a name="call-recording-management"></a>Управление записью вызовов

Запись вызовов является функцией расширения базового API `Call`. Сначала вам нужно получить объект API для функции записи:

```js
const callRecordingApi = call.api(Features.Recording);
```

Затем вы можете убедиться, что вызов записывается, проверив свойство `isRecordingActive` объекта `callRecordingApi`, которое возвращает значение `Boolean`.

```js
const isResordingActive = callRecordingApi.isRecordingActive;
```

Вы также можете подписаться на запись изменений:

```js
const isRecordingActiveChangedHandler = () => {
  console.log(callRecordingApi.isRecordingActive);
};

callRecordingApi.on('isRecordingActiveChanged', isRecordingActiveChangedHandler);

```

## <a name="call-transfer-management"></a>Управление перенаправлением вызовов

Перенаправление вызовов является расширенной функцией основного API `Call`. Сначала необходимо получить объект API функции перенаправления:

```js
const callTransferApi = call.api(Features.Transfer);
```

Процесс перенаправления вызова затрагивает трех участников: *передающая сторона*, *передаваемая сторона* и *целевая сторона*. Поток передачи выполняется следующим образом:

1. существует уже установленный вызов между *передающей стороной* и *передаваемой стороной*;
2. *передающая сторона* хочет передать вызов (*передаваемая сторона* -> *целевая сторона*);
3. *передающая сторона* вызывает API `transfer`;
4. *передаваемая сторона* решает подтвердить (`accept`) или отклонить (`reject`) запрос на передачу к *целевой стороне* через событие `transferRequested`;
5. *целевая сторона* получает входящий вызов, если *передаваемая сторона* подтверждает (`accept`) запрос на передачу.

### <a name="transfer-terminology"></a>Терминология, связанная с передачей вызовов

- Передающая сторона — участник вызова, который инициирует запрос на передачу.
- Передаваемая сторона — участник вызова, который перенаправляется к целевой стороне.
- Целевая сторона — новый участник, к которому перенаправляется передаваемая сторона.

Для передачи текущего вызова можно использовать синхронный API `transfer`. `transfer` принимает необязательный аргумент `TransferCallOptions`, который позволяет задать флаг `disableForwardingAndUnanswered`:

- `disableForwardingAndUnanswered` = false — если *целевая сторона* не ответит на входящий вызов, осуществляемый при передаче, применяются параметры переадресации и отсутствия ответа, настроенные для *целевой стороны*;
- `disableForwardingAndUnanswered` = false — если *целевая сторона* не ответит на входящий вызов, осуществляемый при передаче, попытка передачи вызова прерывается.

```js
// transfer target can be ACS user
const id = { communicationUserId: <ACS_USER_ID> };
```

```js
// call transfer API
const transfer = callTransferApi.transfer({targetParticipant: id});
```

При передаче вызова можно подписаться на события `transferStateChanged` и `transferRequested`. Событие `transferRequsted` поступает от экземпляра `call`, а событие `transferStateChanged` и передача `state` и `error` поступают от экземпляра `transfer`.

```js
// transfer state
const transferState = transfer.state; // None | Transferring | Transferred | Failed

// to check the transfer failure reason
const transferError = transfer.error; // transfer error code that describes the failure if transfer request failed
```

Передаваемая сторона может принять или отклонить запрос на передачу, созданный передающей стороной в событии `transferRequested` с помощью `accept()` или `reject()` в `transferRequestedEventArgs`. Вы можете получить доступ к информации `targetParticipant` и методам `accept` и `reject` через `transferRequestedEventArgs`.

```js
// Transferee to accept the transfer request
callTransferApi.on('transferRequested', args => {
  args.accept();
});

// Transferee to reject the transfer request
callTransferApi.on('transferRequested', args => {
  args.reject();
});
```

## <a name="eventing-model"></a>Модель событий
Вам нужно проверить текущие значения и подписаться на события обновления для получения будущих значений.

### <a name="properties"></a>Свойства

```js
// Inspect current value
console.log(object.property);

// Subscribe to value updates
object.on('propertyChanged', () => {
    // Inspect new value
    console.log(object.property)
});

// Unsubscribe from updates:
object.off('propertyChanged', () => {});



// Example for inspecting call state
console.log(call.state);
call.on('stateChanged', () => {
    console.log(call.state);
});
call.off('stateChanged', () => {});
```

### <a name="collections"></a>Коллекции
```js
// Inspect current collection
object.collection.forEach(v => {
    console.log(v);
});

// Subscribe to collection updates
object.on('collectionUpdated', e => {
    // Inspect new values added to the collection
    e.added.forEach(v => {
        console.log(v);
    });
    // Inspect values removed from the collection
    e.removed.forEach(v => {
        console.log(v);
    });
});

// Unsubscribe from updates:
object.off('collectionUpdated', () => {});



// Example for subscribing to remote participants and their video streams
call.remoteParticipants.forEach(p => {
    subscribeToRemoteParticipant(p);
})

call.on('remoteParticipantsUpdated', e => {
    e.added.forEach(p => { subscribeToRemoteParticipant(p) })
    e.removed.forEach(p => { unsubscribeFromRemoteParticipant(p) })
});

function subscribeToRemoteParticipant(p) {
    console.log(p.state);
    p.on('stateChanged', () => { console.log(p.state); });
    p.videoStreams.forEach(v => { subscribeToRemoteVideoStream(v) });
    p.on('videoStreamsUpdated', e => { e.added.forEach(v => { subscribeToRemoteVideoStream(v) }) })
}
```
