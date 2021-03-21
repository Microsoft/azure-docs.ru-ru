---
title: Переместить классическое приложение, вызывающее веб-API, в рабочую среду | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как переместить классическое приложение, вызывающее веб-API, в рабочую среду.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 22e61ea767d781dc9da54d61143c1b2524e06e94
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99584420"
---
# <a name="desktop-app-that-calls-web-apis-move-to-production"></a>Классическое приложение, вызывающее веб-API: переместить в рабочую среду

Из этой статьи вы узнаете, как переместить классическое приложение, которое вызывает веб-API, в рабочую среду.

## <a name="handle-errors-in-desktop-applications"></a>Обработку ошибок в приложениях для настольных систем

В разных потоках вы узнали, как обрабатывались ошибки для безошибочных потоков, как показано в фрагментах кода. Вы также видели случаи, когда требуется взаимодействие, как при добавочном согласии и условном доступе.

## <a name="have-the-user-consent-upfront-for-several-resources"></a>Получите согласие пользователя на получение нескольких ресурсов

> [!NOTE]
> Получение согласия для нескольких ресурсов работает для платформы Microsoft Identity, но не для Azure Active Directory (Azure AD) B2C. Azure AD B2C поддерживает только разрешение администратора, а не согласие пользователя.

Вы не можете получить маркер для нескольких ресурсов одновременно с платформой Microsoft Identity. `scopes`Параметр может содержать области только для одного ресурса. Вы можете убедиться, что пользователь заранее предоставил несколько ресурсов с помощью `extraScopesToConsent` параметра.

Например, у вас может быть два ресурса с двумя областями:

- `https://mytenant.onmicrosoft.com/customerapi` с областями `customer.read` и `customer.write`
- `https://mytenant.onmicrosoft.com/vendorapi` с областями `vendor.read` и `vendor.write`

В этом примере используйте `.WithExtraScopesToConsent` Модификатор с `extraScopesToConsent` параметром.

Например:

### <a name="in-msalnet"></a>В MSAL.NET

```csharp
string[] scopesForCustomerApi = new string[]
{
  "https://mytenant.onmicrosoft.com/customerapi/customer.read",
  "https://mytenant.onmicrosoft.com/customerapi/customer.write"
};
string[] scopesForVendorApi = new string[]
{
 "https://mytenant.onmicrosoft.com/vendorapi/vendor.read",
 "https://mytenant.onmicrosoft.com/vendorapi/vendor.write"
};

var accounts = await app.GetAccountsAsync();
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithAccount(accounts.FirstOrDefault())
                     .WithExtraScopesToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

### <a name="in-msal-for-ios-and-macos"></a>В MSAL для iOS и macOS

Objective-C.

```objc
NSArray *scopesForCustomerApi = @[@"https://mytenant.onmicrosoft.com/customerapi/customer.read",
                                @"https://mytenant.onmicrosoft.com/customerapi/customer.write"];

NSArray *scopesForVendorApi = @[@"https://mytenant.onmicrosoft.com/vendorapi/vendor.read",
                              @"https://mytenant.onmicrosoft.com/vendorapi/vendor.write"]

MSALInteractiveTokenParameters *interactiveParams = [[MSALInteractiveTokenParameters alloc] initWithScopes:scopesForCustomerApi webviewParameters:[MSALWebviewParameters new]];
interactiveParams.extraScopesToConsent = scopesForVendorApi;
[application acquireTokenWithParameters:interactiveParams completionBlock:^(MSALResult *result, NSError *error) { /* handle result */ }];
```

Swift:

```swift
let scopesForCustomerApi = ["https://mytenant.onmicrosoft.com/customerapi/customer.read",
                            "https://mytenant.onmicrosoft.com/customerapi/customer.write"]

let scopesForVendorApi = ["https://mytenant.onmicrosoft.com/vendorapi/vendor.read",
                          "https://mytenant.onmicrosoft.com/vendorapi/vendor.write"]

let interactiveParameters = MSALInteractiveTokenParameters(scopes: scopesForCustomerApi, webviewParameters: MSALWebviewParameters())
interactiveParameters.extraScopesToConsent = scopesForVendorApi
application.acquireToken(with: interactiveParameters, completionBlock: { (result, error) in /* handle result */ })
```

Этот вызов получает маркер доступа для первого веб-API.

При вызове второго веб-API вызовите `AcquireTokenSilent` API.

```csharp
AcquireTokenSilent(scopesForVendorApi, accounts.FirstOrDefault()).ExecuteAsync();
```

### <a name="microsoft-personal-account-requires-reconsent-each-time-the-app-runs"></a>При каждом запуске приложения личная учетная запись Майкрософт требует согласия пользователя

Для пользователей личных учетных записей Майкрософт необходимо повторно запрашивать согласие на каждое собственное клиентское (настольное или мобильное приложение) обращение к авторизации. Идентификация собственного клиента по своей природе небезопасна, что противоречит конфиденциальному удостоверению клиентского приложения. Конфиденциальные клиентские приложения обмениваются секретом с платформой Microsoft Identity, чтобы доказать их подлинность. Платформа Microsoft Identity решила устранить эту небезопасность для потребительских услуг, запрашивая согласие пользователя при каждой авторизации приложения.

[!INCLUDE [Common steps to move to production](../../../includes/active-directory-develop-scenarios-production.md)]

## <a name="next-steps"></a>Дальнейшие действия

Чтобы испытать дополнительные примеры, см. раздел [общедоступные клиентские приложения для настольных систем и мобильных устройств](sample-v2-code.md#desktop-and-mobile-public-client-apps).



