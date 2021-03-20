---
title: Ссылка на элемент управления отображением
titleSuffix: Azure AD B2C
description: Справочник по элементам управления отображением Azure AD B2C. Используйте элементы управления отображением для настройки пути взаимодействия пользователя, определенного в пользовательских политиках.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/11/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 441a77823c77305e567e9e1436715bc51ca48c11
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97387060"
---
# <a name="display-controls"></a>Элементы управления отображением

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

**Элемент управления "Отображение** " — это элемент пользовательского интерфейса, который имеет специальные функции и взаимодействует с серверной службой Azure Active Directory B2C (Azure AD B2C). Она позволяет пользователю выполнять действия на странице, которая вызывает [технический профиль проверки](validation-technical-profile.md) на серверной части. Элементы управления отображением отображаются на странице, на которые ссылается [самостоятельно утвержденный технический профиль](self-asserted-technical-profile.md).

На следующем рисунке показана самоподтвержденная страница регистрации с двумя элементами управления отображением, проверяющими основной и дополнительный адреса электронной почты.

![Пример отображаемого элемента управления отображением](media/display-controls/display-control-email.png)

## <a name="prerequisites"></a>Предварительные условия

 В разделе [метаданных](self-asserted-technical-profile.md#metadata) [самостоятельно утвержденного технического профиля](self-asserted-technical-profile.md) [контентдефинитион](contentdefinitions.md) должен иметь `DataUri` значение контракта страницы Version 2.0.0 или выше. Пример:

```xml
<ContentDefinition Id="api.selfasserted">
  <LoadUri>~/tenant/default/selfAsserted.cshtml</LoadUri>
  <RecoveryUri>~/common/default_page_error.html</RecoveryUri>
  <DataUri>urn:com:microsoft:aad:b2c:elements:selfasserted:2.0.0</DataUri>
  ...
```

## <a name="defining-display-controls"></a>Определение элементов управления отображением

Элемент **DisplayControl** содержит следующие атрибуты:

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| Идентификатор | Да | Идентификатор, используемый для элемента управления отображением. На него можно [ссылаться](#referencing-display-controls). |
| усеринтерфацеконтролтипе | Да | Тип элемента управления отображением. В настоящее время поддерживается [верификатионконтрол](display-control-verification.md) |

Элемент **DisplayControl** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| InputClaims | 0:1 | **Inputclaim** используются для предварительного заполнения значений утверждений, собираемых от пользователя. Дополнительные сведения см. в разделе элемент [inputclaim](technicalprofiles.md#input-claims) . |
| дисплайклаимс | 0:1 | **Дисплайклаимс** используются для представления утверждений, собираемых от пользователя. Дополнительные сведения см. в разделе элемент [дисплайклаим](technicalprofiles.md#displayclaim) .|
| OutputClaims | 0:1 | **OutputClaims** используются для представления утверждений, которые должны быть временно сохранены для этого элемента **DisplayControl**. Дополнительные сведения см. в разделе элемент [OutputClaims](technicalprofiles.md#output-claims) .|
| Действия | 0:1 | **Действия** используются для перечисления технических профилей проверки для вызова действий пользователя, происходящих во внешнем интерфейсе. |

### <a name="input-claims"></a>Входящие утверждения

В элементе управления отображением можно использовать элементы **inputclaim** для предварительного заполнения значения заявок, которые будут собраны от пользователя на странице. Любой **инпутклаимстрансформатионс** может быть определен в самостоятельно утвержденном техническом профиле, который ссылается на этот элемент управления отображением.

В следующем примере предварительно заполняется адрес электронной почты, который должен быть проверен с помощью уже существующего адреса.

```xml
<DisplayControl Id="emailControl" UserInterfaceControlType="VerificationControl">
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="emailAddress" />
  </InputClaims>
  ...
```

### <a name="display-claims"></a>Отображение утверждений

Для каждого типа элементов управления отображением требуется другой набор утверждений, утверждений [вывода](#output-claims)и [действий](#display-control-actions) , которые должны быть выполнены.

Как и в случае с **утверждениями** , определенными в [самостоятельно утвержденном техническом профиле](self-asserted-technical-profile.md#display-claims), заявки на отображение представляют утверждения, которые должны быть собраны от пользователя в элементе управления отображением. Элементу, на который указывает **ссылка, необходимо** указать элемент **усеринпуттипе** для пользовательского типа ввода, поддерживаемого Azure AD B2C, например `TextBox` или `DropdownSingleSelect` . Если для **действия** требуется значение утверждения, задайте для атрибута **обязательный** атрибут `true` значение, чтобы пользователь предоставил для этого конкретного утверждения.

Для определенных типов элементов управления отображением требуются определенные утверждения на отображение. Например, **верификатионкоде** требуется для элемента управления отображением типа **верификатионконтрол**. Используйте атрибут **контролклаимтипе** , чтобы указать, какой дисплайклаим назначен для этого обязательного утверждения. Пример:

```xml
<DisplayClaim ClaimTypeReferenceId="otpCode" ControlClaimType="VerificationCode" Required="true" />
```

### <a name="output-claims"></a>Исходящие утверждения

**Выходные утверждения** элемента управления отображением не отправляются на следующий этап оркестрации. Они сохраняются временно только для текущего сеанса управления отображением. Эти временные утверждения могут совместно использоваться разными действиями одного элемента управления отображением.

Чтобы перефильтровать выходные данные на следующий этап оркестрации, используйте **OutputClaims** фактического самопроверочного технического профиля, который ссылается на этот элемент управления отображением.

### <a name="display-control-actions"></a>Отображение действий элемента управления

**Действия** элемента управления отображением являются процедурами, происходящими в Azure AD B2C серверной части, когда пользователь выполняет определенное действие на стороне клиента (браузер). Например, проверки, выполняемые, когда пользователь выбирает кнопку на странице.

Действие определяет список **технических профилей проверки**. Они используются для проверки некоторых или всех утверждений, отображаемых элементом управления отображением. Технический профиль проверки проверяет введенные пользователем данные и может вернуть пользователю сообщение об ошибке. Можно использовать **ContinueOnError**, **континуеонсукцесс** и **предусловия** в действии элемента управления отображением аналогично тому, как они используются в [технических профилях проверки](validation-technical-profile.md) в самостоятельно утвержденном техническом профиле.

#### <a name="actions"></a>Действия

Элемент **Actions** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Действие | 1:n | Список действий для выполнения. |

#### <a name="action"></a>Действие

Элемент **Action** содержит следующий атрибут:

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| Идентификатор | Да | Тип операции. Возможные значения: `SendCode` или `VerifyCode`. `SendCode`Значение отправляет код пользователю. Это действие может содержать два технических профиля проверки: один для создания кода и один для его отправки. `VerifyCode`Значение проверяет код, введенный пользователем в текстовое поле ввода. |

Элемент **Action** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| валидатионклаимсексчанже | 1:1 | Идентификаторы технических профилей, которые используются для проверки некоторых или всех утверждений, ссылающихся на технический профиль. Все входящие утверждения для технического профиля, на который указывает ссылка, должны отображаться в заявках на отображение, ссылающихся на технический профиль. |

#### <a name="validationclaimsexchange"></a>валидатионклаимсексчанже

Элемент **валидатионклаимсексчанже** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| ValidationTechnicalProfile | 1:n | Технический профиль, который будет использоваться для проверки некоторых или всех утверждений, ссылающихся на технический профиль. |

Элемент **валидатионтечникалпрофиле** содержит следующие атрибуты:

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| ReferenceId | Да | Идентификатор технического профиля, который уже определен в политике или родительской политике. |
|ContinueOnError|Нет| Указывает, следует ли продолжать проверку всех последующих технических профилей проверки, если этот технический профиль проверки порождает ошибку. Возможные значения: `true` или `false` (по умолчанию обработка дальнейших профилей проверки будет прервана, и будет возвращена ошибка). |
|ContinueOnSuccess | Нет | Указывает, следует ли продолжать проверку всех последующих профилей проверки, если этот технический профиль прошел проверку. Возможные значения: `true` или `false`. По умолчанию используется значение `true`, это значит, что обработка профилей проверки будет продолжена. |

Элемент **ValidationTechnicalProfile** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Preconditions | 0:1 | Список предварительных условий, которые необходимо соблюдать для выполнения технического профиля проверки. |

Элемент **предусловия** содержит следующие атрибуты:

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| `Type` | Да | Тип выполняемой проверки или запроса для данного предварительного условия. Возможные значения: `ClaimsExist` или `ClaimEquals`. `ClaimsExist` Указывает, что действия должны выполняться, если указанные утверждения существуют в текущем наборе утверждений пользователя. `ClaimEquals` Указывает, что действия должны выполняться, если существует указанное утверждение и его значение равно указанному значению. |
| `ExecuteActionsIf` | Да | Указывает, следует ли выполнять действия в предварительном условии, если тест возвращает true или false. |

Элементы **Precondition** содержат следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Значение | 1:n | Данные, используемые при проверке. Если тип этой проверки равен `ClaimsExist`, то это поле задает ClaimTypeReferenceId для запроса. Если тип проверки равен `ClaimEquals`, то это поле задает ClaimTypeReferenceId для запроса. Укажите значение для проверки в другом элементе value.|
| Действие | 1:1 | Действие, которое требуется выполнить, если в результате проверки предварительного условия в рамках шага оркестрации получено значение true. Значение **действия** равно `SkipThisValidationTechnicalProfile` , что указывает, что соответствующий технический профиль проверки не должен выполняться. |

В следующем примере адрес электронной почты отправляется и проверяется с помощью [технического профиля Azure AD SSPR](aad-sspr-technical-profile.md).

```xml
<DisplayControl Id="emailVerificationControl" UserInterfaceControlType="VerificationControl">
  <InputClaims></InputClaims>
  <DisplayClaims>
    <DisplayClaim ClaimTypeReferenceId="email" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="verificationCode" ControlClaimType="VerificationCode" Required="true" />
  </DisplayClaims>
  <OutputClaims></OutputClaims>
  <Actions>
    <Action Id="SendCode">
      <ValidationClaimsExchange>
        <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="AadSspr-SendCode" />
      </ValidationClaimsExchange>
    </Action>
    <Action Id="VerifyCode">
      <ValidationClaimsExchange>
        <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="AadSspr-VerifyCode" />
      </ValidationClaimsExchange>
    </Action>
  </Actions>
</DisplayControl>
```

В следующем примере код отправляется в сообщении электронной почты или в SMS на основе выбранного пользователем утверждения **мфатипе** с предусловиями.

```xml
<Action Id="SendCode">
  <ValidationClaimsExchange>
    <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="AzureMfa-SendSms">
      <Preconditions>
        <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
          <Value>mfaType</Value>
          <Value>email</Value>
          <Action>SkipThisValidationTechnicalProfile</Action>
        </Precondition>
      </Preconditions>
    </ValidationClaimsExchangeTechnicalProfile>
    <ValidationClaimsExchangeTechnicalProfile TechnicalProfileReferenceId="AadSspr-SendEmail">
      <Preconditions>
        <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
          <Value>mfaType</Value>
          <Value>phone</Value>
          <Action>SkipThisValidationTechnicalProfile</Action>
        </Precondition>
      </Preconditions>
    </ValidationClaimsExchangeTechnicalProfile>
  </ValidationClaimsExchange>
</Action>
```

## <a name="referencing-display-controls"></a>Ссылки на элементы управления отображением

Ссылки на элементы управления отображением отображаются в [утверждениях](self-asserted-technical-profile.md#display-claims) о [самостоятельно утвержденном техническом профиле](self-asserted-technical-profile.md).

Пример:

```xml
<TechnicalProfile Id="SelfAsserted-ProfileUpdate">
  ...
  <DisplayClaims>
    <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
    <DisplayClaim DisplayControlReferenceId="PhoneVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
```

## <a name="next-steps"></a>Дальнейшие действия

Примеры использования элемента управления отображением см. в следующих статьях: 

- [Пользовательская проверка электронной почты с помощью Маилжет](custom-email-mailjet.md)
- [Пользовательская проверка электронной почты с помощью SendGrid](custom-email-sendgrid.md)
