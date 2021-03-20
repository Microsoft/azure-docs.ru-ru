---
title: Управление сеансами единого входа с помощью пользовательских политик
titleSuffix: Azure AD B2C
description: Узнайте, как для управлять сеансами единого входа с помощью настраиваемых политик в Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/07/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: f690f4a416e86b02de0d35fc673849c1293df577
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102095771"
---
# <a name="single-sign-on-session-management-in-azure-active-directory-b2c"></a>Управление сеансами единого входа в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Управление [сеансами единого входа (SSO)](session-behavior.md) использует ту же семантику, что и любой другой технический профиль в пользовательских политиках. При выполнении шага оркестрации из технического профиля, связанного с шагом, запрашиваются ссылка на `UseTechnicalProfileForSessionManagement`. Затем, при наличии такового, указанный поставщик сеансов единого входа проверяется, чтобы определить, является ли пользователь участником сеанса. Если это так, то поставщик сеансов единого входа используется для повторного заполнения данных сеанса. Аналогичным образом после выполнения шага оркестрации поставщик используется для сохранения сведений в сеансе, если был указан поставщик сеансов единого входа.

В Azure AD B2C можно использовать определенное количество поставщиков сеансов единого входа.

|Поставщик сеанса  |Область  |
|---------|---------|
|[NoopSSOSessionProvider](#noopssosessionprovider)     |  Нет       |       
|[DefaultSSOSessionProvider](#defaultssosessionprovider)    | Azure AD B2C диспетчер внутренних сеансов.      |       
|[ExternalLoginSSOSessionProvider](#externalloginssosessionprovider)     | Между Azure AD B2C и OAuth1, OAuth2 или OpenID Connect Connect Identity Provider.        |         |
|[OAuthSSOSessionProvider](#oauthssosessionprovider)     | Между приложением проверяющей стороны OAuth2 или OpenID Connect и Azure AD B2C.        |        
|[SamlSSOSessionProvider](#samlssosessionprovider)     | Между Azure AD B2C и поставщиком удостоверений SAML. И между поставщиком службы SAML (приложение проверяющей стороны) и Azure AD B2C.  |        




Классы управления единым входом указываются с помощью элемента `<UseTechnicalProfileForSessionManagement ReferenceId="{ID}" />` технического профиля.

## <a name="input-claims"></a>Входящие утверждения

`InputClaims`Элемент пуст или отсутствует.

## <a name="persisted-claims"></a>Материализованные утверждения

Утверждения, которые должны быть возвращены в приложение или использоваться предусловиями в последующих шагах, должны храниться в сеансе или дополняться считыванием из профиля пользователя в каталоге. Использование материализованных утверждений гарантирует, что пути проверки подлинности не будут завершаться сбоем в случае отсутствия утверждений. Чтобы добавить утверждения в сеанс, используйте элемент `<PersistedClaims>` технического профиля. При использовании этого поставщика для повторного заполнения данных сеанса сохраненные утверждения добавляются в контейнер утверждений.

## <a name="output-claims"></a>Исходящие утверждения

`<OutputClaims>`Используется для получения утверждений из сеанса.

## <a name="session-providers"></a>Поставщики сеансов

### <a name="noopssosessionprovider"></a>NoopSSOSessionProvider

Как можно понять из названия, этот поставщик не выполняет никаких действий. Его можно использовать для подавления реакции службы единого входа на события для конкретного технического профиля. Следующий `SM-Noop` технический профиль включен в [начальный пакет пользовательской политики](custom-policy-get-started.md#custom-policy-starter-pack).

```xml
<TechnicalProfile Id="SM-Noop">
  <DisplayName>Noop Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.NoopSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</TechnicalProfile>
```

### <a name="defaultssosessionprovider"></a>DefaultSSOSessionProvider

Этот поставщик можно использовать для хранения утверждений в сеансе. Этот поставщик обычно упоминается в техническом профиле, который используется для управления локальными и федеративными учетными записями. Следующий `SM-AAD` технический профиль включен в [начальный пакет пользовательской политики](custom-policy-get-started.md#custom-policy-starter-pack).

```xml
<TechnicalProfile Id="SM-AAD">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.DefaultSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="objectId" />
    <PersistedClaim ClaimTypeReferenceId="signInName" />
    <PersistedClaim ClaimTypeReferenceId="authenticationSource" />
    <PersistedClaim ClaimTypeReferenceId="identityProvider" />
    <PersistedClaim ClaimTypeReferenceId="newUser" />
    <PersistedClaim ClaimTypeReferenceId="executed-SelfAsserted-Input" />
  </PersistedClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectIdFromSession" DefaultValue="true"/>
  </OutputClaims>
</TechnicalProfile>
```


Следующий `SM-MFA` технический профиль включен в [начальный пакет пользовательской политики](custom-policy-get-started.md#custom-policy-starter-pack) `SocialAndLocalAccountsWithMfa` . Этот технический профиль управляет сеансом многофакторной проверки подлинности.

```xml
<TechnicalProfile Id="SM-MFA">
  <DisplayName>Session Mananagement Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.DefaultSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="Verified.strongAuthenticationPhoneNumber" />
  </PersistedClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isActiveMFASession" DefaultValue="true"/>
  </OutputClaims>
</TechnicalProfile>
```

### <a name="externalloginssosessionprovider"></a>ExternalLoginSSOSessionProvider

Этот поставщик используется для подавления экрана "Выбор поставщика удостоверений" и выхода из федеративного поставщика удостоверений. Он обычно упоминается в техническом профиле, настроенном для федеративного поставщика удостоверений, например Facebook, или Azure Active Directory. Следующий `SM-SocialLogin` технический профиль включен в [начальный пакет пользовательской политики](custom-policy-get-started.md#custom-policy-starter-pack).

```xml
<TechnicalProfile Id="SM-SocialLogin">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.ExternalLoginSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="AlternativeSecurityId" />
  </PersistedClaims>
</TechnicalProfile>
```

#### <a name="metadata"></a>Метаданные

| attribute | Обязательно | Описание|
| --- | --- | --- |
| алвайсфетчклаимсфромпровидер | Нет | В настоящее время не используется, может игнорироваться. |

### <a name="oauthssosessionprovider"></a>OAuthSSOSessionProvider

Этот поставщик используется для управления Azure AD B2C сеансами между OAuth2 или OpenID Connect и Azure AD B2C.

```xml
<TechnicalProfile Id="SM-jwt-issuer">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.OAuthSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</TechnicalProfile>
```

### <a name="samlssosessionprovider"></a>SamlSSOSessionProvider

Этот поставщик используется для управления Azure AD B2C сеансов SAML между приложением проверяющей стороны или федеративным поставщиком удостоверений SAML. При использовании поставщика единого входа для хранения сеанса поставщика удостоверений SAML `RegisterServiceProviders` необходимо установить значение `false` . `SM-Saml-idp` [Поставщик удостоверений SAML](identity-provider-generic-saml.md)использует следующий технический профиль.

```xml
<TechnicalProfile Id="SM-Saml-idp">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="RegisterServiceProviders">false</Item>
  </Metadata>
</TechnicalProfile>
```

При использовании поставщика для хранения сеанса SAML B2C параметр `RegisterServiceProviders` должен иметь значение `true` . Для завершения выхода из сеанса SAML требуются `SessionIndex` и `NameID`.

`SM-Saml-issuer` [Технический профиль издателя SAML](saml-service-provider.md) использует следующий технический профиль

```xml
<TechnicalProfile Id="SM-Saml-issuer">
  <DisplayName>Session Management Provider</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"/>
</TechnicalProfile>
```

#### <a name="metadata"></a>Метаданные

| attribute | Обязательно | Описание|
| --- | --- | --- |
| IncludeSessionIndex | Нет | В настоящее время не используется, может игнорироваться.|
| RegisterServiceProviders | Нет | Указывает, что поставщик должен зарегистрировать все поставщики услуг SAML, которыми было выдано утверждение. Возможные значения: `true` (по умолчанию) или `false`.|


## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [настроить поведение сеанса](session-behavior.md).
