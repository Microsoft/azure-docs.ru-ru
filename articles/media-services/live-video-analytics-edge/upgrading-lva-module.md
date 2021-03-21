---
title: Обновление службы Live Video Analytics на IoT Edge с 1,0 по 2,0
description: В этой статье рассматриваются различия и различные моменты, которые следует учитывать при обновлении функции Live Video Analytics (лва) в модуле Azure IoT Edge.
author: naiteeks
ms.topic: how-to
ms.author: naiteeks
ms.date: 12/14/2020
ms.openlocfilehash: 49c17946203bc6c3655b1aaf7b04a1ee3ea67388
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98955655"
---
# <a name="upgrading-live-video-analytics-on-iot-edge-from-10-to-20"></a>Обновление службы Live Video Analytics на IoT Edge с 1,0 по 2,0

В этой статье рассматриваются различия и различные моменты, которые следует учитывать при обновлении функции Live Video Analytics (лва) в модуле Azure IoT Edge.

## <a name="change-list"></a>Список изменений

> [!div class="mx-tdCol4BreakAll"]
> |Заголовок|Live Video Analytics 1,0|Live Video Analytics 2,0|Описание|
> |-------------|----------|---------|---------|
> |Образ контейнера|mcr.microsoft.com/media/live-video-analytics:1|mcr.microsoft.com/media/live-video-analytics:2|Опубликованные образы DOCKER для Live Video Analytics на Azure IoT Edge|
> |**Узлы Медиаграф** |    |   |   |
> |Источники|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Источник RTSP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Источник сообщения центра Интернета вещей |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Источник RTSP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Источник сообщения центра Интернета вещей | Узлы Медиаграф, выступающие в качестве источников для приема мультимедиа и сообщений.|
> |Процессоры|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Процессор обнаружения движения </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Обработчик фильтра частоты кадров </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Обработчик расширений HTTP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Обработчик расширения GRPC </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Обработчик сигнальных шлюзов |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Процессор обнаружения движения </br>:::image type="icon" source="./././media/upgrading-lva/remove.png":::**Обработчик фильтра частоты кадров**</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Обработчик расширений HTTP </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Обработчик расширения GRPC </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Обработчик сигнальных шлюзов | Узлы Медиаграф, которые позволяют форматировать носитель перед отправкой на серверы вывода AI.|
> |Приемники|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Приемник актива </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Приемник файла </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Приемник сообщений центра Интернета вещей|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Приемник актива </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Приемник файла </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Приемник сообщений центра Интернета вещей| Узлы Медиаграф, которые позволяют хранить обработанные файлы мультимедиа.|
> |**Поддерживаемые Медиаграфс** |    |   |   |
> |Топологии|:::image type="icon" source="./././media/upgrading-lva/check.png"::: Непрерывная запись видео </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Анализ движения и непрерывная запись видео </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Внешний анализ и непрерывная запись видео </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Запись на основе событий при перемещении </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Запись на основе событий в AI</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Запись на основе событий для внешнего события </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Анализ движения </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Анализ движения с последующим использованием искусственного интеллекта |:::image type="icon" source="./././media/upgrading-lva/check.png"::: Непрерывная запись видео </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Анализ движения и непрерывная запись видео </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Внешний анализ и непрерывная запись видео </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Запись на основе событий при перемещении </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Запись на основе событий в AI</br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Запись на основе событий для внешнего события </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Анализ движения </br>:::image type="icon" source="./././media/upgrading-lva/check.png"::: Анализ движения с последующим использованием искусственного интеллекта </br>:::image type="icon" source="./././media/upgrading-lva/check.png":::**Композиция AI** </br>:::image type="icon" source="./././media/upgrading-lva/check.png":::**Запись аудио и видео** </br>  | Топологии Медиаграф, позволяющие определить схему графа с параметрами в качестве заполнителей для значений.|

