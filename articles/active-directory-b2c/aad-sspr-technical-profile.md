---
title: Технические профили Azure AD SSPR в пользовательских политиках
titleSuffix: Azure AD B2C
description: Справочник по настраиваемой политике для технических профилей Azure AD SSPR в Azure AD B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 06/23/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 3e6fcf956639d827a8654c5ee80e7cab8cadf930
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85383603"
---
# <a name="define-an-azure-ad-sspr-technical-profile-in-an-azure-ad-b2c-custom-policy"></a>Определение технического профиля Azure AD SSPR в Azure AD B2C настраиваемой политике

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) обеспечивает поддержку проверки адреса электронной почты для самостоятельного сброса пароля (SSPR). Используйте технический профиль Azure AD SSPR, чтобы создать и отправить код на адрес электронной почты, а затем проверить код. Технический профиль Azure AD SSPR также может возвращать сообщение об ошибке. Этот профиль проверяет данные, предоставленные пользователем, прежде чем продолжать путь взаимодействия пользователя. В техническом профиле проверки отображается сообщение об ошибке на странице с автоматическим подтверждением.

Этот технический профиль:

- Не предоставляет интерфейс для взаимодействия с пользователем. Вместо этого пользовательский интерфейс вызывается из [самостоятельно утвержденного](self-asserted-technical-profile.md) технического профиля или [элемента управления отображением](display-controls.md) в качестве [технического профиля проверки](validation-technical-profile.md).
- Использует службу Azure AD SSPR для создания и отправки кода на адрес электронной почты, а затем проверяет код.  
- Проверяет адрес электронной почты с помощью кода проверки.

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

## <a name="protocol"></a>Протокол

Атрибуту **Name** элемента **Protocol** необходимо присвоить значение `Proprietary`. Атрибут **handler** должен содержать полное имя сборки обработчика протокола, используемой Azure AD B2C:

```
Web.TPEngine.Providers.AadSsprProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
```

В следующем примере показан технический профиль Azure AD SSPR:

```XML
<TechnicalProfile Id="AadSspr-SendCode">
  <DisplayName>Send Code</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AadSsprProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    ...
```

## <a name="send-email"></a>Отправить электронное письмо

Первый режим этого технического профиля — создание кода и его отправка. Для этого режима можно настроить следующие параметры.

### <a name="input-claims"></a>Входящие утверждения

Элемент **inputclaim** содержит список утверждений для отправки в Azure AD SSPR. Вы также можете сопоставлять имя утверждения с именем, определенным в техническом профиле SSPR.

| клаимреференцеид | Обязательно | Описание |
| --------- | -------- | ----------- |
| emailAddress | Да | Идентификатор пользователя, владеющего адресом электронной почты. `PartnerClaimType`Свойству входного утверждения должно быть присвоено значение `emailAddress` . |


Элемент **инпутклаимстрансформатионс** может содержать коллекцию элементов **инпутклаимстрансформатион** , которые используются для изменения входных утверждений или создания новых перед отправкой в службу Azure AD SSPR.

### <a name="output-claims"></a>Исходящие утверждения

Поставщик протокола Azure AD SSPR не возвращает никаких **OutputClaims**, поэтому нет необходимости указывать выходные утверждения. Однако можно включить утверждения, которые не возвращаются поставщиком протокола Azure AD SSPR, при условии, что задан `DefaultValue` атрибут.

Элемент **OutputClaimsTransformations** может содержать коллекцию элементов **OutputClaimsTransformation**, которые используются для изменения исходящих утверждений или создания новых.

### <a name="metadata"></a>Метаданные

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| Операция | Да | Должен быть **SendCode**.  |

#### <a name="ui-elements"></a>Элементы пользовательского интерфейса

