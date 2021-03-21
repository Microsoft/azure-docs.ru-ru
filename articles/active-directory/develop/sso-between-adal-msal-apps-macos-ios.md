---
title: Единый вход между приложениями ADAL & MSAL (iOS/macOS) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как совместно использовать единый вход для приложений ADAL и MSAL
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: ''
ms.custom: aaddev
ms.openlocfilehash: 396e9cfeace8791a59dec4a9c9c7203212f57304
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99584253"
---
# <a name="how-to-sso-between-adal-and-msal-apps-on-macos-and-ios"></a>Практические руководства. единый вход между приложениями ADAL и MSAL в macOS и iOS

Библиотека проверки подлинности Майкрософт (MSAL) для iOS может совместно использовать единый режим SSO с помощью [ADAL цели-C](https://github.com/AzureAD/azure-activedirectory-library-for-objc) между приложениями. Вы можете перенести свои приложения в MSAL в удобном для вас темпе, гарантируя, что пользователи будут иметь преимущество от единого входа между приложениями, даже с помощью приложений на основе ADAL и MSAL.

Если вы ищете рекомендации по настройке единого входа между приложениями с помощью пакета SDK для MSAL, см. статью [Автоматический единый вход между несколькими приложениями](single-sign-on-macos-ios.md#silent-sso-between-apps). В этой статье рассматривается единый вход между ADAL и MSAL.

Конкретные особенности реализации единого входа зависят от версии ADAL, которую вы используете.

## <a name="adal-27x"></a>ADAL 2.7. x

В этом разделе рассматриваются различия единого входа между MSAL и ADAL 2.7. x

### <a name="cache-format"></a>Формат кэша

ADAL 2.7. x может читать формат кэша MSAL. Нет необходимости делать ничего особенного для единого входа между приложениями с помощью версии ADAL 2.7. x. Однако следует иметь в виду различия в идентификаторах учетных записей, поддерживаемых этими двумя библиотеками.

### <a name="account-identifier-differences"></a>Различия между идентификаторами учетных записей

MSAL и ADAL используют разные идентификаторы учетных записей. ADAL использует UPN в качестве первичного идентификатора учетной записи. MSAL использует неотображаемый идентификатор учетной записи, основанный на ИДЕНТИФИКАТОРе объекта и ИДЕНТИФИКАТОРе клиента для учетных записей AAD, и `sub` утверждение для других типов учетных записей.

При получении `MSALAccount` объекта в MSAL результат он содержит идентификатор учетной записи в `identifier` свойстве. Приложение должно использовать этот идентификатор для последующих запросов в автоматическом режиме.

Кроме `identifier` `MSALAccount` того, объект содержит отображаемый Идентификатор с именем `username` . Это преобразуется `userId` в ADAL. `username` не считается уникальным идентификатором и может изменяться в любое время, поэтому его следует использовать только для сценариев обратной совместимости с ADAL. MSAL поддерживает запросы кэша с помощью `username` или `identifier` , где `identifier` рекомендуется использовать запрос.

В следующей таблице приведены различия между идентификаторами учетных записей ADAL и MSAL:

| Идентификатор учетной записи                | MSAL                                                         | ADAL 2.7. x      | Старая версия ADAL (до ADAL 2.7. x) |
| --------------------------------- | ------------------------------------------------------------ | --------------- | ------------------------------ |
| отображаемый Идентификатор            | `username`                                                   | `userId`        | `userId`                       |
| уникальный неотображаемый идентификатор | `identifier`                                                 | `homeAccountId` | Н/Д                            |
| Идентификатор учетной записи не известен               | Запрос всех учетных записей через `allAccounts:` API в `MSALPublicClientApplication` | Н/Д             | Н/Д                            |

Это интерфейс, `MSALAccount` предоставляющий такие идентификаторы:

```objc
@protocol MSALAccount <NSObject>

/*!
 Displayable user identifier. Can be used for UI and backward compatibility with ADAL.
 */
@property (readonly, nullable) NSString *username;

/*!
 Unique identifier for the account.
 Save this for account lookups from cache at a later point.
 */
@property (readonly, nullable) NSString *identifier;

/*!
 Host part of the authority string used for authentication based on the issuer identifier.
 */
@property (readonly, nonnull) NSString *environment;

/*!
 ID token claims for the account.
 Can be used to read additional information about the account, e.g. name
 Will only be returned if there has been an id token issued for the client Id for the account's source tenant.
 */
@property (readonly, nullable) NSDictionary<NSString *, NSString *> *accountClaims;

@end
```

### <a name="sso-from-msal-to-adal"></a>Единый вход из MSAL в ADAL

Если у вас есть приложение MSAL и ADAL, и пользователь сначала подписывается в приложение на основе MSAL, вы можете получить единый вход в приложении ADAL, сохранив `username` `MSALAccount` объект из объекта и передав его в приложение на основе ADAL как `userId` . ADAL может затем найти сведения об учетной записи автоматически с помощью `acquireTokenSilentWithResource:clientId:redirectUri:userId:completionBlock:` API.

### <a name="sso-from-adal-to-msal"></a>Единый вход из ADAL в MSAL

Если у вас есть приложение MSAL и ADAL, и пользователь сначала подписывается в приложение на основе ADAL, вы можете использовать идентификаторы пользователя ADAL для поиска учетных записей в MSAL. Это также применимо при переходе с ADAL на MSAL.

#### <a name="adals-homeaccountid"></a>Хомеаккаунтид ADAL

ADAL 2.7. x возвращает `homeAccountId` `ADUserInformation` объект в результате через это свойство:

```objc
/*! Unique AAD account identifier across tenants based on user's home OID/home tenantId. */
@property (readonly) NSString *homeAccountId;
```

`homeAccountId` в ADAL является аналогом `identifier` в MSAL. Этот идентификатор можно сохранить для использования в MSAL для поиска учетных записей с помощью `accountForIdentifier:error:` API.

#### <a name="adals-userid"></a>ADAL `userId`

Если параметр `homeAccountId` недоступен или имеется только отображаемый Идентификатор, можно использовать ADAL `userId` для поиска учетной записи в MSAL.

В MSAL сначала найдите учетную запись с помощью `username` или `identifier` . Всегда используйте `identifier` для запроса, если он есть и используется только `username` в качестве резервного. Если учетная запись найдена, используйте учетную запись в `acquireTokenSilent` вызовах.

Objective-C.

```objc
NSString *msalIdentifier = @"previously.saved.msal.account.id";
MSALAccount *account = nil;
    
if (msalIdentifier)
{
    // If you have MSAL account id returned either from MSAL as identifier or ADAL as homeAccountId, use it
    account = [application accountForIdentifier:@"my.account.id.here" error:nil];
}
else
{
    // Fallback to ADAL userId for migration
    account = [application accountForUsername:@"adal.user.id" error:nil];
}
    
if (!account)
{
  // Account not found.
  return;
}

MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"user.read"] account:account];
[application acquireTokenSilentWithParameters:silentParameters completionBlock:completionBlock];
```

Swift:

```swift
        
let msalIdentifier: String?
var account: MSALAccount
        
do {
  if let msalIdentifier = msalIdentifier {
    account = try application.account(forIdentifier: msalIdentifier)
  }
  else {
    account = try application.account(forUsername: "adal.user.id") 
  }
             
  let silentParameters = MSALSilentTokenParameters(scopes: ["user.read"], account: account)          
  application.acquireTokenSilent(with: silentParameters) {
    (result: MSALResult?, error: Error?) in
    // handle result
  }  
} catch let error as NSError {
  // handle error or account not found
}
```



MSAL поддерживаемые API для поиска учетной записи:

```objc
/*!
 Returns account for the given account identifier (received from an account object returned in a previous acquireToken call)
 
 @param  error      The error that occurred trying to get the accounts, if any, if you're
                    not interested in the specific error pass in nil.
 */
- (nullable MSALAccount *)accountForIdentifier:(nonnull NSString *)identifier
                                         error:(NSError * _Nullable __autoreleasing * _Nullable)error;
    
/*!
Returns account for for the given username (received from an account object returned in a previous acquireToken call or ADAL)
    
@param  username    The displayable value in UserPrincipleName(UPN) format
@param  error       The error that occurred trying to get the accounts, if any, if you're
                    not interested in the specific error pass in nil.
*/
- (MSALAccount *)accountForUsername:(NSString *)username
                              error:(NSError * __autoreleasing *)error;
```

## <a name="adal-2x-266"></a>ADAL 2. x — 2.6.6

В этом разделе рассматриваются различия единого входа между MSAL и ADAL 2. x-2.6.6.

Старые версии ADAL изначально не поддерживают формат кэша MSAL. Однако, чтобы обеспечить плавное перемещение из ADAL в MSAL, MSAL может читать старый формат кэша ADAL без повторного запроса учетных данных пользователя.

Так как `homeAccountId` она недоступна в более старых версиях ADAL, вам потребуется выполнить поиск учетных записей с помощью `username` :

```objc
/*!
 Returns account for for the given username (received from an account object returned in a previous acquireToken call or ADAL)

 @param  username    The displayable value in UserPrincipleName(UPN) format
 @param  error       The error that occurred trying to get the accounts, if any.  If you're not interested in the specific error pass in nil.
 */
- (MSALAccount *)accountForUsername:(NSString *)username
                              error:(NSError * __autoreleasing *)error;
```

Пример:

Objective-C.


```objc
MSALAccount *account = [application accountForUsername:@"adal.user.id" error:nil];;
MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"user.read"] account:account];
[application acquireTokenSilentWithParameters:silentParameters completionBlock:completionBlock];
```

Swift:

```swift
do {
  let account = try application.account(forUsername: "adal.user.id")          
  let silentParameters = MSALSilentTokenParameters(scopes: ["user.read"], account: account)
  application.acquireTokenSilent(with: silentParameters) { 
    (result: MSALResult?, error: Error?) in
    // handle result
  }   
} catch let error as NSError { 
  // handle error or account not found
}
```



Кроме того, можно прочитать все учетные записи, которые также будут считывать сведения об учетной записи из ADAL:

Objective-C.

```objc
NSArray *accounts = [application allAccounts:nil];
    
if ([accounts count] == 0)
{
  // No account found.
  return; 
}
if ([accounts count] > 1)
{
  // You might want to display an account picker to user in actual application
  // For this sample we assume there's only ever one account in cache
  return;
}
    ``
MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"user.read"] account:accounts[0]];
[application acquireTokenSilentWithParameters:silentParameters completionBlock:completionBlock];
```

Swift:

```swift
      
do {
  let accounts = try application.allAccounts()
  if accounts.count == 0 {
    // No account found.
    return
  }
  if accounts.count > 1 {
    // You might want to display an account picker to user in actual application
    // For this sample we assume there's only ever one account in cache
    return
  }
  
  let silentParameters = MSALSilentTokenParameters(scopes: ["user.read"], account: accounts[0])
  application.acquireTokenSilent(with: silentParameters) {
    (result: MSALResult?, error: Error?) in
    // handle result or error  
  }  
} catch let error as NSError { 
  // handle error
}
```



## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в [Потоки проверки подлинности и сценарии приложений](authentication-flows-app-scenarios.md)
