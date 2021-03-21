---
title: Настройка Azure Active Directory B2C в качестве IdPа SAML для приложений
title-suffix: Azure Active Directory B2C
description: Настройка Azure Active Directory B2C для предоставления приложениям утверждений протокола SAML (поставщики служб). Azure AD B2C будет использоваться в качестве единого поставщика удостоверений (IdP) для приложения SAML.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/03/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 1035f43642f3884e7cc0f6ab47e9c9afd1f29170
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102107770"
---
# <a name="register-a-saml-application-in-azure-ad-b2c"></a>Регистрация приложения SAML в Azure AD B2C

Из этой статьи вы узнаете, как подключить приложения язык разметки зявлений системы безопасности (SAML) (SAML) (поставщики служб) к Azure Active Directory B2C (Azure AD B2C) для проверки подлинности.

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="overview"></a>Обзор

Организациям, использующим Azure AD B2C в качестве решения для управления удостоверениями и доступом клиентов, может потребоваться интеграция с приложениями, которые проходят проверку подлинности с помощью протокола SAML. На следующей схеме показано, как Azure AD B2C выступает в качестве *поставщика удостоверений* (IDP) для обеспечения единого входа (SSO) с помощью приложений на основе SAML.

![Схема с B2C в качестве поставщика удостоверений слева и B2C в качестве поставщика услуг справа.](media/saml-service-provider/saml-service-provider-integration.png)

1. Приложение создает запрос SAML AuthN, который отправляется в конечную точку входа SAML Azure AD B2c.
2. Для проверки подлинности пользователь может использовать локальную учетную запись Azure AD B2C или любой другой федеративный поставщик удостоверений (если настроен).
3. Если пользователь входит в систему с помощью федеративного поставщика удостоверений, в Azure AD B2C отправляется ответ маркера.
4. Azure AD B2C создает утверждение SAML и отправляет его в приложение.

## <a name="prerequisites"></a>Предварительные требования

* Выполните шаги, описанные в статье [Начало работы с настраиваемыми политиками в Azure AD B2C](custom-policy-get-started.md). Вам потребуется настраиваемая политика *SocialAndLocalAccounts* из начального пакета настраиваемых политик, который обсуждается в этой статье.
* Основные сведения о протоколе SAML и знакомство с реализацией SAML приложения.
* Веб-приложение, настроенное как приложение SAML. В рамках этого руководства вы можете использовать предоставляемое нами [тестовое приложение SAML][samltest].

## <a name="components"></a>Компоненты

Для этого сценария требуется три основных компонента:

* **Приложение** SAML с возможностью отправки запросов SAML AuthN, а также получения, декодирования и проверки ответов saml от Azure AD B2C. Приложение SAML также называется приложением проверяющей стороны или поставщиком услуг.
* Общедоступная **Конечная точка МЕТАДАННЫХ** SAML приложения SAML или XML-документ.
* [Клиент Azure AD B2C](tutorial-create-tenant.md)

Если у вас еще нет приложения SAML и связанной конечной точки метаданных, вы можете использовать этот пример приложения SAML, который мы сделали доступным для тестирования:

[Тестовое приложение SAML][samltest]

## <a name="set-up-certificates"></a>Настройка сертификатов

Чтобы создать отношение доверия между приложением и Azure AD B2C, обе службы должны иметь возможность создавать и проверять подписи друг друга. Настройте сертификаты X509 в Azure AD B2C и в приложении.

**сертификаты приложения.**

| Использование | Обязательно | Описание |
| --------- | -------- | ----------- |
| Подписывание запросов SAML  | Нет | Сертификат с закрытым ключом, хранящимся в веб-приложении и используемый приложением для подписи запросов SAML, отправленных в Azure AD B2C. Веб-приложение должно предоставлять открытый ключ через свою конечную точку метаданных SAML. Azure AD B2C проверяет подпись запроса SAML с помощью открытого ключа из метаданных приложения.|
| Шифрование утверждения SAML  | Нет | Сертификат с закрытым ключом, хранящимся в веб-приложении. Веб-приложение должно предоставлять открытый ключ через свою конечную точку метаданных SAML. Azure AD B2C может шифровать утверждения для приложения с помощью открытого ключа. Приложение использует закрытый ключ для расшифровки утверждения.|

