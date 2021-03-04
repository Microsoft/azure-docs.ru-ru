---
title: TechnicalProfiles
titleSuffix: Azure AD B2C
description: Использование элемента UserJourneys в настраиваемой политике для Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/04/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: a70b8be6c034f002f0e387b5d3b4ba5deafa1a5e
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102120510"
---
# <a name="technicalprofiles"></a>TechnicalProfiles

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Технический профиль предоставляет платформу со встроенным механизмом взаимодействия с разными типами сторон. Технические профили служат для обмена данными с клиентом Azure AD B2C, создания пользователя или чтения его профиля. Технический профиль поддерживает самостоятельное подтверждение для взаимодействия с пользователем. Например, он может получать учетные данные пользователя для входа, а затем отображать страницу регистрации или сброса пароля.

## <a name="type-of-technical-profiles"></a>Типы технических профилей

Технический профиль поддерживает следующие типы сценариев:

- [Application Insights](analytics-with-application-insights.md) — отправка данных событий в [Application Insights](../azure-monitor/app/app-insights-overview.md).
- [Azure Active Directory](active-directory-technical-profile.md): обеспечивает управление пользователями Azure Active Directory B2C.
- [Многофакторная идентификация Azure AD](multi-factor-auth-technical-profile.md) . обеспечивает поддержку проверки номера телефона с помощью многофакторной идентификации (MFA) Azure AD. 
- [Преобразование утверждений](claims-transformation-technical-profile.md): вызов преобразований исходящих утверждений для изменения значений утверждений, их проверки или установки значений по умолчанию для набора исходящих утверждений.
- [Указание токена идентификатора](id-token-hint.md) — проверяет `id_token_hint` подпись маркера JWT, имя издателя и аудиторию маркера и извлекает утверждение из входящего токена.
- [Издатель маркеров безопасности JWT](jwt-issuer-technical-profile.md): выдает маркер безопасности JWT, который возвращается в приложение проверяющей стороны.
- [OAuth1](oauth1-technical-profile.md): федерация с любым поставщиком удостоверений по протоколу OAuth 1.0.
- [OAuth2](oauth2-technical-profile.md): федерация с любым поставщиком удостоверений по протоколу OAuth 2.0.
- [Одноразовый пароль](one-time-password-technical-profile.md) . обеспечивает поддержку управления поколением и проверкой одноразового пароля.
- [OpenID Connect Connect](openid-connect-technical-profile.md) -Federation с любым поставщиком удостоверений протокола OpenID Connect Connect.
- [Телефонный фактор](phone-factor-technical-profile.md) — поддержка регистрации и проверки номеров телефонов.
- [Поставщик RESTful](restful-technical-profile.md) — вызов служб REST API Services, таких как проверка вводимых пользователем данных, обогащение пользовательской информации или интеграция с бизнес-приложениями.
- [Поставщик удостоверений SAML](identity-provider-generic-saml.md) — Федерация с любым поставщиком удостоверений протокола SAML.
- [Издатель токена SAML](saml-service-provider.md) — создает токен SAML, возвращаемый обратно в приложение проверяющей стороны.
- [Самостоятельное подтверждение](self-asserted-technical-profile.md): взаимодействие с пользователем. Например, получение учетных данных пользователя для входа, а затем отображение страницы регистрации или сброса пароля.
- [Управление сеансами](custom-policy-reference-sso.md): работа с различными типами сеансов.

## <a name="technical-profile-flow"></a>Процесс работы с техническим профилем

Все типы технических профилей созданы в рамках одной концепции. Начните с считывания входных утверждений, выполнение преобразования утверждений. Затем установите связь с настроенной стороной, например с поставщиком удостоверений, REST API или службами каталогов Azure AD. После завершения процесса технический профиль возвращает выходные утверждения и может выполнять преобразование выходных утверждений. Ниже показана схема обработки трансформаций и сопоставлений, на которые ссылается технический профиль. После выполнения преобразования заявок выходные утверждения немедленно сохраняются в контейнере утверждений. Независимо от того, с какой стороной работает технический профиль.

![Схема, иллюстрирующая поток технического профиля](./media/technical-profiles/technical-profile-flow.png)

