---
title: Службы мультимедиа Azure в качестве источника "Сетка событий"
description: В этой статье приведены свойства событий Служб мультимедиа, используемых со службой "Сетка событий Azure".
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 1f2f62f0a5ceed0e000c8bb7690fff009593bf82
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104591934"
---
# <a name="azure-media-services-as-an-event-grid-source"></a>Службы мультимедиа Azure в качестве источника "Сетка событий"

В этой статье приведены схемы и свойства событий Служб мультимедиа.

## <a name="job-related-event-types"></a>Типы событий, связанные с заданиями

Службы мультимедиа выдают **связанные с заданием**  типы событий, описанные ниже. Существуют две категории событий, **связанных с заданием** : "мониторинг изменений состояния задания" и "изменения состояния выходных данных задания мониторинга". 

Вы можете зарегистрироваться на все события, подписавшись на событие JobStateChange. Или, вы можете подписаться на отдельные события (к примеру, такие конечные состояния как JobErrored, JobFinished и JobCanceled).   

### <a name="monitoring-job-state-changes"></a>Наблюдение за изменениями состояния задания

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft.Media.JobStateChange| Получить событие о всех изменениях состояние задания. |
| Microsoft.Media.JobScheduled| Получить событие, когда задание переходит в состояние запланированного. |
| Microsoft.Media.JobProcessing| Получить событие, когда задание переходит в состояние обработки. |
| Microsoft.Media.JobCanceling| Получить событие, когда задание переходит в состояние отмены. |
| Microsoft.Media.JobCanceled| Получить событие, когда задание переходит в состояние отмененного. Это конечное состояние, которое включает в себя выходные данные задания.|
| Microsoft.Media.JobErrored | Получить событие, когда задание переходит в состояние ошибки. Это конечное состояние, которое включает в себя выходные данные задания.|

