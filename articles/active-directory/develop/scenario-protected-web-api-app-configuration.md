---
title: Настройка защищенных веб-API приложений | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать защищенный веб-API и настроить код своего приложения.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 5206c2295ee7c01b4a2908e59da1cfdd8782bccd
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102517724"
---
# <a name="protected-web-api-code-configuration"></a>Защищенный веб-API: конфигурация кода

Чтобы настроить код для защищенного веб-API, изучите следующие сведения:

- Определяет интерфейсы API как защищенные.
- Настройка токена носителя.
- Проверка маркера.

## <a name="what-defines-aspnet-and-aspnet-core-apis-as-protected"></a>Что определяет интерфейсы API ASP.NET и ASP.NET Core как защищенные?

Как и веб-приложения, веб-API ASP.NET и ASP.NET Core защищены, так как их действия контроллера начинаются с атрибута **[авторизовать]** . Действия контроллера могут вызываться только в том случае, если API вызывается с разрешенным удостоверением.

Оцените следующие вопросы.

- Только приложение может вызывать веб-API. Как API знает удостоверение приложения, которое его вызывает?
- Если приложение вызывает API от имени пользователя, каково удостоверение пользователя?

## <a name="bearer-token"></a>Маркер носителя

Токен носителя, заданный в заголовке при вызове приложения, содержит сведения об удостоверении приложения. Он также содержит сведения о пользователе, если веб-приложение не принимает вызовы между службами из управляющего приложения.

Ниже приведен пример кода C#, в котором показан клиент, вызывающий API после получения маркера с помощью библиотеки проверки подлинности Майкрософт для .NET (MSAL.NET):

```csharp
var scopes = new[] {$"api://.../access_as_user"};
var result = await app.AcquireToken(scopes)
                      .ExecuteAsync();

httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call the web API.
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
```

> [!IMPORTANT]
> Клиентское приложение запрашивает токен носителя на платформе идентификации Майкрософт *для веб-API*. Веб-API — это единственное приложение, которое должно проверять маркер и просматривать содержащиеся в нем утверждения. Клиентские приложения никогда не должны пытаться проверить утверждения в токенах.
>
> В будущем для веб-API может потребоваться шифрование маркера. Это требование помешает доступу к клиентским приложениям, которые могут просматривать маркеры доступа.

## <a name="jwtbearer-configuration"></a>Конфигурация JwtBearer

В этом разделе описывается настройка токена носителя.

### <a name="config-file"></a>Файл конфигурации

```Json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    /*
      You need specify the TenantId only if you want to accept access tokens from a single tenant
     (line-of-business app).
      Otherwise, you can leave them set to common.
      This can be:
      - A GUID (Tenant ID = Directory ID)
      - 'common' (any organization and personal accounts)
      - 'organizations' (any organization)
      - 'consumers' (Microsoft personal accounts)
    */
    "TenantId": "common"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

#### <a name="case-where-you-used-a-custom-app-id-uri-for-your-web-api"></a>Если вы использовали идентификатор URI пользовательского приложения для веб-API

Если вы приняли URI идентификатора приложения, предложенный порталом регистрации приложений, вам не нужно указывать аудиторию (см. [URI идентификатора приложения и области действия](scenario-protected-web-api-app-registration.md#application-id-uri-and-scopes)). В противном случае следует добавить `Audience` свойство, значение которого является универсальным кодом ресурса (URI) идентификатора приложения для веб-API.

```Json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common",
    "Audience": "custom App ID URI for your web API"
  },
  // more lines
}
```

### <a name="code-initialization"></a>Инициализация кода

При вызове приложения в действии контроллера, которое содержит атрибут **[Authorization]** , ASP.NET и ASP.NET Core извлекают маркер доступа из токена носителя заголовка авторизации. Затем маркер доступа перенаправляется в по промежуточного слоя JwtBearer, который вызывает расширения Microsoft IdentityModel для .NET.

#### <a name="microsoftidentityweb"></a>Microsoft. Identity. Web

Корпорация Майкрософт рекомендует использовать пакет NuGet [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web) при разработке веб-API с ASP.NET Core.

_Microsoft. Identity. Web_ обеспечивает связь между ASP.NET Core, по промежуточного слоя для проверки подлинности и [библиотекой проверки подлинности Майкрософт (MSAL)](msal-overview.md) для .NET. Она обеспечивает более четкий и надежный опыт разработки и использует мощь платформы идентификации Майкрософт и Azure AD B2C.

#### <a name="using-microsoftidentityweb-templates"></a>Использование шаблонов Microsoft. Identity. Web

Вы можете создать веб-API с нуля с помощью шаблонов проектов Microsoft. Identity. Web. Дополнительные сведения см. в [статье шаблон проекта Microsoft. Identity. Web-Web API](https://aka.ms/ms-id-web/webapi-project-templates).

#### <a name="starting-from-an-existing-aspnet-core-31-application"></a>Начиная с существующего приложения ASP.NET Core 3,1

Сегодня ASP.NET Core 3,1 использует библиотеку Microsoft. AspNetCore. AzureAD. UI. По промежуточного слоя инициализируется в файле startup. cs.

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
```