1. **Управление сеансами единого входа (SSO)** — восстанавливает состояние сеанса технического профиля с помощью [управления сеансами единого входа](custom-policy-reference-sso.md).
1. **Преобразование "входные утверждения** " — перед запуском технического профиля Azure AD B2C выполняет [Преобразование](claimstransformations.md)"входные утверждения".
1. **Входные утверждения** — утверждения выбираются из контейнера утверждений, которые используются для технического профиля.
1. **Выполнение технического профиля**: технический профиль обменивается утверждениями с настроенной службой. Пример:
    - Пользователь перенаправляется к поставщику удостоверений для завершения процедуры входа. После успешного входа пользователь возвращается назад, и выполнение технического профиля продолжается.
    - Вызов REST API с последующей отправкой параметров в поле InputClaims и получение сведений в поле OutputClaims.
    - Создание или обновление учетной записи пользователя.
    - Отправка и проверка текстовых сообщений многофакторной проверки подлинности.
1. **Технические профили проверки** . [самостоятельно утвержденный технический профиль](self-asserted-technical-profile.md) может вызывать [технические профили проверки](validation-technical-profile.md) для проверки данных, которые пользователь проверил.
1. **Исходящие утверждения** — утверждения возвращаются в контейнер утверждений. Эти утверждения можно использовать на следующем шаге оркестрации или для преобразования исходящих утверждений.
1. Преобразования **выходных утверждений** . После завершения технического профиля Azure AD B2C запускает преобразование "выходные [утверждения](claimstransformations.md)". 
1. **Управление сеансами единого входа (SSO)** — сохраняет данные технического профиля в сеансе с помощью [управления сеансами единого входа](custom-policy-reference-sso.md).

Элемент **TechnicalProfiles** содержит набор технических профилей, поддерживаемых поставщиком утверждений. Каждый поставщик утверждений должен иметь по крайней мере один технический профиль. Технический профиль определяет конечные точки и протоколы, необходимые для взаимодействия с поставщиком утверждений. Поставщик утверждений может иметь несколько технических профилей.

