---
title: Краткое руководство. Добавление функции вызова в приложение iOS с помощью Служб коммуникации Azure
description: Из этого краткого руководства вы узнаете, как использовать пакет SDK Служб коммуникации Azure для вызовов для iOS.
author: chpalm
ms.author: mikben
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 22c9d8f8bdf3e6195bf152fa0431ad5ce9bcdfeb
ms.sourcegitcommit: edc7dc50c4f5550d9776a4c42167a872032a4151
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "106073661"
---
Из этого краткого руководства вы узнаете, как начать вызов с помощью пакета SDK Служб коммуникации Azure для вызовов для iOS.

[!INCLUDE [Public Preview Notice](../../../includes/public-preview-include-android-ios.md)]

> [!NOTE]
> В этом документе используется пакет SDK для вызовов версии 1.0.0-beta.8.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно. 
- компьютер Mac с [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), а также действительный сертификат разработчика, установленный в цепочку ключей;
- Развернутый ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- [Маркер доступа пользователя](../../access-tokens.md) для Службы коммуникации Azure.

## <a name="setting-up"></a>Настройка

### <a name="creating-the-xcode-project"></a>Создание проекта Xcode

В Xcode создайте новый проект iOS и выберите шаблон **Single View App** (Приложение с одним представлением). В этом руководстве используется [платформа SwiftUI](https://developer.apple.com/xcode/swiftui/), поэтому для параметра **Language** (Язык) нужно задать значение **Swift**, а для параметра **User Interface** (Пользовательский интерфейс) — значение **SwiftUI**. В рамках этого краткого руководства вы не будете создавать тесты. Вы можете снять флажок **Include Tests** (Включить тесты).

:::image type="content" source="../media/ios/xcode-new-ios-project.png" alt-text="Снимок экрана с окном New Project (Новый проект) в Xcode.":::

### <a name="install-the-package-and-dependencies-with-cocoapods"></a>Установка пакета и его зависимостей с помощью CocoaPods

1. Чтобы создать Podfile для приложения, откройте терминал, перейдите в папку проекта и выполните команду ```pod init```.
3. Добавьте следующий код в Podfile и сохраните (убедитесь, что "target" соответствует имени проекта):

   ```
   platform :ios, '13.0'
   use_frameworks!

   target 'AzureCommunicationCallingSample' do
     pod 'AzureCommunicationCalling', '~> 1.0.0-beta.8'
     pod 'AzureCommunication', '~> 1.0.0-beta.8'
     pod 'AzureCore', '~> 1.0.0-beta.8'
   end
   ```

3. Выполните `pod install`.
3. Откройте файл `.xcworkspace` с помощью Xcode.

### <a name="request-access-to-the-microphone"></a>Запрос доступа к микрофону

Чтобы получить доступ к микрофону устройства, вам необходимо указать ключ `NSMicrophoneUsageDescription` в списке свойств сведений приложения. Задайте связанное значение для строки `string`, которая будет включена в диалоговое окно, отображаемое системой при запрашивании доступа у пользователя.

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

Замените реализацию структуры `ContentView` простыми элементами управления пользовательского интерфейса, которые позволяют начать и завершить вызов. В рамках этого краткого руководства к таким элементам управления мы присоединим бизнес-логику.

```swift
struct ContentView: View {
    @State var callee: String = ""
    @State var callClient: CallClient?
    @State var callAgent: CallAgent?
    @State var call: Call?

    var body: some View {
        NavigationView {
            Form {
                Section {
                    TextField("Who would you like to call?", text: $callee)
                    Button(action: startCall) {
                        Text("Start Call")
                    }.disabled(callAgent == nil)
                    Button(action: endCall) {
                        Text("End Call")
                    }.disabled(call == nil)
                }
            }
            .navigationBarTitle("Calling Quickstart")
        }.onAppear {
            // Initialize call agent
        }
    }

    func startCall() {
        // Ask permissions
        AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
            if granted {
                // Add start call logic
            }
        }
    }

    func endCall() {
        // Add end call logic
    }
}
```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK Служб коммуникации Azure для вызовов:

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| CallClient | CallClient — это основная точка входа в пакет SDK для вызовов.|
| CallAgent | CallAgent используется для инициирования вызовов и управления ими. |
| CommunicationTokenCredential | CommunicationTokenCredential используется в качестве учетных данных маркера для создания экземпляра CallAgent.| 
| CommunicationUserIdentifier | CommunicationUserIdentifier используется для представления идентификатора пользователя, который может иметь один из следующих типов: CommunicationUserIdentifier, PhoneNumberIdentifier или CallingApplication. |

## <a name="authenticate-the-client"></a>Аутентификация клиента

Инициализируйте экземпляр `CallAgent` с помощью маркера доступа пользователя, что позволит нам выполнять и принимать вызовы. Добавьте следующий код в обратный вызов `onAppear` в **ContentView.swift**:

```swift
var userCredential: CommunicationTokenCredential?
do {
    userCredential = try CommunicationTokenCredential(token: "<USER ACCESS TOKEN>")
} catch {
    print("ERROR: It was not possible to create user credential.")
    return
}

self.callClient = CallClient()

// Creates the call agent
self.callClient?.createCallAgent(userCredential: userCredential) { (agent, error) in
    if error != nil {
        print("ERROR: It was not possible to create a call agent.")
        return
    }

    if let agent = agent {
        self.callAgent = agent
        print("Call agent successfully created.")
    }
}
```

Вам необходимо заменить `<USER ACCESS TOKEN>` допустимым маркером доступа пользователя для вашего ресурса. Если у вас еще нет доступного маркера, см. документацию по [маркеру доступа пользователя](../../access-tokens.md).

## <a name="start-a-call"></a>Инициирование вызова

Метод `startCall` задан в качестве действия, которое будет выполняться при нажатии кнопки *Начать вызов*. Измените реализацию, чтобы вызов начинался с помощью `ASACallAgent`:

```swift
func startCall()
{
    // Ask permissions
    AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
        if granted {
            // start call logic
            let callees:[CommunicationIdentifier] = [CommunicationUserIdentifier(identifier: self.callee)]
            self.call = self.callAgent?.call(participants: callees, options: StartCallOptions())
        }
    }
}
```

Вы также можете использовать свойства в `StartCallOptions`, чтобы задать первоначальные параметры для вызова (т. е. можно начать вызов с отключенным микрофоном).

## <a name="end-a-call"></a>Завершение вызова

Реализуйте метод `endCall`, чтобы завершать текущий вызов при нажатии кнопки *Завершить вызов*.

```swift
func endCall()
{    
    self.call!.hangup(HangupOptions()) { (error) in
        if (error != nil) {
            print("ERROR: It was not possible to hangup the call.")
        }
    }
}
```

## <a name="run-the-code"></a>Выполнение кода

Вы можете создать и запустить свое приложение в симуляторе iOS, выбрав **Product** > **Run** (Продукт > Выполнить) или с помощью клавиш (&#8984;-R).

:::image type="content" source="../media/ios/quick-start-make-call.png" alt-text="Окончательный вид приложения из этого краткого руководства":::

Вы можете выполнить исходящий VOIP-вызов, указав идентификатор пользователя в текстовом поле и нажав кнопку **Начать вызов**. При вызове `8:echo123` вы будете подключены к эхо-боту, что позволит начать работу и проверить работоспособность ваших аудиоустройств. 

> [!NOTE]
> При первом вызове в системе отобразится запрос на получение доступа к микрофону. В приложении в рабочей среде используйте API `AVAudioSession` для [проверки состояния разрешения](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/requesting_access_to_protected_resources) и корректно обновите поведение приложения, если разрешение не предоставлено.

## <a name="sample-code"></a>Пример кода

Пример приложения можно скачать в репозитории [GitHub](https://github.com/Azure-Samples/communication-services-ios-quickstarts/tree/main/Add%20Voice%20Calling).
