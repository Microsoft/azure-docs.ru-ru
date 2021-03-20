---
title: Преобразования утверждений номеров телефона в настраиваемых политиках
titleSuffix: Azure AD B2C
description: Справочные материалы по настраиваемой политике для преобразования утверждений номеров телефона в Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 02/26/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: e175a81efc1ab0950c1fda314efb206ff97a2b7f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85385388"
---
# <a name="define-phone-number-claims-transformations-in-azure-ad-b2c"></a>Определение преобразований утверждений номеров телефона в Azure AD B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

В этой статье приведены справочные сведения и примеры использования преобразований утверждений номеров телефона схемы Identity Experience Framework в Azure Active Directory B2C (Azure AD B2C). Дополнительные общие сведения о преобразованиях утверждений см. в разделе [ClaimsTransformations](claimstransformations.md).

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="convertphonenumberclaimtostring"></a>ConvertPhoneNumberClaimToString

Преобразовывает тип данных `phoneNumber` в тип данных `string`.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | phoneNumber | phoneNumber |  ClaimType для преобразования в строку. |
| outputClaim | phoneNumberString | строка | Параметр ClaimType, который создается после вызова этого преобразования утверждений. |

В этом примере утверждение cellPhoneNumber с типом значения `phoneNumber` преобразуется в утверждение cellPhone с типом значения `string`.

```xml
<ClaimsTransformation Id="PhoneNumberToString" TransformationMethod="ConvertPhoneNumberClaimToString">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="cellPhoneNumber" TransformationClaimType="phoneNumber" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="cellPhone" TransformationClaimType="phoneNumberString" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **phoneNumber**: +11234567890 (phoneNumber)
- Исходящие утверждения:
  - **phoneNumberString**: +11234567890 (строка)


## <a name="convertstringtophonenumberclaim"></a>ConvertStringToPhoneNumberClaim

Это преобразование утверждения проверяет формат номера телефона. Если номер телефона имеет допустимый формат, он преобразовывается в стандартный формат, используемый Azure AD B2C. Если номер телефона указан в недопустимом формате, возвращается сообщение об ошибке.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | phoneNumberString | строка |  Строковое утверждение для номера телефона. Номер телефона должен быть указан в международном формате, включая знак "+" и код страны или региона. Если добавлено входящее утверждение `country`, номер телефона указывается в местном формате (без кода страны или региона). |
| InputClaim | country | строка | (Необязательно.) Строковое утверждение для кода страны или региона номера телефона в формате ISO 3166 (двухбуквенный код страны и региона в формате ISO 3166). |
| outputClaim | outputClaim | phoneNumber | Результат преобразования утверждений. |

Преобразование утверждений **ConvertStringToPhoneNumberClaim** всегда выполняется из [технического профиля проверки](validation-technical-profile.md), вызываемого с помощью [самоподтвержденного технического профиля](self-asserted-technical-profile.md) или [элемента управления отображением](display-controls.md). Метаданные самоподтвержденного технического профиля **UserMessageIfClaimsTransformationInvalidPhoneNumber** управляют сообщением об ошибке, представляемым пользователю.

![Схема пути выполнения сообщения об ошибке](./media/phone-authentication/assert-execution.png)

Это преобразование утверждений можно использовать, чтобы убедиться, что указанное строковое утверждение является допустимым номером телефона. В противном случае выдается сообщение об ошибке. Следующий пример проверяет, что значение ClaimType для **phoneString** действительно соответствует допустимому номеру телефона, а затем возвращает номер телефона в стандартном формате Azure AD B2C. В противном случае отображается сообщение об ошибке.

```xml
<ClaimsTransformation Id="ConvertStringToPhoneNumber" TransformationMethod="ConvertStringToPhoneNumberClaim">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phoneString" TransformationClaimType="phoneNumberString" />
    <InputClaim ClaimTypeReferenceId="countryCode" TransformationClaimType="country" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

Самоподтвержденный технический профиль, который вызывает технический профиль проверки, содержащий это преобразование утверждений, может определять сообщение об ошибке.

```xml
<TechnicalProfile Id="SelfAsserted-LocalAccountSignup-Phone">
  <Metadata>
    <Item Key="UserMessageIfClaimsTransformationInvalidPhoneNumber">Custom error message if the phone number is not valid.</Item>
  </Metadata>
  ...
</TechnicalProfile>
```

