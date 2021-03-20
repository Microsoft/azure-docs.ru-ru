---
title: Удостоверение модуля центра Интернета вещей Azure и двойника модуля (Python)
description: Узнайте, как создать удостоверение модуля и обновить двойник модуля с помощью пакетов SDK Центра Интернета вещей для Python.
author: chrissie926
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 04/03/2020
ms.author: menchi
ms.custom: devx-track-python
ms.openlocfilehash: 665281adc892e6b3655c0b1d0533cb3148e62940
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92139395"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-python"></a>Начало работы с удостоверениями модулей центра Интернета вещей и модулем двойника (Python)

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Удостоверения модулей и двойников модуля](iot-hub-devguide-module-twins.md) похожи на удостоверения устройств центра Интернета вещей Azure и двойников устройств, но обеспечивают более детальную детализацию. Хотя удостоверения устройств и двойников устройств центра Интернета вещей Azure позволяют внутреннему приложению настраивать устройство и обеспечивать видимость условий устройства, идентификаторы модулей и модули двойников предоставляют эти возможности для отдельных компонентов устройства. На устройствах с поддержкой нескольких компонентов, таких как устройства на основе операционной системы или устройства встроенного по, они обеспечивают изолированную конфигурацию и условия для каждого компонента.
>

По завершении работы с этим руководством у вас будет три приложения Python:

* **Креатемодуле**, который создает удостоверение устройства, идентификатор модуля и связанные ключи безопасности для подключения клиентов устройства и модуля.

* **Упдатемодулетвиндесиредпропертиес**, который отправляет обновленные требуемые свойства модуля двойника в центр Интернета вещей.

* **Рецеивемодулетвиндесиредпропертиеспатч**, который получает обновление требуемых свойств модуля двойника на устройстве.

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Предварительные условия

