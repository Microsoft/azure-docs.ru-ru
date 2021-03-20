---
title: Настройка поставщиков удостоверений (MSAL iOS/macOS) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как использовать различные центры, такие как B2C, облака независимых и гостевые пользователи, с MSAL для iOS и macOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev
ms.openlocfilehash: d8a176fff0da932d0fafd40b9ab895b635acc5f6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96169449"
---
# <a name="how-to-configure-msal-for-ios-and-macos-to-use-different-identity-providers"></a>Как настроить MSAL для iOS и macOS для использования разных поставщиков удостоверений

В этой статье показано, как настроить приложение библиотеки проверки подлинности Майкрософт для iOS и macOS (MSAL) для различных центров, таких как Azure Active Directory (Azure AD), "бизнес-потребитель" (B2C), независимых Clouds и Guest Users.  В этой статье вы обычно можете представить себе полномочия в качестве поставщика удостоверений.

## <a name="default-authority-configuration"></a>Конфигурация центра по умолчанию

`MSALPublicClientApplication` настроен с URL-адресом центра по умолчанию `https://login.microsoftonline.com/common` , который подходит для большинства сценариев Azure Active Directory (AAD). Если вы не реализуете сложные сценарии, такие как национальные облака или работа с B2C, вам не нужно изменять их.

> [!NOTE]
> Современная проверка подлинности с службы федерации Active Directory (AD FS) в качестве поставщика удостоверений (ADFS) не поддерживается (Дополнительные сведения см. в разделе [ADFS для разработчиков](/windows-server/identity/ad-fs/overview/ad-fs-openid-connect-oauth-flows-scenarios) ). Службы ADFS поддерживаются через Федерацию.

## <a name="change-the-default-authority"></a>Изменение центра по умолчанию

В некоторых сценариях, таких как "бизнес-потребитель" (B2C), может потребоваться изменить центр по умолчанию.

### <a name="b2c"></a>B2C

Для работы с B2C для [библиотеки проверки подлинности Майкрософт (MSAL)](reference-v2-libraries.md) требуется другая конфигурация центра. MSAL распознает один формат URL-адреса центра сертификации как B2C сам по себе. Формат распознанного центра B2C — `https://<host>/tfp/<tenant>/<policy>` , например `https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/B2C_1_SignInPolicy` . Тем не менее можно также использовать любые другие поддерживаемые URL-адреса центра B2C, объявляя полномочия как B2C Authority явным образом.

Для поддержки произвольного формата URL-адреса для B2C `MSALB2CAuthority` можно задать произвольный URL-адрес, например:

Objective-C
```objc
NSURL *authorityURL = [NSURL URLWithString:@"arbitrary URL"];
MSALB2CAuthority *b2cAuthority = [[MSALB2CAuthority alloc] initWithURL:authorityURL
                                                                     error:&b2cAuthorityError];
```
Swift
```swift
guard let authorityURL = URL(string: "arbitrary URL") else {
    // Handle error
    return
}
let b2cAuthority = try MSALB2CAuthority(url: authorityURL)
```

Все центры B2C, которые не используют формат центра B2C по умолчанию, должны быть объявлены как известные центры.

Добавьте разные центры B2C в список известных центров, даже если они отличаются только политикой.

Objective-C
```objc
MSALPublicClientApplicationConfig *b2cApplicationConfig = [[MSALPublicClientApplicationConfig alloc]
                                                               initWithClientId:@"your-client-id"
                                                               redirectUri:@"your-redirect-uri"
                                                               authority:b2cAuthority];
b2cApplicationConfig.knownAuthorities = @[b2cAuthority];
```
Swift
```swift
let b2cApplicationConfig = MSALPublicClientApplicationConfig(clientId: "your-client-id", redirectUri: "your-redirect-uri", authority: b2cAuthority)
b2cApplicationConfig.knownAuthorities = [b2cAuthority]
```

