---
title: Настройка регистрации и входа с помощью поставщика удостоверений SAML
titleSuffix: Azure Active Directory B2C
description: Настройте регистрацию и вход с помощью любого поставщика удостоверений SAML (IdP) в Azure Active Directory B2C.
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
ms.openlocfilehash: 9e47171fc20ba07823e73f71713307e3a0e37278
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103488930"
---
# <a name="set-up-sign-up-and-sign-in-with-saml-identity-provider-using-azure-active-directory-b2c"></a>Настройка регистрации и входа с помощью поставщика удостоверений SAML с использованием Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) поддерживает федерацию с поставщиками удостоверений SAML 2,0. В этой статье показано, как включить вход с помощью учетной записи пользователя поставщика удостоверений SAML, что позволяет пользователям входить в систему с помощью существующих социальных удостоверений или идентификаторов предприятия, таких как [ADFS](identity-provider-adfs2016-custom.md) и [Salesforce](identity-provider-salesforce-saml.md).

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="scenario-overview"></a>Общие сведения о сценарии

Вы можете настроить Azure AD B2C, чтобы пользователи могли входить в приложение с учетными данными из внешних социальных сетей или поставщиков удостоверений SAML (IdP). Если Azure AD B2C Федерации с поставщиком удостоверений SAML, он выступает в качестве **поставщика услуг** , ИНИЦИИРУЮЩЕГО запрос SAML **поставщику удостоверений** SAML и ожидая ответа SAML. На следующей схеме:

1. Приложение инициирует запрос авторизации для Azure AD B2C. Приложение может быть приложением [OAuth 2,0](protocols-overview.md) или [OpenID Connect Connect](openid-connect.md) или [поставщиком службы SAML](saml-service-provider.md). 
1. На странице входа Azure AD B2C пользователь выбирает вход с использованием учетной записи поставщика удостоверений SAML (например, *contoso*). Azure AD B2C инициирует запрос авторизации SAML и выполняет вход в поставщик удостоверений SAML для завершения входа.
1. Поставщик удостоверений SAML возвращает ответ SAML.
1. Azure AD B2C проверяет маркер SAML, извлекает утверждения, выдает свой собственный маркер и переводит пользователя в приложение.  

![Вход с помощью потока поставщика удостоверений SAML](./media/identity-provider-generic-saml/sign-in-with-saml-identity-provider-flow.png)

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]

## <a name="components-of-the-solution"></a>Компоненты решения

Для этого сценария необходимы следующие компоненты:

* **Поставщик удостоверений** SAML с возможностью получения, декодирования и реагирования на запросы SAML от Azure AD B2C.
* Общедоступная **Конечная точка МЕТАДАННЫХ** SAML для поставщика удостоверений.
* [Клиент Azure AD B2C](tutorial-create-tenant.md).
 

## <a name="create-a-policy-key"></a>Создание ключа политики

Чтобы установить отношение доверия между Azure AD B2C и поставщиком удостоверений SAML, необходимо предоставить действительный сертификат X509 с закрытым ключом. Azure AD B2C подписывает запросы SAML с помощью закрытого ключа сертификата. Поставщик удостоверений проверяет запрос с помощью открытого ключа сертификата. Открытый ключ доступен через метаданные технического профиля. Кроме того, можно вручную отправить CER-файл поставщику удостоверений SAML.

Самозаверяющий сертификат приемлем для большинства сценариев. В рабочей среде рекомендуется использовать сертификат X509, выданный центром сертификации. Кроме того, как описано далее в этом документе, для нерабочей среды можно отключить подписывание SAML на обеих сторонах.

### <a name="obtain-a-certificate"></a>Получение сертификата

[!INCLUDE [active-directory-b2c-create-self-signed-certificate](../../includes/active-directory-b2c-create-self-signed-certificate.md)]

### <a name="upload-the-certificate"></a>Загрузка сертификата

Вам необходимо сохранить сертификат в клиенте Azure AD B2C.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню и выберите каталог, который содержит ваш клиент.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. На странице "Обзор" выберите **Identity Experience Framework**.
1. Выберите **Ключи политики**, а затем щелкните **Добавить**.
1. Для пункта **Параметры** выберите `Upload`.
1. Введите **имя** ключа политики. Например, `SAMLSigningCert`. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
1. Найдите и выберите PFX-файл сертификата с закрытым ключом.
1. Нажмите кнопку **Создать**.

## <a name="configure-the-saml-technical-profile"></a>Настройка технического профиля SAML

Определите поставщик удостоверений SAML, добавив его в элемент **клаимспровидерс** в файле расширения политики. Поставщики утверждений содержат технический профиль SAML, который определяет конечные точки и протоколы, необходимые для взаимодействия с поставщиком удостоверений SAML. Чтобы добавить поставщик утверждений с техническим профилем SAML, выполните следующие действия.

1. Откройте файл *TrustFrameworkExtensions.xml*.
1. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
1. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>Contoso.com</Domain>
      <DisplayName>Contoso</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Contoso-SAML2">
          <DisplayName>Contoso</DisplayName>
          <Description>Login with your SAML identity provider account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="PartnerEntity">https://your-AD-FS-domain/federationmetadata/2007-06/federationmetadata.xml</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SAMLSigningCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="assertionSubjectName" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="first_name" />
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="last_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="http://schemas.microsoft.com/identity/claims/displayname" />
            <OutputClaim ClaimTypeReferenceId="email"  />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="contoso.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
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