По промежуточного слоя добавляется в веб-API с помощью следующей инструкции:

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
  services.AddAuthentication(AzureADDefaults.JwtBearerAuthenticationScheme)
          .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
}
```

 В настоящее время шаблоны ASP.NET Core создают веб-интерфейсы API Azure Active Directory (Azure AD), которые входят в состав пользователей вашей организации или в любой организации. Они не входят в систему пользователей с личными учетными записями. Однако можно изменить шаблоны для использования платформы Microsoft Identity с помощью [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web) замените код в *Startup. CS*:

```csharp
using Microsoft.Identity.Web;
```

```csharp
public void ConfigureServices(IServiceCollection services)
{
 // Adds Microsoft Identity platform (AAD v2.0) support to protect this API
 services.AddMicrosoftIdentityWebApiAuthentication(Configuration, "AzureAd");

 services.AddControllers();
}
```

можно также написать следующий (эквивалентный)

```csharp
public void ConfigureServices(IServiceCollection services)
{
 // Adds Microsoft Identity platform (AAD v2.0) support to protect this API
 services.AddAuthentication(AzureADDefaults.JwtBearerAuthenticationScheme)
             .AddMicrosoftIdentityWebApi(Configuration, "AzureAd");

services.AddControllers();
}
```

> [!NOTE]
> Если используется Microsoft. Identity. Web и не задан параметр `Audience` in *appsettings.json*, используется следующее:
> -  `$"{ClientId}"` Если вы установили [маркер доступа](scenario-protected-web-api-app-registration.md#accepted-token-version) , который принимает версию `2` , или для Azure AD B2C веб-API.
> - `$"api://{ClientId}` во всех остальных случаях (для [маркеров доступа](access-tokens.md)v 1.0).
> Дополнительные сведения см. в разделе [Исходный код](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/Resource/RegisterValidAudience.cs#L70-L83)Microsoft. Identity. Web.

Предыдущий фрагмент кода извлекается из [пошагового руководства по ASP.NET Core веб-API](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/blob/63087e83326e6a332d05fee6e1586b66d840b08f/1.%20Desktop%20app%20calls%20Web%20API/TodoListService/Startup.cs#L23-L28). Подробные сведения о **аддмикрософтидентитивебапиаусентикатион** доступны в [Microsoft. Identity. Web](microsoft-identity-web.md). Этот метод вызывает [аддмикрософтидентитивебапи](/dotnet/api/microsoft.identity.web.microsoftidentitywebapiauthenticationbuilderextensions.addmicrosoftidentitywebapi), который сам указывает по промежуточного слоя на проверку маркера.

## <a name="token-validation"></a>Проверка маркеров

В предыдущем фрагменте по промежуточного слоя JwtBearer (например, по промежуточного слоя OpenID Connect Connect в веб-приложениях) проверяет маркер на основе значения `TokenValidationParameters` . Маркер расшифровывается по мере необходимости, извлекаются утверждения и проверяется подпись. По промежуточного слоя затем проверяет маркер, проверяя эти данные:

- Аудитория: маркер предназначен для веб-API.
- IT: она была выдана для приложения, которому разрешено вызывать веб-API.
- Издатель: он был выдан доверенной службой маркеров безопасности (STS).
- Срок действия: время существования в диапазоне.
- Сигнатура: она не была изменена.

Также могут быть специальные проверки. Например, можно проверить, что ключи подписывания, внедренные в маркер, являются доверенными и что этот маркер не воспроизводится. Наконец, для некоторых протоколов требуются определенные проверки.

### <a name="validators"></a>Проверяющие элементы управления

Шаги проверки фиксируются в проверяющих элементах управления, предоставляемых библиотеками [Microsoft IdentityModel Extensions для .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) с открытым исходным кодом. Проверяющие элементы управления определяются в исходном файле библиотеки [Microsoft. IdentityModel. Tokens/проверяющих элементов. CS](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/blob/master/src/Microsoft.IdentityModel.Tokens/Validators.cs).

В следующей таблице описаны проверяющие элементы управления.

| Проверяющий элемент управления | Описание |
|---------|---------|
| **валидатеаудиенце** | Гарантирует, что маркер предназначен для приложения, которое проверяет маркер. |
| **ValidateIssuer** | Гарантирует, что маркер был выдан доверенной службой STS, то есть от доверенного лица. |
| **валидатеиссуерсигнингкэй** | Гарантирует, что приложение, проверяющее маркер, доверяет ключу, который использовался для подписания маркера. Существует особый случай, когда ключ внедряется в маркер. Но в этом случае обычно не возникает. |
| **валидателифетиме** | Гарантирует, что маркер все еще или уже является допустимым. Проверяющий элемент управления проверяет, находится ли время существования маркера в диапазоне, указанном в **NotBefore** и **истечении срока действия** утверждений. |
| **валидатесигнатуре** | Гарантирует, что маркер не был изменен. |
| **валидатетокенреплай** | Гарантирует, что маркер не воспроизводится. Существует особый случай использования некоторых протоколов OneTime. |

#### <a name="customizing-token-validation"></a>Настройка проверки маркера

Проверяющие элементы управления связаны со свойствами класса **TokenValidationParameters** . Свойства инициализируются из конфигурации ASP.NET и ASP.NET Core.

В большинстве случаев изменять параметры не требуется. Приложения, не являющиеся отдельными клиентами, являются исключениями. Эти веб-приложения принимают пользователей из любой организации или из личных учетных записей Майкрософт. Издатели в этом случае должны быть проверены. Microsoft. Identity. Web также отвечает за проверку издателя. Дополнительные сведения см. в разделе Microsoft. Identity. Web [аадиссуервалидатор](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web/Resource/AadIssuerValidator.cs).

Если вы хотите настроить параметры проверки маркера в ASP.NET Core, используйте следующий фрагмент кода в *загрузочном. CS*:

```c#
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApi(Configuration);
services.Configure<JwtBearerOptions>(JwtBearerDefaults.AuthenticationScheme, options =>
{
  var existingOnTokenValidatedHandler = options.Events.OnTokenValidated;
  options.Events.OnTokenValidated = async context =>
  {
       await existingOnTokenValidatedHandler(context);
      // Your code to add extra configuration that will be executed after the current event implementation.
      options.TokenValidationParameters.ValidIssuers = new[] { /* list of valid issuers */ };
      options.TokenValidationParameters.ValidAudiences = new[] { /* list of valid audiences */};
  };
});
```

В следующем примере кода для ASP.NET MVC показано, как выполнить проверку пользовательской лексемы:

https://github.com/azure-samples/active-directory-dotnet-webapi-manual-jwt-validation

## <a name="token-validation-in-azure-functions"></a>Проверка токенов в функциях Azure

Вы также можете проверить входящие маркеры доступа в функциях Azure. Примеры такой проверки можно найти в следующих примерах кода на сайте GitHub:

- .NET: [Azure-Samples/MS-Identity-DotNet-webapi-азурефунктионс](https://github.com/Azure-Samples/ms-identity-dotnet-webapi-azurefunctions)
- Node.js: [Azure-Samples/MS-Identity-NodeJS-webapi-азурефунктионс](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-azurefunctions)
- Python: [Azure-Samples/MS-Identity-Python-webapi-азурефунктионс)](https://github.com/Azure-Samples/ms-identity-python-webapi-azurefunctions)

## <a name="next-steps"></a>Следующие шаги

Перейдите к следующей статье в этом сценарии, [Проверьте области и роли приложений в коде](scenario-protected-web-api-verification-scope-app-roles.md).
