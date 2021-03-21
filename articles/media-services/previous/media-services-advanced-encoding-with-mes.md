---
title: Настройка предустановок MES для расширенного кодирования | Документация Майкрософт
description: В этом разделе показано, как выполнять расширенные задачи кодирования, настраивая предустановки задач Media Encoder Standard.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.assetid: 2a4ade25-e600-4bce-a66e-e29cf4a38369
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 3/10/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 345fd56e5ff069ea222661a4820e75d96a9e2395
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103014508"
---
# <a name="perform-advanced-encoding-by-customizing-mes-presets"></a>Настройка предустановок MES для расширенного кодирования

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Обзор

В этом разделе показано, как настроить предустановки Media Encoder Standard. В разделе [Настройка предустановок Media Encoder Standard](media-services-custom-mes-presets-with-dotnet.md) показано, как использовать .NET для создания задачи кодирования и задания, которое выполняет эту задачу. Настроив предустановку, укажите пользовательские предустановки для задачи кодирования. 

При использовании предустановки XML обязательно сохраните порядок элементов, как показано в примерах XML ниже (например, элемент KeyFrameInterval должен предшествовать элементу SceneChangeDetection).

> [!NOTE] 
> Многие расширенные функции служб мультимедиа версии 2 Media Encoder Standard в настоящее время недоступны в версии 3. Дополнительные сведения см. [в разделе руководств по миграции](../latest/migrate-v-2-v-3-migration-introduction.md).

## <a name="support-for-relative-sizes"></a>Поддержка относительных размеров

При создании эскизов вам не нужно всегда указывать выходную ширину и высоту в пикселях. Эти значения можно указать в процентах, используя диапазон [1%, ..., 100%].

### <a name="json-preset"></a>Предустановка JSON

```json
"Width": "100%",
"Height": "100%"
```

### <a name="xml-preset"></a>Предустановка XML

```xml
<Width>100%</Width>
<Height>100%</Height>
```

## <a name="generate-thumbnails"></a>Создание эскизов

В этом разделе показано, как настроить предустановку, которая создает эскизы. Предустановка, определенная ниже, содержит сведения о том, как должен кодироваться файл, а также сведения, необходимые для создания эскизов. Вы можете использовать любую из предустановок Media Encoder Standard (MES), которые приведены в [этом](media-services-mes-presets-overview.md) разделе, и добавить в нее код для создания эскизов.  

> [!NOTE]
> Если видео преобразуется в односкоростной формат, параметр **SceneChangeDetection** в следующей конфигурации может иметь только значение True. Если видео преобразуется в формат с несколькими скоростями, а параметр **SceneChangeDetection** имеет значение True, то кодировщик выдает ошибку.  
>
>

Сведения о схеме см. [здесь](media-services-mes-schema.md).

