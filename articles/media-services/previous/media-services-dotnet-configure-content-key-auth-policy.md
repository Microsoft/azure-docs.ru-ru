---
title: Настройка политики авторизации ключей содержимого с помощью пакета SDK Служб мультимедиа для .NET | Документация Майкрософт
description: Узнайте, как настроить политику авторизации ключа содержимого с помощью пакета SDK Служб мультимедиа для .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 1a0aedda-5b87-4436-8193-09fc2f14310c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 63cc20a9d7f6878961519aa28ae6478d9145852d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103014219"
---
# <a name="configure-a-content-key-authorization-policy-by-using-the-media-services-net-sdk"></a>Настройка политики авторизации ключей содержимого с помощью пакета SDK Служб мультимедиа для .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

[!INCLUDE [media-services-selector-content-key-auth-policy](../../../includes/media-services-selector-content-key-auth-policy.md)]

## <a name="overview"></a>Обзор
 Службы мультимедиа Azure позволяют защищать потоковое содержимое MPEG-DASH, Smooth Streaming и HTTP Live Streaming (HLS) с помощью стандарта AES (использующего 128-разрядные ключи шифрования) или технологии [управления цифровыми правами (DRM) PlayReady](https://www.microsoft.com/playready/overview/). Кроме того, Службы мультимедиа позволяют передавать потоки MPEG DASH с шифрованием Widevine DRM. PlayReady и Widevine шифруются согласно спецификации общего шифрования (ISO/IEC 23001-7 CENC).

Службы мультимедиа также включают в себя службы доставки ключей и лицензий, с помощью которых клиенты могут получать ключи AES либо лицензии PlayReady или Widevine для воспроизведения зашифрованного содержимого.

Если требуется, чтобы службы мультимедиа зашифровали ресурс, необходимо связать ключ шифрования (CommonEncryption или EnvelopeEncryption) с ресурсом. Дополнительные сведения см. в статье [Создание ContentKey с использованием .NET](media-services-dotnet-create-contentkey.md). Также потребуется настроить политики авторизации для ключа (как описано в этой статье).

Когда поток запрашивается проигрывателем, Службы мультимедиа, используя указанный ключ, выполняют динамическое шифрование содержимого с помощью AES или DRM. Чтобы расшифровать поток, проигрыватель запросит ключ у службы доставки ключей. Чтобы определить, есть ли у пользователя право на получение ключа, служба оценивает политики авторизации, заданные для ключа.

Службы мультимедиа поддерживают несколько способов аутентификации пользователей, которые запрашивают ключи. Политике авторизации ключа содержимого можно задать одно или несколько ограничений: открытая авторизация или авторизация с помощью токена. При ограничении с помощью маркера к политике должен прилагаться маркер, выданный службой маркеров безопасности (STS). Службы мультимедиа поддерживают токены в формате простого веб-маркера ([SWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_2)) и формате JSON Web Token ([JWT](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3)).

Службы мультимедиа не предоставляют службу маркеров безопасности. Вы можете создать настраиваемую службу STS или выдавать токены с помощью службы контроля доступа Azure. Чтобы создать маркер, подписанный указанным ключом, и получить утверждения, указанные в конфигурации ограничения по маркерам, должна быть настроена служба маркеров безопасности (как описано в этой статье). Если маркер является допустимым и утверждения маркера соответствуют утверждениям, настроенным для ключа содержимого, служба доставки ключей для служб мультимедиа возвращает клиенту ключ шифрования.

Дополнительные сведения см. в следующих статьях:

- [JWT token Authentication in Azure Media Services and Dynamic Encryption](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/) (Аутентификация токена JWT в службах мультимедиа Azure и динамическое шифрование)
- [Integrate Azure Media Services OWIN MVC based app with Azure Active Directory and restrict content key delivery based on JWT claims](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/) (Интеграция приложения на основе OWIN MVC Служб мультимедиа Azure с Azure Active Directory и ограничение доставки ключей содержимого на основе утверждений JWT)

### <a name="some-considerations-apply"></a>Важные особенности
* При создании учетной записи служб мультимедиа в нее добавляется конечная точка потоковой передачи по умолчанию в состоянии "Остановлена". Чтобы начать потоковую передачу содержимого и воспользоваться динамической упаковкой и динамическим шифрованием, конечная точка потоковой передачи должна находиться в состоянии "Выполняется". 
* Ресурс должен содержать набор MP4-файлов с адаптивной скоростью или файлов Smooth Streaming с адаптивной скоростью. Дополнительные сведения см. в статье о [кодировании ресурсов](media-services-encode-asset.md).
* Отправляйте и кодируйте ресурсы с помощью параметра AssetCreationOptions.StorageEncrypted.
* Если вы планируете использовать несколько ключей содержимого, для которых требуется одинаковая конфигурация политики, рекомендуем создать единую политику авторизации и повторно использовать ее с несколькими ключами содержимого.
* Служба доставки ключей кэширует политику ContentKeyAuthorizationPolicy и связанные с ней объекты (параметры и ограничения политики) за 15 минут. Можно создать ContentKeyAuthorizationPolicy и указать ограничение авторизации по маркеру, протестировать его, а затем обновить политику, задав открытое ограничение. Этот процесс занимает примерно 15 минут, после чего политика переключается на открытую версию.
* При добавлении или обновлении политики доставки ресурсов необходимо удалить все существующие указатели и создать новый.
* Сейчас невозможно шифровать последовательно скачиваемые данные.
* Конечная точка потоковой передачи Служб мультимедиа задает значение CORS-заголовка Access-Control-Allow-Origin в предварительном ответе как подстановочный знак "\*". Это значение подходит для большинства проигрывателей, включая Проигрыватель мультимедиа Azure, Roku, JWPlayer и др. Но некоторые проигрыватели, в которых используется dash.js, при таком сценарии не работают. Если режим учетных данных имеет значение include, запрос XMLHttpRequest в dash.js не допускает использования подстановочного знака "\*" в качестве значения Access-Control-Allow-Origin. Это ограничение в dash.js можно обойти. Если вы размещаете клиент из одного домена, в Службах мультимедиа Azure этот домен можно указать в заголовке предварительного ответа. Для получения помощи отправьте запрос в службу поддержки через портал Azure.

## <a name="aes-128-dynamic-encryption"></a>Динамическое шифрование AES-128
### <a name="open-restriction"></a>Ограничение открытого типа
Ограничение открытого типа означает, что система доставляет ключ всем, кто подает на него запрос. Это ограничение подходит для тестирования.

В следующем примере создается политика авторизации типа "открытая", которая затем добавляется в ключ содержимого:
```csharp
    static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
    {
        // Create ContentKeyAuthorizationPolicy with Open restrictions
        // and create authorization policy
        IContentKeyAuthorizationPolicy policy = _context.
        ContentKeyAuthorizationPolicies.
        CreateAsync("Open Authorization Policy").Result;
        
        List<ContentKeyAuthorizationPolicyRestriction> restrictions =
            new List<ContentKeyAuthorizationPolicyRestriction>();

        ContentKeyAuthorizationPolicyRestriction restriction =
            new ContentKeyAuthorizationPolicyRestriction
            {
                Name = "HLS Open Authorization Policy",
                KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
                Requirements = null // no requirements needed for HLS
            };

        restrictions.Add(restriction);

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create(
            "policy", 
            ContentKeyDeliveryType.BaselineHttp, 
            restrictions, 
            "");

        policy.Options.Add(policyOption);

        // Add ContentKeyAuthorizationPolicy to ContentKey
        contentKey.AuthorizationPolicyId = policy.Id;
        IContentKey updatedKey = contentKey.UpdateAsync().Result;
        Console.WriteLine("Adding Key to Asset: Key ID is " + updatedKey.Id);
    }
```

### <a name="token-restriction"></a>Ограничение по маркеру
В этом разделе рассказывается о том, как создать политику авторизации ключа содержимого и связать ее с ключом содержимого. Политика авторизации определяет, какие требования авторизации должны быть соблюдены, чтобы у пользователя было право на получение ключа. Например, должен ли список ключей проверки содержать ключ, с помощью которого был подписан маркер?

Чтобы настроить параметр ограничения маркера, необходимо использовать XML для описания требований к авторизации маркера. XML-файл конфигурации ограничений по маркеру должен соответствовать следующей схеме XML:
```csharp
#### Token restriction schema
    <?xml version="1.0" encoding="utf-8"?>
    <xs:schema xmlns:tns="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" elementFormDefault="qualified" targetNamespace="http://schemas.microsoft.com/Azure/MediaServices/KeyDelivery/TokenRestrictionTemplate/v1" xmlns:xs="https://www.w3.org/2001/XMLSchema">
      <xs:complexType name="TokenClaim">
        <xs:sequence>
          <xs:element name="ClaimType" nillable="true" type="xs:string" />
          <xs:element minOccurs="0" name="ClaimValue" nillable="true" type="xs:string" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="TokenClaim" nillable="true" type="tns:TokenClaim" />
      <xs:complexType name="TokenRestrictionTemplate">
        <xs:sequence>
          <xs:element minOccurs="0" name="AlternateVerificationKeys" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
          <xs:element name="Audience" nillable="true" type="xs:anyURI" />
          <xs:element name="Issuer" nillable="true" type="xs:anyURI" />
          <xs:element name="PrimaryVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
          <xs:element minOccurs="0" name="RequiredClaims" nillable="true" type="tns:ArrayOfTokenClaim" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="TokenRestrictionTemplate" nillable="true" type="tns:TokenRestrictionTemplate" />
      <xs:complexType name="ArrayOfTokenVerificationKey">
        <xs:sequence>
          <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="ArrayOfTokenVerificationKey" nillable="true" type="tns:ArrayOfTokenVerificationKey" />
      <xs:complexType name="TokenVerificationKey">
        <xs:sequence />
      </xs:complexType>
      <xs:element name="TokenVerificationKey" nillable="true" type="tns:TokenVerificationKey" />
      <xs:complexType name="ArrayOfTokenClaim">
        <xs:sequence>
          <xs:element minOccurs="0" maxOccurs="unbounded" name="TokenClaim" nillable="true" type="tns:TokenClaim" />
        </xs:sequence>
      </xs:complexType>
      <xs:element name="ArrayOfTokenClaim" nillable="true" type="tns:ArrayOfTokenClaim" />
      <xs:complexType name="SymmetricVerificationKey">
        <xs:complexContent mixed="false">
          <xs:extension base="tns:TokenVerificationKey">
            <xs:sequence>
              <xs:element name="KeyValue" nillable="true" type="xs:base64Binary" />
            </xs:sequence>
          </xs:extension>
        </xs:complexContent>
      </xs:complexType>
      <xs:element name="SymmetricVerificationKey" nillable="true" type="tns:SymmetricVerificationKey" />
    </xs:schema>
```
При настройке политики ограничения по маркеру необходимо задать такие параметры, как основной ключ проверки, издатель и аудитория. Основной ключ проверки содержит ключ, которым был подписан маркер. Издателем является служба STS, которая выдала маркер. Аудитория (иногда называется областью) описывает назначение маркера или ресурс, доступ к которому обеспечивает маркер. Служба доставки ключей служб мультимедиа проверяет, соответствуют ли эти значения в маркере значениям в шаблоне.

При использовании пакета SDK Служб мультимедиа для .NET создать токен ограничения можно с помощью класса TokenRestrictionTemplate.
В следующем примере создается политика авторизации с ограничением "по маркеру". В этом примере клиент должен предоставить токен, в котором содержатся: ключ подписывания (VerificationKey), поставщик токена и требуемые утверждения.
```csharp
    public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
    {
        string tokenTemplateString = GenerateTokenRequirements();

        IContentKeyAuthorizationPolicy policy = _context.
                                ContentKeyAuthorizationPolicies.
                                CreateAsync("HLS token restricted authorization policy").Result;

        List<ContentKeyAuthorizationPolicyRestriction> restrictions =
                new List<ContentKeyAuthorizationPolicyRestriction>();

        ContentKeyAuthorizationPolicyRestriction restriction =
                new ContentKeyAuthorizationPolicyRestriction
                {
                    Name = "Token Authorization Policy",
                    KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                    Requirements = tokenTemplateString
                };

        restrictions.Add(restriction);

        //You could have multiple options 
        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create(
                "Token option for HLS",
                ContentKeyDeliveryType.BaselineHttp,
                restrictions,
                null  // no key delivery data is needed for HLS
                );

        policy.Options.Add(policyOption);

        // Add ContentKeyAuthorizationPolicy to ContentKey
        contentKey.AuthorizationPolicyId = policy.Id;
        IContentKey updatedKey = contentKey.UpdateAsync().Result;
        Console.WriteLine("Adding Key to Asset: Key ID is " + updatedKey.Id);

        return tokenTemplateString;
    }

    static private string GenerateTokenRequirements()
    {
        TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

        template.PrimaryVerificationKey = new SymmetricVerificationKey();
        template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();

        template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

        return TokenRestrictionTemplateSerializer.Serialize(template);
    }
```
#### <a name="test-token"></a>Тестовый маркер
Чтобы получить токен тестирования на основе токена ограничения, который использовался для политики авторизации ключа, выполните приведенные ниже действия.
```csharp
    // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
    // back into a TokenRestrictionTemplate class instance.
    TokenRestrictionTemplate tokenTemplate =
        TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

    // Generate a test token based on the data in the given TokenRestrictionTemplate.
    // Note, you need to pass the key id Guid because we specified 
    // TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
    Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);

    //The GenerateTestToken method returns the token without the word “Bearer” in front
    //so you have to add it in front of the token string. 
    string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey);
    Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
    Console.WriteLine();
```

## <a name="playready-dynamic-encryption"></a>Динамическое шифрование PlayReady
С помощью служб мультимедиа можно настроить права и ограничения, которые должны применяться в среде выполнения PlayReady DRM при попытке пользователя воспроизвести защищенное содержимое. 

При защите содержимого с помощью PlayReady в политике авторизации среди прочего необходимо указать XML-строку, определяющую [шаблон лицензии PlayReady](media-services-playready-license-template-overview.md). Классы PlayReadyLicenseResponseTemplate и PlayReadyLicenseTemplate в пакете SDK Служб мультимедиа для .NET помогают определить шаблон лицензии PlayReady.

Сведения о шифровании содержимого на основе технологий PlayReady и Widevine см. в статье [Использование общего динамического шифрования PlayReady и (или) Widevine DRM](media-services-protect-with-playready-widevine.md).

### <a name="open-restriction"></a>Ограничение открытого типа
Ограничение открытого типа означает, что система доставляет ключ всем, кто подает на него запрос. Это ограничение подходит для тестирования.

В следующем примере создается политика авторизации типа "открытая", которая затем добавляется в ключ содержимого:

```csharp
    static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
    {

        // Create ContentKeyAuthorizationPolicy with Open restrictions 
        // and create authorization policy          

        List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
        {
            new ContentKeyAuthorizationPolicyRestriction 
            { 
                Name = "Open", 
                KeyRestrictionType = (int)ContentKeyRestrictionType.Open, 
                Requirements = null
            }
        };

        // Configure PlayReady license template.
        string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
                ContentKeyDeliveryType.PlayReadyLicense,
                    restrictions, newLicenseTemplate);

        IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                    ContentKeyAuthorizationPolicies.
                    CreateAsync("Deliver Common Content Key with no restrictions").
                    Result;


        contentKeyAuthorizationPolicy.Options.Add(policyOption);

        // Associate the content key authorization policy with the content key.
        contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
        contentKey = contentKey.UpdateAsync().Result;
    }
```

### <a name="token-restriction"></a>Ограничение по маркеру
Чтобы настроить параметр ограничения маркера, необходимо использовать XML для описания требований к авторизации маркера. XML-файл конфигурации ограничения по маркеру должен соответствовать схеме XML, показанной в разделе "Схема ограничения по маркеру".

```csharp
    public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
    {
        string tokenTemplateString = GenerateTokenRequirements();

        IContentKeyAuthorizationPolicy policy = _context.
                                ContentKeyAuthorizationPolicies.
                                CreateAsync("HLS token restricted authorization policy").Result;

        List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
        {
            new ContentKeyAuthorizationPolicyRestriction 
            { 
                Name = "Token Authorization Policy", 
                KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                Requirements = tokenTemplateString, 
            }
        };

        // Configure PlayReady license template.
        string newLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

        IContentKeyAuthorizationPolicyOption policyOption =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                ContentKeyDeliveryType.PlayReadyLicense,
                    restrictions, newLicenseTemplate);

        IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                    ContentKeyAuthorizationPolicies.
                    CreateAsync("Deliver Common Content Key with no restrictions").
                    Result;

        policy.Options.Add(policyOption);

        // Add ContentKeyAuthorizationPolicy to ContentKey
        contentKeyAuthorizationPolicy.Options.Add(policyOption);

        // Associate the content key authorization policy with the content key
        contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
        contentKey = contentKey.UpdateAsync().Result;

        return tokenTemplateString;
    }

    static private string GenerateTokenRequirements()
    {

        TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

        template.PrimaryVerificationKey = new SymmetricVerificationKey();
        template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();


        template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

        return TokenRestrictionTemplateSerializer.Serialize(template);
    } 

    static private string ConfigurePlayReadyLicenseTemplate()
    {
        // The following code configures PlayReady License Template using .NET classes
        // and returns the XML string.

        //The PlayReadyLicenseResponseTemplate class represents the template for the response sent back to the end user. 
        //It contains a field for a custom data string between the license server and the application 
        //(may be useful for custom app logic) as well as a list of one or more license templates.
        PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();

        // The PlayReadyLicenseTemplate class represents a license template for creating PlayReady licenses
        // to be returned to the end users. 
        //It contains the data on the content key in the license and any rights or restrictions to be 
        //enforced by the PlayReady DRM runtime when using the content key.
        PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
        //Configure whether the license is persistent (saved in persistent storage on the client) 
        //or non-persistent (only held in memory while the player is using the license).  
        licenseTemplate.LicenseType = PlayReadyLicenseType.Nonpersistent;

        // AllowTestDevices controls whether test devices can use the license or not.  
        // If true, the MinimumSecurityLevel property of the license
        // is set to 150.  If false (the default), the MinimumSecurityLevel property of the license is set to 2000.
        licenseTemplate.AllowTestDevices = true;


        // You can also configure the Play Right in the PlayReady license by using the PlayReadyPlayRight class. 
        // It grants the user the ability to play back the content subject to the zero or more restrictions 
        // configured in the license and on the PlayRight itself (for playback specific policy). 
        // Much of the policy on the PlayRight has to do with output restrictions 
        // which control the types of outputs that the content can be played over and 
        // any restrictions that must be put in place when using a given output.
        // For example, if the DigitalVideoOnlyContentRestriction is enabled, 
        //then the DRM runtime will only allow the video to be displayed over digital outputs 
        //(analog video outputs won’t be allowed to pass the content).

        //IMPORTANT: These types of restrictions can be very powerful but can also affect the consumer experience. 
        // If the output protections are configured too restrictive, 
        // the content might be unplayable on some clients. For more information, see the PlayReady Compliance Rules document.

        // For example:
        //licenseTemplate.PlayRight.AgcAndColorStripeRestriction = new AgcAndColorStripeRestriction(1);

        responseTemplate.LicenseTemplates.Add(licenseTemplate);

        return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
    }
```

Чтобы получить тестовый токен на основе токена ограничения, который использовался для политики авторизации ключа, см. [этот](#test-token) раздел. 

## <a name="types-used-when-you-define-contentkeyauthorizationpolicy"></a><a id="types"></a>Типы, используемые при определении ContentKeyAuthorizationPolicy
### <a name="contentkeyrestrictiontype"></a><a id="ContentKeyRestrictionType"></a>ContentKeyRestrictionType

```csharp
    public enum ContentKeyRestrictionType
    {
        Open = 0,
        TokenRestricted = 1,
        IPRestricted = 2,
    }
```

### <a name="contentkeydeliverytype"></a><a id="ContentKeyDeliveryType"></a>ContentKeyDeliveryType

```csharp 
    public enum ContentKeyDeliveryType
    {
      None = 0,
      PlayReadyLicense = 1,
      BaselineHttp = 2,
      Widevine = 3
    }
```

### <a name="tokentype"></a><a id="TokenType"></a>TokenType

```csharp
    public enum TokenType
    {
        Undefined = 0,
        SWT = 1,
        JWT = 2,
    }
```

## <a name="additional-notes"></a>Дополнительные замечания

* Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда политика авторизации для ключа содержимого настроена, перейдите к статье о [настройке политики доставки ресурсов](media-services-dotnet-configure-asset-delivery-policy.md).
