---
title: Создание веб-приложения, которое входит в систему и выполняет вход пользователей | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать веб-приложение, которое входит в систему и выполнит вход пользователей.
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
ms.openlocfilehash: 10ddee404de21c5bc04672fdb6dd32c30f481ba3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104578249"
---
# <a name="web-app-that-signs-in-users-sign-in-and-sign-out"></a>Веб-приложение, которое входит в систему пользователей: вход и выход

Узнайте, как добавить вход в код для веб-приложения, который входит в систему пользователей. Затем Узнайте, как разрешить выход.

## <a name="sign-in"></a>Вход

Вход состоит из двух частей:

- Кнопка входа на HTML-странице
- Действие входа в коде программной части контроллера

### <a name="sign-in-button"></a>Кнопка входа

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

В ASP.NET Core для приложений платформы Microsoft Identity кнопка **входа** предоставляется в `Views\Shared\_LoginPartial.cshtml` (для приложения MVC) или `Pages\Shared\_LoginPartial.cshtm` (для приложения Razor). Он отображается только в том случае, если пользователь не прошел проверку подлинности. То есть он отображается, когда пользователь еще не вошел в приложение или не выполнил выход из него. Напротив, кнопка **выйти** отображается, когда пользователь уже вошел в систему. Обратите внимание, что контроллер учетной записи определен в пакете NuGet **Microsoft. Identity. Web. UI** в области с именем **микрософтидентити**

```html
<ul class="navbar-nav">
  @if (User.Identity.IsAuthenticated)
  {
    <li class="nav-item">
        <span class="navbar-text text-dark">Hello @User.Identity.Name!</span>
    </li>
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignOut">Sign out</a>
    </li>
  }
  else
  {
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignIn">Sign in</a>
    </li>
  }
</ul>
```

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

В ASP.NET MVC кнопка выхода предоставляется в `Views\Shared\_LoginPartial.cshtml` . Он отображается только при наличии учетной записи с проверкой подлинности. То есть он отображается, когда пользователь ранее выполнил вход.

```html
@if (Request.IsAuthenticated)
{
 // Code omitted code for clarity
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li>@Html.ActionLink("Sign in", "SignIn", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
    </ul>
}
```

# <a name="java"></a>[Java](#tab/java)

В нашем кратком руководстве по Java кнопка входа находится в файле [Main/Resources/Templates/index.html](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/master/msal-java-webapp-sample/src/main/resources/templates/index.html) .

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>HomePage</title>
</head>
<body>
<h3>Home Page</h3>

<form action="/msal4jsample/secure/aad">
    <input type="submit" value="Login">
</form>

</body>
</html>
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

В кратком руководстве по Node.js нет кнопки входа. При достижении корня веб-приложения код программной части автоматически предлагает пользователю выполнить вход.

```javascript
app.get('/', (req, res) => {
    // authentication logic
});
```

# <a name="python"></a>[Python](#tab/python)

