---
title: Получение маркера для вызова веб-API (классическое приложение) | Azure
titleSuffix: Microsoft identity platform
description: Узнайте, как создать классическое приложение, которое вызывает веб-API для получения маркера для приложения
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 01/06/2021
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 4a244c543aa83ae84891e3f942995dc340a7209d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99582661"
---
# <a name="desktop-app-that-calls-web-apis-acquire-a-token"></a>Классическое приложение, которое вызывает веб-API: Получение маркера

После создания экземпляра общедоступного клиентского приложения вы будете использовать его для получения маркера, который затем будет использоваться для вызова веб-API.

## <a name="recommended-pattern"></a>Рекомендуемый шаблон

Веб-API определяется по `scopes`. Независимо от интерфейса приложения используйте следующий шаблон:

- Систематически пытайтесь получить маркер из кэша маркеров, вызывая `AcquireTokenSilent`.
- Если этот вызов завершается неудачей, используйте нужный поток `AcquireToken`, который представлен здесь как `AcquireTokenXX`.

# <a name="net"></a>[.NET](#tab/dotnet)

### <a name="in-msalnet"></a>В MSAL.NET

```csharp
AuthenticationResult result;
var accounts = await app.GetAccountsAsync();
IAccount account = ChooseAccount(accounts); // for instance accounts.FirstOrDefault
                                            // if the app manages is at most one account
try
{
 result = await app.AcquireTokenSilent(scopes, account)
                   .ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
  result = await app.AcquireTokenXX(scopes, account)
                    .WithOptionalParameterXXX(parameter)
                    .ExecuteAsync();
}
```

# <a name="java"></a>[Java](#tab/java)

```java

Set<IAccount> accountsInCache = pca.getAccounts().join();
// Take first account in the cache. In a production application, you would filter
// accountsInCache to get the right account for the user authenticating.
IAccount account = accountsInCache.iterator().next();

IAuthenticationResult result;
try {
    SilentParameters silentParameters =
            SilentParameters
                    .builder(SCOPE, account)
                    .build();

    // try to acquire token silently. This call will fail since the token cache
    // does not have any data for the user you are trying to acquire a token for
    result = pca.acquireTokenSilently(silentParameters).join();
} catch (Exception ex) {
    if (ex.getCause() instanceof MsalException) {

        InteractiveRequestParameters parameters = InteractiveRequestParameters
                .builder(new URI("http://localhost"))
                .scopes(SCOPE)
                .build();

        // Try to acquire a token interactively with system browser. If successful, you should see
        // the token and account information printed out to console
        result = pca.acquireToken(parameters).join();
    } else {
        // Handle other exceptions accordingly
        throw ex;
    }
}
return result;

```

# <a name="python"></a>[Python](#tab/python)

```Python
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    result = app.acquire_token_by_xxx(scopes=config["scope"])
```

# <a name="macos"></a>[macOS](#tab/macOS)

### <a name="in-msal-for-ios-and-macos"></a>В MSAL для iOS и macOS

Objective-C.

```objc
MSALAccount *account = [application accountForIdentifier:accountIdentifier error:nil];

MSALSilentTokenParameters *silentParams = [[MSALSilentTokenParameters alloc] initWithScopes:scopes account:account];
[application acquireTokenSilentWithParameters:silentParams completionBlock:^(MSALResult *result, NSError *error) {

    // Check the error
    if (error && [error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
    {
        // Interactive auth will be required, call acquireTokenWithParameters:error:
    }
}];
```
Swift:

```swift
guard let account = try? application.account(forIdentifier: accountIdentifier) else { return }
let silentParameters = MSALSilentTokenParameters(scopes: scopes, account: account)
application.acquireTokenSilent(with: silentParameters) { (result, error) in

    guard let authResult = result, error == nil else {

    let nsError = error! as NSError

        if (nsError.domain == MSALErrorDomain &&
            nsError.code == MSALError.interactionRequired.rawValue) {

            // Interactive auth will be required, call acquireToken()
            return
        }
        return
    }
}
```
---

Ниже приведены различные способы получения маркеров в классическом приложении.

## <a name="acquire-a-token-interactively"></a>Интерактивное получение маркера

В следующем примере показан минимальный код для интерактивного получения маркера для чтения профиля пользователя с Microsoft Graph.

# <a name="net"></a>[.NET](#tab/dotnet)
### <a name="in-msalnet"></a>В MSAL.NET

```csharp
string[] scopes = new string[] {"user.read"};
var app = PublicClientApplicationBuilder.Create(clientId).Build();
var accounts = await app.GetAccountsAsync();
AuthenticationResult result;
try
{
 result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
             .ExecuteAsync();
}
catch(MsalUiRequiredException)
{
 result = await app.AcquireTokenInteractive(scopes)
             .ExecuteAsync();
}
```

### <a name="mandatory-parameters"></a>Обязательные параметры

`AcquireTokenInteractive` имеет только один обязательный параметр, ``scopes``, содержащий перечисление строк, определяющих области, для которых требуется маркер. Если маркер предназначен для Microsoft Graph, необходимые области можно найти в справочнике по API каждого Microsoft Graph API в разделе "Разрешения". Например, для [списка контактов пользователя](/graph/api/user-list-contacts) необходимо использовать область "User.Read", "Contacts.Read". См. сведения о [разрешениях Microsoft Graph](/graph/permissions-reference).

В Android также необходимо указать родительское действие с помощью `.WithParentActivityOrWindow`, как показано, чтобы маркер был возвращен родительскому действию после взаимодействия. Если оно не указано, при вызове `.ExecuteAsync()` возникает исключение.

### <a name="specific-optional-parameters-in-msalnet"></a>Определенные необязательные параметры в MSAL.NET

#### <a name="withparentactivityorwindow"></a>WithParentActivityOrWindow

Пользовательский интерфейс важен, так как он является интерактивным. `AcquireTokenInteractive` имеет один конкретный необязательный параметр, который может указывать родительский пользовательский интерфейс для поддерживающих платформ. При использовании в классическом приложении `.WithParentActivityOrWindow` имеет другой тип, который зависит от платформы. Кроме того, можно опустить необязательный параметр родительского окна, чтобы создать окно, если вы не хотите управлять тем, где на экране появляется диалоговое окно входа. Это применимо к приложениям, которые являются командной строкой и используются для передачи вызовов в любую другую серверную службу и не требуют каких бы то ни было окон для взаимодействия с пользователем.

```csharp
// net45
WithParentActivityOrWindow(IntPtr windowPtr)
WithParentActivityOrWindow(IWin32Window window)

// Mac
WithParentActivityOrWindow(NSWindow window)

// .Net Standard (this will be on all platforms at runtime, but only on NetStandard at build time)
WithParentActivityOrWindow(object parent).
```

Примечания:

