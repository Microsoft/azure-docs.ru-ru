---
title: Создание эскизов с помощью Media Encoder Standard .NET
description: В этой статье показано, как использовать .NET для кодирования ресурса и создания эскизов в то же время с помощью Media Encoder Standard.
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
ms.date: 12/01/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 3efb5aa864f0940b7b5aa6a9131da9ce8bd732ee
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101092818"
---
# <a name="how-to-generate-thumbnails-using-encoder-standard-with-net"></a>Создание эскизов с помощью кодировщика Standard в .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Media Encoder Standard можно использовать для создания одного или нескольких эскизов из входного видео в [формате JPEG](https://en.wikipedia.org/wiki/JPEG) или [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) .

## <a name="recommended-reading-and-practice"></a>Рекомендуемое чтение и практическое занятие

Рекомендуется ознакомиться с пользовательскими преобразованиями, прочитав [способы кодирования с помощью пользовательского преобразования — .NET](customize-encoder-presets-how-to.md).

## <a name="transform-code-example"></a>Пример кода преобразования

В приведенном ниже примере кода создается только эскиз.  Необходимо задать следующие параметры:

- **Start** — место во входном видео, с которого начинается создание эскизов. Значение может быть в формате ISO 8601 (например, PT05S для начала 5 секунд) или в количестве кадров (например, 10 для начала 10-го кадра) или относительное значение длительности потока (например, 10%, чтобы начать с 10% длительности потока). Также поддерживает макрос {Best}, который указывает кодировщику выбрать лучший эскиз из первых нескольких секунд видео и создаст только один эскиз, независимо от других параметров для шага и диапазона. Значением по умолчанию является макрос {Best}.
- **шаг** — интервалы, по которым создаются эскизы. Значение может быть в формате ISO 8601 (например, PT05S для одного образа каждые 5 секунд) или в количестве кадров (например, 30 для одного изображения каждые 30 кадров), или относительное значение длительности потока (например, 10% для одного изображения, каждые 10% от длительности потока). Значение шага повлияет на первый созданный эскиз, который может не совпадать с указанным во время начала преобразования предустановки. Это происходит из-за кодировщика, который пытается выбрать оптимальный эскиз между временем начала и позицией шага от времени начала до первого выхода. Как значение по умолчанию — 10%, это означает, что если поток имеет большую длительность, первый созданный эскиз может находиться далеко от указанного во время начала. Попробуйте выбрать разумное значение для шага, если ожидается, что первый эскиз является близким к времени начала, или задайте для параметра Range значение 1, если в качестве времени начала требуется только один эскиз.
- **Range** — позиция относительно предустановки преобразования. время начала во входном видео, с которого следует прерывать создание эскизов. Значение может быть в формате ISO 8601 (например, PT5M30S, чтобы останавливаться в течение 5 минут и 30 секунд с момента начала), или на число кадров (например, 300 для завершения в кадре 1/300 из кадра во время начала. Если это значение равно 1, это означает создание только одного эскиза во время начала, или относительное значение длительности потока (например, 50%, чтобы останавливаться на половину длительности потока от времени начала). Значение по умолчанию — 100%, что означает завершение работы в конце потока.
- **слои** — Коллекция выходных слоев изображений, создаваемых кодировщиком.

```csharp

private static Transform EnsureTransformExists(IAzureMediaServicesClient client, string resourceGroupName, string accountName, string transformName)
{
    // Does a Transform already exist with the desired name? Assume that an existing Transform with the desired name
    // also uses the same recipe or Preset for processing content.
    Transform transform = client.Transforms.Get(resourceGroupName, accountName, transformName);

    if (transform == null)
    {
        // Create a new Transform Outputs array - this defines the set of outputs for the Transform
        TransformOutput[] outputs = new TransformOutput[]
        {
            // Create a new TransformOutput with a custom Standard Encoder Preset
            // This demonstrates how to create custom codec and layer output settings

          new TransformOutput(
                new StandardEncoderPreset(
                    codecs: new Codec[]
                    {
                        // Generate a set of PNG thumbnails
                        new PngImage(
                            start: "25%",
                            step: "25%",
                            range: "80%",
                            layers: new PngLayer[]{
                                new PngLayer(
                                    width: "50%",
                                    height: "50%"
                                )
                            }
                        )
                    },
                    // Specify the format for the output files for the thumbnails
                    formats: new Format[]
                    {
                        new PngFormat(
                            filenamePattern:"Thumbnail-{Basename}-{Index}{Extension}"
                        )
                    }
                ),
                onError: OnErrorType.StopProcessingJob,
                relativePriority: Priority.Normal
            )
        };

        string description = "A transform that includes thumbnails.";
        // Create the custom Transform with the outputs defined above
        transform = client.Transforms.CreateOrUpdate(resourceGroupName, accountName, transformName, outputs, description);
    }

    return transform;
}
```

## <a name="next-steps"></a>Дальнейшие действия

[Создание эскизов с помощью функции "ОСТАВШАЯся"](media-services-generate-thumbnails-rest.md)
