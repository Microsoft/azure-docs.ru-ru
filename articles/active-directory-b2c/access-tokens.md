---
title: Запрос маркера доступа в Azure Active Directory B2C | Документация Майкрософт
description: Узнайте, как запросить маркер доступа из Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 10/26/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: e5168d5e5e3935da267fb26f38735a88bdfd7837
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101654482"
---
# <a name="request-an-access-token-in-azure-active-directory-b2c"></a>Запрос маркера доступа в Azure Active Directory B2C

*Маркер доступа* содержит утверждения, которые можно использовать в Azure Active Directory B2C (Azure AD B2C) для определения разрешений, предоставленных API-интерфейсам. При вызове сервера ресурсов маркер доступа должен присутствовать в HTTP-запросе. В ответах из Azure AD B2C маркер доступа обозначается как **access_token**.

В этой статье содержатся сведения о запросе маркер доступа для веб-приложения и веб-API. Дополнительные сведения о маркерах в Azure AD B2C см. в статье с [обзором маркеров в Azure Active Directory B2C](tokens-overview.md).

> [!NOTE]
> **Цепочки веб-API ("от имени") не поддерживаются в Azure AD B2C.** Многие архитектуры включают в себя веб-API, которому требуется вызывать другой нисходящий веб-API. При этом Azure AD B2C защищает оба интерфейса. Это стандартный сценарий в клиентах, имеющих серверный веб-API, который, в свою очередь, вызывает другую службу. Этот сценарий веб-API с цепочками может поддерживаться с помощью предоставления учетных данных носителя маркера JWT OAuth 2.0 или потока On-Behalf-Of. Однако в Azure AD B2C поток On-Behalf-Of еще не реализован.

## <a name="prerequisites"></a>Предварительные требования

- [Создайте поток пользователя](tutorial-create-user-flows.md), чтобы пользователи могли зарегистрироваться и войти в ваше приложение.
- [Добавьте приложение веб-API приложения в клиент Azure Active Directory B2C](add-web-api-application.md), если это еще не сделано.

## <a name="scopes"></a>Области действия

Области предоставляют способ управления разрешениями для защищенных ресурсов. При запросе маркера доступа клиентскому приложению необходимо указать нужные разрешения в параметре **scope** запроса. Например, чтобы указать **значение области** `read` для **API с URI идентификатора приложения** `https://contoso.onmicrosoft.com/api` используется область `https://contoso.onmicrosoft.com/api/read`.

Области используются веб-API для реализации управления доступом на уровне области. Например, пользователи веб-API могут иметь доступ на чтение и запись или доступ только на чтение. Чтобы получить несколько разрешений в одном запросе, можно добавить несколько записей в одном параметре **scope** запроса, разделив их пробелами.

В следующем примере показаны области, декодированные в URL-адресе:

```
scope=https://contoso.onmicrosoft.com/api/read openid offline_access
```

В следующем примере показаны области, закодированные в URL-адресе:

```
scope=https%3A%2F%2Fcontoso.onmicrosoft.com%2Fapi%2Fread%20openid%20offline_access
```

Если число запрашиваемых областей превышает число областей, предоставленных клиентскому приложению, вызов будет выполнен в случае предоставления по меньшей мере одного разрешения. Утверждение **scp** полученного маркера заполняется только успешно предоставленными разрешениями. 

### <a name="openid-connect-scopes"></a>Области OpenId Connect

Стандартное утверждение OpenID Connect определяет несколько специальных значений области. Следующие области представляют разрешение на доступ к профилю пользователя:

- **openid** — запрашивает маркер идентификатора.
- **offline_access** — запрашивает маркер обновления с помощью [потоков кода аутентификации](authorization-code-flow.md).
- **00000000-0000-0000-0000-000000000000** — использование идентификатора клиента в качестве области означает, что приложению требуется маркер доступа, который можно использовать для собственной службы или веб-API, представленного одним и тем же идентификатором клиента.

Если параметр **response_type** в запросе `/authorize` включает `token`, параметр **scope** должен содержать хотя бы одну область ресурса (отличную от `openid` и `offline_access`), которая будет предоставлена. В противном случае запрос `/authorize` завершается ошибкой.

## <a name="request-a-token"></a>Запрос маркера

Для запроса маркера доступа необходим код авторизации. Ниже приведен пример запроса кода авторизации к конечной точке `/authorize`. Использование личных доменов с маркерами доступа не поддерживается. В URL-адресе запроса следует использовать домен tenant-name.onmicrosoft.com.

В следующем примере замените следующие значения.

- `<tenant-name>` — имя вашего клиента Azure Active Directory B2C.
- `<policy-name>` — имя вашей пользовательской политики или потока пользователя.
- `<application-ID>` — идентификатор веб-приложения, которое вы зарегистрировали для поддержки потока пользователя.
- `<redirect-uri>` — **URI перенаправления**, который вы ввели при регистрации клиентского приложения.

```http
GET https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/authorize?
client_id=<application-ID>
&nonce=anyRandomValue
&redirect_uri=https://jwt.ms
&scope=https://<tenant-name>.onmicrosoft.com/api/read
&response_type=code
```

Ответ с кодом авторизации должен быть аналогичен следующему примеру:

```
https://jwt.ms/?code=eyJraWQiOiJjcGltY29yZV8wOTI1MjAxNSIsInZlciI6IjEuMC...
```

После успешного получения кода авторизации его можно использовать для запроса маркера доступа:

```http
POST <tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/token HTTP/1.1
Host: <tenant-name>.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_id=<application-ID>
&scope=https://<tenant-name>.onmicrosoft.com/api/read
&code=eyJraWQiOiJjcGltY29yZV8wOTI1MjAxNSIsInZlciI6IjEuMC...
&redirect_uri=https://jwt.ms
&client_secret=2hMG2-_:y12n10vwH...
```

Должен отобразиться результат, аналогичный следующему ответу:

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrN...",
    "token_type": "Bearer",
    "not_before": 1549647431,
    "expires_in": 3600,
    "expires_on": 1549651031,
    "resource": "f2a76e08-93f2-4350-833c-965c02483b11",
    "profile_info": "eyJ2ZXIiOiIxLjAiLCJ0aWQiOiJjNjRhNGY3ZC0zMDkxLTRjNzMtYTcyMi1hM2YwNjk0Z..."
}
```

При использовании https://jwt.ms для проверки возвращенного маркера доступа вы должны увидеть примерно следующий результат:

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dl..."
}.{
  "iss": "https://contoso0926tenant.b2clogin.com/c64a4f7d-3091-4c73-a7.../v2.0/",
  "exp": 1549651031,
  "nbf": 1549647431,
  "aud": "f2a76e08-93f2-4350-833c-965...",
  "oid": "1558f87f-452b-4757-bcd1-883...",
  "sub": "1558f87f-452b-4757-bcd1-883...",
  "name": "David",
  "tfp": "B2C_1_signupsignin1",
  "nonce": "anyRandomValue",
  "scp": "read",
  "azp": "38307aee-303c-4fff-8087-d8d2...",
  "ver": "1.0",
  "iat": 1549647431
}.[Signature]
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [настраивать маркеры в Azure AD B2C](configure-tokens.md).
