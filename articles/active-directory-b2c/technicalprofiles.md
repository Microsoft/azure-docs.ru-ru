---
title: Технические профили
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
ms.openlocfilehash: bcff1ffd574db910c3206d82e4da0e9428db788f
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102631801"
---
# <a name="technical-profiles"></a>Технические профили

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

*Технический профиль* предоставляет платформу со встроенным механизмом взаимодействия с разными типами сторон. Технические профили используются для взаимодействия с клиентом Azure Active Directory B2C (Azure AD B2C) для создания пользователя или чтения профиля пользователя. Технический профиль поддерживает самостоятельное подтверждение для взаимодействия с пользователем. Например, технический профиль может получать учетные данные пользователя для входа, а затем отображать страницу регистрации или страницу сброса пароля.

## <a name="types-of-technical-profiles"></a>Типы технических профилей

Технический профиль поддерживает следующие типы сценариев:

- [Application Insights](analytics-with-application-insights.md): отправляет данные событий в [Application Insights](../azure-monitor/app/app-insights-overview.md).
- [Azure AD](active-directory-technical-profile.md): обеспечивает поддержку Azure AD B2C управления пользователями.
- [Многофакторная проверка подлинности Azure AD](multi-factor-auth-technical-profile.md): обеспечивает поддержку проверки подлинности на основе многофакторной идентификации в Azure AD.
- [Преобразование утверждений](claims-transformation-technical-profile.md): вызывает преобразования выходных утверждений для управления значениями утверждений, проверки утверждений или установки значений по умолчанию для набора исходящих утверждений.
- [Указание маркера идентификации](id-token-hint.md): проверяет `id_token_hint` подпись маркера JWT, имя издателя и аудиторию токена и извлекает утверждение из входящего токена.
- [Издатель токена JWT](jwt-issuer-technical-profile.md): выдает маркер JWT, возвращаемый обратно в приложение проверяющей стороны.
- [OAuth1](oauth1-technical-profile.md): Федерация с любым поставщиком удостоверений протокола OAuth 1,0.
- [OAuth2](oauth2-technical-profile.md): Федерация с любым поставщиком удостоверений протокола OAuth 2,0.
- [Одноразовый пароль](one-time-password-technical-profile.md): обеспечивает поддержку для управления созданием и проверкой одноразового пароля.
- [OpenID Connect Connect](openid-connect-technical-profile.md): Федерация с любым поставщиком удостоверений протокола OpenID Connect Connect.
- [Телефонный фактор](phone-factor-technical-profile.md): поддерживает регистрацию и проверку номеров телефонов.
- [Поставщик RESTful](restful-technical-profile.md): вызывает REST API службы, такие как проверка вводимых пользователем данных, дополнение к пользовательским данным или интеграция с бизнес-приложениями.
- [Поставщик удостоверений SAML](identity-provider-generic-saml.md). Федерация с любым поставщиком удостоверений протокола SAML.
- [Издатель токена SAML](saml-service-provider.md): создает токен SAML, возвращаемый обратно в приложение проверяющей стороны.
- [Самостоятельное утверждение](self-asserted-technical-profile.md): взаимодействует с пользователем. Например, собирает учетные данные пользователя для входа, подготовки к просмотру страницы регистрации или сброса пароля.
- [Управление сеансами](custom-policy-reference-sso.md): обрабатывает различные типы сеансов.

## <a name="technical-profile-flow"></a>Процесс работы с техническим профилем

Все типы технических профилей созданы в рамках одной концепции. Они начинают с чтения входных утверждений и выполнения преобразований утверждений. Затем они взаимодействуют с настроенной стороной, например с поставщиком удостоверений, REST API или службами каталогов Azure AD. После завершения процесса технический профиль возвращает выходные утверждения и может выполнять преобразования выходных утверждений. Ниже показана схема обработки трансформаций и сопоставлений, на которые ссылается технический профиль. После выполнения преобразования заявок выходные утверждения немедленно сохраняются в контейнере утверждений независимо от того, с какой стороной работает технический профиль.

![Схема, на которой показан поток технического профиля.](./media/technical-profiles/technical-profile-flow.png)