## <a name="upgrading-the-live-video-analytics-on-iot-edge-module-from-10-to-20"></a>Обновление интерактивной аналитики видео в модуле IoT Edge с 1,0 до 2,0
При обновлении функции Live Video Analytics в модуле IoT Edge необходимо обновить следующие сведения.
### <a name="container-image"></a>Образ контейнера
В шаблоне развертывания найдите модуль Live Video Analytics в модуле IoT Edge в `modules` узле и обновите `image` поле следующим образом:
```
"image": "mcr.microsoft.com/media/live-video-analytics:2"
```
> [!TIP]
> Если вы еще не изменили имя функции Live Video Analytics в модуле IoT Edge, найдите `lvaEdge` его в узле Module.

### <a name="topology-file-changes"></a>Изменения в файле топологии
Убедитесь, что в файлах топологии **`apiVersion`** установлено значение 2,0.

### <a name="mediagraph-node-changes"></a>Изменения узла Медиаграф


* Теперь звук из источника камеры можно передать вместе с видео. С помощью любого узла графа можно передавать данные **только из аудио** или **видео** либо **аудио и видео** **`outputSelectors`** . Например, если вы хотите выбрать видео только из источника RTSP и передать его нисходящим, добавьте следующий код в узел источника RTSP:
    ```
    "outputSelectors": [
      {
        "property": "mediaType",
        "operator": "is",
        "value": "video"
      }
    ```
>[!NOTE]
>**`outputSelectors`** Является необязательным свойством. Если этот параметр не используется, граф мультимедиа передаст звук (если он включен) и видео с подчиненной камеры RTSP. 

* В `MediaGraphHttpExtension` и `MediaGraphGrpcExtension` процессорах Обратите внимание на следующие изменения:  
    #### <a name="image-properties"></a>Свойства образа
    * `MediaGraphImageFormatEncoded` более не поддерживаются. 
      * Вместо этого используйте **`MediaGraphImageFormatBmp`** или **`MediaGraphImageFormatJpeg`** или **`MediaGraphImageFormatPng`** . Например,
        ```
        "image": {
                "scale": 
                {
                    "mode": "preserveAspectRatio",
                    "width": "416",
                    "height": "416"
                },
                "format": 
                {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatJpeg"
                }
            }
        ```
        * Если вы хотите использовать необработанные образы, используйте **`MediaGraphImageFormatRaw`** . Чтобы использовать его, обновите узел образа следующим образом:
        ```
        "image": {
                "scale": 
                {
                    "mode": "preserveAspectRatio",
                    "width": "416",
                    "height": "416"
                },
                "format": 
                {
                    "@type": "#Microsoft.Media.MediaGraphImageFormatRaw",
                    "pixelFormat": "rgba"
                }
            }
        ```
        >[!NOTE]
        > Возможные значения pixelFormat включают: `yuv420p` , `rgb565be` ,, `rgb565le` , `rgb555be` `rgb555le` , `rgb24` , `bgr24` , `argb` , `rgba` , `abgr` , `bgra`  

    #### <a name="extensionconfiguration-for-grpc-extension-processor"></a>Екстенсионконфигуратион для обработчика расширений GRPC  
    * В `MediaGraphGrpcExtension` процессоре доступно новое свойство с именем **`extensionConfiguration`** , которое является необязательной строкой, которая может использоваться как часть контракта gRPC. Это поле можно использовать для передачи любых данных на сервер вывода, а также для определения того, как сервер вывода использует эти данные.  
    Один из вариантов использования этого свойства — при наличии нескольких моделей искусственного интеллекта, упакованных в один сервер вывода. С этим свойством не нужно предоставлять узел для каждой модели искусственного интеллекта. Вместо этого для экземпляра Graph в качестве поставщика расширений можно определить, как выбрать различные модели AI с помощью **`extensionConfiguration`** Свойства и во время выполнения лва передаст эту строку серверу, который может использовать его для вызова требуемой модели искусственного интеллекта.  

    #### <a name="ai-composition"></a>Композиция AI
    * Live Video Analytics 2,0 теперь поддерживает использование нескольких процессоров расширения Media Graph в пределах топологии. Кадры мультимедиа из камеры RTSP можно передавать в различные модели ИСКУССТВЕНно последовательно, параллельно или вместе. См. Пример топологии, в которой показаны две модели искусственного интеллекта, используемые последовательно.

