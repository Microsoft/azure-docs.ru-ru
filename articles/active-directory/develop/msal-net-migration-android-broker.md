---
title: Перенос приложений Xamarin Android с помощью брокеров в MSAL.NET
titleSuffix: Microsoft identity platform
description: Узнайте, как перенести приложения Xamarin Android, использующие Microsoft Authenticator или Корпоративный портал Intune из ADAL.NET в MSAL.NET.
author: aiwang
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/31/2020
ms.author: aiwang
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 01af0e620ecb100839f7b1101e5ff9fcfc874eea
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92206675"
---
# <a name="migrate-android-applications-that-use-a-broker-from-adalnet-to-msalnet"></a>Перенос приложений Android, использующих брокер из ADAL.NET, в MSAL.NET

Если у вас уже есть приложение Xamarin Android, использующее библиотеку проверки подлинности Azure Active Directory для .NET (ADAL.NET) и [брокер проверки подлинности](msal-android-single-sign-on.md), то пора перейти на [библиотеку проверки подлинности Майкрософт для .NET ](msal-overview.md) (MSAL.NET).

## <a name="prerequisites"></a>Предварительные условия

* Приложение Xamarin Android, уже интегрированное с брокером ([Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator) или [Корпоративный портал Intune](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal)) и ADAL.NET, которое необходимо перенести в MSAL.NET.

## <a name="step-1-enable-the-broker"></a>Шаг 1. Включение брокера

<table>
<tr><td>Текущий код ADAL:</td><td>MSAL аналога:</td></tr>
<tr><td>
В ADAL.NET поддержка брокера включена для каждого контекста проверки подлинности.

Для вызова брокера в конструкторе было присвоено значение `useBroker` *true* `PlatformParameters` :

```CSharp
public PlatformParameters(
        Activity callerActivity,
        bool useBroker)
```

В коде модуля подготовки страниц для устройств Android для конкретной платформы установите `useBroker` для флага значение true:

```CSharp
page.BrokerParameters = new PlatformParameters(
        this,
        true,
        PromptBehavior.SelectAccount);
```

Затем включите параметры в вызов маркера получения:

```CSharp
AuthenticationResult result =
        await
            AuthContext.AcquireTokenAsync(
                Resource,
                ClientId,
                new Uri(RedirectURI),
                platformParameters)
                .ConfigureAwait(false);
```

</td><td>
В MSAL.NET поддержка брокера включена для каждого Публикклиентаппликатион.

Используйте `WithBroker()` параметр (который по умолчанию имеет значение true), чтобы вызвать Broker:

```CSharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithRedirectUri(redirectUriOnAndroid)
                .Build();
```

Затем в вызове AcquireToken:

```CSharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```
</table>

## <a name="step-2-set-an-activity"></a>Шаг 2. Задание действия

В ADAL.NET вы передали действие (обычно MainActivity) как часть Платформпараметерс, как показано на [шаге 1: включение брокера](#step-1-enable-the-broker).

MSAL.NET также использует действие, но не требуется в обычном использовании Android без брокера. Чтобы использовать брокер, настройте действие для отправки и получения ответов от брокера.

<table>
<tr><td>Текущий код ADAL:</td><td>MSAL аналога:</td></tr>
<tr><td>
Действие передается в Платформпараметерс на платформе Android.

```CSharp
page.BrokerParameters = new PlatformParameters(
          this,
          true,
          PromptBehavior.SelectAccount);
```
</td><td>

В MSAL.NET выполните два действия, чтобы настроить действие для Android:

1. В `MainActivity.cs` Задайте для значение, `App.RootViewController` `MainActivity` чтобы убедиться в наличии действия с вызовом брокера.

    Если оно задано неправильно, может появиться следующее сообщение об ошибке: `"Activity_required_for_android_broker":"Activity is null, so MSAL.NET cannot invoke the Android broker. See https://aka.ms/Brokered-Authentication-for-Android"`

1. В вызове Аккуиретокенинтерактиве используйте `.WithParentActivityOrWindow(App.RootViewController)` и передайте ссылку на действие, которое вы будете использовать. В этом примере будет использоваться MainActivity.

**Пример:**

В *app. CS*:

```CSharp
   public static object RootViewController { get; set; }
```

В *MainActivity. CS*:

```CSharp
   LoadApplication(new App());
   App.RootViewController = this;
```

В вызове AcquireToken:

```CSharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```
</table>

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о вопросах, связанных с Android, при использовании MSAL.NET с Xamarin см. в [статье требования к конфигурации и советы по устранению неполадок для Xamarin Android с MSAL.NET](msal-net-xamarin-android-considerations.md).