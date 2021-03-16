---
title: Настройка регистрации и входа с учетной записью Azure AD B2C из другого клиента Azure AD B2C
titleSuffix: Azure AD B2C
description: Обеспечение регистрации и входа для клиентов с учетными записями Azure AD B2C из другого клиента в приложениях с помощью Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit, project-no-code
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 4b357213f4e552fd791fb575d8b7a287b924c7f9
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103489076"
---
# <a name="set-up-sign-up-and-sign-in-with-an-azure-ad-b2c-account-from-another-azure-ad-b2c-tenant"></a>Настройка регистрации и входа с учетной записью Azure AD B2C из другого клиента Azure AD B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="overview"></a>Обзор

В этой статье описывается, как настроить федерацию с другим клиентом Azure AD B2C. Если приложения защищены с помощью Azure AD B2C, это позволяет пользователям из других Azure AD B2c входить в систему с использованием существующих учетных записей. На следующей схеме пользователи могут войти в приложение, защищенное Azure AD B2C *contoso*, с учетной записью, управляемой клиентом Azure AD B2C *компании Fabrikam*. 

![Azure AD B2C Федерации с другим клиентом Azure AD B2C](./media/identity-provider-azure-ad-b2c/azure-ad-b2c-federation.png)


## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="create-an-azure-ad-b2c-application"></a>Создание приложения Azure AD B2C

Чтобы включить вход для пользователей с учетной записью из другого клиента Azure AD B2C (например, Fabrikam), в Azure AD B2C (например, Contoso):

1. Создайте пользовательский [поток](tutorial-create-user-flows.md)или [пользовательскую политику](custom-policy-get-started.md).
1. Затем создайте приложение на Azure AD B2C, как описано в этом разделе. 

Для создания приложения.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что вы используете каталог, содержащий другой клиент Azure AD B2C (например, Fabrikam.com).
1. На портале Azure найдите и выберите **Azure AD B2C**.
1. Щелкните **Регистрация приложений** и выберите **Новая регистрация**.
1. Введите **имя** приложения. Например, *контосоапп*.
1. В разделе **Поддерживаемые типы учетных записей** выберите элемент **Accounts in any identity provider or organizational directory (for authenticating users with user flows)** (Учетные записи в любом поставщике удостоверений или каталоге организации (для аутентификации пользователей с использованием сведений о маршрутах пользователей)).
1. В разделе **URI перенаправления** выберите **веб**, а затем введите следующий URL-адрес в строчных буквах, где `your-B2C-tenant-name` заменяется именем клиента Azure AD B2C (например, Contoso).

    ```
    https://your-B2C-tenant-name.b2clogin.com/your-B2C-tenant-name.onmicrosoft.com/oauth2/authresp
    ```

    Например, `https://contoso.b2clogin.com/contoso.onmicrosoft.com/oauth2/authresp`.

    Если используется [личный домен](custom-domain.md), введите `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp` . Замените `your-domain-name` пользовательским доменом и `your-tenant-name` именем своего клиента.

1. В разделе Разрешения установите флажок **предоставить согласие администратора для OpenID Connect и offline_access разрешения** .
1. Выберите **Зарегистрировать**.
1. На странице **Azure AD B2C-регистрация приложений** выберите созданное приложение, например *контосоапп*.
1. Запишите значение параметра **Идентификатор приложения (клиента)** , отображаемого на странице обзора приложения. Этот идентификатор потребуется при настройке поставщика удостоверений в следующем разделе.
1. В меню слева в разделе **Управление** выберите **Сертификаты и секреты**.
1. Выберите **Создать секрет клиента**.
1. Введите описание секрета клиента в поле **Описание**. Например, *clientsecret1*.
1. В разделе **Истекает** выберите срок действия секрета, а затем выберите **Добавить**.
1. Запишите значение секрета в поле **Значение**. Этот идентификатор потребуется при настройке поставщика удостоверений в следующем разделе.


::: zone pivot="b2c-user-flow"

