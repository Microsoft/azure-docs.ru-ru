---
title: Управление ресурсами Служб мультимедиа в нескольких учетных записях хранения | Документация Майкрософт
description: В этой статье приведены рекомендации по управлению файлами Служб мультимедиа в нескольких учетных записях хранения.
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
ms.openlocfilehash: b3b2cb4328a68c7718101c5acd362258706cb6d8
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103013267"
---
# <a name="managing-media-services-assets-across-multiple-storage-accounts"></a>Управление ресурсами Служб мультимедиа в нескольких учетных записях хранения  

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Можно подключить несколько учетных записей хранения к одной учетной записи Служб мультимедиа. Возможность подключить несколько учетных записей хранения к учетной записи служб мультимедиа дает несколько преимуществ:

* балансирование нагрузки ваших ресурсов-контейнеров между несколькими учетными записями хранения;
* масштабирование служб мультимедиа для больших объемов обработки содержимого (сейчас действует ограничение в 500 ТБ на одну учетную запись хранения). 

В этой статье показано, как подключить несколько учетных записей хранения к учетной записи служб мультимедиа с помощью [Azure Resource Manager API](/rest/api/media/operations/azure-media-services-rest-api-reference) и [PowerShell](/powershell/module/az.media). Здесь также описывается, как указывать другие учетные записи хранения при создании файлов с помощью пакета SDK служб мультимедиа. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="considerations"></a>Рекомендации

При подключении нескольких учетных записей хранения к своей учетной записи служб мультимедиа обратите внимание на следующее.

* Учетная запись Служб мультимедиа и все связанные учетные записи хранения должны размещаться в одной подписке Azure. Рекомендуется использовать учетные записи хранения из того же расположения, что и учетная запись Служб мультимедиа.
* После подключения учетной записи хранения к указанной учетной записи Служб мультимедиа ее невозможно отключить.
* Основная учетная запись хранения — это запись, указанная во время создания учетной записи служб мультимедиа. Сейчас нельзя изменить учетную запись хранения по умолчанию. 
* Для добавления учетной записи холодного хранилища в учетную запись AMS она должна иметь тип большого двоичного объекта и не должна быть задана как основная.

Дополнительные рекомендации

