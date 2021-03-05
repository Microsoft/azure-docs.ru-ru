---
title: Добавление атрибутов пользователя и настройка пользовательского ввода
titleSuffix: Azure AD B2C
description: Узнайте, как настроить ввод пользователя и добавить атрибуты пользователя для процесса регистрации или входа в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/04/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: e76fe1c26f428403a79a3605b7a41f761fe2a4bb
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102171626"
---
#  <a name="add-user-attributes-and-customize-user-input-in-azure-active-directory-b2c"></a>Добавление атрибутов пользователя и настройка пользовательского ввода в Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-custom-policy"

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

::: zone-end

В этой статье вы соберете новый атрибут во время регистрации в Azure Active Directory B2C (Azure AD B2C). Вы получите город пользователя, настроите его в виде раскрывающегося списка и определите, требуется ли предоставлять его.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

::: zone pivot="b2c-user-flow"

## <a name="add-user-attributes-your-user-flow"></a>Добавление атрибутов пользователя в поток пользователя

1. В клиенте Azure AD B2C выберите **Потоки пользователей**.
1. Откройте политику (например B2C_1_SignupSignin), щелкнув ее.
1. Выберите **атрибуты пользователя** , а затем выберите атрибут User (например, "City"). 
1. Щелкните **Сохранить**.

## <a name="provide-optional-claims-to-your-app"></a>Предоставление приложению дополнительных утверждений

Утверждения приложения — это значения, возвращаемые приложению. Обновите поток пользователя, чтобы он содержал нужные утверждения.

1. Откройте политику (например B2C_1_SignupSignin), щелкнув ее.
1. Выберите элемент **Утверждения приложения**.
1. Выберите атрибуты, которые необходимо отправить обратно в приложение (например, "City").
1. Щелкните **Сохранить**.
 
## <a name="configure-user-attributes-input-type"></a>Настройка типа входных атрибутов пользователя

1. Откройте политику (например B2C_1_SignupSignin), щелкнув ее.
1. Выберите **макеты страниц**.
1. Выберите **страницу регистрации локальной учетной записи**.
1. В разделе **атрибуты пользователя** выберите **City**.
    1. В раскрывающемся списке **Пользовательский тип ввода** выберите **дропдовнсинглеселект**.
    1. В раскрывающемся списке **необязательный** выберите **нет**.
1. Щелкните **Сохранить**. 

### <a name="provide-a-list-of-values-by-using-localized-collections"></a>Предоставление списка значений с помощью локализованных коллекций

Чтобы задать список значений для атрибута City, сделайте следующее: 

