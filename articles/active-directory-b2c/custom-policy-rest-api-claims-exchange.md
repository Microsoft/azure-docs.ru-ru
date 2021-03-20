---
title: Обмен утверждениями REST API в Azure Active Directory B2C
description: Добавление возможности обмена утверждениями REST API в настраиваемые политики в Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 84053df34ffda0d4686ad80a9e5f3af00ac53d72
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94949502"
---
# <a name="walkthrough-add-rest-api-claims-exchanges-to-custom-policies-in-azure-active-directory-b2c"></a>Пошаговое руководство. Добавление возможности обмена утверждениями REST API в настраиваемые политики в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) позволяет разработчикам служб удостоверений интегрировать взаимодействие с REST API в путь взаимодействия пользователя. Изучив это пошаговое руководство, вы сможете создавать путь взаимодействия пользователя Azure AD B2C, который взаимодействует со [службами RESTful](custom-policy-rest-api-intro.md).

В этом сценарии вы обогатите данные токена пользователя за счет интеграции с корпоративным бизнес-процессом. Во время регистрации или входа с использованием локальной или федеративной учетной записи Azure AD B2C вызывает REST API для получения расширенных данных профиля пользователя из удаленного источника данных. В этом примере Azure AD B2C отправляет уникальный идентификатор пользователя — objectId. Затем REST API возвращает значение суммы остатка на счете пользователя (случайное число). Используйте этот пример в качестве отправной точки для интеграции с собственной системой CRM, маркетинговой базой данных или любым рабочим бизнес-процессом.

Взаимодействие можно также реализовать как технический профиль проверки. Этот вариант подходит, когда REST API будет проверять данные на экране и возвращать утверждения. См. дополнительные сведения в руководствах по [ Интеграция обмена утверждениями REST API в пути взаимодействия пользователя Azure AD B2C как проверка входных данных](custom-policy-rest-api-claims-validation.md).

## <a name="prerequisites"></a>Предварительные требования

- Выполните шаги, описанные в статье [Начало работы с настраиваемыми политиками в Azure Active Directory B2C](custom-policy-get-started.md). Для регистрации и входа с использованием локальных учетных записей необходимо иметь рабочую настраиваемую политику.
- Прочтите статью [Интеграция обмена утверждениями REST API в настраиваемую политику Azure AD B2C](custom-policy-rest-api-intro.md).

## <a name="prepare-a-rest-api-endpoint"></a>Подготовка конечной точки REST API

Для работы с этим пошаговым руководством у вас должен быть REST API, который проверяет, зарегистрирован ли идентификатор Azure AD B2C objectId пользователя в серверной системе. Если идентификатор зарегистрирован, REST API возвращает значение суммы остатка на счете. В противном случае REST API регистрирует новый счет в каталоге и возвращает начальный остаток на счете `50.00`.

В следующем коде JSON показаны данные, которые Azure AD B2C отправит в конечную точку REST API. 

```json
{
    "objectId": "User objectId",
    "lang": "Current UI language"
}
```

После того как REST API проверит данные, он должен вернуть HTTP 200 (ОК) со следующими данными JSON:

```json
{
    "balance": "760.50"
}
```

