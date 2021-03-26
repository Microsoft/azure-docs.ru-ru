---
title: Разработка функций Azure с помощью служб мультимедиа v3
description: В этом разделе показано, как приступить к разработке функций Azure с помощью служб мультимедиа v3, используя портал Azure.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.workload: media
ms.devlang: dotnet
ms.topic: article
ms.date: 03/22/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: e0bb8ccf3be6038c228034a55cd15cadcebddbb7
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105572491"
---
# <a name="develop-azure-functions-with-media-services-v3"></a>Разработка функций Azure с помощью служб мультимедиа v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье описывается создание функций Azure, использующих службы мультимедиа. Функция Azure, определенная в этой статье, отслеживает контейнер учетной записи хранения **input** на наличие новых MP4-файлов. Когда в контейнер хранилища попадет файл, триггер большого двоичного объекта выполнит функцию. Чтобы просмотреть функции Azure, ознакомьтесь с  [обзором](../../azure-functions/functions-overview.md) и другими разделами в разделе **функции Azure** .

Если вы хотите изучить и развернуть существующие службы "Функции Azure", использующие службы мультимедиа Azure, ознакомьтесь с [функциями Azure для служб мультимедиа](https://github.com/Azure-Samples/media-services-v3-dotnet-core-functions-integration). Этот репозиторий содержит примеры, которые используют службы мультимедиа, чтобы показать рабочие процессы, связанные с приемом содержимого напрямую от хранилища BLOB-объектов, шифрованием и записью содержимого обратно в хранилище BLOB-объектов.

## <a name="prerequisites"></a>Предварительные условия

- Чтобы создавать функции, вам нужна активная учетная запись Azure. Если у вас ее нет, воспользуйтесь [бесплатной учетной записью Azure](https://azure.microsoft.com/free/).
- Если вы хотите создать Функции Azure, которые выполняют определенные действия в учетной записи служб мультимедиа Azure (AMS) или прослушивают события, отправляемые службами мультимедиа, вам необходимо создать учетную запись AMS, следуя приведенным [здесь](create-account-howto.md) инструкциям.

## <a name="create-a-function-app"></a>Создание приложения-функции

1. Перейдите на [портал Azure](https://portal.azure.com) и войдите, используя свою учетную запись Azure.
2. Создайте приложение-функцию, как описано [здесь](../../azure-functions/functions-create-function-app-portal.md).

>[!NOTE]
> Указанная учетная запись хранения должна находиться в том же регионе, что и ваше приложение.

## <a name="configure-function-app-settings"></a>Настройка параметров приложения-функции

При разработке функций служб мультимедиа удобно добавить переменные среды, которые будут использоваться в функциях. Чтобы настроить параметры приложения, щелкните ссылку "Настроить параметры приложения". Дополнительную информацию см. в разделе [Настройка параметров приложения-функции Azure](../../azure-functions/functions-how-to-use-azure-function-app-settings.md).

## <a name="create-a-function"></a>Создание функции

После развертывания приложения-функции его можно найти в Функциях Azure **службы приложений**.

1. Выберите свое приложение-функцию и щелкните **Новая функция**.
1. Выберите язык **C#** и сценарий **Обработка данных**.
1. Выберите шаблон **BlobTrigger**. Эта функция активируется каждый раз, когда в контейнер **input** передается большой двоичный объект. На следующем шаге в **Path** указывается имя **input**.
1. После выбора **BlobTrigger** на странице появится несколько дополнительных элементов управления.
1. Нажмите кнопку **Создать**.

## <a name="files"></a>Файлы

Функция Azure связана с файлами кода и другими файлами, описание которых представлено в данной статье. При создании функции с помощью портала Azure файлы **function.json** и **run.csx** создаются автоматически. Вам потребуется добавить или отправить файл **project.json**. В оставшейся части этого раздела содержится краткое описание и определение каждого из этих файлов.

### <a name="functionjson"></a>function.json

Файл function.json определяет привязки функций и другие параметры конфигурации. В среде выполнения этот файл используется для определения событий, которые необходимо отслеживать, и способа передачи данных в выполнение функции и возвращения данных из него. Дополнительные сведения см. в статье [привязки HTTP и веб-перехватчика в функциях Azure](../../azure-functions/functions-reference.md#function-code).

>[!NOTE]
>Чтобы функция не выполнялась, задайте для свойства **disabled** значение **true**.

Замените содержимое существующего файла function.json следующим кодом:

```json
{
  "bindings": [
    {
      "name": "myBlob",
      "type": "blobTrigger",
      "direction": "in",
      "path": "input/{filename}.mp4",
      "connection": "ConnectionString"
    }
  ],
  "disabled": false
}
```

### <a name="projectjson"></a>project.json

Файл project.json содержит зависимости. Ниже приведен пример **project.jsв** файле, который содержит необходимые пакеты служб мультимедиа Azure .NET из NuGet. Обратите внимание, что номера версий изменятся после выпуска последних обновлений для пакетов, поэтому следует убедиться в наличии самых последних версий.

Добавьте следующее определение в файл project.json.

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "windowsazure.mediaservices": "4.0.0.4",
        "windowsazure.mediaservices.extensions": "4.0.0.4",
        "Microsoft.IdentityModel.Clients.ActiveDirectory": "3.13.1",
        "Microsoft.IdentityModel.Protocol.Extensions": "1.0.2.206221351"
      }
    }
   }
}

