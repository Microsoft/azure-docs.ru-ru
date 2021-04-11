---
title: Использование служб мультимедиа Azure для доставки лицензий DRM или ключей AES | Документация Майкрософт
description: В этой статье описано, как с помощью Служб мультимедиа Azure доставлять лицензии PlayReady и (или) Widevine и ключи AES, а также как выполнять другие задачи (кодирование, шифрование, потоковая передача) на локальных серверах.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 8546c2c1-430b-4254-a88d-4436a83f9192
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: f2d965ec17fb605362c1e8cd8ef781a6bd2029d0
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106067390"
---
# <a name="use-media-services-to-deliver-drm-licenses-or-aes-keys"></a>Доставка лицензий DRM или ключей AES с помощью Служб мультимедиа

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

Службы мультимедиа Azure позволяют принимать и кодировать содержимое, добавлять функции защиты, а также выполнять его потоковую передачу. Дополнительные сведения см. в статье [Использование общего динамического шифрования PlayReady и (или) Widevine DRM](media-services-protect-with-playready-widevine.md). Некоторые клиенты с помощью Служб мультимедиа только доставляют лицензии и (или) ключи, а также выполняют кодирование, шифрование и потоковую передачу содержимого на локальных серверах. В этой статье описано, как с помощью Служб мультимедиа доставлять лицензии PlayReady и (или) Widevine, а также как выполнять другие задачи на локальных серверах. 

Для работы с этим учебником требуется учетная запись Azure. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/pricing/free-trial/).

## <a name="overview"></a>Обзор
Службы мультимедиа обеспечивают доставку лицензий PlayReady, лицензий управления цифровыми правами (DRM) Widevine, а также ключей AES-128. Они также предоставляют API-интерфейсы для настройки прав и ограничений, которые должны применяться в среде выполнения DRM, когда пользователь воспроизводит защищенное DRM содержимое. Когда пользователь запрашивает защищенное содержимое, приложение проигрывателя, в свою очередь, запрашивает лицензию из службы лицензий Служб мультимедиа. Если лицензию авторизовано, служба лицензий Служб мультимедиа выдает ее проигрывателю. Лицензии PlayReady и Widevine содержат ключ расшифровки, который может использоваться клиентским проигрывателем для расшифровки и потоковой передачи содержимого.

Службы мультимедиа поддерживают несколько способов авторизации пользователей, которые запрашивают лицензии или ключи. Политику авторизации ключа содержимого настраиваете вы. Такая политика может иметь одно или несколько ограничений, например открытая авторизация или авторизация с помощью токена. При ограничении с помощью маркера к политике должен прилагаться маркер, выданный службой маркеров безопасности (STS). Службы мультимедиа поддерживают токены в формате простого веб-маркера (SWT) и формате JSON Web Token (JWT).

На следующей схеме представлены основные действия, которые необходимо выполнить, чтобы доставлять лицензии PlayReady и Widevine с помощью Служб мультимедиа. При этом остальные задачи выполняются на локальных серверах:

![Защита с помощью PlayReady](./media/media-services-deliver-keys-and-licenses/media-services-diagram1.png)

## <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

1. Настройте среду разработки и укажите в файле app.config сведения о подключении, как описано в статье [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md).

2. Добавьте следующие элементы в **appSettings**, определенные в файле app.config:

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```
 
## <a name="net-code-example"></a>Пример кода .NET
В приведенном ниже примере кода показано создание общего ключа содержимого и получение URL-адресов для получения лицензий PlayReady или Widevine. Чтобы настроить локальный сервер, требуется ключ содержимого, идентификатор ключа и URL-адрес для приобретения лицензии. После настройки локального сервера вы можете выполнять потоковую передачу данных с собственного сервера потоковой передачи. Так как зашифрованный поток указывает на сервер лицензирования Служб мультимедиа, проигрыватель запрашивает лицензию из Служб мультимедиа. При выборе проверки подлинности с использованием токена сервер лицензирования Служб мультимедиа проверяет токен, отправленный по протоколу HTTPS. Если токен допустимый, сервер лицензирования доставляет лицензию обратно в проигрыватель. В приведенном ниже примере кода показано создание общего ключа содержимого и получение URL-адресов для получения лицензий PlayReady или Widevine. Если вам требуется доставка ключей AES-128, нужно создать ключ содержимого типа "конверт" и получить URL-адрес для получения ключа. Дополнительные сведения см. в статье [Использование динамического шифрования AES-128 и службы доставки ключей](media-services-playready-license-template-overview.md).

```csharp
using System;
using System.Collections.Generic;
using System.Configuration;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
using Microsoft.WindowsAzure.MediaServices.Client.Widevine;
using Newtonsoft.Json;

