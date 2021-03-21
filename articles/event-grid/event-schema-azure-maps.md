---
title: Azure Maps в качестве источника сетки событий
description: Описание свойств и схемы для событий Azure Maps в службе "Сетка событий Azure"
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: 88cf0c8274d685a45862bc7b7884b5e4a686c22d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363690"
---
# <a name="azure-maps-as-an-event-grid-source"></a>Azure Maps в качестве источника сетки событий

В этой статье описаны свойства и схема для событий Azure Maps. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](./event-schema.md). Он также содержит список быстрых запусков и учебников по использованию Azure Maps в качестве источника событий.

## <a name="available-event-types"></a>Доступные типы событий

Учетная запись Azure Maps выводит следующие типы событий.

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft.Maps.GeofenceEntered | Создается, когда полученные координаты переходят из заданной геозоны наружу. |
| Microsoft.Maps.GeofenceExited | Создается, когда полученные координаты переходят из заданной геозоны наружу |
| Microsoft.Maps.GeofenceResult | Создается каждый раз, когда запрос геозоны возвращает результат с любым состоянием. |

## <a name="example-events"></a>Примеры событий

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)
В следующем примере показана схема события **GeofenceEntered**.

```JSON
{   
   "id":"7f8446e2-1ac7-4234-8425-303726ea3981", 
   "topic":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Maps/accounts/{accountName}", 
   "subject":"/spatial/geofence/udid/{udid}/id/{eventId}", 
   "data":{   
      "geometries":[   
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"2", 
            "distance":-999.0, 
            "nearestLat":47.618786, 
            "nearestLon":-122.132151 
         } 
      ], 
      "expiredGeofenceGeometryId":[   
      ], 
      "invalidPeriodGeofenceGeometryId":[   
      ] 
   }, 
   "eventType":"Microsoft.Maps.GeofenceEntered", 
   "eventTime":"2018-11-08T00:54:17.6408601Z", 
   "metadataVersion":"1", 
   "dataVersion":"1.0" 
}
```

В следующем примере показана схема для **GeofenceResult**. 

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
         }, 
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"2", 
            "distance":999.0, 
            "nearestLat":47.621954, 
            "nearestLon":-122.131841 
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

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)
В следующем примере показана схема события **GeofenceEntered**.

```JSON
{   
   "id":"7f8446e2-1ac7-4234-8425-303726ea3981", 
   "source":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Maps/accounts/{accountName}", 
   "subject":"/spatial/geofence/udid/{udid}/id/{eventId}", 
   "data":{   
      "geometries":[   
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"2", 
            "distance":-999.0, 
            "nearestLat":47.618786, 
            "nearestLon":-122.132151 
         } 
      ], 
      "expiredGeofenceGeometryId":[   
      ], 
      "invalidPeriodGeofenceGeometryId":[   
      ] 
   }, 
   "type":"Microsoft.Maps.GeofenceEntered", 
   "time":"2018-11-08T00:54:17.6408601Z", 
   "specversion":"1.0" 
}
```

В следующем примере показана схема для **GeofenceResult**. 

