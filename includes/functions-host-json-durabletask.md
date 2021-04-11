---
title: включить файл
description: включить файл
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/14/2019
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: d1b2cbefe64add5048d4e3c7946277b1461a5434
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105607100"
---
Параметры конфигурации для [устойчивых функций](../articles/azure-functions/durable/durable-functions-overview.md).

> [!NOTE]
> Все основные версии Устойчивых функций поддерживаются во всех версиях среды выполнения Функций Azure. При этом схема конфигурации host.json немного отличается в зависимости от версии среды выполнения Функций Azure и используемой версии расширения Устойчивых функций. Следующие примеры предназначены для использования с Функциями Azure 2.0 и 3.0. Если вы используете Функции Azure 1.0, в обоих примерах доступные параметры будут одинаковыми, но раздел durableTask файла host.json должен находиться в корне конфигурации host.json, а не в поле extensions.

### <a name="durable-functions-2x"></a><a name="durable-functions-2-0-host-json"></a>Устойчивые функции 2.x

```json
{
 "extensions": {
  "durableTask": {
    "hubName": "MyTaskHub",
    "storageProvider": {
      "connectionStringName": "AzureWebJobsStorage",
      "controlQueueBatchSize": 32,
      "controlQueueBufferThreshold": 256,
      "controlQueueVisibilityTimeout": "00:05:00",
      "maxQueuePollingInterval": "00:00:30",
      "partitionCount": 4,
      "trackingStoreConnectionStringName": "TrackingStorage",
      "trackingStoreNamePrefix": "DurableTask",
      "useLegacyPartitionManagement": true,
      "workItemQueueVisibilityTimeout": "00:05:00",
    },
    "tracing": {
      "traceInputsAndOutputs": false,
      "traceReplayEvents": false,
    },
    "notifications": {
      "eventGrid": {
        "topicEndpoint": "https://topic_name.westus2-1.eventgrid.azure.net/api/events",
        "keySettingName": "EventGridKey",
        "publishRetryCount": 3,
        "publishRetryInterval": "00:00:30",
        "publishEventTypes": [
          "Started",
          "Pending",
          "Failed",
          "Terminated"
        ]
      }
    },
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10,
    "extendedSessionsEnabled": false,
    "extendedSessionIdleTimeoutInSeconds": 30,
    "useAppLease": true,
    "useGracefulShutdown": false
  }
 }
}

```

### <a name="durable-functions-1x"></a>Устойчивые функции 1.x

```json
{
  "extensions": {
    "durableTask": {
      "hubName": "MyTaskHub",
      "controlQueueBatchSize": 32,
      "partitionCount": 4,
      "controlQueueVisibilityTimeout": "00:05:00",
      "workItemQueueVisibilityTimeout": "00:05:00",
      "maxConcurrentActivityFunctions": 10,
      "maxConcurrentOrchestratorFunctions": 10,
      "maxQueuePollingInterval": "00:00:30",
      "azureStorageConnectionStringName": "AzureWebJobsStorage",
      "trackingStoreConnectionStringName": "TrackingStorage",
      "trackingStoreNamePrefix": "DurableTask",
      "traceInputsAndOutputs": false,
      "logReplayEvents": false,
      "eventGridTopicEndpoint": "https://topic_name.westus2-1.eventgrid.azure.net/api/events",
      "eventGridKeySettingName":  "EventGridKey",
      "eventGridPublishRetryCount": 3,
      "eventGridPublishRetryInterval": "00:00:30",
      "eventGridPublishEventTypes": ["Started", "Completed", "Failed", "Terminated"]
    }
  }
}
```

Имена центров задач должны начинаться с буквы и содержать только буквы и цифры. Если имя не указано, центру задач в приложении-функции назначается имя по умолчанию **DurableFunctionsHub**. Дополнительные сведения см. в статье о [центрах задач](../articles/azure-functions/durable/durable-functions-task-hubs.md).