[!INCLUDE [iot-hub-include-python-v2-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

В этой статье вы создадите серверную службу, которая добавляет устройство в реестр удостоверений, а затем добавляет модуль на это устройство. Для этой службы требуется разрешение на **запись в реестр** (которое также включает **Чтение реестра**). Вы также создадите службу, которая добавляет требуемые свойства в модуль двойника для только что созданного модуля. Этой службе требуется разрешение на **Подключение к службе** . Хотя существуют политики общего доступа по умолчанию, предоставляющие эти разрешения по отдельности, в этом разделе создается пользовательская политика общего доступа, которая содержит оба этих разрешения.

[!INCLUDE [iot-hub-include-find-service-regrw-connection-string](../../includes/iot-hub-include-find-service-regrw-connection-string.md)]

## <a name="create-a-device-identity-and-a-module-identity-in-iot-hub"></a>Создание удостоверения устройства и удостоверения модуля в Центре Интернета вещей

В этом разделе вы создадите приложение службы Python, которое создает удостоверение устройства и удостоверение модуля в реестре удостоверений в центре Интернета вещей. Устройство или модуль не может подключиться к центру Интернета вещей, если в реестре удостоверений нет записи. Дополнительные сведения см. в статье сведения о [реестре удостоверений в центре Интернета вещей](iot-hub-devguide-identity-registry.md). Запущенное консольное приложение создает уникальные идентификаторы и ключи одновременно для устройства и для модуля. Устройство и модуль применяют эти значения для идентификации при отправке сообщений с устройства в облако в Центр Интернета вещей. В идентификаторах учитывается регистр символов.

1. В командной строке выполните следующую команду, чтобы установить пакет **azure-iot-hub**:

    ```cmd/sh
    pip install azure-iot-hub
    ```

1. В командной строке выполните следующую команду, чтобы установить пакет **мсрест** . Этот пакет необходим для перехвата исключений **хттпоператионеррор** .

    ```cmd/sh
    pip install msrest
    ```

1. С помощью текстового редактора создайте файл с именем **CreateModule.py** в рабочем каталоге.

1. Добавьте следующий код в файл Python. Замените *йоуриосубконнектионстринг* строкой подключения, скопированной в [поле получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string).

    ```python
    import sys
    from msrest.exceptions import HttpOperationError
    from azure.iot.hub import IoTHubRegistryManager

    CONNECTION_STRING = "YourIotHubConnectionString"
    DEVICE_ID = "myFirstDevice"
    MODULE_ID = "myFirstModule"

    try:
        # RegistryManager
        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        try:
            # CreateDevice - let IoT Hub assign keys
            primary_key = ""
            secondary_key = ""
            device_state = "enabled"
            new_device = iothub_registry_manager.create_device_with_sas(
                DEVICE_ID, primary_key, secondary_key, device_state
            )
        except HttpOperationError as ex:
            if ex.response.status_code == 409:
                # 409 indicates a conflict. This happens because the device already exists.
                new_device = iothub_registry_manager.get_device(DEVICE_ID)
            else:
                raise

        print("device <" + DEVICE_ID +
              "> has primary key = " + new_device.authentication.symmetric_key.primary_key)

        try:
            # CreateModule - let IoT Hub assign keys
            primary_key = ""
            secondary_key = ""
            managed_by = ""
            new_module = iothub_registry_manager.create_module_with_sas(
                DEVICE_ID, MODULE_ID, managed_by, primary_key, secondary_key
            )
        except HttpOperationError as ex:
            if ex.response.status_code == 409:
                # 409 indicates a conflict. This happens because the module already exists.
                new_module = iothub_registry_manager.get_module(DEVICE_ID, MODULE_ID)
            else:
                raise

        print("device/module <" + DEVICE_ID + "/" + MODULE_ID +
              "> has primary key = " + new_module.authentication.symmetric_key.primary_key)

    except Exception as ex:
        print("Unexpected error {0}".format(ex))
    except KeyboardInterrupt:
        print("IoTHubRegistryManager sample stopped")
    ```

1. В командной строке выполните следующую команду:

    ```cmd/sh
    python CreateModule.py
    ```

Это приложение создает удостоверение устройства с идентификатором **myFirstDevice** и удостоверение модуля с идентификатором **myFirstModule** на устройстве **myFirstDevice**. (Если идентификатор устройства или модуля уже существует в реестре удостоверений, код просто извлекает сведения о существующем устройстве или модуле.) Приложение отображает идентификатор и первичный ключ для каждого удостоверения.

> [!NOTE]
> В реестре удостоверений в Центре Интернета вещей хранятся только удостоверения устройств и модулей, необходимые для защиты доступа к Центру Интернета вещей. Реестр удостоверений хранит идентификаторы устройств и ключи, используемые в качестве учетных данных безопасности. Реестр удостоверений также хранит флаги включения и отключения для каждого устройства. Эти флаги можно использовать для отключения доступа для этого устройства. Если в приложении необходимо хранить другие метаданные для конкретного устройства, следует использовать хранилище конкретного приложения. У вас нет возможности включать и отключать удостоверения модулей. Дополнительные сведения см. в статье сведения о [реестре удостоверений в центре Интернета вещей](iot-hub-devguide-identity-registry.md).
>

## <a name="update-the-module-twin-using-python-service-sdk"></a>Обновление модуля двойника с помощью пакета SDK для службы Python

В этом разделе вы создадите приложение службы Python, которое обновляет требуемые свойства модуля двойника.

1. В командной строке выполните следующую команду, чтобы установить пакет **Azure-IOT-Hub** . Этот шаг можно пропустить, если вы установили пакет **Azure-IOT-Hub** в предыдущем разделе.

    ```cmd/sh
    pip install azure-iot-hub
    ```

1. С помощью текстового редактора создайте файл с именем **UpdateModuleTwinDesiredProperties.py** в рабочем каталоге.

1. Добавьте следующий код в файл Python. Замените *йоуриосубконнектионстринг* строкой подключения, скопированной в [поле получение строки подключения центра Интернета вещей](#get-the-iot-hub-connection-string).

    ```python
    import sys
    from azure.iot.hub import IoTHubRegistryManager
    from azure.iot.hub.models import Twin, TwinProperties

    CONNECTION_STRING = "YourIoTHubConnectionString"
    DEVICE_ID = "myFirstDevice"
    MODULE_ID = "myFirstModule"

    try:
        # RegistryManager
        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        module_twin = iothub_registry_manager.get_module_twin(DEVICE_ID, MODULE_ID)
        print ( "" )
        print ( "Module twin properties before update    :" )
        print ( "{0}".format(module_twin.properties) )

        # Update twin
        twin_patch = Twin()
        twin_patch.properties = TwinProperties(desired={"telemetryInterval": 122})
        updated_module_twin = iothub_registry_manager.update_module_twin(
            DEVICE_ID, MODULE_ID, twin_patch, module_twin.etag
        )
        print ( "" )
        print ( "Module twin properties after update     :" )
        print ( "{0}".format(updated_module_twin.properties) )

    except Exception as ex:
        print ( "Unexpected error {0}".format(ex) )
    except KeyboardInterrupt:
        print ( "IoTHubRegistryManager sample stopped" )
    ```

## <a name="get-updates-on-the-device-side"></a>Получение обновлений на стороне устройства

В этом разделе вы создадите приложение Python для получения двойникаого обновления требуемых свойств модуля на устройстве.

1. Получите строку подключения модуля. В [портал Azure](https://portal.azure.com/)перейдите в центр Интернета вещей и выберите **устройства IOT** в левой области. Выберите **myFirstDevice** в списке устройств и откройте его. В разделе **удостоверения модулей** выберите **мифирстмодуле**. Скопируйте строку подключения модуля. Он понадобится на следующем шаге.

   ![Сведения о модуле на портале Azure](./media/iot-hub-python-python-module-twin-getstarted/module-detail.png)

1. В командной строке выполните следующую команду, чтобы установить пакет **azure-iot-device**:

    ```cmd/sh
    pip install azure-iot-device
    ```

1. С помощью текстового редактора создайте файл с именем **ReceiveModuleTwinDesiredPropertiesPatch.py** в рабочем каталоге.

1. Добавьте следующий код в файл Python. Замените *йоурмодулеконнектионстринг* строкой подключения модуля, скопированным на шаге 1.

    ```python
    import time
    import threading
    from azure.iot.device import IoTHubModuleClient

    CONNECTION_STRING = "YourModuleConnectionString"


    def twin_update_listener(client):
        while True:
            patch = client.receive_twin_desired_properties_patch()  # blocking call
            print("")
            print("Twin desired properties patch received:")
            print(patch)

    def iothub_client_sample_run():
        try:
            module_client = IoTHubModuleClient.create_from_connection_string(CONNECTION_STRING)

            twin_update_listener_thread = threading.Thread(target=twin_update_listener, args=(module_client,))
            twin_update_listener_thread.daemon = True
            twin_update_listener_thread.start()

            while True:
                time.sleep(1000000)

        except KeyboardInterrupt:
            print("IoTHubModuleClient sample stopped")


    if __name__ == '__main__':
        print ( "Starting the IoT Hub Python sample..." )
        print ( "IoTHubModuleClient waiting for commands, press Ctrl-C to exit" )

        iothub_client_sample_run()
    ```

## <a name="run-the-apps"></a>Запуск приложений

В этом разделе вы запустите приложение устройства **рецеивемодулетвиндесиредпропертиеспатч** , а затем запустите приложение службы **упдатемодулетвиндесиредпропертиес** , чтобы обновить требуемые свойства модуля.

1. Откройте командную строку и запустите приложение устройства:

    ```cmd/sh
    python ReceiveModuleTwinDesiredPropertiesPatch.py
    ```

   ![Начальные выходные данные приложения устройства](./media/iot-hub-python-python-module-twin-getstarted/device-1.png)

1. Откройте отдельную командную строку и запустите приложение службы:

    ```cmd/sh
    python UpdateModuleTwinDesiredProperties.py
    ```

    Обратите внимание, что требуемое свойство **TelemetryInterval** отображается в обновленном модуле двойника в выходных данных приложения службы:

   ![Выходные данные приложения службы](./media/iot-hub-python-python-module-twin-getstarted/service.png)

    Одно и то же свойство отображается в исправлении требуемых свойств, полученном в выходных данных приложения устройства:

   ![В выходных данных приложения устройства отображается исправление требуемых свойств](./media/iot-hub-python-python-module-twin-getstarted/device-2.png)

## <a name="next-steps"></a>Дальнейшие действия

Чтобы продолжить знакомство с Центром Интернета вещей и изучить другие сценарии Интернета вещей, см. следующие ресурсы:

* [How to get started with device management (Node)](iot-hub-node-node-device-management-get-started.md) (Начало работы с управлением устройствами (Node))

* [Краткое руководство. Развертывание первого модуля IoT Edge на устройстве под управлением 64-разрядной ОС Linux](../iot-edge/quickstart-linux.md)