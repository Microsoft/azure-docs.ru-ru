---
title: Транскрибирование в реальном времени
titleSuffix: Azure Media Services
description: Сведения о динамической транскрипции служб мультимедиа Azure.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: c3465e294af104c4d9c3b34960f5e95cf41e7cb8
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291421"
---
# <a name="live-transcription-preview"></a>Динамическая транскрипция (Предварительная версия)

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Служба мультимедиа Azure доставляет видео, аудио и текст в разные протоколы. Когда вы публикуете динамический поток с помощью MPEG-ТИРЕ или HLS/КМАФ, а также видео и аудио, наша служба доставляет текст расшифрованной в ИМСК 1.1, совместимом с TTML. Доставка упаковывается в фрагменты MPEG-4, часть 30 (ISO/IEC 14496-30). Если используется доставка через HLS/TS, то текст доставляется как фрагментированный ВТТ.

При включении интерактивной трансляции взимается дополнительная плата. Ознакомьтесь со сведениями о ценах в реальном видео на [странице цен на службы мультимедиа](https://azure.microsoft.com/pricing/details/media-services/).

В этой статье описывается, как включить динамическую транскрипцию при потоковой передаче события в реальном времени с помощью служб мультимедиа Azure. Прежде чем продолжить, убедитесь, что вы знакомы с использованием API-интерфейсов службы мультимедиа v3 версии 3 (Дополнительные сведения см. в [этом руководстве](stream-files-tutorial-with-rest.md) ). Также следует ознакомиться с концепцией [потоковой передачи в реальном времени](live-streaming-overview.md) . Мы рекомендуем завершить работу [со службой "поток Live" с помощью служб мультимедиа](stream-live-tutorial-with-api.md) .

## <a name="live-transcription-preview-regions-and-languages"></a>Регионы и языки интерактивного просмотра транскрипции

Динамическая транскрипция доступна в следующих регионах:

- Southeast Asia
- Западная Европа
- Северная Европа
- Восточная часть США
- Центральная часть США
- Центрально-южная часть США
- западная часть США 2
- Brazil South

Это список доступных языков, которые можно расшифрованной, использовать код языка в API.

| Язык | Код языка |
| -------- | ------------- |
| Каталонский  | ca-ES |
| Датский (Дания) | da-DK |
| Немецкий (Германия) | de-DE |
| Английский (Австралия) | en-AU |
| Английский (Канада) | en-CA |
| Английский (Великобритания) | en-GB |
| Английский (Индия) | en-IN |
| Английский (Новая Зеландия) | en-NZ |
| Английский (США) | en-US |
| испанский (Испания) | es-ES |
| Испанский (Мексика) | es-MX |
| Финский (Финляндия) | fi-FI |
| Французский (Канада) | fr-CA |
| Французский (Франция) | fr-FR |
| Итальянский (Италия) | it-IT |
| Нидерландский (Нидерланды) | nl-NL |
| Португальский (Бразилия) | pt-BR |
| Португальский (Португалия) | pt-PT |
| Шведский (Швеция) | sv-SE |

## <a name="create-the-live-event-with-live-transcription"></a>Создание интерактивного мероприятия с помощью динамической транскрипции

Чтобы создать событие Live с включенной транскрипцией, отправьте операцию размещения с версией API 2019-05-01-Preview, например:

```
PUT https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/liveEvents/:liveEventName?api-version=2019-05-01-preview&autoStart=true 
```

Эта операция имеет следующий текст (где в качестве протокола приема создается передаваемое событие с помощью RTMP). Обратите внимание на добавление свойства "транскрипции".

```
{
  "properties": {
    "description": "Demonstrate how to enable live transcriptions",
    "input": {
      "streamingProtocol": "RTMP",
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "preview": {
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "encoding": {
      "encodingType": "None"
    },
    "transcriptions": [
      {
        "language": "en-US"
      }
    ],
    "vanityUrl": false,
    "streamOptions": [
      "Default"
    ]
  },
  "location": "West US 2"
}
```

## <a name="start-or-stop-transcription-after-the-live-event-has-started"></a>Запуск или окончание записи разговора после запуска события Live

Вы можете запускать и прекращать интерактивную транскрипцию, пока событие Live находится в рабочем состоянии. Дополнительные сведения о запуске и остановке активных событий см. в разделе о долгосрочных операциях в статье [Разработка с помощью API-интерфейсов служб мультимедиа v3](media-services-apis-overview.md#long-running-operations).

Чтобы включить динамическую транскрипцию или обновить язык транскрипции, обновите событие Live, включив в него свойство "транскрипции". Чтобы отключить динамическую транскрипцию, удалите свойство "транскрипции" из объекта Live Event.  

> [!NOTE]
> Включение или выключение транскрипции **более одного раза** во время интерактивного события не является поддерживаемым сценарием.

Это пример вызова для включения Live транскрипций.

ЗАЩИТЫ ```https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/liveEvents/:liveEventName?api-version=2019-05-01-preview```

```
{
  "properties": {
    "description": "Demonstrate how to enable live transcriptions", 
    "input": {
      "streamingProtocol": "RTMP",
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "preview": {
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "encoding": {
      "encodingType": "None"
    },
    "transcriptions": [
      {
        "language": "en-US"
      }
    ],
    "vanityUrl": false,
    "streamOptions": [
      "Default"
    ]
  },
  "location": "West US 2"
}
```

## <a name="transcription-delivery-and-playback"></a>Доставка и воспроизведение транскрипции

Ознакомьтесь со статьей [Обзор динамической упаковки](dynamic-packaging-overview.md#to-prepare-your-source-files-for-delivery) , в которой наша служба использует динамическую упаковку для передачи видео, аудио и текста в разные протоколы. Когда вы публикуете динамический поток с помощью MPEG-ТИРЕ или HLS/КМАФ, а также видео и аудио, наша служба доставляет текст расшифрованной в ИМСК 1.1, совместимом с TTML. Эта доставка упаковывается в фрагменты MPEG-4, часть 30 (ISO/IEC 14496-30). При использовании доставки через HLS/TS текст доставляется как фрагментированный ВТТ. Для воспроизведения потока можно использовать веб-проигрыватель, например [проигрыватель мультимедиа Azure](use-azure-media-player.md) .  

> [!NOTE]
> При использовании Проигрыватель мультимедиа Azure используйте версию 2.3.3 или более позднюю.

## <a name="known-issues"></a>Известные проблемы

Для предварительной версии ниже приведены известные проблемы с динамической обсчетом.

- Приложения должны использовать API предварительной версии, описанные в [спецификации служб мультимедиа v3 OpenAPI](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/preview/2019-05-01-preview/streamingservice.json).
- Защита цифрового управления цифровыми правами (DRM) не применяется к текстовой дорожке, возможно только шифрование с помощью алгоритма AES.

## <a name="next-steps"></a>Дальнейшие действия

* [Общие сведения о службах мультимедиа](media-services-overview.md)
