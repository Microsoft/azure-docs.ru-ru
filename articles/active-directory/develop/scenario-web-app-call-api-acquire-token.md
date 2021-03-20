---
title: Получение маркера в веб-приложении, вызывающем веб-API | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как получить маркер для веб-приложения, которое вызывает веб-API.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/25/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: cfd479382cb69e7355b033312e165699223fdbf0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98756299"
---
# <a name="a-web-app-that-calls-web-apis-acquire-a-token-for-the-app"></a>Веб-приложение, вызывающее веб-API: получение маркера для приложения

Вы создали объект клиентского приложения. Теперь вы будете использовать его для получения маркера для вызова веб-API. В ASP.NET или ASP.NET Core вызов веб-API выполняется в контроллере:

- Получите маркер для веб-API с помощью кэша маркеров. Чтобы получить этот маркер, вызовите метод MSAL `AcquireTokenSilent` (или эквивалент в Microsoft. Identity. Web).
- Вызовите защищенный API, передав ему маркер доступа в качестве параметра.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

*Microsoft. Identity. Web* добавляет методы расширения, которые предоставляют удобные службы для вызова Microsoft Graph или нисходящего веб-API. Эти методы подробно описаны в [веб-приложении, которое вызывает веб-API: вызов API](scenario-web-app-call-api-call-api.md). При использовании этих вспомогательных методов вам не нужно вручную получать маркер.

Однако если вы хотите вручную получить маркер, в следующем коде показан пример использования *Microsoft. Identity. Web* для этого в контроллере home. Он вызывает Microsoft Graph с помощью REST API (вместо пакета SDK для Microsoft Graph). Чтобы получить маркер для вызова подчиненного API, необходимо внедрить `ITokenAcquisition` службу путем внедрения зависимостей в конструктор контроллера (или конструктор страницы при использовании блазор) и использовать его в действиях контроллера, получая маркер для пользователя ( `GetAccessTokenForUserAsync` ) или самого приложения ( `GetAccessTokenForAppAsync` ) в сценарии управляющей программы.

Методы контроллера защищены `[Authorize]` атрибутом, который позволяет использовать веб-приложение только прошедшим проверку подлинности пользователям.

```csharp
[Authorize]
public class HomeController : Controller
{
 readonly ITokenAcquisition tokenAcquisition;

 public HomeController(ITokenAcquisition tokenAcquisition)
 {
  this.tokenAcquisition = tokenAcquisition;
 }

 // Code for the controller actions (see code below)

}
```

Эта `ITokenAcquisition` Служба внедряется функцией ASP.NET с помощью внедрения зависимостей.

Ниже приведен упрощенный код для действия `HomeController` , которое получает маркер для вызова Microsoft Graph:

```csharp
[AuthorizeForScopes(Scopes = new[] { "user.read" })]
public async Task<IActionResult> Profile()
{
 // Acquire the access token.
 string[] scopes = new string[]{"user.read"};
 string accessToken = await tokenAcquisition.GetAccessTokenForUserAsync(scopes);

 // Use the access token to call a protected web API.
 HttpClient client = new HttpClient();
 client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
 string json = await client.GetStringAsync(url);
}
```

Чтобы лучше понять код, необходимый для этого сценария, см. шаг 2[2-1 (вызовы веб-приложения Microsoft Graph](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-1-Call-MSGraph)) в учебнике [MS-Identity-aspnetcore-webapp-Tutorial](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial) .

`AuthorizeForScopes`Атрибут в верхней части действия контроллера (или страницы Razor, если используется шаблон Razor) предоставляется Microsoft. Identity. Web. Это гарантирует, что при необходимости пользователю будет предложено предоставить согласие.

Существуют и другие сложные вариации, например:

- Вызов нескольких интерфейсов API.
- Обработка добавочного согласия и условного доступа.

Эти дополнительные действия описаны в главе 3 учебника по [3-webapp-Multi-API](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/3-WebApp-multi-APIs) .

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Код для ASP.NET похож на код, показанный для ASP.NET Core:

- Действие контроллера, защищенное атрибутом [авторизовать], извлекает идентификатор клиента и идентификатор пользователя для `ClaimsPrincipal` члена контроллера. (ASP.NET использует `HttpContext.User` .)
- После этого он создает `IConfidentialClientApplication` объект MSAL.NET.
- Наконец, он вызывает `AcquireTokenSilent` метод конфиденциального клиентского приложения.
- Если требуется взаимодействие, веб-приложению необходимо выполнить запрос пользователя (повторно войти в систему) и запросить дополнительные утверждения.