```xml
<ClaimsProvider>
  <DisplayName>Display name</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Technical profile identifier">
      <DisplayName>Display name of technical profile</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        ...
      </Metadata>
      ...
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

Элемент **TechnicalProfile** содержит следующий атрибут.

| attribute | Обязательно | Описание |
|---------|---------|---------|
| Идентификатор | Да | Уникальный идентификатор технического профиля. Этот идентификатор позволяет указать технический профиль в других элементах в файле политики. Например, в элементах **OrchestrationSteps** и **ValidationTechnicalProfile**. |

Элемент **TechnicalProfile** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Домен | 0:1 | Доменное имя технического профиля. Например, если в техническом профиле указан поставщик удостоверений Facebook, доменное имя будет иметь значение Facebook.com. |
| DisplayName | 1:1 | Отображаемое имя технического профиля. |
| Описание | 0:1 | Описание технического профиля. |
| Протокол | 1:1 | Протокол, используемый для взаимодействия с другой стороной. |
| Метаданные | 0:1 | Коллекция "ключ — значение", которая управляет поведением технического профиля. |
| InputTokenFormat | 0:1 | Формат входного маркера. Возможные значения: `JSON`, `JWT`, `SAML11` или `SAML2`. Значение `JWT` обозначает JSON Web Token по спецификации IETF. Значение `SAML11` обозначает веб-маркер SAML 1.1 по спецификации OASIS.  Значение `SAML2` обозначает веб-маркер SAML 2.0 по спецификации OASIS. |
| OutputTokenFormat | 0:1 | Формат выходного маркера. Возможные значения: `JSON`, `JWT`, `SAML11` или `SAML2`. |
| CryptographicKeys | 0:1 | Список криптографических ключей, используемых в техническом профиле. |
| InputClaimsTransformations | 0:1 | Список ссылок на предварительно определенные преобразования утверждений, которые должны быть выполнены перед отправкой утверждений поставщику утверждений или проверяющей стороне. |
| InputClaims | 0:1 | Список ссылок на предварительно определенные типы утверждений, которые принимаются как входные данные для технического профиля. |
| PersistedClaims | 0:1 | Список ранее определенных ссылок на типы утверждений, которые будут храниться в техническом профиле. |
| дисплайклаимс | 0:1 | Список ранее определенных ссылок на типы утверждений, представленных в [самостоятельно утвержденном техническом профиле](self-asserted-technical-profile.md). Функция Дисплайклаимс сейчас доступна в **предварительной версии**. |
| OutputClaims | 0:1 | Список ссылок на предварительно определенные типы утверждений, которые используются как выходные данные для технического профиля. |
| OutputClaimsTransformations | 0:1 | Список ссылок на предварительно определенные преобразования утверждений, которые должны быть выполнены после получения утверждений от поставщика утверждений. |
| ValidationTechnicalProfiles | 0:n | Список ссылок на другие технические профили, которые этот технический профиль использует для проверки. Дополнительные сведения см. в разделе [Проверка технического профиля](validation-technical-profile.md) .|
| SubjectNamingInfo | 0:1 | Управляет созданием имени субъекта, для маркеров, в которых имя субъекта указывается отдельно от утверждений. Например, OAuth или SAML.  |
| инклудеинссо | 0:1 |  Следует ли использовать этот технический профиль в качестве поведения единого входа (SSO) для сеанса или вместо этого требовать явное взаимодействие. Этот элемент допустим только в профилях SelfAsserted, используемых в техническом профиле проверки. Возможные значения: `true` (по умолчанию) или `false`. |
| IncludeClaimsFromTechnicalProfile | 0:1 | Идентификатор технического профиля, все входные и выходные утверждения из которого нужно добавить в текущий технический профиль. Технический профиль, указанный в этой ссылке, должен быть определен в том же файле политики. |
| IncludeTechnicalProfile |0:1 | Идентификатор технического профиля, все данные из которого нужно добавить в текущий технический профиль. |
| UseTechnicalProfileForSessionManagement | 0:1 | Другой технический профиль, который нужно использовать для управления сеансами. |
|EnabledForUserJourneys| 0:1 |Указывает, выполняется ли технический профиль в пути взаимодействия пользователя.  |

## <a name="protocol"></a>Протокол

**Протокол** указывает протокол, используемый для связи с другой стороной. Элемент **Protocol** содержит следующие атрибуты.

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| name | Да | Имя допустимого протокола, поддерживаемого в Azure AD B2C и используемого в составе технического профиля. Возможные значения: `OAuth1` , `OAuth2` , `SAML2` , `OpenIdConnect` , `Proprietary` или `None` . |
| Обработчик | Нет | Если для параметра имя протокола задано значение `Proprietary` , укажите имя сборки, используемой Azure AD B2C для определения обработчика протокола. |

## <a name="metadata"></a>Метаданные

Элемент **Metadata** содержит соответствующие параметры конфигурации для конкретного протокола. Список поддерживаемых метаданных приведен в соответствующей спецификации [технического профиля](#type-of-technical-profiles) . Элемент **метаданных** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Item | 0:n | Метаданные, относящиеся к техническому профилю. Каждый тип технического профиля имеет свой набор элементов метаданных. Дополнительные сведения см. в разделе о типах технических профилей.  |

### <a name="item"></a>Элемент

Элемент **Item** элемента **метаданных** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| Клавиши | Да | Ключ метаданных. Список элементов метаданных см. в статье каждый [тип технического профиля](#type-of-technical-profiles). |

В следующем примере показано использование метаданных, относящихся к [техническому профилю OAuth2](oauth2-technical-profile.md#metadata).

```xml
<TechnicalProfile Id="Facebook-OAUTH">
  ...
  <Metadata>
    <Item Key="ProviderName">facebook</Item>
    <Item Key="authorization_endpoint">https://www.facebook.com/dialog/oauth</Item>
    <Item Key="AccessTokenEndpoint">https://graph.facebook.com/oauth/access_token</Item>
    <Item Key="HttpBinding">GET</Item>
    <Item Key="UsePolicyInRedirectUri">0</Item>
    ...
  </Metadata>
  ...
</TechnicalProfile>
```

В следующем примере показано использование метаданных, относящихся к [REST API техническом профиле](restful-technical-profile.md#metadata).

```xml
<TechnicalProfile Id="REST-Validate-Email">
  ...
  <Metadata>
    <Item Key="ServiceUrl">https://api.sendgrid.com/v3/mail/send</Item>
    <Item Key="AuthenticationType">Bearer</Item>
    <Item Key="SendClaimsIn">Body</Item>
    ...
  </Metadata>
  ...
