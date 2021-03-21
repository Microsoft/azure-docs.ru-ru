---
title: Настройка мобильных приложений, вызывающих веб-API | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как настроить код мобильного приложения для вызова веб-API.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 06/16/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 6f13d789cd63bb568bb8940ce614ebdb2dbcdb83
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103199752"
---
# <a name="configure-a-mobile-app-that-calls-web-apis"></a>Настройка мобильного приложения, вызывающего веб-API

После создания приложения вы узнаете, как настроить код с помощью параметров регистрации приложения. Мобильные приложения представляют некоторые сложности, связанные с подгонками своей инфраструктуры создания.

## <a name="microsoft-libraries-supporting-mobile-apps"></a>Библиотеки Майкрософт, поддерживающие мобильные приложения

Следующие библиотеки Майкрософт поддерживают мобильные приложения:

[!INCLUDE [active-directory-develop-libraries-mobile](../../../includes/active-directory-develop-libraries-mobile.md)]

## <a name="instantiate-the-application"></a>Создание экземпляра приложения

### <a name="android"></a>Android

Мобильные приложения используют `PublicClientApplication` класс. Вот как создать его экземпляр:

```Java
PublicClientApplication sampleApp = new PublicClientApplication(
                    this.getApplicationContext(),
                    R.raw.auth_config);
```

### <a name="ios"></a>iOS

Мобильным приложениям в iOS необходимо создать экземпляр `MSALPublicClientApplication` класса. Чтобы создать экземпляр класса, используйте следующий код.

```objc
NSError *msalError = nil;

MSALPublicClientApplicationConfig *config = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"];
MSALPublicClientApplication *application = [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&msalError];
```

```swift
let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>")
if let application = try? MSALPublicClientApplication(configuration: config){ /* Use application */}
```

