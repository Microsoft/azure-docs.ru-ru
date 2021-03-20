---
title: Метаданные федерации Azure AD | Документация Майкрософт
description: В этой статье описывается документ метаданных федерации, который Azure Active Directory публикует для каждой службы, принимающей токены Azure Active Directory.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 01/07/2017
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 7f3bd8851fe723461c618499e539c987d79c0d68
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101650147"
---
# <a name="federation-metadata"></a>Метаданные федерации

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Azure Active Directory (Azure AD) публикует документ метаданных федерации для служб, который настроен на прием маркеров безопасности, выдаваемых Azure AD. Формат документа метаданных федерации описан в стандарте [Web Services Federation Language (WS-Federation) Version 1.2](https://docs.oasis-open.org/wsfed/federation/v1.2/os/ws-federation-1.2-spec-os.html) (язык федерации веб-служб (WS-Federation) версии 1.2), который является расширением стандарта [Metadata for the OASIS Security Assertion Markup Language (SAML) v2.0](https://docs.oasis-open.org/security/saml/v2.0/saml-metadata-2.0-os.pdf) (метаданные для языка разметки заявлений системы безопасности (SAML) OASIS версии 2.0).

## <a name="tenant-specific-and-tenant-independent-metadata-endpoints"></a>Клиентские и общие конечные точки метаданных
Azure AD публикует клиентские и общие конечные точки.

Клиентские конечные точки предназначены только для одного конкретного клиента. Клиентские метаданные федерации содержат сведения о конкретном клиенте, в том числе данные о его издателе и конечной точке. Приложения, которые предлагают доступ только к одному клиенту, используют клиентские конечные точки.

Общие конечные точки предоставляют сведения, общие для всех клиентов Azure AD. Эти сведения относятся к клиентам, которые размещены в домене *login.microsoftonline.com* , и данные одинаковы для всех клиентов. Мы рекомендуем использовать общие конечные точки для мультитенантных приложений, так как они не связаны с каким-либо определенным клиентом.

## <a name="federation-metadata-endpoints"></a>Конечные точки метаданных федерации
Azure AD публикует метаданные федерации по адресу `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`.

Для **клиентских конечных точек**`TenantDomainName` может быть одного из следующих типов:

* зарегистрированное доменное имя клиента Azure AD, например `contoso.onmicrosoft.com`;
* неизменяемый идентификатор клиента для домена, например `72f988bf-86f1-41af-91ab-2d7cd011db45`.

Для **общих конечных точек**`TenantDomainName` является `common`. В этом документе перечислены только те элементы метаданных федерации, которые являются общими для всех клиентов Azure AD, размещенных на login.microsoftonline.com.

Клиентская конечная точка может иметь такой адрес: `https://login.microsoftonline.com/contoso.onmicrosoft.com/FederationMetadata/2007-06/FederationMetadata.xml`. Независимая от клиента конечная точка — [https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml](https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml) . Документ метаданных федерации можно просмотреть, введя этот URL-адрес в браузере.

## <a name="contents-of-federation-metadata"></a>Содержимое метаданных федерации
В следующем разделе приведены сведения, необходимые для служб, которые используют токены, выданные службой Azure AD.

### <a name="entity-id"></a>Идентификатор сущности
Элемент `EntityDescriptor` содержит атрибут `EntityID`. Значение атрибута `EntityID` представляет издателя, то есть службу токенов безопасности, которая выдала этот токен. Очень важно проверить издателя при получении токена.

В следующих метаданных показан пример клиентского элемента `EntityDescriptor` с элементом `EntityID`.

```
<EntityDescriptor
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="_b827a749-cfcb-46b3-ab8b-9f6d14a1294b"
entityID="https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db45/">
```
Чтобы получить клиентское значение `EntityID` , замените в общей конечной точке стандартный идентификатор клиента своим собственным идентификатором клиента. Полученное значение будет совпадать со значением издателя токена. Такая стратегия позволяет мультитенантному приложению проверять издателя для конкретного клиента.

В следующих метаданных приведен пример общего элемента `EntityID` . Обратите внимание, что `{tenant}` здесь является литералом, а не заполнителем.

```
<EntityDescriptor
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="="_0e5bd9d0-49ef-4258-bc15-21ce143b61bd"
entityID="https://sts.windows.net/{tenant}/">
```

### <a name="token-signing-certificates"></a>Сертификаты подписи токенов
Когда служба получает маркер, выданный клиентом Azure AD, подпись маркера должна быть проверена с помощью ключа подписывания, опубликованного в документе метаданных федерации. В метаданных федерации содержатся открытые ключи сертификатов, которые используются клиентами для подписывания токенов. Код сертификата отображается в элементе `KeyDescriptor` . Сертификат для подписи маркера можно использовать, только если атрибут `use` имеет значение `signing`.

Документ метаданных федерации, опубликованный службой Azure AD, может иметь несколько ключей подписывания, к примеру, когда Azure AD готовится к обновлению сертификата для подписывания. Если документ метаданных федерации содержит более одного сертификата, служба, проверяющая токены, должна поддерживать все сертификаты в документе.

В следующих метаданных приведен пример элемента `KeyDescriptor` с ключом подписывания.

```
<KeyDescriptor use="signing">
<KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
<X509Data>
<X509Certificate>
MIIDPjCCAiqgAwIBAgIQVWmXY/+9RqFA/OG9kFulHDAJBgUrDgMCHQUAMC0xKzApBgNVBAMTImFjY291bnRzLmFjY2Vzc2NvbnRyb2wud2luZG93cy5uZXQwHhcNMTIwNjA3MDcwMDAwWhcNMTQwNjA3MDcwMDAwWjAtMSswKQYDVQQDEyJhY2NvdW50cy5hY2Nlc3Njb250cm9sLndpbmRvd3MubmV0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArCz8Sn3GGXmikH2MdTeGY1D711EORX/lVXpr+ecGgqfUWF8MPB07XkYuJ54DAuYT318+2XrzMjOtqkT94VkXmxv6dFGhG8YZ8vNMPd4tdj9c0lpvWQdqXtL1TlFRpD/P6UMEigfN0c9oWDg9U7Ilymgei0UXtf1gtcQbc5sSQU0S4vr9YJp2gLFIGK11Iqg4XSGdcI0QWLLkkC6cBukhVnd6BCYbLjTYy3fNs4DzNdemJlxGl8sLexFytBF6YApvSdus3nFXaMCtBGx16HzkK9ne3lobAwL2o79bP4imEGqg+ibvyNmbrwFGnQrBc1jTF9LyQX9q+louxVfHs6ZiVwIDAQABo2IwYDBeBgNVHQEEVzBVgBCxDDsLd8xkfOLKm4Q/SzjtoS8wLTErMCkGA1UEAxMiYWNjb3VudHMuYWNjZXNzY29udHJvbC53aW5kb3dzLm5ldIIQVWmXY/+9RqFA/OG9kFulHDAJBgUrDgMCHQUAA4IBAQAkJtxxm/ErgySlNk69+1odTMP8Oy6L0H17z7XGG3w4TqvTUSWaxD4hSFJ0e7mHLQLQD7oV/erACXwSZn2pMoZ89MBDjOMQA+e6QzGB7jmSzPTNmQgMLA8fWCfqPrz6zgH+1F1gNp8hJY57kfeVPBiyjuBmlTEBsBlzolY9dd/55qqfQk6cgSeCbHCy/RU/iep0+UsRMlSgPNNmqhj5gmN2AFVCN96zF694LwuPae5CeR2ZcVknexOWHYjFM0MgUSw0ubnGl0h9AJgGyhvNGcjQqu9vd1xkupFgaN+f7P3p3EVN5csBg5H94jEcQZT7EKeTiZ6bTrpDAnrr8tDCy8ng
</X509Certificate>
</X509Data>
</KeyInfo>
</KeyDescriptor>
  ```

Элемент `KeyDescriptor` отображается в двух местах в документе метаданных федерации: в разделах WS-Federation и SAML. Сертификаты, опубликованные в обоих разделах, будут одинаковыми.

В разделе WS-Federation средство чтения метаданных WS-Federation будет считывать сертификаты из элемента `RoleDescriptor` с типом `SecurityTokenServiceType`.

В следующих метаданных приведен пример элемента `RoleDescriptor` .

```
<RoleDescriptor xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns:fed="https://docs.oasis-open.org/wsfed/federation/200706" xsi:type="fed:SecurityTokenServiceType" protocolSupportEnumeration="https://docs.oasis-open.org/wsfed/federation/200706">
```

В разделе SAML средство чтения метаданных WS-Federation будет читать сертификаты из элемента `IDPSSODescriptor` .

В следующих метаданных приведен пример элемента `IDPSSODescriptor` .

```
<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
```
Форматы клиентских и общих сертификатов не отличаются.

### <a name="ws-federation-endpoint-url"></a>URL-адрес конечной точки WS-Federation
Метаданные федерации включают URL-адрес, используемый службой Azure AD для единого входа и единого выхода в протоколе WS-Federation. Эта конечная точка отображается в элементе `PassiveRequestorEndpoint` .

В следующих метаданных приведен пример элемента `PassiveRequestorEndpoint` для клиентской конечной точки.

```
<fed:PassiveRequestorEndpoint>
<EndpointReference xmlns="https://www.w3.org/2005/08/addressing">
<Address>
https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db45/wsfed
</Address>
</EndpointReference>
</fed:PassiveRequestorEndpoint>
```
Для общей конечной точки URL-адрес WS-Federation отображается в конечной точке WS-Federation, как показано в следующем примере.

```
<fed:PassiveRequestorEndpoint>
<EndpointReference xmlns="https://www.w3.org/2005/08/addressing">
<Address>
https://login.microsoftonline.com/common/wsfed
</Address>
</EndpointReference>
</fed:PassiveRequestorEndpoint>
```

### <a name="saml-protocol-endpoint-url"></a>URL-адрес конечной точки протокола SAML
Метаданные федерации содержат URL-адрес, используемый службой Azure AD для единого входа и единого выхода в протоколе SAML 2.0. Эти конечные точки отображаются в элементе `IDPSSODescriptor` .

URL-адреса для входа и выхода содержатся в элементах `SingleSignOnService` и `SingleLogoutService`.

В следующих метаданных приведен пример элемента `PassiveResistorEndpoint` для клиентской конечной точки.

```
<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
…
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/contoso.onmicrosoft.com/saml2" />
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/contoso.onmicrosoft.com /saml2" />
  </IDPSSODescriptor>
```

Так же конечные точки для общих конечных точек протокола SAML 2.0 публикуются в общих метаданных федерации, как показано в следующем примере.

```
<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
…
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/common/saml2" />
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/common/saml2" />
  </IDPSSODescriptor>
```
