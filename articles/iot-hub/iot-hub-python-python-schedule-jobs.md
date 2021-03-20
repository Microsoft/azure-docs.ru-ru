---
title: Планирование заданий с помощью Центра Интернета вещей Azure (Python) | Документация Майкрософт
description: Планирование заданий с помощью Центра Интернета вещей Azure для вызова прямого метода на нескольких устройствах. Используйте пакеты SDK для Центра Интернета вещей Azure для Python, чтобы реализовать приложения имитации устройства и приложение службы, на которых будет выполнено задание.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 03/17/2020
ms.author: robinsh
ms.custom: devx-track-python
ms.openlocfilehash: 733e3be21a1a1305b5c7947de1ae54ddce5e0d2f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87876688"
---
# <a name="schedule-and-broadcast-jobs-python"></a>Планирование и трансляция заданий (Python)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Центр Интернета вещей Azure — это полностью управляемая служба, которая позволяет внутреннему приложению создавать и отслеживать задания, осуществляющие планирование и обновление миллионов устройств.  Задания можно использовать для следующих действий:

* Обновление требуемых свойств
* Обновление тегов
* Вызов прямых методов

По сути, задание включает одно из этих действий, отслеживая ход его выполнения на наборе устройств (определяется запросом двойника устройства).  Например, с помощью задания внутреннее приложение может вызывать метод перезагрузки на 10 000 устройств, определенных запросом двойника устройства и запланированных в будущем.  Затем это приложение может отследить ход выполнения задания по мере получения и выполнения метода Reboot на каждом из этих устройств.

Дополнительные сведения о каждой из этих возможностей см. в следующих статьях:

* Двойник устройства и свойства: [Начало работы с двойниками устройств](iot-hub-python-twin-getstarted.md) и [Руководство. Использование свойств двойников устройств](tutorial-device-twins.md)

