---
title: Кэш Azure для Redis в качестве источника службы "Сетка событий"
description: Описание свойств кэша Azure для событий Redis с помощью службы "Сетка событий Azure"
ms.topic: conceptual
ms.date: 02/11/2021
author: curib
ms.author: cauribeg
ms.openlocfilehash: 1a2995bc9ef40cd4eab320ce1bb4c5faf61e0e6e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100371283"
---
# <a name="azure-cache-for-redis-as-an-event-grid-source"></a>Кэш Azure для Redis в качестве источника службы "Сетка событий"

В этой статье приводятся свойства и схема для событий Redis в кэше Azure. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](event-schema.md). 

## <a name="available-event-types"></a>Доступные типы событий
Эти события активируются при экспорте, импорте или масштабировании клиента путем вызова Azure Cache для интерфейсов API Redis. Событие исправления инициируется обновлением Redis.

 |Имя события |Описание|
 |----------|-----------|
 |**Microsoft. Cache. Експортрдбкомплетед** |Активируется при экспорте данных кэша. |
 |**Microsoft. Cache. Импортрдбкомплетед** |Активируется при импорте данных кэша. |
 |**Microsoft. Cache. Патчингкомплетед** |Активируется после завершения установки исправлений. |
 |**Microsoft. Cache. Скалингкомплетед** |Активируется при завершении масштабирования. |

## <a name="example-event"></a>Пример события
При активации события служба Сетки событий отправляет данные о нем на подписанную конечную точку. В этом разделе содержится пример того, как будут выглядеть данные для каждого события кэша Azure для Redis.

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

### <a name="microsoftcachepatchingcompleted-event"></a>Событие Microsoft. Cache. Патчингкомплетед

```json
[{
"id":"9b87886d-21a5-4af5-8e3e-10c4b8dac73b",
"eventType":"Microsoft.Cache.PatchingCompleted",
"topic":"/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
"data":{
    "name":"PatchingCompleted",
    "timestamp":"2020-12-09T21:50:19.9995668+00:00",
    "status":"Succeeded"},
"subject":"PatchingCompleted",
"dataversion":"1.0",
"metadataVersion":"1",
"eventTime":"2020-12-09T21:50:19.9995668+00:00"}]
```

### <a name="microsoftcacheimportrdbcompleted-event"></a>Событие Microsoft. Cache. Импортрдбкомплетед

```json
[{
"id":"9b87886d-21a5-4af5-8e3e-10c4b8dac73b",
"eventType":"Microsoft.Cache.ImportRDBCompleted",
"topic":"/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
"data":{
    "name":"ImportRDBCompleted",
    "timestamp":"2020-12-09T21:50:19.9995668+00:00",
    "status":"Succeeded"},
"subject":"ImportRDBCompleted",
"dataversion":"1.0",
"metadataVersion":"1",
"eventTime":"2020-12-09T21:50:19.9995668+00:00"}]
```

### <a name="microsoftcacheexportrdbcompleted-event"></a>Событие Microsoft. Cache. Експортрдбкомплетед

```json
[{
"id":"9b87886d-21a5-4af5-8e3e-10c4b8dac73b",
"eventType":"Microsoft.Cache.ExportRDBCompleted",
"topic":"/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
"data":{
    "name":"ExportRDBCompleted",
    "timestamp":"2020-12-09T21:50:19.9995668+00:00",
    "status":"Succeeded"},
"subject":"ExportRDBCompleted",
"dataversion":"1.0",
"metadataVersion":"1",
"eventTime":"2020-12-09T21:50:19.9995668+00:00"}]
```

### <a name="microsoftcachescalingcompleted"></a>Microsoft. Cache. Скалингкомплетед

