---
title: Поток On-Behalf-Of в OAuth 2.0 и платформа удостоверений Майкрософт | Azure
titleSuffix: Microsoft identity platform
description: В этой статье описывается, как использовать HTTP-сообщения для проверки подлинности между службами с помощью потока On-Behalf-Of в OAuth 2.0.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/7/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: ff8e03b813e2cb890192667e3466d920eaabc72c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98756087"
---
# <a name="microsoft-identity-platform-and-oauth-20-on-behalf-of-flow"></a>Поток On-Behalf-Of в OAuth 2.0 и платформа удостоверений Майкрософт


Поток On-Behalf-Of (OBO) в OAuth 2.0 используется в том случае, когда приложение вызывает службу или веб-API, который, в свою очередь, должен вызывать другую службу или веб-API. Идея состоит в том, чтобы распространить делегированное удостоверение пользователя и разрешения с помощью цепочки запросов. Для того чтобы служба среднего уровня могла выполнять запросы к службе нижнего уровня с проверкой подлинности, служба среднего уровня должна защитить маркер доступа с платформы удостоверений Microsoft от имени пользователя.

В этой статье описывается, как программировать непосредственно протокол в приложении.  По возможности рекомендуется использовать поддерживаемые библиотеки проверки подлинности Майкрософт (MSAL) вместо [получения маркеров и вызова защищенных веб-API](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows).  Также ознакомьтесь с [примерами приложений, которые используют MSAL](sample-v2-code.md).


