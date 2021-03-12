---
title: Скрытие лиц с помощью аналитики мультимедиа Azure | Документация Майкрософт
description: Azure Media Redactor — это Аналитика мультимедиа Azureный обработчик мультимедиа, который обеспечивает масштабируемую оттенок в облаке. В этой статье показано, как исправить лица с помощью Azure Media Analytics.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 7d416810f6a39fb36bfa3c5225301fe87fdd128c
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103013471"
---
# <a name="redact-faces-with-azure-media-analytics"></a>Скрытие лиц с помощью аналитики мультимедиа Azure

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Обзор

**Редактор мультимедиа Azure** — это обработчик [медиа-аналитики Azure](./legacy-components.md) с возможностью масштабируемого скрытия лиц в облаке. Функция скрытия лиц позволяет изменять видео, размывая изображения лиц выбранных пользователей. Вы можете использовать функцию скрытия лиц в ситуациях, требующих соблюдения общественной безопасности, а также при работе с новостями. Редактирование короткого материала с несколькими лицами вручную может занять несколько часов, тогда как при использовании функции скрытия лиц достаточно выполнить несколько простых действий.

Эта статья посвящена **Azure Media Redactor** и способам его использования с пакетом SDK служб мультимедиа для .NET.

## <a name="face-redaction-modes"></a>Режимы скрытия лиц

Во время работы этой функции в каждом видеокадре лица определяются и отслеживаются на протяжении всей записи. Таким образом, лицо одного и того же человека можно размыть, даже если оно снято с другого ракурса. Автоматическое скрытие — это сложный процесс, который не всегда справляется с поставленной задачей. По этой причине служба "Аналитика мультимедиа" предусматривает несколько способов изменения полученного результата.

Помимо полностью автоматического режима, доступен двухэтапный рабочий процесс, в рамках которого вы можете выбирать и отменять выбор обнаруженных лиц с помощью списка идентификаторов. Кроме того, чтобы сделать изменения в каждом кадре произвольными, обработчик мультимедиа использует файл метаданных в формате JSON. Этот рабочий процесс разделен на два режима: **анализ** и **скрытие**. Эти два режима можно объединить, чтобы они выполнялись в рамках одного задания. Такой режим называется **объединенным**.

   > [!NOTE]
   > Обработчик мультимедиа детектора лиц устарел до июня 2020, [устаревшие компоненты служб мультимедиа Azure](./legacy-components.md). Рассмотрите возможность использования API служб мультимедиа Azure v3.

### <a name="combined-mode"></a>Объединенный режим

В результате вы получаете MP4-файл с автоматическим скрытием, который не нужно править вручную.

| Этап | Имя файла | Примечания |
| --- | --- | --- |
| Входной ресурс-контейнер |foo.bar |Видео в формате WMV, MOV или MP4 |
| Входная конфигурация |Конфигурация задания (предустановка) |{'version':'1.0', 'options': {'mode':'combined'}} |
| Выходной ресурс-контейнер |foo_redacted.mp4 |Видео с размытием |

### <a name="analyze-mode"></a>Режим анализа

В режиме **анализа** двухэтапный рабочий процесс принимает входное видео и создает JSON-файл с регистрацией расположения лиц, а также изображения всех обнаруженных лиц в формате JPG.

| Этап | Имя файла | Примечания |
| --- | --- | --- |
| Входной ресурс-контейнер |foo.bar |Видео в формате WMV, MPV или MP4 |
| Входная конфигурация |Конфигурация задания (предустановка) |{'version':'1.0', 'options': {'mode':'analyze'}} |
| Выходной ресурс-контейнер |foo_annotations.json |Аннотация с данными о расположении лиц в формате JSON. Вы можете отредактировать эти данные, чтобы изменить границы размытия. См. пример ниже. |
| Выходной ресурс-контейнер |foo_thumb%06d.jpg [foo_thumb000001.jpg, foo_thumb000002.jpg] |Обрезанный JPG-файл с изображением каждого обнаруженного лица, где номер соответствует идентификатору метки лица. |

