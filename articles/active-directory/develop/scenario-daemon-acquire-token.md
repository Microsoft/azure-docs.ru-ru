---
title: Получение маркеров для вызова веб-API (управляющее приложение) — платформа Microsoft Identity | Службы
description: Узнайте, как создать управляющее приложение, вызывающее веб-API (получение маркеров).
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
ms.openlocfilehash: 19ead7fe063992e95588641f7fd739081cf54a2f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578419"
---
# <a name="daemon-app-that-calls-web-apis---acquire-a-token"></a>Управляющее приложение, вызывающее веб-API — получение маркера

После создания конфиденциального клиентского приложения можно получить маркер для приложения путем вызова `AcquireTokenForClient` , передачи области и, при необходимости, принудительного обновления маркера.

## <a name="scopes-to-request"></a>Области для запроса

Область запросов для потока учетных данных клиента — это имя ресурса с добавлением `/.default`. Эта нотация указывает Azure Active Directory (Azure AD) использовать *разрешения уровня приложения* , объявленные статически во время регистрации приложения. Кроме того, эти разрешения API должны быть предоставлены администратором арендатора.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
ResourceId = "someAppIDURI";
var scopes = new [] {  ResourceId+"/.default"};
```

# <a name="java"></a>[Java](#tab/java)

```Java
final static String GRAPH_DEFAULT_SCOPE = "https://graph.microsoft.com/.default";
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

```JavaScript
const tokenRequest = {
    scopes: [process.env.GRAPH_ENDPOINT + '.default'], // e.g. 'https://graph.microsoft.com/.default'
};
```

# <a name="python"></a>[Python](#tab/python)

В MSAL Python файл конфигурации выглядит как следующий фрагмент кода:

```Json
{
    "scope": ["https://graph.microsoft.com/.default"],
}
```

---

### <a name="azure-ad-v10-resources"></a>Ресурсы по Azure AD (v 1.0)

Областью, используемой для учетных данных клиента, всегда должен быть идентификатор ресурса, за которым следует `/.default` .