**Сертификаты Azure AD B2C**

| Использование | Обязательно | Описание |
| --------- | -------- | ----------- |
| Подписывание ответа SAML | Да | Сертификат с закрытым ключом, хранящимся в Azure AD B2C. Этот сертификат используется Azure AD B2C для подписи ответа SAML, отправленного в приложение. Приложение считывает открытый ключ метаданных Azure AD B2C для проверки подписи ответа SAML. |

В рабочей среде рекомендуется использовать сертификаты, выданные общедоступным центром сертификации. Однако эту процедуру можно также выполнить с помощью самозаверяющих сертификатов.

### <a name="prepare-a-self-signed-certificate-for-saml-response-signing"></a>Подготовка самозаверяющего сертификата для подписывания ответа SAML

Необходимо создать сертификат подписи ответа SAML, чтобы приложение может доверять утверждению от Azure AD B2C.

[!INCLUDE [active-directory-b2c-create-self-signed-certificate](../../includes/active-directory-b2c-create-self-signed-certificate.md)]

## <a name="enable-your-policy-to-connect-with-a-saml-application"></a>Включение политики для подключения с помощью приложения SAML

Чтобы подключиться к приложению SAML, Azure AD B2C должен иметь возможность создавать ответы SAML.

Откройте `SocialAndLocalAccounts\`**`TrustFrameworkExtensions.xml`** в начальном пакете настраиваемых политик.

Откройте `<ClaimsProviders>` раздел и добавьте следующий фрагмент XML-кода, чтобы реализовать генератор ответов SAML.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>

    <!-- SAML Token Issuer technical profile -->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <Item Key="IssuerUri">https://issuerUriMyAppExpects</Item>
      </Metadata>
      <CryptographicKeys>
        <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_SamlIdpCert"/>
      </CryptographicKeys>
      <InputClaims/>
      <OutputClaims/>
      <UseTechnicalProfileForSessionManagement ReferenceId="SM-Saml-issuer"/>
    </TechnicalProfile>

    <!-- Session management technical profile for SAML based tokens -->
    <TechnicalProfile Id="SM-Saml-issuer">
      <DisplayName>Session Management Provider</DisplayName>
      <Protocol Name="Proprietary" Handler="Web.TPEngine.SSO.SamlSSOSessionProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null"/>
    </TechnicalProfile>

  </TechnicalProfiles>
</ClaimsProvider>
```

#### <a name="configure-the-issueruri-of-the-saml-response"></a>Настройка IssuerUri для ответа SAML

Значение `IssuerUri` элемента метаданных можно изменить в техническом профиле издателя маркера SAML. Это изменение будет отражено в `issuerUri` атрибуте, возвращенном в ответе SAML от Azure AD B2C. Приложение должно быть настроено на принятие одинаковых значений `issuerUri` во время проверки ответа SAML.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <!-- SAML Token Issuer technical profile -->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <Item Key="IssuerUri">https://issuerUriMyAppExpects</Item>
      </Metadata>
      ...
    </TechnicalProfile>
```

#### <a name="sign-the-azure-ad-b2c-idp-saml-metadata-optional"></a>Подпишите Azure AD B2C метаданные SAML IdP (необязательно)

Можно указать Azure AD B2C подписать свой документ метаданных IdP для SAML, если это требуется для приложения. Для этого создайте и отправьте ключ политики подписи метаданных SAML IdP, как показано в разделе [Подготовка самозаверяющего сертификата для подписания ответа SAML](#prepare-a-self-signed-certificate-for-saml-response-signing). Затем настройте `MetadataSigning` элемент метаданных в техническом профиле издателя маркера SAML. Объект `StorageReferenceId` должен ссылаться на имя ключа политики.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <!-- SAML Token Issuer technical profile -->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
        ...
      <CryptographicKeys>
        <Key Id="MetadataSigning" StorageReferenceId="B2C_1A_SamlMetadataCert"/>
        ...
      </CryptographicKeys>
    ...
    </TechnicalProfile>
```

#### <a name="sign-the-azure-ad-b2c-idp-saml-response-element-optional"></a>Подписывание Azure AD B2C элемента ответа SAML IdP (необязательно)

