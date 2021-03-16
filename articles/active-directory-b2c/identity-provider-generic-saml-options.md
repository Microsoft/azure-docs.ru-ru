---
title: Настройка входа с помощью параметров поставщика удостоверений SAML
titleSuffix: Azure Active Directory B2C
description: Настройка параметров поставщика удостоверений SAML для входа (IdP) в Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/15/2021
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 43c57950d317de42df666ddd25cbcb2e9a4c9611
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103488879"
---
# <a name="configure-saml-identity-provider-options-with-azure-active-directory-b2c"></a>Настройка параметров поставщика удостоверений SAML с помощью Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) поддерживает федерацию с поставщиками удостоверений SAML 2,0. В этой статье описываются параметры конфигурации, доступные при включении входа с помощью поставщика удостоверений SAML.

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

::: zone pivot="b2c-user-flow"

[!INCLUDE [active-directory-b2c-limited-to-custom-policy](../../includes/active-directory-b2c-limited-to-custom-policy.md)]

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="claims-mapping"></a>Сопоставление утверждений

Элемент **OutputClaims** содержит список заявок, возвращенных поставщиком удостоверений SAML. Необходимо соотнести имя утверждения, определенного в политике, с именем, определенным в поставщике удостоверений. В поставщике удостоверений проверьте список утверждений (утверждений). Вы также можете проверить содержимое ответа SAML, возвращаемого поставщиком удостоверений. Дополнительные сведения см. [в разделе Отладка сообщений SAML](#debug-saml-protocol). Чтобы добавить утверждение, сначала [Определите утверждение](claimsschema.md), а затем добавьте утверждение в коллекцию исходящих утверждений.

Вы также можете добавить утверждения, которые не возвращаются поставщиком удостоверений, установив атрибут `DefaultValue`. Значение по умолчанию может быть статическим или динамическим, используя [утверждения контекста](#enable-use-of-context-claims).

Элемент исходящего утверждения содержит следующие атрибуты:

- **ClaimTypeReferenceId** — это ссылка на тип утверждения. 
- **Партнерклаимтипе** — имя свойства, которое отображается в утверждении SAML. 
- **DefaultValue** — это стандартное значение по умолчанию. Если утверждение пустое, будет использоваться значение по умолчанию. Можно также использовать [арбитры утверждений](claim-resolver-overview.md) с контекстным значением, например идентификатор корреляции или IP-адрес пользователя.

### <a name="subject-name"></a>Имя субъекта

Чтобы считать утверждение SAML **NameId** в поле **subject** как нормализованное утверждение, задайте для утверждения **партнерклаимтипе** значение `SPNameQualifier` атрибута. Если `SPNameQualifier` атрибут не представлен, установите для утверждения **партнерклаимтипе** значение `NameQualifier` атрибута.

Утверждение SAML:

```xml
<saml:Subject>
  <saml:NameID SPNameQualifier="http://your-idp.com/unique-identifier" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient">david@contoso.com</saml:NameID>
  <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
    <SubjectConfirmationData InResponseTo="_cd37c3f2-6875-4308-a9db-ce2cf187f4d1" NotOnOrAfter="2020-02-15T16:23:23.137Z" Recipient="https://<your-tenant>.b2clogin.com/<your-tenant>.onmicrosoft.com/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer" />
    </SubjectConfirmation>
  </saml:SubjectConfirmation>
</saml:Subject>
```

Исходящее утверждение:

```xml
<OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="http://your-idp.com/unique-identifier" />
```

Если `SPNameQualifier` атрибуты или `NameQualifier` не представлены в утверждении SAML, задайте для утверждения **партнерклаимтипе** значение `assertionSubjectName` . Убедитесь, что **NameId** является первым значением в XML утверждения. При определении нескольких утверждений Azure AD B2C выбирает значения субъекта из последнего утверждения.


## <a name="configure-saml-protocol-bindings"></a>Настройка привязок протокола SAML

Запросы SAML отправляются поставщику удостоверений, как указано в элементе метаданных поставщика удостоверений `SingleSignOnService` . Большинство запросов авторизации поставщиков удостоверений переносятся непосредственно в строку запроса URL-адреса запроса HTTP GET (так как сообщения относительно короткие). Сведения о настройке привязок для обоих запросов SAML см. в документации поставщика удостоверений.

Ниже приведен пример службы единого входа Azure AD с двумя привязками. `HTTP-Redirect`Имеет приоритет над, `HTTP-POST` так как он отображается первым в метаданных поставщика удостоверений SAML.

```xml
<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
  ...
  <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/saml2"/>
  <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/saml2"/>
</IDPSSODescriptor>
```

Ответы SAML передаются в Azure AD B2C через привязку HTTP POST. Azure AD B2C метаданных политики задает `AssertionConsumerService` привязку к `urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST` .

Ниже приведен пример элемента службы утверждения метаданных политики Azure AD B2C.

```xml
<SPSSODescriptor AuthnRequestsSigned="true" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
  ...
  <AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://your-tenant.b2clogin.com/your-tenant/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer" index="0" isDefault="true"/>
</SPSSODescriptor>
```

## <a name="configure-the-saml-request-signature"></a>Настройка подписи запроса SAML

Azure AD B2C подписывает все исходящие запросы проверки подлинности с помощью криптографического ключа **самлмессажесигнинг** . Чтобы отключить подпись запроса SAML, установите для параметра **вантссигнедрекуестс** значение `false` . Если для метаданных задано значение `false` , параметры **SigAlg** и **Signature** (строка запроса или параметр POST) пропускаются из запроса.

Эти метаданные также контролирует атрибут **ауснрекуестссигнед** , который включается в метаданные Azure AD B2C технического профиля, который является общим для поставщика удостоверений. Azure AD B2C не подписывает запрос, если параметр **вантссигнедрекуестс** в метаданных технического профиля имеет значение `false` , а **вантауснрекуестссигнед** метаданных поставщика удостоверений имеет значение `false` или не указано.

В следующем примере удаляется подпись из запроса SAML.

```xml
<Metadata>
  ...
  <Item Key="WantsSignedRequests">false</Item>
</Metadata>
```

### <a name="signature-algorithm"></a>Алгоритм подписи

Azure AD B2C использует `Sha1` для подписания запроса SAML. Используйте метаданные **ксмлсигнатуреалгорисм** , чтобы настроить используемый алгоритм. Возможные значения: `Sha256` , `Sha384` , `Sha512` или `Sha1` (по умолчанию). Эти метаданные определяют значение параметра **SigAlg** (строка запроса или параметр отправки) в запросе SAML. Настройте алгоритм подписи на обеих сторонах, используя одно и то же значение. Используйте только тот алгоритм, который поддерживается вашим сертификатом.

### <a name="include-key-info"></a>Включить сведения о ключе

Если поставщик удостоверений указывает, что Azure AD B2Cная привязка имеет значение `HTTP-POST` , Azure AD B2C включает подпись и алгоритм в тексте запроса SAML. Вы также можете настроить в Azure AD включение открытого ключа сертификата, если привязка имеет значение `HTTP-POST` . Используйте метаданные **инклудекэйинфо** в `true` , или `false` . В следующем примере Azure AD не будет включать открытый ключ сертификата.

```xml
<Metadata>
  ...
  <Item Key="IncludeKeyInfo">false</Item>
</Metadata>
```

## <a name="configure-saml-request-name-id"></a>Настройка идентификатора имени запроса SAML

Элемент запроса авторизации SAML `<NameID>` указывает формат идентификатора имени SAML. В этом разделе описывается конфигурация по умолчанию и настройка элемента идентификатора имени.

## <a name="preferred-username"></a>Предпочитаемое имя пользователя

Во время входа пользователя проверяющей стороны приложение может ориентироваться на конкретного пользователя. Azure AD B2C позволяет отправить поставщику удостоверений SAML предпочитаемое имя пользователя. Элемент **inputclaim** используется для отправки **NameId** внутри **субъекта** запроса авторизации SAML.

Чтобы включить идентификатор имени субъекта в запрос авторизации, добавьте следующий `<InputClaims>` элемент сразу после `<CryptographicKeys>` . **Партнерклаимтипе** должен иметь значение `subject` .

```xml
<InputClaims>
  <InputClaim ClaimTypeReferenceId="signInName" PartnerClaimType="subject" />
</InputClaims>
```

В этом примере Azure AD B2C отправляет значение утверждения **signInName** с **NameId** в **субъекте** запроса авторизации SAML.

```xml
<samlp:AuthnRequest ... >
  ...
  <saml:Subject>
    <saml:NameID>sam@contoso.com</saml:NameID>
  </saml:Subject>
</samlp:AuthnRequest>
```

Можно использовать [утверждения контекста](claim-resolver-overview.md), например `{OIDC:LoginHint}` для заполнения значения утверждения.

```xml
<Metadata>
  ...
  <Item Key="IncludeClaimResolvingInClaimsHandling">true</Item>
</Metadata>
  ...
<InputClaims>
  <InputClaim ClaimTypeReferenceId="signInName" PartnerClaimType="subject" DefaultValue="{OIDC:LoginHint}" AlwaysUseDefaultValue="true" />
</InputClaims>
```

### <a name="name-id-policy-format"></a>Идентификатор имени в формате политики

По умолчанию в запросе авторизации SAML указана `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified` политика. Это означает, что можно использовать любой тип идентификатора, поддерживаемый поставщиком удостоверений для запрошенной темы.

Чтобы изменить это поведение, обратитесь к документации поставщика удостоверений за инструкциями о том, какие политики ИДЕНТИФИКАТОРов имен поддерживаются. Затем добавьте `NameIdPolicyFormat` метаданные в соответствующий формат политики. Пример:

```xml
<Metadata>
  ...
  <Item Key="NameIdPolicyFormat">urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</Item>
</Metadata>
```

Следующий запрос авторизации SAML содержит политику идентификатора имени.

```xml
<samlp:AuthnRequest ... >
  ...
  <samlp:NameIDPolicy Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress" />
</samlp:AuthnRequest>
```

### <a name="allow-creating-new-accounts"></a>Разрешить создание новых учетных записей

Если указать [Формат имени политики ID](#name-id-policy-format), можно также указать `AllowCreate` свойство **NameIDPolicy** , чтобы указать, разрешено ли поставщику удостоверений создавать новую учетную запись во время потока входа. Инструкции см. в документации поставщика удостоверений.

`AllowCreate`По умолчанию свойство пропускается Azure AD B2C. Это поведение можно изменить с помощью `NameIdPolicyAllowCreate` метаданных. Значение этих метаданных равно `true` или `false` .

В следующем примере показано, как задать `AllowCreate` для свойства `NameIDPolicy` значение `true` .

```xml
<Metadata>
  ...
  <Item Key="NameIdPolicyFormat">urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</Item>
  <Item Key="NameIdPolicyAllowCreate">true</Item>
</Metadata>
```


В следующем примере демонстрируется запрос авторизации с **алловкреате** элемента **NameIDPolicy** в запросе авторизации.


```xml
<samlp:AuthnRequest ... >
  ...
  <samlp:NameIDPolicy 
      Format="urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress" 
      AllowCreate="true" />
</samlp:AuthnRequest>
```

### <a name="include-authentication-context-class-references"></a>Включить ссылки на классы контекста проверки подлинности

Запрос авторизации SAML может содержать элемент **AuthnContext** , указывающий контекст запроса авторизации. Элемент может содержать ссылку на класс контекста проверки подлинности, который сообщает поставщику удостоверений SAML, какой механизм проверки подлинности должен предоставлять пользователю.

Чтобы настроить ссылки на классы контекста проверки подлинности, добавьте метаданные **инклудеауснконтекстклассреференцес** . В поле значение укажите одну или несколько ссылок URI, определяющих классы контекста проверки подлинности. Укажите несколько URI в качестве списка с разделителями-запятыми. Дополнительные сведения о поддерживаемых URI **AuthnContextClassRef** см. в документации поставщика удостоверений.

В следующем примере пользователи могут выполнять вход с использованием имени пользователя и пароля, а также выполнять вход с использованием имени пользователя и пароля в защищенном сеансе (SSL/TLS).

```xml
<Metadata>
  ...
  <Item Key="IncludeAuthnContextClassReferences">urn:oasis:names:tc:SAML:2.0:ac:classes:Password,urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</Item>
</Metadata>
```

Следующий запрос авторизации SAML содержит ссылки на классы контекста проверки подлинности.

```xml
<samlp:AuthnRequest ... >
  ...
  <samlp:RequestedAuthnContext>
    <saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:Password</saml:AuthnContextClassRef>
    <saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</saml:AuthnContextClassRef>
  </samlp:RequestedAuthnContext>
</samlp:AuthnRequest>
```

## <a name="include-custom-data-in-the-authorization-request"></a>Включить пользовательские данные в запрос авторизации

При необходимости можно включить элементы расширения сообщений протокола, которые согласовываются как с помощью Azure AD BC, так и с поставщиком удостоверений. Расширение представлено в формате XML. Элементы расширения включаются путем добавления XML-данных в элемент CDATA `<![CDATA[Your IDP metadata]]>` . Сведения о поддержке элемента расширений см. в документации к поставщику удостоверений.

В следующем примере показано использование данных расширения.

```xml
<Metadata>
  ...
  <Item Key="AuthenticationRequestExtensions"><![CDATA[
            <ext:MyCustom xmlns:ext="urn:ext:custom">
              <ext:AssuranceLevel>1</ext:AssuranceLevel>
              <ext:AssuranceDescription>Identity verified to level 1.</ext:AssuranceDescription>
            </ext:MyCustom>]]></Item>
</Metadata>
```

При использовании расширения сообщения протокола SAML ответ SAML будет выглядеть, как в следующем примере:

```xml
<samlp:AuthnRequest ... >
  ...
  <samlp:Extensions>
    <ext:MyCustom xmlns:ext="urn:ext:custom">
      <ext:AssuranceLevel>1</ext:AssuranceLevel>
      <ext:AssuranceDescription>Identity verified to level 1.</ext:AssuranceDescription>
    </ext:MyCustom>
  </samlp:Extensions>
</samlp:AuthnRequest>
```

## <a name="require-signed-saml-responses"></a>Требовать подписанные ответы SAML

Azure AD B2C требует, чтобы все входящие утверждения были подписаны. Это требование можно удалить, задав для **вантссигнедассертионс** значение `false` . Поставщик удостоверений не должен подписывать утверждения в этом случае, но даже если это так, Azure AD B2C не будет проверять подпись.

Метаданные **вантссигнедассертионс** контролирует флаг метаданных SAML **вантассертионссигнед**, который включен в метаданные Azure AD B2C технического профиля, который является общим для поставщика удостоверений.

```xml
<SPSSODescriptor AuthnRequestsSigned="true" WantAssertionsSigned="true" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
```

При отключении проверки утверждений может также потребоваться отключить проверку подписи ответного сообщения. Задайте для метаданных **респонсессигнед** значение `false` . В этом случае поставщик удостоверений не должен подписывать ответное сообщение SAML, но даже если это так, Azure AD B2C не будет проверять подпись.

В следующем примере удаляются как сообщение, так и подпись утверждения:

```xml
<Metadata>
  ...
  <Item Key="WantsSignedAssertions">false</Item>
  <Item Key="ResponsesSigned">false</Item>
</Metadata>
```

## <a name="require-encrypted-saml-responses"></a>Требовать зашифрованные ответы SAML

Чтобы включить шифрование всех входящих утверждений, установите метаданные **вантсенкриптедассертионс** . Если требуется шифрование, поставщик удостоверений использует открытый ключ сертификата шифрования в Azure AD B2C техническом профиле. Azure AD B2C расшифровывает утверждение ответа с помощью закрытой части сертификата шифрования.

Если включить шифрование утверждений, также может потребоваться отключить проверку подписи ответа (Дополнительные сведения см. в разделе [требование подписанных ответов SAML](#require-signed-saml-responses).

Если для метаданных **вантсенкриптедассертионс** задано значение `true` , метаданные Azure AD B2C технического профиля включают раздел **ENCRYPTION** . Поставщик удостоверений считывает метаданные и шифрует утверждение ответа SAML с помощью открытого ключа, указанного в метаданных технического профиля Azure AD B2C.

В следующем примере показан раздел дескриптора ключа метаданных SAML, используемый для шифрования:

```xml
<SPSSODescriptor AuthnRequestsSigned="true"  protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
  <KeyDescriptor use="encryption">
    <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
      <X509Data>
        <X509Certificate>valid certificate</X509Certificate>
      </X509Data>
    </KeyInfo>
  </KeyDescriptor>
  ...
</SPSSODescriptor>
```

Шифрование утверждения ответа SAML:

1. [Создайте ключ политики](identity-provider-generic-saml.md#create-a-policy-key) с уникальным идентификатором. Например, `B2C_1A_SAMLEncryptionCert`.
2. В наборе **криптографиккэйс** в техническом профиле SAML. Задайте **идентификатором storagereferenceid** в качестве имени ключа политики, созданного на первом шаге. `SamlAssertionDecryption`Идентификатор указывает на использование криптографического ключа для шифрования и расшифровки утверждения SAML Response.

    ```xml
    <CryptographicKeys>
            ...
      <Key Id="SamlAssertionDecryption" StorageReferenceId="B2C_1A_SAMLEncryptionCert"/>
    </CryptographicKeys>
    ```

3. Задайте для метаданных технического профиля **WantsEncryptedAssertions** значение `true`.
    
    ```xml
    <Metadata>
      ...
      <Item Key="WantsEncryptedAssertions">true</Item>
    </Metadata>
    ```

4. Обновите поставщик удостоверений, указав новые Azure AD B2C метаданные технического профиля. В параметре **KeyDescriptor** свойству **use** должно быть присвоено значение `encryption`, содержащее открытый ключ сертификата.

## <a name="enable-use-of-context-claims"></a>Разрешить использование контекстных утверждений

В коллекции утверждений входных и выходных данных можно включать утверждения, которые не возвращаются поставщиком удостоверений, при условии, что задан `DefaultValue` атрибут. Можно также использовать [утверждения контекста](claim-resolver-overview.md) для включения в технический профиль. Использование утверждения контекста:

1. Добавьте тип утверждения в элемент [ClaimsSchema](claimsschema.md) в [BuildingBlocks](buildingblocks.md).
2. Добавьте выходное утверждение в входную или выходную коллекцию. В следующем примере первое утверждение задает значение поставщика удостоверений. Второе утверждение использует [утверждения контекста](claim-resolver-overview.md)IP-адреса пользователя.
    
    ```xml
    <OutputClaims>
      <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="contoso.com" AlwaysUseDefaultValue="true" />
      <OutputClaim ClaimTypeReferenceId="IpAddress" DefaultValue="{Context:IPAddress}" AlwaysUseDefaultValue="true" />
    </OutputClaims>
    ```

## <a name="disable-single-logout"></a>Отключение единого выхода

При запросе выхода из приложения Azure AD B2C пытается выйти из поставщика удостоверений SAML. Дополнительные сведения см. в разделе [Azure AD B2C выхода из сеанса](session-behavior.md#sign-out).  Чтобы отключить режим единого выхода, задайте для метаданных **синглелогаутенаблед** значение `false` .

```xml
<Metadata>
  ...
  <Item Key="SingleLogoutEnabled">false</Item>
</Metadata>
```

## <a name="debug-saml-protocol"></a>Протокол SAML Debug

Чтобы настроить и отладить Федерацию с помощью поставщика удостоверений SAML, можно использовать расширение браузера для протокола SAML, например [SAML DevTools Extension](https://chrome.google.com/webstore/detail/saml-devtools-extension/jndllhgbinhiiddokbeoeepbppdnhhio) для Chrome, [SAML-трассировщик](https://addons.mozilla.org/es/firefox/addon/saml-tracer/) для Firefox или [средства разработчика IE](https://techcommunity.microsoft.com/t5/microsoft-sharepoint-blog/gathering-a-saml-token-using-edge-or-ie-developer-tools/ba-p/320957).

С помощью этих средств можно проверить интеграцию между Azure AD B2C и поставщиком удостоверений SAML. Пример:

* Проверьте, содержит ли запрос SAML подпись, и определите, какой алгоритм используется для входа в запрос авторизации.
* Получение утверждений (утверждений) в `AttributeStatement` разделе.
* Проверьте, возвращает ли поставщик удостоверений сообщение об ошибке.
* Проверьте, зашифрован ли раздел утверждения.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как диагностировать проблемы с пользовательскими политиками с помощью [Application Insights](troubleshoot-with-application-insights.md). 

::: zone-end
