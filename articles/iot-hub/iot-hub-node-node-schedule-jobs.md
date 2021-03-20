---
title: Планирование заданий с помощью Центра Интернета вещей Azure (Node) | Документация Майкрософт
description: Планирование заданий с помощью Центра Интернета вещей Azure для вызова прямого метода на нескольких устройствах. Используйте пакеты SDK для Центра Интернета вещей Azure для Node.js, чтобы реализовать приложения имитации устройства и приложение службы, на которых будет выполнено задание.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 08/16/2019
ms.custom: mqtt, devx-track-js
ms.openlocfilehash: e1992c806619154fa7b3c33500b2e54fbc919f20
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92151441"
---
# <a name="schedule-and-broadcast-jobs-nodejs"></a>Планирование и вещание заданий (Node.js)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Центр Интернета вещей Azure — это полностью управляемая служба, которая позволяет внутреннему приложению создавать и отслеживать задания, осуществляющие планирование и обновление миллионов устройств.  Задания можно использовать для следующих действий:

* Обновление требуемых свойств
* Обновление тегов
* Вызов прямых методов

По сути, задание включает одно из этих действий, отслеживая ход его выполнения на наборе устройств (определяется запросом двойника устройства).  Например, с помощью задания внутреннее приложение может вызывать метод перезагрузки на 10 000 устройств, определенных запросом двойника устройства и запланированных в будущем. Затем это приложение может отследить ход выполнения задания по мере получения и выполнения метода Reboot на каждом из этих устройств.

Дополнительные сведения о каждой из этих возможностей см. в следующих статьях:

* Двойник устройства и свойства: [Начало работы с двойниками устройств](iot-hub-node-node-twin-getstarted.md) и [Руководство. Использование свойств двойников устройств](tutorial-device-twins.md)