[Дополнительные свойства мсалпубликклиентаппликатионконфиг](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALPublicClientApplicationConfig.html#/Configuration%20options) могут переопределить центр по умолчанию, указать URI перенаправления или изменить поведение кэширования маркеров MSAL.

### <a name="xamarin-or-uwp"></a>Xamarin или UWP

В этом разделе объясняется, как создать приложение для приложений Xamarin. iOS, Xamarin. Android и UWP.

#### <a name="instantiate-the-application"></a>Создание экземпляра приложения

В Xamarin или UWP самым простым способом создания экземпляра приложения является использование следующего кода. В этом коде `ClientId` — идентификатор GUID зарегистрированного приложения.

```csharp
var app = PublicClientApplicationBuilder.Create(clientId)
                                        .Build();
```

Дополнительные `With<Parameter>` Методы устанавливают родительский элемент пользовательского интерфейса, переопределяют центр по умолчанию, указывают имя и версию клиента для телеметрии, указывает URI перенаправления и укажите используемую фабрику HTTP. Фабрика HTTP может использоваться, например, для обработки прокси-серверов и указания телеметрии и ведения журнала.

В следующих разделах содержатся дополнительные сведения о создании экземпляра приложения.

##### <a name="specify-the-parent-ui-window-or-activity"></a>Укажите родительский пользовательский интерфейс, окно или действие

В Android перед выполнением интерактивной проверки подлинности передайте родительское действие. В iOS при использовании брокера передается `ViewController` . Точно так же, как и в UWP, может потребоваться передать родительское окно. Он передается при получении маркера. Но при создании приложения можно также указать обратный вызов в качестве делегата, возвращающего `UIParent` .

```csharp
IPublicClientApplication application = PublicClientApplicationBuilder.Create(clientId)
  .ParentActivityOrWindowFunc(() => parentUi)
  .Build();
```

В Android рекомендуется использовать [`CurrentActivityPlugin`](https://github.com/jamesmontemagno/CurrentActivityPlugin) . Полученный `PublicClientApplication` код построителя выглядит как в следующем примере:

```csharp
// Requires MSAL.NET 4.2 or above
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => CrossCurrentActivity.Current)
  .Build();
```

##### <a name="find-more-app-building-parameters"></a>Найти дополнительные параметры сборки приложений

Список всех методов, доступных в `PublicClientApplicationBuilder` , см. в [списке методов](/dotnet/api/microsoft.identity.client.publicclientapplicationbuilder#methods).

Описание всех параметров, предоставляемых в `PublicClientApplicationOptions` , см. в [справочной документации](/dotnet/api/microsoft.identity.client.publicclientapplicationoptions).

## <a name="tasks-for-xamarin-ios"></a>Задачи для Xamarin iOS

Если вы используете MSAL.NET в Xamarin iOS, выполните следующие задачи.

* [Переопределите и реализуйте `OpenUrl` функцию в `AppDelegate`](msal-net-xamarin-ios-considerations.md#implement-openurl)
* [Включение групп ключей цепочки](msal-net-xamarin-ios-considerations.md#enable-keychain-access)
* [Включить общий доступ к кэшу маркеров](msal-net-xamarin-ios-considerations.md#enable-token-cache-sharing-across-ios-applications)
* [Включение доступа к цепочке ключей](msal-net-xamarin-ios-considerations.md#enable-keychain-access)

Дополнительные сведения см. в статье [замечания по Xamarin iOS](msal-net-xamarin-ios-considerations.md).

## <a name="tasks-for-msal-for-ios-and-macos"></a>Задачи для MSAL для iOS и macOS

Эти задачи необходимы при использовании MSAL для iOS и macOS:

* [Реализация `openURL` обратного вызова](#brokered-authentication-for-msal-for-ios-and-macos)
* [Включить группы доступа к цепочке ключей](howto-v2-keychain-objc.md)
* [Настройка браузеров и веб-представлений](customize-webviews.md)

## <a name="tasks-for-xamarinandroid"></a>Задачи для Xamarin. Android

При использовании Xamarin. Android выполните следующие задачи.

- [Убедитесь, что управление возвращается к MSAL после завершения интерактивной части потока проверки подлинности.](msal-net-xamarin-android-considerations.md#ensure-that-control-returns-to-msal)
- [Обновление манифеста Android](msal-net-xamarin-android-considerations.md#update-the-android-manifest-for-system-webview-support)
- [Использовать внедренное веб-представление (необязательно)](msal-net-xamarin-android-considerations.md#use-the-embedded-web-view-optional)
- [Устранение неполадок при необходимости](msal-net-xamarin-android-considerations.md#troubleshooting)

Дополнительные сведения см. в разделе [рекомендации по Xamarin. Android](msal-net-xamarin-android-considerations.md).

Рекомендации по работе с браузерами в Android см. в разделе [вопросы, относящиеся к Xamarin. Android с MSAL.NET](msal-net-system-browser-android-considerations.md).

#### <a name="tasks-for-uwp"></a>Задачи для UWP

В UWP можно использовать корпоративные сети. В следующих разделах объясняются задачи, которые необходимо выполнить в корпоративном сценарии.

Дополнительные сведения см. [в разделе вопросы, относящиеся к UWP, в MSAL.NET](msal-net-uwp-considerations.md).

## <a name="configure-the-application-to-use-the-broker"></a>Настройка приложения для использования брокера

В Android и iOS, брокеры включают:

- **Единый вход (SSO)**. Вы можете использовать SSO для устройств, зарегистрированных в Azure Active Directory (Azure AD). При использовании единого входа пользователям не нужно входить в каждое приложение.
- **Идентификация устройства**. Этот параметр включает политики условного доступа, связанные с устройствами Azure AD. В процессе проверки подлинности используется сертификат устройства, созданный при присоединении устройства к рабочей области.
- **Проверка идентификации приложения**. когда приложение вызывает брокер, ему передается URL-адрес перенаправления. Затем брокер проверяет его.

### <a name="enable-the-broker-on-xamarin"></a>Включение брокера в Xamarin

Чтобы включить брокер в Xamarin, используйте `WithBroker()` параметр при вызове `PublicClientApplicationBuilder.CreateApplication` метода. По умолчанию `.WithBroker()` для задано значение true.

Чтобы включить проверку подлинности через посредника для Xamarin. iOS, выполните действия, описанные в [разделе о Xamarin. iOS](#enable-brokered-authentication-for-xamarin-ios) этой статьи.

### <a name="enable-the-broker-for-msal-for-android"></a>Включение брокера для MSAL для Android

Сведения о включении брокера в Android см. [в статье Проверка подлинности через посредника в Android](msal-android-single-sign-on.md).

### <a name="enable-the-broker-for-msal-for-ios-and-macos"></a>Включение брокера для MSAL для iOS и macOS

Проверка подлинности через посредника включена по умолчанию для сценариев Azure AD в MSAL для iOS и macOS.

В следующих разделах приведены инструкции по настройке приложения для поддержки проверки подлинности через посредника для MSAL для Xamarin. iOS или MSAL для iOS и macOS. В двух наборах инструкций некоторые шаги отличаются.

### <a name="enable-brokered-authentication-for-xamarin-ios"></a>Включение проверки подлинности через посредника для Xamarin iOS

Выполните действия, описанные в этом разделе, чтобы разрешить приложению Xamarin. iOS взаимодействовать с приложением [Microsoft Authenticator](https://itunes.apple.com/us/app/microsoft-authenticator/id983156458) .

#### <a name="step-1-enable-broker-support"></a>Шаг 1. Включение поддержки брокера

По умолчанию поддержка брокера отключена. Его можно включить для отдельного `PublicClientApplication` класса. Используйте `WithBroker()` параметр при создании `PublicClientApplication` класса с помощью `PublicClientApplicationBuilder` . По `WithBroker()` умолчанию параметр имеет значение true.

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos) // $"msauth.{Bundle.Id}://auth" (see step 6 below)
                .Build();
```

#### <a name="step-2-update-appdelegate-to-handle-the-callback"></a>Шаг 2. обновление AppDelegate для обработки обратного вызова

Когда MSAL.NET вызывает брокер, брокер выполняет обратный вызов к приложению. Он вызывает обратную передачу с помощью `AppDelegate.OpenUrl` метода. Так как MSAL ожидает ответа от брокера, приложение должно взаимодействовать для вызова MSAL.NET назад. Это поведение настраивается путем обновления `AppDelegate.cs` файла для переопределения метода, как показано в следующем коде.

```csharp
public override bool OpenUrl(UIApplication app, NSUrl url,
                             string sourceApplication,
                             NSObject annotation)
{
 if (AuthenticationContinuationHelper.IsBrokerResponse(sourceApplication))
 {
  AuthenticationContinuationHelper.SetBrokerContinuationEventArgs(url);
  return true;
 }
 else if (!AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url))
 {
  return false;
 }
 return true;
}
```

Этот метод вызывается при каждом запуске приложения. Можно обработать ответ от брокера и завершить процесс проверки подлинности, MSAL.NET запущенный.

#### <a name="step-3-set-a-uiviewcontroller"></a>Шаг 3. Задание UIViewController ()

Для Xamarin iOS обычно не требуется задавать окно объекта. Но в этом случае следует задать его, чтобы можно было отправлять и получать ответы от брокера. Чтобы задать окно объекта в `AppDelegate.cs` , необходимо задать `ViewController` .

Чтобы задать окно объекта, выполните следующие действия.

1. В `AppDelegate.cs` Задайте новое значение `App.RootViewController` `UIViewController()` . Этот параметр обеспечивает включение в вызов брокера `UIViewController` . Если оно задано неправильно, может появиться следующее сообщение об ошибке:

    `"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker."`

1. В `AcquireTokenInteractive` вызове используйте `.WithParentActivityOrWindow(App.RootViewController)` . Передайте ссылку на окно объекта, которое вы будете использовать. Приведем пример:

    В `App.cs`:
    ```csharp
       public static object RootViewController { get; set; }
    ```
    В `AppDelegate.cs`:
    ```csharp
       LoadApplication(new App());
       App.RootViewController = new UIViewController();
    ```
    В `AcquireToken` вызове:
    ```csharp
    result = await app.AcquireTokenInteractive(scopes)
                 .WithParentActivityOrWindow(App.RootViewController)
                 .ExecuteAsync();
    ```

#### <a name="step-4-register-a-url-scheme"></a>Шаг 4. Регистрация схемы URL-адреса

MSAL.NET использует URL-адреса для вызова брокера, а затем возвращает ответ брокера обратно в приложение. Чтобы завершить цикл обработки, зарегистрируйте в файле схему URL-адресов приложения `Info.plist` .

Чтобы зарегистрировать схему URL-адресов приложения, выполните следующие действия.

1. С префиксом `CFBundleURLSchemes` `msauth` .
1. Добавьте `CFBundleURLName` в конец. Используйте следующий шаблон:

   `$"msauth.(BundleId)"`

   Здесь `BundleId` уникально идентифицирует ваше устройство. Например, если `BundleId` имеет значение `yourcompany.xforms` , то схема URL-адреса будет иметь значение `msauth.com.yourcompany.xforms` .

  
      Эта схема URL-адресов станет частью URI перенаправления, который уникальным образом идентифицирует ваше приложение при получении ответа брокера.

   ```XML
    <key>CFBundleURLTypes</key>
       <array>
         <dict>
           <key>CFBundleTypeRole</key>
           <string>Editor</string>
           <key>CFBundleURLName</key>
           <string>com.yourcompany.xforms</string>
           <key>CFBundleURLSchemes</key>
           <array>
             <string>msauth.com.yourcompany.xforms</string>
           </array>
         </dict>
       </array>
   ```

#### <a name="step-5-add-to-the-lsapplicationqueriesschemes-section"></a>Шаг 5. Добавление в раздел Лсаппликатионкуериессчемес

MSAL использует `–canOpenURL:` для проверки установки брокера на устройстве. В iOS 9 Компания Apple заблокировала схемы, которые может запрашивать приложение.

Добавьте `msauthv2` в `LSApplicationQueriesSchemes` раздел `Info.plist` файла, как показано в следующем примере кода:

```XML
<key>LSApplicationQueriesSchemes</key>
    <array>
      <string>msauthv2</string>
    </array>
```

### <a name="brokered-authentication-for-msal-for-ios-and-macos"></a>Аутентификация через посредника для MSAL для iOS и macOS

Проверка подлинности через посредника включена по умолчанию для сценариев Azure AD.

#### <a name="step-1-update-appdelegate-to-handle-the-callback"></a>Шаг 1. обновление AppDelegate для обработки обратного вызова

Когда MSAL для iOS и macOS вызывает брокер, брокер выполняет обратный вызов к приложению с помощью `openURL` метода. Так как MSAL ожидает ответа от брокера, ваше приложение должно сотрудничать для вызова обратного MSAL. Настройте эту возможность, обновив `AppDelegate.m` файл, чтобы переопределить метод, как показано в следующих примерах кода.

```objc
- (BOOL)application:(UIApplication *)app
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
    return [MSALPublicClientApplication handleMSALResponse:url
                                         sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
}
```

```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        guard let sourceApplication = options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String else {
            return false
        }

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApplication)
    }
```

Если вы приняли `UISceneDelegate` в iOS 13 или более поздней версии, поставьте обратный вызов MSAL в `scene:openURLContexts:` `UISceneDelegate` вместо. MSAL `handleMSALResponse:sourceApplication:` необходимо вызывать только один раз для каждого URL-адреса.

Дополнительные сведения см. в [документации Apple](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238059-scene?language=objc).

#### <a name="step-2-register-a-url-scheme"></a>Шаг 2. Регистрация схемы URL-адреса

MSAL для iOS и macOS использует URL-адреса для вызова брокера, а затем возвращает ответ брокера в приложение. Чтобы завершить цикл обработки, зарегистрируйте схему URL-адресов для приложения в `Info.plist` файле.

Чтобы зарегистрировать схему для приложения, сделайте следующее:

1. Добавьте в префикс настраиваемой схемы URL-адреса `msauth` .

1. Добавьте идентификатор пакета в конец схемы. Используйте следующий шаблон:

   `$"msauth.(BundleId)"`

   Здесь `BundleId` уникально идентифицирует ваше устройство. Например, если `BundleId` имеет значение `yourcompany.xforms` , то схема URL-адреса будет иметь значение `msauth.com.yourcompany.xforms` .

    Эта схема URL-адресов станет частью URI перенаправления, который уникальным образом идентифицирует ваше приложение при получении ответа брокера. Убедитесь, что URI перенаправления в формате `msauth.(BundleId)://auth` зарегистрирован для вашего приложения в [портал Azure](https://portal.azure.com).

   ```XML
   <key>CFBundleURLTypes</key>
   <array>
       <dict>
           <key>CFBundleURLSchemes</key>
           <array>
               <string>msauth.[BUNDLE_ID]</string>
           </array>
       </dict>
   </array>
   ```

#### <a name="step-3-add-lsapplicationqueriesschemes"></a>Шаг 3. Добавление Лсаппликатионкуериессчемес

Добавьте `LSApplicationQueriesSchemes` , чтобы разрешить вызовы Microsoft Authenticatorного приложения, если оно установлено.

> [!NOTE]
> `msauthv3`Схема необходима, когда приложение компилируется с помощью Xcode 11 и более поздних версий.

Ниже приведен пример добавления `LSApplicationQueriesSchemes` :

```XML
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>msauthv2</string>
  <string>msauthv3</string>
</array>
```

### <a name="brokered-authentication-for-xamarinandroid"></a>Аутентификация через посредника для Xamarin. Android

Сведения о включении брокера в Android см. [в статье Проверка подлинности через посредника в Xamarin. Android](msal-net-use-brokers-with-xamarin-apps.md#brokered-authentication-for-android).

## <a name="next-steps"></a>Следующие шаги

Перейдите к следующей статье в этом сценарии, [получая маркер](scenario-mobile-acquire-token.md).