Когда приложение запрашивает новую политику, необходимо изменить URL-адрес центра, так как URL-адрес центра отличается для каждой политики. 

Чтобы настроить приложение B2C, задайте `@property MSALAuthority *authority` экземпляр `MSALB2CAuthority` в `MSALPublicClientApplicationConfig` перед созданием `MSALPublicClientApplication` , например:

Objective-C
```ObjC
    // Create B2C authority URL
    NSURL *authorityURL = [NSURL URLWithString:@"https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/B2C_1_SignInPolicy"];
    
    MSALB2CAuthority *b2cAuthority = [[MSALB2CAuthority alloc] initWithURL:authorityURL
                                                                     error:&b2cAuthorityError];
    if (!b2cAuthority)
    {
        // Handle error
        return;
    }
    
    // Create MSALPublicClientApplication configuration
    MSALPublicClientApplicationConfig *b2cApplicationConfig = [[MSALPublicClientApplicationConfig alloc]
                                                                   initWithClientId:@"your-client-id"
                                                                   redirectUri:@"your-redirect-uri"
                                                                   authority:b2cAuthority];

    // Initialize MSALPublicClientApplication
    MSALPublicClientApplication *b2cApplication =
    [[MSALPublicClientApplication alloc] initWithConfiguration:b2cApplicationConfig error:&error];
    
    if (!b2cApplication)
    {
        // Handle error
        return;
    }
```
Swift
```swift
do{
    // Create B2C authority URL
    guard let authorityURL = URL(string: "https://login.microsoftonline.com/tfp/contoso.onmicrosoft.com/B2C_1_SignInPolicy") else {
        // Handle error
        return
    }
    let b2cAuthority = try MSALB2CAuthority(url: authorityURL)

    // Create MSALPublicClientApplication configuration
    let b2cApplicationConfig = MSALPublicClientApplicationConfig(clientId: "your-client-id", redirectUri: "your-redirect-uri", authority: b2cAuthority)

    // Initialize MSALPublicClientApplication
    let b2cApplication = try MSALPublicClientApplication(configuration: b2cApplicationConfig)
} catch {
    // Handle error
}
```

### <a name="sovereign-clouds"></a>Национальные облака

Если приложение выполняется в независимых облаке, может потребоваться изменить URL-адрес центра в `MSALPublicClientApplication` . В следующем примере задается URL-адрес центра для работы с немецким облаком AAD:

Objective-C
```objc
    NSURL *authorityURL = [NSURL URLWithString:@"https://login.microsoftonline.de/common"];
    MSALAuthority *sovereignAuthority = [MSALAuthority authorityWithURL:authorityURL error:&authorityError];
    
    if (!sovereignAuthority)
    {
        // Handle error
        return;
    }
    
    MSALPublicClientApplicationConfig *applicationConfig = [[MSALPublicClientApplicationConfig alloc]
                                                               initWithClientId:@"your-client-id"
                                                               redirectUri:@"your-redirect-uri"
                                                               authority:sovereignAuthority];
    
    
    MSALPublicClientApplication *sovereignApplication = [[MSALPublicClientApplication alloc] initWithConfiguration:applicationConfig error:&error];
    
    
    if (!sovereignApplication)
    {
        // Handle error
        return;
    }
```
Swift
```swift
do{
    guard let authorityURL = URL(string: "https://login.microsoftonline.de/common") else {
        //Handle error
        return
    }
    let sovereignAuthority = try MSALAuthority(url: authorityURL)
            
    let applicationConfig = MSALPublicClientApplicationConfig(clientId: "your-client-id", redirectUri: "your-redirect-uri", authority: sovereignAuthority)
            
    let sovereignApplication = try MSALPublicClientApplication(configuration: applicationConfig)
} catch {
    // Handle error
}
```

