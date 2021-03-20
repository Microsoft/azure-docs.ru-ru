---
title: REST API-IoT Edge службы "Сетка событий Azure" | Документация Майкрософт
description: REST API в сетке событий на IoT Edge.
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.openlocfilehash: 414487d460d897eff787b11915db560706b29eb4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86171760"
---
# <a name="rest-api"></a>REST API
В этой статье описываются интерфейсы API службы "Сетка событий Azure" для IoT Edge

## <a name="common-api-behavior"></a>Общее поведение API

### <a name="base-url"></a>Базовый URL-адрес
Сетка событий на IoT Edge имеет следующие интерфейсы API, предоставляемые через HTTP (порт 5888) и HTTPS (порт 4438).

* Базовый URL-адрес для HTTP: http://eventgridmodule:5888
* Базовый URL-адрес для HTTPS: https://eventgridmodule:4438

### <a name="request-query-string"></a>Запрос строки запроса
Для всех запросов API требуется следующий параметр строки запроса:

```?api-version=2019-01-01-preview```

### <a name="request-content-type"></a>Тип содержимого запроса
Все запросы API должны иметь **тип содержимого**.

В случае с **евентгридсчема** или **кустомсчема** значением Content-Type может быть одно из следующих значений:

```Content-Type: application/json```

```Content-Type: application/json; charset=utf-8```

В случае **CloudEventSchemaV1_0** в структурированном режиме значение Content-Type может иметь одно из следующих значений:

```Content-Type: application/cloudevents+json```
    
```Content-Type: application/cloudevents+json; charset=utf-8```
    
```Content-Type: application/cloudevents-batch+json```
    
```Content-Type: application/cloudevents-batch+json; charset=utf-8```

