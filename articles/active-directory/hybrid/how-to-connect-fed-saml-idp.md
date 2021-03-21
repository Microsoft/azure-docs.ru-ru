---
title: 'Azure AD Connect выполняет следующие функции: Использование поставщика удостоверений (IdP) SAML 2.0 для единого входа в Azure'
description: В этом документе описывается использование поставщика удостоверений, совместимого с SAML 2.0, для единого входа.
services: active-directory
author: billmath
manager: daveba
ms.custom: it-pro
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/13/2017
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: b26c24149d422021dcb86f75c915ade89cbccdec
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97589881"
---
#  <a name="use-a-saml-20-identity-provider-idp-for-single-sign-on"></a>Использование поставщика удостоверений (IdP) SAML 2.0 для единого входа

В этом документе содержатся сведения об использовании поставщика удостоверений на основе профилей SP-Lite, совместимого с SAML 2.0, в качестве предпочитаемой службы токенов безопасности (STS) или поставщика удостоверений. Это удобно при наличии локального каталога пользователей и хранилища паролей, доступ к которым возможен с помощью SAML 2.0. Этот существующий каталог пользователей можно использовать для входа в Microsoft 365 и другие ресурсы, защищенные с использованием Azure AD. Профиль SAML 2.0 SP-Lite основан на широко использующемся стандарте федеративных удостоверений SAML (Security Assertion Markup Language) для предоставления единого входа и структуры обмена атрибутами.

>[!NOTE]
>Перечень сторонних поставщиков удостоверений, проверенных для использования в Azure AD см. в [списке совместимости с федерацией Azure AD](how-to-connect-fed-compatibility.md).

Корпорация Майкрософт поддерживает этот процесс входа в качестве интеграции облачной службы Майкрософт, например Microsoft 365, с правильной настройкой IdP на основе профиля SAML 2,0. Поставщики удостоверений SAML 2.0 — это сторонние продукты, поэтому корпорация Майкрософт не оказывает поддержку по развертыванию, настройке, устранению неполадок и предоставлению рекомендаций, касающихся их. После настройки интеграции с поставщиком удостоверений SAML 2.0 ее правильность можно протестировать с помощью анализатора подключений Майкрософт, который подробно описывается ниже. Для получения дополнительных сведений о своем поставщике удостоверений на основе профилей SAML 2.0 SP-Lite обратитесь в предоставившую его организацию.

> [!IMPORTANT]
> В этом сценарии единого входа с помощью поставщиков удостоверений SAML 2.0 доступен только ограниченный набор клиентов, в число которых входят приведенные далее.
> 
> - Веб-клиенты, такие как Outlook Web Access и SharePoint Online
> - Полнофункциональные почтовые клиенты, использующие базовую проверку подлинности и поддерживаемый метод доступа Exchange, такой как IMAP, POP, Active Sync, MAPI и т. д. (требуется развертывание конечной точки расширенного клиентского протокола), включая следующие:
>     - Microsoft Outlook 2010/Outlook 2013/Outlook 2016, Apple iPhone (различные версии iOS);
>     - различные устройства Google Android;
>     - Windows Phone 7, Windows Phone 7.8 и Windows Phone 8.0;
>     - почтовый клиент Windows 8 и Windows 8.1;
>     - почтовый клиент Windows 10.

Все остальные клиенты не поддерживаются в сценарии единого входа с помощью поставщика удостоверений SAML 2.0. Например, клиент Lync 2010 для настольных ПК не сможет войти в службу с помощью поставщика удостоверений SAML 2.0, настроенного для единого входа.

## <a name="azure-ad-saml-20-protocol-requirements"></a>Требования Azure AD к протоколу SAML 2.0
В этом документе содержатся подробные требования к протоколу и форматированию сообщений, которые должен реализовать поставщик удостоверений SAML 2,0 для Федерации с Azure AD, чтобы включить вход в одну или несколько облачных служб Майкрософт (например, Microsoft 365). Проверяющей стороной SAML 2.0 (SP-STS) для облачной службы Майкрософт в этом сценарии является Azure AD.

Рекомендуется обеспечить максимальную схожесть выходных сообщений поставщика удостоверений SAML 2.0 с предоставленными образцами трассировки. Кроме того, по возможности используйте значения атрибутов из предоставленных метаданных Azure AD. Добившись требуемых выходных сообщений, проведите тестирование с помощью анализатора подключений Майкрософт, как описано ниже.

