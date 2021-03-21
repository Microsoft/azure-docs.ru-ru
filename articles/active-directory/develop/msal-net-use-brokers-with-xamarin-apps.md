---
title: Использование брокеров с Xamarin iOS & Android | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как настроить приложения Xamarin iOS, которые могут использовать Microsoft Authenticator и библиотеку проверки подлинности Майкрософт для .NET (MSAL.NET). Также Узнайте, как выполнить миграцию из библиотеки аутентификация Azure AD для .NET (ADAL.NET) в библиотеку проверки подлинности Майкрософт для .NET (MSAL.NET).
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/08/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 226e94510709b37a7e6b1aae90a7e0ec5b4222b9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103199572"
---
# <a name="use-microsoft-authenticator-or-intune-company-portal-on-xamarin-applications"></a>Использование Microsoft Authenticator или Корпоративный портал Intune в приложениях Xamarin

В Android и iOS такие брокеры, как Microsoft Authenticator и Microsoft Intune Корпоративный портал для Android, включают:

- **Единый вход (SSO)**. пользователям не нужно входить в каждое приложение.
- **Идентификация устройства**. брокер обращается к сертификату устройства. Этот сертификат создается на устройстве при его присоединении к рабочей области.
- **Проверка идентификации приложения**. когда приложение вызывает брокер, ему передается URL-адрес перенаправления. Брокер проверяет URL-адрес.

Чтобы включить одну из этих функций, используйте `WithBroker()` параметр при вызове `PublicClientApplicationBuilder.CreateApplication` метода. По `.WithBroker()` умолчанию параметр имеет значение true.

Настройка проверки подлинности через посредника в библиотеке проверки подлинности Майкрософт для .NET (MSAL.NET) зависит от платформы:

