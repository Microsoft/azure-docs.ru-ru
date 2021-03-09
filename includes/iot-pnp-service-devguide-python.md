---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 10/20/2020
ms.openlocfilehash: a925c3a17988ef6f4b95a1e3cf4dd5fb8baa4829
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102510773"
---
Также доступны следующие ресурсы:

- [Справочная документация по пакету SDK для Python](/python/api/azure-iot-hub/azure.iot.hub)
- [Примеры клиента службы](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-hub/samples/iothub_registry_manager_method_sample.py)
- [Примеры цифровых двойников](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-hub/samples/get_digital_twin_sample.py)

## <a name="iot-hub-service-client-examples"></a>Примеры клиентов службы центра Интернета вещей

В этом разделе показаны примеры Python с использованием клиента службы центра Интернета вещей и классов **иосубрегистриманажер** и **клаудтодевицемесод** . Используйте класс **иосубрегистриманажер** для взаимодействия с состоянием устройства с помощью двойникова устройства. Вы также можете использовать класс **иосубрегистриманажер** для [запроса регистрации устройств](../articles/iot-hub/iot-hub-devguide-query-language.md) в центре Интернета вещей. Для вызова команд на устройстве используется класс **клаудтодевицемесод** . Модель [дтдл](../articles/iot-pnp/concepts-digital-twin.md) для устройства определяет свойства и команды, реализуемые устройством. В фрагментах кода `device_id` переменная содержит идентификатор устройства Самонастраивающийся устройства IOT, зарегистрированного в центре Интернета вещей.

### <a name="get-the-device-twin-and-model-id"></a>Получение двойникаа устройства и идентификатора модели

Чтобы получить двойника устройства и идентификатор модели для устройства IoT самонастраивающийся, подключенного к центру Интернета вещей:

```python
from azure.iot.hub import IoTHubRegistryManager
from azure.iot.hub.models import Twin, TwinProperties

iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

# ...

twin = iothub_registry_manager.get_twin(device_id)
print("The device twin is: ")
print("")
print(twin)
print("")

additional_props = twin.additional_properties
if "modelId" in additional_props:
    print("The Model ID for this device is:")
    print(additional_props["modelId"])
    print("")
```

### <a name="update-device-twin"></a>Обновление двойников устройств

В следующем фрагменте кода показано, как обновить `targetTemperature` свойство на устройстве. В этом примере показано, как получить двойника `etag` перед обновлением. Свойство определено в компоненте по умолчанию устройства:

```python
iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

twin = iothub_registry_manager.get_twin(device_id)

twin_patch = Twin()

twin_patch.properties = TwinProperties(
    desired={"targetTemperature": 42}
)
updated_twin = iothub_registry_manager.update_twin(device_id, twin_patch, twin.etag)
```

В следующем фрагменте кода показано, как обновить `targetTemperature` свойство компонента. В этом примере показано, как получить двойника `ETag` перед обновлением. Свойство определено в компоненте **thermostat1** :

```python
iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

twin = iothub_registry_manager.get_twin(device_id)

twin_patch = Twin()

twin_patch.properties = TwinProperties(
    desired={ "thermostat1": {
        "__t": "c",
        "targetTemperature": 42}
    }
)
updated_twin = iothub_registry_manager.update_twin(device_id, twin_patch, twin.etag)
```

Для свойства в компоненте исправление свойства выглядит как в следующем примере:

```json
{
"thermostat1":
  {
    "__t": "c",
    "targetTemperature": 20
  }
}
```

### <a name="call-command"></a>Call, команда

В следующем фрагменте кода показано, как вызвать `getMaxMinReport` команду, определенную в компоненте по умолчанию.

```python
from azure.iot.hub import IoTHubRegistryManager
from azure.iot.hub.models import CloudToDeviceMethod

# ...

iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

method_payload = datetime.datetime.now() - datetime.timedelta(minutes=2)
device_method = CloudToDeviceMethod(method_name="getMaxMinReport", payload=method_payload)
result = iothub_registry_manager.invoke_device_method(device_id, device_method)
print(result.payload)
```