```JSON
{   
   "id":"451675de-a67d-4929-876c-5c2bf0b2c000", 
   "source":"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Maps/accounts/{accountName}", 
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
         }, 
         {   
            "deviceId":"device_1", 
            "udId":"1a13b444-4acf-32ab-ce4e-9ca4af20b169", 
            "geometryId":"2", 
            "distance":999.0, 
            "nearestLat":47.621954, 
            "nearestLon":-122.131841 
         } 
      ], 
      "expiredGeofenceGeometryId":[   
      ], 
      "invalidPeriodGeofenceGeometryId":[   
      ] 
   }, 
   "type":"Microsoft.Maps.GeofenceResult", 
   "time":"2018-11-08T00:52:08.0954283Z", 
   "specversion":"1.0" 
}
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
| `data` | object | Географическое зонирование данных события. |
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
| `data` | object | Географическое зонирование данных события. |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |

---

Объект данных имеет следующие свойства:

| Свойство | Type | Описание |
| -------- | ---- | ----------- |
| `apiCategory` | строка | Категория API для события. |
| `apiName` | строка | Имя API для события. |
| `issues` | object | Выводит список проблем, возникших во время обработки. Если выводится список проблем, в ответе не возвращаются геометрические объекты. |
| `responseCode` | Число | Код ответа HTTP |
| `geometries` | object | Перечисляет геометрические объекты границ, которые содержат координаты позиции или перекрытие searchBuffer вокруг этой позиции. |

Объект error возвращается, если в API Maps произошла ошибка. Этот объект имеет следующие свойства.

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `error` | ErrorDetails |Этот объект возвращается, если в API Maps произошла ошибка.  |

Объект ErrorDetails возвращается, если в API Maps произошла ошибка. Этот объект имеет следующие свойства.

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `code` | строка | Код состояния HTTP. |
| `message` | строка | Понятное текстовое описание ошибки, если оно доступно. |
| `innererror` | InnerError | Объект, со сведениями службы об ошибке, если они доступны. |

Объект InnerError содержит сведения службы об ошибке, если они доступны. Этот объект имеет следующие свойства. 

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `code` | строка | Сообщение об ошибке. |

Объект geometries выводит список идентификаторов геометрических объектов геозоны, срок действия которых истек относительно времени в запросе пользователя. Объект geometries содержит геометрические объекты со следующими свойствами. 

| Свойство. | Type | Описание |
|:-------- |:---- |:----------- |
| `deviceid` | строка | Идентификатор устройства. |
| `distance` | строка | <p>Расстояние от точки с указанными координатами до ближайшей границы геозоны. Положительный результат означает, что координаты выходят за пределы геозоны. Если координаты выходят за пределы геозоны и расстояние до ближайшей границы превышает размер значения searchBuffer, возвращается значение 999. Отрицательный результат означает, что координаты находятся в пределах геозоны. Если координаты находятся в пределах многоугольника и расстояние до ближайшей границы превышает размер значения searchBuffer, возвращается значение –999. Значение 999 означает, что координаты с высоким уровнем надежности расположены за пределами геозоны. Значение –999 означает, что координаты с высоким уровнем надежности расположены в пределах геозоны.<p> |
| `geometryid` |строка | Уникальный идентификатор определяет геометрию геозоны. |
| `nearestlat` | Число | Широта ближайшей точки геометрии. |
| `nearestlon` | Число | Долгота ближайшей точки геометрии. |
| `udId` | строка | Уникальный идентификатор, возвращенный службой отправки пользователей при отправке геозоны. Не будет включаться в API-интерфейс геозоны POST. |

Объект данных имеет следующие свойства:

| Свойство | Type | Описание |
| -------- | ---- | ----------- |
| `expiredGeofenceGeometryId` | string[] | Список идентификаторов геометрических объектов геозоны, срок действия которых истек относительно времени в запросе пользователя. |
| `geometries` | geometries[] |Перечисляет геометрические объекты границ, которые содержат координаты позиции или перекрытие searchBuffer вокруг этой позиции. |
| `invalidPeriodGeofenceGeometryId` | string[]  | Список идентификаторов геометрических объектов геозоны, в период действия которых не попадает время, указанное в запросе пользователя. |
| `isEventPublished` | Логическое | Имеет значение truе, если хотя бы одно событие опубликовано в подписчике событий Azure Maps, или false в противном случае. |

## <a name="tutorials-and-how-tos"></a>Руководства и инструкции
|Заголовок  |Описание  |
|---------|---------|
| [Реагирование на события Azure Maps c помощью Сетки событий](../azure-maps/azure-maps-event-grid-integration.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Общие сведения об интеграции Azure Maps со службой "Сетка событий". |
| [Учебник. Настройка геозоны](../azure-maps/tutorial-geofence.md?toc=%2fazure%2fevent-grid%2ftoc.json) | Это руководство поможет выполнить основные действия по настройке геозоны с помощью Azure Maps. Используйте Сетку событий Azure для выполнения потоковой передачи результатов геозоны и на их основе создавайте уведомления. |

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о создании подписки на Сетку событий Azure см. в статье [Схема подписки для службы "Сетка событий"](subscription-creation-schema.md).
