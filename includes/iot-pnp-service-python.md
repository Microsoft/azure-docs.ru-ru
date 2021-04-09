---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 6b861baea93c2c57b8f66ebac928a7e4fd3adfa8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95487805"
---
IoT Plug and Play упрощает работу с Интернетом вещей, позволяя взаимодействовать с моделью устройства без необходимости понимать базовую реализацию устройства. В этом кратком руководстве показано, как подключиться к устройству IoT Plug and Play, подключенному к решению, а также управлять им с помощью Python.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Для выполнения инструкций, приведенных в этом кратком руководстве, на компьютере для разработки необходимо установить Python 3.7. Вы можете скачать последнюю рекомендуемую версию для нескольких платформ на сайте [python.org](https://www.python.org/). Проверить версию Python можно с помощью следующей команды:  

```cmd/sh
python --version
```

Пакет **azure-iot-device** опубликован как PIP.

В локальной среде Python установите пакет следующим образом:

```cmd/sh
pip install azure-iot-device
```

Установите пакет **azure-iot-hub**, выполнив следующую команду:

```cmd/sh
pip install azure-iot-hub
```

## <a name="run-the-sample-device"></a>Запуск примера устройства

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Дополнительные сведения о примере конфигурации см. в [образце файла сведений](https://github.com/Azure/azure-iot-sdk-python/blob/master/azure-iot-device/samples/pnp/README.md).

В рамках этого краткого руководства вы сможете применить пример термостата, написанный на Python, в качестве устройства IoT Plug and Play. Чтобы запустить пример устройства, сделайте следующее:

1. Откройте окно терминала в любой папке. Выполните следующую команду, чтобы клонировать репозиторий GitHub для [пакета средств разработки Python для Интернета вещей Azure](https://github.com/Azure/azure-iot-sdk-python) в следующее расположение:

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-python
    ```

1. Это окно терминала используется в качестве терминала **устройства**. Перейдите в папку клонированного репозитория, а затем в папку */azure-iot-sdk-python/azure-iot-device/samples/pnp*.

1. Запустите пример термостата с помощью следующей команды:

    ```cmd/sh
    python simple_thermostat.py
    ```

1. Отобразится сообщение о том, что устройство отправило определенные сведения и находится в сети. Эти сообщения означают, что устройство начало отправлять в концентратор данные телеметрии и теперь готово получать команды и обновления свойств. Не закрывайте этот терминал, он понадобится вам, чтобы проверить работу примера службы.

## <a name="run-the-sample-solution"></a>Запуск примера решения

В рамках этого краткого руководства вы примените пример решения Интернета вещей на Python для взаимодействия с только что настроенным примером устройства.

1. Откройте другое окно терминала, которое будет терминалом **службы**.

1. Перейдите в папку */azure-iot-sdk-python/azure-iot-hub/samples* в клонированном репозитории SDK Python.

1. Откройте файл *registry_manager_pnp_sample.py* и проверьте код. В этом примере показано, как использовать класс **IoTHubRegistryManager** для взаимодействия с устройством IoT Plug and Play.

> [!NOTE]
> В этих примерах служб мы используем класс **IoTHubRegistryManager** из **клиента службы Центра Интернета вещей**. Дополнительные сведения об интерфейсах API, включая API цифровых двойников, см. в [руководстве для разработчиков служб](../articles/iot-pnp/concepts-developer-guide-service.md).

### <a name="get-the-device-twin"></a>Получение двойника устройства

Во время прохождения статьи [Настройка среды для кратких руководств и учебников IoT Plug and Play](../articles/iot-pnp/set-up-environment.md) вы создали две переменные среды, чтобы настроить пример для подключения к центру Интернета вещей и устройству:

* **IOTHUB_CONNECTION_STRING**: строка подключения центра Интернета вещей, которую вы записали ранее.
* **IOTHUB_DEVICE_ID**: `"my-pnp-device"`.

Чтобы запустить этот пример, используйте следующую команду в терминале **службы**:

```cmd/sh
set IOTHUB_METHOD_NAME="getMaxMinReport"
set IOTHUB_METHOD_PAYLOAD="hello world"
python registry_manager_pnp_sample.py
```

> [!NOTE]
> Если вы выполняете этот пример в Linux, используйте `export` вместо `set`.

В выходных данных отобразится двойник устройства и соответствующий идентификатор модели:

```cmd/sh
The Model ID for this device is:
dtmi:com:example:Thermostat;1
```

В следующем фрагменте показан пример кода из файла *registry_manager_pnp_sample.py*:

```python
    # Create IoTHubRegistryManager
    iothub_registry_manager = IoTHubRegistryManager(iothub_connection_str)

    # Get device twin
    twin = iothub_registry_manager.get_twin(device_id)
    print("The device twin is: ")
    print("")
    print(twin)
    print("")

    # Print the device's model ID
    additional_props = twin.additional_properties
    if "modelId" in additional_props:
        print("The Model ID for this device is:")
        print(additional_props["modelId"])
        print("")
```

### <a name="update-a-device-twin"></a>Обновление двойника устройства

В этом примере показано, как обновить доступное для записи свойство `targetTemperature` на устройстве.

```python
    # Update twin
    twin_patch = Twin()
    twin_patch.properties = TwinProperties(
        desired={"targetTemperature": 42}
    )  # this is relevant for the thermostat device sample
    updated_twin = iothub_registry_manager.update_twin(device_id, twin_patch, twin.etag)
    print("The twin patch has been successfully applied")
    print("")
```

Вы можете убедиться, что обновление успешно применено, в терминале **устройства**, где отображаются следующие выходные данные:

```cmd/sh
the data in the desired properties patch was: {'targetTemperature': 42, '$version': 2}
```

В терминале **службы** подтверждается, что обновление применено успешно:

```cmd/sh
The twin patch has been successfully applied
```

### <a name="invoke-a-command"></a>Вызов команды

Затем в примере вызывается команда.

В терминале **службы** отобразится сообщение с подтверждением, полученное от устройства:

```cmd/sh
The device method has been successfully invoked
```

В терминале **устройства** вы увидите, что устройство успешно получило команду:

```cmd/sh
Command request received with payload
hello world
Will return the max, min and average temperature from the specified time hello to the current time
Done generating
{"tempReport": {"avgTemp": 34.2, "endTime": "09/07/2020 09:58:11", "maxTemp": 49, "minTemp": 10, "startTime": "09/07/2020 09:56:51"}}
```