```json
[{
"id":"9b87886d-21a5-4af5-8e3e-10c4b8dac73b",
"eventType":"Microsoft.Cache.ScalingCompleted",
"topic":"/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
"data":{
    "name":"ScalingCompleted",
    "timestamp":"2020-12-09T21:50:19.9995668+00:00",
    "status":"Succeeded"},
"subject":"ScalingCompleted",
"dataversion":"1.0",
"metadataVersion":"1",
"eventTime":"2020-12-09T21:50:19.9995668+00:00"}]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)


### <a name="microsoftcachepatchingcompleted-event"></a>Событие Microsoft. Cache. Патчингкомплетед

```json
[{
    "id": "9b87886d-21a5-4af5-8e3e-10c4b8dac73b",
    "type": "Microsoft.Cache.PatchingCompleted",
    "source": "/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
    "data": {
        "name": "PatchingCompleted",
        "timestamp": "2020-12-09T21:50:19.9995668+00:00",
        "status": "Succeeded"
    },
    "subject": "PatchingCompleted",
    "time": "2020-12-09T21:50:19.9995668+00:00",
    "specversion": "1.0"
}]
```

### <a name="microsoftcacheimportrdbcompleted-event"></a>Событие Microsoft. Cache. Импортрдбкомплетед

```json
[{
    "id": "9b87886d-21a5-4af5-8e3e-10c4b8dac73b",
    "type": "Microsoft.Cache.ImportRDBCompleted",
    "source": "/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
    "data": {
        "name": "ImportRDBCompleted",
        "timestamp": "2020-12-09T21:50:19.9995668+00:00",
        "status": "Succeeded"
    },
    "subject": "ImportRDBCompleted",
    "eventTime": "2020-12-09T21:50:19.9995668+00:00",
    "specversion": "1.0"
}]
```

### <a name="microsoftcacheexportrdbcompleted-event"></a>Событие Microsoft. Cache. Експортрдбкомплетед

```json
[{
    "id": "9b87886d-21a5-4af5-8e3e-10c4b8dac73b",
    "type": "Microsoft.Cache.ExportRDBCompleted",
    "source": "/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
    "data": {
        "name": "ExportRDBCompleted",
        "timestamp": "2020-12-09T21:50:19.9995668+00:00",
        "status": "Succeeded"
    },
    "subject": "ExportRDBCompleted",
    "time": "2020-12-09T21:50:19.9995668+00:00",
    "specversion": "1.0"
}]
```

### <a name="microsoftcachescalingcompleted"></a>Microsoft. Cache. Скалингкомплетед

```json
[{
    "id": "9b87886d-21a5-4af5-8e3e-10c4b8dac73b",
    "type": "Microsoft.Cache.ScalingCompleted",
    "source": "/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Cache/Redis/{cache_name}",
    "data": {
        "name": "ScalingCompleted",
        "timestamp": "2020-12-09T21:50:19.9995668+00:00",
        "status": "Succeeded"
    },
    "subject": "ScalingCompleted",
    "time": "2020-12-09T21:50:19.9995668+00:00",
    "specversion": "1.0"
}]
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
| `data` | object | Кэш Azure для данных событий Redis. |
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
| `data` | object | Кэш Azure для данных событий Redis. |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |

---


Объект данных имеет следующие свойства:

| Свойство | Type | Описание |
| -------- | ---- | ----------- |
| `timestamp` | строка | Время возникновения события. |
| `name` | строка | Имя события. |
| `status` | строка | Состояние события. Сбой или успешно выполнен. |

## <a name="quickstarts"></a>Краткие руководства

Если вы хотите попробовать использовать кэш Azure для событий Redis, см. любую из следующих кратких руководств:

|Если вы хотите использовать это средство:    |См. следующую статью: |
|--|-|
|Портал Azure    |[Краткое руководство. Маршрутизация событий Кэша Azure для Redis в конечную веб-точку с помощью портала Azure](../azure-cache-for-redis/cache-event-grid-quickstart-portal.md)|
|PowerShell    |[Краткое руководство. Маршрутизация событий Кэша Azure для Redis в конечную веб-точку с помощью PowerShell](../azure-cache-for-redis/cache-event-grid-quickstart-powershell.md)|
|Azure CLI    |[Краткое руководство. Маршрутизация событий Кэша Azure для Redis в конечную веб-точку с помощью Azure CLI](../azure-cache-for-redis/cache-event-grid-quickstart-cli.md)|

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).

