---
title: Создание эскизов с помощью Media Encoder Standard c использованием .NET
description: В данной статье рассказывается, как использовать .NET для кодирования ресурсов и одновременно с этим создавать эскизы с помощью Media Encoder Standard.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: b8dab73a-1d91-4b6d-9741-a92ad39fc3f7
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 0159a706bd5f5229a12ab52bc92d240f01a2ed11
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103014049"
---
# <a name="how-to-generate-thumbnails-using-media-encoder-standard-with-net"></a>Создание эскизов с помощью Media Encoder Standard c использованием .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Media Encoder Standard можно использовать для создания одного или нескольких эскизов из входящего видео в форматах файлов изображений [JPEG](https://en.wikipedia.org/wiki/JPEG), [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) или [BMP](https://en.wikipedia.org/wiki/BMP_file_format). Вы можете отправлять задачи, которые создают только изображения, или объединить создание эскизов с кодированием. В этой статье представлено несколько примеров предустановок эскизов XML и JSON для таких сценариев. В конце этой статьи приведен [пример кода](#code_sample), показывающий, как с помощью пакета SDK для .NET для служб мультимедиа выполнить задачу кодирования.

Дополнительные сведения об элементах, используемых в примерах предустановок см. в статье [Схема Media Encoder Standard](media-services-mes-schema.md).

