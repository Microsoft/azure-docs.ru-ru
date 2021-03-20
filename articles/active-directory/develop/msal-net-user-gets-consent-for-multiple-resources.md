---
title: Получение согласия для нескольких ресурсов (MSAL.NET) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как пользователь может получить предварительное согласие на использование нескольких ресурсов с помощью библиотеки проверки подлинности Майкрософт для .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 04/30/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: d22b4109cdcdc965b1b2e03aba592022c58f773b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99583796"
---
# <a name="user-gets-consent-for-several-resources-using-msalnet"></a>Пользователь получает согласие на использование нескольких ресурсов с помощью MSAL.NET
Платформа Microsoft Identity не позволяет получить маркер для нескольких ресурсов одновременно. При использовании библиотеки проверки подлинности Microsoft для .NET (MSAL.NET) параметр областей в методе получения маркера должен содержать только области для одного ресурса. Однако можно заранее согласиться с несколькими ресурсами, указав дополнительные области с помощью `.WithExtraScopeToConsent` метода Builder.

> [!NOTE]
> Получение согласия для нескольких ресурсов работает для платформы Microsoft Identity, но не для Azure AD B2C. Azure AD B2C поддерживает только разрешение администратора, а не согласие пользователя.

Например, при наличии двух ресурсов с двумя областями:

- HTTPS: \/ /mytenant.onmicrosoft.com/customerapi (с 2 областями `customer.read` и `customer.write` )
- HTTPS: \/ /mytenant.onmicrosoft.com/vendorapi (с 2 областями `vendor.read` и `vendor.write` )

Следует использовать модификатор, `.WithExtraScopeToConsent` который имеет параметр *екстраскопестоконсент* , как показано в следующем примере:

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
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

Это позволит получить маркер доступа для первого веб-API. Затем, чтобы получить доступ к второму веб-API, можно получить маркер из кэша маркеров без вывода сообщений.

```csharp
AcquireTokenSilent(scopesForVendorApi, accounts.FirstOrDefault()).ExecuteAsync();
```