#### <a name="output-example"></a>Пример выходных данных

```json
{
  "version": 1,
  "timescale": 24000,
  "offset": 0,
  "framerate": 23.976,
  "width": 1280,
  "height": 720,
  "fragments": [
    {
      "start": 0,
      "duration": 48048,
      "interval": 1001,
      "events": [
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [],
        [
          {
            "index": 13,
            "id": 1138,
            "x": 0.29537,
            "y": -0.18987,
            "width": 0.36239,
            "height": 0.80335
          },
          {
            "index": 13,
            "id": 2028,
            "x": 0.60427,
            "y": 0.16098,
            "width": 0.26958,
            "height": 0.57943
          }
        ],

    ... truncated
```

### <a name="redact-mode"></a>Режим скрытия

Во время второго этапа рабочий процесс принимает большее количество входных данных для объединения в один ресурс-контейнер.

Сюда входит список идентификаторов размываемых лиц, исходное видео и аннотации в формате JSON. Этот режим использует аннотации для применения размытия ко входному видео.

Выходные данные этапа анализа не включают исходное видео. Видео необходимо передать во входной ресурс-контейнер, чтобы задача режима скрытия обработала его, а затем выбрать в качестве первичного файла.

| Этап | Имя файла | Примечания |
| --- | --- | --- |
| Входной ресурс-контейнер |foo.bar |Видео в формате WMV, MPV или MP4. То же видео, что и на этапе 1. |
| Входной ресурс-контейнер |foo_annotations.json |Файл аннотации с метаданными, полученными на этапе 1, с необязательными изменениями. |
| Входной ресурс-контейнер |foo_IDList.txt (необязательный) |(Необязательно.) Новый список идентификаторов скрываемых лиц со строками-разделителями. Если оставить его пустым, будут размыты все лица. |
| Входная конфигурация |Конфигурация задания (предустановка) |{'version':'1.0', 'options': {'mode':'redact'}} |
| Выходной ресурс-контейнер |foo_redacted.mp4 |Видео с размытием, примененным на основе аннотаций |

#### <a name="example-output"></a>Пример выходных данных

Выходные данные из списка идентификаторов с одним выбранным идентификатором.

Пример foo_IDList.txt

```
1
2
3
```

## <a name="blur-types"></a>Типы размытия

В режиме **Combined** (Объединенный) или **Redact** (Скрытие) существует 5 различных режимов размытия, которые можно выбирать с помощью входной конфигурации JSON: **Low** (Низкий), **Med** (Средний), **High** (Высокий), **Box** (Рамка) и **Black** (Черный). По умолчанию используется режим **Med** (Средний).

Примеры типов размытия можно просмотреть ниже.

### <a name="example-json"></a>Пример JSON

```json
{
    'version':'1.0',
    'options': {
        'Mode': 'Combined',
        'BlurType': 'High'
    }
}
```

#### <a name="low"></a>Низкий

![Низкий](./media/media-services-face-redaction/blur1.png)

#### <a name="med"></a>Средний

![Средний](./media/media-services-face-redaction/blur2.png)

#### <a name="high"></a>Высокий

![Высокий](./media/media-services-face-redaction/blur3.png)

#### <a name="box"></a>Box

![Box](./media/media-services-face-redaction/blur4.png)

#### <a name="black"></a>Черный

![Черный](./media/media-services-face-redaction/blur5.png)

## <a name="elements-of-the-output-json-file"></a>Элементы выходного JSON-файла

Обработчик мультимедиа с функцией скрытия лиц обеспечивает высокую точность обнаружения и отслеживания лиц с возможностью определения до 64 человеческих лиц в одном видеокадре. Скрытие лиц в анфас выполняется с лучшим результатом по сравнению с лицами в профиль или небольшим лицами (не более 24x24 пикселей), которые сложнее обработать.

[!INCLUDE [media-services-analytics-output-json](../../../includes/media-services-analytics-output-json.md)]

