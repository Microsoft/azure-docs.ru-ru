---
title: Настройка предустановок Media Encoder Standard | Документация Майкрософт
description: В этом разделе показано, как выполнять расширенные задачи кодирования, настраивая предустановки задач Media Encoder Standard. В этом разделе показано, как использовать пакет SDK служб мультимедиа для .NET для создания задания и задачи кодирования. В нем также показано, как предоставить пользовательские предустановки для задания кодирования.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: ec95392f-d34a-4c22-a6df-5274eaac445b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: c2ec7c8fe312787d153c76b0b0194354e1717db7
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103016718"
---
# <a name="customizing-media-encoder-standard-presets"></a>Настройка предустановок Media Encoder Standard

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

## <a name="overview"></a>Обзор

В этой статье показано, как выполнять расширенные задачи кодирования с помощью Media Encoder Standard (MES) и пользовательской предустановки. В этой статье с помощью .NET создаются задача кодирования и задание, которое выполняет эту задачу.  

В этой статье показано, как настроить предустановку. Для примера взята предустановка [H264 Multiple Bitrate 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md), в которой уменьшается количество уровней. В статье [Настройка предустановок MES для расширенного кодирования](media-services-advanced-encoding-with-mes.md) показаны пользовательские предустановки, которые могут использоваться для выполнения расширенных задач кодирования.

> [!NOTE]
> Пользовательские предустановки, описанные в этой статье, нельзя использовать в преобразованиях [служб мультимедиа v3](../latest/index.yml) или командах CLI. Дополнительные сведения см. в [руководстве по миграции с версии 2 на v3](../latest/migrate-v-2-v-3-migration-introduction.md) .

## <a name="customizing-a-mes-preset"></a><a id="customizing_presets"></a> Настройка предустановки MES

### <a name="original-preset"></a>Первоначальная предустановка

Сохраните код JSON, определенный в статье [H264 Multiple Bitrate 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md), в отдельный JSON-файл. Например, **CustomPreset_JSON.json**.

### <a name="customized-preset"></a>Настроенная предустановка

Откройте файл **CustomPreset_JSON.json** и удалите первые три слоя из **H264Layers**, чтобы файл выглядел, как показано ниже.

```json 
  {  
    "Version": 1.0,  
    "Codecs": [  
      {  
        "KeyFrameInterval": "00:00:02",  
        "H264Layers": [  
          {  
            "Profile": "Auto",  
            "Level": "auto",  
            "Bitrate": 1000,  
            "MaxBitrate": 1000,  
            "BufferWindow": "00:00:05",  
            "Width": 640,  
            "Height": 360,  
            "BFrames": 3,  
            "ReferenceFrames": 3,  
            "AdaptiveBFrame": true,  
            "Type": "H264Layer",  
            "FrameRate": "0/1"  
          },  
          {  
            "Profile": "Auto",  
            "Level": "auto",  
            "Bitrate": 650,  
            "MaxBitrate": 650,  
            "BufferWindow": "00:00:05",  
            "Width": 640,  
            "Height": 360,  
            "BFrames": 3,  
            "ReferenceFrames": 3,  
            "AdaptiveBFrame": true,  
            "Type": "H264Layer",  
            "FrameRate": "0/1"  
          },  
          {  
            "Profile": "Auto",  
            "Level": "auto",  
            "Bitrate": 400,  
            "MaxBitrate": 400,  
            "BufferWindow": "00:00:05",  
            "Width": 320,  
            "Height": 180,  
            "BFrames": 3,  
            "ReferenceFrames": 3,  
            "AdaptiveBFrame": true,  
            "Type": "H264Layer",  
            "FrameRate": "0/1"  
          }  
        ],  
        "Type": "H264Video"  
      },  
      {  
        "Profile": "AACLC",  
        "Channels": 2,  
        "SamplingRate": 48000,  
        "Bitrate": 128,  
        "Type": "AACAudio"  
      }  
    ],  
    "Outputs": [  
      {  
        "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",  
        "Format": {  
          "Type": "MP4Format"  
        }  
      }  
    ]  
  }  
```

## <a name="encoding-with-media-services-net-sdk"></a><a id="encoding_with_dotnet"></a>Кодирование с помощью пакета SDK служб мультимедиа для .NET

В следующем примере кода пакет SDK служб мультимедиа используется для выполнения следующих задач.

- Создание задания кодирования.
- Получение ссылки на стандартный кодировщик мультимедиа.
- Загрузите пользовательскую предустановку JSON, созданную в предыдущем разделе. 

    ```csharp
    // Load the JSON from the local file.
    string configuration = File.ReadAllText(fileName);  
    ```

- Добавление задачи кодирования в задание. 
- Указание входного ресурса-контейнера для кодирования.
- Создание выходного ресурса-контейнера с закодированным ресурсом.
- Добавление обработчика событий для проверки хода выполнения задания.
- Отправка задания.
   
#### <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

Настройте среду разработки и заполните файл app.config данными о соединении, как описано в разделе [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md). 

#### <a name="example"></a>Пример   

```csharp
using System;
using System.Configuration;
using System.IO;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Threading;

namespace CustomizeMESPresests
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

            // Get an uploaded asset.
            var asset = _context.Assets.FirstOrDefault();

            // Encode and generate the output using custom presets.
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

            // Load the XML (or JSON) from the local file.
            string configuration = File.ReadAllText("CustomPreset_JSON.json");

            // Create a task
            ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
            processor,
            configuration,
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

## <a name="see-also"></a>См. также раздел

- [Как кодировать с пользовательским преобразованием с помощью интерфейса командной строки](../latest/custom-preset-cli-howto.md)
- [Кодирование с помощью Служб мультимедиа версии 3](../latest/encoding-concept.md)

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