В кратком руководстве по Python отсутствует кнопка входа. При достижении корня веб-приложения код программной части автоматически предлагает пользователю выполнить вход. См. раздел [app. Корректировка # L14-L18](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/0.1.0/app.py#L14-L18).

```Python
@app.route("/")
def index():
    if not session.get("user"):
        return redirect(url_for("login"))
    return render_template('index.html', user=session["user"])
```

---

### <a name="signin-action-of-the-controller"></a>`SignIn` действие контроллера

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

В ASP.NET при нажатии кнопки **входа** в веб-приложении действие будет инициировано `SignIn` на `AccountController` контроллере. В предыдущих версиях шаблонов ASP.NET Core `Account` контроллер был внедрен в веб-приложение. Это не так, потому что контроллер теперь входит в пакет NuGet **Microsoft. Identity. Web. UI** . Дополнительные сведения см. в разделе [AccountController. CS](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web.UI/Areas/MicrosoftIdentity/Controllers/AccountController.cs) .

Этот контроллер также обрабатывает Azure AD B2C приложения.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

В ASP.NET выход запускается из `SignOut()` метода на контроллере (например, [AccountController. CS # L16-L23](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Controllers/AccountController.cs#L16-L23)). Этот метод не является частью ASP.NET Framework (в отличие от того, что происходит в ASP.NET Core). При предложении URI перенаправления он отправляет запрос на вход OpenID Connect.

```csharp
public void SignIn()
{
    // Send an OpenID Connect sign-in request.
    if (!Request.IsAuthenticated)
    {
        HttpContext.GetOwinContext().Authentication.Challenge(new AuthenticationProperties { RedirectUri = "/" }, OpenIdConnectAuthenticationDefaults.AuthenticationType);
    }
}
```

# <a name="java"></a>[Java](#tab/java)

В Java выход осуществляется путем прямого вызова конечной точки платформы Microsoft Identity `logout` и предоставления `post_logout_redirect_uri` значения. Дополнительные сведения см. в разделе [ауспажеконтроллер. Java # L30-L48](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthPageController.java#L30-L48).

```Java
@Controller
public class AuthPageController {

    @Autowired
    AuthHelper authHelper;

    @RequestMapping("/msal4jsample")
    public String homepage(){
        return "index";
    }

    @RequestMapping("/msal4jsample/secure/aad")
    public ModelAndView securePage(HttpServletRequest httpRequest) throws ParseException {
        ModelAndView mav = new ModelAndView("auth_page");

        setAccountInfo(mav, httpRequest);

        return mav;
    }

    // More code omitted for simplicity
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

В отличие от других платформ, в этом случае узел MSAL берет на себя разрешение на вход пользователя со страницы входа.

```javascript

// 1st leg of auth code flow: acquire a code
app.get('/', (req, res) => {
    const authCodeUrlParameters = {
        scopes: ["user.read"],
        redirectUri: REDIRECT_URI,
    };

    // get url to sign user in and consent to scopes needed for application
    pca.getAuthCodeUrl(authCodeUrlParameters).then((response) => {
        res.redirect(response);
    }).catch((error) => console.log(JSON.stringify(error)));
});

// 2nd leg of auth code flow: exchange code for token
app.get('/redirect', (req, res) => {
    const tokenRequest = {
        code: req.query.code,
        scopes: ["user.read"],
        redirectUri: REDIRECT_URI,
    };

    pca.acquireTokenByCode(tokenRequest).then((response) => {
        console.log("\nResponse: \n:", response);
        res.sendStatus(200);
    }).catch((error) => {
        console.log(error);
        res.status(500).send(error);
    });
});
```

# <a name="python"></a>[Python](#tab/python)

В отличие от других платформ, MSAL Python берет на себя предоставление пользователю возможности входа со страницы входа. См. раздел [app. Корректировка # L20-L28](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L20-L28).

```Python
@app.route("/login")
def login():
    session["state"] = str(uuid.uuid4())
    auth_url = _build_msal_app().get_authorization_request_url(
        app_config.SCOPE,  # Technically we can use an empty list [] to just sign in
                           # Here we choose to also collect user consent up front
        state=session["state"],
        redirect_uri=url_for("authorized", _external=True))
    return "<a href='%s'>Login with Microsoft Identity</a>" % auth_url
```

`_build_msal_app()`Метод определяется в [app. Корректировка # L81-L88](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/app.py#L81-L88) следующим образом:

```Python
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

def _get_token_from_cache(scope=None):
    cache = _load_cache()  # This web app maintains one cache per session
    cca = _build_msal_app(cache)
    accounts = cca.get_accounts()
    if accounts:  # So all accounts belong to the current signed-in user
        result = cca.acquire_token_silent(scope, account=accounts[0])
        _save_cache(cache)
        return result

```

---

После входа пользователя в приложение необходимо включить его для выхода из него.

## <a name="sign-out"></a>Функция выхода

Выход из веб-приложения требует больше, чем удаление сведений о учетной записи, выполнившего вход, из состояния веб-приложения.
Веб-приложение также должно перенаправить пользователя на `logout` конечную точку платформы идентификации Майкрософт для выхода.

Когда веб-приложение перенаправляет пользователя на `logout` конечную точку, эта конечная точка удаляет сеанс пользователя из браузера. Если приложение не пойдет в `logout` конечную точку, пользователь повторно получит повторную проверку подлинности, не вводя учетные данные снова. Причина заключается в том, что у них будет действительный сеанс единого входа с платформой Microsoft Identity.

Дополнительные сведения см. в разделе [Отправка запроса на](v2-protocols-oidc.md#send-a-sign-out-request) выход на [платформе Microsoft Identity и в документации по протоколу OpenID Connect Connect](v2-protocols-oidc.md) .

### <a name="application-registration"></a>Регистрация приложения

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Во время регистрации приложения вы регистрируете URL-адрес выхода для внешнего канала. В нашем руководстве вы зарегистрировались `https://localhost:44321/signout-oidc` в поле **URL-адрес внешнего канала для выхода** на странице **Проверка подлинности** . Дополнительные сведения см. [в разделе Регистрация приложения webApp](scenario-web-app-sign-user-app-registration.md#register-an-app-by-using-the-azure-portal).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

Во время регистрации приложения не нужно регистрировать дополнительный URL-адрес для выхода на переднюю канал. Приложение будет вызываться обратно по его основному URL-адресу. 

# <a name="java"></a>[Java](#tab/java)

В регистрации приложения не требуется URL-адрес для выхода из внешнего канала.

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

В регистрации приложения не требуется URL-адрес для выхода из внешнего канала.

# <a name="python"></a>[Python](#tab/python)

Во время регистрации приложения не нужно регистрировать дополнительный URL-адрес для выхода на переднюю канал. Приложение будет вызываться обратно по его основному URL-адресу.

---

### <a name="sign-out-button"></a>Кнопка выхода

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

В ASP.NET, нажав кнопку **выйти** в веб-приложении, вы запускаете `SignOut` действие на `AccountController` контроллере (см. ниже).

```html
<ul class="navbar-nav">
  @if (User.Identity.IsAuthenticated)
  {
    <li class="nav-item">
        <span class="navbar-text text-dark">Hello @User.Identity.Name!</span>
    </li>
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignOut">Sign out</a>
    </li>
  }
  else
  {
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="MicrosoftIdentity" asp-controller="Account" asp-action="SignIn">Sign in</a>
    </li>
  }
</ul>
```

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

В ASP.NET MVC кнопка выхода предоставляется в `Views\Shared\_LoginPartial.cshtml` . Он отображается только при наличии учетной записи с проверкой подлинности. То есть он отображается, когда пользователь ранее выполнил вход.

```html
@if (Request.IsAuthenticated)
{
    <text>
        <ul class="nav navbar-nav navbar-right">
            <li class="navbar-text">
                Hello, @User.Identity.Name!
            </li>
            <li>
                @Html.ActionLink("Sign out", "SignOut", "Account")
            </li>
        </ul>
    </text>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li>@Html.ActionLink("Sign in", "SignIn", "Account", routeValues: null, htmlAttributes: new { id = "loginLink" })</li>
    </ul>
}
```

# <a name="java"></a>[Java](#tab/java)

В нашем кратком руководстве по Java кнопка выхода находится в файле main/Resources/Templates/auth_page.html.

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<body>

<form action="/msal4jsample/sign_out">
    <input type="submit" value="Sign out">
</form>
...
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Этот пример приложения не реализует выход.

# <a name="python"></a>[Python](#tab/python)

В кратком руководстве по Python кнопка выхода находится в файле [Templates/index.html # L10](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/e03be352914bfbd58be0d4170eba1fb7a4951d84/templates/index.html#L10) .

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
</head>
<body>
    <h1>Microsoft Identity Python web app</h1>
    Welcome {{ user.get("name") }}!
    <li><a href='/graphcall'>Call Microsoft Graph API</a></li>
    <li><a href="/logout">Logout</a></li>
</body>
</html>
```

---

### <a name="signout-action-of-the-controller"></a>`SignOut` действие контроллера

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

В предыдущих версиях шаблонов ASP.NET Core `Account` контроллер был внедрен в веб-приложение. Это не так, потому что контроллер теперь входит в пакет NuGet **Microsoft. Identity. Web. UI** . Дополнительные сведения см. в разделе [AccountController. CS](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web.UI/Areas/MicrosoftIdentity/Controllers/AccountController.cs) .

- Задает универсальный код ресурса (URI) перенаправления OpenID Connect, чтобы `/Account/SignedOut` контроллер вызывался обратно, когда Azure AD завершает выход.
- Вызовы `Signout()` , которые позволяют по промежуточного слоя OpenID Connect Connect обращаться к `logout` конечной точке платформы идентификации Майкрософт. Затем конечная точка:

  - Удаляет файл cookie сеанса из браузера.
  - Вызывает обратный URI перенаправления после выхода. По умолчанию URI перенаправления после выхода отображает страницу представление выхода с подписью [. cshtml. CS](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web.UI/Areas/MicrosoftIdentity/Pages/Account/SignedOut.cshtml.cs). Эта страница также предоставляется как часть Microsoft. Identity. Web.

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

В ASP.NET выход запускается из `SignOut()` метода на контроллере (например, [AccountController. CS # L25-l31](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/a2da310539aa613b77da1f9e1c17585311ab22b7/WebApp/Controllers/AccountController.cs#L25-L31)). Этот метод не является частью ASP.NET Framework, в отличие от того, что происходит в ASP.NET Core. Им

- Отправляет запрос на выход из OpenID Connect.
- Очищает кэш.
- Выполняет перенаправление на страницу, которую он хочет.

```csharp
/// <summary>
/// Send an OpenID Connect sign-out request.
/// </summary>
public void SignOut()
{
 HttpContext.GetOwinContext()
            .Authentication
            .SignOut(CookieAuthenticationDefaults.AuthenticationType);
 Response.Redirect("/");
}
```

# <a name="java"></a>[Java](#tab/java)

В Java выход осуществляется путем прямого вызова конечной точки платформы Microsoft Identity `logout` и предоставления `post_logout_redirect_uri` значения. Дополнительные сведения см. в разделе [ауспажеконтроллер. Java # L50-L60](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthPageController.java#L50-L60).

```Java
@RequestMapping("/msal4jsample/sign_out")
    public void signOut(HttpServletRequest httpRequest, HttpServletResponse response) throws IOException {

        httpRequest.getSession().invalidate();

        String endSessionEndpoint = "https://login.microsoftonline.com/common/oauth2/v2.0/logout";

        String redirectUrl = "http://localhost:8080/msal4jsample/";
        response.sendRedirect(endSessionEndpoint + "?post_logout_redirect_uri=" +
                URLEncoder.encode(redirectUrl, "UTF-8"));
    }
```

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Этот пример приложения не реализует выход.

# <a name="python"></a>[Python](#tab/python)

Код, который входит в систему пользователя, находится в [app. Корректировка # L46-l52](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/48637475ed7d7733795ebeac55c5d58663714c60/app.py#L47-L48).

```Python
@app.route("/logout")
def logout():
    session.clear()  # Wipe out the user and the token cache from the session
    return redirect(  # Also need to log out from the Microsoft Identity platform
        "https://login.microsoftonline.com/common/oauth2/v2.0/logout"
        "?post_logout_redirect_uri=" + url_for("index", _external=True))
```

---

### <a name="intercepting-the-call-to-the-logout-endpoint"></a>Перехват вызова к `logout` конечной точке

URI, выполняемый после выхода, позволяет приложениям участвовать в глобальном выходе.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

По промежуточного слоя ASP.NET Core OpenID Connect Connect позволяет приложению перехватывать вызовы конечной точки платформы идентификации Майкрософт `logout` , предоставляя событие OpenID Connect Connect с именем `OnRedirectToIdentityProviderForSignOut` . Это автоматически обрабатывается службой Microsoft. Identity. Web (которая очищает учетные записи в случае, когда веб-приложение вызывает веб-API).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

В ASP.NET вы делегируем по промежуточного слоя, чтобы выполнить выход, очистив файл cookie сеанса:

```csharp
public class AccountController : Controller
{
 ...
 public void EndSession()
 {
  Request.GetOwinContext().Authentication.SignOut();
  Request.GetOwinContext().Authentication.SignOut(Microsoft.AspNet.Identity.DefaultAuthenticationTypes.ApplicationCookie);
  this.HttpContext.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
 }
}
```

# <a name="java"></a>[Java](#tab/java)

В кратком руководстве по Java в URI перенаправления после выхода просто отображается страница index.html.

# <a name="nodejs"></a>[Node.js](#tab/nodejs)

Этот пример приложения не реализует выход.

# <a name="python"></a>[Python](#tab/python)

В кратком руководстве по Python URI перенаправления после выхода просто отображает страницу index.html.

---

## <a name="protocol"></a>Протокол

Если вы хотите узнать больше о выходе, прочитайте документацию по протоколу, доступную по адресу [Open ID Connect](./v2-protocols-oidc.md).

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [перейдите в рабочую среду](scenario-web-app-sign-user-production.md).