В следующем фрагменте кода показано, как вызвать `getMaxMinReport` команду для компонента. Команда определена в компоненте **thermostat1** :

```python
from azure.iot.hub import IoTHubRegistryManager
from azure.iot.hub.models import CloudToDeviceMethod

# ...

iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

method_payload = datetime.datetime.now() - datetime.timedelta(minutes=2)
device_method = CloudToDeviceMethod(method_name="thermostat1*getMaxMinReport", payload=method_payload)
result = iothub_registry_manager.invoke_device_method(device_id, device_method)
print(result.payload)
```

## <a name="iot-hub-digital-twin-examples"></a>Примеры цифровых двойника в центре Интернета вещей

Для взаимодействия с состоянием устройства с помощью цифрового двойников используется класс **дигиталтвинклиент** . Модель [дтдл](../articles/iot-pnp/concepts-digital-twin.md) для устройства определяет свойства и команды, реализуемые устройством.

`device_id`Переменная содержит идентификатор устройства Самонастраивающийся устройства IOT, зарегистрированного в центре Интернета вещей.

### <a name="get-the-digital-twin-and-model-id"></a>Получение цифровых двойника и идентификатора модели

Чтобы получить цифровой двойника и идентификатор модели устройства IoT самонастраивающийся, подключенного к центру Интернета вещей:

```python
from azure.iot.hub import DigitalTwinClient

digital_twin_client = DigitalTwinClient(iothub_connection_str)

digital_twin = digital_twin_client.get_digital_twin(device_id)
if digital_twin:
    print(digital_twin)
    print("Model Id: " + digital_twin["$metadata"]["$model"])
else:
    print("No digital_twin found")
```

### <a name="update-digital-twin"></a>Обновление цифровых двойника

В следующем фрагменте кода показано, как обновить `targetTemperature` свойство на устройстве. Свойство определено в компоненте по умолчанию устройства:

```python
from azure.iot.hub import DigitalTwinClient

digital_twin_client = DigitalTwinClient(iothub_connection_str)

patch = [{"op": "add", "path": "/targetTemperature", "value": 42}]
digital_twin_client.update_digital_twin(device_id, patch)
```

В следующем фрагменте кода показано, как обновить `targetTemperature` свойство компонента. Свойство определено в компоненте **thermostat1** :

```python
from azure.iot.hub import DigitalTwinClient

digital_twin_client = DigitalTwinClient(iothub_connection_str)

patch = [{"op": "add", "path": "/targetTemperature", "value": 42}]
digital_twin_client.update_digital_twin(device_id, patch)
```

### <a name="call-command"></a>Call, команда

В следующем фрагменте кода показано, как вызвать `getMaxMinReport` команду, определенную в компоненте по умолчанию.

```python
from azure.iot.hub import DigitalTwinClient

payload = datetime.datetime.now() - datetime.timedelta(minutes=2)

connect_timeout_in_seconds = 3
response_timeout_in_seconds = 7


digital_twin_client = DigitalTwinClient(iothub_connection_str)

invoke_command_result = digital_twin_client.invoke_command(
    device_id, "getMaxMinReport", payload, connect_timeout_in_seconds, response_timeout_in_seconds
)
if invoke_command_result:
    print(invoke_command_result)
else:
    print("No invoke_command_result found")
```

В следующем фрагменте кода показано, как вызвать `getMaxMinReport` команду для компонента. Команда определена в компоненте **thermostat1** :

```python
from azure.iot.hub import DigitalTwinClient

payload = datetime.datetime.now() - datetime.timedelta(minutes=2)

connect_timeout_in_seconds = 3
response_timeout_in_seconds = 7


digital_twin_client = DigitalTwinClient(iothub_connection_str)

invoke_command_result = digital_twin_client.invoke_component_command(
    device_id, "thermostat1", "getMaxMinReport", payload, connect_timeout_in_seconds, response_timeout_in_seconds
)
if invoke_command_result:
    print(invoke_command_result)
else:
    print("No invoke_command_result found")
```

