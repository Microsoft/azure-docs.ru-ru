---
title: Передача маркера доступа поставщика удостоверений в приложение
titleSuffix: Azure AD B2C
description: Узнайте, как передать маркер доступа для поставщиков удостоверений OAuth 2,0 в качестве утверждения в потоке пользователя в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/15/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: a99d41f5f9fc9538aaf563bd3ae56075d269c94a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97584652"
---
# <a name="pass-an-identity-provider-access-token-to-your-application-in-azure-active-directory-b2c"></a>Передайте маркер доступа поставщика удостоверений в приложение в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

[Пользовательский поток](user-flow-overview.md) в Azure Active Directory B2C (Azure AD B2C) предоставляет пользователям приложения возможность зарегистрироваться или войти в систему с помощью поставщика удостоверений. При этом Azure AD B2C получает [маркер доступа](tokens-overview.md) от поставщика удостоверений. Azure AD B2C использует этот маркер для извлечения сведений о пользователе. Включите утверждение в свой поток пользователя, чтобы передать маркер через приложения, которые вы регистрируете в Azure AD B2C.

::: zone pivot="b2c-user-flow"

Azure AD B2C поддерживает передачу маркера доступа для поставщиков удостоверений [OAuth 2,0](add-identity-provider.md) , включая [Facebook](identity-provider-facebook.md) и [Google](identity-provider-google.md). Для остальных поставщиков удостоверений утверждение возвращается пустым.

::: zone-end

::: zone pivot="b2c-custom-policy"

Azure AD B2C поддерживает передачу маркеров доступа от поставщиков удостоверений по протоколу [OAuth 2.0](authorization-code-flow.md) и стандарту [Подключение OpenID](openid-connect.md). Для остальных поставщиков удостоверений утверждение возвращается пустым.

::: zone-end

На следующей схеме показано, как токен поставщика удостоверений возвращается в приложение. 

![Поставщик удостоверений проходит через поток](./media/idp-pass-through-user-flow/identity-provider-pass-through-flow.png)

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

::: zone pivot="b2c-user-flow"

## <a name="enable-the-claim"></a>Включение утверждения

1. Войдите на [портал Azure](https://portal.azure.com/) с правами глобального администратора клиента Azure AD B2C.
2. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C. Выберите фильтр **Каталог и подписка** в верхнем меню и выберите каталог, который содержит ваш клиент.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, найдите службу **Azure AD B2C** и выберите ее.
4. Выберите **потоки пользователей (политики)**, а затем выберите пользовательский поток. (например, **B2C_1_signupsignin1**).
5. Выберите элемент **Утверждения приложения**.
6. Включите утверждение **маркера доступа поставщика удостоверений** .

    ![Включение утверждения маркера доступа поставщика удостоверений](./media/idp-pass-through-user-flow/identity-provider-pass-through-app-claim.png)

7. Нажмите кнопку **Сохранить**, чтобы сохранить поток пользователя.

## <a name="test-the-user-flow"></a>Тестирование потока пользователя

При тестировании приложений в Azure AD B2C может потребоваться вернуть маркер Azure AD B2C в `https://jwt.ms` для просмотра в нем утверждений.

1. На странице "Обзор" потока пользователя выберите **Выполнить поток пользователя**.
2. В разделе **Приложение** выберите зарегистрированное ранее приложение. Чтобы маркер отображался, как в приведенном ниже примере, **URL-адрес ответа** должен быть следующим: `https://jwt.ms`.
3. Щелкните **Выполнить поток пользователя**, а затем войдите, используя свои учетные данные. Вы должны увидеть маркер доступа поставщика удостоверений в утверждении **idp_access_token**.

    Должен отобразиться результат, аналогичный следующему примеру:

    ![Декодированный маркер в jwt.ms с выделенным блоком idp_access_token](./media/idp-pass-through-user-flow/identity-provider-pass-through-token.png)

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="add-the-claim-elements"></a>Добавление элементов утверждения

1. Откройте файл *TrustframeworkExtensions.xml* и добавьте элемент **ClaimType** с идентификатором `identityProviderAccessToken` в элемент **ClaimsSchema**:

    ```xml
    <BuildingBlocks>
      <ClaimsSchema>
        <ClaimType Id="identityProviderAccessToken">
          <DisplayName>Identity Provider Access Token</DisplayName>
          <DataType>string</DataType>
          <AdminHelpText>Stores the access token of the identity provider.</AdminHelpText>
        </ClaimType>
        ...
      </ClaimsSchema>
    </BuildingBlocks>
    ```

2. Добавьте элемент **OutputClaim** в элемент **TechnicalProfile** для каждого поставщика удостоверений OAuth 2.0, для которого нужно использовать маркер доступа. В следующем примере показано добавление элемента в технический профиль Facebook:

    ```xml
    <ClaimsProvider>
      <DisplayName>Facebook</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Facebook-OAUTH">
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="identityProviderAccessToken" PartnerClaimType="{oauth2:access_token}" />
          </OutputClaims>
          ...
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

3. Сохраните файл *TrustframeworkExtensions.xml*.
4. Откройте файл политики проверяющей стороны, например *SignUpOrSignIn.xml*, и добавьте элемент **OutputClaim** в **TechnicalProfile**:

    ```xml
    <RelyingParty>
      <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
      <TechnicalProfile Id="PolicyProfile">
        <OutputClaims>
          <OutputClaim ClaimTypeReferenceId="identityProviderAccessToken" PartnerClaimType="idp_access_token"/>
        </OutputClaims>
        ...
      </TechnicalProfile>
    </RelyingParty>
    ```

5. Сохраните файл политики.

## <a name="test-your-policy"></a>Тестирование политики

При тестировании приложений в Azure AD B2C может потребоваться вернуть маркер Azure AD B2C в `https://jwt.ms` для просмотра в нем утверждений.

### <a name="upload-the-files"></a>Передача файлов

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Убедитесь, что вы используете каталог, содержащий клиент Azure AD B2C, щелкнув фильтр **каталогов и подписок** в верхнем меню и выбрав Каталог, содержащий ваш клиент.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
4. Выберите **Инфраструктура процедур идентификации**.
5. На странице "Настраиваемые политики" щелкните **Отправить политику**.
6. Выберите **Перезаписать политику, если она существует**, а затем найдите и выберите файл *TrustFrameworkExtensions.xml*.
7. Щелкните **Отправить**.
8. Повторите шаги с 5 по 7 для файла проверяющей стороны, например *SignUpOrSignIn.xml*.

### <a name="run-the-policy"></a>Запуск политики

1. Откройте измененную политику. Например, *B2C_1A_signup_signin*.
2. В разделе **Приложение** выберите зарегистрированное ранее приложение. Чтобы маркер отображался, как в приведенном ниже примере, **URL-адрес ответа** должен быть следующим: `https://jwt.ms`.
3. Выберите **Запустить сейчас**.

    Должен отобразиться результат, аналогичный следующему примеру:

    ![Декодированный маркер в jwt.ms с выделенным блоком idp_access_token](./media/idp-pass-through-user-flow/identity-provider-pass-through-token-custom.png)

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в [обзоре маркеров Azure AD B2C](tokens-overview.md).
