---
title: Краткое руководство. Добавление возможности присоединения к собранию Teams к приложению iOS с помощью Служб коммуникации Azure
description: Из этого краткого руководства вы узнаете, как использовать библиотеку Teams Embed Служб коммуникации Azure для iOS.
author: palatter
ms.author: palatter
ms.date: 01/25/2021
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: 4d28864d41d6540afc87126daf589ed2929f891d
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104803888"
---
В этом кратком руководстве описывается, как присоединиться к собранию Teams с помощью библиотеки Teams Embed Служб коммуникации Azure для iOS.

## <a name="prerequisites"></a>Предварительные условия

Для работы с данным руководством необходимо следующее:

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно. 
- компьютер Mac с [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), а также действительный сертификат разработчика, установленный в цепочку ключей;
- Развернутый ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- [Маркер доступа пользователя](../../access-tokens.md) для Службы коммуникации Azure.
- [CocoaPods](https://cocoapods.org/) для среды сборки.

## <a name="setting-up"></a>Настройка

### <a name="creating-the-xcode-project"></a>Создание проекта Xcode

В Xcode создайте новый проект iOS и выберите шаблон **App** (Приложение). Мы будем использовать раскадровки UIKit. В рамках этого краткого руководства вы не будете создавать тесты. Вы можете снять флажок **Include Tests** (Включить тесты).

:::image type="content" source="../media/ios/xcode-new-project-template-select.png" alt-text="Снимок экрана с выбором шаблона New Project (Новый проект) в Xcode.":::

Задайте для проекта имя `TeamsEmbedGettingStarted`.

:::image type="content" source="../media/ios/xcode-new-project-details.png" alt-text="Снимок экрана с информацией о проекте New Project (Новый проект) в Xcode.":::

### <a name="install-the-package-and-dependencies-with-cocoapods"></a>Установка пакета и его зависимостей с помощью CocoaPods

1. Создайте файл Podfile для своего приложения:

```
platform :ios, '12.0'
use_frameworks!

target 'TeamsEmbedGettingStarted' do
    pod 'AzureCommunication', '~> 1.0.0-beta.8'
end

azure_libs = [
'AzureCommunication',
'AzureCore']

post_install do |installer|
    installer.pods_project.targets.each do |target|
    if azure_libs.include?(target.name)
        puts "Adding BUILD_LIBRARY_FOR_DISTRIBUTION to #{target.name}"
        target.build_configurations.each do |config|
        xcconfig_path = config.base_configuration_reference.real_path
        File.open(xcconfig_path, "a") {|file| file.puts "BUILD_LIBRARY_FOR_DISTRIBUTION = YES"}
        end
    end
    end
end
```

2. Выполните команду `pod install`.
3. Откройте созданный файл `.xcworkspace` с помощью Xcode.

### <a name="request-access-to-the-microphone-camera-etc"></a>Запрос доступа к микрофону, камере и т. д.

Чтобы получить доступ к оборудованию устройства, обновите список информационных свойств приложения. Задайте связанное значение для строки `string`, которая будет включена в диалоговое окно, отображаемое системой при запрашивании доступа у пользователя.

В дереве проекта щелкните правой кнопкой мыши запись `Info.plist` и выберите **Open As** > **Source Code** (Открыть как > Исходный код). Добавьте в раздел верхнего уровня `<dict>` следующие строки, а затем сохраните файл.

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string></string>
<key>NSBluetoothPeripheralUsageDescription</key>
<string></string>
<key>NSCameraUsageDescription</key>
<string></string>
<key>NSContactsUsageDescription</key>
<string></string>
<key>NSMicrophoneUsageDescription</key>
<string></string>
```

### <a name="add-the-teams-embed-framework"></a>Добавление платформы Teams Embed

1. Скачайте платформу.
2. Создайте папку `Frameworks` в корневом каталоге проекта. Например: `\TeamsEmbedGettingStarted\Frameworks\`
3. Скопируйте скачанные платформы `TeamsAppSDK.framework` и `MeetingUIClient.framework` в эту папку.
4. Добавьте `TeamsAppSDK.framework` и `MeetingUIClient.framework` в целевой объект проекта на вкладке "Общие". Используйте команды `Add Other`  ->  `Add Files...` для перехода к файлам платформы и их добавления.

:::image type="content" source="../media/ios/xcode-add-frameworks.png" alt-text="Снимок экрана, показывающий добавленные платформы в Xcode.":::

5. Добавьте `$(PROJECT_DIR)/Frameworks` в `Framework Search Paths` на вкладке параметров сборки целевого объекта проекта. Чтобы найти параметр, измените фильтр с `basic` на `all`, также можно использовать панель поиска справа.

:::image type="content" source="../media/ios/xcode-add-framework-search-path.png" alt-text="Снимок экрана, показывающий путь поиска платформы в Xcode.":::

### <a name="turn-off-bitcode"></a>Отключение bitcode

Установите для параметра `Enable Bitcode` значение `No` в параметрах сборки проекта. Чтобы найти параметр, измените фильтр с `basic` на `all`, также можно использовать панель поиска справа.

:::image type="content" source="../media/ios/xcode-bitcode-option.png" alt-text="Снимок экрана, показывающий параметр bitcode в Xcode.":::

### <a name="add-framework-signing-script"></a>Добавление скрипт подписывания платформы

Выберите целевой объект приложения и перейдите на вкладку `Build Phases`. Затем щелкните `+`, а затем — `New Run Script Phase`. Этот этап должен выполняться после этапов `Embed Frameworks`.



:::image type="content" source="../media/ios/xcode-build-script.png" alt-text="Снимок экрана, показывающий добавление скрипта сборки в Xcode.":::

```bash
#!/bin/sh
if [ -d "${TARGET_BUILD_DIR}"/"${PRODUCT_NAME}".app/Frameworks/TeamsAppSDK.framework/Frameworks ]; then
    pushd "${TARGET_BUILD_DIR}"/"${PRODUCT_NAME}".app/Frameworks/TeamsAppSDK.framework/Frameworks
    for EACH in *.framework; do
        echo "-- signing ${EACH}"
        /usr/bin/codesign --force --deep --sign "${EXPANDED_CODE_SIGN_IDENTITY}" --entitlements "${TARGET_TEMP_DIR}/${PRODUCT_NAME}.app.xcent" --timestamp=none $EACH
        echo "-- moving ${EACH}"
        mv -nv ${EACH} ../../
    done
    rm -rf "${TARGET_BUILD_DIR}"/"${PRODUCT_NAME}".app/Frameworks/TeamsAppSDK.framework/Frameworks
    popd
    echo "BUILD DIR ${TARGET_BUILD_DIR}"
fi
```

### <a name="turn-on-voice-over-ip-background-mode"></a>Включите фоновый режим голосовых вызовов через IP.

Выберите целевой объект приложения и перейдите на вкладку "Возможности".

Включите `Background Modes`, щелкнув `+ Capabilities` вверху и выбрав `Background Modes`.

Установите флажок `Voice over IP`.

:::image type="content" source="../media/ios/xcode-background-voip.png" alt-text="Снимок экрана, на котором показано включение VOIP в Xcode.":::

### <a name="add-a-window-reference-to-appdelegate"></a>Добавление ссылки на окно в AppDelegate

Откройте файл **AppDelegate. SWIFT** проекта и добавьте ссылку на объект window.

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
```

### <a name="add-a-button-to-the-viewcontroller"></a>Добавление кнопки в ViewController

Создайте кнопку в обратном вызове `viewDidLoad` в **ViewController. SWIFT**.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    
    let button = UIButton(frame: CGRect(x: 100, y: 100, width: 200, height: 50))
    button.backgroundColor = .black
    button.setTitle("Join Meeting", for: .normal)
    button.addTarget(self, action: #selector(joinMeetingTapped), for: .touchUpInside)
    
    button.translatesAutoresizingMaskIntoConstraints = false
    self.view.addSubview(button)
    button.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    button.centerYAnchor.constraint(equalTo: view.centerYAnchor).isActive = true
```

Создайте вывод для кнопки в **ViewController. SWIFT**.

```swift
@IBAction func joinMeetingTapped(_ sender: UIButton) {

}
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Откройте файл **ViewController.swift** проекта и добавьте объявление `import` в начало файла, чтобы импортировать `AzureCommunication library` и `MeetingUIClient`. 

```swift
import UIKit
import AzureCommunication
import MeetingUIClient
```

Замените реализацию класса `ViewController` кнопкой, с помощью которой пользователи будут присоединяться к собранию. В рамках этого краткого руководства мы присоединим бизнес-логику к этой кнопке.

```swift
class ViewController: UIViewController {

    private var meetingUIClient: MeetingUIClient?
    
    override func viewDidLoad() {
        super.viewDidLoad()

        // Initialize meetingClient
    }

    @IBAction func joinMeetingTapped(_ sender: UIButton) {
        joinMeeting()
    }
    
    private func joinMeeting() {
        // Add join meeting logic
    }
}
```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции библиотеки Teams Embed Служб коммуникации Azure:

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| MeetingUIClient | MeetingUIClient — это главная точка входа в библиотеку Teams Embed. |
| MeetingUIClientDelegate | MeetingUIClientDelegate используется для получения событий, например изменений в состоянии вызова. |
| MeetingJoinOptions | MeetingJoinOptions используются для настраиваемых параметров, таких как отображаемое имя. | 
| CallState | CallState используется для создания отчетов об изменении состояния вызова. Возможны следующие варианты: connecting, waitingInLobby, connected и ended. |

## <a name="create-and-authenticate-the-client"></a>Создание и проверка подлинности клиента

Инициализируйте экземпляр `MeetingUIClient` с помощью маркера доступа пользователя, что позволит нам присоединяться к встречам. Добавьте следующий код в обратный вызов `viewDidLoad` в **ViewController.swift**:

```swift
do {
    let communicationTokenRefreshOptions = CommunicationTokenRefreshOptions(initialToken: "<USER_ACCESS_TOKEN>", refreshProactively: true, tokenRefresher: fetchTokenAsync(completionHandler:))
    let credential = try CommunicationTokenCredential(with: communicationTokenRefreshOptions)
    meetingUIClient = MeetingUIClient(with: credential)
}
catch {
    print("Failed to create communication token credential")
}
```

Замените `<USER_ACCESS_TOKEN>` допустимым маркером доступа пользователя для вашего ресурса. Если у вас еще нет доступного маркера, см. документацию по [маркеру доступа пользователя](../../access-tokens.md).

### <a name="setup-token-refreshing"></a>Обновление маркера установки

Создайте метод `fetchTokenAsync`. Затем добавьте логику `fetchToken`, чтобы получить маркер пользователя.

```swift
private func fetchTokenAsync(completionHandler: @escaping TokenRefreshOnCompletion) {
    func getTokenFromServer(completionHandler: @escaping (String) -> Void) {
        completionHandler("<USER_ACCESS_TOKEN>")
    }
    getTokenFromServer { newToken in
        completionHandler(newToken, nil)
    }
}
```

Замените `<USER_ACCESS_TOKEN>` допустимым маркером доступа пользователя для вашего ресурса.

## <a name="join-a-meeting"></a>Присоединиться к собранию

Метод `joinMeeting` задан в качестве действия, которое будет выполняться при нажатии кнопки *Присоединиться к собранию*. Измените реализацию, чтобы к собранию можно было присоединиться с помощью `MeetingUIClient`:

```swift
private func joinMeeting() {
    let meetingJoinOptions = MeetingJoinOptions(displayName: "John Smith")
        
    meetingUIClient?.join(meetingUrl: "<MEETING_URL>", meetingJoinOptions: meetingJoinOptions, completionHandler: { (error: Error?) in
        if (error != nil) {
            print("Join meeting failed: \(error!)")
        }
    })
}
```

Замените `<MEETING URL>` на ссылку на собрание в Teams.

### <a name="get-a-teams-meeting-link"></a>Получение ссылки на собрание Teams

Ссылку на собрание Teams можно получить с помощью интерфейсов API Graph. Это действие подробно описано в [документации по Graph](/graph/api/onlinemeeting-createorget?tabs=http&view=graph-rest-beta&preserve-view=true).
Пакет SDK вызовов Служб коммуникации принимает полную ссылку на собрание Teams. Эта ссылка возвращается как часть ресурса `onlineMeeting`, доступного в [свойстве `joinWebUrl`](/graph/api/resources/onlinemeeting?view=graph-rest-beta&preserve-view=true). Кроме того, вы можете получить необходимые сведения о собрании, воспользовавшись URL-адресом для **участия в собрании** в самом приглашении на собрание Teams.

## <a name="run-the-code"></a>Выполнение кода

Вы можете создать и запустить свое приложение в симуляторе iOS, выбрав **Product** > **Run** (Продукт > Выполнить) или с помощью клавиш (&#8984;-R).

:::image type="content" source="../media/ios/quick-start-join-meeting.png" alt-text="Окончательный вид приложения из этого краткого руководства":::

:::image type="content" source="../media/ios/quick-start-meeting-joined.png" alt-text="Окончательный вид после присоединения к собранию":::

> [!NOTE]
> При первом присоединении к собранию в системе отобразится запрос на получение доступа к микрофону. В приложении в рабочей среде используйте API `AVAudioSession` для [проверки состояния разрешения](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/requesting_access_to_protected_resources) и корректно обновите поведение приложения, если разрешение не предоставлено.

## <a name="add-localization-support-based-on-your-app"></a>Добавление поддержки локализации на основе приложения

Пакет SDK для Microsoft Teams поддерживает свыше 100 строк и ресурсов. Набор платформы содержит базовый и английский языки. Остальные из них входят в файл `Localizations.zip`, входящий в состав пакета.

#### <a name="add-localizations-to-the-sdk-based-on-what-your-app-supports"></a>Добавление локализаций в пакет SDK в зависимости от того, что поддерживает ваше приложение

1. Проверьте, какие виды локализации ваше приложение поддерживает. Для этого откройте в Xcode список локализаций (Project > Info > Localizations)
2. Распакуйте файл Localizations.zip, прилагаемый к пакету
3. Скопируйте нужные папки локализации из распакованной папки и переместите их в корневой каталог платформы TeamsAppSDK.framework

## <a name="preparation-for-app-store-upload"></a>Подготовка к отправке в App Store

Если вы планируете архивировать проект, удалите из платформы архитектуры i386 и x86_64.

Для этого добавьте скрипт, удаляющий архитектуры `i386` и `x86_64`, на этапе сборки до этапа совместного проектирования платформы.

В навигаторе проектов выберите свой проект. В области редактирования перейдите к разделу Build Phases (Этапы сборки), щелкните знак + и выберите Create a New Run Script Phase (Создать новый этап выполнения скрипта).

```bash
echo "Target architectures: $ARCHS"
APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"
find "$APP_PATH" -name '*.framework' -type d | while read -r FRAMEWORK
do
FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"
echo $(lipo -info "$FRAMEWORK_EXECUTABLE_PATH")
FRAMEWORK_TMP_PATH="$FRAMEWORK_EXECUTABLE_PATH-tmp"
# remove simulator's archs if location is not simulator's directory
case "${TARGET_BUILD_DIR}" in
*"iphonesimulator")
    echo "No need to remove archs"
    ;;
*)
    if $(lipo "$FRAMEWORK_EXECUTABLE_PATH" -verify_arch "i386") ; then
    lipo -output "$FRAMEWORK_TMP_PATH" -remove "i386" "$FRAMEWORK_EXECUTABLE_PATH"
    echo "i386 architecture removed"
    rm "$FRAMEWORK_EXECUTABLE_PATH"
    mv "$FRAMEWORK_TMP_PATH" "$FRAMEWORK_EXECUTABLE_PATH"
    fi
    if $(lipo "$FRAMEWORK_EXECUTABLE_PATH" -verify_arch "x86_64") ; then
    lipo -output "$FRAMEWORK_TMP_PATH" -remove "x86_64" "$FRAMEWORK_EXECUTABLE_PATH"
    echo "x86_64 architecture removed"
    rm "$FRAMEWORK_EXECUTABLE_PATH"
    mv "$FRAMEWORK_TMP_PATH" "$FRAMEWORK_EXECUTABLE_PATH"
    fi
    ;;
esac
echo "Completed for executable $FRAMEWORK_EXECUTABLE_PATH"
echo $(lipo -info "$FRAMEWORK_EXECUTABLE_PATH")
done
```

## <a name="sample-code"></a>Пример кода

Пример приложения можно скачать в репозитории [GitHub](https://github.com/Azure-Samples/teams-embed-ios-getting-started).