|Свойство  |По умолчанию | Описание |
|---------|---------|----------|
|hubName|DurableFunctionsHub|Альтернативные имена [центра задач](../articles/azure-functions/durable/durable-functions-task-hubs.md) позволяют изолировать несколько приложений устойчивых функций друг от друга, даже если они используют один и тот же интерфейс хранилища.|
|controlQueueBatchSize|32|Количество сообщений, одновременно извлекаемых из очереди управления.|
|controlQueueBufferThreshold| **План потребления**: 32 <br> **План "Выделенный" или "Премиум"** : 256 |Количество сообщений в очереди управления, которые можно поместить в память одновременно, после чего диспетчер будет ожидать исключения дополнительных сообщений из очереди.|
|partitionCount |4|Число разделов для очереди управления. Допускается целочисленное значение в диапазоне от 1 до 16.|
|controlQueueVisibilityTimeout |5 минут|Время видимости для сообщений, выведенных из очереди управления.|
|workItemQueueVisibilityTimeout |5 минут|Время видимости для сообщений, выведенных из очереди рабочих элементов.|
|maxConcurrentActivityFunctions | **План потребления**: 10 <br> **План "Выделенный" или "Премиум"** : 10 × количество процессоров на текущем компьютере|Максимальное число функции действия, которые могут параллельно обрабатываться на одном экземпляре узла.|
|maxConcurrentOrchestratorFunctions | **План потребления**: 5 <br> **План "Выделенный" или "Премиум"** : 10 × количество процессоров на текущем компьютере |Максимальное количество функций оркестратора, которые могут быть обработаны параллельно в одном экземпляре узла.|
|maxQueuePollingInterval|30 секунд|Максимальный интервал опроса очереди управления и рабочих элементов в формате *чч:мм:сс*. Высокие значения могут привести к увеличению задержек при обработке сообщений. Низкие значения могут привести к повышению затрат на хранение из-за увеличенного объема транзакций с хранилищем.|
|azureStorageConnectionStringName |AzureWebJobsStorage|Имя параметра приложения, в котором хранится строка подключения к службе хранилища Azure для управления базовыми ресурсами этой службы.|
|trackingStoreConnectionStringName||Имя строки подключения, используемой для таблиц журнала и экземпляров. Если значение не указано, используется подключение `connectionStringName` (Устойчивые функции 2.x) или `azureStorageConnectionStringName` (Устойчивые функции 1.x).|
|trackingStoreNamePrefix||Префикс, используемый для таблиц журнала и экземпляров, если задано значение `trackingStoreConnectionStringName`. Если значение не задано, по умолчанию будет использоваться префикс `DurableTask`. Если значение `trackingStoreConnectionStringName` не задано, для таблиц журнала и экземпляров в качестве префикса будет использоваться значение `hubName`, а любой параметр для `trackingStoreNamePrefix` будет игнорироваться.|
|traceInputsAndOutputs |false|Это значение указывает, нужно ли отслеживать входы и выходы вызовов функций. При трассировке событий выполнения функции по умолчанию для вызовов функций фиксируется количество байтов в сериализованных входных и выходных данных. Такое поведение позволяет получить некоторое представление о входах и выходах, не увеличивая размеры журналов и не раскрывая конфиденциальные сведения. Если вы присвоите этому свойству значение true, в журналы выполнения функций будет включаться полное содержимое их входов и выходов.|
|logReplayEvents|false|Значение, указывающее, следует ли записывать повторные события оркестрации в Application Insights.|
|eventGridTopicEndpoint ||URL-адрес конечной точки пользовательского раздела службы "Сетка событий Azure". Если установлен этот параметр, события уведомления о жизненном цикле оркестрации публикуются в указанную конечную точку. Это свойство поддерживает разрешение параметров приложения.|
|eventGridKeySettingName ||Имя параметра приложения, содержащего ключ для аутентификации в пользовательском разделе службы "Сетка событий Azure" в `EventGridTopicEndpoint`.|
|eventGridPublishRetryCount|0|Число повторных попыток, если публикация в разделе "Сетка событий" завершается сбоем.|
|eventGridPublishRetryInterval|5 минут|Интервал повторных попыток для публикации в разделе "Сетка событий" указывается в формате *чч: мм:сс*.|
|eventGridPublishEventTypes||Список типов событий для публикации в Сетке событий. Если значение не указано, будут опубликованы все типы событий. Допустимые значения: `Started`, `Completed`, `Failed`, `Terminated`.|
|useAppLease|Да|Если задано значение `true`, перед обработкой сообщений центра задач приложениям потребуется получить аренду большого двоичного объекта уровня приложения. Дополнительные сведения см. в документации по [аварийному восстановлению и географическом распределению](../articles/azure-functions/durable/durable-functions-disaster-recovery-geo-distribution.md). Доступно, начиная с версии 2.3.0.
|useLegacyPartitionManagement|false|Если задано значение `false`, используется алгоритм управления секциями, который снижает вероятность дублирования выполнения функций при горизонтальном масштабировании.  Доступно, начиная с версии 2.3.0.|
|useGracefulShutdown|false|Предварительная версия. Включение корректного завершения работы для снижения вероятности того, что при завершении работы основного приложения произойдет сбой внутрипроцессного выполнения функции.|

Многие из этих параметров предназначены для оптимизации производительности. Дополнительные сведения см. в разделе [Производительность и масштабируемость](../articles/azure-functions/durable/durable-functions-perf-and-scale.md).
