---
title: Краткое руководство. Добавление возможности входа в приложение iOS или macOS с помощью учетной записи Майкрософт | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как в приложении iOS или macOS реализовать вход пользователей, получение маркера доступа от платформы удостоверений Майкрософт и вызов API Microsoft Graph.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/24/2019
ms.author: marsma
ms.reviewer: jmprieur, saeeda
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:iOS
ms.openlocfilehash: d2c5af6aeccfbae0851513ff575bde3c39e3ca5f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100103793"
---
# <a name="quickstart-sign-in-users-and-call-the-microsoft-graph-api-from-an-ios-or-macos-app"></a>Краткое руководство. Вход пользователей и вызов Microsoft API Graph из приложения iOS или macOS

Из этого краткого руководства вы узнаете, как скачать и выполнить пример кода. В примере кода показано, как в нативном приложении iOS или macOS реализовать вход пользователей и получение маркера доступа для вызова API Microsoft Graph.

Это краткое руководство распространяется на приложения iOS и macOS. Некоторые действия нужно выполнять только для приложений iOS. Для таких действий будут добавлены соответствующие указания.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
* XCode 10 или более поздней версии
* iOS 10 или более поздней версии
* macOS 10.12+

## <a name="how-the-sample-works"></a>Как работает этот пример

![Схема работы приложения, создаваемого в этом кратком руководстве](media/quickstart-v2-ios/ios-intro.svg)

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>Регистрация и скачивание приложения, используемого в этом кратком руководстве
> У вас есть два варианта запуска приложения, используемого в этом кратком руководстве:
> * [Экспресс-способ] [Вариант 1. Регистрация и автоматическая настройка приложения, а затем скачивание примера кода](#option-1-register-and-auto-configure-your-app-and-then-download-the-code-sample)
> * [Вручную] [Вариант 2. Регистрация и настройка приложения и примера кода вручную](#option-2-register-and-manually-configure-your-application-and-code-sample)
>
> ### <a name="option-1-register-and-auto-configure-your-app-and-then-download-the-code-sample"></a>Вариант 1. Регистрация и автоматическая настройка приложения, а затем скачивание примера кода.
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
> Регистрация приложения:
> 1. Откройте страницу <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/IosQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Следуйте инструкциям, чтобы быстро скачать и автоматически настроить новое приложение.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Вариант 2. Регистрация и настройка приложения и примера кода вручную
>
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
> Чтобы зарегистрировать приложение и добавить сведения о его регистрации в решение вручную, сделайте следующее:
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким клиентам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать клиент, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.    
> 1. В разделе **Управление** выберите **Регистрация приложений** > **Создать регистрацию**.
> 1. Введите значение **Name** (Имя) для приложения. Пользователи приложения могут видеть это имя. Вы можете изменить его позже.
> 1. Выберите **Зарегистрировать**.
> 1. В разделе **Управление** выберите **Проверка подлинности** > **Добавить платформу** > **iOS**.
> 1. Введите **идентификатор пакета** вашего приложения. Идентификатор пакета — это уникальная строка, однозначно идентифицирующая ваше приложение, например `com.<yourname>.identitysample.MSALMacOS`. Запишите значение, которое используете. Обратите внимание, что конфигурация iOS также применима к приложениям macOS.
> 1. Щелкните элемент **Настройка** и сохраните сведения о **конфигурации MSAL** для последующего использования в рамках этого краткого руководства.
> 1. Нажмите кнопку **Готово**.

> [!div renderon="portal" class="sxs-lookup"]
>
> #### <a name="step-1-configure-your-application"></a>Шаг 1. Настройка приложения
> Для работы примера кода, приведенного в этом кратком руководстве, необходимо добавить **URI перенаправления**, совместимый с брокером авторизации.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести это изменение для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-ios/green-check.png). Ваше приложение настроено с помощью этих атрибутов
>
> #### <a name="step-2-download-the-sample-project"></a>Шаг 2. Загрузка примера проекта
> > [!div id="autoupdate_ios" class="nextstepaction"]
> > [Загрузка примера кода для iOS]()
>
> > [!div id="autoupdate_macos" class="nextstepaction"]
> > [Загрузка примера кода для macOS]()
> [!div renderon="docs"]
> #### <a name="step-2-download-the-sample-project"></a>Шаг 2. Загрузка примера проекта
>
> - [Загрузка примера кода для iOS](https://github.com/Azure-Samples/active-directory-ios-swift-native-v2/archive/master.zip)
> - [Загрузка примера кода для macOS](https://github.com/Azure-Samples/active-directory-macOS-swift-native-v2/archive/master.zip)

#### <a name="step-3-install-dependencies"></a>Шаг 3. Установка зависимостей

В окне терминала перейдите к папке со скачанным примером кода и выполните `pod install`, чтобы установить последнюю библиотеку MSAL.

> [!div renderon="portal" class="sxs-lookup"]
> #### <a name="step-4-your-app-is-configured-and-ready-to-run"></a>Шаг 4. Приложение настроено и готово к запуску
> Мы уже настроили для проекта нужные значения свойств приложения, и его можно запускать.
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`
>
> [!div renderon="docs"]
>#### <a name="step-4-configure-your-project"></a>Шаг 4. Настройка проекта
> Если вы выбрали вариант 1 выше, можно пропустить эти шаги.
> 1. Извлеките ZIP-файл и откройте проект в XCode.
> 1. Измените **ViewController.swift** и замените строку, начинающуюся с "let kClientID", следующим фрагментом кода. Не забудьте обновить значение `kClientID`, указав идентификатор клиента, который вы ранее сохранили при регистрации приложения на портале в этом кратком руководстве:
>    ```swift
>    let kClientID = "Enter_the_Application_Id_Here"
>    ```
> 1. Если вы создаете приложение для [национальных облаков Azure AD](/graph/deployments#app-registration-and-token-service-root-endpoints), замените строку, начинающуюся с let kGraphEndpoint и let kAuthority соответствующими конечными точками. Для предоставления глобального доступа используйте значения по умолчанию:
>     ```swift
>     let kGraphEndpoint = "https://graph.microsoft.com/"
>     let kAuthority = "https://login.microsoftonline.com/common"
>     ```
> 1. См. описание [других конечных точек](/graph/deployments#app-registration-and-token-service-root-endpoints). Например, чтобы выполнить процесс для Azure AD Germany, используйте следующее:
>     ```swift
>     let kGraphEndpoint = "https://graph.microsoft.de/"
>     let kAuthority = "https://login.microsoftonline.de/common"
>     ```
> 1. Откройте параметры проекта. В разделе **Идентификатор** введите **идентификатор пакета**, введенный на портале.
> 1. Щелкните **Info.plist**, а затем выберите **Открыть как** > **Исходный код**.
> 1. В корневом узле словаря замените `Enter_the_bundle_Id_Here` на ***идентификатор пакета***, использованный на портале.
>
>    ```xml
>    <key>CFBundleURLTypes</key>
>    <array>
>       <dict>
>          <key>CFBundleURLSchemes</key>
>          <array>
>             <string>msauth.Enter_the_Bundle_Id_Here</string>
>          </array>
>       </dict>
>    </array>
>
>    ```
> 1. Создайте и запустите приложение.

## <a name="more-information"></a>Дополнительные сведения

Дополнительные сведения о кратком руководстве содержатся в этих разделах.

### <a name="get-msal"></a>Получение MSAL

MSAL ([MSAL.framework](https://github.com/AzureAD/microsoft-authentication-library-for-objc)) — это библиотека, используемая для выполнения входа пользователей и запросов маркеров, которые нужны для доступа к API, защищенному платформой удостоверений Майкрософт. MSAL можно добавить в приложение, используя следующее действие.

```
$ vi Podfile

```
Добавьте в файл Podfile следующий код (целевую исполняющую среду проекта):

```
use_frameworks!

target 'MSALiOS' do
   pod 'MSAL'
end

```

Выполните команду установки CocoaPods:

```pod install```

### <a name="initialize-msal"></a>Инициализация MSAL

Добавив следующий код, вы можете добавить ссылку на MSAL.

```swift
import MSAL
```

Затем выполните инициализацию MSAL с помощью следующего кода.

```swift
let authority = try MSALAADAuthority(url: URL(string: kAuthority)!)

let msalConfiguration = MSALPublicClientApplicationConfig(clientId: kClientID, redirectUri: nil, authority: authority)
self.applicationContext = try MSALPublicClientApplication(configuration: msalConfiguration)
```

> |Где: | Описание |
> |---------|---------|
> | `clientId` | Идентификатор зарегистрированного приложения на *portal.azure.com* |
> | `authority` | Платформа удостоверений Майкрософт. Как правило, это `https://login.microsoftonline.com/common`. |
> | `redirectUri` | URI перенаправления приложения. Можете передать нуль, чтобы использовать значение по умолчанию, или ваш пользовательский URI перенаправления. |

### <a name="for-ios-only-additional-app-requirements"></a>(Только для iOS) Дополнительные требования для приложения

Ваше приложение также должно содержать следующий код в объекте `AppDelegate`. Это позволит пакету SDK MSAL обрабатывать ответ токена из приложения брокера авторизации при выполнении проверки подлинности.

 ```swift
 func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
    }

 ```

> [!NOTE]
> (В iOS 13+) Если вы принимаете `UISceneDelegate` вместо `UIApplicationDelegate`, поместите этот код в обратный вызов `scene:openURLContexts:` (см. [Документацию Apple](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238059-scene?language=objc)).
> Если UISceneDelegate и UIApplicationDelegate поддерживаются для совместимости с более старыми версиями iOS, то обратный вызов MSAL потребуется поместить в оба места.

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

Наконец, ваше приложение должно содержать запись `LSApplicationQueriesSchemes` в файле ***Info.plist*** вместе с элементом `CFBundleURLTypes`. Пример уже содержит эту запись.

   ```xml
   <key>LSApplicationQueriesSchemes</key>
   <array>
      <string>msauthv2</string>
      <string>msauthv3</string>
   </array>
   ```

### <a name="sign-in-users--request-tokens"></a>Выполнение входа пользователей и запрос токенов

MSAL имеет два метода получения маркеров безопасности: `acquireToken` и `acquireTokenSilent`.

#### <a name="acquiretoken-get-a-token-interactively"></a>acquireToken. Интерактивное получение маркера

Иногда требуется взаимодействие пользователей с платформой удостоверений Майкрософт. В таких случаях пользователю может потребоваться выбрать свою учетную запись, ввести учетные данные или предоставить согласие на разрешения приложения. Например,

* первый вход пользователей в приложение;
* Если пользователь сбрасывает пароль, то потребуется вводить свои учетные данные.
* Когда ваше приложение впервые запрашивает доступ к ресурсу.
* Когда требуются политики условного доступа или MFA.

```swift
let parameters = MSALInteractiveTokenParameters(scopes: kScopes, webviewParameters: self.webViewParamaters!)
self.applicationContext!.acquireToken(with: parameters) { (result, error) in /* Add your handling logic */}
```

> |Где:| Описание |
> |---------|---------|
> | `scopes` | Содержит запрашиваемые области, то есть `[ "user.read" ]` для Microsoft Graph или `[ "<Application ID URL>/scope" ]` для пользовательских веб-API (`api://<Application ID>/access_as_user`). |

#### <a name="acquiretokensilent-get-an-access-token-silently"></a>acquireTokenSilent. Автоматическое получение маркера доступа

Приложения не требуют, чтобы пользователи выполняли вход каждый раз при запросе маркера. Если пользователь уже вошел, этот метод позволяет приложениям запрашивать маркеры автоматически.

```swift
self.applicationContext!.getCurrentAccount(with: nil) { (currentAccount, previousAccount, error) in

   guard let account = currentAccount else {
      return
   }

   let silentParams = MSALSilentTokenParameters(scopes: self.kScopes, account: account)
   self.applicationContext!.acquireTokenSilent(with: silentParams) { (result, error) in /* Add your handling logic */}
}
```

> |Где: | Описание |
> |---------|---------|
> | `scopes` | Содержит запрашиваемые области, то есть `[ "user.read" ]` для Microsoft Graph или `[ "<Application ID URL>/scope" ]` для пользовательских веб-API (`api://<Application ID>/access_as_user`). |
> | `account` | Учетная запись, для которой запрашивается токен. В этом кратком руководстве рассматривается приложение с одной учетной записью. Если вы хотите создать приложение с несколькими учетными записями, нужно определить логику для выбора учетной записи, которая будет использоваться для запросов токенов, с помощью `accountsFromDeviceForParameters:completionBlock:` и правильного идентификатора `accountIdentifier`. |

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к пошаговому учебнику, с помощью которого вы создадите приложение iOS или macOS, получающее маркер доступа от платформы удостоверений Майкрософт и использующее его для вызова API Microsoft Graph.

> [!div class="nextstepaction"]
> [Учебник. Вход пользователей и вызов Microsoft Graph из приложения iOS или macOS](tutorial-v2-ios.md)