* Прямые методы: [Руководство разработчика для центра Интернета вещей. прямые методы](iot-hub-devguide-direct-methods.md) и [учебник. прямые методы](quickstart-control-device-node.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

В этом учебнике описаны следующие процедуры.

* Создание приложения Node.js имитации устройства с прямым методом, который позволяет выполнить действие **lockDoor** путем вызова из серверной части решения.

* Создание консольного приложения Node.js, которое с помощью задания вызывает в приложении имитации устройства прямой метод **lockDoor** и обновляет требуемые свойства с помощью задания устройства.

По завершении работы с этим руководством у вас будет два приложения Node.js:

* **simDevice.js**, который подключается к центру Интернета вещей с удостоверением устройства и получает прямой метод **lockDoor** .

* **scheduleJobService.js**, которое вызывает прямой метод в приложении имитации устройства и обновляет требуемые свойства двойника устройства с помощью задания.

## <a name="prerequisites"></a>Предварительные требования

* Node.js 10.0.x или более поздней версии. В статье [Подготовка среды разработки](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) описывается, как установить Node.js для работы с этим учебником в ОС Windows или Linux.

* Активная учетная запись Azure. Если ее нет, можно создать [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/) всего за несколько минут.

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этой статье использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-a-simulated-device-app"></a>Создание приложения виртуального устройства

В этом разделе вы создадите консольное приложение Node.js, отвечающее на прямой метод, вызванный из облака, который активирует имитированный метод **lockDoor**.

1. Создайте пустую папку с именем **simDevice**.  В папке **simDevice** создайте файл package.json, используя следующую команду в командной строке.  Примите значения по умолчанию:

   ```console
   npm init
   ```

2. В командной строке в папке **simDevice** выполните следующую команду, чтобы установить пакет SDK для устройств **azure-iot-device** и пакет **azure-iot-device-mqtt**.

   ```console
   npm install azure-iot-device azure-iot-device-mqtt --save
   ```

3. В текстовом редакторе создайте файл **simDevice.js** в папке **simDevice**.

4. Добавьте следующие инструкции require в начало файла **simDevice.js**:

    ```javascript
    'use strict';

    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```

5. Добавьте переменную **connectionString**, чтобы создать с ее помощью экземпляр **клиента**. Замените `{yourDeviceConnectionString}` значение заполнителя строкой подключения устройства, скопированным ранее.

    ```javascript
    var connectionString = '{yourDeviceConnectionString}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

6. Добавьте следующую функцию для обработчика метода **lockDoor** .

    ```javascript
    var onLockDoor = function(request, response) {

        // Respond the cloud app for the direct method
        response.send(200, function(err) {
            if (err) {
                console.error('An error occurred when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.');
            }
        });

        console.log('Locking Door!');
    };
    ```

7. Добавьте следующий код, чтобы зарегистрировать обработчик для метода **lockDoor** .

   ```javascript
   client.open(function(err) {
        if (err) {
            console.error('Could not connect to IotHub client.');
        }  else {
            console.log('Client connected to IoT Hub. Register handler for lockDoor direct method.');
            client.onDeviceMethod('lockDoor', onLockDoor);
        }
   });
   ```

8. Сохраните и закройте файл **simDevice.js**.

> [!NOTE]
> Для простоты в этом руководстве не реализуются политики повтора. В рабочем коде следует реализовать политики повторных попыток (например, с экспоненциальной задержкой), как указано в статье [Обработка временных сбоев](/azure/architecture/best-practices/transient-faults).
>

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

[!INCLUDE [iot-hub-howto-schedule-jobs-shared-access-policy-text](../../includes/iot-hub-howto-schedule-jobs-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="schedule-jobs-for-calling-a-direct-method-and-updating-a-device-twins-properties"></a>Планирование заданий для вызова прямого метода и обновления свойств двойника устройства

В этом разделе вы создадите Node.js консольное приложение, которое инициирует удаленный **lockDoor** на устройстве с помощью прямого метода и обновляет свойства двойникаа устройства.

1. Создайте пустую папку с именем **scheduleJobService**.  В папке **scheduleJobService** создайте файл package.json, используя следующую команду в командной строке.  Примите значения по умолчанию:

    ```console
    npm init
    ```

2. В командной строке в папке **scheduleJobService** выполните следующую команду, чтобы установить пакет SDK для устройств **azure-iothub** и пакет **azure-iot-device-mqtt**.

    ```console
    npm install azure-iothub uuid --save
    ```

3. В текстовом редакторе создайте файл **scheduleJobService.js** в папке **scheduleJobService**.

4. Добавьте следующие операторы "обязательно" в начало файла **scheduleJobService.js** :

    ```javascript
    'use strict';

    var uuid = require('uuid');
    var JobClient = require('azure-iothub').JobClient;
    ```

5. Добавьте следующие объявления переменных. Замените `{iothubconnectionstring}` значение заполнителя значением, скопированным в [поле получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string). Если вы зарегистрировали устройство, отличное от **myDeviceId**, не забудьте изменить его в условии запроса.

    ```javascript
    var connectionString = '{iothubconnectionstring}';
    var queryCondition = "deviceId IN ['myDeviceId']";
    var startTime = new Date();
    var maxExecutionTimeInSeconds =  300;
    var jobClient = JobClient.fromConnectionString(connectionString);
    ```

6. Добавьте следующую функцию, которая используется, чтобы отслеживать выполнение задания:

    ```javascript
    function monitorJob (jobId, callback) {
        var jobMonitorInterval = setInterval(function() {
            jobClient.getJob(jobId, function(err, result) {
            if (err) {
                console.error('Could not get job status: ' + err.message);
            } else {
                console.log('Job: ' + jobId + ' - status: ' + result.status);
                if (result.status === 'completed' || result.status === 'failed' || result.status === 'cancelled') {
                clearInterval(jobMonitorInterval);
                callback(null, result);
                }
            }
            });
        }, 5000);
    }
    ```

7. Добавьте следующий код, чтобы запланировать задание, которое вызывает метод устройства:
  
    ```javascript
    var methodParams = {
        methodName: 'lockDoor',
        payload: null,
        responseTimeoutInSeconds: 15 // Timeout after 15 seconds if device is unable to process method
    };

    var methodJobId = uuid.v4();
    console.log('scheduling Device Method job with id: ' + methodJobId);
    jobClient.scheduleDeviceMethod(methodJobId,
                                queryCondition,
                                methodParams,
                                startTime,
                                maxExecutionTimeInSeconds,
                                function(err) {
        if (err) {
            console.error('Could not schedule device method job: ' + err.message);
        } else {
            monitorJob(methodJobId, function(err, result) {
                if (err) {
                    console.error('Could not monitor device method job: ' + err.message);
                } else {
                    console.log(JSON.stringify(result, null, 2));
                }
            });
        }
    });
    ```

8. Добавьте следующий код, чтобы запланировать задание, которое обновляет двойник устройства:

    ```javascript
    var twinPatch = {
       etag: '*',
       properties: {
           desired: {
               building: '43',
               floor: 3
           }
       }
    };

    var twinJobId = uuid.v4();

    console.log('scheduling Twin Update job with id: ' + twinJobId);
    jobClient.scheduleTwinUpdate(twinJobId,
                                queryCondition,
                                twinPatch,
                                startTime,
                                maxExecutionTimeInSeconds,
                                function(err) {
        if (err) {
            console.error('Could not schedule twin update job: ' + err.message);
        } else {
            monitorJob(twinJobId, function(err, result) {
                if (err) {
                    console.error('Could not monitor twin update job: ' + err.message);
                } else {
                    console.log(JSON.stringify(result, null, 2));
                }
            });
        }
    });
    ```

9. Сохраните и закройте файл **scheduleJobService.js**.

## <a name="run-the-applications"></a>Запуск приложений

Теперь все готово к запуску приложений.

1. В командной строке в папке **simDevice** выполните следующую команду, чтобы начать прослушивание прямого метода перезагрузки:

    ```console
    node simDevice.js
    ```

2. В командной строке в папке **scheduleJobService** выполните следующую команду, чтобы активировать задачи для блокировки дверей и обновления двойника.

    ```console
    node scheduleJobService.js
    ```

3. Вы увидите ответ устройства на прямой метод и состояние задания в консоли.

   Ниже показан ответ устройства на прямой метод:

   ![Выходные данные приложения имитированного устройства](./media/iot-hub-node-node-schedule-jobs/sim-device.png)

   Ниже показаны задания планирования обслуживания для прямого метода и обновления двойникаов устройств, а также задания, выполняемые до завершения:

   ![Запуск приложения виртуального устройства](./media/iot-hub-node-node-schedule-jobs/schedule-job-service.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике описано использование задания для планирования прямого метода на устройстве и обновления свойств двойника устройства.

Чтобы продолжить знакомство с центром Интернета вещей и шаблонами управления устройствами, такими как удаленный через обновление встроенного по Air, см. раздел [учебник. как выполнить обновление встроенного по](tutorial-firmware-update.md).

Чтобы продолжить знакомство с Центром Интернета вещей, см. сведения в статье [Приступая к работе с архитектурой Azure IoT Edge в Linux](../iot-edge/quickstart-linux.md).