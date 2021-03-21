---
title: Получение маркера для веб-API, который вызывает веб-API | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать веб-API, который вызывает веб-API, требующие получения маркера для приложения.
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
ms.openlocfilehash: 9f9758ec765ad34e5ef5d8b4d4e0a420a6701b6e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98756397"
---
# <a name="a-web-api-that-calls-web-apis-acquire-a-token-for-the-app"></a>Веб-API, вызывающий веб-API: получение маркера для приложения

После построения объекта клиентского приложения используйте его для получения маркера, который можно использовать для вызова веб-API.

## <a name="code-in-the-controller"></a>Код в контроллере

### <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

*Microsoft. Identity. Web* добавляет методы расширения, которые предоставляют удобные службы для вызова Microsoft Graph или нисходящего веб-API. Эти методы подробно описаны в [веб-API, который вызывает веб-API: вызов API](scenario-web-api-call-api-call-api.md). При использовании этих вспомогательных методов вам не нужно вручную получать маркер.

Однако если вы хотите вручную получить маркер, в следующем коде показан пример использования *Microsoft. Identity. Web* для этого в контроллере API. Он вызывает нисходящий API с именем *ToDoList*.
Чтобы получить маркер для вызова подчиненного API, необходимо внедрить `ITokenAcquisition` службу путем внедрения зависимостей в конструктор контроллера (или конструктор страницы при использовании блазор) и использовать его в действиях контроллера, получая маркер для пользователя ( `GetAccessTokenForUserAsync` ) или самого приложения ( `GetAccessTokenForAppAsync` ) в случае сценария управляющей программы.

```csharp
[Authorize]
public class MyApiController : Controller
{
    /// <summary>
    /// The web API will accept only tokens 1) for users, 2) that have the `access_as_user` scope for
    /// this API.
    /// </summary>
    static readonly string[] scopeRequiredByApi = new string[] { "access_as_user" };

     static readonly string[] scopesToAccessDownstreamApi = new string[] { "api://MyTodolistService/access_as_user" };

    private readonly ITokenAcquisition _tokenAcquisition;

    public MyApiController(ITokenAcquisition tokenAcquisition)
    {
        _tokenAcquisition = tokenAcquisition;
    }

    public IActionResult Index()
    {
        HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);

        string accessToken = _tokenAcquisition.GetAccessTokenForUserAsync(scopesToAccessDownstreamApi);
        return await callTodoListService(accessToken);
    }
}
```

Дополнительные сведения о `callTodoListService` методе см.  [в веб-API, который вызывает веб-API: вызов API](scenario-web-api-call-api-call-api.md).

### <a name="java"></a>[Java](#tab/java)

Ниже приведен пример кода, который вызывается в действиях контроллеров API. Он вызывает нисходящий Microsoft Graph API.

```java
@RestController
public class ApiController {

    @Autowired
    MsalAuthHelper msalAuthHelper;

    @RequestMapping("/graphMeApi")
    public String graphMeApi() throws MalformedURLException {

        String oboAccessToken = msalAuthHelper.getOboToken("https://graph.microsoft.com/.default");

        return callMicrosoftGraphMeEndpoint(oboAccessToken);
    }

}
```

### <a name="python"></a>[Python](#tab/python)
 
Веб-API Python требует использования по промежуточного слоя для проверки токена носителя, полученного от клиента. Затем веб-API может получить маркер доступа для нисходящих API с помощью библиотеки MSAL Python, вызвав [`acquire_token_on_behalf_of`](https://msal-python.readthedocs.io/en/latest/?badge=latest#msal.ConfidentialClientApplication.acquire_token_on_behalf_of) метод.
 
Ниже приведен пример кода, который получает маркер доступа с помощью `acquire_token_on_behalf_of` метода и платформы Flask. Он вызывает нисходящий API — конечную точку подписок управления Azure.
 
```python
def get(self):
 
        _scopes = ["https://management.azure.com/user_impersonation"]
        _azure_management_subscriptions_uri = "https://management.azure.com/subscriptions?api-version=2020-01-01"
 
        current_access_token = request.headers.get("Authorization", None)
        
        #This example only uses the default memory token cache and should not be used for production
        msal_client = msal.ConfidentialClientApplication(
                client_id=os.environ.get("CLIENT_ID"),
                authority=os.environ.get("AUTHORITY"),
                client_credential=os.environ.get("CLIENT_SECRET"))
 
        #acquire token on behalf of the user that called this API
        arm_resource_access_token = msal_client.acquire_token_on_behalf_of(
            user_assertion=current_access_token.split(' ')[1],
            scopes=_scopes
        )
 
        headers = {'Authorization': arm_resource_access_token['token_type'] + ' ' + arm_resource_access_token['access_token']}
 
        subscriptions_list = req.get(_azure_management_subscriptions_uri), headers=headers).json()
 
        return jsonify(subscriptions_list)
```

## <a name="advanced-accessing-the-signed-in-users-token-cache-from-background-apps-apis-and-services"></a>Продвинут Доступ к кэшу маркера пользователя, выполнившего вход, из фоновых приложений, интерфейсов API и служб

[!INCLUDE [advanced-token-caching](../../../includes/advanced-token-cache.md)]

---

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [вызовите API](scenario-web-api-call-api-call-api.md).