</TechnicalProfile>
```

## <a name="cryptographic-keys"></a>Криптографические ключи

Чтобы установить отношение доверия со службами, с которыми он интегрируется, Azure AD B2C хранит секреты и сертификаты в форме [ключей политики](policy-keys-overview.md). Во время выполнения технического профиля Azure AD B2C извлекает криптографические ключи из Azure AD B2C ключей политики. Затем использует ключи для установления доверия, шифрования или подписи маркера. Эти отношения доверия состоят из следующих:

- Федерация с поставщиками удостоверений [OAuth1](oauth1-technical-profile.md#cryptographic-keys), [OAuth2](oauth2-technical-profile.md#cryptographic-keys)и [SAML](identity-provider-generic-saml.md)
- Защита подключения с помощью [служб REST API Services](secure-rest-api.md)
- Подписывание и шифрование токенов [JWT](jwt-issuer-technical-profile.md#cryptographic-keys) и [SAML](saml-service-provider.md)

Элемент **CryptographicKeys** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Клавиши | 1:n | Криптографический ключ, используемый в этом техническом профиле. |

### <a name="key"></a>Клавиши

Элемент **Key** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| Идентификатор | Нет | Уникальный идентификатор конкретной пары ключей, который можно указывать в других элементах файла политики. |
| StorageReferenceId | Да | Идентификатор контейнера хранения ключей, который можно указывать в других элементах файла политики. |

## <a name="input-claims-transformations"></a>Преобразования входящих утверждений

Элемент **инпутклаимстрансформатионс** может содержать коллекцию элементов преобразования входящих утверждений, которые используются для изменения входных утверждений или создания нового. 

Выходные утверждения предыдущего преобразования заявок в коллекции преобразований утверждений могут быть входящими утверждениями последующего преобразования заявок на входные данные, что позволяет иметь последовательность преобразования заявок в зависимости друг от друга.

Элемент **InputClaimsTransformations** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| InputClaimsTransformation | 1:n | Идентификатор преобразования утверждений, который должен быть выполнен перед отправкой утверждений поставщику утверждений или проверяющей стороне. Преобразование утверждений позволяет изменить существующие утверждения ClaimsSchema или создать новые. |

### <a name="inputclaimstransformation"></a>InputClaimsTransformation

Элемент **InputClaimsTransformation** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ReferenceId | Да | Идентификатор преобразования утверждений, который уже определен в файле политики или файле родительской политики. |

Следующие технические профили ссылаются на преобразование **креатеосермаилсфромемаил** Claims. Преобразование «утверждения» добавляет значение `email` утверждения в `otherMails` коллекцию перед сохранением данных в каталоге.

```xml
<TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
  ...
  <InputClaimsTransformations>
    <InputClaimsTransformation ReferenceId="CreateOtherMailsFromEmail" />
  </InputClaimsTransformations>
  <PersistedClaims>
    <PersistedClaim ClaimTypeReferenceId="alternativeSecurityId" />
    <PersistedClaim ClaimTypeReferenceId="userPrincipalName" />
    <PersistedClaim ClaimTypeReferenceId="mailNickName" DefaultValue="unknown" />
    <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />
    <PersistedClaim ClaimTypeReferenceId="otherMails" />
    <PersistedClaim ClaimTypeReferenceId="givenName" />
    <PersistedClaim ClaimTypeReferenceId="surname" />
  </PersistedClaims>
  ...