Обязательно изучите раздел [Рекомендации](#considerations) .

### <a name="json-preset"></a>Предустановка JSON

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
          "Width": 640,
          "Height": 360
        }
      ],
      "Start": "{Best}",
      "Type": "JpgImage"
    },
    {
      "PngLayers": [
        {
          "Type": "PngLayer",
          "Width": 640,
          "Height": 360,
        }
      ],
      "Start": "00:00:01",
      "Step": "00:00:10",
      "Range": "00:00:58",
      "Type": "PngImage"
    },
    {
      "BmpLayers": [
        {
          "Type": "BmpLayer",
          "Width": 640,
          "Height": 360
        }
      ],
      "Start": "10%",
      "Step": "10%",
      "Range": "90%",
      "Type": "BmpImage"
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
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "PngFormat"
      }
    },
    {
      "FileName": "{Basename}_{Index}{Extension}",
      "Format": {
        "Type": "BmpFormat"
      }
    },
    {
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
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
          <Width>640</Width>
          <Height>360</Height>
          <Quality>90</Quality>
        </JpgLayer>
      </JpgLayers>
    </JpgImage>
    <BmpImage Start="10%" Step="10%" Range="90%">
      <BmpLayers>
        <BmpLayer>
          <Width>640</Width>
          <Height>360</Height>
        </BmpLayer>
      </BmpLayers>
    </BmpImage>
    <PngImage Start="00:00:01" Step="00:00:10" Range="00:00:58">
      <PngLayers>
        <PngLayer>
          <Width>640</Width>
          <Height>360</Height>
        </PngLayer>
      </PngLayers>
    </PngImage>
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
      <MP4Format />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <JpgFormat />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <BmpFormat />
    </Output>
    <Output FileName="{Basename}_{Index}{Extension}">
      <PngFormat />
    </Output>
  </Outputs>
</Preset>
```

### <a name="considerations"></a>Рекомендации

Действительны следующие условия.

* Использование явных меток времени для элементов Start, Step или Range предполагает, что входные данные составляют не менее одной минуты.
* Элементы Jpg, Png и BmpImage обладают строковыми атрибутами Start, Step и Range, которые можно интерпретировать следующим образом:

  * номер кадра, если эти атрибуты выражены неотрицательными целыми числами, например "Start": "120";
  * значение относительно длительности источника, если атрибуты указаны с суффиксом "%", например "Start": "15%";
  * Отметка времени, если атрибуты имеют формат ЧЧ:ММ:СС, например "Start" : "00:01:00".

    При желании условные обозначения можно комбинировать.

    Кроме того, Start также поддерживает специальный макрос: {Best}, который пытается определить первый "интересный" кадр содержимого Примечание: (шаг и диапазон не учитываются, если для параметра Start задано значение {Best}).
  * По умолчанию Start:{Best}
* Для атрибута Image должен быть указан формат выходных данных: Jpg/Png/BmpFormat. Если он присутствует, Media Encoder Standard сопоставляет JpgVideo с JpgFormat и т. д. OutputFormat представляет новый макрос, связанный с кодеком изображений: {Index}, который необходимо указывать для форматов вывода изображений (один и только один раз).

## <a name="trim-a-video-clipping"></a><a id="trim_video"></a>Монтаж видео (обрезка)
В этом разделе рассказывается об изменении предустановок кодировщика для обрезки входного видеоролика, при котором входной видеоролик представляет собой так называемый мезонинный файл или файл по требованию. Кодировщик может также использоваться для обрезки ресурса-контейнера, который записывается или архивируется из потока в реальном времени. Подробные сведения об этом см. в [этом блоге](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/).

Для обрезки видео можно взять любую из предустановок стандартного кодировщика мультимедиа, которые описаны в [этом](media-services-mes-presets-overview.md) разделе, и изменить элемент **Sources** (как показано ниже). Значение StartTime должно соответствовать абсолютной метке времени входящего видео. Например, если первый кадр входящего видео имеет отметку времени "12:00:10.000", значение StartTime должно равняться или быть больше 12:00:10.000. В приведенном ниже примере мы предполагаем, что входящее видео имеет отметку времени начала, равную нулю. **Sources** должен размещаться в начале предустановки.

### <a name="json-preset"></a><a id="json"></a>Предустановка JSON

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "StartTime": "00:00:04",
      "Duration": "00:00:16"
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "StretchMode": "AutoSize",
      "H264Layers": [
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 3400,
          "MaxBitrate": 3400,
          "BufferWindow": "00:00:05",
          "Width": 1280,
          "Height": 720,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 2250,
          "MaxBitrate": 2250,
          "BufferWindow": "00:00:05",
          "Width": 960,
          "Height": 540,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 1500,
          "MaxBitrate": 1500,
          "BufferWindow": "00:00:05",
          "Width": 960,
          "Height": 540,
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "FrameRate": "0/1"
        },
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

### <a name="xml-preset"></a>Предустановка XML
Для обрезки видео можно взять любую из предустановок стандартного кодировщика мультимедиа, которые описаны [здесь](media-services-mes-presets-overview.md) , и изменить элемент **Sources** (как показано ниже).

```xml
<?xml version="1.0" encoding="utf-16"?>
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
  <Sources>
    <Source StartTime="PT4S" Duration="PT14S"/>
  </Sources>
  <Encoding>
    <H264Video>
      <KeyFrameInterval>00:00:02</KeyFrameInterval>
      <H264Layers>
        <H264Layer>
          <Bitrate>3400</Bitrate>
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
          <MaxBitrate>3400</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>2250</Bitrate>
          <Width>960</Width>
          <Height>540</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>2250</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>1500</Bitrate>
          <Width>960</Width>
          <Height>540</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1500</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>1000</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1000</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>650</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>650</MaxBitrate>
        </H264Layer>
        <H264Layer>
          <Bitrate>400</Bitrate>
          <Width>320</Width>
          <Height>180</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>3</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cabac</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>400</MaxBitrate>
        </H264Layer>
      </H264Layers>
    </H264Video>
    <AACAudio>
      <Profile>AACLC</Profile>
      <Channels>2</Channels>
      <SamplingRate>48000</SamplingRate>
      <Bitrate>128</Bitrate>
    </AACAudio>
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">
      <MP4Format />
    </Output>
  </Outputs>
</Preset>
```

## <a name="create-an-overlay"></a><a id="overlay"></a>Создание наложения

Стандартный кодировщик служб мультимедиа позволяет наложить изображение на существующее видео. В настоящее время поддерживаются следующие форматы: png, jpg, gif и bmp. Предустановка, определенная ниже, представляет собой базовый пример наложения видео.

Наряду с определением файла предустановки также необходимо сообщить службам мультимедиа, какой файл в ресурсе-контейнере содержит изображение для наложения, а какой — исходное видео, на которое вы хотите наложить изображение. Видеофайл должен быть **первичным** файлом.

При использовании .NET добавьте две приведенные ниже функции в пример .NET, определенный в [этом](media-services-custom-mes-presets-with-dotnet.md#encoding_with_dotnet) разделе. Функция **UploadMediaFilesFromFolder** передает файлы из папки (например, BigBuckBunny.mp4 и Image001.png) и устанавливает MP4-файл в качестве первичного файла в ресурсе. Функция **EncodeWithOverlay** использует настраиваемый файл предустановки, который был передан в нее (например, приведенную ниже предустановку), для создания задачи кодирования.

```csharp
static public IAsset UploadMediaFilesFromFolder(string folderPath)
{
    IAsset asset = _context.Assets.CreateFromFolder(folderPath, AssetCreationOptions.None);

    foreach (var af in asset.AssetFiles)
    {
        // The following code assumes 
        // you have an input folder with one MP4 and one overlay image file.
        if (af.Name.Contains(".mp4"))
            af.IsPrimary = true;
        else
            af.IsPrimary = false;

        af.Update();
    }

    return asset;
}

static public IAsset EncodeWithOverlay(IAsset assetSource, string customPresetFileName)
{
    // Declare a new job.
    IJob job = _context.Jobs.Create("Media Encoder Standard Job");
    // Get a media processor reference, and pass to it the name of the 
    // processor to use for the specific task.
    IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

    // Load the XML (or JSON) from the local file.
    string configuration = File.ReadAllText(customPresetFileName);

    // Create a task
    ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
        processor,
        configuration,
        TaskOptions.None);

    // Specify the input assets to be encoded.
    // This asset contains a source file and an overlay file.
    task.InputAssets.Add(assetSource);

    // Add an output asset to contain the results of the job. 
    task.OutputAssets.AddNew("Output asset",
        AssetCreationOptions.None);

    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
    job.Submit();
    job.GetExecutionProgressTask(CancellationToken.None).Wait();

    return job.OutputMediaAssets[0];
}
```

> [!NOTE]
> Текущие ограничения:
>
> Настройка прозрачности наложения не поддерживается.
>
> Исходный видеофайл и файл образа наложения должны находиться в одном ресурсе, причем видеофайл должен быть задан как основной файл в этом ресурсе.
>
>

### <a name="json-preset"></a>Предустановка JSON

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "Streams": [],
      "Filters": {
        "VideoOverlay": {
          "Position": {
            "X": 100,
            "Y": 100,
            "Width": 100,
            "Height": 50
          },
          "AudioGainLevel": 0.0,
          "MediaParams": [
            {
              "OverlayLoopCount": 1
            },
            {
              "IsOverlay": true,
              "OverlayLoopCount": 1
            }
          ],
          "Source": "Image001.png",
          "Clip": {
            "Duration": "00:00:05"
          },
          "FadeInDuration": {
            "Duration": "00:00:01"
          },
          "FadeOutDuration": {
            "StartTime": "00:00:03",
            "Duration": "00:00:04"
          }
        }
      },
      "Pad": true
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "H264Layers": [
        {
          "Profile": "Auto",
          "Level": "auto",
          "Bitrate": 1045,
          "MaxBitrate": 1045,
          "BufferWindow": "00:00:05",
          "ReferenceFrames": 3,
          "EntropyMode": "Cavlc",
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "Width": "640",
          "Height": "360",
          "FrameRate": "0/1"
        }
      ],
      "Type": "H264Video"
    },
    {
      "Type": "CopyAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}{Extension}",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

### <a name="xml-preset"></a>Предустановка XML

```xml
<?xml version="1.0" encoding="utf-16"?>
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">
  <Sources>
    <Source>
      <Streams />
      <Filters>
        <VideoOverlay>
          <Source>Image001.png</Source>
          <Clip Duration="PT5S" />
          <FadeInDuration Duration="PT1S" />
          <FadeOutDuration StartTime="PT3S" Duration="PT4S" />
          <Position X="100" Y="100" Width="100" Height="50" />
          <Opacity>0</Opacity>
          <AudioGainLevel>0</AudioGainLevel>
          <MediaParams>
            <MediaParam>
              <IsOverlay>false</IsOverlay>
              <OverlayLoopCount>1</OverlayLoopCount>
            </MediaParam>
            <MediaParam>
              <IsOverlay>true</IsOverlay>
              <OverlayLoopCount>1</OverlayLoopCount>
            </MediaParam>
          </MediaParams>
        </VideoOverlay>
      </Filters>
      <Pad>true</Pad>
    </Source>
  </Sources>
  <Encoding>
    <H264Video>
      <KeyFrameInterval>00:00:02</KeyFrameInterval>
      <H264Layers>
        <H264Layer>
          <Bitrate>1045</Bitrate>
          <Width>640</Width>
          <Height>360</Height>
          <FrameRate>0/1</FrameRate>
          <Profile>Auto</Profile>
          <Level>auto</Level>
          <BFrames>0</BFrames>
          <ReferenceFrames>3</ReferenceFrames>
          <Slices>0</Slices>
          <AdaptiveBFrame>true</AdaptiveBFrame>
          <EntropyMode>Cavlc</EntropyMode>
          <BufferWindow>00:00:05</BufferWindow>
          <MaxBitrate>1045</MaxBitrate>
        </H264Layer>
      </H264Layers>
    </H264Video>
    <CopyAudio />
  </Encoding>
  <Outputs>
    <Output FileName="{Basename}{Extension}">
      <MP4Format />
    </Output>
  </Outputs>
