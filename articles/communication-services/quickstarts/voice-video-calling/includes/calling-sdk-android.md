---
author: mikben
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: mikben
ms.openlocfilehash: 3eadf556d467fb67be0b0fe3a83dfd4ed89e6463
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103488146"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно. 
- Развернутый ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- `User Access Token` для включения клиента вызова. Дополнительные сведения о том, [как получить `User Access Token`](../../access-tokens.md)
- (Необязательно) Выполните инструкции из краткого руководства [Добавление функции голосового вызова в приложение](../getting-started-with-calling.md).

## <a name="setting-up"></a>Настройка

### <a name="install-the-package"></a>Установка пакета

> [!NOTE]
> В этом документе используется клиентская библиотека вызовов версии 1.0.0-beta.8.

<!-- TODO: update with instructions on how to download, install and add package to project -->
Выберите build.gradle на уровне проекта и добавьте `mavenCentral()` в список репозиториев в разделах `buildscript` и `allprojects`.
```groovy
buildscript {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```

```groovy
allprojects {
    repositories {
    ...
        mavenCentral()
    ...
    }
}
```
Затем добавьте в build.gradle на уровне модуля следующие строки в разделе dependencies:

```groovy
dependencies {
    ...
    implementation 'com.azure.android:azure-communication-calling:1.0.0-beta.8'
    ...
}

```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки Служб коммуникации Azure для вызовов:

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient| CallClient — это основная точка входа в клиентскую библиотеку для вызовов.|
| CallAgent | CallAgent используется для инициирования вызовов и управления ими. |
| CommunicationTokenCredential | CommunicationTokenCredential используется в качестве учетных данных маркера для создания экземпляра CallAgent.|
| CommunicationIdentifier | CommunicationIdentifier используется в качестве участника другого типа, который может быть частью вызова.|

## <a name="initialize-the-callclient-create-a-callagent-and-access-the-devicemanager"></a>Инициализация CallClient, создание CallAgent и получение доступа к DeviceManager

Чтобы создать экземпляр `CallAgent`, необходимо вызвать метод `createCallAgent` для экземпляра `CallClient`. Это действие асинхронно возвращает объект экземпляра `CallAgent`.
Метод `createCallAgent` принимает `CommunicationUserCredential` в качестве аргумента, который инкапсулирует [маркер доступа](../../access-tokens.md).
Чтобы получить доступ к `DeviceManager`, нужно сначала создать экземпляр CallAgent, а затем получить DeviceManager с помощью метода `CallClient.getDeviceManager`.

```java
String userToken = '<user token>';
CallClient callClient = new CallClient();
CommunicationTokenCredential tokenCredential = new CommunicationTokenCredential(userToken);
android.content.Context appContext = this.getApplicationContext(); // From within an Activity for instance
CallAgent callAgent = callClient.createCallAgent((appContext, tokenCredential).get();
DeviceManage deviceManager = callClient.getDeviceManager().get();
```
Чтобы задать отображаемое имя для вызывающей стороны, используйте следующий альтернативный метод:

```java
String userToken = '<user token>';
CallClient callClient = new CallClient();
CommunicationTokenCredential tokenCredential = new CommunicationTokenCredential(userToken);
android.content.Context appContext = this.getApplicationContext(); // From within an Activity for instance
CallAgentOptions callAgentOptions = new CallAgentOptions();
callAgentOptions.setDisplayName("Alice Bob");
CallAgent callAgent = callClient.createCallAgent((appContext, tokenCredential, callAgentOptions).get();
DeviceManage deviceManager = callClient.getDeviceManager().get();
```


## <a name="place-an-outgoing-call-and-join-a-group-call"></a>Создание исходящего вызова и присоединение к групповому вызову

Чтобы создать и начать вызов, необходимо вызвать метод `CallAgent.startCall()` и предоставить `Identifier` вызываемой стороны.
Чтобы присоединиться к групповому вызову, нужно вызвать метод `CallAgent.join()` и предоставить значение groupId. Идентификаторы групп должны иметь формат GUID или UUID.