Можно указать сертификат, который будет использоваться для подписи сообщений SAML. Сообщение — это `<samlp:Response>` элемент в ответе SAML, отправляемом в приложение.

Чтобы указать сертификат, создайте и отправьте ключ политики, как показано в подразделе [Подготовка самозаверяющего сертификата для подписания ответа SAML](#prepare-a-self-signed-certificate-for-saml-response-signing). Затем настройте `SamlMessageSigning` элемент метаданных в техническом профиле издателя маркера SAML. Объект `StorageReferenceId` должен ссылаться на имя ключа политики.

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <!-- SAML Token Issuer technical profile -->
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
        ...
      <CryptographicKeys>
        <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_SamlMessageCert"/>
        ...
      </CryptographicKeys>
    ...
    </TechnicalProfile>
```
## <a name="configure-your-policy-to-issue-a-saml-response"></a>Настройка политики для выдаче ответа SAML

Теперь, когда ваша политика может создавать ответы SAML, необходимо настроить политику на выдачу ответа SAML вместо ответа JWT по умолчанию приложению.

### <a name="create-a-sign-up-or-sign-in-policy-configured-for-saml"></a>Создание политики регистрации или входа, настроенной для SAML

1. Создайте копию файла *SignUpOrSignin.xml* в рабочем каталоге начального пакета и сохраните ее под новым названием. Например, *SignUpOrSigninSAML.xml*. Этот файл является файлом политики проверяющей стороны, и по умолчанию он настроен на выдачу ответа JWT.

1. Откройте файл *SignUpOrSigninSAML.xml* в любом удобном редакторе.

1. Измените `PolicyId` и `PublicPolicyUri` политики на _B2C_1A_signup_signin_saml_ и `http://<tenant-name>.onmicrosoft.com/B2C_1A_signup_signin_saml`, как показано ниже.

    ```xml
    <TrustFrameworkPolicy
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"
    xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
    PolicySchemaVersion="0.3.0.0"
    TenantId="tenant-name.onmicrosoft.com"
    PolicyId="B2C_1A_signup_signin_saml"
    PublicPolicyUri="http://<tenant-name>.onmicrosoft.com/B2C_1A_signup_signin_saml">
    ```

1. В конце пути взаимодействия пользователя Azure AD B2C содержит `SendClaims` шаг. Этот шаг ссылается на технический профиль издателя маркера. Чтобы выдать ответ SAML, а не ответ JWT по умолчанию, измените `SendClaims` шаг, чтобы он ссылался на новый технический профиль издателя маркера SAML, `Saml2AssertionIssuer` .

Добавьте следующий фрагмент XML непосредственно перед `<RelyingParty>` элементом. Этот XML-файл перезаписывает шаг номер 7 в пути взаимодействия пользователя _SignUpOrSignIn_ . Если вы начали из другой папки в начальном пакете или настроили путь взаимодействия пользователя, добавив или удалив шаги оркестрации, убедитесь, что число в `order` элементе соответствует числу, указанному в пути взаимодействия пользователя для шага издателя маркера. Например, в других папках начального пакета соответствующий номер шага равен 4 для `LocalAccounts` , 6 — `SocialAccounts` и 9 — для `SocialAndLocalAccountsWithMfa` ).

```xml
<UserJourneys>
  <UserJourney Id="SignUpOrSignIn">
    <OrchestrationSteps>
      <OrchestrationStep Order="7" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="Saml2AssertionIssuer"/>
    </OrchestrationSteps>
  </UserJourney>
</UserJourneys>
```

Элемент проверяющей стороны определяет, какой протокол использует приложение. Значение по умолчанию — `OpenId`. `Protocol`Элемент должен быть изменен на `SAML` . Выходные утверждения будут создавать сопоставление утверждений с утверждением SAML.

Замените весь элемент `<TechnicalProfile>` в элементе `<RelyingParty>` следующим XML-кодом технического профиля. Обновите `tenant-name`, указав имя клиента Azure AD B2C.

```xml
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" DefaultValue="" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="objectId"/>
      </OutputClaims>
      <SubjectNamingInfo ClaimType="objectId" ExcludeAsClaim="true"/>
    </TechnicalProfile>
```

