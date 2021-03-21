---
title: Получение маркера для вызова веб-API (мобильные приложения) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как создать мобильное приложение, вызывающее веб-API. (Получите маркер для приложения.)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: 6e6eda3d711710ea7450165ab02d7a260067bfcb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99582559"
---
# <a name="get-a-token-for-a-mobile-app-that-calls-web-apis"></a>Получение маркера для мобильного приложения, вызывающего веб-API

Прежде чем приложение сможет вызывать защищенные веб-API, ему нужен маркер доступа. В этой статье описывается процесс получения маркера с помощью библиотеки проверки подлинности Майкрософт (MSAL).

## <a name="define-a-scope"></a>Определение области

При запросе маркера определите область. Область определяет, к каким данным имеет доступ приложение.

Самый простой способ определить область — объединить нужный веб-API `App ID URI` с областью `.default` . Это определение указывает платформе удостоверений Майкрософт, что приложению требуются все области, заданные на портале.

### <a name="android"></a>Android
```Java
String[] SCOPES = {"https://graph.microsoft.com/.default"};
```

### <a name="ios"></a>iOS
```swift
let scopes = ["https://graph.microsoft.com/.default"]
```

### <a name="xamarin"></a>Xamarin
```csharp
var scopes = new [] {"https://graph.microsoft.com/.default"};
```

## <a name="get-tokens"></a>Получить токены

### <a name="acquire-tokens-via-msal"></a>Получение маркеров через MSAL

MSAL позволяет приложениям получать маркеры автоматически и интерактивно. При вызове метода `AcquireTokenSilent()` или `AcquireTokenInteractive()` MSAL возвращает маркер доступа для запрошенных областей. Правильный шаблон заключается в том, чтобы выполнить автоматический запрос, а затем вернуться к интерактивному запросу.

#### <a name="android"></a>Android

```Java
String[] SCOPES = {"https://graph.microsoft.com/.default"};
PublicClientApplication sampleApp = new PublicClientApplication(
                    this.getApplicationContext(),
                    R.raw.auth_config);

// Check if there are any accounts we can sign in silently.
// Result is in the silent callback (success or error).
sampleApp.getAccounts(new PublicClientApplication.AccountsLoadedCallback() {
    @Override
    public void onAccountsLoaded(final List<IAccount> accounts) {

        if (accounts.isEmpty() && accounts.size() == 1) {
            // TODO: Create a silent callback to catch successful or failed request.
            sampleApp.acquireTokenSilentAsync(SCOPES, accounts.get(0), getAuthSilentCallback());
        } else {
            /* No accounts or > 1 account. */
        }
    }
});

[...]

// No accounts found. Interactively request a token.
// TODO: Create an interactive callback to catch successful or failed requests.
sampleApp.acquireToken(getActivity(), SCOPES, getAuthInteractiveCallback());
```

#### <a name="ios"></a>iOS

Сначала попытайтесь получить маркер автоматически:

```objc

NSArray *scopes = @[@"https://graph.microsoft.com/.default"];
NSString *accountIdentifier = @"my.account.id";

MSALAccount *account = [application accountForIdentifier:accountIdentifier error:nil];

MSALSilentTokenParameters *silentParams = [[MSALSilentTokenParameters alloc] initWithScopes:scopes account:account];
[application acquireTokenSilentWithParameters:silentParams completionBlock:^(MSALResult *result, NSError *error) {

    if (!error)
    {
        // You'll want to get the account identifier to retrieve and reuse the account
        // for later acquireToken calls
        NSString *accountIdentifier = result.account.identifier;

        // Access token to call the web API
        NSString *accessToken = result.accessToken;
    }

    // Check the error
    if (error && [error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
    {
        // Interactive auth will be required, call acquireTokenWithParameters:error:
        return;
    }
}];
```

