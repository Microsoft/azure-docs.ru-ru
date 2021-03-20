---
title: Обмен утверждениями REST API в пользовательской политике B2C
titleSuffix: Azure AD B2C
description: Основные сведения о создании пути взаимодействия пользователя Azure AD B2C, который взаимодействует со службами RESTful.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/18/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: bc2b72779460c2b7e3999204ace50ca57388b9a2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89594192"
---
# <a name="integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-custom-policy"></a>Интеграция обмена утверждениями REST API в пользовательскую политику Azure AD B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Инфраструктуру Identity Experience Framework, лежащая в основе Azure Active Directory B2C (Azure AD B2C), можно интегрировать с RESTful API в пути взаимодействия пользователя. В этой статье показано, как создать путь взаимодействия пользователя, взаимодействующий со службой RESTful, с помощью [технического профиля RESTful](restful-technical-profile.md).

Azure AD B2C позволяет добавлять собственную бизнес-логику в пути взаимодействия пользователя с помощью вызова службы RESTful. Identity Experience Framework позволяет отправлять и получать данные от службы RESTful для обмена утверждениями. Например, администратор может сделать следующее:

- **Проверять входные данные пользователя.** Например, вы можете проверить, существует ли адрес электронной почты, предоставленный пользователем, в базе данных клиента, и, если нет, вывести ошибку.
- **Обрабатывать утверждения**. Если пользователь вводит имя только строчными или только заглавными буквами, REST API может изменить его формат, оставив прописной только первую букву, и вернуть его в Azure AD B2C.
- **Дополнять данные пользователя за счет дальнейшей интеграции с корпоративным бизнес-приложением**. Например, служба RESTful может получить адрес электронной почты пользователя, обратиться к базе данных клиента и вернуть в Azure AD B2C номер личного счета пользователя. Возвратные утверждения могут храниться в учетной записи Azure AD пользователя, проверяться в следующих действиях оркестрации или быть включены в маркер доступа.
- **Запускать пользовательскую бизнес-логику**. Вы можете отправлять push-уведомления, обновлять корпоративные базы данных, запускать процесс переноса пользователей, управлять разрешениями, производить аудит баз данных и выполнять любые другие рабочие процессы.

![Схема обмена утверждениями службы RESTful](media/custom-policy-rest-api-intro/restful-service-claims-exchange.png)

> [!NOTE]
> Если в Azure AD B2C службе RESTFUL возникла задержка или нет ответа, время ожидания составляет 30 секунд, а число повторов — 2 раза (это означает, что всего три попытки). Параметры времени ожидания и числа повторных попыток сейчас не настраиваются.

## <a name="calling-a-restful-service"></a>Вызов службы RESTful

Взаимодействие включает в себя обмен утверждениями между REST API и Azure AD B2C. Интеграцию со службами RESTful можно спроектировать одним из следующих способов:

- **Технический профиль проверки**. Вызов службы RESTful происходит в [техническом профиле проверки](validation-technical-profile.md) указанного [самоподтвержденного технического профиля](self-asserted-technical-profile.md) или в [элементе управления отображением проверки](display-control-verification.md) для [элемента управления отображением](display-controls.md). Этот профиль проверяет данные, предоставленные пользователем, в начале пути взаимодействия пользователя. С помощью технического профиля проверки вы можете:

  - отправлять утверждения в REST API;
  - проверять утверждения и инициировать пользовательские сообщения об ошибках;
  - отправлять утверждения из REST API на последующие шаги оркестрации.

- **Обмен утверждениями**. Прямой обмен утверждениями можно настроить, вызвав технический профиль REST API непосредственно из этапа оркестрации [пути взаимодействия пользователя](userjourneys.md). Может только:

  - отправлять утверждения в REST API;
  - проверять утверждения и инициировать пользовательские сообщения об ошибках, которые возвращаются в приложение;
  - отправлять утверждения из REST API на последующие шаги оркестрации.

