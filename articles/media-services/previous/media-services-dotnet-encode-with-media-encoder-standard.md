---
title: Кодирование ресурса-контейнера с помощью Media Encoder Standard и .NET | Документация Майкрософт
description: В этой статье показано, как использовать .NET для кодирования ресурса с помощью Media Encoder Standard.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 03431b64-5518-478a-a1c2-1de345999274
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.reviewer: anilmur
ms.custom: devx-track-csharp
ms.openlocfilehash: b94ec0b50182381d2a4e1bca09f4c8bc482d13ee
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103014100"
---
# <a name="encode-an-asset-with-media-encoder-standard-using-net"></a>Кодирование ресурса-контейнера с помощью Media Encoder Standard и .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

Задания кодирования — одни из самых распространенных операций обработки в службах мультимедиа. Они создаются для преобразования файлов мультимедиа из одного формата кодирования в другой. При кодировании можно использовать встроенный кодировщик служб мультимедиа. Также можно использовать кодировщик, предоставленный партнером служб мультимедиа. Кодировщики сторонних производителей доступны в Azure Marketplace. 

В этой статье показано, как использовать .NET для кодирования ресурсов с помощью Media Encoder Standard (MES). Media Encoder Standard настраивается с помощью одной из предустановок кодировщика, описанных [здесь](./media-services-mes-presets-overview.md).

Мы советуем всегда кодировать исходные файлы в набор MP4-файлов с переменной скоростью, а затем преобразовывать его в нужный формат, используя [динамическую упаковку](media-services-dynamic-packaging-overview.md). 

Если выходящий ресурс зашифрован в хранилище, необходимо настроить политику доставки ресурсов. Дополнительные сведения см. в статье [Настройка политик доставки ресурсов-контейнеров](media-services-dotnet-configure-asset-delivery-policy.md).

> [!NOTE]
> Кодировщик MES создает выходной файл с именем, содержащим первые 32 символа имени входного файла. Имя основывается на параметрах, указанных в файле предустановки. Например, "FileName": "{Basename}_{Index}{Extension}". {Basename} заменяется первыми 32 символами имени входного файла.
> 
> 

### <a name="mes-formats"></a>Форматы стандартного кодировщика мультимедиа
[Форматы и кодеки](media-services-media-encoder-standard-formats.md)

### <a name="mes-presets"></a>Предустановки стандартного кодировщика мультимедиа
Media Encoder Standard настраивается с помощью одной из предустановок кодировщика, описанных [здесь](./media-services-mes-presets-overview.md).

### <a name="input-and-output-metadata"></a>Входные и выходные метаданные
Кодируя входной актив (или активы) с использованием MES, вы получаете выходной актив после успешного выполнения задачи кодирования. Выходной актив содержит видео- и аудиофайлы, эскизы, манифест и т. д., в зависимости от используемой предустановки кодирования.

Выходной актив также содержит файл с метаданными входного актива. Имя XML-файла метаданных имеет следующий формат: <ИД_ресурса-контейнера>_metadata.xml (например, 41114ad3-eb5e-4c57-8d92-5354e2b7d4a4_metadata.xml), где <ИД_ресурса-контейнера> — это значение AssetId для входного ресурса-контейнера. Схема входного XML-файла метаданных описана [здесь](media-services-input-metadata-schema.md).

Выходной актив также содержит файл с метаданными выходного актива. Имя XML-файла метаданных имеет следующий формат: <имя_исходного_файла>_manifest.xml (например, BigBuckBunny_manifest.xml). Схема выходного XML-файла метаданных описана [здесь](media-services-output-metadata-schema.md).

Если вы хотите изучить один из двух файлов метаданных, создайте указатель SAS и загрузите файл на локальный компьютер. Пример создания указателя SAS и загрузки файла см. в статье "Использование расширения пакета SDK служб мультимедиа для .NET".

## <a name="download-sample"></a>Скачивание образца
[Отсюда](https://azure.microsoft.com/documentation/samples/media-services-dotnet-on-demand-encoding-with-media-encoder-standard/) можно скачать и выполнить пример, показывающий, как кодировать с помощью MES.

## <a name="net-sample-code"></a>Пример кода .NET

В следующем примере кода пакет SDK служб мультимедиа используется для выполнения следующих задач.

* Создание задания кодирования.
* Получение ссылки на стандартный кодировщик мультимедиа.
* Указание предустановки для [адаптивной потоковой передачи ](media-services-autogen-bitrate-ladder-with-mes.md). 
* Добавление одной задачи кодирования в задание. 
* Указание входного ресурса-контейнера для кодирования.
* Создание выходного ресурса-контейнера с закодированным ресурсом.
* Добавление обработчика событий для проверки хода выполнения задания.
* Отправка задания.

#### <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

Настройте среду разработки и укажите в файле app.config сведения о подключении, как описано в статье [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md). 

#### <a name="example"></a>Пример 

```csharp
using System;
using System.Linq;
using System.Configuration;
using System.IO;
using System.Threading;
using Microsoft.WindowsAzure.MediaServices.Client;

namespace MediaEncoderStandardSample
{
    class Program
    {
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        // Field for service context.
        private static CloudMediaContext _context = null;

        private static readonly string _supportFiles =
            Path.GetFullPath(@"../..\Media");

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);


            // Get an uploaded asset.
            var asset = _context.Assets.FirstOrDefault();

            // Encode and generate the output using the "Adaptive Streaming" preset.
            EncodeToAdaptiveBitrateMP4Set(asset);

            Console.ReadLine();
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("Media Encoder Standard Job");
            // Get a media processor reference, and pass to it the name of the 
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

            // Create a task with the encoding details, using a string preset.
            // In this case "Adaptive Streaming" preset is used.
            ITask task = job.Tasks.AddNew("My encoding task",
                processor,
                "Adaptive Streaming",
                TaskOptions.None);

            // Specify the input asset to be encoded.
            task.InputAssets.Add(asset);
            // Add an output asset to contain the results of the job. 
            // This output is specified as AssetCreationOptions.None, which 
            // means the output asset is not encrypted. 
            task.OutputAssets.AddNew("Output asset",
                AssetCreationOptions.None);

            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
            job.Submit();
            job.GetExecutionProgressTask(CancellationToken.None).Wait();

            return job.OutputMediaAssets[0];
        }

        private static void JobStateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);
            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("Job is finished. Please wait while local tasks or downloads complete...");
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
                    break;
                default:
                    break;
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
    }
}
```

## <a name="advanced-encoding-features-to-explore"></a>Функции расширенного кодирования для ознакомления
* [Как создавать эскизы](media-services-dotnet-generate-thumbnail-with-mes.md)
* [Создание эскизов во время кодирования](media-services-dotnet-generate-thumbnail-with-mes.md#example-of-generating-a-thumbnail-while-encoding)
* [Обрезка видео во время кодирования](media-services-crop-video.md)
* [Настройка предустановок для кодирования](media-services-custom-mes-presets-with-dotnet.md)
* [Создание наложения или подложки для видео с использованием изображением](media-services-advanced-encoding-with-mes.md#overlay)

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Дальнейшие действия
[Создание эскиза с помощью стандартного кодировщика мультимедиа и платформы .NET](media-services-dotnet-generate-thumbnail-with-mes.md)
[Обзор кодирования с помощью служб мультимедиа](media-services-encode-asset.md)