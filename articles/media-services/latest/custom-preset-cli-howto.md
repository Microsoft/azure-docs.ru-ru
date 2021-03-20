---
title: Кодирование интерфейса командной строки настраиваемого преобразования
description: В этом разделе показано, как использовать службы мультимедиа Azure v3 для кодирования пользовательского преобразования с помощью Azure CLI.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.custom: devx-track-azurecli
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: a91123b22eaef4cbaf2e96c93200777c8a85abab
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101092393"
---
# <a name="how-to-encode-with-a-custom-transform---azure-cli"></a>Как кодировать с помощью пользовательского преобразования Azure CLI

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

При кодировании со службами мультимедиа Azure можно быстро приступить к работе с помощью одной из рекомендуемых встроенных предустановок на основе отраслевых рекомендаций, как показано в кратком руководстве по [файлам потоковой передачи](stream-files-cli-quickstart.md#create-a-transform-for-adaptive-bitrate-encoding) . Вы также можете создать пользовательскую предустановку, предназначенную для конкретных сценариев или требований к устройствам.

## <a name="considerations"></a>Рекомендации

При создании пользовательских предустановок применяются следующие соображения.

* Все значения высоты и ширины в содержимом AVC должны быть кратными 4.
* В службах мультимедиа Azure v3 все скорости кодирования находятся в битах в секунду. Это отличается от предустановок с помощью API v2, которые использовались в килобитах в секунду в качестве единицы измерения. Например, если скорость в v2 была указана как 128 (килобит в секунду), то в v3 будет установлено значение 128000 (бит/сек).

## <a name="prerequisites"></a>Предварительные требования

[Создание учетной записи Служб мультимедиа](./create-account-howto.md).

Обязательно запомните имя группы ресурсов и имя учетной записи Служб мультимедиа.

## <a name="define-a-custom-preset"></a>Определение пользовательской предустановки

В следующем примере определяется текст запроса нового преобразования. Мы определяем набор выходов, которые нужно создать при использовании этого преобразования.

В этом примере мы сначала добавляем слой AacAudio для кодирования звука и два слоя H264Video для кодирования видео. В видеослоях мы присваиваем метки, чтобы их можно было использовать в именах выходных файлов. Далее мы хотим, чтобы выходные данные также включали эскизы. В приведенном ниже примере мы указываем изображения в формате PNG, созданные на 50% от разрешения входного видео и три метки времени — {25%, 50%, 75} длиной входного видео. Наконец, мы указываем формат выходных файлов — один для видео + аудио, а другой — для эскизов. Так как у нас есть несколько H264Layers, необходимо использовать макросы, которые создают уникальные имена для каждого слоя. Можно использовать `{Label}` `{Bitrate}` макрос или, а в примере — первый.

Мы будем сохранять это преобразование в файле. В этом примере мы создаем имя файла `customPreset.json` .

```json
{
    "@odata.type": "#Microsoft.Media.StandardEncoderPreset",
    "codecs": [
        {
            "@odata.type": "#Microsoft.Media.AacAudio",
            "channels": 2,
            "samplingRate": 48000,
            "bitrate": 128000,
            "profile": "AacLc"
        },
        {
            "@odata.type": "#Microsoft.Media.H264Video",
            "keyFrameInterval": "PT2S",
            "stretchMode": "AutoSize",
            "sceneChangeDetection": false,
            "complexity": "Balanced",
            "layers": [
                {
                    "width": "1280",
                    "height": "720",
                    "label": "HD",
                    "bitrate": 3400000,
                    "maxBitrate": 3400000,
                    "bFrames": 3,
                    "slices": 0,
                    "adaptiveBFrame": true,
                    "profile": "Auto",
                    "level": "auto",
                    "bufferWindow": "PT5S",
                    "referenceFrames": 3,
                    "entropyMode": "Cabac"
                },
                {
                    "width": "640",
                    "height": "360",
                    "label": "SD",
                    "bitrate": 1000000,
                    "maxBitrate": 1000000,
                    "bFrames": 3,
                    "slices": 0,
                    "adaptiveBFrame": true,
                    "profile": "Auto",
                    "level": "auto",
                    "bufferWindow": "PT5S",
                    "referenceFrames": 3,
                    "entropyMode": "Cabac"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Media.PngImage",
            "stretchMode": "AutoSize",
            "start": "25%",
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
```

## <a name="create-a-new-transform"></a>Создание нового преобразования  

В этом примере мы создадим **Преобразование** на основе пользовательской предустановки, определенной ранее. При создании преобразования необходимо сначала проверить, существует ли он. Если преобразование существует, используйте его повторно. Следующая `show` команда возвращает преобразование, `customTransformName` если оно существует:

```azurecli-interactive
az ams transform show -a amsaccount -g amsResourceGroup -n customTransformName
```

Следующая команда Azure CLI создает преобразование на основе пользовательской предустановки (определенную ранее).

```azurecli-interactive
az ams transform create -a amsaccount -g amsResourceGroup -n customTransformName --description "Basic Transform using a custom encoding preset" --preset customPreset.json
```

Чтобы службы мультимедиа применяли преобразование к указанному видео или аудио, необходимо отправить задание с этим преобразованием. Полный пример, демонстрирующий, как отправить задание под преобразованием, см. в разделе [Краткое руководство. Streaming Video Files-Azure CLI](stream-files-cli-quickstart.md).

## <a name="see-also"></a>См. также раздел

[Azure CLI](/cli/azure/ams)
