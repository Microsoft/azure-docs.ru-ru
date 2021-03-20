---
title: Примеры преобразования "утверждения даты" для пользовательских политик
description: Примеры преобразования "утверждения даты" для схемы Azure Active Directory B2C в схеме "инфраструктура процедур идентификации" (инфраструктура процедур идентификации).
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 02/16/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: eaf58b964517162ee7f7eb925e1e64830eedc087
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85202557"
---
# <a name="date-claims-transformations"></a>Преобразования утверждений даты

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

В этой статье приведены примеры использования преобразований «Дата утверждения» схемы инфраструктуры процедур идентификации в Azure Active Directory B2C (Azure AD B2C). Дополнительные сведения см. в статье о [преобразовании утверждений](claimstransformations.md).

## <a name="assertdatetimeisgreaterthan"></a>AssertDateTimeIsGreaterThan

Проверяет, что одно утверждение даты и времени (строковый тип данных) соответствует более позднему моменту, чем второе утверждение даты и времени (строковый тип данных), и создает исключение.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | leftOperand | строка | Тип первого утверждения, который должен быть больше (позже) второго утверждения. |
| InputClaim | rightOperand | строка | Тип второго утверждения, который должен быть меньше (раньше) первого утверждения. |
| InputParameter | AssertIfEqualTo | Логическое | Указывает, выполняется ли это утверждение, если левый операнд равен правому. |
| InputParameter | AssertIfRightOperandIsNotPresent | Логическое | Указывает, выполняется ли это утверждение, если правый операнд отсутствует. |
| InputParameter | TreatAsEqualIfWithinMillseconds | INT | Указывает количество миллисекунд между двумя датами и временем, после которого они считаются равными (например, чтобы учесть разницу в показаниях часов). |

