---
title: Локализация — Azure Active Directory B2C
description: Использование элемента Localization в настраиваемой политике для Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.custom: b2c-support
ms.openlocfilehash: c12a171faaf069cc81aa5f467642dba6616a866f
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102033730"
---
# <a name="localization-element"></a>Элемент Localization

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Элемент **Localization** обеспечивает поддержку несколько языковых стандартов или языков в политике для путей взаимодействия пользователя. Поддержка локализации в политиках позволяет выполнить следующее:

- настроить в политике явный список поддерживаемых языков и выбрать язык по умолчанию;
- использовать строки и коллекции, зависящие от языка.

```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="ReplaceAll">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
  </SupportedLanguages>
  <LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedResources Id="api.localaccountsignup.es">
  ...
```

Элемент **Localization** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| Активировано | Нет | Возможные значения: `true` или `false`. |

Элемент **Localization** содержит следующие элементы XML:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| SupportedLanguages | 1:n | Список поддерживаемых языков. |
| LocalizedResources | 0:n | Список локализованных ресурсов. |

## <a name="supportedlanguages"></a>SupportedLanguages

Элемент **SupportedLanguages** содержит следующие атрибуты.

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| DefaultLanguage | Да | Язык, который будет использоваться по умолчанию для локализованных ресурсов. |
| MergeBehavior | Нет | Перечисление значений, которые объединяются с любым элементом ClaimType с таким же идентификатором в родительской политике. Используйте этот атрибут, чтобы переопределить определенное в базовой политике утверждение. Возможные значения: `Append`, `Prepend` или `ReplaceAll`. Значение `Append` означает, что указанная коллекция данных должна добавляться в конец коллекции, указанной в родительской политике. Значение `Prepend` означает, что указанная коллекция данных должна добавляться в начало коллекции, указанной в родительской политике. Значение `ReplaceAll` означает, что определенную в родительской политике коллекцию данных нужно игнорировать и заменить данными, определенными в текущей политике. |

### <a name="supportedlanguages"></a>SupportedLanguages

Элемент **SupportedLanguages** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| SupportedLanguage | 1:n | Отображает содержимое, которое соответствует тега языка в RFC 5646 (теги для идентификации языков). |

## <a name="localizedresources"></a>LocalizedResources

Элемент **LocalizedResources** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| Идентификатор | Да | Идентификатор, который используется для уникальной идентификации локализованных ресурсов. |

Элемент **LocalizedResources** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| LocalizedCollections | 0:n | Определяет целые коллекции для разных языков и региональных параметров. Коллекция может содержать разное количество элементов и разных строк для разных языков и региональных параметров. Примерами коллекций можно считать перечисления, которые указываются в типах утверждений. Например, содержимое раскрывающегося списка стран или регионов, отображаемое для пользователя. |
| LocalizedStrings | 0:n | Определяет все строки, за исключением включенных в коллекции, для разных языков и региональных параметров. |

### <a name="localizedcollections"></a>LocalizedCollections

Элемент **LocalizedCollections** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| LocalizedCollection | 1:n | Список поддерживаемых языков. |

#### <a name="localizedcollection"></a>LocalizedCollection

Элемент **LocalizedCollections** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ElementType | Да | Ссылается на элемент ClaimType или элемент пользовательского интерфейса в файле политики. |
| ElementId | Да | Строка, содержащая ссылку на тип утверждения, который уже определен в разделе ClaimsSchema и используется, если **ElementType** имеет значение ClaimType. |
| TargetCollection | Да | Целевая коллекция. |

Элемент **LocalizedCollections** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Item | 0:n | Определяет варианты, которые пользователь может выбрать для утверждения в пользовательском интерфейсе, например, значения в раскрывающемся списке. |

Элемент **Item** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| текст | Да | Понятная для пользователя отображаемая строка, которая размещается в пользовательском интерфейсе для этого параметра. |
| Значение | Да | Строковое значение утверждения, связанное с этим вариантом. |
| SelectByDefault | Нет | Указывает, следует ли по умолчанию выбирать этот параметр в пользовательском интерфейсе. Возможные значения: True или False. |

Следующий пример демонстрирует использование элемента **LocalizedCollections**. Он содержит два элемента **LocalizedCollection** по одному для английского и испанского языков. Оба они устанавливают для утверждения `Gender` коллекцию **Restriction** со списком элементов на английском и испанском языках.

```xml
<LocalizedResources Id="api.selfasserted.en">
 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Female" Value="F" />
      <Item Text="Male" Value="M" />
    </LocalizedCollection>
</LocalizedCollections>

<LocalizedResources Id="api.selfasserted.es">
 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Femenino" Value="F" />
      <Item Text="Masculino" Value="M" />
    </LocalizedCollection>
</LocalizedCollections>
```

