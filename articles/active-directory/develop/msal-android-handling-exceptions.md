---
title: Ошибки и исключения (MSAL Android) | Службы
titleSuffix: Microsoft identity platform
description: Сведения об обработке ошибок и исключений, условном доступе и проблемах с заявками в приложениях MSAL Android.
services: active-directory
author: hamiltonha
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: troubleshooting
ms.workload: identity
ms.date: 08/07/2020
ms.author: hahamil
ms.reviewer: marsma
ms.openlocfilehash: 4185206e92a78c2684ba1690f03700ef2532cc5e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98761398"
---
# <a name="handle-exceptions-and-errors-in-msal-for-android"></a>Обработку исключений и ошибок в MSAL для Android

Исключения в библиотеке проверки подлинности Майкрософт (MSAL) предназначены для того, чтобы помочь разработчикам приложений устранять неполадки в их приложениях. Сообщения об исключениях не локализованы.

При обработке ошибок и исключений вы можете различать их по типам исключений и кодам ошибки.  Список кодов ошибок аутентификации и авторизации можно найти [здесь](reference-aadsts-error-codes.md).

Во время входа в систему могут происходить ошибки, связанные с согласованностью, условным доступом (MFA, управлением устройствами, ограничениями на основе расположения), выдачей и возвратом токенов и свойствами пользователей.


