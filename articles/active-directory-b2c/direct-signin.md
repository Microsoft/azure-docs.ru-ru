---
title: Настройка прямого входа в систему с помощью Azure Active Directory B2C | Документация Майкрософт
description: Узнайте, как настроить предварительное заполнение имени для входа или перенаправление непосредственно в поставщик удостоверений в социальных сетях.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/14/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 35e8efa269ab72477b06e86824d368d0a3dced03
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103197329"
---
# <a name="set-up-direct-sign-in-using-azure-active-directory-b2c"></a>Настройка прямого входа в систему с помощью Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

При настройке входа для приложения с помощью Azure Active Directory (AD) B2C вы можете настроить предварительное заполнение имени для входа или прямой вход в поставщик удостоверений в определенных социальных сетях, например Facebook или LinkedIn, или учетной записи Майкрософт.

## <a name="prepopulate-the-sign-in-name"></a>Предварительное заполнение имени входа в систему

Во время входа пользователя в систему приложение проверяющей стороны может предложить определенное имя пользователя или домена. При этом приложение может указать параметр запроса `login_hint` с именем пользователя в запросе на авторизацию. Azure AD B2C автоматически заполняет имя для входа в систему — пользователю нужно ввести только пароль.

![Страница входа с login_hint параметром запроса, выделенным в URL-адресе](./media/direct-signin/login-hint.png)

Пользователь может изменить значение, отображаемое в текстовом поле, для входа в систему.

::: zone pivot="b2c-custom-policy"

Для поддержки параметра указания имени входа Переопределите `SelfAsserted-LocalAccountSignin-Email` технический профиль. В разделе `<InputClaims>` задайте для DefaultValue утверждения signInName значение `{OIDC:LoginHint}`. Переменная `{OIDC:LoginHint}` содержит значение параметра `login_hint`. Azure AD B2C считывает значение утверждения signInName и заполняет текстовое поле signInName.

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
      <InputClaims>
        <!-- Add the login hint value to the sign-in names claim type -->
        <InputClaim ClaimTypeReferenceId="signInName" DefaultValue="{OIDC:LoginHint}" />
      </InputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

::: zone-end

## <a name="redirect-sign-in-to-a-social-provider"></a>Перенаправление операции входа в поставщик социальных сетей

Если вы настроили для приложения возможность входа с использованием учетных записей социальных сетей, таких как Facebook, LinkedIn или Google, вы можете указать параметр `domain_hint`. Этот параметр запроса сообщает Azure AD B2C о поставщике удостоверений социальных сетей, который должен использоваться для входа в систему. Например, если приложение указывает `domain_hint=facebook.com`, вход в систему выполняется непосредственно на странице входа в Facebook.

![Страница входа с domain_hint параметром запроса, выделенным в URL-адресе](./media/direct-signin/domain-hint.png)

::: zone pivot="b2c-user-flow"

Параметр строки запроса Domain указание может иметь один из следующих доменов:

- amazon.com
- facebook.com
- github.com
- google.com
- linkedin.com
- microsoft.com
- qq.com
- twitter.com
- wechat.com
- weibo.com 
- Сведения об [универсальном OpenID Connect Connect](identity-provider-generic-openid-connect.md)см. в разделе [Указание домена](identity-provider-generic-openid-connect.md#response-mode).

::: zone-end

::: zone pivot="b2c-custom-policy"

Для поддержки параметра указания домена можно настроить имя домена с помощью `<Domain>domain name</Domain>` XML-элемента Any `<ClaimsProvider>` .

```xml
<ClaimsProvider>
    <!-- Add the domain hint value to the claims provider -->
    <Domain>facebook.com</Domain>
    <DisplayName>Facebook</DisplayName>
    <TechnicalProfiles>
    ...
```

::: zone-end