Обязательно изучите раздел [Рекомендации](media-services-dotnet-generate-thumbnail-with-mes.md#considerations) .
    
## <a name="example-of-a-single-png-file-preset"></a>Пример предустановки "один файл PNG"

Предустановки JSON и XML ниже можно использовать для создания одного выходного файла PNG на основе первых секунд входного видео, где кодировщик выполняет поиск "интересного" кадра. Обратите внимание, что для размера выходных изображений выбрано значение 100 %. То есть они совпадают с размером входного видео. При этом параметр Format в разделе Outputs должен соответствовать использованию PngLayers в разделе Codecs. 

### <a name="json-preset"></a>Предустановка JSON

```json
    {
      "Version": 1.0,
      "Codecs": [
        {
          "PngLayers": [
            {
              "Type": "PngLayer",
              "Width": "100%",
              "Height": "100%"
            }
          ],
          "Start": "{Best}",
          "Type": "PngImage"
        }
      ],
      "Outputs": [
        {
          "FileName": "{Basename}_{Index}{Extension}",
          "Format": {
            "Type": "PngFormat"
          }
        }
      ]
    }
```
    
### <a name="xml-preset"></a>Предустановка XML

```xml
    <?xml version="1.0" encoding="utf-16"?>
    <Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
      <Encoding>
        <PngImage Start="{Best}">
          <PngLayers>
            <PngLayer>
              <Width>100%</Width>
              <Height>100%</Height>
            </PngLayer>
          </PngLayers>
        </PngImage>
      </Encoding>
      <Outputs>
        <Output FileName="{Basename}_{Index}{Extension}">
          <PngFormat />
        </Output>
      </Outputs>
    </Preset>
```

## <a name="example-of-a-series-of-jpeg-images-preset"></a>Пример предустановки "несколько изображений JPEG"

Следующую предустановку JSON и XML можно использовать для создания 10 изображений на метках времени 5 %, 15 %, ..., 95 % входной временной шкалы, где размер изображения установлен как одна четвертая от размера входного видео.

### <a name="json-preset"></a>Предустановка JSON

```json
    {
      "Version": 1.0,
      "Codecs": [
        {
          "JpgLayers": [
            {
              "Quality": 90,
              "Type": "JpgLayer",
              "Width": "25%",
              "Height": "25%"
            }
          ],
          "Start": "5%",
          "Step": "10%",
          "Range": "96%",
          "Type": "JpgImage"
        }
      ],
      "Outputs": [
        {
          "FileName": "{Basename}_{Index}{Extension}",
          "Format": {
            "Type": "JpgFormat"
          }
        }
      ]
    }
```

### <a name="xml-preset"></a>Предустановка XML
    
```xml
    <?xml version="1.0" encoding="utf-16"?>
    <Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
      <Encoding>
        <JpgImage Start="5%" Step="10%" Range="96%">
          <JpgLayers>
            <JpgLayer>
              <Width>25%</Width>
              <Height>25%</Height>
              <Quality>90</Quality>
            </JpgLayer>
          </JpgLayers>
        </JpgImage>
      </Encoding>
      <Outputs>
        <Output FileName="{Basename}_{Index}{Extension}">
          <JpgFormat />
        </Output>
      </Outputs>
    </Preset>
```

## <a name="example-of-a-one-image-at-a-specific-timestamp-preset"></a>Пример предустановки "одно изображение на определенной метке времени"

Следующую предустановку JSON и XML можно использовать для создания одного изображения JPEG на 30-секундной отметке времени входного видео. При использовании этой предустановки предполагается, что длительность входного видео более 30 секунд (иначе задание завершается ошибкой).

### <a name="json-preset"></a>Предустановка JSON

```json
    {
      "Version": 1.0,
      "Codecs": [
        {
          "JpgLayers": [
            {
              "Quality": 90,
              "Type": "JpgLayer",
              "Width": "25%",
              "Height": "25%"
            }
          ],
          "Start": "00:00:30",
          "Step": "1",
          "Range": "1",
          "Type": "JpgImage"
        }
      ],
      "Outputs": [
        {
          "FileName": "{Basename}_{Index}{Extension}",
          "Format": {
            "Type": "JpgFormat"
          }
        }
      ]
    }
```

### <a name="xml-preset"></a>Предустановка XML
```xml
    <?xml version="1.0" encoding="utf-16"?>
    <Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
      <Encoding>
        <JpgImage Start="00:00:30" Step="00:00:01" Range="00:00:01">
          <JpgLayers>
            <JpgLayer>
              <Width>25%</Width>
              <Height>25%</Height>
              <Quality>90</Quality>
            </JpgLayer>
          </JpgLayers>
        </JpgImage>
      </Encoding>
      <Outputs>
        <Output FileName="{Basename}_{Index}{Extension}">
          <JpgFormat />
        </Output>
      </Outputs>
    </Preset>
```

## <a name="example-of-a-thumbnails-at-different-resolutions-preset"></a>Пример предустановки "эскизы с разными разрешениями"

Эту предустановку можно использовать для создания эскизов с разными разрешениями в одной задаче. В примере в позициях входной временной шкалы 5 %, 15 %, …, 95 % кодировщик создает два изображения — один с использованием 100 % входного разрешения видео, а другой с использованием 50 %.

Обратите внимание на использование макроса {Resolution} в FileName. Он указывает кодировщику использовать ширину и высоту, заданные в разделе Encoding предустановки при создании имени файла выходных изображений. Кроме того, он позволяет легко различать разные изображения.

### <a name="json-preset"></a>Предустановка JSON

```json
    {
      "Version": 1.0,
      "Codecs": [
        {
          "JpgLayers": [
        {
          "Quality": 90,
          "Type": "JpgLayer",
          "Width": "100%",
          "Height": "100%"
        },
        {
          "Quality": 90,
          "Type": "JpgLayer",
          "Width": "50%",
          "Height": "50%"
        }

          ],
          "Start": "5%",
          "Step": "10%",
          "Range": "96%",
          "Type": "JpgImage"
        }
      ],
      "Outputs": [
        {
          "FileName": "{Basename}_{Resolution}_{Index}{Extension}",
          "Format": {
        "Type": "JpgFormat"
          }
        }
      ]
    }
```

### <a name="xml-preset"></a>Предустановка XML
```xml
    <?xml version="1.0" encoding="utf-8"?>
    <Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
    <Encoding>
    <JpgImage Start="5%" Step="10%" Range="96%"><JpgImage Start="00:00:01" Step="00:00:15">
      <JpgLayers>
       <JpgLayer>
        <Width>100%</Width>
        <Height>100%</Height>
        <Quality>90</Quality>
       </JpgLayer>
       <JpgLayer>
        <Width>50%</Width>
        <Height>50%</Height>
        <Quality>90</Quality>
       </JpgLayer>
      </JpgLayers>
    </JpgImage>
    </Encoding>
    <Outputs>
      <Output FileName="{Basename}_{Resolution}_{Index}{Extension}">
        <JpgFormat/>
      </Output>
    </Outputs>
    </Preset>
```

## <a name="example-of-generating-a-thumbnail-while-encoding"></a>Пример создания эскиза во время кодирования

Во всех примерах выше рассматривались способы отправки задачи кодирования для создания изображений. Но также можно объединить кодирование видео и аудио с созданием эскизов. Предустановки JSON и XML ниже указывают **стандартному кодировщику служб мультимедиа** создавать эскиз во время кодирования.

### <a name="json-preset"></a><a id="json"></a>Предустановка JSON
Сведения о схеме см. [здесь](./media-services-mes-schema.md).

```json
    {
      "Version": 1.0,
      "Codecs": [
        {
          "KeyFrameInterval": "00:00:02",
          "SceneChangeDetection": "true",
          "H264Layers": [
            {
              "Profile": "Auto",
              "Level": "auto",
              "Bitrate": 4500,
              "MaxBitrate": 4500,
              "BufferWindow": "00:00:05",
              "Width": 1280,
              "Height": 720,
              "ReferenceFrames": 3,
              "EntropyMode": "Cabac",
              "AdaptiveBFrame": true,
              "Type": "H264Layer",
              "FrameRate": "0/1"
    
            }
          ],
          "Type": "H264Video"
        },
        {
          "JpgLayers": [
            {
              "Quality": 90,
              "Type": "JpgLayer",
              "Width": "100%",
              "Height": "100%"
            }
          ],
          "Start": "{Best}",
          "Type": "JpgImage"
        },
        {
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 128,
          "Type": "AACAudio"
        }
      ],
      "Outputs": [
        {
          "FileName": "{Basename}_{Index}{Extension}",
          "Format": {
            "Type": "JpgFormat"
          }
        },
        {
          "FileName": "{Basename}_{Resolution}_{VideoBitrate}.mp4",
          "Format": {
            "Type": "MP4Format"
          }
        }
      ]
    }
```

### <a name="xml-preset"></a><a id="xml"></a>Предустановка XML
Сведения о схеме см. [здесь](./media-services-mes-schema.md).

```xml
    <?xml version="1.0" encoding="utf-16"?>
    <Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
      <Encoding>
        <H264Video>
          <KeyFrameInterval>00:00:02</KeyFrameInterval>
          <SceneChangeDetection>true</SceneChangeDetection>
          <H264Layers>
            <H264Layer>
              <Bitrate>4500</Bitrate>
              <Width>1280</Width>
              <Height>720</Height>
              <FrameRate>0/1</FrameRate>
              <Profile>Auto</Profile>
              <Level>auto</Level>
              <BFrames>3</BFrames>
              <ReferenceFrames>3</ReferenceFrames>
              <Slices>0</Slices>
              <AdaptiveBFrame>true</AdaptiveBFrame>
              <EntropyMode>Cabac</EntropyMode>
              <BufferWindow>00:00:05</BufferWindow>
              <MaxBitrate>4500</MaxBitrate>
            </H264Layer>
          </H264Layers>
        </H264Video>
        <AACAudio>
          <Profile>AACLC</Profile>
          <Channels>2</Channels>
          <SamplingRate>48000</SamplingRate>
          <Bitrate>128</Bitrate>
        </AACAudio>
        <JpgImage Start="{Best}">
          <JpgLayers>
            <JpgLayer>
              <Width>100%</Width>
              <Height>100%/Height>
              <Quality>90</Quality>
            </JpgLayer>
          </JpgLayers>
        </JpgImage>
      </Encoding>
      <Outputs>
        <Output FileName="{Basename}_{Resolution}_{VideoBitrate}.mp4">
          <MP4Format />
        </Output>
        <Output FileName="{Basename}_{Index}{Extension}">
          <JpgFormat />
        </Output>
      </Outputs>
    </Preset>   
```

## <a name="encode-video-and-generate-thumbnail-with-net"></a><a id="code_sample"></a>Кодирование видео и создание эскиза при помощи .NET

В следующем примере кода пакет SDK служб мультимедиа используется для выполнения следующих задач.

* Создание задания кодирования.
* Получение ссылки на стандартный кодировщик мультимедиа.
* Загрузка предопределенного кода [XML](media-services-dotnet-generate-thumbnail-with-mes.md#xml) или [JSON](media-services-dotnet-generate-thumbnail-with-mes.md#json), содержащего предопределенную кодировку, а также сведения, необходимые для создания эскизов. Вы можете сохранить этот [XML](media-services-dotnet-generate-thumbnail-with-mes.md#xml)- или [JSON](media-services-dotnet-generate-thumbnail-with-mes.md#json)-код в файл и использовать указанный ниже код для загрузки файла.

    ```csharp
    // Load the XML (or JSON) from the local file.
    string configuration = File.ReadAllText(fileName);  
    ```

* Добавление одной задачи кодирования в задание. 
* Указание входного ресурса-контейнера для кодирования.
* Создание выходного ресурса-контейнера с закодированным ресурсом.
* Добавление обработчика событий для проверки хода выполнения задания.
* Отправка задания.

Дополнительные сведения о настройке среды разработки см. в статье [Media Services development with .NET](media-services-dotnet-how-to-use.md) (Разработка служб мультимедиа с помощью .NET).

```csharp
using System;
using System.Configuration;
using System.IO;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Threading;

namespace EncodeAndGenerateThumbnails
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

            // Encode and generate the thumbnails.
            EncodeToAdaptiveBitrateMP4Set(asset);

            Console.ReadLine();
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset asset)
        {
            // Declare a new job.
            IJob job = _context.Jobs.Create("Media Encoder Standard Thumbnail Job");
            // Get a media processor reference, and pass to it the name of the 
            // processor to use for the specific task.
            IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

            // Load the XML (or JSON) from the local file.
            string configuration = File.ReadAllText("ThumbnailPreset_JSON.json");

            // Create a task
            ITask task = job.Tasks.AddNew("Media Encoder Standard Thumbnail task",
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

## <a name="considerations"></a>Рекомендации
Действительны следующие условия.

* Использование явных меток времени для элементов Start, Step или Range предполагает, что входные данные составляют не менее одной минуты.
* Элементы Jpg, Png и BmpImage обладают строковыми атрибутами Start, Step и Range, которые можно интерпретировать следующим образом:
  
  * номер кадра, если эти атрибуты выражены неотрицательными целыми числами, например "Start": "120";
  * значение относительно длительности источника, если атрибуты указаны с суффиксом "%", например "Start": "15%";
  * Отметка времени, если атрибуты имеют формат ЧЧ:ММ:СС... Например: "Start" : "00:01:00".
    
    При желании условные обозначения можно комбинировать.
    
    Кроме того, атрибут Start поддерживает также специальный макрос {Best}, который пытается определить первый "интересный" кадр содержимого NOTE: (если атрибут Start имеет значение {Best}, атрибуты Step и Range игнорируются)
  * По умолчанию Start:{Best}
* Для атрибута Image должен быть указан формат выходных данных: Jpg/Png/BmpFormat. Если он присутствует, Media Encoder Standard сопоставляет JpgVideo с JpgFormat и т. д. OutputFormat представляет новый макрос, связанный с кодеком изображений: {Index}, который необходимо указывать для форматов вывода изображений (один и только один раз).

## <a name="next-steps"></a>Дальнейшие действия

Вы можете проверить [ход выполнения задания](media-services-check-job-progress.md), пока задание кодировки находится в ожидании.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>См. также:
[Обзор кодирования с помощью служб мультимедиа](media-services-encode-asset.md)