---
title: Приступая к работе с доставкой содержимого по запросу с помощью пакета SDK для .NET | Документация Майкрософт
description: В этом учебнике показаны шаги по реализации приложения доставки содержимого по запросу с помощью служб мультимедиа Azure и .NET.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 388b8928-9aa9-46b1-b60a-a918da75bd7b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: f0e7f9fa33af43fb846255815d862d4cd3a07e99
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103013981"
---
# <a name="get-started-with-delivering-content-on-demand-using-net-sdk"></a>Приступая к работе с доставкой содержимого по запросу с помощью пакета SDK для .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

[!INCLUDE [media-services-selector-get-started](../../../includes/media-services-selector-get-started.md)]

В этом руководстве описано, как реализовать простую службу доставки видео по запросу (VOD) с помощью приложения служб мультимедиа Azure (AMS) и пакета .NET SDK для служб мультимедиа Azure.

## <a name="prerequisites"></a>Предварительные требования

Ниже перечислены необходимые условия для выполнения действий, описанных в этом учебнике.

* Учетная запись Azure. Дополнительные сведения см. в разделе [Бесплатная пробная версия Azure](https://azure.microsoft.com/pricing/free-trial/).
* Учетная запись служб мультимедиа. Инструкции по созданию учетной записи служб мультимедиа см. в статье [Создание учетной записи служб мультимедиа Azure с помощью портала Azure](media-services-portal-create-account.md).
* .NET Framework 4.0 или более поздней версии.
* приведенному.

Учебник включает в себя следующие разделы:

1. Запуск конечных точек потоковой передачи с помощью портала Azure
2. Создание и настройка проекта Visual Studio.
3. Подключение к учетной записи служб мультимедиа.
2. Загрузка видеофайла.
3. Кодирование исходного файла в набор MP4-файлов с адаптивным битрейтом.
4. Публикация ресурса и получение URL-адресов потоковой передачи и поэтапного скачивания.  
5. Воспроизведение содержимого.

## <a name="overview"></a>Обзор
В этом учебнике содержатся шаги по реализации приложения доставки видео по запросу с помощью пакета SDK служб мультимедиа Azure для .NET.

В учебнике описывается базовый рабочий процесс служба мультимедиа и наиболее распространенные объекты и задачи программирования, необходимые для разработки с использованием служб мультимедиа. По завершении работы с учебником вы сможете выполнить потоковую передачу и поэтапно скачать пример файла мультимедиа, который вы отправили, закодировали и скачали.

### <a name="ams-model"></a>Модель AMS

На следующем рисунке показаны некоторые объекты, которые чаще всего используются при разработке приложений VoD с использованием модели OData служб мультимедиа.

Щелкните изображение, чтобы просмотреть его полноразмерную версию.  

[![Схема, на которой показаны некоторые наиболее часто используемые объекты в модели данных объектов служб мультимедиа Azure для разработки приложений видео по запросу.](./media/media-services-dotnet-get-started/media-services-overview-object-model-small.png)](./media/media-services-dotnet-get-started/media-services-overview-object-model.png#lightbox)

Всю модель можно просмотреть [здесь](https://media.windows.net/API/$metadata?api-version=2.15).  

## <a name="start-streaming-endpoints-using-the-azure-portal"></a>Запуск конечных точек потоковой передачи с помощью портала Azure

При работе со службами мультимедиа Azure один из самых частых сценариев — потоковая передача видео с переменной скоростью. Службы мультимедиа обеспечивают динамическую упаковку, которая позволяет своевременно доставлять закодированное содержимое MP4-файлов с переменной скоростью в форматах потоковой передачи, которые поддерживаются службами мультимедиа (MPEG DASH, HLS, Smooth Streaming), без необходимости хранения предварительно упакованных версий каждого из этих форматов потоковой передачи.

>[!NOTE]
>При создании учетной записи AMS в нее добавляется конечная точка потоковой передачи **по умолчанию** в состоянии **Остановлена**. Чтобы начать потоковую передачу содержимого и воспользоваться динамической упаковкой и динамическим шифрованием, конечная точка потоковой передачи, из которой необходимо выполнять потоковую передачу содержимого, должна находиться в состоянии **Выполняется**.

Чтобы запустить конечную точку потоковой передачи, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).
2. В окне "Параметры" щелкните элемент "Потоковые конечные точки".
3. Щелкните конечную точку потоковой передачи по умолчанию.

    Появится окно сведений о конечной точке потоковой передачи по умолчанию.

4. Щелкните значок "Пуск".
5. Нажмите кнопку "Сохранить", чтобы сохранить изменения.

## <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

1. Настройте среду разработки и заполните файл app.config данными о соединении, как описано в разделе [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md). 
2. Создайте папку (в любом расположении на локальном диске) и скопируйте MP4-файл, для которого нужно выполнить кодирование, потоковую передачу или поэтапное скачивание. В этом примере используется путь C:\VideoFiles.

## <a name="connect-to-the-media-services-account"></a>Подключение к учетной записи служб мультимедиа

Если вы используете службы мультимедиа с помощью .NET, то для большинства задач программирования этих служб следует использовать класс **CloudMediaContext**. Это могут быть такие задачи, как подключение к учетной записи служб мультимедиа, создание, обновление и удаление различных объектов и получение доступа к этим объектам: ресурсам-контейнерам, файлам ресурсов-контейнеров, заданиям, политикам доступа, указателям и т. д.

Перезапишите класс Program по умолчанию приведенным ниже кодом. В коде показано, как считать значения подключения из файла App.config и как создать объект **CloudMediaContext** для подключения к службам мультимедиа. Дополнительные сведения см. в статье [Доступ к API служб мультимедиа Azure с помощью аутентификации Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

Не забудьте обновить имя файла мультимедиа и путь к нему.

Функция **Main** вызывает методы, которые будут определены далее в этом разделе.

> [!NOTE]
> Пока вы не добавите определения для всех функций, будут поступать сообщения об ошибках компиляции. Инструкции по определению дисков см. далее в этой статье.

```csharp
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

        private static CloudMediaContext _context = null;

        static void Main(string[] args)
        {
        try
        {
            AzureAdTokenCredentials tokenCredentials = 
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Add calls to methods defined in this section.
            // Make sure to update the file name and path to where you have your media file.
            IAsset inputAsset =
            UploadFile(@"C:\VideoFiles\BigBuckBunny.mp4", AssetCreationOptions.None);

            IAsset encodedAsset =
            EncodeToAdaptiveBitrateMP4s(inputAsset, AssetCreationOptions.None);

            PublishAssetGetURLs(encodedAsset);
        }
        catch (Exception exception)
        {
            // Parse the XML error message in the Media Services response and create a new
            // exception with its content.
            exception = MediaServicesExceptionParser.Parse(exception);

            Console.Error.WriteLine(exception.Message);
        }
        finally
        {
            Console.ReadLine();
        }
        }
```

## <a name="create-a-new-asset-and-upload-a-video-file"></a>Создание нового актива и отправка видеофайла

В службах мультимедиа цифровые файлы отправляются (или принимаются) в актив. Сущность **Asset** может содержать видео, аудио, изображения, коллекции эскизов, текстовые дорожки и файлы скрытых субтитров (а также метаданные этих файлов).  После передачи файлов содержимое будет безопасно сохранено в облаке для дальнейшей обработки и потоковой передачи. Файлы в ресурсе называются **файлами ресурса**.

Метод **UploadFile**, определенный ниже, вызывает **CreateFromFile** (этот метод определен в расширениях пакета SDK для .NET). **CreateFromFile** создает новый ресурс-контейнер, в который отправляется указанный исходный файл.

Метод **CreateFromFile** принимает **AssetCreationOptions**, который позволяет указать один из следующих параметров создания ресурсов:

* **None** — шифрование не используется. Это значение по умолчанию. Обратите внимание, что при использовании этого параметра содержимое не защищено при передаче и хранении.
  Используйте этот параметр, если MP4-файл планируется доставить с помощью поэтапного скачивания.
* **StorageEncrypted**. Используйте этот параметр, чтобы локально зашифровать содержимое с помощью 256-битового шифрования AES, а затем отправить его в хранилище Azure для хранения в зашифрованном виде. Активы, защищенные с помощью шифрования хранилища, автоматически расшифровываются и помещаются в систему зашифрованных файлов до кодирования и при необходимости повторно кодируются до отправки в виде нового выходного актива. Шифрование хранилища чаще всего используется, если нужно защитить входные файлы мультимедиа высокого качества с помощью стойкого шифрования при хранении на диске.
* **CommonEncryptionProtected**. Используйте этот параметр при отправке содержимого, которое уже зашифровано и защищено с помощью стандартного шифрования или PlayReady DRM (например, Smooth Streaming с защитой PlayReady DRM).
* **EnvelopeEncryptionProtected**. Используйте этот параметр при отправке HLS с шифрованием AES. Обратите внимание, что файлы должны быть закодированы и зашифрованы с помощью Transform Manager.

Метод **CreateFromFile**, кроме того, позволяет указывать метод обратного вызова, чтобы сообщать о ходе отправки файла.

В следующем примере в качестве параметров ресурса-контейнера используется **None** .

Добавьте следующий метод в класс Program.

```csharp
    static public IAsset UploadFile(string fileName, AssetCreationOptions options)
    {
        IAsset inputAsset = _context.Assets.CreateFromFile(
            fileName,
            options,
            (af, p) =>
            {
                Console.WriteLine("Uploading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Asset {0} created.", inputAsset.Id);

        return inputAsset;
    }
```

## <a name="encode-the-source-file-into-a-set-of-adaptive-bitrate-mp4-files"></a>Кодирование исходного файла в набор MP4-файлов с адаптивным битрейтом
После приема ресурсов-контейнеров в службы мультимедиа файлы мультимедиа можно кодировать, менять их формат, добавлять к ним водяные знаки и т. д. до их доставки клиентам. Эти действия запланированы и выполняются в нескольких экземплярах фоновых ролей для обеспечения высокой производительности и доступности. Эти действия называются заданиями, и каждое задание состоит из атомарных задач, которые выполняют фактическую работу с файлом ресурса-контейнера.

Как упоминалось ранее, один из наиболее распространенных сценариев при работе со службами мультимедиа Azure — доставка клиентам потоковой передачи с переменной скоростью. Службы мультимедиа могут динамически упаковывать набор MP4-файлов с переменной скоростью в один из следующих форматов: HTTP Live Streaming (HLS), Smooth Streaming и MPEG DASH.

Чтобы воспользоваться динамической упаковкой, закодируйте или перекодируйте мезонинный (исходный) файл в набор файлов формата MP4 или потоковой передачи Smooth Streaming с переменной скоростью.  

В следующем коде показано, как отправить задание кодирования. Задание содержит одну задачу, которая определяет перекодировку мезонинного файла в набор MP4-файлов с переменной скоростью с помощью **Media Encoder Standard**. Код отправляет задание и ждет его выполнения.

После выполнения заданий вы сможете выполнить потоковую передачу актива и поэтапное скачивание MP4-файлов, созданных в результате перекодирования.

Добавьте следующий метод в класс Program.

```csharp
    static public IAsset EncodeToAdaptiveBitrateMP4s(IAsset asset, AssetCreationOptions options)
    {

        // Prepare a job with a single task to transcode the specified asset
        // into a multi-bitrate asset.

        IJob job = _context.Jobs.CreateWithSingleTask(
            "Media Encoder Standard",
            "Adaptive Streaming",
            asset,
            "Adaptive Bitrate MP4",
            options);

        Console.WriteLine("Submitting transcoding job...");


        // Submit the job and wait until it is completed.
        job.Submit();

        job = job.StartExecutionProgressTask(
            j =>
            {
                Console.WriteLine("Job state: {0}", j.State);
                Console.WriteLine("Job progress: {0:0.##}%", j.GetOverallProgress());
            },
            CancellationToken.None).Result;

        Console.WriteLine("Transcoding job finished.");

        IAsset outputAsset = job.OutputMediaAssets[0];

        return outputAsset;
    }
```

## <a name="publish-the-asset-and-get-urls-for-streaming-and-progressive-download"></a>Публикация ресурса-контейнера и получение URL-адресов для потоковой передачи и поэтапного скачивания

Чтобы выполнить потоковую передачу ресурса-контейнера или скачать его, сначала необходимо "опубликовать" его, создав указатель. Указатели предоставляют доступ к файлам, содержащимся в активе. Службы мультимедиа поддерживают два типа указателей: Указатели OnDemandOrigin, использующиеся для потоковой передачи мультимедиа (например, MPEG DASH, HLS или Smooth Streaming), и указатели подписанного URL-адреса, использующиеся для скачивания файлов мультимедиа.

### <a name="some-details-about-url-formats"></a>Некоторые сведения о форматах URL-адреса

Создав указатели, вы можете создать URL-адреса, которые будут использоваться для потоковой передачи или скачивания файлов. При помощи приведенного в нашем руководстве примера выводятся URL-адреса, которые затем можно использовать в подходящих браузерах. В этом разделе представлены краткие примеры разных форматов.

#### <a name="a-streaming-url-for-mpeg-dash-has-the-following-format"></a>URL-адрес потоковой передачи для MPEG DASH имеет следующий формат:

{имя конечной точки потоковой передачи — имя учетной записи служб мультимедиа}. Streaming. mediaservices. Windows. NET/{ИД локатора}/{Филенаме}.ИСМ/манифест **(формат = MPD-Time-CSF)**

#### <a name="a-streaming-url-for-hls-has-the-following-format"></a>URL-адрес потоковой передачи для HLS имеет следующий формат:

{имя_конечной_точки_потоковой_передачи-имя_учетной_записи_служб_мультимедиа}.streaming.mediaservices.windows.net/{идентификатор_указателя}/{имя_файла}.ism/Manifest **(format=m3u8-aapl)**

#### <a name="a-streaming-url-for-smooth-streaming-has-the-following-format"></a>URL-адрес потоковой передачи для Smooth Streaming имеет следующий формат:

{имя конечной точки потоковой передачи - имя учетной записи служб мультимедиа}.streaming.mediaservices.windows.net/{идентификатор указателя}/{имя файла}.ism/Manifest


#### <a name="a-sas-url-used-to-download-files-has-the-following-format"></a>URL-адрес SAS, используемый для скачивания файлов, имеет следующий формат:

{имя_контейнера_больших_двоичных_объектов}/{имя_ресурса-контейнера}/{имя_файла}/{подпись_SAS}

Расширения пакета SDK .NET служб мультимедиа обеспечивают удобные вспомогательные методы, которые возвращают отформатированные URL-адреса для опубликованного актива.

Следующий код использует расширения SDK .NET при создании указателей и получении URL-адресов для потоковой передачи и поэтапного скачивания. Код также показывает, как скачать файлы в локальную папку.

Добавьте следующий метод в класс Program.

```csharp
    static public void PublishAssetGetURLs(IAsset asset)
    {
        // Publish the output asset by creating an Origin locator for adaptive streaming,
        // and a SAS locator for progressive download.

        _context.Locators.Create(
            LocatorType.OnDemandOrigin,
            asset,
            AccessPermissions.Read,
            TimeSpan.FromDays(30));

        _context.Locators.Create(
            LocatorType.Sas,
            asset,
            AccessPermissions.Read,
            TimeSpan.FromDays(30));


        IEnumerable<IAssetFile> mp4AssetFiles = asset
                .AssetFiles
                .ToList()
                .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));

        // Get the Smooth Streaming, HLS and MPEG-DASH URLs for adaptive streaming,
        // and the Progressive Download URL.
        Uri smoothStreamingUri = asset.GetSmoothStreamingUri();
        Uri hlsUri = asset.GetHlsUri();
        Uri mpegDashUri = asset.GetMpegDashUri();

        // Get the URls for progressive download for each MP4 file that was generated as a result
        // of encoding.
        List<Uri> mp4ProgressiveDownloadUris = mp4AssetFiles.Select(af => af.GetSasUri()).ToList();


        // Display  the streaming URLs.
        Console.WriteLine("Use the following URLs for adaptive streaming: ");
        Console.WriteLine(smoothStreamingUri);
        Console.WriteLine(hlsUri);
        Console.WriteLine(mpegDashUri);
        Console.WriteLine();

        // Display the URLs for progressive download.
        Console.WriteLine("Use the following URLs for progressive download.");
        mp4ProgressiveDownloadUris.ForEach(uri => Console.WriteLine(uri + "\n"));
        Console.WriteLine();

        // Download the output asset to a local folder.
        string outputFolder = "job-output";
        if (!Directory.Exists(outputFolder))
        {
            Directory.CreateDirectory(outputFolder);
        }

        Console.WriteLine();
        Console.WriteLine("Downloading output asset files to a local folder...");
        asset.DownloadToFolder(
            outputFolder,
            (af, p) =>
            {
                Console.WriteLine("Downloading '{0}' - Progress: {1:0.##}%", af.Name, p.Progress);
            });

        Console.WriteLine("Output asset files available at '{0}'.", Path.GetFullPath(outputFolder));
    }
```

## <a name="test-by-playing-your-content"></a>Проверка посредством воспроизведения содержимого

После запуска программы, определенной в предыдущем разделе, в окне консоли отобразятся URL-адреса, аналогичные следующим.

URL-адреса адаптивной потоковой передачи:

Smooth Streaming

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest`

HLS

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=m3u8-aapl)`

MPEG DASH

`http://amstestaccount001.streaming.mediaservices.windows.net/ebf733c4-3e2e-4a68-b67b-cc5159d1d7f2/BigBuckBunny.ism/manifest(format=mpd-time-csf)`

URL-адреса поэтапного скачивания (аудио и видео).

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_96kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`

`https://storagetestaccount001.blob.core.windows.net/asset-38058602-a4b8-4b33-b9f0-6880dc1490ea/BigBuckBunny_AAC_und_ch2_56kbps.mp4?sv=2012-02-12&sr=c&si=166d5154-b801-410b-a226-ee2f8eac1929&sig=P2iNZJAvAWpp%2Bj9yV6TQjoz5DIIaj7ve8ARynmEM6Xk%3D&se=2015-02-14T01:13:05Z`


Чтобы выполнить потоковую передачу видео, вставьте URL-адрес в соответствующее текстовое поле в [проигрывателе служб мультимедиа Azure](https://aka.ms/azuremediaplayer).

Чтобы протестировать поэтапное скачивание, вставьте URL-адрес в браузер (например, Internet Explorer, Chrome или Safari).

Дополнительные сведения см. в следующих разделах:

- [Воспроизведение содержимого с помощью существующих проигрывателей](media-services-playback-content-with-existing-players.md)
- [Встраивание адаптивного потокового видео MPEG-DASH в приложение HTML5 с помощью DASH.js](media-services-embed-mpeg-dash-in-html5.md)

## <a name="download-sample"></a>Скачивание образца
Следующий пример содержит код, созданный в ходе работы с этим руководством: [образец](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/).

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]



<!-- Anchors. -->


<!-- URLs. -->
[Web Platform Installer]: https://go.microsoft.com/fwlink/?linkid=255386
[Portal]: https://portal.azure.com/
