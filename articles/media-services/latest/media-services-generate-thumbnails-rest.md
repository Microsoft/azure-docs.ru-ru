---
title: Создание эскизов с помощью Media Encoder Standard RESTFUL
description: В этой статье показано, как использовать функции RESTFUL для кодирования ресурса и создания эскизов одновременно с помощью Media Encoder Standard.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: f4456a65f422c23da73f36dd74680fbb598db186
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98955840"
---
# <a name="how-to-generate-thumbnails-using-encoder-standard-with-rest"></a>Создание эскизов с помощью кодировщика Standard

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Media Encoder Standard можно использовать для создания одного или нескольких эскизов из входящего видео в форматах файлов изображений [JPEG](https://en.wikipedia.org/wiki/JPEG), [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) или [BMP](https://en.wikipedia.org/wiki/BMP_file_format).

## <a name="recommended-reading-and-practice"></a>Рекомендуемое чтение и практическое занятие

Рекомендуется ознакомиться с пользовательскими преобразованиями, прочитав [способы кодирования с помощью пользовательского преобразования-остальное](custom-preset-rest-howto.md).

## <a name="thumbnail-parameters"></a>Параметры эскиза

Необходимо задать следующие параметры:

- **Start** — место во входном видео, с которого начинается создание эскизов. Значение может быть в формате ISO 8601 (например, PT05S для начала 5 секунд) или в количестве кадров (например, 10 для начала 10-го кадра) или относительное значение длительности потока (например, 10%, чтобы начать с 10% длительности потока). Также поддерживает макрос {Best}, который указывает кодировщику выбрать лучший эскиз из первых нескольких секунд видео и создаст только один эскиз, независимо от других параметров для шага и диапазона. Значением по умолчанию является макрос {Best}.
- **шаг** — интервалы, по которым создаются эскизы. Значение может быть в формате ISO 8601 (например, PT05S для одного образа каждые 5 секунд) или в количестве кадров (например, 30 для одного изображения каждые 30 кадров), или относительное значение длительности потока (например, 10% для одного изображения, каждые 10% от длительности потока). Значение шага повлияет на первый созданный эскиз, который может не совпадать с указанным во время начала преобразования предустановки. Это происходит из-за кодировщика, который пытается выбрать оптимальный эскиз между временем начала и позицией шага от времени начала до первого выхода. Как значение по умолчанию — 10%, это означает, что если поток имеет большую длительность, первый созданный эскиз может находиться далеко от указанного во время начала. Попробуйте выбрать разумное значение для шага, если ожидается, что первый эскиз является близким к времени начала, или задайте для параметра Range значение 1, если в качестве времени начала требуется только один эскиз.
- **Range** — позиция относительно предустановки преобразования. время начала во входном видео, с которого следует прерывать создание эскизов. Значение может быть в формате ISO 8601 (например, PT5M30S, чтобы останавливаться в течение 5 минут и 30 секунд с момента начала), или на число кадров (например, 300 для завершения в кадре 1/300 из кадра во время начала. Если это значение равно 1, это означает создание только одного эскиза во время начала, или относительное значение длительности потока (например, 50%, чтобы останавливаться на половину длительности потока от времени начала). Значение по умолчанию — 100%, что означает завершение работы в конце потока.
- **слои** — Коллекция выходных слоев изображений, создаваемых кодировщиком.

## <a name="example-of-a-single-png-file-preset"></a>Пример предустановки "один файл PNG"

Приведенную ниже предустановку JSON можно использовать для создания одного выходного PNG-файла из первых нескольких секунд входного видео, где кодировщик выполняет наиболее эффективную попытку поиска "интересного" кадра. Обратите внимание, что для размера выходных изображений выбрано значение 100 %. То есть они совпадают с размером входного видео. При этом параметр Format в разделе Outputs должен соответствовать использованию PngLayers в разделе Codecs.  

```json
{
    "properties": {
        "description": "Basic Transform using a custom encoding preset for thumbnails",
        "outputs": [
            {
                "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                "preset": {
                    "@odata.type": "#Microsoft.Media.StandardEncoderPreset",
                    "codecs": [
                        {
                            "@odata.type": "#Microsoft.Media.PngImage",
                            "stretchMode": "AutoSize",
                            "start": "{Best}",
                            "step": "25%",
                            "range": "80%",
                            "layers": [
                                {
                                    "width": "50%",
                                    "height": "50%"
                                }
                            ]
                        }
                    ],
                    "formats": [
                        {
                            "@odata.type": "#Microsoft.Media.Mp4Format",
                            "filenamePattern": "Video-{Basename}-{Label}-{Bitrate}{Extension}",
                            "outputFiles": []
                        },
                        {
                            "@odata.type": "#Microsoft.Media.PngFormat",
                            "filenamePattern": "Thumbnail-{Basename}-{Index}{Extension}"
                        }
                    ]
                }
            }
        ]
    }
}

```

## <a name="example-of-a-series-of-jpeg-images-preset"></a>Пример предустановки "несколько изображений JPEG"

Приведенную ниже предустановку JSON можно использовать для создания набора из 10 изображений по отметкам времени 5%, 15%,..., 95% входной временной шкалы, где размер изображения определяется как один квартал входного видео.

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

## <a name="example-of-a-one-image-at-a-specific-timestamp-preset"></a>Пример предустановки "одно изображение на определенной метке времени"

Приведенную ниже предустановку JSON можно использовать для создания одного изображения JPEG по 30-секундной отметке входного видео. При использовании этой предустановки предполагается, что длительность входного видео более 30 секунд (иначе задание завершается ошибкой).

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

## <a name="example-of-a-thumbnails-at-different-resolutions-preset"></a>Пример предустановки "эскизы с разными разрешениями"

Эту предустановку можно использовать для создания эскизов с разными разрешениями в одной задаче. В примере в позициях входной временной шкалы 5 %, 15 %, …, 95 % кодировщик создает два изображения — один с использованием 100 % входного разрешения видео, а другой с использованием 50 %.

Обратите внимание на использование макроса {Resolution} в FileName. Он указывает кодировщику использовать ширину и высоту, заданные в разделе Encoding предустановки при создании имени файла выходных изображений. Это также помогает легко различать разные образы.

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

## <a name="example-of-generating-a-thumbnail-while-encoding"></a>Пример создания эскиза во время кодирования

Во всех примерах выше рассматривались способы отправки задачи кодирования для создания изображений. Но также можно объединить кодирование видео и аудио с созданием эскизов. Следующая Предустановка JSON сообщает стандарту кодировщика создать эскиз во время кодирования.

### <a name="json-preset"></a>Предустановка JSON

Сведения о схеме см. [здесь](../previous/media-services-mes-schema.md).

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

## <a name="next-steps"></a>Дальнейшие действия
[Создание эскизов с помощью .NET](media-services-generate-thumbnails-dotnet.md)