Преобразование утверждений **AssertDateTimeIsGreaterThan** всегда выполняется из [технического профиля проверки](validation-technical-profile.md), вызываемого с помощью [самоподтвержденного технического профиля](self-asserted-technical-profile.md). В метаданных самоподтвержденного технического профиля **DateTimeGreaterThan** задаются сообщения об ошибках, которые технический профиль отображает пользователю. Сообщения об ошибках можно [локализовать](localization-string-ids.md#claims-transformations-error-messages).

![Выполнение AssertStringClaimsAreEqual](./media/date-transformations/assert-execution.png)

В следующем примере сравнивается утверждение `currentDateTime` с утверждением `approvedDateTime`. Ошибка возникает в случае, когда `currentDateTime` больше (позже), чем `approvedDateTime`. Преобразование считает значения равными, если они отличаются не более, чем на 5 минут (30 000 миллисекунд).

```xml
<ClaimsTransformation Id="AssertApprovedDateTimeLaterThanCurrentDateTime" TransformationMethod="AssertDateTimeIsGreaterThan">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="approvedDateTime" TransformationClaimType="leftOperand" />
    <InputClaim ClaimTypeReferenceId="currentDateTime" TransformationClaimType="rightOperand" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="AssertIfEqualTo" DataType="boolean" Value="false" />
    <InputParameter Id="AssertIfRightOperandIsNotPresent" DataType="boolean" Value="true" />
    <InputParameter Id="TreatAsEqualIfWithinMillseconds" DataType="int" Value="300000" />
  </InputParameters>
</ClaimsTransformation>
```

Технический профиль проверки `login-NonInteractive` вызывает преобразование утверждений `AssertApprovedDateTimeLaterThanCurrentDateTime`.
```xml
<TechnicalProfile Id="login-NonInteractive">
  ...
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertApprovedDateTimeLaterThanCurrentDateTime" />
  </OutputClaimsTransformations>
</TechnicalProfile>
```

Самоподтвержденный технический профиль вызывает технический профиль проверки **login-NonInteractive**.

```xml
<TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
  <Metadata>
    <Item Key="DateTimeGreaterThan">Custom error message if the provided left operand is greater than the right operand.</Item>
  </Metadata>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="login-NonInteractive" />
  </ValidationTechnicalProfiles>
</TechnicalProfile>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **лефтоперанд**: 2020-03-01T15:00:00.0000000 z
    - **ригхтоперанд**: 2020-03-01T14:00:00.0000000 z
- Результат: возникла ошибка.

## <a name="convertdatetodatetimeclaim"></a>ConvertDateToDateTimeClaim

Преобразует ClaimType **Date** в тип **DateTime**. Преобразование утверждений преобразует формат времени и добавляет 00:00:00 к дате.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | Дата | ClaimType, который необходимо преобразовать. |
| outputClaim | outputClaim | dateTime | ClaimType, который создается после вызова ClaimsTransformation. |

В следующем примере показано преобразование утверждения `dateOfBirth` (тип данных date) в другое утверждение `dateOfBirthWithTime` (тип данных dateTime).

```xml
  <ClaimsTransformation Id="ConvertToDateTime" TransformationMethod="ConvertDateToDateTimeClaim">
    <InputClaims>
      <InputClaim ClaimTypeReferenceId="dateOfBirth" TransformationClaimType="inputClaim" />
    </InputClaims>
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="dateOfBirthWithTime" TransformationClaimType="outputClaim" />
    </OutputClaims>
  </ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **inputClaim**: 2020-15-03
- Исходящие утверждения:
    - **outputClaim**: 2020-15-03T00:00:00.0000000 z

## <a name="convertdatetimetodateclaim"></a>конвертдатетиметодатеклаим

Преобразует тип данных DateTime в **дату** - **время** . Преобразование «утверждения» удаляет формат времени из даты.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | dateTime | ClaimType, который необходимо преобразовать. |
| outputClaim | outputClaim | Дата | ClaimType, который создается после вызова ClaimsTransformation. |

В следующем примере показано преобразование утверждения `systemDateTime` (тип данных DateTime) в другое утверждение `systemDate` (тип данных Date).

```xml
<ClaimsTransformation Id="ConvertToDate" TransformationMethod="ConvertDateTimeToDateClaim">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="systemDateTime" TransformationClaimType="inputClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="systemDate" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **inputClaim**: 2020-15-03T11:34:22.0000000 z
- Исходящие утверждения:
  - **outputClaim**: 2020-15-03

## <a name="getcurrentdatetime"></a>GetCurrentDateTime

Получает текущую дату и время в формате UTC и добавляет это значение к ClaimType.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| outputClaim | currentDateTime | dateTime | ClaimType, который создается после вызова ClaimsTransformation. |

```xml
<ClaimsTransformation Id="GetSystemDateTime" TransformationMethod="GetCurrentDateTime">
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="systemDateTime" TransformationClaimType="currentDateTime" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

* Исходящие утверждения:
    * **куррентдатетиме**: 2020-15-03T11:40:35.0000000 z

## <a name="datetimecomparison"></a>DateTimeComparison

Определяет, является ли одно утверждение dateTime больше (позже), меньше (раньше) или равно другому. Результатом является новый логический параметр ClaimType со значением `true` или `false`.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | firstDateTime | dateTime | Первое значение dateTime для сравнения со вторым значением dateTime. При значении NULL возникает исключение. |
| InputClaim | secondDateTime | dateTime | Второе значение dateTime для сравнения с первым значением dateTime. Значение NULL обрабатывается как текущее значение dateTime. |
| InputParameter | оператор | строка | Одно из следующих значений: "same", "later than" или "earlier than" ("равно", "позже" или "раньше"). |
| InputParameter | timeSpanInSeconds | INT | Добавление временного диапазона к первым дате и времени. |
| outputClaim | result | Логическое | ClaimType, который создается после вызова ClaimsTransformation. |

Это преобразование позволяет определить, являются ли два утверждения ClaimType равными, больше (позже) или меньше (раньше) друг друга. Например, вы можете сохранить время, когда пользователь последний раз принимал ваши условия предоставления услуг (TOS). Через 3 месяца вы можете попросить пользователя принять TOS еще раз.
Чтобы выполнить преобразование утверждений, необходимо сначала получить текущее значение даты и времени, а также время, когда пользователь последний раз принимал TOS.

```xml
<ClaimsTransformation Id="CompareLastTOSAcceptedWithCurrentDateTime" TransformationMethod="DateTimeComparison">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="currentDateTime" TransformationClaimType="firstDateTime" />
    <InputClaim ClaimTypeReferenceId="extension_LastTOSAccepted" TransformationClaimType="secondDateTime" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="operator" DataType="string" Value="later than" />
    <InputParameter Id="timeSpanInSeconds" DataType="int" Value="7776000" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="isLastTOSAcceptedGreaterThanNow" TransformationClaimType="result" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
    - **фирстдатетиме**: 2020-01-01T00:00:00.100000 z
    - **секонддатетиме**: 2020-04-01T00:00:00.100000 z
- Входные параметры:
    - **operator**: later than
    - **timeSpanInSeconds**: 7776000 (90 дней)
- Исходящие утверждения:
    - **result**: true
