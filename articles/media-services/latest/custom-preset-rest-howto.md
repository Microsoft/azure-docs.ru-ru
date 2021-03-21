---
title: Кодирование пользовательских преобразований
description: В этом разделе показано, как использовать службы мультимедиа Azure v3 для кодирования пользовательского преобразования с помощью функции RESTFUL.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.custom: ''
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 2c0416e2787bc69465e18a71f6b9912f7571c24d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98956095"
---
# <a name="how-to-encode-with-a-custom-transform---rest"></a>Как кодировать с пользовательским преобразованием — остальное

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

При кодировании со службами мультимедиа Azure можно быстро приступить к работе с помощью одной из рекомендуемых встроенных предустановок на основе отраслевых рекомендаций, как показано в руководстве по [потоковой передаче файлов](stream-files-tutorial-with-rest.md#create-a-transform) . Вы также можете создать пользовательскую предустановку, предназначенную для конкретных сценариев или требований к устройствам.

## <a name="considerations"></a>Рекомендации

При создании пользовательских предустановок применяются следующие соображения.

* Все значения высоты и ширины в содержимом AVC должны быть кратными 4.
* В службах мультимедиа Azure v3 все скорости кодирования находятся в битах в секунду. Это отличается от предустановок с помощью API v2, которые использовались в килобитах в секунду в качестве единицы измерения. Например, если скорость в v2 была указана как 128 (килобит в секунду), то в v3 будет установлено значение 128000 (бит/сек).

## <a name="prerequisites"></a>Предварительные требования 

- [Создание учетной записи Служб мультимедиа](./create-account-howto.md). <br/>Обязательно запомните имя группы ресурсов и имя учетной записи Служб мультимедиа. 
- [Настройте Postman для вызовов REST API Служб мультимедиа Azure](media-rest-apis-with-postman.md).<br/>Не забудьте выполнить последний шаг, указанный в разделе [Получение токена Azure AD](media-rest-apis-with-postman.md#get-azure-ad-token). 

## <a name="define-a-custom-preset"></a>Определение пользовательской предустановки

В следующем примере определяется текст запроса нового преобразования. Мы определяем набор выходов, которые нужно создать при использовании этого преобразования. 

В этом примере мы сначала добавляем слой AacAudio для кодирования звука и два слоя H264Video для кодирования видео. В видеослоях мы присваиваем метки, чтобы их можно было использовать в именах выходных файлов. Далее мы хотим, чтобы выходные данные также включали эскизы. В приведенном ниже примере мы указываем изображения в формате PNG, созданные на 50% от разрешения входного видео и три метки времени — {25%, 50%, 75} длиной входного видео. Наконец, мы указываем формат выходных файлов — один для видео + аудио, а другой — для эскизов. Так как у нас есть несколько H264Layers, необходимо использовать макросы, которые создают уникальные имена для каждого слоя. Можно использовать `{Label}` `{Bitrate}` макрос или, а в примере — первый.

```json
{
    "properties": {
        "description": "Basic Transform using a custom encoding preset",
        "outputs": [
            {
                "onError": "StopProcessingJob",
                "relativePriority": "Normal",
                "preset": {
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
            }
        ]
    }
}

```

## <a name="create-a-new-transform"></a>Создание нового преобразования  

В этом примере мы создадим **Преобразование** на основе пользовательской предустановки, определенной ранее. При создании преобразования сначала следует использовать [Get](/rest/api/media/transforms/get) , чтобы проверить, существует ли он. Если преобразование существует, используйте его повторно. 

В скачанной коллекции POST выберите преобразования **и задания** -> **создать или обновить преобразование**.

Метод HTTP-запроса **PUT** аналогичен следующему:

```
PUT https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/transforms/:transformName?api-version={{api-version}}
```

Перейдите на вкладку **текст** и замените текст кодом JSON, который вы [определили ранее](#define-a-custom-preset). Чтобы службы мультимедиа применяли преобразование к указанному видео или аудио, необходимо отправить задание с этим преобразованием.

Нажмите кнопку **Отправить**. 

Чтобы службы мультимедиа применяли преобразование к указанному видео или аудио, необходимо отправить задание с этим преобразованием. Полный пример, демонстрирующий, как отправить задание под преобразованием, см. в разделе [учебник. потоковые видео-файлы — остальное](stream-files-tutorial-with-rest.md).

## <a name="next-steps"></a>Дальнейшие действия

Просмотреть [другие операции RESTful](/rest/api/media/)
