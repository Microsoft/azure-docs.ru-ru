---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/19/2020
ms.openlocfilehash: 0aa13d8d23f4f18004131a25f8eb42388d79834b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104582811"
---
## <a name="model-id-announcement"></a>Объявление идентификатора модели

Чтобы объявить идентификатор модели, устройство должно включить его в сведения о подключении:

```nodejs
const modelIdObject = { modelId: 'dtmi:com:example:Thermostat;1' };
const client = Client.fromConnectionString(deviceConnectionString, Protocol);
await client.setOptions(modelIdObject);
await client.open();
```

> [!TIP]
> Для модулей и IoT Edge используйте `ModuleClient` вместо `Client` .

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

```nodejs
async function sendTelemetry(deviceClient, data, index, componentName) {
  const msg = new Message(data);
  if (!!(componentName)) {
    msg.properties.add(messageSubjectProperty, componentName);
  }
  msg.contentType = 'application/json';
  msg.contentEncoding = 'utf-8';
  await deviceClient.sendEvent(msg);
}
```

### <a name="read-only-properties"></a>Свойства только для чтения

Для сообщения свойства из компонента по умолчанию не требуется специальная конструкция:

```nodejs
const createReportPropPatch = (propertiesToReport) => {
  let patch;
  patch = { };
  patch = propertiesToReport;
  return patch;
};

deviceTwin = await client.getTwin();
patchThermostat = createReportPropPatch({
  maxTempSinceLastReboot: 38.7
});

deviceTwin.properties.reported.update(patchThermostat, function (err) {
  if (err) throw err;
});
```

Двойника устройства обновляется со следующим сообщаемым свойством:

```json
{
  "reported": {
      "maxTempSinceLastReboot" : 38.7
  }
}
```

При использовании вложенных компонентов свойства должны быть созданы в пределах имени компонента.

```nodejs
helperCreateReportedPropertiesPatch = (propertiesToReport, componentName) => {
  let patch;
  if (!!(componentName)) {
    patch = { };
    propertiesToReport.__t = 'c';
    patch[componentName] = propertiesToReport;
  } else {
    patch = { };
    patch = propertiesToReport;
  }
  return patch;
};

deviceTwin = await client.getTwin();
patchThermostat1Info = helperCreateReportedPropertiesPatch({
  maxTempSinceLastReboot: 38.7,
}, 'thermostat1');

deviceTwin.properties.reported.update(patchThermostat1Info, function (err) {
  if (err) throw err;
});
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

Эти свойства можно задать на устройстве или обновить с помощью решения. Если решение обновляет свойство, клиент получает уведомление в качестве обратного вызова в `Client` или `ModuleClient` . Чтобы следовать правилам самонастраивающийся IoT, устройство должно информировать службу о том, что свойство было успешно получено.

#### <a name="report-a-writable-property"></a>Сообщить о свойстве, доступном для записи

Когда устройство сообщает о доступном для записи свойстве, оно должно включать `ack` значения, определенные в соглашениях.

Чтобы сообщить свойство, доступное для записи, из компонента по умолчанию:

```nodejs
patch = {
  targetTemperature:
    {
      'value': 23.2,
      'ac': 200,  // using HTTP status codes
      'ad': 'reported default value',
      'av': 0  // not read from a desired property
    }
};
deviceTwin.properties.reported.update(patch, function (err) {
  if (err) throw err;
});
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

```nodejs
patch = {
  thermostat1: {
    '__t' : 'c',
    targetTemperature: {
      'value': 23.2,
      'ac': 200,  // using HTTP status codes
      'ad': 'reported default value',
      'av': 0  // not read from a desired property
    }
  }
};
deviceTwin.properties.reported.update(patch, function (err) {
  if (err) throw err;
});
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

```nodejs
const propertyUpdateHandler = (deviceTwin, propertyName, reportedValue, desiredValue, version) => {
  const patch = createReportPropPatch(
    { [propertyName]:
      {
        'value': desiredValue,
        'ac': 200,
        'ad': 'Successfully executed patch for ' + propertyName,
        'av': version
      }
    });
  updateComponentReportedProperties(deviceTwin, patch);
};

desiredPropertyPatchHandler = (deviceTwin) => {
  deviceTwin.on('properties.desired', (delta) => {
    const versionProperty = delta.$version;

    Object.entries(delta).forEach(([propertyName, propertyValue]) => {
      if (propertyName !== '$version') {
        propertyUpdateHandler(deviceTwin, propertyName, null, propertyValue, versionProperty);
      }
    });
  });
};
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

```nodejs
const desiredPropertyPatchListener = (deviceTwin, componentNames) => {
  deviceTwin.on('properties.desired', (delta) => {
    Object.entries(delta).forEach(([key, values]) => {
      const version = delta.$version;
      if (!!(componentNames) && componentNames.includes(key)) { // then it is a component we are expecting
        const componentName = key;
        const patchForComponents = { [componentName]: {} };
        Object.entries(values).forEach(([propertyName, propertyValue]) => {
          if (propertyName !== '__t' && propertyName !== '$version') {
            const propertyContent = { value: propertyValue };
            propertyContent.ac = 200;
            propertyContent.ad = 'Successfully executed patch';
            propertyContent.av = version;
            patchForComponents[componentName][propertyName] = propertyContent;
          }
        });
        updateComponentReportedProperties(deviceTwin, patchForComponents, componentName);
      }
      else if  (key !== '$version') { // individual property for root
        const patchForRoot = { };
        const propertyContent = { value: values };
        propertyContent.ac = 200;
        propertyContent.ad = 'Successfully executed patch';
        propertyContent.av = version;
        patchForRoot[key] = propertyContent;
        updateComponentReportedProperties(deviceTwin, patchForRoot, null);
      }
    });
  });
};
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

```nodejs
const commandHandler = async (request, response) => {
  switch (request.methodName) {
  
  // ...

  case 'thermostat1*reboot': {
    await response.send(200, 'reboot response');
    break;
  }
  default:
    await response.send(404, 'unknown method');
    break;
  }
};

client.onDeviceMethod('thermostat1*reboot', commandHandler);
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

```nodejs
class TemperatureSensor {

  // ...

  getMaxMinReportObject() {
    return {
      maxTemp: this.maxTemp,
      minTemp: this.minTemp,
      avgTemp: this.cumulativeTemperature / this.numberOfTemperatureReadings,
      endTime: (new Date(Date.now())).toISOString(),
      startTime: this.startTime
    };
  }
}

// ...

const deviceTemperatureSensor = new TemperatureSensor();

const commandHandler = async (request, response) => {
  switch (request.methodName) {
  case commandMaxMinReport: {
    console.log('MaxMinReport ' + request.payload);
    await response.send(200, deviceTemperatureSensor.getMaxMinReportObject());
    break;
  }
  default:
    await response.send(404, 'unknown method');
    break;
  }
};
```

> [!Tip]
> Имена запросов и ответов отсутствуют в сериализованных полезных данных, передаваемых по сети.
