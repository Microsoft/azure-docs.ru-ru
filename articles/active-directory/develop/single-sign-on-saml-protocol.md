---
title: Протокол единого входа SAML в Azure
titleSuffix: Microsoft identity platform
description: В этой статье описывается протокол единого входа SAML в Azure Active Directory
services: active-directory
documentationcenter: .net
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 05/18/2020
ms.author: kenwith
ms.custom: aaddev
ms.reviewer: paulgarn
ms.openlocfilehash: 40bf202e0f14f18d817e4e918f8372ba3c0a4ad8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91950675"
---
# <a name="single-sign-on-saml-protocol"></a>Протокол единого входа SAML

В этой статье рассматриваются запросы и ответы аутентификации SAML 2.0, которые поддерживаются Azure Active Directory (Azure AD) для единого входа.

На схеме протокола ниже описана последовательность единого входа. Облачная служба (поставщик услуг) использует привязку перенаправления HTTP для передачи `AuthnRequest` (запроса проверки подлинности) в Azure AD (поставщику удостоверений). Затем Azure AD использует привязку HTTP POST, чтобы опубликовать элемент `Response` в облачной службе.

![Рабочий процесс единого входа (SSO)](./media/single-sign-on-saml-protocol/active-directory-saml-single-sign-on-workflow.png)