Ознакомьтесь с [примерами схемы событий](#event-schema-examples) приведенными ниже.

### <a name="monitoring-job-output-state-changes"></a>Изменения состояния выходных данных задания мониторинга

Задание может содержать несколько выходных данных задания (если преобразование настроено для вывода нескольких выходных данных задания). Если вы хотите отслеживать данные отдельного задания, прослушайте событие изменения выходных данных задания.

Каждое **Задание** будет находиться на более высоком уровне, чем **жобаутпут**, поэтому события вывода задания будут срабатывать в соответствующем задании. 

Сообщения об ошибках `JobFinished` в `JobCanceled` , `JobError` выводят агрегированные результаты для каждого выходного задания — когда все они завершаются. В то время как события вывода задания срабатывают по завершении каждой задачи. Например, если у вас есть выходная кодировка, а затем выходные данные аналитики видео, то в качестве выходных событий задания будут выводиться два события, прежде чем окончательное событие Жобфинишед будет срабатывать с агрегированными данными.

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft.Media.JobOutputStateChange| Получить событие о всех изменениях состояния выходных данных задания. |
| Microsoft.Media.JobOutputScheduled| Получить событие, когда выходные данные задания переходят в состояние запланированных. |
| Microsoft.Media.JobOutputProcessing| Получить событие, когда выходные данные задания переходят в состояние обработки. |
| Microsoft.Media.JobOutputCanceling| Получить событие, когда выходные данные задания переходят в состояние отмены.|
| Microsoft.Media.JobOutputFinished| Получить событие, когда выходные данные задания переходят в состояние завершения.|
| Microsoft.Media.JobOutputCanceled| Получить событие, когда выходные данные задания переходят в состояние отмененных.|
| Microsoft.Media.JobOutputErrored| Получить событие, когда выходные данные задания переходят в состояние ошибки.|

Ознакомьтесь с [примерами схемы событий](#event-schema-examples) приведенными ниже.

### <a name="monitoring-job-output-progress"></a>Ход выполнения задания мониторинга

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft.Media.JobOutputProgress| Данное событие отображает ход выполнения задания, от 0 % до 100 %. Служба попытается отправить событие, когда ход выполнения задания увеличился на 5 % или более или прошло более 30 секунд с момента последнего события (пакета пульса). Значение хода выполнения не гарантированно начинается с 0% или до 100%. также не гарантируется, что оно будет увеличиваться по постоянной частоте с течением времени. Это событие не должно быть использовано для определения завершения обработки. Вместо этого следует использовать события изменения состояния.|

Ознакомьтесь с [примерами схемы событий](#event-schema-examples) приведенными ниже.

## <a name="live-event-types"></a>Типы событий, связанных с прямой трансляцией

Служба мультимедиа также выдает следующие типы событий, связанных с **прямой трансляцией**. Существуют две категории для событий **прямой трансляции**: события уровня потока и события уровня дорожки. 

### <a name="stream-level-events"></a>События уровня потока

События уровня потока вызываются в потоке или подключении. Каждое событие имеет параметр `StreamId`, который идентифицирует подключение или поток. Каждый поток или подключение имеет одну или несколько дорожек разных типов. Например, одно подключение от кодировщика может иметь одну аудиодорожку и четыре видеодорожки. К типам событий потока относятся следующие:

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft.Media.LiveEventConnectionRejected | Попытка подключения кодировщика отклоняется. |
| Microsoft.Media.LiveEventEncoderConnected | Кодировщик устанавливает подключение с событием прямой трансляции. |
| Microsoft.Media.LiveEventEncoderDisconnected | Отключение кодировщика. |

Ознакомьтесь с [примерами схемы событий](#event-schema-examples) приведенными ниже.

### <a name="track-level-events"></a>События уровня дорожки

События уровня дорожки вызываются в каждой дорожке. 

> [!NOTE]
> Все события уровня Track вызываются после подключения динамического кодировщика.

Типы событий уровня Track:

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft.Media.LiveEventIncomingDataChunkDropped | Сервер мультимедиа удаляет блок данных, потому что он устарел или имеет перекрывающуюся метку времени (метка времени нового блока данных меньше времени окончания предыдущего блока данных). |
| Microsoft.Media.LiveEventIncomingStreamReceived | Сервер мультимедиа получает первый блок данных для каждой дорожки в потоковой передаче или подключении. |
| Microsoft.Media.LiveEventIncomingStreamsOutOfSync | Сервер мультимедиа обнаруживает, что потоки аудио и видео не синхронизированы. Используйте в качестве предупреждения, так как взаимодействие с пользователем не может быть затронуто. |
| Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync | Сервер мультимедиа обнаруживает, что любой из двух видеопотоков, поступающих от внешнего кодировщика, не синхронизирован. Используйте в качестве предупреждения, так как взаимодействие с пользователем не может быть затронуто. |
| Microsoft.Media.LiveEventIngestHeartbeat | Публикуется каждые 20 секунд для каждой дорожки, когда выполняется событие прямой трансляции. Предоставляет сводку работоспособности приема.<br/><br/>После первоначального подключения кодировщика событие пульса продолжает отправлять каждые 20 секунд, независимо от того, подключен ли кодировщик. |
| Microsoft.Media.LiveEventTrackDiscontinuityDetected | Сервер мультимедиа обнаруживает разрыв во входящей дорожке. |

Ознакомьтесь с [примерами схемы событий](#event-schema-examples) приведенными ниже.

## <a name="event-schema-examples"></a>Примеры схемы событий

### <a name="jobstatechange"></a>JobStateChange

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

В примере ниже показана схема события **JobStateChange**. 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
    "eventType": "Microsoft.Media.JobStateChange",
    "eventTime": "2018-04-20T21:26:13.8978772",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "previousState": "Processing",
      "state": "Finished"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

В примере ниже показана схема события **JobStateChange**. 

```json
[
  {
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
    "type": "Microsoft.Media.JobStateChange",
    "time": "2018-04-20T21:26:13.8978772",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "previousState": "Processing",
      "state": "Finished"
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `previousState` | строка | Состояние задания перед событием. |
| `state` | строка | Новое состояние задания в этом событии. Например, "запланировано: Задание готово к запуску" или "завершено: задание завершено".|

Возможные значения состояния задания: *В очереди*, *Запланировано*, *Обработка*, *Завершено*, *Ошибка*, *Отменено*, *Отмена*.

> [!NOTE]
> Состояние *Queued* (В очереди) может быть присвоено только свойству **previousState**, но не свойству **state**.

### <a name="jobscheduled-jobprocessing-jobcanceling"></a>JobScheduled, JobProcessing, JobCanceling

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Для каждого неокончательного изменения состояния задания (например, JobScheduled, JobProcessing, JobCanceling) пример схемы выглядит следующим образом:

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobProcessing",
  "eventTime": "2018-10-12T16:12:18.0839935",
  "id": "a0a6efc8-f647-4fc2-be73-861fa25ba2db",
  "data": {
    "previousState": "Scheduled",
    "state": "Processing",
    "correlationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="jobfinished-jobcanceled-joberrored"></a>JobFinished, JobCanceled, JobErrored

Для каждого окончательного изменения состояния задания (например, JobScheduled, JobProcessing, JobCanceling) пример схемы выглядит следующим образом:

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobFinished",
  "eventTime": "2018-10-12T16:25:56.4115495",
  "id": "9e07e83a-dd6e-466b-a62f-27521b216f2a",
  "data": {
    "outputs": [
      {
        "@odata.type": "#Microsoft.Media.JobOutputAsset",
        "assetName": "output-7640689F",
        "error": null,
        "label": "VideoAnalyzerPreset_0",
        "progress": 100,
        "state": "Finished"
      }
    ],
    "previousState": "Processing",
    "state": "Finished",
    "correlationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Для каждого неокончательного изменения состояния задания (например, JobScheduled, JobProcessing, JobCanceling) пример схемы выглядит следующим образом:

```json
[{
  "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "type": "Microsoft.Media.JobProcessing",
  "time": "2018-10-12T16:12:18.0839935",
  "id": "a0a6efc8-f647-4fc2-be73-861fa25ba2db",
  "data": {
    "previousState": "Scheduled",
    "state": "Processing",
    "correlationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="jobfinished-jobcanceled-joberrored"></a>JobFinished, JobCanceled, JobErrored

Для каждого окончательного изменения состояния задания (например, JobScheduled, JobProcessing, JobCanceling) пример схемы выглядит следующим образом:

```json
[{
  "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "type": "Microsoft.Media.JobFinished",
  "time": "2018-10-12T16:25:56.4115495",
  "id": "9e07e83a-dd6e-466b-a62f-27521b216f2a",
  "data": {
    "outputs": [
      {
        "@odata.type": "#Microsoft.Media.JobOutputAsset",
        "assetName": "output-7640689F",
        "error": null,
        "label": "VideoAnalyzerPreset_0",
        "progress": 100,
        "state": "Finished"
      }
    ],
    "previousState": "Processing",
    "state": "Finished",
    "correlationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "specversion": "1.0"
}]
```

---


Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `outputs` | Array | Получить выходные данные задания.|

### <a name="joboutputstatechange"></a>JobOutputStateChange

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

В примере ниже показана схема события **JobOutputStateChange**:

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobOutputStateChange",
  "eventTime": "2018-10-12T16:25:56.0242854",
  "id": "dde85f46-b459-4775-b5c7-befe8e32cf90",
  "data": {
    "previousState": "Processing",
    "output": {
      "@odata.type": "#Microsoft.Media.JobOutputAsset",
      "assetName": "output-7640689F",
      "error": null,
      "label": "VideoAnalyzerPreset_0",
      "progress": 100,
      "state": "Finished"
    },
    "jobCorrelationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="joboutputscheduled-joboutputprocessing-joboutputfinished-joboutputcanceling-joboutputcanceled-joboutputerrored"></a>JobOutputScheduled, JobOutputProcessing, JobOutputFinished, JobOutputCanceling, JobOutputCanceled, JobOutputErrored

Для каждого изменения состояния JobOutput пример схемы выглядит следующим образом:

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobOutputProcessing",
  "eventTime": "2018-10-12T16:12:18.0061141",
  "id": "f1fd5338-1b6c-4e31-83c9-cd7c88d2aedb",
  "data": {
    "previousState": "Scheduled",
    "output": {
      "@odata.type": "#Microsoft.Media.JobOutputAsset",
      "assetName": "output-7640689F",
      "error": null,
      "label": "VideoAnalyzerPreset_0",
      "progress": 0,
      "state": "Processing"
    },
    "jobCorrelationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```
### <a name="joboutputprogress"></a>JobOutputProgress

Схема, указанная в примере, должна соответствовать приведенной ниже.

 ```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/belohGroup/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/job-5AB6DE32",
  "eventType": "Microsoft.Media.JobOutputProgress",
  "eventTime": "2018-12-10T18:20:12.1514867",
  "id": "00000000-0000-0000-0000-000000000000",
  "data": {
    "jobCorrelationData": {
      "TestKey1": "TestValue1",
      "testKey2": "testValue2"
    },
    "label": "VideoAnalyzerPreset_0",
    "progress": 86
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="liveeventconnectionrejected"></a>LiveEventConnectionRejected;

Следующий пример демонстрирует схему события **LiveEventConnectionRejected**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "eventType": "Microsoft.Media.LiveEventConnectionRejected",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "b303db59-d5c1-47eb-927a-3650875fded1",
    "data": { 
      "streamId":"Mystream1",
      "ingestUrl": "http://abc.ingest.isml",
      "encoderIp": "118.238.251.xxx",
      "encoderPort": 52859,
      "resultCode": "MPE_INGEST_CODEC_NOT_SUPPORTED"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

В примере ниже показана схема события **JobOutputStateChange**:

```json
[{
  "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "type": "Microsoft.Media.JobOutputStateChange",
  "time": "2018-10-12T16:25:56.0242854",
  "id": "dde85f46-b459-4775-b5c7-befe8e32cf90",
  "data": {
    "previousState": "Processing",
    "output": {
      "@odata.type": "#Microsoft.Media.JobOutputAsset",
      "assetName": "output-7640689F",
      "error": null,
      "label": "VideoAnalyzerPreset_0",
      "progress": 100,
      "state": "Finished"
    },
    "jobCorrelationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "specversion": "1.0"
}]
```

### <a name="joboutputscheduled-joboutputprocessing-joboutputfinished-joboutputcanceling-joboutputcanceled-joboutputerrored"></a>JobOutputScheduled, JobOutputProcessing, JobOutputFinished, JobOutputCanceling, JobOutputCanceled, JobOutputErrored

Для каждого изменения состояния JobOutput пример схемы выглядит следующим образом:

```json
[{
  "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "type": "Microsoft.Media.JobOutputProcessing",
  "time": "2018-10-12T16:12:18.0061141",
  "id": "f1fd5338-1b6c-4e31-83c9-cd7c88d2aedb",
  "data": {
    "previousState": "Scheduled",
    "output": {
      "@odata.type": "#Microsoft.Media.JobOutputAsset",
      "assetName": "output-7640689F",
      "error": null,
      "label": "VideoAnalyzerPreset_0",
      "progress": 0,
      "state": "Processing"
    },
    "jobCorrelationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "specversion": "1.0"
}]
```
### <a name="joboutputprogress"></a>JobOutputProgress

Схема, указанная в примере, должна соответствовать приведенной ниже.

 ```json
[{
  "source": "/subscriptions/<subscription-id>/resourceGroups/belohGroup/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/job-5AB6DE32",
  "type": "Microsoft.Media.JobOutputProgress",
  "time": "2018-12-10T18:20:12.1514867",
  "id": "00000000-0000-0000-0000-000000000000",
  "data": {
    "jobCorrelationData": {
      "TestKey1": "TestValue1",
      "testKey2": "testValue2"
    },
    "label": "VideoAnalyzerPreset_0",
    "progress": 86
  },
  "specversion": "1.0"
}]
```

### <a name="liveeventconnectionrejected"></a>LiveEventConnectionRejected;

Следующий пример демонстрирует схему события **LiveEventConnectionRejected**: 

```json
[
  {
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "type": "Microsoft.Media.LiveEventConnectionRejected",
    "time": "2018-01-16T01:57:26.005121Z",
    "id": "b303db59-d5c1-47eb-927a-3650875fded1",
    "data": { 
      "streamId":"Mystream1",
      "ingestUrl": "http://abc.ingest.isml",
      "encoderIp": "118.238.251.xxx",
      "encoderPort": 52859,
      "resultCode": "MPE_INGEST_CODEC_NOT_SUPPORTED"
    },
    "specversion": "1.0"
  }
]
```

---


Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `streamId` | строка | Идентификатор потока или подключения. Кодировщик или клиент несет ответственность за добавление этого идентификатора в URL-адрес приема. |  
| `ingestUrl` | строка | URL-адрес приема, предоставленный событием прямой трансляции. |  
| `encoderIp` | строка | IP-адрес кодировщика. |
| `encoderPort` | строка | Порт кодировщика из источника этого потока. |
| `resultCode` | строка | Причина, по которой подключение было отклонено. Коды результатов перечислены в следующей таблице. |

Коды результатов ошибок можно найти в [кодах ошибок динамических событий](../media-services/latest/live-event-error-codes.md).

### <a name="liveeventencoderconnected"></a>LiveEventEncoderConnected;

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Следующий пример демонстрирует схему события **LiveEventEncoderConnected**: 

```json
[
  { 
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventEncoderConnected",
    "eventTime": "2018-08-07T23:08:09.1710643",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Следующий пример демонстрирует схему события **LiveEventEncoderConnected**: 

```json
[
  { 
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "type": "Microsoft.Media.LiveEventEncoderConnected",
    "time": "2018-08-07T23:08:09.1710643",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485"
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `streamId` | строка | Идентификатор потока или подключения. Кодировщик или клиент несет ответственность за указание этого идентификатора в URL-адресе приема. |
| `ingestUrl` | строка | URL-адрес приема, предоставленный событием прямой трансляции. |
| `encoderIp` | строка | IP-адрес кодировщика. |
| `encoderPort` | строка | Порт кодировщика из источника этого потока. |

### <a name="liveeventencoderdisconnected"></a>LiveEventEncoderDisconnected.

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Следующий пример демонстрирует схему события **LiveEventEncoderDisconnected**: 

```json
[
  { 
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventEncoderDisconnected",
    "eventTime": "2018-08-07T23:08:09.1710872",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "resultCode": "S_OK"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Следующий пример демонстрирует схему события **LiveEventEncoderDisconnected**: 

```json
[
  { 
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "type": "Microsoft.Media.LiveEventEncoderDisconnected",
    "time": "2018-08-07T23:08:09.1710872",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "resultCode": "S_OK"
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `streamId` | строка | Идентификатор потока или подключения. Кодировщик или клиент несет ответственность за добавление этого идентификатора в URL-адрес приема. |  
| `ingestUrl` | строка | URL-адрес приема, предоставленный событием прямой трансляции. |  
| `encoderIp` | строка | IP-адрес кодировщика. |
| `encoderPort` | строка | Порт кодировщика из источника этого потока. |
| `resultCode` | строка | Причина отключения кодировщика. Это может быть нормальное отключение или ошибка. Коды результатов перечислены в следующей таблице. |

Коды результатов ошибок можно найти в [кодах ошибок динамических событий](../media-services/latest/live-event-error-codes.md).

Коды результатов нормального отключения:

| Код результата | Описание: |
| ----------- | ----------- |
| S_OK | Кодировщик отключен успешно. |
| MPE_CLIENT_TERMINATED_SESSION | Кодировщик отключен (RTMP). |
| MPE_CLIENT_DISCONNECTED | Кодировщик отключен (FMP4). |
| MPI_REST_API_CHANNEL_RESET | Получена команда сброса канала. |
| MPI_REST_API_CHANNEL_STOP | Получена команда остановки канала. |
| MPI_REST_API_CHANNEL_STOP | Канал находится в режиме обслуживания. |
| MPI_STREAM_HIT_EOF | Поток конца файла отправлен кодировщиком. |

### <a name="liveeventincomingdatachunkdropped"></a>LiveEventIncomingDataChunkDropped;

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Следующий пример демонстрирует схему события **LiveEventIncomingDataChunkDropped**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "eventType": "Microsoft.Media.LiveEventIncomingDataChunkDropped",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "03da9c10-fde7-48e1-80d8-49936f2c3e7d",
    "data": { 
      "trackType": "Video",
      "trackName": "Video",
      "bitrate": 300000,
      "timestamp": 36656620000,
      "timescale": 10000000,
      "resultCode": "FragmentDrop_OverlapTimestamp"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Следующий пример демонстрирует схему события **LiveEventIncomingDataChunkDropped**: 

```json
[
  {
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "type": "Microsoft.Media.LiveEventIncomingDataChunkDropped",
    "time": "2018-01-16T01:57:26.005121Z",
    "id": "03da9c10-fde7-48e1-80d8-49936f2c3e7d",
    "data": { 
      "trackType": "Video",
      "trackName": "Video",
      "bitrate": 300000,
      "timestamp": 36656620000,
      "timescale": 10000000,
      "resultCode": "FragmentDrop_OverlapTimestamp"
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `trackType` | строка | Тип дорожки (аудио/видео). |
| `trackName` | строка | Имя дорожки. |
| `bitrate` | Целое число | Скорость дорожки. |
| `timestamp` | строка | Удалена метка времени блока данных. |
| `timescale` | строка | Шкала времени метки времени. |
| `resultCode` | строка | Причина удаления блока данных. **FragmentDrop_OverlapTimestamp** или **FragmentDrop_NonIncreasingTimestamp**. |

### <a name="liveeventincomingstreamreceived"></a>LiveEventIncomingStreamReceived;

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Следующий пример демонстрирует схему события **LiveEventIncomingStreamReceived**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIncomingStreamReceived",
    "eventTime": "2018-08-07T23:08:10.5069288Z",
    "id": "7f939a08-320c-47e7-8250-43dcfc04ab4d",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml/Streams(15864-stream0)15864-stream0",
      "trackType": "video",
      "trackName": "video",
      "bitrate": 2962000,
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "timestamp": "15336831655032322",
      "duration": "20000000",
      "timescale": "10000000"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Следующий пример демонстрирует схему события **LiveEventIncomingStreamReceived**: 

```json
[
  {
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "type": "Microsoft.Media.LiveEventIncomingStreamReceived",
    "time": "2018-08-07T23:08:10.5069288Z",
    "id": "7f939a08-320c-47e7-8250-43dcfc04ab4d",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml/Streams(15864-stream0)15864-stream0",
      "trackType": "video",
      "trackName": "video",
      "bitrate": 2962000,
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "timestamp": "15336831655032322",
      "duration": "20000000",
      "timescale": "10000000"
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `trackType` | строка | Тип дорожки (аудио/видео). |
| `trackName` | строка | Название дорожки (предоставляется кодировщиком или, в случае RTMP, создается сервером в формате *TrackType_Bitrate*). |
| `bitrate` | Целое число | Скорость дорожки. |
| `ingestUrl` | строка | URL-адрес приема, предоставленный событием прямой трансляции. |
| `encoderIp` | строка  | IP-адрес кодировщика. |
| `encoderPort` | строка | Порт кодировщика из источника этого потока. |
| `timestamp` | строка | Получена первая метка времени. |
| `timescale` | строка | Шкала времени, в которой представлена метка времени. |

### <a name="liveeventincomingstreamsoutofsync"></a>LiveEventIncomingStreamsOutOfSync;

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Следующий пример демонстрирует схему события **LiveEventIncomingStreamsOutOfSync**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIncomingStreamsOutOfSync",
    "eventTime": "2018-08-10T02:26:20.6269183Z",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "minLastTimestamp": "319996",
      "typeOfStreamWithMinLastTimestamp": "Audio",
      "maxLastTimestamp": "366000",
      "typeOfStreamWithMaxLastTimestamp": "Video",
      "timescaleOfMinLastTimestamp": "10000000", 
      "timescaleOfMaxLastTimestamp": "10000000"       
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Следующий пример демонстрирует схему события **LiveEventIncomingStreamsOutOfSync**: 

```json
[
  {
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "type": "Microsoft.Media.LiveEventIncomingStreamsOutOfSync",
    "time": "2018-08-10T02:26:20.6269183Z",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "minLastTimestamp": "319996",
      "typeOfStreamWithMinLastTimestamp": "Audio",
      "maxLastTimestamp": "366000",
      "typeOfStreamWithMaxLastTimestamp": "Video",
      "timescaleOfMinLastTimestamp": "10000000", 
      "timescaleOfMaxLastTimestamp": "10000000"       
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `minLastTimestamp` | строка | Минимальная из последних меток времени среди всех дорожек (аудио или видео). |
| `typeOfTrackWithMinLastTimestamp` | строка | Тип дорожки (аудио или видео) с минимальной последней меткой времени. |
| `maxLastTimestamp` | строка | Максимальная из всех меток времени среди всех дорожек (аудио или видео). |
| `typeOfTrackWithMaxLastTimestamp` | строка | Тип дорожки (аудио или видео) с максимальной последней меткой времени. |
| `timescaleOfMinLastTimestamp`| строка | Получить шкалу времени, в которой представлена "MinLastTimestamp".|
| `timescaleOfMaxLastTimestamp`| строка | Получить шкалу времени, в которой представлена "MaxLastTimestamp".|

### <a name="liveeventincomingvideostreamsoutofsync"></a>LiveEventIncomingVideoStreamsOutOfSync;

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Следующий пример демонстрирует схему события **LiveEventIncomingVideoStreamsOutOfSync**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/LiveEvent1",
    "eventType": "Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "6dd4d862-d442-40a0-b9f3-fc14bcf6d750",
    "data": {
      "firstTimestamp": "2162058216",
      "firstDuration": "2000",
      "secondTimestamp": "2162057216",
      "secondDuration": "2000",
      "timescale": "10000000"      
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Следующий пример демонстрирует схему события **LiveEventIncomingVideoStreamsOutOfSync**: 

```json
[
  {
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/LiveEvent1",
    "type": "Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync",
    "time": "2018-01-16T01:57:26.005121Z",
    "id": "6dd4d862-d442-40a0-b9f3-fc14bcf6d750",
    "data": {
      "firstTimestamp": "2162058216",
      "firstDuration": "2000",
      "secondTimestamp": "2162057216",
      "secondDuration": "2000",
      "timescale": "10000000"      
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `firstTimestamp` | строка | Метка времени, полученная для одной из дорожек или уровней качества типа видео. |
| `firstDuration` | строка | Длительность блока данных с первой меткой времени. |
| `secondTimestamp` | строка  | Метка времени, полученная для другого уровня дорожки или уровня качества типа видео. |
| `secondDuration` | строка | Длительность блока данных со второй меткой времени. |
| `timescale` | строка | Шкала времени, состоящая из отметок времени и длительности.|

### <a name="liveeventingestheartbeat"></a>LiveEventIngestHeartbeat;

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Следующий пример демонстрирует схему события **LiveEventIngestHeartbeat**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIngestHeartbeat",
    "eventTime": "2018-08-07T23:17:57.4610506",
    "id": "7f450938-491f-41e1-b06f-c6cd3965d786",
    "data": {
      "trackType": "audio",
      "trackName": "audio",
      "bitrate": 160000,
      "incomingBitrate": 155903,
      "lastTimestamp": "15336837535253637",
      "timescale": "10000000",
      "overlapCount": 0,
      "discontinuityCount": 0,
      "nonincreasingCount": 0,
      "unexpectedBitrate": false,
      "state": "Running",
      "healthy": true
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)


Следующий пример демонстрирует схему события **LiveEventIngestHeartbeat**: 

```json
[
  {
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "type": "Microsoft.Media.LiveEventIngestHeartbeat",
    "time": "2018-08-07T23:17:57.4610506",
    "id": "7f450938-491f-41e1-b06f-c6cd3965d786",
    "data": {
      "trackType": "audio",
      "trackName": "audio",
      "bitrate": 160000,
      "incomingBitrate": 155903,
      "lastTimestamp": "15336837535253637",
      "timescale": "10000000",
      "overlapCount": 0,
      "discontinuityCount": 0,
      "nonincreasingCount": 0,
      "unexpectedBitrate": false,
      "state": "Running",
      "healthy": true
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `trackType` | строка | Тип дорожки (аудио/видео). |
| `trackName` | строка | Название дорожки (предоставляется кодировщиком или, в случае RTMP, создается сервером в формате *TrackType_Bitrate*). |
| `bitrate` | Целое число | Скорость дорожки. |
| `incomingBitrate` | Целое число | Расчетная скорость на основе блоков данных, поступающих из кодировщика. |
| `lastTimestamp` | строка | Последняя метка времени, полученная для дорожки за последние 20 секунд. |
| `timescale` | строка | Шкала времени, в которой выражены метки времени. |
| `overlapCount` | Целое число | Количество блоков данных с перекрывающимися метками времени за последние 20 секунд. |
| `discontinuityCount` | Целое число | Количество прерываний, наблюдаемое за последние 20 секунд. |
| `nonIncreasingCount` | Целое число | Количество блоков данных с метками времени в прошлом, полученных за последние 20 секунд. |
| `unexpectedBitrate` | bool | Если ожидаемые и фактические скорости отличаются более чем на допустимый предел за последние 20 секунд. Это верно, только если incomingBitrate> = 2 * bitrate, incomingBitrate <= bitrate/2 ИЛИ IncomingBitrate = 0. |
| `state` | строка | Состояние события прямой трансляции. |
| `healthy` | bool | Указывает состояние работоспособности приема на основе счетчиков и ​​флагов. Работоспособно, если overlapCount = 0 && discontinuityCount = 0 && nonIncreasingCount = 0 && unexpectedBitrate = false. |

### <a name="liveeventtrackdiscontinuitydetected"></a>LiveEventTrackDiscontinuityDetected.

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Следующий пример демонстрирует схему события **LiveEventTrackDiscontinuityDetected**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventTrackDiscontinuityDetected",
    "eventTime": "2018-08-07T23:18:06.1270405Z",
    "id": "5f4c510d-5be7-4bef-baf0-64b828be9c9b",
    "data": {
      "trackName": "video",
      "previousTimestamp": "15336837615032322",
      "trackType": "video",
      "bitrate": 2962000,
      "newTimestamp": "15336837619774273",
      "discontinuityGap": "575284",
      "timescale": "10000000"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Следующий пример демонстрирует схему события **LiveEventTrackDiscontinuityDetected**: 

```json
[
  {
    "source": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "type": "Microsoft.Media.LiveEventTrackDiscontinuityDetected",
    "time": "2018-08-07T23:18:06.1270405Z",
    "id": "5f4c510d-5be7-4bef-baf0-64b828be9c9b",
    "data": {
      "trackName": "video",
      "previousTimestamp": "15336837615032322",
      "trackType": "video",
      "bitrate": 2962000,
      "newTimestamp": "15336837619774273",
      "discontinuityGap": "575284",
      "timescale": "10000000"
    },
    "specversion": "1.0"
  }
]
```

---

Объект данных имеет следующие свойства:

| Свойство | Тип | Описание |
| -------- | ---- | ----------- |
| `trackType` | строка | Тип дорожки (аудио/видео). |
| `trackName` | строка | Название дорожки (предоставляется кодировщиком или, в случае RTMP, создается сервером в формате *TrackType_Bitrate*). |
| `bitrate` | Целое число | Скорость дорожки. |
| `previousTimestamp` | строка | Метка времени предыдущего фрагмента. |
| `newTimestamp` | строка | Метка времени текущего фрагмента. |
| `discontinuityGap` | строка | Разрыв между двумя метками времени выше. |
| `timescale` | строка | Шкала времени, в которой представлены метка времени и разрыв. |

### <a name="common-event-properties"></a>Общие свойства событий

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Событие содержит следующие высокоуровневые данные:

| Свойство. | Тип | Описание |
| -------- | ---- | ----------- |
| `topic` | строка | Раздел "Сетка событий". Это свойство имеет идентификатор ресурса для учетной записи Служб мультимедиа. |
| `subject` | строка | Путь ресурсов для канала Служб мультимедиа в учетной записи Служб мультимедиа. Объединение темы и объекта предоставляет вам идентификатор ресурса для задания. |
| `eventType` | строка | Один из зарегистрированных типов событий для этого источника событий. Пример: Microsoft.Media.JobStateChange. |
| `eventTime` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | object | Данные события Служб мультимедиа. |
| `dataVersion` | строка | Версия схемы для объекта данных. Версию схемы определяет издатель. |
| `metadataVersion` | строка | Версия схемы для метаданных события. Служба "Сетка событий" определяет схему свойств верхнего уровня. Это значение предоставляет служба "Сетка событий". |

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Событие содержит следующие высокоуровневые данные:

| Свойство. | Тип | Описание |
| -------- | ---- | ----------- |
| `source` | строка | Раздел "Сетка событий". Это свойство имеет идентификатор ресурса для учетной записи Служб мультимедиа. |
| `subject` | строка | Путь ресурсов для канала Служб мультимедиа в учетной записи Служб мультимедиа. Объединение темы и объекта предоставляет вам идентификатор ресурса для задания. |
| `type` | строка | Один из зарегистрированных типов событий для этого источника событий. Пример: Microsoft.Media.JobStateChange. |
| `time` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | object | Данные события Служб мультимедиа. |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |


---

## <a name="next-steps"></a>Дальнейшие действия

[Зарегистрируйтесь на получение событий изменения состояния задания](../media-services/latest/monitoring/job-state-events-cli-how-to.md).

## <a name="see-also"></a>См. также раздел

- [Пакет SDK .NET EventGrid, который включает события Служб мультимедиа](https://www.nuget.org/packages/Microsoft.Azure.EventGrid/)
- [Определения событий Служб мультимедиа](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/eventgrid/data-plane/Microsoft.Media/stable/2018-01-01/MediaServices.json)
- [Коды динамических ошибок событий](../media-services/latest/live-event-error-codes.md)
