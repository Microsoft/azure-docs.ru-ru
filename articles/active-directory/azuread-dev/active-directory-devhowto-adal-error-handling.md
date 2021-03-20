---
title: Рекомендации по обработке ошибок клиентского приложения ADAL | Службы
description: Здесь содержатся инструкции по обработке ошибок и рекомендации для клиентских приложений ADAL.
services: active-directory
author: rwike77
manager: CelesteDG
ms.author: ryanwi
ms.service: active-directory
ms.subservice: azuread-dev
ms.custom: aaddev
ms.topic: how-to
ms.workload: identity
ms.date: 02/27/2017
ROBOTS: NOINDEX
ms.openlocfilehash: ad5595f7eebc8feca2f00a6f95e10c547ded9529
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85383740"
---
# <a name="error-handling-best-practices-for-azure-active-directory-authentication-library-adal-clients"></a>Руководство по обработке ошибок для клиентов библиотеки аутентификации Active Directory (ADAL)

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Эта статья содержит рекомендации о типах ошибок, которые могут встретить разработчики при использовании ADAL для аутентификации пользователей. При использовании ADAL есть несколько случаев, в которых разработчику потребуется вмешаться и обработать ошибки. Правильная обработка ошибок обеспечивает отличную работу пользователя и ограничивает количество попыток входа в систему.

В этой статье рассматриваются определенные случаи для каждой платформы, поддерживаемой ADAL, а также то, как приложение может обработать каждый случай должным образом. Руководство по ошибкам разбито на две более широкие категории, основанные на шаблонах получения токенов, предоставляемых API ADAL:

- **AcquireTokenSilent**. Клиент пытается получить токен автоматически (не используя пользовательский интерфейс). Эта операция может завершится сбоем, если ADAL завершится с ошибкой. 
- **AcquireToken**. Клиент может попытаться получить токен автоматически, однако также он может выполнять интерактивные запросы, для которых требуется выполнить вход.