### <a name="localizedstrings"></a>LocalizedStrings

Элемент **LocalizedStrings** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| LocalizedString | 1:n | Локализованная строка. |

Элемент **LocalizedStrings** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ElementType | Да | Возможны следующие значения: [поставщика утверждений](#claimsprovider) [, с](#claimtype), назначением, [ErrorMessage](#errormessage), [жетлокализедстрингстрансформатионклаимтипе](#getlocalizedstringstransformationclaimtype), [predicate](#predicate), [инпутвалидатион](#inputvalidation)или [укселемент](#uxelement).   | 
| ElementId | Да | Если для **ElementType** задано значение `ClaimType` , `Predicate` или `InputValidation` , этот элемент содержит ссылку на тип утверждения, уже определенный в разделе ClaimsSchema. |
| StringId | Да | Если **ElementType** имеет значение `ClaimType`, этот элемент содержит ссылку на атрибут типа утверждения. Возможные значения: `DisplayName`, `AdminHelpText` или `PatternHelpText`. Значение `DisplayName` задает отображаемое имя утверждения. Значение `AdminHelpText` задает имя текста справки для пользователя утверждения. Значение `PatternHelpText` задает текста справки по шаблону утверждения. Если **ElementType** имеет значение `UxElement`, этот элемент содержит ссылку на атрибут элемента пользовательского интерфейса. Если **ElementType** имеет значение `ErrorMessage`, этот элемент задает идентификатор сообщения об ошибке. Полный список идентификаторов `UxElement` для строк локализации можно найти на [этой странице](localization-string-ids.md).|

## <a name="elementtype"></a>ElementType

Ссылка ElementType на тип утверждения, преобразование утверждения или элемент пользовательского интерфейса в политике для локализации.

| Элемент для локализации | ElementType | ElementId |StringId |
| --------- | -------- | ----------- |----------- |
| Имя поставщика удостоверений |`ClaimsProvider`| | Идентификатор элемента ClaimsExchange|
| Атрибуты типа утверждения|`ClaimType`|Имя типа утверждения| Атрибут для локализованного утверждения. Возможные значения: `AdminHelpText` , `DisplayName` , `PatternHelpText` и `UserHelpText` .|
|Сообщение об ошибке|`ErrorMessage`||Идентификатор сообщения об ошибке |
|Копирует локализованные строки в утверждения|`GetLocalizedStringsTra nsformationClaimType`||Имя выходного утверждения|
|Сообщение пользователя предиката|`Predicate`|Имя предиката| Атрибут для локализованного предиката. Возможные значения: `HelpText` .|
|Сообщение пользователя группы предикатов|`InputValidation`|Идентификатор элемента Предикатевалидатион.|Идентификатор элемента Предикатеграуп. Группа предикатов должна быть дочерним элементом элемента проверки предиката, как определено в ElementId.|
|Элементы пользовательского интерфейса |`UxElement` | | Идентификатор элемента пользовательского интерфейса для локализации.|
|[Элемент управления отображением](display-controls.md) |`DisplayControl` |Идентификатор элемента управления отображением. | Идентификатор элемента пользовательского интерфейса для локализации.|

## <a name="examples"></a>Примеры

### <a name="claimsprovider"></a>ClaimsProvider

Значение поставщика утверждений используется для локализации одного из отображаемых имен поставщиков утверждений. 

```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="LinkedInExchange" TechnicalProfileReferenceId="LinkedIn-OAUTH" />
  </ClaimsExchanges>
</OrchestrationStep>

```

В следующем примере показано, как локализовать отображаемое имя поставщиков утверждений.

```xml
<LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">Facebook</LocalizedString>
<LocalizedString ElementType="ClaimsProvider" StringId="GoogleExchange">Google</LocalizedString>
<LocalizedString ElementType="ClaimsProvider" StringId="LinkedInExchange">LinkedIn</LocalizedString>
```

### <a name="claimtype"></a>ClaimType

Значение параметра с + + используется для локализации одного из атрибутов утверждения. 

```xml
<ClaimType Id="email">
  <DisplayName>Email Address</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Email address that can be used to contact you.</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```

В следующем примере показано, как локализовать атрибуты DisplayName, Усерхелптекст и Паттернхелптекст типа утверждения сообщения электронной почты.

```xml
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email</LocalizedString>
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Please enter your email</LocalizedString>
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address</LocalizedString>
```

### <a name="errormessage"></a>ErrorMessage

Значение ErrorMessage используется для локализации одного из системных сообщений об ошибках. 

```xml
<TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
  <Metadata>
    <Item Key="RaiseErrorIfClaimsPrincipalAlreadyExists">true</Item>
    <Item Key="UserMessageIfClaimsPrincipalAlreadyExists">You are already registered, please press the back button and sign in instead.</Item>
  </Metadata>
  ...
</TechnicalProfile>
```

В следующем примере показано, как локализовать сообщение об ошибке УсермессажеифклаимспринЦипалалреадексистс.


```xml
<LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">The account you are trying to create already exists, please sign-in.</LocalizedString>
```

### <a name="getlocalizedstringstransformationclaimtype"></a>жетлокализедстрингстрансформатионклаимтипе

Значение Жетлокализедстрингстрансформатионклаимтипе используется для копирования локализованных строк в утверждения. Дополнительные сведения см. в разделе [Преобразование утверждений жетлокализедстрингстрансформатион](string-transformations.md#getlocalizedstringstransformation) .


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

В следующем примере показано, как локализовать утверждения вывода преобразования Жетлокализедстрингстрансформатион Claims.

```xml
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for verifying your account!</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
```

### <a name="predicate"></a>Predicate

Значение предиката используется для локализации одного из сообщений об ошибках [предиката](predicates.md) . 

```xml
<Predicates>
  <Predicate Id="LengthRange" Method="IsLengthRange"  HelpText="The password must be between 6 and 64 characters.">
    <Parameters>
      <Parameter Id="Minimum">6</Parameter>
      <Parameter Id="Maximum">64</Parameter>
    </Parameters>
  </Predicate>
  <Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">a-z</Parameter>
    </Parameters>
  </Predicate>
  <Predicate Id="Uppercase" Method="IncludesCharacters" HelpText="an uppercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">A-Z</Parameter>
    </Parameters>
  </Predicate>
</Predicates>
```

В следующем примере показано, как локализовать текст справки предикатов.

```xml
<LocalizedString ElementType="Predicate" ElementId="LengthRange" StringId="HelpText">The password must be between 6 and 64 characters.</LocalizedString>
<LocalizedString ElementType="Predicate" ElementId="Lowercase" StringId="HelpText">a lowercase letter</LocalizedString>
<LocalizedString ElementType="Predicate" ElementId="Uppercase" StringId="HelpText">an uppercase letter</LocalizedString>
```

### <a name="inputvalidation"></a>инпутвалидатион

Значение Инпутвалидатион используется для локализации одного из сообщений об ошибках группы [предикатевалидатион](predicates.md) . 

```xml
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
```

В следующем примере показано, как локализовать текст справки по группе проверки предиката.

```xml
<LocalizedString ElementType="InputValidation" ElementId="CustomPassword" StringId="CharacterClasses">The password must have at least 3 of the following:</LocalizedString>
```

### <a name="uxelement"></a>укселемент

Значение Укселемент используется для локализации одного из элементов пользовательского интерфейса. В следующем примере показано, как локализовать кнопки продолжить и Отмена.

```xml
<LocalizedString ElementType="UxElement" StringId="button_continue">Create new account</LocalizedString>
<LocalizedString ElementType="UxElement" StringId="button_cancel">Cancel</LocalizedString>
```

### <a name="displaycontrol"></a>DisplayControl

Значение [элемента управления](display-controls.md) типа (DisplayControl) используется для локализации одного из элементов пользовательского интерфейса дисплея. Если этот параметр включен, элемент управления отображением Локализедстрингс принимает ***приоритет** _ для некоторых из _ *укселемент** StringIDs, таких как **ver_but_send**, **ver_but_edit**, **ver_but_resend** и **ver_but_verify**. В следующем примере показано, как локализовать кнопки "Отправить" и "проверить". 

```xml
<LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_send_code">Send verification code</LocalizedString>
<LocalizedString ElementType="DisplayControl" ElementId="emailVerificationControl" StringId="but_verify_code">Verify code</LocalizedString>
```

В разделе метаданных самоподтвержденного технического профиля Контентдефинитион должен иметь значение DataUri, установленное в [Макет страницы](page-layout.md) 2.1.0 или более поздней версии. Например:

```xml
<ContentDefinition Id="api.selfasserted">
  <DataUri>urn:com:microsoft:aad:b2c:elements:selfasserted:2.1.0</DataUri>
  ...
```

## <a name="next-steps"></a>Дальнейшие действия

Примеры локализации см. в следующих статьях:

- [Настройка языка с помощью настраиваемой политики в Azure Active Directory B2C](language-customization.md)
- [Настройка языка с помощью потоков пользователя в Azure Active Directory B2C](language-customization.md)
