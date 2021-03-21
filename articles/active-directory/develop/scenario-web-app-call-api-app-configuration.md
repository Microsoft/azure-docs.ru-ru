---
title: Настройка веб-приложения, вызывающего веб-API | Службы
titleSuffix: Microsoft identity platform
description: Сведения о настройке кода веб-приложения, которое вызывает веб-API
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/25/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: aa377547f7f4961e199ec8d62bf0f1435296f983
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104669310"
---
# <a name="a-web-app-that-calls-web-apis-code-configuration"></a>Веб-приложение, которое вызывает веб-API. Конфигурация кода

Как описано на странице [Веб-приложение, которое выполняет вход пользователей в систему](scenario-web-app-sign-user-overview.md), веб-приложение использует для входа пользователей [поток кода авторизации OAuth 2.0](v2-oauth2-auth-code-flow.md). Этот поток включает два этапа:

1. Запрос кода авторизации. Эта часть делегирует частный диалог с пользователем на платформу удостоверений Майкрософт. Во время этого диалога пользователь входит в систему и предоставляет согласие на использование веб-API. После успешного завершения частного диалога веб-приложение получает код авторизации в URI перенаправления.
1. Запрос маркера доступа для API с помощью активации кода авторизации.

На странице [Веб-приложение, которое выполняет вход пользователей в систему](scenario-web-app-sign-user-overview.md) описан лишь первый этап. На этой странице содержатся сведения об изменении веб-приложения, чтобы оно не только выполняло вход пользователей в систему, но и вызывало веб-API.

## <a name="microsoft-libraries-supporting-web-apps"></a>Библиотеки Майкрософт, поддерживающие веб-приложения

Следующие библиотеки Майкрософт поддерживают веб-приложения.

[!INCLUDE [active-directory-develop-libraries-webapp](../../../includes/active-directory-develop-libraries-webapp.md)]

Выберите нужную вкладку для платформы:

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

## <a name="client-secrets-or-client-certificates"></a>Секреты клиента или сертификаты клиента

Учитывая, что веб-приложение теперь вызывает нисходящий веб-API, предоставьте секрет клиента или сертификат клиента в *appsettings.js* в файле. Можно также добавить раздел, который указывает:

- URL-адрес подчиненного веб-API
- Области, необходимые для вызова API

В следующем примере `GraphBeta` эти параметры задаются в разделе.

```JSON
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-app-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common"

   // To call an API
   "ClientSecret": "[Copy the client secret added to the app from the Azure portal]",
   "ClientCertificates": [
  ]
 },
 "GraphBeta": {
    "BaseUrl": "https://graph.microsoft.com/beta",
    "Scopes": "user.read"
    }
}
```

Вместо секрета клиента можно предоставить сертификат клиента. В следующем фрагменте кода показано использование сертификата, хранящегося в Azure Key Vault.

```JSON
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-app-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common"

   // To call an API
   "ClientCertificates": [
      {
        "SourceType": "KeyVault",
        "KeyVaultUrl": "https://msidentitywebsamples.vault.azure.net",
        "KeyVaultCertificateName": "MicrosoftIdentitySamplesCert"
      }
   ]
  },
  "GraphBeta": {
    "BaseUrl": "https://graph.microsoft.com/beta",
    "Scopes": "user.read"
  }
}
```