> [!IMPORTANT]
> Когда MSAL запрашивает маркер доступа для ресурса, который принимает маркер доступа версии 1,0, Azure AD анализирует нужную аудиторию из запрошенной области, принимая все до последней косой черты и используя ее в качестве идентификатора ресурса.
> Таким образом, если, как и база данных SQL Azure (**https: \/ /Database.Windows.NET**), ресурс ожидает аудиторию, которая заканчивается косой чертой (для базы данных SQL Azure `https://database.windows.net/` ), необходимо запросить область действия `https://database.windows.net//.default` . (Обратите внимание на двойную косую черту.) См. также MSAL.NET проблема [#747: пропущена Конечная косая черта URL-адреса ресурса, что привело к сбою проверки подлинности SQL](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/747).

## <a name="acquiretokenforclient-api"></a>API Аккуиретокенфорклиент

Чтобы получить маркер для приложения, вы будете использовать `AcquireTokenForClient` или его эквивалент в зависимости от платформы.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
using Microsoft.Identity.Client;

// With client credentials flows, the scope is always of the shape "resource/.default" because the
// application permissions need to be set statically (in the portal or by PowerShell), and then granted by
// a tenant administrator.
string[] scopes = new string[] { "https://graph.microsoft.com/.default" };

AuthenticationResult result = null;
try
{
 result = await app.AcquireTokenForClient(scopes)
                  .ExecuteAsync();
}
catch (MsalUiRequiredException ex)
{
    // The application doesn't have sufficient permissions.
    // - Did you declare enough app permissions during app creation?
    // - Did the tenant admin grant permissions to the application?
}
catch (MsalServiceException ex) when (ex.Message.Contains("AADSTS70011"))
{
    // Invalid scope. The scope has to be in the form "https://resourceurl/.default"
    // Mitigation: Change the scope to be as expected.
}
```

### <a name="acquiretokenforclient-uses-the-application-token-cache"></a>Аккуиретокенфорклиент использует кэш маркеров приложений

В MSAL.NET `AcquireTokenForClient` использует кэш маркеров приложений. (Все остальные методы AcquireToken *XX* используют кэш пользовательских маркеров.) Не вызывайте метод `AcquireTokenSilent` перед вызовом `AcquireTokenForClient` , так как `AcquireTokenSilent` использует кэш *пользовательских* маркеров. `AcquireTokenForClient` проверяет сам кэш маркера *приложения* и обновляет его.

# <a name="java"></a>[Java](#tab/java)

Этот код извлекается из [примеров разработки Java MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/confidential-client/).

```Java
private static IAuthenticationResult acquireToken() throws Exception {

     // Load token cache from file and initialize token cache aspect. The token cache will have
     // dummy data, so the acquireTokenSilently call will fail.
     TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

     IClientCredential credential = ClientCredentialFactory.createFromSecret(CLIENT_SECRET);
     ConfidentialClientApplication cca =
             ConfidentialClientApplication
                     .builder(CLIENT_ID, credential)
                     .authority(AUTHORITY)
                     .setTokenCacheAccessAspect(tokenCacheAspect)
                     .build();

     IAuthenticationResult result;
     try {
         SilentParameters silentParameters =
                 SilentParameters
                         .builder(SCOPE)
                         .build();

         // try to acquire token silently. This call will fail since the token cache does not
         // have a token for the application you are requesting an access token for
         result = cca.acquireTokenSilently(silentParameters).join();
     } catch (Exception ex) {
         if (ex.getCause() instanceof MsalException) {

             ClientCredentialParameters parameters =
                     ClientCredentialParameters
                             .builder(SCOPE)
                             .build();

             // Try to acquire a token. If successful, you should see
             // the token information printed out to console
             result = cca.acquireToken(parameters).join();
         } else {
             // Handle other exceptions accordingly
             throw ex;
         }
     }
     return result;
 }
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Приведенный ниже фрагмент кода иллюстрирует получение токена в конфиденциальном клиентском приложении MSAL Node:

```JavaScript
try {
    const authResponse = await cca.acquireTokenByClientCredential(tokenRequest);
    console.log(authResponse.accessToken) // display access token
} catch (error) {
    console.log(error);
}
```

# <a name="python"></a>[Python](#tab/python)

```Python
# The pattern to acquire a token looks like this.
result = None

# First, the code looks up a token from the cache.
# Because we're looking for a token for the current app, not for a user,
# use None for the account parameter.
result = app.acquire_token_silent(config["scope"], account=None)

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    result = app.acquire_token_for_client(scopes=config["scope"])

if "access_token" in result:
    # Call a protected API with the access token.
    print(result["token_type"])
else:
    print(result.get("error"))
    print(result.get("error_description"))
    print(result.get("correlation_id"))  # You might need this when reporting a bug.
```

---

### <a name="protocol"></a>Протокол

Если у вас еще нет библиотеки для выбранного языка, вы можете использовать протокол напрямую:

#### <a name="first-case-access-the-token-request-by-using-a-shared-secret"></a>Первый случай: доступ к запросу маркера с помощью общего секрета

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1           //Line breaks for clarity.
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_secret=qWgdYAmab0YSkuL1qKv5bPX
&grant_type=client_credentials
```

#### <a name="second-case-access-the-token-request-by-using-a-certificate"></a>Второй случай: доступ к запросу маркера с помощью сертификата.

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity.
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&grant_type=client_credentials
```

Дополнительные сведения см. в документации по протоколу: [платформа удостоверений Майкрософт и поток учетных данных клиента OAuth 2,0](v2-oauth2-client-creds-grant-flow.md).

## <a name="troubleshooting"></a>Диагностика

### <a name="did-you-use-the-resourcedefault-scope"></a>Вы использовали область ресурсов/. по умолчанию?

Если появляется сообщение об ошибке, сообщающее, что вы использовали недопустимую область, то, вероятно, вы не использовали эту `resource/.default` область.

### <a name="did-you-forget-to-provide-admin-consent-daemon-apps-need-it"></a>Вы забыли предоставить согласие администратора? Требуются приложения управляющей программы!

Если у вас возникли **недостаточные привилегии для выполнения операции** при вызове API, администратор клиента должен предоставить приложению разрешения. См. шаг 6 в разделе Регистрация клиентского приложения выше.
Как правило, отображается сообщение об ошибке, похожее на следующее:

```json
Failed to call the web API: Forbidden
Content: {
  "error": {
    "code": "Authorization_RequestDenied",
    "message": "Insufficient privileges to complete the operation.",
    "innerError": {
      "request-id": "<guid>",
      "date": "<date>"
    }
  }
}
```

### <a name="are-you-calling-your-own-api"></a>Вы вызываете собственный API?

Если вы вызываете собственный веб-API и не смогли добавить разрешение приложения в регистрацию приложения для приложения управляющей программы, вы предоставили роль приложения в веб-API?

Дополнительные сведения см. в статьях [предоставление разрешений приложению (роли приложений)](scenario-protected-web-api-app-registration.md#exposing-application-permissions-app-roles) и, в частности, [обеспечение того, что Azure AD выдает маркеры для веб-API только для клиентов](scenario-protected-web-api-app-registration.md#ensuring-that-azure-ad-issues-tokens-for-your-web-api-to-only-allowed-clients).

## <a name="next-steps"></a>Дальнейшие действия

# <a name="net"></a>[.NET](#tab/dotnet)

Перейдите к следующей статье в этом сценарии, [вызвав веб-API](./scenario-daemon-call-api.md?tabs=dotnet).

# <a name="java"></a>[Java](#tab/java)

Перейдите к следующей статье в этом сценарии, [вызвав веб-API](./scenario-daemon-call-api.md?tabs=java).

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Перейдите к следующей статье в этом сценарии, [вызвав веб-API](./scenario-daemon-call-api.md?tabs=nodejs).

# <a name="python"></a>[Python](#tab/python)

Перейдите к следующей статье в этом сценарии, [вызвав веб-API](./scenario-daemon-call-api.md?tabs=python).

---
