---
title: Примеры преобразования утверждений JSON для пользовательских политик
titleSuffix: Azure AD B2C
description: Примеры преобразования утверждений JSON для схемы Azure Active Directory B2C (инфраструктура процедур идентификации).
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/04/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: c5c8e21f2ce3f6907547bf1b2fe4681eb937864b
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102119881"
---
# <a name="json-claims-transformations"></a>Преобразования утверждений JSON

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

В этой статье приведены примеры использования преобразований утверждений JSON схемы инфраструктуры процедур идентификации в Azure Active Directory B2C (Azure AD B2C). Дополнительные сведения см. в статье о [преобразовании утверждений](claimstransformations.md).

## <a name="generatejson"></a>женератежсон

Чтобы создать строку JSON, используйте либо значения утверждения, либо константы. Строка пути, следующая за точкой, используется для указания места вставки данных в строку JSON. После разделения по точкам любые целые числа интерпретируется как индекс массива JSON, а Нецелочисленные значения — как индекс объекта JSON.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | Любая строка в следующей нотации с точкой | строка | JsonPath JSON, в который будет вставлено значение утверждения. |
| InputParameter | Любая строка в следующей нотации с точкой | строка | JsonPath JSON, в который будет вставлено строковое значение константы. |
| outputClaim | outputClaim | строка | Созданная строка JSON. |

### <a name="example-1"></a>Пример 1

В следующем примере создается строка JSON на основе значения утверждения "Email" и "OTP", а также константных строк.

```xml
<ClaimsTransformation Id="GenerateRequestBody" TransformationMethod="GenerateJson">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="personalizations.0.to.0.email" />
    <InputClaim ClaimTypeReferenceId="otp" TransformationClaimType="personalizations.0.dynamic_template_data.otp" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="template_id" DataType="string" Value="d-4c56ffb40fa648b1aa6822283df94f60"/>
    <InputParameter Id="from.email" DataType="string" Value="service@contoso.com"/>
    <InputParameter Id="personalizations.0.subject" DataType="string" Value="Contoso account email verification code"/>
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="requestBody" TransformationClaimType="outputClaim"/>
  </OutputClaims>
</ClaimsTransformation>
```

В следующем преобразовании утверждений выводится строковый запрос JSON, который будет являться телом запроса, отправленного в SendGrid (поставщик электронной почты стороннего поставщика). Структура объекта JSON определяется идентификаторами в точечной нотации InputParameters и Трансформатионклаимтипес Inputclaim. Числа в точечной нотации подразумевают массивы. Значения берутся из значений Inputclaim и свойств InputParameters "" value ".

- Входные утверждения:
  - **адрес электронной почты**, персонализации для типа утверждения  **. 0. в. 0. Электронная почта**: " someone@example.com "
  - **OTP**, тип утверждения преобразования **personalizations.0.dynamic_template_data. OTP** "346349"
- Входной параметр:
  - **template_id**: "d-4c56ffb40fa648b1aa6822283df94f60"
  - **из. адрес электронной почты**: " service@contoso.com "
  - **Личные настройки. 0. Тема** "код проверки электронной почты учетной записи Contoso"
- Исходящее утверждение:
  - **requestBody**: значение JSON

```json
{
  "personalizations": [
    {
      "to": [
        {
          "email": "someone@example.com"
        }
      ],
      "dynamic_template_data": {
        "otp": "346349",
        "verify-email" : "someone@example.com"
      },
      "subject": "Contoso account email verification code"
    }
  ],
  "template_id": "d-989077fbba9746e89f3f6411f596fb96",
  "from": {
    "email": "service@contoso.com"
  }
}
```

### <a name="example-2"></a>Пример 2

В следующем примере создается строка JSON на основе значений утверждений, а также константных строк.

```xml
<ClaimsTransformation Id="GenerateRequestBody" TransformationMethod="GenerateJson">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" TransformationClaimType="customerEntity.email" />
    <InputClaim ClaimTypeReferenceId="objectId" TransformationClaimType="customerEntity.userObjectId" />
    <InputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="customerEntity.firstName" />
    <InputClaim ClaimTypeReferenceId="surname" TransformationClaimType="customerEntity.lastName" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="customerEntity.role.name" DataType="string" Value="Administrator"/>
    <InputParameter Id="customerEntity.role.id" DataType="long" Value="1"/>
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="requestBody" TransformationClaimType="outputClaim"/>
  </OutputClaims>
</ClaimsTransformation>
```

В следующем преобразовании утверждений выводится строковый запрос JSON, который будет основным текстом запроса, отправленного в REST API. Структура объекта JSON определяется идентификаторами в точечной нотации InputParameters и Трансформатионклаимтипес Inputclaim. Значения берутся из значений Inputclaim и свойств InputParameters "" value ".