*Microsoft. Identity. Web* предоставляет несколько способов описания сертификатов как по конфигурации, так и по коду. Дополнительные сведения см. в [статье Microsoft. Identity. Web — использование сертификатов](https://github.com/AzureAD/microsoft-identity-web/wiki/Using-certificates) на GitHub.

## <a name="startupcs"></a>Startup.cs

Веб-приложению потребуется получить маркер для подчиненного API. Его можно указать, добавив `.EnableTokenAcquisitionToCallDownstreamApi()` строку после `.AddMicrosoftIdentityWebApi(Configuration)` . Эта строка предоставляет `ITokenAcquisition` службу, которую можно использовать в действиях контроллера и страницы. Однако, как вы увидите в следующих двух вариантах, это можно сделать более просто. Вам также потребуется выбрать реализацию кэша маркеров, например `.AddInMemoryTokenCaches()` , в *Startup. CS*:

   ```csharp
   using Microsoft.Identity.Web;

   public class Startup
   {
     // ...
     public void ConfigureServices(IServiceCollection services)
     {
     // ...
     services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
             .AddMicrosoftIdentityWebApp(Configuration, Configuration.GetSection("AzureAd"))
               .EnableTokenAcquisitionToCallDownstreamApi(new string[]{"user.read" })
               .AddInMemoryTokenCaches();
      // ...
     }
     // ...
   }
   ```

Области, передаваемые в `EnableTokenAcquisitionToCallDownstreamApi` , являются необязательными и позволяют веб-приложению запрашивать области и согласие пользователя на эти области при входе в систему. Если не указать области, *Microsoft. Identity. Web* предоставит возможность добавочного согласия.

Если вы не хотите получать маркер самостоятельно, *Microsoft. Identity. Web* предоставляет два механизма вызова веб-API из веб-приложения. Выбор варианта зависит от того, требуется ли вызывать Microsoft Graph или другой API.

### <a name="option-1-call-microsoft-graph"></a>Вариант 1. вызов Microsoft Graph

Если вы хотите вызвать Microsoft Graph, *Microsoft. Identity. Web* позволяет напрямую использовать `GraphServiceClient` (предоставляется Microsoft Graph SDK) в действиях API. Чтобы предоставить Microsoft Graph:

1. Добавьте в проект пакет NuGet [Microsoft. Identity. Web. MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph) .
1. Добавьте `.AddMicrosoftGraph()` после `.EnableTokenAcquisitionToCallDownstreamApi()` в файл *Startup. CS* . `.AddMicrosoftGraph()` имеет несколько переопределений. При использовании переопределения, которое принимает раздел конфигурации в качестве параметра, код становится следующим:

   ```csharp
   using Microsoft.Identity.Web;

   public class Startup
   {
     // ...
     public void ConfigureServices(IServiceCollection services)
     {
     // ...
     services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
             .AddMicrosoftIdentityWebApp(Configuration, Configuration.GetSection("AzureAd"))
               .EnableTokenAcquisitionToCallDownstreamApi(new string[]{"user.read" })
                  .AddMicrosoftGraph(Configuration.GetSection("GraphBeta"))
               .AddInMemoryTokenCaches();
      // ...
     }
     // ...
   }
   ```

### <a name="option-2-call-a-downstream-web-api-other-than-microsoft-graph"></a>Вариант 2. вызов подчиненного веб-интерфейса API, отличного от Microsoft Graph

Для вызова веб-API, отличного от Microsoft Graph, *Microsoft. Identity. Web* предоставляет `.AddDownstreamWebApi()` , который запрашивает маркеры и вызывает нисходящий веб-API.

   ```csharp
   using Microsoft.Identity.Web;

   public class Startup
   {
     // ...
     public void ConfigureServices(IServiceCollection services)
     {
     // ...
     services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
             .AddMicrosoftIdentityWebApp(Configuration, "AzureAd")
               .EnableTokenAcquisitionToCallDownstreamApi(new string[]{"user.read" })
                  .AddDownstreamWebApi("MyApi", Configuration.GetSection("GraphBeta"))
               .AddInMemoryTokenCaches();
      // ...
     }
     // ...
   }
   ```

### <a name="summary"></a>Итоги

Как и в случае с веб-API, можно выбрать различные реализации кэша маркеров. Дополнительные сведения см. в [статье сериализация кэша маркеров в Microsoft. Identity. Web](https://aka.ms/ms-id-web/token-cache-serialization) .

На следующем рисунке показаны различные возможности *Microsoft. Identity. Web* и их влияние на файл *Startup. CS* :

:::image type="content" source="media/scenarios/microsoft-identity-web-startup-cs.svg" alt-text="Блок-схема, показывающая параметры конфигурации службы в загрузочном точке C S для вызова Web API и указания реализации кэша маркеров":::

> [!NOTE]
> Чтобы полностью понять примеры кода, ознакомьтесь с [ASP.NET Core фундаментальными принципами](/aspnet/core/fundamentals)и в частности с [внедрением](/aspnet/core/fundamentals/dependency-injection) и [параметрами](/aspnet/core/fundamentals/configuration/options)зависимостей.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Поскольку вход пользователя делегируется на по промежуточного слоя OpenID Connect Connect (OIDC), необходимо взаимодействовать с процессом OIDC. Взаимодействие зависит от используемой платформы.

Для ASP.NET необходимо подписаться на события ПО промежуточного слоя OIDC:

- Вы можете разрешить ASP.NET Core запросить код авторизации с помощью ПО промежуточного слоя Open ID Connect. ASP.NET или ASP.NET Core разрешит пользователю выполнить вход в систему и предоставить согласие.
- Подпишитесь на веб-приложение, чтобы получить код авторизации. Эта подписка выполняется с помощью делегата C#.
- При получении кода авторизации вы будете использовать библиотеки MSAL для его активации. Полученные маркеры доступа и обновления хранятся в кэше маркеров. Кэш можно использовать в других частях приложения, таких как контроллеры, для автоматического получения других маркеров.

Примеры кода в этой и следующей статьях взяты из [примера веб-приложения ASP.NET](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect). Этот пример можно просмотреть для получения подробной информации о его реализации.

# <a name="java"></a>[Java](#tab/java)

Примеры кода в этой и следующей статьях взяты из примера веб-приложения [Веб-приложение Java, которое вызывает Microsoft Graph](https://github.com/Azure-Samples/ms-identity-java-webapp) с использованием MSAL для Java.
В настоящее время пример позволяет MSAL для Java создать URL-адрес кода авторизации и обработать переход к конечной точке авторизации для платформы удостоверений Майкрософт. Кроме того, можно использовать безопасность спринта для входа пользователя в систему. Этот пример можно просмотреть для получения подробной информации о его реализации.

# <a name="python"></a>[Python](#tab/python)

Примеры кода в этой и следующей статьях взяты из примера веб-приложения [Веб-приложение Python, которое вызывает Microsoft Graph](https://github.com/Azure-Samples/ms-identity-python-webapp) с использованием MSAL.Python.
В настоящее время пример позволяет MSAL.Python создать URL-адрес кода авторизации и обработать переход к конечной точке авторизации для платформы удостоверений Майкрософт. Этот пример можно просмотреть для получения подробной информации о его реализации.

---

## <a name="code-that-redeems-the-authorization-code"></a>Код, который активирует код авторизации

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Microsoft.Identity.Web упрощает ваш код, устанавливая правильные параметры OpenID Connect, подписываясь на событие получения кода и его активации. Для активации кода авторизации дополнительный код не требуется. Сведения о том, как это работает, см. в разделе [Исходный код Microsoft. Identity. Web](https://github.com/AzureAD/microsoft-identity-web/blob/c29f1a7950b940208440bebf0bcb524a7d6bee22/src/Microsoft.Identity.Web/WebAppExtensions/WebAppCallsWebApiAuthenticationBuilderExtensions.cs#L140) .

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

ASP.NET функционирует так же, как ASP.NET Core, за исключением того, что конфигурация OpenID Connect и подписка на событие `OnAuthorizationCodeReceived` выполняется в файле [App_Start\Startup.Auth.cs](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/App_Start/Startup.Auth.cs). Основные понятия также аналогичны ASP.NET Core, за исключением того, что в ASP.NET необходимо указать `RedirectUri` в [Web.config#L15](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/master/WebApp/Web.config#L15). Такая конфигурация менее надежна, чем в ASP.NET Core, так как ее необходимо изменить при развертывании приложения.

Ниже приведен код для Startup.Auth.cs:

```csharp
public partial class Startup
{
  public void ConfigureAuth(IAppBuilder app)
  {
    app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

    app.UseCookieAuthentication(new CookieAuthenticationOptions());

    // Custom middleware initialization. This is activated when the code obtained from a code_grant is present in the query string (&code=<code>).
    app.UseOAuth2CodeRedeemer(
        new OAuth2CodeRedeemerOptions
        {
            ClientId = AuthenticationConfig.ClientId,
            ClientSecret = AuthenticationConfig.ClientSecret,
            RedirectUri = AuthenticationConfig.RedirectUri
        }
      );

  app.UseOpenIdConnectAuthentication(
      new OpenIdConnectAuthenticationOptions
      {
        // The `Authority` represents the v2.0 endpoint - https://login.microsoftonline.com/common/v2.0.
        Authority = AuthenticationConfig.Authority,
        ClientId = AuthenticationConfig.ClientId,
        RedirectUri = AuthenticationConfig.RedirectUri,
        PostLogoutRedirectUri = AuthenticationConfig.RedirectUri,
        Scope = AuthenticationConfig.BasicSignInScopes + " Mail.Read", // A basic set of permissions for user sign-in and profile access "openid profile offline_access"
        TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = false,
            // In a real application, you would use IssuerValidator for additional checks, such as making sure the user's organization has signed up for your app.
            //     IssuerValidator = (issuer, token, tvp) =>
            //     {
            //        //if(MyCustomTenantValidation(issuer))
            //        return issuer;
            //        //else
            //        //    throw new SecurityTokenInvalidIssuerException("Invalid issuer");
            //    },
            //NameClaimType = "name",
        },
        Notifications = new OpenIdConnectAuthenticationNotifications()
        {
            AuthorizationCodeReceived = OnAuthorizationCodeReceived,
            AuthenticationFailed = OnAuthenticationFailed,
        }
      });
  }

  private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification context)
  {
      // Upon successful sign-in, get the access token and cache it by using MSAL.
      IConfidentialClientApplication clientApp = MsalAppBuilder.BuildConfidentialClientApplication(new ClaimsPrincipal(context.AuthenticationTicket.Identity));
      AuthenticationResult result = await clientApp.AcquireTokenByAuthorizationCode(new[] { "Mail.Read" }, context.Code).ExecuteAsync();
  }

  private Task OnAuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> notification)
  {
      notification.HandleResponse();
      notification.Response.Redirect("/Error?message=" + notification.Exception.Message);
      return Task.FromResult(0);
  }
}
```

# <a name="java"></a>[Java](#tab/java)

См. страницу [Веб-приложение, которое выполняет вход пользователей в систему. Конфигурация кода](scenario-web-app-sign-user-app-configuration.md?tabs=java#initialization-code), чтобы понять, как пример кода Java получает код авторизации. После того как приложение получит код, [AuthFilter.java#L51-L56](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthFilter.java#L51-L56) выполняет следующее:

1. Делегирует метод `AuthHelper.processAuthenticationCodeRedirect` в [AuthHelper.java#L67-L97](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthHelper.java#L67-L97).
1. Вызывает `getAuthResultByAuthCode`.

```Java
class AuthHelper {
  // Code omitted
  void processAuthenticationCodeRedirect(HttpServletRequest httpRequest, String currentUri, String fullUrl)
            throws Throwable {

  // Code omitted
  AuthenticationResponse authResponse = AuthenticationResponseParser.parse(new URI(fullUrl), params);

  // Code omitted
  IAuthenticationResult result = getAuthResultByAuthCode(
                    httpRequest,
                    oidcResponse.getAuthorizationCode(),
                    currentUri);

// Code omitted
  }
}
```

Метод `getAuthResultByAuthCode` определяется в [AuthHelper.java#L176](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthHelper.java#L176). Он создает MSAL `ConfidentialClientApplication`, а затем вызывает `acquireToken()` с объектом `AuthorizationCodeParameters`, созданным из кода авторизации.

```Java
   private IAuthenticationResult getAuthResultByAuthCode(
            HttpServletRequest httpServletRequest,
            AuthorizationCode authorizationCode,
            String currentUri) throws Throwable {

        IAuthenticationResult result;
        ConfidentialClientApplication app;
        try {
            app = createClientApplication();

            String authCode = authorizationCode.getValue();
            AuthorizationCodeParameters parameters = AuthorizationCodeParameters.builder(
                    authCode,
                    new URI(currentUri)).
                    build();

            Future<IAuthenticationResult> future = app.acquireToken(parameters);

            result = future.get();
        } catch (ExecutionException e) {
            throw e.getCause();
        }

        if (result == null) {
            throw new ServiceUnavailableException("authentication result was null");
        }

        SessionManagementHelper.storeTokenCacheInSession(httpServletRequest, app.tokenCache().serialize());

        return result;
    }

    private ConfidentialClientApplication createClientApplication() throws MalformedURLException {
        return ConfidentialClientApplication.builder(clientId, ClientCredentialFactory.create(clientSecret)).
                authority(authority).
                build();
    }
```

# <a name="python"></a>[Python](#tab/python)

Будет запрошен поток кода авторизации, как показано на странице [Веб-приложение, которое выполняет вход пользователей в систему. Конфигурация кода](scenario-web-app-sign-user-app-configuration.md?tabs=python#initialization-code). Затем код поступает в функцию `authorized`, которую Flask направляет из URL-адреса `/getAToken`. Полный контекст этого кода см. в разделе [app.py#L30-L44](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L30-L44):

```python
 @app.route("/getAToken")  # Its absolute URL must match your app's redirect_uri set in AAD.
def authorized():
    if request.args['state'] != session.get("state"):
        return redirect(url_for("login"))
    cache = _load_cache()
    result = _build_msal_app(cache).acquire_token_by_authorization_code(
        request.args['code'],
        scopes=app_config.SCOPE,  # Misspelled scope would cause an HTTP 400 error here.
        redirect_uri=url_for("authorized", _external=True))
    if "error" in result:
        return "Login failure: %s, %s" % (
            result["error"], result.get("error_description"))
    session["user"] = result.get("id_token_claims")
    _save_cache(cache)
    return redirect(url_for("index"))
```

---

Вместо секрета клиента конфиденциальное клиентское приложение также может подтвердить свою подлинность с помощью сертификата клиента или утверждения клиента.
Использование утверждений клиента является расширенным сценарием, подробно описанным на странице [Конфиденциальные утверждения клиентов](msal-net-client-assertions.md).

## <a name="token-cache"></a>Кэш маркеров

> [!IMPORTANT]
> Реализация кэша маркеров для веб-приложений или веб-API отличается от реализации для классических приложений, которая часто [основана на файлах](scenario-desktop-acquire-token.md#file-based-token-cache).
> В целях повышения безопасности и производительности важно убедиться, что для веб-приложений и веб-API существует отдельный кэш маркеров в каждой учетной записи пользователя. Сериализацию кэша маркеров также следует выполнять отдельно для каждой учетной записи.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

В руководстве по ASP.NET Core используется внедрение зависимостей, позволяющее выбрать реализацию кэша маркеров в файле Startup.cs для приложения. Microsoft.Identity.Web поставляется с предварительно созданными сериализаторами кэша маркеров, описанными в разделе [о сериализации кэша маркеров](msal-net-token-cache-serialization.md#token-cache-for-a-web-app-confidential-client-application). Интересной возможностью является выбор [кэшей распределенной памяти](/aspnet/core/performance/caching/distributed#distributed-memory-cache) ASP.NET Core:

```csharp
// Use a distributed token cache by adding:
    services.AddMicrosoftIdentityWebAppAuthentication(Configuration, "AzureAd")
            .EnableTokenAcquisitionToCallDownstreamApi(
                initialScopes: new string[] { "user.read" })
            .AddDistributedTokenCaches();

// Then, choose your implementation.
// For instance, the distributed in-memory cache (not cleared when you stop the app):
services.AddDistributedMemoryCache();

// Or a Redis cache:
services.AddStackExchangeRedisCache(options =>
{
 options.Configuration = "localhost";
 options.InstanceName = "SampleInstance";
});

// Or even a SQL Server token cache:
services.AddDistributedSqlServerCache(options =>
{
 options.ConnectionString = _config["DistCache_ConnectionString"];
 options.SchemaName = "dbo";
 options.TableName = "TestCache";
});
```

Дополнительные сведения о поставщиках кэша маркеров см. в статье о [сериализации кэша маркеров](https://aka.ms/ms-id-web/token-cache-serialization) Microsoft. Identity. Web, а также в [учебниках по веб-приложениям ASP.NET Core | Стадия кэша маркеров](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-2-TokenCache) в учебнике по веб-приложениям.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Реализация кэша маркеров для веб-приложений или веб-API отличается от реализации для классических приложений, которая часто [основана на файлах](scenario-desktop-acquire-token.md#file-based-token-cache).

Для реализации веб-приложения можно использовать сеанс ASP.NET или память сервера. Например, ознакомьтесь со сведениями о подключении реализации кэша после создания приложения MSAL.NET в разделе [MsalAppBuilder.cs#L39-L51](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Utils/MsalAppBuilder.cs#L39-L51):

```csharp
public static class MsalAppBuilder
{
 // Omitted code
    public static IConfidentialClientApplication BuildConfidentialClientApplication(ClaimsPrincipal currentUser)
    {
      IConfidentialClientApplication clientapp = ConfidentialClientApplicationBuilder.Create(AuthenticationConfig.ClientId)
            .WithClientSecret(AuthenticationConfig.ClientSecret)
            .WithRedirectUri(AuthenticationConfig.RedirectUri)
            .WithAuthority(new Uri(AuthenticationConfig.Authority))
            .Build();

      // After the ConfidentialClientApplication is created, we overwrite its default UserTokenCache with our implementation.
      MSALPerUserMemoryTokenCache userTokenCache = new MSALPerUserMemoryTokenCache(clientapp.UserTokenCache, currentUser ?? ClaimsPrincipal.Current);

      return clientapp;
  }
```

# <a name="java"></a>[Java](#tab/java)

MSAL Java предоставляет методы для сериализации и десериализации кэша маркеров. Пример для Java обрабатывает сериализацию из сеанса, как показано в методе `getAuthResultBySilentFlow` в [AuthHelper.java#L99-L122](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthHelper.java#L99-L122):

```Java
IAuthenticationResult getAuthResultBySilentFlow(HttpServletRequest httpRequest, HttpServletResponse httpResponse)
      throws Throwable {

  IAuthenticationResult result =  SessionManagementHelper.getAuthSessionObject(httpRequest);

  IConfidentialClientApplication app = createClientApplication();

  Object tokenCache = httpRequest.getSession().getAttribute("token_cache");
  if (tokenCache != null) {
      app.tokenCache().deserialize(tokenCache.toString());
  }

  SilentParameters parameters = SilentParameters.builder(
          Collections.singleton("User.Read"),
          result.account()).build();

  CompletableFuture<IAuthenticationResult> future = app.acquireTokenSilently(parameters);
  IAuthenticationResult updatedResult = future.get();

  // Update session with latest token cache.
  SessionManagementHelper.storeTokenCacheInSession(httpRequest, app.tokenCache().serialize());

  return updatedResult;
}
```

Подробные сведения о классе `SessionManagementHelper` приведены в [примере MSAL для Java](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/SessionManagementHelper.java).

# <a name="python"></a>[Python](#tab/python)

В примере Python один кэш на учетную запись обеспечивается путем повторного создания конфиденциального клиентского приложения для каждого запроса и последующей его сериализации в кэше сеанса Flask:

```python
from flask import Flask, render_template, session, request, redirect, url_for
from flask_session import Session  # https://pythonhosted.org/Flask-Session
import msal
import app_config


app = Flask(__name__)
app.config.from_object(app_config)
Session(app)

# Code omitted here for simplicity

def _load_cache():
    cache = msal.SerializableTokenCache()
    if session.get("token_cache"):
        cache.deserialize(session["token_cache"])
    return cache

def _save_cache(cache):
    if cache.has_state_changed:
        session["token_cache"] = cache.serialize()

def _build_msal_app(cache=None):
    return msal.ConfidentialClientApplication(
        app_config.CLIENT_ID, authority=app_config.AUTHORITY,
        client_credential=app_config.CLIENT_SECRET, token_cache=cache)
```

---

## <a name="next-steps"></a>Дальнейшие действия

На этом этапе при входе пользователя маркер сохраняется в кэше маркеров. Ознакомьтесь со сведениями об использовании кэша в других частях веб-приложения.

[Удаление учетных записей из кэша при глобальном выходе](scenario-web-app-call-api-sign-in.md)
