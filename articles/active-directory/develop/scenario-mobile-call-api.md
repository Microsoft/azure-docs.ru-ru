---
title: Вызов веб-API из мобильного приложения | Azure
titleSuffix: Microsoft identity platform
description: Узнайте, как создать мобильное приложение, вызывающее веб-API. (Вызовите веб-API.)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/18/2020
ms.author: jmprieur
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: 656cb167fb47e44c28922afed75d8c46a460aaf1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99584403"
---
# <a name="call-a-web-api-from-a-mobile-app"></a>Вызов веб-API из мобильного приложения

После того как приложение подпишет пользователя и получит маркеры, Библиотека проверки подлинности Майкрософт (MSAL) предоставляет сведения о пользователе, среде пользователя и выданных маркерах. Эти значения можно использовать для вызова веб-API или отображения приветственного сообщения для пользователя.

В этой статье мы сначала рассмотрим результат, полученный MSAL. Затем мы рассмотрим, как использовать маркер доступа из `AuthenticationResult` или `result` для вызова защищенного веб-API.

## <a name="msal-result"></a>Результат, полученный MSAL
MSAL предоставляет следующие значения: 

- `AccessToken` вызывает защищенные веб-API в запросе носителя HTTP.
- `IdToken` содержит полезную информацию об авторизованном пользователе. Эта информация включает имя пользователя, домашнего клиента и уникальный идентификатор хранилища.
- `ExpiresOn` — это время истечения срока действия маркера. MSAL обрабатывает автоматическое обновление приложения.
- `TenantId` — это идентификатор клиента, где выполнил вход пользователь. Для гостевых пользователей в B2B Azure Active Directory (Azure AD) это значение обозначает клиента, где выполнил вход пользователь. Это значение не идентифицирует домашний клиент пользователя.  
- `Scopes` обозначает области, предоставленные вместе с вашим маркером. Предоставленные области могут представлять собой подмножество запрошенных вами областей.

MSAL также предоставляет абстракцию значения `Account`. Значение `Account` представляет учетную запись, с которой выполнил вход текущий пользователь:

- `HomeAccountIdentifier` обозначает домашний клиент пользователя.
- `UserName` — это предпочитаемое пользователем имя пользователя. Для пользователей B2C Azure AD это значение может быть пустым.
- `AccountIdentifier` обозначает авторизованного пользователя. В большинстве случаев это значение совпадает со значением `HomeAccountIdentifier` (кроме случаев, когда пользователь является гостем в другом клиенте).

## <a name="call-an-api"></a>Вызов API

Получив маркер доступа, можно вызывать веб-API. Ваше приложение будет использовать этот токен для создания HTTP-запроса с последующим выполнением этого запроса.

### <a name="android"></a>Android

```Java
        RequestQueue queue = Volley.newRequestQueue(this);
        JSONObject parameters = new JSONObject();

        try {
            parameters.put("key", "value");
        } catch (Exception e) {
            // Error when constructing.
        }
        JsonObjectRequest request = new JsonObjectRequest(Request.Method.GET, MSGRAPH_URL,
                parameters,new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                // Successfully called Graph. Process data and send to UI.
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                // Error.
            }
        }) {
            @Override
            public Map<String, String> getHeaders() throws AuthFailureError {
                Map<String, String> headers = new HashMap<>();
                
                // Put access token in HTTP request.
                headers.put("Authorization", "Bearer " + authResult.getAccessToken());
                return headers;
            }
        };

        request.setRetryPolicy(new DefaultRetryPolicy(
                3000,
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES,
                DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        queue.add(request);
```

### <a name="msal-for-ios-and-macos"></a>MSAL для iOS и MacOS

Методы получения маркеров возвращают объект `MSALResult`. `MSALResult` предоставляет свойство `accessToken`. Можно использовать `accessToken` для вызова веб-API. Добавьте это свойство в заголовок авторизации HTTP, прежде чем отправлять запрос на доступ к защищенному веб-API.

```objc
NSMutableURLRequest *urlRequest = [NSMutableURLRequest new];
urlRequest.URL = [NSURL URLWithString:"https://contoso.api.com"];
urlRequest.HTTPMethod = @"GET";
urlRequest.allHTTPHeaderFields = @{ @"Authorization" : [NSString stringWithFormat:@"Bearer %@", accessToken] };
        
NSURLSessionDataTask *task =
[[NSURLSession sharedSession] dataTaskWithRequest:urlRequest
     completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {}];
[task resume];
```

```swift
let urlRequest = NSMutableURLRequest()
urlRequest.url = URL(string: "https://contoso.api.com")!
urlRequest.httpMethod = "GET"
urlRequest.allHTTPHeaderFields = [ "Authorization" : "Bearer \(accessToken)" ]
     
let task = URLSession.shared.dataTask(with: urlRequest as URLRequest) { (data: Data?, response: URLResponse?, error: Error?) in }
task.resume()
```

### <a name="xamarin"></a>Xamarin

[!INCLUDE [Call web API in .NET](../../../includes/active-directory-develop-scenarios-call-apis-dotnet.md)]

## <a name="make-several-api-requests"></a>Выполнение нескольких запросов API

Чтобы вызвать один API несколько раз или вызвать несколько API-интерфейсов, при сборке приложения учитывайте следующие темы:

- **Добавочное согласие**. платформа Microsoft Identity позволяет приложениям получать согласие пользователя, когда требуются разрешения, а не все в начале. Всякий раз, когда ваше приложение готово вызвать API, оно должно запрашивать только необходимые ему области.

- **Условный доступ**. При отправке нескольких запросов API в некоторых сценариях вам может потребоваться выполнение дополнительных требований к условному доступу. Требования могут увеличиваться, если первый запрос не имеет политик условного доступа и ваше приложение попытается автоматически получить доступ к новому API, требующему условного доступа. Для решения этой проблемы необходимо перехватывать ошибки, генерируемые такими автоматическими запросами, и быть готовым к выполнению интерактивного запроса.  Для получения дополнительной информации см. раздел [Инструкции по осуществлению условного доступа](../azuread-dev/conditional-access-dev-guide.md).

## <a name="call-several-apis-by-using-incremental-consent-and-conditional-access"></a>Вызов нескольких API с использованием добавочного согласия и условного доступа

Чтобы вызвать несколько API для одного пользователя, после получения маркера для пользователя можно избежать неоднократного запроса учетных данных для `AcquireTokenSilent` получения маркера.

```csharp
var result = await app.AcquireTokenXX("scopeApi1")
                      .ExecuteAsync();

result = await app.AcquireTokenSilent("scopeApi2")
                  .ExecuteAsync();
```

Взаимодействие необходимо, если:

- Пользователь предоставил согласие для первого API, а теперь ему требуется согласие для других областей. В этом случае следует использовать добавочное согласие.
- Первый API не требует [многофакторной проверки подлинности](../authentication/concept-mfa-howitworks.md), в отличие от следующего API.

```csharp
var result = await app.AcquireTokenXX("scopeApi1")
                      .ExecuteAsync();

try
{
 result = await app.AcquireTokenSilent("scopeApi2")
                  .ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
 result = await app.AcquireTokenInteractive("scopeApi2")
                  .WithClaims(ex.Claims)
                  .ExecuteAsync();
}
```

## <a name="next-steps"></a>Дальнейшие действия

Перейдите к следующей статье в этом сценарии, [перейдите в рабочую среду](scenario-mobile-production.md).
