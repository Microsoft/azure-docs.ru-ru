---
title: Центр Интернета вещей Azure как источник службы "Сетка событий"
description: В этой статье описаны свойства и схема для событий Центра Интернета вещей Azure. В нем перечислены доступные типы событий, пример события и свойства событий.
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: 5f43b9d0041fa5842bc2557a61c5145ce588758a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363532"
---
# <a name="azure-iot-hub-as-an-event-grid-source"></a>Центр Интернета вещей Azure как источник службы "Сетка событий"
В этой статье описаны свойства и схема для событий Центра Интернета вещей Azure. Общие сведения о схемах событий см. в статье [Схема событий службы "Сетка событий Azure"](event-schema.md). 

## <a name="available-event-types"></a>Доступные типы событий

Центр Интернета вещей выдает следующие типы событий:

| Тип события | Описание |
| ---------- | ----------- |
| Microsoft.Devices.DeviceCreated | Публикуется при регистрации устройства в Центре Интернета вещей. |
| Microsoft.Devices.DeviceDeleted | Публикуется при удалении устройства из Центра Интернета вещей. | 
| Microsoft.Devices.DeviceConnected | Публикуется при подключении устройства к Центру Интернета вещей. |
| Microsoft.Devices.DeviceDisconnected | Публикуется при отключении устройства от Центра Интернета вещей. | 
| Microsoft.Devices.DeviceTelemetry | Публикуется при отправке сообщения телеметрии в центр Интернета вещей. |

## <a name="example-event"></a>Пример события

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Структура схемы для событий DeviceConnected и DeviceDisconnected идентична. Этот пример демонстрирует схему события, возникающего при подключении устройства в Центре Интернета вещей:

```json
[{
  "id": "f6bbf8f4-d365-520d-a878-17bf7238abd8", 
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>", 
  "subject": "devices/LogicAppTestDevice", 
  "eventType": "Microsoft.Devices.DeviceConnected", 
  "eventTime": "2018-06-02T19:17:44.4383997Z", 
  "data": {
    "deviceConnectionStateEventInfo": {
      "sequenceNumber":
        "000000000000000001D4132452F67CE200000002000000000000000000000001"
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice",
    "moduleId" : "DeviceModuleID"
  }, 
  "dataVersion": "1", 
  "metadataVersion": "1" 
}]
```

Событие Девицетелеметри возникает при отправке события телеметрии в центр Интернета вещей. Пример схемы для этого события приведен ниже.