Операции создания и начала вызовов выполняются синхронно. Экземпляр вызова позволяет подписываться на все события в вызове.

### <a name="place-a-11-call-to-a-user"></a>Осуществление персонального вызова к пользователю
Чтобы осуществить вызов к другому пользователю Служб коммуникации, вызовите метод `call` для `callAgent` и передайте в нем объект с ключом `communicationUserId`.
```java
StartCallOptions startCallOptions = new StartCallOptions();
Context appContext = this.getApplicationContext();
CommunicationUserIdentifier acsUserId = new CommunicationUserIdentifier(<USER_ID>);
CommunicationUserIdentifier participants[] = new CommunicationUserIdentifier[]{ acsUserId };
call oneToOneCall = callAgent.startCall(appContext, participants, startCallOptions);
```

### <a name="place-a-1n-call-with-users-and-pstn"></a>Осуществление группового вызова к пользователям и ТСОП
> [!WARNING]
> В настоящее время вызов ТСОП недоступен

Чтобы осуществить групповой вызов к пользователю и номеру ТСОП, необходимо указать номер телефона вызываемой стороны.
Для осуществления вызовов к ТСОП необходимо настроить ресурс Служб коммуникации.
```java
CommunicationUserIdentifier acsUser1 = new CommunicationUserIdentifier(<USER_ID>);
PhoneNumberIdentifier acsUser2 = new PhoneNumberIdentifier("<PHONE_NUMBER>");
CommunicationIdentifier participants[] = new CommunicationIdentifier[]{ acsUser1, acsUser2 };
StartCallOptions startCallOptions = new StartCallOptions();
Context appContext = this.getApplicationContext();
Call groupCall = callAgent.startCall(participants, startCallOptions);
```

### <a name="place-a-11-call-with-video-camera"></a>Осуществление персонального вызова с использованием видеокамеры
> [!WARNING]
> В настоящее время поддерживается только один исходящий локальный видеопоток. Чтобы осуществить вызов с поддержкой видео, необходимо перечислить локальные камеры через API `deviceManager` `getCameras`.
После выбора нужной камеры создайте на ее основе экземпляр `LocalVideoStream` и передайте его в `videoOptions` в качестве элемента массива `localVideoStream` в метод `call`.
Когда вызов будет осуществлен, автоматически начнется отправка видеопотока с выбранной камеры другим участникам вызова.

