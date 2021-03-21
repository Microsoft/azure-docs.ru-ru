---
title: Настройка кода и устранение неполадок в Xamarin Android (MSAL.NET) | Службы
titleSuffix: Microsoft identity platform
description: Сведения о вопросах использования Xamarin Android с библиотекой проверки подлинности Майкрософт для .NET (MSAL.NET).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 11642480ac817b50d102e396b8ab5e200948a615
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103199567"
---
# <a name="configuration-requirements-and-troubleshooting-tips-for-xamarin-android-with-msalnet"></a>Требования к конфигурации и советы по устранению неполадок для Xamarin Android с MSAL.NET

Существует несколько изменений конфигурации, которые необходимо внести в код при использовании Xamarin Android с библиотекой проверки подлинности Майкрософт для .NET (MSAL.NET). В следующих разделах описываются необходимые изменения, а затем раздел " [Устранение неполадок](#troubleshooting) ", который поможет избежать некоторых наиболее распространенных проблем.

## <a name="set-the-parent-activity"></a>Задание родительского действия

В Xamarin Android задайте родительское действие, чтобы токен возвращался после взаимодействия:

```csharp
var authResult = AcquireTokenInteractive(scopes)
 .WithParentActivityOrWindow(parentActivity)
 .ExecuteAsync();
```

В MSAL.NET 4,2 и более поздних версиях эту функцию можно также установить на уровне [публикклиентаппликатион][PublicClientApplication]. Для этого используйте обратный вызов:

```csharp
// Requires MSAL.NET 4.2 or later
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => parentActivity)
  .Build();
```

