---
title: Создание экземпляра конфиденциального клиентского приложения (MSAL.NET) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать конфиденциальное клиентское приложение с параметрами конфигурации с помощью библиотеки проверки подлинности Microsoft для .NET (MSAL.NET).
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
ms.openlocfilehash: d477c419bb677a6b8f24a3aae26c403e47cc96cb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99583948"
---
# <a name="instantiate-a-confidential-client-application-with-configuration-options-using-msalnet"></a>Создание экземпляра конфиденциального клиентского приложения с параметрами конфигурации с помощью MSAL.NET

В этой статье описывается создание экземпляра [конфиденциального клиентского приложения](msal-client-applications.md) с помощью библиотеки проверки подлинности Майкрософт для .net (MSAL.NET).  Экземпляр приложения создается с параметрами конфигурации, определенными в файле параметров.

Перед инициализацией приложения необходимо сначала [зарегистрировать](quickstart-register-app.md) его, чтобы приложение можно было интегрировать с платформой Microsoft Identity. После регистрации может потребоваться следующая информация (которую можно найти в портал Azure):

- Идентификатор клиента (строка, представляющая GUID)
- URL-адрес поставщика удостоверений (с именем экземпляра) и аудитория входа для приложения. Эти два параметра вместе называются полномочиями.
- Идентификатор клиента, если вы пишете бизнес-приложение исключительно для вашей организации (также называется одноклиентским приложением).
- Секрет приложения (строка секрета клиента) или сертификат (типа X509Certificate2), если это конфиденциальное клиентское приложение.
- Для веб-приложений и иногда для общедоступных клиентских приложений (в частности, если приложению требуется использовать брокер) вы также настроили redirectUri, где поставщик удостоверений свяжется с приложением с маркерами безопасности.

## <a name="configure-the-application-from-the-config-file"></a>Настройка приложения из файла конфигурации
Имена свойств параметров в MSAL.NET соответствуют именам свойств `AzureADOptions` в ASP.NET Core, поэтому не нужно писать какой бы то ни было никакого связывающего кода.

Конфигурация ASP.NET Core приложения описана в *appsettings.js* файле:

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "[Enter the domain of your tenant, e.g. contoso.onmicrosoft.com]",
    "TenantId": "[Enter 'common', or 'organizations' or the Tenant Id (Obtained from the Azure portal. Select 'Endpoints' from the 'App registrations' blade and use the GUID in any of the URLs), e.g. da41245a5-11b3-996c-00a8-4d99re19f292]",
    "ClientId": "[Enter the Client Id (Application ID obtained from the Azure portal), e.g. ba74781c2-53c2-442a-97c2-3d60re42f403]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath ": "/signout-callback-oidc",

    "ClientSecret": "[Copy the client secret added to the app from the Azure portal]"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

Начиная с версии MSAL.NET v3. x, можно настроить конфиденциальное клиентское приложение в файле конфигурации.

В классе, где требуется настроить и создать экземпляр приложения, объявите `ConfidentialClientApplicationOptions` объект.  Свяжите конфигурацию, считанную из источника (включая appconfig.jsв файле), с экземпляром параметров приложения, используя `IConfigurationRoot.Bind()` метод из [Microsoft.Extensions.Configфигурации. Пакет NuGet BINDER](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Binder):

```csharp
using Microsoft.Identity.Client;

private ConfidentialClientApplicationOptions _applicationOptions;
_applicationOptions = new ConfidentialClientApplicationOptions();
configuration.Bind("AzureAD", _applicationOptions);
```

Это позволяет привязать содержимое раздела "AzureAD" *appsettings.jsв* файле к соответствующим свойствам `ConfidentialClientApplicationOptions` объекта.  Затем создайте `ConfidentialClientApplication` объект:

```csharp
IConfidentialClientApplication app;
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
        .Build();
```

## <a name="add-runtime-configuration"></a>Добавить конфигурацию среды выполнения
В конфиденциальном клиентском приложении обычно имеется кэш для каждого пользователя. Поэтому необходимо получить кэш, связанный с пользователем, и сообщить построителю приложений, что вы хотите его использовать. Аналогичным образом, может иметься динамически вычисленный URI перенаправления. В этом случае код выглядит следующим образом:

```csharp
IConfidentialClientApplication app;
var request = httpContext.Request;
var currentUri = UriHelper.BuildAbsolute(request.Scheme, request.Host, request.PathBase, _azureAdOptions.CallbackPath ?? string.Empty);
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
       .WithRedirectUri(currentUri)
       .Build();
TokenCache userTokenCache = _tokenCacheProvider.SerializeCache(app.UserTokenCache,httpContext, claimsPrincipal);
```