> [!NOTE]
> В этой статье рассматривается использование SAML для единого входа. Дополнительные сведения о других способах управления единым входом (например, с помощью OpenID Connect или Встроенной проверки подлинности Windows) см. в статье [Единый вход в приложения в Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="authnrequest"></a>AuthnRequest

Чтобы запросить проверку подлинности пользователя, облачные службы отправляют элемент `AuthnRequest` в Azure AD. Пример SAML 2.0 `AuthnRequest` должен выглядеть примерно так:

```
<samlp:AuthnRequest
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="id6c1c178c166d486687be4aaf5e482730"
Version="2.0" IssueInstant="2013-03-18T03:28:54.1839884Z"
xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
</samlp:AuthnRequest>
```

| Параметр | Type | Описание |
| --- | --- | --- |
| ID | Обязательно | Azure AD использует этот атрибут для заполнения атрибута `InResponseTo` возвращенного ответа. Идентификатор не должен начинаться с цифры, поэтому общая стратегия предусматривает добавление такой строки, как id, в начало строкового представления GUID. Например, `id6c1c178c166d486687be4aaf5e482730` — допустимый идентификатор. |
| Версия | Обязательно | Этот параметр должен иметь значение **2.0**. |
| IssueInstant | Обязательно | Это строка DateTime со значением в формате всемирного времени (UTC) и с [преобразованием без потери данных ("o")](/dotnet/standard/base-types/standard-date-and-time-format-strings). Azure AD ожидает значение DateTime этого типа, но не оценивает и не использует его. |
| AssertionConsumerServiceURL | Необязательно | Если указан, то он должен соответствовать параметру `RedirectUri` облачной службы в Azure AD. |
| ForceAuthn | Необязательно | Это логическое значение. Если задано значение true, то это означает, что пользователь должен будет повторно выполнить проверку подлинности, даже если время его сеанса в Azure AD еще не истекло. |
| IsPassive | Необязательно | Это логическое значение, которое указывает, должна ли служба Azure AD автоматически выполнять аутентификацию пользователя, без прямого его участия, используя файл cookie сеанса (если он существует). Если задано значение true, то Azure AD попытается аутентифицировать пользователя с помощью файла cookie сеанса. |

Все остальные атрибуты `AuthnRequest`, включая Consent, Destination, AssertionConsumerServiceIndex, AttributeConsumerServiceIndex и ProviderName, **игнорируются**.

Azure AD также игнорирует элемент `Conditions` в `AuthnRequest`.

### <a name="issuer"></a>Издатель

Элемент `Issuer` в `AuthnRequest` должен точно соответствовать одному из имен в списке **ServicePrincipalNames** для облачной службы в Azure AD. Обычно здесь передается **URI идентификатора приложения** , указанный во время регистрации приложения.

Фрагмент SAML, в котором содержится элемент `Issuer`, выглядит так:

```
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
```

### <a name="nameidpolicy"></a>NameIDPolicy

Этот элемент запрашивает определенный формат идентификатора имени в ответе и является необязательным в элементах `AuthnRequest` , отправленных в Azure AD.

Элемент `NameIdPolicy` выглядит так:

```
<NameIDPolicy Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"/>
```

Если элемент `NameIDPolicy` указан, можно включить его необязательный атрибут `Format`. У атрибута `Format` может быть только одно из следующих значений, так как любое другое значение приводит к ошибке.

* `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`: Azure Active Directory выдает утверждение NameID в качестве парного идентификатора.
* `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`: Azure Active Directory выдает утверждение NameID в формате адреса электронной почты.
* `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`: это значение позволяет Azure Active Directory выбирать формат утверждений. Azure Active Directory выдает NameID в качестве парного идентификатора.
* `urn:oasis:names:tc:SAML:2.0:nameid-format:transient`: Azure Active Directory выдает случайным образом созданное значение утверждения NameID, которое является уникальным для текущей операции единого входа. Это означает, что значение является временным и не может использоваться для идентификации пользователя, выполняющего аутентификацию.

Если `SPNameQualifier` указан параметр, Azure AD будет включать в ответ то же самое `SPNameQualifier` .

Azure AD игнорирует атрибут `AllowCreate` .

### <a name="requestauthncontext"></a>RequestAuthnContext
Элемент `RequestedAuthnContext` указывает нужные методы проверки подлинности. Он необязателен в элементах `AuthnRequest` , отправленных в Azure AD. Azure AD поддерживает значения `AuthnContextClassRef`, такие как `urn:oasis:names:tc:SAML:2.0:ac:classes:Password`.

### <a name="scoping"></a>Scoping
Элемент `Scoping`, который включает список поставщиков удостоверений, необязателен в элементах `AuthnRequest`, отправленных в Azure AD.

Если он указан, не включайте атрибут `ProxyCount`, а также элемент `IDPListOption` или `RequesterID`, так как они не поддерживаются.

### <a name="signature"></a>Сигнатура
`Signature`Элемент в `AuthnRequest` элементах является необязательным. Azure AD не проверяет подписанные запросы проверки подлинности при наличии подписи. Проверка запрашивающей стороны предоставляется только в ответах для зарегистрированных URL-адресов службы обработчика утверждений.

### <a name="subject"></a>Тема
Не включайте элемент `Subject`. Azure AD не поддерживает указание темы для запроса и возвращает ошибку, если она указана.

## <a name="response"></a>Ответ
После успешного выполнения запрошенного входа Azure AD отправляет ответ в облачную службу. Ответ на успешную попытку входа может выглядеть следующим образом:

```
<samlp:Response ID="_a4958bfd-e107-4e67-b06d-0d85ade2e76a" Version="2.0" IssueInstant="2013-03-18T07:38:15.144Z" Destination="https://contoso.com/identity/inboundsso.aspx" InResponseTo="id758d0ef385634593a77bdf7e632984b6" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
    ...
  </ds:Signature>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
  </samlp:Status>
  <Assertion ID="_bf9c623d-cc20-407a-9a59-c2d0aee84d12" IssueInstant="2013-03-18T07:38:15.144Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">
    <Issuer>https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
    <ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
      ...
    </ds:Signature>
    <Subject>
      <NameID>Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="id758d0ef385634593a77bdf7e632984b6" NotOnOrAfter="2013-03-18T07:43:15.144Z" Recipient="https://contoso.com/identity/inboundsso.aspx" />
      </SubjectConfirmation>
    </Subject>
    <Conditions NotBefore="2013-03-18T07:38:15.128Z" NotOnOrAfter="2013-03-18T08:48:15.128Z">
      <AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
      </AudienceRestriction>
    </Conditions>
    <AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <AttributeValue>testuser@contoso.com</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
        <AttributeValue>3F2504E0-4F89-11D3-9A0C-0305E82C3301</AttributeValue>
      </Attribute>
      ...
    </AttributeStatement>
    <AuthnStatement AuthnInstant="2013-03-18T07:33:56.000Z" SessionIndex="_bf9c623d-cc20-407a-9a59-c2d0aee84d12">
      <AuthnContext>
        <AuthnContextClassRef> urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
      </AuthnContext>
    </AuthnStatement>
  </Assertion>
</samlp:Response>
```

### <a name="response"></a>Ответ

Элемент `Response` включает результат запроса на авторизацию. Azure AD устанавливает значения `ID`, `Version` и `IssueInstant` для элемента `Response`. Он также задает следующие атрибуты:

* `Destination`: после успешного входа для этого атрибута задается значение `RedirectUri` поставщика услуг (облачной службы).
* `InResponseTo`: для этого атрибута задается значение `ID` элемента `AuthnRequest`, инициировавшего ответ.

### <a name="issuer"></a>Издатель

Azure AD устанавливает для элемента `Issuer` значение `https://sts.windows.net/<TenantIDGUID>/`, где \<TenantIDGUID> — это идентификатор клиента Azure AD.

Например, ответ с элементом Issuer может выглядеть следующим образом:

```
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
```

### <a name="status"></a>Состояние

Элемент `Status` передает сведения об успешной или неудачной попытке входа. Он включает элемент `StatusCode`, содержащий код или набор блоков вложенного кода, который представляет состояние запроса. Он также включает элемент `StatusMessage` , содержащий настраиваемые сообщения об ошибках, создаваемые во время входа.

<!-- TODO: Add an authentication protocol error reference -->

Ниже приведен пример ответа SAML на неудачную попытку входа.

```
<samlp:Response ID="_f0961a83-d071-4be5-a18c-9ae7b22987a4" Version="2.0" IssueInstant="2013-03-18T08:49:24.405Z" InResponseTo="iddce91f96e56747b5ace6d2e2aa9d4f8c" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Requester">
      <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:RequestUnsupported" />
    </samlp:StatusCode>
    <samlp:StatusMessage>AADSTS75006: An error occurred while processing a SAML2 Authentication request. AADSTS90011: The SAML authentication request property 'NameIdentifierPolicy/SPNameQualifier' is not supported.
Trace ID: 66febed4-e737-49ff-ac23-464ba090d57c
Timestamp: 2013-03-18 08:49:24Z</samlp:StatusMessage>
  </samlp:Status>
```

### <a name="assertion"></a>Assertion

В дополнение к `ID`, `IssueInstant` и `Version` Azure AD устанавливает следующие элементы в элементе `Assertion` ответа.

#### <a name="issuer"></a>Издатель

Ему задается значение `https://sts.windows.net/<TenantIDGUID>/`, где \<TenantIDGUID> — это идентификатор клиента Azure AD.

```
<Issuer>https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
```

#### <a name="signature"></a>Сигнатура

Azure AD подписывает утверждение в ответ на успешный вход. В элементе `Signature` содержится цифровая подпись, которую может использовать облачная служба для проверки подлинности источника, чтобы проверять целостность утверждения.

Чтобы создать цифровую подпись, Azure AD использует ключ подписывания в элементе `IDPSSODescriptor` документа метаданных.

```
<ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
      digital_signature_here
    </ds:Signature>
```

#### <a name="subject"></a>Тема

Указывает субъект, который является субъектом операторов в утверждении. Он содержит элемент `NameID`, который представляет пользователя, прошедшего аутентификацию. Значение `NameID` — целевой идентификатор, направляемый только поставщику услуг, который является аудиторией маркера. Оно носит постоянный характер. Это значение можно отменить, но нельзя переназначить. Оно также непрозрачно, так как не раскрывает никаких сведений о пользователе и его нельзя использовать в качестве идентификатора для запросов на атрибуты.

Для атрибута `Method` элемента `SubjectConfirmation` всегда задано значение `urn:oasis:names:tc:SAML:2.0:cm:bearer`.

```
<Subject>
      <NameID>Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="id758d0ef385634593a77bdf7e632984b6" NotOnOrAfter="2013-03-18T07:43:15.144Z" Recipient="https://contoso.com/identity/inboundsso.aspx" />
      </SubjectConfirmation>
</Subject>
```

#### <a name="conditions"></a>Условия

Этот элемент указывает условия, определяющие приемлемое использование утверждений SAML.

```
<Conditions NotBefore="2013-03-18T07:38:15.128Z" NotOnOrAfter="2013-03-18T08:48:15.128Z">
      <AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
      </AudienceRestriction>
</Conditions>
```

Атрибуты `NotBefore` и `NotOnOrAfter` указывают интервал, в течение которого утверждение допустимо.

* Значение атрибута `NotBefore` равно значению атрибута `IssueInstant` в элементе `Assertion` или немного больше (менее чем на секунду). Azure AD не учитывает разницу между своим временем и временем облачной службы (поставщика услуг), а также не добавляет ничего к этому времени.
* Значение атрибута `NotOnOrAfter` на 70 минут больше, чем значение атрибута `NotBefore`.

#### <a name="audience"></a>Аудитория

Он содержит код URI, который определяет предполагаемую аудиторию. Azure AD присваивает этому элементу значение элемента `Issuer` из `AuthnRequest`, инициировавшего вход. Чтобы оценить значение `Audience`, используйте значение `App ID URI`, указанное при регистрации приложения.

```
<AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
</AudienceRestriction>
```

Как и значение `Issuer`, значение `Audience` должно точно соответствовать одному из имен субъектов-служб, представляющих облачную службу в Azure AD. Но если значения элемента `Issuer` и URI отличаются, значение `Audience` в ответе — это `Issuer` с префиксом `spn:`.

#### <a name="attributestatement"></a>AttributeStatement

В этом элементе содержатся утверждения о субъекте или пользователе. В следующем фрагменте приведен пример элемента `AttributeStatement` . Многоточие указывает, что в элементе могут содержаться несколько атрибутов и их значений.

```
<AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <AttributeValue>testuser@contoso.com</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
        <AttributeValue>3F2504E0-4F89-11D3-9A0C-0305E82C3301</AttributeValue>
      </Attribute>
      ...
</AttributeStatement>
```        

* **Утверждение Name**. Значение атрибута `Name` (`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`) — это имя участника-пользователя, прошедшего проверку подлинности, например `testuser@managedtenant.com`.
* **Утверждение ObjectIdentifier**. Значение `ObjectIdentifier` атрибута (`http://schemas.microsoft.com/identity/claims/objectidentifier`) — это `ObjectId` объекта каталога, который представляет пользователя, прошедшего проверку подлинности в Azure AD. Значение `ObjectId` неизменяемое, глобально уникальное и повторно использует безопасный идентификатор пользователя, прошедшего проверку подлинности.

#### <a name="authnstatement"></a>AuthnStatement

Этот элемент служит утверждением того, что проверка подлинности субъекта утверждения выполнена определенным средством в определенное время.

* Атрибут `AuthnInstant` указывает время аутентификации пользователя с помощью Azure AD.
* Элемент `AuthnContext` указывает контекст аутентификации, использованный для аутентификации пользователя.

```
<AuthnStatement AuthnInstant="2013-03-18T07:33:56.000Z" SessionIndex="_bf9c623d-cc20-407a-9a59-c2d0aee84d12">
      <AuthnContext>
        <AuthnContextClassRef> urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
      </AuthnContext>
</AuthnStatement>
```