- Входные утверждения:
  - **адрес электронной почты**, утверждение преобразования типа  **customerEntity. email**: " john.s@contoso.com "
  - **ObjectID**, тип утверждения преобразования **customerEntity. значение userobjectid** "01234567-89ab-cdef-0123-456789abcdef"
  - **givenName**, утверждение преобразования тип **customerEntity. FirstName** "Джон"
  - **Фамилия**, утверждение преобразования тип **customerEntity. LastName** "Смит"
- Входной параметр:
  - **customerEntity.Role.Name**: "Администратор"
  - **customerEntity.Role.ID** 1
- Исходящее утверждение:
  - **requestBody**: значение JSON

```json
{
   "customerEntity":{
      "email":"john.s@contoso.com",
      "userObjectId":"01234567-89ab-cdef-0123-456789abcdef",
      "firstName":"John",
      "lastName":"Smith",
      "role":{
         "name":"Administrator",
         "id": 1
      }
   }
}
```

## <a name="getclaimfromjson"></a>GetClaimFromJson

Возвращает указанный элемент из данных JSON.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputJson | строка | Параметр ClaimTypes, используемый в преобразовании утверждений для получения элемента. |
| InputParameter | claimToExtract | строка | Имя элемента JSON, который требуется извлечь. |
| outputClaim | extractedClaim | строка | Параметр ClaimType, созданный после вызова преобразования утверждений (значение элемента указано во входном параметре _claimToExtract_). |

В следующем примере в процессе преобразования утверждений извлекается элемент `emailAddress` из данных JSON: `{"emailAddress": "someone@example.com", "displayName": "Someone"}`

```xml
<ClaimsTransformation Id="GetEmailClaimFromJson" TransformationMethod="GetClaimFromJson">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="customUserData" TransformationClaimType="inputJson" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="claimToExtract" DataType="string" Value="emailAddress" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="extractedEmail" TransformationClaimType="extractedClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **inputJson**: {"emailAddress": "someone@example.com", "displayName": "Someone"}
- Входной параметр:
    - **claimToExtract**: emailAddress.
- Исходящие утверждения:
  - **екстрактедклаим**: someone@example.com


## <a name="getclaimsfromjsonarray"></a>GetClaimsFromJsonArray

Возвращает список указанных элементов из данных JSON.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | jsonSourceClaim | строка | Параметр ClaimTypes, используемый в преобразовании утверждений для получения утверждений. |
| InputParameter | errorOnMissingClaims | Логическое | Указывает, следует ли выдавать ошибку, если одно из утверждений отсутствует. |
| InputParameter | includeEmptyClaims | строка | Указывает, следует ли включать пустые утверждения. |
| InputParameter | jsonSourceKeyName | строка | Имя ключа элемента |
| InputParameter | jsonSourceValueName | строка | Имя значения элемента |
| outputClaim | Коллекция | string, int, boolean и datetime |Список утверждений для извлечения. Имя утверждения должно соответствовать указанному имени во входящем утверждении _jsonSourceClaim_. |

В следующем примере в процессе преобразования утверждений из данных JSON извлекаются следующие утверждения: email (string), displayName (string), membershipNum (int), active (boolean) и birthdate (datetime).

```json
[{"key":"email","value":"someone@example.com"}, {"key":"displayName","value":"Someone"}, {"key":"membershipNum","value":6353399}, {"key":"active","value":true}, {"key":"birthdate","value":"1980-09-23T00:00:00Z"}]
```

```xml
<ClaimsTransformation Id="GetClaimsFromJson" TransformationMethod="GetClaimsFromJsonArray">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="jsonSourceClaim" TransformationClaimType="jsonSource" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="errorOnMissingClaims" DataType="boolean" Value="false" />
    <InputParameter Id="includeEmptyClaims" DataType="boolean" Value="false" />
    <InputParameter Id="jsonSourceKeyName" DataType="string" Value="key" />
    <InputParameter Id="jsonSourceValueName" DataType="string" Value="value" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" />
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="membershipNum" />
    <OutputClaim ClaimTypeReferenceId="active" />
    <OutputClaim ClaimTypeReferenceId="birthdate" />
  </OutputClaims>
