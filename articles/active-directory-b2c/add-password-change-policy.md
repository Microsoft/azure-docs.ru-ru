---
title: Настройка смены пароля с помощью настраиваемых политик
titleSuffix: Azure AD B2C
description: Сведения о том, как разрешить пользователям изменять пароль с помощью настраиваемых политик в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/22/2021
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 21da8f79772d9648836bedec89cb5d7014486dc6
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104798365"
---
# <a name="configure-password-change-using-custom-policies-in-azure-active-directory-b2c"></a>Настройка смены пароля в настраиваемых политиках в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

В Azure Active Directory B2C (Azure AD B2C) можно разрешить пользователям, вошедшим в локальную учетную запись, изменить пароль, не подтверждая их подлинность при проверке по электронной почте. Поток смены пароля включает следующие шаги.

1. Пользователь входит в локальную учетную запись. Если сеанс все еще активен, Azure AD B2C выполняет авторизацию пользователя и переходит к следующему шагу.
1. Пользователь проверяет **старый пароль**, а затем создает и подтверждает **новый пароль**.

![Поток смены пароля](./media/add-password-change-policy/password-change-flow.png)  

> [!TIP]
> Поток смены пароля позволяет пользователям изменять свой пароль только в том случае, если пользователь знает пароль и хочет изменить его. Рекомендуется также включить [самостоятельный сброс пароля](add-password-reset-policy.md) для поддержки случаев, когда пользователь забыл свой пароль.

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="prerequisites"></a>Предварительные требования

* Выполните шаги, описанные в статье [Начало работы с настраиваемыми политиками в Azure Active Directory B2C](custom-policy-get-started.md).
* Если вы еще не сделали этого, [зарегистрируйте веб-приложение в Azure Active Directory B2C](tutorial-register-applications.md).

## <a name="add-the-elements"></a>Добавление элементов

1. Откройте файл *TrustframeworkExtensions.xml* и добавьте элемент **ClaimType** с идентификатором `oldPassword` в элемент [ClaimsSchema](claimsschema.md):

    ```xml
    <BuildingBlocks>
      <ClaimsSchema>
        <ClaimType Id="oldPassword">
          <DisplayName>Old Password</DisplayName>
          <DataType>string</DataType>
          <UserHelpText>Enter your old password</UserHelpText>
          <UserInputType>Password</UserInputType>
        </ClaimType>
      </ClaimsSchema>
    </BuildingBlocks>
    ```

2. Элемент [ClaimsProvider](claimsproviders.md) содержит технический профиль, с помощью которого выполняется проверка подлинности пользователя. Добавьте следующие поставщики утверждений в элемент **ClaimsProviders**.

    ```xml
    <ClaimsProviders>
      <ClaimsProvider>
        <DisplayName>Local Account SignIn</DisplayName>
        <TechnicalProfiles>
          <TechnicalProfile Id="login-NonInteractive-PasswordChange">
            <DisplayName>Local Account SignIn</DisplayName>
            <InputClaims>
              <InputClaim ClaimTypeReferenceId="oldPassword" PartnerClaimType="password" Required="true" />
              </InputClaims>
            <IncludeTechnicalProfile ReferenceId="login-NonInteractive" />
          </TechnicalProfile>
        </TechnicalProfiles>
      </ClaimsProvider>
      <ClaimsProvider>
        <DisplayName>Local Account Password Change</DisplayName>
        <TechnicalProfiles>
          <TechnicalProfile Id="LocalAccountWritePasswordChangeUsingObjectId">
            <DisplayName>Change password (username)</DisplayName>
            <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
            <Metadata>
              <Item Key="ContentDefinitionReferenceId">api.selfasserted</Item>
            </Metadata>
            <InputClaims>
              <InputClaim ClaimTypeReferenceId="objectId" />
            </InputClaims>
            <OutputClaims>
              <OutputClaim ClaimTypeReferenceId="oldPassword" Required="true" />
              <OutputClaim ClaimTypeReferenceId="newPassword" Required="true" />
              <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
            </OutputClaims>
            <ValidationTechnicalProfiles>
              <ValidationTechnicalProfile ReferenceId="login-NonInteractive-PasswordChange" />
              <ValidationTechnicalProfile ReferenceId="AAD-UserWritePasswordUsingObjectId" />
            </ValidationTechnicalProfiles>
          </TechnicalProfile>
        </TechnicalProfiles>
      </ClaimsProvider>
    </ClaimsProviders>
    ```