## <a name="configure-azure-ad-b2c-as-an-identity-provider"></a>Настройка Azure AD B2C в качестве поставщика удостоверений

1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C, для которого требуется настроить федерацию (например, Contoso). Выберите фильтр **каталог и подписка** в верхнем меню и выберите каталог, содержащий клиент Azure AD B2C (например, Contoso).
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. Щелкните элемент **Поставщики удостоверений**, а затем выберите **Новый поставщик OpenID Connect**.
1. Введите **Имя**. Например, введите *Fabrikam*.
1. В поле **URL-адрес метаданных** введите следующий URL-адрес, заменяющий `{tenant}` доменное имя клиента Azure AD B2C (например, Fabrikam). Замените на `{policy}` имя политики, настроенной в другом клиенте:

    ```
    https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/v2.0/.well-known/openid-configuration
    ```

    Например, `https://fabrikam.b2clogin.com/fabrikam.onmicrosoft.com/B2C_1_susi/v2.0/.well-known/openid-configuration`.

1. В поле **Идентификатор клиента** введите ранее записанное значение идентификатора приложения.
1. В поле **Секрет клиента** введите ранее записанное значение секрета клиента.
1. Для параметра **Область** введите значение `openid`.
1. Сохраните значения по умолчанию для параметров **Тип ответа** и **Режим ответа**.
1. Используемых В качестве указания **домена** введите имя домена, которое вы хотите использовать для [прямого входа](direct-signin.md#redirect-sign-in-to-a-social-provider). Например, *Fabrikam.com*.
1. В разделе **Сопоставление утверждений поставщика удостоверений** выберите следующие утверждения:

    - **Идентификатор пользователя**: *подсистема*
    - **Display name**: *name*;
    - **Given name**: *given_name*;
    - **Surname**: *family_name*;
    - **Электронная почта**: *Электронная почта*

1. Щелкните **Сохранить**.

## <a name="add-azure-ad-b2c-identity-provider-to-a-user-flow"></a>Добавление поставщика удостоверений Azure AD B2C в поток пользователя 

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Щелкните поток пользователя, который требуется добавить Azure AD B2C поставщика удостоверений.
1. В разделе **поставщики удостоверений социальных сетей** выберите **Fabrikam**.
1. Щелкните **Сохранить**.
1. Чтобы проверить политику, выберите пункт **выполнить пользовательскую последовательность**.
1. Для **приложения** выберите веб-приложение с именем *testapp1* , которое вы зарегистрировали ранее. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **запустить поток пользователя** .
1. На странице Регистрация или вход выберите **Fabrikam** , чтобы войти с другим клиентом Azure AD B2C.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="create-a-policy-key"></a>Создание ключа политики

Необходимо сохранить ключ приложения, созданный ранее в клиенте Azure AD B2C.

1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C, для которого требуется настроить федерацию (например, Contoso). Выберите фильтр **каталог и подписка** в верхнем меню и выберите каталог, содержащий клиент Azure AD B2C (например, Contoso).
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. В разделе **Политики** выберите **Identity Experience Framework**.
1. Выберите **ключи политики** и нажмите кнопку **Добавить**.
1. Для пункта **Параметры** выберите `Manual`.
1. Введите **имя** ключа политики. Например, `FabrikamAppSecret`.  Префикс `B2C_1A_` добавляется автоматически к имени ключа при его создании, поэтому ссылка в XML-файле в следующем разделе *B2C_1A_FabrikamAppSecret*.
1. В качестве **секрета** введите секрет клиента, записанный ранее.
1. Для параметра **Использование ключа** выберите `Signature`.
1. Нажмите кнопку **создания**.

## <a name="configure-azure-ad-b2c-as-an-identity-provider"></a>Настройка Azure AD B2C в качестве поставщика удостоверений

Чтобы пользователи могли входить с помощью учетной записи из другого клиента Azure AD B2C (Fabrikam), необходимо определить другие Azure AD B2C как поставщик утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Вы можете определить Azure AD B2C в качестве поставщика утверждений, добавив Azure AD B2C в элемент **поставщика утверждений** в файле расширения политики.

1. Откройте файл *TrustFrameworkExtensions.xml*.
1. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
1. Добавьте новый элемент **ClaimsProvider** следующим образом.
    ```xml
    <ClaimsProvider>
      <Domain>fabrikam.com</Domain>
      <DisplayName>Federation with Fabrikam tenant</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="AzureADB2CFabrikam-OpenIdConnect">
        <DisplayName>Fabrikam</DisplayName>
        <Protocol Name="OpenIdConnect"/>
        <Metadata>
          <!-- Update the Client ID below to the Application ID -->
          <Item Key="client_id">00000000-0000-0000-0000-000000000000</Item>
          <!-- Update the metadata URL with the other Azure AD B2C tenant name and policy name -->
          <Item Key="METADATA">https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/v2.0/.well-known/openid-configuration</Item>
          <Item Key="UsePolicyInRedirectUri">false</Item>
          <Item Key="response_types">code</Item>
          <Item Key="scope">openid</Item>
          <Item Key="response_mode">form_post</Item>
          <Item Key="HttpBinding">POST</Item>
        </Metadata>
        <CryptographicKeys>
          <Key Id="client_secret" StorageReferenceId="B2C_1A_FabrikamAppSecret"/>
        </CryptographicKeys>
        <OutputClaims>
          <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub" />
          <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
          <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
          <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
          <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
          <OutputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="iss"  />
          <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
          <OutputClaim ClaimTypeReferenceId="otherMails" PartnerClaimType="emails"/>    
        </OutputClaims>
        <OutputClaimsTransformations>
          <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
          <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
          <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
        </OutputClaimsTransformations>
        <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin"/>
      </TechnicalProfile>
     </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. Обновите следующие XML-элементы с помощью соответствующего значения:

    |Элемент XML  |Значение  |
    |---------|---------|
    |клаимспровидер\домаин  | Имя домена, используемое для [прямого входа](direct-signin.md?pivots=b2c-custom-policy#redirect-sign-in-to-a-social-provider). Введите имя домена, которое будет использоваться при прямом входе. Например, *Fabrikam.com*. |
    |течникалпрофиле\дисплайнаме|Это значение будет отображаться на кнопке входа на экране входа в систему. Например, *Fabrikam*. |
    |Метаданные \ client_id|Идентификатор приложения поставщика удостоверений. Обновите идентификатор клиента с помощью идентификатора приложения, созданного ранее в другом клиенте Azure AD B2C.|
    |метадата\метадата|URL-адрес, указывающий на документ конфигурации поставщика удостоверений OpenID Connect Connect, который также называется OpenID Connect хорошо известной конечной точкой конфигурации. Введите следующий URL-адрес, заменив `{tenant}` именем домена другого Azure AD B2C клиента (Fabrikam). Замените `{tenant}` именем политики, настроенным в другом клиенте, и `{policy]` именем политики: `https://{tenant}.b2clogin.com/{tenant}.onmicrosoft.com/{policy}/v2.0/.well-known/openid-configuration` . Например, `https://fabrikam.b2clogin.com/fabrikam.onmicrosoft.com/B2C_1_susi/v2.0/.well-known/openid-configuration`.|
    |CryptographicKeys| Измените значение **идентификатором storagereferenceid** на имя созданного ранее ключа политики. Например, `B2C_1A_FabrikamAppSecret`.| 
    

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="AzureADB2CFabrikamExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="AzureADB2CFabrikamExchange" TechnicalProfileReferenceId="AzureADB2CFabrikam-OpenIdConnect" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]


## <a name="test-your-custom-policy"></a>Тестирование настраиваемой политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице Регистрация или вход выберите **Fabrikam** , чтобы войти с другим клиентом Azure AD B2C.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [передать в приложение другой токен Azure AD B2C](idp-pass-through-user-flow.md).