Метаданные Azure AD можно скачать по этому URL-адресу: [https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml](https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml).
Для клиентов в Китае, использующих экземпляр Microsoft 365, предназначенный для Китая, следует использовать следующую конечную точку Федерации: [https://nexus.partner.microsoftonline-p.cn/federationmetadata/saml20/federationmetadata.xml](https://nexus.partner.microsoftonline-p.cn/federationmetadata/saml20/federationmetadata.xml) .

## <a name="saml-protocol-requirements"></a>Требования к протоколу SAML
В этом разделе подробно описывается, как составляются пары, состоящие из запроса и ответного сообщения. Это поможет вам правильно форматировать свои сообщения.

При настройке Azure AD для работы с поставщиками удостоверений, использующими профиль SAML 2.0 SP Lite, предъявляется ряд особых требований, которые приведены ниже. Используя образцы запросов и ответных сообщений SAML в сочетании с автоматическим и ручным тестированием, вы можете обеспечить взаимодействие с Azure AD.

## <a name="signature-block-requirements"></a>Требования к блоку подписи
В ответном сообщении SAML узел Signature содержит сведения о цифровой подписи самого сообщения. К блоку подписи предъявляются указанные ниже требования.

1. Узел утверждения должен быть подписан.
2. В качестве метода DigestMethod необходимо использовать алгоритм RSA-sha1. Другие алгоритмы цифровой подписи не допускаются.
   `<ds:DigestMethod Algorithm="https://www.w3.org/2000/09/xmldsig#sha1"/>`
3. Также можно подписать XML-документ. 
4. Алгоритм преобразования должен соответствовать значениям в следующем примере:     `<ds:Transform Algorithm="https://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
       <ds:Transform Algorithm="https://www.w3.org/2001/10/xml-exc-c14n#"/>`
9. Алгоритм SignatureMethod должен соответствовать следующему примеру:    `<ds:SignatureMethod Algorithm="https://www.w3.org/2000/09/xmldsig#rsa-sha1"/>`

## <a name="supported-bindings"></a>Поддерживаемые привязки
Привязки — это обязательные параметры взаимодействия, связанные с транспортом. В отношении привязок действуют указанные ниже требования.

1. HTTPS — это обязательный транспорт.
2. Службе Azure AD потребуется запрос HTTP POST для отправки токена во время входа.
3. Служба Azure AD будет использовать метод HTTP POST для отправки запроса аутентификации поставщику удостоверений и метод REDIRECT для отправки ему сообщения о выходе.

## <a name="required-attributes"></a>Требуемые атрибуты
В таблице ниже приведены требования к определенным атрибутам в сообщении SAML 2.0.
 
|attribute|Описание|
| ----- | ----- |
|NameID|Значение этого утверждения должно совпадать с идентификатором ImmutableID пользователя в Azure AD. Это должно быть буквенно-цифровое значение длиной до 64 символов. Все небезопасные знаки HTML должны кодироваться, например, знак "+" должен быть представлен как ".2B".|
|IDPEmail|Имя участника-пользователя (UPN) указано в ответе SAML как элемент с именем IDPEmail пользователя UserPrincipalName (UPN) в Azure AD/Microsoft 365. Имя субъекта-пользователя указывается в формате адреса электронной почты. Значение UPN в Microsoft 365 Windows (Azure Active Directory).|
|Издатель|Это должен быть универсальный код ресурса (URI) поставщика удостоверений. Не следует использовать значение Issuer из примеров сообщений. Если в клиентах Azure AD несколько доменов верхнего уровня, значение Issuer должно совпадать с универсальным кодом ресурса (URI), настроенным для домена.|

>[!IMPORTANT]
>Сейчас Azure AD поддерживает следующий формат URI для атрибута NameID в SAML 2.0: urn:oasis:names:tc:SAML:2.0:nameid-format:persistent.

## <a name="sample-saml-request-and-response-messages"></a>Примеры сообщений SAML с запросом и ответом
Ниже приведена пара сообщений, состоящая из запроса и ответа и используемая при обмене сообщениями единого входа.
Это пример сообщения с запросом, отправляемого из Azure AD поставщику удостоверений SAML 2.0. Поставщик удостоверений SAML 2.0 в этом примере представляет собой службы федерации Active Directory (AD FS), настроенные для использования протокола SAML-P. Тестирование взаимодействия было выполнено и для других поставщиков удостоверений SAML 2.0.

```xml
  <samlp:AuthnRequest 
    xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" 
    xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" 
    ID="_7171b0b2-19f2-4ba2-8f94-24b5e56b7f1e" 
    IssueInstant="2014-01-30T16:18:35Z" 
    Version="2.0" 
    AssertionConsumerServiceIndex="0" >
        <saml:Issuer>urn:federation:MicrosoftOnline</saml:Issuer>
        <samlp:NameIDPolicy Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"/>
  </samlp:AuthnRequest>
```

Ниже приведен пример ответного сообщения, которое отправляется из примера поставщика удостоверений, совместимых с SAML 2,0, в Azure AD/Microsoft 365.

```xml
    <samlp:Response ID="_592c022f-e85e-4d23-b55b-9141c95cd2a5" Version="2.0" IssueInstant="2014-01-31T15:36:31.357Z" Destination="https://login.microsoftonline.com/login.srf" Consent="urn:oasis:names:tc:SAML:2.0:consent:unspecified" InResponseTo="_049917a6-1183-42fd-a190-1d2cbaf9b144" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
    <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">http://WS2012R2-0.contoso.com/adfs/services/trust</Issuer>
    <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
    </samlp:Status>
    <Assertion ID="_7e3c1bcd-f180-4f78-83e1-7680920793aa" IssueInstant="2014-01-31T15:36:31.279Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">
    <Issuer>http://WS2012R2-0.contoso.com/adfs/services/trust</Issuer>
    <ds:Signature xmlns:ds="https://www.w3.org/2000/09/xmldsig#">
      <ds:SignedInfo>
        <ds:CanonicalizationMethod Algorithm="https://www.w3.org/2001/10/xml-exc-c14n#" />
        <ds:SignatureMethod Algorithm="https://www.w3.org/2000/09/xmldsig#rsa-sha1" />
        <ds:Reference URI="#_7e3c1bcd-f180-4f78-83e1-7680920793aa">
          <ds:Transforms>
            <ds:Transform Algorithm="https://www.w3.org/2000/09/xmldsig#enveloped-signature" />
            <ds:Transform Algorithm="https://www.w3.org/2001/10/xml-exc-c14n#" />
          </ds:Transforms>
          <ds:DigestMethod Algorithm="https://www.w3.org/2000/09/xmldsig#sha1" />
          <ds:DigestValue>CBn/5YqbheaJP425c0pHva9PhNY=</ds:DigestValue>
        </ds:Reference>
      </ds:SignedInfo>
      <ds:SignatureValue>TciWMyHW2ZODrh/2xrvp5ggmcHBFEd9vrp6DYXp+hZWJzmXMmzwmwS8KNRJKy8H7XqBsdELA1Msqi8I3TmWdnoIRfM/ZAyUppo8suMu6Zw+boE32hoQRnX9EWN/f0vH6zA/YKTzrjca6JQ8gAV1ErwvRWDpyMcwdYCiWALv9ScbkAcebOE1s1JctZ5RBXggdZWrYi72X+I4i6WgyZcIGai/rZ4v2otoWAEHS0y1yh1qT7NDPpl/McDaTGkNU6C+8VfjD78DrUXEcAfKvPgKlKrOMZnD1lCGsViimGY+LSuIdY45MLmyaa5UT4KWph6dA==</ds:SignatureValue>
      <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
        <ds:X509Data>
          <ds:X509Certificate>MIIC7jCCAdagAwIBAgIQRrjsbFPaXIlOG3GTv50fkjANBgkqhkiG9w0BAQsFADAzMTEwLwYDVQQDEyhBREZTIFNpZ25pbmcgLSBXUzIwMTJSMi0wLnN3aW5mb3JtZXIuY29tMB4XDTE0MDEyMDE1MTY0MFoXDTE1MDEyMDE1MTY0MFowMzExMC8GA1UEAxMoQURGUyBTaWduaW5nIC0gV1MyMDEyUjItMC5zd2luZm9ybWVyLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKe+rLVmXy1QwCwZwqgbbp1/+3ZWxd9T/jV0hpLIIWr+LCOHqq8n8beJvlivgLmDJo8f+EITnAxWcsJUvVai/35AhHCUq9tc9sqMp5PWtabAEMb2AU72/QlX/72D2/NbGQq1BWYbqUpgpCZ2nSgvlWDHlCiUo//UGsvfox01kjTFlmqQInsJVfRxF5AcCAwEAATANBgkqhkiG9w0BAQsFAAOCAQEAi8c6C4zaTEc7aQiUgvnGQgCbMZbhUXXLGRpjvFLKaQzkwa9eq7WLJibcSNyGXBa/SfT5wJgsm3TPKgSehGAOTirhcqHheZyvBObAScY7GOT+u9pVYp6raFrc7ez3c+CGHeV/tNvy1hJNs12FYH4X+ZCNFIT9tprieR25NCdi5SWUbPZL0tVzJsHc1y92b2M2FxqRDohxQgJvyJOpcg2mSBzZZIkvDg7gfPSUXHVS1MQs0RHSbwq/XdQocUUhl9/e/YWCbNNxlM84BxFsBUok1dH/gzBySx+Fc8zYi7cOq9yaBT3RLT6cGmFGVYZJW4FyhPZOCLVNsLlnPQcX3dDg9A==</ds:X509Certificate>
        </ds:X509Data>
      </KeyInfo>
    </ds:Signature>
    <Subject>
      <NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">ABCDEG1234567890</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="_049917a6-1183-42fd-a190-1d2cbaf9b144" NotOnOrAfter="2014-01-31T15:41:31.357Z" Recipient="https://login.microsoftonline.com/login.srf" />
      </SubjectConfirmation>
    </Subject>
    <Conditions NotBefore="2014-01-31T15:36:31.263Z" NotOnOrAfter="2014-01-31T16:36:31.263Z">
      <AudienceRestriction>
        <Audience>urn:federation:MicrosoftOnline</Audience>
      </AudienceRestriction>
    </Conditions>
    <AttributeStatement>
      <Attribute Name="IDPEmail">
        <AttributeValue>administrator@contoso.com</AttributeValue>
      </Attribute>
    </AttributeStatement>
    <AuthnStatement AuthnInstant="2014-01-31T15:36:30.200Z" SessionIndex="_7e3c1bcd-f180-4f78-83e1-7680920793aa">
      <AuthnContext>
        <AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</AuthnContextClassRef>
      </AuthnContext>
    </AuthnStatement>
    </Assertion>
    </samlp:Response>
```

## <a name="configure-your-saml-20-compliant-identity-provider"></a>Настройка поставщика удостоверений, совместимого с SAML 2.0
В этом разделе содержатся рекомендации по настройке поставщика удостоверений SAML 2,0 для Федерации с Azure AD, чтобы обеспечить единый вход для одной или нескольких облачных служб Майкрософт (например, Microsoft 365) с помощью протокола SAML 2,0. В качестве проверяющей стороны SAML 2.0 для облачной службы (Майкрософт) в этом сценарии используется Azure AD.

## <a name="add-azure-ad-metadata"></a>Добавление метаданных Azure AD
Поставщик удостоверений SAML 2.0 должен придерживаться информации о проверяющей стороне Azure AD. Azure AD публикует метаданные по адресу https://nexus.microsoftonline-p.com/federationmetadata/saml20/federationmetadata.xml.

При настройке поставщика удостоверений SAML 2.0 рекомендуется всегда импортировать новейшие метаданные Azure AD.

>[!NOTE]
>Служба Azure AD не считывает метаданные поставщика удостоверений.

## <a name="add-azure-ad-as-a-relying-party"></a>Добавление Azure AD в качестве проверяющей стороны
Необходимо включить обмен данными между поставщиком удостоверений SAML 2.0 и Azure AD. Эта конфигурация будет зависеть от используемого поставщика удостоверений. Обратитесь к документации по нему. Идентификатор проверяющей стороны, как правило, должен совпадать с идентификатором entityID, указанным в метаданных Azure AD.

>[!NOTE]
>Убедитесь, что часы на сервере поставщика удостоверений SAML 2.0 синхронизированы с источником точного времени. Неточное время часов может приводить к ошибкам при выполнении федеративных входов.

## <a name="install-windows-powershell-for-sign-on-with-saml-20-identity-provider"></a>Установка Windows PowerShell для единого входа с помощью поставщика удостоверений SAML 2.0
После настройки поставщика удостоверений SAML 2.0 для единого входа с помощью Azure AD далее необходимо загрузить и установить модуль Azure Active Directory для Windows PowerShell. После установки эти командлеты будут использоваться для настройки доменов Azure AD в качестве федеративных доменов.

Модуль Azure Active Directory для Windows PowerShell — это загружаемый компонент для управления данными организации в Azure AD. Он устанавливает набор командлетов в Windows PowerShell, которые служат для настройки единого входа в Azure AD и далее во все облачные службы, на которые вы подписаны. Инструкции по скачиванию и установке командлетов см. в разделе [/Previous-Versions/Azure/jj151815 (v = Azure. 100)](/previous-versions/azure/jj151815(v=azure.100)) .

## <a name="set-up-a-trust-between-your-saml-identity-provider-and-azure-ad"></a>Настройка отношения доверия между поставщиком удостоверений SAML и Azure AD
Перед настройкой федерации в домене Azure AD в нем должен быть настроен личный домен. Установить федерацию с доменом по умолчанию, предоставленным корпорацией Майкрософт, нельзя. Домен по умолчанию от корпорации Майкрософт заканчивается на onmicrosoft.com.
Чтобы добавить или преобразовать домены для единого входа, вам потребуется выполнить ряд командлетов в интерфейсе командной строки PowerShell.

Каждый домен Azure Active Directory, который необходимо включить в федерацию с помощью поставщика удостоверений SAML 2.0, должен быть добавлен как домен единого входа или преобразован из стандартного домена в домен единого входа. При добавлении или преобразовании домена между поставщиком удостоверений SAML 2.0 и Azure AD настраивается отношение доверия.

В следующей процедуре содержатся указания по преобразованию существующего стандартного домена в федеративный домен с помощью SAML 2.0 SP-Lite. 

>[!NOTE]
>После выполнения этого действия домен может стать недоступен для пользователей на срок до 2 часов.

## <a name="configuring-a-domain-in-your-azure-ad-directory-for-federation"></a>Настройка домена в каталоге Azure AD для федерации


1. Подключитесь к каталогу Azure AD в качестве администратора клиента:

  ```powershell
  Connect-MsolService
  ```
  
2. Настройте требуемый домен Microsoft 365 для использования Федерации с SAML 2,0:

  ```powershell
  $dom = "contoso.com" 
  $BrandName - "Sample SAML 2.0 IDP" 
  $LogOnUrl = "https://WS2012R2-0.contoso.com/passiveLogon" 
  $LogOffUrl = "https://WS2012R2-0.contoso.com/passiveLogOff" 
  $ecpUrl = "https://WS2012R2-0.contoso.com/PAOS" 
  $MyURI = "urn:uri:MySamlp2IDP" 
  $MySigningCert = "MIIC7jCCAdagAwIBAgIQRrjsbFPaXIlOG3GTv50fkjANBgkqhkiG9w0BAQsFADAzMTEwLwYDVQQDEyh BREZTIFNpZ25pbmcgLSBXUzIwMTJSMi0wLnN3aW5mb3JtZXIuY29tMB4XDTE0MDEyMDE1MTY0MFoXDT E1MDEyMDE1MTY0MFowMzExMC8GA1UEAxMoQURGUyBTaWduaW5nIC0gV1MyMDEyUjItMC5zd2luZm9yb WVyLmNvbTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAKe+rLVmXy1QwCwZwqgbbp1/kupQ VcjKuKLitVDbssFyqbDTjP7WRjlVMWAHBI3kgNT7oE362Gf2WMJFf1b0HcrsgLin7daRXpq4Qi6OA57 sW1YFMj3sqyuTP0eZV3S4+ZbDVob6amsZIdIwxaLP9Zfywg2bLsGnVldB0+XKedZwDbCLCVg+3ZWxd9 T/jV0hpLIIWr+LCOHqq8n8beJvlivgLmDJo8f+EITnAxWcsJUvVai/35AhHCUq9tc9sqMp5PWtabAEM b2AU72/QlX/72D2/NbGQq1BWYbqUpgpCZ2nSgvlWDHlCiUo//UGsvfox01kjTFlmqQInsJVfRxF5AcC AwEAATANBgkqhkiG9w0BAQsFAAOCAQEAi8c6C4zaTEc7aQiUgvnGQgCbMZbhUXXLGRpjvFLKaQzkwa9 eq7WLJibcSNyGXBa/SfT5wJgsm3TPKgSehGAOTirhcqHheZyvBObAScY7GOT+u9pVYp6raFrc7ez3c+ CGHeV/tNvy1hJNs12FYH4X+ZCNFIT9tprieR25NCdi5SWUbPZL0tVzJsHc1y92b2M2FxqRDohxQgJvy JOpcg2mSBzZZIkvDg7gfPSUXHVS1MQs0RHSbwq/XdQocUUhl9/e/YWCbNNxlM84BxFsBUok1dH/gzBy Sx+Fc8zYi7cOq9yaBT3RLT6cGmFGVYZJW4FyhPZOCLVNsLlnPQcX3dDg9A==" 
  $uri = "http://WS2012R2-0.contoso.com/adfs/services/trust" 
  $Protocol = "SAMLP" 
  Set-MsolDomainAuthentication `
    -DomainName $dom `
    -FederationBrandName $BrandName `
    -Authentication Federated `
    -PassiveLogOnUri $LogOnUrl `
    -ActiveLogOnUri $ecpUrl `
    -SigningCertificate $MySigningCert `
    -IssuerUri $MyURI `
    -LogOffUri $LogOffUrl `
    -PreferredAuthenticationProtocol $Protocol
  ``` 

3.  Получить строку сертификата подписи в кодировке base64 можно из файла метаданных IDP. Пример его расположения приведен, но в зависимости от особенностей развертывания оно может быть немного иным.

  ```xml
  <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
    <KeyDescriptor use="signing">
      <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
       <X509Data>
         <X509Certificate> MIIC5jCCAc6gAwIBAgIQLnaxUPzay6ZJsC8HVv/QfTANBgkqhkiG9w0BAQsFADAvMS0wKwYDVQQDEyRBREZTIFNpZ25pbmcgLSBmcy50ZWNobGFiY2VudHJhbC5vcmcwHhcNMTMxMTA0MTgxMzMyWhcNMTQxMTA0MTgxMzMyWjAvMS0wKwYDVQQDEyRBREZTIFNpZ25pbmcgLSBmcy50ZWNobGFiY2VudHJhbC5vcmcwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCwMdVLTr5YTSRp+ccbSpuuFeXMfABD9mVCi2wtkRwC30TIyPdORz642MkurdxdPCWjwgJ0HW6TvXwcO9afH3OC5V//wEGDoNcI8PV4enCzTYFe/h//w51uqyv48Fbb3lEXs+aVl8155OAj2sO9IX64OJWKey82GQWK3g7LfhWWpp17j5bKpSd9DBH5pvrV+Q1ESU3mx71TEOvikHGCZYitEPywNeVMLRKrevdWI3FAhFjcCSO6nWDiMqCqiTDYOURXIcHVYTSof1YotkJ4tG6mP5Kpjzd4VQvnR7Pjb47nhIYG6iZ3mR1F85Ns9+hBWukQWNN2hcD/uGdPXhpdMVpBAgMBAAEwDQYJKoZIhvcNAQELBQADggEBAK7h7jF7wPzhZ1dPl4e+XMAr8I7TNbhgEU3+oxKyW/IioQbvZVw1mYVCbGq9Rsw4KE06eSMybqHln3w5EeBbLS0MEkApqHY+p68iRpguqa+W7UHKXXQVgPMCpqxMFKonX6VlSQOR64FgpBme2uG+LJ8reTgypEKspQIN0WvtPWmiq4zAwBp08hAacgv868c0MM4WbOYU0rzMIR6Q+ceGVRImlCwZ5b7XKp4mJZ9hlaRjeuyVrDuzBkzROSurX1OXoci08yJvhbtiBJLf3uPOJHrhjKRwIt2TnzS9ElgFZlJiDIA26Athe73n43CT0af2IG6yC7e6sK4L3NEXJrwwUZk=</X509Certificate>
        </X509Data>
      </KeyInfo>
    </KeyDescriptor>
  </IDPSSODescriptor>
  ``` 

Дополнительные сведения о "Set-MsolDomainAuthentication" см. в разделе: [/Previous-Versions/Azure/dn194112 (v = Azure. 100)](/previous-versions/azure/dn194112(v=azure.100)).

>[!NOTE]
>Использовать атрибут `$ecpUrl = "https://WS2012R2-0.contoso.com/PAOS"` следует только в том случае, если для поставщика удостоверений настраивается расширение ECP. Клиенты Exchange Online, исключая Outlook Web Application (OWA), используют активную конечную точку на основе метода POST. Если служба токенов безопасности SAML 2.0 реализует активную конечную точку, аналогичную реализации активной конечной точки с помощью расширения ECP для Shibboleth, эти полнофункциональные клиенты могут взаимодействовать со службой Exchange Online.

После настройки федерации можно вернуться к управляемой реализации (без поддержки федерации), однако на применение этого изменения требуется до двух часов, а каждому пользователю необходимо назначить новый случайный пароль для входа в облачные службы. Переключение к управляемой реализации может потребоваться в некоторых сценариях для сброса ошибочных параметров. Дополнительные сведения о преобразовании доменов см. в статье [/Previous-Versions/Azure/dn194122 (v = Azure. 100)](/previous-versions/azure/dn194122(v=azure.100)).

## <a name="provision-user-principals-to-azure-ad--microsoft-365"></a>Предоставление субъектов-пользователей в Azure AD/Microsoft 365
Прежде чем можно будет выполнять проверку подлинности пользователей в Microsoft 365, необходимо подготавливать Azure AD с субъектами-пользователями, которые соответствуют утверждению в утверждении SAML 2,0. Если эти субъекты-пользователи заранее не известны службе Azure AD, их нельзя использовать для федеративного входа. Подготовить субъектов-пользователей можно с помощью Azure AD Connect или Windows PowerShell.

С помощью Azure AD Connect можно подготавливать субъектов-пользователей для доменов каталога Azure AD из локальной службы Active Directory. Дополнительные сведения см. в статье [Интеграция локальных каталогов с Azure Active Directory](whatis-hybrid-identity.md).

С помощью Windows PowerShell также можно автоматизировать добавление новых пользователей в Azure AD и синхронизировать изменения из локального каталога. Для использования командлетов Windows PowerShell необходимо скачать [модули Azure Active Directory](/powershell/azure/active-directory/install-adv2).

Ниже приведена процедура добавления отдельного пользователя в Azure AD.


1. Подключитесь к каталогу Azure AD в качестве администратора клиента: Connect-MsolService.
2. Создайте субъекта-пользователя:

    ```powershell
    New-MsolUser `
      -UserPrincipalName elwoodf1@contoso.com `
      -ImmutableId ABCDEFG1234567890 `
      -DisplayName "Elwood Folk" `
      -FirstName Elwood `
      -LastName Folk `
      -AlternateEmailAddresses "Elwood.Folk@contoso.com" `
      -LicenseAssignment "samlp2test:ENTERPRISEPACK" `
      -UsageLocation "US" 
    ```

Дополнительные сведения о извлечении New-MsolUser см. в статье [/Previous-Versions/Azure/dn194096 (v = Azure. 100)](/previous-versions/azure/dn194096(v=azure.100)) .

>[!NOTE]
>Значение UserPrincipalName должно совпадать со значением, которое будет отправлено для "IDPEmail" в утверждении SAML 2,0, а значение "ImmutableID" должно совпадать со значением, отправленным в утверждении "NameID".

## <a name="verify-single-sign-on-with-your-saml-20-idp"></a>Проверка единого входа с помощью поставщика удостоверений SAML 2.0
Перед проверкой единого входа (также называемого федерацией удостоверений) и управлением им администратору следует ознакомиться с информацией и выполнить инструкции в указанных ниже статьях, чтобы настроить единый вход с помощью поставщика удостоверений на основе SAML 2.0 SP-Lite:

1. Ознакомление с требованиями к протоколу SAML 2.0 для Azure AD.
2. Настройки поставщика удостоверений SAML 2.0.
3. Установка Windows PowerShell для единого входа с помощью поставщика удостоверений SAML 2.0.
4. Настройка отношения доверия между поставщиком удостоверений SAML 2.0 и Azure AD.
5. Выполнив подготовку известного тестового участника-пользователя для Azure Active Directory (Microsoft 365) с помощью Windows PowerShell или Azure AD Connect.
6. Настройка синхронизации каталогов с помощью [Azure AD Connect](whatis-hybrid-identity.md).

После настройки единого входа с помощью поставщика удостоверений на основе SAML 2.0 SP-Lite следует проверить, правильно ли он работает.

>[!NOTE]
>Если вместо добавления домен преобразуется, то для настройки единого входа может понадобиться до 24 часов.
Перед проверкой единого входа следует завершить настройку синхронизации Active Directory, синхронизировать каталоги и активировать синхронизированных пользователей.

### <a name="use-the-tool-to-verify-that-single-sign-on-has-been-set-up-correctly"></a>Проверка правильной настройки единого входа с помощью средства
Чтобы проверить, правильно ли настроен единый вход, можно выполнить следующую процедуру, позволяющую подтвердить возможность входа в облачную службу с помощью корпоративных учетных данных.

Корпорация Майкрософт предоставляет средство, позволяющее протестировать поставщика удостоверений на основе SAML 2.0. Перед запуском средства тестирования должна быть настроена федерация клиента Azure AD с поставщиком удостоверений.

>[!NOTE]
>Для работы анализатора подключений требуется браузер Internet Explorer 10 или более поздней версии.



1. Загрузите [анализатор подключений](https://testconnectivity.microsoft.com/?tabid=Client).
2. Чтобы начать загрузку и установку средства, щелкните "Установить сейчас".
3. Выберите пункт "Не удается настроить федерацию с Office 365, Azure или другими службами, использующими Azure Active Directory".
4. После загрузки и запуска средства откроется окно "Диагностика подключений". Средство предоставляет пошаговые инструкции по тестированию подключения федерации.
5. Анализатор подключений откроет SAML 2,0 IDP для входа, введите учетные данные для тестируемого участника-пользователя:

    ![Снимок экрана, на котором показано окно входа для SAML 2,0 IDP.](./media/how-to-connect-fed-saml-idp/saml1.png)

6.  В диалоговом окне "Вход для выполнения тестирования федерации" следует ввести имя учетной записи и пароль клиента Azure AD, который настроен для федерации с поставщиком удостоверений SAML 2.0. Средство попытается выполнить вход, используя эти учетные данные, после чего выведет подробные результаты тестов, выполненных во время попытки входа.

    ![SAML](./media/how-to-connect-fed-saml-idp/saml2.png)

7. В этом окне показано, что тест не пройден. Щелкнув ссылку "Просмотреть подробные результаты", можно просмотреть сведения о результатах каждого выполненного теста. Также можно сохранить результаты на диск, чтобы поделиться ими.
 
> [!NOTE]
> Кроме того, анализатор подключений тестирует активную федерацию с помощью протоколов на основе WS*, а также протоколов ECP и PAOS. Если вы их не используете, игнорируйте следующую ошибку. Тестирование потока активного входа в систему с помощью конечной точки активной федерации вашего поставщика удостоверений.

### <a name="manually-verify-that-single-sign-on-has-been-set-up-correctly"></a>Проверка правильной настройки единого входа с помощью средства

Проверка вручную — это дополнительный способ, с помощью которого можно убедиться в том, что поставщик удостоверений SAML 2.0 работает правильно во многих сценариях.
Чтобы проверить, правильно ли настроен единый вход, выполните указанные ниже действия.

1. На присоединенном к домену компьютере войдите в облачную службу, используя то же имя для входа, что и в ваших корпоративных учетных данных.
2. Щелкните в поле "Пароль". Если единый вход настроен, поле "Пароль" станет темнее и отобразится следующее сообщение: "Вам нужно войти на веб-сайт &lt;ваша организация&gt;".
3. Щелкните ссылку "Sign-in at &lt;ваша организация&gt;" (Войти на веб-сайт "ваша организация"). Если вход возможен, то единый вход настроен.

## <a name="next-steps"></a>Next Steps

- [Управление службами федерации Active Directory и их настройка с помощью Azure AD Connect](how-to-connect-fed-management.md)
- [Список совместимости с федерацией Azure AD](how-to-connect-fed-compatibility.md)
- [Выборочная установка Azure AD Connect](how-to-connect-install-custom.md)