- В .NET Standard ожидаемый `object` — `Activity` в Android, `UIViewController` в iOS, `NSWindow` на MAC и `IWin32Window` или `IntPr` в Windows.
- В Windows необходимо вызвать `AcquireTokenInteractive` из потока пользовательского интерфейса, чтобы встроенный браузер получил соответствующий контекст синхронизации пользовательского интерфейса. Отсутствие вызова из потока пользовательского интерфейса может привести к тому, что сообщения не будут передаваться должным образом и возникнут сценарии взаимоблокировок с пользовательским интерфейсом. Один из способов вызова библиотек проверки подлинности Майкрософт (MSAL) из потока пользовательского интерфейса, если вы еще не находитесь в потоке пользовательского интерфейса, — использовать `Dispatcher` в WPF.
- Если вы используете WPF для получения окна из элемента управления WPF, можно использовать класс `WindowInteropHelper.Handle`. Затем вызов осуществляется из элемента управления WPF (`this`):

  ```csharp
  result = await app.AcquireTokenInteractive(scopes)
                    .WithParentActivityOrWindow(new WindowInteropHelper(this).Handle)
                    .ExecuteAsync();
  ```

#### <a name="withprompt"></a>WithPrompt

`WithPrompt()` используется для управления интерактивным взаимодействием с пользователем путем указания запроса.

