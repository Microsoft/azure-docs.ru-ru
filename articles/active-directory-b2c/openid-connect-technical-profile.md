---
title: Определение технического профиля OpenID Connect Connect в пользовательской политике
titleSuffix: Azure AD B2C
description: Определите технический профиль OpenID Connect Connect в пользовательской политике в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/04/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 670fbeeb006d21e29675f88895018d1a453a1c54
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102120306"
---
# <a name="define-an-openid-connect-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Определение технического профиля OpenID Connect Connect в Azure Active Directory B2C настраиваемой политике

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) обеспечивает поддержку поставщика удостоверений протокола [OpenID Connect Connect](https://openid.net/2015/04/17/openid-connect-certification-program/) . OpenID Connect 1.0 определяет уровень идентификации поверх OAuth 2.0 и представляет собой оптимизированный протокол среди современных протоколов аутентификации. С помощью технического профиля OpenID Connect Connect можно создать федерацию с помощью поставщика удостоверений на основе OpenID Connect, например Azure AD. Федерацию с поставщиком удостоверений позволяет пользователям входить в систему, используя существующие социальные или корпоративные удостоверения.

## <a name="protocol"></a>Протокол

Атрибуту **Name** элемента **Protocol** необходимо присвоить значение `OpenIdConnect`. Например, `OpenIdConnect` — это протокол для технического профиля **MSA-OIDC**.

```xml
<TechnicalProfile Id="MSA-OIDC">
  <DisplayName>Microsoft Account</DisplayName>
  <Protocol Name="OpenIdConnect" />
  ...
```

## <a name="input-claims"></a>Входящие утверждения

Элементы **InputClaims** и **InputClaimsTransformations** не являются обязательными. Но при необходимости можно отправлять дополнительные параметры поставщику удостоверений. В следующем примере к запросу авторизации добавляется параметр строки запроса **domain_hint** со значением `contoso.com`.

```xml
<InputClaims>
  <InputClaim ClaimTypeReferenceId="domain_hint" DefaultValue="contoso.com" />
</InputClaims>
```

## <a name="output-claims"></a>Исходящие утверждения

Элемент **OutputClaims** содержит список заявок, возвращенных поставщиком удостоверений OpenID Connect Connect. Возможно, потребуется сопоставить имя утверждения, определенное в вашей политике, с именем, определенным у поставщика удостоверений. Вы также можете добавить утверждения, которые не возвращаются поставщиком удостоверений, установив атрибут `DefaultValue`.

Элемент **OutputClaimsTransformations** может содержать коллекцию элементов **OutputClaimsTransformation**, которые используются для изменения исходящих утверждений или создания новых.

В этом примере показаны утверждения, возвращаемые поставщиком удостоверений Microsoft Account:

- Подзапрос **,** сопоставленный с утверждением **issuerUserId** .
- утверждение **name**, которое сопоставляется с утверждением **displayName**;
- утверждение **email** без сопоставления с именем.

Технический профиль также возвращает утверждения, которые не возвращаются поставщиком удостоверений:

- Утверждение **IdentityProvider**, содержащее имя поставщика удостоверений.
- утверждение **AuthenticationSource** со значением по умолчанию **socialIdpAuthentication**.

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="live.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
  <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" />
</OutputClaims>
```

## <a name="metadata"></a>Метаданные

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| client_id | Да | Идентификатор приложения поставщика удостоверений. |
| IdTokenAudience | Нет | Аудитория id_token. Если указано, Azure AD B2C проверяет, `aud` равен ли утверждение в токене, возвращенном поставщиком удостоверений, указанному в метаданных идтокенаудиенце.  |
| METADATA | Да | URL-адрес, указывающий на документ конфигурации поставщика удостоверений OpenID Connect Connect, который также называется OpenID Connect хорошо известной конечной точкой конфигурации. URL-адрес может содержать `{tenant}` выражение, которое заменяется именем клиента.  |
| authorization_endpoint | Нет | URL-адрес, указывающий на конечную точку авторизации конфигурации поставщика удостоверений OpenID Connect Connect. Значение метаданных authorization_endpoint имеет приоритет над `authorization_endpoint` указанным в конечной точке конфигурации OpenID Connect. URL-адрес может содержать `{tenant}` выражение, которое заменяется именем клиента. |
| end_session_endpoint | Нет | URL-адрес конечной точки конечного сеанса. Значение метаданных authorization_endpoint имеет приоритет над `end_session_endpoint` указанным в конечной точке конфигурации OpenID Connect. |
| issuer | Нет | Уникальный идентификатор поставщика удостоверений OpenID Connect Connect. Значение метаданных издателя имеет приоритет над `issuer` указанным в конечной точке конфигурации OpenID Connect.  Если указано, Azure AD B2C проверяет, `iss` равен ли утверждение в токене, возвращенном поставщиком удостоверений, указанному в метаданных издателя. |
| ProviderName | Нет | Имя поставщика удостоверений.  |
| response_types | Нет | Тип ответа в соответствии со спецификацией OpenID Connect Core 1.0. Возможные значения: `id_token`, `code` или `token`. |
| response_mode | Нет | Метод, который использует поставщик удостоверений, чтобы отправить результат обратно в Azure AD B2C. Возможные значения: `query`, `form_post` (по умолчанию) или `fragment`. |
| область | Нет | Область запроса, определяемая в соответствии со спецификацией OpenID Connect Connect Core 1,0. Возможные значения: `openid`, `profile` и `email`. |
| HttpBinding | Нет | Ожидаемая привязка HTTP для маркера доступа и конечных точек маркера утверждений. Возможные значения: `GET` или `POST`.  |
| ValidTokenIssuerPrefixes | Нет | Ключ, который можно использовать для входа во все клиенты при использовании мультитенантного поставщика удостоверений, например Azure Active Directory. |
| UsePolicyInRedirectUri | Нет | Указывает, следует ли использовать политику при создании универсального кода ресурса (URI) перенаправления. При настройке приложения в поставщике удостоверений необходимо указать URI перенаправления. URI перенаправления указывает на Azure AD B2C, `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp` .  Если вы указываете `true`, необходимо добавить URI перенаправления для каждой используемой политики. Например: `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/{policy-name}/oauth2/authresp`. |
| MarkAsFailureOnStatusCode5xx | Нет | Указывает, должен ли запрос внешней службы помечаться как запрос, завершившийся сбоем, если код состояния Http находится в диапазоне 5xx. Значение по умолчанию — `false`. |
| DiscoverMetadataByTokenIssuer | Нет | Указывает, следует ли обнаруживать метаданные OIDC с помощью издателя в маркере JWT. |
| инклудеклаимресолвингинклаимшандлинг  | Нет | Для входных и выходных утверждений указывает, включено ли [разрешение утверждений](claim-resolver-overview.md) в технический профиль. Возможные значения: `true` или `false` (по умолчанию). Если вы хотите использовать сопоставитель утверждений в техническом профиле, задайте для этого параметра значение `true` . |
| token_endpoint_auth_method | Нет | Указывает, как Azure AD B2C отправляет заголовок проверки подлинности в конечную точку маркера. Возможные значения: `client_secret_post` (по умолчанию) и `client_secret_basic` (общедоступная Предварительная версия). Дополнительные сведения см. в [разделе OpenID Connect Connect Client Authentication](https://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication). |
| token_signing_algorithm | Нет | Алгоритм подписи, используемый для утверждений клиента, если для метаданных **token_endpoint_auth_method** задано значение `private_key_jwt` . Возможные значения: `RS256` (по умолчанию). |
| синглелогаутенаблед | Нет | Указывает, будет ли при входе в технический профиль попытаться выйти из федеративных поставщиков удостоверений. Дополнительные сведения см. в разделе [Azure AD B2C выхода из сеанса](./session-behavior.md#sign-out).  Возможные значения: `true` (по умолчанию) или `false` . |
|реадбодиклаимсонидпредирект| Нет| Задайте значение для `true` чтения утверждений из текста ответа при перенаправлении поставщика удостоверений. Эти метаданные используются с [идентификаторами Apple ID](identity-provider-apple-id.md), где утверждения возвращаются в полезных данных ответа.|

```xml
<Metadata>
  <Item Key="ProviderName">https://login.live.com</Item>
  <Item Key="METADATA">https://login.live.com/.well-known/openid-configuration</Item>
  <Item Key="response_types">code</Item>
  <Item Key="response_mode">form_post</Item>
  <Item Key="scope">openid profile email</Item>
  <Item Key="HttpBinding">POST</Item>
  <Item Key="UsePolicyInRedirectUri">false</Item>
  <Item Key="client_id">Your Microsoft application client ID</Item>
</Metadata>
```

### <a name="ui-elements"></a>Элементы пользовательского интерфейса
 
Следующие параметры можно использовать для настройки сообщения об ошибке, отображаемого при сбое. Метаданные должны быть настроены в техническом профиле OpenID Connect Connect. Сообщения об ошибках можно [локализовать](localization-string-ids.md#sign-up-or-sign-in-error-messages).

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| UserMessageIfClaimsPrincipalDoesNotExist | Нет | Сообщение, отображаемое пользователю, если учетная запись с указанным именем пользователя не найдена в каталоге. |
| UserMessageIfInvalidPassword | Нет | Сообщение, отображаемое пользователю, если пароль неверен. |
| UserMessageIfOldPasswordUsed| Нет |  Сообщение, отображаемое пользователю, если используется старый пароль.|

## <a name="cryptographic-keys"></a>Криптографические ключи

Элемент **CryptographicKeys** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| client_secret | Да | Секрет клиента приложения поставщика удостоверений. Этот криптографический ключ необходим только в том случае, если для метаданных **response_types** задано значение `code` , а **token_endpoint_auth_method** имеет значение `client_secret_post` или `client_secret_basic` . В этом случае Azure AD B2C выполняет другой вызов для обмена кода авторизации на маркер доступа. Если для метаданных задано значение `id_token`, криптографический ключ можно не указывать.  |
| assertion_signing_key | Да | Закрытый ключ RSA, который будет использоваться для подписи утверждения клиента. Этот криптографический ключ необходим только в том случае, если для метаданных **token_endpoint_auth_method** задано значение `private_key_jwt` . |

## <a name="redirect-uri"></a>URI перенаправления

При настройке URI перенаправления поставщика удостоверений введите `https://{your-tenant-name}.b2clogin.com/{your-tenant-name}.onmicrosoft.com/oauth2/authresp`. Обязательно замените `{your-tenant-name}` именем своего клиента. URI перенаправления должен содержать только строчные символы.

Примеры:

- [Azure Active Directory B2C. Добавление учетной записи Майкрософт (MSA) в качестве поставщика удостоверений с помощью пользовательских политик](identity-provider-microsoft-account.md)
- [Azure Active Directory B2C. Выполнение входа с помощью учетных записей Azure AD](identity-provider-azure-ad-single-tenant.md)
- [Azure Active Directory B2C. Предоставление пользователям возможности входить в мультитенантный поставщик удостоверений Azure AD с помощью пользовательских политик](identity-provider-azure-ad-multi-tenant.md)