1. **Управление сеансами единого входа (SSO)**. восстанавливает состояние сеанса технического профиля с помощью [управления сеансами единого входа](custom-policy-reference-sso.md).
1. **Преобразование "входные утверждения**". перед запуском технического профиля Azure AD B2C выполняет [Преобразование](claimstransformations.md)"входные утверждения".
1. **Входные утверждения**: утверждения выбираются из контейнера утверждений, которые используются для технического профиля.
1. **Техническое выполнение профиля**. технический профиль обменивается утверждениями с настроенной стороной. Пример.
    - Перенаправляет пользователя к поставщику удостоверений, чтобы завершить вход. После успешного входа пользователь возвращается назад, и выполнение технического профиля продолжается.
    - Вызывает REST API при отправке параметров как Inputclaim и получении сведений обратно в виде OutputClaims.
    - Создает или обновляет учетную запись пользователя.
    - Отправляет и проверяет текстовое сообщение многофакторной проверки подлинности.
1. **Технические профили проверки**. [самостоятельно утвержденный технический профиль](self-asserted-technical-profile.md) может вызывать [технические профили проверки](validation-technical-profile.md) для проверки данных, которые пользователь проверил.
1. **Выходные утверждения**. утверждения возвращаются в контейнер утверждений. Эти утверждения можно использовать в следующих преобразованиях действий согласований или выходных утверждений.
1. **Преобразования исходящих утверждений**: после завершения технического профиля Azure AD B2C выполняет [преобразования выходных утверждений](claimstransformations.md).
1. **Управление сеансами единого входа**: сохраняет данные технического профиля в сеансе с помощью [управления сеансами единого входа](custom-policy-reference-sso.md).

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
| Идентификатор | Да | Уникальный идентификатор технического профиля. На технический профиль можно ссылаться с помощью этого идентификатора из других элементов в файле политики. Примерами являются **орчестратионстепс** и **валидатионтечникалпрофиле**. |

Элемент **техническом профиле** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Домен | 0:1 | Доменное имя технического профиля. Например, если в техническом профиле указан поставщик удостоверений Facebook, доменное имя будет иметь значение Facebook.com. |
| DisplayName | 1:1 | Отображаемое имя технического профиля. |
| Описание | 0:1 | Описание технического профиля. |
| Протокол | 1:1 | Протокол, используемый для взаимодействия с другой стороной. |
| Метаданные | 0:1 | Набор ключей и значений, управляющих поведением технического профиля. |
| InputTokenFormat | 0:1 | Формат входного маркера. Возможные значения: `JSON`, `JWT`, `SAML11` или `SAML2`. `JWT`Значение представляет JSON Web token в соответствии со спецификацией IETF. `SAML11`Значение представляет токен безопасности SAML 1,1 для спецификации Oasis. `SAML2`Значение представляет токен безопасности SAML 2,0 для спецификации Oasis. |
| OutputTokenFormat | 0:1 | Формат выходного маркера. Возможные значения: `JSON`, `JWT`, `SAML11` или `SAML2`. |
| CryptographicKeys | 0:1 | Список криптографических ключей, используемых в техническом профиле. |
| InputClaimsTransformations | 0:1 | Список ссылок на предварительно определенные преобразования утверждений, которые должны быть выполнены перед отправкой утверждений поставщику утверждений или проверяющей стороне. |
| InputClaims | 0:1 | Список ранее определенных ссылок на типы утверждений, которые берутся в качестве входных данных в техническом профиле. |
| PersistedClaims | 0:1 | Список ранее определенных ссылок на типы утверждений, которые будут сохраняться техническим профилем. |
| дисплайклаимс | 0:1 | Список ранее определенных ссылок на типы утверждений, представленных в [самостоятельно утвержденном техническом профиле](self-asserted-technical-profile.md). Функция Дисплайклаимс сейчас доступна в предварительной версии. |
| OutputClaims | 0:1 | Список ранее определенных ссылок на типы утверждений, которые берутся в качестве выходных данных в техническом профиле. |
| OutputClaimsTransformations | 0:1 | Список ссылок на предварительно определенные преобразования утверждений, которые должны быть выполнены после получения утверждений от поставщика утверждений. |
| ValidationTechnicalProfiles | 0:n | Список ссылок на другие технические профили, которые этот технический профиль использует для проверки. Дополнительные сведения см. в разделе [Проверка технического профиля](validation-technical-profile.md).|
| SubjectNamingInfo | 0:1 | Управляет созданием имени субъекта, для маркеров, в которых имя субъекта указывается отдельно от утверждений. Примерами являются OAuth или SAML. |
| инклудеинссо | 0:1 | Следует ли использовать этот технический профиль в качестве поведения единого входа для сеанса или вместо этого требовать явное взаимодействие. Этот элемент допустим только в профилях SelfAsserted, используемых в техническом профиле проверки. Возможные значения: `true` (по умолчанию) или `false`. |
| IncludeClaimsFromTechnicalProfile | 0:1 | Идентификатор технического профиля, все входные и выходные утверждения из которого нужно добавить в текущий технический профиль. Технический профиль, указанный в этой ссылке, должен быть определен в том же файле политики. |
| IncludeTechnicalProfile |0:1 | Идентификатор технического профиля, все данные из которого нужно добавить в текущий технический профиль. |
| UseTechnicalProfileForSessionManagement | 0:1 | Другой технический профиль, который нужно использовать для управления сеансами. |
|EnabledForUserJourneys| 0:1 |Указывает, выполняется ли технический профиль в пути взаимодействия пользователя. |

