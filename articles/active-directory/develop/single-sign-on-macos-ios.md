---
title: Настройка единого входа для macOS и iOS
titleSuffix: Microsoft identity platform
description: Узнайте, как настроить единый вход (SSO) в macOS и iOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/03/2020
ms.author: marsma
ms.reviewer: ''
ms.custom: aaddev
ms.openlocfilehash: 25389348476552298ddb947ccb59acb8b3d5bc57
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "80881254"
---
# <a name="how-to-configure-sso-on-macos-and-ios"></a>Как настроить единый вход для macOS и iOS

Библиотека проверки подлинности Microsoft (MSAL) для macOS и iOS поддерживает единый вход (SSO) между приложениями macOS/iOS и браузерами. В этой статье рассматриваются следующие сценарии единого входа:

- [Автоматический единый вход между несколькими приложениями](#silent-sso-between-apps)

Этот тип единого входа работает между несколькими приложениями, распространяемыми одним и тем же разработчиком Apple. Он предоставляет автоматический единый вход (т. е. пользователю не предлагается ввести учетные данные), считывая маркеры обновления, записанные другими приложениями из цепочки ключей, и переводите их на маркер доступа автоматически.  

- [Единый вход через брокер аутентификации](#sso-through-authentication-broker-on-ios)

> [!IMPORTANT]
> Этот поток недоступен в macOS.

Корпорация Майкрософт предоставляет приложения, называемые брокерами, которые обеспечивают единый вход между приложениями разных поставщиков при условии, что мобильное устройство зарегистрировано в Azure Active Directory (AAD). Этот тип единого входа требует установки приложения брокера на устройстве пользователя.

- **Единый вход между MSAL и Safari**

Единый вход достигается с помощью класса [асвебаусентикатионсессион](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession?language=objc) . Он использует существующее состояние входа из других приложений и браузера Safari. Оно не ограничено приложениями, распространяемыми одним и тем же разработчиком Apple, но требует определенного взаимодействия с пользователем.

Если вы используете веб-представление по умолчанию в приложении для входа пользователей, вы получаете автоматический единый вход между приложениями на основе MSAL и Safari. Дополнительные сведения о веб-представлениях, поддерживаемых MSAL, см. в статье [Настройка браузеров и представлений](customize-webviews.md).

> [!IMPORTANT]
> Этот тип единого входа в настоящее время недоступен в macOS. MSAL в macOS поддерживает только Вквебвиев, для которых не поддерживается единый вход с помощью Safari. 

- **Автоматический единый вход между приложениями ADAL и MSAL macOS/iOS**

Цель MSAL — C поддерживает миграцию и единый вход с приложениями ADAL, основанными на C. Приложения должны распространяться одним и тем же разработчиком Apple.

Инструкции по единого входа между приложениями ADAL и MSAL см. [в статье единый вход между приложениями ADAL и MSAL в macOS и iOS](sso-between-adal-msal-apps-macos-ios.md) .

## <a name="silent-sso-between-apps"></a>Автоматический единый вход между приложениями

MSAL поддерживает общий доступ SSO через группы доступа к цепочке ключей iOS.

Чтобы включить единый вход для приложений, необходимо выполнить следующие действия, которые более подробно описаны ниже.

1. Убедитесь, что все приложения используют один и тот же идентификатор клиента или идентификатор приложения.
1. Убедитесь, что все приложения имеют один и тот же самозаверяющий сертификат от Apple для общего доступа к цепочкам ключей.
1. Запросите одинаковое назначение цепочки ключей для каждого из приложений.
1. Сообщите пакетам SDK MSAL о общей цепочке ключей, которую вы хотите использовать, если она отличается от используемой по умолчанию.

### <a name="use-the-same-client-id-and-application-id"></a>Использовать тот же идентификатор клиента и идентификатор приложения

Чтобы платформа Microsoft Identity могла знать, какие приложения могут совместно использовать токены, эти приложения должны использовать один и тот же идентификатор клиента или приложения. Это уникальный идентификатор, предоставленный вам при регистрации первого приложения на портале.

Способ, которым платформа Microsoft Identity сообщает приложениям, использующим одинаковый идентификатор приложения, определяется их **URI перенаправления**. Каждое приложение может иметь несколько кодов URI перенаправления, зарегистрированных на портале подключения. Каждое приложение в наборе получит уникальный код URI перенаправления. Пример:

URI перенаправления App1: `msauth.com.contoso.mytestapp1://auth`  
URI перенаправления App2: `msauth.com.contoso.mytestapp2://auth`  
URI перенаправления App3: `msauth.com.contoso.mytestapp3://auth`  

> [!IMPORTANT]
> Формат URI перенаправления должен быть совместим с форматом, поддерживаемым MSAL, который описан в статье [требования к формату URI перенаправления MSAL](redirect-uris-ios.md#msal-redirect-uri-format-requirements).

### <a name="setup-keychain-sharing-between-applications"></a>Настройка совместного использования цепочки ключей между приложениями

Дополнительные сведения о включении общего доступа к цепочке ключей см. в статье [Добавление возможностей](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html) Apple. Важно то, что вы решаете, что нужно вызвать цепочку ключей, и добавляете эту возможность ко всем приложениям, которые будут задействованы в единый вход.

После правильной настройки прав вы увидите `entitlements.plist` в каталоге проекта файл, который содержит примерно следующий пример:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>keychain-access-groups</key>
    <array>
        <string>$(AppIdentifierPrefix)com.myapp.mytestapp</string>
        <string>$(AppIdentifierPrefix)com.myapp.mycache</string>
    </array>
</dict>
</plist>
```

#### <a name="add-a-new-keychain-group"></a>Добавить новую группу цепочки ключей

Добавьте новую группу цепочки ключей в **возможности** проекта. Группа цепочки ключей должна быть:
* `com.microsoft.adalcache` в iOS 
* `com.microsoft.identity.universalstorage` в macOS.

![Пример цепочки ключей](media/single-sign-on-macos-ios/keychain-example.png)

Дополнительные сведения см. в разделе [цепочки ключей](howto-v2-keychain-objc.md).

## <a name="configure-the-application-object"></a>Настройка объекта приложения

После того как в каждом из ваших приложений будет включено назначение цепочки ключей, и вы готовы использовать единый вход, настройте `MSALPublicClientApplication` его с помощью группы доступа к цепочке ключей, как показано в следующем примере:

Objective-C.

```objc
NSError *error = nil;
MSALPublicClientApplicationConfig *configuration = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<my-client-id>"];
configuration.cacheConfig.keychainSharingGroup = @"my.keychain.group";
    
MSALPublicClientApplication *application = [[MSALPublicClientApplication alloc] initWithConfiguration:configuration error:&error];
```

Swift:

```swift
let config = MSALPublicClientApplicationConfig(clientId: "<my-client-id>")
config.cacheConfig.keychainSharingGroup = "my.keychain.group"

do {
   let application = try MSALPublicClientApplication(configuration: config)
  // continue on with application
} catch let error as NSError {
  // handle error here
}
```

> [!WARNING]
> При совместном использовании цепочки ключей в приложениях любое приложение может удалять пользователей или даже все маркеры в приложении.
> Это особенно важно, если у вас есть приложения, использующие маркеры для выполнения фоновой работы.
> Совместное использование цепочки ключей означает, что вы должны быть очень осторожными, когда приложение использует Microsoft Identity SDK для удаления операций.

Вот и все! Пакет SDK для Microsoft Identity теперь будет предоставлять учетные данные для совместного использования во всех приложениях. Список учетных записей также будет совместно использоваться несколькими экземплярами приложения.

## <a name="sso-through-authentication-broker-on-ios"></a>Единый вход через брокер аутентификации в iOS

MSAL обеспечивает поддержку проверки подлинности через посредника с Microsoft Authenticator. Microsoft Authenticator предоставляет единый вход для зарегистрированных устройств AAD, а также помогает вашему приложению следовать политикам условного доступа.

Чтобы включить единый вход с помощью брокера проверки подлинности для приложения, выполните следующие действия.

1. Зарегистрируйте формат URI перенаправления, совместимого с брокером, для приложения в файле info. plist приложения. Формат URI для перенаправления, совместимый с брокером: `msauth.<app.bundle.id>://auth` . Замените "<app.bundle.id>" "ИДЕНТИФИКАТОРом пакета вашего приложения. Пример:

    ```xml
    <key>CFBundleURLSchemes</key>
    <array>
        <string>msauth.<app.bundle.id></string>
    </array>
    ```

1. Добавьте следующие схемы в сведения о приложении. plist в разделе `LSApplicationQueriesSchemes` :

    ```xml
    <key>LSApplicationQueriesSchemes</key>
    <array>
         <string>msauthv2</string>
         <string>msauthv3</string>
    </array>
    ```

1. Добавьте следующий объект в `AppDelegate.m` файл для управления обратными вызовами:

    Objective-C.
    
    ```objc
    - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options
    {
        return [MSALPublicClientApplication handleMSALResponse:url sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```
    
    Swift:
    
    ```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
    }
    ```
    
**Если вы используете Xcode 11**, вместо этого следует поместить обратный вызов MSAL в `SceneDelegate` файл.
Если UISceneDelegate и UIApplicationDelegate поддерживаются для совместимости с более старыми версиями iOS, то обратный вызов MSAL потребуется поместить в оба файла.

Objective-C.

```objc
 - (void)scene:(UIScene *)scene openURLContexts:(NSSet<UIOpenURLContext *> *)URLContexts
 {
     UIOpenURLContext *context = URLContexts.anyObject;
     NSURL *url = context.URL;
     NSString *sourceApplication = context.options.sourceApplication;
     
     [MSALPublicClientApplication handleMSALResponse:url sourceApplication:sourceApplication];
 }
```

Swift:

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

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в [Потоки проверки подлинности и сценарии приложений](authentication-flows-app-scenarios.md)