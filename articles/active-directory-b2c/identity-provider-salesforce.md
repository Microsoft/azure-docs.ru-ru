---
title: Настройка регистрации и входа в систему с помощью учетной записи Salesforce
titleSuffix: Azure AD B2C
description: Обеспечение регистрации и входа для клиентов с учетными записями Salesforce в приложениях с помощью Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/08/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 578a248de3ee6aacc0db7c5116b66031b404c9ae
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102448172"
---
# <a name="set-up-sign-up-and-sign-in-with-a-salesforce-account-using-azure-active-directory-b2c"></a>Настройка регистрации и входа с помощью учетной записи SalesForce с помощью Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]


## <a name="create-a-salesforce-application"></a>Создание приложения Salesforce

Чтобы включить вход для пользователей с учетной записью Salesforce в Azure Active Directory B2C (Azure AD B2C), необходимо создать приложение в [диспетчере приложений](https://login.salesforce.com/)Salesforce. Дополнительные сведения см. в статьях [Настройка основных параметров подключенного приложения](https://help.salesforce.com/articleView?id=connected_app_create_basics.htm)и [Включение параметров OAuth для интеграции API](https://help.salesforce.com/articleView?id=connected_app_create_api_integration.htm) .

1. [Войдите в Salesforce](https://login.salesforce.com/).
1. В меню выберите пункт **Настройка**.
1.  Разверните узел **приложения** и выберите **app Manager**.
1. Выберите **новое подключенное приложение**.
1. В разделе **Основные сведения** введите:
    1. **Имя подключенного приложения** . имя подключенного приложения отображается в диспетчере приложений и на плитке запуска приложения. Имя должно быть уникальным в пределах организации. 
    1. **Имя API** 
    1. **Контактный адрес электронной** почты — Контактное письмо для Salesforce
1. В разделе **API (включение параметров OAuth)** выберите **включить параметры OAuth** .
    1. В окне **URL-адрес обратного вызова** введите `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp` . Замените `your-tenant-name` именем вашего клиента. При вводе имени вашего клиента необходимо использовать только строчные буквы, даже если в Azure AD B2C имя клиента определено с прописными буквами.
    1. В **выбранных областях OAuth** выберите **доступ к основным сведениям (идентификатор, профиль, электронная почта, адрес, Телефон)** и **разрешите доступ к своему уникальному идентификатору (OpenID Connect)**.
    1. Выберите параметр **требовать секрет для потока веб-сервера**.
1. Выберите **Настройка токена идентификации** 
    1. Задайте **срок действия маркера в течение** 5 минут.
    1. Выберите **включить стандартные утверждения**.
1. Выберите команду **Сохранить**.
1. Скопируйте значения **ключа потребителя** и **секрета потребителя**. Оба они понадобятся для настройки Salesforce в качестве поставщика удостоверений в клиенте. **Секрет клиента** — это важные учетные данные безопасности.

::: zone pivot="b2c-user-flow"

## <a name="configure-salesforce-as-an-identity-provider"></a>Настройка Salesforce в качестве поставщика удостоверений

1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем каталог, содержащий клиент Azure AD B2C.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. Щелкните элемент **Поставщики удостоверений**, а затем выберите **Новый поставщик OpenID Connect**.
1. Введите **Имя**. Например, введите *Salesforce*.
1. В поле **URL-адрес метаданных** введите URL-адрес [документа конфигурации Salesforce OpenID Connect Connect](https://help.salesforce.com/articleView?id=remoteaccess_using_openid_discovery_endpoint.htm). Для песочницы login.salesforce.com заменяется на test.salesforce.com. Для сообщества login.salesforce.com заменяется URL-адресом сообщества, например username.force.com/.well-known/openid-configuration. URL-адрес должен быть HTTPS.

    ```
    https://login.salesforce.com/.well-known/openid-configuration
    ```

1. В поле **Идентификатор клиента** введите ранее записанное значение идентификатора приложения.
1. В поле **Секрет клиента** введите ранее записанное значение секрета клиента.
1. Для параметра **Область** введите значение `openid id profile email`.
1. Сохраните значения по умолчанию для параметров **Тип ответа** и **Режим ответа**.
1. (Необязательно) В поле **Указание домена** Введите `contoso.com`. Дополнительные сведения см. в статье [Настройка прямого входа в систему с помощью Azure Active Directory B2C](direct-signin.md#redirect-sign-in-to-a-social-provider).
1. В разделе **Сопоставление утверждений поставщика удостоверений** выберите следующие утверждения:

    - **Идентификатор пользователя**: *подсистема*
    - **Display name**: *name*;
    - **Given name**: *given_name*;
    - **Surname**: *family_name*;
    - **Электронная почта**: *Электронная почта*

1. Щелкните **Сохранить**.

## <a name="add-salesforce-identity-provider-to-a-user-flow"></a>Добавление поставщика удостоверений Salesforce в поток пользователя 

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Щелкните поток пользователя, который требуется добавить в поставщик удостоверений Salesforce.
1. В разделе **поставщики удостоверений социальных сетей** выберите **Salesforce**.
1. Щелкните **Сохранить**.
1. Чтобы проверить политику, выберите пункт **выполнить пользовательскую последовательность**.
1. Для **приложения** выберите веб-приложение с именем *testapp1* , которое вы зарегистрировали ранее. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **запустить поток пользователя** .
1. На странице Регистрация или вход выберите **Salesforce** , чтобы войти с помощью учетной записи Salesforce.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="create-a-policy-key"></a>Создание ключа политики

Вам необходимо сохранить секрет клиента, который ранее был записан в клиенте Azure AD B2C.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню и выберите каталог, который содержит ваш клиент.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
4. На странице "Обзор" выберите **Identity Experience Framework**.
5. Выберите **Ключи политики**, а затем щелкните **Добавить**.
6. Для пункта **Параметры** выберите `Manual`.
7. Введите **имя** ключа политики. Например, `SalesforceSecret`. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
8. В поле **Секрет** введите ранее записанный секрет клиента.
9. Для параметра **Использование ключа** выберите `Signature`.
10. Нажмите кнопку **Создать**.

## <a name="configure-salesforce-as-an-identity-provider"></a>Настройка Salesforce в качестве поставщика удостоверений

Чтобы пользователи могли входить с помощью учетной записи Salesforce, необходимо определить учетную запись как поставщика утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Чтобы определить учетную запись Salesforce в качестве поставщика утверждений, добавьте ее в элемент **ClaimsProviders** в файле расширения политики.

1. Откройте файл *TrustFrameworkExtensions.xml*.
2. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
3. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>salesforce.com</Domain>
      <DisplayName>Salesforce</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Salesforce-OpenIdConnect">
          <DisplayName>Salesforce</DisplayName>
          <Protocol Name="OpenIdConnect" />
          <Metadata>
            <Item Key="METADATA">https://login.salesforce.com/.well-known/openid-configuration</Item>
            <Item Key="response_types">code</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="scope">openid id profile email</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="UsePolicyInRedirectUri">0</Item>
            <!-- Update the Client ID below to the Application ID -->
            <Item Key="client_id">Your Salesforce application ID</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_SalesforceSecret"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="salesforce.com" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

4. Для **метаданных** задается URL-адрес [документа конфигурации Salesforce OpenID Connect Connect](https://help.salesforce.com/articleView?id=remoteaccess_using_openid_discovery_endpoint.htm). Для песочницы login.salesforce.com заменяется на test.salesforce.com. Для сообщества login.salesforce.com заменяется URL-адресом сообщества, например username.force.com/.well-known/openid-configuration. URL-адрес должен быть HTTPS.
5. Задайте для параметра **client_id** значение идентификатора приложения из регистрации приложения.
6. Сохраните файл.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="SalesforceExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="SalesforceExchange" TechnicalProfileReferenceId="Salesforce-OpenIdConnect" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Тестирование пользовательской политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице Регистрация или вход выберите **Salesforce** , чтобы войти с помощью учетной записи Salesforce.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.


::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [передать токен Salesforce в приложение](idp-pass-through-user-flow.md).
