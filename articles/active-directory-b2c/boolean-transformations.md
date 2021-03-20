---
title: Примеры преобразования логических утверждений для пользовательских политик
titleSuffix: Azure AD B2C
description: Примеры преобразования логических утверждений для схемы Identity Experience Framework (IEF) в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 06/06/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 7c292f939339add06168c55236f8666651e4aace
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85201282"
---
# <a name="boolean-claims-transformations"></a>Преобразования логических утверждений

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

В этой статье приведены примеры использования преобразований логических утверждений схемы инфраструктуры процедур идентификации в Azure Active Directory B2C (Azure AD B2C). Дополнительные сведения см. в статье о [преобразовании утверждений](claimstransformations.md).

## <a name="andclaims"></a>Утверждение AndClaims

Выполняет операцию AND для двух логических элементов InputClaim и задает элемент OutputClaim с результатом операции.

| Item  | TransformationClaimType  | Тип данных  | Примечания |
|-------| ------------------------ | ---------- | ----- |
| InputClaim | inputClaim1 | Логическое | Первый оцениваемый элемент ClaimType. |
| InputClaim | inputClaim2  | Логическое | Второй оцениваемый элемент ClaimType. |
|outputClaim | outputClaim | Логическое | Элементы ClaimType, создаваемые после вызова этого преобразования утверждений (true или false). |

В следующем преобразовании утверждений показано, как выполнять операцию AND для двух логических элементов ClaimType: `isEmailNotExist` и `isSocialAccount`. Для исходящего утверждения `presentEmailSelfAsserted` присваивается значение `true`, если значения обоих входящих утверждений — `true`. На шаге оркестрации можно использовать необходимое условие для предварительной настройки страницы самоподтверждения, только если электронная почта учетной записи социальной сети пуста.

```xml
<ClaimsTransformation Id="CheckWhetherEmailBePresented" TransformationMethod="AndClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="isEmailNotExist" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="isSocialAccount" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="presentEmailSelfAsserted" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example-of-andclaims"></a>Пример Андклаимс

- Входящие утверждения:
    - **inputClaim1**: true.
    - **inputClaim2**: false.
- Исходящие утверждения:
    - **outputClaim**: false.


## <a name="assertbooleanclaimisequaltovalue"></a>AssertBooleanClaimIsEqualToValue

Проверяет, равны ли логические значения двух утверждений, и создает исключение, если это не так.

| Item | TransformationClaimType  | Тип данных  | Примечания |
| ---- | ------------------------ | ---------- | ----- |
| InputClaim | InputClaim | Логическое | Элемент ClaimType, который необходимо подтвердить. |
| InputParameter |valueToCompareTo | Логическое | Значение для сравнения (true или false). |

Преобразование утверждений **AssertBooleanClaimIsEqualToValue** всегда выполняется с помощью [технического профиля проверки](validation-technical-profile.md), вызываемого через [самоподтвержденный технический профиль](self-asserted-technical-profile.md). Метаданные самоподтвержденного технического профиля **UserMessageIfClaimsTransformationBooleanValueIsNotEqual** управляют сообщением, поступающим пользователю из технического профиля. Сообщения об ошибках можно [локализовать](localization-string-ids.md#claims-transformations-error-messages).

![Выполнение AssertStringClaimsAreEqual](./media/boolean-transformations/assert-execution.png)

В следующем преобразовании утверждений показано, как проверить значение логического элемента ClaimType на основе значения `true`. Если значение ClaimType `accountEnabled` — false, выдается сообщение об ошибке.

```xml
<ClaimsTransformation Id="AssertAccountEnabledIsTrue" TransformationMethod="AssertBooleanClaimIsEqualToValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="accountEnabled" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="valueToCompareTo" DataType="boolean" Value="true" />
  </InputParameters>
</ClaimsTransformation>
```


Технический профиль проверки `login-NonInteractive` вызывает преобразование утверждений `AssertAccountEnabledIsTrue`.

```xml
<TechnicalProfile Id="login-NonInteractive">
  ...
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertAccountEnabledIsTrue" />
  </OutputClaimsTransformations>