Может потребоваться передать различные области в каждое облако независимых. Какие области для отправки зависят от ресурса, который вы используете. Например, вы можете использовать `"https://graph.microsoft.com/user.read"` в облаке по всему миру и `"https://graph.microsoft.de/user.read"` в немецком облаке.

### <a name="signing-a-user-into-a-specific-tenant"></a>Вход пользователя в конкретный клиент

Если URL-адрес центра имеет значение `"login.microsoftonline.com/common"` , пользователь будет входить в его домашний клиент. Однако некоторым приложениям может потребоваться подписать пользователя в другом клиенте, и некоторые приложения работают только с одним клиентом.

Чтобы подписать пользователя в определенном клиенте, настройте `MSALPublicClientApplication` с помощью определенного центра. Пример:

`https://login.microsoftonline.com/469fdeb4-d4fd-4fde-991e-308a78e4bea4`

Ниже показано, как подписать пользователя в определенном клиенте.

Objective-C
```objc
    NSURL *authorityURL = [NSURL URLWithString:@"https://login.microsoftonline.com/469fdeb4-d4fd-4fde-991e-308a78e4bea4"];
    MSALAADAuthority *tenantedAuthority = [[MSALAADAuthority alloc] initWithURL:authorityURL error:&authorityError];
    
    if (!tenantedAuthority)
    {
        // Handle error
        return;
    }
    
    MSALPublicClientApplicationConfig *applicationConfig = [[MSALPublicClientApplicationConfig alloc]
                                                               initWithClientId:@"your-client-id"
                                                               redirectUri:@"your-redirect-uri"
                                                               authority:tenantedAuthority];
    
    MSALPublicClientApplication *application =
    [[MSALPublicClientApplication alloc] initWithConfiguration:applicationConfig error:&error];
    
    if (!application)
    {
        // Handle error
        return;
    }
```
Swift
```swift
do{
    guard let authorityURL = URL(string: "https://login.microsoftonline.com/469fdeb4-d4fd-4fde-991e-308a78e4bea4") else {
        //Handle error
        return
    }    
    let tenantedAuthority = try MSALAADAuthority(url: authorityURL)
            
    let applicationConfig = MSALPublicClientApplicationConfig(clientId: "your-client-id", redirectUri: "your-redirect-uri", authority: tenantedAuthority)
            
    let application = try MSALPublicClientApplication(configuration: applicationConfig)
} catch {
    // Handle error
}
```

## <a name="supported-authorities"></a>Поддерживаемые центры

### <a name="msalauthority"></a>мсалаусорити

`MSALAuthority`Класс является базовым абстрактным классом для классов центра MSAL. Не пытайтесь создать экземпляр с помощью `alloc` или `new` . Вместо этого либо создайте один из его подклассов напрямую ( `MSALAADAuthority` , `MSALB2CAuthority` ), либо используйте метод фабрики `authorityWithURL:error:` для создания подклассов с помощью URL-адреса центра.

`url`Чтобы получить нормализованный URL-адрес центра, используйте свойство. Дополнительные параметры и компоненты пути или фрагменты, не являющиеся частью полномочий, не будут находиться в возвращенном нормализованном URL-адресе центра.

Ниже приведены подклассы `MSALAuthority` , которые можно создать в зависимости от используемого центра.

### <a name="msalaadauthority"></a>мсалаадаусорити

`MSALAADAuthority` представляет центр AAD. URL-адрес центра сертификации должен иметь следующий формат, где `<port>` является необязательным: `https://<host>:<port>/<tenant>`

### <a name="msalb2cauthority"></a>MSALB2CAuthority

`MSALB2CAuthority` представляет центр B2C. По умолчанию URL-адрес центра B2C должен иметь следующий формат, где `<port>` является необязательным: `https://<host>:<port>/tfp/<tenant>/<policy>` . Однако MSAL также поддерживает другие произвольные форматы центра B2C.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в [Потоки проверки подлинности и сценарии приложений](authentication-flows-app-scenarios.md)