По состоянию на май 2018 года какой-либо производный маркер неявного потока `id_token` не может использоваться для потока OBO. Вместо этого одностраничные приложения (SPA) должны передавать свои маркеры **доступа** в конфиденциальный клиент среднего уровня для выполнения потоков OBO. Дополнительные сведения о том, какие клиенты могут выполнять вызовы OBO, см. в разделе об [ограничениях](#client-limitations).

## <a name="protocol-diagram"></a>Схема протокола

Предположим, что пользователь прошел проверку подлинности для приложения с использованием [потока для предоставления кода проверки подлинности в OAuth 2.0](v2-oauth2-auth-code-flow.md) или другого потока входа. На этом этапе приложение содержит маркер доступа для *API А* (маркер A) с утверждениями пользователя и разрешением на доступ к веб-API среднего уровня (API A). Теперь API A должен выполнить запрос к веб-API нижнего уровня (API B) с проверкой подлинности.

Последующие шаги образуют поток OBO. Эти шаги поясняются на следующей схеме.

![Показывает поток On-Behalf-Of в OAuth 2.0](./media/v2-oauth2-on-behalf-of-flow/protocols-oauth-on-behalf-of-flow.png)

1. Клиентское приложение отправляет запрос к API A с маркером A (с утверждением `aud` API А).
1. API A выполняет проверку подлинности на конечной точке выдачи маркера платформы удостоверений Майкрософт и запрашивает маркер доступа к API B.
1. Конечная точка выдачи маркера платформы удостоверений Майкрософт проверяет учетные данные API A и маркер A и выдает маркера доступа для API B (маркер B) к API A.
1. Маркер B задается API A в заголовке авторизации запроса к API B.
1. Данные из защищенного ресурса возвращаются API B в API A, а затем — клиенту.

В этом сценарии служба среднего уровня не взаимодействует с пользователем, чтобы получить согласие пользователя на доступ к подчиненному API. Поэтому согласие на доступ к API нижнего уровня необходимо получить заранее при принятии условий во время проверки подлинности. Чтобы узнать, как настроить это для приложения, ознакомьтесь с разделом [Получение согласия для приложения среднего уровня](#gaining-consent-for-the-middle-tier-application).

## <a name="middle-tier-access-token-request"></a>Запрос маркера доступа среднего уровня

Чтобы запросить маркер доступа, отправьте HTTP-запрос POST к конечной точке маркера платформы удостоверений Майкрософт для конкретного клиента с указанными ниже параметрами.

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/token
```

Существуют два сценария. Их выбор зависит от типа защиты клиентского приложения — с помощью общего секрета или с помощью сертификата.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Первый вариант. Запрос маркера доступа с помощью общего секрета

При использовании общего секрета запрос маркера взаимного доступа между службами содержит следующие параметры:

| Параметр | Type | Описание |
| --- | --- | --- |
| `grant_type` | Обязательно | Тип запроса маркера. Для запроса с использованием JWT нужно указать значение `urn:ietf:params:oauth:grant-type:jwt-bearer`. |
| `client_id` | Обязательно | Идентификатор приложения (клиента), назначенный вашему приложению на странице [регистрации приложений на портале Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `client_secret` | Обязательно | Секрет клиента, созданный для приложения на портале Azure — страница регистрации приложений. |
| `assertion` | Обязательно | Маркер доступа, который был отправлен в API среднего уровня.  Этот маркер должен содержать утверждение аудитории ( `aud` ) приложения, выполняющего этот запрос OBO (приложение, обозначенное `client-id` полем). Приложения не могут активировать токен для другого приложения (например, если клиент отправляет API маркер, предназначенный для MS Graph, API не может активировать его с помощью OBO.  Вместо этого маркер должен отклоняться.  |
| `scope` | Обязательно | Список областей для запроса токена, разделенный пробелами. Дополнительные сведения см. в разделе [Области](v2-permissions-and-consent.md). |
| `requested_token_use` | Обязательно | Указывает, как должен быть обработан запрос. В потоке OBO нужно указать значение `on_behalf_of`. |

#### <a name="example"></a>Пример

Следующий HTTP-запрос POST запрашивает маркер и маркер обновления доступа с областью `user.read` для веб-API https://graph.microsoft.com.

```HTTP
//line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com/<tenant>
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&client_id=2846f71b-a7a4-4987-bab3-760035b2f389
&client_secret=BYyVnAt56JpLwUcyo47XODd
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiIyO{a lot of characters here}
&scope=https://graph.microsoft.com/user.read+offline_access
&requested_token_use=on_behalf_of
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Второй вариант. Запрос маркера доступа с помощью сертификата

Запрос маркера взаимного доступа между службами с помощью сертификата содержит следующие параметры:

| Параметр | Type | Описание |
| --- | --- | --- |
| `grant_type` | Обязательно | Тип запроса токена. Для запроса с использованием JWT нужно указать значение `urn:ietf:params:oauth:grant-type:jwt-bearer`. |
| `client_id` | Обязательно |  Идентификатор приложения (клиента), назначенный вашему приложению на странице [регистрации приложений на портале Azure](https://go.microsoft.com/fwlink/?linkid=2083908). |
| `client_assertion_type` | Обязательно | Значение должно быть равно `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| `client_assertion` | Обязательно | Утверждение (JSON Web Token), которое необходимо создать и подписать с помощью сертификата, зарегистрированного как учетные данные для приложения. Ознакомьтесь с информацией об [учетных данных сертификата](active-directory-certificate-credentials.md), чтобы узнать, как зарегистрировать сертификат и задать формат утверждения. |
| `assertion` | Обязательно |  Маркер доступа, который был отправлен в API среднего уровня.  Этот маркер должен содержать утверждение аудитории ( `aud` ) приложения, выполняющего этот запрос OBO (приложение, обозначенное `client-id` полем). Приложения не могут активировать токен для другого приложения (например, если клиент отправляет API маркер, предназначенный для MS Graph, API не может активировать его с помощью OBO.  Вместо этого маркер должен отклоняться.  |
| `requested_token_use` | Обязательно | Указывает, как должен быть обработан запрос. В потоке OBO нужно указать значение `on_behalf_of`. |
| `scope` | Обязательно | Список областей для запроса маркера, разделенный пробелами. Дополнительные сведения см. в разделе [Области](v2-permissions-and-consent.md).|

Обратите внимание на то, что параметры являются почти такими же, как и при использовании запроса с помощью общего секрета, за исключением параметра `client_secret`, который заменяется двумя параметрами: `client_assertion_type` и `client_assertion`.

#### <a name="example"></a>Пример

Ниже приведен HTTP-запрос POST маркера доступа с областью `user.read` для веб-API https://graph.microsoft.com с сертификатом.

```HTTP
// line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com/<tenant>
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiO{a lot of characters here}
&requested_token_use=on_behalf_of
&scope=https://graph.microsoft.com/user.read+offline_access
```

## <a name="middle-tier-access-token-response"></a>Ответ маркера доступа среднего уровня

Если доступ предоставлен, ответ будет содержать JSON-файл OAuth 2.0 со следующими параметрами.

| Параметр | Описание |
| --- | --- |
| `token_type` | Указывает значение типа маркера. Единственным типом, поддерживаемым платформой Microsoft Identity, является `Bearer` . Дополнительные сведения о маркерах носителей см. в спецификации [OAuth 2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (OAuth2.0 Authorization Framework: использование маркера носителя (RFC 6750)). |
| `scope` | Область доступа, предоставляемая токеном. |
| `expires_in` | Срок действия маркера доступа (в секундах). |
| `access_token` | Запрашиваемый маркер доступа. Вызывающая служба может использовать этот токен для проверки подлинности принимающей службы. |
| `refresh_token` | Токен обновления для запрошенного токена доступа. Вызывающая служба может использовать этот токен для запроса другого токена доступа после того, как срок действия текущего токена доступа истек. Маркер обновления предоставляется только в том случае, если запрошена область `offline_access`. |

### <a name="success-response-example"></a>Пример ответа с успешным предоставлением доступа

В следующем примере показано сообщение о предоставлении доступа в ответ на запрос маркера доступа к веб-API https://graph.microsoft.com.

```json
{
  "token_type": "Bearer",
  "scope": "https://graph.microsoft.com/user.read",
  "expires_in": 3269,
  "ext_expires_in": 0,
  "access_token": "eyJ0eXAiOiJKV1QiLCJub25jZSI6IkFRQUJBQUFBQUFCbmZpRy1tQTZOVGFlN0NkV1c3UWZkQ0NDYy0tY0hGa18wZE50MVEtc2loVzRMd2RwQVZISGpnTVdQZ0tQeVJIaGlDbUN2NkdyMEpmYmRfY1RmMUFxU21TcFJkVXVydVJqX3Nqd0JoN211eHlBQSIsImFsZyI6IlJTMjU2IiwieDV0IjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIiwia2lkIjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIn0.eyJhdWQiOiJodHRwczovL2dyYXBoLm1pY3Jvc29mdC5jb20iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaWF0IjoxNDkzOTMwMzA1LCJuYmYiOjE0OTM5MzAzMDUsImV4cCI6MTQ5MzkzMzg3NSwiYWNyIjoiMCIsImFpbyI6IkFTUUEyLzhEQUFBQU9KYnFFWlRNTnEyZFcxYXpKN1RZMDlYeDdOT29EMkJEUlRWMXJ3b2ZRc1k9IiwiYW1yIjpbInB3ZCJdLCJhcHBfZGlzcGxheW5hbWUiOiJUb2RvRG90bmV0T2JvIiwiYXBwaWQiOiIyODQ2ZjcxYi1hN2E0LTQ5ODctYmFiMy03NjAwMzViMmYzODkiLCJhcHBpZGFjciI6IjEiLCJmYW1pbHlfbmFtZSI6IkNhbnVtYWxsYSIsImdpdmVuX25hbWUiOiJOYXZ5YSIsImlwYWRkciI6IjE2Ny4yMjAuMC4xOTkiLCJuYW1lIjoiTmF2eWEgQ2FudW1hbGxhIiwib2lkIjoiZDVlOTc5YzctM2QyZC00MmFmLThmMzAtNzI3ZGQ0YzJkMzgzIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTIxMjc1MjExODQtMTYwNDAxMjkyMC0xODg3OTI3NTI3LTI2MTE4NDg0IiwicGxhdGYiOiIxNCIsInB1aWQiOiIxMDAzM0ZGRkEwNkQxN0M5Iiwic2NwIjoiVXNlci5SZWFkIiwic3ViIjoibWtMMHBiLXlpMXQ1ckRGd2JTZ1JvTWxrZE52b3UzSjNWNm84UFE3alVCRSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInVuaXF1ZV9uYW1lIjoibmFjYW51bWFAbWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hY2FudW1hQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJWR1ItdmtEZlBFQ2M1dWFDaENRSkFBIiwidmVyIjoiMS4wIn0.cubh1L2VtruiiwF8ut1m9uNBmnUJeYx4x0G30F7CqSpzHj1Sv5DCgNZXyUz3pEiz77G8IfOF0_U5A_02k-xzwdYvtJUYGH3bFISzdqymiEGmdfCIRKl9KMeoo2llGv0ScCniIhr2U1yxTIkIpp092xcdaDt-2_2q_ql1Ha_HtjvTV1f9XR3t7_Id9bR5BqwVX5zPO7JMYDVhUZRx08eqZcC-F3wi0xd_5ND_mavMuxe2wrpF-EZviO3yg0QVRr59tE3AoWl8lSGpVc97vvRCnp4WVRk26jJhYXFPsdk4yWqOKZqzr3IFGyD08WizD_vPSrXcCPbZP3XWaoTUKZSNJg",
  "refresh_token": "OAQABAAAAAABnfiG-mA6NTae7CdWW7QfdAALzDWjw6qSn4GUDfxWzJDZ6lk9qRw4An{a lot of characters here}"
}
```

Приведенный выше маркер доступа — это маркер в формате v 1.0 для Microsoft Graph. Это обусловлено тем, что формат маркера основан на **ресурсе** , к которому осуществляется доступ, и не связан с конечными точками, используемыми для запроса. Microsoft Graph настроен на принятие маркеров версии 1.0, поэтому платформа Microsoft Identity создает маркеры доступа v 1.0, когда клиент запрашивает маркеры для Microsoft Graph. Другие приложения могут указывать, что им требуются маркеры формата версии 2.0, маркеры формата v 1.0 или даже собственные или зашифрованные форматы маркеров.  Конечные точки версии 1.0 и v 2.0 могут выдавать любой формат маркера. Таким образом, ресурс всегда может получить правильный формат маркера независимо от того, как или где маркер был запрошен клиентом. 

Только приложениям следует просматривать маркеры доступа. Клиенты **не должны** проверять их. Проверка маркеров доступа для других приложений в коде приведет к неожиданному прерыванию работы приложения, когда приложение изменит формат маркеров или начнет их шифрование. 

### <a name="error-response-example"></a>Пример ответа с сообщением об ошибке

Сообщение об ошибке возвращается конечной точкой маркера при попытке получить маркер доступа для нисходящего API в том случае, если для нисходящего API настроена политика условного доступа, например [многофакторная проверка подлинности](../authentication/concept-mfa-howitworks.md). Служба среднего уровня должна передать эту ошибку в клиентское приложение, чтобы пользователь мог выполнить необходимые действия для соблюдения политики условного доступа.

```json
{
    "error":"interaction_required",
    "error_description":"AADSTS50079: Due to a configuration change made by your administrator, or because you moved to a new location, you must enroll in multi-factor authentication to access 'bf8d80f9-9098-4972-b203-500f535113b1'.\r\nTrace ID: b72a68c3-0926-4b8e-bc35-3150069c2800\r\nCorrelation ID: 73d656cf-54b1-4eb2-b429-26d8165a52d7\r\nTimestamp: 2017-05-01 22:43:20Z",
    "error_codes":[50079],
    "timestamp":"2017-05-01 22:43:20Z",
    "trace_id":"b72a68c3-0926-4b8e-bc35-3150069c2800",
    "correlation_id":"73d656cf-54b1-4eb2-b429-26d8165a52d7",
    "claims":"{\"access_token\":{\"polids\":{\"essential\":true,\"values\":[\"9ab03e19-ed42-4168-b6b7-7001fb3e933a\"]}}}"
}
```

## <a name="use-the-access-token-to-access-the-secured-resource"></a>Использование токена доступа для доступа к защищенному ресурсу

Теперь служба среднего уровня может использовать полученный выше токен для запросов к веб-API нижнего уровня с проверкой подлинности путем установки токена в заголовке `Authorization`.

### <a name="example"></a>Пример

```HTTP
GET /v1.0/me HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiO ... 0X2tnSQLEANnSPHY0gKcgw
```

## <a name="saml-assertions-obtained-with-an-oauth20-obo-flow"></a>Утверждения SAML, полученные с помощью потока OBO в OAuth2.0

Некоторые веб-службы на основе OAuth должны получать доступ к другим API веб-служб, получающим утверждения SAML в неинтерактивных потоках. Azure Active Directory может предоставить утверждение SAML в ответ на поток On-Behalf-Of с использованием веб-службы на основе SAML в качестве целевого ресурса.

Это нестандартное расширение для потока On-Behalf-Of в OAuth 2.0, позволяющее приложению на основе OAuth2 получать доступ к конечным точкам API веб-служб, использующим токены SAML.

> [!TIP]
> Чтобы вызвать защищенную с помощью SAML веб-службу из веб-приложения переднего плана, достаточно вызвать API и инициировать обычный интерактивный поток проверки подлинности в имеющемся сеансе пользователя. Если для предоставления контекста пользователя вызову между службами необходим токен SAML, то достаточно использовать поток OBO.

## <a name="gaining-consent-for-the-middle-tier-application"></a>Получение согласия для приложения среднего уровня

В зависимости от архитектуры приложения можно использовать разные стратегии, обеспечивающие успешную работу потока OBO. Во всех случаях конечная цель заключается в обеспечении правильного согласия, чтобы клиентское приложение могло вызвать приложение среднего уровня, а приложение среднего уровня имело разрешение на вызов ресурса серверной части.

> [!NOTE]
> Ранее система учетных записей Майкрософт (личные учетные записи) не поддерживала поле "Известное клиентское приложение" и не могла отображать комбинированное согласие.  Эта функция была добавлена, и все приложения на платформе удостоверений Майкрософт могут использовать подход "Известное клиентское приложение" для получения согласия на вызовы OBO.

### <a name="default-and-combined-consent"></a>Область /.default и объединенное согласие

Приложение среднего уровня добавляет клиент в список известных клиентских приложений в своем манифесте, после чего клиент может активировать поток объединенного согласия как для себя, так и для приложения среднего уровня. На платформе Microsoft Identity это делается с помощью [ `/.default` области](v2-permissions-and-consent.md#the-default-scope). При активации процесса известных клиентских приложений и `/.default` на экране получения согласия отображаются разрешения клиента и API среднего уровня **и** запрос разрешений, необходимых для API среднего уровня. Пользователь дает согласие для обоих приложений, после чего выполняется поток OBO.

### <a name="pre-authorized-applications"></a>Предварительно авторизованные приложения

Ресурс может указать, что у заданного приложения всегда есть разрешение на прием определенных областей. Это особенно удобно для упрощения подключений между интерфейсным клиентом и внутренним ресурсом. Ресурс может объявить несколько предварительно авторизованных приложений. Любое из них может запросить соответствующие разрешения в потоке OBO и получить их без согласия пользователя.

### <a name="admin-consent"></a>Согласие администратора

Администратор клиента может обеспечить приложения разрешениями для вызова необходимых им API, предоставив согласие администратора для приложения среднего уровня. Для этого администратор может найти приложение среднего уровня в своем клиенте, открыть страницу необходимых разрешений и выбрать предоставляемые разрешения для приложения. Дополнительные сведения о согласии администратора доступны в [документации о согласии и разрешениях](v2-permissions-and-consent.md).

### <a name="use-of-a-single-application"></a>Использование одного приложения

В некоторых сценариях может использоваться только одна пара приложения среднего уровня и интерфейсного клиента. В этом случае вам может быть проще создать одно приложение, избавившись от потребности в приложении среднего уровня. Для аутентификации между внешним интерфейсом и веб-API можно использовать файлы cookie, маркер id_token или маркер доступа, запрошенный для приложения. Затем можно запросить согласие у этого приложения для доступа к внутреннему ресурсу.

## <a name="client-limitations"></a>Ограничения клиентов

Если клиент использует неявный поток для получения id_token, а его URL-адрес ответа содержит подстановочные знаки, то этот id_token не может использоваться для потока OBO.  Тем не менее, конфиденциальный клиент может активировать маркеры доступа, полученные с помощью потока неявного предоставления, даже если для инициирующего клиента зарегистрирован URL-адрес ответа с подстановочными знаками.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о протоколе OAuth 2.0 и другом способе проверки подлинности между службами с использованием учетных данных клиента.

* [Предоставление учетных данных клиента OAuth 2.0 на платформе удостоверений Майкрософт](v2-oauth2-client-creds-grant-flow.md)
* [Поток кода OAuth 2.0 на платформе удостоверений Майкрософт](v2-oauth2-auth-code-flow.md)
* [Использование области `/.default`](v2-permissions-and-consent.md#the-default-scope)
