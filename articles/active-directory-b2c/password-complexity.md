---
title: Настройка требований к сложности пароля
titleSuffix: Azure AD B2C
description: Узнайте, как настроить требования к сложности паролей, которые клиенты указывают в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/12/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 81c6e58e34f30d5736c40c77a308321dee28ae34
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103224271"
---
# <a name="configure-complexity-requirements-for-passwords-in-azure-active-directory-b2c"></a>Настройка требований к сложности паролей в Azure Active Directory B2C.

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Azure Active Directory B2C (Azure AD B2C) поддерживает изменение требований сложности к паролям, которые пользователи указывают при создании учетной записи. По умолчанию Azure AD B2C использует **надежные** пароли. Azure AD B2C также поддерживает параметры конфигурации для управления сложностью паролей, используемых клиентами.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

::: zone pivot="b2c-user-flow"

## <a name="password-rule-enforcement"></a>Применение правила к паролю

Во время регистрации или сброса паролей пользователю необходимо ввести пароль, отвечающий требованиям сложности. Правила сложности пароля применяются согласно потоку пользователя. Во время регистрации для одного потока пользователей может потребоваться ПИН-код из четырех цифр, а для другого — из восьми. Например, вы можете использовать поток пользователя с разной сложностью пароля для взрослых и детей.

Требования к сложности пароля не применяются во время входа. Во время входа пользователям никогда не предлагается сменить пароль, так как это не соответствует текущим требованиям сложности.

Сложность пароля можно настроить в следующих типах потоков пользователей.

- поток пользователя регистрации или входа в систему;
- поток пользователя сброса пароля;

При использовании пользовательских политик вы можете настраивать сложность пароля в них (подробнее см. [здесь](password-complexity.md)).

## <a name="configure-password-complexity"></a>Настройка сложности пароля

