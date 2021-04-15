---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/04/2018
ms.author: glenga
ms.openlocfilehash: fe80a71125d43220e408eab7b07aeedcafa0a526
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102473687"
---
### <a name="functions-2x-and-higher"></a>Функции 2.x и более поздних версий

```json
{
    "version": "2.0",
    "extensions": {
        "eventHubs": {
            "batchCheckpointFrequency": 5,
            "eventProcessorOptions": {
                "maxBatchSize": 256,
                "prefetchCount": 512
            },
            "initialOffsetOptions": {
                "type": "fromStart",
                "enqueuedTimeUtc": ""
            }
        }
    }
}  
```

|Свойство  |По умолчанию | Описание |
|---------|---------|---------|
|batchCheckpointFrequency|1|Количество пакетов событий, которые необходимо обработать перед созданием контрольной точки курсора EventHub.|
|eventProcessorOptions/maxBatchSize|10|Максимальное число событий, получаемых в цикле получения.|
|eventProcessorOptions/prefetchCount|300|Счетчик предварительной выборки по умолчанию, используемый базовым `EventProcessorHost`. Минимальное допустимое значение — 10.|
|initialOffsetOptions/type<sup>1</sup>|fromStart|Расположение в потоке событий, с которого следует начать обработку, когда контрольная точка не существует в хранилище. Может принимать значение `fromStart`, `fromEnd` или `fromEnqueuedTime`. `fromEnd` обрабатывает новые события, которые были поставлены в очередь после начала выполнения приложения-функции. Применяется ко всем разделам.  Дополнительные сведения см. в [документации по EventProcessorOptions](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.initialoffsetprovider).|
|initialOffsetOptions/enqueuedTimeUtc<sup>1</sup>|Недоступно| Указывает время в очереди для события в потоке, с которого начинается обработка. Если для `initialOffsetOptions/type` настроено значение `fromEnqueuedTime`, этот параметр является обязательным. Поддерживает время в любом формате, поддерживаемом [DateTime.Parse()](/dotnet/standard/base-types/parsing-datetime), например `2020-10-26T20:31Z`. Для ясности следует также указать часовой пояс. Если часовой пояс не указан, Функции предполагают, что используется местный часовой пояс компьютера, на котором запущено приложение-функция (при выполнении в Azure это время в формате UTC). Дополнительные сведения см. в [документации по EventProcessorOptions](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.initialoffsetprovider).|

<sup>1</sup> Поддержка `intitialOffsetOptions` реализована в [EventHubs v 4.2.0](https://github.com/Azure/azure-functions-eventhubs-extension/releases/tag/v4.2.0) и более поздних версиях.

> [!NOTE]
> См. сведения о [файле host.json в Функциях Azure версии 2.x и выше](../articles/azure-functions/functions-host-json.md).

### <a name="functions-1x"></a>Функции 1.x

```json
{
    "eventHub": {
      "maxBatchSize": 64,
      "prefetchCount": 256,
      "batchCheckpointFrequency": 1
    }
}
```

|Свойство  |По умолчанию | Описание |
|---------|---------|---------| 
|maxBatchSize|64|Максимальное число событий, получаемых в цикле получения.|
|prefetchCount|Недоступно|Предварительная выборка по умолчанию, используемая базовым `EventProcessorHost`.| 
|batchCheckpointFrequency|1|Количество пакетов событий, которые необходимо обработать перед созданием контрольной точки курсора EventHub.| 

> [!NOTE]
> См. сведения о [файле host.json в Функциях Azure версии 1.x](../articles/azure-functions/functions-host-json-v1.md).