> [!TIP]
> При использовании ADAL и Azure AD рекомендуется записывать в журнал все ошибки и исключения. Журналы не только полезны для понимания общей работоспособности приложения, но также важны при устранении более широких проблем. Хотя приложение может восстановиться после некоторых ошибок, они могут указать на более широкие проблемы проектирования, для решения которых требуется изменение кода. 
> 
> При реализации условий ошибки, описанных в этом документе, вы должны зарегистрировать код ошибки и описание по причинам, о которых уже было сказано ранее. Примеры кода ведения журнала см. в разделе [Справочник по ведению журнала и ошибкам](#error-and-logging-reference). 
>

## <a name="acquiretokensilent"></a>AcquireTokenSilent

AcquireTokenSilent предпринимает попытку получит токен с гарантией, что пользователь не видит пользовательский интерфейс. Есть несколько случаев, когда автоматическое получение токена может завершится со сбоем и потребовать обработки через интерактивные запросы или с помощью обработчиков по умолчанию. Особенности того, когда и как использовать каждый случай, рассматриваются в следующих разделах.

Для некоторых ошибок ОС может потребоваться обработка, характерная для приложения. Дополнительные сведения см. в подразделе "Ошибки операционной системы" раздела [Справочник по ведению журнала и ошибкам](#error-and-logging-reference). 

### <a name="application-scenarios"></a>Сценарии приложений

- [Собственные клиентские](../develop/developer-glossary.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json#native-client) приложения (iOS, Android, Настольный компьютер .NET или Xamarin)
- Приложения [веб-клиента](../develop/developer-glossary.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json#web-client), вызывающие [ресурсы](../develop/developer-glossary.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json#resource-server) (.NET).

### <a name="error-cases-and-actionable-steps"></a>Сценарии ошибок и практические действия

По сути есть два типа ошибок AcquireTokenSilent:

| Случай | Описание |
|------|-------------|
| **Тип 1.** Ошибку можно устранить путем интерактивного входа | Для ошибок, вызванных нехваткой допустимых токенов, требуется интерактивный запрос. В частности для поиска по кэшу и обновления недопустимого либо просроченного токена требуется разрешение вызова AcquireToken.<br><br>В таких случаях пользователю требуется отобразить запрос на вход. Приложение может сразу сделать интерактивный запрос, после взаимодействия с пользователем (например, нажатие кнопки входа) или позже. Выбор зависит от требуемого поведения приложения.<br><br>Дополнительные сведения о данном конкретном сценарии и ошибках, по которым его диагностируют, см. в коде в следующем разделе.|
| **Тип 2.** Ошибку невозможно устранить путем интерактивного входа | При сетевых и временных ошибках или других сбоях выполнения интерактивного запроса AcquireToken не разрешает проблему. Ненужные интерактивные запросы на вход могут вызвать трудности у пользователей. Для большинства ошибок при сбоях AcquireTokenSilent ADAL автоматически пытается выполнить одну попытку повтора.<br><br>Клиентское приложение также может повторить попытку позже, но когда и как зависит от поведения приложения и желаемого взаимодействия с конечным пользователем. Например, приложение может повторить попытку запроса AcquireTokenSilent через несколько минут или в ответ на некоторые действия пользователя. Немедленная повторная попытка приведет к регулированию приложения, поэтому предпринимать ее не следует.<br><br>Если последующая попытка повторения завершится с той же ошибкой, это значит, что клиенту не нужно выполнять интерактивный запрос с использованием AcquireToken, так как он не устраняет ошибку.<br><br>Дополнительные сведения о данном конкретном сценарии и ошибках, по которым его диагностируют, см. в коде в следующем разделе. |

### <a name="net"></a>.NET

Далее представлены примеры для обработки ошибок в сочетании с методами ADAL: 

- acquireTokenSilentAsync(…);
- acquireTokenSilentSync(…); 
- acquireTokenSilent(…) [не рекомендуется использовать].
- acquireTokenByRefreshToken(…) [не рекомендуется использовать]. 

Код будет реализован следующим образом:

```csharp
try{
    AcquireTokenSilentAsync(…);
} 

catch (AdalSilentTokenAcquisitionException e) {
    // Exception: AdalSilentTokenAcquisitionException
    // Caused when there are no tokens in the cache or a required refresh failed. 

    // Action: Case 1, resolvable with an interactive request. 
} 

catch(AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Action: Case 2, not resolvable with an interactive request.
    // Attempt retry after a timed interval or user action.
} 
    
catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET. 
    // e.ErrorCode contains the error code. 

    // Action: Case 2, not resolvable with an interactive request.
    // Attempt retry after a timed interval or user action.
    // Example Error: network_not_available, default case.
}
```

### <a name="android"></a>Android

Далее представлены примеры для обработки ошибок в сочетании с методами ADAL: 

- acquireTokenSilentSync(…);
- acquireTokenSilentAsync(...);
- acquireTokenSilent(…) [не рекомендуется использовать].

Код будет реализован следующим образом:

```java
// *Inside callback*
public void onError(Exception e) {

    if (e instanceof AuthenticationException) {
        // Exception: AdalException
        // Represents a library exception generated by ADAL Android.
        // Error Code: e.getCode().

        // Errors: ADALError.ERROR_SILENT_REQUEST,
        // ADALError.AUTH_REFRESH_FAILED_PROMPT_NOT_ALLOWED,
        // ADALError.INVALID_TOKEN_CACHE_ITEM
        // Description: Request failed due to no tokens in
        // cache or failed a required refresh. 

        // Action: Case 1, resolvable with an interactive request. 

        // Action: Case 2, not resolvable with an interactive request.
        // Attempt retry after a timed interval or user action.
        // Example Errors: default case,
        // DEVICE_CONNECTION_IS_NOT_AVAILABLE, 
        // BROKER_AUTHENTICATOR_ERROR_GETAUTHTOKEN,
    }
}
```

### <a name="ios"></a>iOS

Далее представлены примеры для обработки ошибок в сочетании с методами ADAL: 

- acquireTokenSilentWithResource(…)

Код будет реализован следующим образом:

```objc
[context acquireTokenSilentWithResource:[ARGS], completionBlock:^(ADAuthenticationResult *result) {
    if (result.status == AD_FAILED) {
        if ([error.domain isEqualToString:ADAuthenticationErrorDomain]){
            // Exception: AD_FAILED 
            // Represents a library error generated by ADAL Objective-C.
            // Error Code: result.error.code

            // Errors: AD_ERROR_SERVER_REFRESH_TOKEN_REJECTED, AD_ERROR_CACHE_NO_REFRESH_TOKEN
            // Description: No tokens in cache or failed a required token refresh failed. 
            // Action: Case 1, resolvable with an interactive request. 

            // Error: AD_ERROR_CACHE_MULTIPLE_USERS
            // Description: There was ambiguity in the silent request resulting in multiple cache items.
            // Action: Special Case, application should perform another silent request and specify the user using ADUserIdentifier. 
            // Can be caused in cases of a multi-user application. 

            // Action: Case 2, not resolvable with an interactive request.
            // Attempt retry after some time or user action.
            // Example Errors: default case,
            // AD_ERROR_CACHE_BAD_FORMAT
        }
    }
}]
```

## <a name="acquiretoken"></a>AcquireToken

AcquireToken является методом ADAL по умолчанию, который используется для получения токенов. В случаях, когда требуется удостоверение пользователя, сначала AcquireToken пытается получить токен автоматически, а затем отображает пользовательский интерфейс, если это необходимо (если не передано свойство PromptBehavior.Never). В случаях, когда требуется удостоверение приложения, AcquireToken пытается получить токен, но не отображает пользовательский интерфейс, так как пользователь отсутствует. 

Обработка ошибок AcquireToken зависит от платформы и сценария, который пытается выполнить приложение. 

В операционной системе могут также возникать ошибки, требующие обработки в зависимости от конкретного приложения. Дополнительные сведения см. в подразделе "Ошибки операционной системы" раздела [Справочник по ведению журнала и ошибкам](#error-and-logging-reference). 

### <a name="application-scenarios"></a>Сценарии приложений

- Приложения собственного клиента (iOS, Android, классические приложения .NET или Xamarin).
- Веб-приложение, вызывающие API ресурса (.NET).
- Одностраничные приложения (Javascript)
- Приложения для обмена данными между службами (.NET, Java).
  - Все сценарии, включая сценарии типа "от имени"
  - Конкретные сценарии типа "от имени"

### <a name="error-cases-and-actionable-steps-native-client-applications"></a>Сценарии ошибок и практические действия. Собственные клиентские приложения

При создании собственного клиентского приложения есть несколько типов обработки ошибок, связанных с сетевыми проблемами, временными сбоями и другими ошибками на платформе. В большинстве случаев приложение не должно выполнять немедленные повторы, а ожидает взаимодействия с пользователем, запрашивающего вход. 

В некоторых особых случаях одна попытка повтора может устранить проблему. Например, когда пользователю необходимо включить данные на устройстве или завершить скачивание брокера Azure AD после первоначального отказа. 

В случае сбоя приложение может предоставлять пользовательский интерфейс, чтобы позволить пользователю выполнить какое-либо действие, при котором запрашивается повторная попытка. Например, если в устройстве возникла автономная ошибка, появится кнопка "Попробуйте войти снова", предлагающая повторить попытку запроса AcquireToken, а не сразу же повторять попытку, которая вызвала ошибку. 

Обработку ошибок в собственных приложений можно разделить на два типа:

|  |  |
|------|-------------|
| **Вариант 1**.<br>Ошибка без возможности повторной попытки (в большинстве случаев) | 1. не пытайтесь немедленно повторить попытку. Предоставление пользовательского интерфейса для конечного пользователя на основе конкретной ошибки, вызывающей повторную попытку (например, "повторить попытку входа" или "скачать приложение Azure AD Broker"). |
| **Вариант 2**.<br>Ошибка с возможностью повтора | 1. Выполните однократную попытку, так как конечный пользователь мог перейти в состояние, которое приводит к успешному выполнению.<br><br>2. Если повторная попытка завершится неудачей, представим пользовательский интерфейс пользователя на основе конкретной ошибки, вызывающей повторную попытку ("повторить попытку входа", "скачать приложение брокера Azure AD" и т. д.). |

> [!IMPORTANT]
> Если учетная запись пользователя передается в ADAL при автоматическом вызове и не выполняется, последующий интерактивный запрос позволяет конечному пользователю войти в систему, используя другую учетную запись. После успешного выполнения AcquireToken с помощью учетной записи пользователя приложение должно подтвердить, что пользователь, подключенный к сети, соответствует локальному пользовательскому объекту приложения. Несоответствие не создает исключение (за исключением Objective C), но должно учитываться в случаях, когда пользователь известен локально перед выполнением запросов аутентификации (например, неудачных автоматических вызовом).
>

#### <a name="net"></a>.NET

Далее представлены примеры обработки ошибок и все методы ADAL с участием пользователей для AcquireToken(…), *кроме следующих*: 

- AcquireTokenAsync(…, IClientAssertionCertification, …);
- AcquireTokenAsync (..., ClientCredential,...)
- AcquireTokenAsync (..., Клиентассертион,...)
- AcquireTokenAsync (..., Усерассертион,...)   

Код будет реализован следующим образом:

```csharp
try {
    AcquireTokenAsync(…);
} 
    
catch(AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.
    
    // Design time consideration: Certain errors may be caused at development and exposed through this exception. 
    // Looking inside the description will give more guidance on resolving the specific issue. 

    // Action: Case 1: Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: default case

    } 

catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET.
    // e.ErrorCode contains the error code

    // Action: Case 1, Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: network_not_available, default case
}
```

> [!NOTE]
> Для ADAL .NET имеются дополнительные рекомендации, так как он поддерживает PromptBehavior.Never, поведение которого аналогично AcquireTokenSilent.
>

Далее представлены примеры для обработки ошибок в сочетании с методами ADAL: 

- acquireToken(…, PromptBehavior.Never)

Код будет реализован следующим образом:

```csharp
    try {acquireToken(…, PromptBehavior.Never);
    } 

catch(AdalServiceException e) {
    // Exception: AdalServiceException represents 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Action: Case 1: Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: default case

} catch (AdalException e) {
    // Error Code: e.ErrorCode == "user_interaction_required"
    // Description: user_interaction_required indicates the silent request failed 
    // in a way that's resolvable with an interactive request.
    // Action: Resolvable with an interactive request. 

    // Action: Case 1, Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: network_not_available, default case
}
```

#### <a name="android"></a>Android

Далее представлены примеры обработки событий и все методы ADAL с участием пользователей для AcquireToken(…). 

Код будет реализован следующим образом:

```java
AcquireTokenAsync(…);

// *Inside callback*
public void onError(Exception e) {
    if (e instanceof AuthenticationException) {
        // Exception: AdalException 
        // Represents a library exception generated by ADAL Android.
        // Error Code: e.getCode();

        // Error: ADALError.BROKER_APP_INSTALLATION_STARTED
        // Description: Broker app not installed, user will be prompted to download the app. 

        // Action: Case 2, Retriable Error 
        // Perform a single retry. If that fails, only try again after user action. 

        // Action: Case 1, Non-Retriable 
        // Do not perform an immediate retry. Only retry after user action. 
        // Example Errors: default case, DEVICE_CONNECTION_IS_NOT_AVAILABLE
    }
}
```

#### <a name="ios"></a>iOS

Далее представлены примеры обработки событий и все методы ADAL с участием пользователей для AcquireToken(…). 

Код будет реализован следующим образом:

```objc
[context acquireTokenWithResource:[ARGS], completionBlock:^(ADAuthenticationResult *result) {
    if (result.status == AD_FAILED) {
        if ([error.domain isEqualToString:ADAuthenticationErrorDomain]){
            // Exception: AD_FAILED 
            // Represents a library error generated by ADAL ObjC.
            // Error Code: result.error.code 

            // Error: AD_ERROR_SERVER_WRONG_USER
            // Description: App passed a user into ADAL and the end user signed in with a different account. 
            // Action: Case 1, Non-retriable (as is) and up to the application on how to handle this case. 
            // It can attempt a new request without specifying the user, or use UI to clarify the user account to sign in. 

            // Action: Case 1, Non-Retriable 
            // Do not perform an immediate retry. Only retry after user action. 
            // Example Errors: default case
        }
    }
}]
```

### <a name="error-cases-and-actionable-steps-web-applications-that-call-a-resource-api-net"></a>Сценарии ошибок и практические действия. Веб-приложения, вызывающие API ресурсов (.NET)

При создании веб-приложения .NET, которое вызывает получение токена с использованием кода авторизации для ресурса, единственным необходимым кодом является обработчик по умолчанию для общего сценария. 

Далее представлены примеры для обработки ошибок в сочетании с методами ADAL: 

- AcquireTokenByAuthorizationCodeAsync(…).

Код будет реализован следующим образом:

```csharp
try {
    AcquireTokenByAuthorizationCodeAsync(…);
} 

catch (AdalException e) {
    // Exception: AdalException
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action or wait until much later. 
    // Example Errors: default case
}
```

### <a name="error-cases-and-actionable-steps-single-page-applications-adaljs"></a>Сценарии ошибок и практические действия. Одностраничные приложения (adal.js)

При создании одностраничных приложений с использованием adal.js с AcquireToken код обработки ошибок будет аналогичен коду распространенных автоматических вызовов. В частности в adal.js AcquireToken никогда не отображает пользовательский интерфейс. 

Есть несколько типов ошибок AcquireToken:

|  |  |
|------|-------------|
| **Вариант 1**.<br>Ошибка устраняется с помощью интерактивного запроса | 1. Если произойдет сбой входа в систему (), не выполнять немедленную повторную попытку. Выполняйте повтор только после запроса пользователя.|
| **Вариант 2**.<br>Ошибка не устраняется с помощью интерактивного запроса Ошибка предусматривает возможность повтора. | 1. Выполните однократную попытку, так как основная конечная пользователь перешла в состояние, которое приводит к успешному выполнению.<br><br>2. Если повторная попытка завершается неудачно, представим пользователю действие, основанное на конкретной ошибке, которая может вызвать повторную попытку ("повторить попытку входа"). |
| **Вариант 3**.<br>Ошибка не устраняется с помощью интерактивного запроса Ошибка не предусматривает возможность повтора. | 1. не пытайтесь немедленно повторить попытку. Предоставьте пользователю действие на основе конкретной ошибки, которое может вызвать повторную попытку ("Повторите попытку входа"). |

Код будет реализован следующим образом:

```javascript
AuthContext.acquireToken(…, function(error, errorDesc, token) {
    if (error || errorDesc) {
        // Represents any token acquisition failure that occurred. 
        // Error Code: error.indexOf("<ERROR_STRING>")

        // Errors: if (error.indexOf("interaction_required"))
        //         if (error.indexOf("login required"))
        // Description: ADAL wasn't able to silently acquire a token because of expire or fresh session. 
        // Action: Case 1, Resolvable with an interactive login() request. 

        // Error: if (error.indexOf("Token Renewal Failed")) 
        // Description: Timeout when refreshing the token.
        // Action: Case 2, Not resolvable interactively, error is retriable.
        // Perform a single retry. Only try again after user action.

        // Action: Case 3, Not resolvable interactively, error is not retriable. 
        // Do not perform an immediate retry. Only retry after user action.
        // Example Errors: default case
    }
}
```

### <a name="error-cases-and-actionable-steps-service-to-service-applications-net-only"></a>Сценарии ошибок и практические действия. Приложения для обмена данными между службами (только .NET)

При создании приложения для обмена данными между службами, которое использует AcquireToken, есть несколько ключевых ошибок, которые должен обрабатывать код. Единственным средством разрешения сбоя является возвращение ошибки обратно к вызывающему приложению (для сценариев типа "от имени") или применение стратегии повторов. 

#### <a name="all-scenarios"></a>Все сценарии

Для *всех* сценариев приложения для обмена данными между службами, включая сценарии типа "от имени":

- Не пытайтесь немедленно повторить попытку. ADAL пытается выполнить одну попытку для некоторых неудачных запросов. 
- Продолжайте повторные попытки только после того, как пользователь или действие приложения предложат выполнить повторную попытку. Например, управляющая программа, работающая с определенным заданным интервалом, должна ожидать следующего интервала для повтора.

Далее представлены примеры для обработки ошибок в сочетании с методами ADAL: 

- AcquireTokenAsync(…, IClientAssertionCertification, …);
- AcquireTokenAsync(…,ClientCredential, …);
- AcquireTokenAsync(…,ClientAssertion, …);
- AcquireTokenAsync(…,UserAssertion, …).

Код будет реализован следующим образом:

```csharp
try {
    AcquireTokenAsync(…);
} 

catch (AdalException e) {
    // Exception: AdalException
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action (if applicable) or wait until much later. 
    // Example Errors: default case
}  
```

#### <a name="on-behalf-of-scenarios"></a>Сценарии типа "от имени"

Для сценариев типа *от имени*, в которых задействованы приложения для обмена данными между службами.

Далее представлены примеры для обработки ошибок в сочетании с методами ADAL: 

- AcquireTokenAsync(…, UserAssertion, …).

Код будет реализован следующим образом:

```csharp
try {
AcquireTokenAsync(…);
} 

catch (AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Error: On-Behalf-Of Error Handler
    if (e.ErrorCode == "interaction_required") {
        // Description: The client needs to perform some action due to a config from admin. 
        // Action: Capture `claims` parameter inside ex.InnerException.InnerException.
        // Generate HTTP error 403 with claims, throw back HTTP error to client.
        // Wait for client to retry. 
    }
} 
        
catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action (if applicable) or wait until much later. 
    // Example Error: default case
}
```

Мы создали [полный пример](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof-ca), демонстрирующий этот сценарий.

## <a name="error-and-logging-reference"></a>Справочник по ведению журнала и ошибкам

### <a name="logging-personal-identifiable-information--organizational-identifiable-information"></a>Ведение журнала личных сведений & сведения об Организации 
По умолчанию ведение журнала ADAL не записывает и не регистрирует личные сведения или сведения, предоставляемые Организацией. Библиотека позволяет разработчикам приложений включить их запись с помощью метода задания в классе Logger. Записывая личные сведения или сведения, предоставляемые Организацией, приложение несет ответственность за безопасную обработку высокочувствительных данных и соблюдение нормативных требований.

### <a name="net"></a>.NET

#### <a name="adal-library-errors"></a>Библиотека ошибок ADAL

При изучении определенных ошибок ADAL исходный код в [репозитории azure-activedirectory-library-for-dotnet](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/8f6d560fbede2247ec0e217a21f6929d4375dcaa/src/ADAL.PCL/Utilities/Constants.cs#L58) является наилучшим справочников по ошибкам.

#### <a name="guidance-for-error-logging-code"></a>Рекомендации по коду ведения журнала ошибок

Изменения ведения журнала ADAL .NET зависят от используемой платформы. Код для включения ведения журнала приведен на [вики-сайте о ведении журнала](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Logging-in-ADAL.Net).

### <a name="android"></a>Android

#### <a name="adal-library-errors"></a>Библиотека ошибок ADAL

При изучении определенных ошибок ADAL исходный код в [репозитории azure-activedirectory-library-for-android](https://github.com/AzureAD/azure-activedirectory-library-for-android/blob/dev/adal/src/main/java/com/microsoft/aad/adal/ADALError.java#L33) является наилучшим справочников по ошибкам.

#### <a name="operating-system-errors"></a>Ошибки операционной системы

Ошибки ОС Android обнаруживаются через AuthenticationException в ADAL, определяются как SERVER_INVALID_REQUEST и в описании ошибок о них можно получить дополнительные сведения. 

Полный список распространенных ошибок в работе приложений и пользователей, а также действия по их устранению можно найти на [вики-сайте ADAL для Android](https://github.com/AzureAD/azure-activedirectory-library-for-android/wiki). 

#### <a name="guidance-for-error-logging-code"></a>Рекомендации по коду ведения журнала ошибок

```java
// 1. Configure Logger
Logger.getInstance().setExternalLogger(new ILogger() {    
    @Override   
    public void Log(String tag, String message, String additionalMessage, LogLevel level, ADALError errorCode) { 
    // …
    // You can write this to logfile depending on level or errorcode. 
    writeToLogFile(getApplicationContext(), tag +":" + message + "-" + additionalMessage);    
    }
}

// 2. Set the log level
Logger.getInstance().setLogLevel(Logger.LogLevel.Verbose);

// By default, the `Logger` does not capture any PII or OII

// PII or OII will be logged
Logger.getInstance().setEnablePII(true);

// To STOP logging PII or OII, use the following setter
Logger.getInstance().setEnablePII(false);


// 3. Send logs to logcat.
adb logcat > "C:\logmsg\logfile.txt";
```

### <a name="ios"></a>iOS

#### <a name="adal-library-errors"></a>Библиотека ошибок ADAL

При изучении определенных ошибок ADAL исходный код в [репозитории azure-activedirectory-library-for-objc](https://github.com/AzureAD/azure-activedirectory-library-for-objc/blob/dev/ADAL/src/ADAuthenticationError.m#L295) является наилучшим справочников по ошибкам.

#### <a name="operating-system-errors"></a>Ошибки операционной системы

Ошибки iOS могут возникать во время входа в систему, когда пользователи используют веб-представления. Кроме того, они могут возникать из-за типа аутентификации. Это может быть вызвано такими условиями, как ошибки TLS, время ожидания или ошибки сети:

- При совместном использовании прав доступа имена входа не являются постоянными и кэш остается пустым. Эту проблему можно решить, добавив следующую строку кода в цепочку ключей: `[[ADAuthenticationSettings sharedInstance] setSharedCacheKeychainGroup:nil];`.
- Для ошибок NsUrlDomain действие изменяется в зависимости от логики приложения. Дополнительные сведения об обрабатываемых экземплярах см. в [справочной документации о NSURLErrorDomain](https://developer.apple.com/documentation/foundation/nsurlerrordomain#declarations).
- Дополнительные сведения о списке распространенных проблем, устранением которых занимается группа ADAL Objective-C, см. в [этом разделе](https://github.com/AzureAD/azure-activedirectory-library-for-objc#adauthenticationerror).

#### <a name="guidance-for-error-logging-code"></a>Рекомендации по коду ведения журнала ошибок

```objc
// 1. Enable NSLogging
[ADLogger setNSLogging:YES];

// 2. Set the log level (if you want verbose)
[ADLogger setLevel:ADAL_LOG_LEVEL_VERBOSE];

// 3. Set up a callback block to simply print out
[ADLogger setLogCallBack:^(ADAL_LOG_LEVEL logLevel, NSString *message, NSString *additionalInformation, NSInteger errorCode, NSDictionary *userInfo) {
     NSString* log = [NSString stringWithFormat:@"%@ %@", message, additionalInformation];    
     NSLog(@"%@", log);
}];
```

### <a name="guidance-for-error-logging-code---javascript"></a>Рекомендации по коду ведения журнала ошибок. JavaScript 

```javascript
0: Error1: Warning2: Info3: Verbose
window.Logging = {
    level: 3,
    log: function (message) {
        console.log(message);
    }
};
```

## <a name="related-content"></a>См. также

* [Библиотеки аутентификация Azure AD][AAD-Auth-Libraries]
* [Azure AD Authentication Scenarios][AAD-Auth-Scenarios] (Сценарии аутентификации в Azure Active Directory)
* [Интеграция приложений с Azure Active Directory][AAD-Integrating-Apps]

Оставляйте свои замечания и пожелания в разделе ниже. Они помогают нам улучшать содержимое веб-сайта.

[![Отображает кнопку "Войдите с помощью учетной записи Майкрософт"][AAD-Sign-In]][AAD-Sign-In]
<!--Reference style links -->

[AAD-Auth-Libraries]: ./active-directory-authentication-libraries.md
[AAD-Auth-Scenarios]:v1-authentication-scenarios.md
[AAD-Integrating-Apps]:../develop/quickstart-register-app.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json
[AZURE-portal]: https://portal.azure.com

<!--Image references-->
[AAD-Sign-In]:./media/active-directory-devhowto-multi-tenant-overview/sign-in-with-microsoft-light.png

