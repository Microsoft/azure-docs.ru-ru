---
title: Конечная точка UserInfo | Документация Майкрософт
description: Определите конечную точку UserInfo в пользовательской политике в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/09/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: c060a029b1cdbdd890ced96cab732966cb652de0
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102500586"
---
# <a name="userinfo-endpoint"></a>Конечная точка UserInfo

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Конечная точка UserInfo является частью спецификации [OpenID Connect Connect Standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo) (OIDC) и предназначена для возврата утверждений о пользователе, прошедшем проверку подлинности. Конечная точка UserInfo определяется в политике проверяющей стороны с помощью элемента [Endpoint](relyingparty.md#endpoints) .  

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [active-directory-b2c-customization-prerequisites-custom-policy](../../includes/active-directory-b2c-customization-prerequisites-custom-policy.md)]

## <a name="userinfo-endpoint-overview"></a>Обзор конечной точки UserInfo

Сведения о пользователе UserJourney указывают:

- **Авторизация**. Конечная точка UserInfo защищена токеном носителя. Выданный маркер доступа представлен в заголовке авторизации для конечной точки UserInfo. Политика указывает технический профиль, который проверяет входящий токен и извлекает утверждения, например objectId пользователя. ObjectId пользователя используется для получения утверждений, возвращаемых в ответе пути к конечной точке UserInfo. 
- **Этап оркестрации**: 
  - Этап оркестрации используется для сбора сведений о пользователе. В зависимости от утверждений входящего маркера доступа пользователь вызывает [Azure Active Directory технический профиль](active-directory-technical-profile.md) для получения данных о пользователе, например чтение пользователя по ObjectID. 
  - Дополнительные **шаги оркестрации** . Вы можете добавить дополнительные шаги оркестрации, например REST API технический профиль, чтобы получить дополнительные сведения о пользователе. 
  - Имя **поставщика UserInfo** — указывает список утверждений, возвращаемых конечной точкой UserInfo.

## <a name="create-a-userinfo-endpoint"></a>Создание конечной точки UserInfo

### <a name="1-add-the-token-issuer-technical-profile"></a>1. Добавьте технический профиль издателя маркера

1. Откройте файл *TrustFrameworkExtensions.xml*.
1. Если он еще не существует, добавьте элемент поставщика утверждений и его дочерние элементы в качестве первого элемента в элементе BuildingBlocks.
1. Добавьте следующий поставщик утверждений:

    ```xml
    <!-- 
    <ClaimsProviders> -->
      <ClaimsProvider>
        <DisplayName>Token Issuer</DisplayName>
        <TechnicalProfiles>
          <TechnicalProfile Id="UserInfoIssuer">
            <DisplayName>JSON Issuer</DisplayName>
            <Protocol Name="None" />
            <OutputTokenFormat>JSON</OutputTokenFormat>
            <CryptographicKeys>
              <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
            </CryptographicKeys>
            <!-- The Below claims are what will be returned on the UserInfo Endpoint if in the Claims Bag-->
            <InputClaims>
              <InputClaim ClaimTypeReferenceId="objectId"/>
              <InputClaim ClaimTypeReferenceId="givenName"/>
              <InputClaim ClaimTypeReferenceId="surname"/>
              <InputClaim ClaimTypeReferenceId="displayName"/>
              <InputClaim ClaimTypeReferenceId="signInNames.emailAddress"/>
            </InputClaims>
          </TechnicalProfile>
          <TechnicalProfile Id="UserInfoAuthorization">
            <DisplayName>UserInfo authorization</DisplayName>
            <Protocol Name="None" />
            <InputTokenFormat>JWT</InputTokenFormat>
            <Metadata>
              <!-- Update the Issuer and Audience below -->
              <!-- Audience is optional, Issuer is required-->
              <Item Key="issuer">https://yourtenant.b2clogin.com/11111111-1111-1111-1111-111111111111/v2.0/</Item>
              <Item Key="audience">[ "22222222-2222-2222-2222-222222222222", "33333333-3333-3333-3333-333333333333" ]</Item>
              <Item Key="client_assertion_type">urn:ietf:params:oauth:client-assertion-type:jwt-bearer</Item>
            </Metadata>
            <CryptographicKeys>
              <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
            </CryptographicKeys>
            <OutputClaims>
              <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
              <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" PartnerClaimType="email"/>
              <!-- Optional claims to read from the access token. -->
              <!-- <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
                 <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
                 <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name"/> -->
            </OutputClaims>
          </TechnicalProfile>
        </TechnicalProfiles>
      </ClaimsProvider>
    <!-- 
    </ClaimsProviders> -->
    ``` 

