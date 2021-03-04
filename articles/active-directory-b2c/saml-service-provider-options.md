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
ms.date: 03/03/2021
ms.author: mimart
ms.subservice: B2C
ms.custom: fasttrack-edit
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: b9a491b639cd1b960ffe3b7164a0940770792148
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102107778"
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

Настройте алгоритм подписи с помощью `XmlSignatureAlgorithm` ключа метаданных в узле метаданных релингпарти.

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

Можно настроить период времени, в течение которого ответ SAML остается действительным. Задайте время существования с помощью `TokenLifeTimeInSeconds` элемента метаданных в техническом профиле издателя маркера SAML. Это значение равно количеству секунд, которое может пройти из `NotBefore` метки времени, вычисленной на момент выдачи маркера. Время, выбранное для текущего времени, будет автоматически. Время жизни по умолчанию составляет 300 секунд (5 минут).

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
