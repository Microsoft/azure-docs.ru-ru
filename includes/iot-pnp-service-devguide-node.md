---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 10/20/2020
ms.openlocfilehash: dfeeb451912dd32770a6ae92d73de83851d9d8f6
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102244547"
---
Также доступны следующие ресурсы:

- [ Справочная документация поNode.js SDK](/javascript/api/azure-iothub)
- [Примеры клиента службы](https://github.com/Azure/azure-iot-sdk-node/blob/master/service/samples/javascript/twin.js)
- [Примеры цифровых двойников](https://github.com/Azure/azure-iot-sdk-node/blob/master/service/samples/javascript/get_digital_twin.js)

## <a name="iot-hub-service-client-examples"></a>Примеры клиентов службы центра Интернета вещей

В этом разделе приведены примеры JavaScript с использованием клиента службы центра Интернета вещей, а также **реестра** и **клиентских** классов. Используйте класс **реестра** для взаимодействия с состоянием устройства с помощью двойникова устройства. Вы также можете использовать класс **реестра** для [запроса регистрации устройств](../articles/iot-hub/iot-hub-devguide-query-language.md) в центре Интернета вещей. Для вызова команд на устройстве используется класс **клиента** . Модель [дтдл](../articles/iot-pnp/concepts-digital-twin.md) для устройства определяет свойства и команды, реализуемые устройством. В фрагментах кода `deviceId` переменная содержит идентификатор устройства Самонастраивающийся устройства IOT, зарегистрированного в центре Интернета вещей.

### <a name="get-the-device-twin-and-model-id"></a>Получение двойникаа устройства и идентификатора модели

Чтобы получить двойника устройства и идентификатор модели для устройства IoT самонастраивающийся, подключенного к центру Интернета вещей:

```javascript
var Registry = require('azure-iothub').Registry;

// ...

var registry = Registry.fromConnectionString(connectionString);
registry.getTwin(deviceId, function(err, twin) {
  if (err) {
    console.error(err.message);
  } else {
    console.log('Model Id: ' + twin.modelId);
    console.log(JSON.stringify(twin, null, 2));
  }
}
```

### <a name="update-device-twin"></a>Обновление двойников устройств

В следующем фрагменте кода показано, как обновить `targetTemperature` свойство на устройстве. В этом примере показано, как необходимо получить двойника перед обновлением. Свойство определено в компоненте по умолчанию устройства:

```javascript
var Registry = require('azure-iothub').Registry;
var registry = Registry.fromConnectionString(connectionString);

registry.getTwin(deviceId, function(err, twin) {
  if (err) {
    console.error(err.message);
  } else {
    var twinPatch = {
      properties: {
        desired: {
          targetTemperature: 42
        }
      }
    };
    twin.update(twinPatch, function(err, twin) {
      if (err) {
        console.error(err.message);
      } else {
        console.log(JSON.stringify(twin, null, 2));
      }
    }
  }
}
```

В следующем фрагменте кода показано, как обновить `targetTemperature` свойство компонента. В этом примере показано, как необходимо получить двойника перед обновлением. Свойство определено в компоненте **thermostat1** :

```javascript
var Registry = require('azure-iothub').Registry;
var registry = Registry.fromConnectionString(connectionString);

registry.getTwin(deviceId, function(err, twin) {
  if (err) {
    console.error(err.message);
  } else {
    var twinPatch = {
      properties: {
        desired: {
          thermostat1:
          {
            __t: "c",
            targetTemperature: 45
          }
        }
      }
    };
    twin.update(twinPatch, function(err, twin) {
      if (err) {
        console.error(err.message);
      } else {
        console.log(JSON.stringify(twin, null, 2));
      }
    }
  }
}
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

```javascript
var Client = require('azure-iothub').Client;

// ...

var client = Client.fromConnectionString(connectionString);

var methodParams = {
  methodName: "getMaxMinReport",
  payload: new Date().getMinutes -2,
  responseTimeoutInSeconds: 15
};

client.invokeDeviceMethod(deviceId, methodParams, function (err, result) {
  if (err) {
    console.error('Failed to invoke method \'' + methodParams.methodName + '\': ' + err.message);
  } else {
    console.log(methodParams.methodName + ' on ' + deviceId + ':');
    console.log(JSON.stringify(result, null, 2));
  }
});
```

В следующем фрагменте кода показано, как вызвать `getMaxMinReport` команду для компонента. Команда определена в компоненте **thermostat1** :

```javascript
var Client = require('azure-iothub').Client;

// ...

var client = Client.fromConnectionString(connectionString);

var methodParams = {
  methodName: "thermostat1*getMaxMinReport",
  payload: new Date().getMinutes -2,
  responseTimeoutInSeconds: 15
};

client.invokeDeviceMethod(deviceId, methodParams, function (err, result) {
  if (err) {
    console.error('Failed to invoke method \'' + methodParams.methodName + '\': ' + err.message);
  } else {
    console.log(methodParams.methodName + ' on ' + deviceId + ':');
    console.log(JSON.stringify(result, null, 2));
  }
});
```

## <a name="iot-hub-digital-twin-examples"></a>Примеры цифровых двойника в центре Интернета вещей

Для взаимодействия с состоянием устройства с помощью цифрового двойников используется класс **дигиталтвинклиент** . Модель [дтдл](../articles/iot-pnp/concepts-digital-twin.md) для устройства определяет свойства и команды, реализуемые устройством.

В этом разделе показаны примеры JavaScript с использованием цифрового двойников API.

`digitalTwinId`Переменная содержит идентификатор устройства Самонастраивающийся устройства IOT, зарегистрированного в центре Интернета вещей.

### <a name="get-the-digital-twin-and-model-id"></a>Получение цифровых двойника и идентификатора модели

Чтобы получить цифровой двойника и идентификатор модели устройства IoT самонастраивающийся, подключенного к центру Интернета вещей:

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;
const { inspect } = require('util');

// ...

const credentials = new IoTHubTokenCredentials(connectionString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const digitalTwin = await digitalTwinClient.getDigitalTwin(digitalTwinId);

console.log(inspect(digitalTwin));
console.log('Model Id: ' + inspect(digitalTwin.$metadata.$model));
```

### <a name="update-digital-twin"></a>Обновление цифровых двойника

В следующем фрагменте кода показано, как обновить `targetTemperature` свойство на устройстве. Свойство определено в компоненте по умолчанию устройства:

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;

// ...

const credentials = new IoTHubTokenCredentials(connString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const patch = [{
  op: 'add',
  path: '/targetTemperature',
  value: 42
}];
await digitalTwinClient.updateDigitalTwin(digitalTwinId, patch);
```

В следующем фрагменте кода показано, как обновить `targetTemperature` свойство компонента. Свойство определено в компоненте **thermostat1** :

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;

// ...

const credentials = new IoTHubTokenCredentials(connString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const patch = [{
  op: 'add',
  path: '/thermostat1/targetTemperature',
  value: 42
}];
await digitalTwinClient.updateDigitalTwin(digitalTwinId, patch);
```

### <a name="call-command"></a>Call, команда

В следующем фрагменте кода показано, как вызвать `getMaxMinReport` команду, определенную в компоненте по умолчанию.

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;
const { inspect } = require('util');

// ...

const commandPayload = new Date().getMinutes -2;

const credentials = new IoTHubTokenCredentials(connectionString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const options = {
  connectTimeoutInSeconds: 30,
  responseTimeoutInSeconds: 40
};
const commandResponse = await digitalTwinClient.invokeCommand(digitalTwinId, "getMaxMinReport", commandPayload, options);

console.log(inspect(commandResponse));
```

В следующем фрагменте кода показано, как вызвать `getMaxMinReport` команду для компонента. Команда определена в компоненте **thermostat1** :

```javascript
const IoTHubTokenCredentials = require('azure-iothub').IoTHubTokenCredentials;
const DigitalTwinClient = require('azure-iothub').DigitalTwinClient;
const { inspect } = require('util');

// ...

const commandPayload = new Date().getMinutes -2;

const credentials = new IoTHubTokenCredentials(connectionString);
const digitalTwinClient = new DigitalTwinClient(credentials);

const options = {
  connectTimeoutInSeconds: 30,
  responseTimeoutInSeconds: 40
};
const commandResponse = await digitalTwinClient.invokeComponentCommand(digitalTwinId, "thermostat1", "getMaxMinReport", commandPayload, options);

console.log(inspect(commandResponse));
```

## <a name="read-device-telemetry"></a>Чтение телеметрии устройства

Устройства IoT самонастраивающийся отправляют данные телеметрии, определенные в модели ДТДЛ, в центр Интернета вещей. По умолчанию центр Интернета вещей направляет данные телеметрии в конечную точку концентратора событий, где их можно использовать. Дополнительные сведения см. в статье [Использование маршрутизации сообщений центра Интернета вещей для отправки сообщений с устройства в облако в разные конечные точки](../articles/iot-hub/iot-hub-devguide-messages-d2c.md).

В следующем фрагменте кода показано, как считывать данные телеметрии из конечной точки концентраторов событий по умолчанию. Код в этом фрагменте кода взят из руководства по использованию центра Интернета вещей [Отправка данных телеметрии с устройства в центр Интернета вещей и их чтение с помощью серверного приложения](../articles/iot-hub/quickstart-send-telemetry-node.md):

```javascript
const { EventHubConsumerClient } = require("@azure/event-hubs");

var printError = function (err) {
  console.log(err.message);
};

var printMessages = function (messages) {
  for (const message of messages) {
    console.log("Telemetry received: ");
    console.log(JSON.stringify(message.body));
    console.log("Properties (set by device): ");
    console.log(JSON.stringify(message.properties));
    console.log("System properties (set by IoT Hub): ");
    console.log(JSON.stringify(message.systemProperties));
    console.log("");
  }
};

// ...

const clientOptions = {};

const consumerClient = new EventHubConsumerClient("$Default", connectionString, clientOptions);

consumerClient.subscribe({
  processEvents: printMessages,
  processError: printError,
});
```

В следующих выходных данных из предыдущего кода показаны данные телеметрии температуры, отправленные с помощью многокомпонентного устройства **температуреконтроллер** IOT Самонастраивающийся. `dt-subject`Свойство System показывает имя компонента, который отправил телеметрию. В этом примере два компонента являются `thermostat1` и `thermostat2` определены в модели дтдл. В `dt-dataschema` системном свойстве отображается идентификатор модели:

```cmd/sh
Telemetry received:
{"temperature":68.77370855171125}
Properties (set by device):
undefined
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-connection-auth-method":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","iothub-connection-auth-generation-id":"637388034455888246","iothub-enqueuedtime":1603206669320,"iothub-message-source":"Telemetry","dt-subject":"thermostat1","dt-dataschema":"dtmi:com:example:TemperatureController;1","contentType":"application/json","contentEncoding":"utf-8"}

Telemetry received:
{"temperature":30.833394506549226}
Properties (set by device):
undefined
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-connection-auth-method":"{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","iothub-connection-auth-generation-id":"637388034455888246","iothub-enqueuedtime":1603206665835,"iothub-message-source":"Telemetry","dt-subject":"thermostat2","dt-dataschema":"dtmi:com:example:TemperatureController;1","contentType":"application/json","contentEncoding":"utf-8"}
```

## <a name="read-device-twin-change-notifications"></a>Чтение уведомлений об изменениях двойникаов устройств

Вы можете настроить центр Интернета вещей для создания уведомлений об изменениях двойникаов устройств для маршрутизации в поддерживаемую конечную точку. Дополнительные сведения см. в статье [Использование маршрутизации сообщений центра Интернета вещей для отправки сообщений с устройства в облако в разные конечные точки > события, не связанные с телеметрии](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

Код, приведенный в предыдущем фрагменте кода JavaScript, создает следующий результат, когда центр Интернета вещей создает уведомления об изменениях двойникаа устройства для устройства термостата без компонента. Свойства приложения `iothub-message-schema` и `opType` предоставляют сведения о типе уведомления об изменении:

```cmd/sh
Telemetry received:
{"version":4,"properties":{"reported":{"maxTempSinceLastReboot":42.1415152639582,"$metadata":{"$lastUpdated":"2020-10-21T10:01:40.1281138Z","maxTempSinceLastReboot":{"$lastUpdated":"2020-10-21T10:01:40.1281138Z"}},"$version":3}}}
Properties (set by device):
{"hubName":"my-pnp-hub","deviceId":"my-pnp-device","operationTimestamp":"2020-10-21T10:01:40.1281138+00:00","iothub-message-schema":"twinChangeNotification","opType":"updateTwin"}
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-enqueuedtime":1603274500282,"iothub-message-source":"twinChangeEvents","userId":{"type":"Buffer","data":[109,121,45,112,110,112,45,104,117,98]},"correlationId":"11ed82d13f50","contentType":"application/json","contentEncoding":"utf-8"}
```

Код, приведенный в предыдущем фрагменте кода JavaScript, создает следующий результат, когда центр Интернета вещей создает уведомления об изменении двойникаов устройств для устройства с компонентами. В этом примере показаны выходные данные, когда устройство датчика температуры с компонентом термостата создает уведомления. Свойства приложения `iothub-message-schema` и `opType` предоставляют сведения о типе уведомления об изменении:

```cmd/sh
Telemetry received:
{"version":4,"properties":{"reported":{"thermostat1":{"maxTempSinceLastReboot":3.5592971602417913,"__t":"c"},"$metadata":{"$lastUpdated":"2020-10-21T10:07:51.8284866Z","thermostat1":{"$lastUpdated":"2020-10-21T10:07:51.8284866Z","maxTempSinceLastReboot":{"$lastUpdated":"2020-10-21T10:07:51.8284866Z"},"__t":{"$lastUpdated":"2020-10-21T10:07:51.8284866Z"}}},"$version":3}}}
Properties (set by device):
{"hubName":"my-pnp-hub","deviceId":"my-pnp-device","operationTimestamp":"2020-10-21T10:07:51.8284866+00:00","iothub-message-schema":"twinChangeNotification","opType":"updateTwin"}
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-enqueuedtime":1603274871951,"iothub-message-source":"twinChangeEvents","userId":{"type":"Buffer","data":[109,121,45,112,110,112,45,104,117,98]},"correlationId":"11ee605b195f","contentType":"application/json","contentEncoding":"utf-8"}
```

## <a name="read-digital-twin-change-notifications"></a>Чтение уведомлений об изменениях цифровых двойника

Вы можете настроить центр Интернета вещей для создания уведомлений об изменении цифровых двойника для маршрутизации в поддерживаемую конечную точку. Дополнительные сведения см. в статье [Использование маршрутизации сообщений центра Интернета вещей для отправки сообщений с устройства в облако в разные конечные точки > события, не связанные с телеметрии](../articles/iot-hub/iot-hub-devguide-messages-d2c.md#non-telemetry-events).

Код, приведенный в предыдущем фрагменте кода JavaScript, создает следующий результат, когда центр Интернета вещей создает уведомления об изменениях цифровых двойника для устройства без компонента термостата. Свойства приложения `iothub-message-schema` и `opType` предоставляют сведения о типе уведомления об изменении:

```cmd/sh
Telemetry received:
[{"op":"add","path":"/$metadata/maxTempSinceLastReboot","value":{"lastUpdateTime":"2020-10-21T10:01:40.1281138Z"}},{"op":"add","path":"/maxTempSinceLastReboot","value":42.1415152639582}]
Properties (set by device):
{"hubName":"my-pnp-hub","deviceId":"my-pnp-device","operationTimestamp":"2020-10-21T10:01:40.1281138+00:00","iothub-message-schema":"digitalTwinChangeNotification","opType":"updateTwin"}
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-enqueuedtime":1603274500282,"iothub-message-source":"digitalTwinChangeEvents","userId":{"type":"Buffer","data":[109,121,45,112,110,112,45,104,117,98]},"correlationId":"11ed82d13f50","contentType":"application/json-patch+json","contentEncoding":"utf-8"}
```

Код, приведенный в предыдущем фрагменте кода JavaScript, создает следующий результат, когда центр Интернета вещей создает уведомления об изменении цифровых двойника для устройства с компонентами. В этом примере показаны выходные данные, когда устройство датчика температуры с компонентом термостата создает уведомления. Свойства приложения `iothub-message-schema` и `opType` предоставляют сведения о типе уведомления об изменении:

```cmd/sh
Telemetry received:
[{"op":"add","path":"/thermostat1","value":{"$metadata":{"maxTempSinceLastReboot":{"lastUpdateTime":"2020-10-21T10:07:51.8284866Z"}},"maxTempSinceLastReboot":3.5592971602417913}}]
Properties (set by device):
{"hubName":"my-pnp-hub","deviceId":"my-pnp-device","operationTimestamp":"2020-10-21T10:07:51.8284866+00:00","iothub-message-schema":"digitalTwinChangeNotification","opType":"updateTwin"}
System properties (set by IoT Hub):
{"iothub-connection-device-id":"my-pnp-device","iothub-enqueuedtime":1603274871951,"iothub-message-source":"digitalTwinChangeEvents","userId":{"type":"Buffer","data":[109,121,45,112,110,112,45,104,117,98]},"correlationId":"11ee605b195f","contentType":"application/json-patch+json","contentEncoding":"utf-8"}
```