1. Раздел Inputclaim в техническом профиле **усеринфоиссуер** указывает атрибуты, которые требуется вернуть. Технический профиль Усеринфоиссуер вызывается в конце пути взаимодействия пользователя. 
1. Технический профиль **усеринфоаусоризатион** проверяет подпись, имя издателя и аудиторию маркера и извлекает утверждение из входящего токена. Измените следующие метаданные в соответствии с вашей средой:
    1. **Issuer** — это значение должно совпадать с `iss` утверждением в утверждении маркера доступа. Токены, выданные Azure AD B2C, используют издатель в формате `https://yourtenant.b2clogin.com/your-tenant-id/v2.0/` . Дополнительные сведения о [настройке маркеров](configure-tokens.md).
    1. **Идтокенаудиенце** — должно совпадать с `aud` утверждением в утверждении маркера доступа. В Azure AD B2C `aud` утверждение является идентификатором приложения проверяющей стороны. Это значение является коллекцией и поддерживает несколько значений с помощью разделителя запятыми.

        В следующем маркере доступа `iss` значение утверждения — `https://contoso.b2clogin.com/11111111-1111-1111-1111-111111111111/v2.0/` . `aud`Значение утверждения — `22222222-2222-2222-2222-222222222222` .

        ```json
        {
          "exp": 1605549468,
          "nbf": 1605545868,
          "ver": "1.0",
          "iss": "https://contoso.b2clogin.com/11111111-1111-1111-1111-111111111111/v2.0/",
          "sub": "44444444-4444-4444-4444-444444444444",
          "aud": "22222222-2222-2222-2222-222222222222",
          "acr": "b2c_1a_signup_signin",
          "nonce": "defaultNonce",
          "iat": 1605545868,
          "auth_time": 1605545868,
          "name": "John Smith",
          "given_name": "John",
          "family_name": "Smith",
          "tid": "11111111-1111-1111-1111-111111111111"
        }
        ```
    
1.  Элемент OutputClaims в техническом профиле **усеринфоаусоризатион** указывает атрибуты, которые требуется считать из маркера доступа. **Claimtypereferenceid соединяется** — это ссылка на тип утверждения. Необязательный **партнерклаимтипе** — это имя утверждения, определенного в маркере доступа.



### <a name="2-add-the-userjourney-element"></a>2. Добавьте элемент UserJourney 

Элемент [UserJourney](userjourneys.md) определяет путь взаимодействия пользователя с приложением. Добавьте элемент **UserJourneys**, если он не существует. Присвойте **UserJourney** идентификатор `UserInfoJourney`.

```xml
<!-- 
<UserJourneys> -->
  <UserJourney Id="UserInfoJourney" DefaultCpimIssuerTechnicalProfileReferenceId="UserInfoIssuer">
    <Authorization>
      <AuthorizationTechnicalProfiles>
        <AuthorizationTechnicalProfile ReferenceId="UserInfoAuthorization" />
      </AuthorizationTechnicalProfiles>
    </Authorization>
    <OrchestrationSteps >
      <OrchestrationStep Order="1" Type="ClaimsExchange">
        <Preconditions>
          <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
            <Value>objectId</Value>
            <Action>SkipThisOrchestrationStep</Action>
          </Precondition>
        </Preconditions>
        <ClaimsExchanges UserIdentity="false">
          <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
        </ClaimsExchanges>
      </OrchestrationStep>
      <OrchestrationStep Order="2" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="UserInfoIssuer" />
    </OrchestrationSteps>
  </UserJourney>
<!-- 
</UserJourneys> -->
```

### <a name="3-include-the-endpoint-to-the-relying-party-policy"></a>3. Включите конечную точку в политику проверяющей стороны.

