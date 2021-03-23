---
title: Настройка веб-приложения, которое входит в систему пользователей | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать веб-приложение, которое входит в систему пользователей (конфигурация кода)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: cafd42653ca220670081cff102ba8be2de58f4a1
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104779959"
---
# <a name="web-app-that-signs-in-users-code-configuration"></a>Веб-приложение, которое входит в систему пользователей: конфигурация кода

Узнайте, как настроить код для веб-приложения, которое входит в систему пользователей.

## <a name="microsoft-libraries-supporting-web-apps"></a>Библиотеки Майкрософт, поддерживающие веб-приложения

<!-- This section can be in an include for web app and web APIs -->
Для защиты веб-приложения (и веб-API) используются следующие библиотеки Майкрософт:

[!INCLUDE [active-directory-develop-libraries-webapp](../../../includes/active-directory-develop-libraries-webapp.md)]

Выберите вкладку, соответствующую интересующей вас платформе:

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Фрагменты кода в этой статье и следующие извлекаются из [пошагового руководства по ASP.NET Core веб-приложению, главе 1](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-1-MyOrg).

Для получения подробных сведений о реализации может потребоваться обратиться к этому учебнику.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Фрагменты кода в этой статье и следующие извлекаются из [примера веб-приложения ASP.NET](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect).

Для получения полной информации о реализации можно использовать этот пример.

# <a name="java"></a>[Java](#tab/java)

