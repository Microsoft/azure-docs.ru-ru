---
title: Инициализация клиентских приложений MSAL.NET | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как инициализировать общедоступные клиентские и конфиденциальные клиентские приложения с помощью библиотеки проверки подлинности Майкрософт для .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/18/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 6616b0573019703f287d3be521569f5953aeb032
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98600291"
---
# <a name="initialize-client-applications-using-msalnet"></a>Инициализация клиентских приложений с помощью MSAL.NET
В этой статье описывается инициализация общедоступного клиента и конфиденциальных клиентских приложений с помощью библиотеки проверки подлинности Майкрософт для .NET (MSAL.NET).  Дополнительные сведения о типах клиентских приложений и параметрах конфигурации приложений см. в [обзоре](msal-client-applications.md).

При использовании MSAL.NET 3. x рекомендуемым способом создания экземпляра приложения является использование построителей приложений: `PublicClientApplicationBuilder` и `ConfidentialClientApplicationBuilder` . Они предлагают мощный механизм настройки приложения либо из кода, либо из файла конфигурации, либо путем смешивания обоих подходов.

## <a name="prerequisites"></a>Предварительные условия
Перед инициализацией приложения необходимо сначала [зарегистрировать его](quickstart-register-app.md) , чтобы приложение можно было интегрировать с платформой Microsoft Identity.  После регистрации может потребоваться следующая информация (которую можно найти в портал Azure):

- Идентификатор клиента (строка, представляющая GUID)
- URL-адрес поставщика удостоверений (с именем экземпляра) и аудитория входа для приложения. Эти два параметра вместе называются полномочиями.
- Идентификатор клиента, если вы пишете бизнес-приложение исключительно для вашей организации (также называется одноклиентским приложением).
- Секрет приложения (строка секрета клиента) или сертификат (типа X509Certificate2), если это конфиденциальное клиентское приложение.
- Для веб-приложений и иногда для общедоступных клиентских приложений (в частности, если приложению требуется использовать брокер) вы также настроили redirectUri, где поставщик удостоверений свяжется с приложением с маркерами безопасности.

## <a name="ways-to-initialize-applications"></a>Способы инициализации приложений
Существует множество различных способов создания экземпляров клиентских приложений.

### <a name="initializing-a-public-client-application-from-code"></a>Инициализация общедоступного клиентского приложения из кода

Следующий код создает экземпляр общедоступного клиентского приложения, пользователей для входа в Microsoft Azure общедоступное облако с их рабочими и учебными учетными записями или их личными учетными записями Майкрософт.

```csharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-code"></a>Инициализация конфиденциального клиентского приложения из кода

Аналогичным образом, следующий код создает конфиденциальное приложение (веб-приложение, расположенное в `https://myapp.azurewebsites.net` ), обрабатывающее маркеры от пользователей в Microsoft Azure общедоступном облаке с рабочими и учебными учетными записями или их личными учетными записями Майкрософт. Приложение определяется поставщиком удостоверений путем предоставления секрета клиента:

```csharp
string redirectUri = "https://myapp.azurewebsites.net";
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(clientSecret)
    .WithRedirectUri(redirectUri )
    .Build();
```

Как вы, возможно, знакомы в рабочей среде, вместо того чтобы использовать секрет клиента, может потребоваться предоставить сертификат Azure AD. Код будет выглядеть следующим образом:

```csharp
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithCertificate(certificate)
    .WithRedirectUri(redirectUri )
    .Build();
```

### <a name="initializing-a-public-client-application-from-configuration-options"></a>Инициализация общедоступного клиентского приложения на основе параметров конфигурации

Следующий код создает экземпляр общедоступного клиентского приложения на основе объекта конфигурации, который может быть заполнен программно или прочитан из файла конфигурации:

```csharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-configuration-options"></a>Инициализация конфиденциального клиентского приложения на основе параметров конфигурации

Тот же тип шаблона применяется к конфиденциальным клиентским приложениям. Кроме того, можно добавить другие параметры с помощью `.WithXXX` модификаторов (здесь есть сертификат).

```csharp
ConfidentialClientApplicationOptions options = GetOptions(); // your own method
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(options)
    .WithCertificate(certificate)
    .Build();
