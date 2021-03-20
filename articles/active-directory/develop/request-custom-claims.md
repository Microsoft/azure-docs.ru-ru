---
title: Запрос настраиваемых утверждений (MSAL iOS/macOS) | Службы
titleSuffix: Microsoft identity platform
description: Узнайте, как запросить настраиваемые утверждения.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 08/26/2019
ms.author: marsma
ms.custom: aaddev
ms.openlocfilehash: a570dccad5f14cf9adf5ca2825d8a3b31ae60d3f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85477198"
---
# <a name="how-to-request-custom-claims-using-msal-for-ios-and-macos"></a>Как запросить настраиваемые утверждения с помощью MSAL для iOS и macOS

OpenID Connect Connect позволяет при необходимости запросить возврат индивидуальных утверждений из конечной точки UserInfo и/или в маркере идентификации. Запрос утверждений представлен в виде объекта JSON, содержащего список запрошенных утверждений. Дополнительные сведения см. в разделе [OpenID Connect Connect Core 1,0](https://openid.net/specs/openid-connect-core-1_0-final.html#ClaimsParameter) .

Библиотека проверки подлинности Microsoft (MSAL) для iOS и macOS позволяет запрашивать определенные утверждения как в интерактивном, так и в автоматическом режиме получения маркеров. Это делается с помощью `claimsRequest` параметра.

Существует несколько сценариев, в которых это необходимо. Пример:

- Запрос утверждений за пределами стандартного набора для вашего приложения.
- Запрос определенных сочетаний стандартных заявок, которые не могут быть указаны с помощью областей для приложения. Например, если маркер доступа отклоняется из-за отсутствия утверждений, приложение может запросить недостающие утверждения с помощью MSAL.

> [!NOTE]
> MSAL обходит кэш маркера доступа всякий раз, когда указан запрос на утверждение. Важно предоставлять `claimsRequest` параметр только в том случае, если требуются дополнительные утверждения (в отличие от того, что всегда предоставляет один и тот же `claimsRequest` параметр в каждом ВЫЗОВЕ API MSAL).

`claimsRequest` можно указать в `MSALSilentTokenParameters` и `MSALInteractiveTokenParameters` :

```objc
/*!
 MSALTokenParameters is the base abstract class for all types of token parameters (silent and interactive).
 */
@interface MSALTokenParameters : NSObject

/*!
 The claims parameter that needs to be sent to authorization or token endpoint.
 If claims parameter is passed in silent flow, access token will be skipped and refresh token will be tried.
 */
@property (nonatomic, nullable) MSALClaimsRequest *claimsRequest;

@end
```
`MSALClaimsRequest` можно сформировать из NSString представления запроса на утверждение JSON. 

Objective-C.

```objc
NSError *claimsError = nil;
MSALClaimsRequest *request = [[MSALClaimsRequest alloc] initWithJsonString:@"{\"id_token\":{\"auth_time\":{\"essential\":true},\"acr\":{\"values\":[\"urn:mace:incommon:iap:silver\"]}}}" error:&claimsError];
```

Swift:

```swift
var requestError: NSError? = nil
let request = MSALClaimsRequest(jsonString: "{\"id_token\":{\"auth_time\":{\"essential\":true},\"acr\":{\"values\":[\"urn:mace:incommon:iap:silver\"]}}}",
                                        error: &requestError)
```



Его также можно изменить, запросив дополнительные конкретные утверждения:

Objective-C.

```objc
MSALIndividualClaimRequest *individualClaimRequest = [[MSALIndividualClaimRequest alloc] initWithName:@"custom_claim"];
individualClaimRequest.additionalInfo = [MSALIndividualClaimRequestAdditionalInfo new];
individualClaimRequest.additionalInfo.essential = @1;
individualClaimRequest.additionalInfo.value = @"myvalue";
[request requestClaim:individualClaimRequest forTarget:MSALClaimsRequestTargetIdToken error:&claimsError];
```

Swift:

```swift
let individualClaimRequest = MSALIndividualClaimRequest(name: "custom-claim")
let additionalInfo = MSALIndividualClaimRequestAdditionalInfo()
additionalInfo.essential = 1
additionalInfo.value = "myvalue"
individualClaimRequest.additionalInfo = additionalInfo
do {
  try request.requestClaim(individualClaimRequest, for: .idToken)
} catch let error as NSError {
  // handle error here  
}
        
```



`MSALClaimsRequest` следует задать в параметрах токена и предоставить одному из API-интерфейсов получения маркера MSAL:

Objective-C.

```objc
MSALPublicClientApplication *application = ...;
MSALWebviewParameters *webParameters = ...;

MSALInteractiveTokenParameters *parameters = [[MSALInteractiveTokenParameters alloc] initWithScopes:@[@"user.read"]
                                                                                  webviewParameters:webParameters];
parameters.claimsRequest = request;
    
[application acquireTokenWithParameters:parameters completionBlock:completionBlock];
```

Swift:

```swift
let application: MSALPublicClientApplication!
let webParameters: MSALWebviewParameters!
        
let parameters = MSALInteractiveTokenParameters(scopes: ["user.read"], webviewParameters: webParameters)
parameters.claimsRequest = request
        
application.acquireToken(with: parameters) { (result: MSALResult?, error: Error?) in            ...

```



## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в [Потоки проверки подлинности и сценарии приложений](authentication-flows-app-scenarios.md)