</TechnicalProfile>
```

## <a name="input-claims"></a>Входящие утверждения

**Inputclaim** берет утверждения из контейнера утверждений и используется для технического профиля. Например, [технический профиль с самостоятельным подтверждением](self-asserted-technical-profile.md) использует входящие утверждения для предварительного заполнения исходящих утверждений, предоставляемых пользователем. Технический профиль REST API использует входящие утверждения для отправки входящих параметров в конечную точку REST API. Azure Active Directory использует входящее утверждение в качестве уникального идентификатора для чтения, обновления или удаления учетной записи.

Элемент **InputClaims** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| InputClaim | 1:n | Ожидаемый тип входящего утверждения. |

### <a name="inputclaim"></a>InputClaim

Элемент **InputClaim** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Да | Идентификатор типа утверждения. Утверждение уже определено в разделе схемы утверждений в файле политики или в родительском файле политики. |
| DefaultValue | Нет | Значение по умолчанию, которое будет использоваться для создания утверждения, если указанное в ClaimTypeReferenceId утверждение не существует, чтобы технический профиль мог использовать созданное утверждение в качестве InputClaim. |
| PartnerClaimType | Нет | Идентификатор типа утверждения внешнего партнера, с которым сопоставляется указанный тип утверждения политики. Если атрибут PartnerClaimType не указан, тип утверждения указанной политики сопоставляется с одноименным типом утверждения партнера. Это свойство используется, если стороны используют разные имена типов утверждения. Например, вы используете утверждение с именем givenName, а ваш партнер — утверждение с именем first_name. |

## <a name="display-claims"></a>Отображение утверждений

Элемент **дисплайклаимс** содержит список заявок, которые должны быть представлены на экране для получения данных от пользователя. В коллекции заявок на отображение можно включить ссылку на [тип утверждения](claimsschema.md)или созданный [вами элемент ввода-вывода](display-controls.md) . 

- Тип утверждения — это ссылка на утверждение, которое должно отображаться на экране. 
  - Чтобы заставить пользователя предоставить значение для конкретного утверждения, установите атрибут **Required** элемента **дисплайклаим** в `true` .
  - Чтобы предварительно заполнить значения для отображаемых утверждений, используйте входные утверждения, которые были описаны ранее. Элемент также может содержать значение по умолчанию.
  - Элементу **"множество" в** коллекции **дисплайклаимс** необходимо задать для элемента **усеринпуттипе** любой пользовательский тип ввода, поддерживаемый Azure AD B2C. Например, `TextBox` или `DropdownSingleSelect`.
- Элемент управления "Отображение" — это элемент пользовательского интерфейса, который имеет специальные функции и взаимодействует с серверной службой Azure AD B2C. Она позволяет пользователю выполнять действия на странице, которая вызывает технический профиль проверки на серверной части. Например, проверьте адрес электронной почты, номер телефона или номер клиента по программе лояльности.

Порядок элементов в **дисплайклаимс** указывает порядок, в котором Azure AD B2C отображает утверждения на экране. 

Элемент **дисплайклаимс** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| дисплайклаим | 1:n | Ожидаемый тип входящего утверждения. |

### <a name="displayclaim"></a>дисплайклаим

Элемент **дисплайклаим** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Нет | Идентификатор типа утверждения, который уже определен в разделе ClaimsSchema файла политики или файла родительской политики. |
| дисплайконтролреференцеид | Нет | Идентификатор [элемента управления отображением](display-controls.md) , который уже определен в разделе ClaimsSchema файла политики или родительского файла политики. |
| Обязательно | Нет | Указывает, требуется ли утверждение на отображение. |

В следующем примере показано использование утверждений и элементов управления отображением в самостоятельно утвержденном техническом профиле.

![Самостоятельно утвержденный технический профиль с утверждениями на отображение](./media/technical-profiles/display-claims.png)

В следующем техническом профиле:

- Первое утверждение вывода создает ссылку на `emailVerificationControl` элемент управления отображением, который собирает и проверяет адрес электронной почты.
- Пятый запрос на отображение создает ссылку на `phoneVerificationControl` элемент управления отображением, который собирает и проверяет номер телефона.
- Другие утверждения на отображение ClaimTypesся для сбора от пользователя.

```xml
<TechnicalProfile Id="Id">
  <DisplayClaims>
    <DisplayClaim DisplayControlReferenceId="emailVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="displayName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="givenName" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="surName" Required="true" />
    <DisplayClaim DisplayControlReferenceId="phoneVerificationControl" />
    <DisplayClaim ClaimTypeReferenceId="newPassword" Required="true" />
    <DisplayClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
  </DisplayClaims>
</TechnicalProfile>
```

### <a name="persisted-claims"></a>Материализованные утверждения

Элемент **персистедклаимс** содержит все значения, которые должны быть сохранены в [техническом профиле Azure AD](active-directory-technical-profile.md) с возможными сведениями о сопоставлении между типом утверждения, уже определенным в разделе [ClaimsSchema](claimsschema.md) политики, и именем атрибута Azure AD.

Имя утверждения — это имя [атрибута Azure AD](user-profile-attributes.md) , если не указан атрибут **партнерклаимтипе** , который содержит имя атрибута Azure AD.

Элемент **PersistedClaims** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| PersistedClaim | 1:n | Тип утверждения, который нужно сохранять. |

### <a name="persistedclaim"></a>PersistedClaim

Элемент **PersistedClaim** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Да | Идентификатор типа утверждения, который уже определен в разделе ClaimsSchema файла политики или файла родительской политики. |
| DefaultValue | Нет | Значение по умолчанию, используемое для создания утверждения, если утверждение не существует. |
| PartnerClaimType | Нет | Идентификатор типа утверждения внешнего партнера, с которым сопоставляется указанный тип утверждения политики. Если атрибут PartnerClaimType не указан, тип утверждения указанной политики сопоставляется с одноименным типом утверждения партнера. Это свойство используется, если стороны используют разные имена типов утверждения. Например, вы используете утверждение с именем givenName, а ваш партнер — утверждение с именем first_name. |

В следующем примере — технический профиль **AAD-усервритеусинглогонемаил** или [начальный пакет](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/SocialAndLocalAccounts), который создает новую локальную учетную запись, сохраняет следующие утверждения:

```xml
<PersistedClaims>
  <PersistedClaim ClaimTypeReferenceId="email" PartnerClaimType="signInNames.emailAddress" />
  <PersistedClaim ClaimTypeReferenceId="newPassword" PartnerClaimType="password"/>
  <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="unknown" />
  <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration" />
  <PersistedClaim ClaimTypeReferenceId="givenName" />
  <PersistedClaim ClaimTypeReferenceId="surname" />
