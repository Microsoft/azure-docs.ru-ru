---
title: Использование Media Encoder Standard для автоматического создания поскоростной лестнице в Azure | Документация Майкрософт
description: В этом разделе показано, как использовать Media Encoder Standard (MES) для автоматического создания таблицы скоростей и разрешений на основе разрешения и скорости входных данных.
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
ms.openlocfilehash: 6ea28d61cc142c3191d591721b92e08d651c7ed5
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103014406"
---
#  <a name="use-media-encoder-standard-to-auto-generate-a-bitrate-ladder"></a>Использование Media Encoder Standard для автоматического создания лестнице скорости

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]  

## <a name="overview"></a>Обзор

В этой статье показано, как использовать Media Encoder Standard (MES) для автоматического создания таблицы скоростей и разрешений (пар "скорость-разрешение") на основе разрешения и скорости входных данных. Созданная автоматически предустановка никогда не превышает разрешение и скорость входных данных. Например, если указаны входные данные с разрешением 720p и скоростью 3 Мбит/с, то разрешение выходных данных не выше 720p, а скорость их запуска ниже 3 Мбит/с.

### <a name="encoding-for-streaming-only"></a>Кодирование только для потоковой передачи

Чтобы применить к источнику видео кодирование только для потоковой передачи, используйте при создании задачи кодирования предустановленный режим "Adaptive Streaming". При использовании предустановки **Adaptive Streaming** кодировщик MES интеллектуально ограничит таблицу скоростей и разрешений. Однако вы не сможете контролировать затраты на кодирование, так как служба определяет, сколько уровней использовать и с каким разрешением. В конце этой статьи вы найдете примеры слоев выходных данных, созданных MES с применением кодирования с предустановленным режимом **Adaptive Streaming**. Выходной ресурс содержит MP4-файлы без чередования аудио и видео.

### <a name="encoding-for-streaming-and-progressive-download"></a>Кодирование для потоковой передачи и прогрессивного скачивания

Чтобы кодировать источник видео не только для потоковой передачи, но и для создания MP4-файлов для прогрессивного скачивания, используйте при создании задачи кодирования предустановленный режим "Content Adaptive Multiple Bitrate MP4". При выборе предустановки **Content Adaptive Multiple Bitrate MP4** кодировщик MES применяет ту же логику кодирования, которая описана выше, но в исходящий ресурс будут помещены файлы MP4 с чередованием аудио и видео. Такой MP4-файл (например, версию с самым высоким битрейтом) можно использовать для прогрессивного скачивания.

## <a name="encoding-with-media-services-net-sdk"></a><a id="encoding_with_dotnet"></a>Кодирование с помощью пакета SDK служб мультимедиа для .NET

В следующем примере кода пакет SDK служб мультимедиа используется для выполнения следующих задач.

- Создание задания кодирования.
- Получение ссылки на стандартный кодировщик мультимедиа.
- Добавьте задачу кодирования в задание и укажите предустановку **Adaptive Streaming**. 
- Создание выходного ресурса-контейнера с закодированным ресурсом.
- Добавление обработчика событий для проверки хода выполнения задания.
- Отправка задания.

#### <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

Настройте среду разработки и заполните файл app.config данными о соединении, как описано в разделе [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md). 

#### <a name="example"></a>Пример

```
using System;
using System.Configuration;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Threading;

namespace AdaptiveStreamingMESPresest
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

            // Create a task
            ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
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

## <a name="output"></a><a id="output"></a>Выходные данные

Этот раздел содержит три примера выходных данных слоев, созданных MES в результате кодирования с предустановкой **Adaptive Streaming**. 

### <a name="example-1"></a>Пример 1
При исходной высоте 1080 и частоте кадров 29,970 создается 6 слоев видео.

|Уровень|Высота|Ширина|Скорость (кбит/с)|
|---|---|---|---|
|1|1080|1920|6780|
|2|720|1280|3520|
|3|540|960|2210|
|4|360|640|1150|
|5|270|480|720|
|6|180|320|380|

### <a name="example-2"></a>Пример 2
При исходной высоте 720 и частоте кадров 23,970 создается 5 слоев видео.

|Уровень|Высота|Ширина|Скорость (кбит/с)|
|---|---|---|---|
|1|720|1280|2940|
|2|540|960|1850|
|3|360|640|960|
|4|270|480|600|
|5|180|320|320|

### <a name="example-3"></a>Пример 3
При исходной высоте 360 и частоте кадров 29,970 создается 3 слоя видео.

|Уровень|Высота|Ширина|Скорость (кбит/с)|
|---|---|---|---|
|1|360|640|700|
|2|270|480|440|
|3|180|320|230|
## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>См. также:
[Обзор кодирования с помощью служб мультимедиа](media-services-encode-asset.md)

