---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 7ab17f0d34ba7682778120e11aab562e106b5df7
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95487814"
---
IoT Plug and Play упрощает работу с Интернетом вещей, позволяя взаимодействовать с возможностями устройства без знаний базовой реализации устройства. В этом кратком руководстве показано, как подключиться к устройству IoT Plug and Play, подключенному к решению, а также управлять им с помощью Node.js.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Для выполнения инструкций, приведенных в этом кратком руководстве, необходимо установить платформу Node.js на компьютере для разработки. Вы можете скачать последнюю рекомендуемую версию для нескольких платформ на сайте [nodejs.org](https://nodejs.org).

Текущую версию Node.js на компьютере, на котором ведется разработка, можно проверить, используя следующую команду:

```cmd/sh
node --version
```

### <a name="clone-the-sdk-repository-with-the-sample-code"></a>Клонирование репозитория пакета SDK с помощью примера кода

Клонируйте примеры из [репозитория пакета SDK для Node](https://github.com/Azure/azure-iot-sdk-node). Откройте окно терминала в любой папке. С помощью следующей команды клонируйте репозиторий [пакета SDK Интернета вещей Microsoft Azure для Node.js](https://github.com/Azure/azure-iot-sdk-node) с сайта GitHub.

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-node
```

## <a name="run-the-sample-device"></a>Запуск примера устройства

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Дополнительные сведения о примере конфигурации см. в [образце файла сведений](https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/pnp/readme.md).

С помощью этого краткого руководства вы сможете использовать пример термостата, написанный на Node.js, как устройство IoT Plug and Play. Чтобы запустить пример устройства, сделайте следующее:

1. Откройте окно терминала и перейдите в локальную папку, содержащую пакет SDK Microsoft Azure IoT для Node.js, клонированный из репозитория GitHub.

1. Это окно терминала используется в качестве терминала **устройства**. Перейдите в папку клонированного репозитория, а затем в папку */azure-iot-sdk-node/device/samples/pnp*. Установите все зависимости, выполнив следующую команду:

    ```cmd/sh
    npm install
    ```

1. Запустите пример термостата с помощью следующей команды:

    ```cmd/sh
    node simple_thermostat.js
    ```

1. Отобразится сообщение о том, что устройство отправило определенные сведения и находится в сети. Эти сообщения означают, что устройство начало отправлять в концентратор данные телеметрии и теперь готово получать команды и обновления свойств. Не закрывайте этот терминал, он понадобится вам, чтобы проверить работу примера службы.

## <a name="run-the-sample-solution"></a>Запуск примера решения

Во время прохождения статьи [Настройка среды для кратких руководств и учебников IoT Plug and Play](../articles/iot-pnp/set-up-environment.md) вы создали две переменные среды, чтобы настроить пример для подключения к центру Интернета вещей и устройству:

* **IOTHUB_CONNECTION_STRING**: строка подключения центра Интернета вещей, которую вы записали ранее.
* **IOTHUB_DEVICE_ID**: `"my-pnp-device"`.

С помощью этого краткого руководства вы примените пример решения Интернета вещей на Node.js для взаимодействия с настроенным примером устройства.

1. Откройте другое окно терминала, которое будет терминалом **службы**.

1. В клонированном репозитории пакета SDK для Node перейдите в папку */azure-iot-sdk-node/service/samples/javascript*. Установите все зависимости, выполнив следующую команду:

    ```cmd/sh
    npm install
    ```

### <a name="read-a-property"></a>Чтение свойства

1. Когда вы выполняли пример устройства термостата в терминале **устройства**, отображались следующие сведения о состоянии подключения.

    ```cmd/sh
    properties have been reported for component
    sending telemetry message 0...
    ```

1. Перейдите в окно терминала **службы** и запустите пример с помощью следующей команды для чтения сведений об устройстве:

    ```cmd/sh
    node twin.js
    ```

1. В выходных данных терминала **службы** найдите ответ двойника устройства. Вы увидите сведения об идентификаторе модели устройства и его свойствах.

    ```json
    Model Id: dtmi:com:example:Thermostat;1
    {
      "deviceId": "my-pnp-device",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag": "Njc3MDMxNDcy",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00Z",
      "connectionState": "Connected",
      "lastActivityTime": "0001-01-01T00:00:00Z",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "modelId": "dtmi:com:example:Thermostat;1",
      "version": 4,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "2020-10-05T11:35:19.4574755Z"
          },
          "$version": 1
        },
        "reported": {
          "maxTempSinceLastReboot": 31.343640523762232,
          "serialNumber": "123abc",
          "$metadata": {
            "$lastUpdated": "2020-10-05T11:35:23.7339042Z",
            "maxTempSinceLastReboot": {
              "$lastUpdated": "2020-10-05T11:35:23.7339042Z"
            },
            "serialNumber": {
              "$lastUpdated": "2020-10-05T11:35:23.7339042Z"
            }
          },
          "$version": 3
        }
      },
      "capabilities": {
        "iotEdge": false
      },
      "tags": {}
    }
    ```

1. Следующий фрагмент кода из файла *twin.js* получает идентификатор модели двойника устройства.

    ```javascript
    var registry = Registry.fromConnectionString(connectionString);
    registry.getTwin(deviceId, function(err, twin) {
      if (err) {
        console.error(err.message);
      } else {
        console.log('Model Id: ' + twin.modelId);
        //...
      }
      //...
    }
    ```

В нашем примере он выводит значение `Model Id: dtmi:com:example:Thermostat;1`.

> [!NOTE]
> В этих примерах служб мы используем класс **Registry** из **клиента службы Центра Интернета вещей**. Дополнительные сведения об интерфейсах API, включая API цифровых двойников, см. в [руководстве для разработчиков служб](../articles/iot-pnp/concepts-developer-guide-service.md).

### <a name="update-a-writable-property"></a>Обновление доступного для записи свойства

1. Откройте файл *twin.js* в редакторе кода.

1. Просмотрите пример кода. В нем показаны два способа обновления двойника устройства. Чтобы использовать первый способ, измените переменную `twinPatch`, как показано ниже.

    ```javascript
    var twinPatch = {
      tags: {
        city: "Redmond"
      },
      properties: {
        desired: {
          targetTemperature: 42
        }
      }
    };
    ```

    В модели устройства термостата свойство `targetTemperature` определено как записываемое.

1. В терминале **службы** с помощью следующей команды запустите следующий пример для обновления свойства.

    ```cmd/sh
    node twin.js
    ```

1. В терминале **устройства** вы увидите, что устройство успешно получило обновление.

    ```cmd/sh
    The following properties will be updated for the default component:
    {
      targetTemperature: {
        value: 42,
        ac: 200,
        ad: 'Successfully executed patch for targetTemperature',
        av: 2
      }
    }
    updated the property
    ```

1. В терминале **службы** выполните следующую команду, чтобы убедиться, что свойство обновлено.

    ```cmd/sh
    node twin.js
    ```

1. В терминале **службы** в разделе свойств `reported` вы увидите обновленное значение температуры. Устройству может потребоваться некоторое время, чтобы завершить обновление. Повторяйте этот шаг, пока устройство не завершит обновление свойства.

    ```json
    "reported": {
      //...
      "targetTemperature": {
        "value": 42,
        "ac": 200,
        "ad": "Successfully executed patch for targetTemperature",
        "av": 4
      },
      //...
    }
    ```

### <a name="invoke-a-command"></a>Вызов команды

1. Откройте файл *device_method.js* и изучите его код.

1. Перейдите к терминалу **службы**. Чтобы запустить пример для вызова команды, используйте следующую команду:

    ```cmd/sh
    set IOTHUB_METHOD_NAME=getMaxMinReport
    set IOTHUB_METHOD_PAYLOAD=commandpayload
    node device_method.js
    ```

1. В выходных данных в терминале **службы** отобразится следующее подтверждение.

    ```cmd/sh
    getMaxMinReport on my-pnp-device:
    {
      "status": 200,
      "payload": {
        "maxTemp": 23.460596940801928,
        "minTemp": 23.460596940801928,
        "avgTemp": 23.460596940801928,
        "endTime": "2020-10-05T12:48:08.562Z",
        "startTime": "2020-10-05T12:47:54.450Z"
      }
    }
    ```

1. В терминале **устройства** вы увидите, что команда успешно подтверждена.

    ```cmd/sh
    MaxMinReport commandpayload
    Response to method 'getMaxMinReport' sent successfully.
    ```