Следующий фрагмент кода извлекается из [HomeController. CS # L157-L192](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/257c8f96ec3ff875c351d1377b36403eed942a18/WebApp/Controllers/HomeController.cs#L157-L192) в образце кода [MS-Identity-ASPNET-webapp-openidconnect](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect) ASP.NET MVC:

```C#
public async Task<ActionResult> ReadMail()
{
    IConfidentialClientApplication app = MsalAppBuilder.BuildConfidentialClientApplication();
    AuthenticationResult result = null;
    var account = await app.GetAccountAsync(ClaimsPrincipal.Current.GetMsalAccountId());
    string[] scopes = { "Mail.Read" };

    try
    {
        // try to get token silently
        result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync().ConfigureAwait(false);
    }
    catch (MsalUiRequiredException)
    {
        ViewBag.Relogin = "true";
        return View();
    }

    // More code here
    return View();
}
```

Дополнительные сведения см. в коде для [буилдконфидентиалклиентаппликатион ()](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/master/WebApp/Utils/MsalAppBuilder.cs) и [жетмсалаккаунтид](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/257c8f96ec3ff875c351d1377b36403eed942a18/WebApp/Utils/ClaimPrincipalExtension.cs#L38) в примере кода.


# <a name="java"></a>[Java](#tab/java)

В примере Java код, вызывающий API, находится в методе Жетусерсфромграф в [ауспажеконтроллер. Java # L62](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthPageController.java#L62).

Метод пытается вызвать `getAuthResultBySilentFlow` . Если пользователь должен согласиться с дополнительными областями, код обрабатывает `MsalInteractionRequiredException` объект для вызова пользователю.

```java
@RequestMapping("/msal4jsample/graph/me")
public ModelAndView getUserFromGraph(HttpServletRequest httpRequest, HttpServletResponse response)
        throws Throwable {

    IAuthenticationResult result;
    ModelAndView mav;
    try {
        result = authHelper.getAuthResultBySilentFlow(httpRequest, response);
    } catch (ExecutionException e) {
        if (e.getCause() instanceof MsalInteractionRequiredException) {

            // If the silent call returns MsalInteractionRequired, redirect to authorization endpoint
            // so user can consent to new scopes.
            String state = UUID.randomUUID().toString();
            String nonce = UUID.randomUUID().toString();

            SessionManagementHelper.storeStateAndNonceInSession(httpRequest.getSession(), state, nonce);

            String authorizationCodeUrl = authHelper.getAuthorizationCodeUrl(
                    httpRequest.getParameter("claims"),
                    "User.Read",
                    authHelper.getRedirectUriGraph(),
                    state,
                    nonce);

            return new ModelAndView("redirect:" + authorizationCodeUrl);
        } else {

            mav = new ModelAndView("error");
            mav.addObject("error", e);
            return mav;
        }
    }

    if (result == null) {
        mav = new ModelAndView("error");
        mav.addObject("error", new Exception("AuthenticationResult not found in session."));
    } else {
        mav = new ModelAndView("auth_page");
        setAccountInfo(mav, httpRequest);

        try {
            mav.addObject("userInfo", getUserInfoFromGraph(result.accessToken()));

            return mav;
        } catch (Exception e) {
            mav = new ModelAndView("error");
            mav.addObject("error", e);
        }
    }
    return mav;
}
// Code omitted here
```

# <a name="python"></a>[Python](#tab/python)

В образце Python код, вызывающий Microsoft Graph, находится в [app. Корректировка l53-L62](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/48637475ed7d7733795ebeac55c5d58663714c60/app.py#L53-L62).

Код пытается получить маркер из кэша маркеров. После установки заголовка авторизации он вызывает веб-API. Если не удается получить маркер, пользователь снова подписывается.

```python
@app.route("/graphcall")
def graphcall():
    token = _get_token_from_cache(app_config.SCOPE)
    if not token:
        return redirect(url_for("login"))
    graph_data = requests.get(  # Use token to call downstream service.
        app_config.ENDPOINT,
        headers={'Authorization': 'Bearer ' + token['access_token']},
        ).json()
    return render_template('display.html', result=graph_data)
```

---

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [вызовите веб-API](scenario-web-app-call-api-call-api.md).
