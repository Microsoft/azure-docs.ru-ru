---
title: Руководство по Создание веб-приложения iOS или macOS, которое использует платформу удостоверений Майкрософт для аутентификации | Azure
titleSuffix: Microsoft identity platform
description: В этом учебнике показано, как создать приложение iOS или macOS, которое использует платформу удостоверений Майкрософт для реализации входа пользователей, и как получить маркер доступа для вызова API Microsoft Graph от имени пользователей.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 09/18/2020
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: e273b1c41a9c418bf0121e87e73ec59e65f2242e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100102960"
---
# <a name="tutorial-sign-in-users-and-call-microsoft-graph-from-an-ios-or-macos-app"></a>Руководство по Вход пользователей и вызов Microsoft Graph из приложения iOS или macOS

В этом руководстве показано, как создать приложение iOS или macOS, которое интегрируется с платформой удостоверений Майкрософт для реализации входа пользователей, и как получить маркер доступа для вызова API Microsoft Graph.

Когда вы завершите работу с этим руководством, ваше приложение сможет принимать операции входа с помощью личных учетных записей Майкрософт (включая outlook.com, live.com и другие), а также рабочих или учебных учетных записей из любой компании или организации, в которых используется Azure Active Directory. Этот учебник применим к приложениям iOS и macOS. Некоторые шаги для этих двух платформ отличаются.

В этом руководстве рассматриваются следующие темы:

> [!div class="checklist"]
> * создание проекта приложения для iOS или macOS в *Xcode*;
> * регистрация приложения на портале Azure;
> * добавление кода для поддержки входа и выхода пользователей;
> * добавление кода для вызова API Microsoft Graph.
> * Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