Фрагменты кода в этой статье и следующие извлекаются из примера [веб-приложения Java, вызывающего Microsoft Graph](https://github.com/Azure-Samples/ms-identity-java-webapp) в MSAL Java.

Для получения полной информации о реализации можно использовать этот пример.

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Фрагменты кода в этой статье и следующие извлекаются из [Node.js вход пользователей в веб-приложение в](https://github.com/Azure-Samples/ms-identity-node) примере в узле MSAL.

Для получения полной информации о реализации можно использовать этот пример.

# <a name="python"></a>[Python](#tab/python)

Фрагменты кода в этой статье и следующие извлекаются из примера [веб-приложения Python, вызывающего Microsoft Graph](https://github.com/Azure-Samples/ms-identity-python-webapp) в MSAL Python.

Для получения полной информации о реализации можно использовать этот пример.

---

## <a name="configuration-files"></a>Файлы конфигурации.

Веб-приложения, которые входят в систему пользователей с помощью платформы Microsoft Identity, настраиваются с помощью файлов конфигурации. Ниже приведены значения, которые необходимо указать в конфигурации.

- Облачный экземпляр ( `Instance` ), если вы хотите, чтобы приложение выполнялось в национальных облаках, например
- Аудитория в ИДЕНТИФИКАТОРе клиента ( `TenantId` )
- Идентификатор клиента ( `ClientId` ) для приложения, скопированный из портал Azure

Также можно увидеть ссылки на `Authority` . `Authority`Значение представляет собой объединение `Instance` `TenantId` значений и.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

В ASP.NET Core эти параметры расположены в файле [appsettings.js](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/bc564d68179c36546770bf4d6264ce72009bc65a/1-WebApp-OIDC/1-1-MyOrg/appsettings.json#L2-L8) в разделе "AzureAd".

```Json
{
  "AzureAd": {
    // Azure cloud instance among:
    // - "https://login.microsoftonline.com/" for Azure public cloud
    // - "https://login.microsoftonline.us/" for Azure US government
    // - "https://login.microsoftonline.de/" for Azure AD Germany
    // - "https://login.partner.microsoftonline.cn/common" for Azure AD China operated by 21Vianet
    "Instance": "https://login.microsoftonline.com/",

    // Azure AD audience among:
    // - "TenantId" as a GUID obtained from the Azure portal to sign in users in your organization
    // - "organizations" to sign in users in any work or school account
    // - "common" to sign in users with any work or school account or Microsoft personal account
    // - "consumers" to sign in users with a Microsoft personal account only
    "TenantId": "[Enter the tenantId here]",

    // Client ID (application ID) obtained from the Azure portal
    "ClientId": "[Enter the Client Id]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath": "/signout-oidc"
  }
}
```

В ASP.NET Core другой файл ([properties\launchSettings.jsв](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/bc564d68179c36546770bf4d6264ce72009bc65a/1-WebApp-OIDC/1-1-MyOrg/Properties/launchSettings.json#L6-L7)) содержит URL-адрес ( `applicationUrl` ) и порт TLS/SSL ( `sslPort` ) для приложения и различных профилей.

```Json
{
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:3110/",
      "sslPort": 44321
    }
  },
  "profiles": {
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "webApp": {
      "commandName": "Project",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:3110/"
    }
  }
}
```

В портал Azure идентификаторы URI перенаправления, регистрируемые на странице **проверки подлинности** для вашего приложения, должны соответствовать этим URL-адресам. Для двух предыдущих файлов конфигурации они бы были `https://localhost:44321/signin-oidc` . Причина в том, что `applicationUrl` это `http://localhost:3110` , но `sslPort` указано (44321). `CallbackPath` имеет `/signin-oidc` , как определено в `appsettings.json` .

Таким же образом URI выхода будет иметь значение `https://localhost:44321/signout-oidc` .

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

В ASP.NET приложение настраивается с помощью файла [Web.config](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Web.config#L12-L15) , строки 12 – 15.

```XML
<?xml version="1.0" encoding="utf-8"?>
<!--
  For more information on how to configure your ASP.NET application, visit
  https://go.microsoft.com/fwlink/?LinkId=301880
  -->
<configuration>
  <appSettings>
    <add key="webpages:Version" value="3.0.0.0" />
    <add key="webpages:Enabled" value="false" />
    <add key="ClientValidationEnabled" value="true" />
    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
    <add key="ida:ClientId" value="[Enter your client ID, as obtained from the app registration portal]" />
    <add key="ida:ClientSecret" value="[Enter your client secret, as obtained from the app registration portal]" />
    <add key="ida:AADInstance" value="https://login.microsoftonline.com/{0}{1}" />
    <add key="ida:RedirectUri" value="https://localhost:44326/" />
    <add key="vs:EnableBrowserLink" value="false" />
  </appSettings>
```

В портал Azure идентификаторы URI ответа, регистрируемые на странице **проверки подлинности** для вашего приложения, должны соответствовать этим URL-адресам. То есть они должны быть `https://localhost:44326/` .

# <a name="java"></a>[Java](#tab/java)

В Java конфигурация находится в файле [приложения. Properties](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/resources/application.properties) , расположенном в папке `src/main/resources` .

```Java
aad.clientId=Enter_the_Application_Id_here
aad.authority=https://login.microsoftonline.com/Enter_the_Tenant_Info_Here/
aad.secretKey=Enter_the_Client_Secret_Here
aad.redirectUriSignin=http://localhost:8080/msal4jsample/secure/aad
aad.redirectUriGraph=http://localhost:8080/msal4jsample/graph/me
```

В портал Azure идентификаторы URI ответа, регистрируемые на странице **проверки подлинности** для приложения, должны совпадать с `redirectUri` экземплярами, которые определяет приложение. То есть они должны быть `http://localhost:8080/msal4jsample/secure/aad` и `http://localhost:8080/msal4jsample/graph/me` .

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Здесь параметры конфигурации находятся в `index.js`

```javascript

const REDIRECT_URI = "http://localhost:3000/redirect";

const config = {
    auth: {
        clientId: "Enter_the_Application_Id_Here",
        authority: "https://login.microsoftonline.com/Enter_the_Tenant_Info_Here/",
        clientSecret: "Enter_the_Client_Secret_Here"
    },
    system: {
        loggerOptions: {
            loggerCallback(loglevel, message, containsPii) {
                console.log(message);
            },
            piiLoggingEnabled: false,
            logLevel: msal.LogLevel.Verbose,
        }
    }
};
```

В портал Azure URI-коды ответов, регистрируемые на странице проверки подлинности для приложения, должны соответствовать экземплярам redirectUri, определяемым приложением ( `http://localhost:3000/redirect` ).

> [!NOTE]
> В этом кратком руководстве предлагается сохранить секрет клиента в файле конфигурации для простоты. В рабочем приложении необходимо использовать другие способы хранения секрета, например хранилище ключей или переменную среды.

# <a name="python"></a>[Python](#tab/python)

Ниже приведен файл конфигурации Python в [app_config. Корректировка](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/0.1.0/app_config.py):

```Python
CLIENT_SECRET = "Enter_the_Client_Secret_Here"
AUTHORITY = "https://login.microsoftonline.com/common""
CLIENT_ID = "Enter_the_Application_Id_here"
ENDPOINT = 'https://graph.microsoft.com/v1.0/users'
SCOPE = ["User.ReadBasic.All"]
SESSION_TYPE = "filesystem"  # So the token cache will be stored in a server-side session
```

> [!NOTE]
> В этом кратком руководстве предлагается сохранить секрет клиента в файле конфигурации для простоты. В рабочем приложении необходимо использовать другие способы хранения секрета, например хранилище ключей или переменную среды, как описано в [документации по Flask](https://flask.palletsprojects.com/en/1.1.x/config/#configuring-from-environment-variables).
>
> ```python
> CLIENT_SECRET = os.getenv("CLIENT_SECRET")
> if not CLIENT_SECRET:
>     raise ValueError("Need to define CLIENT_SECRET environment variable")
> ```

---

## <a name="initialization-code"></a>Код инициализации

Код инициализации различается в зависимости от платформы. Для ASP.NET Core и ASP.NET при входе в систему пользователи делегируются по промежуточного слоя OpenID Connect Connect. Шаблон ASP.NET или ASP.NET Core создает веб-приложения для конечной точки Azure Active Directory (Azure AD) v 1.0. Для адаптации к платформе Microsoft Identity требуется определенная конфигурация. В случае с Java оно обрабатывается пружиной с совместным использованием приложения.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

В ASP.NET Core веб-приложениях (и веб-API) приложение защищено, так как у вас есть `[Authorize]` атрибут на контроллерах или действия контроллера. Этот атрибут проверяет, прошел ли пользователь проверку подлинности. Код, который инициализирует приложение, находится в файле *Startup. CS* .

Чтобы добавить проверку подлинности на платформе Microsoft Identity (прежнее название — Azure AD 2.0), необходимо добавить следующий код. Комментарии в коде должны быть описательными.

> [!NOTE]
> Если вы хотите непосредственно начать с новых шаблонов ASP.NET Core для платформы Microsoft Identity, использующих Microsoft. Identity. Web, можно скачать предварительную версию пакета NuGet, содержащую шаблоны проектов для .NET Core 3,1 и .NET 5,0. После установки можно напрямую создать экземпляр ASP.NET Core веб-приложений (MVC или Блазор). Дополнительные сведения см. в [статье шаблоны проектов веб-приложений Microsoft. Identity. Web App](https://aka.ms/ms-id-web/webapp-project-templates) . Это самый простой подход, так как он выполняет все приведенные ниже действия.
>
> Если вы предпочитаете запускать проект с текущим веб-проектом ASP.NET Core по умолчанию в Visual Studio или с помощью `dotnet new mvc --auth SingleAuth` или `dotnet new webapp --auth SingleAuth` , вы увидите код, подобный следующему:
>
>```c#
>  services.AddAuthentication(AzureADDefaults.AuthenticationScheme)
>          .AddAzureAD(options => Configuration.Bind("AzureAd", options));
> ```
>
> В этом коде используется устаревший пакет NuGet **Microsoft. AspNetCore. Authentication. AzureAD. UI** , который используется для создания приложения Azure AD версии 1.0. В этой статье объясняется, как создать приложение платформы удостоверений Microsoft Identity (Azure AD 2.0), заменяющее этот код.
>

1. Добавьте в проект пакеты NuGet [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web) и [Microsoft. Identity. Web. UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI) . Удалите пакет NuGet Microsoft. AspNetCore. Authentication. AzureAD. UI, если он имеется.

2. Обновите код в `ConfigureServices` , чтобы он использовал `AddMicrosoftIdentityWebAppAuthentication` методы и `AddMicrosoftIdentityUI` .

   ```c#
   public class Startup
   {
    ...
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
     services.AddMicrosoftIdentityWebAppAuthentication(Configuration, "AzureAd");

     services.AddRazorPages().AddMvcOptions(options =>
     {
      var policy = new AuthorizationPolicyBuilder()
                    .RequireAuthenticatedUser()
                    .Build();
      options.Filters.Add(new AuthorizeFilter(policy));
     }).AddMicrosoftIdentityUI();
    ```

3. В `Configure` методе в *Startup. CS* включите проверку подлинности с помощью вызова `app.UseAuthentication();`

   ```c#
   // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
   public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
   {
    // more code here
    app.UseAuthentication();
    app.UseAuthorization();
    // more code here
   }
   ```

В приведенном выше коде:
- `AddMicrosoftIdentityWebAppAuthentication`Метод расширения определен в **Microsoft. Identity. Web**. Им
  - Добавляет службу проверки подлинности.
  - Настраивает параметры для чтения файла конфигурации (здесь из раздела "AzureAD").
  - Настраивает параметры OpenID Connect Connect, чтобы центр сертификации был платформой Microsoft Identity.
  - Проверяет издателя маркера.
  - Гарантирует, что утверждения, соответствующие имени, сопоставляются с `preferred_username` утверждением в маркере идентификации.

- В дополнение к объекту конфигурации можно указать имя раздела конфигурации при вызове `AddMicrosoftIdentityWebAppAuthentication` . По умолчанию это `AzureAd`.

- `AddMicrosoftIdentityWebAppAuthentication` имеет другие параметры для расширенных сценариев. Например, трассировка событий по промежуточного слоя OpenID Connect позволяет устранить неполадки в веб-приложении, если проверка подлинности не работает. При установке необязательного параметра `subscribeToOpenIdConnectMiddlewareDiagnosticsEvents` `true` будет показано, как обрабатывается информация по по промежуточного слоя ASP.NET Core по мере продвижения от HTTP-ответа к удостоверению пользователя в `HttpContext.User` .

- `AddMicrosoftIdentityUI`Метод расширения определен в **Microsoft. Identity. Web. UI**. Он предоставляет контроллер по умолчанию для управления входом и выходом.

Дополнительные сведения о том, как Microsoft. Identity. Web позволяет создавать веб-приложения, можно найти в <https://aka.ms/ms-id-web/webapp>

> [!WARNING]
> В настоящее время Microsoft. Identity. Web не поддерживает сценарий использования **отдельных учетных записей пользователей** (хранение учетных записей пользователей в приложении) при использовании Azure AD как и внешнего поставщика входа. Дополнительные сведения см. в [статье AzureAD/Microsoft-Identity-Web # 133](https://github.com/AzureAD/microsoft-identity-web/issues/133)

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Код, связанный с проверкой подлинности в веб-приложении ASP.NET и веб-API, находится в файле [App_Start/стартуп.АУС.КС](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/App_Start/Startup.Auth.cs#L17-L61) .

```csharp
 public void ConfigureAuth(IAppBuilder app)
 {
  app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

  app.UseCookieAuthentication(new CookieAuthenticationOptions());

  app.UseOpenIdConnectAuthentication(
    new OpenIdConnectAuthenticationOptions
    {
     // Authority` represents the identity platform endpoint - https://login.microsoftonline.com/common/v2.0.
     // `Scope` describes the initial permissions that your app will need.
     //  See https://azure.microsoft.com/documentation/articles/active-directory-v2-scopes/.
     ClientId = clientId,
     Authority = String.Format(CultureInfo.InvariantCulture, aadInstance, "common", "/v2.0"),
     RedirectUri = redirectUri,
     Scope = "openid profile",
     PostLogoutRedirectUri = redirectUri,
    });
 }
```

# <a name="java"></a>[Java](#tab/java)

В примере Java используется пружинная платформа. Приложение защищено, так как вы реализуете фильтр, который перехватывает каждый HTTP-ответ. В кратком руководстве по веб-приложениям Java этот фильтр находится `AuthFilter` в `src/main/java/com/microsoft/azure/msalwebsample/AuthFilter.java` .

Фильтр обрабатывает поток кода авторизации OAuth 2,0 и проверяет, прошел ли пользователь проверку подлинности ( `isAuthenticated()` метод). Если пользователь не прошел проверку подлинности, он рассчитывает URL-адрес конечных точек авторизации Azure AD и перенаправляет браузер на этот универсальный код ресурса (URI).

Когда получен ответ, содержащий код авторизации, он получает маркер с помощью MSAL Java. Когда он наконец получает маркер из конечной точки маркера (в URI перенаправления), пользователь вошел в систему.

Дополнительные сведения см. в описании `doFilter()` метода в [аусфилтер. Java](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/master/msal-java-webapp-sample/src/main/java/com/microsoft/azure/msalwebsample/AuthFilter.java).

> [!NOTE]
> Код `doFilter()` написан немного иначе, но последовательность описана в этом порядке.

Дополнительные сведения о потоке кода авторизации, который запускается этим методом, см. в [статье поток кода авторизации для платформы Microsoft Identity и OAuth 2,0](v2-oauth2-auth-code-flow.md).

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

```javascript
const msal = require('@azure/msal-node');

// Create msal application object
const cca = new msal.ConfidentialClientApplication(config);
```

# <a name="python"></a>[Python](#tab/python)

В образце Python используется Flask. Инициализация Flask и MSAL Python выполняется в [app. копировать # L1-L28](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L1-L28).

```Python
import uuid
import requests
from flask import Flask, render_template, session, request, redirect, url_for
from flask_session import Session  # https://pythonhosted.org/Flask-Session
import msal
import app_config


app = Flask(__name__)
app.config.from_object(app_config)
Session(app)
```

---

## <a name="next-steps"></a>Следующие шаги

В следующей статье вы узнаете, как активировать вход и выход.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Перейдите к следующей статье в этом сценарии, [Войдите и выйдите](./scenario-web-app-sign-user-sign-in.md?tabs=aspnetcore)из нее.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Перейдите к следующей статье в этом сценарии, [Войдите и выйдите](./scenario-web-app-sign-user-sign-in.md?tabs=aspnet)из нее.

# <a name="java"></a>[Java](#tab/java)

Перейдите к следующей статье в этом сценарии, [Войдите и выйдите](./scenario-web-app-sign-user-sign-in.md?tabs=java)из нее.

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Перейдите к следующей статье в этом сценарии, [Войдите в](./scenario-web-app-sign-user-sign-in.md?tabs=nodejs)систему.

# <a name="python"></a>[Python](#tab/python)

Перейдите к следующей статье в этом сценарии, [Войдите и выйдите](./scenario-web-app-sign-user-sign-in.md?tabs=python)из нее.

---