### <a name="disk-space-management-with-sink-nodes"></a>Управление дисковым пространством с помощью узлов приемника
* В узле **приемника файлов** теперь можно указать, сколько места на диске может использовать модуль Live Video Analytics в IOT Edge модуле для хранения обработанных изображений. Для этого добавьте **`maximumSizeMiB`** поле в узел филесинк. Пример узла приемника файла выглядит следующим образом:
    ```
    "sinks": [
      {
        "@type": "#Microsoft.Media.MediaGraphFileSink",
        "name": "fileSink",
        "inputs": [
          {
            "nodeName": "signalGateProcessor",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value": "video"
              }
            ]
          }
        ],
        "fileNamePattern": "sampleFiles-${System.DateTime}",
        "maximumSizeMiB":"512",
        "baseDirectoryPath":"/var/media"
      }
    ]
    ```
* В узле **приемника ресурса** можно указать, сколько места на диске может использоваться модулем Live Video Analytics в IOT Edge модуля для хранения обработанных изображений. Для этого добавьте **`localMediaCacheMaximumSizeMiB`** поле в узел приемника ресурса. Пример узла приемника ресурса выглядит следующим образом:
    ```
    "sinks": [
      {
        "@type": "#Microsoft.Media.MediaGraphAssetSink",
        "name": "AssetSink",
        "inputs": [
          {
            "nodeName": "signalGateProcessor",
            "outputSelectors": [
              {
                "property": "mediaType",
                "operator": "is",
                "value": "video"
              }
            ]
          }
        ],
        "assetNamePattern": "sampleAsset-${System.GraphInstanceName}",
        "segmentLength": "PT30S",
        "localMediaCacheMaximumSizeMiB":"200",
        "localMediaCachePath":"/var/lib/azuremediaservices/tmp/"
      }
    ]
    ```
    >[!NOTE]
    >  Путь к **приемнику файла** разбивается на базовый путь к каталогу и шаблон имени файла, тогда как путь к **приемнику ресурса** включает базовый путь к каталогу.  

### <a name="frame-rate-management"></a>Управление частотой кадров
* **`MediaGraphFrameRateFilterProcessor`** не рекомендуется использовать в модуле **Live Video Analytics для модуля IoT Edge 2,0** .
    * Чтобы получить пример входящего видео для обработки, добавьте **`samplingOptions`** свойство в обработчики расширения медиаграф ( `MediaGraphHttpExtension` или `MediaGraphGrpcExtension` ).  
     ```
          "samplingOptions": 
          {
            "skipSamplesWithoutAnnotation": "false",  // If true, limits the samples submitted to the extension to only samples which have associated inference(s) 
            "maximumSamplesPerSecond": "20"   // Maximum rate of samples submitted to the extension
          },
    ```
### <a name="module-metrics-in-the-prometheus-format-using-telegraf"></a>Метрики модуля в формате Prometheus с помощью Telegraf
В этом выпуске Telegraf можно использовать для отправки метрик в Azure Monitor. После этого метрики можно перенаправлять в Log Analytics или концентратор событий.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/telemetry-schema/telegraf.png" alt-text="Классификация событий":::

С помощью DOCKER можно легко создать образ Telegraf с пользовательской конфигурацией. Дополнительные сведения см. на странице [мониторинг и ведение журнала](monitoring-logging.md#azure-monitor-collection-via-telegraf) .

## <a name="next-steps"></a>Дальнейшие действия

[Приступая к работе с Video Analytics на IoT Edge](get-started-detect-motion-emit-events-quickstart.md)