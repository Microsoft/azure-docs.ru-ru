---
title: Настройка параметров поставщика службы SAML
title-suffix: Azure Active Directory B2C
description: Настройка параметров поставщика службы Azure Active Directory B2C SAML
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 09cfdd026105a34db976118f38b011e2c4578a24
ms.sourcegitcommit: 66ce33826d77416dc2e4ba5447eeb387705a6ae5
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/15/2021
ms.locfileid: "103470773"
---
# <a name="options-for-registering-a-saml-application-in-azure-ad-b2c"></a>Параметры регистрации приложения SAML в Azure AD B2C

В этой статье описываются параметры конфигурации, доступные при подключении Azure Active Directory (Azure AD B2C) к приложению SAML.

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="encrypted-saml-assertions"></a>Зашифрованные утверждения SAML

Если приложение предполагает, что утверждения SAML должны быть в зашифрованном формате, необходимо убедиться в том, что шифрование включено в политике Azure AD B2C.

Azure AD B2C использует сертификат открытого ключа поставщика услуг для шифрования утверждения SAML. Открытый ключ должен существовать в конечной точке метаданных приложения SAML с параметром Кэйдескриптор "use", имеющим значение "Encryption", как показано в следующем примере:

```xml
<KeyDescriptor use="encryption">
  <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
    <X509Data>
      <X509Certificate>valid certificate</X509Certificate>
    </X509Data>
  </KeyInfo>
</KeyDescriptor>
```