```

### <a name="runcsx"></a>run.csx

Это код C# для функции.  Функция, определенная ниже, отслеживает контейнер учетной записи хранения **input** (который был указан в пути) на наличие новых MP4-файлов. Когда в контейнер хранилища попадет файл, триггер большого двоичного объекта выполнит функцию.

В примере, определенном в этом разделе, показано следующее.

1. Как принять ресурс в учетную запись служб мультимедиа (скопировав большой двоичный объект в ресурс AMS)
2. Как отправить задание кодирования, использующее предустановку Media Encoder Standard "Адаптивная потоковая передача"

Замените содержимое имеющегося файла run.csx следующим кодом. Определив функцию, щелкните **Сохранить и запустить**.

```csharp
#r "Microsoft.WindowsAzure.Storage"
#r "Newtonsoft.Json"
#r "System.Web"

using System;
using System.Net;
using System.Net.Http;
using Newtonsoft.Json;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
using System.IO;
using System.Web;
using Microsoft.Azure;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.Azure.WebJobs;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
  
// Read values from the App.config file.

static readonly string _AADTenantDomain = Environment.GetEnvironmentVariable("AMSAADTenantDomain");
static readonly string _RESTAPIEndpoint = Environment.GetEnvironmentVariable("AMSRESTAPIEndpoint");
 
static readonly string _mediaservicesClientId = Environment.GetEnvironmentVariable("AMSClientId");
static readonly string _mediaservicesClientSecret = Environment.GetEnvironmentVariable("AMSClientSecret");

static readonly string _connectionString = Environment.GetEnvironmentVariable("ConnectionString");  

private static CloudMediaContext _context = null;
private static CloudStorageAccount _destinationStorageAccount = null;

public static void Run(CloudBlockBlob myBlob, string fileName, TraceWriter log)
{
    // NOTE that the variables {fileName} here come from the path setting in function.json
    // and are passed into the  Run method signature above. We can use this to make decisions on what type of file
    // was dropped into the input container for the function. 

    // No need to do any Retry strategy in this function, By default, the SDK calls a function up to 5 times for a 
    // given blob. If the fifth try fails, the SDK adds a message to a queue named webjobs-blobtrigger-poison.

    log.Info($"C# Blob trigger function processed: {fileName}.mp4");
    log.Info($"Media Services REST endpoint : {_RESTAPIEndpoint}");

    try
    {
        AzureAdTokenCredentials tokenCredentials = new AzureAdTokenCredentials(_AADTenantDomain,
                            new AzureAdClientSymmetricKey(_mediaservicesClientId, _mediaservicesClientSecret),
                            AzureEnvironments.AzureCloudEnvironment);
 
        AzureAdTokenProvider tokenProvider = new AzureAdTokenProvider(tokenCredentials);
 
        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

        IAsset newAsset = CreateAssetFromBlob(myBlob, fileName, log).GetAwaiter().GetResult();

        // Step 2: Create an Encoding Job

        // Declare a new encoding job with the Standard encoder
        IJob job = _context.Jobs.Create("Azure Function - MES Job");

        // Get a media processor reference, and pass to it the name of the 
        // processor to use for the specific task.
        IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

        // Create a task with the encoding details, using a custom preset
        ITask task = job.Tasks.AddNew("Encode with Adaptive Streaming",
            processor,
            "Adaptive Streaming",
            TaskOptions.None); 

        // Specify the input asset to be encoded.
        task.InputAssets.Add(newAsset);

        // Add an output asset to contain the results of the job. 
        // This output is specified as AssetCreationOptions.None, which 
        // means the output asset is not encrypted. 
        task.OutputAssets.AddNew(fileName, AssetCreationOptions.None);

        job.Submit();
        log.Info("Job Submitted");

    }
    catch (Exception ex)
    {
        log.Error("ERROR: failed.");
        log.Info($"StackTrace : {ex.StackTrace}");
        throw ex;
    }
}