В рамках этой статьи настройка конечной точки REST API не рассматривается. Вы создали пример [Функций Azure](../azure-functions/functions-reference.md). Полный код функции Azure доступен на сайте [GitHub](https://github.com/azure-ad-b2c/rest-api/tree/master/source-code/azure-function).

## <a name="define-claims"></a>Определение утверждений

Утверждение предоставляет временное хранилище данных во время выполнения политики Azure AD B2C. Утверждения можно объявить в разделе [схемы утверждений](claimsschema.md). 

1. Откройте файл расширения политики. например <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Найдите элемент [BuildingBlocks](buildingblocks.md). Если такой элемент не существует, добавьте его.
1. Найдите элемент [ClaimsSchema](claimsschema.md). Если такой элемент не существует, добавьте его.
1. Добавьте следующие утверждения в элемент **ClaimsSchema**.  

```xml
<ClaimType Id="balance">
  <DisplayName>Your Balance</DisplayName>
  <DataType>string</DataType>
</ClaimType>
<ClaimType Id="userLanguage">
  <DisplayName>User UI language (used by REST API to return localized error messages)</DisplayName>
  <DataType>string</DataType>
</ClaimType>
```

## <a name="add-the-restful-api-technical-profile"></a>Добавление технического профиля RESTFUL API 

[Технический профиль RESTful](restful-technical-profile.md) обеспечивает поддержку взаимодействия с вашей собственной службой RESTFUL. Azure AD B2C отправляет данные в службу RESTful в виде коллекции `InputClaims` ответы в виде коллекции `OutputClaims`. Найдите элемент **ClaimsProviders** в файле <em> **`TrustFrameworkExtensions.xml`**</em> и добавьте новый поставщик утверждений, как показано ниже.

```xml
<ClaimsProvider>
  <DisplayName>REST APIs</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="REST-GetProfile">
      <DisplayName>Get user extended profile Azure Function web hook</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
      <Metadata>
        <!-- Set the ServiceUrl with your own REST API endpoint -->
        <Item Key="ServiceUrl">https://your-account.azurewebsites.net/api/GetProfile?code=your-code</Item>
        <Item Key="SendClaimsIn">Body</Item>
        <!-- Set AuthenticationType to Basic or ClientCertificate in production environments -->
        <Item Key="AuthenticationType">None</Item>
        <!-- REMOVE the following line in production environments -->
        <Item Key="AllowInsecureAuthInProduction">true</Item>
      </Metadata>
      <InputClaims>
        <!-- Claims sent to your REST API -->
        <InputClaim ClaimTypeReferenceId="objectId" />
        <InputClaim ClaimTypeReferenceId="userLanguage" PartnerClaimType="lang" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
      </InputClaims>
      <OutputClaims>
        <!-- Claims parsed from your REST API -->
        <OutputClaim ClaimTypeReferenceId="balance" />
      </OutputClaims>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
``` 

В этом примере `userLanguage` отправляется в службу REST как `lang` в полезных данных JSON. Значение утверждения `userLanguage` содержит текущий идентификатор языка пользователя. Дополнительные сведения см. в статье об [арбитрах утверждений](claim-resolver-overview.md).

### <a name="configure-the-restful-api-technical-profile"></a>Настройка технического профиля RESTful API 

После развертывания REST API Задайте метаданные `REST-ValidateProfile` технического профиля в соответствии с собственными REST API, включая:

- **ServiceURL**. Задайте URL-адрес конечной точки REST API.
- **Сендклаимсин**. Укажите, как входящие утверждения отправляются поставщику утверждений RESTFUL.
- **AuthenticationType**. Задайте тип проверки подлинности, выполняемой поставщиком утверждений RESTFUL. 
- **Алловинсекуреаусинпродуктион**. Убедитесь, что в рабочей среде для этих метаданных задано значение `true`
    
Дополнительные конфигурации см. в [метаданных технического профиля RESTful](restful-technical-profile.md#metadata) .

В комментариях над `AuthenticationType` и `AllowInsecureAuthInProduction` указаны изменения, которые следует внести при переходе в рабочую среду. Сведения о защите RESTful API для рабочей среды см. в [этой статье](secure-rest-api.md).

## <a name="add-an-orchestration-step"></a>Добавление шага оркестрации

В [путях взаимодействия пользователя](userjourneys.md) указываются явные пути, через которые политика позволяет приложению, основанному на утверждениях, предоставлять пользователю требуемые утверждения. Путь взаимодействия пользователя представляется в виде последовательности оркестрации, которая должна быть соблюдена для успешного выполнения транзакции. Можно добавлять или удалять шаги оркестрации. Здесь вы добавите новый шаг оркестрации, который будет использоваться для расширения сведений, предоставленных приложению после регистрации или входа пользователя путем вызова REST API.

1. Откройте базовый файл политики. Например, <em>`SocialAndLocalAccounts/`**`TrustFrameworkBase.xml`**</em>.
1. Найдите элемент `<UserJourneys>`. Скопируйте весь элемент, а затем удалите его.
1. Откройте файл расширения политики. Например, <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.
1. Вставьте `<UserJourneys>` в файл расширений после закрытия элемента `<ClaimsProviders>`.
1. Найдите `<UserJourney Id="SignUpOrSignIn">` и добавьте следующий шаг оркестрации перед последним.

    ```xml
    <OrchestrationStep Order="7" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="RESTGetProfile" TechnicalProfileReferenceId="REST-GetProfile" />
      </ClaimsExchanges>
    </OrchestrationStep>
    ```

1. Выполните рефакторинг последнего шага оркестрации, изменив `Order` на `8`. Последние два шага оркестрации должны выглядеть следующим образом.

    ```xml
    <OrchestrationStep Order="7" Type="ClaimsExchange">
      <ClaimsExchanges>
        <ClaimsExchange Id="RESTGetProfile" TechnicalProfileReferenceId="REST-GetProfile" />
      </ClaimsExchanges>
    </OrchestrationStep>

    <OrchestrationStep Order="8" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    ```

1. Повторите два последних шага для путей взаимодействия пользователя **ProfileEdit** и **PasswordReset**.


## <a name="include-a-claim-in-the-token"></a>Включение утверждения в токен 

Чтобы вернуть утверждение `balance` в приложение проверяющей стороны, добавьте выходное утверждение в файл <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em>. При добавлении выходного утверждения утверждение включается в токен после успешного пути взаимодействия пользователя и будет отправлено в приложение. Измените элемент технического профиля в разделе проверяющей стороны, добавив `balance` в качестве выходного утверждения.
 
```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="OpenIdConnect" />
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="displayName" />
      <OutputClaim ClaimTypeReferenceId="givenName" />
      <OutputClaim ClaimTypeReferenceId="surname" />
      <OutputClaim ClaimTypeReferenceId="email" />
      <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="sub"/>
      <OutputClaim ClaimTypeReferenceId="identityProvider" />
      <OutputClaim ClaimTypeReferenceId="tenantId" AlwaysUseDefaultValue="true" DefaultValue="{Policy:TenantObjectId}" />
      <OutputClaim ClaimTypeReferenceId="balance" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

Повторите этот шаг для путей взаимодействия пользователя **ProfileEdit.xml** и **PasswordReset.xml**.

Сохраните измененные файлы: *TrustFrameworkBase.xml* и *TrustFrameworkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* и *PasswordReset.xml*. 

## <a name="test-the-custom-policy"></a>Проверка пользовательской политики

1. Войдите на [портал Azure](https://portal.azure.com).
1. Убедитесь, что вы используете каталог с клиентом Azure AD, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом Azure AD.
1. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Регистрация приложений**.
1. Выберите **Инфраструктура процедур идентификации**.
1. Выберите **Отправка пользовательской политики** и отправьте файлы политики, которые вы изменили ранее *TrustFrameworkBase.xml* и *TrustFrameworkExtensions.xml*, *SignUpOrSignin.xml*, *ProfileEdit.xml* и *PasswordReset.xml*. 
1. Выберите отправленную вами политику регистрации или входа и нажмите кнопку **Выполнить**.
1. Вы сможете зарегистрироваться, используя адрес электронной почты и учетную запись Facebook.
1. Токен, отправленный обратно в ваше приложение, включает утверждение `balance`.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1584961516,
  "nbf": 1584957916,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1584957916,
  "auth_time": 1584957916,
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "balance": "202.75"
  ...
}
```

## <a name="next-steps"></a>Дальнейшие действия

Из следующих статей вы узнаете, как защитить API-интерфейсы.

- [Пошаговое руководство. Интеграция обмена утверждениями REST API в пути взаимодействия пользователя Azure AD B2C как этап оркестрации](custom-policy-rest-api-claims-exchange.md)
- [Защита интерфейса RESTful API](secure-rest-api.md)
- [Общие сведения ](restful-technical-profile.md)Технический профиль RESTful