|Класс ошибки | Причина/строка ошибки| Как работать |
|-----------|------------|----------------|
|`MsalUiRequiredException`| <ul><li>`INVALID_GRANT`: Маркер обновления, используемый для активации маркера доступа, недопустим, просрочен или отозван. Это исключение может быть вызвано изменением пароля. </li><li>`NO_TOKENS_FOUND`: Маркер доступа не существует, и не удается найти токен обновления для активации токена доступа.</li> <li>Требуется шаг с заходом<ul><li>MFA</li><li>Отсутствующие утверждения</li></ul></li><li>Заблокировано условным доступом (например, требуется установка [брокера проверки подлинности](./msal-android-single-sign-on.md) )</li><li>`NO_ACCOUNT_FOUND`: В кэше нет доступных учетных записей для автоматической проверки подлинности.</li></ul> |Вызовите, `acquireToken()` чтобы предложить пользователю ввести имя пользователя и пароль и, возможно, предоставить согласие и выполнить многофакторную проверку подлинности.|
|`MsalDeclinedScopeException`|<ul><li>`DECLINED_SCOPE`: Пользователь или сервер не принял все области. Сервер может отклонить область, если запрошенная область не поддерживается, не распознана или не поддерживается для конкретной учетной записи. </li></ul>| Разработчик должен решить, следует ли продолжать проверку подлинности с предоставленными областями или завершить процесс проверки подлинности. Параметр для повторной отправки запроса на получение маркера получения только для предоставленных областей и указания разрешений, которые были предоставлены передачей `silentParametersForGrantedScopes` и вызовом `acquireTokenSilent` . |
|`MsalServiceException`|<ul><li>`INVALID_REQUEST`: В этом запросе отсутствует обязательный параметр, он содержит недопустимый параметр, включает параметр несколько раз или в противном случае имеет неправильный формат. </li><li>`SERVICE_NOT_AVAILABLE`— Представляет коды ошибок 500/503/506 из-за отключения службы. </li><li>`UNAUTHORIZED_REQUEST`: Клиент не авторизован для запроса кода авторизации.</li><li>`ACCESS_DENIED`: Владелец ресурса или сервер авторизации отклонил запрос.</li><li>`INVALID_INSTANCE`: `AuthorityMetadata` Проверка не пройдена</li><li>`UNKNOWN_ERROR`: Запрос к серверу не выполнен, но нет ошибок и `error_description` возвращено обратно из службы.</li><ul>| Этот класс исключений представляет ошибки при взаимодействии со службой, может быть из конечных точек авторизации или маркеров. MSAL считывает ошибку и error_description от ответа сервера. Как правило, эти ошибки устраняются путем исправления конфигураций приложений либо в коде, либо на портале регистрации приложений. Нередкое отключение службы может вызвать это предупреждение, которое может быть устранено только путем ожидания восстановления службы.  |
|`MsalClientException`|<ul><li> `MULTIPLE_MATCHING_TOKENS_DETECTED`: Обнаружено несколько записей кэша, и пакет SDK не может опознать правильный маркер доступа или обновления из кэша. Это исключение обычно указывает на ошибку в пакете SDK для хранения маркеров или на то, что центр не указан в автоматическом запросе и найдено несколько соответствующих токенов. </li><li>`DEVICE_NETWORK_NOT_AVAILABLE`: На устройстве нет доступной активной сети. </li><li>`JSON_PARSE_FAILURE`: Пакету SDK не удалось проанализировать формат JSON.</li><li>`IO_ERROR`. `IOException` возможно, это может быть устройство или сетевая ошибка. </li><li>`MALFORMED_URL`: URL-адрес имеет неправильный формат. Вероятно, это вызвано созданием запроса проверки подлинности, центра или URI перенаправления. </li><li>`UNSUPPORTED_ENCODING`: Кодировка не поддерживается устройством. </li><li>`NO_SUCH_ALGORITHM`: Алгоритм, используемый для создания запроса [PKCE](https://tools.ietf.org/html/rfc7636) , не поддерживается. </li><li>`INVALID_JWT`: `JWT` возвращенный сервером является недопустимым или пуст или имеет неправильный формат. </li><li>`STATE_MISMATCH`: Состояние из ответа авторизации не соответствует состоянию в запросе авторизации. Для запросов авторизации пакет SDK будет проверять состояние, возвращенное из перенаправления, и отправленное в запросе. </li><li>`UNSUPPORTED_URL`: Неподдерживаемый URL-адрес, не удается выполнить проверку центра ADFS. </li><li> `AUTHORITY_VALIDATION_NOT_SUPPORTED`: Центр проверки полномочий не поддерживается. Пакет SDK поддерживает B2Cные центры, но не поддерживает проверку центра B2C. Будет поддерживаться только хорошо известный узел. </li><li>`CHROME_NOT_INSTALLED`: Chrome не установлен на устройстве. Пакет SDK использует настраиваемую вкладку Chrome для запросов авторизации, если она доступна, и будет возвращаться браузеру Chrome. </li><li>`USER_MISMATCH`: Пользователь, указанный в запросе маркера получения, не соответствует пользователю, возвращенному сервером.</li></ul>|Этот класс исключений представляет общие ошибки, которые являются локальными для библиотеки. Эти исключения могут быть обработаны путем исправления запроса.|
|`MsalUserCancelException`|<ul><li>`USER_CANCELED`: Пользователь инициировал интерактивный поток и перед получением токенов отменил запрос. </li></ul>||
|`MsalArgumentException`|<ul><li>`ILLEGAL_ARGUMENT_ERROR_CODE`</li><li>`AUTHORITY_REQUIRED_FOR_SILENT`: Необходимо указать Authority для `acquireTokenSilent` .</li></ul>|Эти ошибки могут быть устранены с помощью корректирующих аргументов разработчиками, а также обеспечивать действия для интерактивной проверки подлинности, обратного вызова завершения, областей и учетной записи с допустимым ИДЕНТИФИКАТОРом.|


## <a name="catching-errors"></a>выявления ошибок;

В следующем фрагменте кода показан пример перехвата ошибок при автоматическом `acquireToken` вызове.

```java
/**
 * Callback used in for silent acquireToken calls.
 */
private SilentAuthenticationCallback getAuthSilentCallback() {
    return new SilentAuthenticationCallback() {

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            Log.d(TAG, "Successfully authenticated");

            /* Successfully got a token, use it to call a protected resource - MSGraph */
            callGraphAPI(authenticationResult);
        }

        @Override
        public void onError(MsalException exception) {
            /* Failed to acquireToken */
            Log.d(TAG, "Authentication failed: " + exception.toString());
            displayError(exception);

            if (exception instanceof MsalClientException) {
                /* Exception inside MSAL, more info inside MsalError.java */
            } else if (exception instanceof MsalServiceException) {
                /* Exception when communicating with the STS, likely config issue */
            } else if (exception instanceof MsalUiRequiredException) {
                /* Tokens expired or no session, retry with interactive */
            }
        }
    };
}
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [ведении журналов в MSAL для Android](msal-logging-android.md).