Если вы используете [куррентактивитиплугин](https://github.com/jamesmontemagno/CurrentActivityPlugin), [`PublicClientApplication`][PublicClientApplication] код построителя должен выглядеть примерно так:

```csharp
// Requires MSAL.NET 4.2 or later
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => CrossCurrentActivity.Current)
  .Build();
```

## <a name="ensure-that-control-returns-to-msal"></a>Убедитесь, что управление возвращается к MSAL.

После завершения интерактивной части потока проверки подлинности верните управление в MSAL, переопределив [`Activity`][Activity] .[`OnActivityResult()`][OnActivityResult] метод.

В переопределении вызовите MSAL. NET `AuthenticationContinuationHelper` .`SetAuthenticationContinuationEventArgs()` метод, возвращающий управление MSAL в конце интерактивной части потока проверки подлинности.

```csharp
protected override void OnActivityResult(int requestCode,
                                         Result resultCode,
                                         Intent data)
{
    base.OnActivityResult(requestCode, resultCode, data);

    // Return control to MSAL
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(requestCode,
                                                                            resultCode,
                                                                            data);
}
```

## <a name="update-the-android-manifest-for-system-webview-support"></a>Обновление манифеста Android для поддержки System WebView 

Для поддержки System WebView файл *AndroidManifest.xml* должен содержать следующие значения:

```xml
<activity android:name="microsoft.identity.client.BrowserTabActivity" android:configChanges="orientation|screenSize">
  <intent-filter>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="msal{Client Id}" android:host="auth" />
  </intent-filter>
</activity>
```

`android:scheme`Значение создается из URI перенаправления, настроенного на портале приложения. Например, если URI перенаправления — `msal4a1aa1d5-c567-49d0-ad0b-cd957a47f842://auth` , `android:scheme` запись в манифесте будет выглядеть, как в следующем примере:

```xml
<data android:scheme="msal4a1aa1d5-c567-49d0-ad0b-cd957a47f842" android:host="auth" />
```

Кроме того, можно [создать действие в коде](/xamarin/android/platform/android-manifest#the-basics) , а не вручную изменять *AndroidManifest.xml*. Чтобы создать действие в коде, сначала создайте класс, включающий `Activity` атрибут и `IntentFilter` атрибут.

Ниже приведен пример класса, представляющего значения XML-файла:

```csharp
  [Activity]
  [IntentFilter(new[] { Intent.ActionView },
        Categories = new[] { Intent.CategoryBrowsable, Intent.CategoryDefault },
        DataHost = "auth",
        DataScheme = "msal{client_id}")]
  public class MsalActivity : BrowserTabActivity
  {
  }
```

### <a name="use-system-webview-in-brokered-authentication"></a>Использование System WebView при проверке подлинности через посредника

Чтобы использовать System WebView в качестве резервной для интерактивной проверки подлинности, когда вы настроили приложение для использования проверки подлинности через посредника и на устройстве не установлен брокер, включите MSAL для записи ответа проверки подлинности с помощью URI перенаправления брокера. MSAL попытается выполнить проверку подлинности с помощью системной WebView по умолчанию на устройстве, когда обнаружит, что брокер недоступен. Использование этого значения по умолчанию приведет к сбою, так как URI перенаправления настроен для использования брокера, а System WebView не знает, как использовать его для возврата в MSAL. Чтобы устранить эту проблему, создайте _Фильтр с намерением_ с помощью URI перенаправления брокера, настроенного ранее. Добавьте фильтр намерения, изменив манифест приложения, как показано в следующем примере:

```xml
<!--Intent filter to capture System WebView or Authenticator calling back to our app after sign-in-->
<activity
      android:name="microsoft.identity.client.BrowserTabActivity">
    <intent-filter>
          <action android:name="android.intent.action.VIEW" />
          <category android:name="android.intent.category.DEFAULT" />
          <category android:name="android.intent.category.BROWSABLE" />
          <data android:scheme="msauth"
              android:host="Enter_the_Package_Name"
              android:path="/Enter_the_Signature_Hash" />
    </intent-filter>
</activity>
```

Замените имя пакета, зарегистрированное в портал Azure, на `android:host=` значение. Замените хэш ключа, зарегистрированный в портал Azure, на `android:path=` значение. Хэш подписи *не* должен быть закодирован в URL-адресе. Убедитесь, что в начале хэша подписи отображается начальная косая черта ( `/` ).

### <a name="xamarinforms-43x-manifest"></a>Манифест Xamarin. Forms 4.3. x

Xamarin. Forms 4.3. x создает код, который присваивает `package` атрибуту значение `com.companyname.{appName}` в *AndroidManifest.xml*. Если используется `DataScheme` как `msal{client_id}` , может потребоваться изменить значение в соответствии со значением `MainActivity.cs` пространства имен.

## <a name="android-11-support"></a>Поддержка Android 11

Чтобы использовать системный браузер и аутентификацию через посредника в Android 11, необходимо сначала объявить эти пакеты, чтобы они были видны приложению. Приложения, предназначенные для Android 10 (API 29) и более ранних версий, могут запрашивать в операционной системе список пакетов, доступных на устройстве в любое заданное время. Для обеспечения конфиденциальности и безопасности Android 11 сокращает видимость пакетов до списка пакетов ОС по умолчанию и пакетов, указанных в файле *AndroidManifest.xml* приложения. 

Чтобы разрешить приложению проходить проверку подлинности с помощью системного браузера и брокера, добавьте следующий раздел в *AndroidManifest.xml*:

```xml
<!-- Required for API Level 30 to make sure the app can detect browsers and other apps where communication is needed.-->
<!--https://developer.android.com/training/basics/intents/package-visibility-use-cases-->
<queries>
  <package android:name="com.azure.authenticator" />
  <package android:name="{Package Name}" />
  <package android:name="com.microsoft.windowsintune.companyportal" />
  <!-- Required for API Level 30 to make sure the app detect browsers
      (that don't support custom tabs) -->
  <intent>
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.BROWSABLE" />
    <data android:scheme="https" />
  </intent>
  <!-- Required for API Level 30 to make sure the app can detect browsers that support custom tabs -->
  <!-- https://developers.google.com/web/updates/2020/07/custom-tabs-android-11#detecting_browsers_that_support_custom_tabs -->
  <intent>
    <action android:name="android.support.customtabs.action.CustomTabsService" />
  </intent>
</queries>
``` 

Замените на `{Package Name}` имя пакета приложения. 

Обновленный манифест, который теперь включает поддержку для браузера системы и проверки подлинности через посредника, должен выглядеть следующим образом:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:versionCode="1" android:versionName="1.0" package="com.companyname.XamarinDev">
    <uses-sdk android:minSdkVersion="21" android:targetSdkVersion="30" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <application android:theme="@android:style/Theme.NoTitleBar">
        <activity android:name="microsoft.identity.client.BrowserTabActivity" android:configChanges="orientation|screenSize">
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="msal4a1aa1d5-c567-49d0-ad0b-cd957a47f842" android:host="auth" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />
                <data android:scheme="msauth" android:host="com.companyname.XamarinDev" android:path="/Fc4l/5I4mMvLnF+l+XopDuQ2gEM=" />
            </intent-filter>
        </activity>
    </application>
    <!-- Required for API Level 30 to make sure we can detect browsers and other apps we want to
     be able to talk to.-->
    <!--https://developer.android.com/training/basics/intents/package-visibility-use-cases-->
    <queries>
        <package android:name="com.azure.authenticator" />
        <package android:name="com.companyname.xamarindev" />
        <package android:name="com.microsoft.windowsintune.companyportal" />
        <!-- Required for API Level 30 to make sure we can detect browsers
        (that don't support custom tabs) -->
        <intent>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="https" />
        </intent>
        <!-- Required for API Level 30 to make sure we can detect browsers that support custom tabs -->
        <!-- https://developers.google.com/web/updates/2020/07/custom-tabs-android-11#detecting_browsers_that_support_custom_tabs -->
        <intent>
            <action android:name="android.support.customtabs.action.CustomTabsService" />
        </intent>
    </queries>
</manifest>
```

## <a name="use-the-embedded-web-view-optional"></a>Использовать внедренное веб-представление (необязательно)

По умолчанию MSAL.NET использует системный веб-браузер. Этот браузер позволяет получить единый вход (SSO) с помощью веб-приложений и других приложений. В некоторых редких случаях может потребоваться, чтобы система использовала встроенное веб-представление.

В этом примере кода показано, как настроить встроенное веб-представление.

```csharp
bool useEmbeddedWebView = !app.IsSystemWebViewAvailable;

var authResult = AcquireTokenInteractive(scopes)
 .WithParentActivityOrWindow(parentActivity)
 .WithEmbeddedWebView(useEmbeddedWebView)
 .ExecuteAsync();
```

Дополнительные сведения см. в статье [Использование веб-браузеров для MSAL.NET и в](msal-net-web-browsers.md) [браузере Xamarin Android System](msal-net-system-browser-android-considerations.md).

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="general-tips"></a>Общие советы

- Обновите существующий пакет NuGet MSAL.NET до [последней версии MSAL.NET](https://www.nuget.org/packages/Microsoft.Identity.Client/).
- Убедитесь, что Xamarin. Forms находится в последней версии.
- Убедитесь, что Xamarin. Android. support. v4 находится в последней версии.
- Убедитесь, что все пакеты Xamarin. Android. support предназначены для последней версии.
- Очистите или перестройте приложение.
- В Visual Studio попробуйте установить максимальное число параллельных сборок проекта равным **1**. Для этого выберите **Параметры**  >  **проекты и решения**  >  **Создание и запуск**  >  **максимального числа параллельных сборок проектов**.
- Если вы создаете из командной строки и используете команду `/m` , попробуйте удалить этот элемент из команды.

### <a name="error-the-name-authenticationcontinuationhelper-doesnt-exist-in-the-current-context"></a>Ошибка: имя Аусентикатионконтинуатионхелпер не существует в текущем контексте

Если ошибка указывает, что `AuthenticationContinuationHelper` не существует в текущем контексте, возможно, Visual Studio неправильно обновил файл *Android. csproj \** . Иногда путь к файлу в `<HintPath>` элементе неправильно содержит `netstandard13` вместо `monoandroid90` .

Этот пример содержит правильный путь к файлу:

```xml
<Reference Include="Microsoft.Identity.Client, Version=3.0.4.0, Culture=neutral, PublicKeyToken=0a613f4dd989e8ae,
           processorArchitecture=MSIL">
  <HintPath>..\..\packages\Microsoft.Identity.Client.3.0.4-preview\lib\monoandroid90\Microsoft.Identity.Client.dll</HintPath>
</Reference>
```

## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения см. в примере [мобильного приложения Xamarin, использующего платформу Microsoft Identity](https://github.com/azure-samples/active-directory-xamarin-native-v2#android-specific-considerations). В следующей таблице перечислены соответствующие сведения в файле сведений.

| Образец | Платформа | Описание |
| ------ | -------- | ----------- |
|[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin. iOS, Android, UWP | Простое приложение Xamarin. Forms, которое показывает, как использовать MSAL для проверки подлинности личных учетных записей Майкрософт и Azure AD через конечную точку Azure AD 2,0. В приложении также показано, как получить доступ к Microsoft Graph и показать получившийся маркер. <br>![Схема потока проверки подлинности](media/msal-net-xamarin-android-considerations/topology.png) |

<!-- REF LINKS -->
[PublicClientApplication]: /dotnet/api/microsoft.identity.client.publicclientapplication
[OnActivityResult]: /dotnet/api/android.app.activity.onactivityresult
[Activity]: /dotnet/api/android.app.activity