- [Xcode 11.x +](https://developer.apple.com/xcode/)

## <a name="how-tutorial-app-works"></a>Принципы работы приложения из учебника

![Схема работы примера приложения, создаваемого в этом кратком руководстве](../../../includes/media/active-directory-develop-guidedsetup-ios-introduction/iosintro.svg)

В этом учебнике приложение может авторизовать пользователя и получать данные из Microsoft Graph от его имени. Доступ к этим данным будет осуществляться через защищенный API (в данном случае API Microsoft Graph), который требует авторизации и защищен платформой удостоверений Майкрософт.

В частности:

* Приложение авторизует пользователя через браузер или Microsoft Authenticator.
* Пользователь принимает разрешения, которые запрашивает приложение.
* Приложение выдает маркер доступа для API Microsoft Graph.
* Этот маркер доступа будет включен в HTTP-запрос к веб-API.
* Затем обрабатывается ответ Microsoft Graph.

В этом примере используется библиотека аутентификации Майкрософт (MSAL) для проведения проверки подлинности. MSAL автоматически обновляет маркеры, обеспечивает единый вход для других приложений на устройстве, а также управляет учетными записями.

Если вы хотите скачать готовую версию приложения, созданного в этом руководстве, обе версии можно найти в GitHub:

- [Пример кода iOS](https://github.com/Azure-Samples/active-directory-ios-swift-native-v2/) (GitHub)
- [Пример кода macOS](https://github.com/Azure-Samples/active-directory-macOS-swift-native-v2/) (GitHub)

## <a name="create-a-new-project"></a>Создание нового проекта

1. Запустите Xcode и щелкните **Create a new Xcode project** (Создать новый проект Xcode).
2. Для приложений iOS выберите **iOS** > **Приложение с одним представлением**, а затем **Далее**.
3. Для приложений macOS выберите **macOS** > **Cocoa App**, а затем **Далее**.
4. Укажите название продукта.
5. Установите для параметра **Язык** значение **Swift** и выберите **Далее**.
6. Выберите папку для создания приложения и нажмите кнопку **Создать**.

## <a name="register-your-application"></a>Регистрация приложения

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
1. Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
1. Введите значение **Name** (Имя) для приложения. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
1. В разделе **Поддерживаемые типы учетных записей** выберите **Учетные записи в любом каталоге организации (любой каталог Azure AD — мультитенантный) и персональные учетные записи Майкрософт (например, Skype, Xbox)** .
1. Выберите **Зарегистрировать**.
1. В разделе **Управление** выберите **Проверка подлинности** > **Добавить платформу** > **iOS/macOS**.
1. Введите идентификатор пакета проекта. Если вы скачали код, это будет `com.microsoft.identitysample.MSALiOS`. Если вы создаете собственный проект, выберите свой проект в Xcode и откройте вкладку **Общие сведения**. Идентификатор пакета появляется в разделе **Удостоверение**.
1. Щелкните **Настройка** и сохраните **конфигурацию MSAL**, отображаемую на странице **Конфигурация MSAL**, чтобы указать ее при последующей настройке приложения. 
1. Нажмите кнопку **Готово**.

## <a name="add-msal"></a>Добавление MSAL

Выберите один из следующих способов установки библиотеки MSAL в приложении:

### <a name="cocoapods"></a>CocoaPods

1. Если вы используете [CocoaPods](https://cocoapods.org/), установите `MSAL`, сначала создав пустой файл `podfile` в папке проекта `.xcodeproj`. Добавьте следующую строку в файл `podfile`:

   ```
   use_frameworks!

   target '<your-target-here>' do
      pod 'MSAL'
   end
   ```

2. Замените `<your-target-here>` именем проекта.
3. В окне терминала перейдите в папку, где находится `podfile`, и запустите `pod install`, чтобы установить библиотеку MSAL.
4. Чтобы перезагрузить проект в XCode закройте его и откройте `<your project name>.xcworkspace`.

### <a name="carthage"></a>Carthage

Если вы используете [Carthage](https://github.com/Carthage/Carthage), установите библиотеку `MSAL`, добавив ее в `Cartfile`:

```
github "AzureAD/microsoft-authentication-library-for-objc" "master"
```

В окне терминала в том же каталоге, в котором находится обновленный `Cartfile`, выполните следующую команду для обновления Carthage зависимостей в проекте.

iOS:

```bash
carthage update --platform iOS
```

macOS:

```bash
carthage update --platform macOS
```

### <a name="manually"></a>Вручную

Установить библиотеку также можно с помощью подмодуля Git. Кроме того, вы можете извлечь последний выпуск чтобы использовать его в качестве платформы в своем приложении.

## <a name="add-your-app-registration"></a>Добавление регистрации приложения

Затем добавьте регистрацию приложения в код.

Сначала добавьте в начало файлов `ViewController.swift`, `AppDelegate.swift` или `SceneDelegate.swift` следующие инструкции import:

```swift
import MSAL
```

Затем добавьте следующий код в `ViewController.swift` до `viewDidLoad()`.

```swift
// Update the below to your client ID you received in the portal. The below is for running the demo only
let kClientID = "Your_Application_Id_Here"
let kGraphEndpoint = "https://graph.microsoft.com/" // the Microsoft Graph endpoint
let kAuthority = "https://login.microsoftonline.com/common" // this authority allows a personal Microsoft account and a work or school account in any organization's Azure AD tenant to sign in

let kScopes: [String] = ["user.read"] // request permission to read the profile of the signed-in user

var accessToken = String()
var applicationContext : MSALPublicClientApplication?
var webViewParameters : MSALWebviewParameters?
var currentAccount: MSALAccount?
```

Единственное значение выше, которое необходимо изменить, — это значение, присваиваемое `kClientID` в качестве [идентификатора приложения](./developer-glossary.md#application-id-client-id). Это значение является частью данных конфигурации MSAL, сохраненного на шаге в начале этого руководства для регистрации приложения на портале Azure.

## <a name="configure-xcode-project-settings"></a>Настройка параметров проекта Xcode

Добавьте новую группу цепочки ключей в раздел **Подписывание и возможности** для проекта. Группа цепочки ключей должна иметь значение `com.microsoft.adalcache` в iOS и `com.microsoft.identity.universalstorage` в macOS.

![Пользовательский интерфейс Xcode с правильной настройкой группы цепочки ключей](../../../includes/media/active-directory-develop-guidedsetup-ios-introduction/iosintro-keychainShare.png)

## <a name="for-ios-only-configure-url-schemes"></a>(Только для iOS) Настройка схем URL-адресов

На этом шаге вы зарегистрируете `CFBundleURLSchemes`, чтобы перенаправить пользователя обратно в приложение после входа в систему. Кстати, `LSApplicationQueriesSchemes` также позволяет вашему приложению использовать Microsoft Authenticator.

В Xcode откройте `Info.plist` как файл исходного кода и добавьте следующее в раздел `<dict>` Замените `[BUNDLE_ID]` значением, использованным на портале Azure. Если вы загрузили код, идентификатор пакета будет `com.microsoft.identitysample.MSALiOS`. Если вы создаете собственный проект, выберите свой проект в Xcode и откройте вкладку **Общие сведения**. Идентификатор пакета появляется в разделе **Удостоверение**.

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>msauth.[BUNDLE_ID]</string>
        </array>
    </dict>
</array>
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>msauthv2</string>
    <string>msauthv3</string>
</array>
```

## <a name="for-macos-only-configure-app-sandbox"></a>(Только для macOS) Настройка песочницы приложения

1. Перейдите к параметрам проекта Xcode, а затем выберите **вкладку "Возможности"**  > **Песочница приложения**
2. Выберите флажок **Исходящие подключения (Клиент)** .

## <a name="create-your-apps-ui"></a>Создание пользовательского интерфейса для приложения

Теперь создайте пользовательский интерфейс, в который включены кнопки для вызова Microsoft Graph API, для выхода и для текстового представления для просмотра некоторых результатов, добавив следующий код в класс `ViewController`:

### <a name="ios-ui"></a>Пользовательский интерфейс iOS

```swift
var loggingText: UITextView!
var signOutButton: UIButton!
var callGraphButton: UIButton!
var usernameLabel: UILabel!

func initUI() {

    usernameLabel = UILabel()
    usernameLabel.translatesAutoresizingMaskIntoConstraints = false
    usernameLabel.text = ""
    usernameLabel.textColor = .darkGray
    usernameLabel.textAlignment = .right

    self.view.addSubview(usernameLabel)

    usernameLabel.topAnchor.constraint(equalTo: view.topAnchor, constant: 50.0).isActive = true
    usernameLabel.rightAnchor.constraint(equalTo: view.rightAnchor, constant: -10.0).isActive = true
    usernameLabel.widthAnchor.constraint(equalToConstant: 300.0).isActive = true
    usernameLabel.heightAnchor.constraint(equalToConstant: 50.0).isActive = true

    // Add call Graph button
    callGraphButton  = UIButton()
    callGraphButton.translatesAutoresizingMaskIntoConstraints = false
    callGraphButton.setTitle("Call Microsoft Graph API", for: .normal)
    callGraphButton.setTitleColor(.blue, for: .normal)
    callGraphButton.addTarget(self, action: #selector(callGraphAPI(_:)), for: .touchUpInside)
    self.view.addSubview(callGraphButton)

    callGraphButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    callGraphButton.topAnchor.constraint(equalTo: view.topAnchor, constant: 120.0).isActive = true
    callGraphButton.widthAnchor.constraint(equalToConstant: 300.0).isActive = true
    callGraphButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true

    // Add sign out button
    signOutButton = UIButton()
    signOutButton.translatesAutoresizingMaskIntoConstraints = false
    signOutButton.setTitle("Sign Out", for: .normal)
    signOutButton.setTitleColor(.blue, for: .normal)
    signOutButton.setTitleColor(.gray, for: .disabled)
    signOutButton.addTarget(self, action: #selector(signOut(_:)), for: .touchUpInside)
    self.view.addSubview(signOutButton)

    signOutButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    signOutButton.topAnchor.constraint(equalTo: callGraphButton.bottomAnchor, constant: 10.0).isActive = true
    signOutButton.widthAnchor.constraint(equalToConstant: 150.0).isActive = true
    signOutButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true

    let deviceModeButton = UIButton()
    deviceModeButton.translatesAutoresizingMaskIntoConstraints = false
    deviceModeButton.setTitle("Get device info", for: .normal);
    deviceModeButton.setTitleColor(.blue, for: .normal);
    deviceModeButton.addTarget(self, action: #selector(getDeviceMode(_:)), for: .touchUpInside)
    self.view.addSubview(deviceModeButton)

    deviceModeButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    deviceModeButton.topAnchor.constraint(equalTo: signOutButton.bottomAnchor, constant: 10.0).isActive = true
    deviceModeButton.widthAnchor.constraint(equalToConstant: 150.0).isActive = true
    deviceModeButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true

    // Add logging textfield
    loggingText = UITextView()
    loggingText.isUserInteractionEnabled = false
    loggingText.translatesAutoresizingMaskIntoConstraints = false

    self.view.addSubview(loggingText)

    loggingText.topAnchor.constraint(equalTo: deviceModeButton.bottomAnchor, constant: 10.0).isActive = true
    loggingText.leftAnchor.constraint(equalTo: self.view.leftAnchor, constant: 10.0).isActive = true
    loggingText.rightAnchor.constraint(equalTo: self.view.rightAnchor, constant: -10.0).isActive = true
    loggingText.bottomAnchor.constraint(equalTo: self.view.bottomAnchor, constant: 10.0).isActive = true
}

func platformViewDidLoadSetup() {

    NotificationCenter.default.addObserver(self,
                        selector: #selector(appCameToForeGround(notification:)),
                        name: UIApplication.willEnterForegroundNotification,
                        object: nil)

}

@objc func appCameToForeGround(notification: Notification) {
    self.loadCurrentAccount()
}

```

### <a name="macos-ui"></a>Пользовательский интерфейс macOS

```swift

var callGraphButton: NSButton!
var loggingText: NSTextView!
var signOutButton: NSButton!

var usernameLabel: NSTextField!

func initUI() {

    usernameLabel = NSTextField()
    usernameLabel.translatesAutoresizingMaskIntoConstraints = false
    usernameLabel.stringValue = ""
    usernameLabel.isEditable = false
    usernameLabel.isBezeled = false
    self.view.addSubview(usernameLabel)

    usernameLabel.topAnchor.constraint(equalTo: view.topAnchor, constant: 30.0).isActive = true
    usernameLabel.rightAnchor.constraint(equalTo: view.rightAnchor, constant: -10.0).isActive = true

    // Add call Graph button
    callGraphButton  = NSButton()
    callGraphButton.translatesAutoresizingMaskIntoConstraints = false
    callGraphButton.title = "Call Microsoft Graph API"
    callGraphButton.target = self
    callGraphButton.action = #selector(callGraphAPI(_:))
    callGraphButton.bezelStyle = .rounded
    self.view.addSubview(callGraphButton)

    callGraphButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    callGraphButton.topAnchor.constraint(equalTo: view.topAnchor, constant: 50.0).isActive = true
    callGraphButton.heightAnchor.constraint(equalToConstant: 34.0).isActive = true

    // Add sign out button
    signOutButton = NSButton()
    signOutButton.translatesAutoresizingMaskIntoConstraints = false
    signOutButton.title = "Sign Out"
    signOutButton.target = self
    signOutButton.action = #selector(signOut(_:))
    signOutButton.bezelStyle = .texturedRounded
    self.view.addSubview(signOutButton)

    signOutButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    signOutButton.topAnchor.constraint(equalTo: callGraphButton.bottomAnchor, constant: 10.0).isActive = true
    signOutButton.heightAnchor.constraint(equalToConstant: 34.0).isActive = true
    signOutButton.isEnabled = false

    // Add logging textfield
    loggingText = NSTextView()
    loggingText.translatesAutoresizingMaskIntoConstraints = false

    self.view.addSubview(loggingText)

    loggingText.topAnchor.constraint(equalTo: signOutButton.bottomAnchor, constant: 10.0).isActive = true
    loggingText.leftAnchor.constraint(equalTo: self.view.leftAnchor, constant: 10.0).isActive = true
    loggingText.rightAnchor.constraint(equalTo: self.view.rightAnchor, constant: -10.0).isActive = true
    loggingText.bottomAnchor.constraint(equalTo: self.view.bottomAnchor, constant: -10.0).isActive = true
    loggingText.widthAnchor.constraint(equalToConstant: 500.0).isActive = true
    loggingText.heightAnchor.constraint(equalToConstant: 300.0).isActive = true
}

func platformViewDidLoadSetup() {}

```

Затем, также внутри класса `ViewController` следует заменить метод `viewDidLoad()` на:

```swift
    override func viewDidLoad() {

        super.viewDidLoad()

        initUI()

        do {
            try self.initMSAL()
        } catch let error {
            self.updateLogging(text: "Unable to create Application Context \(error)")
        }

        self.loadCurrentAccount()
        self.platformViewDidLoadSetup()
    }
```

## <a name="use-msal"></a>Использование MSAL

### <a name="initialize-msal"></a>Инициализация MSAL

Добавьте в класс `ViewController` метод `initMSAL`.

```swift
    func initMSAL() throws {

        guard let authorityURL = URL(string: kAuthority) else {
            self.updateLogging(text: "Unable to create authority URL")
            return
        }

        let authority = try MSALAADAuthority(url: authorityURL)

        let msalConfiguration = MSALPublicClientApplicationConfig(clientId: kClientID, redirectUri: nil, authority: authority)
        self.applicationContext = try MSALPublicClientApplication(configuration: msalConfiguration)
        self.initWebViewParams()
    }
```

Добавьте следующее после метода `initMSAL` в класс `ViewController`.

### <a name="ios-code"></a>код iOS:

```swift
func initWebViewParams() {
        self.webViewParameters = MSALWebviewParameters(authPresentationViewController: self)
    }
```

### <a name="macos-code"></a>код macOS:

```swift
func initWebViewParams() {
        self.webViewParameters = MSALWebviewParameters()
    }
```

### <a name="for-ios-only-handle-the-sign-in-callback"></a>(Только для iOS) Работа с обратным вызовом входа

Откройте файл `AppDelegate.swift` . Чтобы обработать обратный вызов после входа, добавьте `MSALPublicClientApplication.handleMSALResponse` в класс `appDelegate`, например:

```swift
// Inside AppDelegate...
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
}

```

**Если вы используете Xcode 11**, то вместо этого в `SceneDelegate.swift` следует поместить обратный вызов MSAL.
Если UISceneDelegate и UIApplicationDelegate поддерживаются для совместимости с более старыми версиями iOS, то обратный вызов MSAL потребуется поместить в оба файла.

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {

        guard let urlContext = URLContexts.first else {
            return
        }

        let url = urlContext.url
        let sourceApp = urlContext.options.sourceApplication

        MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApp)
    }
```

#### <a name="acquire-tokens"></a>Получение маркеров

Теперь мы можем реализовать логику обработки пользовательского интерфейса приложения и интерактивно получать маркеры через MSAL.

MSAL предоставляет два основных метода получения маркеров: `acquireTokenSilently()` и `acquireTokenInteractively()`.

- `acquireTokenSilently()` пытается выполнить вход и получить маркеры без какого-либо взаимодействия с пользователем пока учетная запись существует. Для `acquireTokenSilently()` требуется предоставить допустимое значение `MSALAccount`, которое можно получить с помощью одного из API перечисления учетной записи MSAL. В этом примере для получения текущей учетной записи используется `applicationContext.getCurrentAccount(with: msalParameters, completionBlock: {})`.

- Метод `acquireTokenInteractively()` всегда отображает пользовательский интерфейс при попытке входа пользователя. Он может использовать файлы cookie сеанса в браузере или учетную запись в Microsoft Authenticator для реализации интерактивного единого входа.

Добавьте в класс `ViewController` приведенный ниже код.

```swift
    func getGraphEndpoint() -> String {
        return kGraphEndpoint.hasSuffix("/") ? (kGraphEndpoint + "v1.0/me/") : (kGraphEndpoint + "/v1.0/me/");
    }

    @objc func callGraphAPI(_ sender: AnyObject) {

        self.loadCurrentAccount { (account) in

            guard let currentAccount = account else {

                // We check to see if we have a current logged in account.
                // If we don't, then we need to sign someone in.
                self.acquireTokenInteractively()
                return
            }

            self.acquireTokenSilently(currentAccount)
        }
    }

    typealias AccountCompletion = (MSALAccount?) -> Void

    func loadCurrentAccount(completion: AccountCompletion? = nil) {

        guard let applicationContext = self.applicationContext else { return }

        let msalParameters = MSALParameters()
        msalParameters.completionBlockQueue = DispatchQueue.main

        applicationContext.getCurrentAccount(with: msalParameters, completionBlock: { (currentAccount, previousAccount, error) in

            if let error = error {
                self.updateLogging(text: "Couldn't query current account with error: \(error)")
                return
            }

            if let currentAccount = currentAccount {

                self.updateLogging(text: "Found a signed in account \(String(describing: currentAccount.username)). Updating data for that account...")

                self.updateCurrentAccount(account: currentAccount)

                if let completion = completion {
                    completion(self.currentAccount)
                }

                return
            }

            self.updateLogging(text: "Account signed out. Updating UX")
            self.accessToken = ""
            self.updateCurrentAccount(account: nil)

            if let completion = completion {
                completion(nil)
            }
        })
    }
```

#### <a name="get-a-token-interactively"></a>Интерактивное получение маркера

Приведенный ниже фрагмент кода первый раз возвращает маркер путем создания объекта `MSALInteractiveTokenParameters` и вызова `acquireToken`. Далее будет добавлен код, который делает следующее.

1. Создает `MSALInteractiveTokenParameters` с областями.
2. Вызывает `acquireToken()` с созданными параметрами.
3. Обрабатывает ошибки. Дополнительные сведения см. в [руководстве по обработке ошибок MSAL для iOS и macOS](msal-error-handling-ios.md).
4. Обрабатывает успешные выходные данные.

Добавьте в класс `ViewController` приведенный далее код.

```swift
func acquireTokenInteractively() {

    guard let applicationContext = self.applicationContext else { return }
    guard let webViewParameters = self.webViewParameters else { return }

    // #1
    let parameters = MSALInteractiveTokenParameters(scopes: kScopes, webviewParameters: webViewParameters)
    parameters.promptType = .selectAccount

    // #2
    applicationContext.acquireToken(with: parameters) { (result, error) in

        // #3
        if let error = error {

            self.updateLogging(text: "Could not acquire token: \(error)")
            return
        }

        guard let result = result else {

            self.updateLogging(text: "Could not acquire token: No result returned")
            return
        }

        // #4
        self.accessToken = result.accessToken
        self.updateLogging(text: "Access token is \(self.accessToken)")
        self.updateCurrentAccount(account: result.account)
        self.getContentWithToken()
    }
}
```

Свойство `promptType` объекта `MSALInteractiveTokenParameters` настраивает поведение запроса аутентификации и согласия. Поддерживаются следующие значения.

- `.promptIfNecessary` (по умолчанию) — пользователю предлагается только при необходимости. Процесс единого входа определяется наличием файлов cookie в веб-представлении и типом учетной записи. При входе в систему нескольких пользователей отображается возможность выбора учетной записи. *Это поведение по умолчанию*.
- `.selectAccount` — если пользователь не указан, в веб-представлении аутентификации отображается список текущих учетных записей, выполнивших вход, для выбора пользователем.
- `.login` — требует, чтобы пользователь прошел аутентификацию в веб-представлении. Если указано это значение, то одновременно может быть выполнен вход только одной учетной записи.
- `.consent` — требует, чтобы пользователь предоставил согласие на текущий набор областей для запроса.

#### <a name="get-a-token-silently"></a>Автоматическое получение маркера

Чтобы получить обновленный маркер без вывода сообщений, добавьте следующий код в класс `ViewController`. Он создает `MSALSilentTokenParameters` объект и вызывает `acquireTokenSilent()`:

```swift

    func acquireTokenSilently(_ account : MSALAccount!) {

        guard let applicationContext = self.applicationContext else { return }

        /**

         Acquire a token for an existing account silently

         - forScopes:           Permissions you want included in the access token received
         in the result in the completionBlock. Not all scopes are
         guaranteed to be included in the access token returned.
         - account:             An account object that we retrieved from the application object before that the
         authentication flow will be locked down to.
         - completionBlock:     The completion block that will be called when the authentication
         flow completes, or encounters an error.
         */

        let parameters = MSALSilentTokenParameters(scopes: kScopes, account: account)

        applicationContext.acquireTokenSilent(with: parameters) { (result, error) in

            if let error = error {

                let nsError = error as NSError

                // interactionRequired means we need to ask the user to sign-in. This usually happens
                // when the user's Refresh Token is expired or if the user has changed their password
                // among other possible reasons.

                if (nsError.domain == MSALErrorDomain) {

                    if (nsError.code == MSALError.interactionRequired.rawValue) {

                        DispatchQueue.main.async {
                            self.acquireTokenInteractively()
                        }
                        return
                    }
                }

                self.updateLogging(text: "Could not acquire token silently: \(error)")
                return
            }

            guard let result = result else {

                self.updateLogging(text: "Could not acquire token: No result returned")
                return
            }

            self.accessToken = result.accessToken
            self.updateLogging(text: "Refreshed Access token is \(self.accessToken)")
            self.updateSignOutButton(enabled: true)
            self.getContentWithToken()
        }
    }
```

### <a name="call-the-microsoft-graph-api"></a>Вызов API Microsoft Graph

После получения маркера приложение может его использовать в заголовке HTTP, чтобы отправить авторизованный запрос к Microsoft Graph:

| ключ заголовка    | value                 |
| ------------- | --------------------- |
| Авторизация | Маркер носителя \<access-token> |

Добавьте в класс `ViewController` приведенный ниже код.

```swift
    func getContentWithToken() {

        // Specify the Graph API endpoint
        let graphURI = getGraphEndpoint()
        let url = URL(string: graphURI)
        var request = URLRequest(url: url!)

        // Set the Authorization header for the request. We use Bearer tokens, so we specify Bearer + the token we got from the result
        request.setValue("Bearer \(self.accessToken)", forHTTPHeaderField: "Authorization")

        URLSession.shared.dataTask(with: request) { data, response, error in

            if let error = error {
                self.updateLogging(text: "Couldn't get graph result: \(error)")
                return
            }

            guard let result = try? JSONSerialization.jsonObject(with: data!, options: []) else {

                self.updateLogging(text: "Couldn't deserialize result JSON")
                return
            }

            self.updateLogging(text: "Result from Graph: \(result))")

            }.resume()
    }
```

Дополнительные сведения приведены на странице [Microsoft Graph](https://graph.microsoft.com).

### <a name="use-msal-for-sign-out"></a>Использование MSAL для выхода

Теперь добавим поддержку функции выхода.

> [!Important]
> Если функция выхода реализована с помощью MSAL, из приложения удаляется вся известная информация о пользователе, а также прекращается сеанс на устройстве пользователя, если это разрешено настройками устройства. При необходимости можно также выполнить выход пользователя в браузере.

Чтобы добавить функцию выхода, добавьте следующий код в класс `ViewController`.

```swift
@objc func signOut(_ sender: AnyObject) {

        guard let applicationContext = self.applicationContext else { return }

        guard let account = self.currentAccount else { return }

        do {

            /**
             Removes all tokens from the cache for this application for the provided account

             - account:    The account to remove from the cache
             */

            let signoutParameters = MSALSignoutParameters(webviewParameters: self.webViewParameters!)
            signoutParameters.signoutFromBrowser = false // set this to true if you also want to signout from browser or webview

            applicationContext.signout(with: account, signoutParameters: signoutParameters, completionBlock: {(success, error) in

                if let error = error {
                    self.updateLogging(text: "Couldn't sign out account with error: \(error)")
                    return
                }

                self.updateLogging(text: "Sign out completed successfully")
                self.accessToken = ""
                self.updateCurrentAccount(account: nil)
            })

        }
    }
```

### <a name="enable-token-caching"></a>Включение кэширования маркеров

По умолчанию MSAL кэширует маркеры приложения в цепочку ключей iOS или macOS.

Чтобы включить кэширования маркеров:

1. Убедитесь, что приложение подписано правильно
1. Перейдите к параметрам проекта Xcode, а затем выберите **Вкладка "Возможности"**  > **Включить общий доступ к цепочке ключей**.
1. Выберите **+** и введите одну из следующих записей **Групп цепочек ключей**.
    - iOS: `com.microsoft.adalcache`
    - macOS: `com.microsoft.identity.universalstorage`

### <a name="add-helper-methods"></a>Добавление вспомогательных методов
Чтобы выполнить пример, добавьте в класс `ViewController` следующие вспомогательные методы.

### <a name="ios-ui"></a>Пользовательский интерфейс iOS:

``` swift

    func updateLogging(text : String) {

        if Thread.isMainThread {
            self.loggingText.text = text
        } else {
            DispatchQueue.main.async {
                self.loggingText.text = text
            }
        }
    }

    func updateSignOutButton(enabled : Bool) {
        if Thread.isMainThread {
            self.signOutButton.isEnabled = enabled
        } else {
            DispatchQueue.main.async {
                self.signOutButton.isEnabled = enabled
            }
        }
    }

    func updateAccountLabel() {

        guard let currentAccount = self.currentAccount else {
            self.usernameLabel.text = "Signed out"
            return
        }

        self.usernameLabel.text = currentAccount.username
    }

    func updateCurrentAccount(account: MSALAccount?) {
        self.currentAccount = account
        self.updateAccountLabel()
        self.updateSignOutButton(enabled: account != nil)
    }
```

### <a name="macos-ui"></a>Пользовательский интерфейс macOS:

```swift
    func updateLogging(text : String) {

        if Thread.isMainThread {
            self.loggingText.string = text
        } else {
            DispatchQueue.main.async {
                self.loggingText.string = text
            }
        }
    }

    func updateSignOutButton(enabled : Bool) {
        if Thread.isMainThread {
            self.signOutButton.isEnabled = enabled
        } else {
            DispatchQueue.main.async {
                self.signOutButton.isEnabled = enabled
            }
        }
    }

     func updateAccountLabel() {

         guard let currentAccount = self.currentAccount else {
            self.usernameLabel.stringValue = "Signed out"
            return
        }

        self.usernameLabel.stringValue = currentAccount.username ?? ""
        self.usernameLabel.sizeToFit()
     }

     func updateCurrentAccount(account: MSALAccount?) {
        self.currentAccount = account
        self.updateAccountLabel()
        self.updateSignOutButton(enabled: account != nil)
    }
```

### <a name="for-ios-only-get-additional-device-information"></a>Получение дополнительных сведения об устройстве (только iOS)

Используйте следующий код для получения сведений о текущей конфигурации устройства, в том числе, о том, настроен ли общий доступ на устройстве:

```swift
    @objc func getDeviceMode(_ sender: AnyObject) {

        if #available(iOS 13.0, *) {
            self.applicationContext?.getDeviceInformation(with: nil, completionBlock: { (deviceInformation, error) in

                guard let deviceInfo = deviceInformation else {
                    self.updateLogging(text: "Device info not returned. Error: \(String(describing: error))")
                    return
                }

                let isSharedDevice = deviceInfo.deviceMode == .shared
                let modeString = isSharedDevice ? "shared" : "private"
                self.updateLogging(text: "Received device info. Device is in the \(modeString) mode.")
            })
        } else {
            self.updateLogging(text: "Running on older iOS. GetDeviceInformation API is unavailable.")
        }
    }
```

### <a name="multi-account-applications"></a>Приложения для нескольких учетных записей

Это приложение создано для сценария с одной учетной записью. MSAL также поддерживает сценарии с несколькими учетными записями, но это требует дополнительной работы с приложениями. Вам нужно будет создать пользовательский интерфейс, чтобы помочь пользователю выбрать нужную учетную запись для каждого действия, для которого требуются маркеры. Кроме того, приложение может реализовать эвристику для выбора нужной учетной записи с помощью запроса всех учетных записей из MSAL. См. [пример API](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALPublicClientApplication.html#/c:objc(cs)MSALPublicClientApplication(im)accountsFromDeviceForParameters:completionBlock:) `accountsFromDeviceForParameters:completionBlock:`.

## <a name="test-your-app"></a>Тестирование приложения

Выполните сборку и развертывание приложения на тестовом устройстве или в симуляторе. При этом должен выполняться вход в приложение и получение маркеров для учетных записей Azure AD или личных учетных записей Майкрософт.

При первом входе пользователя в приложение от платформы удостоверений Майкрософт появится запрос на предоставление согласия на запрашиваемые разрешения. Хотя большинство пользователей могут дать согласие, некоторые арендаторы Azure AD отключили согласие пользователей, что предусматривает получение согласия администраторов от имени всех пользователей. Для поддержки этого сценария обязательно зарегистрируйте области своего приложения на портале Azure.

После выполнения входа в приложении будут отображаться данные, возвращенные из конечной точки Microsoft Graph `/me`.

## <a name="next-steps"></a>Дальнейшие действия

Из нашей серии сценариев узнайте, как создавать мобильные приложения, вызывающие защищенные веб-API.

> [!div class="nextstepaction"]
> [Scenario: Создание мобильного приложения, которое вызывает веб-API](scenario-mobile-overview.md)