</ClaimsTransformation>
```

- Входящие утверждения:
  - **jsonSourceClaim**: [{"key":"email","value":"someone@example.com"}, {"key":"displayName","value":"Someone"}, {"key":"membershipNum","value":6353399}, {"key":"active","value": true}, {"key":"birthdate","value":"1980-09-23T00:00:00Z"}]
- Входные параметры:
    - **errorOnMissingClaims**: false.
    - **includeEmptyClaims**: false.
    - **jsonSourceKeyName**: key.
    - **jsonSourceValueName**: value.
- Исходящие утверждения:
  - **адрес электронной почты**: " someone@example.com "
  - **displayName**: "Someone".
  - **membershipNum**: 6353399.
  - **active**: true.
  - **birthdate**: 1980-09-23T00:00:00Z.

## <a name="getnumericclaimfromjson"></a>GetNumericClaimFromJson

Возвращает указанный числовой элемент (long) из данных JSON.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputJson | строка | Параметр ClaimTypes, используемый в преобразовании утверждений для получения утверждения. |
| InputParameter | claimToExtract | строка | Имя извлекаемого элемента JSON. |
| outputClaim | extractedClaim | long | Параметр ClaimType, который создается после вызова ClaimsTransformation (значение элемента указано во входных параметрах _claimToExtract_). |

В следующем примере в процессе преобразования утверждений из данных JSON извлекается элемент `id`.

```json
{
    "emailAddress": "someone@example.com",
    "displayName": "Someone",
    "id" : 6353399
}
```

```xml
<ClaimsTransformation Id="GetIdFromResponse" TransformationMethod="GetNumericClaimFromJson">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="exampleInputClaim" TransformationClaimType="inputJson" />
  </InputClaims>
  <InputParameters>
    <InputParameter Id="claimToExtract" DataType="string" Value="id" />
  </InputParameters>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="membershipId" TransformationClaimType="extractedClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **inputJson**: {"emailAddress": "someone@example.com", "displayName": "Someone", "id" : 6353399}
- Входные параметры
    - **claimToExtract**: id.
- Исходящие утверждения:
    - **extractedClaim**: 6353399.

## <a name="getsingleitemfromjson"></a>жетсинглеитемфромжсон

Возвращает первый элемент из данных JSON.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputJson | строка | ClaimTypes, используемые преобразованием «утверждения» для получения элемента из данных JSON. |
| outputClaim | ключ | строка | Первый ключ элемента в JSON. |
| outputClaim | value | строка | Значение первого элемента в JSON. |

В следующем примере преобразование «утверждения» извлекает первый элемент (с заданным именем) из данных JSON.

```xml
<ClaimsTransformation Id="GetGivenNameFromResponse" TransformationMethod="GetSingleItemFromJson">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="json" TransformationClaimType="inputJson" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="givenNameKey" TransformationClaimType="key" />
    <OutputClaim ClaimTypeReferenceId="givenName" TransformationClaimType="value" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **инпутжсон**: {"givenName": "емилти", "LastName": "Иванов"}
- Исходящие утверждения:
  - **ключ**: givenName
  - **значение**: емилти


## <a name="getsinglevaluefromjsonarray"></a>GetSingleValueFromJsonArray

Возвращает первый элемент из массива данных JSON.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | inputJsonClaim | строка | Параметр ClaimTypes, используемый в преобразовании утверждений для получения элемента из массива JSON. |
| outputClaim | extractedClaim | строка | ClaimType, который создается после вызова ClaimsTransformation (первый элемент в массиве JSON). |

В следующем примере в процессе преобразования утверждений из массива JSON `["someone@example.com", "Someone", 6353399]` извлекается первый элемент (адрес электронной почты).

```xml
<ClaimsTransformation Id="GetEmailFromJson" TransformationMethod="GetSingleValueFromJsonArray">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="userData" TransformationClaimType="inputJsonClaim" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="email" TransformationClaimType="extractedClaim" />
  </OutputClaims>
</ClaimsTransformation>
```

### <a name="example"></a>Пример

- Входящие утверждения:
  - **инпутжсонклаим**: [" someone@example.com ", "кто", 6353399]
- Исходящие утверждения:
  - **екстрактедклаим**: someone@example.com

## <a name="xmlstringtojsonstring"></a>XmlStringToJsonString

Преобразует данные XML в формат JSON.

| Item | TransformationClaimType | Тип данных | Примечания |
| ---- | ----------------------- | --------- | ----- |
| InputClaim | Xml | строка | Параметр ClaimTypes, используемый в преобразовании утверждений для преобразования данных из языка XML в формат JSON. |
| outputClaim | json | строка | ClaimType, который создается после вызова ClaimsTransformation (данные в формате JSON). |

```xml
<ClaimsTransformation Id="ConvertXmlToJson" TransformationMethod="XmlStringToJsonString">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="intpuXML" TransformationClaimType="xml" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="outputJson" TransformationClaimType="json" />
  </OutputClaims>
</ClaimsTransformation>
```

В следующем примере в процессе преобразования утверждений в формат JSON преобразовываются следующие данные XML.

#### <a name="example"></a>Пример
Входящее утверждение:

```xml
<user>
  <name>Someone</name>
  <email>someone@example.com</email>
</user>
```

Исходящее утверждение:

```json
{
  "user": {
    "name":"Someone",
    "email":"someone@example.com"
  }
}
```