* [Приложения для iOS](#brokered-authentication-for-ios)
* [Приложения для Android](#brokered-authentication-for-android)

## <a name="brokered-authentication-for-ios"></a>Аутентификация через посредника для iOS

Чтобы включить взаимодействие приложения Xamarin. iOS с приложением [Microsoft Authenticator](https://itunes.apple.com/us/app/microsoft-authenticator/id983156458) , выполните следующие действия. Если вы намерены ориентироваться на iOS 13, рекомендуем ознакомиться с [изменениями в API нарушения Apple](./msal-net-xamarin-ios-considerations.md).

### <a name="step-1-enable-broker-support"></a>Шаг 1. Включение поддержки брокера

Необходимо включить поддержку брокера для отдельных экземпляров `PublicClientApplication` . По умолчанию поддержка отключена. При создании `PublicClientApplication` с помощью `PublicClientApplicationBuilder` используйте параметр, `WithBroker()` как показано в следующем примере. По `WithBroker()` умолчанию параметр имеет значение true.

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos) // $"msauth.{Bundle.Id}://auth" (see step 6 below)
                .Build();
```

### <a name="step-2-enable-keychain-access"></a>Шаг 2. Включение доступа к цепочке ключей

Чтобы включить доступ к цепочке ключей, необходимо иметь группу доступа к цепочке ключей для приложения. Вы можете использовать `WithIosKeychainSecurityGroup()` API для настройки группы доступа к цепочке ключей при создании приложения:

```csharp
var builder = PublicClientApplicationBuilder
     .Create(ClientId)
     .WithIosKeychainSecurityGroup("com.microsoft.adalcache")
     .Build();
```

Дополнительные сведения см. в разделе [Включение доступа к цепочке ключей](msal-net-xamarin-ios-considerations.md#enable-keychain-access).

### <a name="step-3-update-appdelegate-to-handle-the-callback"></a>Шаг 3. обновление AppDelegate для обработки обратного вызова

Когда MSAL.NET вызывает брокер, брокер выполняет обратный вызов к приложению через `OpenUrl` метод `AppDelegate` класса. Так как MSAL ожидает ответа брокера, приложение должно взаимодействовать для вызова MSAL.NET назад. Чтобы обеспечить взаимодействие, обновите файл *AppDelegate. CS* , чтобы переопределить следующий метод.

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

Этот метод вызывается при каждом запуске приложения. Он используется как возможность обработки ответа от брокера и завершения процесса проверки подлинности, MSAL.NET запущен.

### <a name="step-4-set-uiviewcontroller"></a>Шаг 4. Настройка UIViewController ()

Находясь в файле *AppDelegate. CS* , задайте окно объекта. Обычно вам не нужно задавать окно объекта для Xamarin iOS, но для отправки и получения ответов от брокера необходимо окно объекта.

Чтобы настроить окно объекта, выполните следующие действия.

1. В файле *AppDelegate. CS* задайте новое значение `App.RootViewController` `UIViewController()` . Это назначение гарантирует, что вызов брокера включает `UIViewController` . Если этот параметр назначен неправильно, может появиться следующее сообщение об ошибке:

      `"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker"`

1. В `AcquireTokenInteractive` вызове используйте, `.WithParentActivityOrWindow(App.RootViewController)` а затем передайте ссылку на окно объекта, которое вы будете использовать.

    В *app. CS*:

    ```csharp
       public static object RootViewController { get; set; }
    ```

    В *AppDelegate. CS*:

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

### <a name="step-5-register-a-url-scheme"></a>Шаг 5. Регистрация схемы URL-адреса

MSAL.NET использует URL-адреса для вызова брокера, а затем возвращает ответ брокера в приложение. Чтобы завершить цикл обработки, зарегистрируйте схему URL-адресов для своего приложения в файле *info. plist* .

`CFBundleURLSchemes`Имя должно включать в `msauth.` качестве префикса. После префикса используйте `CFBundleURLName` .

В схеме URL-адресов `BundleId` уникальным образом идентифицирует приложение: `$"msauth.(BundleId)"` . Поэтому `BundleId` , если имеет значение, используется `com.yourcompany.xforms` Схема URL-адресов `msauth.com.yourcompany.xforms` .

> [!NOTE]
> Эта схема URL-адреса преобразуется в URI перенаправления, который уникальным образом идентифицирует ваше приложение при получении ответа от брокера.

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

### <a name="step-6-add-the-broker-identifier-to-the-lsapplicationqueriesschemes-section"></a>Шаг 6. Добавление идентификатора брокера в раздел Лсаппликатионкуериессчемес

MSAL использует, `–canOpenURL:` чтобы проверить, установлен ли брокер на устройстве. В iOS 9 Компания Apple заблокировала схемы, которые может запрашивать приложение.

Добавьте `msauthv2` в `LSApplicationQueriesSchemes` раздел файла *info. plist* , как показано в следующем примере:

```XML
<key>LSApplicationQueriesSchemes</key>
    <array>
      <string>msauthv2</string>
      <string>msauthv3</string>
    </array>
```

### <a name="step-7-add-a-redirect-uri-to-your-app-registration"></a>Шаг 7. Добавление URI перенаправления в регистрацию приложения

При использовании брокера URI перенаправления имеет дополнительное требование. URI перенаправления _должен_ иметь следующий формат:

```csharp
$"msauth.{BundleId}://auth"
```

Приведем пример:

```csharp
public static string redirectUriOnIos = "msauth.com.yourcompany.XForms://auth";
```

Обратите внимание, что URI перенаправления соответствует `CFBundleURLSchemes` имени, которое вы включили в файл *info. plist* .

Добавьте URI перенаправления в регистрацию приложения в [портал Azure](https://portal.azure.com). Чтобы создать правильно отформатированный URI перенаправления, используйте **Регистрация приложений** в портал Azure для создания URI перенаправления с брокером из идентификатора пакета.

**Создание URI перенаправления:**

1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
1. Выберите **Azure Active Directory**  >  **Регистрация приложений** > зарегистрированное приложение
1. Выбор **проверки подлинности**  >  **Добавление платформы**  >  **iOS/macOS**
1. Введите идентификатор пакета и нажмите кнопку **настроить**.

    Скопируйте созданный URI перенаправления, который отображается в текстовом поле **URI перенаправления** для включения в код:

    :::image type="content" source="media/msal-net-use-brokers-with-xamarin-apps/portal-01-ios-platform-settings.png" alt-text="параметры платформы iOS с созданным URI перенаправления в портал Azure":::
1. Выберите **Готово** , чтобы завершить создание URI перенаправления.

## <a name="brokered-authentication-for-android"></a>Аутентификация через посредника для Android

### <a name="step-1-enable-broker-support"></a>Шаг 1. Включение поддержки брокера

Поддержка брокера включается отдельно для каждого компонента `PublicClientApplication` . Она отключена по умолчанию. Используйте `WithBroker()` параметр (по умолчанию задано значение true) при создании с `IPublicClientApplication` помощью `PublicClientApplicationBuilder` .

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithRedirectUri(redirectUriOnAndroid) // See step #4
                .Build();
```

### <a name="step-2-update-appdelegate-to-handle-the-callback"></a>Шаг 2. обновление AppDelegate для обработки обратного вызова

Когда MSAL.NET вызывает брокер, брокер, в свою очередь, выполняет обратный вызов к приложению с помощью `OnActivityResult()` метода. Так как MSAL будет ждать ответа от брокера, приложению нужно направить результат в MSAL.NET.

Направьте результат в `SetAuthenticationContinuationEventArgs(int requestCode, Result resultCode, Intent data)` метод, переопределив метод, `OnActivityResult()` как показано ниже:

```csharp
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
   base.OnActivityResult(requestCode, resultCode, data);
   AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

Этот метод вызывается каждый раз при запуске приложения брокера и используется как возможность обработки ответа от брокера и завершения процесса проверки подлинности, запущенного MSAL.NET.

### <a name="step-3-set-an-activity"></a>Шаг 3. Задание действия

Чтобы включить проверку подлинности через посредника, настройте действие таким образом, чтобы MSAL мог отправлять и получать ответ от брокера. Для этого укажите действие (обычно `MainActivity` ) для `WithParentActivityOrWindow(object parent)` родительского объекта.

Например, в вызове `AcquireTokenInteractive()` :

```csharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow((Activity)context))
             .ExecuteAsync();
```

### <a name="step-4-add-a-redirect-uri-to-your-app-registration"></a>Шаг 4. Добавление URI перенаправления в регистрацию приложения

MSAL использует URL-адреса для вызова брокера, а затем возвращается в приложение. Чтобы завершить цикл обработки, зарегистрируйте **URI перенаправления** для приложения с помощью [портал Azure](https://portal.azure.com).

Формат URI перенаправления для приложения зависит от сертификата, используемого для подписи APK. Пример:

```
msauth://com.microsoft.xforms.testApp/hgbUYHVBYUTvuvT&Y6tr554365466=
```

Последняя часть URI — `hgbUYHVBYUTvuvT&Y6tr554365466=` это версия сигнатуры в кодировке Base64, с которой подписывается apk. При разработке приложения в Visual Studio при отладке кода без подписи APK с помощью определенного сертификата Visual Studio подписывает APK для вас в целях отладки. Когда Visual Studio подписывает APK для вас таким образом, он предоставляет ей уникальную подпись для компьютера, на котором она создана. Таким же, каждый раз при сборке приложения на другом компьютере необходимо обновить URI перенаправления в коде приложения и регистрацию приложения в портал Azure для проверки подлинности с помощью MSAL.

При отладке может возникнуть исключение MSAL (или сообщение журнала), сообщающее, что указанный URI перенаправления неверен. **Исключение или сообщение журнала также указывает URI перенаправления, который следует использовать** с текущим компьютером, на котором выполняется отладка. Вы можете использовать указанный универсальный код ресурса (URI) перенаправления, чтобы продолжить разработку приложения при условии обновления URI перенаправления в коде и добавления предоставленного URI перенаправления в регистрацию приложения в портал Azure.

Когда вы будете готовы завершить код, обновите URI перенаправления в коде и регистрацию приложения в портал Azure, чтобы использовать подпись сертификата, с помощью которого вы подписаны APK.

На практике это означает, что следует добавить URI перенаправления для каждого члена команды разработчиков, *а также* URI перенаправления для версии APK, подписанной в рабочей среде.

Вы можете вычислить подпись самостоятельно, как это делает MSAL:

```csharp
   private string GetRedirectUriForBroker()
   {
      string packageName = Application.Context.PackageName;
      string signatureDigest = this.GetCurrentSignatureForPackage(packageName);
      if (!string.IsNullOrEmpty(signatureDigest))
      {
            return string.Format(CultureInfo.InvariantCulture, "{0}://{1}/{2}", RedirectUriScheme,
               packageName.ToLowerInvariant(), signatureDigest);
      }

      return string.Empty;
   }

   private string GetCurrentSignatureForPackage(string packageName)
   {
            PackageInfo info = Application.Context.PackageManager.GetPackageInfo(packageName,
               PackageInfoFlags.Signatures);
            if (info != null && info.Signatures != null && info.Signatures.Count > 0)
            {
               // First available signature. Applications can be signed with multiple signatures.
               // The order of Signatures is not guaranteed.
               Signature signature = info.Signatures[0];
               MessageDigest md = MessageDigest.GetInstance("SHA");
               md.Update(signature.ToByteArray());
               return Convert.ToBase64String(md.Digest(), Base64FormattingOptions.None);
               // Server side needs to register all other tags. ADAL will
               // send one of them.
            }
   }
```

Вы также можете получить подпись для пакета, используя **keytool** с помощью следующих команд:

* Windows:
    ```console
    keytool.exe -list -v -keystore "%LocalAppData%\Xamarin\Mono for Android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
    ````
* macOS:
    ```console
    keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
    ````

### <a name="step-5-optional-fall-back-to-the-system-browser"></a>Шаг 5 (необязательно). Возврат к обозревателю системы

Если MSAL настроен для использования брокера, но брокер не установлен, MSAL будет использовать веб-представление (браузер). MSAL попытается пройти проверку подлинности с помощью системного браузера по умолчанию на устройстве, что приведет к сбою, так как универсальный код ресурса (URI) перенаправления настроен для брокера, а системный браузер не знает, как использовать его для перехода к MSAL. Чтобы избежать сбоя, можно настроить *Фильтр намерения* с помощью URI перенаправления брокера, который использовался на шаге 4.

Измените манифест приложения, чтобы добавить фильтр намерения:

```xml
<!-- NOTE the SLASH (required) that prefixes the signature value in the path attribute.
     The signature value is the Base64-encoded signature discussed above. -->
<intent-filter>
      <data android:scheme="msauth"
                    android:host="Package Name"
                    android:path="/Package Signature"/>
```

Например, если у вас есть URI перенаправления для `msauth://com.microsoft.xforms.testApp/hgbUYHVBYUTvuvT&Y6tr554365466=` , манифест должен выглядеть как следующий фрагмент кода XML.

Требуется прямая косая черта ( `/` ) перед сигнатурой в `android:path` значении. 

```xml
<!-- NOTE the SLASH (required) that prefixes the signature value in the path attribute.
     The signature value is the Base64-encoded signature discussed above. -->
<intent-filter>
      <data android:scheme="msauth"
                    android:host="com.microsoft.xforms.testApp"
                    android:path="/hgbUYHVBYUTvuvT&Y6tr554365466="/>
```

Дополнительные сведения о настройке приложения для браузера системы и поддержки Android 11 см. [в статье обновление манифеста Android для поддержки системных браузеров](msal-net-xamarin-android-considerations.md#update-the-android-manifest-for-system-webview-support).

В качестве альтернативы можно настроить MSAL для возврата к встроенному браузеру, который не полагается на URI перенаправления:

```csharp
.WithUseEmbeddedWebUi(true)
```

## <a name="troubleshooting-tips-for-android-brokered-authentication"></a>Советы по устранению неполадок при проверке подлинности через брокер Android

Вот несколько советов по предотвращению проблем при реализации проверки подлинности через посредника в Android:

- **URI перенаправления** — Добавьте URI перенаправления в регистрацию приложения в [портал Azure](https://portal.azure.com/). Отсутствующий или неверный URI перенаправления является распространенной проблемой, возникшей разработчиками.
- **Версия компонента Service Broker** — установите минимальную требуемую версию приложений брокера. Одно из этих двух приложений можно использовать для проверки подлинности через посредника в Android.
  - [Корпоративный портал Intune](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal) (версия 5.0.4689.0 или более поздняя)
  - [Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator) (версия 6.2001.0140 или более поздняя).
- **Приоритет компонента Service Broker** . MSAL взаимодействует с *первым брокером, установленным* на устройстве при установке нескольких брокеров.

    Пример. Если сначала установить Microsoft Authenticator, а затем установить Корпоративный портал Intune, то проверка подлинности через посредник будет выполняться *только* в Microsoft Authenticator.
- **Журналы** . при возникновении проблемы с проверкой подлинности через посредника Просмотр журналов брокера может помочь в диагностике причины.
  - Получение журналов Microsoft Authenticator:

    1. Нажмите кнопку меню в правом верхнем углу приложения.
    1. Выберите **Отправить отзыв о**  >  **проблемах?**.
    1. В разделе **что вы пытаетесь сделать?** выберите вариант и добавьте описание.
    1. Чтобы отправить журналы, щелкните стрелку в правом верхнем углу приложения.

    После отправки журналов в диалоговом окне отображается идентификатор инцидента. Запишите идентификатор инцидента и включите его при запросе помощи.

  - Получение журналов Корпоративный портал Intune:

    1. Нажмите кнопку меню в левом верхнем углу приложения.
    1. Выберите **Справка**  >  **Электронная почта поддержка**.
    1. Чтобы отправить журналы, выберите **отправить только журналы**.

    После отправки журналов в диалоговом окне отображается идентификатор инцидента. Запишите идентификатор инцидента и включите его при запросе помощи.

## <a name="next-steps"></a>Следующие шаги

Сведения о [вопросах использования универсальная платформа Windows с MSAL.NET](msal-net-uwp-considerations.md).