```

## <a name="builder-modifiers"></a>Модификаторы построителя

В фрагментах кода, использующих построители приложений, `.With` можно применить несколько методов в качестве модификаторов (например, `.WithCertificate` и `.WithRedirectUri` ). 

### <a name="modifiers-common-to-public-and-confidential-client-applications"></a>Модификаторы, общие для общедоступных и конфиденциальных клиентских приложений

Модификаторы, которые можно задать в построителе общедоступных клиентов или конфиденциальных клиентских приложений:

|Модификатор | Описание|
|--------- | --------- |
|`.WithAuthority()` 7 переопределений | Устанавливает центр по умолчанию для центра приложений Azure AD с возможностью выбора облака Azure, аудитории, клиента (идентификатора клиента или имени домена) или непосредственного URI центра.|
|`.WithAdfsAuthority(string)` | Задает центр по умолчанию приложения в качестве центра ADFS.|
|`.WithB2CAuthority(string)` | Задает центр по умолчанию приложения как центр Azure AD B2C.|
|`.WithClientId(string)` | Переопределяет идентификатор клиента.|
|`.WithComponent(string)` | Задает имя библиотеки с помощью MSAL.NET (по причинам телеметрии). |
|`.WithDebugLoggingCallback()` | При вызове приложение будет вызывать `Debug.Write` просто включение трассировки отладки. Дополнительные сведения см. в статье, посвященной [ведению журналов](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging).|
|`.WithExtraQueryParameters(IDictionary<string,string> eqp)` | Задайте дополнительные параметры запроса уровня приложения, которые будут отправляться во все запросы проверки подлинности. Это может быть переопределяемым на каждом уровне метода получения маркера (с тем же `.WithExtraQueryParameters pattern` ).|
|`.WithHttpClientFactory(IMsalHttpClientFactory httpClientFactory)` | Включает расширенные сценарии, такие как настройка для HTTP-прокси, или принудительное MSAL для использования определенного HttpClient (например, в ASP.NET Core веб-приложений и API).|
|`.WithLogging()` | При вызове приложение вызывает обратный вызов с трассировкой отладки. Дополнительные сведения см. в статье, посвященной [ведению журналов](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging).|
|`.WithRedirectUri(string redirectUri)` | Переопределяет URI перенаправления по умолчанию. В случае общедоступных клиентских приложений это будет полезно для сценариев, использующих брокер.|
|`.WithTelemetry(TelemetryCallback telemetryCallback)` | Задает делегат, используемый для отправки данных телеметрии.|
|`.WithTenantId(string tenantId)` | Переопределяет идентификатор клиента или описание клиента.|

### <a name="modifiers-specific-to-xamarinios-applications"></a>Модификаторы, характерные для приложений Xamarin. iOS

Модификаторы, которые можно задать в построителе общедоступных клиентских приложений для Xamarin. iOS:

|Модификатор | Описание|
|--------- | --------- |
|`.WithIosKeychainSecurityGroup()` | **Только Xamarin. iOS**. задает группу безопасности цепочки ключей iOS (для сохраняемости кэша).|

### <a name="modifiers-specific-to-confidential-client-applications"></a>Модификаторы, характерные для конфиденциальных клиентских приложений

Ниже перечислены модификаторы, которые можно задать в построителе конфиденциальных клиентских приложений.

|Модификатор | Описание|
|--------- | --------- |
|`.WithCertificate(X509Certificate2 certificate)` | Задает сертификат, идентифицирующий приложение в Azure AD.|
|`.WithClientSecret(string clientSecret)` | Задает секрет клиента (пароль приложения), определяющий приложение в Azure AD.|

Эти модификаторы являются взаимоисключающими. Если указать оба значения, MSAL выдаст осмысленное исключение.

### <a name="example-of-usage-of-modifiers"></a>Пример использования модификаторов

Предположим, что ваше приложение является бизнес-приложением, которое предназначено только для вашей организации.  Затем можно написать:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
        .Build();
```

Когда он становится интересным, программирование для национальных облаков теперь упрощено. Если вы хотите, чтобы приложение было многопользовательским приложением в национальном облаке, вы можете написать, например:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAuthority(AzureCloudInstance.AzureUsGovernment, AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

Также существует переопределение для служб ADFS (ADFS 2019 сейчас не поддерживается):
```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

Наконец, если вы являетесь Azure AD B2C разработчиком, вы можете указать свой клиент следующим образом:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```

## <a name="next-steps"></a>Дальнейшие действия

После инициализации клиентского приложения необходимо добавить поддержку входа пользователя, доступного API-интерфейса или и того, и другого.

В документации по сценарию приложения содержатся рекомендации по входу пользователя и получению маркера доступа для доступа к API от имени этого пользователя:

- [Веб-приложение, которое входит в систему пользователей: вход и выход](scenario-web-app-sign-user-sign-in.md)
- [Веб-приложение, вызывающее веб-API: получение маркера](scenario-web-app-call-api-acquire-token.md)
