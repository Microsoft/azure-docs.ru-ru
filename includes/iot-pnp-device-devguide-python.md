---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/19/2020
ms.openlocfilehash: f4eb312aff200389f59a3e342305b8eda98f213e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104582825"
---
## <a name="model-id-announcement"></a>Объявление идентификатора модели

Чтобы объявить идентификатор модели, устройство должно включить его в сведения о подключении:

```python
device_client = IoTHubDeviceClient.create_from_symmetric_key(
    symmetric_key=symmetric_key,
    hostname=registration_result.registration_state.assigned_hub,
    device_id=registration_result.registration_state.device_id,
    product_info=model_id,
)
```

> [!TIP]
> Для модулей и IoT Edge используйте `IoTHubModuleClient` вместо `IoTHubDeviceClient` .

> [!TIP]
> Это единственный момент, когда устройство может задать идентификатор модели, оно не может быть обновлено после подключения устройства.

## <a name="dps-payload"></a>Полезные данные DPS

Устройства, использующие [службу подготовки устройств (DPS)](../articles/iot-dps/about-iot-dps.md) , могут включать в себя `modelId` для использования в процессе подготовки с помощью следующих полезных данных JSON.

```json
{
    "modelId" : "dtmi:com:example:Thermostat;1"
}
```

## <a name="implement-telemetry-properties-and-commands"></a>Реализация телеметрии, свойств и команд

Как описано в разделе [Общие сведения о компонентах Самонастраивающийся моделей Интернета вещей](../articles/iot-pnp/concepts-modeling-guide.md), построители устройств должны решить, хотят ли они использовать компоненты для описания своих устройств. При использовании компонентов устройства должны соответствовать правилам, описанным в этом разделе.

### <a name="telemetry"></a>Телеметрия

Компонент по умолчанию не требует какого-либо специального свойства.

При использовании вложенных компонентов устройства должны задать свойство Message с именем компонента:

```python
async def send_telemetry_from_temp_controller(device_client, telemetry_msg, component_name=None):
    msg = Message(json.dumps(telemetry_msg))
    msg.content_encoding = "utf-8"
    msg.content_type = "application/json"
    if component_name:
        msg.custom_properties["$.sub"] = component_name
    await device_client.send_message(msg)
}
```

### <a name="read-only-properties"></a>Свойства только для чтения

Для сообщения свойства из компонента по умолчанию не требуется специальная конструкция:

```python
await device_client.patch_twin_reported_properties({"maxTempSinceLastReboot": 38.7})
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
      "maxTempSinceLastReboot" : 38.7
  }
}
```

При использовании вложенных компонентов свойства должны быть созданы в имени компонента:

```python
inner_dict = {}
inner_dict["targetTemperature"] = 38.7
inner_dict["__t"] = "c"
prop_dict = {}
prop_dict["thermostat1"] = inner_dict

await device_client.patch_twin_reported_properties(prop_dict)
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
    "thermostat1" : {  
      "__t" : "c",  
      "maxTempSinceLastReboot" : 38.7
     }
  }
}
```

### <a name="writable-properties"></a>Доступные для записи свойства

Эти свойства можно задать на устройстве или обновить с помощью решения. Если решение обновляет свойство, клиент получает уведомление в качестве обратного вызова в `IoTHubDeviceClient` или `IoTHubModuleClient` . Чтобы следовать правилам самонастраивающийся IoT, устройство должно информировать службу о том, что свойство было успешно получено.

#### <a name="report-a-writable-property"></a>Сообщить о свойстве, доступном для записи

Когда устройство сообщает о доступном для записи свойстве, оно должно включать `ack` значения, определенные в соглашениях.

Чтобы сообщить свойство, доступное для записи, из компонента по умолчанию:

```python
prop_dict = {}
prop_dict["targetTemperature"] = {
    "ac": 200,
    "ad": "reported default value",
    "av": 0,
    "value": 23.2
}

await device_client.patch_twin_reported_properties(prop_dict)
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
    "targetTemperature": {
      "value": 23.2,
      "ac": 200,
      "av": 0,
      "ad": "reported default value"
    }
  }
}
```

Чтобы сообщить свойство, доступное для записи, из вложенного компонента, двойника должен включать маркер:

```python
inner_dict = {}
inner_dict["targetTemperature"] = {
    "ac": 200,
    "ad": "reported default value",
    "av": 0,
    "value": 23.2
}
inner_dict["__t"] = "c"
prop_dict = {}
prop_dict["thermostat1"] = inner_dict

await device_client.patch_twin_reported_properties(prop_dict)
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
    "thermostat1": {
      "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 0,
          "ad": "complete"
      }
    }
  }
}
```

#### <a name="subscribe-to-desired-property-updates"></a>Подпишитесь на обновления требуемых свойств

