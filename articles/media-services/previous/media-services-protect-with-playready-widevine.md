---
title: Использование общего динамического шифрования PlayReady и (или) Widevine DRM | Документация Майкрософт
description: Службы мультимедиа Microsoft Azure можно использовать для защиты потокового содержимого, которое доставляется по протоколам MPEG-DASH, Smooth Streaming и HTTP Live Streaming (HLS), с помощью технологии Microsoft PlayReady DRM. Кроме того, вы можете использовать их, чтобы передавать DASH с шифрованием, выполненным с помощью Widevine DRM. Здесь описывается выполнение динамического шифрования с помощью PlayReady и Widevine DRM.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 548d1a12-e2cb-45fe-9307-4ec0320567a2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 9a0e8a513023c093157a3f14a64de9f87de543ab
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103009459"
---
# <a name="use-playready-andor-widevine-dynamic-common-encryption"></a>Использование общего динамического шифрования PlayReady и (или) Widevine DRM

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Для работы с этим учебником требуется учетная запись Azure. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/pricing/free-trial/).   > новые функции или функции не добавляются в службы мультимедиа версии 2. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).
>   

## <a name="overview"></a>Обзор

 Службы мультимедиа можно использовать для защиты потокового содержимого, которое доставляется по протоколам MPEG-DASH, Smooth Streaming и HTTP Live Streaming (HLS), с помощью [технологии управления цифровыми правами (DRM) PlayReady](https://www.microsoft.com/playready/overview/). Вы также можете передавать зашифрованное потоковое содержимое DASH с использованием лицензий Widevine DRM. PlayReady и Widevine шифруются согласно спецификации общего шифрования (ISO/IEC 23001-7 CENC). Настроить AssetDeliveryConfiguration для использования Widevine можно с помощью пакета [SDK .NET для служб мультимедиа](https://www.nuget.org/packages/windowsazure.mediaservices/) (начиная с версии 3.5.1) или REST API.

Службы мультимедиа обеспечивают доставку лицензий PlayReady и DRM Widevine. Они также предоставляют API-интерфейсы, которые вы можете использовать для настройки прав и ограничений, применяемых в среде выполнения PlayReady или Widevine DRM, когда пользователь воспроизводит защищенное содержимое. Когда пользователь запрашивает защищенное содержимое DRM, приложение проигрывателя в свою очередь запрашивает лицензию из службы лицензий служб мультимедиа. Если приложение проигрывателя авторизовано, служба лицензий служб мультимедиа выдает лицензию для проигрывателя. Лицензия PlayReady или Widevine содержит ключ расшифровки, который может использоваться клиентским проигрывателем для расшифровки и потоковой передачи содержимого.

Кроме того, предоставить лицензии Widevine могут следующие партнеры служб мультимедиа: 

* [Axinom](https://www.axinom.com) 
* [EZDRM](https://ezdrm.com/); 
* [castLabs](https://castlabs.com/company/partners/azure/) 

Дополнительные сведения см. в разделе Интеграция с [Axinom](media-services-axinom-integration.md) и [castLabs](media-services-castlabs-integration.md).

Службы мультимедиа поддерживают несколько способов авторизации пользователей, которые запрашивают ключи. Для политики авторизации ключа содержимого можно задать одно или несколько ограничений: открытая авторизация или авторизация с помощью маркера. При ограничении с помощью маркера к политике должен прилагаться маркер, выданный службой маркеров безопасности (STS). Службы мультимедиа поддерживают маркеры в форматах [простого веб-маркера](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_2) (SWT) и [JSON Web Token](/previous-versions/azure/azure-services/gg185950(v=azure.100)#BKMK_3) (JWT). 

Дополнительные сведения см. [в статье Настройка политики авторизации для ключа содержимого](media-services-protect-with-aes128.md#configure_key_auth_policy).

Чтобы воспользоваться преимуществами динамического шифрования, требуется ресурс, содержащий набор многоскоростных MP4-файлов или многоскоростных исходных файлов Smooth Streaming. Вам также потребуется настроить политику доставки для ресурса (описывается далее в этой статье). В зависимости от формата, указанного в URL-адресе потоковой передачи, сервер потокового воспроизведения по запросу обеспечивает доставку содержимого по выбранному протоколу. Благодаря этому можно хранить и оплачивать файлы в одном формате хранения. Службы мультимедиа создают и обрабатывают соответствующий ответ HTTP на каждый запрос клиента.

Эта статья полезна для разработчиков приложений, которые предоставляют мультимедийные файлы, защищенные с помощью нескольких лицензий DRM, например PlayReady и Widevine. В этой статье объясняется, как настроить политики авторизации для службы доставки лицензий PlayReady, чтобы только авторизованные клиенты могли получать лицензии PlayReady или Widevine. Также показано, как использовать динамическое шифрование с помощью PlayReady или Widevine DRM через DASH.

>[!NOTE]
>При создании учетной записи служб мультимедиа в нее добавляется конечная точка потоковой передачи по умолчанию в состоянии "Остановлена". Чтобы начать потоковую передачу содержимого и воспользоваться динамической упаковкой и динамическим шифрованием, конечная точка потоковой передачи, из которой необходимо выполнять потоковую передачу содержимого, должна находиться в состоянии "Выполняется". 

## <a name="download-the-sample"></a>Скачивание примера приложения
Пример, описанный в этой статье, можно скачать на странице с [примерами Azure в GitHub](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm).

## <a name="configure-dynamic-common-encryption-and-drm-license-delivery-services"></a>Настройка общего динамического шифрования и службы доставки лицензий DRM

Выполните описанные ниже общие действия для защиты файлов с помощью PlayReady, используя службу доставки лицензий служб мультимедиа и динамическое шифрование.

1. Создайте ресурс и отправьте в него другие файлы.

2. Закодируйте ресурс с файлами в набор MP4-файлов с переменной скоростью.

3. Создайте ключ содержимого и свяжите его с закодированным ресурсом. В службах мультимедиа ключ содержимого содержит ключ шифрования ресурса.

4. Настройте политику авторизации ключа содержимого. Потребуется настроить политику авторизации для ключа содержимого. Прежде чем ключ содержимого будет доставлен в клиент, его нужно привести в соответствие с политикой.

    Когда вы создаете политику авторизации ключа содержимого, задайте метод доставки (PlayReady или Widevine) и ограничения (открытая авторизация или авторизация с помощью маркера). Также укажите сведения, характерные для типа доставки ключей, которые определяют порядок доставки ключа к клиенту (шаблон лицензии [PlayReady](media-services-playready-license-template-overview.md) или [Widevine](media-services-widevine-license-template-overview.md)).

5. Настройте политику доставки для ресурса-контейнера. Конфигурация политики доставки включает протокол доставки (например, MPEG-DASH, HLS, Smooth Streaming или все перечисленные). Конфигурация также включает тип динамического шифрования (например, общее шифрование) и PlayReady или URL-адрес получения лицензии Widevine.

    К разным протоколам можно применять разные политики в отношении одного и того же ресурса. Например, вы можете применить шифрование PlayReady при использовании Smooth или DASH и конвертное шифрование AES при использовании HLS. Потоковая передача по тем протоколам, которые не определены в политике доставки (например, если вы добавили одну политику, которая предусматривает использование только протокола HLS), блокируется. Исключением являются те случаи, когда политика доставки ресурсов вообще не определена. Тогда все протоколы разрешено использовать в незашифрованном виде.

6. Создайте указатель OnDemand, чтобы получить URL-адрес для потоковой передачи.

В конце статьи вы найдете полный пример для .NET.

На следующем изображении показан описанный выше рабочий процесс. В этом случае для аутентификации используется маркер.

![Защита с помощью PlayReady](media/media-services-content-protection-overview/media-services-content-protection-with-drm.png)

В оставшейся части статьи приводятся подробные объяснения, примеры кода и ссылки на статьи с инструкциями для выполнения описанных выше задач.

## <a name="current-limitations"></a>Текущие ограничения
При добавлении или обновлении политики для доставки ресурсов необходимо удалить любой связанный указатель и создать новый.

Сейчас для шифрования с использованием Widevine с помощью служб мультимедиа не поддерживается использование нескольких ключей содержимого. 

## <a name="create-an-asset-and-upload-files-into-the-asset"></a>Создание ресурса-контейнера и отправка в него файлов
Для кодирования и потоковой передачи видео, а также управления ими сначала необходимо отправить содержимое в службы мультимедиа. Отправленное содержимое будет сохранено в безопасном облачном хранилище для последующей обработки и потоковой передачи.

Дополнительные сведения см. в статье [Передача файлов в учетную запись служб мультимедиа с помощью .NET](media-services-dotnet-upload-files.md).

## <a name="encode-the-asset-that-contains-the-file-to-the-adaptive-bitrate-mp4-set"></a>Кодирование ресурса с файлами в набор MP4-файлов с переменной скоростью
При использовании динамического шифрования создается ресурс, содержащий набор MP4-файлов или исходных файлов Smooth Streaming (оба типа относятся к файлам с несколькими скоростями). Затем с учетом формата, указанного в манифесте или запросе фрагмента, сервер потоковой передачи по запросу организует передачу содержимого по выбранному протоколу. Так можно хранить и оплачивать файлы в одном формате хранения. Службы мультимедиа создают и обрабатывают соответствующий ответ с учетом запросов клиента. Чтобы узнать больше, ознакомьтесь с [общими сведениями о динамической упаковке](media-services-dynamic-packaging-overview.md).

Инструкции по кодированию см. в статье [Кодирование ресурса-контейнера с помощью Media Encoder Standard и .NET](media-services-dotnet-encode-with-media-encoder-standard.md).

## <a name="create-a-content-key-and-associate-it-with-the-encoded-asset"></a><a id="create_contentkey"></a>Создание ключа содержимого и связывание его с закодированным ресурсом-контейнером
В службах мультимедиа ключ содержимого содержит ключ, который используется для шифрования ресурса-контейнера.

Дополнительные сведения см. в руководстве по [созданию ключей содержимого](media-services-dotnet-create-contentkey.md).

## <a name="configure-the-content-keys-authorization-policy"></a><a id="configure_key_auth_policy"></a>Настройка политики авторизации для ключа содержимого
Службы мультимедиа поддерживают несколько способов аутентификации пользователей, которые запрашивают ключи. Потребуется настроить политику авторизации для ключа содержимого. Прежде чем ключ будет доставлен в клиент (проигрыватель), его нужно привести в соответствие с политикой. Для политики авторизации ключа содержимого можно задать одно или несколько ограничений: открытая авторизация или авторизация с помощью маркера.

Дополнительную информацию см. в статье о [настройке политики авторизации для ключа содержимого](media-services-dotnet-configure-content-key-auth-policy.md#playready-dynamic-encryption).

## <a name="configure-an-asset-delivery-policy"></a><a id="configure_asset_delivery_policy"></a>Настройка политики доставки для ресурса
Настройте политику доставки для ресурса-контейнера. Вот некоторые элементы, входящие в конфигурацию политики доставки:

* URL-адрес для приобретения лицензии DRM.
* Протокол доставки ресурсов-контейнеров (например, MPEG-DASH, HLS, Smooth Streaming или все перечисленные).
* Тип динамического шифрования (в этом случае — стандартное шифрование).

Дополнительные сведения см. в статье о [настройке политики доставки для ресурсов-контейнеров](media-services-dotnet-configure-asset-delivery-policy.md).

## <a name="create-an-ondemand-streaming-locator-to-get-a-streaming-url"></a><a id="create_locator"></a>Создание указателя потоковой передачи по запросу для получения URL-адреса для потоковой передачи
При использовании протоколов Smooth Streaming, DASH или HLS вам необходимо предоставить пользователю URL-адрес для потоковой передачи.

> [!NOTE]
> При добавлении или обновлении политики доставки ресурсов необходимо удалить все существующие указатели и создать новый.
>
>

Указания по публикации ресурса и созданию URL-адреса потоковой передачи см. в статье [Создание URL-адреса потоковой передачи](media-services-deliver-streaming-content.md).

## <a name="get-a-test-token"></a>Получение маркера тестирования
Получите маркер тестирования в зависимости от ограничения по маркеру, заданного в политике авторизации ключа.

```csharp
// Deserializes a string containing an XML representation of a TokenRestrictionTemplate
// back into a TokenRestrictionTemplate class instance.
TokenRestrictionTemplate tokenTemplate =
TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

// Generate a test token based on the data in the given TokenRestrictionTemplate.
//The GenerateTestToken method returns the token without the word "Bearer" in front,
//so you have to add it in front of the token string.
string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate);
Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
```

Для проверки потока можно использовать [проигрыватель служб мультимедиа Azure](https://aka.ms/azuremediaplayer).

## <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

1. Настройте среду разработки и укажите в файле app.config сведения о подключении, как описано в статье [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md).

2. Добавьте следующие элементы в **appSettings**, определенные в файле app.config:

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

## <a name="example"></a>Пример

В приведенном ниже примере демонстрируются функциональные возможности, представленные в пакете SDK служб мультимедиа для .NET версии 3.5.2 (в частности, возможность определить шаблон лицензии Widevine и подать на нее запрос из служб мультимедиа).

Замените код в файле Program.cs кодом, приведенным в этом разделе.

>[!NOTE]
>Для разных политик служб мультимедиа действует ограничение в 1 млн записей (например, для политики Locator или ContentKeyAuthorizationPolicy). Если вы используете одни и те же даты, разрешения доступа и т. д., указывайте один и тот же идентификатор политики. Например, это политики для указателей, которые должны оставаться на месте в течение длительного времени (не политики передачи). 

Дополнительные сведения см. в статье [Управление активами и связанными сущностями с помощью пакета SDK служб мультимедиа для .NET](media-services-dotnet-manage-entities.md#limit-access-policies).

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
using Microsoft.WindowsAzure.MediaServices.Client.Widevine;
using Newtonsoft.Json;

namespace DynamicEncryptionWithDRM
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

            IContentKey key = CreateCommonTypeContentKey(encodedAsset);
            Console.WriteLine("Created key {0} for the asset {1} ", key.Id, encodedAsset.Id);
            Console.WriteLine("PlayReady License Key delivery URL: {0}", key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense));
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
                // Deserializes a string containing an XML representation of a TokenRestrictionTemplate
                // back into a TokenRestrictionTemplate class instance.
                TokenRestrictionTemplate tokenTemplate =
                    TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

                // Generate a test token based on the data in the given TokenRestrictionTemplate.
                // Note that you need to pass the key ID GUID because 
                // TokenClaim.ContentKeyIdentifierClaim was specified during the creation of TokenRestrictionTemplate.
                Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);
                string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey,
                                            DateTime.UtcNow.AddDays(365));
                Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
                Console.WriteLine();
            }

            // You can use the https://amsplayer.azurewebsites.net/azuremediaplayer.html player to test streams.
            // Note that DASH works on Internet Explorer 11 (via PlayReady), Microsoft Edge (via PlayReady), and Chrome (via Widevine).

            string url = GetStreamingOriginLocator(encodedAsset);
            Console.WriteLine("Encrypted DASH URL: {0}/manifest(format=mpd-time-csf)", url);

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

            IJob job = _context.Jobs.Create(String.Format("Encoding into Mp4 {0} to {1}",
                        inputAsset.Name,
                        encodingPreset));

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


        static public IContentKey CreateCommonTypeContentKey(IAsset asset)
        {

            Guid keyId = Guid.NewGuid();
            byte[] contentKey = GetRandomBuffer(16);

            IContentKey key = _context.ContentKeys.Create(
                        keyId,
                        contentKey,
                        "ContentKey",
                        ContentKeyType.CommonEncryption);

            // Associate the key with the asset.
            asset.ContentKeys.Add(key);

            return key;
        }

        static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
        {

            // Create ContentKeyAuthorizationPolicy with open restrictions
            // and create an authorization policy.         

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
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

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
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

            // Associate the content key authorization policy with the content key.
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
            // The following code configures the PlayReady license template by using .NET classes
            // and returns the XML string.

            //The PlayReadyLicenseResponseTemplate class represents the template for the response sent back to the end user.
            //It contains a field for a custom data string between the license server and the application
            //(which might be useful for custom app logic) as well as a list of one or more license templates.
            PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();

            // The PlayReadyLicenseTemplate class represents a license template you can use to create PlayReady licenses
            // to be returned to end users.
            //It contains the data on the content key in the license and any rights or restrictions to be
            //enforced by the PlayReady DRM runtime when you use the content key.
            PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
            //Configure whether the license is persistent (saved in persistent storage on the client)
            //or nonpersistent (held in memory only while the player uses the license).  
            licenseTemplate.LicenseType = PlayReadyLicenseType.Nonpersistent;

            // AllowTestDevices controls whether test devices can use the license or not.  
            // If true, the MinimumSecurityLevel property of the license
            // is set to 150. If false (the default), the MinimumSecurityLevel property of the license is set to 2,000.
            licenseTemplate.AllowTestDevices = true;

            // You also can configure the PlayRight in the PlayReady license by using the PlayReadyPlayRight class.
            // It grants the user the ability to play back the content subject to the zero or more restrictions
            // configured in the license and on the PlayRight itself (for playback-specific policy).
            // Much of the policy on the PlayRight has to do with output restrictions,
            // which control the types of outputs that the content can be played over and
            // any restrictions that must be put in place when you use a given output.
            // For example, if DigitalVideoOnlyContentRestriction is enabled,
            // the DRM runtime allows the video to be displayed only over digital outputs
            //(analog video outputs aren't allowed to pass the content).

            //IMPORTANT: These types of restrictions can be very powerful but also can affect the consumer experience.
            // If output protections are too restrictive, 
            // content might be unplayable on some clients. For more information, see the PlayReady Compliance Rules document.

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
                    required_output_protection = new RequiredOutputProtection { hdcp = Hdcp.HDCP_NONE},
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

        static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
        {
            // Get the PlayReady license service URL.
            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);

            // GetKeyDeliveryUrl for Widevine attaches the KID to the URL.
            // For example: https://amsaccount1.keydelivery.mediaservices.windows.net/Widevine/?KID=268a6dcb-18c8-4648-8c95-f46429e4927c.  
            // WidevineBaseLicenseAcquisitionUrl (used in the following) also tells dynamic encryption
            // to append /? KID =< keyId > to the end of the URL when you create the manifest.
            // As a result, the Widevine license acquisition URL has the KID appended twice,
            // so you need to remove the KID in the URL when you call GetKeyDeliveryUrl.

            Uri widevineUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.Widevine);
            UriBuilder uriBuilder = new UriBuilder(widevineUrl);
            uriBuilder.Query = String.Empty;
            widevineUrl = uriBuilder.Uri;

            Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
            {
                    {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()},
                    {AssetDeliveryPolicyConfigurationKey.WidevineBaseLicenseAcquisitionUrl, widevineUrl.ToString()}

            };

            // In this case, we specify only the DASH streaming protocol in the delivery policy.
            // All other protocols are blocked from streaming.
            var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryption,
            AssetDeliveryProtocol.Dash,
            assetDeliveryPolicyConfiguration);


            // Add AssetDelivery Policy to the asset.
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

            var assetFile = asset.AssetFiles.ToList().Where(f => f.Name.ToLower().
                         EndsWith(".ism")).
                         FirstOrDefault();

            // Create a 30-day read-only access policy.
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

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>См. также раздел

* [Использование CENC с несколькими подсистемами DRM и контролем доступа](media-services-cenc-with-multidrm-access-control.md)
* [Настройка упаковки Widevine с использованием служб мультимедиа](https://mingfeiy.com/how-to-configure-widevine-packaging-with-azure-media-services)
* [Начало работы с клиентским пакетом SDK для Java для служб мультимедиа Azure](./media-services-java-how-to-use.md)
* Чтобы скачать последний пакет SDK для PHP для служб мультимедиа, найдите версию 0.5.7 пакета Microsoft/WindowsAzure в [репозитории Packagist](https://packagist.org/packages/microsoft/windowsazure#v0.5.7). 