Следующие метаданные можно использовать для настройки сообщений об ошибках, отображаемых при отправке ошибки SMS. Метаданные должны быть настроены в [самостоятельно утвержденном](self-asserted-technical-profile.md) техническом профиле. Сообщения об ошибках можно [локализовать](localization-string-ids.md#azure-ad-sspr).

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
| усермессажеифинтерналеррор | Нет | Сообщение об ошибке пользователя, если на сервере произошла внутренняя ошибка. |
| UserMessageIfThrottled| Нет | Сообщение об ошибке пользователя, если запрос был отрегулирован.|


### <a name="example-send-an-email"></a>Пример: Отправка сообщения электронной почты

В следующем примере показан технический профиль Azure AD SSPR, который используется для отправки кода через электронную почту.

```XML
<TechnicalProfile Id="AadSspr-SendCode">
  <DisplayName>Send Code</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AadSsprProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="Operation">SendCode</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="emailAddress"/>
  </InputClaims>
</TechnicalProfile>
```

## <a name="verify-code"></a>Проверить код

Второй режим этого технического профиля — проверка кода. Для этого режима можно настроить следующие параметры.

### <a name="input-claims"></a>Входящие утверждения

Элемент **inputclaim** содержит список утверждений для отправки в Azure AD SSPR. Вы также можете сопоставлять имя утверждения с именем, определенным в техническом профиле SSPR.

| клаимреференцеид | Обязательно | Описание |
| --------- | -------- | ----------- | ----------- |
| emailAddress| Да | Тот же адрес электронной почты, который использовался ранее для отправки кода. Он также используется для нахождение сеанса проверки по электронной почте. `PartnerClaimType`Свойству входного утверждения должно быть присвоено значение `emailAddress` .|
| верификатионкоде  | Да | Код проверки, предоставленный пользователем для проверки. `PartnerClaimType`Свойству входного утверждения должно быть присвоено значение `verificationCode` . |

Элемент **инпутклаимстрансформатионс** может содержать коллекцию элементов **инпутклаимстрансформатион** , которые используются для изменения входных утверждений или создания новых перед ВЫЗОВОМ службы Azure AD SSPR.

### <a name="output-claims"></a>Исходящие утверждения

Поставщик протокола Azure AD SSPR не возвращает никаких **OutputClaims**, поэтому нет необходимости указывать выходные утверждения. Однако можно включить утверждения, которые не возвращаются поставщиком протокола Azure AD SSPR, при условии, что задан `DefaultValue` атрибут.

Элемент **OutputClaimsTransformations** может содержать коллекцию элементов **OutputClaimsTransformation**, которые используются для изменения исходящих утверждений или создания новых.

### <a name="metadata"></a>Метаданные

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| Операция | Да | Должно быть **VerifyCode** |

#### <a name="ui-elements"></a>Элементы пользовательского интерфейса

Следующие метаданные можно использовать для настройки сообщений об ошибках, отображаемых при сбое проверки кода. Метаданные должны быть настроены в [самостоятельно утвержденном](self-asserted-technical-profile.md) техническом профиле. Сообщения об ошибках можно [локализовать](localization-string-ids.md#azure-ad-sspr).

| Атрибут | Обязательно | Описание |
| --------- | -------- | ----------- |
|усермессажеифчалленжеекспиред | Сообщение, отображаемое пользователю, если истек срок действия сеанса проверки кода. Либо истек срок действия кода, либо код для данного идентификатора никогда не создавался.|
|усермессажеифинтерналеррор | Сообщение об ошибке пользователя, если на сервере произошла внутренняя ошибка.|
|UserMessageIfThrottled | Сообщение об ошибке пользователя, если запрос был отрегулирован.|
|усермессажеифверификатионфаиледноретри | Сообщение, отображаемое пользователю, если он предоставил недопустимый код, и пользователю не разрешено предоставлять правильный код.|
|UserMessageIfVerificationFailedRetryAllowed | Сообщение, отображаемое пользователю, если он предоставил недопустимый код, и пользователь может предоставить правильный код.|

### <a name="example-verify-a-code"></a>Пример: Проверка кода

В следующем примере показан технический профиль Azure AD SSPR, используемый для проверки кода.

```XML
<TechnicalProfile Id="AadSspr-VerifyCode">
  <DisplayName>Verify Code</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AadSsprProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="Operation">VerifyCode</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="verificationCode" PartnerClaimType="verificationCode" />
    <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="emailAddress"/>
  </InputClaims>
</TechnicalProfile>
```