![Изображение, показывающее поля в структуре запроса. Эти постоянные значения управляют взаимодействием с пользователем, определяя тип запроса, отображаемого методом Виспромпт ().](https://user-images.githubusercontent.com/13203188/53438042-3fb85700-39ff-11e9-9a9e-1ff9874197b3.png)

Класс определяет следующие константы:

- ``SelectAccount`` указывает STS представлять диалоговое окно выбора учетной записи, содержащее учетные записи, для которых у пользователя есть сеанс. Этот параметр полезен, если разработчики приложений хотят позволить пользователям выбирать между разными удостоверениями. Этот параметр позволяет MSAL отправлять ``prompt=select_account`` поставщику удостоверений. Этот параметр выбирается по умолчанию. Это хороший способ обеспечить оптимальное взаимодействие на основе доступной информации, такой как учетная запись и присутствие сеанса для пользователя. Не изменяйте его, если у вас нет веских причин.
- ``Consent`` позволяет разработчику приложения принудительно запрашивать согласие пользователя, даже если оно уже было предоставлено. В этом случае MSAL отправляет `prompt=consent` поставщику удостоверений. Этот вариант можно использовать в некоторых приложениях, ориентированных на безопасность, где управление организацией требует, чтобы пользователю предоставлялось диалоговое окно согласия при каждом использовании приложения.
- ``ForceLogin`` позволяет разработчику приложения отправлять пользователю запрос учетных данных от службы, даже если этот запрос может не потребоваться. Этот вариант может быть полезен, когда нужно позволить пользователю снова войти в систему при сбое получения маркера. В этом случае MSAL отправляет `prompt=login` поставщику удостоверений. Иногда этот вариант используется в приложениях, ориентированных на безопасность, где управление организацией требует, чтобы пользователь повторно выполнял вход при каждом обращении к конкретным частям приложения.
- ``Never`` (только для .NET 4.5 и WinRT) не запрашивает пользователя, а вместо этого пытается использовать файл cookie, хранящийся в скрытом внедренном веб-представлении. Дополнительные сведения см. в разделе о веб-представлениях для MSAL.NET. Использование этого варианта может привести к сбою. В этом случае `AcquireTokenInteractive` создает исключение, чтобы уведомить о необходимости взаимодействия с пользовательским интерфейсом. Необходимо использовать другой параметр `Prompt`.
- ``NoPrompt`` не отправляет запрос поставщику удостоверений. Этот параметр полезен только для политик изменения профилей Azure Active Directory (Azure AD) B2C. Дополнительные сведения см. в разделе об [особенностях Azure AD B2C](https://aka.ms/msal-net-b2c-specificities).

#### <a name="withextrascopetoconsent"></a>WithExtraScopeToConsent

Этот модификатор используется в расширенном сценарии, где пользователь должен заранее согласиться на несколько ресурсов и вы не хотите использовать добавочное согласие, которое обычно используется с платформой удостоверений Microsoft или MSAL.NET. Дополнительные сведения см. в разделе [Получение согласия пользователя на несколько ресурсов заранее](scenario-desktop-production.md#have-the-user-consent-upfront-for-several-resources).

```csharp
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

#### <a name="withcustomwebui"></a>WithCustomWebUi

Веб-интерфейс — это механизм для вызова браузера. Этот механизм может быть выделенным элементом управления WebBrowser пользовательского интерфейса или способом делегирования открытия браузера.
MSAL предоставляет реализации пользовательского веб-интерфейса для большинства платформ, но бывают случаи, когда может потребоваться разместить браузер самостоятельно:

- Платформы, которые не были явно включены в MSAL, например Blazor, Unity и Mono на настольных компьютерах.
- Необходимо протестировать пользовательский интерфейс приложения и использовать автоматизированный браузер, который можно использовать с Selenium.
- Браузер и приложение, в котором выполняется MSAL, находятся в отдельных процессах.

##### <a name="at-a-glance"></a>Вкратце

Для этого вы передаете MSAL `start Url`, который должен отображаться в выбранном браузере, чтобы конечный пользователь мог ввести такие элементы, как имя пользователя.
После завершения проверки подлинности приложению необходимо передать обратно в MSAL `end Url`, который содержит код, предоставленный Azure AD.
Узел `end Url` — это всегда `redirectUri`. Чтобы перехватить `end Url`, выполните одно из следующих действий.

- Отслеживайте перенаправления браузера, пока не будет достигнут `redirect Url`.
- Настройте перенаправление браузера на URL-адрес, который вы отслеживаете.

##### <a name="withcustomwebui-is-an-extensibility-point"></a>WithCustomWebUi — это точка расширяемости

`WithCustomWebUi` — это точка расширяемости, которую можно использовать для предоставления собственного пользовательского интерфейса в общедоступных клиентских приложениях. Вы также можете разрешить пользователям проходить через конечную точку /Authorize поставщика удостоверений и предоставить им возможность входа и согласия. MSAL.NET может затем активировать код проверки подлинности и получить маркер. Например, он используется в Visual Studio для веб-взаимодействия в приложениях, например для отзывов Visual Studio, но MSAL.NET может выполнить большую часть этой задачи. Его также можно использовать, если требуется обеспечить автоматизацию пользовательского интерфейса. В общедоступных клиентских приложениях MSAL.NET использует стандарт ключа проверки для обмена кодом (PKCE), чтобы обеспечить соблюдение безопасности. Только MSAL.NET может активировать код. Дополнительные сведения см. в разделе [RFC 7636 — ключ проверки для обмена кодом с помощью общедоступных клиентов OAuth](https://tools.ietf.org/html/rfc7636).

  ```csharp
  using Microsoft.Identity.Client.Extensions;
  ```

##### <a name="use-withcustomwebui"></a>Использование WithCustomWebUi

Чтобы использовать `.WithCustomWebUI`, выполните следующие действия.

  1. Реализуйте интерфейс `ICustomWebUi`. Дополнительные сведения см. [на этом веб-сайте](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/blob/053a98d16596be7e9ca1ab916924e5736e341fe8/src/Microsoft.Identity.Client/Extensibility/ICustomWebUI.cs#L32-L70). Реализуйте один метод `AcquireAuthorizationCodeAsync` и примите URL-адрес кода авторизации, вычисленный MSAL.NET. Затем позвольте пользователю взаимодействовать с поставщиком удостоверений и получить URL-адрес, по которому поставщик удостоверений может вызвать свою реализацию с кодом авторизации. Если возникли проблемы, ваша реализация должна вызвать исключение `MsalExtensionException`, чтобы корректно взаимодействовать с MSAL.
  2. В вызове `AcquireTokenInteractive` используйте модификатор `.WithCustomUI()`, передающий экземпляр пользовательского веб-интерфейса.

     ```csharp
     result = await app.AcquireTokenInteractive(scopes)
                       .WithCustomWebUi(yourCustomWebUI)
                       .ExecuteAsync();
     ```

##### <a name="examples-of-implementation-of-icustomwebui-in-test-automation-seleniumwebui"></a>Примеры реализации ICustomWebUi в автоматизации тестирования: SeleniumWebUI

Команда MSAL.NET переписала тесты пользовательского интерфейса для использования этого механизма расширяемости. Если вам интересно, ознакомьтесь с классом [SeleniumWebUI](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/blob/053a98d16596be7e9ca1ab916924e5736e341fe8/tests/Microsoft.Identity.Test.Integration/Infrastructure/SeleniumWebUI.cs#L15-L160) в исходном коде MSAL.NET.

##### <a name="provide-a-great-experience-with-systemwebviewoptions"></a>Удобная работа с SystemWebViewOptions

В MSAL.NET 4.1 [`SystemWebViewOptions`](/dotnet/api/microsoft.identity.client.systemwebviewoptions) можно указать:

- URI для перехода (`BrowserRedirectError`) или отображаемый фрагмент HTML (`HtmlMessageError`) в случае ошибок входа или согласия в системном веб-браузере.
- URI для перехода (`BrowserRedirectSuccess`) или отображаемый фрагмент HTML (`HtmlMessageSuccess`) в случае успешного входа или согласия.
- Действие, выполняемое для запуска системного браузера. Вы можете предоставить собственную реализацию, установив делегат `OpenBrowserAsync`. Класс также предоставляет реализацию по умолчанию для двух браузеров: `OpenWithEdgeBrowserAsync` и `OpenWithChromeEdgeBrowserAsync` для Microsoft Edge и [Microsoft Edge в Chromium](https://www.windowscentral.com/faq-edge-chromium) соответственно.

Чтобы использовать эту структуру, напишите примерно следующий код:

```csharp
IPublicClientApplication app;
...

options = new SystemWebViewOptions
{
 HtmlMessageError = "<b>Sign-in failed. You can close this tab ...</b>",
 BrowserRedirectSuccess = "https://contoso.com/help-for-my-awesome-commandline-tool.html"
};

var result = app.AcquireTokenInteractive(scopes)
                .WithEmbeddedWebView(false)       // The default in .NET Core
                .WithSystemWebViewOptions(options)
                .Build();
```

#### <a name="other-optional-parameters"></a>Другие необязательные параметры

Дополнительные сведения о других необязательных параметрах для `AcquireTokenInteractive` см. в разделе [AcquireTokenInteractiveParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokeninteractiveparameterbuilder#methods).

# <a name="java"></a>[Java](#tab/java)

```java
private static IAuthenticationResult acquireTokenInteractive() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            InteractiveRequestParameters parameters = InteractiveRequestParameters
                    .builder(new URI("http://localhost"))
                    .scopes(SCOPE)
                    .build();

            // Try to acquire a token interactively with system browser. If successful, you should see
            // the token and account information printed out to console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

MSAL Python не предоставляет прямой интерактивный метод получения маркера. Вместо этого для получения кода авторизации приложение должно отправить запрос авторизации в своей реализации потока взаимодействия с пользователем. Этот код затем может быть передан методу `acquire_token_by_authorization_code` для получения маркера.

```Python
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    result = app.acquire_token_by_authorization_code(
         request.args['code'],
         scopes=config["scope"])

```

# <a name="macos"></a>[macOS](#tab/macOS)

### <a name="in-msal-for-ios-and-macos"></a>В MSAL для iOS и macOS

Objective-C.

```objc
MSALInteractiveTokenParameters *interactiveParams = [[MSALInteractiveTokenParameters alloc] initWithScopes:scopes webviewParameters:[MSALWebviewParameters new]];
[application acquireTokenWithParameters:interactiveParams completionBlock:^(MSALResult *result, NSError *error) {
    if (!error)
    {
        // You'll want to get the account identifier to retrieve and reuse the account
        // for later acquireToken calls
        NSString *accountIdentifier = result.account.identifier;

        NSString *accessToken = result.accessToken;
    }
}];
```

Swift:

```swift
let interactiveParameters = MSALInteractiveTokenParameters(scopes: scopes, webviewParameters: MSALWebviewParameters())
application.acquireToken(with: interactiveParameters, completionBlock: { (result, error) in

    guard let authResult = result, error == nil else {
        print(error!.localizedDescription)
        return
    }

    // Get access token from result
    let accessToken = authResult.accessToken
})
```
---

## <a name="integrated-windows-authentication"></a>Встроенная проверка подлинности Windows

Чтобы реализовать вход в систему для пользователя домена на компьютере, присоединенном к домену или Azure AD, используйте встроенную проверку подлинности Windows (IWA).

### <a name="constraints"></a>Ограничения

- Встроенная проверка подлинности Windows может использоваться только для *федеративных* пользователей, то есть пользователей, созданных в Active Directory и поддерживаемых Azure AD. Пользователи, созданные непосредственно в Azure AD без поддержки Active Directory (*управляемые* пользователи), не могут использовать этот поток проверки подлинности. Это ограничение не влияет на поток имени пользователя и пароля.
- IWA предназначена для приложений, написанных для платформ .NET Framework, .NET Core и универсальной платформы Windows.
- IWA не обходит [многофакторную проверку подлинности](../authentication/concept-mfa-howitworks.md). Если настроена MFA, IWA может завершиться ошибкой, если требуется запрос MFA, так как для MFA требуется вмешательство пользователя.
  
    IWA не является интерактивным, но MFA требует взаимодействия с пользователем. Вы не контролируете, когда поставщик удостоверений запрашивает MFA. Это задача администратора клиента. По нашим наблюдениям, MFA требуется при входе из другой страны или региона, если вы не подключены через VPN к корпоративной сети, и иногда даже при подключении через VPN. Не следует рассчитывать на детерминированный набор правил. Azure AD использует искусственный интеллект для непрерывного изучения необходимости MFA. Вернитесь к использованию запросов пользователя, например к интерактивной проверке подлинности или потоку кода устройства, при сбое IWA.

- Центр, переданный `PublicClientApplicationBuilder`, должен иметь следующие характеристики:
  - Клиентский, вида `https://login.microsoftonline.com/{tenant}/`, где `tenant` — это идентификатор GUID, представляющий идентификатор клиента или домен, связанный с клиентом.
  - Для всех рабочих и учебных учетных записей: `https://login.microsoftonline.com/organizations/`.
  - Личные учетные записи Майкрософт не поддерживаются. Нельзя использовать клиенты /common или /consumers.

- Встроенная проверка подлинности Windows является автоматическим потоком, поэтому:
  - Пользователь приложения должен быть предварительно одобрен, чтобы использовать приложение.
  - Администратор клиента должен предварительно одобрить всех пользователей в клиенте для использования приложения.
  - Другими словами:
    - Вы как разработчик выбрали **Предоставить разрешения** на портале Azure для себя.
    - Администратор клиента выбрал **Предоставить или отозвать согласие администратора для {домен клиента}** на вкладке **Разрешения API** в разделе регистрации приложения. Дополнительные сведения см. [в статье Добавление разрешений для доступа к веб-API](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api).
    - Или вы предоставили пользователям возможность согласиться на приложение. Дополнительные сведения см. в разделе [Запрос на получение согласия одного пользователя](./v2-permissions-and-consent.md#requesting-individual-user-consent).
    - Или вы предоставили администратору клиента возможность согласиться на приложение. Дополнительные сведения см. в разделе [Согласие администратора](./v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

- Этот поток включен для классических приложений .NET, приложений .NET Core и UWP.

Дополнительные сведения о согласии см. в разделе [разрешения и согласие платформы Microsoft Identity](./v2-permissions-and-consent.md).

### <a name="learn-how-to-use-it"></a>Сведения об использовании

# <a name="net"></a>[.NET](#tab/dotnet)

В MSAL.NET используйте:

```csharp
AcquireTokenByIntegratedWindowsAuth(IEnumerable<string> scopes)
```

Обычно требуется только один параметр (`scopes`). В зависимости от способа настройки политик администратором Windows приложениям на компьютере Windows может быть запрещено выполнять поиск пользователя, выполнившего вход. В этом случае используйте второй метод, `.WithUsername()`, и передайте имя пользователя, выполнившего вход, в формате UPN, например `joe@contoso.com`.

В следующем примере представлен наиболее актуальный вариант с объяснениями типов исключений, которые могут возникать, и их устранения.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app = PublicClientApplicationBuilder
      .Create(clientId)
      .WithAuthority(authority)
      .Build();

 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
      .ExecuteAsync();
 }
 else
 {
  try
  {
   result = await app.AcquireTokenByIntegratedWindowsAuth(scopes)
      .ExecuteAsync(CancellationToken.None);
  }
  catch (MsalUiRequiredException ex)
  {
   // MsalUiRequiredException: AADSTS65001: The user or administrator has not consented to use the application
   // with ID '{appId}' named '{appName}'.Send an interactive authorization request for this user and resource.

   // you need to get user consent first. This can be done, if you are not using .NET Core (which does not have any Web UI)
   // by doing (once only) an AcquireToken interactive.

   // If you are using .NET core or don't want to do an AcquireTokenInteractive, you might want to suggest the user to navigate
   // to a URL to consent: https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read

   // AADSTS50079: The user is required to use multi-factor authentication.
   // There is no mitigation - if MFA is configured for your tenant and AAD decides to enforce it,
   // you need to fallback to an interactive flows such as AcquireTokenInteractive or AcquireTokenByDeviceCode
   }
   catch (MsalServiceException ex)
   {
    // Kind of errors you could have (in ex.Message)

    // MsalServiceException: AADSTS90010: The grant type is not supported over the /common or /consumers endpoints. Please use the /organizations or tenant-specific endpoint.
    // you used common.
    // Mitigation: as explained in the message from Azure AD, the authority needs to be tenanted or otherwise organizations

    // MsalServiceException: AADSTS70002: The request body must contain the following parameter: 'client_secret or client_assertion'.
    // Explanation: this can happen if your application was not registered as a public client application in Azure AD
    // Mitigation: in the Azure portal, edit the manifest for your application and set the `allowPublicClient` to `true`
   }
   catch (MsalClientException ex)
   {
      // Error Code: unknown_user Message: Could not identify logged in user
      // Explanation: the library was unable to query the current Windows logged-in user or this user is not AD or AAD
      // joined (work-place joined users are not supported).

      // Mitigation 1: on UWP, check that the application has the following capabilities: Enterprise Authentication,
      // Private Networks (Client and Server), User Account Information

      // Mitigation 2: Implement your own logic to fetch the username (e.g. john@contoso.com) and use the
      // AcquireTokenByIntegratedWindowsAuth form that takes in the username

      // Error Code: integrated_windows_auth_not_supported_managed_user
      // Explanation: This method relies on an a protocol exposed by Active Directory (AD). If a user was created in Azure
      // Active Directory without AD backing ("managed" user), this method will fail. Users created in AD and backed by
      // AAD ("federated" users) can benefit from this non-interactive method of authentication.
      // Mitigation: Use interactive authentication
   }
 }


 Console.WriteLine(result.Account.Username);
}
```

Список возможных модификаторов AcquireTokenByIntegratedWindowsAuthentication см. в разделе [AcquireTokenByIntegratedWindowsAuthParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokenbyintegratedwindowsauthparameterbuilder#methods).

# <a name="java"></a>[Java](#tab/java)

Это фрагмент из [примеров разработки в MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```Java
private static IAuthenticationResult acquireTokenIwa() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            IntegratedWindowsAuthenticationParameters parameters =
                    IntegratedWindowsAuthenticationParameters
                            .builder(SCOPE, USER_NAME)
                            .build();

            // Try to acquire a IWA. You will need to generate a Kerberos ticket.
            // If successful, you should see the token and account information printed out to
            // console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Этот поток пока не поддерживается в MSAL Python.

# <a name="macos"></a>[macOS](#tab/macOS)

Этот поток не применяется к macOS.

---

## <a name="username-and-password"></a>Имя пользователя и пароль

Можно также получить маркер, указав имя пользователя и пароль. Этот поток ограничен и не рекомендуется, но по-прежнему необходим в некоторых вариантах использования.

### <a name="this-flow-isnt-recommended"></a>Этот поток не рекомендуется.

*Не рекомендуется использовать* имя пользователя и поток пароля, так как приложение запрашивает у пользователя пароль небезопасен. Дополнительные сведения см. в разделе [что такое решение для растущей проблемы паролей?](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/) Предпочтительным потоком для автоматического получения маркера на компьютерах, присоединенных к домену Windows, является [встроенная проверка подлинности Windows](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Integrated-Windows-Authentication). Также можно также использовать [поток кода устройства](https://aka.ms/msal-net-device-code-flow).

Использование имени пользователя и пароля полезно в некоторых случаях, например в сценариях DevOps. Но если вы хотите использовать имя пользователя и пароль в интерактивных сценариях, где вы предоставляете свой собственный пользовательский интерфейс, подумайте о том, как отказаться от этого варианта. Используя имя пользователя и пароль, вы отказываетесь от ряда преимуществ:

- Основные принципы современных удостоверений. Пароль можно выманить и воспроизвести, так как можно перехватить общий секрет. Этот вариант несовместим со входом без пароля.
- Пользователи, которым необходимо выполнять MFA, не могут войти в систему, так как взаимодействие отсутствует.
- Пользователи не могут применять единый вход (SSO).

### <a name="constraints"></a>Ограничения

Также действуют следующие ограничения.

- Поток имени пользователя и пароля несовместим с условным доступом и многофакторной проверкой подлинности. Как следствие, если приложение выполняется в клиенте Azure AD, где администратор клиента требует многофакторную проверку подлинности, вы не сможете использовать этот поток. Так поступают многие организации.
- Это подходит только для рабочих и учебных учетных записей (не MSA).
- Поток доступен для классических приложений .NET и .NET Core, но не для UWP.

### <a name="b2c-specifics"></a>Особенности B2C

Дополнительные сведения см. в разделе [Учетные данные владельца ресурса (ROPC) с B2C](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics#resource-owner-password-credentials-ropc-with-b2c).

### <a name="use-it"></a>Использование

# <a name="net"></a>[.NET](#tab/dotnet)

`IPublicClientApplication` содержит метод `AcquireTokenByUsernamePassword`.

В следующем примере представлен упрощенный вариант.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app;
 app = PublicClientApplicationBuilder.Create(clientId)
       .WithAuthority(authority)
       .Build();
 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
 }
 else
 {
  try
  {
   var securePassword = new SecureString();
   foreach (char c in "dummy")        // you should fetch the password
    securePassword.AppendChar(c);  // keystroke by keystroke

   result = await app.AcquireTokenByUsernamePassword(scopes,
                                                    "joe@contoso.com",
                                                     securePassword)
                      .ExecuteAsync();
  }
  catch(MsalException)
  {
   // See details below
  }
 }
 Console.WriteLine(result.Account.Username);
}
```

В следующем примере представлен наиболее актуальный вариант с объяснениями типов исключений, которые могут возникать, и их устранения.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app;
 app = PublicClientApplicationBuilder.Create(clientId)
                                   .WithAuthority(authority)
                                   .Build();
 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
 }
 else
 {
  try
  {
   var securePassword = new SecureString();
   foreach (char c in "dummy")        // you should fetch the password keystroke
    securePassword.AppendChar(c);  // by keystroke

   result = await app.AcquireTokenByUsernamePassword(scopes,
                                                    "joe@contoso.com",
                                                    securePassword)
                    .ExecuteAsync();
  }
  catch (MsalUiRequiredException ex) when (ex.Message.Contains("AADSTS65001"))
  {
   // Here are the kind of error messages you could have, and possible mitigations

   // ------------------------------------------------------------------------
   // MsalUiRequiredException: AADSTS65001: The user or administrator has not consented to use the application
   // with ID '{appId}' named '{appName}'. Send an interactive authorization request for this user and resource.

   // Mitigation: you need to get user consent first. This can be done either statically (through the portal),
   /// or dynamically (but this requires an interaction with Azure AD, which is not possible with
   // the username/password flow)
   // Statically: in the portal by doing the following in the "API permissions" tab of the application registration:
   // 1. Click "Add a permission" and add all the delegated permissions corresponding to the scopes you want (for instance
   // User.Read and User.ReadBasic.All)
   // 2. Click "Grant/revoke admin consent for <tenant>") and click "yes".
   // Dynamically, if you are not using .NET Core (which does not have any Web UI) by
   // calling (once only) AcquireTokenInteractive.
   // remember that Username/password is for public client applications that is desktop/mobile applications.
   // If you are using .NET core or don't want to call AcquireTokenInteractive, you might want to:
   // - use device code flow (See https://aka.ms/msal-net-device-code-flow)
   // - or suggest the user to navigate to a URL to consent: https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ErrorCode: invalid_grant
   // SubError: basic_action
   // MsalUiRequiredException: AADSTS50079: The user is required to use multi-factor authentication.
   // The tenant admin for your organization has chosen to oblige users to perform multi-factor authentication.
   // Mitigation: none for this flow
   // Your application cannot use the Username/Password grant.
   // Like in the previous case, you might want to use an interactive flow (AcquireTokenInteractive()),
   // or Device Code Flow instead.
   // Note this is one of the reason why using username/password is not recommended;
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ex.ErrorCode: invalid_grant
   // subError: null
   // Message = "AADSTS70002: Error validating credentials.
   // AADSTS50126: Invalid username or password
   // In the case of a managed user (user from an Azure AD tenant opposed to a
   // federated user, which would be owned
   // in another IdP through ADFS), the user has entered the wrong password
   // Mitigation: ask the user to re-enter the password
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ex.ErrorCode: invalid_grant
   // subError: null
   // MsalServiceException: ADSTS50034: To sign into this application the account must be added to
   // the {domainName} directory.
   // or The user account does not exist in the {domainName} directory. To sign into this application,
   // the account must be added to the directory.
   // The user was not found in the directory
   // Explanation: wrong username
   // Mitigation: ask the user to re-enter the username.
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_request")
  {
   // ------------------------------------------------------------------------
   // AADSTS90010: The grant type is not supported over the /common or /consumers endpoints.
   // Please use the /organizations or tenant-specific endpoint.
   // you used common.
   // Mitigation: as explained in the message from Azure AD, the authority you use in the application needs
   // to be tenanted or otherwise "organizations". change the
   // "Tenant": property in the appsettings.json to be a GUID (tenant Id), or domain name (contoso.com)
   // if such a domain is registered with your tenant
   // or "organizations", if you want this application to sign-in users in any Work and School accounts.
   // ------------------------------------------------------------------------

  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "unauthorized_client")
  {
   // ------------------------------------------------------------------------
   // AADSTS700016: Application with identifier '{clientId}' was not found in the directory '{domain}'.
   // This can happen if the application has not been installed by the administrator of the tenant or consented
   // to by any user in the tenant.
   // You may have sent your authentication request to the wrong tenant
   // Cause: The clientId in the appsettings.json might be wrong
   // Mitigation: check the clientId and the app registration
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_client")
  {
   // ------------------------------------------------------------------------
   // AADSTS70002: The request body must contain the following parameter: 'client_secret or client_assertion'.
   // Explanation: this can happen if your application was not registered as a public client application in Azure AD
   // Mitigation: in the Azure portal, edit the manifest for your application and set the `allowPublicClient` to `true`
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException)
  {
   throw;
  }

  catch (MsalClientException ex) when (ex.ErrorCode == "unknown_user_type")
  {
   // Message = "Unsupported User Type 'Unknown'. Please see https://aka.ms/msal-net-up"
   // The user is not recognized as a managed user, or a federated user. Azure AD was not
   // able to identify the IdP that needs to process the user
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "user_realm_discovery_failed")
  {
   // The user is not recognized as a managed user, or a federated user. Azure AD was not
   // able to identify the IdP that needs to process the user. That's for instance the case
   // if you use a phone number
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "unknown_user")
  {
   // the username was probably empty
   // ex.Message = "Could not identify the user logged into the OS. See https://aka.ms/msal-net-iwa for details."
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "parsing_wstrust_response_failed")
  {
   // ------------------------------------------------------------------------
   // In the case of a Federated user (that is owned by a federated IdP, as opposed to a managed user owned in an Azure AD tenant)
   // ID3242: The security token could not be authenticated or authorized.
   // The user does not exist or has entered the wrong password
   // ------------------------------------------------------------------------
  }
 }

 Console.WriteLine(result.Account.Username);
}
```

Дополнительные сведения обо всех модификаторах, которые можно применить к `AcquireTokenByUsernamePassword`, см. в разделе [AcquireTokenByUsernamePasswordParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokenbyusernamepasswordparameterbuilder#methods).

# <a name="java"></a>[Java](#tab/java)

Это фрагмент из [примеров разработки в MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```Java
private static IAuthenticationResult acquireTokenUsernamePassword() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();
        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            UserNamePasswordParameters parameters =
                    UserNamePasswordParameters
                            .builder(SCOPE, USER_NAME, USER_PASSWORD.toCharArray())
                            .build();
            // Try to acquire a token via username/password. If successful, you should see
            // the token and account information printed out to console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Это фрагмент из [примеров разработки в MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/dev/sample/).

```Python
# Create a preferably long-lived app instance which maintains a token cache.
app = msal.PublicClientApplication(
    config["client_id"], authority=config["authority"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )

# The pattern to acquire a token looks like this.
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    logging.info("Account(s) exists in cache, probably with token too. Let's try.")
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    # See this page for constraints of Username Password Flow.
    # https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki/Username-Password-Authentication
    result = app.acquire_token_by_username_password(
        config["username"], config["password"], scopes=config["scope"])
```

# <a name="macos"></a>[macOS](#tab/macOS)

Этот поток не поддерживается в MSAL для macOS.

---

## <a name="command-line-tool-without-a-web-browser"></a>Средство командной строки без веб-браузера

### <a name="device-code-flow"></a>Поток кода устройства

Если вы создаете программу командной строки, которая не содержит веб-элементы управления, и вы не можете использовать предыдущие последовательности или не хотите, используйте поток кода устройства.

Для интерактивной проверки подлинности в Azure AD требуется веб-браузер. Дополнительные сведения см. в разделе [Использование веб-браузеров](https://aka.ms/msal-net-uses-web-browser). Для проверки подлинности пользователей на устройствах или в операционных системах без веб-браузера поток кода устройства позволяет пользователю использовать другое устройство (например, другой компьютер или мобильный телефон) для входа в интерактивном режиме. С помощью потока кода устройства приложение получает маркеры через двухэтапный процесс, предназначенный для этих устройств или операционных систем. Это могут быть приложения, работающие в системе Интернета вещей или программах командной строки (CLI). Идея состоит в следующем.

1. Каждый раз, когда требуется проверка подлинности пользователя, приложение предоставляет код для пользователя. Пользователю предлагается использовать другое устройство, например смартфон, подключенный к Интернету, для перехода по URL-адресу, например `https://microsoft.com/devicelogin`. Пользователь получает запрос на ввод кода. После ввода веб-страница предоставляет обычный процесс проверки подлинности, включая запросы согласия и многофакторную проверку подлинности, если это необходимо.

2. После успешной проверки подлинности приложение командной строки получает необходимые маркеры через канал обратного вызова и использует их для выполнения необходимых вызовов веб-API.

### <a name="use-it"></a>Использование

# <a name="net"></a>[.NET](#tab/dotnet)

`IPublicClientApplication` содержит метод `AcquireTokenWithDeviceCode`.

```csharp
 AcquireTokenWithDeviceCode(IEnumerable<string> scopes,
                            Func<DeviceCodeResult, Task> deviceCodeResultCallback)
```

Этот метод принимает три параметра:

- `scopes` для запроса маркера доступа.
- Обратный вызов, получающий `DeviceCodeResult`.

  ![Свойства DeviceCodeResult](https://user-images.githubusercontent.com/13203188/56024968-7af1b980-5d11-11e9-84c2-5be2ef306dc5.png)

В следующем образце кода представлен краткий обзор большинства текущих случаев с объяснениями типов исключений, которые можно получить, и их устранение. Полностью функциональный пример кода см. в разделе [Active-Directory-команда dotnetcore-девицекодефлов-v2](https://github.com/azure-samples/active-directory-dotnetcore-devicecodeflow-v2) на сайте GitHub.

```csharp
private const string ClientId = "<client_guid>";
private const string Authority = "https://login.microsoftonline.com/contoso.com";
private readonly string[] scopes = new string[] { "user.read" };

static async Task<AuthenticationResult> GetATokenForGraph()
{
    IPublicClientApplication pca = PublicClientApplicationBuilder
            .Create(ClientId)
            .WithAuthority(Authority)
            .WithDefaultRedirectUri()
            .Build();

    var accounts = await pca.GetAccountsAsync();

    // All AcquireToken* methods store the tokens in the cache, so check the cache first
    try
    {
        return await pca.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
            .ExecuteAsync();
    }
    catch (MsalUiRequiredException ex)
    {
        // No token found in the cache or AAD insists that a form interactive auth is required (e.g. the tenant admin turned on MFA)
        // If you want to provide a more complex user experience, check out ex.Classification

        return await AcquireByDeviceCodeAsync(pca);
    }
}

private static async Task<AuthenticationResult> AcquireByDeviceCodeAsync(IPublicClientApplication pca)
{
    try
    {
        var result = await pca.AcquireTokenWithDeviceCode(scopes,
            deviceCodeResult =>
            {
                    // This will print the message on the console which tells the user where to go sign-in using
                    // a separate browser and the code to enter once they sign in.
                    // The AcquireTokenWithDeviceCode() method will poll the server after firing this
                    // device code callback to look for the successful login of the user via that browser.
                    // This background polling (whose interval and timeout data is also provided as fields in the
                    // deviceCodeCallback class) will occur until:
                    // * The user has successfully logged in via browser and entered the proper code
                    // * The timeout specified by the server for the lifetime of this code (typically ~15 minutes) has been reached
                    // * The developing application calls the Cancel() method on a CancellationToken sent into the method.
                    //   If this occurs, an OperationCanceledException will be thrown (see catch below for more details).
                    Console.WriteLine(deviceCodeResult.Message);
                return Task.FromResult(0);
            }).ExecuteAsync();

        Console.WriteLine(result.Account.Username);
        return result;
    }

    // TODO: handle or throw all these exceptions depending on your app
    catch (MsalServiceException ex)
    {
        // Kind of errors you could have (in ex.Message)

        // AADSTS50059: No tenant-identifying information found in either the request or implied by any provided credentials.
        // Mitigation: as explained in the message from Azure AD, the authoriy needs to be tenanted. you have probably created
        // your public client application with the following authorities:
        // https://login.microsoftonline.com/common or https://login.microsoftonline.com/organizations

        // AADSTS90133: Device Code flow is not supported under /common or /consumers endpoint.
        // Mitigation: as explained in the message from Azure AD, the authority needs to be tenanted

        // AADSTS90002: Tenant <tenantId or domain you used in the authority> not found. This may happen if there are
        // no active subscriptions for the tenant. Check with your subscription administrator.
        // Mitigation: if you have an active subscription for the tenant this might be that you have a typo in the
        // tenantId (GUID) or tenant domain name.
    }
    catch (OperationCanceledException ex)
    {
        // If you use a CancellationToken, and call the Cancel() method on it, then this *may* be triggered
        // to indicate that the operation was cancelled.
        // See https://docs.microsoft.com/dotnet/standard/threading/cancellation-in-managed-threads
        // for more detailed information on how C# supports cancellation in managed threads.
    }
    catch (MsalClientException ex)
    {
        // Possible cause - verification code expired before contacting the server
        // This exception will occur if the user does not manage to sign-in before a time out (15 mins) and the
        // call to `AcquireTokenWithDeviceCode` is not cancelled in between
    }
}
```

# <a name="java"></a>[Java](#tab/java)

Это фрагмент из [примеров разработки в MSAL Java](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```java
private static IAuthenticationResult acquireTokenDeviceCode() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            Consumer<DeviceCode> deviceCodeConsumer = (DeviceCode deviceCode) ->
                    System.out.println(deviceCode.message());

            DeviceCodeFlowParameters parameters =
                    DeviceCodeFlowParameters
                            .builder(SCOPE, deviceCodeConsumer)
                            .build();

            // Try to acquire a token via device code flow. If successful, you should see
            // the token and account information printed out to console, and the sample_cache.json
            // file should have been updated with the latest tokens.
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Это фрагмент из [примеров разработки в MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/dev/sample/).

```Python
# Create a preferably long-lived app instance which maintains a token cache.
app = msal.PublicClientApplication(
    config["client_id"], authority=config["authority"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )

# The pattern to acquire a token looks like this.
result = None

# Note: If your device-flow app does not have any interactive ability, you can
#   completely skip the following cache part. But here we demonstrate it anyway.
# We now check the cache to see if we have some end users signed in before.
accounts = app.get_accounts()
if accounts:
    logging.info("Account(s) exists in cache, probably with token too. Let's try.")
    print("Pick the account you want to use to proceed:")
    for a in accounts:
        print(a["username"])
    # Assuming the end user chose this one
    chosen = accounts[0]
    # Now let's try to find a token in cache for this account
    result = app.acquire_token_silent(config["scope"], account=chosen)

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")

    flow = app.initiate_device_flow(scopes=config["scope"])
    if "user_code" not in flow:
        raise ValueError(
            "Fail to create device flow. Err: %s" % json.dumps(flow, indent=4))

    print(flow["message"])
    sys.stdout.flush()  # Some terminal needs this to ensure the message is shown

    # Ideally you should wait here, in order to save some unnecessary polling
    # input("Press Enter after signing in from another device to proceed, CTRL+C to abort.")

    result = app.acquire_token_by_device_flow(flow)  # By default it will block
        # You can follow this instruction to shorten the block time
        #    https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.acquire_token_by_device_flow
        # or you may even turn off the blocking behavior,
        # and then keep calling acquire_token_by_device_flow(flow) in your own customized loop
```

# <a name="macos"></a>[macOS](#tab/macOS)

Этот поток не применяется к macOS.

---

## <a name="file-based-token-cache"></a>Кэш маркеров на основе файлов

В MSAL.NET по умолчанию предоставляется кэш маркеров в памяти.

### <a name="serialization-is-customizable-in-windows-desktop-apps-and-web-apps-or-web-apis"></a>Сериализация настраивается для классических приложений Windows, веб-приложений и веб-API

В случае .NET Framework и .NET Core, если не выполнить никаких дополнительных действий, кэш маркеров в памяти существует в течение всего времени работы приложения. Чтобы понять, почему сериализация не предоставлялась без настройки, следует помнить, что классические приложения MSAL .NET или приложения .NET Core могут быть консольными или предназначенными для Windows (то есть могут иметь доступ к файловой системе), *но также* веб-приложениями или веб-API. Эти веб-приложения и веб-API могут использовать некоторые определенные механизмы кэширования, такие как базы данных, распределенные кэши и кэши Redis. Чтобы приложение .NET или .NET Core могло использовать постоянный кэш маркеров, необходимо настроить сериализацию.

Для сериализации кэша маркеров используются следующие классы и интерфейсы.

- ``ITokenCache`` определяет события для подписки на запросы сериализации кэша маркеров, а также методы для сериализации и десериализации кэша в разных форматах (ADAL версии 3.0, MSAL 2.x и MSAL 3.x = ADAL версии 5.0).
- Обратный вызов ``TokenCacheCallback`` передается событиям, что позволяет обрабатывать операции сериализации. Они будут вызываться с аргументами типа ``TokenCacheNotificationArgs``.
- ``TokenCacheNotificationArgs`` предоставляет только ``ClientId`` приложения и ссылку на пользователя, для которого доступен маркер.

  ![Схема сериализации кэша маркеров](https://user-images.githubusercontent.com/13203188/56027172-d58d1480-5d15-11e9-8ada-c0292f1800b3.png)

> [!IMPORTANT]
> MSAL.NET создает кэши маркеров и предоставляет кэш `IToken` при вызове свойств `UserTokenCache` и `AppTokenCache` приложения. Вам не нужно самостоятельно реализовать этот интерфейс. При реализации пользовательской сериализации для кэша маркеров в вашу зону ответственности входит следующее:
>
> - Реагирование на события `BeforeAccess` и `AfterAccess` или их *асинхронные* версии. Делегат `BeforeAccess` отвечает за десериализацию кэша. Делегат `AfterAccess` отвечает за сериализацию кэша.
> - Часть этих событий сохраняют или загружают большие двоичные объекты, которые передаются через аргумент события в нужное хранилище.

Стратегии будут разными в зависимости от того, выполняется ли сериализация кэша маркеров для общедоступного клиентского приложения, например классического приложения, или конфиденциального клиентского приложения, например веб-приложения, веб-API, управляющей программы.

Начиная с версии MSAL v2.x у вас есть несколько вариантов. Выбор зависит от того, хотите ли вы сериализовать кэш только в формате MSAL.NET, который является единым форматом, общим для MSAL и различных платформ. Вы можете также поддерживать [устаревшую](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Token-cache-serialization) сериализацию кэша маркеров из ADAL версии 3.

Настройка сериализации кэша маркеров для совместного использования состояния единого входа в ADAL.NET 3.x, ADAL.NET 5.x и MSAL.NET описана в одном из разделов следующего примера: [active-directory-dotnet-v1-to-v2](https://github.com/Azure-Samples/active-directory-dotnet-v1-to-v2).

### <a name="simple-token-cache-serialization-msal-only"></a>Простая сериализация кэша маркеров (только для MSAL)

Ниже приведен пример основной реализации пользовательской сериализации кэша маркеров для классических приложений. В этом случае кэш маркера пользователя находится в файле в той же папке, что и приложение, или, если приложение является [упакованным классом для настольных приложений](/windows/msix/desktop/desktop-to-uwp-behind-the-scenes), в папке на пользователя в приложении. Полный код см. в следующем примере: [Active-Directory-DotNet-Desktop-мсграф-v2](https://github.com/Azure-Samples/active-directory-dotnet-desktop-msgraph-v2).

После сборки приложения вы можете включить сериализацию, вызвав ``TokenCacheHelper.EnableSerialization()`` и передав ему `UserTokenCache` приложения.

```csharp
app = PublicClientApplicationBuilder.Create(ClientId)
    .Build();
TokenCacheHelper.EnableSerialization(app.UserTokenCache);
```

Этот вспомогательный класс выглядит как следующий фрагмент кода:

```csharp
static class TokenCacheHelper
 {
  public static void EnableSerialization(ITokenCache tokenCache)
  {
   tokenCache.SetBeforeAccess(BeforeAccessNotification);
   tokenCache.SetAfterAccess(AfterAccessNotification);
   try
   {
    // For packaged desktop apps (MSIX packages) the executing assembly folder is read-only. 
    // In that case we need to use Windows.Storage.ApplicationData.Current.LocalCacheFolder.Path + "\msalcache.bin" 
    // which is a per-app read/write folder for packaged apps.
    // See https://docs.microsoft.com/windows/msix/desktop/desktop-to-uwp-behind-the-scenes
    CacheFilePath = System.IO.Path.Combine(Windows.Storage.ApplicationData.Current.LocalCacheFolder.Path, "msalcache.bin3");
   }
   catch (System.InvalidOperationException)
   {
    // Fall back for an un-packaged desktop app
    CacheFilePath = System.Reflection.Assembly.GetExecutingAssembly().Location + ".msalcache.bin";
   }
  }

  /// <summary>
  /// Path to the token cache
  /// </summary>
  public static string CacheFilePath { get; private set; }

  private static readonly object FileLock = new object();

  private static void BeforeAccessNotification(TokenCacheNotificationArgs args)
  {
   lock (FileLock)
   {
    args.TokenCache.DeserializeMsalV3(File.Exists(CacheFilePath)
            ? ProtectedData.Unprotect(File.ReadAllBytes(CacheFilePath),
                                      null,
                                      DataProtectionScope.CurrentUser)
            : null);
   }
  }

  private static void AfterAccessNotification(TokenCacheNotificationArgs args)
  {
   // if the access operation resulted in a cache update
   if (args.HasStateChanged)
   {
    lock (FileLock)
    {
     // reflect changesgs in the persistent store
     File.WriteAllBytes(CacheFilePath,
                         ProtectedData.Protect(args.TokenCache.SerializeMsalV3(),
                                                 null,
                                                 DataProtectionScope.CurrentUser)
                         );
    }
   }
  }
 }
```

Предварительную версию сериализатора для производственных сред с файлом кэша маркеров, предназначенную для общедоступных клиентских приложений (классических приложений под управлением Windows, Mac и Linux), вы можете получить в библиотеке открытого кода [Microsoft.Identity.Client.Extensions.Msal](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet/tree/master/src/Microsoft.Identity.Client.Extensions.Msal). Это решение можно включить в приложение из следующего пакета NuGet: [Microsoft.Identity.Client.Extensions.Msal](https://www.nuget.org/packages/Microsoft.Identity.Client.Extensions.Msal/).

> [!NOTE]
> Отказ от ответственности. Библиотека Microsoft.Identity.Client.Extensions.Msal является расширением для MSAL.NET. Классы в этих библиотеках могут в будущем быть перенесены в MSAL.NET без изменений или с критическими изменениями.

### <a name="dual-token-cache-serialization-msal-unified-cache--adal-v3"></a>Сериализация сдвоенного кэша маркеров (единого кэша для MSAL и ADAL версии 3)

Возможно, потребуется реализовать сериализацию кэша маркеров с единым форматом кэша. Этот формат является общим для ADAL.NET 4.x и MSAL.NET 2.x, а также других MSAL того же или предыдущего поколения на той же платформе. Изучите следующий код:

```csharp
string appLocation = Path.GetDirectoryName(Assembly.GetEntryAssembly().Location;
string cacheFolder = Path.GetFullPath(appLocation) + @"..\..\..\..");
string adalV3cacheFileName = Path.Combine(cacheFolder, "cacheAdalV3.bin");
string unifiedCacheFileName = Path.Combine(cacheFolder, "unifiedCache.bin");

IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
                                    .Build();
FilesBasedTokenCacheHelper.EnableSerialization(app.UserTokenCache,
                                               unifiedCacheFileName,
                                               adalV3cacheFileName);

```

На этот раз вспомогательный класс выглядит как следующим образом:

```csharp
using System;
using System.IO;
using System.Security.Cryptography;
using Microsoft.Identity.Client;

namespace CommonCacheMsalV3
{
 /// <summary>
 /// Simple persistent cache implementation of the dual cache serialization (ADAL V3 legacy
 /// and unified cache format) for a desktop applications (from MSAL 2.x)
 /// </summary>
 static class FilesBasedTokenCacheHelper
 {
  /// <summary>
  /// Get the user token cache
  /// </summary>
  /// <param name="adalV3CacheFileName">File name where the cache is serialized with the
  /// ADAL V3 token cache format. Can
  /// be <c>null</c> if you don't want to implement the legacy ADAL V3 token cache
  /// serialization in your MSAL 2.x+ application</param>
  /// <param name="unifiedCacheFileName">File name where the cache is serialized
  /// with the Unified cache format, common to
  /// ADAL V4 and MSAL V2 and above, and also across ADAL/MSAL on the same platform.
  ///  Should not be <c>null</c></param>
  /// <returns></returns>
  public static void EnableSerialization(ITokenCache cache, string unifiedCacheFileName, string adalV3CacheFileName)
  {
   UnifiedCacheFileName = unifiedCacheFileName;
   AdalV3CacheFileName = adalV3CacheFileName;

   cache.SetBeforeAccess(BeforeAccessNotification);
   cache.SetAfterAccess(AfterAccessNotification);
  }

  /// <summary>
  /// File path where the token cache is serialized with the unified cache format
  /// (ADAL.NET V4, MSAL.NET V3)
  /// </summary>
  public static string UnifiedCacheFileName { get; private set; }

  /// <summary>
  /// File path where the token cache is serialized with the legacy ADAL V3 format
  /// </summary>
  public static string AdalV3CacheFileName { get; private set; }

  private static readonly object FileLock = new object();

  public static void BeforeAccessNotification(TokenCacheNotificationArgs args)
  {
   lock (FileLock)
   {
    args.TokenCache.DeserializeAdalV3(ReadFromFileIfExists(AdalV3CacheFileName));
    try
    {
     args.TokenCache.DeserializeMsalV3(ReadFromFileIfExists(UnifiedCacheFileName));
    }
    catch(Exception ex)
    {
     // Compatibility with the MSAL v2 cache if you used one
     args.TokenCache.DeserializeMsalV2(ReadFromFileIfExists(UnifiedCacheFileName));
    }
   }
  }

  public static void AfterAccessNotification(TokenCacheNotificationArgs args)
  {
   // if the access operation resulted in a cache update
   if (args.HasStateChanged)
   {
    lock (FileLock)
    {
     WriteToFileIfNotNull(UnifiedCacheFileName, args.TokenCache.SerializeMsalV3());
     if (!string.IsNullOrWhiteSpace(AdalV3CacheFileName))
     {
      WriteToFileIfNotNull(AdalV3CacheFileName, args.TokenCache.SerializeAdalV3());
     }
    }
   }
  }

  /// <summary>
  /// Read the content of a file if it exists
  /// </summary>
  /// <param name="path">File path</param>
  /// <returns>Content of the file (in bytes)</returns>
  private static byte[] ReadFromFileIfExists(string path)
  {
   byte[] protectedBytes = (!string.IsNullOrEmpty(path) && File.Exists(path))
       ? File.ReadAllBytes(path) : null;
   byte[] unprotectedBytes = encrypt ?
       ((protectedBytes != null) ? ProtectedData.Unprotect(protectedBytes, null, DataProtectionScope.CurrentUser) : null)
       : protectedBytes;
   return unprotectedBytes;
  }

  /// <summary>
  /// Writes a blob of bytes to a file. If the blob is <c>null</c>, deletes the file
  /// </summary>
  /// <param name="path">path to the file to write</param>
  /// <param name="blob">Blob of bytes to write</param>
  private static void WriteToFileIfNotNull(string path, byte[] blob)
  {
   if (blob != null)
   {
    byte[] protectedBytes = encrypt
      ? ProtectedData.Protect(blob, null, DataProtectionScope.CurrentUser)
      : blob;
    File.WriteAllBytes(path, protectedBytes);
   }
   else
   {
    File.Delete(path);
   }
  }

  // Change if you want to test with an un-encrypted blob (this is a json format)
  private static bool encrypt = true;
 }
}
```

## <a name="advanced-accessing-the-users-cached-tokens-in-background-apps-and-services"></a>Продвинут Доступ к кэшированным маркерам пользователя в фоновых приложениях и службах

[!INCLUDE [advanced-token-caching](../../../includes/advanced-token-cache.md)]

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [вызовите веб-API из классического приложения](scenario-desktop-call-api.md).