</PersistedClaims>
```

## <a name="output-claims"></a>Исходящие утверждения

**OutputClaims** — это коллекция утверждений, возвращаемых обратно в контейнер утверждений после завершения технического профиля. Эти утверждения можно использовать на следующем шаге оркестрации или для преобразования исходящих утверждений. Элемент **PersistedClaim** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| outputClaim | 1:n | Ожидаемый тип исходящего утверждения. |

### <a name="outputclaim"></a>outputClaim

Элемент **OutputClaim** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Да | Идентификатор типа утверждения, который уже определен в разделе ClaimsSchema файла политики или файла родительской политики. |
| DefaultValue | Нет | Значение по умолчанию, используемое для создания утверждения, если утверждение не существует. |
|AlwaysUseDefaultValue |Нет |Принудительное использование значения по умолчанию.  |
| PartnerClaimType | Нет | Идентификатор типа утверждения внешнего партнера, с которым сопоставляется указанный тип утверждения политики. Если атрибут типа утверждения партнера не указан, указанный тип утверждения политики сопоставляется с типом утверждения партнера с тем же именем. Это свойство используется, если стороны используют разные имена типов утверждения. Например, вы используете утверждение с именем givenName, а ваш партнер — утверждение с именем first_name. |

## <a name="output-claims-transformations"></a>Преобразования исходящих утверждений

Элемент **аутпутклаимстрансформатионс** может содержать коллекцию элементов **аутпутклаимстрансформатион** . Преобразования исходящих утверждений используются для изменения исходящих утверждений или создания новых. После выполнения исходящие утверждения возвращаются в хранилище утверждений. Эти утверждения можно использовать на следующем шаге оркестрации.

Выходные утверждения предыдущего преобразования заявок в коллекции преобразований заявок могут быть входящими утверждениями последующего преобразования заявок на входные данные, что позволяет использовать последовательность преобразования заявок в зависимости друг от друга.

Элемент **OutputClaimsTransformations** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| OutputClaimsTransformation | 1:n | Идентификаторы преобразований утверждений, которые должны быть выполнены перед отправкой утверждений поставщику утверждений или проверяющей стороне. Преобразование утверждений позволяет изменить существующие утверждения ClaimsSchema или создать новые. |

### <a name="outputclaimstransformation"></a>OutputClaimsTransformation

Элемент **OutputClaimsTransformations** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ReferenceId | Да | Идентификатор преобразования утверждений, который уже определен в файле политики или файле родительской политики. |

Следующий технический профиль ссылается на преобразование Ассертаккаунтенабледиструе Claims, чтобы определить, включена ли учетная запись после считывания `accountEnabled` утверждения из каталога.    

```xml
<TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
  ...
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="accountEnabled" />
    <OutputClaim ClaimTypeReferenceId="otherMails" />
    <OutputClaim ClaimTypeReferenceId="signInNames.emailAddress" />
  </OutputClaims>
  <OutputClaimsTransformations>
    <OutputClaimsTransformation ReferenceId="AssertAccountEnabledIsTrue" />
  </OutputClaimsTransformations>
  ...