* Прямые методы: [Руководство разработчика для центра Интернета вещей. прямые методы](iot-hub-devguide-direct-methods.md) и [учебник. прямые методы](quickstart-control-device-python.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

В этом учебнике описаны следующие процедуры.

* Создание приложения имитированного устройства Python с прямым методом, который позволяет выполнить действие **lockDoor** путем вызова из серверной части решения.

* Создание консольного приложения для Python, которое с помощью задания вызывает в приложении имитации устройства прямой метод **lockDoor** и обновляет требуемые свойства с помощью задания устройства.

По завершении работы с этим руководством у вас будет два приложения Python.

**simDevice.py**, которое подключается к Центру Интернета вещей с удостоверением устройства и получает прямой метод **lockDoor**.

**scheduleJobService.py**, которое вызывает прямой метод в приложении имитации устройства и обновляет требуемые свойства двойника устройства с помощью задания.

> [!NOTE]
> **Пакет SDK Центра Интернета вещей Azure для Python** непосредственно не поддерживает функциональную возможность **Задания**. Вместо этого в рамках этого руководства предлагается использовать альтернативное решение путем применения асинхронных потоков и таймеров. Последующие выпуски обновлений см. в списке компонентов **клиентских пакетов SDK** на [этой](https://github.com/Azure/azure-iot-sdk-python) странице.
>

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-hub-include-python-v2-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Регистрация нового устройства в центре Интернета вещей

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-simulated-device-app"></a>Создание приложения виртуального устройства

В этом разделе вы создадите консольное приложение Python, отвечающее на прямой метод, вызванный из облака, который активирует имитированный метод **lockDoor**.

1. В командной строке выполните следующую команду, чтобы установить пакет **azure-iot-device**:

    ```cmd/sh
    pip install azure-iot-device
    ```

2. В текстовом редакторе создайте файл **simDevice.py** в рабочей папке.

3. Добавьте следующие инструкции и переменные `import` в начало файла **simDevice.py**. Замените `deviceConnectionString` строкой подключения созданного ранее устройства.

    ```python
    import threading
    import time
    from azure.iot.device import IoTHubDeviceClient, MethodResponse

    CONNECTION_STRING = "{deviceConnectionString}"
    ```

4. Добавьте следующую функцию обратного вызова для обработки метода **lockDoor**.

    ```python
    def lockdoor_listener(client):
        while True:
            # Receive the direct method request
            method_request = client.receive_method_request("lockDoor")  # blocking call
            print( "Locking Door!" )

            resp_status = 200
            resp_payload = {"Response": "lockDoor called successfully"}
            method_response = MethodResponse(method_request.request_id, resp_status, resp_payload)
            client.send_method_response(method_response)
    ```

5. Добавьте другую функцию обратного вызова для обработки обновлений двойников устройства.

    ```python
    def twin_update_listener(client):
        while True:
            patch = client.receive_twin_desired_properties_patch()  # blocking call
            print ("")
            print ("Twin desired properties patch received:")
            print (patch)
    ```

6. Добавьте следующий код, чтобы зарегистрировать обработчик для метода **lockDoor** . Добавьте также подпрограмму `main`.

    ```python
    def iothub_jobs_sample_run():
        try:
            client = IoTHubDeviceClient.create_from_connection_string(CONNECTION_STRING)

            print( "Beginning to listen for 'lockDoor' direct method invocations...")
            lockdoor_listener_thread = threading.Thread(target=lockdoor_listener, args=(client,))
            lockdoor_listener_thread.daemon = True
            lockdoor_listener_thread.start()

            # Begin listening for updates to the Twin desired properties
            print ( "Beginning to listen for updates to Twin desired properties...")
            twin_update_listener_thread = threading.Thread(target=twin_update_listener, args=(client,))
            twin_update_listener_thread.daemon = True
            twin_update_listener_thread.start()
            
            while True:
                time.sleep(1000)

        except KeyboardInterrupt:
            print ( "IoTHubDeviceClient sample stopped" )

    if __name__ == '__main__':
        print ( "Starting the IoT Hub Python jobs sample..." )
        print ( "IoTHubDeviceClient waiting for commands, press Ctrl-C to exit" )

        iothub_jobs_sample_run()
    ```

7. Сохраните и закройте файл **simDevice.py**.

> [!NOTE]
> Для простоты в этом руководстве не реализуются политики повтора. В рабочем коде следует реализовать политики повторных попыток (например, с экспоненциальной задержкой), как указано в статье [Обработка временных сбоев](/azure/architecture/best-practices/transient-faults).
>

## <a name="get-the-iot-hub-connection-string"></a>Получение строки подключения центра Интернета вещей

В этой статье вы создадите серверную службу, которая вызывает прямой метод на устройстве и обновляет двойника устройства. Службе требуется разрешение **Connect** для вызова прямого метода на устройстве. Службе также требуются разрешения на **Чтение реестра** и **запись в реестр** для чтения и записи реестра удостоверений. Политика общего доступа по умолчанию, которая содержит только эти разрешения, не существует, поэтому ее необходимо создать.

Чтобы создать политику общего доступа, которая предоставляет разрешения на подключение **к службе**, **Чтение реестра** и **запись в реестр** , а затем получите строку подключения для этой политики, выполните следующие действия.

1. Откройте центр Интернета вещей в [портал Azure](https://portal.azure.com). Самый простой способ открыть центр Интернета вещей — выбрать **группы ресурсов**, выбрать группу ресурсов, в которой находится центр Интернета вещей, а затем выбрать центр Интернета вещей в списке ресурсов.

2. В левой части центра Интернета вещей выберите **Политики общего доступа**.

3. В верхнем меню над списком политик выберите **Добавить**.

4. В области **Добавить политику общего доступа** введите описательное имя политики. Например: *сервицеандрегистриреадврите*. В разделе **разрешения** выберите **Подключение к службе** и **запись в реестр** (**Чтение реестра** будет автоматически выбрано при выборе **записи в реестр**). Щелкните **Создать**.

    ![Показывает, как добавить новую политику общего доступа](./media/iot-hub-python-python-schedule-jobs/add-policy.png)

5. Вернитесь на панель **политики общего доступа** и выберите в списке политик новую политику.

6. В разделе **Общие ключи доступа** нажмите на значок копирования рядом с пунктом **Строка подключения — первичный ключ** и сохраните значение.

    ![Получение строки подключения](./media/iot-hub-python-python-schedule-jobs/get-connection-string.png)

Дополнительные сведения о политиках и разрешениях общего доступа Центра Интернета вещей см. в разделе [Управления доступом и разрешения](./iot-hub-devguide-security.md#access-control-and-permissions).

## <a name="schedule-jobs-for-calling-a-direct-method-and-updating-a-device-twins-properties"></a>Планирование заданий для вызова прямого метода и обновления свойств двойника устройства

В этом разделе вы создадите консольное приложение Python, которое инициирует удаленный **lockDoor** на устройстве с помощью прямого метода, а также обновляет требуемые свойства двойникаа устройства.

1. В командной строке выполните следующую команду, чтобы установить пакет **azure-iot-hub**:

    ```cmd/sh
    pip install azure-iot-hub
    ```

2. В текстовом редакторе создайте файл **scheduleJobService.py** в рабочей папке.

3. Добавьте следующие `import` инструкции и переменные в начало файла **scheduleJobService.py** . Замените `{IoTHubConnectionString}` заполнитель строкой подключения центра Интернета вещей, скопированным ранее в [поле получение строки подключения для центра Интернета вещей](#get-the-iot-hub-connection-string). Замените `{deviceId}` заполнитель идентификатором устройства, зарегистрированным в окне [Регистрация нового устройства в центре Интернета вещей](#register-a-new-device-in-the-iot-hub):

    ```python
    import sys
    import time
    import threading
    import uuid

    from azure.iot.hub import IoTHubRegistryManager
    from azure.iot.hub.models import Twin, TwinProperties, CloudToDeviceMethod, CloudToDeviceMethodResult, QuerySpecification, QueryResult

    CONNECTION_STRING = "{IoTHubConnectionString}"
    DEVICE_ID = "{deviceId}"

    METHOD_NAME = "lockDoor"
    METHOD_PAYLOAD = "{\"lockTime\":\"10m\"}"
    UPDATE_PATCH = {"building":43,"floor":3}
    TIMEOUT = 60
    WAIT_COUNT = 5
    ```

4. Добавьте следующую функцию, которая используется для отправки запросов на устройства.

    ```python
    def query_condition(iothub_registry_manager, device_id):

        query_spec = QuerySpecification(query="SELECT * FROM devices WHERE deviceId = '{}'".format(device_id))
        query_result = iothub_registry_manager.query_iot_hub(query_spec, None, 1)

        return len(query_result.items)
    ```

5. Добавьте следующие методы для выполнения заданий, вызывающих прямой метод и двойник устройств.

    ```python
    def device_method_job(job_id, device_id, wait_time, execution_time):
        print ( "" )
        print ( "Scheduling job: " + str(job_id) )
        time.sleep(wait_time)

        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)


        if query_condition(iothub_registry_manager, device_id):
            deviceMethod = CloudToDeviceMethod(method_name=METHOD_NAME, payload=METHOD_PAYLOAD)

            response = iothub_registry_manager.invoke_device_method(DEVICE_ID, deviceMethod)

            print ( "" )
            print ( "Direct method " + METHOD_NAME + " called." )

    def device_twin_job(job_id, device_id, wait_time, execution_time):
        print ( "" )
        print ( "Scheduling job " + str(job_id) )
        time.sleep(wait_time)

        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        if query_condition(iothub_registry_manager, device_id):

            twin = iothub_registry_manager.get_twin(DEVICE_ID)
            twin_patch = Twin(properties= TwinProperties(desired=UPDATE_PATCH))
            twin = iothub_registry_manager.update_twin(DEVICE_ID, twin_patch, twin.etag)

            print ( "" )
            print ( "Device twin updated." )
    ```

6. Добавьте следующий код, чтобы запланировать задания и обновить их состояние. Добавьте также подпрограмму `main`.

    ```python
    def iothub_jobs_sample_run():
        try:
            method_thr_id = uuid.uuid4()
            method_thr = threading.Thread(target=device_method_job, args=(method_thr_id, DEVICE_ID, 20, TIMEOUT), kwargs={})
            method_thr.start()

            print ( "" )
            print ( "Direct method called with Job Id: " + str(method_thr_id) )

            twin_thr_id = uuid.uuid4()
            twin_thr = threading.Thread(target=device_twin_job, args=(twin_thr_id, DEVICE_ID, 10, TIMEOUT), kwargs={})
            twin_thr.start()

            print ( "" )
            print ( "Device twin called with Job Id: " + str(twin_thr_id) )

            while True:
                print ( "" )

                if method_thr.is_alive():
                    print ( "...job " + str(method_thr_id) + " still running." )
                else:
                    print ( "...job " + str(method_thr_id) + " complete." )

                if twin_thr.is_alive():
                    print ( "...job " + str(twin_thr_id) + " still running." )
                else:
                    print ( "...job " + str(twin_thr_id) + " complete." )

                print ( "Job status posted, press Ctrl-C to exit" )

                status_counter = 0
                while status_counter <= WAIT_COUNT:
                    time.sleep(1)
                    status_counter += 1

        except Exception as ex:
            print ( "" )
            print ( "Unexpected error {0}" % ex )
            return
        except KeyboardInterrupt:
            print ( "" )
            print ( "IoTHubService sample stopped" )

    if __name__ == '__main__':
        print ( "Starting the IoT Hub jobs Python sample..." )
        print ( "    Connection string = {0}".format(CONNECTION_STRING) )
        print ( "    Device ID         = {0}".format(DEVICE_ID) )

        iothub_jobs_sample_run()
    ```

7. Сохраните и закройте файл **scheduleJobService.py**.

## <a name="run-the-applications"></a>Запуск приложений

Теперь все готово к запуску приложений.

1. В командной строке в рабочей папке выполните следующую команду, чтобы начать прослушивание прямого метода перезагрузки.

    ```cmd/sh
    python simDevice.py
    ```

2. В командной строке в рабочей папке выполните следующую команду, чтобы активировать задачи для блокировки дверей и обновления двойника.
  
    ```cmd/sh
    python scheduleJobService.py
    ```

3. В консоли отобразятся ответы устройства на прямой метод и двойники устройства.

    ![Пример задания центра Интернета вещей 1 — выходные данные устройства](./media/iot-hub-python-python-schedule-jobs/sample1-deviceoutput.png)

    ![Пример 2 задания центра Интернета вещей — выходные данные устройства](./media/iot-hub-python-python-schedule-jobs/sample2-deviceoutput.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике описано использование задания для планирования прямого метода на устройстве и обновления свойств двойника устройства.

Чтобы продолжить знакомство с Центром Интернета вещей и шаблонами управления устройствами, такими как удаленное обновление встроенного ПО, см. [Реализация процесса обновления встроенного ПО](tutorial-firmware-update.md).
