---
title: Добавление пользовательских данных в события в концентраторах событий Azure
description: В этой статье показано, как добавить пользовательские данные в события в концентраторах событий Azure.
ms.topic: how-to
ms.date: 03/19/2021
ms.openlocfilehash: 3362b6ac4b0d624969aa3ba36d2ebc83b8777cf5
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104893481"
---
# <a name="add-custom-data-to-events-in-azure-event-hubs"></a>Добавление пользовательских данных в события в концентраторах событий Azure
Поскольку событие состоит в основном из непрозрачного набора байтов, для потребителей этих событий может быть трудно принимать обоснованные решения о способах их обработки. Чтобы разрешить издателям событий предоставлять более эффективный контекст для потребителей, события могут также содержать пользовательские метаданные в виде набора пар "ключ-значение". Одним из распространенных сценариев включения метаданных является предоставление подсказки о типе данных, содержащихся в событии, чтобы потребители понимали его формат и могли соответствующим образом десериализовать его.

> [!NOTE]
> Эти метаданные не используются службой концентраторов событий или каким-либо образом не имеет смысла. он существует только для координации между издателями и потребителями событий. 

В следующих разделах показано, как добавлять пользовательские данные в события на разных языках программирования. 

## <a name="net"></a>.NET 

```csharp
var eventBody = new BinaryData("Hello, Event Hubs!");
var eventData = new EventData(eventBody);
eventData.Properties.Add("EventType", "com.microsoft.samples.hello-event");
eventData.Properties.Add("priority", 1);
eventData.Properties.Add("score", 9.0);
```

Полный пример кода см. в разделе [Публикация событий с пользовательскими метаданными](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/samples/Sample04_PublishingEvents.md#publishing-events-with-custom-metadata).

## <a name="java"></a>Java

```java
EventData firstEvent = new EventData("EventData Sample 1".getBytes(UTF_8));
firstEvent.getProperties().put("EventType", "com.microsoft.samples.hello-event");
firstEvent.getProperties().put("priority", 1);
firstEvent.getProperties().put("score", 9.0);
```

Полный пример кода см. в разделе [Публикация событий с пользовательскими метаданными](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples/java/com/azure/messaging/eventhubs/PublishEventsWithCustomMetadata.java).


## <a name="python"></a>Python

```python
event_data = EventData('Message with properties')
event_data.properties = {'event-type': 'com.microsoft.samples.hello-event', 'priority': 1, "score": 9.0}
```

Полный пример кода см. в разделе [Отправка пакета данных событий со свойствами](https://github.com/Azure/azure-sdk-for-python/blob/azure-eventhub_5.3.1/sdk/eventhub/azure-eventhub/samples/async_samples/send_async.py).

## <a name="javascript"></a>JavaScript

```javascript
let eventData = { body: "First event", properties: { "event-type": "com.microsoft.samples.hello-event", "priority": 1, "score": 9.0  } };
```


## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими краткими руководствами и примерами. 

- Краткие руководства: [.NET](event-hubs-dotnet-standard-getstarted-send.md), [Java](event-hubs-java-get-started-send.md), [Python](event-hubs-python-get-started-send.md), [JavaScript](event-hubs-node-get-started-send.md)
- Примеры на GitHub: [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs/samples), [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples), [Python](https://github.com/Azure/azure-sdk-for-python/blob/azure-eventhub_5.3.1/sdk/eventhub/azure-eventhub/samples), [JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples/javascript), [TypeScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs/samples/typescript)