```swift

let scopes = ["https://graph.microsoft.com/.default"]
let accountIdentifier = "my.account.id"

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

    // You'll want to get the account identifier to retrieve and reuse the account
    // for later acquireToken calls
    let accountIdentifier = authResult.account.identifier

    // Access token to call the web API
    let accessToken = authResult.accessToken
}
```

Если MSAL возвращает `MSALErrorInteractionRequired` , то попробуйте получить маркеры в интерактивном режиме:

```objc
UIViewController *viewController = ...; // Pass a reference to the view controller that should be used when getting a token interactively
MSALWebviewParameters *webParameters = [[MSALWebviewParameters alloc] initWithAuthPresentationViewController:viewController];
MSALInteractiveTokenParameters *interactiveParams = [[MSALInteractiveTokenParameters alloc] initWithScopes:scopes webviewParameters:webParameters];
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

```swift
let viewController = ... // Pass a reference to the view controller that should be used when getting a token interactively
let webviewParameters = MSALWebviewParameters(authPresentationViewController: viewController)
let interactiveParameters = MSALInteractiveTokenParameters(scopes: scopes, webviewParameters: webviewParameters)
application.acquireToken(with: interactiveParameters, completionBlock: { (result, error) in

    guard let authResult = result, error == nil else {
        print(error!.localizedDescription)
        return
    }

    // Get access token from result
    let accessToken = authResult.accessToken
})
```

MSAL для iOS и macOS поддерживает различные модификаторы для интерактивного или автоматического получения маркера:
* [Общие параметры для получения маркера](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALTokenParameters.html#/Configuration%20parameters)
* [Параметры для получения интерактивного токена](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALInteractiveTokenParameters.html#/Configuring%20MSALInteractiveTokenParameters)
* [Параметры для получения автоматического токена](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALSilentTokenParameters.html)

#### <a name="xamarin"></a>Xamarin

В следующем примере показан минимальный код для интерактивного получения маркера. В примере для чтения профиля пользователя используется Microsoft Graph.

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

#### <a name="mandatory-parameters-in-msalnet"></a>Обязательные параметры в MSAL.NET

`AcquireTokenInteractive` имеет только один обязательный параметр: `scopes` . `scopes`Параметр перечисляет строки, определяющие области, для которых требуется токен. Если маркер предназначен для Microsoft Graph, необходимые области можно найти в справочнике по API для каждого Microsoft Graph API. В справочной документации перейдите к разделу "разрешения".

Например, чтобы получить [список контактов пользователя](/graph/api/user-list-contacts), используйте область "пользователь. чтение", "Контакты. чтение". См. сведения о [разрешениях Microsoft Graph](/graph/permissions-reference).

В Android можно указать родительские действия при создании приложения с помощью `PublicClientApplicationBuilder` . Если вы не укажете родительские действия в это время, позже его можно будет указать с помощью команды `.WithParentActivityOrWindow` , как в следующем разделе. При указании родительского действия маркер возвращается к родительскому действию после взаимодействия. Если он не указан, `.ExecuteAsync()` вызов создает исключение.

#### <a name="specific-optional-parameters-in-msalnet"></a>Определенные необязательные параметры в MSAL.NET

В следующих разделах описываются необязательные параметры в MSAL.NET.

##### <a name="withprompt"></a>WithPrompt

`WithPrompt()`Параметр управляет взаимодействием с пользователем, указывая запрос.

![Изображение, показывающее поля в структуре запроса. Эти постоянные значения управляют взаимодействием с пользователем, определяя тип запроса, отображаемого параметром Виспромпт ().](https://user-images.githubusercontent.com/13203188/53438042-3fb85700-39ff-11e9-9a9e-1ff9874197b3.png)

Класс определяет следующие константы:

- `SelectAccount` заставляет службу маркеров безопасности (STS) представлять диалоговое окно выбора учетной записи. Диалоговое окно содержит учетные записи, для которых у пользователя есть сеанс. Этот параметр можно использовать, если вы хотите разрешить пользователю выбирать между разными удостоверениями. Этот параметр позволяет MSAL отправлять `prompt=select_account` поставщику удостоверений.

    `SelectAccount`Константа является значением по умолчанию, и она эффективно предоставляет лучшие возможности на основе доступных данных. Доступные сведения могут включать учетную запись, присутствие сеанса для пользователя и т. д. Не изменяйте это значение по умолчанию, если у вас нет веских причин для этого.
- `Consent` позволяет запрашивать согласие пользователя, даже если было предоставлено согласие. В этом случае MSAL отправляет `prompt=consent` поставщику удостоверений.

    Эту `Consent` константу можно использовать в приложениях, ориентированных на безопасность, когда управление Организацией требует, чтобы пользователи могли видеть диалоговое окно согласия при каждом использовании приложения.
- `ForceLogin` позволяет службе предлагать пользователю ввести учетные данные, даже если запрос не требуется.

    Этот параметр может быть полезен в случае сбоя при получении маркера и необходимости повторного входа пользователя. В этом случае MSAL отправляет `prompt=login` поставщику удостоверений. Этот параметр может потребоваться использовать в приложениях, ориентированных на безопасность, где управление Организацией требует от пользователя входа при каждом доступе к конкретным частям приложения.
- `Never` предназначен только для .NET 4,5 и среда выполнения Windows (WinRT). Эта константа не будет запрашивать пользователя, но будет пытаться использовать файл cookie, хранящийся в скрытом внедренном веб-представлении. Дополнительные сведения см. [в разделе Использование веб-браузеров с MSAL.NET](./msal-net-web-browsers.md).

    При сбое этого параметра `AcquireTokenInteractive` выдается исключение, уведомляющее о необходимости взаимодействия с ИП. Затем используйте другой `Prompt` параметр.
- `NoPrompt` не отправляет запрос поставщику удостоверений.

    Этот параметр полезен только для политик редактирования профилей в Azure Active Directory B2C. Дополнительные сведения см. в разделе [особенности B2C](https://aka.ms/msal-net-b2c-specificities).

##### <a name="withextrascopetoconsent"></a>WithExtraScopeToConsent

Используйте `WithExtraScopeToConsent` Модификатор в расширенном сценарии, в котором пользователь должен предоставить предварительное согласие на несколько ресурсов. Этот модификатор можно использовать, если вы не хотите использовать добавочное согласие, которое обычно используется с MSAL.NET или платформой идентификации Майкрософт. Дополнительные сведения см. в разделе [Получение согласия пользователя на несколько ресурсов заранее](scenario-desktop-production.md#have-the-user-consent-upfront-for-several-resources).

Приведем пример кода:

```csharp
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

