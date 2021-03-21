---
title: Azure SignalR в качестве источника службы "Сетка событий"
description: Описание свойств, предоставленных для событий Azure SignalR с помощью службы "Сетка событий Azure"
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: 321dc4d21485af23a9cca0d42d74da0a3e121b7c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363700"
---
# <a name="azure-event-grid-event-schema-for-signalr-service"></a>Схема событий сетки событий Azure для службы SignalR

В этой статье приводятся свойства и схема событий службы SignalR. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](event-schema.md). Здесь также приводится список быстрых запусков и руководств по использованию Azure SignalR в качестве источника событий.


## <a name="available-event-types"></a>Доступные типы событий

Служба SignalR выдает следующие типы событий:

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft. Сигналрсервице. Клиентконнектионконнектед | Возникает, когда соединение с клиентом установлено. |
| Microsoft. Сигналрсервице. Клиентконнектиондисконнектед | Возникает, когда клиентское соединение разорвано. |

## <a name="example-event"></a>Пример события

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)
В следующем примере показана схема события подключения клиента: 

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/signalr-rg/providers/Microsoft.SignalRService/SignalR/signalr-resource",
  "subject": "/hub/chat",
  "eventType": "Microsoft.SignalRService.ClientConnectionConnected",
  "eventTime": "2019-06-10T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "timestamp": "2019-06-10T18:41:00.9584103Z",
    "hubName": "chat",
    "connectionId": "crH0uxVSvP61p5wkFY1x1A",
    "userId": "user-eymwyo23"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

Схема для события отключенного подключения клиента аналогична: 

```json
[{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/signalr-rg/providers/Microsoft.SignalRService/SignalR/signalr-resource",
  "subject": "/hub/chat",
  "eventType": "Microsoft.SignalRService.ClientConnectionDisconnected",
  "eventTime": "2019-06-10T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "timestamp": "2019-06-10T18:41:00.9584103Z",
    "hubName": "chat",
    "connectionId": "crH0uxVSvP61p5wkFY1x1A",
    "userId": "user-eymwyo23",
    "errorMessage": "Internal server error."
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

В следующем примере показана схема события подключения клиента: 

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/signalr-rg/providers/Microsoft.SignalRService/SignalR/signalr-resource",
  "subject": "/hub/chat",
  "type": "Microsoft.SignalRService.ClientConnectionConnected",
  "time": "2019-06-10T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "timestamp": "2019-06-10T18:41:00.9584103Z",
    "hubName": "chat",
    "connectionId": "crH0uxVSvP61p5wkFY1x1A",
    "userId": "user-eymwyo23"
  },
  "specversion": "1.0"
}]
```

Схема для события отключенного подключения клиента аналогична: 

```json
[{
  "source": "/subscriptions/{subscription-id}/resourceGroups/signalr-rg/providers/Microsoft.SignalRService/SignalR/signalr-resource",
  "subject": "/hub/chat",
  "type": "Microsoft.SignalRService.ClientConnectionDisconnected",
  "time": "2019-06-10T18:41:00.9584103Z",
  "id": "831e1650-001e-001b-66ab-eeb76e069631",
  "data": {
    "timestamp": "2019-06-10T18:41:00.9584103Z",
    "hubName": "chat",
    "connectionId": "crH0uxVSvP61p5wkFY1x1A",
    "userId": "user-eymwyo23",
    "errorMessage": "Internal server error."
  },
  "specversion": "1.0"
}]
```

---


### <a name="event-properties"></a>Свойства события


# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)
Событие содержит следующие высокоуровневые данные:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `topic` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `eventType` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `eventTime` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `id` | строка | Уникальный идентификатор события. |
| `data` | object | Данные события службы SignalR. |
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
| `data` | object | Данные события службы SignalR. |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |

---

Объект данных имеет следующие свойства:

| Свойство | Type | Описание |
| -------- | ---- | ----------- |
| `timestamp` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `hubName` | строка | Концентратор, к которому принадлежит клиентское соединение. |
| `connectionId` | строка | Уникальный идентификатор клиентского соединения. |
| `userId` | строка | Идентификатор пользователя, определенный в утверждении. |
| `errorMessage` | строка | Ошибка, вызывающая отключение подключения. |

## <a name="tutorials-and-how-tos"></a>Руководства и инструкции
|Заголовок | Описание |
|---------|---------|
| [Реагирование на события службы SignalR Azure с помощью сетки событий](../azure-signalr/signalr-concept-event-grid-integration.md) | Общие сведения об интеграции службы Azure SignalR со службой "Сетка событий". |
| [Отправка событий службы Azure SignalR в службу "Сетка событий"](../azure-signalr/signalr-howto-event-grid-integration.md) | Показано, как отправить события службы Azure SignalR в приложение через службу "Сетка событий". |

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).