## <a name="read-device-telemetry"></a>Чтение телеметрии устройства

Устройства IoT самонастраивающийся отправляют данные телеметрии, определенные в модели ДТДЛ, в центр Интернета вещей. По умолчанию центр Интернета вещей направляет данные телеметрии в конечную точку концентратора событий, где их можно использовать. Дополнительные сведения см. в статье [Использование маршрутизации сообщений центра Интернета вещей для отправки сообщений с устройства в облако в разные конечные точки](../articles/iot-hub/iot-hub-devguide-messages-d2c.md).

В следующем фрагменте кода показано, как считывать данные телеметрии из конечной точки концентраторов событий по умолчанию. Код в этом фрагменте кода взят из руководства по использованию центра Интернета вещей [Отправка данных телеметрии с устройства в центр Интернета вещей и их чтение с помощью серверного приложения](../articles/iot-hub/quickstart-send-telemetry-python.md):

```python
import asyncio
from azure.eventhub import TransportType
from azure.eventhub.aio import EventHubConsumerClient

# Define callbacks to process events
async def on_event_batch(partition_context, events):
    for event in events:
        print("Received event from partition: {}.".format(partition_context.partition_id))
        print("Telemetry received: ", event.body_as_str())
        print("Properties (set by device): ", event.properties)
        print("System properties (set by IoT Hub): ", event.system_properties)
        print()
    await partition_context.update_checkpoint()

async def on_error(partition_context, error):
    # ...

loop = asyncio.get_event_loop()
client = EventHubConsumerClient.from_connection_string(
    conn_str=CONNECTION_STR,
    consumer_group="$default",
)

try:
    loop.run_until_complete(client.receive_batch(on_event_batch=on_event_batch, on_error=on_error))
except KeyboardInterrupt:
    print("Receiving has stopped.")
finally:
    loop.run_until_complete(client.close())
    loop.stop()
```

В следующих выходных данных из предыдущего кода показаны данные телеметрии температуры, отправленные устройством самонастраивающийся IOT без компонента **термостата** , которое имеет только компонент по умолчанию. В `dt-dataschema` системном свойстве отображается идентификатор модели:

```cmd/sh
Received event from partition: 1.
Telemetry received:  {"temperature": 12}
Properties (set by device):  None
System properties (set by IoT Hub):  {b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-connection-auth-method': b'{"scope":"device","type":"sas","issuer":"iothub","acceptingIpFilterRule":null}', b'iothub-connection-auth-generation-id': b'637388855582764406', b'iothub-enqueuedtime': 1603288810715, b'iothub-message-source': b'Telemetry', b'dt-dataschema': b'dtmi:com:example:Thermostat;1', b'x-opt-sequence-number': 13280, b'x-opt-offset': b'12890070640', b'x-opt-enqueued-time': 1603288810824}
```

В следующих выходных данных из предыдущего кода показаны данные телеметрии температуры, отправленные с помощью многокомпонентного устройства **температуреконтроллер** IOT Самонастраивающийся. `dt-subject`Свойство System показывает имя компонента, который отправил телеметрию. В этом примере два компонента являются `thermostat1` и `thermostat2` определены в модели дтдл. В `dt-dataschema` системном свойстве отображается идентификатор модели:

