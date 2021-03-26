---
title: Настройка регистрации и входа с помощью идентификатора Apple ID
titleSuffix: Azure AD B2C
description: Обеспечение регистрации и входа для клиентов с идентификаторами Apple ID в приложениях с помощью Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/22/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 63183eb6a77b3a7aecfb6f3e8a7c9ee7c2544de2
ms.sourcegitcommit: 44edde1ae2ff6c157432eee85829e28740c6950d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105543913"
---
# <a name="set-up-sign-up-and-sign-in-with-an-apple-id--using-azure-active-directory-b2c-preview"></a>Настройка регистрации и входа с помощью идентификатора Apple ID с помощью Azure Active Directory B2C (Предварительная версия)

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

## <a name="create-an-apple-id-application"></a>Создание приложения Apple ID

Чтобы включить вход для пользователей с идентификатором Apple ID в Azure Active Directory B2C (Azure AD B2C), необходимо создать приложение в [https://developer.apple.com](https://developer.apple.com) . Дополнительные сведения см. [в разделе Вход с помощью Apple](https://developer.apple.com/sign-in-with-apple/get-started/). Если у вас еще нет учетной записи разработчика Apple, вы можете зарегистрироваться в [программе для разработчиков Apple](https://developer.apple.com/programs/enroll/).

1. Войдите на [портал разработчика Apple](https://developer.apple.com/account) с помощью учетных данных вашей учетной записи.
1. В меню выберите **сертификаты, идентификаторы & профили**, а затем выберите **(+)**.
1. Для **регистрации нового идентификатора** выберите **Идентификаторы приложений**, а затем нажмите кнопку **продолжить**.
1. Для **выбора типа** выберите **приложение** и нажмите кнопку **продолжить**.
1. Для **регистрации идентификатора приложения**:
    1. Введите **Описание** 
    1. Введите **идентификатор пакета**, например `com.contoso.azure-ad-b2c` . 
    1. Для **возможностей** выберите **Вход с помощью Apple** из списка возможностей. 
    1. Запишите префикс идентификатора приложения (идентификатор команды) на этом шаге. Он понадобится вам позднее.
    1. Нажмите кнопку **Continue** (Продолжить), а затем кнопку **Register** (Зарегистрировать).
1. В меню выберите **сертификаты, идентификаторы & профили**, а затем выберите **(+)**.
1. Для **регистрации нового идентификатора** выберите **службы идентификаторы** и нажмите кнопку **продолжить**.
1. Для **регистрации идентификатора служб** выполните следующие действия.
    1. Введите **Описание**. Описание отображается для пользователя на экране согласия.
    1. Введите **идентификатор**, например `com.consoto.azure-ad-b2c-service` . Идентификатор — это идентификатор клиента для потока подключения OpenID Connect.
    1. Нажмите кнопку **продолжить** и выберите **зарегистрировать**.
1. В поле **идентификаторы** выберите созданный идентификатор.
1. Выберите **войти с помощью Apple**, а затем щелкните **настроить**.
    1. Выберите **идентификатор основного приложения** , для которого вы хотите настроить вход с помощью Apple.
    1. В **области домены и поддомены** введите `your-tenant-name.b2clogin.com` . Замените your-tenant-name именем вашего клиента. Если используется [личный домен](custom-domain.md), введите `https://your-domain-name` .
    1. В окне **возвращаемые URL-адреса** введите `https://your-tenant-name.b2clogin.com/your-tenant-name.onmicrosoft.com/oauth2/authresp` . Если используется [личный домен](custom-domain.md), введите `https://your-domain-name/your-tenant-name.onmicrosoft.com/oauth2/authresp` . Замените `your-tenant-name` именем своего клиента и `your-domain-name` личным доменом.
    1. Нажмите кнопку **Далее** и выберите **Готово**.
    1. Когда всплывающее окно закроется, выберите **продолжить** и нажмите кнопку **сохранить**.

## <a name="creating-an-apple-client-secret"></a>Создание секрета клиента Apple

1. В меню портала разработчика Apple выберите **ключи**, а затем нажмите кнопку **(+)**.
1. Для **регистрации нового ключа**:
    1. Введите **имя ключа**.
    1. Выберите **войти с помощью Apple**, а затем щелкните **настроить**.
    1. В поле **идентификатор основного приложения** выберите ранее созданное приложение и нажмите кнопку **сохранить**.
    1. Выберите **Настройка** и нажмите кнопку **зарегистрировать** , чтобы завершить процесс регистрации ключа.
1. Чтобы **скачать свой ключ**, нажмите кнопку **скачать** , чтобы скачать файл. P8, содержащий ваш ключ.


::: zone pivot="b2c-user-flow"

## <a name="configure-apple-as-an-identity-provider"></a>Настройка Apple в качестве поставщика удостоверений

1. Войдите в [портал Azure](https://portal.azure.com/) в качестве глобального администратора клиента Azure AD B2C.
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем каталог, содержащий клиент Azure AD B2C.
1. В разделе **Службы Azure** выберите **Azure AD B2C**. Или используйте поле поиска, чтобы найти и выбрать **Azure AD B2C**.
1. Выберите **поставщики удостоверений**, а затем выберите **Apple (Предварительная версия)**.
1. Введите **Имя**. Например, *Apple*.
1. Введите **Идентификатор разработчика Apple (идентификатор команды)**.
1. Введите **идентификатор службы Apple (идентификатор клиента)**.
1. Введите **идентификатор ключа Apple**.
1. Выберите и отправьте **данные сертификата Apple**.
1. Нажмите кнопку **Сохранить**.


> [!IMPORTANT] 
> - Для входа с помощью Apple необходимо, чтобы администратор обновлял свой секрет клиента каждые 6 месяцев. 
> - В рамках общедоступной предварительной версии этой функции необходимо вручную продлить секрет клиента Apple, если он истечет. Предупреждение будет отображаться заранее на странице поставщиков удостоверений Apple Identity Provider configure IDP, но мы рекомендуем задать собственное напоминание. 
> - Если необходимо продлить секрет, откройте Azure AD B2C в портал Azure, перейдите в раздел **поставщики удостоверений**  >  **Apple** и выберите **Обновить секрет**.

## <a name="add-the-apple-identity-provider-to-a-user-flow"></a>Добавление поставщика удостоверений Apple в поток пользователя

Чтобы разрешить пользователям выполнять вход с помощью идентификатора Apple ID, необходимо добавить поставщик удостоверений Apple в поток пользователя. Вход с помощью Apple можно настроить только для **рекомендованной** версии потоков пользователей. Чтобы добавить поставщик удостоверений Apple в поток пользователя, выполните следующие действия.

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Выберите поток пользователя, для которого требуется добавить поставщик удостоверений Apple. 
1. В разделе **поставщики удостоверений социальных сетей** выберите **Apple (Предварительная версия)**.
1. Нажмите кнопку **Сохранить**.
1. Чтобы проверить политику, выберите пункт **выполнить пользовательскую последовательность**.
1. Для **приложения** выберите веб-приложение с именем *testapp1* , которое вы зарегистрировали ранее. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **запустить поток пользователя** .
1. На странице регистрации или входа выберите **Apple** для входа с помощью Apple ID.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="signing-the-client-secret"></a>Подписывание секрета клиента

Используйте файл. P8, скачанный ранее, чтобы подписать секрет клиента в маркер JWT. Существует множество библиотек, которые могут создавать и подписывать JWT. Используйте [функцию Azure, которая создает маркер](https://github.com/azure-ad-b2c/samples/tree/master/policies/sign-in-with-apple/azure-function) . 

1. Создайте [функцию Azure](../azure-functions/functions-create-function-app-portal.md).
1. В разделе **разработчик** выберите **код + тест**. 
1. Скопируйте содержимое файла [Run. CSX](https://github.com/azure-ad-b2c/samples/blob/master/policies/sign-in-with-apple/azure-function/run.csx) и вставьте его в редактор.
1. Нажмите кнопку **Сохранить**.
1. Выполните HTTP- `POST` запрос и укажите следующие сведения:

    - **апплетеамид**: ваш идентификатор группы разработчиков Apple
    - **апплесервицеид**: идентификатор службы Apple (также идентификатор клиента).
    - **p8key**: ключ формата PEM. Это можно получить, открыв файл. P8 в текстовом редакторе и скопировав все между `-----BEGIN PRIVATE KEY-----` и `-----END PRIVATE KEY-----` без разрывов строк.
 
Следующий код JSON является примером вызова функции Azure:

```json
{
    "appleTeamId": "ABC123DEFG",
    "appleKeyId": "URKEYID001",
    "appleServiceId": "com.yourcompany.app1",
    "p8key": "MIGTAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBHkwdwIBAQQg+s07NiAcuGEu8rxsJBG7ttupF6FRe3bXdHxEipuyK82gCgYIKoZIzj0DAQehRANCAAQnR1W/KbbaihTQayXH3tuAXA8Aei7u7Ij5OdRy6clOgBeRBPy1miObKYVx3ki1msjjG2uGqRbrc1LvjLHINWRD"
}
```

Функция Azure отвечает с помощью правильно отформатированного и подписанного секрета клиента JWT в ответе, например:

```json
{
    "token": "eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJjb20ueW91cmNvbXBhbnkuYXBwMSIsIm5iZiI6MTU2MDI2OTY3NSwiZXhwIjoxNTYwMzU2MDc1LCJpc3MiOiJBQkMxMjNERUZHIiwiYXVkIjoiaHR0cHM6Ly9hcHBsZWlkLmFwcGxlLmNvbSJ9.Dt9qA9NmJ_mk6tOqbsuTmfBrQLFqc9BnSVKR6A-bf9TcTft2XmhWaVODr7Q9w1PP3QOYShFXAnNql5OdNebB4g"
}
```

## <a name="create-a-policy-key"></a>Создание ключа политики

Вам необходимо сохранить секрет клиента, который ранее был записан в клиенте Azure AD B2C.

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем каталог, содержащий клиент Azure AD B2C.
2. В разделе **Службы Azure** выберите **Azure AD B2C**. Или используйте поле поиска, чтобы найти и выбрать **Azure AD B2C**.
1. На странице **Обзор** выберите инфраструктура процедур **идентификации**.
1. Выберите **Ключи политики**, а затем щелкните **Добавить**.
1. В качестве **параметров** выберите **ручной**.
1. Введите **имя** ключа политики. Например, "Апплесекрет". Префикс "B2C_1A_" автоматически добавляется к имени ключа.
1. В поле **секрет** введите значение токена, возвращенного функцией Azure (маркер JWT).
1. Для параметра **Использование ключа** выберите **Подпись**.
1. Нажмите кнопку **создания**.

> [!IMPORTANT] 
> - Для входа с помощью Apple необходимо, чтобы администратор обновлял свой секрет клиента каждые 6 месяцев.
> - Необходимо вручную продлить секрет клиента Apple, если он истечет, и сохранить новое значение в ключе политики.
> - Мы рекомендуем задать собственное напоминание в течение 6 месяцев, чтобы создать новый секрет клиента. 

## <a name="configure-apple-as-an-identity-provider"></a>Настройка Apple в качестве поставщика удостоверений

Чтобы пользователи могли входить с помощью идентификатора Apple ID, необходимо определить учетную запись в качестве поставщика утверждений, который Azure AD B2C может взаимодействовать с помощью конечной точки. Конечная точка предоставляет набор утверждений, которые используются Azure AD B2C для проверки подлинности конкретного пользователя.

Вы можете определить идентификатор Apple ID в качестве поставщика утверждений, добавив его в элемент **клаимспровидерс** в файле расширения политики.

1. Откройте файл *TrustFrameworkExtensions.xml*.
2. Найдите элемент **ClaimsProviders**. Если он не существует, добавьте его в корневой элемент.
3. Добавьте новый элемент **ClaimsProvider** следующим образом.

    ```xml
    <ClaimsProvider>
      <Domain>apple.com</Domain>
      <DisplayName>Apple</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Apple-OIDC">
          <DisplayName>Apple</DisplayName>
          <Protocol Name="OpenIdConnect" />
          <Metadata>
            <Item Key="ProviderName">apple</Item>
            <Item Key="authorization_endpoint">https://appleid.apple.com/auth/authorize</Item>
            <Item Key="AccessTokenEndpoint">https://appleid.apple.com/auth/token</Item>
            <Item Key="JWKS">https://appleid.apple.com/auth/keys</Item>
            <Item Key="issuer">https://appleid.apple.com</Item>
            <Item Key="scope">name email openid</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="response_types">code</Item>
            <Item Key="external_user_identity_claim_id">sub</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="ReadBodyClaimsOnIdpRedirect">user.name.firstName user.name.lastName user.email</Item>
            <Item Key="client_id">You Apple ID</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_AppleSecret"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="sub" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="https://appleid.apple.com" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="user.name.firstName"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="user.name.lastName"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="user.email"/>
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin" />
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

4. Задайте **client_id** идентификатору службы. Например, `com.consoto.azure-ad-b2c-service`.
5. Сохраните файл.

[!INCLUDE [active-directory-b2c-add-identity-provider-to-user-journey](../../includes/active-directory-b2c-add-identity-provider-to-user-journey.md)]


```xml
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
  <ClaimsProviderSelections>
    ...
    <ClaimsProviderSelection TargetClaimsExchangeId="AppleExchange" />
  </ClaimsProviderSelections>
  ...
</OrchestrationStep>

<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="AppleExchange" TechnicalProfileReferenceId="Apple-OIDC" />
  </ClaimsExchanges>
</OrchestrationStep>
```

[!INCLUDE [active-directory-b2c-configure-relying-party-policy](../../includes/active-directory-b2c-configure-relying-party-policy-user-journey.md)]

## <a name="test-your-custom-policy"></a>Тестирование настраиваемой политики

1. Выберите политику проверяющей стороны, например `B2C_1A_signup_signin` .
1. Для **приложения** выберите [ранее зарегистрированное](troubleshoot-custom-policies.md#troubleshoot-the-runtime)веб-приложение. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Нажмите кнопку **Запустить сейчас** .
1. На странице регистрации или входа выберите **Apple** для входа с помощью Apple ID.

Если процесс входа прошел успешно, браузер перенаправляется на `https://jwt.ms` , который отображает содержимое маркера, возвращенного Azure AD B2C.

::: zone-end
