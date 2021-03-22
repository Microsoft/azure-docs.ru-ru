---
title: Краткое руководство по отправке телеметрии устройства в Центр Интернета вещей Azure (Node.js)
description: Из этого краткого руководства вы узнаете, как отправлять данные телеметрии с устройства в центр Интернета вещей с помощью пакета SDK для устройств Центра Интернета вещей Azure для Node.js.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: node
ms.topic: quickstart
ms.date: 01/11/2021
ms.openlocfilehash: 97fcff4706d6da968c93426f85569fed9af95aac
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102197815"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-nodejs"></a>Краткое руководство. Отправка данных телеметрии с устройства в центр Интернета вещей (Node.js)

**Область применения**: [разработка приложений для устройств](about-iot-develop.md#device-application-development)

В этом кратком руководстве описан базовый рабочий процесс разработки приложений для устройств Интернета вещей. Вы создадите центр Интернета вещей Azure и имитированное устройство с помощью Azure CLI, а затем с помощью пакета SDK для Интернета вещей Azure для Node.js получите доступ к устройству и отправите данные телеметрии в центр Интернета вещей.

## <a name="prerequisites"></a>Предварительные требования
- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Azure CLI. Вы можете выполнить все команды в этом кратком руководстве, используя Azure Cloud Shell, интерактивную оболочку CLI, которая работает в браузере. При использовании Cloud Shell вам не нужно ничего устанавливать. Если вы решили использовать CLI локально, для выполнения инструкций, приведенных в этом руководстве вам потребуется Azure CLI 2.0.76 или более поздней версии. Чтобы узнать версию, выполните команду az --version. Чтобы выполнить установку или обновление, см. сведения в статье [Установка Azure CLI]( /cli/azure/install-azure-cli).
- [Node.js версии 10 и выше](https://nodejs.org). Если вы используете Azure Cloud Shell, не обновляйте установленную версию Node.js. Azure Cloud Shell уже имеет последнюю версию Node.js.

    Проверьте текущую версию Node.js на компьютере для разработки, используя следующую команду:

    ```cmd/sh
        node --version
    ```

[!INCLUDE [iot-hub-include-create-hub-cli](../../includes/iot-hub-include-create-hub-cli.md)]

## <a name="use-the-nodejs-sdk-to-send-messages"></a>Отправка сообщений с помощью пакета SDK для Node.js
В этом разделе вы научитесь отправлять сообщения с имитированного устройства в центр Интернета вещей с помощью пакета SDK для Node.js. 

1. Откройте новое окно терминала. Этот терминал будет использоваться для установки пакета SDK для Node.js и работы с примером кода Node.js. Теперь у вас должно быть открыто два терминала: тот, который вы только что открыли для работы с Node.js, и оболочка CLI, которая использовалась в предыдущих разделах для ввода команд Azure CLI. 

1. Скопируйте [примеры устройств с пакетом SDK для Центра Интернета вещей Azure для Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples) на локальный компьютер:

    ```console
    git clone https://github.com/Azure/azure-iot-sdk-node
    ```

1. Перейдите в каталог *azure-iot-sdk-node/device/samples*:

    ```console
    cd azure-iot-sdk-node/device/samples
    ```
1. Установите пакет SDK для Интернета вещей Azure для Node.js и необходимые зависимости:

    ```console
    npm install
    ```
    Эта команда устанавливает необходимые зависимости, как указано в файле *package.json* в каталоге примеров устройств.

1. Задайте строку подключения устройства как переменную среды с именем `DEVICE_CONNECTION_STRING`. Используемое строковое значение — это строка, полученная в предыдущем разделе после создания имитированного устройства Node.js. 

    **Windows (CMD)**

    ```console
    set DEVICE_CONNECTION_STRING=<your connection string here>
    ```

    > [!NOTE]
    > В Windows CMD строка подключения не берется в кавычки.

    **Linux (Bash)**

    ```bash
    export DEVICE_CONNECTION_STRING="<your connection string here>"
    ```

1. В открытой оболочке CLI выполните команду [az iot hub monitor-event](/cli/azure/ext/azure-iot/iot/hub#ext-azure-iot-az-iot-hub-monitor-events), чтобы начать мониторинг событий на имитированном устройстве Интернета вещей.  Сообщения о событиях будут выводиться в терминале по мере поступления.

    ```azurecli
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```

1. В окне терминала Node.js выполните код для установленного примера файла *simple_sample_device.js*. Этот код обращается к имитированному устройству Интернета вещей и отправляет сообщение в центр Интернета вещей.

    Чтобы выполнить пример Node.js из терминала, используйте следующий код:
    ```console
    node ./simple_sample_device.js
    ```

    При желании вы можете выполнить код Node.js из примера в своей интегрированной среде разработки JavaScript:
    ```javascript
    'use strict';

    const Protocol = require('azure-iot-device-mqtt').Mqtt;
    // Uncomment one of these transports and then change it in fromConnectionString to test other transports
    // const Protocol = require('azure-iot-device-amqp').AmqpWs;
    // const Protocol = require('azure-iot-device-http').Http;
    // const Protocol = require('azure-iot-device-amqp').Amqp;
    // const Protocol = require('azure-iot-device-mqtt').MqttWs;
    const Client = require('azure-iot-device').Client;
    const Message = require('azure-iot-device').Message;

    // String containing Hostname, Device Id & Device Key in the following formats:
    //  "HostName=<iothub_host_name>;DeviceId=<device_id>;SharedAccessKey=<device_key>"
    const deviceConnectionString = process.env.DEVICE_CONNECTION_STRING;
    let sendInterval;

    function disconnectHandler () {
    clearInterval(sendInterval);
    client.open().catch((err) => {
        console.error(err.message);
    });
    }

    // The AMQP and HTTP transports have the notion of completing, rejecting or abandoning the message.
    // For example, this is only functional in AMQP and HTTP:
    // client.complete(msg, printResultFor('completed'));
    // If using MQTT calls to complete, reject, or abandon are no-ops.
    // When completing a message, the service that sent the C2D message is notified that the message has been processed.
    // When rejecting a message, the service that sent the C2D message is notified that the message won't be processed by the device. the method to use is client.reject(msg, callback).
    // When abandoning the message, IoT Hub will immediately try to resend it. The method to use is client.abandon(msg, callback).
    // MQTT is simpler: it accepts the message by default, and doesn't support rejecting or abandoning a message.
    function messageHandler (msg) {
    console.log('Id: ' + msg.messageId + ' Body: ' + msg.data);
    client.complete(msg, printResultFor('completed'));
    }

    function generateMessage () {
    const windSpeed = 10 + (Math.random() * 4); // range: [10, 14]
    const temperature = 20 + (Math.random() * 10); // range: [20, 30]
    const humidity = 60 + (Math.random() * 20); // range: [60, 80]
    const data = JSON.stringify({ deviceId: 'myFirstDevice', windSpeed: windSpeed, temperature: temperature, humidity: humidity });
    const message = new Message(data);
    message.properties.add('temperatureAlert', (temperature > 28) ? 'true' : 'false');
    return message;
    }

    function errorCallback (err) {
    console.error(err.message);
    }

    function connectCallback () {
    console.log('Client connected');
    // Create a message and send it to the IoT Hub every two seconds
    sendInterval = setInterval(() => {
        const message = generateMessage();
        console.log('Sending message: ' + message.getData());
        client.sendEvent(message, printResultFor('send'));
    }, 2000);

    }

    // fromConnectionString must specify a transport constructor, coming from any transport package.
    let client = Client.fromConnectionString(deviceConnectionString, Protocol);

    client.on('connect', connectCallback);
    client.on('error', errorCallback);
    client.on('disconnect', disconnectHandler);
    client.on('message', messageHandler);

    client.open()
    .catch(err => {
    console.error('Could not connect: ' + err.message);
    });

    // Helper function to print results in the console
    function printResultFor(op) {
    return function printResult(err, res) {
        if (err) console.log(op + ' error: ' + err.toString());
        if (res) console.log(op + ' status: ' + res.constructor.name);
    };
    }
    ```

Когда код Node.js отправляет смоделированное сообщение телеметрии с устройства в центр Интернета вещей, в оболочке CLI для мониторинга событий отображается сообщение:

```output
event:
  component: ''
  interface: ''
  module: ''
  origin: <your device name>
  payload: '{"deviceId":"myFirstDevice","windSpeed":11.853592092144627,"temperature":22.62484121157508,"humidity":66.17960805575937}'
```

Теперь устройство безопасно подключено и отправляет данные телеметрии в Центр Интернета вещей Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вам больше не нужны ресурсы Azure, созданные в этом кратком руководстве, используйте Azure CLI, чтобы удалить их.

> [!IMPORTANT]
> Удаление группы ресурсов — процесс необратимый. Группа ресурсов и все содержащиеся в ней ресурсы удаляются без возможности восстановления. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. 

Удаление группы ресурсов по имени:
1. Выполните команду [az group delete](/cli/azure/group#az-group-delete). При этом будут удалены созданные группа ресурсов, центр Интернета вещей и регистрация устройства.

    ```azurecli
    az group delete --name MyResourceGroup
    ```
1. Выполните команду [az group list](/cli/azure/group#az-group-list), чтобы подтвердить удаление группы ресурсов.  

    ```azurecli
    az group list
    ```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали о базовом рабочем процессе разработки приложений Интернета вещей Azure для безопасного подключения устройства к облаку и отправки данных телеметрии с устройства в облако. Вы создали центр Интернета вещей и имитированное устройство с помощью Azure CLI, а затем с помощью пакета SDK для Интернета вещей Azure для Node.js получили доступ к устройству и отправили данные телеметрии в центр Интернета вещей. 

Теперь изучите пакет SDK для Интернета вещей Azure для Node.js на примерах приложений.

- [Другие примеры Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples). Этот каталог содержит дополнительные примеры из репозитория SDK для Node.js для демонстрации сценариев Центра Интернета вещей.