</Preset>
```

## <a name="insert-a-silent-audio-track-when-input-has-no-audio"></a><a id="silent_audio"></a>Вставка звуковой дорожки с тишиной, если входные данные не содержат звука
По умолчанию при отправке кодировщику входных данных, которые содержат только видео и не содержат звука, выходной ресурс-контейнер будет содержать файлы только с видеоданными. Некоторые проигрыватели не смогут обработать такие выходные потоки. Этот параметр можно использовать для принудительного добавления кодировщиком звуковой дорожки с тишиной к выходным данным в этом сценарии.

Чтобы заставить кодировщик создать ресурс, содержащий звуковую дорожку с тишиной, когда входные данные не содержат звука, укажите значение "InsertSilenceIfNoAudio".

Вы можете использовать любую из предустановок MES, которые определены в [этом](media-services-mes-presets-overview.md) разделе, и внести в нее следующие изменения.

### <a name="json-preset"></a>Предустановка JSON

```json
{
  "Channels": 2,
  "SamplingRate": 44100,
  "Bitrate": 96,
  "Type": "AACAudio",
  "Condition": "InsertSilenceIfNoAudio"
}
```

### <a name="xml-preset"></a>Предустановка XML

```xml
<AACAudio Condition="InsertSilenceIfNoAudio">
  <Channels>2</Channels>
  <SamplingRate>44100</SamplingRate>
  <Bitrate>96</Bitrate>
