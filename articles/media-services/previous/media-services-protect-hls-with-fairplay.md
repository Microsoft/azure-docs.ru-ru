---
title: Защита содержимого HLS с помощью Microsoft PlayReady или Apple FairPlay в Azure | Документация Майкрософт
description: В этом разделе приводятся общие сведения и показывается, как использовать службы мультимедиа Azure для динамического шифрования содержимого HTTP Live Streaming (HLS) с использованием Apple FairPlay. Здесь же рассказывается, как использовать службу доставки лицензий для служб мультимедиа для доставки клиентам лицензий FairPlay.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 15aab28b7dfbaf305412f1080346b54cc6827437
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103009646"
---
# <a name="protect-your-hls-content-with-apple-fairplay-or-microsoft-playready"></a>Защита содержимого HLS с помощью Apple FairPlay или Microsoft PlayReady

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Для работы с этим учебником требуется учетная запись Azure. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/pricing/free-trial/).   > новые функции или функции не добавляются в службы мультимедиа версии 2. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).
>

Службы мультимедиа Azure позволяют использовать динамическое шифрование содержимого HTTP Live Streaming (HLS) в следующих форматах:  

* **Незащищенный ключ конверта AES-128**

    Весь фрагмент шифруется в режиме **AES-128 CBC**. Проигрыватели iOS и OS X поддерживают расшифровку потока по умолчанию. Дополнительные сведения см. [в статье Использование динамического шифрования AES-128 и службы доставки ключей](media-services-protect-with-aes128.md).
* **Apple FairPlay**

    Отдельные образцы видео и аудиоданных шифруются в режиме **AES-128 CBC** . **Потоковая передача FairPlay** (FPS): интегрируется в операционные системы устройств и поддерживается iOS и Apple TV по умолчанию. Включение FPS в Safari (OS X) реализуется благодаря поддержке интерфейса расширений зашифрованных носителей (EME).
* **Microsoft PlayReady**

На следующем изображении показан рабочий процесс **динамического шифрования HLS + FairPlay или PlayReady**.

![Схема рабочего процесса динамического шифрования](./media/media-services-content-protection-overview/media-services-content-protection-with-FairPlay.png)

В этой статье показано, как использовать службы мультимедиа для динамического шифрования содержимого HLS с помощью Apple FairPlay. Здесь же рассказывается, как использовать службу доставки лицензий для служб мультимедиа для доставки клиентам лицензий FairPlay.

> [!NOTE]
> Если также требуется шифрование содержимого HLS с помощью PlayReady, необходимо создать общий ключ содержимого и связать его с ресурсом-контейнером. Также необходимо настроить политику авторизации ключей содержимого, как описано в статье [Использование общего динамического шифрования PlayReady и (или) Widevine DRM](media-services-protect-with-playready-widevine.md).
>
>

## <a name="requirements-and-considerations"></a>Требования и рекомендации