</TechnicalProfile>
```

Самоподтвержденный технический профиль вызывает технический профиль проверки **login-NonInteractive**.

```xml
<TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Email">
  <Metadata>
    <Item Key="UserMessageIfClaimsTransformationBooleanValueIsNotEqual">Custom error message if account is disabled.</Item>
  </Metadata>
  <ValidationTechnicalProfiles>
    <ValidationTechnicalProfile ReferenceId="login-NonInteractive" />
  </ValidationTechnicalProfiles>
</TechnicalProfile>
```

### <a name="example-of-assertbooleanclaimisequaltovalue"></a>Пример Ассертбулеанклаимисекуалтовалуе

- Входящие утверждения:
    - **inputClaim**: false.
    - **valueToCompareTo**: true.
- Результат: возникла ошибка.

## <a name="comparebooleanclaimtovalue"></a>CompareBooleanClaimToValue

Проверяет, что логическое значение утверждения равно `true` или `false`, и возвращает результат сжатия.

| Item | TransformationClaimType  | Тип данных  | Примечания |
| ---- | ------------------------ | ---------- | ----- |
| InputClaim | InputClaim | Логическое | Элемент ClaimType, который необходимо подтвердить. |
| InputParameter |valueToCompareTo | Логическое | Значение для сравнения (true или false). |
| outputClaim | compareResult | Логическое | ClaimType, который создается после вызова ClaimsTransformation. |

В следующем преобразовании утверждений показано, как проверить значение логического элемента ClaimType на основе значения `true`. Если значение `IsAgeOver21Years` ClaimType равно `true`, преобразование утверждений возвращает `true`, в противном случае — `false`.

```xml
<ClaimsTransformation Id="AssertAccountEnabled" TransformationMethod="CompareBooleanClaimToValue">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="IsAgeOver21Years" TransformationClaimType="inputClaim" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="valueToCompareTo" DataType="boolean" Value="true" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim  ClaimTypeReferenceId="accountEnabled" TransformationClaimType="compareResult"/>
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example-of-comparebooleanclaimtovalue"></a>Пример Компаребулеанклаимтовалуе

- Входящие утверждения:
    - **inputClaim**: false.
- Входные параметры:
    - **valueToCompareTo**: true.
- Исходящие утверждения:
    - **compareResult**: false

## <a name="notclaims"></a>NotClaims

Выполняет операцию Not для логического элемента inputClaim и задает элемент outputClaim с результатом операции.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | InputClaim | Логическое | Обрабатываемое утверждение. |
| outputClaim | outputClaim | Логическое | Элементы ClaimType, создаваемые после вызова данного ClaimsTransformation (true или false). |

Это преобразование утверждений используется для выполнения логического отрицания утверждения.

```xml
<ClaimsTransformation Id="CheckWhetherEmailBePresented" TransformationMethod="NotClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="userExists" TransformationClaimType="inputClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="userExists" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example-of-notclaims"></a>Пример Нотклаимс

- Входящие утверждения:
    - **inputClaim**: false.
- Исходящие утверждения:
    - **outputClaim**: true.

## <a name="orclaims"></a>OrClaims

Вычисляет значение Or для двух логических элементов InputClaim и задает элемент outputClaim с результатом операции.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputClaim1 | Логическое | Первый оцениваемый элемент ClaimType. |
| InputClaim | inputClaim2 | Логическое | Второй оцениваемый элемент ClaimType. |
| outputClaim | outputClaim | Логическое | Элементы ClaimType, создаваемые после вызова ClaimsTransformation (true или false). |

В следующем преобразовании утверждений показано, как выполнять операцию `Or` для двух логических элементов ClaimType. На шаге оркестрации можно использовать необходимое условие для предварительной настройки страницы самоподтверждения, если значение одного из утверждений — `true`.

```xml
<ClaimsTransformation Id="CheckWhetherEmailBePresented" TransformationMethod="OrClaims">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="isLastTOSAcceptedNotExists" TransformationClaimType="inputClaim1" />
    <InputClaim ClaimTypeReferenceId="isLastTOSAcceptedGreaterThanNow" TransformationClaimType="inputClaim2" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="presentTOSSelfAsserted" TransformationClaimType="outputClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example-of-orclaims"></a>Пример Орклаимс

- Входящие утверждения:
    - **inputClaim1**: true.
    - **inputClaim2**: false.
- Исходящие утверждения:
    - **outputClaim**: true.
