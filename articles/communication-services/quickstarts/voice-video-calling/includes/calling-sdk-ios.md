---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: d36bf92a1b1bdef4e45b22b934728b3e8c46c3da
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105107762"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно. 
- Развернутый ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- `User Access Token` для включения клиента вызова. Дополнительные сведения о том, [как получить `User Access Token`](../../access-tokens.md)
- (Необязательно) Выполните инструкции из краткого руководства [Добавление функции голосового вызова в приложение](../getting-started-with-calling.md).

## <a name="setting-up"></a>Настройка

### <a name="creating-the-xcode-project"></a>Создание проекта Xcode

> [!NOTE]
> В этом документе используется версия 1.0.0-Beta. 8 вызывающего пакета SDK.

В Xcode создайте новый проект iOS и выберите шаблон **Single View App** (Приложение с одним представлением). В этом кратком руководстве используется [платформа SwiftUI](https://developer.apple.com/xcode/swiftui/), поэтому для параметра **Language** (Язык) нужно задать значение **Swift**, а для параметра **User Interface** (Пользовательский интерфейс) — значение **SwiftUI**. В рамках этого краткого руководства вы не будете создавать модульные тесты или тесты пользовательского интерфейса. Поэтому можно снять флажки **Include Unit Tests** (Включить модульные тесты) и **Include UI Tests** (Включить тесты пользовательского интерфейса).

:::image type="content" source="../media/ios/xcode-new-ios-project.png" alt-text="Снимок экрана с демонстрацией создания окна New Project (Новый проект) в Xcode.":::

### <a name="install-the-package-and-dependencies-with-cocoapods"></a>Установка пакета и его зависимостей с помощью CocoaPods

1. Создайте файл Podfile для своего приложения с таким содержимым:

   ```
   platform :ios, '13.0'
   use_frameworks!
   target 'AzureCommunicationCallingSample' do
     pod 'AzureCommunicationCalling', '~> 1.0.0-beta.8'
     pod 'AzureCommunication', '~> 1.0.0-beta.8'
     pod 'AzureCore', '~> 1.0.0-beta.8'
   end
   ```

2. Выполните `pod install`.
3. Откройте файл `.xcworkspace` с помощью Xcode.

### <a name="request-access-to-the-microphone"></a>Запрос доступа к микрофону

Чтобы получить доступ к микрофону устройства, вам необходимо указать ключ `NSMicrophoneUsageDescription` в списке свойств сведений приложения. Задайте связанное значение для строки `string`, которая будет включена в диалоговое окно, отображаемое системой при запросе доступа у пользователя.

В дереве проекта щелкните правой кнопкой мыши запись `Info.plist` и выберите **Open As** > **Source Code** (Открыть как > Исходный код). Добавьте в раздел верхнего уровня `<dict>` следующие строки, а затем сохраните файл.

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Need microphone access for VOIP calling.</string>
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Откройте файл **ContentView.swift** проекта и добавьте объявление `import` в начало файла, чтобы импортировать `AzureCommunicationCalling library`. Кроме того, импортируйте `AVFoundation`, чтобы мы могли реализовать в коде запрос на доступ к аудио.

```swift
import AzureCommunicationCalling
import AVFoundation
```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы обрабатывали некоторые основные функции вызывающего пакета SDK служб связи Azure для iOS.


| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient | Каллклиент — это Главная точка входа для вызывающего пакета SDK.|
| CallAgent | CallAgent используется для инициирования вызовов и управления ими. |
| CommunicationTokenCredential | CommunicationTokenCredential используется в качестве учетных данных маркера для создания экземпляра CallAgent.| 
| CommunicationIdentifier | CommunicationIdentifier используется для представления идентификатора пользователя, который может иметь один из следующих типов: CommunicationUserIdentifier, PhoneNumberIdentifier, CallingApplication. |

> [!NOTE]
> Для реализации делегатов событий требуется, чтобы приложение содержало строгую ссылку на объекты, которым требуются подписки на события. Например, если объект `RemoteParticipant` возвращается при вызове метода `call.addParticipant`, а приложение настраивает делегата для прослушивания `RemoteParticipantDelegate`, приложение должно содержать строгую ссылку на объект `RemoteParticipant`. В противном случае, если этот объект будет собран, делегат выдаст неустранимое исключение, когда вызывающий пакет SDK попытается вызвать объект.

## <a name="initialize-the-callagent"></a>Инициализация CallAgent

Чтобы создать экземпляр `CallAgent` из `CallClient`, необходимо использовать метод `callClient.createCallAgent`, который асинхронно возвращает объект `CallAgent` после его инициализации.

Для создания клиента вызова нужно передать объект `CommunicationTokenCredential`.

```swift

import AzureCommunication

let tokenString = "token_string"
var userCredential: CommunicationTokenCredential?
var userCredential: CommunicationTokenCredential?
   do {
       userCredential = try CommunicationTokenCredential(with: CommunicationTokenRefreshOptions(initialToken: token, 
                                                                     refreshProactively: true,
                                                                     tokenRefresher: self.fetchTokenSync))
   } catch {
       return
}

// tokenProvider needs to be implemented by contoso which fetches new token
public func fetchTokenSync(then onCompletion: TokenRefreshOnCompletion) {
    let newToken = self.tokenProvider!.fetchNewToken()
    onCompletion(newToken, nil)
}
```

Передайте созданный ранее объект `CommunicationTokenCredential` в `CallClient` и задайте отображаемое имя.

```swift

callClient = CallClient()
let callAgentOptions:CallAgentOptions = CallAgentOptions()!
options.displayName = " iOS User"

callClient?.createCallAgent(userCredential: userCredential!,
    options: callAgentOptions) { (callAgent, error) in
        if error == nil {
            print("Create agent succeeded")
            self.callAgent = callAgent
        } else {
            print("Create agent failed")
        }
})

```

## <a name="place-an-outgoing-call"></a>Осуществление исходящего вызова

Чтобы создать и запустить вызов, необходимо вызвать один из API в `CallAgent` и предоставить удостоверение служб связи для пользователя, подготовленного с помощью пакета SDK для управления службами связи.

Создание и запуск вызова активируются синхронно. Вы получите экземпляр вызова, который позволяет подписываться на все события в этом вызове.

### <a name="place-a-11-call-to-a-user-or-a-1n-call-with-users-and-pstn"></a>Осуществление персонального вызова к пользователю или группового вызова к пользователям и ТСОП

```swift

let callees = [CommunicationUser(identifier: 'UserId')]
let oneToOneCall = self.callAgent.call(participants: callees, options: StartCallOptions())

```

### <a name="place-a-1n-call-with-users-and-pstn"></a>Осуществление группового вызова к пользователям и ТСОП
Чтобы создать вызов к ТСОП, необходимо указать номер телефона, полученный от Служб коммуникации:
```swift

let pstnCallee = PhoneNumberIdentifier(phoneNumber: '+1999999999')
let callee = CommunicationUserIdentifier(identifier: 'UserId')
let groupCall = self.callAgent.call(participants: [pstnCallee, callee], options: StartCallOptions())

```

### <a name="place-a-11-call-with-with-video"></a>Осуществление персонального вызова с поддержкой видео
Сведения о том, как получить экземпляр диспетчера устройств, см. [здесь](#device-management).

```swift

let camera = self.deviceManager!.cameras!.first
let localVideoStream = LocalVideoStream(camera: camera)
let videoOptions = VideoOptions(localVideoStream: localVideoStream)

let startCallOptions = StartCallOptions()
startCallOptions?.videoOptions = videoOptions

let callee = CommunicationUserIdentifier(identifier: 'UserId')
let call = self.callAgent?.call(participants: [callee], options: startCallOptions)

```

### <a name="join-a-group-call"></a>Присоединение к групповому вызову
Чтобы присоединиться к вызову, необходимо вызвать один из API для *CallAgent*.

```swift

let groupCallLocator = GroupCallLocator(groupId: UUID(uuidString: "uuid_string"))!
let call = self.callAgent?.join(with: groupCallLocator, joinCallOptions: JoinCallOptions())

```

### <a name="subscribe-for-incoming-call"></a>Подписка на входящий вызов
Подписка на событие входящего вызова

```
final class IncomingCallHandler: NSObject, CallAgentDelegate, IncomingCallDelegate
{
    // Event raised when there is an incoming call
    public func onIncomingCall(_ callAgent: CallAgent!, incomingcall: IncomingCall!) {
        self.incomingCall = incomingcall
        // Subscribe to get OnCallEnded event
        self.incomingCall?.delegate = self
    }

    // Event raised when incoming call was not answered
    public func onCallEnded(_ incomingCall: IncomingCall!, args: PropertyChangedEventArgs!) {
        self.incomingCall = nil
    }
}
```

### <a name="accept-an-incoming-call"></a>Прием входящего вызова
Чтобы принять вызов, вызовите метод accept для объекта вызова.
Задание делегата для CallAgent 
```swift
final class CallHandler: NSObject, CallAgentDelegate
{
    public var incomingCall: Call?
 
    public func onCallsUpdated(_ callAgent: CallAgent!, args: CallsUpdatedEventArgs!) {
        if let incomingCall = args.addedCalls?.first(where: { $0.isIncoming }) {
            self.incomingCall = incomingCall
        }
    }
}

let firstCamera: VideoDeviceInfo? = self.deviceManager!.cameras!.first
let localVideoStream = LocalVideoStream(camera: firstCamera)
let acceptCallOptions = AcceptCallOptions()
acceptCallOptions!.videoOptions = VideoOptions(localVideoStream:localVideoStream!)
if let incomingCall = CallHandler().incomingCall {
   incomingCall.accept(options: acceptCallOptions) { (call, error) in
               if error == nil {
                   print("Incoming call accepted")
               } else {
                   print("Failed to accept incoming call")
               }
           }
} else {
   print("No incoming call found to accept")
}
```

## <a name="push-notification"></a>push-уведомление

Мобильное push-уведомление — это всплывающее уведомление, которое вы получаете на мобильном устройстве. Для вызовов мы будем использовать push-уведомления протокола VoIP (голосовые вызовы через IP). Мы предоставим вам возможности регистрации на получение push-уведомлений, управления push-уведомлениями и отмены регистрации на получение push-уведомлений.

### <a name="prerequisite"></a>Предварительное требование

- Шаг 1. Выберите в Xcode -> Signing & Capabilities (Подписывание и возможности) -> Add Capability (Добавить возможность) -> Push Notifications (Push-уведомления).
- Шаг 2. Выберите в Xcode -> Signing & Capabilities (Подписывание и возможности) -> Add Capability (Добавить возможность) -> Background Modes (Режимы фоновой работы).
- Шаг 3. Выберите для Background Modes (Режимы фоновой работы) -> Voice over IP (Голосовая связь через IP) и Remote notifications (Удаленные уведомления).

:::image type="content" source="../media/ios/xcode-push-notification.png" alt-text="Снимок экрана: как добавить новые возможности в Xcode." lightbox="../media/ios/xcode-push-notification.png":::

#### <a name="register-for-push-notifications"></a>Регистрация для получения push-уведомлений

Чтобы зарегистрироваться для получения push-уведомления, вызовите registerPushNotification() в экземпляре *CallAgent*, предоставив маркер регистрации устройства.

Регистрацию для получения push-уведомлений нужно вызывать после успешной инициализации. При уничтожении объекта `callAgent` будет вызван метод `logout`, который автоматически отменит регистрацию на получение push-уведомлений.


```swift

let deviceToken: Data = pushRegistry?.pushToken(for: PKPushType.voIP)
callAgent.registerPushNotifications(deviceToken: deviceToken) { (error) in
    if(error == nil) {
        print("Successfully registered to push notification.")
    } else {
        print("Failed to register push notification.")
    }
}

```

#### <a name="push-notification-handling"></a>Обработка push-уведомлений
Чтобы получать push-уведомления о входящих вызовах, вызовите *handlePushNotification()* для экземпляра *CallAgent*, передав полезные данные словаря.

```swift

let callNotification = IncomingCallInformation.from(payload: pushPayload?.dictionaryPayload)

callAgent.handlePush(notification: callNotification) { (error) in
    if (error != nil) {
        print("Handling of push notification failed")
    } else {
        print("Handling of push notification was successful")
    }
}

```
#### <a name="unregister-push-notification"></a>Отмена регистрации для получения push-уведомлений

В приложениях можно в любое время отменить регистрацию для получения push-уведомлений. Просто вызовите метод `unregisterPushNotification` для *CallAgent*.
> [!NOTE]
> Регистрация приложений для получения push-уведомлений не отменяется автоматически при выходе из системы.

```swift

callAgent.unregisterPushNotifications { (error) in
    if (error != nil) {
        print("Unregister of push notification failed, please try again")
    } else {
        print("Unregister of push notification was successful")
    }
}

```

## <a name="mid-call-operations"></a>Операции в процессе вызова

Во время вызова вы можете выполнять различные операции для управления параметрами видео и аудио.

### <a name="mute-and-unmute"></a>Отключение и включение звука

Чтобы отключить или включить звук для локальной конечной точки, можно использовать асинхронные вызовы API `mute` и `unmute`:

```swift
call!.mute { (error) in
    if error == nil {
        print("Successfully muted")
    } else {
        print("Failed to mute")
    }
}

```

(Асинхронно) Локальное включение звука:

```swift
call!.unmute { (error) in
    if error == nil {
        print("Successfully un-muted")
    } else {
        print("Failed to unmute")
    }
}
```

### <a name="start-and-stop-sending-local-video"></a>Запуск и остановка отправки локального видео

Чтобы начать отправку локального видео другим участникам вызова, воспользуйтесь вызовом API `startVideo` и передайте `localVideoStream` с `camera`.

```swift

let firstCamera: VideoDeviceInfo? = self.deviceManager!.cameras!.first
let localVideoStream = LocalVideoStream(camera: firstCamera)

call!.startVideo(stream: localVideoStream) { (error) in
    if (error == nil) {
        print("Local video started successfully")
    } else {
        print("Local video failed to start")
    }
}

```

После начала отправки видео экземпляр `LocalVideoStream` добавляется в коллекцию `localVideoStreams` в экземпляре вызова:

```swift

call.localVideoStreams[0]

```

(Асинхронно) Чтобы отключить локальное видео, передайте `localVideoStream`, полученный при вызове `call.startVideo`:

```swift

call!.stopVideo(stream: localVideoStream) { (error) in
    if (error == nil) {
        print("Local video stopped successfully")
    } else {
        print("Local video failed to stop")
    }
}

```

## <a name="remote-participants-management"></a>Управление удаленными участниками

Все удаленные участники представлены типом `RemoteParticipant` и доступны через коллекцию `remoteParticipants` в экземпляре вызова.

### <a name="list-participants-in-a-call"></a>Вывод списка участников в вызове

```swift

call.remoteParticipants

```

### <a name="remote-participant-properties"></a>Свойства удаленного участника

```swift

// [RemoteParticipantDelegate] delegate - an object you provide to receive events from this RemoteParticipant instance
var remoteParticipantDelegate = remoteParticipant.delegate

// [CommunicationIdentifier] identity - same as the one used to provision token for another user
var identity = remoteParticipant.identity

// ParticipantStateIdle = 0, ParticipantStateEarlyMedia = 1, ParticipantStateConnecting = 2, ParticipantStateConnected = 3, ParticipantStateOnHold = 4, ParticipantStateInLobby = 5, ParticipantStateDisconnected = 6
var state = remoteParticipant.state

// [Error] callEndReason - reason why participant left the call, contains code/subcode/message
var callEndReason = remoteParticipant.callEndReason

// [Bool] isMuted - indicating if participant is muted
var isMuted = remoteParticipant.isMuted

// [Bool] isSpeaking - indicating if participant is currently speaking
var isSpeaking = remoteParticipant.isSpeaking

// RemoteVideoStream[] - collection of video streams this participants has
var videoStreams = remoteParticipant.videoStreams // [RemoteVideoStream, RemoteVideoStream, ...]

```

### <a name="add-a-participant-to-a-call"></a>Добавление участника в вызов

Чтобы добавить в вызов участника (пользователя или номер телефона), можно вызвать `addParticipant`. Этот вызов в синхронном режиме возвращает экземпляр удаленного участника.

```swift

let remoteParticipantAdded: RemoteParticipant = call.add(participant: CommunicationUserIdentifier(identifier: "userId"))

```

### <a name="remove-a-participant-from-a-call"></a>Удаление участника из вызова
Чтобы удалить участника (пользователя или номер телефона) из вызова, можно воспользоваться вызовом API `removeParticipant`. Разрешение выполняется асинхронно.

```swift

call!.remove(participant: remoteParticipantAdded) { (error) in
    if (error == nil) {
        print("Successfully removed participant")
    } else {
        print("Failed to remove participant")
    }
}

```

## <a name="render-remote-participant-video-streams"></a>Отрисовка видеопотоков удаленных участников

Удаленные участники могут предоставить общий доступ к видео или экрану во время вызова.

### <a name="handle-remote-participant-videoscreen-sharing-streams"></a>Обработка потоков доступа к видео или экрану от удаленного участника

Чтобы получить список потоков от удаленных участников, изучите коллекции `videoStreams`:

```swift

var remoteParticipantVideoStream = call.remoteParticipants[0].videoStreams[0]

```

### <a name="remote-video-stream-properties"></a>Свойства удаленного видеопотока

```swift

var type: MediaStreamType = remoteParticipantVideoStream.type // 'MediaStreamTypeVideo'

var isAvailable: Bool = remoteParticipantVideoStream.isAvailable // indicates if remote stream is available

var id: Int = remoteParticipantVideoStream.id // id of remoteParticipantStream

```

### <a name="render-remote-participant-stream"></a>Отрисовка потока удаленного участника

Чтобы начать отрисовку потоков удаленного участника, используйте следующий код:

```swift

let renderer: Renderer? = Renderer(remoteVideoStream: remoteParticipantVideoStream)
let targetRemoteParticipantView: RendererView? = renderer?.createView(with: RenderingOptions(scalingMode: ScalingMode.crop))
// To update the scaling mode later
targetRemoteParticipantView.update(scalingMode: ScalingMode.fit)

```

### <a name="remote-video-renderer-methods-and-properties"></a>Методы и свойства отрисовщика удаленного видео

```swift
// [Synchronous] dispose() - dispose renderer and all `RendererView` associated with this renderer. To be called when you have removed all associated views from the UI.
remoteVideoRenderer.dispose()
```

## <a name="device-management"></a>Управление устройствами

`DeviceManager` позволяет получить список локальных устройств, которые можно использовать в вызове для передачи аудио- и видеопотоков. Он также позволяет запросить у пользователя разрешение на доступ к микрофону или камере. Вы можете получить доступ к `deviceManager` через объект `callClient`:

```swift

self.callClient!.getDeviceManager { (deviceManager, error) in
        if (error == nil) {
            print("Got device manager instance")
            self.deviceManager = deviceManager
        } else {
            print("Failed to get device manager instance")
        }
    }
```

### <a name="enumerate-local-devices"></a>Перечисление локальных устройств

Для доступа к локальным устройствам можно использовать методы перечисления из Диспетчера устройств. Действие перечисления выполняется асинхронно.

```swift
// enumerate local cameras
var localCameras = deviceManager.cameras! // [VideoDeviceInfo, VideoDeviceInfo...]
// enumerate local cameras
var localMicrophones = deviceManager.microphones! // [AudioDeviceInfo, AudioDeviceInfo...]
// enumerate local cameras
var localSpeakers = deviceManager.speakers! // [AudioDeviceInfo, AudioDeviceInfo...]
``` 

### <a name="set-default-microphonespeaker"></a>Назначение микрофона или динамика по умолчанию

Диспетчер устройств позволяет задать устройство по умолчанию, которое будет использоваться при осуществлении вызова. Если значения по умолчанию для стека не заданы, Службы коммуникации будут использовать значения по умолчанию для ОС.

```swift
// get first microphone
var firstMicrophone = self.deviceManager!.cameras!.first
// [Synchronous] set microphone
deviceManager.setMicrophone(microphoneDevice: firstMicrophone)
// get first speaker
var firstSpeaker = self.deviceManager!.speakers!
// [Synchronous] set speaker
deviceManager.setSpeaker(speakerDevice: firstSpeaker)
```

### <a name="local-camera-preview"></a>Предварительный просмотр изображения с локальной камеры

С помощью `Renderer` можно начать отрисовку потока с локальной камеры. Этот поток не отправляется другим участникам, а используется как локальный канал предварительного просмотра. Это действие выполняется асинхронно.

```swift

let camera: VideoDeviceInfo = self.deviceManager!.getCameraList()![0]
let localVideoStream: LocalVideoStream = LocalVideoStream(camera: camera)
let renderer: Renderer = Renderer(localVideoStream: localVideoStream)
self.view = try renderer!.createView()

```

### <a name="local-camera-preview-properties"></a>Свойства предварительного просмотра изображения с локальной камеры

Отрисовщик предоставляет ряд свойств и методов для управления отрисовкой:

```swift

// Constructor can take in LocalVideoStream or RemoteVideoStream
let localRenderer = Renderer(localVideoStream:localVideoStream)
let remoteRenderer = Renderer(remoteVideoStream:remoteVideoStream)

// [StreamSize] size of the rendering view
localRenderer.size

// [RendererDelegate] an object you provide to receive events from this Renderer instance
localRenderer.delegate

// [Synchronous] create view
try! localRenderer.createView()

// [Synchronous] create view with rendering options
try! localRenderer.createView(with: RenderingOptions(scalingMode: ScalingMode.fit))

// [Synchronous] dispose rendering view
localRenderer.dispose()

```

## <a name="eventing-model"></a>Модель событий

Вы можете подписываться на большинство свойств и коллекций, чтобы получать уведомления при изменении их значений.

### <a name="properties"></a>Свойства
Чтобы подписаться на события `property changed`, выполните следующий код:

```swift
call.delegate = self
// Get the property of the call state by doing get on the call's state member
public func onCallStateChanged(_ call: Call!,
                               args: PropertyChangedEventArgs!)
{
    print("Callback from SDK when the call state changes, current state: " + call.state.rawValue)
}

 // to unsubscribe
 self.call.delegate = nil

```

### <a name="collections"></a>Коллекции
Чтобы подписаться на события `collection updated`, выполните следующий код:

```swift
call.delegate = self
// Collection contains the streams that were added or removed only
public func onLocalVideoStreamsChanged(_ call: Call!,
                                       args: LocalVideoStreamsUpdatedEventArgs!)
{
    print(args.addedStreams.count)
    print(args.removedStreams.count)
}
// to unsubscribe
self.call.delegate = nil
```