3. Элемент [UserJourney](userjourneys.md) определяет путь взаимодействия пользователя с приложением. Добавьте элемент **UserJourneys**, если он не существует. Присвойте **UserJourney** идентификатор `PasswordChange`.

    ```xml
    <UserJourneys>
      <UserJourney Id="PasswordChange">
        <OrchestrationSteps>
          <OrchestrationStep Order="1" Type="ClaimsProviderSelection" ContentDefinitionReferenceId="api.signuporsignin">
            <ClaimsProviderSelections>
              <ClaimsProviderSelection TargetClaimsExchangeId="LocalAccountSigninEmailExchange" />
            </ClaimsProviderSelections>
          </OrchestrationStep>
          <OrchestrationStep Order="2" Type="ClaimsExchange">
            <ClaimsExchanges>
              <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
            </ClaimsExchanges>
          </OrchestrationStep>
          <OrchestrationStep Order="3" Type="ClaimsExchange">
            <ClaimsExchanges>
              <ClaimsExchange Id="NewCredentials" TechnicalProfileReferenceId="LocalAccountWritePasswordChangeUsingObjectId" />
            </ClaimsExchanges>
          </OrchestrationStep>
          <OrchestrationStep Order="4" Type="ClaimsExchange">
            <ClaimsExchanges>
              <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
            </ClaimsExchanges>
          </OrchestrationStep>
          <OrchestrationStep Order="5" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
        </OrchestrationSteps>
        <ClientDefinition ReferenceId="DefaultWeb" />
      </UserJourney>
    </UserJourneys>
    ```

4. Сохраните файл политики *TrustFrameworkExtensions.xml*.
5. Скопируйте файл *ProfileEdit.xml*, который был скачан вместе с начальным пакетом, и присвойте ему имя *ProfileEditPasswordChange.xml*.
6. Откройте новый файл и задайте для атрибута **PolicyId** уникальное значение. Это значение определяет имя вашей политики. Например, *B2C_1A_profile_edit_password_change*.
7. Задайте для атрибута **ReferenceId** в `<DefaultUserJourney>` идентификатор созданного пути взаимодействия пользователя. Например, *PasswordChange*.
8. Сохраните изменения.

## <a name="upload-and-test-the-policy"></a>Отправка и тестирование политики

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
4. Выберите **Инфраструктура процедур идентификации**.
5. На странице "Настраиваемые политики" щелкните **Отправить политику**.
6. Выберите **Перезаписать политику, если она существует**, а затем найдите и выберите файл *TrustFrameworkExtensions.xml*.
7. Щелкните **Отправить**.
8. Повторите шаги с 5 по 7 для файла проверяющей стороны, например *ProfileEditPasswordChange.xml*.

### <a name="run-the-policy"></a>Запуск политики

1. Откройте измененную политику. Например, *B2C_1A_profile_edit_password_change*.
2. В разделе **Приложение** выберите зарегистрированное ранее приложение. Чтобы маркер отображался, **URL-адрес ответа** должен быть следующим: `https://jwt.ms`.
3. Щелкните **Запустить сейчас**. Войдите, используя созданную ранее учетную запись. Теперь вы сможете изменять пароль.

## <a name="next-steps"></a>Следующие шаги

- Найдите пример политики на сайте [GitHub](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/password-change).
- Узнайте, как можно [настроить сложность пароля в Azure AD B2C](password-complexity.md).
- Настройка [потока сброса пароля](add-password-reset-policy.md).

::: zone-end
