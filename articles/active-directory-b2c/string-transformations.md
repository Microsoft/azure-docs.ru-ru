---
title: Примеры преобразования утверждений строк для пользовательских политик
titleSuffix: Azure AD B2C
description: Примеры преобразования утверждений строк для схемы Identity Experience Framework (IEF) в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/08/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 85574b7d33af6d9abfe25f5af4d811255f08ce4b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102452243"
---
# <a name="string-claims-transformations"></a>Преобразования утверждений строк

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

В этой статье приведены примеры использования преобразований утверждений строк схемы Identity Experience Framework в Azure Active Directory B2C (Azure AD B2C). Дополнительные сведения см. в статье о [преобразовании утверждений](claimstransformations.md).

## <a name="assertstringclaimsareequal"></a>AssertStringClaimsAreEqual

Сравнение двух утверждений и создание исключения, если они не равны, в соответствии с указанными для сравнения элементами inputClaim1, inputClaim2 и stringComparison.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | строка | Тип первого утверждения, которое необходимо сравнить. |
| InputClaim | inputClaim2 | строка | Тип второго утверждения, которое необходимо сравнить. |
| InputParameter | stringComparison | строка | сравнение строк, одно из значений: Ordinal, OrdinalIgnoreCase. |

Преобразование строк **AssertStringClaimsAreEqual** всегда выполняется из [технического профиля проверки](validation-technical-profile.md), вызываемого с помощью [самоподтвержденного технического профиля](self-asserted-technical-profile.md), или [DisplayControl](display-controls.md). Метаданные `UserMessageIfClaimsTransformationStringsAreNotEqual` самоподтвержденного технического профиля позволяют управлять сообщением об ошибке, представленным пользователю. Сообщения об ошибках можно [локализовать](localization-string-ids.md#claims-transformations-error-messages).


![Выполнение AssertStringClaimsAreEqual](./media/string-transformations/assert-execution.png)

С помощью этого преобразования можно убедиться, что два параметра ClaimTypes имеют одинаковые значения. В противном случае выдается сообщение об ошибке. В следующем примере проверяется, равен ли параметр ClaimType **strongAuthenticationEmailAddress** параметру ClaimType **email**. В противном случае выдается сообщение об ошибке.

```xml
<ClaimsTransformation Id="AssertEmailAndStrongAuthenticationEmailAddressAreEqual" TransformationMethod="AssertStringClaimsAreEqual">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="strongAuthenticationEmailAddress" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
  </InputParameters>
</ClaimsTransformation>
```


Технический профиль проверки **login-NonInteractive** вызывает преобразование утверждений **AssertEmailAndStrongAuthenticationEmailAddressAreEqual**.
```xml
<TechnicalProfile Id="login-NonInteractive">
  ...
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertEmailAndStrongAuthenticationEmailAddressAreEqual" />
  </OutputClaimsTransformations>
</TechnicalProfile>
```

Самоподтвержденный технический профиль вызывает технический профиль проверки **login-NonInteractive**.

```xml
<TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
  <Metadata>
    <Item Key="UserMessageIfClaimsTransformationStringsAreNotEqual">Custom error message the email addresses you provided are not the same.</Item>
  </Metadata>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="login-NonInteractive" />
  </ValidationTechnicalProfiles>
</TechnicalProfile>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **inputClaim1**: someone@contoso.com.
  - **inputClaim2**: someone@outlook.com.
- Входные параметры:
  - **stringComparison**: ordinalIgnoreCase.
- Результат: возникла ошибка.

## <a name="changecase"></a>ChangeCase

Изменяет предоставленное утверждение в нижний или верхний регистр в зависимости от оператора.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | строка | ClaimType, который необходимо изменить. |
| InputParameter | toCase | строка | Одно из следующих значений: `LOWER` или `UPPER`. |
| outputClaim | outputClaim | строка | Параметр ClaimType, который создается после вызова этого преобразования утверждений. |

С помощью этого преобразования утверждений можно изменить любой строковый тип ClaimType в нижний или верхний регистр.

```xml
<ClaimsTransformation Id="ChangeToLower" TransformationMethod="ChangeCase">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="inputClaim1" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="toCase" DataType="string" Value="LOWER" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **email**: SomeOne@contoso.com.
- Входные параметры:
    - **toCase**: LOWER
- Исходящие утверждения:
  - **email**: someone@contoso.com.

## <a name="createstringclaim"></a>CreateStringClaim

Создает строковое утверждение на основе предоставленного входного параметра в преобразовании.

| Item | TransformationClaimType | Тип данных | Примечания |
|----- | ----------------------- | --------- | ----- |
| InputParameter | value | строка | Строка, которую необходимо задать. Этот входной параметр поддерживает [выражения преобразования строковых утверждений](string-transformations.md#string-claim-transformations-expressions). |
| outputClaim | createdClaim | строка | Параметр ClaimType, который создается после вызова этого преобразования утверждений, со значением, указанным во входном параметре. |

Это преобразование утверждения позволяет задать строковое значение параметра ClaimType.

```xml
<ClaimsTransformation Id="CreateTermsOfService" TransformationMethod="CreateStringClaim">
  <InputParameters>
    <InputParameter Id="value" DataType="string" Value="Contoso terms of service..." />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="TOS" TransformationClaimType="createdClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входной параметр:
    - **value**: условия предоставления услуг компании Contoso...
- Исходящие утверждения:
    - **createdClaim**: параметр ClaimType TOS содержит значение "Условия предоставления услуг компании Contoso...".

## <a name="copyclaimifpredicatematch"></a>копиклаимифпредикатематч

Копировать значение утверждения в другое, если значение входного утверждения совпадает с выходным предикатом исходящего утверждения. 

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | строка | Тип утверждения, который будет скопирован. |
| outputClaim | outputClaim | строка | Тип утверждения, созданный после вызова этого преобразования утверждений. Значение входного утверждения проверяется по предикату утверждения. |

В следующем примере значение утверждения signInName копируется в утверждение phoneNumber, только если signInName является номером телефона. Полный пример см. в разделе [номер телефона или политика начального пакета для входа в электронную почту](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/blob/master/scenarios/phone-number-passwordless/Phone_Email_Base.xml) .

```xml
<ClaimsTransformation Id="SetPhoneNumberIfPredicateMatch" TransformationMethod="CopyClaimIfPredicateMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="signInName" TransformationClaimType="inputClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example-1"></a>Пример 1

- Входящие утверждения:
    - **inputClaim**: bob@contoso.com
- Исходящие утверждения:
    - **outputClaim**: выходное утверждение не будет изменено из исходного значения.

### <a name="example-2"></a>Пример 2

- Входящие утверждения:
    - **inputClaim**: + 11234567890
- Исходящие утверждения:
    - **outputClaim**: +11234567890

## <a name="compareclaims"></a>CompareClaims

Определяет, равно ли одно строковое утверждение другому. Результатом является новый логический параметр ClaimType со значением `true` или `false`.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | строка | Тип первого утверждения, которое необходимо сравнить. |
| InputClaim | inputClaim2 | строка | Тип второго утверждения, которое необходимо сравнить. |
| InputParameter | оператор | строка | Возможные значения: `EQUAL` или `NOT EQUAL`. |
| InputParameter | ignoreCase | Логическое | Указывает, следует ли в этом сравнении игнорировать регистр сравниваемых строк. |
| outputClaim | outputClaim | Логическое | Параметр ClaimType, который создается после вызова этого преобразования утверждений. |

Используйте это преобразование, чтобы проверить, равно ли одно утверждение другому. Например, следующее преобразование утверждений проверяет, равно ли значение утверждения **email** утверждению **Verified.Email**.

```xml
<ClaimsTransformation Id="CheckEmail" TransformationMethod="CompareClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="Email" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="Verified.Email" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="operator" DataType="string" Value="NOT EQUAL" />
    <InputParameter Id="ignoreCase" DataType="string" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="SameEmailAddress" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **inputClaim1**: someone@contoso.com.
  - **inputClaim2**: someone@outlook.com.
- Входные параметры:
    - **operator**:  NOT EQUAL.
    - **ignoreCase**: true.
- Исходящие утверждения:
    - **outputClaim**: true.

## <a name="compareclaimtovalue"></a>CompareClaimToValue

Определяет, равно ли значение утверждения значению входного параметра.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | строка | Тип утверждения, которое необходимо сравнить. |
| InputParameter | оператор | строка | Возможные значения: `EQUAL` или `NOT EQUAL`. |
| InputParameter | compareTo | строка | сравнение строк, одно из значений: Ordinal, OrdinalIgnoreCase. |
| InputParameter | ignoreCase | Логическое | Указывает, следует ли в этом сравнении игнорировать регистр сравниваемых строк. |
| outputClaim | outputClaim | Логическое | Параметр ClaimType, который создается после вызова этого преобразования утверждений. |

С помощью этого преобразования можно проверить, равно ли утверждение указанному значению. Например, следующее преобразование утверждений проверяет, равно ли значение утверждения **termsOfUseConsentVersion** значению `v1`.

```xml
<ClaimsTransformation Id="IsTermsOfUseConsentRequiredForVersion" TransformationMethod="CompareClaimToValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="inputClaim1" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="compareTo" DataType="string" Value="V1" />
    <InputParameter Id="operator" DataType="string" Value="not equal" />
    <InputParameter Id="ignoreCase" DataType="string" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentRequired" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример
- Входящие утверждения:
    - **inputClaim1**: v1.
- Входные параметры:
    - **compareTo**: V1
    - **operator**: EQUAL
    - **ignoreCase**: true.
- Исходящие утверждения:
    - **outputClaim**: true.

## <a name="createrandomstring"></a>CreateRandomString

Создает случайную строку с помощью генератора случайных чисел. Если тип генератора случайных чисел — `integer`, при необходимости можно указать параметр начального значения и максимальное число. Необязательный параметр формата строки позволяет форматировать выходные данные, а необязательный параметр Base64 указывает, соответствуют ли выходные данные randomGeneratorType [идентификатор GUID, целое число] outputClaim (строка) в кодировке Base64.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputParameter | randomGeneratorType | строка | Указывает случайное значение, которое необходимо создать: `GUID` (глобальный уникальный идентификатор) или `INTEGER` (число). |
| InputParameter | stringFormat | строка | [Необязательно.] Формат случайного значения. |
| InputParameter | base64 | Логическое | [Необязательно.] Преобразование случайного значения в Base64. Если применяется формат строки, значение после формата строки кодируется в Base64. |
| InputParameter | maximumNumber | INT | [Необязательно.] Только для типа randomGeneratorType `INTEGER`. Укажите максимальное число. |
| InputParameter | seed  | INT | [Необязательно.] Только для типа randomGeneratorType `INTEGER`. Укажите начальное значение для случайного значения. Примечание. Такое же начальное значение обеспечивает такую же последовательность случайных чисел. |
| outputClaim | outputClaim | строка | Параметр ClaimType, который будет создан после вызова этого преобразования утверждений. Случайное значение. |

В следующем примере создается глобальный уникальный идентификатор. С помощью этого преобразования утверждений создается случайное имя участника-пользователя.

```xml
<ClaimsTransformation Id="CreateRandomUPNUserName" TransformationMethod="CreateRandomString">
  <InputParameters>
    <InputParameter Id="randomGeneratorType" DataType="string" Value="GUID" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="upnUserName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Пример

- Входные параметры:
    - **randomGeneratorType**: GUID
- Исходящие утверждения:
    - **outputClaim**: bc8bedd2-aaa3-411e-bdee-2f1810b73dfc.

В следующем примере создается случайное целое число от 0 до 1000. Значение форматируется в формат OTP_{случайное значение}.

```xml
<ClaimsTransformation Id="SetRandomNumber" TransformationMethod="CreateRandomString">
  <InputParameters>
    <InputParameter Id="randomGeneratorType" DataType="string" Value="INTEGER" />
    <InputParameter Id="maximumNumber" DataType="int" Value="1000" />
    <InputParameter Id="stringFormat" DataType="string" Value="OTP_{0}" />
    <InputParameter Id="base64" DataType="boolean" Value="false" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="randomNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входные параметры:
    - **randomGeneratorType**: INTEGER
    - **maximumNumber**: 1000
    - **stringFormat**: OTP_{0}.
    - **Base64**: false.
- Исходящие утверждения:
    - **outputClaim**: OTP_853.


## <a name="formatlocalizedstring"></a>форматлокализедстринг

Форматирование нескольких утверждений в соответствии с предоставленной строкой локализованного формата. Это преобразование использует метод C# `String.Format`.


| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaims |  |строка | Коллекция входных утверждений, которые действуют как строковый формат {0} , {1} , — {2} Parameters. |
| InputParameter | стрингформатид | строка |  Объект `StringId` [локализованной строки](localization.md).   |
| outputClaim | outputClaim | строка | Параметр ClaimType, который создается после вызова этого преобразования утверждений. |

> [!NOTE]
> Максимальный допустимый размер строкового формата — 4000.

Чтобы использовать преобразование Форматлокализедстринг Claims, выполните следующие действия.

1. Определите [строку локализации](localization.md)и свяжите ее с [самостоятельно подтвержденным-техническим профилем](self-asserted-technical-profile.md).
1. Для `ElementType` элемента `LocalizedString` необходимо задать значение `FormatLocalizedStringTransformationClaimType`.
1. `StringId`— Это уникальный идентификатор, который вы определяете и используете позже в преобразовании утверждений `stringFormatId` .
1. В преобразовании утверждений укажите список утверждений, которые должны быть заданы в локализованной строке. Затем присвойте свойству значение `stringFormatId` `StringId` из локализованного строкового элемента. 
1. В [самоподтвержденном техническом профиле](self-asserted-technical-profile.md) или [элементе управления отображением](display-controls.md) преобразования входных или выходных утверждений укажите ссылку на преобразование утверждений.


В следующем примере создается сообщение об ошибке, если учетная запись уже находится в каталоге. В примере определяются локализованные строки для английского языка (по умолчанию) и испанского.

```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
   </SupportedLanguages>

  <LocalizedResources Id="api.localaccountsignup.en">
    <LocalizedStrings>
      <LocalizedString ElementType="FormatLocalizedStringTransformationClaimType" StringId="ResponseMessge_EmailExists">The email '{0}' is already an account in this organization. Click Next to sign in with that account.</LocalizedString>
      </LocalizedStrings>
    </LocalizedResources>
  <LocalizedResources Id="api.localaccountsignup.es">
    <LocalizedStrings>
      <LocalizedString ElementType="FormatLocalizedStringTransformationClaimType" StringId="ResponseMessge_EmailExists">Este correo electrónico "{0}" ya es una cuenta de esta organización. Haga clic en Siguiente para iniciar sesión con esa cuenta.</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
</Localization>
```

Преобразование «утверждения» создает ответное сообщение на основе локализованной строки. Сообщение содержит адрес электронной почты пользователя, внедренный в локализованный *ResponseMessge_EmailExists* проверочного.

```xml
<ClaimsTransformation Id="SetResponseMessageForEmailAlreadyExists" TransformationMethod="FormatLocalizedString">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormatId" DataType="string" Value="ResponseMessge_EmailExists" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="responseMsg" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputClaim**: sarah@contoso.com
- Входные параметры:
    - **StringFormat**: ResponseMessge_EmailExists
- Исходящие утверждения:
  - **outputClaim**: адрес электронной почты " sarah@contoso.com " уже является учетной записью в этой Организации. Нажмите кнопку Далее, чтобы войти в систему с помощью этой учетной записи.


## <a name="formatstringclaim"></a>FormatStringClaim

Форматирование утверждения в указанный формат строки. Это преобразование использует метод C# `String.Format`.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim |строка |Элемент ClaimType, который выступает в качестве параметра {0} формата строки. |
| InputParameter | stringFormat | строка | Формат строки, включая параметр {0}. Этот входной параметр поддерживает [выражения преобразования строковых утверждений](string-transformations.md#string-claim-transformations-expressions).  |
| outputClaim | outputClaim | строка | Параметр ClaimType, который создается после вызова этого преобразования утверждений. |

> [!NOTE]
> Максимальный допустимый размер строкового формата — 4000.

Используйте это преобразование утверждений для форматирования любой строки с одним параметром {0}. В следующем примере создается **userPrincipalName**. Технические профили всех поставщиков удостоверений в социальных сетях, такие как `Facebook-OAUTH`, вызывают **CreateUserPrincipalName** для создания **userPrincipalName**.

```xml
<ClaimsTransformation Id="CreateUserPrincipalName" TransformationMethod="FormatStringClaim">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="upnUserName" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormat" DataType="string" Value="cpim_{0}@{RelyingPartyTenantId}" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputClaim**: 5164db16-3eee-4629-bfda-dcc3326790e9.
- Входные параметры:
    - **stringFormat**: cpim_{0}@{ИД клиента проверяющей стороны}.
- Исходящие утверждения:
  - **outputClaim**: cpim_5164db16-3eee-4629-bfda-dcc3326790e9@b2cdemo.onmicrosoft.com.

## <a name="formatstringmultipleclaims"></a>FormatStringMultipleClaims

Форматирование двух утверждений в указанный формат строки. Это преобразование использует метод C# `String.Format`.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim |строка | Элемент ClaimType, который выступает в качестве параметра {0} формата строки. |
| InputClaim | InputClaim | строка | Элемент ClaimType, который выступает в качестве параметра {1} формата строки. |
| InputParameter | stringFormat | строка | Формат строки, включая параметры {0} и {1}. Этот входной параметр поддерживает [выражения преобразования строковых утверждений](string-transformations.md#string-claim-transformations-expressions).   |
| outputClaim | outputClaim | строка | Параметр ClaimType, который создается после вызова этого преобразования утверждений. |

> [!NOTE]
> Максимальный допустимый размер строкового формата — 4000.

Используйте это преобразование утверждений для форматирования любой строки с двумя параметрами: {0} и {1}. В следующем примере создается **displayName** с указанным форматом:

```xml
<ClaimsTransformation Id="CreateDisplayNameFromFirstNameAndLastName" TransformationMethod="FormatStringMultipleClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="surName" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="stringFormat" DataType="string" Value="{0} {1}" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="displayName" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputClaim1**: Ivan
    - **inputClaim2**: Voronkov.
- Входные параметры:
    - **stringFormat**: {0} {1}
- Исходящие утверждения:
    - **outputClaim**: Ivan Voronkov.

## <a name="getlocalizedstringstransformation"></a>GetLocalizedStringsTransformation

Копирует локализованные строки в утверждения.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| outputClaim | Имя локализованной строки | строка | Список типов утверждений, которые создаются после вызова этого преобразования утверждений. |

Чтобы использовать утверждения GetLocalizedStringsTransformation, выполните следующие действия.

1. Определите [строку локализации](localization.md) и свяжите ее с [самоподтвержденным техническим профилем](self-asserted-technical-profile.md).
1. Для `ElementType` элемента `LocalizedString` необходимо задать значение `GetLocalizedStringsTransformationClaimType`.
1. `StringId` — это уникальный определяемый вами идентификатор, который вы используете позже в преобразовании утверждений.
1. В преобразовании утверждений укажите список утверждений, которые должны быть заданы в локализованной строке. `ClaimTypeReferenceId` — это ссылка на ClaimType, уже определенная в разделе ClaimsSchema файла политики. `TransformationClaimType` — это имя локализованной строки, определенное в `StringId` элемента `LocalizedString`.
1. В [самоподтвержденном техническом профиле](self-asserted-technical-profile.md) или [элементе управления отображением](display-controls.md) преобразования входных или выходных утверждений укажите ссылку на преобразование утверждений.

![GetLocalizedStringsTransformation](./media/string-transformations/get-localized-strings-transformation.png)

В следующем примере выполняется поиск темы сообщения электронной почты, текста, кодового сообщения и подписи сообщения из локализованных строк. Эти утверждения позже используются настраиваемым шаблоном проверки электронной почты.

Определите локализованные строки для английского языка (по умолчанию) и испанского.

```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="Append">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
   </SupportedLanguages>

  <LocalizedResources Id="api.localaccountsignup.en">
    <LocalizedStrings>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for verifying your account!</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
     </LocalizedStrings>
   </LocalizedResources>
   <LocalizedResources Id="api.localaccountsignup.es">
     <LocalizedStrings>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Código de verificación del correo electrónico de la cuenta de Contoso</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Gracias por comprobar la cuenta de </LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Su código es</LocalizedString>
      <LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Atentamente</LocalizedString>
    </LocalizedStrings>
  </LocalizedResources>
</Localization>
```

Преобразование утверждений задает значение типа утверждения *subject* со значением `StringId` *email_subject*.

```xml
<ClaimsTransformation Id="GetLocalizedStringsForEmail" TransformationMethod="GetLocalizedStringsTransformation">
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="subject" TransformationClaimType="email_subject" />
    <OutputClaim ClaimTypeReferenceId="message" TransformationClaimType="email_message" />
    <OutputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="email_code" />
    <OutputClaim ClaimTypeReferenceId="signature" TransformationClaimType="email_signature" />
   </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Исходящие утверждения:
  - **subject.** Код проверки учетной записи электронной почты Contoso
  - **message**: Благодарим за подтверждение учетной записи!
  - **codeIntro**: Ваш код:
  - **signature**: С уважением,


## <a name="getmappedvaluefromlocalizedcollection"></a>GetMappedValueFromLocalizedCollection

Выполняет поиск элемента из коллекции **Restriction** утверждения.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | mapFromClaim | строка | Утверждение, содержащее текст, поиск которого будет выполняться в утверждениях **restrictionValueClaim** с коллекцией **Restriction**.  |
| outputClaim | restrictionValueClaim | строка | Утверждение, содержащее коллекцию **Restriction**. После вызова преобразования утверждений значение этого утверждения будет содержать значение выбранного элемента. |

В следующем примере выполняется поиск описания сообщения об ошибке на основе ключа ошибки. Утверждение **ResponseMsg** содержит коллекцию сообщений об ошибках для представления пользователю или отправки проверяющей стороне.

```xml
<ClaimType Id="responseMsg">
  <DisplayName>Error message: </DisplayName>
  <DataType>string</DataType>
  <UserInputType>Paragraph</UserInputType>
  <Restriction>
    <Enumeration Text="B2C_V1_90001" Value="You cannot sign in because you are a minor" />
    <Enumeration Text="B2C_V1_90002" Value="This action can only be performed by gold members" />
    <Enumeration Text="B2C_V1_90003" Value="You have not been enabled for this operation" />
  </Restriction>
</ClaimType>
```
Преобразование утверждений ищет текст элемента и возвращает его значение. Если ограничение локализовано с помощью `<LocalizedCollection>`, преобразование утверждений возвращает локализованное значение.

```xml
<ClaimsTransformation Id="GetResponseMsgMappedToResponseCode" TransformationMethod="GetMappedValueFromLocalizedCollection">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="responseCode" TransformationClaimType="mapFromClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="responseMsg" TransformationClaimType="restrictionValueClaim" />        
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **mapFromClaim**: B2C_V1_90001.
- Исходящие утверждения:
    - **restrictionValueClaim**: Вы не можете войти, так как являетесь несовершеннолетним.

## <a name="lookupvalue"></a>LookupValue

Выполняет поиск значения утверждения из списка значений, исходя из значения другого утверждения.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputParameterId | строка | Утверждение, содержащее значение подстановки. |
| InputParameter | |строка | Коллекция inputParameters. |
| InputParameter | errorOnFailedLookup | Логическое | Контролирует, будет ли возвращена ошибка, если значение подстановки не найдено. |
| outputClaim | inputParameterId | строка | Параметр ClaimType, который будет создан после вызова этого преобразования утверждений. Значение соответствующего `Id`. |

В следующем примере выполняется поиск доменного имени в одной из коллекций inputParameters. Преобразование утверждений ищет доменное имя в идентификаторе и возвращает его значение (идентификатор приложения).

```xml
 <ClaimsTransformation Id="DomainToClientId" TransformationMethod="LookupValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="inputParameterId" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="contoso.com" DataType="string" Value="13c15f79-8fb1-4e29-a6c9-be0d36ff19f1" />
    <InputParameter Id="microsoft.com" DataType="string" Value="0213308f-17cb-4398-b97e-01da7bd4804e" />
    <InputParameter Id="test.com" DataType="string" Value="c7026f88-4299-4cdb-965d-3f166464b8a9" />
    <InputParameter Id="errorOnFailedLookup" DataType="boolean" Value="false" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainAppId" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputParameterId**: test.com.
- Входные параметры:
    - **contoso.com**: 13c15f79-8fb1-4e29-a6c9-be0d36ff19f1.
    - **microsoft.com**: 0213308f-17cb-4398-b97e-01da7bd4804e.
    - **test.com**: c7026f88-4299-4cdb-965d-3f166464b8a9.
    - **errorOnFailedLookup**: false.
- Исходящие утверждения:
    - **outputClaim**:    c7026f88-4299-4cdb-965d-3f166464b8a9

Если входной параметр `errorOnFailedLookup` имеет значение `true`, преобразование утверждений **LookupValue** всегда выполняется из [технического профиля проверки](validation-technical-profile.md), вызываемого с помощью [самоподтвержденного технического профиля](self-asserted-technical-profile.md) или [DisplayControl](display-controls.md). Метаданные `LookupNotFound` самоподтвержденного технического профиля позволяют управлять сообщением об ошибке, представленным пользователю.

![Выполнение AssertStringClaimsAreEqual](./media/string-transformations/assert-execution.png)

В следующем примере выполняется поиск доменного имени в одной из коллекций inputParameters. Преобразование утверждений ищет доменное имя в идентификаторе и возвращает его значение (идентификатор приложения) или выдает сообщение об ошибке.

```xml
 <ClaimsTransformation Id="DomainToClientId" TransformationMethod="LookupValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="inputParameterId" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="contoso.com" DataType="string" Value="13c15f79-8fb1-4e29-a6c9-be0d36ff19f1" />
    <InputParameter Id="microsoft.com" DataType="string" Value="0213308f-17cb-4398-b97e-01da7bd4804e" />
    <InputParameter Id="test.com" DataType="string" Value="c7026f88-4299-4cdb-965d-3f166464b8a9" />
    <InputParameter Id="errorOnFailedLookup" DataType="boolean" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainAppId" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputParameterId**: live.com
- Входные параметры:
    - **contoso.com**: 13c15f79-8fb1-4e29-a6c9-be0d36ff19f1.
    - **microsoft.com**: 0213308f-17cb-4398-b97e-01da7bd4804e.
    - **test.com**: c7026f88-4299-4cdb-965d-3f166464b8a9.
    - **errorOnFailedLookup**: true
- Ошибка:
    - Не найдено совпадений для значения входного утверждения в списке идентификаторов входных параметров, а errorOnFailedLookup имеет значение true.


## <a name="nullclaim"></a>NullClaim

Удаление значения данного утверждения.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| outputClaim | claim_to_null | строка | Для утверждения задано значение NULL. |

Это преобразование утверждений используется для удаления ненужных данных из контейнера свойств утверждения, чтобы сократить размер файла cookie сеанса. В следующем примере удаляется значение типа утверждения `TermsOfService`.

```xml
<ClaimsTransformation Id="SetTOSToNull" TransformationMethod="NullClaim">
  <OutputClaims>
  <OutputClaim ClaimTypeReferenceId="TermsOfService" TransformationClaimType="claim_to_null" />
  </OutputClaims>
</ClaimsTransformation>
```

- Входящие утверждения:
    - **outputClaim**: "Добро пожаловать в приложение Contoso". Если вы по-прежнему просматриваете и используете этот веб-сайт, вы соглашаетесь соблюдать следующие положения и условия...
- Исходящие утверждения:
    - **outputClaim**: NULL

## <a name="parsedomain"></a>ParseDomain

Получает доменную часть адреса электронной почты.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | emailAddress | строка | Параметр ClaimType, содержащий адрес электронной почты. |
| outputClaim | домен | строка | Параметр ClaimType, который создается после вызова этого преобразования утверждений, со значением домена. |

Это преобразование утверждений используется для анализа доменного имени после символа @ пользователя. В следующих преобразованиях утверждений показано, как проанализировать доменное имя из утверждения **email**.

```xml
<ClaimsTransformation Id="SetDomainName" TransformationMethod="ParseDomain">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="emailAddress" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="domainName" TransformationClaimType="domain" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **EmailAddress**: joe@outlook.com.
- Исходящие утверждения:
    - **domain**: outlook.com.

## <a name="setclaimsifregexmatch"></a>SetClaimsIfRegexMatch

Проверяет, равны ли строка утверждения `claimToMatch` и входной параметр `matchTo`, и задает для исходящих утверждений значение, указанное во входном параметре `outputClaimIfMatched`, а также задает для исходящего утверждения результата сравнения значение `true` или `false` в зависимости от результата сравнения.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | claimToMatch | строка | Тип утверждения, которое необходимо сравнить. |
| InputParameter | matchTo | строка | Регулярное выражение для сравнения. |
| InputParameter | outputClaimIfMatched | строка | Значение, которое необходимо задать, если строки равны. |
| InputParameter | extractGroups | Логическое | [Необязательно] Указывает, должно ли соответствие регулярного выражения извлекать значения групп. Возможные значения: `true` или `false` (по умолчанию). | 
| outputClaim | outputClaim | строка | Если регулярное выражение совпадает, это исходящее утверждение содержит значение входного параметра `outputClaimIfMatched`. Или NULL, если совпадений нет. |
| outputClaim | regexCompareResultClaim | Логическое | Регулярное выражение соответствует типу исходящего утверждения результата, для которого необходимо задать значение `true` или `false` на основе результата сравнения. |
| outputClaim| Имя утверждения| строка | Если входной параметр extractGroups имеет значение true, список типов утверждений, которые создаются после вызова этого преобразования утверждений. Имя claimType должно соответствовать имени группы регулярного выражения. | 

### <a name="example-1"></a>Пример 1

Проверяет, является ли указанный номер телефона действительным, на основе шаблона регулярного выражения номера телефона.

```xml
<ClaimsTransformation Id="SetIsPhoneRegex" TransformationMethod="SetClaimsIfRegexMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phone" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="^[0-9]{4,16}$" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="isPhone" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="validationResult" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isPhoneBoolean" TransformationClaimType="regexCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

- Входящие утверждения:
    - **claimToMatch**: "64854114520"
- Входные параметры:
    - **matchTo**: "^[0-9]{4,16}$"
    - **outputClaimIfMatched**:  "isPhone"
- Исходящие утверждения:
    - **outputClaim**: "isPhone"
    - **regexCompareResultClaim**: true

### <a name="example-2"></a>Пример 2

Проверяет, является ли указанный адрес электронной почты действительным, и возвращает псевдоним электронной почты.

```xml
<ClaimsTransformation Id="GetAliasFromEmail" TransformationMethod="SetClaimsIfRegexMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="(?&lt;mailAlias&gt;.*)@(.*)$" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="isEmail" />
    <InputParameter Id="extractGroups" DataType="boolean" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="validationResult" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isEmailString" TransformationClaimType="regexCompareResultClaim" />
    <OutputClaim ClaimTypeReferenceId="mailAlias" />
  </OutputClaims>
</ClaimsTransformation>
```

- Входящие утверждения:
    - **claimToMatch**: "emily@contoso.com"
- Входные параметры:
    - **matchTo**: `(?&lt;mailAlias&gt;.*)@(.*)$`
    - **outputClaimIfMatched**:  "isEmail"
    - **extractGroups**: true
- Исходящие утверждения:
    - **outputClaim**: "isEmail"
    - **regexCompareResultClaim**: true
    - **mailAlias**: emily
    
## <a name="setclaimsifstringsareequal"></a>SetClaimsIfStringsAreEqual

Проверяет, равны ли строка утверждения и входной параметр `matchTo`, и задает для исходящих утверждений значение, указанное во входных параметрах `stringMatchMsg` и `stringMatchMsgCode`, а также задает для исходящего утверждения результата сравнения значение `true` или `false` в зависимости от результата сравнения.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | строка | Тип утверждения, которое необходимо сравнить. |
| InputParameter | matchTo | строка | Строка для сравнения с `inputClaim`. |
| InputParameter | stringComparison | строка | Возможные значения: `Ordinal` или `OrdinalIgnoreCase`. |
| InputParameter | stringMatchMsg | строка | Первое значение, которое необходимо задать, если строки равны. |
| InputParameter | stringMatchMsgCode | строка | Второе значение, которое необходимо задать, если строки равны. |
| outputClaim | outputClaim1 | строка | Если строки равны, это исходящее утверждение содержит значение входного параметра `stringMatchMsg`. |
| outputClaim | outputClaim2 | строка | Если строки равны, это исходящее утверждение содержит значение входного параметра `stringMatchMsgCode`. |
| outputClaim | stringCompareResultClaim | Логическое | Тип исходящего утверждения результата сравнения, для которого необходимо задать значение `true` или `false` на основе результата сравнения. |

С помощью этого преобразования можно проверить, равно ли утверждение указанному значению. Например, следующее преобразование утверждений проверяет, равно ли значение утверждения **termsOfUseConsentVersion** значению `v1`. Если да, измените значение на `v2`.

```xml
<ClaimsTransformation Id="CheckTheTOS" TransformationMethod="SetClaimsIfStringsAreEqual">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="v1" />
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    <InputParameter Id="stringMatchMsg" DataType="string" Value="B2C_V1_90005" />
    <InputParameter Id="stringMatchMsgCode" DataType="string" Value="The TOS is upgraded to v2" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersion" TransformationClaimType="outputClaim1" />
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersionUpgradeCode" TransformationClaimType="outputClaim2" />
    <OutputClaim ClaimTypeReferenceId="termsOfUseConsentVersionUpgradeResult" TransformationClaimType="stringCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputClaim**: v1.
- Входные параметры:
    - **matchTo**: V1
    - **stringComparison**: ordinalIgnoreCase.
    - **stringMatchMsg**:  B2C_V1_90005.
    - **stringMatchMsgCode**:  "TOS обновлено до версии 2".
- Исходящие утверждения:
    - **outputClaim1**: B2C_V1_90005.
    - **outputClaim2**: "TOS обновлено до версии 2".
    - **stringCompareResultClaim**: true.

## <a name="setclaimsifstringsmatch"></a>SetClaimsIfStringsMatch

Проверяет, равны ли строка утверждения и входной параметр `matchTo`, и задает для исходящих утверждений значение, указанное во входном параметре `outputClaimIfMatched`, а также задает для исходящего утверждения результата сравнения значение `true` или `false` в зависимости от результата сравнения.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | claimToMatch | строка | Тип утверждения, которое необходимо сравнить. |
| InputParameter | matchTo | строка | Строка для сравнения с inputClaim. |
| InputParameter | stringComparison | строка | Возможные значения: `Ordinal` или `OrdinalIgnoreCase`. |
| InputParameter | outputClaimIfMatched | строка | Значение, которое необходимо задать, если строки равны. |
| outputClaim | outputClaim | строка | Если строки равны, это исходящее утверждение содержит значение входного параметра `outputClaimIfMatched`. Или значение NULL, если строки не совпадают. |
| outputClaim | stringCompareResultClaim | Логическое | Тип исходящего утверждения результата сравнения, для которого необходимо задать значение `true` или `false` на основе результата сравнения. |

Например, следующее преобразование утверждений проверяет, равно ли значение утверждения **ageGroup** значению `Minor`. Если да, возвращает значение для `B2C_V1_90001`.

```xml
<ClaimsTransformation Id="SetIsMinor" TransformationMethod="SetClaimsIfStringsMatch">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="ageGroup" TransformationClaimType="claimToMatch" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="matchTo" DataType="string" Value="Minor" />
    <InputParameter Id="stringComparison" DataType="string" Value="ordinalIgnoreCase" />
    <InputParameter Id="outputClaimIfMatched" DataType="string" Value="B2C_V1_90001" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isMinor" TransformationClaimType="outputClaim" />
    <OutputClaim ClaimTypeReferenceId="isMinorResponseCode" TransformationClaimType="stringCompareResultClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **claimToMatch**: Minor.
- Входные параметры:
    - **matchTo**: Minor.
    - **stringComparison**: ordinalIgnoreCase.
    - **outputClaimIfMatched**:  B2C_V1_90001.
- Исходящие утверждения:
    - **isMinorResponseCode**: B2C_V1_90001.
    - **isMinor**: true.


## <a name="stringcontains"></a>StringContains;

Определите, находится ли заданная подстрока во входном утверждении. Результатом является новый логический параметр ClaimType со значением `true` или `false`. Значение `true`, если параметр значения встречается в этой строке; в противном случае — значение `false`.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | строка | Тип утверждения, которое необходимо найти. |
|InputParameter|contains|строка|Значение, которое нужно найти.|
|InputParameter|ignoreCase|строка|Указывает, следует ли в этом сравнении игнорировать регистр сравниваемых строк.|
| outputClaim | outputClaim | строка | ClaimType, который создается после вызова ClaimsTransformation. Логический индикатор, если подстрока встречается во входном утверждении. |

Используйте это преобразование, чтобы проверить, содержит ли тип строкового утверждения подстроку. В следующем примере проверяется, содержит ли тип строкового утверждения `roles` значение **admin**.

```xml
<ClaimsTransformation Id="CheckIsAdmin" TransformationMethod="StringContains">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="roles" TransformationClaimType="inputClaim"/>
  </InputClaims>
  <InputParameters>
    <InputParameter  Id="contains" DataType="string" Value="admin"/>
    <InputParameter  Id="ignoreCase" DataType="string" Value="true"/>
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isAdmin" TransformationClaimType="outputClaim"/>
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputClaim**: "Admin, Approver, Editor"
- Входные параметры:
    - **contains**: "admin,"
    - **ignoreCase**: true.
- Исходящие утверждения:
    - **outputClaim**: true.

## <a name="stringsubstring"></a>StringSubstring

Извлекает части типа строкового утверждения, начиная с символа в указанной позиции, и возвращает указанное число символов.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | строка | Тип утверждения, который содержит строку. |
| InputParameter | startIndex | INT | Отсчитываемая от нуля позиция первого знака подстроки в данном экземпляре. |
| InputParameter | length | INT | Число символов в подстроке. |
| outputClaim | outputClaim | Логическое | Строка, эквивалентная подстроке с длиной, которая начинается с startIndex в данном экземпляре, или Empty, если значение startIndex равно длине данного экземпляра, а значение длины равно нулю. |

Например, получите префикс страны/региона для номера телефона.


```xml
<ClaimsTransformation Id="GetPhonePrefix" TransformationMethod="StringSubstring">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="inputClaim" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="startIndex" DataType="int" Value="0" />
  <InputParameter Id="length" DataType="int" Value="2" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phonePrefix" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputClaim**: "+1644114520"
- Входные параметры:
    - **startIndex**: 0
    - **length**:  2
- Исходящие утверждения:
    - **outputClaim**: "+1"

## <a name="stringreplace"></a>StringReplace

Выполняет поиск указанного значения в строке типа утверждения и возвращает новую строку типа утверждения, в которой все вхождения указанной строки в текущей строке заменяются другой заданной строкой.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | строка | Тип утверждения, который содержит строку. |
| InputParameter | oldValue | строка | Строка, в которой выполняется поиск. |
| InputParameter | newValue | строка | Строка для замены всех вхождений `oldValue`. |
| outputClaim | outputClaim | Логическое | Строка, эквивалентная текущей строке, но с тем отличием, что все вхождения oldValue заменены на newValue. Если oldValue не обнаружено в текущем экземпляре, метод возвращает текущий экземпляр без изменений. |

Например, нормализация номера телефона путем удаления символов `-`


```xml
<ClaimsTransformation Id="NormalizePhoneNumber" TransformationMethod="StringReplace">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="inputClaim" />
  </InputClaims>
<InputParameters>
  <InputParameter Id="oldValue" DataType="string" Value="-" />
  <InputParameter Id="newValue" DataType="string" Value="" />
</InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```
### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputClaim**: "+164-411-452-054"
- Входные параметры:
    - **oldValue**: "-"
    - **newValue**: ""
- Исходящие утверждения:
    - **outputClaim**: "+164411452054"

## <a name="stringjoin"></a>StringJoin

Сцепляет элементы указанного типа утверждения коллекции строк, помещая заданный разделитель между каждым элементом или членом.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | stringCollection | Коллекция, содержащая строки для сцепления. |
| InputParameter | разделитель | строка | Строка, используемая в качестве разделителя, например запятая `,`. |
| outputClaim | outputClaim | строка | Строка, состоящая из элементов коллекции строк `inputClaim`, разделяемых входным параметром `delimiter`. |

В следующем примере коллекция строк для ролей пользователей преобразуется в строку с разделителем запятой. Этот метод можно использовать для хранения коллекции строк в учетной записи пользователя Azure AD. Позже, при чтении учетной записи из каталога, используйте `StringSplit` для преобразования строки с разделителем запятой обратно в коллекцию строк.

```xml
<ClaimsTransformation Id="ConvertRolesStringCollectionToCommaDelimiterString" TransformationMethod="StringJoin">
  <InputClaims>
   <InputClaim ClaimTypeReferenceId="roles" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter DataType="string" Id="delimiter" Value="," />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="rolesCommaDelimiterConverted" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **inputClaim**: [ "Admin", "Author", "Reader" ]
- Входные параметры:
  - **delimiter**: ","
- Исходящие утверждения:
  - **outputClaim**: "Admin,Author,Reader"


## <a name="stringsplit"></a>StringSplit

Возвращает строковый массив, содержащий подстроки в этом экземпляре, разделенные элементами заданной строки.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | строка | Тип строкового утверждения, который содержит подстроки для разбиения. |
| InputParameter | разделитель | строка | Строка, используемая в качестве разделителя, например запятая `,`. |
| outputClaim | outputClaim | stringCollection | Коллекция строк, элементы которой содержат подстроки данной строки, разделенные входным параметром `delimiter`. |

В следующем примере строка ролей пользователей с разделителями запятыми преобразуется в коллекцию строк.

```xml
<ClaimsTransformation Id="ConvertRolesToStringCollection" TransformationMethod="StringSplit">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="rolesCommaDelimiter" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
  <InputParameter DataType="string" Id="delimiter" Value="," />
    </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="roles" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **inputClaim**: "Admin,Author,Reader"
- Входные параметры:
  - **delimiter**: ","
- Исходящие утверждения:
  - **outputClaim**: [ "Admin", "Author", "Reader" ]

## <a name="string-claim-transformations-expressions"></a>Выражения преобразований строкового утверждения
Выражения преобразования утверждений в пользовательских политиках Azure AD B2C предоставляют контекстные сведения об идентификаторе клиента и технического профиля.

  | Выражение | Описание | Пример |
 | ----- | ----------- | --------|
 | `{TechnicalProfileId}` | Имя идентификатора технического профиля. | Facebook-OAUTH |
 | `{RelyingPartyTenantId}` | Идентификатор клиента для политики проверяющей стороны. | your-tenant.onmicrosoft.com |
 | `{TrustFrameworkTenantId}` | Идентификатор клиента инфраструктуры доверия. | your-tenant.onmicrosoft.com |
