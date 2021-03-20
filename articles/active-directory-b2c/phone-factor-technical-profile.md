---
title: Определение технического профиля телефонного фактора в настраиваемой политике
titleSuffix: Azure AD B2C
description: Определите технический профиль телефонного фактора в пользовательской политике в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 10/12/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 322e4b78fbfb38f1822fb7a7cdcdbfcc0738b303
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91950403"
---
# <a name="define-a-phone-factor-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Определение технического профиля телефонного фактора в Azure Active Directory B2C настраиваемой политике

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) обеспечивает поддержку регистрации и проверки номеров телефонов. Этот технический профиль:

- Предоставляет пользовательский интерфейс для взаимодействия с пользователем для проверки или регистрации номера телефона.
- Поддерживает телефонные звонки и текстовые сообщения для проверки номера телефона.
- Поддерживает несколько телефонных номеров. Пользователь может выбрать один из номеров телефона для проверки.  
- Возвращает утверждение, указывающее, предоставил ли пользователь новый номер телефона. Это утверждение можно использовать, чтобы решить, следует ли сохранять номер телефона в Azure AD B2C профиле пользователя.  
- Использует [Определение содержимого](contentdefinitions.md) для управления внешним видом и поведением.

## <a name="protocol"></a>Протокол

Атрибуту **Name** элемента **Protocol** необходимо присвоить значение `Proprietary`. Атрибут **handler** должен содержать полное имя сборки обработчика протокола, используемое Azure AD B2C для параметра "телефонный фактор": `Web.TPEngine.Providers.PhoneFactorProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null`

В следующем примере показан технический профиль телефонного фактора для регистрации и проверки:

```xml
<TechnicalProfile Id="PhoneFactor-InputOrVerify">
  <DisplayName>PhoneFactor</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.PhoneFactorProtocolProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
</TechnicalProfile>
```

## <a name="input-claims-transformations"></a>Преобразования входящих утверждений

Элемент Инпутклаимстрансформатионс может содержать коллекцию преобразований входящих утверждений, которые используются для изменения входных утверждений или создания новых. Следующее преобразование «входные утверждения» создает `UserId` утверждение, которое будет использоваться позже в коллекции входных утверждений.

```xml
<InputClaimsTransformations>
  <InputClaimsTransformation ReferenceId="CreateUserIdForMFA" />
</InputClaimsTransformations>
```

## <a name="input-claims"></a>Входящие утверждения

Элемент Inputclaim должен содержать следующие утверждения. Вы также можете сопоставлять имя заявки с именем, определенным в техническом профиле телефонного фактора. 

|  Тип данных| Обязательно | Описание |
| --------- | -------- | ----------- | 
| строка| Да | Уникальный идентификатор пользователя. Имя утверждения или Партнерклаимтипе должно иметь значение `UserId` . Это утверждение не должно содержать личных сведений.|
| строка| Да | Список типов утверждений. Каждое утверждение содержит один номер телефона. Если какой-либо из входящих утверждений не содержит номер телефона, пользователю будет предложено зарегистрироваться и проверить новый номер телефона. Проверенный номер телефона возвращается в виде исходящего утверждения. Если один из входных утверждений содержит номер телефона, пользователю предлагается проверить его. Если несколько входных утверждений содержат номер телефона, пользователю предлагается выбрать и проверить один из телефонных номеров. |

В следующем примере демонстрируется использование нескольких телефонных номеров. Дополнительные сведения см. в разделе [Пример политики](https://github.com/azure-ad-b2c/samples/tree/master/policies/mfa-add-secondarymfa).

```xml
<InputClaims>
  <InputClaim ClaimTypeReferenceId="userIdForMFA" PartnerClaimType="UserId" />
  <InputClaim ClaimTypeReferenceId="strongAuthenticationPhoneNumber" />
  <InputClaim ClaimTypeReferenceId="secondaryStrongAuthenticationPhoneNumber" />
</InputClaims>
```

## <a name="output-claims"></a>Исходящие утверждения

Элемент OutputClaims содержит список заявок, возвращенных техническим профилем телефонного фактора.

|  Тип данных| Обязательно | Описание |
|  -------- | ----------- |----------- |
| Логическое | Да | Указывает, введен ли новый номер телефона пользователем. Имя утверждения или Партнерклаимтипе должно иметь значение `newPhoneNumberEntered`|
| строка| Да | Проверенный номер телефона. Имя утверждения или Партнерклаимтипе должно иметь значение `Verified.OfficePhone` .|

Элемент Аутпутклаимстрансформатионс может содержать коллекцию элементов Аутпутклаимстрансформатион, которые используются для изменения исходящих заявок или для создания новых.

## <a name="cryptographic-keys"></a>Криптографические ключи

Элемент **CryptographicKeys** не используется.


## <a name="metadata"></a>Метаданные

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ContentDefinitionReferenceId | Да | Идентификатор [определения содержимого](contentdefinitions.md), связанного с этим техническим профилем. |
| мануалфоненумберентрялловед| Нет | Укажите, разрешено ли пользователю вручную вводить номер телефона. Возможные значения: `true` или `false` (по умолчанию).|
| задание. authenticationMode | Нет | Метод для проверки номера телефона. Возможные значения: `sms` , `phone` или `mixed` (по умолчанию).|
| Настройка. автонабор номера| Нет| Укажите, должен ли технический профиль выполнять автоматический автонабор или автоматическую отправку SMS. Возможные значения: `true` или `false` (по умолчанию). Для автоматического набора необходимо `setting.authenticationMode` , чтобы метаданные были установлены в `sms` , или `phone` . Входная коллекция утверждений должна иметь один номер телефона. |

### <a name="ui-elements"></a>Элементы пользовательского интерфейса

Элементы пользовательского интерфейса страницы проверки подлинности на странице "телефонный фактор" могут быть [локализованы](localization-string-ids.md#phone-factor-authentication-page-user-interface-elements).

## <a name="next-steps"></a>Дальнейшие действия

- Проверьте [социальные и локальные учетные записи с помощью пакета MFA](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccountsWithMfa) Starter Pack.
