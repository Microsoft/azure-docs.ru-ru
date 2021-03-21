---
title: Обработка ошибок и исключений в MSAL для iOS или macOS
titleSuffix: Microsoft identity platform
description: Сведения об обработке ошибок и исключений, проблемах условного доступа и повторных попыток в MSAL для приложений iOS и macOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/26/2020
ms.author: marsma
ms.reviewer: saeeda, oldalton
ms.custom: aaddev
ms.openlocfilehash: f7f2abadb7586e1b19168eb46a54bad53caa05cc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98761102"
---
# <a name="handle-errors-and-exceptions-in-msal-for-iosmacos"></a>Обработка ошибок и исключений в MSAL для iOS или macOS

[!INCLUDE [Active directory error handling introduction](../../../includes/active-directory-develop-error-handling-introduction.md)]

## <a name="error-handling-in-msal-for-iosmacos"></a>Обработка ошибок в MSAL для iOS/macOS

Полный список MSAL для ошибок iOS и macOS приведен в [перечислении MSALError](https://github.com/AzureAD/microsoft-authentication-library-for-objc/blob/master/MSAL/src/public/MSALError.h#L128).

Все ошибки, вызванные MSAL, возвращаются с доменом `MSALErrorDomain`.

Для системных ошибок MSAL возвращает исходное значение `NSError` из системного интерфейса API. Например, если получение токена завершается сбоем из-за отсутствия сетевого подключения, MSAL возвращает ошибку с доменом `NSURLErrorDomain` и кодом `NSURLErrorNotConnectedToInternet`.

На стороне клиента рекомендуется выполнять обработку по крайней мере двух следующих ошибок MSAL.

- `MSALErrorInteractionRequired`: пользователь должен выполнить интерактивный запрос. Причин такой ошибки может быть много, например истечение срока действия сеанса проверки подлинности или необходимость выполнить дополнительные требования для ее проведения. Для исправления ошибки вызовите интерактивный API получения токена MSAL. 

- `MSALErrorServerDeclinedScopes`: доступ для некоторых или всех областей отклонен. Решите, следует ли использовать только области, для которых предоставлен доступ, или же остановить процесс входа.

> [!NOTE]
> Перечисление `MSALInternalError` следует использовать только для справки и отладки. Не пытайтесь автоматически обрабатывать эти ошибки во время выполнения. Если в приложении происходят какие-либо ошибки из-за перечисления `MSALInternalError`, возможно, потребуется отобразить для пользователя общее сообщение, объясняющее, что произошло.

Например, `MSALInternalErrorBrokerResponseNotReceived` означает, что пользователь не прошел проверку подлинности и вручную возвращался в приложение. В этом случае приложение должно показывать общее сообщение об ошибке, в котором объясняется, что проверка подлинности не пройдена, и предлагается ее повторить.

В следующем образце кода цели-C демонстрируются рекомендации по обработке некоторых распространенных условий возникновения ошибок.

```objc
    MSALInteractiveTokenParameters *interactiveParameters = ...;
    MSALSilentTokenParameters *silentParameters = ...;
    
    MSALCompletionBlock completionBlock;
    __block __weak MSALCompletionBlock weakCompletionBlock;
    
    weakCompletionBlock = completionBlock = ^(MSALResult *result, NSError *error)
    {
        if (!error)
        {
            // Use result.accessToken
            NSString *accessToken = result.accessToken;
            return;
        }
        
        if ([error.domain isEqualToString:MSALErrorDomain])
        {
            switch (error.code)
            {
                case MSALErrorInteractionRequired:
                {
                    // Interactive auth will be required
                    [application acquireTokenWithParameters:interactiveParameters
                                            completionBlock:weakCompletionBlock];
                    
                    break;
                }
                    
                case MSALErrorServerDeclinedScopes:
                {
                    // These are list of granted and declined scopes.
                    NSArray *grantedScopes = error.userInfo[MSALGrantedScopesKey];
                    NSArray *declinedScopes = error.userInfo[MSALDeclinedScopesKey];
                    
                    // To continue acquiring token for granted scopes only, do the following
                    silentParameters.scopes = grantedScopes;
                    [application acquireTokenSilentWithParameters:silentParameters
                                                  completionBlock:weakCompletionBlock];
                    
                    // Otherwise, instead, handle error fittingly to the application context
                    break;
                }
                    
                case MSALErrorServerProtectionPoliciesRequired:
                {
                    // Integrate the Intune SDK and call the
                    // remediateComplianceForIdentity:silent: API.
                    // Handle this error only if you integrated Intune SDK.
                    // See more info here: https://aka.ms/intuneMAMSDK
                    
                    break;
                }
                    
                case MSALErrorUserCanceled:
                {
                    // The user cancelled the web auth session.
                    // You may want to ask the user to try again.
                    // Handling of this error is optional.
                    
                    break;
                }
                    
                case MSALErrorInternal:
                {
                    // Log the error, then inspect the MSALInternalErrorCodeKey
                    // in the userInfo dictionary.
                    // Display generic error message to the end user
                    // More detailed information about the specific error
                    // under MSALInternalErrorCodeKey can be found in MSALInternalError enum.
                    NSLog(@"Failed with error %@", error);
                    
                    break;
                }
                    
                default:
                    NSLog(@"Failed with unknown MSAL error %@", error);
                    
                    break;
            }
            
            return;
        }
        
        // Handle no internet connection.
        if ([error.domain isEqualToString:NSURLErrorDomain] && error.code == NSURLErrorNotConnectedToInternet)
        {
            NSLog(@"No internet connection.");
            return;
        }
        
        // Other errors may require trying again later,
        // or reporting authentication problems to the user.
        NSLog(@"Failed with error %@", error);
    };
    
    // Acquire token silently
    [application acquireTokenSilentWithParameters:silentParameters
                                  completionBlock:completionBlock];

     // or acquire it interactively.
     [application acquireTokenWithParameters:interactiveParameters
                             completionBlock:completionBlock];
```

```swift
    let interactiveParameters: MSALInteractiveTokenParameters = ...
    let silentParameters: MSALSilentTokenParameters = ...
            
    var completionBlock: MSALCompletionBlock!
    completionBlock = { (result: MSALResult?, error: Error?) in
                
        if let result = result
        {
            // Use result.accessToken
            let accessToken = result.accessToken
            return
        }

        guard let error = error as NSError? else { return }

        if error.domain == MSALErrorDomain, let errorCode = MSALError(rawValue: error.code)
        {
            switch errorCode
            {
                case .interactionRequired:
                    // Interactive auth will be required
                    application.acquireToken(with: interactiveParameters, completionBlock: completionBlock)

                case .serverDeclinedScopes:
                    let grantedScopes = error.userInfo[MSALGrantedScopesKey]
                    let declinedScopes = error.userInfo[MSALDeclinedScopesKey]

                    if let scopes = grantedScopes as? [String] {
                        silentParameters.scopes = scopes
                        application.acquireTokenSilent(with: silentParameters, completionBlock: completionBlock)
                    }
                        
                    case .serverProtectionPoliciesRequired:
                        // Integrate the Intune SDK and call the
                        // remediateComplianceForIdentity:silent: API.
                        // Handle this error only if you integrated Intune SDK.
                        // See more info here: https://aka.ms/intuneMAMSDK
                        break
                        
                    case .userCanceled:
                       // The user cancelled the web auth session.
                       // You may want to ask the user to try again.
                       // Handling of this error is optional.
                       break
                        
                    case .internal:
                        // Log the error, then inspect the MSALInternalErrorCodeKey
                        // in the userInfo dictionary.
                        // Display generic error message to the end user
                        // More detailed information about the specific error
                        // under MSALInternalErrorCodeKey can be found in MSALInternalError enum.
                        print("Failed with error \(error)");
                        
                    default:
                        print("Failed with unknown MSAL error \(error)")
            }
        }
                
        // Handle no internet connection.
        if error.domain == NSURLErrorDomain && error.code == NSURLErrorNotConnectedToInternet
        {
            print("No internet connection.")
            return
        }
                
        // Other errors may require trying again later,
        // or reporting authentication problems to the user.
        print("Failed with error \(error)");    
    }
   
    // Acquire token silently
    application.acquireToken(with: interactiveParameters, completionBlock: completionBlock)
 
    // or acquire it interactively.
    application.acquireTokenSilent(with: silentParameters, completionBlock: completionBlock)
```

---

[!INCLUDE [Active directory error handling claims challenges](../../../includes/active-directory-develop-error-handling-claims-challenges.md)]

MSAL для iOS и macOS позволяет запрашивать определенные утверждения как в интерактивном, так и в автоматическом сценарии получения токенов.

Чтобы запросить пользовательские утверждения, укажите `claimsRequest` в `MSALSilentTokenParameters` или `MSALInteractiveTokenParameters`.

Дополнительные сведения см. в статье [Запрос пользовательских утверждений с помощью MSAL для iOS и macOS](request-custom-claims.md).

[!INCLUDE [Active directory error handling retries](../../../includes/active-directory-develop-error-handling-retries.md)]

## <a name="next-steps"></a>Дальнейшие действия

Рекомендуется включить [ведение журнала в MSAL для iOS/macOS](msal-logging-ios.md) , чтобы помочь в диагностике и отладке проблем.