```json
[{
  "id": "9af86784-8d40-fe2g-8b2a-bab65e106785",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>", 
  "subject": "devices/LogicAppTestDevice", 
  "eventType": "Microsoft.Devices.DeviceTelemetry",
  "eventTime": "2019-01-07T20:58:30.48Z",
  "data": {        
      "body": {            
          "Weather": {                
              "Temperature": 900            
          },
          "Location": "USA"        
      },
        "properties": {            
          "Status": "Active"        
        },
        "systemProperties": {            
            "iothub-content-type": "application/json",
            "iothub-content-encoding": "utf-8",
            "iothub-connection-device-id": "d1",
            "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
            "iothub-connection-auth-generation-id": "123455432199234570",
            "iothub-enqueuedtime": "2019-01-07T20:58:30.48Z",
            "iothub-message-source": "Telemetry"        
        }    
    },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

Структура схемы для событий DeviceCreated и DeviceDeleted идентична. Этот пример демонстрирует схему события, возникающего при регистрации устройства в Центре Интернета вещей:

```json
[{
  "id": "56afc886-767b-d359-d59e-0da7877166b2",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceCreated",
  "eventTime": "2018-01-02T19:17:44.4383997Z",
  "data": {
    "twin": {
      "deviceId": "LogicAppTestDevice",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag": "null",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00",
      "connectionState": "Disconnected",
      "lastActivityTime": "0001-01-01T00:00:00",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "version": 2,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        },
        "reported": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        }
      }
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Структура схемы для событий DeviceConnected и DeviceDisconnected идентична. Этот пример демонстрирует схему события, возникающего при подключении устройства в Центре Интернета вещей:

```json
[{
  "id": "f6bbf8f4-d365-520d-a878-17bf7238abd8", 
  "source": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>", 
  "subject": "devices/LogicAppTestDevice", 
  "type": "Microsoft.Devices.DeviceConnected", 
  "time": "2018-06-02T19:17:44.4383997Z", 
  "data": {
    "deviceConnectionStateEventInfo": {
      "sequenceNumber":
        "000000000000000001D4132452F67CE200000002000000000000000000000001"
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice",
    "moduleId" : "DeviceModuleID"
  }, 
  "specversion": "1.0"
}]
```

Событие Девицетелеметри возникает при отправке события телеметрии в центр Интернета вещей. Пример схемы для этого события приведен ниже.

```json
[{
  "id": "9af86784-8d40-fe2g-8b2a-bab65e106785",
  "source": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>", 
  "subject": "devices/LogicAppTestDevice", 
  "type": "Microsoft.Devices.DeviceTelemetry",
  "time": "2019-01-07T20:58:30.48Z",
  "data": {        
      "body": {            
          "Weather": {                
              "Temperature": 900            
          },
          "Location": "USA"        
      },
        "properties": {            
          "Status": "Active"        
        },
        "systemProperties": {            
            "iothub-content-type": "application/json",
            "iothub-content-encoding": "utf-8",
            "iothub-connection-device-id": "d1",
            "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
            "iothub-connection-auth-generation-id": "123455432199234570",
            "iothub-enqueuedtime": "2019-01-07T20:58:30.48Z",
            "iothub-message-source": "Telemetry"        
        }    
    },
  "specversion": "1.0"
}]
```

Структура схемы для событий DeviceCreated и DeviceDeleted идентична. Этот пример демонстрирует схему события, возникающего при регистрации устройства в Центре Интернета вещей:

```json
[{
  "id": "56afc886-767b-d359-d59e-0da7877166b2",
  "source": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "type": "Microsoft.Devices.DeviceCreated",
  "time": "2018-01-02T19:17:44.4383997Z",
  "data": {
    "twin": {
      "deviceId": "LogicAppTestDevice",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag": "null",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00",
      "connectionState": "Disconnected",
      "lastActivityTime": "0001-01-01T00:00:00",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "version": 2,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        },
        "reported": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        }
      }
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice"
  },
  "specversion": "1.0"
}]
```

---


### <a name="event-properties"></a>Свойства события

# <a name="event-grid-event-schema"></a>[Схема событий службы "Сетка событий Azure"](#tab/event-grid-event-schema)

Все события содержат одинаковые данные верхнего уровня: 

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `id` | строка | Уникальный идентификатор события. |
| `topic` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `eventType` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `eventTime` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `data` | object | Данные событий в Центре Интернета вещей.  |
| `dataVersion` | строка | Версия схемы для объекта данных. Версию схемы определяет издатель. |
| `metadataVersion` | строка | Версия схемы для метаданных события. Служба "Сетка событий" определяет схему свойств верхнего уровня. Это значение предоставляет служба "Сетка событий". |

# <a name="cloud-event-schema"></a>[Схема событий облака](#tab/cloud-event-schema)

Все события содержат одинаковые данные верхнего уровня: 


| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `id` | строка | Уникальный идентификатор события. |
| `source` | строка | Полный путь к ресурсу для источника событий. Это поле защищено от записи. Это значение предоставляет служба "Сетка событий". |
| `subject` | строка | Определенный издателем путь к субъекту событий. |
| `type` | строка | Один из зарегистрированных типов событий для этого источника событий. |
| `time` | строка | Время создания события с учетом времени поставщика в формате UTC. |
| `data` | object | Данные событий в Центре Интернета вещей.  |
| `specversion` | строка | Версия спецификации схемы Клаудевентс. |

---

Для всех событий Центра Интернета вещей объект данных содержит следующие свойства:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `hubName` | строка | Имя Центра Интернета вещей, в котором было создано или из которого было удалено устройство. |
| `deviceId` | строка | Уникальный идентификатор устройства. Это строка с учетом регистра длиной до 128 знаков, состоящая из букв и цифр в 7-битном формате ASCII, а также следующие специальные символы: `- : . + % _ # * ? ! ( ) , = @ ; $ '`. |

Для всех издателей событий используется различное содержимое объекта данных. 

Для событий **устройство подключено** и **устройство отключено** Центра Интернета вещей объект данных содержит следующие свойства:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `moduleId` | строка | Уникальный идентификатор модуля. Это поле выводится только для устройств модуля. Это строка с учетом регистра длиной до 128 знаков, состоящая из букв и цифр в 7-битном формате ASCII, а также следующие специальные символы: `- : . + % _ # * ? ! ( ) , = @ ; $ '`. |
| `deviceConnectionStateEventInfo` | object | Сведения о событии состояния подключения устройства.
| `sequenceNumber` | строка | Номер, который помогает указать порядок событий подключения или отключения устройства. У последнего события будет порядковый номер, который выше предыдущего. Это число может изменяться более чем на 1, но строго в сторону увеличения. См. [способы использования порядковых номеров](../iot-hub/iot-hub-how-to-order-connection-state-events.md). |

Для события центра Интернета вещей для **телеметрии устройства** объект данных содержит сообщение с устройства в облако в [формате сообщений центра Интернета вещей](../iot-hub/iot-hub-devguide-messages-construct.md) и имеет следующие свойства:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `body` | строка | Содержимое сообщения с устройства. |
| `properties` | строка | Свойства приложения — это определяемые пользователем строки, которые можно добавить в сообщение. Эти поля необязательные. |
| `system properties` | строка | [Системные свойства](../iot-hub/iot-hub-devguide-routing-query-syntax.md#system-properties) помогают определить содержимое и источник сообщений. Сообщение телеметрии устройства должно быть в допустимом формате JSON с типом contentType, равным JSON, а contentEncoding — UTF-8 в свойствах системы сообщений. Если этот параметр не задан, центр Интернета вещей будет записывать сообщения в формате Base 64.  |

Для событий **устройство создано** и **устройство удалено** Центра Интернета вещей объект данных содержит следующие свойства:

| Свойство. | Type | Описание |
| -------- | ---- | ----------- |
| `twin` | object | Сведения о двойникае устройства, которое является облачным представлением метаданных устройства приложения. | 
| `deviceID` | строка | Уникальный идентификатор двойника устройства. | 
| `etag` | строка | Проверяющий элемент управления для обеспечения согласованности обновлений двойника устройства. Для каждого двойника устройства обеспечена уникальность каждого тега etag. |  
| `deviceEtag` | строка | Проверяющий элемент управления для обеспечения согласованности обновлений реестра устройства. Каждый deviceEtag гарантированно будет уникальным для каждого реестра устройства. |
| `status` | строка | Состояние двойника устройства: включен или отключен. | 
| `statusUpdateTime` | строка | Метка времени последнего обновления состояния для двойника устройства в формате ISO8601. |
| `connectionState` | строка | Состояние подключения двойника устройства: подключен или не подключен. | 
| `lastActivityTime` | строка | Метка времени последнего действия в формате ISO8601. | 
| `cloudToDeviceMessageCount` | Целое число | Число сообщений, отправленных из облака на это устройство. | 
| `authenticationType` | строка | Тип аутентификации, используемый для этого устройства: `SAS`, `SelfSigned` или `CertificateAuthority`. |
| `x509Thumbprint` | строка | Отпечаток — это уникальное значение для сертификата x509, которое обычно используется для поиска определенного сертификата в хранилище сертификатов. Отпечаток динамически создается с помощью алгоритма SHA1 и физически не существует в сертификате. | 
| `primaryThumbprint` | строка | Первичный отпечаток для сертификата x509. |
| `secondaryThumbprint` | строка | Вторичный отпечаток для сертификата x509. | 
| `version` | Целое число | Целое число, которое увеличивается на единицу при каждом обновлении двойника устройства. |
| `desired` | object | Часть свойств, которые могут быть записаны только серверной частью приложения и считаны устройством. | 
| `reported` | object | Часть свойств, которые могут быть записаны только устройством и считаны серверной частью приложения. |
| `lastUpdated` | строка | Метка времени последнего обновления свойств для двойника устройства в формате ISO8601. | 

## <a name="tutorials-and-how-tos"></a>Руководства и инструкции
|Заголовок  |Описание  |
|---------|---------|
| [Отправка уведомлений электронной почты о событиях в Центре Интернета вещей Azure с помощью Logic Apps](publish-iot-hub-events-to-logic-apps.md) | Приложение логики отправляет уведомление по электронной почте каждый раз, когда добавляется устройство в Центр Интернета вещей. |
| [Реагирование на события в Центре Интернета вещей с использованием службы "Сетка событий" для запуска действий](../iot-hub/iot-hub-event-grid.md) | Общие сведения об интеграции Центра Интернета вещей со службой "Сетка событий". |
| [Order device connection events from Azure IoT Hub using Azure Cosmos DB](../iot-hub/iot-hub-how-to-order-connection-state-events.md) (Упорядочение событий подключения устройств из Центра Интернета вещей с помощью Azure Cosmos DB) | Сведения об упорядочении событий изменения состояния подключения устройства. |

## <a name="next-steps"></a>Дальнейшие действия

* См. общие сведения о [службе "Сетка событий Azure"](overview.md).
* Дополнительные сведения о совместной работе Центра Интернета вещей и службы "Сетка событий" см. в статье о [реагировании на события Центра Интернета вещей с использованием службы "Сетка событий" для активации действий](../iot-hub/iot-hub-event-grid.md).