---
title: Добавление настраиваемых утверждений в потоки для самостоятельной регистрации в Azure AD
description: Добавление соединителей API для пользовательских рабочих процессов утверждения во внешних удостоверениях самостоятельная регистрация Azure Active Directory (Azure AD)
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: article
ms.date: 03/02/2021
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 95274f42da7f6cac9b193504df834232d7c0eb90
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102609996"
---
# <a name="add-a-custom-approval-workflow-to-self-service-sign-up"></a>Добавление пользовательского рабочего процесса утверждения для самостоятельной регистрации

С помощью [соединителей API](api-connectors-overview.md)можно интегрироваться с собственными настраиваемыми рабочими процессами утверждения с помощью самостоятельной регистрации, чтобы вы могли управлять тем, какие учетные записи гостевых пользователей создаются в клиенте.

В этой статье приводится пример интеграции с системой утверждения. В этом примере пользовательская процедура регистрации самостоятельно собирает данные пользователя во время регистрации и передает их в систему утверждения. Затем система утверждения может:

- Автоматически утвердите пользователя и разрешите Azure AD создать учетную запись пользователя.
- Активация проверки вручную. Если запрос утвержден, система утверждения использует Microsoft Graph для предоставления учетной записи пользователя. Система утверждения также может уведомлять пользователя о том, что учетная запись создана.