Окончательный файл политики проверяющей стороны должен выглядеть как следующий XML-код:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<TrustFrameworkPolicy
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="contoso.onmicrosoft.com"
  PolicyId="B2C_1A_signup_signin_saml"
  PublicPolicyUri="http://contoso.onmicrosoft.com/B2C_1A_signup_signin_saml">

  <BasePolicy>
    <TenantId>contoso.onmicrosoft.com</TenantId>
    <PolicyId>B2C_1A_TrustFrameworkExtensions</PolicyId>
  </BasePolicy>

  <UserJourneys>
    <UserJourney Id="SignUpOrSignIn">
      <OrchestrationSteps>
        <OrchestrationStep Order="7" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="Saml2AssertionIssuer"/>
      </OrchestrationSteps>
    </UserJourney>
  </UserJourneys>

  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="displayName" />
        <OutputClaim ClaimTypeReferenceId="givenName" />
        <OutputClaim ClaimTypeReferenceId="surname" />
        <OutputClaim ClaimTypeReferenceId="email" DefaultValue="" />
        <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="" />
        <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="objectId"/>
      </OutputClaims>
      <SubjectNamingInfo ClaimType="objectId" ExcludeAsClaim="true"/>
    </TechnicalProfile>
  </RelyingParty>
</TrustFrameworkPolicy>
```

> [!NOTE]
> Этот же процесс можно выполнить для реализации других типов потоков пользователей (например, для входа, сброса пароля или потоков редактирования профиля).

### <a name="upload-your-policy"></a>Отправка политики

Сохраните изменения и передайте новые файлы политики **TrustFrameworkExtensions.xml** и **SignUpOrSigninSAML.xml** в портал Azure.

### <a name="test-the-azure-ad-b2c-idp-saml-metadata"></a>Проверка Azure AD B2C метаданных SAML IdP

После передачи файлов политики Azure AD B2C использует сведения о конфигурации для создания документа метаданных SAML поставщика удостоверений, который будет использоваться приложением. В документе метаданных SAML содержатся расположения служб, такие как методы входа и выхода, сертификаты и т. д.

Метаданные политики Azure AD B2C доступны по следующему URL-адресу:

`https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/samlp/metadata`

Замените `<tenant-name>` именем клиента Azure AD B2C и `<policy-name>` именем (ID) политики, например:

`https://contoso.b2clogin.com/contoso.onmicrosoft.com/B2C_1A_signup_signin_saml/samlp/metadata`

## <a name="register-your-saml-application-in-azure-ad-b2c"></a>Регистрация приложения SAML в Azure AD B2C

Чтобы Azure AD B2C доверять приложению, создайте Azure AD B2C регистрации приложения, которая содержит сведения о конфигурации, такие как конечная точка метаданных приложения.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите фильтр **Каталог и подписка** в верхнем меню, а затем выберите каталог, содержащий клиент Azure AD B2C.
1. В меню слева выберите **Azure AD B2C**. Либо щелкните **Все службы**, а затем найдите и выберите **Azure AD B2C**
1. Щелкните **Регистрация приложений** и выберите **Новая регистрация**.
1. Введите **имя** приложения. Например, *SAMLApp1*.
1. В разделе **Поддерживаемые типы учетных записей** выберите **Учетные записи только в этом каталоге организации**.
1. В разделе **URI перенаправления** выберите **Интернет** и введите `https://localhost`. Это значение будет изменено позже в манифесте регистрации приложения.
1. Выберите **Зарегистрировать**.

### <a name="configure-your-application-in-azure-ad-b2c"></a>Настройка приложения в Azure AD B2C

Для приложений SAML необходимо настроить несколько свойств в манифесте регистрации приложения.