Чтобы разрешить Azure AD B2C отправку зашифрованных утверждений, задайте для элемента метаданных **вантсенкриптедассертион** значение `true` в [техническом профиле проверяющей](relyingparty.md#technicalprofile)стороны. Также можно настроить алгоритм, используемый для шифрования утверждения SAML.

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="WantsEncryptedAssertions">true</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

### <a name="encryption-method"></a>Метод шифрования

Чтобы настроить метод шифрования, используемый для шифрования данных утверждения SAML, установите `DataEncryptionMethod` ключ метаданных в пределах проверяющей стороны. Возможные значения: `Aes256` (по умолчанию), `Aes192` , `Sha512` или `Aes128` . Метаданные контролирует значение `<EncryptedData>` элемента в ответе SAML.

Чтобы настроить метод шифрования, используемый для шифрования копии ключа, который использовался для шифрования данных утверждения SAML, установите `KeyEncryptionMethod` ключ метаданных в пределах проверяющей стороны. Допустимые значения: `Rsa15` (по умолчанию) — алгоритм шифрования по стандарту криптографии для открытого ключа RSA (PKCS) версии 1,5, а `RsaOaep` -алгоритм (с оптимальным асимметричным шифрованием OAEP) RSA.  Метаданные контролирует значение  `<EncryptedKey>` элемента в ответе SAML.

В следующем примере показан `EncryptedAssertion` раздел утверждения SAML. Метод зашифрованных данных — `Aes128` , а зашифрованный метод ключа — `Rsa15` .

```xml
<saml:EncryptedAssertion>
  <xenc:EncryptedData xmlns:xenc="http://www.w3.org/2001/04/xmlenc#"
    xmlns:dsig="http://www.w3.org/2000/09/xmldsig#" Type="http://www.w3.org/2001/04/xmlenc#Element">
    <xenc:EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#aes128-cbc" />
    <dsig:KeyInfo>
      <xenc:EncryptedKey>
        <xenc:EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#rsa-1_5" />
        <xenc:CipherData>
          <xenc:CipherValue>...</xenc:CipherValue>
        </xenc:CipherData>
      </xenc:EncryptedKey>
    </dsig:KeyInfo>
    <xenc:CipherData>
      <xenc:CipherValue>...</xenc:CipherValue>
    </xenc:CipherData>
  </xenc:EncryptedData>
</saml:EncryptedAssertion>
```

Можно изменить формат зашифрованных утверждений. Чтобы настроить формат шифрования, установите `UseDetachedKeys` ключ метаданных в пределах проверяющей стороны. Возможные значения: `true` или `false` (по умолчанию). Если для этого параметра задано значение `true` , отсоединенные ключи добавляют зашифрованное утверждение в качестве дочернего элемента, а не объекта `EncrytedAssertion` `EncryptedData` .

Настройте метод шифрования и формат, используя ключи метаданных в [техническом профиле проверяющей](relyingparty.md#technicalprofile)стороны:

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="DataEncryptionMethod">Aes128</Item>
      <Item Key="KeyEncryptionMethod">Rsa15</Item>
      <Item Key="UseDetachedKeys">false</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

## <a name="identity-provider-initiated-flow"></a>Поставщик удостоверений — инициированный поток

Если приложение ожидает получения утверждения SAML без предварительной отправки запроса SAML AuthN поставщику удостоверений, необходимо настроить Azure AD B2C для потока, инициированного поставщиком удостоверений.

В потоке, инициированном поставщиком удостоверений, процесс входа инициируется поставщиком удостоверений (Azure AD B2C), который отправляет незапрошенный ответ SAML поставщику услуг (приложение проверяющей стороны).

В настоящее время мы не поддерживаем сценарии, в которых инициирующий поставщик удостоверений является внешним поставщиком удостоверений, объединенным с Azure AD B2C, например [AD-FS](identity-provider-adfs.md)или [Salesforce](identity-provider-salesforce-saml.md). Она поддерживается только для Azure AD B2C проверки подлинности локальной учетной записи.

Чтобы включить поток, инициированный поставщиком удостоверений, задайте для элемента метаданных **идпинитиатедпрофилинаблед** значение `true` в [техническом профиле проверяющей](relyingparty.md#technicalprofile)стороны.

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="IdpInitiatedProfileEnabled">true</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

Чтобы войти или зарегистрировать пользователя с помощью потока, инициированного поставщиком удостоверений, используйте следующий URL-адрес:

```
https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/generic/login?EntityId=app-identifier-uri 
```

Измените следующие значения:

* **имя клиента** с именем клиента
* **Политика — имя** с именем политики проверяющей стороны SAML.
* **app-identifier-URI** с параметром `identifierUris` в файле метаданных, например `https://contoso.onmicrosoft.com/app-name`

### <a name="sample-policy"></a>Пример политики

Мы предоставляем полный пример политики, который можно использовать для тестирования с помощью тестового приложения SAML.

1. Скачайте [Пример политики для входа с помощью SAML-SP-инициируемые](https://github.com/azure-ad-b2c/saml-sp/tree/master/policy/SAML-SP-Initiated).
1. Обновите `TenantId` в соответствии с именем клиента, например *contoso.b2clogin.com*.
1. Не заключайте имя политики в *B2C_1A_signup_signin_saml*.

## <a name="saml-response-signature-algorithm"></a>Алгоритм подписи ответа SAML

Вы можете настроить алгоритм подписи, используемый для подписания утверждения SAML. Возможные значения: `Sha256`, `Sha384`, `Sha512` или `Sha1`. Убедитесь, что в техническом профиле и приложении используется один и тот же алгоритм подписи. Используйте только тот алгоритм, который поддерживается вашим сертификатом.

Настройте алгоритм подписи с помощью `XmlSignatureAlgorithm` ключа метаданных в элементе метаданных проверяющей стороны.

```xml
<RelyingParty>
  <DefaultUserJourney ReferenceId="SignUpOrSignIn" />
  <TechnicalProfile Id="PolicyProfile">
    <DisplayName>PolicyProfile</DisplayName>
    <Protocol Name="SAML2"/>
    <Metadata>
      <Item Key="XmlSignatureAlgorithm">Sha256</Item>
    </Metadata>
   ..
  </TechnicalProfile>
</RelyingParty>
```

## <a name="saml-response-lifetime"></a>Время существования ответа SAML

Можно настроить период времени, в течение которого ответ SAML остается действительным. Задайте время существования с помощью `TokenLifeTimeInSeconds` элемента метаданных в техническом профиле издателя маркера SAML. Это значение равно количеству секунд, которое может пройти из `NotBefore` метки времени, вычисленной на момент выдачи маркера. Время жизни по умолчанию составляет 300 секунд (5 минут).

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <Item Key="TokenLifeTimeInSeconds">400</Item>
      </Metadata>
      ...
    </TechnicalProfile>
```

## <a name="saml-response-valid-from-skew"></a>Ответ SAML действителен от наклона

Можно настроить отклонения времени, применяемые к метке времени ответа SAML `NotBefore` . Такая конфигурация гарантирует, что если время между двумя платформами не синхронизировано, утверждение SAML будет по-прежнему считаться действительным в течение этого времени.

Задайте отклонения времени с помощью `TokenNotBeforeSkewInSeconds` элемента метаданных в техническом профиле издателя маркера SAML. Значение смещения задается в секундах со значением по умолчанию 0. Максимальное значение — 3600 (один час).

Например, если установлено `TokenNotBeforeSkewInSeconds` значение `120` seconds:

- Маркер выдается в 13:05:10 UTC
- Токен действителен с 13:03:10 UTC

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <Item Key="TokenNotBeforeSkewInSeconds">120</Item>
      </Metadata>
      ...
    </TechnicalProfile>
```

## <a name="remove-milliseconds-from-date-and-time"></a>Удалить миллисекунды из даты и времени

Можно указать, будут ли миллисекунды удаляться из значений типа DateTime в ответе SAML (включая Иссуеинстант, NotBefore, Нотонорафтер и Ауснинстант). Чтобы удалить миллисекунды, установите `RemoveMillisecondsFromDateTime
` ключ метаданных в пределах проверяющей стороны. Возможные значения: `false` (по умолчанию) или `true` .

```xml
<ClaimsProvider>
  <DisplayName>Token Issuer</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="Saml2AssertionIssuer">
      <DisplayName>Token Issuer</DisplayName>
      <Protocol Name="SAML2"/>
      <OutputTokenFormat>SAML2</OutputTokenFormat>
      <Metadata>
        <Item Key="RemoveMillisecondsFromDateTime">true</Item>
      </Metadata>
      ...
    </TechnicalProfile>
```

## <a name="azure-ad-b2c-issuer-id"></a>Идентификатор издателя Azure AD B2C

При наличии нескольких приложений SAML, зависящих от разных `entityID` значений, можно переопределить `issueruri` значение в файле проверяющей стороны. Чтобы переопределить URI издателя, скопируйте технический профиль с ИДЕНТИФИКАТОРом "Saml2AssertionIssuer" из базового файла и переопределите `issueruri` значение.

> [!TIP]
> Скопируйте `<ClaimsProviders>` раздел из базы данных и сохраните эти элементы в поставщике утверждений: `<DisplayName>Token Issuer</DisplayName>` , `<TechnicalProfile Id="Saml2AssertionIssuer">` и `<DisplayName>Token Issuer</DisplayName>` .
 
Пример

```xml
   <ClaimsProviders>   
    <ClaimsProvider>
      <DisplayName>Token Issuer</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Saml2AssertionIssuer">
          <DisplayName>Token Issuer</DisplayName>
          <Metadata>
            <Item Key="IssuerUri">customURI</Item>
          </Metadata>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
  </ClaimsProviders>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpInSAML" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
     …
```

## <a name="session-management"></a>Управление сеансом

Вы можете управлять сеансом между Azure AD B2C и приложением проверяющей стороны SAML с помощью `UseTechnicalProfileForSessionManagement` элемента и [самлссосессионпровидер](custom-policy-reference-sso.md#samlssosessionprovider).

## <a name="force-users-to-re-authenticate"></a>Принудительная повторная проверка подлинности пользователей 

Чтобы принудительно выполнить повторную проверку подлинности пользователей, приложение может включить `ForceAuthn` атрибут в запрос проверки подлинности SAML. `ForceAuthn`Атрибут является логическим значением. Если задано значение true, сеанс пользователя становится недействительным на Azure AD B2C, и пользователь вынужден пройти повторную проверку подлинности. Следующий запрос проверки подлинности SAML показывает, как задать `ForceAuthn` для атрибута значение true. 


```xml
<samlp:AuthnRequest 
       Destination="https://contoso.b2clogin.com/contoso.onmicrosoft.com/B2C_1A_SAML2_signup_signin/samlp/sso/login"
       ForceAuthn="true" ...>
    ...
</samlp:AuthnRequest>
```

## <a name="debug-the-saml-protocol"></a>Отладка протокола SAML

Чтобы настроить и отладить интеграцию с поставщиком услуг, можно использовать расширение браузера для протокола SAML, например [SAML DevTools Extension](https://chrome.google.com/webstore/detail/saml-devtools-extension/jndllhgbinhiiddokbeoeepbppdnhhio) для Chrome, [SAML-трассировщик](https://addons.mozilla.org/es/firefox/addon/saml-tracer/) для Firefox или [средства разработчика IE](https://techcommunity.microsoft.com/t5/microsoft-sharepoint-blog/gathering-a-saml-token-using-edge-or-ie-developer-tools/ba-p/320957).

С помощью этих средств можно проверить интеграцию приложения с Azure AD B2C. Пример:

* Проверьте, содержит ли запрос SAML подпись, и определите, какой алгоритм используется для входа в запрос авторизации.
* Проверьте, возвращает ли Azure AD B2C сообщение об ошибке.
* Проверьте, что раздел утверждения зашифрован.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [протоколе SAML см. на веб-сайте Oasis](https://www.oasis-open.org/).
- Получите тестовое веб-приложение SAML из [репозитория сообщества Azure AD B2C GitHub](https://github.com/azure-ad-b2c/saml-sp-tester).

<!-- LINKS - External -->
[samltest]: https://aka.ms/samltestapp

::: zone-end