</TechnicalProfile>
```

## <a name="validation-technical-profiles"></a>Технические профили проверки

Технический профиль проверки используется для проверки исходящих утверждений в [самостоятельно утвержденном техническом профиле](self-asserted-technical-profile.md#validation-technical-profiles). Технический профиль проверки — это обычный технический профиль любого протокола, такого как [Azure Active Directory](active-directory-technical-profile.md) или [REST API](restful-technical-profile.md). Технический профиль проверки возвращает выходные утверждения или код ошибки. Сообщение об ошибке отображается для пользователя на экране, что позволяет пользователю повторить попытку.

На следующей схеме показано, как Azure AD B2C использует технический профиль проверки для проверки учетных данных пользователя.

![Поток технического профиля проверки схемы](./media/technical-profiles/validation-technical-profile.png) 

Элемент **ValidationTechnicalProfiles** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| ValidationTechnicalProfile | 1:n | Идентификаторы технических профилей, которые используются для проверки некоторых или всех выходных утверждений технического профиля, в который включен этот элемент. Все входящие утверждения указанного технического профиля должны присутствовать в списке выходных утверждений технического профиля, в который включен этот элемент. |

### <a name="validationtechnicalprofile"></a>ValidationTechnicalProfile

Элемент **ValidationTechnicalProfile** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ReferenceId | Да | Идентификатор технического профиля, который уже определен в файле политики или файле родительской политики. |

## <a name="subjectnaminginfo"></a>SubjectNamingInfo

**Субжектнамингинфо** определяет имя субъекта, используемое в маркерах в [политике проверяющей стороны](relyingparty.md#subjectnaminginfo). Элемент **SubjectNamingInfo** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimType | Да | Идентификатор типа утверждения, который уже определен в разделе ClaimsSchema файла политики. |

## <a name="include-technical-profile"></a>Включить технический профиль

Технический профиль может включать еще один технический профиль для изменения параметров или добавления новых функциональных возможностей. Элемент **IncludeTechnicalProfile** — это ссылка на общий технический профиль, от которого наследуется технический профиль. Чтобы сократить избыточность и сложность элементов политики, используйте включение при наличии нескольких технических профилей, совместно использующих основные элементы. Используйте общий технический профиль с общим набором конфигураций, а также с конкретными техническими профилями задач, включающими общий технический профиль. 

Предположим, у вас есть [REST API технический профиль](restful-technical-profile.md) с одной конечной точкой, где необходимо отправить разные наборы заявок для различных сценариев. Создайте общий технический профиль с общими функциями, такими как, REST API URI конечной точки, метаданные, тип проверки подлинности и криптографические ключи. Создание технических профилей конкретной задачи, включающих в себя общий технический профиль. Затем добавьте входные утверждения, исходящие утверждения или перезапишите URI конечной точки REST API, относящийся к этому техническому профилю.

Элемент **IncludeTechnicalProfile** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ReferenceId | Да | Идентификатор технического профиля, уже определенного в файле политики или в родительском файле политики. |


В следующем примере показано использование включения:

- *API-интерфейс RESTful* — общий технический профиль с базовой конфигурацией.
- *Валидатепрофиле* -включает в себя технический профиль *API-интерфейса RESTful* и указывает на входные и выходные утверждения.
- *Упдатепрофиле* -включает в себя технический профиль Write- *API-Common* , задает входные утверждения и перезаписывает `ServiceUrl` метаданные.

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-API-Common">
      <DisplayName>Base REST API configuration</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity</Item>
        <Item Key="AuthenticationType">Basic</Item>
        <Item Key="SendClaimsIn">Body</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="BasicAuthenticationUsername" StorageReferenceId="B2C_1A_B2cRestClientId" />
        <Key Id="BasicAuthenticationPassword" StorageReferenceId="B2C_1A_B2cRestClientSecret" />
      </CryptographicKeys>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>

    <TechnicalProfile Id="REST-ValidateProfile">
      <DisplayName>Validate the account and return promo code</DisplayName>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="email" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="promoCode" />
      </OutputClaims>
      <IncludeTechnicalProfile ReferenceId="REST-API-Common" />
    </TechnicalProfile>

    <TechnicalProfile Id="REST-UpdateProfile">
       <Metadata>
        <Item Key="ServiceUrl">https://your-app-name.azurewebsites.NET/api/identity/update</Item>
      </Metadata>
      <DisplayName>Update the user profile</DisplayName>
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="email" />
      </InputClaims>
      <IncludeTechnicalProfile ReferenceId="REST-API-Common" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

### <a name="multi-level-inclusion"></a>Включение нескольких уровней 

Технический профиль может включать один технический профиль. Количество уровней включения не ограничено. Например технический профиль **AAD-UserReadUsingAlternativeSecurityId-NoError** включает **AAD-UserReadUsingAlternativeSecurityId**. Этот технический профиль задает `RaiseErrorIfClaimsPrincipalDoesNotExist` для элемента метаданных значение `true` и вызывает ошибку, если учетная запись социальных сетей не существует в каталоге. **AAD-усерреадусингалтернативесекуритид-Error** переопределяет это поведение и отключает это сообщение об ошибке.

```xml
<TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId-NoError">
  <Metadata>
    <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">false</Item>
  </Metadata>
  <IncludeTechnicalProfile ReferenceId="AAD-UserReadUsingAlternativeSecurityId" />
