---
title: Использование концентратора событий как обработчика событий Сетки событий Azure
description: Узнайте, как можно использовать концентраторы событий в качестве обработчиков событий Сетки событий Azure.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 446fef6df65f59206519e282c74d59c2ed1bfa9d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96005641"
---
# <a name="event-hub-as-an-event-handler-for-azure-event-grid-events"></a>Использование концентратора событий как обработчика событий Сетки событий Azure
Обработчик событий — это место, куда отправляются события. Обработчик выполняет действие для обработки события. Некоторые службы Azure автоматически настроены для обработки событий. Одна из них — это **Центры событий Azure**. 

**Центры событий** можно использовать, если решение получает больше событий от Сетки событий, чем может обработать. Когда события находятся в концентраторе событий, приложение может обрабатывать такие события по собственному расписанию. Вы можете расширить параметры обработки входящих событий.

## <a name="tutorials"></a>Учебники
См. следующие примеры. 

|Title  |Описание  |
|---------|---------|
| [Краткое руководство. Перенаправление пользовательских событий в Центры событий Azure с помощью Azure CLI](custom-event-to-eventhub.md) | Отправляет пользовательское событие в концентратор событий для обработки приложением. |
| [Шаблон Resource Manager. Создание пользовательского раздела Сетки событий и отправка событий в концентратор событий](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-event-hubs-handler)| Шаблон Resource Manager, который создает подписку для пользовательского раздела. Он отправляет события в Центры событий Azure. |

[!INCLUDE [event-grid-message-headers](../../includes/event-grid-message-headers.md)]


## <a name="rest-examples-for-put"></a>Примеры REST (для PUT)


### <a name="event-hub"></a>Концентратор событий

```json
{
    "properties": 
    {
        "destination": 
        {
            "endpointType": "EventHub",
            "properties": 
            {
                "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventHub/namespaces/<EVENT HUBS NAMESPACE NAME>/eventhubs/<EVENT HUB NAME>"
            }
        },
        "eventDeliverySchema": "EventGridSchema"
    }
}
```

### <a name="event-hub---delivery-with-managed-identity"></a>Концентратор событий — доставка с использованием управляемого удостоверения

```json
{
    "properties": {
        "deliveryWithResourceIdentity": 
        {
            "identity": 
            {
                "type": "SystemAssigned"
            },
            "destination": 
            {
                "endpointType": "EventHub",
                "properties": 
                {
                    "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventHub/namespaces/<EVENT HUBS NAMESPACE NAME>/eventhubs/<EVENT HUB NAME>"
                }
            }
        },
        "eventDeliverySchema": "EventGridSchema"
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
См. список поддерживаемых обработчиков событий в статье [Обработчики событий](event-handlers.md). 
