---
title: Концентраторы событий Azure в качестве источника службы "Сетка событий"
description: Описание свойств для событий концентраторов, используемых со службой &quot;Сетка событий Azure&quot;
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: e9bb4b5a27173181c7295e96a1eb0654a1a929e6
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363515"
---
# <a name="azure-event-hubs-as-an-event-grid-source"></a>Концентратор событий Azure в качестве источника службы "Сетка событий"

В этой статье описаны свойства и схема для событий концентраторов. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](event-schema.md).

## <a name="available-event-types"></a>Доступные типы событий

Тип события **Microsoft.EventHub.CaptureFileCreated** возникает в службе "Центры событий" при создании файла записи.

## <a name="example-event"></a>Пример события

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Этот пример события демонстрирует схему события в Центрах событий, возникающего, когда файл сохраняется при помощи функции Сбора: 

```json
[
    {
        "topic": "/subscriptions/<guid>/resourcegroups/rgDataMigrationSample/providers/Microsoft.EventHub/namespaces/tfdatamigratens",
        "subject": "eventhubs/hubdatamigration",
        "eventType": "Microsoft.EventHub.CaptureFileCreated",
        "eventTime": "2017-08-31T19:12:46.0498024Z",
        "id": "14e87d03-6fbf-4bb2-9a21-92bd1281f247",
        "data": {
            "fileUrl": "https://tf0831datamigrate.blob.core.windows.net/windturbinecapture/tfdatamigratens/hubdatamigration/1/2017/08/31/19/11/45.avro",
            "fileType": "AzureBlockBlob",
            "partitionId": "1",
            "sizeInBytes": 249168,
            "eventCount": 1500,
            "firstSequenceNumber": 2400,
            "lastSequenceNumber": 3899,
            "firstEnqueueTime": "2017-08-31T19:12:14.674Z",
            "lastEnqueueTime": "2017-08-31T19:12:44.309Z"
        },
        "dataVersion": "",
        "metadataVersion": "1"
    }
]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Этот пример события демонстрирует схему события в Центрах событий, возникающего, когда файл сохраняется при помощи функции Сбора: 

```json
[
    {
        "source": "/subscriptions/<guid>/resourcegroups/rgDataMigrationSample/providers/Microsoft.EventHub/namespaces/tfdatamigratens",
        "subject": "eventhubs/hubdatamigration",
        "type": "Microsoft.EventHub.CaptureFileCreated",
        "time": "2017-08-31T19:12:46.0498024Z",
        "id": "14e87d03-6fbf-4bb2-9a21-92bd1281f247",
        "data": {
            "fileUrl": "https://tf0831datamigrate.blob.core.windows.net/windturbinecapture/tfdatamigratens/hubdatamigration/1/2017/08/31/19/11/45.avro",
            "fileType": "AzureBlockBlob",
            "partitionId": "1",
            "sizeInBytes": 249168,
            "eventCount": 1500,
            "firstSequenceNumber": 2400,
            "lastSequenceNumber": 3899,
            "firstEnqueueTime": "2017-08-31T19:12:14.674Z",
            "lastEnqueueTime": "2017-08-31T19:12:44.309Z"
        },
        "specversion": "1.0"
    }
]
```


---


## <a name="event-properties"></a>Свойства события

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)
Событие содержит следующие высокоуровневые данные:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `topic` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `eventType` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `eventTime` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | object | Данные события концентратора. |
| `dataVersion` | строка | Версия схемы для объекта данных. Версию схемы определяет издатель. |
| `metadataVersion` | строка | Версия схемы для метаданных события. Служба "Сетка событий" определяет схему свойств верхнего уровня. Это значение предоставляет служба "Сетка событий". |

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Событие содержит следующие высокоуровневые данные:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `source` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `type` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `time` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | object | Данные события концентратора. |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |

---

Объект данных имеет следующие свойства:

| Свойство | Type | Описание |
| -------- | ---- | ----------- |
| `fileUrl` | строка | Путь к файлу записи. |
| `fileType` | строка | Тип файла записи. |
| `partitionId` | строка | Идентификатор сегмента. |
| `sizeInBytes` | Целое число | Размер файла. |
| `eventCount` | Целое число | Число событий в файле. |
| `firstSequenceNumber` | Целое число | Наименьший порядковый номер в очереди. |
| `lastSequenceNumber` | Целое число | Последний порядковый номер в очереди. |
| `firstEnqueueTime` | строка | Первые данные времени в очереди. |
| `lastEnqueueTime` | строка | Последние данные времени в очереди. |

## <a name="tutorials-and-how-tos"></a>Руководства и инструкции

|Заголовок  |Описание  |
|---------|---------|
| [Потоковая передача больших данных в хранилище данных](event-grid-event-hubs-integration.md) | Когда Центры событий создают файл сбора, служба "Сетка событий" отправляет событие приложению-функции. Это приложение извлекает файл сбора и переносит данные в хранилище данных. |

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).
* Сведения об обработке событий в Центрах событий см. в статье [Потоковая передача больших данных в хранилище данных](event-grid-event-hubs-integration.md).