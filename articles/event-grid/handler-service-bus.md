---
title: Очереди и разделы служебной шины в качестве обработчиков событий для событий службы "Сетка событий Azure"
description: В статье показано, как использовать очереди и разделы служебной шины в качестве обработчиков событий для событий службы "Сетка событий Azure".
ms.topic: conceptual
ms.date: 09/03/2020
ms.openlocfilehash: 12b72420e3475b46a4cd61ce5032b478af740dde
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97399865"
---
# <a name="service-bus-queues-and-topics-as-event-handlers-for-azure-event-grid-events"></a>Очереди и разделы служебной шины в качестве обработчиков событий для событий службы "Сетка событий Azure"
Обработчик событий — это место, куда отправляются события. Обработчик выполняет последующую обработку полученного события. Некоторые службы Azure автоматически настроены для обработки событий. Одна из них — это **Служебная шина Azure**. 

Очередь или раздел служебной шины можно использовать в качестве обработчика событий из службы "Сетка событий". 

## <a name="service-bus-queues"></a>Очереди служебной шины
События в службе "Сетка событий" можно перенаправлять напрямую в очереди служебной шины для буферизации или использования в сценариях управления и контроля в корпоративных приложениях.

На портале Azure при создании подписки на события выберите **Очередь служебной шины** в качестве типа конечной точки, а затем щелкните **Выбрать конечную точку**, чтобы выбрать очередь служебной шины.

### <a name="using-cli-to-add-a-service-bus-queue-handler"></a>Добавление обработчика очереди служебной шины с помощью интерфейса командной строки (CLI)

В следующем примере для Azure CLI выполняется подписка на раздел сетки событий и его подключение к очереди служебной шины:

```azurecli-interactive
az eventgrid event-subscription create \
    --name <my-event-subscription> \
    --source-resource-id /subscriptions/{SubID}/resourceGroups/{RG}/providers/Microsoft.EventGrid/topics/topic1 \
    --endpoint-type servicebusqueue \
    --endpoint /subscriptions/{SubID}/resourceGroups/TestRG/providers/Microsoft.ServiceBus/namespaces/ns1/queues/queue1
```

## <a name="service-bus-topics"></a>Разделы служебной шины

События в службе "Сетка событий" можно перенаправлять непосредственно в разделы служебной шины для обработки системных событий Azure с помощью разделов служебной шины или для сценариев контроля обмена сообщениями и управления им.

На портале Azure при создании подписки на события выберите **Раздел служебной шины** в качестве типа конечной точки, а затем щелкните **Выбрать конечную точку**, чтобы выбрать раздел служебной шины.

### <a name="using-cli-to-add-a-service-bus-topic-handler"></a>Добавление обработчика раздела служебной шины с помощью CLI

Для Azure CLI следующий пример подписывается и подключает раздел сетки событий к разделу служебной шины:

```azurecli-interactive
az eventgrid event-subscription create \
    --name <my-event-subscription> \
    --source-resource-id /subscriptions/{SubID}/resourceGroups/{RG}/providers/Microsoft.EventGrid/topics/topic1 \
    --endpoint-type servicebustopic \
    --endpoint /subscriptions/{SubID}/resourceGroups/TestRG/providers/Microsoft.ServiceBus/namespaces/ns1/topics/topic1
```

[!INCLUDE [event-grid-message-headers](../../includes/event-grid-message-headers.md)]

При отправке события в очередь или раздел служебной шины в виде сообщения с брокером, `messageid` сообщение посредника является внутренним идентификатором системы.

Внутренний идентификатор системы для сообщения будет поддерживаться при повторной доставке события, чтобы избежать дублирования поставок путем включения **обнаружения повторяющихся** данных в сущности служебной шины. Рекомендуется установить длительность поиска повторяющихся данных в сущности служебной шины равной сроку жизни события или максимальному периоду времени между повторами, в зависимости от того, какое из этих значений больше.

## <a name="rest-examples-for-put"></a>Примеры REST (для PUT)

### <a name="service-bus-queue"></a>Очередь служебной шины

```json
{
    "properties": 
    {
        "destination": 
        {
            "endpointType": "ServiceBusQueue",
            "properties": 
            {
                "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.ServiceBus/namespaces/<SERVICE BUS NAMESPACE NAME>/queues/<SERVICE BUS QUEUE NAME>"
            }
        },
        "eventDeliverySchema": "EventGridSchema"
    }
}
```

### <a name="service-bus-queue---delivery-with-managed-identity"></a>Очередь служебной шины — доставка с использованием управляемого удостоверения

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
                "endpointType": "ServiceBusQueue",
                "properties": 
                {
                    "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.ServiceBus/namespaces/<SERVICE BUS NAMESPACE NAME>/queues/<SERVICE BUS QUEUE NAME>"
                }
            }
        },
        "eventDeliverySchema": "EventGridSchema"
    }
}
```

### <a name="service-bus-topic"></a>Раздел служебной шины

```json
{
    "properties": 
    {
        "destination": 
        {
            "endpointType": "ServiceBusTopic",
            "properties": 
            {
                "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.ServiceBus/namespaces/<SERVICE BUS NAMESPACE NAME>/topics/<SERVICE BUS TOPIC NAME>"
            }
        },
        "eventDeliverySchema": "EventGridSchema"
    }
}
```

### <a name="service-bus-topic---delivery-with-managed-identity"></a>Раздел служебной шины — доставка с использованием управляемого удостоверения

```json
{
    "properties": 
    {
        "deliveryWithResourceIdentity": 
        {
            "identity": 
            {
                "type": "SystemAssigned"
            },
            "destination": 
            {
                "endpointType": "ServiceBusTopic",
                "properties": 
                {
                    "resourceId": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.ServiceBus/namespaces/<SERVICE BUS NAMESPACE NAME>/topics/<SERVICE BUS TOPIC NAME>"
                }
            }
        },
        "eventDeliverySchema": "EventGridSchema"
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
См. список поддерживаемых обработчиков событий в статье [Обработчики событий](event-handlers.md). 
