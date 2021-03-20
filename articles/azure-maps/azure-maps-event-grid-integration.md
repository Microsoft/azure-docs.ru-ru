---
title: Реагирование на события Azure Maps c помощью Сетки событий
description: Узнайте, как реагировать на Azure Maps события с использованием геозаборов. Сведения о перенаправлении событий в обработчики событий см. в разделе прослушивание событий Map и использование сетки событий.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/16/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: mvc
ms.openlocfilehash: 44b2cab814d4e51c4eb0c3ce3322b898f5b26414
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92889772"
---
# <a name="react-to-azure-maps-events-by-using-event-grid"></a>Реагирование на события Azure Maps c помощью Сетки событий

Azure Maps интегрируется со службой "Сетка событий Azure", чтобы пользователи могли отправлять уведомления о событиях в другие службы и запускать нисходящие процессы. Цель этой статьи — помочь вам настроить бизнес-приложения для прослушивания Azure Maps событий. Это позволяет пользователям реагировать на критические события в надежном, масштабируемом и безопасном режиме. Например, пользователи могут создавать приложения для обновления базы данных, создавать билеты и предоставлять уведомления по электронной почте при каждом входе устройства в геозоны.

Сетка событий Azure — это полностью управляемая служба маршрутизации событий, использующая модель публикации и подписки. Сетка событий имеет встроенную поддержку служб Azure, таких как [Функции Azure](../azure-functions/functions-overview.md) и [Azure Logic Apps](../azure-functions/functions-overview.md). Она может передавать оповещения о событиях в службы, не относящиеся к Azure, с помощью веб-перехватчиков. Полный список обработчиков событий, которые поддерживает служба "Сетка событий", см. в статье [An introduction to Azure Event Grid](../event-grid/overview.md) (Общие сведения о службе "Сетка событий Azure").


![Функциональная модель Сетки событий Azure](./media/azure-maps-event-grid-integration/azure-event-grid-functional-model.png)


## <a name="azure-maps-events-types"></a>Типы событий Azure Maps

Сетка событий использует [подписки на события](../event-grid/concepts.md#event-subscriptions) для маршрутизации сообщений о событиях подписчикам. Учетная запись Azure Maps выводит следующие типы событий. 

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft.Maps.GeofenceEntered | Вызывается, когда полученные координаты были перемещены извне из заданного геозоны в в пределах |
| Microsoft.Maps.GeofenceExited | Вызывается, когда полученные координаты перемещаются из заданного геозоны в внешний |
| Microsoft.Maps.GeofenceResult | Создается каждый раз, когда запрос геозоны возвращает результат с любым состоянием. |

## <a name="event-schema"></a>Схема событий

В следующем примере показана схема для Жеофенцересулт:

```JSON
{
    "id":"451675de-a67d-4929-876c-5c2bf0b2c000",
    "topic":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Maps/accounts/{accountName}",
    "subject":"/spatial/geofence/udid/{udid}/id/{eventId}",
    "data":{
        "geometries":[
            {
                "deviceId":"device_1",
                "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169",
                "geometryId":"1",
                "distance":999.0,
                "nearestLat":47.609833,
                "nearestLon":-122.148274
            }
        ],
        "expiredGeofenceGeometryId":[
        ],
        "invalidPeriodGeofenceGeometryId":[
        ]
    },
    "eventType":"Microsoft.Maps.GeofenceResult",
    "eventTime":"2018-11-08T00:52:08.0954283Z",
    "metadataVersion":"1",
    "dataVersion":"1.0"
}

```

## <a name="tips-for-consuming-events"></a>Советы по использованию событий

Приложения, которые обрабатывают события геозоны Azure Maps, должны соответствовать нескольким рекомендациям.

* Настройте несколько подписок для маршрутизации событий в один обработчик событий. Не следует предполагать, что события поступают из определенного источника. Всегда проверяйте раздел сообщения, чтобы убедиться, что сообщение пришло от предполагаемого источника.
* Используйте `X-Correlation-id` поле в заголовке ответа, чтобы узнать, не устарели ли ваши сведения об объектах. Сообщения могут поступать не по порядку или с некоторой задержкой.
* Когда запрос GET или POST в API геозоны вызывается с параметром mode, имеющим значение `EnterAndExit` , то событие Enter или Exit создается для каждой геометрии в геозоны, для которой состояние изменилось с предыдущего вызова ГЕОЗОНЫ API.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как использовать геозоны для операций управления на сайте конструкции, см. в статье

> [!div class="nextstepaction"] 
> [Настройка геозоны с использованием Azure Maps](tutorial-geofence.md)