Службы могут обновлять требуемые свойства, которые инициируют уведомление на подключенных устройствах. Это уведомление содержит обновленные требуемые свойства, включая номер версии, определяющий обновление. Устройства должны отвечать с тем же `ack` сообщением, что и сообщаемые свойства.

Компонент по умолчанию видит единственное свойство и создает отчет `ack` с полученной версией:

```python
async def execute_property_listener(device_client):
    ignore_keys = ["__t", "$version"]
    while True:
        patch = await device_client.receive_twin_desired_properties_patch()  # blocking call

        version = patch["$version"]
        prop_dict = {}

        for prop_name, prop_value in patch.items():
            if prop_name in ignore_keys:
                continue
            else:
                prop_dict[prop_name] = {
                    "ac": 200,
                    "ad": "Successfully executed patch",
                    "av": version,
                    "value": prop_value,
                }

        await device_client.patch_twin_reported_properties(prop_dict)
```

Двойника устройства показывает свойство в нужных и сообщаемых разделах:

```json
{
  "desired" : {
    "targetTemperature": 23.2,
    "$version" : 3
  },
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Вложенный компонент получает требуемые свойства, заключенные в оболочку с именем компонента, и должен сообщить `ack` сообщаемое свойство:

```python
def create_reported_properties_from_desired(patch):
    ignore_keys = ["__t", "$version"]
    component_prefix = list(patch.keys())[0]
    values = patch[component_prefix]

    version = patch["$version"]
    inner_dict = {}

    for prop_name, prop_value in values.items():
        if prop_name in ignore_keys:
            continue
        else:
            inner_dict["ac"] = 200
            inner_dict["ad"] = "Successfully executed patch"
            inner_dict["av"] = version
            inner_dict["value"] = prop_value
            values[prop_name] = inner_dict

    properties_dict = dict()
    if component_prefix:
        properties_dict[component_prefix] = values
    else:
        properties_dict = values

    return properties_dict

async def execute_property_listener(device_client):
    while True:
        patch = await device_client.receive_twin_desired_properties_patch()  # blocking call
        properties_dict = create_reported_properties_from_desired(patch)

        await device_client.patch_twin_reported_properties(properties_dict)
```

В двойникае устройства для компонентов отображаются нужные и сообщаемые разделы следующим образом:

```json
{
  "desired" : {
    "thermostat1" : {
        "__t" : "c",
        "targetTemperature": 23.2,
    }
    "$version" : 3
  },
  "reported": {
    "thermostat1" : {
        "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

### <a name="commands"></a>Команды

Компонент по умолчанию получает имя команды в том виде, в котором она была вызвана службой.

Вложенный компонент получает имя команды с префиксом имени компонента и `*` разделителя.

```python
command_request = await device_client.receive_method_request("thermostat1*reboot")
```

#### <a name="request-and-response-payloads"></a>Полезные данные запросов и ответов

Команды используют типы для определения полезных данных запроса и ответа. Устройство должно десериализовать входящий входной параметр и сериализовать ответ. В следующем примере показано, как реализовать команду со сложными типами, определенными в полезных данных:

```json
{
  "@type": "Command",
  "name": "getMaxMinReport",
  "displayName": "Get Max-Min report.",
  "description": "This command returns the max, min and average temperature from the specified time to the current time.",
  "request": {
    "name": "since",
    "displayName": "Since",
    "description": "Period to return the max-min report.",
    "schema": "dateTime"
  },
  "response": {
    "name" : "tempReport",
    "displayName": "Temperature Report",
    "schema": {
      "@type": "Object",
      "fields": [
        {
          "name": "maxTemp",
          "displayName": "Max temperature",
          "schema": "double"
        },
        {
          "name": "minTemp",
          "displayName": "Min temperature",
          "schema": "double"
        },
        {
          "name" : "avgTemp",
          "displayName": "Average Temperature",
          "schema": "double"
        },
        {
          "name" : "startTime",
          "displayName": "Start Time",
          "schema": "dateTime"
        },
        {
          "name" : "endTime",
          "displayName": "End Time",
          "schema": "dateTime"
        }
      ]
    }
  }
}
```

В следующих фрагментах кода показано, как устройство реализует это определение команды, включая типы, используемые для включения сериализации и десериализации:

```python
def create_max_min_report_response(values):
    response_dict = {
        "maxTemp": max_temp,
        "minTemp": min_temp,
        "avgTemp": sum(avg_temp_list) / moving_window_size,
        "startTime": (datetime.now() - timedelta(0, moving_window_size * 8)).isoformat(),
        "endTime": datetime.now().isoformat(),
    }
    # serialize response dictionary into a JSON formatted str
    response_payload = json.dumps(response_dict, default=lambda o: o.__dict__, sort_keys=True)
    return response_payload
```

> [!Tip]
> Имена запросов и ответов отсутствуют в сериализованных полезных данных, передаваемых по сети.