Обновите следующие XML-элементы с помощью соответствующего значения:

|Элемент XML  |Значение  |
|---------|---------|
|клаимспровидер\домаин  | Имя домена, используемое для [прямого входа](direct-signin.md?pivots=b2c-custom-policy#redirect-sign-in-to-a-social-provider). Введите имя домена, которое будет использоваться при прямом входе. Например, *contoso.com*. |
|течникалпрофиле\дисплайнаме|Это значение будет отображаться на кнопке входа на экране входа в систему. Например, *contoso*. |
|метадата\партнерентити|URL-адрес метаданных поставщика удостоверений SAML. Также можно скопировать метаданные поставщика удостоверений и добавить их в элемент CDATA `<![CDATA[Your IDP metadata]]>` .|

## <a name="map-the-claims"></a>Сопоставьте утверждения.

Элемент **OutputClaims** содержит список заявок, возвращенных поставщиком удостоверений SAML. Сопоставьте имя утверждения, определенного в политике, с именем утверждения, определенным в поставщике удостоверений. В поставщике удостоверений проверьте список утверждений (утверждений). Дополнительные сведения см. в разделе [утверждения — сопоставление](identity-provider-generic-saml-options.md#claims-mapping).

В приведенном выше примере *contoso-SAML2* включает утверждения, возвращенные поставщиком удостоверений SAML:

* Утверждение **issuerUserId** сопоставляется с утверждением **ассертионсубжектнаме** .
* Утверждение **first_name** сопоставляется с утверждением **givenName**.
* Утверждение **last_name** сопоставляется с утверждением **surname**.
* Утверждение **DisplayName** сопоставлено с `http://schemas.microsoft.com/identity/claims/displayname` утверждением.
* Утверждение **email** не сопоставляется с именем.

Технический профиль также возвращает утверждения, которые не возвращаются поставщиком удостоверений:

* Утверждение **IdentityProvider**, содержащее имя поставщика удостоверений.
* утверждение **AuthenticationSource** со значением по умолчанию **socialIdpAuthentication**.
 
### <a name="add-the-saml-session-technical-profile"></a>Добавление технического профиля сеанса SAML

Если у вас еще нет `SM-Saml-idp` технического профиля для сеанса SAML, добавьте его в политику расширения. Откройте раздел `<ClaimsProviders>` и добавьте следующий фрагмент кода XML. Если политика уже содержит технический профиль `SM-Saml-idp`, перейдите к следующему шагу. Дополнительные сведения см. в разделе об [управлении сеансом единого входа](custom-policy-reference-sso.md).

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

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="Contoso-SAML2" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-create-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="configure-your-saml-identity-provider"></a>Настройка поставщика удостоверений SAML

После настройки политики необходимо настроить поставщик удостоверений SAML с помощью Azure AD B2C метаданных SAML. Метаданные SAML — это сведения, используемые в протоколе SAML для предоставления конфигурации политики, **поставщика услуг**. Он определяет расположение служб, таких как вход и выход, сертификаты, метод входа и многое другое.

Каждый поставщик удостоверений SAML имеет различные шаги для настройки поставщика услуг. Некоторые поставщики удостоверений SAML запрашивают метаданные Azure AD B2C, а другие требуют, чтобы вы вручную просматриваете файл метаданных и предоставили сведения. Инструкции см. в документации поставщика удостоверений.

В следующем примере показан URL-адрес для метаданных SAML для Azure AD B2C технического профиля:

```
https://<your-tenant-name>.b2clogin.com/<your-tenant-name>.onmicrosoft.com/<your-policy>/samlp/metadata?idptp=<your-technical-profile>
```

При использовании [пользовательского домена](custom-domain.md)используйте следующий формат:

```
https://your-domain-name/<your-tenant-name>.onmicrosoft.com/<your-policy>/samlp/metadata?idptp=<your-technical-profile>
```

Измените следующие значения:

- **имя** клиента с именем клиента, например Your-tenant.onmicrosoft.com.
- **имя вашего домена** с именем пользовательского домена, например Login.contoso.com.
- **your-policy** — именем собственной политики. Например, B2C_1A_signup_signin_adfs.
- **ваш-технический профиль** с именем технического профиля поставщика удостоверений SAML. Например, Contoso-SAML2.

Откройте браузер и перейдите по URL-адресу. Убедитесь, что ввели правильный URL-адрес и что имеете доступ к XML-файлу метаданных.

## <a name="test-your-custom-policy"></a>Тестирование настраиваемой политики

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите значок **Каталог и подписка** в верхней панели инструментов портала, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. На портале Azure найдите и выберите **Azure AD B2C**.
1. В разделе " **политики**" выберите **инфраструктура процедур идентификации** .
1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице регистрации или входа выберите **contoso** для входа с помощью учетной записи contoso.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

## <a name="next-steps"></a>Дальнейшие действия

- [Настройка параметров поставщика удостоверений SAML с помощью Azure Active Directory B2C](identity-provider-generic-saml-options.md)

::: zone-end
