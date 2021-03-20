---
title: Сетка событий Azure — журналы диагностики для разделов или доменов
description: Эта статья содержит общие сведения о журналах диагностики для статьи или домена службы "Сетка событий Azure".
ms.topic: conceptual
ms.date: 12/03/2020
ms.openlocfilehash: 36ade14932b5d25c7a1fe05632da671de68ba3bb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96574992"
---
#  <a name="diagnostic-logs-for-azure-event-grid-topicsdomains"></a>Журналы диагностики для разделов и доменов службы "Сетка событий Azure"
Параметры диагностики позволяют пользователям службы "Сетка событий" записывать и просматривать журналы **сбоев публикации и доставки** в учетной записи хранения, концентраторе событий или рабочей области log Analytics. Эта статья содержит схему журналов и пример записи в журнале.


## <a name="schema-for-publishdelivery-failure-logs"></a>Схема журналов сбоев публикации/доставки

| Имя свойства | Тип данных | Описание |
| ------------- | --------- | ----------- | 
| Time | Дата/время | Время создания записи журнала <p>**Пример значения:**  01-29-2020 09:52:02.700</p> |
| евентсубскриптионнаме | Строка | Имя подписки на события <p>**Пример значения:** "EVENTSUB1"</p> <p>Это свойство существует только для журналов сбоев доставки.</p>  |
| Category | Строка | Имя категории журнала. <p>**Примеры значений:** "Деливерифаилурес" или "Публишфаилурес" | 
| OperationName | Строка | Имя операции привело к сбою.<p>**Примеры значений:** "Доставка" для сбоев доставки. |
| Сообщение | Строка | Сообщение журнала для пользователя, объясняющее причину сбоя, и другие дополнительные сведения. |
| ResourceId | Строка | Идентификатор ресурса для раздела или ресурса домена<p>**Примеры значений:**`/SUBSCRIPTIONS/SAMPLE-SUBSCRIPTION-ID/RESOURCEGROUPS/SAMPLE-RESOURCEGROUP/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/TOPIC1` |

## <a name="example"></a>Пример

```json
{
    "time": "2019-11-01T00:17:13.4389048Z",
    "resourceId": "/SUBSCRIPTIONS/SAMPLE-SUBSCTIPTION-ID /RESOURCEGROUPS/SAMPLE-RESOURCEGROUP-NAME/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/SAMPLE-TOPIC-NAME ",
    "eventSubscriptionName": "SAMPLEDESTINATION",
    "category": "DeliveryFailures",
    "operationName": "Deliver",
    "message": "Message:outcome=NotFound, latencyInMs=2635, id=xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx, systemId=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx, state=FilteredFailingDelivery, deliveryTime=11/1/2019 12:17:10 AM, deliveryCount=0, probationCount=0, deliverySchema=EventGridEvent, eventSubscriptionDeliverySchema=EventGridEvent, fields=InputEvent, EventSubscriptionId, DeliveryTime, State, Id, DeliverySchema, LastDeliveryAttemptTime, SystemId, fieldCount=, requestExpiration=1/1/0001 12:00:00 AM, delivered=False publishTime=11/1/2019 12:17:10 AM, eventTime=11/1/2019 12:17:09 AM, eventType=Type, deliveryTime=11/1/2019 12:17:10 AM, filteringState=FilteredWithRpc, inputSchema=EventGridEvent, publisher=DIAGNOSTICLOGSTEST-EASTUS.EASTUS-1.EVENTGRID.AZURE.NET, size=363, fields=Id, PublishTime, SerializedBody, EventType, Topic, Subject, FilteringHashCode, SystemId, Publisher, FilteringTopic, TopicCategory, DataVersion, MetadataVersion, InputSchema, EventTime, fieldCount=15, url=sb://diagnosticlogstesting-eastus.servicebus.windows.net/, deliveryResponse=NotFound: The messaging entity 'sb://diagnosticlogstesting-eastus.servicebus.windows.net/eh-diagnosticlogstest' could not be found. TrackingId:c98c5af6-11f0-400b-8f56-c605662fb849_G14, SystemTracker:diagnosticlogstesting-eastus.servicebus.windows.net:eh-diagnosticlogstest, Timestamp:2019-11-01T00:17:13, referenceId: ac141738a9a54451b12b4cc31a10dedc_G14:"
}
```

## <a name="next-steps"></a>Дальнейшие действия
Сведения о том, как включить журналы диагностики для разделов или доменов, см. в разделе [Включение журналов диагностики](enable-diagnostic-logs-topic.md).