Службы мультимедиа используют значение свойства **IAssetFile.Name** при создании URL-адресов для потоковой передачи содержимого (например, http://{вамсаккаунт}. Origin. mediaservices. Windows. NET/{GUID}/{IAssetFile. Name}/streamingParameters.) По этой причине процентное кодирование запрещено. Значение свойства Name не может содержать такие [зарезервированные знаки, используемые для кодировки URL-адресов](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters): !*'();:@&=+$,/?%#[]". Кроме того, может использоваться только один знак ".". Кроме того, может использоваться только один символ "." для расширения имени файла.

## <a name="to-attach-storage-accounts"></a>Присоединение учетных записей хранения  

Чтобы присоединить учетные записи хранения к учетной записи AMS, используйте [интерфейсы api Azure Resource Manager](/rest/api/media/operations/azure-media-services-rest-api-reference) и [PowerShell](/powershell/module/az.media), как показано в следующем примере:

```azurepowershell
$regionName = "West US"
$subscriptionId = " xxxxxxxx-xxxx-xxxx-xxxx- xxxxxxxxxxxx "
$resourceGroupName = "SkyMedia-USWest-App"
$mediaAccountName = "sky"
$storageAccount1Name = "skystorage1"
$storageAccount2Name = "skystorage2"
$storageAccount1Id = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccount1Name"
$storageAccount2Id = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Storage/storageAccounts/$storageAccount2Name"
$storageAccount1 = New-AzMediaServiceStorageConfig -StorageAccountId $storageAccount1Id -IsPrimary
$storageAccount2 = New-AzMediaServiceStorageConfig -StorageAccountId $storageAccount2Id
$storageAccounts = @($storageAccount1, $storageAccount2)
    
Set-AzMediaService -ResourceGroupName $resourceGroupName -AccountName $mediaAccountName -StorageAccounts $storageAccounts
```

### <a name="support-for-cool-storage"></a>Поддержка "холодного" хранилища

В настоящее время для добавления учетной записи "холодного" хранилища в учетную запись AMS она должна иметь тип большого двоичного объекта и не быть задана как основная.

## <a name="to-manage-media-services-assets-across-multiple-storage-accounts"></a>Управление файлами служб мультимедиа в нескольких учетных записях хранения
В приведенном ниже коде используется последняя версия пакета SDK служб мультимедиа для выполнения следующих задач.

1. Отображение всех учетных записей хранения, связанных с указанной учетной записью служб мультимедиа.
2. Получение имени учетной записи хранения по умолчанию.
3. Создание нового файла в учетной записи хранения по умолчанию.
4. Создание выходного файла задания кодирования в указанной учетной записи хранения.
   
```cs
using Microsoft.WindowsAzure.MediaServices.Client;
using System;
using System.Collections.Generic;
using System.Configuration;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace MultipleStorageAccounts
{
    class Program
    {
        // Location of the media file that you want to encode. 
        private static readonly string _singleInputFilePath =
            Path.GetFullPath(@"../..\supportFiles\multifile\interview2.wmv");

        // Read values from the App.config file.
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static CloudMediaContext _context;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials = 
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            // Display the storage accounts associated with 
            // the specified Media Services account:
            foreach (var sa in _context.StorageAccounts)
                Console.WriteLine(sa.Name);

            // Retrieve the name of the default storage account.
            var defaultStorageName = _context.StorageAccounts.Where(s => s.IsDefault == true).FirstOrDefault();
            Console.WriteLine("Name: {0}", defaultStorageName.Name);
            Console.WriteLine("IsDefault: {0}", defaultStorageName.IsDefault);

            // Retrieve the name of a storage account that is not the default one.
            var notDefaultStorageName = _context.StorageAccounts.Where(s => s.IsDefault == false).FirstOrDefault();
            Console.WriteLine("Name: {0}", notDefaultStorageName.Name);
            Console.WriteLine("IsDefault: {0}", notDefaultStorageName.IsDefault);

            // Create the original asset in the default storage account.
            IAsset asset = CreateAssetAndUploadSingleFile(AssetCreationOptions.None,
                defaultStorageName.Name, _singleInputFilePath);
            Console.WriteLine("Created the asset in the {0} storage account", asset.StorageAccountName);

            // Create an output asset of the encoding job in the other storage account.
            IAsset outputAsset = CreateEncodingJob(asset, notDefaultStorageName.Name, _singleInputFilePath);
            if (outputAsset != null)
                Console.WriteLine("Created the output asset in the {0} storage account", outputAsset.StorageAccountName);

        }

        static public IAsset CreateAssetAndUploadSingleFile(AssetCreationOptions assetCreationOptions, string storageName, string singleFilePath)
        {
            var assetName = "UploadSingleFile_" + DateTime.UtcNow.ToString();

            // If you are creating an asset in the default storage account, you can omit the StorageName parameter.
            var asset = _context.Assets.Create(assetName, storageName, assetCreationOptions);

            var fileName = Path.GetFileName(singleFilePath);

            var assetFile = asset.AssetFiles.Create(fileName);

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);

            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return asset;
        }

        static IAsset CreateEncodingJob(IAsset asset, string storageName, string inputMediaFilePath)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("My encoding job");
            // Get a media processor reference, and pass to it the name of the 
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My encoding task",
                processor,
                "Adaptive Streaming",
                Microsoft.WindowsAzure.MediaServices.Client.TaskOptions.ProtectedConfiguration);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(asset);
            // Add an output asset to contain the results of the job. 
            // This output is specified as AssetCreationOptions.None, which 
            // means the output asset is not encrypted. 
            task.OutputAssets.AddNew("Output asset", storageName,
                AssetCreationOptions.None);

            // Use the following event handler to check job progress.  
            job.StateChanged += new
                    EventHandler<JobStateChangedEventArgs>(StateChanged);

            // Launch the job.
            job.Submit();

            // Check job execution and wait for job to finish. 
            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
            progressJobTask.Wait();

            // Get an updated job reference.
            job = GetJob(job.Id);

            // If job state is Error the event handling 
            // method for job progress should log errors.  Here we check 
            // for error state and exit if needed.
            if (job.State == JobState.Error)
            {
                Console.WriteLine("\nExiting method due to job error.");
                return null;
            }

            // Get a reference to the output asset from the job.
            IAsset outputAsset = job.OutputMediaAssets[0];

            return outputAsset;
        }

        private static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors.Where(p => p.Name == mediaProcessorName).
                ToList().OrderBy(p => new Version(p.Version)).LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor", mediaProcessorName));

            return processor;
        }

        private static void StateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);

            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("********************");
                    Console.WriteLine("Job is finished.");
                    Console.WriteLine("Please wait while local tasks or downloads complete...");
                    Console.WriteLine("********************");
                    Console.WriteLine();
                    Console.WriteLine();
                    break;
                case JobState.Canceling:
                case JobState.Queued:
                case JobState.Scheduled:
                case JobState.Processing:
                    Console.WriteLine("Please wait...\n");
                    break;
                case JobState.Canceled:
                case JobState.Error:
                    // Cast sender as a job.
                    IJob job = (IJob)sender;
                    // Display or log error details as needed.
                    Console.WriteLine("An error occurred in {0}", job.Id);
                    break;
                default:
                    break;
            }
        }

        static IJob GetJob(string jobId)
        {
            // Use a Linq select query to get an updated 
            // reference by Id. 
            var jobInstance =
                from j in _context.Jobs
                where j.Id == jobId
                select j;
            // Return the job reference as an Ijob. 
            IJob job = jobInstance.FirstOrDefault();

            return job;
        }
    }
}
```

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

