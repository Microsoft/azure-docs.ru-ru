---
title: Настройка входа для нескольких клиентов Azure AD с помощью настраиваемых политик
titleSuffix: Azure AD B2C
description: Добавление поставщика удостоверений Azure AD с несколькими клиентами с помощью пользовательских политик в Azure Active Directory B2C.
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
ms.openlocfilehash: f6e932cb6a6086e4cea6f474f296ca086e48c75e
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102448464"
---
# <a name="set-up-sign-in-for-multi-tenant-azure-active-directory-using-custom-policies-in-azure-active-directory-b2c"></a>Настройка входа для мультитенантного Azure Active Directory с помощью пользовательских политик в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

В этой статье показано, как включить вход для пользователей, использующих конечную точку с несколькими клиентами, для Azure Active Directory (Azure AD). Предоставление пользователям из нескольких клиентов Azure AD возможности входа с помощью Azure AD B2C без необходимости настройки поставщика удостоверений для каждого клиента. Однако гостевые пользователи любого из этих тенантов **не смогут** войти в систему. Для этого необходимо [отдельно настроить каждый клиент](identity-provider-azure-ad-single-tenant.md).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="register-an-application"></a>Регистрация приложения

Чтобы включить вход для пользователей с учетной записью Azure AD в Azure Active Directory B2C (Azure AD B2C), необходимо создать приложение в [портал Azure](https://portal.azure.com). Дополнительные сведения см. [в статье регистрация приложения на платформе Microsoft Identity](../active-directory/develop/quickstart-register-app.md).


1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD Организации (например, contoso.com). Выберите **Фильтр каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий ваш клиент.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Регистрация приложений**.
1. Выберите **Новая регистрация**.
1. Введите значение **Name** (Имя) для приложения. Например, `Azure AD B2C App`.
1. Выберите **учетные записи в любом каталоге организации (любой каталог Azure AD — клиент)** для этого приложения.
1. В качестве **URI перенаправления** оставьте значение **Web** и введите следующий URL-адрес в строчных буквах, где `your-B2C-tenant-name` заменяется именем клиента Azure AD B2C.

    ```
    https://your-B2C-tenant-name.b2clogin.com/your-B2C-tenant-name.onmicrosoft.com/oauth2/authresp
    ```

    Например, `https://fabrikam.b2clogin.com/fabrikam.onmicrosoft.com/oauth2/authresp`.

1. Выберите **Зарегистрировать**. Запишите значение параметра **Идентификатор приложения (клиент)** . Оно вам потребуется в дальнейшем.
1. Выберите **сертификаты & секреты**, а затем выберите **новый секрет клиента**.
1. Введите **Описание** секрета, выберите срок действия, а затем нажмите кнопку **Добавить**. Запишите **значение** секрета для использования на следующем шаге.

## <a name="configuring-optional-claims"></a>Настройка необязательных утверждений

Если вы хотите получить `family_name` , а также `given_name` заявки из Azure AD, можно настроить необязательные утверждения для приложения в пользовательском интерфейсе портал Azure или манифесте приложения. Дополнительные сведения см. в статье о [предоставлении необязательных утверждений для приложения Azure AD](../active-directory/develop/active-directory-optional-claims.md).

1. Войдите на [портал Azure](https://portal.azure.com). Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений**.
1. Выберите в списке приложение, для которого нужно настроить необязательные утверждения.
1. В разделе **Управление** выберите **Конфигурация токена**.
1. Выберите **Добавить необязательное утверждение**.
1. Для **типа токена** выберите **идентификатор**.
1. Выберите необязательные утверждения для добавления, `family_name` и `given_name` .
1. Нажмите кнопку **Добавить**.

## <a name="create-a-policy-key"></a>Создание ключа политики

Вам необходимо сохранить ключ приложения, который вы создали в клиенте Azure AD B2C.

1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите **Фильтр каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. В разделе **Политики** выберите **Identity Experience Framework**.
1. Выберите **ключи политики** и нажмите кнопку **Добавить**.
1. Для пункта **Параметры** выберите `Manual`.
1. Введите **имя** ключа политики. Например, `AADAppSecret`.  Префикс `B2C_1A_` добавляется автоматически к имени ключа при его создании, поэтому ссылка в XML-файле в следующем разделе *B2C_1A_AADAppSecret*.
1. В качестве **секрета** введите секрет клиента, записанный ранее.
1. Для параметра **Использование ключа** выберите `Signature`.
1. Нажмите кнопку **создания**.

## <a name="configure-azure-ad-as-an-identity-provider"></a>Настройка Azure AD в качестве поставщика удостоверений для клиента

Чтобы разрешить пользователям входить в систему с помощью учетной записи Azure AD, необходимо определить Azure AD в качестве поставщика утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Вы можете определить Azure AD в качестве поставщика утверждений, добавив Azure AD к элементу **ClaimsProvider** в файле расширения политики.

1. Откройте файл *TrustFrameworkExtensions.xml*.
1. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
1. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>commonaad</Domain>
      <DisplayName>Common AAD</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="AADCommon-OpenIdConnect">
          <DisplayName>Multi-Tenant AAD</DisplayName>
          <Description>Login with your Contoso account</Description>
          <Protocol Name="OpenIdConnect"/>
          <Metadata>
            <Item Key="METADATA">https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration</Item>
            <!-- Update the Client ID below to the Application ID -->
            <Item Key="client_id">00000000-0000-0000-0000-000000000000</Item>
            <Item Key="response_types">code</Item>
            <Item Key="scope">openid profile</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="DiscoverMetadataByTokenIssuer">true</Item>
            <!-- The key below allows you to specify each of the Azure AD tenants that can be used to sign in. Update the GUIDs below for each tenant. -->
            <Item Key="ValidTokenIssuerPrefixes">https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000,https://login.microsoftonline.com/11111111-1111-1111-1111-111111111111</Item>
            <!-- The commented key below specifies that users from any tenant can sign-in. Uncomment if you would like anyone with an Azure AD account to be able to sign in. -->
            <!-- <Item Key="ValidTokenIssuerPrefixes">https://login.microsoftonline.com/</Item> -->
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_AADAppSecret"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid"/>
            <OutputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="tid"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="iss" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

1. В элементе **ClaimsProvider** укажите для **Domain** уникальное значение, позволяющее отличить этот поставщик удостоверений от других.
1. В элементе **техническом профиле** обновите значение **DisplayName**, например `Contoso Employee` . Это значение отображается на кнопке входа на экране входа.
1. Задайте **client_id** идентификатору приложения для приложения с несколькими клиентами Azure AD, зарегистрированного ранее.
1. В разделе **криптографиккэйс** измените значение **идентификатором storagereferenceid** на имя созданного ранее ключа политики. Например, `B2C_1A_AADAppSecret`.

### <a name="restrict-access"></a>Ограничение доступа

Если использовать `https://login.microsoftonline.com/` в качестве значения для **ValidTokenIssuerPrefixes**, все пользователи Azure AD смогут входить в ваше приложение. Обновите список действительных поставщиков маркеров и ограничьте доступ определенным списком пользователей клиента Azure AD, которые могут войти в систему.

Чтобы получить эти значения, просмотрите метаданные обнаружения OpenID Connect Connect для каждого клиента Azure AD, из которого будут входить пользователи. URL-адрес метаданных аналогичен формату `https://login.microsoftonline.com/your-tenant/v2.0/.well-known/openid-configuration` , где `your-tenant` — имя клиента Azure AD. Пример:

`https://login.microsoftonline.com/fabrikam.onmicrosoft.com/v2.0/.well-known/openid-configuration`

Выполните следующие действия для каждого клиента Azure AD, который должен использоваться для входа:

1. Откройте браузер и перейдите по URL-адресу метаданных OpenID Connect Connect для клиента. Найдите объект **Issuer** и запишите его значение. Он должен выглядеть примерно так `https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/` :.
1. Скопируйте и вставьте значение в ключ **ValidTokenIssuerPrefixes** . Несколько издателей разделяются запятыми. Пример с двумя издателями отображается в предыдущем `ClaimsProvider` примере XML.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="AzureADCommonExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="AzureADCommonExchange" TechnicalProfileReferenceId="AADCommon-OpenIdConnect" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Тестирование пользовательской политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице регистрации или входа выберите **Common AAD** для входа с помощью учетной записи Azure AD.

Чтобы протестировать возможность входа с несколькими клиентами, выполните последние два шага, используя учетные данные пользователя, который уже существует в другом клиенте Azure AD. Скопируйте **конечную точку "запустить сейчас** " и откройте ее в частном окне браузера, например в режиме режиме инкогнито в Google Chrome или в окне InPrivate в Microsoft погранично. Открытие в частном окне браузера позволяет протестировать полное путешествие пользователя, не используя кэшированные учетные данные Azure AD.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

## <a name="next-steps"></a>Дальнейшие действия

При работе с пользовательскими политиками иногда может потребоваться дополнительная информация при устранении неполадок политики во время ее разработки.

Чтобы помочь в диагностике проблем, можно временно перевести политику в режим разработчика и собрать журналы с помощью Azure Application Insights. Узнайте, как в [Azure Active Directory B2C: сбор журналов](troubleshoot-with-application-insights.md).

::: zone-end