</TechnicalProfile>
```

**AAD-UserReadUsingAlternativeSecurityId** включает технический профиль `AAD-Common`.

```xml
<TechnicalProfile Id="AAD-UserReadUsingAlternativeSecurityId">
  <Metadata>
    <Item Key="Operation">Read</Item>
    <Item Key="RaiseErrorIfClaimsPrincipalDoesNotExist">true</Item>
    <Item Key="UserMessageIfClaimsPrincipalDoesNotExist">User does not exist. Please sign up before you can sign in.</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="AlternativeSecurityId" PartnerClaimType="alternativeSecurityId" Required="true" />
  </InputClaims>
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="objectId" />
    <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
    <OutputClaim ClaimTypeReferenceId="displayName" />
    <OutputClaim ClaimTypeReferenceId="otherMails" />
    <OutputClaim ClaimTypeReferenceId="givenName" />
    <OutputClaim ClaimTypeReferenceId="surname" />
  </OutputClaims>
  <IncludeTechnicalProfile ReferenceId="AAD-Common" />
</TechnicalProfile>
```

Как **AAD-усерреадусингалтернативесекуритид-Error** , так и  **AAD-усерреадусингалтернативесекуритид** не указывают требуемый элемент **протокола** , так как он указан в техническом профиле **AAD-Common** .

```xml
<TechnicalProfile Id="AAD-Common">
  <DisplayName>Azure Active Directory</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureActiveDirectoryProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  ...
</TechnicalProfile>
```

## <a name="use-technical-profile-for-session-management"></a>Использование технического профиля для управления сеансами

Ссылка на элемент **усетечникалпрофилефорсессионманажемент** для [технического профиля сеанса единого входа](custom-policy-reference-sso.md). Элемент **UseTechnicalProfileForSessionManagement** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ReferenceId | Да | Идентификатор технического профиля, который уже определен в файле политики или файле родительской политики. |

## <a name="enabled-for-user-journeys"></a>Включено для пути взаимодействия пользователя

[ClaimsProviderSelections](userjourneys.md#identity-provider-selection) в пути взаимодействия пользователя определяет список и порядок расположения вариантов для выбора поставщика утверждений. Элемент **EnabledForUserJourneys** позволяет отфильтровать доступных для пользователя поставщиков утверждений. Элемент **EnabledForUserJourneys** содержит одно из следующих значений:

- **Always** обозначает, что технический профиль выполняется.
- **Never** обозначает, что технический профиль пропускается.
- **OnClaimsExistence** указывает, что выполнение происходит только при наличии в техническом профиле определенного утверждения.
- **OnItemExistenceInStringCollectionClaim** указывает, что выполнение происходит только при наличии элемента в утверждении коллекции строк.
- **OnItemAbsenceInStringCollectionClaim** указывает, что выполнение происходит только при отсутствии элемента в утверждении коллекции строк.

При использовании **онклаимсексистенце**, **онитемексистенцеинстрингколлектионклаим** или **онитемабсенцеинстрингколлектионклаим** необходимо предоставить следующие метаданные: 

- **Клаимтипеонвхичтоенабле** — указывает оцениваемый тип утверждения.
- **Клаимвалуеонвхичтоенабле** — указывает сравниваемое значение.

Следующий технический профиль выполняется, только когда коллекция строк **identityProviders** содержит значение `facebook.com`:

```xml
<TechnicalProfile Id="UnLink-Facebook-OAUTH">
  <DisplayName>Unlink Facebook</DisplayName>
...
    <Metadata>
      <Item Key="ClaimTypeOnWhichToEnable">identityProviders</Item>
      <Item Key="ClaimValueOnWhichToEnable">facebook.com</Item>
    </Metadata>
...
  <EnabledForUserJourneys>OnItemExistenceInStringCollectionClaim</EnabledForUserJourneys>
</TechnicalProfile>
```
