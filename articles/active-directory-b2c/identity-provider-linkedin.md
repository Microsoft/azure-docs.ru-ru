---
title: Настройка регистрации и входа с помощью учетной записи LinkedIn
titleSuffix: Azure AD B2C
description: Вы можете организовать в приложениях регистрацию и вход для клиентов с учетными записями LinkedIn, используя Azure Active Directory B2C.
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
ms.openlocfilehash: ce5e8cfda4a9f51a90c8f26133a710f4d1c258b6
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102448274"
---
# <a name="set-up-sign-up-and-sign-in-with-a-linkedin-account-using-azure-active-directory-b2c"></a>Настройка регистрации и входа с учетной записью LinkedIn через Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="create-a-linkedin-application"></a>Создание приложения LinkedIn

Чтобы включить вход для пользователей с учетной записью LinkedIn в Azure Active Directory B2C (Azure AD B2C), необходимо создать приложение на [веб-сайте LinkedIn Developers](https://www.developer.linkedin.com/). Дополнительные сведения см. в разделе [поток кода авторизации](/linkedin/shared/authentication/authorization-code-flow). Если у вас еще нет учетной записи LinkedIn, вы можете зарегистрироваться по адресу [https://www.linkedin.com/](https://www.linkedin.com/) .

1. Выполните вход на [сайт разработчиков LinkedIn](https://www.developer.linkedin.com/) с учетными данными для учетной записи LinkedIn.
1. Выберите **Мои приложения**, а затем щелкните **создать приложение**.
1. Введите **имя приложения**, **страницу LinkedIn**, **URL-адрес политики конфиденциальности** и **эмблему приложения**.
1. Примите **условия использования API** LinkedIn и нажмите кнопку **создать приложение**.
1. Перейдите на вкладку **Проверка подлинности** . В разделе **ключи проверки подлинности** СКОПИРУЙТЕ значения **идентификатор клиента** и **секрет клиента**. Оба они понадобятся для настройки LinkedIn в качестве поставщика удостоверений в клиенте. **Секрет клиента**— важный элемент обеспечения безопасности.
1. Щелкните значок изменить карандаш рядом с **разадресованными URL-адресами перенаправления для приложения**, а затем выберите **Добавить URL-адрес перенаправления**. Введите `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp` , заменив `your-tenant-name` именем своего клиента. При вводе имени вашего клиента необходимо использовать только строчные буквы, даже если в Azure AD B2C имя клиента определено с прописными буквами. Щелкните **Обновить**.
2. По умолчанию приложение LinkedIn не утверждается для областей, связанных с входом. Чтобы запросить проверку, перейдите на вкладку **продукты** и выберите **Вход с помощью LinkedIn**. После завершения проверки необходимые области будут добавлены в приложение.
   > [!NOTE]
   > Вы можете просмотреть области, которые в настоящее время разрешены для приложения, на вкладке **Проверка подлинности** в разделе **области OAuth 2,0** .

::: zone pivot="b2c-user-flow"

## <a name="configure-linkedin-as-an-identity-provider"></a>Настройка LinkedIn в качестве поставщика удостоверений

1. Войдите на [портал Azure](https://portal.azure.com/) с правами глобального администратора клиента Azure AD B2C.
1. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, найдите службу **Azure AD B2C** и выберите ее.
1. Выберите **поставщики удостоверений**, а затем выберите **LinkedIn**.
1. Введите **Имя**. Например, *LinkedIn*.
1. В поле **идентификатор клиента** введите идентификатор клиента приложения LinkedIn, созданного ранее.
1. В качестве **секрета клиента** введите записанный секрет клиента.
1. Щелкните **Сохранить**.

## <a name="add-linkedin-identity-provider-to-a-user-flow"></a>Добавление поставщика удостоверений LinkedIn в поток пользователя 

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Щелкните поток пользователя, который необходимо добавить в качестве поставщика удостоверений LinkedIn.
1. В разделе **поставщики удостоверений социальных сетей** выберите **LinkedIn**.
1. Щелкните **Сохранить**.
1. Чтобы проверить политику, выберите пункт **выполнить пользовательскую последовательность**.
1. Для **приложения** выберите веб-приложение с именем *testapp1* , которое вы зарегистрировали ранее. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **запустить поток пользователя** .
1. На странице Регистрация или вход выберите **LinkedIn** , чтобы войти с помощью учетной записи LinkedIn.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="create-a-policy-key"></a>Создание ключа политики

Вам необходимо сохранить секрет клиента, который ранее был записан в клиенте Azure AD B2C.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню и выберите каталог, который содержит ваш клиент.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
4. На странице "Обзор" выберите **Identity Experience Framework**.
5. Выберите **ключи политики** и нажмите кнопку **Добавить**.
6. Для пункта **Параметры** выберите `Manual`.
7. Введите **имя** ключа политики. Например, `LinkedInSecret`. Префикс *B2C_1A_* автоматически добавляется к имени ключа.
8. В поле **секрет** введите секрет клиента, который вы записали ранее.
9. Для параметра **Использование ключа** выберите `Signature`.
10. Нажмите кнопку **Создать**.

## <a name="configure-linkedin-as-an-identity-provider"></a>Настройка LinkedIn в качестве поставщика удостоверений

Чтобы пользователи могли входить с помощью учетной записи LinkedIn, необходимо определить учетную запись в качестве поставщика утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Определите учетную запись LinkedIn в качестве поставщика утверждений, добавив ее в элемент **клаимспровидерс** в файле расширения политики.

1. Откройте файл *SocialAndLocalAccounts/* * TrustFrameworkExtensions.xml** * в редакторе. Этот файл находится в [начальном пакете пользовательской политики][starter-pack] , скачанном в рамках одного из необходимых компонентов.
1. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
1. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>linkedin.com</Domain>
      <DisplayName>LinkedIn</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="LinkedIn-OAuth2">
          <DisplayName>LinkedIn</DisplayName>
          <Protocol Name="OAuth2" />
          <Metadata>
            <Item Key="ProviderName">linkedin</Item>
            <Item Key="authorization_endpoint">https://www.linkedin.com/oauth/v2/authorization</Item>
            <Item Key="AccessTokenEndpoint">https://www.linkedin.com/oauth/v2/accessToken</Item>
            <Item Key="ClaimsEndpoint">https://api.linkedin.com/v2/me</Item>
            <Item Key="scope">r_emailaddress r_liteprofile</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="external_user_identity_claim_id">id</Item>
            <Item Key="BearerTokenTransmissionMethod">AuthorizationHeader</Item>
            <Item Key="ResolveJsonPathsInJsonTokens">true</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="client_id">Your LinkedIn application client ID</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_LinkedInSecret" />
          </CryptographicKeys>
          <InputClaims />
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="id" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="firstName.localized" />
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="lastName.localized" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="linkedin.com" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="ExtractGivenNameFromLinkedInResponse" />
            <OutputClaimsTransformation ReferenceId="ExtractSurNameFromLinkedInResponse" />
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName" />
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName" />
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId" />
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId" />
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. Замените значение **client_id** на идентификатор клиента приложения LinkedIn, которое вы записали ранее.
1. Сохраните файл.

### <a name="add-the-claims-transformations"></a>Добавление преобразований утверждений

В техническом профиле LinkedIn необходимо добавить преобразования утверждений **екстрактгивеннамефромлинкединреспонсе** и **Екстрактсурнамефромлинкединреспонсе** в список клаимстрансформатионс. Если в файле не определен элемент **клаимстрансформатионс** , добавьте РОДИТЕЛЬСКИЕ XML-элементы, как показано ниже. Для преобразований утверждений также требуется новый тип утверждения, определенный с именем **нуллстрингклаим**.

Добавьте элемент **BuildingBlocks** в начало файла *TrustFrameworkExtensions.xml* . См. *TrustFrameworkBase.xml* в качестве примера.

```xml
<BuildingBlocks>
  <ClaimsSchema>
    <!-- Claim type needed for LinkedIn claims transformations -->
    <ClaimType Id="nullStringClaim">
      <DisplayName>nullClaim</DisplayName>
      <DataType>string</DataType>
      <AdminHelpText>A policy claim to store output values from ClaimsTransformations that aren't useful. This claim should not be used in TechnicalProfiles.</AdminHelpText>
      <UserHelpText>A policy claim to store output values from ClaimsTransformations that aren't useful. This claim should not be used in TechnicalProfiles.</UserHelpText>
    </ClaimType>
  </ClaimsSchema>

  <ClaimsTransformations>
    <!-- Claim transformations needed for LinkedIn technical profile -->
    <ClaimsTransformation Id="ExtractGivenNameFromLinkedInResponse" TransformationMethod="GetSingleItemFromJson">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="inputJson" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="nullStringClaim" TransformationClaimType="key" />
        <OutputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="value" />
      </OutputClaims>
    </ClaimsTransformation>
    <ClaimsTransformation Id="ExtractSurNameFromLinkedInResponse" TransformationMethod="GetSingleItemFromJson">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="surname" TransformationClaimType="inputJson" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="nullStringClaim" TransformationClaimType="key" />
        <OutputClaim ClaimTypeReferenceId="surname" TransformationClaimType="value" />
      </OutputClaims>
    </ClaimsTransformation>
  </ClaimsTransformations>
</BuildingBlocks>
```

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="LinkedInExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="LinkedInExchange" TechnicalProfileReferenceId="LinkedIn-OAuth2" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Тестирование пользовательской политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице Регистрация или вход выберите **LinkedIn** , чтобы войти с помощью учетной записи LinkedIn.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

## <a name="migration-from-v10-to-v20"></a>Миграция с версии 1.0 на v 2.0

Мы недавно [обновили API-интерфейсы с версии 1.0 до версии 2.0](https://engineering.linkedin.com/blog/2018/12/developer-program-updates). Чтобы перенести существующую конфигурацию в новую конфигурацию, используйте сведения в следующих разделах для обновления элементов в техническом профиле.

### <a name="replace-items-in-the-metadata"></a>Замена элементов в метаданных

В существующем элементе **Metadata** элемента **техническом профиле** обновите следующие элементы **элементов** из:

```xml
<Item Key="ClaimsEndpoint">https://api.linkedin.com/v1/people/~:(id,first-name,last-name,email-address,headline)</Item>
<Item Key="scope">r_emailaddress r_basicprofile</Item>
```

на:

```xml
<Item Key="ClaimsEndpoint">https://api.linkedin.com/v2/me</Item>
<Item Key="scope">r_emailaddress r_liteprofile</Item>
```

### <a name="add-items-to-the-metadata"></a>Добавление элементов в метаданные

В **метаданных** **техническом профиле** добавьте следующие элементы **Item** :

```xml
<Item Key="external_user_identity_claim_id">id</Item>
<Item Key="BearerTokenTransmissionMethod">AuthorizationHeader</Item>
<Item Key="ResolveJsonPathsInJsonTokens">true</Item>
```

### <a name="update-the-outputclaims"></a>Обновление OutputClaims

В существующей **OutputClaims** **техническом профиле** обновите следующие элементы **OutputClaim** из:

```xml
<OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="firstName" />
<OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="lastName" />
```

на:

```xml
<OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="firstName.localized" />
<OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="lastName.localized" />
```

### <a name="add-new-outputclaimstransformation-elements"></a>Добавление новых элементов Аутпутклаимстрансформатион

В **Аутпутклаимстрансформатионс** **техническом профиле** добавьте следующие элементы **аутпутклаимстрансформатион** :

```xml
<OutputClaimsTransformation ReferenceId="ExtractGivenNameFromLinkedInResponse" />
<OutputClaimsTransformation ReferenceId="ExtractSurNameFromLinkedInResponse" />
```

### <a name="define-the-new-claims-transformations-and-claim-type"></a>Определение новых преобразований утверждений и типа утверждения

На последнем шаге были добавлены новые преобразования утверждений, которые необходимо определить. Чтобы определить преобразования утверждений, добавьте их в список **клаимстрансформатионс**. Если в файле не определен элемент **клаимстрансформатионс** , добавьте РОДИТЕЛЬСКИЕ XML-элементы, как показано ниже. Для преобразований утверждений также требуется новый тип утверждения, определенный с именем **нуллстрингклаим**.

Элемент **BuildingBlocks** должен быть добавлен в начало файла. См. *TrustframeworkBase.xml* в качестве примера.

```xml
<BuildingBlocks>
  <ClaimsSchema>
    <!-- Claim type needed for LinkedIn claims transformations -->
    <ClaimType Id="nullStringClaim">
      <DisplayName>nullClaim</DisplayName>
      <DataType>string</DataType>
      <AdminHelpText>A policy claim to store unuseful output values from ClaimsTransformations. This claim should not be used in a TechnicalProfiles.</AdminHelpText>
      <UserHelpText>A policy claim to store unuseful output values from ClaimsTransformations. This claim should not be used in a TechnicalProfiles.</UserHelpText>
    </ClaimType>
  </ClaimsSchema>

  <ClaimsTransformations>
    <!-- Claim transformations needed for LinkedIn technical profile -->
    <ClaimsTransformation Id="ExtractGivenNameFromLinkedInResponse" TransformationMethod="GetSingleItemFromJson">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="inputJson" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="nullStringClaim" TransformationClaimType="key" />
        <OutputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="value" />
      </OutputClaims>
    </ClaimsTransformation>
    <ClaimsTransformation Id="ExtractSurNameFromLinkedInResponse" TransformationMethod="GetSingleItemFromJson">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="surname" TransformationClaimType="inputJson" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="nullStringClaim" TransformationClaimType="key" />
        <OutputClaim ClaimTypeReferenceId="surname" TransformationClaimType="value" />
      </OutputClaims>
    </ClaimsTransformation>
  </ClaimsTransformations>
</BuildingBlocks>
```

### <a name="obtain-an-email-address"></a>Получение адреса электронной почты

В рамках миграции LinkedIn с версии 1.0 на версию 2.0 для получения адреса электронной почты требуется дополнительный вызов другого API. Если необходимо получить адрес электронной почты во время регистрации, выполните следующие действия.

1. Выполните описанные выше действия, чтобы разрешить Azure AD B2C Федерацию с LinkedIn, чтобы пользователь мог войти в систему. В рамках Федерации Azure AD B2C получает маркер доступа для LinkedIn.
1. Сохраните маркер доступа LinkedIn в заявке. [См. инструкции здесь](idp-pass-through-user-flow.md).
1. Добавьте следующий поставщик утверждений, который выполняет запрос к API LinkedIn `/emailAddress` . Чтобы авторизовать этот запрос, необходим маркер доступа LinkedIn.

    ```xml
    <ClaimsProvider>
      <DisplayName>REST APIs</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="API-LinkedInEmail">
          <DisplayName>Get LinkedIn email</DisplayName>
          <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
          <Metadata>
              <Item Key="ServiceUrl">https://api.linkedin.com/v2/emailAddress?q=members&amp;projection=(elements*(handle~))</Item>
              <Item Key="AuthenticationType">Bearer</Item>
              <Item Key="UseClaimAsBearerToken">identityProviderAccessToken</Item>
              <Item Key="SendClaimsIn">Url</Item>
              <Item Key="ResolveJsonPathsInJsonTokens">true</Item>
          </Metadata>
          <InputClaims>
              <InputClaim ClaimTypeReferenceId="identityProviderAccessToken" />
          </InputClaims>
          <OutputClaims>
              <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="elements[0].handle~.emailAddress" />
          </OutputClaims>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. Добавьте следующий шаг оркестрации в путь взаимодействия пользователя, чтобы поставщик утверждений API активировался при входе пользователя с помощью LinkedIn. Обязательно обновите `Order` номер соответствующим образом. Добавьте этот шаг сразу после шага оркестрации, запускающего технический профиль LinkedIn.

    ```xml
    <!-- Extra step for LinkedIn to get the email -->
    <OrchestrationStep Order="3" Type="ClaimsExchange">
      <Preconditions>
        <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
          <Value>identityProvider</Value>
          <Action>SkipThisOrchestrationStep</Action>
        </Precondition>
        <Precondition Type="ClaimEquals" ExecuteActionsIf="false">
          <Value>identityProvider</Value>
          <Value>linkedin.com</Value>
          <Action>SkipThisOrchestrationStep</Action>
        </Precondition>
      </Preconditions>
      <ClaimsExchanges>
        <ClaimsExchange Id="GetEmail" TechnicalProfileReferenceId="API-LinkedInEmail" />
      </ClaimsExchanges>
    </OrchestrationStep>
    ```

Получение адреса электронной почты из LinkedIn во время регистрации является необязательным. Если вы решили не получать сообщение электронной почты из LinkedIn, но во время регистрации его потребуется, пользователь должен вручную ввести адрес электронной почты и проверить его.

Полный пример политики, использующей поставщик удостоверений LinkedIn, см. в разделе [начальный пакет настраиваемой политики](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/linkedin-identity-provider).

<!-- Links - EXTERNAL -->
[starter-pack]: https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack

::: zone-end

::: zone pivot="b2c-user-flow"

## <a name="migration-from-v10-to-v20"></a>Миграция с версии 1.0 на v 2.0

Мы недавно [обновили API-интерфейсы с версии 1.0 до версии 2.0](https://engineering.linkedin.com/blog/2018/12/developer-program-updates). В ходе миграции Azure AD B2C удается получить полное имя пользователя LinkedIn во время регистрации... Если адрес электронной почты является одним из атрибутов, собираемых во время регистрации, пользователь должен вручную ввести адрес электронной почты и проверить его.

::: zone-end