Вы можете добавить вызов REST API в любой шаг пути взаимодействия пользователя, который определен пользовательской политикой. Например, вызовы к REST API можно выполнять:

- во время входа в систему перед тем, как Azure AD B2C проверит учетные данные;
- сразу после входа в систему;
- перед тем как Azure AD B2C создаст учетную запись в каталоге;
- после того как Azure AD B2C создаст учетную запись в каталоге;
- перед тем как Azure AD B2C выдаст маркер доступа.

![Коллекция технических профилей проверки](media/custom-policy-rest-api-intro/validation-technical-profile.png)

## <a name="sending-data"></a>Отправка данных

В [техническом профиле RESTful](restful-technical-profile.md) элемент `InputClaims` содержит список утверждений для отправки в службу RESTful. Вы можете связать имя утверждения с именем, определенным в службе RESTful, задать значение по умолчанию и использовать [сопоставители утверждений](claim-resolver-overview.md).

Настроить отправку входных утверждений в поставщик утверждений RESTful можно с помощью атрибута SendClaimsIn. Вы можете выбрать

- **Body** — отправляется в тексте HTTP-запроса POST в формате JSON.
- **Form** — отправляется в тексте HTTP-запроса POST в формате пар "ключ-значение", разделенных символом амперсанда (&).
- **Header** — отправляется в заголовке HTTP-запроса GET.
- **QueryString** — отправляется в строке HTTP-запроса GET.