## <a name="protocol"></a>Протокол

Элемент **Protocol** указывает протокол, используемый для связи с другой стороной. Элемент **Protocol** содержит следующие атрибуты.

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| name | Да | Имя допустимого протокола, поддерживаемого Azure AD B2C, который используется как часть технического профиля. Возможные значения: `OAuth1` , `OAuth2` , `SAML2` , `OpenIdConnect` , `Proprietary` или `None` . |
| Обработчик | Нет | Если имя протокола установлено в значение `Proprietary` , то указывает имя сборки, используемой Azure AD B2C для определения обработчика протокола. |

## <a name="metadata"></a>Метаданные

Элемент **Metadata** содержит соответствующие параметры конфигурации для конкретного протокола. Список поддерживаемых метаданных приведен в соответствующей спецификации [технического профиля](#types-of-technical-profiles) . Элемент **метаданных** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Item | 0:n | Метаданные, относящиеся к техническому профилю. Каждый тип технического профиля имеет свой набор элементов метаданных. Дополнительные сведения см. в разделе о типах технических профилей. |

### <a name="item"></a>Элемент

Элемент **Item** элемента **метаданных** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| Клавиши | Да | Ключ метаданных. Просмотрите каждый [тип технического профиля](#types-of-technical-profiles) для списка элементов метаданных. |

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

В следующем примере показано использование метаданных, относящихся к [техническому профилю REST API](restful-technical-profile.md#metadata).

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

Чтобы установить отношение доверия со службами, с которыми он интегрируется, Azure AD B2C хранит секреты и сертификаты в форме [ключей политики](policy-keys-overview.md). Во время выполнения технического профиля Azure AD B2C извлекает криптографические ключи из Azure AD B2C ключей политики. Затем Azure AD B2C использует ключи для установления доверия или шифрования или подписи маркера. Эти отношения доверия состоят из следующих:

- Федерация с поставщиками удостоверений [OAuth1](oauth1-technical-profile.md#cryptographic-keys), [OAuth2](oauth2-technical-profile.md#cryptographic-keys)и [SAML](identity-provider-generic-saml.md) .
- Защита подключения с помощью [служб REST API](secure-rest-api.md).
- Подписывание и шифрование токенов [JWT](jwt-issuer-technical-profile.md#cryptographic-keys) и [SAML](saml-service-provider.md) .

Элемент **CryptographicKeys** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| Клавиши | 1:n | Криптографический ключ, используемый в этом техническом профиле. |

### <a name="key"></a>Ключ

Элемент **Key** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| Идентификатор | Нет | Уникальный идентификатор конкретной пары ключей, который можно указывать в других элементах файла политики. |
| StorageReferenceId | Да | Идентификатор контейнера хранения ключей, который можно указывать в других элементах файла политики. |

## <a name="input-claims-transformations"></a>Преобразования входящих утверждений

Элемент **инпутклаимстрансформатионс** может содержать коллекцию элементов преобразования входящих утверждений, которые используются для изменения входных утверждений или создания новых.

Выходные утверждения предыдущего преобразования заявок в коллекции преобразований заявок могут быть входящими утверждениями последующего преобразования заявок на входные данные. Таким образом можно иметь последовательность преобразований заявок, которые зависят друг от друга.

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

Элемент **inputclaim** выбирает утверждения из контейнера утверждений, которые используются для технического профиля. Например, [технический профиль с самостоятельным подтверждением](self-asserted-technical-profile.md) использует входящие утверждения для предварительного заполнения исходящих утверждений, предоставляемых пользователем. Технический профиль REST API использует входящие утверждения для отправки входящих параметров в конечную точку REST API. Azure AD использует входящее утверждение в качестве уникального идентификатора для чтения, обновления или удаления учетной записи.

Элемент **InputClaims** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| InputClaim | 1:n | Ожидаемый тип входящего утверждения. |

### <a name="inputclaim"></a>InputClaim

Элемент **InputClaim** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Да | Идентификатор типа утверждения. Утверждение уже определено в разделе схемы утверждений в файле политики или в родительском файле политики. |
| DefaultValue | Нет | Значение по умолчанию, используемое для создания утверждения, если утверждение, указанное в Claimtypereferenceid соединяется, не существует, чтобы полученное утверждение можно было использовать в качестве элемента InputClaim техническим профилем. |
| PartnerClaimType | Нет | Идентификатор типа утверждения внешнего партнера, с которым сопоставляется указанный тип утверждения политики. Если атрибут Партнерклаимтипе не указан, указанный тип утверждения политики сопоставляется с типом утверждения партнера с тем же именем. Это свойство используется, если стороны используют разные имена типов утверждения. Например, если имя первого утверждения — *givenName*, тогда как партнер использует утверждение с именем *first_name*. |

## <a name="display-claims"></a>Отображение утверждений

Элемент **дисплайклаимс** содержит список заявок, которые должны быть представлены на экране для получения данных от пользователя. В коллекции заявок на отображение можно включить ссылку на [тип утверждения](claimsschema.md) или [элемент управления отображением](display-controls.md) , который вы создали.

- Тип утверждения — это ссылка на утверждение, которое должно отображаться на экране.
  - Чтобы заставить пользователя предоставить значение для конкретного утверждения, установите атрибут **Required** элемента **дисплайклаим** в `true` .
  - Чтобы предварительно заполнить значения для отображаемых утверждений, используйте входные утверждения, которые были описаны ранее. Элемент также может содержать значение по умолчанию.
  - Элементу **"множество" в** коллекции **дисплайклаимс** необходимо задать для элемента **усеринпуттипе** любой пользовательский тип ввода, поддерживаемый Azure AD B2C. Примеры: `TextBox` или `DropdownSingleSelect` .
- Элемент управления "Отображение" — это элемент пользовательского интерфейса, который имеет специальные функции и взаимодействует с серверной службой Azure AD B2C. Она позволяет пользователю выполнять действия на странице, которая вызывает технический профиль проверки на серверной части. В качестве примера вы проверите адрес электронной почты, номер телефона или номер клиента по программе.

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

![Снимок экрана с автоматически подтвержденным техническим профилем с утверждениями отображения.](./media/technical-profiles/display-claims.png)

В следующем техническом профиле:

- Первое утверждение вывода создает ссылку на `emailVerificationControl` элемент управления отображением, который собирает и проверяет адрес электронной почты.
- Пятый запрос на отображение создает ссылку на `phoneVerificationControl` элемент управления отображением, который собирает и проверяет номер телефона.
- Другие утверждения на отображение — это элементы многоэлементного сбора, которые должны быть собраны от пользователя.

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

## <a name="persisted-claims"></a>Материализованные утверждения

Элемент **персистедклаимс** содержит все значения, которые должны быть сохранены в [техническом профиле Azure AD](active-directory-technical-profile.md) с возможными сведениями о сопоставлении между типом утверждения, уже определенным в разделе [ClaimsSchema](claimsschema.md) политики, и именем атрибута Azure AD.

Имя утверждения — это имя [атрибута Azure AD](user-profile-attributes.md) , если не указан атрибут **партнерклаимтипе** , который содержит имя атрибута Azure AD.

Элемент **персистедклаимс** содержит следующий элемент:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| PersistedClaim | 1:n | Тип утверждения, который нужно сохранять. |

### <a name="persistedclaim"></a>PersistedClaim

Элемент **PersistedClaim** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Да | Идентификатор типа утверждения, который уже определен в разделе ClaimsSchema файла политики или файла родительской политики. |
| DefaultValue | Нет | Значение по умолчанию, используемое для создания утверждения, если утверждение не существует. |
| PartnerClaimType | Нет | Идентификатор типа утверждения внешнего партнера, с которым сопоставляется указанный тип утверждения политики. Если атрибут Партнерклаимтипе не указан, указанный тип утверждения политики сопоставляется с типом утверждения партнера с тем же именем. Это свойство используется, если стороны используют разные имена типов утверждения. Например, если имя первого утверждения — *givenName*, тогда как партнер использует утверждение с именем *first_name*. |

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

Элемент **OutputClaims** представляет собой коллекцию утверждений, возвращаемых обратно в контейнер утверждений после завершения технического профиля. Эти утверждения можно использовать в следующих преобразованиях действий согласований или выходных утверждений. Элемент **PersistedClaim** содержит следующие элементы:

| Элемент | Вхождения | Описание |
| ------- | ----------- | ----------- |
| outputClaim | 1:n | Ожидаемый тип исходящего утверждения. |

### <a name="outputclaim"></a>outputClaim

Элемент **OutputClaim** содержит следующие атрибуты:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimTypeReferenceId | Да | Идентификатор типа утверждения, который уже определен в разделе ClaimsSchema файла политики или файла родительской политики. |
| DefaultValue | Нет | Значение по умолчанию, используемое для создания утверждения, если утверждение не существует. |
|AlwaysUseDefaultValue |Нет |Принудительное использование значения по умолчанию. |
| PartnerClaimType | Нет | Идентификатор типа утверждения внешнего партнера, с которым сопоставляется указанный тип утверждения политики. Если атрибут типа утверждения партнера не указан, указанный тип утверждения политики сопоставляется с типом утверждения партнера с тем же именем. Это свойство используется, если стороны используют разные имена типов утверждения. Например, если имя первого утверждения — *givenName*, тогда как партнер использует утверждение с именем *first_name*. |

## <a name="output-claims-transformations"></a>Преобразования исходящих утверждений

Элемент **аутпутклаимстрансформатионс** может содержать коллекцию элементов **аутпутклаимстрансформатион** . Преобразования исходящих утверждений используются для изменения исходящих утверждений или создания новых. После выполнения исходящие утверждения возвращаются в хранилище утверждений. Эти утверждения можно использовать на следующем шаге оркестрации.

Выходные утверждения предыдущего преобразования заявок в коллекции преобразований заявок могут быть входящими утверждениями последующего преобразования заявок на входные данные. Таким образом можно иметь последовательность преобразований заявок, которые зависят друг от друга.

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

Технический профиль проверки используется для проверки исходящих утверждений в [самостоятельно утвержденном техническом профиле](self-asserted-technical-profile.md#validation-technical-profiles). Технический профиль проверки — это обычный технический профиль по любому протоколу, например [Azure AD](active-directory-technical-profile.md) или [REST API](restful-technical-profile.md). Технический профиль проверки возвращает выходные утверждения или возвращает код ошибки. Сообщение об ошибке отображается для пользователя на экране, что позволяет пользователю повторить попытку.

На следующей схеме показано, как Azure AD B2C использует технический профиль проверки для проверки учетных данных пользователя.

![Схема, на которой показан поток технического профиля проверки.](./media/technical-profiles/validation-technical-profile.png)

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

Элемент **субжектнамингинфо** определяет имя субъекта, используемое в маркерах в [политике проверяющей стороны](relyingparty.md#subjectnaminginfo). Элемент **SubjectNamingInfo** содержит следующий атрибут.

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ClaimType | Да | Идентификатор типа утверждения, который уже определен в разделе ClaimsSchema файла политики. |

## <a name="include-technical-profile"></a>Включить технический профиль

Технический профиль может включать еще один технический профиль для изменения параметров или добавления новых функциональных возможностей. Элемент **IncludeTechnicalProfile** — это ссылка на общий технический профиль, от которого наследуется технический профиль. Чтобы сократить избыточность и сложность элементов политики, используйте включение при наличии нескольких технических профилей, совместно использующих основные элементы. Используйте общий технический профиль с общим набором конфигураций, а также с конкретными техническими профилями задач, включающими общий технический профиль.

Предположим, у вас есть [REST API технический профиль](restful-technical-profile.md) с одной конечной точкой, где необходимо отправить разные наборы утверждений для различных сценариев. Создайте общий технический профиль с общими функциями, такими как URI конечной точки REST API, метаданные, тип проверки подлинности и криптографические ключи. Создание технических профилей конкретной задачи, включающих в себя общий технический профиль. Затем добавьте входные и выходные утверждения или перезапишите URI конечной точки REST API, относящийся к этому техническому профилю.

Элемент **IncludeTechnicalProfile** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ReferenceId | Да | Идентификатор технического профиля, который уже определен в файле политики или файле родительской политики. |

В следующем примере показано использование включения:

- **API-интерфейс RESTful**: общий технический профиль с базовой конфигурацией.
- Функция **валидатепрофиле**: включает в себя технический профиль **API-интерфейса "оставшийся интерфейс** ", а также указывает входные и выходные утверждения.
- Write **-упдатепрофиле**: включает в себя технический профиль **API-интерфейса RESTful** , задает входные утверждения и перезаписывает `ServiceUrl` метаданные.

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

### <a name="multilevel-inclusion"></a>Многоуровневый включение

Технический профиль может включать один технический профиль. Количество уровней включения не ограничено. Например, технический профиль **AAD-усерреадусингалтернативесекуритид-Error** включает **AAD-усерреадусингалтернативесекуритид**. Этот технический профиль задает `RaiseErrorIfClaimsPrincipalDoesNotExist` элемент метаданных `true` и вызывает ошибку, если учетная запись социальных сетей не существует в каталоге. **AAD-усерреадусингалтернативесекуритид-Error** переопределяет это поведение и отключает это сообщение об ошибке.

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

Как **AAD-усерреадусингалтернативесекуритид-Error** , так и **AAD-усерреадусингалтернативесекуритид** не указывают необходимый элемент **протокола** , так как он указан в техническом профиле **AAD-Common** .

```xml
<TechnicalProfile Id="AAD-Common">
  <DisplayName>Azure Active Directory</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.AzureActiveDirectoryProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  ...
</TechnicalProfile>
```

## <a name="use-technical-profile-for-session-management"></a>Использование технического профиля для управления сеансами

Элемент **усетечникалпрофилефорсессионманажемент** ссылается на [технический профиль сеанса единого входа](custom-policy-reference-sso.md). Элемент **UseTechnicalProfileForSessionManagement** содержит следующий атрибут:

| attribute | Обязательно | Описание |
| --------- | -------- | ----------- |
| ReferenceId | Да | Идентификатор технического профиля, который уже определен в файле политики или файле родительской политики. |

## <a name="enabled-for-user-journeys"></a>Включено для пути взаимодействия пользователя

[ClaimsProviderSelections](userjourneys.md#claims-provider-selection) в пути взаимодействия пользователя определяет список и порядок расположения вариантов для выбора поставщика утверждений. С помощью элемента **енабледфорусержаурнэйс** вы фильтруете, какой поставщик утверждений доступен пользователю. Элемент **EnabledForUserJourneys** содержит одно из следующих значений:

- **Всегда**: выполняет технический профиль.
- **Никогда**: пропускает технический профиль.
- **Онклаимсексистенце**: выполняется, только если существует определенное утверждение, указанное в техническом профиле.
- **Онитемексистенцеинстрингколлектионклаим**: выполняется, только если элемент существует в утверждении коллекции строк.
- **Онитемабсенцеинстрингколлектионклаим**: выполняется, только если элемент не существует в утверждении коллекции строк.

Для использования **онклаимсексистенце**, **онитемексистенцеинстрингколлектионклаим** или **онитемабсенцеинстрингколлектионклаим** необходимо предоставить следующие метаданные:

- **Клаимтипеонвхичтоенабле**: указывает тип утверждения, который требуется вычислить.
- **Клаимвалуеонвхичтоенабле**: указывает сравниваемое значение.

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