##### <a name="other-optional-parameters"></a>Другие необязательные параметры

Дополнительные сведения о других необязательных параметрах для `AcquireTokenInteractive` см. в [справочной документации по аккуиретокенинтерактивепараметербуилдер](/dotnet/api/microsoft.identity.client.acquiretokeninteractiveparameterbuilder#methods).

### <a name="acquire-tokens-via-the-protocol"></a>Получение маркеров через протокол

Мы не рекомендуем напрямую использовать протокол для получения маркеров. В этом случае приложение не будет поддерживать некоторые сценарии, использующие единый вход (SSO), Управление устройствами и условный доступ.

При использовании протокола для получения маркеров для мобильных приложений выполните два запроса:

* Получите код авторизации.
* Обмен кодом для маркера.

#### <a name="get-an-authorization-code"></a>Получение кода авторизации

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=<CLIENT_ID>
&response_type=code
&redirect_uri=<ENCODED_REDIRECT_URI>
&response_mode=query
&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2F.default
&state=12345
```

#### <a name="get-access-and-refresh-the-token"></a>Получение доступа и обновление токена

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=<CLIENT_ID>
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=<ENCODED_REDIRECT_URI>
&grant_type=authorization_code
```

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [вызвав веб-API](scenario-mobile-call-api.md).