</AACAudio>
```

## <a name="disable-auto-de-interlacing"></a><a id="deinterlacing"></a>Отключение автоматического устранения чересстрочной развертки
Клиентам не нужно ничего делать, если, например, содержимое чересстрочная развертки автоматически переключается из чередования. Если автоматическое устранение чересстрочной развертки включено (по умолчанию), стандартный кодировщик мультимедиа не выполняет автоматическое обнаружение чересстрочных кадров и устраняет чересстрочную развертку кадров, которые помечены как чересстрочные.

Автоматическое удаление чересстрочной развертки можно отключить. Использовать этот параметр не рекомендуется.

### <a name="json-preset"></a>Предустановка JSON

```json
"Sources": [
  {
    "Filters": {
      "Deinterlace": {
        "Mode": "Off"
      }
    },
  }
]
```

### <a name="xml-preset"></a>Предустановка XML

```xml
<Sources>
  <Source>
    <Filters>
      <Deinterlace>
        <Mode>Off</Mode>
      </Deinterlace>
    </Filters>
  </Source>
</Sources>
```

## <a name="audio-only-presets"></a><a id="audio_only"></a>Предустановки только для звука
В этом разделе демонстрируются две предустановки стандартного кодировщика мультимедиа только для звука: "Звук в формате AAC" и "Звук в формате AAC хорошего качества".

### <a name="aac-audio"></a>Звук в формате AAC

```json
{
  "Version": 1.0,
  "Codecs": [
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
      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

### <a name="aac-good-quality-audio"></a>Звук в формате AAC хорошего качества

```json
{
  "Version": 1.0,
  "Codecs": [
    {
      "Profile": "AACLC",
      "Channels": 2,
      "SamplingRate": 48000,
      "Bitrate": 192,
      "Type": "AACAudio"
    }
  ],
  "Outputs": [
    {
      "FileName": "{Basename}_AAC_{AudioBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="concatenate-two-or-more-video-files"></a><a id="concatenate"></a>Объединение двух или нескольких видеофайлов

В следующем примере показано, как создать предустановку для сцепления двух или более видеофайлов. Наиболее распространенный сценарий — это добавление названия или трейлера к основному видео. Используется, когда совместно редактируемые видеофайлы имеют одинаковые свойства (разрешение видео, частоту кадров, количество аудиодорожек и т. д.). Не следует комбинировать видео с разной частотой кадров или разным количеством аудиодорожек.

>[!NOTE]
>Текущая структура функции сцепки предполагает, что исходные видеозаписи имеют согласованное разрешение, частоту кадров и т. д. 

### <a name="requirements-and-considerations"></a>Требования и рекомендации

* В исходных видеофайлах должна быть только одна аудиодорожка.
* У всех исходных видеофайлов должна быть одинаковая частота кадров.
* Необходимо загрузить видеофайлы в отдельные ресурсы и задать видео как основной файл в каждом ресурсе.
* Необходимо знать длительность видеофайлов.
* Приведенные ниже примеры предустановок предполагают, что все исходные видеофайлы начинаются с нулевой отметки. Если видеофайлы начинаются с разных меток времени, необходимо изменить значения StartTime (как при работе с архивами прямых трансляций).
* Предустановка JSON включает прямые ссылки на значения AssetID исходных ресурсов.
* Образец кода предполагает, что предустановка JSON сохранена как локальный файл, например C:\supportFiles\preset.json. Кроме того, предполагается, что ресурсы созданы путем загрузки двух видеофайлов и что вам известны соответствующие значения AssetID.
* Фрагмент кода и предустановка JSON демонстрирует пример сцепления двух видеофайлов. Количество видеофайлов можно увеличить следующим образом.

  1. Вызвать метод task.InputAssets.Add() несколько раз, чтобы добавить несколько видео по порядку.
  2. Внести соответствующие права в элемент Sources (Источники) файла JSON, добавив новые записи в том же порядке.

### <a name="net-code"></a>Код .NET

```csharp
IAsset asset1 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:606db602-efd7-4436-97b4-c0b867ba195b").FirstOrDefault();
IAsset asset2 = _context.Assets.Where(asset => asset.Id == "nb:cid:UUID:a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e").FirstOrDefault();

// Declare a new job.
IJob job = _context.Jobs.Create("Media Encoder Standard Job for Concatenating Videos");
// Get a media processor reference, and pass to it the name of the
// processor to use for the specific task.
IMediaProcessor processor = GetLatestMediaProcessorByName("Media Encoder Standard");

// Load the XML (or JSON) from the local file.
string configuration = File.ReadAllText(@"c:\supportFiles\preset.json");

// Create a task
ITask task = job.Tasks.AddNew("Media Encoder Standard encoding task",
    processor,
    configuration,
    TaskOptions.None);

// Specify the input videos to be concatenated (in order).
task.InputAssets.Add(asset1);
task.InputAssets.Add(asset2);
// Add an output asset to contain the results of the job.
// This output is specified as AssetCreationOptions.None, which
// means the output asset is not encrypted.
task.OutputAssets.AddNew("Output asset",
    AssetCreationOptions.None);

job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
job.Submit();
job.GetExecutionProgressTask(CancellationToken.None).Wait();
```

### <a name="json-preset"></a>Предустановка JSON

Обновите настроенную вами предустановку, указав идентификаторы ресурсов, которые нужно сцепить, и соответствующий промежуток времени для каждого видеофайла.

```json
{
  "Version": 1.0,
  "Sources": [
    {
      "AssetID": "606db602-efd7-4436-97b4-c0b867ba195b",
      "StartTime": "00:00:01",
      "Duration": "00:00:15"
    },
    {
      "AssetID": "a7e2b90f-0565-4a94-87fe-0a9fa07b9c7e",
      "StartTime": "00:00:02",
      "Duration": "00:00:05"
    }
  ],
  "Codecs": [
    {
      "KeyFrameInterval": "00:00:02",
      "SceneChangeDetection": true,
      "H264Layers": [
        {
          "Level": "auto",
          "Bitrate": 1800,
          "MaxBitrate": 1800,
          "BufferWindow": "00:00:05",
          "BFrames": 3,
          "ReferenceFrames": 3,
          "AdaptiveBFrame": true,
          "Type": "H264Layer",
          "Width": "640",
          "Height": "360",
          "FrameRate": "0/1"
        }
      ],
      "Type": "H264Video"
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
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",
      "Format": {
        "Type": "MP4Format"
      }
    }
  ]
}
```

## <a name="crop-videos-with-media-encoder-standard"></a><a id="crop"></a>Обрезка видео с помощью Media Encoder Standard
Ознакомьтесь с разделом [Обрезка видео с помощью стандартного кодировщика мультимедиа](media-services-crop-video.md) .

## <a name="insert-a-video-track-when-input-has-no-video"></a><a id="no_video"></a>Вставка видеодорожки, если во входных данных нет видео

По умолчанию при отправке кодировщику входных данных, которые содержат только звук и не содержат аудио, выходной ресурс-контейнер будет содержать файлы только с аудиоданными. Некоторые проигрыватели, в том числе Проигрыватель мультимедиа Azure ([подробнее здесь](https://feedback.azure.com/forums/169396-azure-media-services/suggestions/8082468-audio-only-scenarios)), могут не поддерживать обработку таких потоков. Этот параметр можно использовать для принудительного добавления кодировщиком монохромной видеодорожки в выходные данные в этом сценарии.

> [!NOTE]
> Это увеличивает размер выходного ресурса-контейнера, а, следовательно, и затраты на такую задачу кодирования. Следует выполнить тесты, чтобы убедиться, что результирующее увеличение незначительно влияет на ежемесячные расходы.
>

### <a name="inserting-video-at-only-the-lowest-bitrate"></a>Вставка видео только с минимальной скоростью

Предположим, что вы используете многоскоростную кодировку, например ["H264 Single bitrate несколько скорость 720p"](media-services-mes-preset-h264-multiple-bitrate-720p.md) , чтобы закодировать весь входной каталог для потоковой передачи, который содержит набор видеофайлов и файлов только для звука. В этом случае, если входные данные не содержат видео, может потребоваться указать кодировщику принудительно вставлять монохромную видеодорожку только с самой низкой скоростью, а не с каждой выходной скоростью. Для этого необходимо указать флаг **InsertBlackIfNoVideoBottomLayerOnly**.

Вы можете использовать любую из предустановок MES, которые определены в [этом](media-services-mes-presets-overview.md) разделе, и внести в нее следующие изменения.

#### <a name="json-preset"></a>Предустановка JSON

```json
{
  "KeyFrameInterval": "00:00:02",
  "StretchMode": "AutoSize",
  "Condition": "InsertBlackIfNoVideoBottomLayerOnly",
  "H264Layers": [
    …
  ]
}
```

#### <a name="xml-preset"></a>Предустановка XML

При использовании XML укажите Condition="InsertBlackIfNoVideoBottomLayerOnly" в качестве атрибута элемента **H264Video** и Condition="InsertSilenceIfNoAudio" — в качестве атрибута элемента **AACAudio**.

```xml
. . .
<Encoding>
  <H264Video Condition="InsertBlackIfNoVideoBottomLayerOnly">
    <KeyFrameInterval>00:00:02</KeyFrameInterval>
    <SceneChangeDetection>true</SceneChangeDetection>
    <StretchMode>AutoSize</StretchMode>
    <H264Layers>
      <H264Layer>
        . . .
      </H264Layer>
    </H264Layers>
    <Chapters />
  </H264Video>
  <AACAudio Condition="InsertSilenceIfNoAudio">
    <Profile>AACLC</Profile>
    <Channels>2</Channels>
    <SamplingRate>48000</SamplingRate>
    <Bitrate>128</Bitrate>
  </AACAudio>
</Encoding>
. . .
```

### <a name="inserting-video-at-all-output-bitrates"></a>Вставка видео со всеми выходными скоростями
Предположим, что вы используете предустановку кодирования с несколькими скоростями, например [H264 Multiple Bitrate 720p](media-services-mes-preset-H264-Multiple-Bitrate-720p.md) , чтобы закодировать для потоковой передачи весь входной каталог, содержащий смесь видео- и аудиофайлов. В этом случае, если входные данные не содержат видео, может потребоваться указать кодировщику принудительно вставлять монохромную видеодорожку с каждой из выходных скоростей. Это гарантирует, что выходные ресурсы-контейнеры будут однородны по количеству видео- и аудиодорожек. Чтобы получить такой результат, необходимо указать флаг InsertBlackIfNoVideo.

Вы можете использовать любую из предустановок MES, которые определены в [этом](media-services-mes-presets-overview.md) разделе, и внести в нее следующие изменения.

#### <a name="json-preset"></a>Предустановка JSON

```json
{
  "KeyFrameInterval": "00:00:02",
  "StretchMode": "AutoSize",
  "Condition": "InsertBlackIfNoVideo",
  "H264Layers": [
    …
  ]
}
```

#### <a name="xml-preset"></a>Предустановка XML

При использовании XML укажите Condition="InsertBlackIfNoVideo" в качестве атрибута элемента **H264Video** и Condition="InsertSilenceIfNoAudio" — в качестве атрибута элемента **AACAudio**.

```xml
. . .
<Encoding>
  <H264Video Condition="InsertBlackIfNoVideo">
    <KeyFrameInterval>00:00:02</KeyFrameInterval>
    <SceneChangeDetection>true</SceneChangeDetection>
    <StretchMode>AutoSize</StretchMode>
    <H264Layers>
      <H264Layer>
        . . .
      </H264Layer>
    </H264Layers>
    <Chapters />
  </H264Video>
  <AACAudio Condition="InsertSilenceIfNoAudio">
    <Profile>AACLC</Profile>
    <Channels>2</Channels>
    <SamplingRate>48000</SamplingRate>
    <Bitrate>128</Bitrate>
  </AACAudio>
</Encoding>
. . .  
```

## <a name="rotate-a-video"></a><a id="rotate_video"></a>Поворот видео
[Стандартный кодировщик служб мультимедиа](media-services-dotnet-encode-with-media-encoder-standard.md) поддерживает поворот на 0, 90, 180 и 270 градусов. По умолчанию задается значение Auto, при котором система пытается обнаружить метаданные поворота во входящем видеофайле и обеспечить соответствующую компенсацию. Включите приведенный ниже элемент **Sources** в одну из предустановок, определенных в [этом](media-services-mes-presets-overview.md) разделе.

### <a name="json-preset"></a>Предустановка JSON

```json
  "Sources": [
    {
      "Streams": [],
      "Filters": {
        "Rotation": "90"
      }
    }
  ],
  "Codecs": [

    ...
```

### <a name="xml-preset"></a>Предустановка XML

```xml
<Sources>
  <Source>
    <Streams />
    <Filters>
      <Rotation>90</Rotation>
    </Filters>
  </Source>
</Sources>
```

Сведения о том, как кодировщик интерпретирует значения ширины и высоты в предустановке, когда запускается компенсация поворота, см. [здесь](media-services-mes-schema.md#PreserveResolutionAfterRotation).

Если указать значение 0, кодировщик будет игнорировать метаданные поворота (если они есть) во входящем видеофайле.

## <a name="media-services-learning-paths"></a>Схемы обучения работе со службами мультимедиа
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Отзывы
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>См. также:
[Обзор кодирования с помощью служб мультимедиа](media-services-encode-asset.md)