1. [Включение настройки языка в потоке пользователя](language-customization.md#support-requested-languages-for-ui_locales)
1. Откройте политику (например B2C_1_SignupSignin), щелкнув ее.
1. На странице **Языки** для потока пользователя выберите язык, который требуется настроить.
1. В разделе **файлы на уровне страницы** выберите **страница Регистрация локальной учетной записи**.
1. Нажмите кнопку **Скачать значения по умолчанию** (или **Скачать переопределения**, если этот язык ранее менялся).
1. Создайте `LocalizedCollections` атрибут.

`LocalizedCollections`— Это массив `Name` `Value` пар и. Порядок размещения элементов определяет порядок их отображения. 

* `ElementId` представляет собой пользовательский атрибут, ответом на который является `LocalizedCollections`.
* `Name` является значением, которое видит пользователь.
* `Value` возвращается в утверждении при выборе этого параметра.

```json
{
  "LocalizedStrings": [...],
  "LocalizedCollections": [
    {
      "ElementType": "ClaimType",
      "ElementId": "city",
      "TargetCollection": "Restriction",
      "Override": false,
      "Items": [
        {
          "Name": "Berlin",
          "Value": "Berlin"
        },
        {
          "Name": "London",
          "Value": "London"
        },
        {
          "Name": "Seattle",
          "Value": "Seattle"
        }
      ]
    }
  ]
}
```

#### <a name="upload-your-changes"></a>Загрузка изменений

1. Внеся изменения в JSON-файл, вернитесь к клиенту B2C.
1. Выберите **потоки пользователей** и выберите политику (например, "B2C_1_SignupSignin"), чтобы открыть ее.
1. Щелкните **Языки**.
1. Выберите язык, на который требуется выполнить перевод.
1. Выберите **страницу регистрации локальной учетной записи**.
1. Щелкните значок папки и выберите JSON-файл для отправки. Изменения автоматически сохраняются в потоке пользователя.

## <a name="test-your-user-flow"></a>Тестирование последовательности пользователей

1. Откройте политику (например B2C_1_SignupSignin), щелкнув ее.
1. Чтобы проверить политику, выберите пункт **выполнить пользовательскую последовательность**.
1. Для **приложения** выберите веб-приложение с именем *testapp1* , которое вы зарегистрировали ранее. В поле **URL-адрес ответа** должно содержаться значение `https://jwt.ms`.
1. Щелкните **выполнить поток пользователя**

::: zone-end

::: zone pivot="b2c-custom-policy"

> [!NOTE]
> В этом образце используется встроенное утверждение "City". Вместо этого можно выбрать один из поддерживаемых [Azure AD B2C встроенных атрибутов](user-profile-attributes.md) или пользовательский атрибут. Чтобы использовать настраиваемый атрибут, [включите настраиваемые атрибуты](user-flow-custom-attributes.md). Чтобы использовать другой встроенный или настраиваемый атрибут, замените City на нужный атрибут, например встроенный атрибут *JobTitle* или пользовательский атрибут, например *extension_loyaltyId*.  

Начальные данные пользователей можно собирать с помощью пути взаимодействия пользователя при регистрации или входа. Дополнительные утверждения можно собрать позже при редактировании профиля. Когда Azure AD B2C собирает информацию непосредственно из пользователя в интерактивном режиме, инфраструктура процедур идентификации использует его [самостоятельно утвержденный технический профиль](self-asserted-technical-profile.md). В этом примере вы выполните следующие действия:

1. Определите утверждение "город". 
1. Попросите пользователя указать город.
1. Сохранить город в профиле пользователя в каталоге Azure AD B2C.
1. Прочитайте утверждение City из Azure AD B2C каталога при каждом входе.
1. Верните город в приложение проверяющей стороны после входа или регистрации.  

## <a name="define-a-claim"></a>Определение утверждения

Утверждение предоставляет временное хранилище данных во время выполнения политики Azure AD B2C. [Схема утверждений](claimsschema.md) — это место, где вы объявляете свои утверждения. Для определения утверждения используются такие элементы:

- **DisplayName** — это строка, определяющая пользовательскую метку.
- [DataType](claimsschema.md#datatype) — тип утверждения.
- **UserHelpText** — позволяет пользователю узнать требования.
- [Усеринпуттипе](claimsschema.md#userinputtype) — тип элемента управления вводом, например текстовое поле, выбор переключателя, раскрывающийся список или множественный выбор.

Откройте файл расширения для политики, например <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.

1. Найдите элемент [BuildingBlocks](buildingblocks.md). Если такой элемент не существует, добавьте его.
1. Найдите элемент [ClaimsSchema](claimsschema.md). Если такой элемент не существует, добавьте его.
1. Добавьте утверждение City в элемент **ClaimsSchema** .  

```xml
<ClaimType Id="city">
  <DisplayName>City where you work</DisplayName>
  <DataType>string</DataType>
  <UserInputType>DropdownSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="Kirkland" Value="kirkland" SelectByDefault="false" />
  </Restriction>
</ClaimType>
```

## <a name="add-a-claim-to-the-user-interface"></a>Добавление утверждения в пользовательский интерфейс

Следующие технические профили [автоматически](self-asserted-technical-profile.md)вызываются, когда пользователь должен предоставить входные данные:

- **Локалаккаунтсигнупвислогонемаил** — поток регистрации локальной учетной записи.
- **SelfAsserted-социальное** -Федеративная учетная запись Вход пользователя в систему в первый раз.
- **SelfAsserted-ProfileUpdate** — изменение потока профиля.

Чтобы получить утверждение "город" во время регистрации, его необходимо добавить в технический профиль в качестве исходящего утверждения `LocalAccountSignUpWithLogonEmail` . Переопределите этот технический профиль в файле расширения. Укажите полный список исходящих утверждений для управления порядком, в котором утверждения представлены на экране. Найдите элемент **ClaimsProviders**. Добавьте новый Клаимспровидерс следующим образом:

```xml
<ClaimsProvider>
  <DisplayName>Local Account</DisplayName>
  <TechnicalProfiles>
    <!--Local account sign-up page-->
    <TechnicalProfile Id="LocalAccountSignUpWithLogonEmail">
      <OutputClaims>
       <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="Verified.Email" Required="true" />
       <OutputClaim ClaimTypeReferenceId="newPassword" Required="true" />
       <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
       <OutputClaim ClaimTypeReferenceId="displayName" />
       <OutputClaim ClaimTypeReferenceId="givenName" />
       <OutputClaim ClaimTypeReferenceId="surName" />
       <OutputClaim ClaimTypeReferenceId="city"/>
     </OutputClaims>
   </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

Чтобы получить утверждение "город" после начального входа с помощью федеративной учетной записи, его необходимо добавить в технический профиль в качестве исходящего утверждения `SelfAsserted-Social` . Чтобы пользователи локальных и федеративных учетных записей могли изменять данные своего профиля позже, добавьте входящие и исходящие утверждения в `SelfAsserted-ProfileUpdate` технический профиль. Переопределите эти технические профили в файле расширения. Укажите весь список исходящих утверждений для управления порядком представления заявок на экране. Найдите элемент **ClaimsProviders**. Добавьте новый Клаимспровидерс следующим образом:

```xml
<ClaimsProvider>
  <DisplayName>Self Asserted</DisplayName>
  <TechnicalProfiles>
    <!--Federated account first-time sign-in page-->
    <TechnicalProfile Id="SelfAsserted-Social">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="city" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName"/>
        <OutputClaim ClaimTypeReferenceId="givenName"/>
        <OutputClaim ClaimTypeReferenceId="surname"/>
        <OutputClaim ClaimTypeReferenceId="city"/>
      </OutputClaims>
    </TechnicalProfile>
    <!--Edit profile page-->
    <TechnicalProfile Id="SelfAsserted-ProfileUpdate">
      <InputClaims>
        <InputClaim ClaimTypeReferenceId="city" />
      </InputClaims>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName"/>
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="city"/>
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="read-and-write-a-claim"></a>Чтение и запись утверждения

Следующие технические профили являются [Active Directory техническими профилями](active-directory-technical-profile.md), которые считывают и записывают данные в Azure Active Directory.  
Используйте `PersistedClaims` для записи данных в профиль пользователя и `OutputClaims` чтения данных из профиля пользователя в соответствующих технических профилях Active Directory.

Переопределите эти технические профили в файле расширения. Найдите элемент **ClaimsProviders**.  Добавьте новый Клаимспровидерс следующим образом:

```xml
<ClaimsProvider>
  <DisplayName>Azure Active Directory</DisplayName>
  <TechnicalProfiles>
    <!-- Write data during a local account sign-up flow. -->
    <TechnicalProfile Id="AAD-UserWriteUsingLogonEmail">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Write data during a federated account first-time sign-in flow. -->
    <TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Write data during edit profile flow. -->
    <TechnicalProfile Id="AAD-UserWriteProfileUsingObjectId">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="city"/>
      </PersistedClaims>
    </TechnicalProfile>
    <!-- Read data after user authenticates with a local account. -->
    <TechnicalProfile Id="AAD-UserReadUsingEmailAddress">
      <OutputClaims>  
        <OutputClaim ClaimTypeReferenceId="city" />
      </OutputClaims>
    </TechnicalProfile>
    <!-- Read data after user authenticates with a federated account. -->
    <TechnicalProfile Id="AAD-UserReadUsingObjectId">
      <OutputClaims>  
        <OutputClaim ClaimTypeReferenceId="city" />
      </OutputClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

## <a name="include-a-claim-in-the-token"></a>Включение утверждения в токен 

Чтобы вернуть заявку City в приложение проверяющей стороны, добавьте в файл выходное утверждение <em>`SocialAndLocalAccounts/`**`SignUpOrSignIn.xml`**</em> . Выходное утверждение будет добавлено в маркер после успешного пути взаимодействия пользователя и будет отправлено в приложение. Измените элемент технического профиля в разделе проверяющей стороны, чтобы добавить город в качестве выходного утверждения.
 
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
      <OutputClaim ClaimTypeReferenceId="city" DefaultValue="" />
    </OutputClaims>
    <SubjectNamingInfo ClaimType="sub" />
  </TechnicalProfile>
</RelyingParty>
```

## <a name="test-the-custom-policy"></a>Проверка пользовательской политики

1. Войдите на [портал Azure](https://portal.azure.com).
2. Убедитесь, что вы используете каталог с клиентом Azure AD, выбрав фильтр **Каталог и подписка** в меню вверху и каталог с вашим клиентом Azure AD.
3. Выберите **Все службы** в левом верхнем углу окна портала Azure, а затем найдите и выберите **Регистрация приложений**.
4. Выберите **Инфраструктура процедур идентификации**.
5. Выберите **Отправка пользовательской политики** и отправьте два файла политики, которые вы изменили ранее.
2. Выберите отправленную вами политику регистрации или входа и нажмите кнопку **Выполнить**.
3. Вы сможете зарегистрироваться, используя адрес электронной почты.

::: zone-end

Экран регистрации должен выглядеть как на следующем снимке экрана:

![Снимок экрана с измененным вариантом регистрации](./media/configure-user-input/signup-with-city-claim-drop-down-example.png)

Токен, отправленный обратно в ваше приложение, включает утверждение `city`.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
}.{
  "exp": 1583500140,
  "nbf": 1583496540,
  "ver": "1.0",
  "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
  "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
  "acr": "b2c_1a_signup_signin",
  "nonce": "defaultNonce",
  "iat": 1583496540,
  "auth_time": 1583496540,
  "name": "Emily Smith",
  "email": "joe@outlook.com",
  "given_name": "Emily",
  "family_name": "Smith",
  "city": "Bellevue"
  ...
}
```

::: zone pivot="b2c-custom-policy"

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об элементе [ClaimsSchema](claimsschema.md) см. в справочнике по инфраструктура процедур идентификации.
- Узнайте, как [использовать настраиваемые атрибуты в Azure AD B2C](user-flow-custom-attributes.md).

::: zone-end
