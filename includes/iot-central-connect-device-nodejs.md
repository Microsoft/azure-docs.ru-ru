---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 03/31/2021
ms.openlocfilehash: 6c97ee01dd1ad5b669142d74a02bada010525e81
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491127"
---
## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этой статье, необходимы следующие ресурсы:

* Приложение Azure IoT Central, созданное на основе шаблона **Пользовательское приложение**. Дополнительные сведения см. в [кратком руководстве по созданию приложения](../articles/iot-central/core/quick-deploy-iot-central.md). Приложение должно быть создано не ранее 14 июля 2020 года.
* Компьютер для разработки с установленным [Node.js](https://nodejs.org/) 6 или более поздней версии. Вы можете запустить `node --version` в командной строке, чтобы проверить версию. В инструкциях этого руководства предполагается, что вы выполняете команду **node** в командной строке Windows. Но вы можете использовать Node.js во множестве других операционных системах.
* Локальная копия репозитория [пакета SDK Интернета вещей Microsoft Azure для Node.js](https://github.com/Azure/azure-iot-sdk-node) с сайта GitHub, содержащая образец кода. Используйте эту ссылку, чтобы скачать копию репозитория: [Скачать ZIP-файл](https://github.com/Azure/azure-iot-sdk-node/archive/master.zip). Затем распакуйте файл в подходящее расположение на локальном компьютере.

## <a name="review-the-code"></a>Просмотр кода

В копии пакета SDK Интернета вещей Microsoft Azure для Node.js, которую вы скачали ранее, откройте файл *azure-iot-sdk-node/device/samples/pnp/pnpTemperatureController.js* в текстовом редакторе.

Когда вы запускаете пример для подключения к IoT Central, для регистрации устройства и создания строки подключения используется служба подготовки устройств (DPS). В примере извлекаются необходимые сведения о подключении DPS из среды командной строки.

Метод `main`:

* создает объект `client` и задает идентификатор модели `dtmi:com:example:TemperatureController;2` перед установкой соединения; IoT Central использует идентификатор модели для идентификации или создания шаблона устройства для этого устройства. Дополнительные сведения см. в разделе [Связывание устройства с шаблоном устройства](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template).
* Создает обработчики для трех команд.
* Запускает цикл для отправки данных телеметрии температуры для каждого компонента термостата каждые 5 секунд.
* Запускает цикл для отправки данных телеметрии о размере рабочего набора для компонента по умолчанию каждые 6 секунд.
* Отправляет свойство `maxTempSinceLastReboot` для каждого компонента термостата.
* Отправляет свойства со сведениями об устройстве.
* Создает обработчики свойств, доступных для записи, для трех компонентов.

```javascript
async function main() {
  // ...

  // fromConnectionString must specify a transport, coming from any transport package.
  const client = Client.fromConnectionString(deviceConnectionString, Protocol);
  console.log('Connecting using connection string: ' + deviceConnectionString);
  let resultTwin;

  try {
    // Add the modelId here
    await client.setOptions(modelIdObject);
    await client.open();
    console.log('Enabling the commands on the client');
    client.onDeviceMethod(commandNameGetMaxMinReport1, commandHandler);
    client.onDeviceMethod(commandNameGetMaxMinReport2, commandHandler);
    client.onDeviceMethod(commandNameReboot, commandHandler);

    // Send Telemetry after some interval
    let index1 = 0;
    let index2 = 0;
    let index3 = 0;
    intervalToken1 = setInterval(() => {
      const data = JSON.stringify(thermostat1.updateSensor().getCurrentTemperatureObject());
      sendTelemetry(client, data, index1, thermostat1ComponentName).catch((err) => console.log('error ', err.toString()));
      index1 += 1;
    }, 5000);

    intervalToken2 = setInterval(() => {
      const data = JSON.stringify(thermostat2.updateSensor().getCurrentTemperatureObject());
      sendTelemetry(client, data, index2, thermostat2ComponentName).catch((err) => console.log('error ', err.toString()));
      index2 += 1;
    }, 5500);


    intervalToken3 = setInterval(() => {
      const data = JSON.stringify({ workingset: 1 + (Math.random() * 90) });
      sendTelemetry(client, data, index3, null).catch((err) => console.log('error ', err.toString()));
      index3 += 1;
    }, 6000);

    // attach a standard input exit listener
    exitListener(client);

    try {
      resultTwin = await client.getTwin();
      // Only report readable properties
      const patchRoot = helperCreateReportedPropertiesPatch({ serialNumber: serialNumber }, null);
      const patchThermostat1Info = helperCreateReportedPropertiesPatch({
        maxTempSinceLastReboot: thermostat1.getMaxTemperatureValue(),
      }, thermostat1ComponentName);

      const patchThermostat2Info = helperCreateReportedPropertiesPatch({
        maxTempSinceLastReboot: thermostat2.getMaxTemperatureValue(),
      }, thermostat2ComponentName);

      const patchDeviceInfo = helperCreateReportedPropertiesPatch({
        manufacturer: 'Contoso Device Corporation',
        model: 'Contoso 47-turbo',
        swVersion: '10.89',
        osName: 'Contoso_OS',
        processorArchitecture: 'Contoso_x86',
        processorManufacturer: 'Contoso Industries',
        totalStorage: 65000,
        totalMemory: 640,
      }, deviceInfoComponentName);

      // the below things can only happen once the twin is there
      updateComponentReportedProperties(resultTwin, patchRoot, null);
      updateComponentReportedProperties(resultTwin, patchThermostat1Info, thermostat1ComponentName);
      updateComponentReportedProperties(resultTwin, patchThermostat2Info, thermostat2ComponentName);
      updateComponentReportedProperties(resultTwin, patchDeviceInfo, deviceInfoComponentName);
      desiredPropertyPatchListener(resultTwin, [thermostat1ComponentName, thermostat2ComponentName, deviceInfoComponentName]);
    } catch (err) {
      console.error('could not retrieve twin or report twin properties\n' + err.toString());
    }
  } catch (err) {
    console.error('could not connect Plug and Play client or could not attach interval function for telemetry\n' + err.toString());
  }
}
```

Функция `provisionDevice` показывает, как устройство использует DPS для регистрации и подключения к IoT Central. Полезные данные включают идентификатор модели, который IoT Central использует для [связывания устройства с шаблоном устройства](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template).

```javascript
async function provisionDevice(payload) {
  var provSecurityClient = new SymmetricKeySecurityClient(registrationId, symmetricKey);
  var provisioningClient = ProvisioningDeviceClient.create(provisioningHost, idScope, new ProvProtocol(), provSecurityClient);

  if (!!(payload)) {
    provisioningClient.setProvisioningPayload(payload);
  }

  try {
    let result = await provisioningClient.register();
    deviceConnectionString = 'HostName=' + result.assignedHub + ';DeviceId=' + result.deviceId + ';SharedAccessKey=' + symmetricKey;
    console.log('registration succeeded');
    console.log('assigned hub=' + result.assignedHub);
    console.log('deviceId=' + result.deviceId);
    console.log('payload=' + JSON.stringify(result.payload));
  } catch (err) {
    console.error("error registering device: " + err.toString());
  }
}
```

Функция `sendTelemetry` показывает, как устройство отправляет данные телеметрии температуры в IoT Central. Для данных телеметрии от компонентов эта функция добавляет свойство `$.sub` с именем компонента:

```javascript
async function sendTelemetry(deviceClient, data, index, componentName) {
  if (!!(componentName)) {
    console.log('Sending telemetry message %d from component: %s ', index, componentName);
  } else {
    console.log('Sending telemetry message %d from root interface', index);
  }
  const msg = new Message(data);
  if (!!(componentName)) {
    msg.properties.add(messageSubjectProperty, componentName);
  }
  msg.contentType = 'application/json';
  msg.contentEncoding = 'utf-8';
  await deviceClient.sendEvent(msg);
}
```

Метод `main` использует вспомогательный метод `helperCreateReportedPropertiesPatch`, вызываемый для создания сообщений об обновлении свойств. Этот метод принимает необязательный параметр для указания компонента, отправляющего свойство:

```javascript
const helperCreateReportedPropertiesPatch = (propertiesToReport, componentName) => {
  let patch;
  if (!!(componentName)) {
    patch = { };
    propertiesToReport.__t = 'c';
    patch[componentName] = propertiesToReport;
  } else {
    patch = { };
    patch = propertiesToReport;
  }
  if (!!(componentName)) {
    console.log('The following properties will be updated for component: ' + componentName);
  } else {
    console.log('The following properties will be updated for root interface.');
  }
  console.log(patch);
  return patch;
};
```

Метод `main` использует следующий метод для обработки обновлений свойств, доступных для записи, от IoT Central. Обратите внимание, что метод создает ответ с версией и кодом состояния:

```javascript
const desiredPropertyPatchListener = (deviceTwin, componentNames) => {
  deviceTwin.on('properties.desired', (delta) => {
    console.log('Received an update for device with value: ' + JSON.stringify(delta));
    Object.entries(delta).forEach(([key, values]) => {
      const version = delta.$version;
      if (!!(componentNames) && componentNames.includes(key)) { // then it is a component we are expecting
        const componentName = key;
        const patchForComponents = { [componentName]: {} };
        Object.entries(values).forEach(([propertyName, propertyValue]) => {
          if (propertyName !== '__t' && propertyName !== '$version') {
            console.log('Will update property: ' + propertyName + ' to value: ' + propertyValue + ' of component: ' + componentName);
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
        console.log('Will update property: ' + key + ' to value: ' + values + ' for root');
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

Метод `main` использует следующие методы для обработки команд от IoT Central:

```javascript
const commandHandler = async (request, response) => {
  helperLogCommandRequest(request);
  switch (request.methodName) {
  case commandNameGetMaxMinReport1: {
    await sendCommandResponse(request, response, 200, thermostat1.getMaxMinReportObject());
    break;
  }
  case commandNameGetMaxMinReport2: {
    await sendCommandResponse(request, response, 200, thermostat2.getMaxMinReportObject());
    break;
  }
  case commandNameReboot: {
    await sendCommandResponse(request, response, 200, 'reboot response');
    break;
  }
  default:
    await sendCommandResponse(request, response, 404, 'unknown method');
    break;
  }
};

const sendCommandResponse = async (request, response, status, payload) => {
  try {
    await response.send(status, payload);
    console.log('Response to method: ' + request.methodName + ' sent successfully.' );
  } catch (err) {
    console.error('An error ocurred when sending a method response:\n' + err.toString());
  }
};
```

## <a name="get-connection-information"></a>Получение сведений о подключении

[!INCLUDE [iot-central-connection-configuration](iot-central-connection-configuration.md)]

## <a name="run-the-code"></a>Выполнение кода

Чтобы запустить пример приложения, откройте среду командной строки и перейдите в папку *azure-iot-sdk-node/device/samples/pnp*, содержащую файл примера *pnpTemperatureController.js*.

[!INCLUDE [iot-central-connection-environment](iot-central-connection-environment.md)]

Установите необходимые пакеты.

```cmd/sh
npm install
```

Запустите пример:

```cmd/sh
node pnpTemperatureController.js
```

В указанных ниже выходных данных показаны регистрация устройства и его подключение к IoT Central. Затем в примере отправляется свойство `maxTempSinceLastReboot` от двух компонентов термостата, а затем начинается отправка данных телеметрии:

```output
registration succeeded
assigned hub=iotc-....azure-devices.net
deviceId=sample-device-01
payload=undefined
Connecting using connection string: HostName=iotc-....azure-devices.net;DeviceId=sample-device-01;SharedAccessKey=qdv...IpAo=
Enabling the commands on the client
Please enter q or Q to exit sample.
The following properties will be updated for root interface.
{ serialNumber: 'alwinexlepaho8329' }
The following properties will be updated for component: thermostat1
{ thermostat1: { maxTempSinceLastReboot: 1.5902294191855972, __t: 'c' } }
The following properties will be updated for component: thermostat2
{ thermostat2: { maxTempSinceLastReboot: 16.181771928614545, __t: 'c' } }
The following properties will be updated for component: deviceInformation
{ deviceInformation:
   { manufacturer: 'Contoso Device Corporation',
     model: 'Contoso 47-turbo',
     swVersion: '10.89',
     osName: 'Contoso_OS',
     processorArchitecture: 'Contoso_x86',
     processorManufacturer: 'Contoso Industries',
     totalStorage: 65000,
     totalMemory: 640,
     __t: 'c' } }
executed sample
Received an update for device with value: {"$version":1}
Properties have been reported for component: thermostat1
Properties have been reported for component: thermostat2
Properties have been reported for component: deviceInformation
Properties have been reported for root interface.
Sending telemetry message 0 from component: thermostat1 
Sending telemetry message 0 from component: thermostat2 
Sending telemetry message 0 from root interface
```

[!INCLUDE [iot-central-monitor-thermostat](iot-central-monitor-thermostat.md)]

Вы можете увидеть, как устройство реагирует на команды и обновления свойств. Команда `getMaxMinReport` находится в компоненте `thermostat2`, а команда `reboot` — в компоненте по умолчанию. Для компонента "thermostat2" задано свойство `targetTemperature`, доступное для записи:

```output
Received command request for command name: thermostat2*getMaxMinReport
The command request payload is:
2021-03-26T06:00:00.000Z
Response to method: thermostat2*getMaxMinReport sent successfully.

...

Received command request for command name: reboot
The command request payload is:
10
Response to method: reboot sent successfully.

...

Received an update for device with value: {"thermostat2":{"targetTemperature":76,"__t":"c"},"$version":2}
Will update property: targetTemperature to value: 76 of component: thermostat2
Properties have been reported for component: thermostat2
```