1. На [портале Azure](https://portal.azure.com) перейдите к регистрации приложения, созданной в предыдущем разделе.
1. В разделе **Управление** выберите **Манифест** , чтобы открыть редактор манифеста, а затем измените свойства, описанные в следующих разделах.

#### <a name="add-the-identifier"></a>Добавление идентификатора

Когда приложение SAML выполняет запрос на Azure AD B2C, запрос SAML AuthN включает `Issuer` атрибут, который обычно имеет то же значение, что и метаданные приложения `entityID` . Azure AD B2C использует это значение для поиска регистрации приложения в каталоге и чтения конфигурации. Для успешности этого уточняющего запроса в `identifierUri` регистрации приложения должно быть заполнено значение, соответствующее `Issuer` атрибуту.

В манифесте регистрации выберите `identifierURIs` параметр и добавьте соответствующее значение. Это значение будет совпадать со значением, настроенным в SAML AuthN запросы для EntityId в приложении, и `entityID` значение в метаданных приложения.

В следующем примере показано `entityID` в МЕТАДАННЫХ SAML:

```xml
<EntityDescriptor ID="id123456789" entityID="https://samltestapp2.azurewebsites.net" validUntil="2099-12-31T23:59:59Z" xmlns="urn:oasis:names:tc:SAML:2.0:metadata">
```

`identifierUris`Свойство будет принимать только URL-адреса в домене `tenant-name.onmicrosoft.com` .

```json
"identifierUris":"https://samltestapp2.azurewebsites.net",
```

#### <a name="share-the-applications-metadata-with-azure-ad-b2c"></a>Совместное использование метаданных приложения с Azure AD B2C

После загрузки регистрации приложения в `identifierUri` Azure AD B2C использует метаданные приложения для проверки запроса SAML AuthN и определения способа реагирования.

Рекомендуется, чтобы ваше приложение предоставляя общедоступную конечную точку метаданных.

Если есть свойства, указанные как в URL-адресе метаданных SAML, *так* и в манифесте регистрации приложения, они *объединяются*. Свойства, указанные в URL-адресе метаданных, обрабатываются первыми и имеют приоритет.

Используя тестовое приложение SAML в качестве примера, `samlMetadataUrl` в манифесте приложения можно использовать следующее значение:

```json
"samlMetadataUrl":"https://samltestapp2.azurewebsites.net/Metadata",
```

#### <a name="override-or-set-the-assertion-consumer-url-optional"></a>Переопределить или задать URL-адрес потребителя утверждений (необязательно)

Можно настроить URL-адрес ответа, по которому Azure AD B2C отправляет ответы SAML. URL-адреса ответа можно настроить в манифесте приложения. Эта конфигурация полезна, если приложение не предоставляет общедоступную конечную точку метаданных.

URL-адрес ответа для приложения SAML — это конечная точка, в которой приложение ожидает получения ответов SAML. Приложение обычно предоставляет этот URL-адрес в документе метаданных в `AssertionConsumerServiceUrl` атрибуте, как показано ниже:

```xml
<SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    ...
    <AssertionConsumerService index="0" isDefault="true" Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://samltestapp2.azurewebsites.net/SP/AssertionConsumer" />        
</SPSSODescriptor>
```

Если вы хотите переопределить метаданные, указанные в `AssertionConsumerServiceUrl` атрибуте, или URL-адрес отсутствует в документе метаданных, можно настроить URL-адрес в манифесте под `replyUrlsWithType` свойством. `BindingType`Для свойства будет задано значение `HTTP POST` .

С помощью тестового приложения SAML в качестве примера свойству необходимо присвоить `url` `replyUrlsWithType` значение, показанное в следующем фрагменте кода JSON.

```json
"replyUrlsWithType":[
  {
    "url":"https://samltestapp2.azurewebsites.net/SP/AssertionConsumer",
    "type":"Web"
  }
],
```

#### <a name="override-or-set-the-logout-url-optional"></a>Переопределение или задание URL-адреса выхода (необязательно)

Можно настроить URL-адрес выхода, по которому Azure AD B2C будет отсылать пользователя после запроса на выход. URL-адреса ответа можно настроить в манифесте приложения.

Если вы хотите переопределить метаданные, указанные в `SingleLogoutService` атрибуте, или URL-адрес отсутствует в документе метаданных, его можно настроить в манифесте под `Logout` свойством. `BindingType`Для свойства будет задано значение `Http-Redirect` .

Приложение обычно предоставляет этот URL-адрес в документе метаданных в `AssertionConsumerServiceUrl` атрибуте, как показано ниже:

```xml
<IDPSSODescriptor WantAuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://samltestapp2.azurewebsites.net/logout" ResponseLocation="https://samltestapp2.azurewebsites.net/logout" />

</IDPSSODescriptor>
```

Используя тестовое приложение SAML в качестве примера, оставьте `logoutUrl` значение `https://samltestapp2.azurewebsites.net/logout` :

```json
"logoutUrl": "https://samltestapp2.azurewebsites.net/logout",
```

> [!NOTE]
> Если вы решили настроить URL-адрес ответа и URL-адрес выхода в манифесте приложения без заполнения конечной точки метаданных приложения с помощью `samlMetadataUrl` свойства, Azure AD B2C не будет проверять сигнатуру запроса SAML и не будет шифровать ответ SAML.

## <a name="configure-azure-ad-b2c-as-a-saml-idp-in-your-saml-application"></a>Настройка Azure AD B2C в качестве SAML IdP в приложении SAML

Последним шагом является включение Azure AD B2C в качестве SAML IdP в приложении SAML. Каждое приложение различается и действия различаются. Дополнительные сведения см. в документации приложения.

Метаданные можно настроить в приложении в виде *статических метаданных* или *динамических метаданных*. В статическом режиме скопируйте все или часть метаданных из метаданных политики Azure AD B2C. В динамическом режиме укажите URL-адрес метаданных и разрешите приложению динамически считывать метаданные.

Обычно требуются некоторые или все перечисленные ниже действия.

* **Метаданные**. Используйте формат `https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/Samlp/metadata` .
* **Issuer**: значение запроса SAML `issuer` должно совпадать с одним из URI, настроенных в `identifierUris` элементе манифеста регистрации приложения. Если имя запроса SAML `issuer` не существует в `identifierUris` элементе, [добавьте его в манифест регистрации приложения](#add-the-identifier). Например, `https://contoso.onmicrosoft.com/app-name`. 
* **URL-адрес входа/конечная точка SAML/URL-адрес SAML**. Проверьте значение в файле МЕТАДАННЫХ политики SAML Azure AD B2C для `<SingleSignOnService>` XML-элемента.
* **Сертификат**. этот сертификат *B2C_1A_SamlIdpCert*, но без закрытого ключа. Чтобы получить открытый ключ сертификата, выполните следующие действия.

    1. Перейдите по URL-адресу метаданных, указанному выше.
    1. Скопируйте значение в элемент `<X509Certificate>`.
    1. Вставьте его в текстовый файл.
    1. Сохраните текстовый файл под именем *.cer*.

### <a name="test-with-the-saml-test-app"></a>Тестирование с помощью тестового приложения SAML

Для тестирования конфигурации можно использовать наше [тестовое приложение SAML][samltest] .

* Обновите имя клиента.
* Обновите имя политики, например *B2C_1A_signup_signin_saml*.
* Укажите этот URI издателя. Используйте один из URI, найденных в `identifierUris` элементе в манифесте регистрации приложения, например `https://contoso.onmicrosoft.com/app-name` .

Выберите **Вход** — отобразится экран входа для пользователей. После входа в систему в пример приложения возвращается ответ SAML.

## <a name="supported-and-unsupported-saml-modalities"></a>Поддерживаемые и неподдерживаемые модальности SAML

Следующие сценарии приложений SAML поддерживаются с помощью собственной конечной точки метаданных:

* Несколько URL-адресов выхода или ОБРАТная привязка для URL-адреса выхода в объекте приложения или участника службы.
* Укажите ключ подписывания, чтобы проверить запросы проверяющей стороны (RP) в объекте приложения или субъекта-службы.
* Укажите ключ шифрования маркера в объекте приложения или субъекта-службы.
* Поставщик удостоверений — инициирован вход в систему, где Azure AD B2C поставщик удостоверений.

## <a name="next-steps"></a>Дальнейшие действия

- Получите тестовое веб-приложение SAML из [репозитория сообщества Azure AD B2C GitHub](https://github.com/azure-ad-b2c/saml-sp-tester).
- См. [Параметры регистрации приложения SAML в Azure AD B2C](saml-service-provider-options.md)

<!-- LINKS - External -->
[samltest]: https://aka.ms/samltestapp

::: zone-end