В случае **CloudEventSchemaV1_0** в двоичном режиме дополнительные сведения см. в [документации](https://github.com/cloudevents/spec/blob/master/http-protocol-binding.md) .

### <a name="error-response"></a>Сообщение об ошибке
Все API-интерфейсы возвращают ошибку со следующими полезными данными:

```json
{
    "error":
    {
        "code": "<HTTP STATUS CODE>",
        "details": 
        {
            "code": "<Detailed Error Code>",
            "message": "..."
        }
    }
}
```

## <a name="manage-topics"></a>Управление разделами

### <a name="put-topic-create--update"></a>Добавить раздел (создать или обновить)

**Запрос**: ``` PUT /topics/<topic_name>?api-version=2019-01-01-preview ```

**Полезные данные**:

```json
    {
        "name": "<topic_name>", // optional, inferred from URL. If specified must match URL topic_name
        "properties":
        {
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0" // optional
        }
    }
```

**Ответ**. HTTP 200

**Полезные данные**:

```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
    "name": "<topic_name>",
    "type": "Microsoft.EventGrid/topics",
    "properties":
    {
        "endpoint": "<get_request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
        "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0" // populated with EventGridSchema if not explicitly specified in PUT request
    }
}
```

### <a name="get-topic"></a>Получить раздел

**Запрос**: ``` GET /topics/<topic_name>?api-version=2019-01-01-preview ```

**Ответ**. HTTP 200

**Полезные данные**:
```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
    "name": "<topic_name>",
    "type": "Microsoft.EventGrid/topics",
    "properties":
    {
        "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
        "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
    }
}
```

### <a name="get-all-topics"></a>Получить все разделы

**Запрос**: ``` GET /topics?api-version=2019-01-01-preview ```

**Ответ**. HTTP 200

**Полезные данные**:
```json
[
    {
        "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
        "name": "<topic_name>",
        "type": "Microsoft.EventGrid/topics",
        "properties":
        {
            "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
        }
    },
    {
        "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>",
        "name": "<topic_name>",
        "type": "Microsoft.EventGrid/topics",
        "properties":
        {
            "endpoint": "<request_base_url>/topics/<topic_name>/events?api-version=2019-01-01-preview",
            "inputSchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0"
        }
    }
]
```

### <a name="delete-topic"></a>Удалить раздел

**Запрос**: ``` DELETE /topics/<topic_name>?api-version=2019-01-01-preview ```

**Ответ**: HTTP 200, пустые полезные данные

## <a name="manage-event-subscriptions"></a>Управление подписками на события
В примерах этого раздела используется `EndpointType=Webhook;` . Примеры JSON для `EndpointType=EdgeHub / EndpointType=EventGrid` приведены в следующем разделе. 

### <a name="put-event-subscription-create--update"></a>Размещение подписки на события (создание или обновление)

**Запрос**: ``` PUT /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Полезные данные**:
```json
{
    "name": "<subscription_name>", // optional, inferred from URL. If specified must match URL subscription_name
    "properties":
    {
        "topicName": "<topic_name>", // optional, inferred from URL. If specified must match URL topic_name
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema | CloudEventSchemaV1_0", // optional
        "retryPolicy":  //optional
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "persistencePolicy": "true",
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // optional
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```

**Ответ**. HTTP 200

**Полезные данные**:

```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>/eventSubscriptions/<subscription_name>",
    "name": "<subscription_name>",
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "properties":
    {
        "topicName": "<topic_name>",
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema  | CloudEventSchemaV1_0", // populated with EventGridSchema if not explicitly specified in PUT request
        "retryPolicy":  // only populated if specified in the PUT request
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // only populated if specified in the PUT request 
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```


### <a name="get-event-subscription"></a>Получить подписку на событие

**Запрос**: ``` GET /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Ответ**. HTTP 200

**Полезные данные**:
```json
{
    "id": "/iotHubs/<iot_hub_name>/devices/<iot_edge_device_id>/modules/<eventgrid_module_name>/topics/<topic_name>/eventSubscriptions/<subscription_name>",
    "name": "<subscription_name>",
    "type": "Microsoft.EventGrid/eventSubscriptions",
    "properties":
    {
        "topicName": "<topic_name>",
        "eventDeliverySchema": "EventGridSchema | CustomEventSchema  | CloudEventSchemaV1_0", // populated with EventGridSchema if not explicitly specified in PUT request
        "retryPolicy":  // only populated if specified in the PUT request
        {
            "eventExpiryInMinutes": 120,
            "maxDeliveryAttempts": 50
        },
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "maxEventsPerBatch": 10, // optional
                "preferredBatchSizeInKilobytes": 1033 // optional
            }
        },
        "filter": // only populated if specified in the PUT request 
        {
            "subjectBeginsWith": "...",
            "subjectEndsWith": "...",
            "isSubjectCaseSensitive": true|false,
            "includedEventTypes": ["...", "..."],
            "advancedFilters":
            [
                {
                    "OperatorType": "BoolEquals",
                    "Key": "...",
                    "Value": "..."
                },
                {
                    "OperatorType": "NumberLessThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThan",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberLessThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberGreaterThanOrEquals",
                    "Key": "...",
                    "Value": <number>
                },
                {
                    "OperatorType": "NumberIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "NumberNotIn",
                    "Key": "...",
                    "Values": [<number>, <number>, <number>]
                },
                {
                    "OperatorType": "StringIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringNotIn",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringBeginsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringEndsWith",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                },
                {
                    "OperatorType": "StringContains",
                    "Key": "...",
                    "Values": ["...", "...", "..."]
                }
            ]
        }
    }
}
```

### <a name="get-event-subscriptions"></a>Получение подписок на события

**Запрос**: ``` GET /topics/<topic_name>/eventSubscriptions?api-version=2019-01-01-preview ```

**Ответ**. HTTP 200

**Полезные данные**:
```json
[
    {
        // same event-subscription json as that returned from Get-EventSubscription above
    },
    {
    },
    ...
]
```

### <a name="delete-event-subscription"></a>Удаление подписки на события

**Запрос**: ``` DELETE /topics/<topic_name>/eventSubscriptions/<subscription_name>?api-version=2019-01-01-preview ```

**Ответ**: HTTP 200, без полезных данных


## <a name="publish-events-api"></a>API публикации событий

### <a name="send-batch-of-events-in-event-grid-schema"></a>Отправка пакета событий (в схеме сетки событий)

**Запрос**: ``` POST /topics/<topic_name>/events?api-version=2019-01-01-preview ```

```json
[
    {
        "id": "<user-defined-event-id>",
        "topic": "<topic_name>",
        "subject": "",
        "eventType": "",
        "eventTime": ""
        "dataVersion": "",
        "metadataVersion": "1",
        "data": 
            ...
    }
]
```

**Ответ**: HTTP 200, пустые полезные данные


**Описания полей полезных данных**
- ```Id``` является обязательным. Это может быть любое строковое значение, заполняемое вызывающим объектом. Сетка событий не выполняет никаких действий по обнаружению дубликатов и не обеспечивает никакой семантики этого поля.
- ```Topic``` аргумент является необязательным, но если он должен соответствовать topic_name из URL-адреса запроса
- ```Subject``` является обязательным, может быть любым строковым значением
- ```EventType``` является обязательным, может быть любым строковым значением
- ```EventTime``` является обязательным, он не проверяется, но должен быть правильным значением DateTime.
- ```DataVersion``` является обязательным параметром.
- ```MetadataVersion``` аргумент является необязательным, если он указан, он должен быть строкой со значением ```"1"```
- ```Data``` аргумент является необязательным и может быть любым токеном JSON (число, строка, логическое значение, массив, объект)

### <a name="send-batch-of-events-in-custom-schema"></a>Отправка пакета событий (в пользовательской схеме)

**Запрос**: ``` POST /topics/<topic_name>/events?api-version=2019-01-01-preview ```

```json
[
    {
        ...
    }
]
```

**Ответ**: HTTP 200, пустые полезные данные


**Ограничения полезной нагрузки**
- ДОЛЖЕН быть массивом событий.
- Каждая запись массива должна быть объектом JSON.
- Нет других ограничений (кроме размера полезных данных).

## <a name="examples"></a>Примеры

### <a name="set-up-topic-with-eventgrid-schema"></a>Настройка раздела с помощью схемы EventGrid
Настраивает раздел для запроса событий, публикуемых в **евентгридсчема**.

```json
    {
        "name": "myeventgridtopic",
        "properties":
        {
            "inputSchema": "EventGridSchema"
        }
    }
```

### <a name="set-up-topic-with-custom-schema"></a>Настройка раздела с помощью настраиваемой схемы
Настраивает раздел, требующий публикации событий в `customschema` .

```json
    {
        "name": "mycustomschematopic",
        "properties":
        {
            "inputSchema": "CustomSchema"
        }
    }
```

### <a name="set-up-topic-with-cloud-event-schema"></a>Настройка раздела с помощью схемы облачных событий
Настраивает раздел, требующий публикации событий в `cloudeventschema` .

```json
    {
        "name": "mycloudeventschematopic",
        "properties":
        {
            "inputSchema": "CloudEventSchemaV1_0"
        }
    }
```

### <a name="set-up-webhook-as-destination-events-to-be-delivered-in-eventgridschema"></a>Настройка веб-перехватчика в качестве назначения, событий, которые будут доставляться в евентгридсчема
Используйте этот тип назначения для отправки событий в любой другой модуль (на котором размещена конечная точка HTTP) или в любую конечную точку HTTP-адресации в сети или Интернете.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "WebHook",
            "properties":
            {
                "endpointUrl": "<webhook_url>",
                "eventDeliverySchema": "eventgridschema",
            }
        }
    }
}
```

Ограничения на `endpointUrl` атрибут:
- Он должен иметь значение, отличное от NULL.
- Он должен быть абсолютным URL-адресом.
- Если для параметра outbound__webhook__httpsOnly задано значение true в параметрах Евентгридмодуле, он должен быть только протоколом HTTPS.
- Если outbound__webhook__httpsOnly задано значение false, это может быть HTTP или HTTPS.

Ограничения на `eventDeliverySchema` свойство:
- Он должен соответствовать входной схеме раздела подписки.
- Может принимать значение NULL. По умолчанию используется Входная схема раздела.

### <a name="set-up-iot-edge-as-destination"></a>Настройка IoT Edge в качестве назначения

Используйте это назначение для отправки событий в центр IoT Edge и попадают в подсистему маршрутизации, фильтрации или перенаправления концентратора пограничных устройств.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "EdgeHub",
            "properties":
            {
                "outputName": "<eventgridmodule_output_port_name>"
            }
        }
    }
}
```

### <a name="set-up-event-grid-cloud-as-destination"></a>Настройка облака "Сетка событий" в качестве назначения

Используйте это назначение для отправки событий в службу "Сетка событий" в облаке (Azure). Сначала необходимо настроить пользовательский раздел в облаке, в который должны отправляться события, прежде чем создавать подписку на события на границе.

```json
{
    "properties":
    {
        "destination":
        {
            "endpointType": "EventGrid",
            "properties":
            {
                "endpointUrl": "<eventgrid_user_topic_url>",
                "sasKey": "<user_topic_sas_key>",
                "topicName": "<new value to populate in forwarded EventGridEvent.Topic>" // if not specified, the Topic field on every event gets nulled out before being sent to Azure Event Grid
            }
        }
    }
}
```

EndpointUrl
- Он должен иметь значение, отличное от NULL.
- Он должен быть абсолютным URL-адресом.
- Путь `/api/events` должен быть определен в пути URL-адреса запроса.
- Он должен иметь `api-version=2018-01-01` строку запроса.
- Если для параметра outbound__eventgrid__httpsOnly задано значение true в параметрах Евентгридмодуле (по умолчанию true), то он должен быть только протоколом HTTPS.
- Если outbound__eventgrid__httpsOnly имеет значение false, это может быть HTTP или HTTPS.
- Если параметру outbound__eventgrid__allowInvalidHostnames присвоено значение false (по умолчанию false), то она должна ориентироваться на одну из следующих конечных точек:
   - `eventgrid.azure.net`
   - `eventgrid.azure.us`
   - `eventgrid.azure.cn`

Саскэй:
- Значение должно быть отлично от NULL.

Топикнаме:
- Если параметр Subscription. Евентделиверисчема имеет значение Евентгридсчема, значение из этого поля помещается в поле раздела каждого события до того, как оно будет переадресовано в сетку событий в облаке.
- Если для свойства Subscription. Евентделиверисчема задано значение Кустомевентсчема, это свойство игнорируется, а полезные данные настраиваемого события пересылаются в том виде, в котором они были получены.

## <a name="set-up-event-hubs-as-a-destination"></a>Настройка концентраторов событий в качестве назначения

Чтобы опубликовать в концентраторе событий, задайте `endpointType` для значение `eventHub` и укажите:

* connectionString: строка подключения для определенного концентратора событий, созданного с помощью политики общего доступа.

    >[!NOTE]
    > Строка подключения должна быть зависящей от сущности. Использование строки подключения к пространству имен не будет работать. Вы можете создать строку подключения для конкретной сущности, перейдя к определенному концентратору событий, который вы хотите опубликовать на портале Azure, и щелкнув **политики общего доступа** , чтобы создать новую коннецектион строку, относящуюся к сущности.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "eventHub",
              "properties": {
                "connectionString": "<your-event-hub-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-service-bus-queues-as-a-destination"></a>Настройка очередей служебной шины в качестве места назначения

Чтобы опубликовать в очереди служебной шины, задайте `endpointType` для значение `serviceBusQueue` и укажите:

* connectionString: строка подключения для конкретной очереди служебной шины, созданной с помощью политики общего доступа.

    >[!NOTE]
    > Строка подключения должна быть зависящей от сущности. Использование строки подключения к пространству имен не будет работать. Создайте строку подключения, относящуюся к конкретной сущности, перейдя к конкретной очереди служебной шины, которую вы хотите опубликовать на портале Azure, и щелкнув **политики общего доступа** , чтобы создать новую коннецектион строку, относящуюся к сущности.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusQueue",
              "properties": {
                "connectionString": "<your-service-bus-queue-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-service-bus-topics-as-a-destination"></a>Настройка разделов служебной шины в качестве места назначения

Чтобы опубликовать данные в разделе служебной шины, задайте `endpointType` для значение `serviceBusTopic` и укажите:

* connectionString: строка подключения для конкретного раздела служебной шины, созданного с помощью политики общего доступа.

    >[!NOTE]
    > Строка подключения должна быть зависящей от сущности. Использование строки подключения к пространству имен не будет работать. Создайте строку подключения, относящуюся к конкретной сущности, перейдя к определенной теме служебной шины, которую вы хотите опубликовать на портале Azure, и щелкнув **политики общего доступа** , чтобы создать новую коннецектион строку, относящуюся к сущности.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "serviceBusTopic",
              "properties": {
                "connectionString": "<your-service-bus-topic-connection-string>"
              }
            }
          }
        }
    ```

## <a name="set-up-storage-queues-as-a-destination"></a>Настройка очередей хранилища в качестве места назначения

Чтобы опубликовать в очереди хранилища, задайте для значение  `endpointType` `storageQueue` и укажите:

* queueName: имя очереди хранилища, в которую выполняется публикация.
* connectionString: строка подключения для учетной записи хранения, в которой находится очередь хранилища.

    >[!NOTE]
    > В случае неспецифичных концентраторов событий, очередей служебной шины и разделов служебной шины строка подключения, используемая для очередей хранилища, не зависит от сущности. Вместо этого он должен иметь строку подключения для учетной записи хранения.

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "storageQueue",
              "properties": {
                "queueName": "<your-storage-queue-name>",
                "connectionString": "<your-storage-account-connection-string>"
              }
            }
          }
        }
    ```