1. Войдите на [портал Azure](https://portal.azure.com).
2. Выберите значок **Каталог и подписка** в верхней панели инструментов портала, а затем выберите каталог, содержащий клиент Azure AD B2C.
3. На портале Azure найдите и выберите **Azure AD B2C**.
4. Выберите **Потоки пользователей**.
2. Выберите поток пользователя и щелкните **Свойства**.
3. В разделе **Сложность пароля** выберите значение **Простой**, **Надежный** или **Настраиваемый**.

### <a name="comparison-chart"></a>Сравнительная таблица

| Сложность | Описание |
| --- | --- |
| Простота | Пароль, который содержит от 8 до 64 знаков. |
| Уровень согласованности "Строгая" | Пароль, который содержит от 8 до 64 знаков. Он требует наличия 3 из 4 знаков нижнего и верхнего регистра, цифр или символов. |
| Особые настройки | Этот параметр обеспечивает полный контроль над правилами сложности пароля.  Он дает возможность настроить пользовательскую длину,  а также принимать только числовые пароли (ПИН-коды). |

## <a name="custom-options"></a>Настраиваемые параметры

### <a name="character-set"></a>Кодировка

Дает возможность принимать только цифры (ПИН-коды) или полную кодировку.

- При использовании типа **Только цифры** во время ввода пароля можно применять только цифры (0–9).
- При использовании типа **Все** можно применять любые буквы, цифры или символы.

### <a name="length"></a>Длина

Дает возможность управлять требованиями к длине пароля.

- **Минимальная длина** предусматривает не менее 4 знаков.
- **Максимальная длина** предусматривает большее количество знаков, чем используется в минимальной длине (или равное ей), но не более 256 знаков.

### <a name="character-classes"></a>Категории знаков

Позволяет управлять типами других знаков, используемых в пароле.

- Тип **2 из 4: знаки нижнего и верхнего регистра, цифры (0–9), символы** гарантирует, что в пароле используются по крайней мере два типа знаков, например цифра и знак нижнего регистра.
- Тип **3 из 4: знаки нижнего и верхнего регистра, цифры (0–9), символы** гарантирует, что в пароле используются по крайней мере три типа знаков, например цифра, знак нижнего и верхнего регистра.
- Тип **4 из 4: знаки нижнего и верхнего регистра, цифры (0–9), символы** гарантирует, что в пароле используются все типы знаков.

    > [!NOTE]
    > Использование типа **4 из 4** может вызвать трудности у пользователей. Некоторые исследования показали, что это требование не повысит энтропию пароля. См. раздел [Appendix A — Strength of Memorized Secrets](https://pages.nist.gov/800-63-3/sp800-63b.html#appA)(Приложение А — надежность запоминаемых секретов).

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="password-predicate-validation"></a>Проверка предиката пароля

Чтобы настроить сложность пароля, переопределите [типы утверждений](claimsschema.md) `newPassword` и `reenterPassword` с помощью ссылки на [проверки предикатов](predicates.md#predicatevalidations). Элемент PredicateValidations группирует набор предикатов для формирования пользовательской проверки ввода, которую можно применить к типу утверждения. Откройте файл расширения для политики, например <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.

1. Найдите элемент [BuildingBlocks](buildingblocks.md). Если такой элемент не существует, добавьте его.
1. Найдите элемент [ClaimsSchema](claimsschema.md). Если такой элемент не существует, добавьте его.
1. Добавьте утверждения `newPassword` и `reenterPassword` в элемент **ClaimsSchema**.

    ```xml
    <!-- 
    <BuildingBlocks>
      <ClaimsSchema> -->
        <ClaimType Id="newPassword">
          <PredicateValidationReference Id="CustomPassword" />
        </ClaimType>
        <ClaimType Id="reenterPassword">
          <PredicateValidationReference Id="CustomPassword" />
        </ClaimType>
      <!-- 
      </ClaimsSchema>
    </BuildingBlocks>-->
    ```

1. Элемент [Predicate](predicates.md) определяет основную проверку (значения типа утверждения) и возвращает true или false. Проверка выполняется с помощью указанного элемента метода и набора соответствующих методу параметров. Добавьте следующие предикаты в элемент **BuildingBlocks** сразу после завершения элемента `</ClaimsSchema>`:

    ```xml
    <!-- 
    <BuildingBlocks>-->
      <Predicates>
        <Predicate Id="LengthRange" Method="IsLengthRange">
          <UserHelpText>The password must be between 6 and 64 characters.</UserHelpText>
          <Parameters>
            <Parameter Id="Minimum">6</Parameter>
            <Parameter Id="Maximum">64</Parameter>
          </Parameters>
        </Predicate>
        <Predicate Id="Lowercase" Method="IncludesCharacters">
          <UserHelpText>a lowercase letter</UserHelpText>
          <Parameters>
            <Parameter Id="CharacterSet">a-z</Parameter>
          </Parameters>
        </Predicate>
        <Predicate Id="Uppercase" Method="IncludesCharacters">
          <UserHelpText>an uppercase letter</UserHelpText>
          <Parameters>
            <Parameter Id="CharacterSet">A-Z</Parameter>
          </Parameters>
        </Predicate>
        <Predicate Id="Number" Method="IncludesCharacters">
          <UserHelpText>a digit</UserHelpText>
          <Parameters>
            <Parameter Id="CharacterSet">0-9</Parameter>
          </Parameters>
        </Predicate>
        <Predicate Id="Symbol" Method="IncludesCharacters">
          <UserHelpText>a symbol</UserHelpText>
          <Parameters>
            <Parameter Id="CharacterSet">@#$%^&amp;*\-_+=[]{}|\\:',.?/`~"();!</Parameter>
          </Parameters>
        </Predicate>
      </Predicates>
    <!-- 
    </BuildingBlocks>-->
    ```

1. Добавьте следующие проверки предиката в элемент **BuildingBlocks** сразу после завершения элемента `</Predicates>`:

    ```xml
    <!-- 
    <BuildingBlocks>-->
      <PredicateValidations>
        <PredicateValidation Id="CustomPassword">
          <PredicateGroups>
            <PredicateGroup Id="LengthGroup">
              <PredicateReferences MatchAtLeast="1">
                <PredicateReference Id="LengthRange" />
              </PredicateReferences>
            </PredicateGroup>
            <PredicateGroup Id="CharacterClasses">
              <UserHelpText>The password must have at least 3 of the following:</UserHelpText>
              <PredicateReferences MatchAtLeast="3">
                <PredicateReference Id="Lowercase" />
                <PredicateReference Id="Uppercase" />
                <PredicateReference Id="Number" />
                <PredicateReference Id="Symbol" />
              </PredicateReferences>
            </PredicateGroup>
          </PredicateGroups>
        </PredicateValidation>
      </PredicateValidations>
    <!-- 
    </BuildingBlocks>-->
    ```

## <a name="disable-strong-password"></a>Отключение надежного пароля 

Следующие технические профили являются [техническими профилями Active Directory](active-directory-technical-profile.md), которые считывают и записывают данные в Azure Active Directory. Переопределите эти технические профили в файле расширения. Используйте `PersistedClaims`, чтобы отключить политику надежных паролей. Найдите элемент **ClaimsProviders**.  Добавьте следующие поставщики утверждений, как показано ниже.

```xml
<!-- 
<ClaimsProviders>-->
  <ClaimsProvider>
    <DisplayName>Azure Active Directory</DisplayName>
    <TechnicalProfiles>
      <TechnicalProfile Id="AAD-UserWriteUsingLogonEmail">
        <PersistedClaims>
          <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration, DisableStrongPassword"/>
        </PersistedClaims>
      </TechnicalProfile>
      <TechnicalProfile Id="AAD-UserWritePasswordUsingObjectId">
        <PersistedClaims>
          <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration, DisableStrongPassword"/>
        </PersistedClaims>
      </TechnicalProfile>
    </TechnicalProfiles>
  </ClaimsProvider>
<!-- 
</ClaimsProviders>-->
```

Если вы используете политику [входа на основе имени пользователя](https://github.com/azure-ad-b2c/samples/tree/master/policies/username-signup-or-signin), обновите технические профили `AAD-UserWriteUsingLogonEmail`, `AAD-UserWritePasswordUsingObjectId` и `LocalAccountWritePasswordUsingObjectId` с помощью политики *DisableStrongPassword*.

Сохраните файл политики.

## <a name="test-your-policy"></a>Тестирование политики

### <a name="upload-the-files"></a>Передача файлов

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Убедитесь, что используете каталог с клиентом Azure AD B2C, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Azure AD B2C**.
4. Выберите **Инфраструктура процедур идентификации**.
5. На странице "Настраиваемые политики" щелкните **Отправить политику**.
6. Выберите **Перезаписать политику, если она существует**, а затем найдите и выберите файл *TrustFrameworkExtensions.xml*.
7. Щелкните **Отправить**.

### <a name="run-the-policy"></a>Запуск политики

1. Откройте политику регистрации или входа. Например, *B2C_1A_signup_signin*.
2. В разделе **Приложение** выберите зарегистрированное ранее приложение. Чтобы маркер отображался, **URL-адрес ответа** должен быть следующим: `https://jwt.ms`.
3. Щелкните **Запустить сейчас**.
4. Выберите **Зарегистрироваться сейчас**, введите адрес электронной почты и новый пароль. Отображаются рекомендации по ограничению для пароля. Введите данные пользователя и нажмите **Создать**. Вы увидите содержимое возвращенного маркера.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше в разделе [Настройка изменения пароля в Azure Active Directory B2C](add-password-change-policy.md).
- Дополнительные сведения об элементах [Predicates](predicates.md) и [PredicateValidations](predicates.md#predicatevalidations) см. в справочнике по IEF.


::: zone-end
