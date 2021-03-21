---
title: Замечания по Xamarin iOS (MSAL.NET) | Службы
titleSuffix: Microsoft identity platform
description: Сведения о вопросах использования Xamarin iOS в библиотеке проверки подлинности Майкрософт для .NET (MSAL.NET).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/09/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 62eb4ab9eb6e4b0e7be0f7aadae1173950d21615
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98064493"
---
# <a name="considerations-for-using-xamarin-ios-with-msalnet"></a>Рекомендации по использованию Xamarin iOS с MSAL.NET

При использовании библиотеки проверки подлинности Майкрософт для .NET (MSAL.NET) в Xamarin iOS необходимо выполнить следующие действия:

- Переопределите и реализуйте `OpenUrl` функцию в `AppDelegate` .
- Включите группы цепочки ключей.
- Включите совместное использование кэша маркеров.
- Включите доступ к цепочке ключей.
- Узнайте об известных проблемах с iOS 12 и iOS 13 и аутентификацией.

## <a name="implement-openurl"></a>Реализация OpenUrl

Переопределите `OpenUrl` метод `FormsApplicationDelegate` производного класса и вызовите `AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs` . Приведем пример:

```csharp
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

Кроме того, выполните следующие задачи.

* Определите схему URI перенаправления.
* Требовать разрешения для приложения на вызов другого приложения.
* Иметь определенную форму для URI перенаправления.
* [Зарегистрируйте URI перенаправления](quickstart-register-app.md#add-a-redirect-uri) в портал Azure.

### <a name="enable-keychain-access"></a>Включение доступа к цепочке ключей

Чтобы включить доступ к цепочке ключей, убедитесь, что у приложения есть группа доступа к цепочке ключей. Группу доступа к цепочке ключей можно задать при создании приложения с помощью `WithIosKeychainSecurityGroup()` API.

Чтобы воспользоваться преимуществами кэша и единого входа (SSO), задайте для группы доступа к цепочке ключей одно и то же значение во всех приложениях.

В этом примере программы установки используется MSAL 4. x:

```csharp
var builder = PublicClientApplicationBuilder
     .Create(ClientId)
     .WithIosKeychainSecurityGroup("com.microsoft.adalcache")
     .Build();
```

Также включите доступ к цепочке ключей в `Entitlements.plist` файле. Используйте следующую группу доступа или собственную группу доступа.

```xml
<dict>
  <key>keychain-access-groups</key>
  <array>
    <string>$(AppIdentifierPrefix)com.microsoft.adalcache</string>
  </array>