```cmd/sh
Received event from partition: 1.
Telemetry received:  {"temperature": 45}
Properties (set by device):  None
System properties (set by IoT Hub):  {b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-connection-auth-method': b'{"scope":"device","type":"sas","issuer":"iothub","acceptingIpFilterRule":null}', b'iothub-connection-auth-generation-id': b'637388858939631652', b'iothub-enqueuedtime': 1603289127844, b'iothub-message-source': b'Telemetry', b'dt-subject': b'thermostat1', b'dt-dataschema': b'dtmi:com:example:TemperatureController;1', b'x-opt-sequence-number': 13328, b'x-opt-offset': b'12890095440', b'x-opt-enqueued-time': 1603289128001}

Received event from partition: 1.
Telemetry received:  {"temperature": 49}
Properties (set by device):  None
System properties (set by IoT Hub):  {b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-connection-auth-method': b'{"scope":"device","type":"sas","issuer":"iothub","acceptingIpFilterRule":null}', b'iothub-connection-auth-generation-id': b'637388858939631652', b'iothub-enqueuedtime': 1603289133017, b'iothub-message-source': b'Telemetry', b'dt-subject': b'thermostat2', b'dt-dataschema': b'dtmi:com:example:TemperatureController;1', b'x-opt-sequence-number': 13329, b'x-opt-offset': b'12890095928', b'x-opt-enqueued-time': 1603289133173}
```

## <a name="read-device-twin-change-notifications"></a>Чтение уведомлений об изменениях двойникаов устройств

