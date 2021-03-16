---
title: Настройка регистрации и входа с помощью учетной записи Google
titleSuffix: Azure AD B2C
description: Вы можете организовать в приложениях регистрацию и вход для клиентов с учетными записями Google, используя Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 433e44e9998376d0bea57264f3daeaff7499a3cf
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103488845"
---
# <a name="set-up-sign-up-and-sign-in-with-a-google-account-using-azure-active-directory-b2c"></a>Настройка регистрации и входа с учетной записью Google через Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]


## <a name="create-a-google-application"></a>Создание приложения Google

Чтобы включить вход для пользователей с учетной записью Google в Azure Active Directory B2C (Azure AD B2C), необходимо создать приложение в [консоли Google Developers](https://console.developers.google.com/). Дополнительные сведения см. в разделе [Настройка OAuth 2,0](https://support.google.com/googleapi/answer/6158849). Если у вас еще нет учетной записи Google, вы можете зарегистрироваться по адресу [https://accounts.google.com/SignUp](https://accounts.google.com/SignUp) .

1. Выполните вход в [консоль разработчиков Google](https://console.developers.google.com/) с учетными данными для учетной записи Google.
1. В левом верхнем углу страницы выберите список проект, а затем выберите **Новый проект**.
1. Введите **имя проекта** и нажмите кнопку **создать**.
1. Убедитесь, что вы используете новый проект, выбрав раскрывающийся список проект в верхней левой части экрана. Выберите свой проект по имени, а затем нажмите кнопку **Открыть**.
1. В меню слева щелкните **экран согласия OAuth** , выберите **внешний** и щелкните **создать**.
Введите значение **Name** (Имя) для приложения. Введите *b2clogin.com* в разделе " **полномочные домены** " и нажмите кнопку **сохранить**.
1. Выберите **Учетные данные** в меню слева, а затем выберите **Создать учетные данные** > **Идентификатор клиента Oauth**.
1. Из списка **Application type** (Тип приложения) выберите **Web application** (Веб-приложение).
    1. Введите значение **Name** (Имя) для приложения.
    1. Для **утвержденных источников JavaScript** введите `https://your-tenant-name.b2clogin.com` . Если используется [личный домен](custom-domain.md), введите `https://your-domain-name` .
    1. Для **правых URI перенаправления** введите `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp` . Если используется [личный домен](custom-domain.md), введите `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp` . Замените `your-domain-name` пользовательским доменом и `your-tenant-name` именем своего клиента. При вводе имени вашего клиента используйте только строчные буквы, даже если в Azure AD B2C имя клиента определено с помощью прописных букв.
1. Нажмите кнопку **Создать**.
1. Скопируйте значения **идентификатора клиента** и **секрета клиента**. Оба значения потребуются для настройки Google в качестве поставщика удостоверений в вашем клиенте. **Секрет клиента** — это важные учетные данные безопасности.

::: zone pivot="b2c-user-flow"

## <a name="configure-google-as-an-identity-provider"></a>Настройка Google в качестве поставщика удостоверений

1. Войдите на [портал Azure](https://portal.azure.com/) с правами глобального администратора клиента Azure AD B2C.
1. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, найдите службу **Azure AD B2C** и выберите ее.
1. Выберите **поставщики удостоверений**, а затем выберите **Google**.
1. Введите **Имя**. Например, *Google*.
1. В поле **идентификатор клиента** введите идентификатор клиента приложения Google, созданного ранее.
1. В качестве **секрета клиента** введите записанный секрет клиента.
1. Щелкните **Сохранить**.

## <a name="add-google-identity-provider-to-a-user-flow"></a>Добавление поставщика удостоверений Google в поток пользователя 

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Щелкните поток пользователя, для которого требуется добавить поставщик удостоверений Google.
1. В разделе **поставщики удостоверений социальных сетей** выберите **Google**.
1. Щелкните **Сохранить**.
1. Чтобы проверить политику, выберите пункт **выполнить пользовательскую последовательность**.
1. Для **приложения** выберите веб-приложение с именем *testapp1* , которое вы зарегистрировали ранее. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **запустить поток пользователя** .
1. На странице регистрации или входа выберите **Google** , чтобы войти с помощью учетной записи Google.

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
7. Введите **имя** ключа политики. Например, `GoogleSecret`. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
8. В поле **Секрет** введите ранее записанный секрет клиента.
9. Для параметра **Использование ключа** выберите `Signature`.
10. Нажмите кнопку **Создать**.

## <a name="configure-google-as-an-identity-provider"></a>Настройка Google в качестве поставщика удостоверений

Чтобы пользователи могли входить с помощью учетной записи Google, необходимо определить учетную запись в качестве поставщика утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Определить учетную запись Google в качестве поставщика утверждений можно путем добавления ее в элемент **ClaimsProviders** в файле расширения политики.

1. Откройте файл *TrustFrameworkExtensions.xml*.
2. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
3. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>google.com</Domain>
      <DisplayName>Google</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Google-OAuth2">
          <DisplayName>Google</DisplayName>
          <Protocol Name="OAuth2" />
          <Metadata>
            <Item Key="ProviderName">google</Item>
            <Item Key="authorization_endpoint">https://accounts.google.com/o/oauth2/auth</Item>
            <Item Key="AccessTokenEndpoint">https://accounts.google.com/o/oauth2/token</Item>
            <Item Key="ClaimsEndpoint">https://www.googleapis.com/oauth2/v1/userinfo</Item>
            <Item Key="scope">email profile</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="client_id">Your Google application ID</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_GoogleSecret" />
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="id" />
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="google.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
          </OutputClaims>
          <OutputClaimsTransformations>
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

4. Задайте для параметра **client_id** значение идентификатора приложения из регистрации приложения.
5. Сохраните файл.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="GoogleExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAuth2" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Тестирование настраиваемой политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице регистрации или входа выберите **Google** , чтобы войти с помощью учетной записи Google.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [передавать токен Google в приложение](idp-pass-through-user-flow.md).