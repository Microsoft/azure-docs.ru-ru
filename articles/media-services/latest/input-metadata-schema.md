---
title: Схема входных метаданных служб мультимедиа Azure v3
description: В этой статье приводятся общие сведения о схеме входных метаданных служб мультимедиа Azure версии 3.
author: IngridAtMicrosoft
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 0acb882410d103cf6f6c34bbecf2006094437b04
ms.sourcegitcommit: b572ce40f979ebfb75e1039b95cea7fce1a83452
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "102634691"
---
# <a name="input-metadata"></a>Входные метаданные

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Задание кодирования связано с входным ресурсом-контейнером (или ресурсами-контейнерами), в котором нужно выполнить эти задачи.  После выполнения задачи создается выходной ресурс-контейнер. Выходной ресурс содержит видео, аудио, эскизы, манифест и другие файлы. 

Выходной актив также содержит файл с метаданными входного актива. Имя JSON-файла метаданных имеет случайный идентификатор, не используйте его для идентификации входного ресурса, к которому принадлежит выходной ресурс. Чтобы определить входной ресурс, к которому он принадлежит, используйте `Uri` поле (Дополнительные сведения см. в разделе [другие дочерние элементы](#other-child-elements)).  

Службы мультимедиа не проверяют входные ресурсы с вытеснением для создания метаданных. Входные метаданные создаются только в качестве артефакта при обработке входного ресурса в задании. Поэтому этот артефакт записывается в выходной ресурс. Для создания метаданных для входных ресурсов и выходных ресурсов используются различные средства. Таким образом, схемы входных и выходных метаданных немного отличаются.

В этой статье обсуждаются элементы и типы схемы JSON, на которых основан входной метаданных ( &lt; asset_id &gt;_metadata.json). Дополнительные сведения о файле, содержащем метаданные о выходном ресурсе, см. в разделе [выходные метаданные](output-metadata-schema.md).  

Пример схемы JSON можно найти в конце этой статьи.  

## <a name="assetfile"></a>AssetFile  

Содержит коллекцию элементов AssetFile для задания кодирования.  

> [!NOTE]
> Следующие четыре дочерних элемента должны появляться последовательно.

| Имя  | Описание |
| --- | --- | 
| **VideoTracks**|Каждый физический файл ресурса может содержать ноль или более видеодорожек, чередуемых в соответствующем формате ресурса. Дополнительные сведения см. в разделе [VideoTracks](#videotracks). |
| **AudioTracks**|Каждый физический файл ресурса-контейнера может содержать ноль или более звуковых дорожек, чередуемых в соответствующем формате ресурса-контейнера. Дополнительные сведения см. в разделе [AudioTracks](#audiotracks) |
| **Метаданные**  |Метаданные файла ресурса-контейнера, представленные в виде строки "ключ —значение". <br />Например: `<Metadata key="language" value="eng" />` |

### <a name="other-child-elements"></a>Другие дочерние элементы

| Имя | Описание |
| --- | --- |
| **имя**;<br />Обязательно |Имя файла ресурса-контейнера. <br /><br />Например, `"Name": "Ignite-short.mp4"`. |
| **URI**<br />Обязательно |URL-адрес для расположения входного ресурса. Для идентификации входного ресурса, к которому принадлежит выходной ресурс, используйте `Uri` поле вместо идентификатора.|
| **Размер**<br />Обязательно |Размер файла ресурса-контейнера в байтах.  <br /><br />Например, `"Size": 75739259`.|
| **Длительность**<br />Обязательно |Длительность воспроизведения содержимого. <br /><br />Например, `"Duration": "PT1M10.304S"`. |
| **NumberOfStreams**<br />Обязательно |Количество потоков в файле ресурса-контейнера.  <br /><br />Например, `"NumberOfStreams": 2`.|
| **FormatNames**<br />Обязательно |Имена форматов.  <br /><br />Например, `"FormatNames": "mov,mp4,m4a,3gp,3g2,mj2"`.|
| **форматвербосенаме**<br /> Обязательно |Подробные имена форматов. <br /><br />Например, `"FormatVerboseName": "QuickTime / MOV"`. |
| **StartTime** |Время начала содержимого.  <br /><br />Например, `"StartTime": "PT0S"`. |
| **OverallBitRate** |Средняя скорость файла ресурса в битах в секунду.  <br /><br />Например, `"OverallBitRate": 8618539`.|

## <a name="videotracks"></a>VideoTracks

| Имя | Описание |
| --- | --- |
| **FourCC**<br />Обязательно |Код видеокодека FourCC, сообщаемый FFmpeg.<br /><br />Например, `"FourCC": "avc1" | "hev1" | "hvc1"`. |
| **Профиль** |Профиль видеодорожки. <br /><br />Например, `"Profile": "Main"`.|
| **Уровень** |Уровень видеодорожки. <br /><br />Например, `"Level": "3.2"`.|
| **PixelFormat** |Формат пикселей видеодорожки. <br /><br />Например, `"PixelFormat": "yuv420p"`.|
| **Width**<br />Обязательно |Ширина закодированного видео в пикселях. <br /><br />Например, `"Width": "1280"`.|
| **Height**<br />Обязательно |Высота закодированного видео в пикселях.<br /><br />Например, `"Height": "720"`. |
| **DisplayAspectRatioNumerator**<br />Обязательно |Числитель пропорции отображения видео.<br /><br />Например, `"DisplayAspectRatioNumerator": 16.0`. |
| **DisplayAspectRatioDenominator**<br />Обязательно |Знаменатель пропорции отображения видео. <br /><br />Например, `"DisplayAspectRatioDenominator": 9.0`.|
| **SampleAspectRatioNumerator** |Числитель пропорции выборки видео. <br /><br />Например, `"SampleAspectRatioNumerator": 1.0`.|
| **самплеаспектратиоденоминатор**|Например, `"SampleAspectRatioDenominator": 1.0`.|
| **Частота**<br />Обязательно |Измеренная частота видеокадров в формате 3F. <br /><br />Например, `"FrameRate": 29.970`.|
| **Bitrate** |Средняя скорость видео в битах в секунду, вычисленная на основе файла ресурса. Подсчитываются только полезные данные элементарного потока, издержки на упаковку не включаются. <br /><br />Например, `"Bitrate": 8421583`.|
| **HasBFrames** |Количество видеодорожек с кадрами B. <br /><br />Например, `"HasBFrames": 2`.|
| **Метаданные** |Общие строки "ключ —значение", которые можно использовать для хранения различных сведений. <br />Полный пример см. в конце статьи. |
| **Id**<br />Обязательно |Отсчитываемый от нуля индекс аудио- или видеодорожки.<br /><br /> Этот **идентификатор** не обязательно представляет собой TrackID, используемый в файле MP4. <br /><br />Например, `"Id": 2`.|
| **Равн** |Строка кодека видеодорожки. <br /><br />Например, `"Codec": "h264 | hev1"`.|
| **CodecLongName** |Полное имя кодека звуковой или видеодорожки. <br /><br />Например, `"CodecLongName": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10"`.|
| **Равн** |Строка кодека видеодорожки. <br /><br />Например, `"Codec": "h264 | hev1"`.|
| **TimeBase**<br />Обязательно |Базовый счетчик времени.<br /><br />Например, `"TimeBase": "1/30000"`.|
| **NumberOfFrames** |Количество кадров (для видеодорожек). <br /><br />Например, `"NumberOfFrames": 2107`.|
| **StartTime** |Время начала дорожки.<br /><br />Например, `"StartTime": "PT0.033S"`. |
| **Длительность** |Продолжительность дорожки. <br /><br />Например, `"Duration": "PT1M10.304S"`.|

## <a name="audiotracks"></a>AudioTracks

| Имя  | Описание |
| --- | --- | 
| **SampleFormat** |Формат выборки. <br /><br />Например, `"SampleFormat": "fltp"`.|
| **ChannelLayout** |Структура канала. <br /><br />Например, `"ChannelLayout": "stereo"`.|
| **Каналы**<br />Обязательно |Количество (0 или более) аудиоканалов. <br /><br />Например, `"Channels": 2`.|
| **SamplingRate**<br />Обязательно |Частота аудиовыборки: выборок/с или Гц. <br /><br />Например, `"SamplingRate": 48000`.|
| **Bitrate** |Средняя скорость аудиопотока в битах в секунду, вычисленная на основе файла ресурса-контейнера. Подсчитываются только полезные данные элементарного потока, издержки на упаковку не включаются. <br /><br />Например, `"Bitrate": 192080`.|
| **Метаданные** |Общие строки "ключ —значение", которые можно использовать для хранения различных сведений.  <br />Полный пример см. в конце статьи. |
| **Id**<br />Обязательно |Отсчитываемый от нуля индекс аудио- или видеодорожки.<br /><br /> MP4-файл не обязательно должен содержать идентификатор дорожки. <br /><br />Например, `"Id": 1`.|
| **Равн** |Строка кодека видеодорожки. <br /><br />Например, `"Codec": "aac"`.|
| **CodecLongName** |Полное имя кодека звуковой или видеодорожки. <br /><br />Например, `"CodecLongName": "AAC (Advanced Audio Coding)"`.|
| **TimeBase**<br />Обязательно |Базовый счетчик времени.<br /><br />Например, `"TimeBase": "1/48000"`. |
| **NumberOfFrames** |Количество кадров (для видеодорожек). <br /><br />Например, `"NumberOfFrames": 3294`.|
| **StartTime** |Время начала дорожки. Дополнительные сведения см. в разделе [ISO8601](https://www.iso.org/iso-8601-date-and-time-format.html). <br /><br />Например, `"StartTime": "PT0S"`. |
| **Длительность** |Продолжительность дорожки. <br /><br />Например, `"Duration": "PT1M10.272S"`. |

## <a name="metadata"></a>Метаданные

| Имя | Описание |
| --- | --- |
| **key**<br />Обязательно |Ключ из пары ключ/значение. |
| **value**<br /> Обязательно |Значение из пары ключ/значение. |

## <a name="schema-example"></a>Пример схемы

```json
{
  "AssetFile": [
    {
      "VideoTracks": [
        {
          "FourCC": "avc1",
          "Profile": "Main",
          "Level": "3.2",
          "PixelFormat": "yuv420p",
          "Width": "1280",
          "Height": "720",
          "DisplayAspectRatioNumerator": 16.0,
          "DisplayAspectRatioDenominator": 9.0,
          "SampleAspectRatioNumerator": 1.0,
          "SampleAspectRatioNumeratorSpecified": true,
          "SampleAspectRatioDenominator": 1.0,
          "SampleAspectRatioDenominatorSpecified": true,
          "FrameRate": 29.970,
          "Bitrate": 8421583,
          "BitrateSpecified": true,
          "HasBFrames": 2,
          "HasBFramesSpecified": true,
          "Disposition": {
            "Default": 1
          },
          "Metadata": [
            {
              "key": "creation_time",
              "value": "2018-02-21T21:42:08.000000Z"
            },
            {
              "key": "language",
              "value": "eng"
            },
            {
              "key": "handler_name",
              "value": "Video Media Handler"
            },
            {
              "key": "encoder",
              "value": "AVC Coding"
            }
          ],
          "Id": 2,
          "Codec": "h264",
          "CodecLongName": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10",
          "TimeBase": "1/30000",
          "NumberOfFrames": 2107,
          "NumberOfFramesSpecified": true,
          "StartTime": "PT0.033S",
          "Duration": "PT1M10.304S"
        }
      ],
      "AudioTracks": [
        {
          "SampleFormat": "fltp",
          "ChannelLayout": "stereo",
          "Channels": 2,
          "SamplingRate": 48000,
          "Bitrate": 192080,
          "BitrateSpecified": true,
          "BitsPerSampleSpecified": true,
          "Disposition": {
            "Default": 1
          },
          "Metadata": [
            {
              "key": "creation_time",
              "value": "2018-02-21T21:42:08.000000Z"
            },
            {
              "key": "language",
              "value": "eng"
            },
            {
              "key": "handler_name",
              "value": "Sound Media Handler"
            }
          ],
          "Id": 1,
          "Codec": "aac",
          "CodecLongName": "AAC (Advanced Audio Coding)",
          "TimeBase": "1/48000",
          "NumberOfFrames": 3294,
          "NumberOfFramesSpecified": true,
          "StartTime": "PT0S",
          "Duration": "PT1M10.272S"
        }
      ],
      "Metadata": [
        {
          "key": "major_brand",
          "value": "mp42"
        },
        {
          "key": "minor_version",
          "value": "19529854"
        },
        {
          "key": "compatible_brands",
          "value": "mp42isom"
        },
        {
          "key": "creation_time",
          "value": "2018-02-21T21:42:08.000000Z"
        }
      ],
      "Name": "Ignite-short.mp4",
      "Uri": "https://amsstorageacct.blob.core.windows.net/asset-00000000-0000-0000-000000000000/ignite.mp4",
      "Size": 75739259,
      "Duration": "PT1M10.304S",
      "NumberOfStreams": 2,
      "FormatNames": "mov,mp4,m4a,3gp,3g2,mj2",
      "FormatVerboseName": "QuickTime / MOV",
      "StartTime": "PT0S",
      "OverallBitRate": 8618539,
      "OverallBitRateSpecified": true
    }
  ]
}
```

## <a name="next-steps"></a>Дальнейшие действия

[Выходные метаданные](output-metadata-schema.md)