Вы можете настроить центр Интернета вещей для создания уведомлений об изменениях двойникаов устройств для маршрутизации в поддерживаемую конечную точку. Дополнительные сведения см. в статье [Использование маршрутизации сообщений центра Интернета вещей для отправки сообщений с устройства в облако в разные конечные точки > события, не связанные с телеметрии](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

Код, приведенный в предыдущем фрагменте кода Python, создает следующий результат, когда центр Интернета вещей создает уведомления об изменениях двойникаа устройства для устройства термостата без компонента. Свойства приложения `iothub-message-schema` и `opType` предоставляют сведения о типе уведомления об изменении:

```cmd/sh
Received event from partition: 1.
Telemetry received:  {"version":3,"properties":{"reported":{"maxTempSinceLastReboot":10.96,"$metadata":{"$lastUpdated":"2020-10-21T14:10:42.4171263Z","maxTempSinceLastReboot":{"$lastUpdated":"2020-10-21T14:10:42.4171263Z"}},"$version":2}}}
Properties (set by device):  {b'hubName': b'my-pnp-hub', b'deviceId': b'my-pnp-device', b'operationTimestamp': b'2020-10-21T14:10:42.4171263+00:00', b'iothub-message-schema': b'twinChangeNotification', b'opType': b'updateTwin'}
System properties (set by IoT Hub):  {b'user-id': b'my-pnp-hub\x81\x0e\xa4\x7f', b'correlation-id': b'12104ced5402', b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-enqueuedtime': 1603289442519, b'iothub-message-source': b'twinChangeEvents', b'x-opt-sequence-number': 13332, b'x-opt-offset': b'12890097392', b'x-opt-enqueued-time': 1603289442738}
```

Код, приведенный в предыдущем фрагменте кода Python, создает следующий результат, когда центр Интернета вещей создает уведомления об изменении двойникаа устройства для устройства с компонентами. В этом примере показаны выходные данные, когда устройство датчика температуры с компонентом термостата создает уведомления. Свойства приложения `iothub-message-schema` и `opType` предоставляют сведения о типе уведомления об изменении:

```cmd/sh
Received event from partition: 1.
Telemetry received:  {"version":4,"properties":{"reported":{"thermostat1":{"maxTempSinceLastReboot":98.34,"__t":"c"},"$metadata":{"$lastUpdated":"2020-10-21T14:13:39.36491Z","thermostat1":{"$lastUpdated":"2020-10-21T14:13:39.36491Z","maxTempSinceLastReboot":{"$lastUpdated":"2020-10-21T14:13:39.36491Z"},"__t":{"$lastUpdated":"2020-10-21T14:13:39.36491Z"}}},"$version":3}}}
Properties (set by device):  {b'hubName': b'my-pnp-hub', b'deviceId': b'my-pnp-device', b'operationTimestamp': b'2020-10-21T14:13:39.3649100+00:00', b'iothub-message-schema': b'twinChangeNotification', b'opType': b'updateTwin'}
System properties (set by IoT Hub):  {b'user-id': b'my-pnp-hub', b'correlation-id': b'1210b664ab83', b'content-type': b'application/json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-enqueuedtime': 1603289619481, b'iothub-message-source': b'twinChangeEvents', b'x-opt-sequence-number': 13341, b'x-opt-offset': b'12890102216', b'x-opt-enqueued-time': 1603289619668}
```

## <a name="read-digital-twin-change-notifications"></a>Чтение уведомлений об изменениях цифровых двойника

Вы можете настроить центр Интернета вещей для создания уведомлений об изменении цифровых двойника для маршрутизации в поддерживаемую конечную точку. Дополнительные сведения см. в статье [Использование маршрутизации сообщений центра Интернета вещей для отправки сообщений с устройства в облако в разные конечные точки > события, не связанные с телеметрии](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

Код, приведенный в предыдущем фрагменте кода Python, создает следующий результат, когда центр Интернета вещей создает уведомления об изменении цифровых двойника для устройства без компонента термостата. Свойства приложения `iothub-message-schema` и `opType` предоставляют сведения о типе уведомления об изменении:

```cmd/sh
Received event from partition: 1.
Telemetry received:  [{"op":"add","path":"/$metadata/maxTempSinceLastReboot","value":{"lastUpdateTime":"2020-10-21T14:10:42.4171263Z"}},{"op":"add","path":"/maxTempSinceLastReboot","value":10.96}]
Properties (set by device):  {b'hubName': b'my-pnp-hub', b'deviceId': b'my-pnp-device', b'operationTimestamp': b'2020-10-21T14:10:42.4171263+00:00', b'iothub-message-schema': b'digitalTwinChangeNotification', b'opType': b'updateTwin'}
System properties (set by IoT Hub):  {b'user-id': b'my-pnp-hub\x81\x0e\xa4\x7f', b'correlation-id': b'12104ced5402', b'content-type': b'application/json-patch+json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-enqueuedtime': 1603289442519, b'iothub-message-source': b'digitalTwinChangeEvents', b'x-opt-sequence-number': 13333, b'x-opt-offset': b'12890098024', b'x-opt-enqueued-time': 1603289442738}
```

Код, приведенный в предыдущем фрагменте кода Python, создает следующий результат, когда центр Интернета вещей создает уведомления об изменениях цифровых двойника для устройства с компонентами. В этом примере показаны выходные данные, когда устройство датчика температуры с компонентом термостата создает уведомления. Свойства приложения `iothub-message-schema` и `opType` предоставляют сведения о типе уведомления об изменении:

```cmd/sh
Received event from partition: 1.
Telemetry received:  [{"op":"add","path":"/thermostat1","value":{"$metadata":{"maxTempSinceLastReboot":{"lastUpdateTime":"2020-10-21T14:13:39.36491Z"}},"maxTempSinceLastReboot":98.34}}]
Properties (set by device):  {b'hubName': b'my-pnp-hub', b'deviceId': b'my-pnp-device', b'operationTimestamp': b'2020-10-21T14:13:39.3649100+00:00', b'iothub-message-schema': b'digitalTwinChangeNotification', b'opType': b'updateTwin'}
System properties (set by IoT Hub):  {b'user-id': b'my-pnp-hub', b'correlation-id': b'1210b664ab83', b'content-type': b'application/json-patch+json', b'content-encoding': b'utf-8', b'iothub-connection-device-id': b'my-pnp-device', b'iothub-enqueuedtime': 1603289619481, b'iothub-message-source': b'digitalTwinChangeEvents', b'x-opt-sequence-number': 13342, b'x-opt-offset': b'12890102984', b'x-opt-enqueued-time': 1603289619668}
```
