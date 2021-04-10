---
title: Анализ содержимого видео для выявления нежелательного содержимого в C# — Content Moderator
titleSuffix: Azure Cognitive Services
description: Как анализировать содержимое видео на наличие различных нежелательных материалов с помощью пакета SDK Content Moderator для .NET
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: how-to
ms.date: 05/18/2020
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: dff3bbaf7584b161dec5b9c6eaeb7da1bf6de170
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106055609"
---
# <a name="analyze-video-content-for-objectionable-material-in-c"></a>Анализ содержимого видео для выявления нежелательного содержимого в C#

В этой статье содержатся сведения и примеры кода, которые помогут вам приступить к работе с [пакетом SDK Content Moderator для .NET](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.ContentModerator/) для сканирования содержимого на выявление видео для взрослых или непристойного характера.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу. 

## <a name="prerequisites"></a>Предварительные требования
- Любой выпуск [Visual Studio 2015 или 2017](https://www.visualstudio.com/downloads/).

## <a name="set-up-azure-resources"></a>Настройка ресурсов Azure

Возможности Content Moderator по обработке видео доступны в виде бесплатной общедоступной предварительной версии **обработчика мультимедиа**, предоставляемого в Службах мультимедиа Azure (AMS). Службы мультимедиа Azure — это специализированные службы Azure для хранения и потоковой передачи содержимого видео. 

### <a name="create-an-azure-media-services-account"></a>Создание учетной записи служб мультимедиа Azure

Следуйте инструкциям в статье [Создание учетной записи служб мультимедиа Azure с помощью портала Azure](../../media-services/previous/media-services-portal-create-account.md), чтобы подписаться на AMS и создать связанную учетную запись хранения Azure. В этой учетной записи хранения создайте контейнер хранилища BLOB-объектов.

### <a name="create-an-azure-active-directory-application"></a>Создание приложения Azure Active Directory

Перейдите в новую подписку AMS на портале Azure и из бокового меню выберите **Доступ через API**. Выберите **Connect to Azure Media Services with service principal** (Подключение к Службам мультимедиа Azure с помощью субъекта-службы). Обратите внимание на значение в поле **Конечная точка REST API**. Оно потребуется позже.

В разделе **Приложение Azure AD** выберите **Создать** и присвойте имя новой регистрации приложения Azure AD (например, VideoModADApp). Щелкните **Сохранить** и подождите несколько минут, пока приложение не будет настроено. Затем вы должны увидеть свою новую регистрацию приложения на странице в разделе **Приложение Azure AD**.

Выберите регистрацию приложения и нажмите кнопку **Управление приложением** под ней. Обратите внимание на значение в поле **Идентификатор приложения**. Оно потребуется позже. Выберите **Параметры** > **Ключи** и введите описание для нового ключа (например, VideoModKey). Щелкните **Сохранить**, а затем обратите внимание на новое значение ключа. Скопируйте эту строку и сохраните ее в надежном месте.

Более подробный обзор описанного выше процесса см. в статье [Приступая к работе с аутентификацией Azure AD с помощью портала Azure](../../media-services/previous/media-services-portal-get-started-with-aad.md).

Сделав это, можно использовать обработчик мультимедиа для модерации видео двумя разными способами.

## <a name="use-azure-media-services-explorer"></a>Использование обозревателя Служб мультимедиа Azure

Обозреватель Служб мультимедиа Azure имеет понятный для пользователя интерфейс для AMS. Используйте его для просмотра своей учетной записи AMS, передачи видео и сканирования содержимого с помощью обработчика мультимедиа Content Moderator. Загрузите и установите его из [GitHub](https://github.com/Azure/Azure-Media-Services-Explorer/releases) или ознакомьтесь с [записью блога о Azure Media Services Explorer](https://azure.microsoft.com/blog/managing-media-workflows-with-the-new-azure-media-services-explorer-tool/) для получения дополнительной информации.

![Обозреватель Служб мультимедиа Azure с Content Moderator](images/ams-explorer-content-moderator.PNG)

## <a name="create-the-visual-studio-project"></a>Создание проекта Visual Studio

1. В Visual Studio создайте проект **Консольное приложение (.NET Framework)** и назовите его **VideoModeration**. 
1. При наличии других проектов в решении выберите этот в качестве единого запускаемого проекта.
1. Получите необходимые пакеты NuGet. Щелкните проект правой кнопкой мыши в обозревателе решений и выберите **Управление пакетами NuGet**, затем найдите и установите следующие пакеты:
    - windowsazure.mediaservices;
    - windowsazure.mediaservices.extensions.

## <a name="add-video-moderation-code"></a>Добавление кода модерации видео

Затем скопируйте и вставьте код из этого руководства в проект, чтобы реализовать основной сценарий модерации содержимого.

### <a name="update-the-programs-using-statements"></a>Обновление инструкций using в программе

Добавьте следующие инструкции `using` в начало файла _Program.cs_.

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.IO;
using System.Threading;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Auth;
using System.Collections.Generic;
```

### <a name="set-up-resource-references"></a>Настройка ссылок на ресурсы

Добавьте следующие статические поля в класс **Program** в файле _Program.cs_. Эти поля содержат информацию, необходимую для подключения к вашей подписке AMS. Заполните их значениями, полученными ранее. Обратите внимание, что `CLIENT_ID` является значением **идентификатора** вашего приложения Azure AD, а `CLIENT_SECRET` — значением созданного для этого приложения ключа VideoModKey.

```csharp
// declare constants and globals
private static CloudMediaContext _context = null;
private static CloudStorageAccount _StorageAccount = null;

// Azure Media Services (AMS) associated Storage Account, Key, and the Container that has 
// a list of Blobs to be processed.
static string STORAGE_NAME = "YOUR AMS ASSOCIATED BLOB STORAGE NAME";
static string STORAGE_KEY = "YOUR AMS ASSOCIATED BLOB STORAGE KEY";
static string STORAGE_CONTAINER_NAME = "YOUR BLOB CONTAINER FOR VIDEO FILES";

private static StorageCredentials _StorageCredentials = null;

// Azure Media Services authentication. 
private const string AZURE_AD_TENANT_NAME = "microsoft.onmicrosoft.com";
private const string CLIENT_ID = "YOUR CLIENT ID";
private const string CLIENT_SECRET = "YOUR CLIENT SECRET";

// REST API endpoint, for example "https://accountname.restv2.westcentralus.media.azure.net/API".      
private const string REST_API_ENDPOINT = "YOUR API ENDPOINT";

// Content Moderator Media Processor Nam
private const string MEDIA_PROCESSOR = "Azure Media Content Moderator";

// Input and Output files in the current directory of the executable
private const string INPUT_FILE = "VIDEO FILE NAME";
private const string OUTPUT_FOLDER = "";

// JSON settings file
private static readonly string CONTENT_MODERATOR_PRESET_FILE = "preset.json";

```

Если вы хотите использовать локальный видеофайл (простейший случай), добавьте его в проект и введите его путь как значение `INPUT_FILE` (относительные пути относятся к каталогу выполнения).

Вам также необходимо создать файл _preset.json_ в текущем каталоге и использовать его для указания номера версии. Пример:

```JSON
{
    "version": "2.0"
}
```

### <a name="load-the-input-videos"></a>Загрузка входных видео

Метод **Main** класса **Program** создаст контекст мультимедиа Azure, а затем контекст службы хранилища Azure (если видео находятся в хранилище BLOB-объектов). Остальная часть кода позволяет проверить видео из локальной папки или одного либо нескольких больших двоичных объектов в контейнере хранилища Azure. Закомментировав разные строки кода, вы можете попробовать все варианты поочередно.

```csharp
// Create Azure Media Context
CreateMediaContext();

// Create Storage Context
CreateStorageContext();

// Use a file as the input.
IAsset asset = CreateAssetfromFile();

// -- OR ---

// Or a blob as the input
// IAsset asset = CreateAssetfromBlob((CloudBlockBlob)GetBlobsList().First());

// Then submit the asset to Content Moderator
RunContentModeratorJob(asset);

//-- OR ----

// Just run the content moderator on all blobs in a list (from a Blob Container)
// RunContentModeratorJobOnBlobs();
```

### <a name="create-an-azure-media-context"></a>Создание контекста мультимедиа Azure

Добавьте следующий метод в класс **Program**. Он использует ваши учетные данные AMS, чтобы разрешить обмен данными с AMS.

```csharp
// Creates a media context from azure credentials
static void CreateMediaContext()
{
    // Get Azure AD credentials
    var tokenCredentials = new AzureAdTokenCredentials(AZURE_AD_TENANT_NAME,
        new AzureAdClientSymmetricKey(CLIENT_ID, CLIENT_SECRET),
        AzureEnvironments.AzureCloudEnvironment);

    // Initialize an Azure AD token
    var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

    // Create a media context
    _context = new CloudMediaContext(new Uri(REST_API_ENDPOINT), tokenProvider);
}
```

### <a name="add-the-code-to-create-an-azure-storage-context"></a>Добавление кода для создания контекста хранилища Azure

Добавьте следующий метод в класс **Program**. Контекст хранилища, созданный на основе учетных данных хранилища, позволяет получить доступ к хранилищу BLOB-объектов.

```csharp
// Creates a storage context from the AMS associated storage name and key
static void CreateStorageContext()
{
    // Get a reference to the storage account associated with a Media Services account. 
    if (_StorageCredentials == null)
    {
        _StorageCredentials = new StorageCredentials(STORAGE_NAME, STORAGE_KEY);
    }
    _StorageAccount = new CloudStorageAccount(_StorageCredentials, false);
}
```

### <a name="add-the-code-to-create-azure-media-assets-from-local-file-and-blob"></a>Добавление кода для создания ресурсов мультимедиа Azure из локального файла и (или) хранилища BLOB-объектов

Обработчик мультимедиа Content Moderator запускает задания для **ресурсов** на платформе Служб мультимедиа Azure.
С помощью этих методов создаются ресурсы из локального файла или связанного BLOB-объекта.

```csharp
// Creates an Azure Media Services Asset from the video file
static IAsset CreateAssetfromFile()
{
    return _context.Assets.CreateFromFile(INPUT_FILE, AssetCreationOptions.None); ;
}

// Creates an Azure Media Services asset from your blog storage
static IAsset CreateAssetfromBlob(CloudBlockBlob Blob)
{
    // Create asset from the FIRST blob in the list and return it
    return _context.Assets.CreateFromBlob(Blob, _StorageCredentials, AssetCreationOptions.None);
}
```

### <a name="add-the-code-to-scan-a-collection-of-videos-as-blobs-within-a-container"></a>Добавление в контейнер кода для проверки коллекции видео (сохраненной в виде больших двоичных объектов)

```csharp
// Runs the Content Moderator Job on all Blobs in a given container name
static void RunContentModeratorJobOnBlobs()
{
    // Get the reference to the list of Blobs. See the following method.
    var blobList = GetBlobsList();

    // Iterate over the Blob list items or work on specific ones as needed
    foreach (var sourceBlob in blobList)
    {
        // Create an Asset
        IAsset asset = _context.Assets.CreateFromBlob((CloudBlockBlob)sourceBlob,
                            _StorageCredentials, AssetCreationOptions.None);
        asset.Update();

        // Submit to Content Moderator
        RunContentModeratorJob(asset);
    }
}

// Get all blobs in your container
static IEnumerable<IListBlobItem> GetBlobsList()
{
    // Get a reference to the Container within the Storage Account
    // that has the files (blobs) for moderation
    CloudBlobClient CloudBlobClient = _StorageAccount.CreateCloudBlobClient();
    CloudBlobContainer MediaBlobContainer = CloudBlobClient.GetContainerReference(STORAGE_CONTAINER_NAME);

    // Get the reference to the list of Blobs 
    var blobList = MediaBlobContainer.ListBlobs();
    return blobList;
}
```

### <a name="add-the-method-to-run-the-content-moderator-job"></a>Добавление метода для выполнения задания Content Moderator

```csharp
// Run the Content Moderator job on the designated Asset from local file or blob storage
static void RunContentModeratorJob(IAsset asset)
{
    // Grab the presets
    string configuration = File.ReadAllText(CONTENT_MODERATOR_PRESET_FILE);

    // grab instance of Azure Media Content Moderator MP
    IMediaProcessor mp = _context.MediaProcessors.GetLatestMediaProcessorByName(MEDIA_PROCESSOR);

    // create Job with Content Moderator task
    IJob job = _context.Jobs.Create(String.Format("Content Moderator {0}",
            asset.AssetFiles.First() + "_" + Guid.NewGuid()));

    ITask contentModeratorTask = job.Tasks.AddNew("Adult and racy classifier task",
            mp, configuration,
            TaskOptions.None);
    contentModeratorTask.InputAssets.Add(asset);
    contentModeratorTask.OutputAssets.AddNew("Adult and racy classifier output",
        AssetCreationOptions.None);

    job.Submit();


    // Create progress printing and querying tasks
    Task progressPrintTask = new Task(() =>
    {
        IJob jobQuery = null;
        do
        {
            var progressContext = _context;
            jobQuery = progressContext.Jobs
            .Where(j => j.Id == job.Id)
                .First();
                Console.WriteLine(string.Format("{0}\t{1}",
                DateTime.Now,
                jobQuery.State));
                Thread.Sleep(10000);
            }
            while (jobQuery.State != JobState.Finished &&
            jobQuery.State != JobState.Error &&
            jobQuery.State != JobState.Canceled);
    });
    progressPrintTask.Start();

    Task progressJobTask = job.GetExecutionProgressTask(
    CancellationToken.None);
    progressJobTask.Wait();

    // If job state is Error, the event handling 
    // method for job progress should log errors.  Here we check 
    // for error state and exit if needed.
    if (job.State == JobState.Error)
    {
        ErrorDetail error = job.Tasks.First().ErrorDetails.First();
        Console.WriteLine(string.Format("Error: {0}. {1}",
        error.Code,
        error.Message));
    }

    DownloadAsset(job.OutputMediaAssets.First(), OUTPUT_FOLDER);
}
```

### <a name="add-helper-functions"></a>Добавление вспомогательных функций

Эти методы позволяют скачать выходной файл Content Moderator в формате JSON из ресурса Служб мультимедиа Azure и отслеживать состояние задания модерации, чтобы программа могла регистрировать в консоли эти сведения о состоянии.

```csharp
static void DownloadAsset(IAsset asset, string outputDirectory)
{
    foreach (IAssetFile file in asset.AssetFiles)
    {
        file.Download(Path.Combine(outputDirectory, file.Name));
    }
}

// event handler for Job State
static void StateChanged(object sender, JobStateChangedEventArgs e)
{
    Console.WriteLine("Job state changed event:");
    Console.WriteLine("  Previous state: " + e.PreviousState);
    Console.WriteLine("  Current state: " + e.CurrentState);
    switch (e.CurrentState)
    {
        case JobState.Finished:
            Console.WriteLine();
            Console.WriteLine("Job finished.");
            break;
        case JobState.Canceling:
        case JobState.Queued:
        case JobState.Scheduled:
        case JobState.Processing:
            Console.WriteLine("Please wait...\n");
            break;
        case JobState.Canceled:
            Console.WriteLine("Job is canceled.\n");
            break;
        case JobState.Error:
            Console.WriteLine("Job failed.\n");
            break;
        default:
            break;
    }
}
```

### <a name="run-the-program-and-review-the-output"></a>Запуск программы и просмотр выходных данных

Когда задание модерации содержимого будет выполнено, изучите полученный ответ в формате JSON. Он состоит из следующих элементов:

- сводка по видео;
- **видеоролики**, обозначенные как "**fragments**";
- **ключевые кадры**, обозначенные как "**events**" с флагом необходимости проверки **"reviewRecommended" (принимает значение true или false)**, который присваивается на основе оценок **Asult** и **Racy**;
- значения **start** (начало), **duration** (длительность), **totalDuration** (общая длительность) и **timestamp** (метка времени), выраженные в тактах. Чтобы получить значение в секундах, поделите величину в тактах на значение **timescale** (шкала времени).
 
> [!NOTE]
> - `adultScore` обозначает потенциальное наличие и прогнозируемую оценку содержимого, которое в определенных обстоятельствах может считаться порнографическим или предназначенным только для взрослых.
> - `racyScore` обозначает потенциальное наличие и прогнозируемую оценку содержимого, которое в определенных обстоятельствах может считаться сексуально окрашенными или не предназначенным для детей.
> - Оценки `adultScore` и `racyScore` принимают значения от 0 до 1. Чем выше оценка, тем более подходящей модель считает соответствующую категорию. Эта предварительная версия использует статистическую модель прогнозирования, а не оценки, кодированные вручную. Корпорация Майкрософт рекомендует протестировать ее на своих данных, чтобы проверить применимость анализа по каждой категории.
> - `reviewRecommended` принимает значения true или false в зависимости от внутренних порогов оценки. Клиенты могут на выбор использовать значения по умолчанию или настраивать собственные пороги в соответствии с действующими политиками.

```json
{
"version": 2,
"timescale": 90000,
"offset": 0,
"framerate": 50,
"width": 1280,
"height": 720,
"totalDuration": 18696321,
"fragments": [
{
    "start": 0,
    "duration": 18000
},
{
    "start": 18000,
    "duration": 3600,
    "interval": 3600,
    "events": [
    [
        {
        "reviewRecommended": false,
        "adultScore": 0.00001,
        "racyScore": 0.03077,
        "index": 5,
        "timestamp": 18000,
        "shotIndex": 0
        }
    ]
    ]
},
{
    "start": 18386372,
    "duration": 119149,
    "interval": 119149,
    "events": [
    [
        {
        "reviewRecommended": true,
        "adultScore": 0.00000,
        "racyScore": 0.91902,
        "index": 5085,
        "timestamp": 18386372,
        "shotIndex": 62
        }
    ]
    ]
}
]
}
```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как создавать [обзоры видео](video-reviews-quickstart-dotnet.md) на основе выходных данных модерации.

[Скачайте решение Visual Studio](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/ContentModerator) для работы с этим и другими краткими руководствами по Content Moderator для .NET.