## <a name="net-sample-code"></a>Пример кода .NET

В следующей программе показано, как выполнить следующие задачи.

1. Создать ресурс-контейнера и отправить в него файл мультимедиа.
2. Создание задания с задачей скрытия лиц на основе файла конфигурации, содержащего следующую предустановку JSON: 

    ```json
            {
                'version':'1.0',
                'options': {
                    'mode':'combined'
                }
            }
    ```

3. Загрузка выходных JSON-файлов.

### <a name="create-and-configure-a-visual-studio-project"></a>Создание и настройка проекта Visual Studio

Настройте среду разработки и заполните файл app.config данными о соединении, как описано в разделе [Разработка служб мультимедиа с помощью .NET](media-services-dotnet-how-to-use.md).

#### <a name="example"></a>Пример

```csharp
using System;
using System.Configuration;
using System.IO;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Threading;
using System.Threading.Tasks;

namespace FaceRedaction
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

            // Run the FaceRedaction job.
            var asset = RunFaceRedactionJob(@"C:\supportFiles\FaceRedaction\SomeFootage.mp4",
                        @"C:\supportFiles\FaceRedaction\config.json");

            // Download the job output asset.
            DownloadAsset(asset, @"C:\supportFiles\FaceRedaction\Output");
        }

        static IAsset RunFaceRedactionJob(string inputMediaFilePath, string configurationFile)
        {
            // Create an asset and upload the input media file to storage.
            IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
            "My Face Redaction Input Asset",
            AssetCreationOptions.None);

            // Declare a new job.
            IJob job = _context.Jobs.Create("My Face Redaction Job");

            // Get a reference to Azure Media Redactor.
            string MediaProcessorName = "Azure Media Redactor";

            var processor = GetLatestMediaProcessorByName(MediaProcessorName);

            // Read configuration from the specified file.
            string configuration = File.ReadAllText(configurationFile);

            // Create a task with the encoding details, using a string preset.
            ITask task = job.Tasks.AddNew("My Face Redaction Task",
            processor,
            configuration,
            TaskOptions.None);

            // Specify the input asset.
            task.InputAssets.Add(asset);

            // Add an output asset to contain the results of the job.
            task.OutputAssets.AddNew("My Face Redaction Output Asset", AssetCreationOptions.None);

            // Use the following event handler to check job progress.  
            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);

            // Launch the job.
            job.Submit();

            // Check job execution and wait for job to finish.
            Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);

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
                return null;
            }

            return job.OutputMediaAssets[0];
        }

        static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
        {
            IAsset asset = _context.Assets.Create(assetName, options);

            var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
            assetFile.Upload(filePath);

            return asset;
        }

        static void DownloadAsset(IAsset asset, string outputDirectory)
        {
            foreach (IAssetFile file in asset.AssetFiles)
            {
                file.Download(Path.Combine(outputDirectory, file.Name));
            }
        }

        static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
        {
            var processor = _context.MediaProcessors
            .Where(p => p.Name == mediaProcessorName)
            .ToList()
            .OrderBy(p => new Version(p.Version))
            .LastOrDefault();

            if (processor == null)
                throw new ArgumentException(string.Format("Unknown media processor",
                                       mediaProcessorName));

            return processor;
        }

        static private void StateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine("Job state changed event:");
            Console.WriteLine("  Previous state: " + e.PreviousState);
            Console.WriteLine("  Current state: " + e.CurrentState);

            switch (e.CurrentState)
            {
                case JobState.Finished:
                    Console.WriteLine();
                    Console.WriteLine("Job is finished.");
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
                    // LogJobStop(job.Id);
                    break;
                default:
                    break;
            }
        }
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-links"></a>Связанные ссылки

[Общие сведения об аналитике служб мультимедиа Azure](./legacy-components.md)

[Демонстрационные материалы для медиааналитики Azure](http://amslabs.azurewebsites.net/demos/Analytics.html)
