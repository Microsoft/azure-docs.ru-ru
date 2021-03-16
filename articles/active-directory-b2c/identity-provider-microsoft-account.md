---
title: Настройка регистрации и входа с учетной записью Microsoft
titleSuffix: Azure AD B2C
description: Вы можете организовать в приложениях регистрацию и вход для клиентов с учетными записями Microsoft, используя Azure Active Directory B2C.
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
ms.openlocfilehash: db6793389df0df169745ddd50551f836d06cf08c
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103488760"
---
# <a name="set-up-sign-up-and-sign-in-with-a-microsoft-account-using-azure-active-directory-b2c"></a>Настройка регистрации и входа с учетной записью Microsoft через Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="create-a-microsoft-account-application"></a>Создание приложения для учетной записи Майкрософт

Чтобы включить вход для пользователей с учетная запись Майкрософт в Azure Active Directory B2C (Azure AD B2C), необходимо создать приложение в [портал Azure](https://portal.azure.com). Дополнительные сведения см. [в статье регистрация приложения на платформе Microsoft Identity](../active-directory/develop/quickstart-register-app.md). Если у вас нет учетной записи Microsoft, вы можете получить ее по адресу [https://www.live.com/](https://www.live.com/).

1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что вы используете каталог с клиентом Azure AD, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом Azure AD.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Регистрация приложений**.
1. Выберите **Новая регистрация**.
1. Введите значение **Name** (Имя) для приложения. Например, *MSAapp1*.
1. В поле **Поддерживаемые типы учетных записей** выберите вариант **Учетные записи в любом каталоге организации (любой каталог Azure AD — мультитенантный) и персональные учетные записи Майкрософт (например, Skype, Xbox)** .

   Дополнительные сведения о различных вариантах выбора типа учетной записи см. в статье [Краткое руководство. Регистрация приложения с помощью платформы удостоверений Майкрософт](../active-directory/develop/quickstart-register-app.md)
1. В разделе **URI перенаправления (необязательно)** выберите **веб** и введите `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp` . Если используется [личный домен](custom-domain.md), введите `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp` . Замените `your-tenant-name` именем своего клиента и `your-domain-name` личным доменом.
1. Нажмите кнопку **Зарегистрировать**.
1. Запишите значение параметра **Идентификатор приложения (клиента)** , отображаемого на странице обзора приложения. При настройке поставщика удостоверений в следующем разделе вам потребуется идентификатор клиента.
1. Выберите **Сертификаты и секреты**.
1. Щелкните **Создать секрет клиента**.
1. В поле **Описание** введите описание секрета, например *Пароль приложения 1*, а затем нажмите кнопку **Добавить**.
1. Запишите пароль приложения, отображаемый в столбце **Значение**. При настройке поставщика удостоверений в следующем разделе требуется секрет клиента.

::: zone pivot="b2c-user-flow"

## <a name="configure-microsoft-as-an-identity-provider"></a>Настройка Microsoft в качестве поставщика удостоверений

1. Войдите на [портал Azure](https://portal.azure.com/) с правами глобального администратора клиента Azure AD B2C.
1. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, найдите службу **Azure AD B2C** и выберите ее.
1. Выберите **Поставщики удостоверений**, а затем — **Учетная запись Microsoft**.
1. Введите **Имя**. Например, *MSA*.
1. В качестве **идентификатора клиента** введите идентификатор приложения (клиента) для приложения Azure AD, созданного ранее.
1. В качестве **секрета клиента** введите секрет клиента, записанный ранее.
1. Щелкните **Сохранить**.

## <a name="add-microsoft-identity-provider-to-a-user-flow"></a>Добавление поставщика удостоверений Майкрософт в поток пользователя 

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Щелкните поток пользователя, для которого требуется добавить поставщик удостоверений Майкрософт.
1. В разделе **поставщики удостоверений социальных сетей** выберите **учетная запись Майкрософт**.
1. Щелкните **Сохранить**.
1. Чтобы проверить политику, выберите пункт **выполнить пользовательскую последовательность**.
1. Для **приложения** выберите веб-приложение с именем *testapp1* , которое вы зарегистрировали ранее. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **запустить поток пользователя** .
1. На странице регистрации или входа выберите **Майкрософт** для входа с помощью учетная запись Майкрософт.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="configuring-optional-claims"></a>Настройка необязательных утверждений

Чтобы получить утверждения `family_name` и `given_name` из Azure AD, можно настроить дополнительные утверждения для приложения в пользовательском интерфейсе портала Azure или манифесте приложения. Дополнительные сведения см. в статье о [предоставлении необязательных утверждений для приложения Azure AD](../active-directory/develop/active-directory-optional-claims.md).

1. Войдите на [портал Azure](https://portal.azure.com). Найдите и выберите **Azure Active Directory**.
1. В разделе **Управление** выберите **Регистрация приложений**.
1. Выберите в списке приложение, для которого нужно настроить необязательные утверждения.
1. В разделе **Управление** выберите **Конфигурация токенов (предварительная версия)** .
1. Выберите **Добавить необязательное утверждение**.
1. Выберите тип токена, который нужно настроить.
1. Выберите необязательные утверждения для добавления.
1. Нажмите кнопку **Добавить**.

## <a name="create-a-policy-key"></a>Создание ключа политики

Теперь, когда вы создали приложение в клиенте Azure AD, необходимо сохранить секрет клиента этого приложения в клиенте Azure AD B2C.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню и выберите каталог, который содержит ваш клиент.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
1. На странице "Обзор" выберите **Identity Experience Framework**.
1. Выберите **Ключи политики**, а затем щелкните **Добавить**.
1. Для пункта **Параметры** выберите `Manual`.
1. Введите **имя** ключа политики. Например, `MSASecret`. Префикс `B2C_1A_` будет автоматически добавлен к имени ключа.
1. В поле **Секрет** введите секрет клиента, записанный в предыдущем разделе.
1. Для параметра **Использование ключа** выберите `Signature`.
1. Нажмите кнопку **Создать**.

## <a name="configure-microsoft-as-an-identity-provider"></a>Настройка Microsoft в качестве поставщика удостоверений

Чтобы разрешить пользователям выполнять вход с помощью учетная запись Майкрософт, необходимо определить учетную запись как поставщика утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, используемых Azure AD B2C, чтобы проверить, была ли выполнена проверка подлинности определенного пользователя.

Вы можете определить Azure AD в качестве поставщика утверждений, добавив элемент **ClaimsProvider** в файл расширения политики.

1. Откройте файл политики *TrustFrameworkExtensions.xml*.
1. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
1. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>live.com</Domain>
      <DisplayName>Microsoft Account</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="MSA-MicrosoftAccount-OpenIdConnect">
          <DisplayName>Microsoft Account</DisplayName>
          <Protocol Name="OpenIdConnect" />
          <Metadata>
            <Item Key="ProviderName">https://login.live.com</Item>
            <Item Key="METADATA">https://login.live.com/.well-known/openid-configuration</Item>
            <Item Key="response_types">code</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="scope">openid profile email</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="client_id">Your Microsoft application client ID</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_MSASecret" />
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="iss" />
            <OutputClaim ClaimTypeReferenceId="email" />
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

1. Замените значение **client_id** *идентификатором приложения (клиента)* приложения Azure AD, записанным ранее.
1. Сохраните файл.

Политика настроена, так что Azure AD B2C знает, как взаимодействовать с приложением учетной записи Майкрософт в Azure AD.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="MicrosoftAccountExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="MicrosoftAccountExchange" TechnicalProfileReferenceId="MSA-MicrosoftAccount-OpenIdConnect" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Тестирование настраиваемой политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице регистрации или входа выберите **Майкрософт** для входа с помощью учетная запись Майкрософт.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end
