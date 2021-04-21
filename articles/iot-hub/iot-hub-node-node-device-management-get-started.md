---
title: Начало работы с функцией управления Центром Интернета вещей Azure (Node) | Документация Майкрософт
description: Запуск удаленной перезагрузки устройства с помощью функции управления устройствами в Центре Интернета вещей. Используйте пакет SDK для Центра Интернета вещей Azure для Node.js, чтобы реализовать приложение имитации устройства, содержащее прямой метод и приложение службы, которое его вызывает.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom: mqtt, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 557cdc5fb71f998dc0b69e52148cc93da7cc8ea7
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107477984"
---
# <a name="get-started-with-device-management-nodejs"></a>Начало работы с управлением устройствами (Node.js)

[!INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]

В этом учебнике описаны следующие процедуры.

* Создание Центра Интернета вещей на [портале Azure](https://portal.azure.com) и удостоверения устройства в центре Интернета вещей.

* Создание приложения для имитации устройства с прямым методом, который позволяет выполнить перезагрузку устройства. Прямые методы вызываются из облака.

* Создание консольного приложения Node.js, вызывающего прямой метод перезагрузки в приложении имитации устройства с помощью Центра Интернета вещей.

По завершении работы с этим руководством у вас будет два консольных приложения Node.js:

* **dmpatterns_getstarted_device.js**, которое подключается к вашему центру Интернета вещей с ранее созданным идентификатором устройства, получает прямой метод перезагрузки, имитирует физическую перезагрузку и сообщает о времени последней перезагрузки.

* **dmpatterns_getstarted_service.js**, которое вызывает прямой метод в приложении для имитации устройства, выводит ответ и отображает обновленные сообщаемые свойства.

## <a name="prerequisites"></a>Предварительные требования

* Node.js 10.0.x или более поздней версии. В статье [Подготовка среды разработки](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) описывается, как установить Node.js для работы с этим учебником в ОС Windows или Linux.

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-a-simulated-device-app"></a>Создание приложения виртуального устройства

В этом разделе выполняются следующие действия:

* создадите консольное приложение Node.js, отвечающее на прямой метод, вызываемый из облака;

* запустите перезагрузку имитации устройства;

* используете сообщаемые свойства в запросах двойника устройства для определения устройств и времени последней перезагрузки.

1. Создайте пустую папку с именем **manageddevice**.  В папке **manageddevice** создайте файл package.json, используя следующую команду в командной строке.  Примите значения по умолчанию:

    ```cmd/sh
    npm init
    ```

2. В командной строке в папке **manageddevice** выполните следующую команду, чтобы установить пакет SDK для устройств **azure-iot-device** и пакет **azure-iot-device-mqtt**.

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

3. В текстовом редакторе создайте файл **dmpatterns_getstarted_device.js** в папке **manageddevice**.

4. Добавьте следующие инструкции require в начале файла **dmpatterns_getstarted_device.js**:

    ```javascript
    'use strict';

    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```

5. Добавьте переменную **connectionString**, чтобы создать с ее помощью экземпляр **клиента**.  Замените значение заполнителя `{yourdeviceconnectionstring}` строкой подключения устройства, скопированной ранее в разделе [Регистрация нового устройства в центре Интернета вещей](#register-a-new-device-in-the-iot-hub).  

    ```javascript
    var connectionString = '{yourdeviceconnectionstring}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

6. Добавьте следующую функцию, чтобы реализовать прямой метод на устройстве:

    ```javascript
    var onReboot = function(request, response) {

        // Respond the cloud app for the direct method
        response.send(200, 'Reboot started', function(err) {
            if (err) {
                console.error('An error occurred when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.');
            }
        });

        // Report the reboot before the physical restart
        var date = new Date();
        var patch = {
            iothubDM : {
                reboot : {
                    lastReboot : date.toISOString(),
                }
            }
        };

        // Get device Twin
        client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                console.log('twin acquired');
                twin.properties.reported.update(patch, function(err) {
                    if (err) throw err;
                    console.log('Device reboot twin state reported')
                });  
            }
        });

        // Add your device's reboot API for physical restart.
        console.log('Rebooting!');
    };
    ```

7. Откройте подключение к Центру Интернета вещей и запустите прослушиватель прямого метода:

    ```javascript
    client.open(function(err) {
        if (err) {
            console.error('Could not open IotHub client');
        }  else {
            console.log('Client opened.  Waiting for reboot method.');
            client.onDeviceMethod('reboot', onReboot);
        }
    });
    ```

8. Сохраните и закройте файл **dmpatterns_getstarted_device.js**.

> [!NOTE]
> Для простоты в этом руководстве не реализуются политики повтора. В рабочем коде следует реализовать политики повторных попыток (например, с экспоненциальной задержкой), как указано в статье [Обработка временных сбоев](/azure/architecture/best-practices/transient-faults).

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

[!INCLUDE [iot-hub-howto-device-management-shared-access-policy-text](../../includes/iot-hub-howto-device-management-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="trigger-a-remote-reboot-on-the-device-using-a-direct-method"></a>Активация удаленной перезагрузки на устройстве с помощью прямого метода

В этом разделе вы создадите консольное приложение Node.js, которое инициирует удаленное обновление устройства с помощью прямого метода. Приложение использует запросы двойника устройства для определения времени последней перезагрузки этого устройства.

1. Создайте пустую папку с именем **triggerrebootondevice**. В папке **triggerrebootondevice** создайте файл package.json, используя следующую команду в командной строке. Примите значения по умолчанию:

    ```cmd/sh
    npm init
    ```

2. В командной строке в папке **triggerrebootondevice** выполните следующую команду, чтобы установить пакет SDK для устройств **azure-iothub** и пакет **azure-iot-device-mqtt**.

    ```cmd/sh
    npm install azure-iothub --save
    ```

3. В текстовом редакторе создайте файл **dmpatterns_getstarted_service.js** в папке **triggerrebootondevice**.

4. Добавьте следующие инструкции require в начале файла **dmpatterns_getstarted_service.js**:

    ```javascript
    'use strict';

    var Registry = require('azure-iothub').Registry;
    var Client = require('azure-iothub').Client;
    ```

5. Добавьте следующие объявления переменной и замените заполнитель `{iothubconnectionstring}` строкой подключения к своему экземпляру центра Интернета вещей, скопированной ранее в разделе [Получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string):

    ```javascript
    var connectionString = '{iothubconnectionstring}';
    var registry = Registry.fromConnectionString(connectionString);
    var client = Client.fromConnectionString(connectionString);
    var deviceToReboot = 'myDeviceId';
    ```

6. Добавьте следующую функцию, чтобы вызвать метод устройства для перезагрузки целевого устройства:

    ```javascript
    var startRebootDevice = function(twin) {

        var methodName = "reboot";

        var methodParams = {
            methodName: methodName,
            payload: null,
            timeoutInSeconds: 30
        };

        client.invokeDeviceMethod(deviceToReboot, methodParams, function(err, result) {
            if (err) {
                console.error("Direct method error: "+err.message);
            } else {
                console.log("Successfully invoked the device to reboot.");  
            }
        });
    };
    ```

7. Добавьте следующую функцию, чтобы запросить устройство и получить сведения о времени последней перезагрузки:

    ```javascript
    var queryTwinLastReboot = function() {

        registry.getTwin(deviceToReboot, function(err, twin){

            if (twin.properties.reported.iothubDM != null)
            {
                if (err) {
                    console.error('Could not query twins: ' + err.constructor.name + ': ' + err.message);
                } else {
                    var lastRebootTime = twin.properties.reported.iothubDM.reboot.lastReboot;
                    console.log('Last reboot time: ' + JSON.stringify(lastRebootTime, null, 2));
                }
            } else 
                console.log('Waiting for device to report last reboot time.');
        });
    };
    ```

8. Добавьте следующий код для вызова функций, которые активируют прямой метод перезагрузки и запросят сведения о времени последней перезагрузки:

    ```javascript
    startRebootDevice();
    setInterval(queryTwinLastReboot, 2000);
    ```

9. Сохраните и закройте файл **dmpatterns_getstarted_service.js**.

## <a name="run-the-apps"></a>Запуск приложений

Теперь все готово к запуску приложений.

1. В командной строке в папке **manageddevice** выполните следующую команду, чтобы начать прослушивание прямого метода перезагрузки.

    ```cmd/sh
    node dmpatterns_getstarted_device.js
    ```

2. В командной строке в папке **triggerrebootondevice** выполните следующую команду, чтобы активировать удаленную перезагрузку и выполнить запрос к двойнику устройства для поиска значения времени последней перезагрузки.

    ```cmd/sh
    node dmpatterns_getstarted_service.js
    ```

3. Вы увидите ответ устройства на прямой метод перезагрузки и состояние перезагрузки в консоли.

   Ниже показан ответ устройства на прямой метод перезагрузки, отправленный службой:

   ![выходные данные приложения manageddevice](./media/iot-hub-node-node-device-management-get-started/device.png)

   Ниже показана служба, которая запускает перезагрузку и опрашивает двойник устройства, чтобы узнать время последней перезагрузки:

   ![выходные данные приложения triggerrebootondevice](./media/iot-hub-node-node-device-management-get-started/service.png)

[!INCLUDE [iot-hub-dm-followup](../../includes/iot-hub-dm-followup.md)]