> [!NOTE]
> Для защиты конфиденциальности видео не будет передаваться участникам вызова, если не просматривается локально.
Дополнительные сведения см. в разделе [Предварительный просмотр изображения с локальной камеры](#local-camera-preview).
```java
Context appContext = this.getApplicationContext();
VideoDeviceInfo desiredCamera = callClient.getDeviceManager().get().getCameras().get(0);
LocalVideoStream currentVideoStream = new LocalVideoStream(desiredCamera, appContext);
VideoOptions videoOptions = new VideoOptions(currentVideoStream);

// Render a local preview of video so the user knows that their video is being shared
Renderer previewRenderer = new Renderer(currentVideoStream, appContext);
View uiView = previewRenderer.createView(new RenderingOptions(ScalingMode.Fit));
// Attach the uiView to a viewable location on the app at this point
layout.addView(uiView);

CommunicationUserIdentifier[] participants = new CommunicationUserIdentifier[]{ new CommunicationUserIdentifier("<acs user id>") };
StartCallOptions startCallOptions = new StartCallOptions();
startCallOptions.setVideoOptions(videoOptions);
Call call = callAgent.startCall(context, participants, startCallOptions);
```

### <a name="join-a-group-call"></a>Присоединение к групповому вызову
Чтобы начать новый групповой вызов или присоединиться к текущему групповому вызову, необходимо вызвать метод "join" и передать ему объект со свойством `groupId`. В качестве значения должен быть указан идентификатор GUID.
```java
Context appContext = this.getApplicationContext();
GroupCallLocator groupCallLocator = new GroupCallLocator("<GUID>");
JoinCallOptions joinCallOptions = new JoinCallOptions();

call = callAgent.join(context, groupCallLocator, joinCallOptions);
```

### <a name="accept-a-call"></a>Прием вызова
Чтобы принять вызов, вызовите метод accept для объекта вызова.

```java
Context appContext = this.getApplicationContext();
IncomingCall incomingCall = retrieveIncomingCall();
Call call = incomingCall.accept(context).get();
```

Чтобы принять вызов с включенной видеокамерой, сделайте следующее:

```java
Context appContext = this.getApplicationContext();
IncomingCall incomingCall = retrieveIncomingCall();
AcceptCallOptions acceptCallOptions = new AcceptCallOptions();
VideoDeviceInfo desiredCamera = callClient.getDeviceManager().get().getCameraList().get(0);
acceptCallOptions.setVideoOptions(new VideoOptions(new LocalVideoStream(desiredCamera, appContext)));
Call call = incomingCall.accept(context, acceptCallOptions).get();
```

Чтобы получить входящий вызов, следует подписаться на событие `onIncomingCall` в объекте `callAgent`:

```java
// Assuming "callAgent" is an instance property obtained by calling the 'createCallAgent' method on CallClient instance 
public Call retrieveIncomingCall() {
    IncomingCall incomingCall;
    callAgent.addOnIncomingCallListener(new IncomingCallListener() {
        void onIncomingCall(IncomingCall inboundCall) {
            // Look for incoming call
            incomingCall = inboundCall;
        }
    });
    return incomingCall;
}
```


## <a name="push-notifications"></a>Push-уведомления

### <a name="overview"></a>Обзор
Мобильные push-уведомления — это всплывающие уведомления, которые отображаются на мобильных устройствах. Для вызовов мы будем использовать push-уведомления протокола VoIP (голосовые вызовы через IP). Мы изучим регистрацию для получения push-уведомлений, обработку push-уведомлений и отмену регистрации для получения push-уведомлений.

### <a name="prerequisites"></a>Предварительные требования

Учетная запись Firebase, для которой включена облачная служба сообщений (FCM), и служба Firebase Cloud Messaging, подключенная к экземпляру центра уведомлений Azure. Дополнительные сведения см. в разделе [Уведомления в Службах коммуникации](../../../concepts/notifications.md).
Кроме того, в этом руководстве предполагается, что для создания приложения вы используете Android Studio версии 3.6 или более поздней.

Чтобы получать уведомления от Firebase Cloud Messaging, приложению Android требуется определенный набор разрешений. В файле `AndroidManifest.xml` добавьте следующий набор разрешений сразу после тега *<manifest ...>* или под тегом *</application>* .

```XML
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
```

### <a name="register-for-push-notifications"></a>Регистрация для получения push-уведомлений

Чтобы зарегистрироваться для получения push-уведомлений, приложению необходимо вызвать `registerPushNotification()` для экземпляра *CallAgent* и передать маркер регистрации устройства.

Чтобы получить этот маркер регистрации устройства, добавьте клиентскую библиотеку Firebase в файл модуля приложения *build.gradle*, добавив в раздел `dependencies` следующие строки, если их там еще нет:

```
    // Add the client library for Firebase Cloud Messaging
    implementation 'com.google.firebase:firebase-core:16.0.8'
    implementation 'com.google.firebase:firebase-messaging:20.2.4'
```

В файле *build.gradle* на уровне проекта добавьте в раздел `dependencies` следующий фрагмент, если его там еще нет:

```
    classpath 'com.google.gms:google-services:4.3.3'
```

Добавьте в начало файла следующий подключаемый модуль, если его там еще нет:

```
apply plugin: 'com.google.gms.google-services'
```

На панели инструментов щелкните *Синхронизировать сейчас*. Добавьте следующий фрагмент кода, чтобы получить маркер регистрации устройства, созданный клиентской библиотекой Firebase Cloud Messaging для экземпляра клиентского приложения. Обязательно добавьте приведенные ниже инструкции import в заголовок главного действия (Activity) для экземпляра. Они необходимы для того, чтобы фрагмент кода получил маркер:

```
import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.iid.FirebaseInstanceId;
import com.google.firebase.iid.InstanceIdResult;
```

Для получения маркера добавьте следующий фрагмент кода:

```
        FirebaseInstanceId.getInstance().getInstanceId()
                .addOnCompleteListener(new OnCompleteListener<InstanceIdResult>() {
                    @Override
                    public void onComplete(@NonNull Task<InstanceIdResult> task) {
                        if (!task.isSuccessful()) {
                            Log.w("PushNotification", "getInstanceId failed", task.getException());
                            return;
                        }

                        // Get new Instance ID token
                        String deviceToken = task.getResult().getToken();
                        // Log
                        Log.d("PushNotification", "Device Registration token retrieved successfully");
                    }
                });
```
Зарегистрируйте маркер регистрации устройства в клиентской библиотеке служб вызовов для получения push-уведомлений о входящих вызовах:

```java
String deviceRegistrationToken = "<Device Token from previous section>";
try {
    callAgent.registerPushNotification(deviceRegistrationToken).get();
}
catch(Exception e) {
    System.out.println("Something went wrong while registering for Incoming Calls Push Notifications.")
}
```

### <a name="push-notification-handling"></a>Обработка push-уведомлений

Чтобы получать push-уведомления о входящих вызовах, вызовите *handlePushNotification()* для экземпляра *CallAgent*, передав полезные данные.

Чтобы получить полезные данные из Firebase Cloud Messaging, для начала создайте новую службу (Файл > Создать > Служба > Служба), которая расширяет класс клиентской библиотеки Firebase *FirebaseMessagingService* и переопределите метод `onMessageReceived`. Этот метод является обработчиком событий и вызывается, когда Firebase Cloud Messaging доставляет в приложение push-уведомление.

```java
public class MyFirebaseMessagingService extends FirebaseMessagingService {
    private java.util.Map<String, String> pushNotificationMessageDataFromFCM;

    @Override
    public void onMessageReceived(RemoteMessage remoteMessage) {
        // Check if message contains a notification payload.
        if (remoteMessage.getNotification() != null) {
            Log.d("PushNotification", "Message Notification Body: " + remoteMessage.getNotification().getBody());
        }
        else {
            pushNotificationMessageDataFromFCM = remoteMessage.getData();
        }
    }
}
```
Добавьте приведенное ниже определение службы внутри тега <application> в файле `AndroidManifest.xml`.

```
        <service
            android:name=".MyFirebaseMessagingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
        </service>
```

- Полученные полезные данные можно передать в клиентскую библиотеку *Служб коммуникации*, где они будут преобразованы во внутренний объект *IncomingCallInformation*, для обработки которого вызывается метод *handlePushNotification* для экземпляра *CallAgent*. Экземпляр `CallAgent` создается путем вызова метода `createCallAgent(...)` для класса `CallClient`.

```java
try {
    IncomingCallInformation notification = IncomingCallInformation.fromMap(pushNotificationMessageDataFromFCM);
    Future handlePushNotificationFuture = callAgent.handlePushNotification(notification).get();
}
catch(Exception e) {
    System.out.println("Something went wrong while handling the Incoming Calls Push Notifications.");
}
```

Если обработка push-уведомления пройдет успешно и все обработчики событий будут правильно зарегистрированы, в приложении прозвучит сигнал вызова.

### <a name="unregister-push-notifications"></a>Отмена регистрации для получения push-уведомлений

В приложениях можно в любое время отменить регистрацию для получения push-уведомлений. Чтобы отменить регистрацию, вызовите метод `unregisterPushNotification()` для CallAgent.

```java
try {
    callAgent.unregisterPushNotifications().get();
}
catch(Exception e) {
    System.out.println("Something went wrong while un-registering for all Incoming Calls Push Notifications.")
}
```

## <a name="call-management"></a>Управление вызовами
Во время вызова вы можете получать доступ ко свойствам вызова и выполнять различные операции для управления параметрами, связанными с видео и аудио.

### <a name="call-properties"></a>Свойства вызова

Получите уникальный идентификатор для вызова:

```java
String callId = call.getId();
```

Чтобы получить данные о других участниках вызова, просмотрите коллекцию `remoteParticipant` для экземпляра `call`:

```java
List<RemoteParticipant> remoteParticipants = call.getRemoteParticipants();
```

Для входящих вызовов это идентификатор вызывающей стороны:

```java
CommunicationIdentifier callerId = call.getCallerId();
```

Получите состояние вызова: 

```java
CallState callState = call.getState();
```

Возвращается строка, которая представляет текущее состояние вызова:
* None — исходное состояние любого вызова;
* Connecting — исходное переходное состояние после осуществления или приема вызова;
* Ringing — для исходящего вызова означает, что на стороне удаленных участников звучит сигнал вызова;
* EarlyMedia — обозначает состояние, при котором перед вызовом воспроизводится объявление;
* Connected — вызов осуществлен;
* LocalHold — вызов переведен локальным участником в режим ожидания, и потоки мультимедиа не передаются между локальной конечной точкой и удаленными участниками;
* RemoteHold — вызов переведен удаленным участником в режим ожидания, и потоки мультимедиа не передаются между локальной конечной точкой и удаленными участниками;
* Disconnecting — переходное состояние перед тем, как вызов перейдет в состояние Disconnected;
* Disconnected — завершающее состояние вызова.


Чтобы узнать, почему вызов завершился, проверьте свойство `callEndReason`. Оно содержит основной код и дополнительный код: 

```java
CallEndReason callEndReason = call.getCallEndReason();
int code = callEndReason.getCode();
int subCode = callEndReason.getSubCode();
```

Чтобы узнать, является ли текущий вызов входящим или исходящим, проверьте свойство `callDirection`:

```java
CallDirection callDirection = call.getCallDirection(); 
// callDirection == CallDirection.Incoming for incoming call
// callDirection == CallDirection.Outgoing for outgoing call
```

Чтобы узнать, отключен ли выбранный микрофон, проверьте свойство `muted`:

```java
boolean muted = call.getIsMicrophoneMuted();
```

Чтобы узнать, записывается ли текущий вызов, проверьте свойство `isRecordingActive`:

```java
boolean recordinggActive = call.getIsRecordingActive();
```

Чтобы проверить активные видеопотоки, проверьте коллекцию `localVideoStreams`:

```java
List<LocalVideoStream> localVideoStreams = call.getLocalVideoStreams();
```

### <a name="mute-and-unmute"></a>Отключение и включение звука

Чтобы отключить или включить звук для локальной конечной точки, можно использовать асинхронные вызовы API `mute` и `unmute`:

```java
call.mute().get();
call.unmute().get();
```

### <a name="start-and-stop-sending-local-video"></a>Запуск и остановка отправки локального видео

Чтобы запустить видео, нужно перечислить камеры с помощью API `getCameraList` для объекта `deviceManager`. Затем создайте новый экземпляр `LocalVideoStream`, передав ему данные о нужной камере, и передайте этот экземпляр в качестве аргумента в API `startVideo`:

```java
VideoDeviceInfo desiredCamera = <get-video-device>;
Context appContext = this.getApplicationContext();
LocalVideoStream currentLocalVideoStream = new LocalVideoStream(desiredCamera, appContext);
VideoOptions videoOptions = new VideoOptions(currentLocalVideoStream);
Future startVideoFuture = call.startVideo(currentLocalVideoStream);
startVideoFuture.get();
```

После успешного запуска отправки видео экземпляр `LocalVideoStream` будет добавлен в коллекцию `localVideoStreams` в экземпляре вызова.

```java
currentLocalVideoStream == call.getLocalVideoStreams().get(0);
```

Чтобы остановить отправку локального видео, передайте экземпляр `LocalVideoStream`, доступный в коллекции `localVideoStreams`:

```java
call.stopVideo(currentLocalVideoStream).get();
```

Вы можете переключиться на другое устройство камеры, не прекращая передачу видео, вызвав `switchSource` для экземпляра `LocalVideoStream`:
```java
currentLocalVideoStream.switchSource(source).get();
```

## <a name="remote-participants-management"></a>Управление удаленными участниками

Все удаленные участники представлены типом `RemoteParticipant` и доступны через коллекцию `remoteParticipants` в экземпляре вызова.

### <a name="list-participants-in-a-call"></a>Вывод списка участников в вызове
Коллекция `remoteParticipants` возвращает список удаленных участников в указанном вызове:
```java
List<RemoteParticipant> remoteParticipants = call.getRemoteParticipants(); // [remoteParticipant, remoteParticipant....]
```

### <a name="remote-participant-properties"></a>Свойства удаленного участника
С каждым конкретным удаленным участником связан набор свойств и коллекций:

* Получите идентификатор для этого удаленного участника.
Идентификатор имеет один из типов Identifier.
```java
CommunicationIdentifier participantIdentifier = remoteParticipant.getIdentifier();
```

* Получите состояние удаленного участника.
```java
ParticipantState state = remoteParticipant.getState();
```
Состояние может иметь одно из следующих значений:
* Idle — исходное состояние;
* EarlyMedia — перед подключением к вызову для участника воспроизводится объявление;
* Ringing — для участника звучит сигнал вызова;
* Connected — переходное состояние во время подключения участника к вызову;
* Connected — участник подключен к вызову;
* Hold — участник находится в режиме ожидания;
* InLobby — участник ожидает допуска в "зале ожидания" (в настоящее время используется только в сценарии взаимодействия с Teams);
* Disconnected — окончательное состояние, которое обозначает завершение вызова для участника.


* Чтобы узнать, почему участник покинул вызов, проверьте свойство `callEndReason`:
```java
CallEndReason callEndReason = remoteParticipant.getCallEndReason();
```

* Чтобы проверить, отключен ли звук для удаленного участника, проверьте свойство `isMuted`:
```java
boolean isParticipantMuted = remoteParticipant.getIsMuted();
```

* Чтобы проверить, говорит ли сейчас удаленный участник, проверьте свойство `isSpeaking`:
```java
boolean isParticipantSpeaking = remoteParticipant.getIsSpeaking();
```

* Чтобы проверить все видеопотоки, отправляемые определенным участником в вызове, проверьте коллекцию `videoStreams`:
```java
List<RemoteVideoStream> videoStreams = remoteParticipant.getVideoStreams(); // [RemoteVideoStream, RemoteVideoStream, ...]
```


### <a name="add-a-participant-to-a-call"></a>Добавление участника в вызов

Чтобы добавить в вызов участника (пользователя или номер телефона), можно вызвать `addParticipant`. Этот вызов в синхронном режиме возвращает экземпляр удаленного участника.

```java
const acsUser = new CommunicationUserIdentifier("<acs user id>");
const acsPhone = new PhoneNumberIdentifier("<phone number>");
RemoteParticipant remoteParticipant1 = call.addParticipant(acsUser);
AddPhoneNumberOptions addPhoneNumberOptions = new AddPhoneNumberOptions(new PhoneNumberIdentifier("<alternate phone number>"));
RemoteParticipant remoteParticipant2 = call.addParticipant(acsPhone, addPhoneNumberOptions);
```

### <a name="remove-participant-from-a-call"></a>Удаление участника из вызова
Чтобы удалить участника (пользователя или номер телефона) из вызова, можно вызвать `removeParticipant`.
Это значение будет разрешаться асинхронно после удаления участника из вызова.
Этот же участник будет удален из коллекции `remoteParticipants`.
```java
RemoteParticipant acsUserRemoteParticipant = call.getParticipants().get(0);
RemoteParticipant acsPhoneRemoteParticipant = call.getParticipants().get(1);
call.removeParticipant(acsUserRemoteParticipant).get();
call.removeParticipant(acsPhoneRemoteParticipant).get();
```

## <a name="render-remote-participant-video-streams"></a>Отрисовка видеопотоков удаленных участников
Чтобы получить список видеопотоков и потоков демонстрации экрана, получаемых от удаленных участников, проверьте коллекции `videoStreams`:
```java
RemoteParticipant remoteParticipant = call.getRemoteParticipants().get(0);
RemoteVideoStream remoteParticipantStream = remoteParticipant.getVideoStreams().get(0);
MediaStreamType streamType = remoteParticipantStream.getType(); // of type MediaStreamType.Video or MediaStreamType.ScreenSharing
```
 
Чтобы реализовать отрисовку `RemoteVideoStream`, полученного от удаленного участника, нужно подписаться на событие `OnVideoStreamsUpdated`.

В этом событии изменение значения свойства `isAvailable` на true указывает на то, что удаленный участник в настоящий момент отправляет поток. Обнаружив такое изменение, создайте новый экземпляр `Renderer`, а затем создайте новое представление `RendererView` с помощью асинхронного API `createView` и подключите `view.target` к любому элементу пользовательского интерфейса в приложении.

При каждом изменении состояния доступности удаленного потока вы можете удалить отрисовщик или определенное представление `RendererView` либо оставить все как есть, но это приведет к отрисовке пустого видеокадра.

```java
Renderer remoteVideoRenderer = new Renderer(remoteParticipantStream, appContext);
View uiView = remoteVideoRenderer.createView(new RenderingOptions(ScalingMode.Fit));
layout.addView(uiView);

remoteParticipant.addOnVideoStreamsUpdatedListener(e -> onRemoteParticipantVideoStreamsUpdated(p, e));

void onRemoteParticipantVideoStreamsUpdated(RemoteParticipant participant, RemoteVideoStreamsEvent args) {
    for(RemoteVideoStream stream : args.getAddedRemoteVideoStreams()) {
        if(stream.getIsAvailable()) {
            startRenderingVideo();
        } else {
            renderer.dispose();
        }
    }
}
```

### <a name="remote-video-stream-properties"></a>Свойства удаленного видеопотока
Удаленный видеопоток имеет несколько свойств:

* `Id` — идентификатор удаленного видеопотока;
```java
int id = remoteVideoStream.getId();
```

* `MediaStreamType` — принимает значения Video или ScreenSharing;
```java
MediaStreamType type = remoteVideoStream.getType();
```

* `isAvailable` — указывает на то, что конечная точка этого удаленного участника активно отправляет поток данных.
```java
boolean availability = remoteVideoStream.getIsAvailable();
```

### <a name="renderer-methods-and-properties"></a>Методы и свойства отрисовщика
Объект отрисовщика имеет следующие API:

* Создайте экземпляр `RendererView`, который позже можно будет подключить к пользовательскому интерфейсу приложения для отрисовке удаленного видеопотока.
```java
// Create a view for a video stream
renderer.createView()
```
* Удалите отрисовщик и все `RendererView`, связанные с ним. Вызывается при удалении всех связанных представлений из пользовательского интерфейса.
```java
renderer.dispose()
```

* `StreamSize` — размер (ширина и высота) удаленного видеопотока;
```java
StreamSize renderStreamSize = remoteVideoStream.getSize();
int width = renderStreamSize.getWidth();
int height = renderStreamSize.getHeight();
```


### <a name="rendererview-methods-and-properties"></a>Методы и свойства RendererView
При создании `RendererView` можно указать свойства `scalingMode` и `mirrored`, которые будут применяться к этому представлению. Режим масштабирования может принимать значения Stretch, Crop или Fit. Если `mirrored` имеет значение `true`, поток будет перевернут по вертикали.

```java
Renderer remoteVideoRenderer = new Renderer(remoteVideoStream, appContext);
RendererView rendererView = remoteVideoRenderer.createView(new RenderingOptions(ScalingMode.Fit));
```

Созданный RendererView можно затем подключить к пользовательскому интерфейсу приложения, используя следующий фрагмент кода:
```java
layout.addView(rendererView);
```

Вы можете позднее изменить режим масштабирования, вызвав API `updateScalingMode` для объекта RendererView с аргументом ScalingMode.Stretch, ScalingMode.Crop или ScalingMode.Fit.
```java
// Update the scale mode for this view.
rendererView.updateScalingMode(ScalingMode.Crop)
```


## <a name="device-management"></a>Управление устройствами

`DeviceManager` позволяет получить список локальных устройств, которые можно использовать в вызове для передачи аудио- и видеопотоков. Вы также можете запрашивать у пользователя разрешение на доступ к его микрофону и камере через собственный API браузера.

Доступ к `deviceManager` осуществляется путем вызова метода `callClient.getDeviceManager()`.
> [!WARNING]
> Сейчас для получения доступа к DeviceManager необходимо сначала создать экземпляр объекта `callAgent`.

```java
DeviceManager deviceManager = callClient.getDeviceManager().get();
```

### <a name="enumerate-local-devices"></a>Перечисление локальных устройств

Для доступа к локальным устройствам можно использовать методы перечисления из Диспетчера устройств. Действие перечисления выполняется асинхронно.

```java
//  Get a list of available video devices for use.
List<VideoDeviceInfo> localCameras = deviceManager.getCameras(); // [VideoDeviceInfo, VideoDeviceInfo...]

// Get a list of available microphone devices for use.
List<AudioDeviceInfo> localMicrophones = deviceManager.getMicrophones(); // [AudioDeviceInfo, AudioDeviceInfo...]

// Get a list of available speaker devices for use.
List<AudioDeviceInfo> localSpeakers = deviceManager.getSpeakers(); // [AudioDeviceInfo, AudioDeviceInfo...]
```

### <a name="set-default-microphonespeaker"></a>Назначение микрофона или динамика по умолчанию

Диспетчер устройств позволяет задать устройство по умолчанию, которое будет использоваться при осуществлении вызова.
Если значения по умолчанию для клиента не заданы, Службы коммуникации будут использовать значения по умолчанию для ОС.

```java

// Get the microphone device that is being used.
AudioDeviceInfo defaultMicrophone = deviceManager.getMicrophones().get(0);

// Set the microphone device to use.
deviceManager.setMicrophone(defaultMicrophone);

// Get the speaker device that is being used.
AudioDeviceInfo defaultSpeaker = deviceManager.getSpeakers().get(0);

// Set the speaker device to use.
deviceManager.setSpeaker(defaultSpeaker);
```

### <a name="local-camera-preview"></a>Предварительный просмотр изображения с локальной камеры

С помощью `DeviceManager` и `Renderer` вы можете реализовать отрисовку потоков из локальной камеры. Этот поток не отправляется другим участникам, а используется как локальный канал предварительного просмотра. Это действие выполняется асинхронно.

```java
VideoDeviceInfo videoDevice = <get-video-device>;
Context appContext = this.getApplicationContext();
currentVideoStream = new LocalVideoStream(videoDevice, appContext);
videoOptions = new VideoOptions(currentVideoStream);

Renderer previewRenderer = new Renderer(currentVideoStream, appContext);
View uiView = previewRenderer.createView(new RenderingOptions(ScalingMode.Fit));

// Attach the uiView to a viewable location on the app at this point
layout.addView(uiView);
```

## <a name="eventing-model"></a>Модель событий
Вы можете подписываться на большинство свойств и коллекций, чтобы получать уведомления при изменении их значений.

### <a name="properties"></a>Свойства
Чтобы подписаться на события `property changed`, выполните следующий код:

```java
// subscribe
PropertyChangedListener callStateChangeListener = new PropertyChangedListener()
{
    @Override
    public void onPropertyChanged(PropertyChangedEvent args)
    {
        Log.d("The call state has changed.");
    }
}
call.addOnStateChangedListener(callStateChangeListener);

//unsubscribe
call.removeOnStateChangedListener(callStateChangeListener);
```

### <a name="collections"></a>Коллекции
Чтобы подписаться на события `collection updated`, выполните следующий код:

```java
LocalVideoStreamsChangedListener localVideoStreamsChangedListener = new LocalVideoStreamsChangedListener()
{
    @Override
    public void onLocalVideoStreamsUpdated(LocalVideoStreamsEvent localVideoStreamsEventArgs) {
        Log.d(localVideoStreamsEventArgs.getAddedStreams().size());
        Log.d(localVideoStreamsEventArgs.getRemovedStreams().size());
    }
}
call.addOnLocalVideoStreamsChangedListener(localVideoStreamsChangedListener);
// To unsubscribe
call.removeOnLocalVideoStreamsChangedListener(localVideoStreamsChangedListener);
```