private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
{
    var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
    ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

    if (processor == null)
    throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

    return processor;
}

public static async Task<IAsset> CreateAssetFromBlob(CloudBlockBlob blob, string assetName, TraceWriter log){
    IAsset newAsset = null;

    try{
        Task<IAsset> copyAssetTask = CreateAssetFromBlobAsync(blob, assetName, log);
        newAsset = await copyAssetTask;
        log.Info($"Asset Copied : {newAsset.Id}");
    }
    catch(Exception ex){
        log.Info("Copy Failed");
        log.Info($"ERROR : {ex.Message}");
        throw ex;
    }

    return newAsset;
}

/// <summary>
/// Creates a new asset and copies blobs from the specifed storage account.
/// </summary>
/// <param name="blob">The specified blob.</param>
/// <returns>The new asset.</returns>
public static async Task<IAsset> CreateAssetFromBlobAsync(CloudBlockBlob blob, string assetName, TraceWriter log)
{
     //Get a reference to the storage account that is associated with the Media Services account. 
    _destinationStorageAccount = CloudStorageAccount.Parse(_connectionString);

    // Create a new asset. 
    var asset = _context.Assets.Create(blob.Name, AssetCreationOptions.None);
    log.Info($"Created new asset {asset.Name}");

    IAccessPolicy writePolicy = _context.AccessPolicies.Create("writePolicy",
    TimeSpan.FromHours(4), AccessPermissions.Write);
    ILocator destinationLocator = _context.Locators.CreateLocator(LocatorType.Sas, asset, writePolicy);
    CloudBlobClient destBlobStorage = _destinationStorageAccount.CreateCloudBlobClient();

    // Get the destination asset container reference
    string destinationContainerName = (new Uri(destinationLocator.Path)).Segments[1];
    CloudBlobContainer assetContainer = destBlobStorage.GetContainerReference(destinationContainerName);

    try{
    assetContainer.CreateIfNotExists();
    }
    catch (Exception ex)
    {
    log.Error ("ERROR:" + ex.Message);
    }

    log.Info("Created asset.");

    // Get hold of the destination blob
    CloudBlockBlob destinationBlob = assetContainer.GetBlockBlobReference(blob.Name);

    // Copy Blob
    try
    {
    using (var stream = await blob.OpenReadAsync()) 
    {            
        await destinationBlob.UploadFromStreamAsync(stream);          
    }

    log.Info("Copy Complete.");

    var assetFile = asset.AssetFiles.Create(blob.Name);
    assetFile.ContentFileSize = blob.Properties.Length;
    assetFile.IsPrimary = true;
    assetFile.Update();
    asset.Update();
    }
    catch (Exception ex)
    {
    log.Error(ex.Message);
    log.Info (ex.StackTrace);
    log.Info ("Copy Failed.");
    throw;
    }

    destinationLocator.Delete();
    writePolicy.Delete();

    return asset;
}
```

## <a name="test-your-function"></a>Тестирование функции

Чтобы протестировать функцию, необходимо передать MP4-файл в контейнер учетной записи хранения **input**, указанный в строке подключения.  

1. Выберите указанную учетную запись хранения.
2. Щелкните **BLOB-объекты**.
3. Щелкните **+ Container** (+ Контейнер). Введите имя для **входных данных** контейнера.
4. Щелкните **Отправить** и найдите MP4-файл, который требуется отправить.

>[!NOTE]
> Если используется триггер BLOB-объекта в плане потребления, то после того как приложение-функция стало неактивным, обработка новых BLOB-объектов может осуществляться с задержкой до 10 минут. После запуска приложения-функции большие двоичные объекты обрабатываются немедленно. Дополнительные сведения см. в статье о [привязках и триггерах хранилища BLOB-объектов](../../azure-functions/functions-bindings-storage-blob.md).

## <a name="next-steps"></a>Дальнейшие действия

На этом этапе вы готовы начать разработку приложения служб мультимедиа.

Дополнительные сведения и примеры и решения по использованию функций Azure и Logic Apps с помощью служб мультимедиа Azure для создания настраиваемых рабочих процессов создания содержимого см. в статье [функции Azure служб мультимедиа](https://github.com/Azure-Samples/media-services-v3-dotnet-core-functions-integration) .