Для доставки HLS с шифрованием FairPlay и лицензий FairPlay с помощью служб мультимедиа необходимо следующее:

  * Учетная запись Azure. Дополнительные сведения см. в статье [Бесплатная пробная версия Azure](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F).
  * Учетная запись служб мультимедиа. Чтобы создать эту учетную запись, см. статью [Создание учетной записи служб мультимедиа Azure с помощью портала Azure](media-services-portal-create-account.md).
  * Регистрация в [программе разработки Apple](https://developer.apple.com/).
  * Корпорация Apple требует от владельца содержимого получить [пакет развертывания](https://developer.apple.com/contact/fps/). Подтвердите, что у вас реализован модуль безопасности ключа (KSM) с использованием служб мультимедиа Azure и что вы подали запрос на окончательный пакет FPS. Окончательный пакет FPS содержит инструкции по сертификации и получению секретного ключа приложения (ASK). ASK используется для настройки FairPlay.
  * Пакет SDK служб мультимедиа Azure для .NET **3.6.0** или более поздней версии.

Перед доставкой ключей служб мультимедиа должны быть настроены следующие параметры.

  * **Сертификат приложения (AC)**  — это PFX-файл, который содержит закрытый ключ. Создайте этот файл и зашифруйте его с помощью пароля.

       Настраивая политику доставки ключей, укажите пароль и PFX-файл в формате Base64.

      В следующих действиях описано, как создать PFX-файл сертификата для FairPlay.

    1. Установите OpenSSL со страницы https://slproweb.com/products/Win32OpenSSL.html.

        Перейдите в папку, где расположены сертификат FairPlay и другие файлы, полученные от Apple.
    2. Выполните следующую команду из командной строки. Она преобразует CER-файл в PEM-файл.

        "C:\OpenSSL-Win32\bin\openssl.exe" x509 -inform der -in FairPlay.cer -out FairPlay-out.pem
    3. Выполните следующую команду из командной строки. Она преобразует PEM-файл в PFX-файл, который содержит закрытый ключ. Затем OpenSSL запросит пароль для PFX-файла.

        "C:\OpenSSL-Win32\bin\openssl.exe" pkcs12 -export -out FairPlay-out.pfx -inkey privatekey.pem -in FairPlay-out.pem -passin file:privatekey-pem-pass.txt
  * **Пароль к сертификату приложения** — пароль для создания PFX-файла.
  * **Идентификатор пароля сертификата приложения** — отправьте пароль, точно так же, как вы отправляете другие ключи служб мультимедиа. Используйте значение перечисления **ContentKeyType.FairPlayPfxPassword**, чтобы получить идентификатор служб мультимедиа. Это идентификатор, который нужно использовать в параметре политики доставки ключей.
  * **iv** — случайное 16-байтовое значение. Оно должно соответствовать значению "IV" в политике доставки файлов. Создайте значение "IV" и присвойте его параметру политики доставки файлов и параметру политики доставки ключей.
  * **ASK** — ключ, полученный при создании сертификата с использованием портала разработчиков Apple. Каждая группа разработчиков получает уникальный ASK. Сохраните копию ASK в безопасном расположении. Впоследствии ASK необходимо настроить как FairPlayAsk для служб мультимедиа.
  * **Идентификатор ASK** — этот идентификатор будет получен при отправке ASK в службы мультимедиа. Отправьте ASK с помощью значения перечисления **ContentKeyType.FairPlayAsk**. После этого возвращается идентификатор служб мультимедиа, который будет использован при настройке параметра политики доставки ключей.

На стороне клиента FPS должны быть заданы следующие значения:

  * **Сертификат приложения (AC)**  — это CER-файл или DER-файл, содержащий открытый ключ, который операционная система использует для шифрования некоторых полезных данных. Службы мультимедиа следует связать с этим сертификатом, так как он используется проигрывателем. Служба доставки ключей расшифровывает их, используя соответствующий закрытый ключ.

Для воспроизведения зашифрованного потока FairPlay получите реальный ASK, а затем создайте реальный сертификат. В результате этого процесса создаются три элемента:

  * DER-файл;
  * PFX-файл;
  * пароль для PFX-файла.

Эти клиенты поддерживают HLS с шифрованием **AES-128 CBC**: Safari в OS X, Apple TV, iOS.

## <a name="configure-fairplay-dynamic-encryption-and-license-delivery-services"></a>Настройка общего динамического шифрования FairPlay и службы доставки лицензий
Ниже описаны общие действия, которые необходимо выполнить для защиты файлов с помощью FairPlay, используя службу доставки лицензий служб мультимедиа и динамическое шифрование.

1. Создайте ресурс и отправьте в него другие файлы.
2. Закодируйте ресурс с файлами в набор MP4-файлов с переменной скоростью.
3. Создайте ключ содержимого и свяжите его с закодированным ресурсом.  
4. Настройте политику авторизации для ключа содержимого. Укажите следующее.

   * Способ доставки (в данном случае FairPlay).
   * Конфигурация параметров политики FairPlay. Дополнительные сведения о настройке FairPlay см. ниже в примере метода **ConfigureFairPlayPolicyOptions()**.

     > [!NOTE]
     > Обычно настраивать параметры политики FairPlay необходимо только один раз, так как у вас будет один набор сертификации и ASK.
     >
     >
   * Ограничения (открытая авторизация или с помощью маркера).
   * Сведения о типе доставки ключей, определяющего порядок доставки ключа к клиенту.
5. Настройте политику доставки ресурсов. Конфигурация политики доставки включает:

   * протокол доставки (HLS);
   * тип динамического шифрования (стандартное шифрование CBC);
   * URL-адрес для приобретения лицензии.

     > [!NOTE]
     > Если вам требуется доставка потока данных, зашифрованного с использованием FairPlay и другой системы управления цифровыми правами, необходимо настроить отдельные политики доставки:
     >
     > * одна политика IAssetDeliveryPolicy для настройки динамической адаптивной потоковой передачи через HTTP (DASH) с общим шифрованием (CENC) (PlayReady + Widevine) и Smooth с PlayReady;
     > * другая политика IAssetDeliveryPolicy для настройки FairPlay для HLS.
     >
     >
6. Создайте указатель OnDemand, чтобы получить URL-адрес для потоковой передачи.

## <a name="use-fairplay-key-delivery-by-player-apps"></a>Использование доставки ключей FairPlay приложениями проигрывателей
С помощью пакета SDK можно разрабатывать приложения проигрывателя. Для воспроизведения содержимого FairPlay необходимо реализовать протокол обмена лицензиями. Этот протокол не указывается компанией Apple. Каждое приложение может само выбирать, как отправлять запросы доставки ключей. Служба доставки ключей FairPlay служб мультимедиа ожидает, что SPC поступит как сообщение публикации www-form-url в следующем формате:

`spc=<Base64 encoded SPC>`

> [!NOTE]
> Проигрыватель мультимедиа Azure поддерживает воспроизведение FairPlay. Дополнительные сведения см. в [документации проигрывателя мультимедиа Azure](https://amp.azure.net/libs/amp/latest/docs/index.html).
>
>

## <a name="streaming-urls"></a>URL-адреса потоковой передачи
Если ресурс-контейнер был зашифрован с помощью нескольких систем DRM, то в URL-адресе для потоковой передачи следует использовать тег шифрования: (format='m3u8-aapl', encryption='xxx').

Действительны следующие условия.

* Можно указать только один тип шифрования или ни одного.
* Если к файлу был применен только один тип шифрования, в URL-адресе тип шифрования можно не указывать.
* Тип шифрования вводится без учета регистра.
* Можно указать следующие типы шифрования:  
  * **Cenc**: общее шифрование (PlayReady или Widevine)
  * **CBCS-AAPL**: Fairplay
  * **CBC**: шифрование конверта AES

## <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

1. Настройте среду разработки и заполните файл app.config данными о соединении, как описано в разделе [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md). 
2. Добавьте следующие элементы в **appSettings**, определенные в файле app.config:

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

## <a name="example"></a>Пример

Следующий пример демонстрирует использование служб мультимедиа для доставки содержимого, зашифрованного с помощью FairPlay. Эта функция появилась в пакете SDK служб мультимедиа Azure для .NET версии 3.6.0. 

Замените код в файле Program.cs кодом, приведенным в этом разделе.

>[!NOTE]
>Действует ограничение в 1 000 000 записей для разных политик AMS (например, для политики Locator или ContentKeyAuthorizationPolicy). Следует указывать один и тот же идентификатор политики, если вы используете те же дни, разрешения доступа и т. д. Например, политики для указателей, которые должны оставаться на месте в течение длительного времени (не политики передачи). Дополнительные сведения см. в [этой статье](media-services-dotnet-manage-entities.md#limit-access-policies).

Обязательно обновите переменные, чтобы они указывали на папки, в которых находятся входные файлы.

```csharp
using System;
using System.Collections.Generic;
using System.Configuration;
using System.IO;
using System.Linq;
using System.Threading;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
using Microsoft.WindowsAzure.MediaServices.Client.FairPlay;
using Newtonsoft.Json;
using System.Security.Cryptography.X509Certificates;

namespace DynamicEncryptionWithFairPlay
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

        private static readonly string _mediaFiles =
            Path.GetFullPath(@"../..\Media");

        private static readonly string _singleMP4File =
            Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            bool tokenRestriction = false;
            string tokenTemplateString = null;

            IAsset asset = UploadFileAndCreateAsset(_singleMP4File);
            Console.WriteLine("Uploaded asset: {0}", asset.Id);

            IAsset encodedAsset = EncodeToAdaptiveBitrateMP4Set(asset);
            Console.WriteLine("Encoded asset: {0}", encodedAsset.Id);

            IContentKey key = CreateCommonCBCTypeContentKey(encodedAsset);
            Console.WriteLine("Created key {0} for the asset {1} ", key.Id, encodedAsset.Id);
            Console.WriteLine("FairPlay License Key delivery URL: {0}", key.GetKeyDeliveryUrl(ContentKeyDeliveryType.FairPlay));
            Console.WriteLine();

            if (tokenRestriction)
                tokenTemplateString = AddTokenRestrictedAuthorizationPolicy(key);
            else
                AddOpenAuthorizationPolicy(key);

            Console.WriteLine("Added authorization policy: {0}", key.AuthorizationPolicyId);
            Console.WriteLine();

            CreateAssetDeliveryPolicy(encodedAsset, key);
            Console.WriteLine("Created asset delivery policy. \n");
            Console.WriteLine();

            if (tokenRestriction && !String.IsNullOrEmpty(tokenTemplateString))
            {
                // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
                // back into a TokenRestrictionTemplate class instance.
                TokenRestrictionTemplate tokenTemplate =
                    TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

                // Generate a test token based on the data in the given TokenRestrictionTemplate.
                // Note, you need to pass the key id Guid because we specified
                // TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
                Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);
                string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey,
                                            DateTime.UtcNow.AddDays(365));
                Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
                Console.WriteLine();
            }

            string url = GetStreamingOriginLocator(encodedAsset);
            Console.WriteLine("Encrypted HLS URL: {0}/manifest(format=m3u8-aapl)", url);

            Console.ReadLine();
        }

        static public IAsset UploadFileAndCreateAsset(string singleFilePath)
        {
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, AssetCreationOptions.None);

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return inputAsset;
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset inputAsset)
        {
            var encodingPreset = "Adaptive Streaming";

            IJob job = _context.Jobs.Create(String.Format("Encoding {0}", inputAsset.Name));

            var mediaProcessors =
            _context.MediaProcessors.Where(p => p.Name.Contains("Media Encoder Standard")).ToList();

            var latestMediaProcessor =
            mediaProcessors.OrderBy(mp => new Version(mp.Version)).LastOrDefault();

            ITask encodeTask = job.Tasks.AddNew("Encoding", latestMediaProcessor, encodingPreset, TaskOptions.None);
            encodeTask.InputAssets.Add(inputAsset);
            encodeTask.OutputAssets.AddNew(String.Format("{0} as {1}", inputAsset.Name, encodingPreset), AssetCreationOptions.StorageEncrypted);

            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
            job.Submit();
            job.GetExecutionProgressTask(CancellationToken.None).Wait();

            return job.OutputMediaAssets[0];
        }

        static public IContentKey CreateCommonCBCTypeContentKey(IAsset asset)
        {
            // Create HLS SAMPLE AES encryption content key
            Guid keyId = Guid.NewGuid();
            byte[] contentKey = GetRandomBuffer(16);

            IContentKey key = _context.ContentKeys.Create(
                        keyId,
                        contentKey,
                        "ContentKey",
                        ContentKeyType.CommonEncryptionCbcs);

            // Associate the key with the asset.
            asset.ContentKeys.Add(key);

            return key;
        }


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


            // Configure FairPlay policy option.
            string FairPlayConfiguration = ConfigureFairPlayPolicyOptions();

            IContentKeyAuthorizationPolicyOption FairPlayPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
            ContentKeyDeliveryType.FairPlay,
            restrictions,
            FairPlayConfiguration);


            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common CBC Content Key with no restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(FairPlayPolicy);

            // Associate the content key authorization policy with the content key.
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;
        }

        public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
        {
            string tokenTemplateString = GenerateTokenRequirements();

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
                    {
                    new ContentKeyAuthorizationPolicyRestriction
                    {
                        Name = "Token Authorization Policy",
                        KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                        Requirements = tokenTemplateString,
                    }
                    };

            // Configure FairPlay policy option.
            string FairPlayConfiguration = ConfigureFairPlayPolicyOptions();


            IContentKeyAuthorizationPolicyOption FairPlayPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                   ContentKeyDeliveryType.FairPlay,
                   restrictions,
                   FairPlayConfiguration);

            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common CBC Content Key with token restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(FairPlayPolicy);

            // Associate the content key authorization policy with the content key
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;

            return tokenTemplateString;
        }

        private static string ConfigureFairPlayPolicyOptions()
        {
            // For testing you can provide all zeroes for ASK bytes together with the cert from Apple FPS SDK.
            // However, for production you must use a real ASK from Apple bound to a real prod certificate.
            byte[] askBytes = Guid.NewGuid().ToByteArray();
            var askId = Guid.NewGuid();
            // Key delivery retrieves askKey by askId and uses this key to generate the response.
            IContentKey askKey = _context.ContentKeys.Create(
                        askId,
                        askBytes,
                        "askKey",
                        ContentKeyType.FairPlayASk);

            //Customer password for creating the .pfx file.
            string pfxPassword = "<customer password for creating the .pfx file>";
            // Key delivery retrieves pfxPasswordKey by pfxPasswordId and uses this key to generate the response.
            var pfxPasswordId = Guid.NewGuid();
            byte[] pfxPasswordBytes = System.Text.Encoding.UTF8.GetBytes(pfxPassword);
            IContentKey pfxPasswordKey = _context.ContentKeys.Create(
                        pfxPasswordId,
                        pfxPasswordBytes,
                        "pfxPasswordKey",
                        ContentKeyType.FairPlayPfxPassword);

            // iv - 16 bytes random value, must match the iv in the asset delivery policy.
            byte[] iv = Guid.NewGuid().ToByteArray();

            //Specify the .pfx file created by the customer.
            var appCert = new X509Certificate2("path to the .pfx file created by the customer", pfxPassword, X509KeyStorageFlags.Exportable);

            string FairPlayConfiguration =
            Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
                appCert,
                pfxPassword,
                pfxPasswordId,
                askId,
                iv);

            return FairPlayConfiguration;
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

        static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
        {
            var kdPolicy = _context.ContentKeyAuthorizationPolicies.Where(p => p.Id == key.AuthorizationPolicyId).Single();

            var kdOption = kdPolicy.Options.Single(o => o.KeyDeliveryType == ContentKeyDeliveryType.FairPlay);

            FairPlayConfiguration configFP = JsonConvert.DeserializeObject<FairPlayConfiguration>(kdOption.KeyDeliveryConfiguration);

            // Get the FairPlay license service URL.
            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.FairPlay);

            // The reason the below code replaces "https://" with "skd://" is because
            // in the IOS player sample code which you obtained in Apple developer account,
            // the player only recognizes a Key URL that starts with skd://.
            // However, if you are using a customized player,
            // you can choose whatever protocol you want.
            // For example, "https".

            Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
            {
                    {AssetDeliveryPolicyConfigurationKey.FairPlayLicenseAcquisitionUrl, acquisitionUrl.ToString().Replace("https://", "skd://")},
                    {AssetDeliveryPolicyConfigurationKey.CommonEncryptionIVForCbcs, configFP.ContentEncryptionIV}
            };

            var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryptionCbcs,
            AssetDeliveryProtocol.HLS,
            assetDeliveryPolicyConfiguration);

            // Add AssetDelivery Policy to the asset
            asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        }


        /// <summary>
        /// Gets the streaming origin locator.
        /// </summary>
        /// <param name="assets"></param>
        /// <returns></returns>
        static public string GetStreamingOriginLocator(IAsset asset)
        {

            // Get a reference to the streaming manifest file from the  
            // collection of files in the asset.

            var assetFile = asset.AssetFiles.LoList().Where(f => f.Name.ToLower().
                         EndsWith(".ism")).
                         FirstOrDefault();

            // Create a 30-day readonly access policy.
            IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

            // Create a locator to the streaming content on an origin.
            ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

            // Create a URL to the manifest file.
            return originLocator.Path + assetFile.Name;
        }

        static private void JobStateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine(string.Format("{0}\n  State: {1}\n  Time: {2}\n\n",
            ((IJob)sender).Name,
            e.CurrentState,
            DateTime.UtcNow.ToString(@"yyyy_M_d__hh_mm_ss")));
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

## <a name="next-steps-media-services-learning-paths"></a>Дальнейшие действия: схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