Если параметр **Body** настроен, технический профиль REST API позволяет отправлять в конечную точку сложные полезные данные JSON. Дополнительные сведения см. в разделе [Отправка полезных данных JSON](restful-technical-profile.md#send-a-json-payload).

## <a name="receiving-data"></a>Получение данных

Элемент `OutputClaims` [технического профиля RESTful](restful-technical-profile.md) содержит список утверждений, возвращаемых REST API. Может потребоваться сопоставить имя утверждения, определенное в вашей политике, с именем, определенным в REST API. Вы также можете добавить утверждения, которые не возвращаются поставщиком удостоверений REST API, установив атрибут DefaultValue.

Выходные утверждения, анализируемые поставщиком утверждений RESTful, всегда должны анализироваться как плоский текст JSON в ответе, например:

```json
{
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "loyaltyNumber":  1234
}
```

Выходные утверждения должны выглядеть следующим образом:

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" />
  <OutputClaim ClaimTypeReferenceId="loyaltyNumber" />
</OutputClaims>
```

Чтобы проанализировать вложенный ответный текст JSON, установите для метаданных ResolveJsonPathsInJsonTokens значение true. В выходном утверждении задайте для PartnerClaimType элемент пути JSON, который нужно вывести.

```json
"contacts": [
  {
    "id": "MAINCONTACT_1",
    "person": {
      "name": "Emily Smith",
      "loyaltyNumber":  1234,
      "emails": [
        {
          "id": "EMAIL_1",
          "type": "WORK",
          "email": "email@domain.com"
        }
      ]
    }
  }
],
```


Выходные утверждения должны выглядеть следующим образом:

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="contacts[0].person.name" />
  <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="contacts[0].person.emails[0].email" />
  <OutputClaim ClaimTypeReferenceId="loyaltyNumber" PartnerClaimType="contacts[0].person.loyaltyNumber" />
</OutputClaims>
```

## <a name="security-considerations"></a>Вопросы безопасности

Необходимо защитить конечную точку REST API, чтобы с ней могли взаимодействовать только клиенты, прошедшие проверку подлинности. REST API должен использовать конечную точку HTTPS. Задайте для метаданных AuthenticationType один из следующих методов проверки подлинности:

- **Сертификат клиента** ограничивает доступ с помощью проверки подлинности на основе сертификата клиента. Доступ к API могут получить только службы, у которых есть соответствующие сертификаты. Сертификат клиента сохраняется в ключе политики Azure AD B2C. Узнайте больше о том, как [защитить службу RESTful с помощью сертификатов клиента](secure-rest-api.md#https-client-certificate-authentication).
- **Обычная** обеспечивает безопасность REST API с помощью обычной проверки подлинности HTTP. Доступ к API могут получить только проверенные пользователи, включая Azure AD B2C. Имя пользователя и пароль хранятся в ключах политики Azure AD B2C. Узнайте, как [защитить службы RESTful с использованием обычной проверки подлинности HTTP](secure-rest-api.md#http-basic-authentication).
- **Маркер носителя** ограничивает доступ с помощью маркера доступа клиента OAuth2. Маркер доступа хранится в ключе политики Azure AD B2C. Узнайте больше о том, как [защитить службу RESTful с помощью маркера носителя](secure-rest-api.md#oauth2-bearer-authentication).

## <a name="rest-api-platform"></a>Платформа REST API
Интерфейс REST API может быть основан на любой платформе и написан на любом языке программирования при условии, что он безопасен и может отправлять и получать утверждения, как указано в [техническом профиле RESTful](restful-technical-profile.md).

## <a name="localize-the-rest-api"></a>Локализация REST API
В техническом профиле RESTful может требоваться отправлять языки и языковой стандарт текущего сеанса, а также при необходимости выдавать локализованное сообщение об ошибке. С помощью [сопоставителя утверждений](claim-resolver-overview.md) можно отправить контекстное утверждение, например язык пользователя. В следующем примере показан технический профиль RESTful, демонстрирующий этот сценарий.

```xml
<TechnicalProfile Id="REST-ValidateUserData">
  <DisplayName>Validate user input data</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app.azurewebsites.net/api/identity</Item>
    <Item Key="AuthenticationType">None</Item>
    <Item Key="SendClaimsIn">Body</Item>
    <Item Key="IncludeClaimResolvingInClaimsHandling">true</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="userLanguage" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
    <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="emailAddress" />
  </InputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

## <a name="handling-error-messages"></a>Обработка сообщений об ошибке

REST API может возвращать сообщения об ошибках, например The user was not found in the CRM system (Пользователь не найден в системе CRM). Если происходит ошибка, REST API возвращает сообщение об ошибке HTTP 409 (код ответа "Конфликт"). Дополнительные сведения см. в [описании технического профиля RESTful](restful-technical-profile.md#returning-validation-error-message).

Это можно сделать только путем вызова технического профиля REST API из технического профиля проверки. Это позволяет пользователю исправить данные на странице и повторно выполнить проверку после отправки страницы.

Ответ HTTP 409 необходим для предотвращения обработки всех последующих технических профилей проверки на данном этапе оркестрации.

Если вы ссылаетесь на технический профиль REST API непосредственно из пути взаимодействия пользователя, пользователь перенаправляется обратно в приложение проверяющей стороны с соответствующим сообщением об ошибке.

## <a name="publishing-your-rest-api"></a>Публикация REST API

Запрос к службе REST API поступает с серверов Azure AD B2C. Служба REST API должна быть опубликована в общедоступной конечной точке HTTPS. Вызовы REST API будут поступать с IP-адреса центра обработки данных Azure.

Спроектируйте службу REST API и ее базовые компоненты (например, базу данных и файловую систему) так, чтобы обеспечить высокую доступность.

## <a name="next-steps"></a>Дальнейшие действия

Примеры использования технического профиля RESTful см. в следующих статьях:

- [Пошаговое руководство. Интеграция обмена утверждениями REST API в путях взаимодействия пользователей Azure AD B2C как проверка входных данных](custom-policy-rest-api-claims-validation.md)
- [Пошаговое руководство. Добавление возможности обмена утверждениями REST API в настраиваемые политики в Azure Active Directory B2C](custom-policy-rest-api-claims-validation.md)
- [Защита служб REST API](secure-rest-api.md)
- [Общие сведения Технический профиль RESTful](restful-technical-profile.md)