namespace DeliverDRMLicenses
{
    class Program
    {
        // Read values from the App.config file.
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static readonly Uri _sampleIssuer =
            new Uri(ConfigurationManager.AppSettings["Issuer"]);
        private static readonly Uri _sampleAudience =
            new Uri(ConfigurationManager.AppSettings["Audience"]);

        // Field for service context.
        private static CloudMediaContext _context = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            bool tokenRestriction = true;
            string tokenTemplateString = null;


            IContentKey key = CreateCommonTypeContentKey();

            // Print out the key ID and Key in base64 string format
            Console.WriteLine("Created key {0} with key value {1} ",
                key.Id, System.Convert.ToBase64String(key.GetClearKeyValue()));

            Console.WriteLine("PlayReady License Key delivery URL: {0}",
                key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense));

            Console.WriteLine("Widevine License Key delivery URL: {0}",
                key.GetKeyDeliveryUrl(ContentKeyDeliveryType.Widevine));

            if (tokenRestriction)
                tokenTemplateString = AddTokenRestrictedAuthorizationPolicy(key);
            else
                AddOpenAuthorizationPolicy(key);

            Console.WriteLine("Added authorization policy: {0}",
                key.AuthorizationPolicyId);
            Console.WriteLine();
            Console.ReadLine();
        }

        static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
        {

            // Create ContentKeyAuthorizationPolicy with Open restrictions 
            // and create authorization policy          

            List<ContentKeyAuthorizationPolicyRestriction> restrictions =
                new List<ContentKeyAuthorizationPolicyRestriction>
            {
                        new ContentKeyAuthorizationPolicyRestriction
                        {
                            Name = "Open",
                            KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
                            Requirements = null
                        }
            };

            // Configure PlayReady and Widevine license templates.
            string PlayReadyLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

            string WidevineLicenseTemplate = ConfigureWidevineLicenseTemplate();

            IContentKeyAuthorizationPolicyOption PlayReadyPolicy =
                _context.ContentKeyAuthorizationPolicyOptions.Create("",
                    ContentKeyDeliveryType.PlayReadyLicense,
                        restrictions, PlayReadyLicenseTemplate);

            IContentKeyAuthorizationPolicyOption WidevinePolicy =
                _context.ContentKeyAuthorizationPolicyOptions.Create("",
                    ContentKeyDeliveryType.Widevine,
                    restrictions, WidevineLicenseTemplate);

            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                        ContentKeyAuthorizationPolicies.
                        CreateAsync("Deliver Common Content Key with no restrictions").
                        Result;


            contentKeyAuthorizationPolicy.Options.Add(PlayReadyPolicy);
            contentKeyAuthorizationPolicy.Options.Add(WidevinePolicy);
            // Associate the content key authorization policy with the content key.
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;
        }

        public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
        {
            string tokenTemplateString = GenerateTokenRequirements();

            List<ContentKeyAuthorizationPolicyRestriction> restrictions =
                new List<ContentKeyAuthorizationPolicyRestriction>
            {
                        new ContentKeyAuthorizationPolicyRestriction
                        {
                            Name = "Token Authorization Policy",
                            KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                            Requirements = tokenTemplateString,
                        }
            };

            // Configure PlayReady and Widevine license templates.
            string PlayReadyLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

            string WidevineLicenseTemplate = ConfigureWidevineLicenseTemplate();

            IContentKeyAuthorizationPolicyOption PlayReadyPolicy =
                _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                    ContentKeyDeliveryType.PlayReadyLicense,
                        restrictions, PlayReadyLicenseTemplate);

            IContentKeyAuthorizationPolicyOption WidevinePolicy =
                _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                    ContentKeyDeliveryType.Widevine,
                        restrictions, WidevineLicenseTemplate);

            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                        ContentKeyAuthorizationPolicies.
                        CreateAsync("Deliver Common Content Key with token restrictions").
                        Result;

            contentKeyAuthorizationPolicy.Options.Add(PlayReadyPolicy);
            contentKeyAuthorizationPolicy.Options.Add(WidevinePolicy);

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

            //The PlayReadyLicenseResponseTemplate class represents the template 
            //for the response sent back to the end user. 
            //It contains a field for a custom data string between the license server 
            //and the application (may be useful for custom app logic) 
            //as well as a list of one or more license templates.

            PlayReadyLicenseResponseTemplate responseTemplate =
                new PlayReadyLicenseResponseTemplate();

            // The PlayReadyLicenseTemplate class represents a license template 
            // for creating PlayReady licenses
            // to be returned to the end users. 
            // It contains the data on the content key in the license 
            // and any rights or restrictions to be 
            // enforced by the PlayReady DRM runtime when using the content key.
            PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();

            // Configure whether the license is persistent 
            // (saved in persistent storage on the client) 
            // or non-persistent (only held in memory while the player is using the license).  
            licenseTemplate.LicenseType = PlayReadyLicenseType.Nonpersistent;

            // AllowTestDevices controls whether test devices can use the license or not.  
            // If true, the MinimumSecurityLevel property of the license
            // is set to 150.  If false (the default), 
            // the MinimumSecurityLevel property of the license is set to 2000.
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

            // IMPORTANT: These types of restrictions can be very powerful 
            // but can also affect the consumer experience. 
            // If the output protections are configured too restrictive, 
            // the content might be unplayable on some clients. 
            // For more information, see the PlayReady Compliance Rules document.

            // For example:
            //licenseTemplate.PlayRight.AgcAndColorStripeRestriction = new AgcAndColorStripeRestriction(1);

            responseTemplate.LicenseTemplates.Add(licenseTemplate);

            return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
        }


        private static string ConfigureWidevineLicenseTemplate()
        {
            var template = new WidevineMessage
            {
                allowed_track_types = AllowedTrackTypes.SD_HD,
                content_key_specs = new[]
                {
                            new ContentKeySpecs
                            {
                                required_output_protection =
                                    new RequiredOutputProtection { hdcp = Hdcp.HDCP_NONE},
                                security_level = 1,
                                track_type = "SD"
                            }
                        },
                policy_overrides = new
                {
                    can_play = true,
                    can_persist = true,
                    can_renew = false
                }
            };

            string configuration = JsonConvert.SerializeObject(template);
            return configuration;
        }


        static public IContentKey CreateCommonTypeContentKey()
        {
            // Create envelope encryption content key
            Guid keyId = Guid.NewGuid();
            byte[] contentKey = GetRandomBuffer(16);

            IContentKey key = _context.ContentKeys.Create(
                                    keyId,
                                    contentKey,
                                    "ContentKey",
                                    ContentKeyType.CommonEncryption);

            return key;
        }

        static private byte[] GetRandomBuffer(int length)
        {
            var returnValue = new byte[length];

            using (var rng =
                new System.Security.Cryptography.RNGCryptoServiceProvider())
            {
                rng.GetBytes(returnValue);
            }

            return returnValue;
        }
    }
}
```

## <a name="additional-notes"></a>Дополнительные замечания

* Widevine — это служба, которая предоставляется компанией Google Inc. и подпадает под условия предоставления услуг и политику конфиденциальности Google Inc.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>См. также раздел
* [Использование общего динамического шифрования PlayReady и (или) Widevine DRM](media-services-protect-with-playready-widevine.md)
* [Использование динамического шифрования AES-128 и службы доставки ключей](media-services-playready-license-template-overview.md)