</dict>
```

При использовании `WithIosKeychainSecurityGroup()` API MSAL автоматически добавляет группу безопасности в конец *идентификатора группы* приложения ( `AppIdentifierPrefix` ). MSAL добавляет группу безопасности, так как при создании приложения в Xcode оно будет выполнять те же действия. Именно поэтому `Entitlements.plist` права в файле должны быть включены `$(AppIdentifierPrefix)` перед группой доступа к цепочке ключей.

Дополнительные сведения см. в документации по назначениям [iOS](https://developer.apple.com/documentation/security/keychain_services/keychain_items/sharing_access_to_keychain_items_among_a_collection_of_apps).

### <a name="enable-token-cache-sharing-across-ios-applications"></a>Включение общего доступа к кэшу маркеров в приложениях iOS

Начиная с MSAL 2. x можно указать группу доступа к цепочке ключей, чтобы сохранить кэш маркеров в нескольких приложениях. Этот параметр позволяет использовать кэш маркеров в нескольких приложениях с одной группой доступа к цепочке ключей. Кэш маркеров можно совместно использовать в приложениях [ADAL.NET](https://aka.ms/adal-net) , MSAL.NET Xamarin. iOS и в собственных приложениях iOS, разработанных в [ADAL. ObjC](https://github.com/AzureAD/azure-activedirectory-library-for-objc) или [MSAL. ObjC](https://github.com/AzureAD/microsoft-authentication-library-for-objc).

При совместном использовании кэша маркеров вы разрешаете единый вход (SSO) для всех приложений, использующих одну группу доступа к цепочке ключей.

Чтобы включить этот общий доступ к кэшу, используйте `WithIosKeychainSecurityGroup()` метод, чтобы задать для группы доступа к цепочке ключей одно и то же значение во всех приложениях, использующих общий кэш. В первом примере кода в этой статье показано, как использовать метод.

Ранее в этой статье вы узнали, что MSAL добавляет `$(AppIdentifierPrefix)` каждый раз при использовании `WithIosKeychainSecurityGroup()` API. MSAL добавляет этот элемент, так как идентификатор команды `AppIdentifierPrefix` гарантирует, что только приложения, созданные одним издателем, могут использовать доступ к цепочке ключей.

> [!NOTE]
> Свойство `KeychainSecurityGroup` является нерекомендуемым. Вместо этого следует использовать свойство `iOSKeychainSecurityGroup`. `TeamId`Префикс не требуется при использовании `iOSKeychainSecurityGroup` .

### <a name="use-microsoft-authenticator"></a>Использование Microsoft Authenticator

Приложение может использовать Microsoft Authenticator в качестве брокера для включения:

- **Единый** вход. при включении единого входа пользователям не нужно входить в каждое приложение.
- **Идентификация устройства**: используйте идентификацию устройства для проверки подлинности при доступе к сертификату устройства. Этот сертификат создается на устройстве при его присоединении к рабочей области. Приложение будет готово, если администраторы клиента включают условный доступ, связанный с устройствами.
- **Проверка идентификации приложения**. когда приложение вызывает брокер, ему передается URL-адрес перенаправления. Брокер проверяет URL-адрес перенаправления.

Дополнительные сведения о том, как включить брокер, см. [в статье использование Microsoft Authenticator или Microsoft Intune корпоративный портал в приложениях Xamarin iOS и Android](msal-net-use-brokers-with-xamarin-apps.md).

## <a name="known-issues-with-ios-12-and-authentication"></a>Известные проблемы с iOS 12 и аутентификацией

Корпорация Майкрософт выпустила [Советы по безопасности](https://github.com/aspnet/AspNetCore/issues/4647) о несовместимости между iOS 12 и некоторыми типами проверки подлинности. Несовместимость нарушит вход в социальные, WSFed и OIDC. Советы по безопасности помогут вам понять, как удалить ASP.NET ограничения безопасности из приложений, чтобы обеспечить их совместимость с iOS 12.

При разработке MSAL.NET приложений в Xamarin iOS вы можете столкнуться с бесконечным циклом при попытке войти на веб-сайты из iOS 12. Такое поведение аналогично этой ошибке ADAL в GitHub: [бесконечный цикл при попытке входа на веб-сайт из iOS 12 #1329](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/issues/1329).

Вы также можете увидеть перерыв в ASP.NET Core проверки подлинности OIDC в iOS 12 Safari. Дополнительные сведения см. в этой [WebKit статье](https://bugs.webkit.org/show_bug.cgi?id=188165).

## <a name="known-issues-with-ios-13-and-authentication"></a>Известные проблемы с iOS 13 и аутентификация

Если приложению требуется поддержка условного доступа или проверки подлинности на сертификатах, включите приложение для взаимодействия с приложением Microsoft Authenticator Broker. Затем MSAL отвечает за обработку запросов и ответов между приложением и Microsoft Authenticator.

В iOS 13 компания Apple внесла изменение в API, удаляя возможность приложения считывать исходное приложение при получении ответа от внешнего приложения через пользовательские схемы URL-адресов.

Документация Apple о состояниях [уиаппликатионопенурлоптионссаурцеаппликатионкэй](https://developer.apple.com/documentation/uikit/uiapplicationopenurloptionssourceapplicationkey?language=objc) :

> *Если запрос поступил из другого приложения, принадлежащего вашей команде, UIKit присваивает этому ключу значение идентификатора этого приложения. Если идентификатор группы исходного приложения отличается от идентификатора текущего приложения, то значение ключа равно nil.*

Это изменение нарушается для MSAL, так как оно использовалось `UIApplication.SharedApplication.OpenUrl` для проверки обмена данными между MSAL и Microsoft Authenticator приложением.

Кроме того, в iOS 13 разработчик должен предоставить контроллер представления при использовании `ASWebAuthenticationSession` .

Приложение затронет, если вы собираетесь с помощью Xcode 11 и используете либо брокер iOS, либо `ASWebAuthenticationSession` .

В таких случаях используйте [MSAL.NET 4.4.0 +](https://www.nuget.org/packages/Microsoft.Identity.Client/) , чтобы включить успешную проверку подлинности.

### <a name="additional-requirements"></a>Дополнительные требования

- При использовании последних библиотек MSAL убедитесь, что на устройстве установлена **версия Microsoft Authenticator 6.3.19 +** .
- При обновлении до MSAL.NET 4.4.0 + обновите `LSApplicationQueriesSchemes` в файле *info. plist* и добавьте `msauthv3` :

    ```xml
    <key>LSApplicationQueriesSchemes</key>
    <array>
         <string>msauthv2</string>
         <string>msauthv3</string>
    </array>
    ```

    Добавление `msauthv3` в *info. plist* необходимо для обнаружения наличия последней Microsoft Authenticator приложения на устройстве, которое поддерживает iOS 13.

## <a name="report-an-issue"></a>Сообщить о проблеме

Если у вас есть вопросы или вы хотите сообщить о проблемах, обнаруженных в MSAL.NET, откройте ошибку в репозитории [AzureAD/Microsoft-Authentication-Library-for-DotNet](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues) на сайте GitHub.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о свойствах для Xamarin iOS см. в разделе [рекомендации для iOS](https://github.com/Azure-Samples/active-directory-xamarin-native-v2/tree/master/1-Basic#ios-specific-considerations) в следующем примере файла readme.md:

Образец | Платформа | Описание
------ | -------- | -----------
[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin iOS, Android, универсальная платформа Windows (UWP) | Простое приложение Xamarin Forms, которое показывает, как использовать MSAL для проверки подлинности личных учетных записей Майкрософт и Azure AD через конечную точку Azure AD 2,0. В приложении также показано, как использовать полученный маркер для доступа к Microsoft Graph.

<!--- https://github.com/Azure-Samples/active-directory-xamarin-native-v2/blob/master/ReadmeFiles/Topology.png -->
