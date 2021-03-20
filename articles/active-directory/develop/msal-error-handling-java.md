---
title: Обработку ошибок и исключений в MSAL4J
titleSuffix: Microsoft identity platform
description: Сведения об обработке ошибок и исключений, проблемах условного доступа и повторных попыток в MSAL4J приложениях.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/27/2020
ms.author: marsma
ms.reviewer: saeeda, nacanuma
ms.custom: aaddev
ms.openlocfilehash: 8563804a736c37acc9a96eb4a186933507f34200
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98760654"
---
# <a name="handle-errors-and-exceptions-in-msal-for-java"></a>Обработка ошибок и исключений в MSAL для Java

[!INCLUDE [Active directory error handling introduction](../../../includes/active-directory-develop-error-handling-introduction.md)]

## <a name="error-handling-in-msal-for-java"></a>Обработка ошибок в MSAL для Java

В MSAL для Java существуют три типа исключений: `MsalClientException`, `MsalServiceException` и `MsalInteractionRequiredException`. Все они наследуются от `MsalException`.

- `MsalClientException` происходит в случае ошибки, которая является локальной для библиотеки или устройства.
- `MsalServiceException` происходит, когда служба токенов безопасности (STS) возвращает ответ со сведениями об ошибке или происходит другая ошибка сети.
- `MsalInteractionRequiredException` происходит, когда для прохождения проверки подлинности требуется взаимодействие с пользовательским интерфейсом.

### <a name="msalserviceexception"></a>MsalServiceException

`MsalServiceException` предоставляет HTTP-заголовки, возвращаемые в запросах к STS. Для доступа к ним используется `MsalServiceException.headers()`.

### <a name="msalinteractionrequiredexception"></a>MsalInteractionRequiredException

Один из распространенных кодов состояния, возвращаемых из MSAL для Java при вызове `AcquireTokenSilently()`, — `InvalidGrantError`. Он означает, что перед выдачей токена проверки подлинности требуется дополнительное взаимодействие с пользователем. Приложение должно снова вызвать библиотеку проверки подлинности, но в интерактивном режиме, отправив `AuthorizationCodeParameters` или `DeviceCodeParameters` для общедоступных клиентских приложений.

В большинстве случаев сбой `AcquireTokenSilently` происходит из-за того, что кэш токенов не содержит токены, соответствующие запросу. Срок действия маркеров доступа истекает через один час, а `AcquireTokenSilently` попытается получить новый на основе токена обновления. В терминах OAuth2 это поток "Обновление токена". Этот поток также может завершиться сбоем по различным причинам, например, если администратор клиента настроит более строгие политики входа в систему.

С некоторыми из ситуаций, которые приводят к такой ошибке, пользователи могут легко справиться. Например, им может потребоваться принять условия использования, или запрос не может быть выполнен с текущей конфигурацией, так как компьютер должен подключаться к определенной корпоративной сети.

MSAL предоставляет поле `reason`, которое можно использовать для повышения удобства работы пользователей. Например, с помощью поля `reason` можно сообщить пользователю о том, что срок действия пароля истек или что ему необходимо дать согласие на использование некоторых ресурсов. Поддерживаемые значения являются частью перечисления `InteractionRequiredExceptionReason`.

| Причина | Значение | Рекомендуемая обработка |
|---------|-----------|-----------------------------|
| `BasicAction` | Ситуацию можно разрешить путем взаимодействия с пользователем во время интерактивного потока проверки подлинности. | Вызовите `acquireToken` с помощью интерактивных параметров. |
| `AdditionalAction` | Ситуацию можно разрешить с помощью дополнительного корректирующего взаимодействия с системой за пределами интерактивного потока проверки подлинности. | Вызов `acquireToken` с интерактивными параметрами для отображения сообщения с объяснением корректирующего действия, которое необходимо выполнить. Вызывающее приложение может скрыть потоки, требующие additional_action, если пользователь, скорее всего, не станет выполнять корректирующее действие. |
| `MessageOnly` | В данный момент разрешить ситуацию невозможно. При запуске интерактивного потока проверки подлинности отобразится сообщение с объяснением сложившейся ситуации. | Вызов `acquireToken` с интерактивными параметрами для отображения сообщения с объяснением сложившейся ситуации. `acquireToken` будет возвращать ошибку `UserCanceled` после того, как пользователь прочтет сообщение и закроет окно. Это приложение может скрыть потоки, которые приводят к отображению сообщения, если оно, скорее всего, будет бесполезным для пользователя. |
| `ConsentRequired`| Согласие пользователя отсутствует или отменено. |Вызов `acquireToken` с интерактивными параметрами для получения согласия от пользователя. |
| `UserPasswordExpired` | Срок действия пароля пользователя истек. | Вызовите `acquireToken` с помощью интерактивного параметра, чтобы пользователь мог сбросить свой пароль. |
| `None` |  Предоставляются дополнительные сведения. Ситуацию можно разрешить путем взаимодействия с пользователем во время интерактивного потока проверки подлинности. | Вызовите `acquireToken` с помощью интерактивных параметров. |

### <a name="code-example"></a>Пример кода

```java
        IAuthenticationResult result;
        try {
            PublicClientApplication application = PublicClientApplication
                    .builder("clientId")
                    .b2cAuthority("authority")
                    .build();

            SilentParameters parameters = SilentParameters
                    .builder(Collections.singleton("scope"))
                    .build();

            result = application.acquireTokenSilently(parameters).join();
        }
        catch (Exception ex){
            if(ex instanceof MsalInteractionRequiredException){
                // AcquireToken by either AuthorizationCodeParameters or DeviceCodeParameters
            } else{
                // Log and handle exception accordingly
            }
        }
```

[!INCLUDE [Active directory error handling claims challenges](../../../includes/active-directory-develop-error-handling-claims-challenges.md)]

[!INCLUDE [Active directory error handling retries](../../../includes/active-directory-develop-error-handling-retries.md)]

## <a name="next-steps"></a>Дальнейшие действия

Рекомендуется включить [ведение журнала в MSAL для Java](msal-logging-java.md) , чтобы помочь в диагностике и отладке проблем.