> [!IMPORTANT]
>**Начиная с 4 января 2021 г.** компания Google [прекращает поддержку входа через веб-представление](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html). Если вы используете федерацию Google или самостоятельную регистрацию с помощью Gmail, вам нужно [проверить нативные бизнес-приложения на совместимость](google-federation.md#deprecation-of-webview-sign-in-support).

## <a name="register-an-application-for-your-approval-system"></a>Регистрация приложения для системы утверждения

Необходимо зарегистрировать систему утверждения в качестве приложения в клиенте Azure AD, чтобы она могла проходить проверку подлинности в Azure AD и иметь разрешение на создание пользователей. Узнайте больше об [основах проверки подлинности и авторизации для Microsoft Graph](/graph/auth/auth-concepts).

1. Войдите на [портал Azure](https://portal.azure.com) с учетной записью администратора Azure AD.
2. В разделе **Службы Azure** щелкните **Azure Active Directory**.
3. В меню слева выберите **Регистрация приложений** и щелкните **Новая регистрация**.
4. Введите **имя** приложения, например _утверждение регистрации_.

   <!-- ![Register an application for the approval system](./self-service-sign-up-add-approvals/approvals/register-an-approvals-application.png) -->

5. Выберите **Зарегистрировать**. Можно оставить другие поля по умолчанию.

   ![Снимок экрана, на котором выделена кнопка "зарегистрировать".](media/self-service-sign-up-add-approvals/register-approvals-app.png)

6. В разделе **Управление** в меню слева выберите **разрешения API**, а затем щелкните **Добавить разрешение**.
7. На странице **разрешения API запроса** выберите **Microsoft Graph**, а затем щелкните **разрешения приложения**.
8. В разделе **Выбор разрешений** разверните узел **пользователь** и установите флажок **User. ReadWrite. ALL** . Это разрешение позволяет системе утверждения создавать пользователя при утверждении. Нажмите кнопку **Добавить разрешения**.

   ![Страница регистрации приложения](media/self-service-sign-up-add-approvals/request-api-permissions.png)

9. На странице **разрешения API** выберите параметр **предоставить согласие администратора для (имя вашего клиента)** и нажмите кнопку **Да**.
10. В разделе **Управление** в меню слева выберите **Сертификаты & секреты**, а затем выберите **новый секрет клиента**.
11. Введите **Описание** секрета, например _утверждение секрета клиента_, и выберите продолжительность **срока действия** секрета клиента. Нажмите кнопку **Добавить**.
12. Скопируйте значение секрета клиента.

    ![Копирование секрета клиента для использования в системе утверждения](media/self-service-sign-up-add-approvals/client-secret-value-copy.png)

13. Настройте систему утверждения для использования **идентификатора приложения** в качестве идентификатора клиента и **секрета клиента** , созданного для проверки подлинности в Azure AD.

## <a name="create-the-api-connectors"></a>Создание соединителей API

Далее предстоит [создать соединители API](self-service-sign-up-add-api-connector.md#create-an-api-connector) для пользовательского потока регистрации самообслуживания. API системы утверждения нуждается в двух соединителях и соответствующих конечных точках, как показано в примерах ниже. Эти соединители API выполняют следующие действия:

- **Проверьте состояние утверждения**. Отправка вызова в систему утверждения сразу после входа пользователя с помощью поставщика удостоверений для проверки наличия запроса на утверждение у пользователя или его запрета. Если система одобрения выполняет только автоматическое утверждение, этот соединитель API может не потребоваться. Пример соединителя API "проверить состояние утверждения".

  ![Проверка конфигурации соединителя API для состояния утверждения](./media/self-service-sign-up-add-approvals/check-approval-status-api-connector-config-alt.png)

- **Запрос утверждения** — отправка вызова в систему утверждения после того, как пользователь завершит страницу коллекции атрибутов, но до создания учетной записи пользователя, чтобы запросить утверждение. Запрос на утверждение может быть автоматически предоставлен или проверен вручную. Пример соединителя API "Утверждение запроса". 

  ![Настройка соединителя API для утверждения запросов](./media/self-service-sign-up-add-approvals/create-approval-request-api-connector-config-alt.png)

Чтобы создать эти соединители, выполните действия, описанные в разделе [Создание СОЕДИНИТЕЛЯ API](self-service-sign-up-add-api-connector.md#create-an-api-connector).

## <a name="enable-the-api-connectors-in-a-user-flow"></a>Включение соединителей API в потоке пользователя

Теперь вы добавите соединители API в процедуру самостоятельной регистрации пользователей, выполнив следующие действия:

1. Войдите на [портал Azure](https://portal.azure.com/) с учетной записью администратора Azure AD.
2. В разделе **Службы Azure** щелкните **Azure Active Directory**.
3. В меню слева щелкните **Внешние удостоверения**.
4. Выберите последовательности **пользователей**, а затем выберите пользовательский поток, для которого необходимо включить соединитель API.
5. Выберите **соединители API**, а затем выберите конечные точки API, которые нужно вызвать, на следующих шагах в потоке пользователя:

   - **После входа с помощью поставщика удостоверений** выберите соединитель API состояния утверждения, например " _проверить состояние утверждения_".
   - **Перед созданием пользователя**: выберите соединитель API запроса на утверждение, например _запрос утверждения_.

   ![Добавление API в поток пользователя](./media/self-service-sign-up-add-approvals/api-connectors-user-flow-api.png)

6. Щелкните **Сохранить**.

## <a name="control-the-sign-up-flow-with-api-responses"></a>Управление потоком регистрации с помощью ответов API

Ваша система утверждения может использовать свои ответы при вызове для управления потоком регистрации. 

### <a name="request-and-responses-for-the-check-approval-status-api-connector"></a>Запрос и ответы для соединителя API "проверить состояние утверждения"

Пример запроса, полученного API из соединителя API "проверить состояние утверждения":

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ //Sent for Google and Facebook identity providers
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "lastName":"Smith",
 "ui_locales":"en-US"
}
```

Точные утверждения, отправляемые в API, зависят от того, какие сведения предоставляются поставщиком удостоверений. "Электронная почта" всегда отправляется.

#### <a name="continuation-response-for-check-approval-status"></a>Ответ на продолжение для "Проверка состояния утверждения"

Конечная точка API **проверки состояния утверждения** должна возвращать ответ на продолжение, если:

- Пользователь ранее не запрашивал утверждение.

Пример ответа на продолжение:

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue"
}
```

#### <a name="blocking-response-for-check-approval-status"></a>Блокировка ответа для "Проверка состояния утверждения"

Конечная точка API **проверки состояния утверждения** должна возвращать блокирующий ответ, если:

- Ожидание утверждения пользователем.
- Пользователь был отклонен, и ему не следует разрешать запрос на утверждение.

Ниже приведены примеры блокировки ответов.

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your access request is already processing. You'll be notified when your request has been approved.",
}
```

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your sign up request has been denied. Please contact an administrator if you believe this is an error",
}
```

### <a name="request-and-responses-for-the-request-approval-api-connector"></a>Запрос и ответы для соединителя API "Утверждение запроса"

Пример HTTP-запроса, полученного API из соединителя API "Утверждение запроса":

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ //Sent for Google and Facebook identity providers
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "surname":"Smith",
 "jobTitle":"Supplier",
 "streetAddress":"1000 Microsoft Way",
 "city":"Seattle",
 "postalCode": "12345",
 "state":"Washington",
 "country":"United States",
 "extension_<extensions-app-id>_CustomAttribute1": "custom attribute value",
 "extension_<extensions-app-id>_CustomAttribute2": "custom attribute value",
 "ui_locales":"en-US"
}
```

Точные утверждения, отправляемые в API, зависят от того, какие сведения собираются от пользователя или предоставлены поставщиком удостоверений.

#### <a name="continuation-response-for-request-approval"></a>Ответ на продолжение для утверждения запроса

Конечная точка API **утверждения запроса** должна возвращать ответ на продолжение, если:

- Пользователь может быть **_утвержден автоматически_**.

Пример ответа на продолжение:

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue"
}
```

> [!IMPORTANT]
> Если получен ответ продолжения, Azure AD создает учетную запись пользователя и направляет ее в приложение.

#### <a name="blocking-response-for-request-approval"></a>Блокировка ответа для утверждения запроса

Конечная точка API **утверждения запроса** должна возвращать блокирующий ответ, если:

- Запрос на утверждение пользователя создан и сейчас находится в состоянии ожидания.
- Запрос на утверждение пользователя был автоматически отклонен.

Ниже приведены примеры блокировки ответов.

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your account is now waiting for approval. You'll be notified when your request has been approved.",
}
```

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your sign up request has been denied. Please contact an administrator if you believe this is an error",
}
```

В `userMessage` ответе отображается пользователь, например:

![Пример страницы утверждения, ожидающей выполнения](./media/self-service-sign-up-add-approvals/approval-pending.png)

## <a name="user-account-creation-after-manual-approval"></a>Создание учетной записи пользователя после утверждения вручную

После получения утверждения вручную система пользовательского утверждения создает учетную запись [пользователя](/graph/azuread-users-concept-overview) с помощью [Microsoft Graph](/graph/use-the-api). Способ, которым система утверждения подготавливает учетную запись пользователя, зависит от поставщика удостоверений, который использовался этим пользователем.

### <a name="for-a-federated-google-or-facebook-user-and-email-one-time-passcode"></a>Для федеративного пользователя Google или Facebook и одноразового секретного кода для электронной почты

> [!IMPORTANT]
> Для использования этого метода система утверждения должна явным образом проверить наличие и наличие `identities` `identities[0]` `identities[0].issuer` и то есть `identities[0].issuer` "Facebook", "Google" или "mail".

Если пользователь вошел в учетную запись Google или Facebook или одноразовый пароль для электронной почты, можно использовать [API создания пользователей](/graph/api/user-post-users?tabs=http).

1. Система утверждения использует получение HTTP-запроса от потока пользователя.

```http
POST <Approvals-API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@outlook.com",
 "identities": [
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value",
 "ui_locales":"en-US"
}
```

2. Система утверждения использует Microsoft Graph для создания учетной записи пользователя.

```http
POST https://graph.microsoft.com/v1.0/users
Content-type: application/json

{
 "userPrincipalName": "johnsmith_outlook.com#EXT@contoso.onmicrosoft.com",
 "accountEnabled": true,
 "mail": "johnsmith@outlook.com",
 "userType": "Guest",
 "identities": [
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value"
}
```

| Параметр                                           | Обязательно | Описание                                                                                                                                                            |
| --------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| userPrincipalName                                   | Да      | Можно создать, `email` отправив заявку в API, заменив символ на `@` и до `_` `#EXT@<tenant-name>.onmicrosoft.com` . |
| AccountEnabled                                      | Да      | Нужно задать значение `true`.                                                                                                                                                 |
| mail                                                | Да      | Эквивалентно `email` утверждению, отправленному в API.                                                                                                               |
| userType                                            | Да      | Этот параметр должен содержать значение `Guest`. Назначает этого пользователя в качестве гостевого пользователя.                                                                                                                 |
| удостоверения;                                          | Да      | Сведения об федеративного удостоверения.                                                                                                                                    |
| \<otherBuiltInAttribute>                            | Нет       | Другие встроенные атрибуты, такие как `displayName` , `city` и другие. Имена параметров совпадают с параметрами, отправленными соединителем API.                            |
| \<extension\_\{extensions-app-id}\_CustomAttribute> | Нет       | Настраиваемые атрибуты пользователя. Имена параметров совпадают с параметрами, отправленными соединителем API.                                                            |

### <a name="for-a-federated-azure-active-directory-user-or-microsoft-account-user"></a>Для федеративного Azure Active Directory пользователя или учетная запись Майкрософт пользователя

Если пользователь выполняет вход с помощью федеративной учетной записи Azure Active Directory или учетная запись Майкрософт, необходимо использовать [API приглашения](/graph/api/invitation-post) для создания пользователя, а затем, при необходимости, [API пользовательского обновления](/graph/api/user-update) , чтобы назначить пользователю дополнительные атрибуты.

1. Система утверждения получает HTTP-запрос от потока пользователя.

```http
POST <Approvals-API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value",
 "ui_locales":"en-US"
}
```

2. Система утверждения создает приглашение с помощью `email` предоставленного СОЕДИНИТЕЛЯ API.

```http
POST https://graph.microsoft.com/v1.0/invitations
Content-type: application/json

{
    "invitedUserEmailAddress": "johnsmith@fabrikam.onmicrosoft.com",
    "inviteRedirectUrl" : "https://myapp.com"
}
```

Пример ответа:

```http
HTTP/1.1 201 OK
Content-type: application/json

{
    ...
    "invitedUser": {
        "id": "<generated-user-guid>"
    }
}
```

3. Система утверждения использует идентификатор приглашенного пользователя для обновления учетной записи пользователя с помощью собранных атрибутов пользователя (необязательно).

```http
PATCH https://graph.microsoft.com/v1.0/users/<generated-user-guid>
Content-type: application/json

{
    "displayName": "John Smith",
    "city": "Redmond",
    "extension_<extensions-app-id>_AttributeName": "custom attribute value"
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Приступая к работе с [примерами краткого руководства по функциям Azure](code-samples-self-service-sign-up.md#api-connector-azure-function-quickstarts).
- Извлечение [самообслуживания для гостевых пользователей с примером утверждения вручную](code-samples-self-service-sign-up.md#custom-approval-workflows).
