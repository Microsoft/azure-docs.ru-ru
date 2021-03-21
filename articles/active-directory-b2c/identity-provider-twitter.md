---
title: Настройка регистрации и входа с помощью учетной записи Twitter
titleSuffix: Azure AD B2C
description: Вы можете организовать в приложениях регистрацию и вход для клиентов с учетными записями Twitter, используя Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/17/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 3699743c5d1b3330715984d2b6116cfebafe74f1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104579847"
---
# <a name="set-up-sign-up-and-sign-in-with-a-twitter-account-using-azure-active-directory-b2c"></a>Настройка регистрации и входа с учетной записью Twitter через Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]
::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="create-an-application"></a>Создание приложения

Чтобы включить вход для пользователей с учетной записью Twitter в Azure AD B2C, необходимо создать приложение Twitter. Если у вас еще нет учетной записи Twitter, вы можете зарегистрироваться по адресу [https://twitter.com/signup](https://twitter.com/signup) . Также необходимо [Применить для учетной записи разработчика](https://developer.twitter.com/en/apply/user.html). Дополнительные сведения см. в разделе [применение для доступа](https://developer.twitter.com/en/apply-for-access).

1. Войдите на [портал разработчиков Twitter](https://developer.twitter.com/portal/projects-and-apps) , используя учетные данные вашей учетной записи Twitter.
1. В разделе **автономные приложения** выберите **+ создать приложение**.
1. Введите **имя приложения** и нажмите кнопку **завершить**.
1. Скопируйте значение **ключа приложения** и **секрет ключа API**.  Они используются для настройки Twitter в качестве поставщика удостоверений в клиенте. 
1. В разделе **Настройка приложения** выберите **Параметры приложения**.
1. В разделе **Параметры проверки подлинности** выберите **изменить** .
    1. Установите флажок **включить 3-legged OAuth** .
    1. Установите флажок **запросить адрес электронной почты у пользователей** .
    1. Для **URL-адресов обратного вызова** введите `https://your-tenant.b2clogin.com/your-tenant-name.onmicrosoft.com/your-user-flow-Id/oauth1/authresp` .  Если используется [личный домен](custom-domain.md), введите `https://your-domain-name/your-tenant-name.onmicrosoft.com/your-user-flow-Id/oauth1/authresp` . Используйте все строчные буквы при вводе имени клиента и идентификатора потока пользователя, даже если они определены с прописными буквами в Azure AD B2C. Замените
        - `your-tenant-name` с именем своего имени клиента.
        - `your-domain-name` с пользовательским доменом.
        - `your-user-flow-Id` с идентификатором потока пользователя. Например, `b2c_1a_signup_signin_twitter`. 
    
    1. Введите в качестве **URL-адреса веб-сайта** `https://your-tenant.b2clogin.com` . Замените `your-tenant` именем вашего клиента. Например, `https://contosob2c.b2clogin.com`. Если используется [личный домен](custom-domain.md), введите `https://your-domain-name` .
    1. Введите URL-адрес **условий предоставления услуг**, например `http://www.contoso.com/tos` . URL-адрес политики — это страница, которую вы поддерживаете для предоставления условий для приложения.
    1. Введите URL-адрес **политики конфиденциальности**, например `http://www.contoso.com/privacy` . URL-адрес политики — это управляемая вами страница, где предоставляются сведения о конфиденциальности для вашего приложения.
    1. Щелкните **Сохранить**.

::: zone pivot="b2c-user-flow"

## <a name="configure-twitter-as-an-identity-provider"></a>Настройка Twitter в качестве поставщика удостоверений

1. Войдите на [портал Azure](https://portal.azure.com/) с правами глобального администратора клиента Azure AD B2C.
1. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, найдите службу **Azure AD B2C** и выберите ее.
1. Выберите **поставщики удостоверений**, а затем выберите **Twitter**.
1. Введите **Имя**. Например, *Twitter*.
1. В поле **идентификатор клиента** введите *ключ API* приложения Twitter, созданного ранее.
1. В поле **секрет клиента** введите *секретный ключ API* , который вы записали.
1. Щелкните **Сохранить**.

## <a name="add-twitter-identity-provider-to-a-user-flow"></a>Добавление поставщика удостоверений Twitter в поток пользователя 

На этом этапе поставщик удостоверений Twitter настроен, но еще не доступен ни на одной из страниц входа. Чтобы добавить поставщик удостоверений Twitter в поток пользователя, выполните следующие действия.

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Выберите поток пользователя, который требуется добавить в качестве поставщика удостоверений Twitter.
1. В разделе **поставщики удостоверений социальных сетей** выберите **Twitter**.
1. Щелкните **Сохранить**.
1. Чтобы проверить политику, выберите пункт **выполнить пользовательскую последовательность**.
1. Для **приложения** выберите веб-приложение с именем *testapp1* , которое вы зарегистрировали ранее. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **запустить поток пользователя** .
1. На странице регистрации или входа выберите **Twitter** для входа с помощью учетной записи Twitter.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="create-a-policy-key"></a>Создание ключа политики

Сохраните секретный ключ, который ранее был записан в клиенте Azure AD B2C.

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню и выберите каталог, который содержит ваш клиент.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
4. На странице "Обзор" выберите **Identity Experience Framework**.
5. Выберите **Ключи политики**, а затем щелкните **Добавить**.
6. Для пункта **Параметры** выберите `Manual`.
7. Введите **имя** ключа политики. Например, `TwitterSecret`. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
8. В поле **Секрет** введите ранее записанный секрет клиента.
9. Для параметра **Использование ключа** выберите `Encryption`.
10. Нажмите кнопку **Создать**.

## <a name="configure-twitter-as-an-identity-provider"></a>Настройка Twitter в качестве поставщика удостоверений

Чтобы пользователи могли входить с помощью учетной записи Twitter, необходимо определить учетную запись в качестве поставщика утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Чтобы определить учетную запись Twitter в качестве поставщика утверждений, можно добавить ее в элемент **ClaimsProviders** в файле расширения политики.

1. Откройте файл *TrustFrameworkExtensions.xml*.
2. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
3. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>twitter.com</Domain>
      <DisplayName>Twitter</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Twitter-OAuth1">
          <DisplayName>Twitter</DisplayName>
          <Protocol Name="OAuth1" />
          <Metadata>
            <Item Key="ProviderName">Twitter</Item>
            <Item Key="authorization_endpoint">https://api.twitter.com/oauth/authenticate</Item>
            <Item Key="access_token_endpoint">https://api.twitter.com/oauth/access_token</Item>
            <Item Key="request_token_endpoint">https://api.twitter.com/oauth/request_token</Item>
            <Item Key="ClaimsEndpoint">https://api.twitter.com/1.1/account/verify_credentials.json?include_email=true</Item>
            <Item Key="ClaimsResponseFormat">json</Item>
            <Item Key="client_id">Your Twitter application API key</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_TwitterSecret" />
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="user_id" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="screen_name" />
            <OutputClaim ClaimTypeReferenceId="email" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="twitter.com" />
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

4. Замените значение **client_id** на *секретный ключ API* , который вы записали ранее.
5. Сохраните файл.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="TwitterExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="TwitterExchange" TechnicalProfileReferenceId="Twitter-OAuth1" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Тестирование настраиваемой политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице регистрации или входа выберите **Twitter** для входа с помощью учетной записи Twitter.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end