Чтобы включить конечную точку UserInfo в приложение проверяющей стороны, добавьте элемент [Endpoint](relyingparty.md#endpoints) в файл *SocialAndLocalAccounts/SignUpOrSignIn.xml* . 

```xml
<!--
<RelyingParty> -->
  <Endpoints>
    <Endpoint Id="UserInfo" UserJourneyReferenceId="UserInfoJourney" />
  </Endpoints>
<!-- 
</RelyingParty> -->
```

Завершенный элемент проверяющей стороны будет выглядеть следующим образом:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" PolicySchemaVersion="0.3.0.0" TenantId="yourtenant.onmicrosoft.com" PolicyId="B2C_1A_signup_signin" PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_signup_signin">
  <BasePolicy>
    <TenantId>yourtenant.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <Endpoints>
      <Endpoint Id="UserInfo" UserJourneyReferenceId="UserInfoJourney" />
    </Endpoints>
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="OpenIdConnect" />
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
        <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      </OutputClaims>
      <SubjectNamingInfo ClaimType="sub" />
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

### <a name="4-upload-the-files"></a>4. Отправка файлов

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. Выберите **Инфраструктура процедур идентификации**.
1. На странице **Настраиваемые политики** выберите **Отправить настраиваемую политику**.
1. Выберите **параметр Перезаписать настраиваемую политику, если она уже существует**, а затем найдите и выберите файл *TrustframeworkExtensions.xml* .
1. Щелкните **Отправить**.
1. Повторите шаги с 5 по 7 для файла проверяющей стороны, например *SignUpOrSignIn.xml*.

## <a name="calling-the-userinfo-endpoint"></a>Вызов конечной точки UserInfo

Конечная точка UserInfo использует стандартный API токена носителя OAuth2, который вызывается с помощью маркера доступа, полученного при получении маркера для приложения. Он возвращает ответ JSON, содержащий утверждения о пользователе. Конечная точка UserInfo размещается на Azure AD B2C:

```http
https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/policy-name/openid/v2.0/userinfo
```

Поле/.Велл-кновн Настройка конечной точки (документ обнаружения OpenID Connect Connect Discovery) содержит список `userinfo_endpoint` . Вы можете программно обнаружить конечную точку UserInfo с помощью конечной точки/.Велл-кновн configure по адресу: 

```http
https://yourtenant.b2clogin.com/yourtenant.onmicrosoft.com/policy-name/v2.0/.well-known/openid-configuration 
```

### <a name="test-the-policy"></a>Проверка политики

1. В разделе **пользовательские политики** выберите политику, которая интегрирована с конечной точкой UserInfo. Например, *B2C_1A_SignUpOrSignIn*.
1. Выберите **Запустить сейчас**. 
1. В разделе **Выбор приложения** выберите приложение, которое вы зарегистрировали ранее. В качестве **URL-адреса ответа выберите** `https://jwt.ms` . Дополнительные сведения см. [в статье регистрация веб-приложения в Azure Active Directory B2C](tutorial-register-applications.md).
1. Зарегистрируйтесь или войдите с помощью адреса электронной почты или учетной записи социальных сетей.
1. Скопируйте id_token в закодированном формате с [https://jwt.ms](https://jwt.ms) веб-сайта. Его можно использовать для отправки запроса GET в конечную точку UserInfo и получения сведений о пользователе.
1. Отправьте запрос GET в конечную точку UserInfo и получите сведения о пользователе.

```http
GET /yourtenant.onmicrosoft.com/b2c_1a_signup_signin/openid/v2.0/userinfo
Host: b2cninja.b2clogin.com
Authorization: Bearer <your access token>
```

Успешный ответ будет выглядеть следующим образом:

```json
{
    "objectId": "44444444-4444-4444-4444-444444444444",
    "givenName": "John",
    "surname": "Smith",
    "displayName": "John Smith",
    "signInNames.emailAddress": "john.s@contoso.com"
}
```

## <a name="next-steps"></a>Дальнейшие действия

- Пример политики для конечных точек UserInfo можно найти на сайте [GitHub](https://github.com/azure-ad-b2c/samples/tree/master/policies/user-info-endpoint).

::: zone-end