### <a name="example-1"></a>Пример 1

- Входящие утверждения:
  - **phoneNumberString**: 033 456-7890
  - **country**: DK
- Исходящие утверждения:
  - **outputClaim**: +450334567890

### <a name="example-2"></a>Пример 2

- Входящие утверждения:
  - **phoneNumberString**: +1 (123) 456-7890
- Исходящие утверждения:
  - **outputClaim**: +11234567890


## <a name="getnationalnumberandcountrycodefromphonenumberstring"></a>GetNationalNumberAndCountryCodeFromPhoneNumberString

Извлекает код страны или региона, а также национальный номер из входного утверждения. При необходимости порождает исключение, если указанный номер телефона является недопустимым.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | phoneNumber | строка | Строковое утверждение для номера телефона. Номер телефона должен быть указан в международном формате, включая знак "+" и код страны или региона. |
| InputParameter | throwExceptionOnFailure | Логическое | (Необязательно.) Параметр, указывающий, порождать ли исключение, если номер телефона является недопустимым. Значение по умолчанию — false. |
| InputParameter | countryCodeType | строка | (Необязательно.) Параметр, указывающий тип кода страны или региона в выходном утверждении. Допустимые значения: **CallingCode** (международный код страны или региона, например +1) или **ISO3166** (двухбуквенный код страны и региона в формате ISO 3166). |
| outputClaim | nationalNumber | строка | Строковое утверждение для национального номера телефона. |
| outputClaim | countryCode | строка | Строковое утверждение для кода страны или региона номера телефона. |


Если преобразование утверждений **GetNationalNumberAndCountryCodeFromPhoneNumberString** выполняется из [технического профиля проверки](validation-technical-profile.md), вызванного [самоподтвержденным техническим профилем](self-asserted-technical-profile.md) или [элементом управления отображением](display-controls.md#display-control-actions), то сообщением об ошибке, отображаемым пользователю, управляют метаданные **UserMessageIfPhoneNumberParseFailure** самоподтвержденного технического профиля.

![Схема пути выполнения сообщения об ошибке](./media/phone-authentication/assert-execution.png)

Это преобразование утверждений можно использовать для разбиения полного номера телефона на код страны или региона и национальный номер. Если указан недопустимый номер телефона, можно отобразить сообщение об ошибке.

В следующем примере производится попытка разделить номер телефона на национальной номер и код страны или региона. Если номер телефона действителен, то номер телефона будет заменен национальным номером. Если номер телефона является недопустимым, то исключение не будет порождено, а номер телефона сохранит свое исходное значение.

```xml
<ClaimsTransformation Id="GetNationalNumberAndCountryCodeFromPhoneNumberString" TransformationMethod="GetNationalNumberAndCountryCodeFromPhoneNumberString">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="phoneNumber" TransformationClaimType="phoneNumber" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="throwExceptionOnFailure" DataType="boolean" Value="false" />
    <InputParameter Id="countryCodeType" DataType="string" Value="ISO3166" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="nationalNumber" TransformationClaimType="nationalNumber" />
    <OutputClaim ClaimTypeReferenceId="countryCode" TransformationClaimType="countryCode" />
  </OutputClaims>
</ClaimsTransformation>
```

Самоподтвержденный технический профиль, который вызывает технический профиль проверки, содержащий это преобразование утверждений, может определять сообщение об ошибке.

```xml
<TechnicalProfile Id="SelfAsserted-LocalAccountSignup-Phone">
  <Metadata>
    <Item Key="UserMessageIfPhoneNumberParseFailure">Custom error message if the phone number is not valid.</Item>
  </Metadata>
  ...
</TechnicalProfile>
```

### <a name="example-1"></a>Пример 1

- Входящие утверждения:
  - **phoneNumber**: +49 (123) 456-7890
- Входные параметры:
  - **throwExceptionOnFailure**: false
  - **countryCodeType**: ISO3166
- Исходящие утверждения:
  - **nationalNumber**: 1234567890
  - **countryCode**: DE

### <a name="example-2"></a>Пример 2

- Входящие утверждения:
  - **phoneNumber**: +49 (123) 456-7890
- Входные параметры
  - **throwExceptionOnFailure**: false
  - **countryCodeType**: CallingCode
- Исходящие утверждения:
  - **nationalNumber**: 1234567890
  - **countryCode**: +49
