---
title: Настройка входа с помощью поставщика SAML SalesForce с помощью протокола SAML
titleSuffix: Azure AD B2C
description: Настройка входа с помощью поставщика SAML SalesForce с помощью протокола SAML в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 01/27/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: cfdd96ffa9e7758fc594f7e7ae53c84495b8b2e8
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102095414"
---
# <a name="set-up-sign-in-with-a-salesforce-saml-provider-by-using-saml-protocol-in-azure-active-directory-b2c"></a>Настройка входа с помощью поставщика SAML SalesForce с помощью протокола SAML в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"
[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

В этой статье описывается включение входа пользователей из организации Salesforce с помощью [пользовательских политик](custom-policy-overview.md) в Azure Active Directory B2C (Azure AD B2C). Чтобы включить вход, добавьте [поставщик удостоверений SAML](identity-provider-generic-saml.md) в настраиваемую политику.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]
- Если это еще не сделано, зарегистрируйтесь для получения [бесплатной учетной записи Developer Edition](https://developer.salesforce.com/signup). В этой статье предполагается использование [Salesforce Lightning Experience](https://developer.salesforce.com/page/Lightning_Experience_FAQ).
- [Настройка собственного домена](https://help.salesforce.com/articleView?id=domain_name_setup.htm&language=en_US&type=0) для организации Salesforce.

## <a name="set-up-salesforce-as-an-identity-provider"></a>Настройка Salesforce в качестве поставщика удостоверений

1. [Войдите в Salesforce](https://login.salesforce.com/).
2. В меню слева в разделе **Параметры** разверните узел **Удостоверение** и выберите **Поставщик удостоверений**.
3. Выберите **Enable Identity Provider** (Включить поставщик удостоверений).
4. В разделе **Select the certificate** (Выберите сертификат) выберите сертификат, который необходимо использовать в Salesforce при взаимодействии с Azure AD B2C. Вы можете использовать сертификат по умолчанию.
5. Выберите команду **Сохранить**.

### <a name="create-a-connected-app-in-salesforce"></a>Создание подключенного приложения в Salesforce

1. На странице **Поставщик удостоверений** выберите **Service Providers are now created via Connected Apps. Click here** (Поставщики услуг теперь создаются с помощью подключенных приложений. Щелкните здесь).
2. В разделе **Основные сведения** введите нужные значения для подключенного приложения.
3. В разделе **Параметры веб-приложения** установите флажок **Включить SAML**.
4. В поле **Идентификатор сущности** введите следующий URL-адрес. Замените значение `your-tenant` на имя клиента Azure AD B2C.

      ```
      https://your-tenant.b2clogin.com/your-tenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase
      ```

6. В поле **ACS URL** (URL-адрес ACS) введите приведенный ниже URL-адрес. Замените значение `your-tenant` на имя клиента Azure AD B2C.

      ```
      https://your-tenant.b2clogin.com/your-tenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer
      ```
7. Прокрутите до нижней части списка и нажмите кнопку **Сохранить**.

### <a name="get-the-metadata-url"></a>Получить URL-адреса метаданных

1. На странице общих сведений о подключенном приложении щелкните **Управление**.
2. Скопируйте значение **конечной точки обнаружения метаданных**, а затем сохраните его. Оно будет использоваться далее в этой статье.

### <a name="set-up-salesforce-users-to-federate"></a>Настройка пользователей Salesforce для федерации

1. На странице **Управление** подключенного приложения щелкните **Управление профилями**.
2. Выберите профили (или группы пользователей) для федерации с Azure AD B2C. Используя права системного администратора, установите флажок **Системный администратор** для федерации с помощью учетной записи Salesforce.

## <a name="create-a-self-signed-certificate"></a>Создание самозаверяющего сертификата

[!INCLUDE [active-directory-b2c-create-self-signed-certificate](../../includes/active-directory-b2c-create-self-signed-certificate.md)]

## <a name="create-a-policy-key"></a>Создание ключа политики

Вам необходимо сохранить сертификат, который вы создали в клиенте Azure AD B2C.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
4. На странице "Обзор" выберите **Identity Experience Framework**.
5. Выберите **Ключи политики**, а затем щелкните **Добавить**.
6. Для пункта **Параметры** выберите `Upload`.
7. Введите **имя** политики. Например, SAMLSigningCert. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
8. Найдите созданный сертификат B2CSigningCert.pfx и выберите его.
9. Введите **пароль** для сертификата.
3. Нажмите кнопку **Создать**.

## <a name="add-a-claims-provider"></a>Добавление поставщика утверждений

Если необходимо разрешить пользователям входить в систему с помощью учетной записи Salesforce, нужно определить учетную запись в качестве поставщика утверждений, с которым Azure AD B2C может взаимодействовать через конечную точку. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Чтобы определить учетную запись Salesforce в качестве поставщика утверждений, добавьте ее в элемент **ClaimsProviders** в файле расширения политики. Дополнительные сведения см. [в разделе Определение поставщика удостоверений SAML](identity-provider-generic-saml.md).

1. Откройте файл *TrustFrameworkExtensions.xml*.
1. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
1. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>salesforce.com</Domain>
      <DisplayName>Salesforce</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Salesforce-SAML2">
          <DisplayName>Salesforce</DisplayName>
          <Description>Login with your Salesforce account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="WantsEncryptedAssertions">false</Item>
            <Item Key="WantsSignedAssertions">false</Item>
            <Item Key="PartnerEntity">https://contoso-dev-ed.my.salesforce.com/.well-known/samlidp.xml</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="userId"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="username"/>
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="salesforce.com" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Saml-idp"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. Измените значение **PartnerEntity** на скопированный ранее URL-адрес метаданных Salesforce.
1. Измените значение обоих экземпляров **StorageReferenceId** на имя ключа сертификата для подписи. Например, B2C_1A_SAMLSigningCert.
1. Откройте раздел `<ClaimsProviders>` и добавьте следующий фрагмент кода XML. Если политика уже содержит технический профиль `SM-Saml-idp`, перейдите к следующему шагу. Дополнительные сведения см. в разделе об [управлении сеансом единого входа](custom-policy-reference-sso.md).

    ```xml
    <ClaimsProvider>
      <DisplayName>Session Management</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="SM-Saml-idp">
          <DisplayName>Session Management Provider</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
            <Item Key="IncludeSessionIndex">false</Item>
            <Item Key="RegisterServiceProviders">false</Item>
          </Metadata>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```
1. Сохраните файл.

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
    <ClaimsExchange Id="SalesforceExchange" TechnicalProfileReferenceId="Salesforce-SAML2" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

[!INCLUDE [active-directory-b2c-test-relying-party-policy](../../includes/active-directory-b2c-test-relying-party-policy-user-journey.md)]

::: zone-end