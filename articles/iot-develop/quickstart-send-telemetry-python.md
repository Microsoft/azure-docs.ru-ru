---
title: Краткое руководство по отправке данных телеметрии устройства в Azure IoT Central (Python)
description: Из этого краткого руководства вы узнаете, как отправлять данные телеметрии с устройства в IoT Central с помощью пакета SDK для устройств Центра Интернета вещей Azure для Python.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: python
ms.topic: quickstart
ms.date: 01/11/2021
ms.openlocfilehash: 5d872dd7c94a0b3ab23623bb246ff7ae81609779
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105047173"
---
# <a name="quickstart-send-telemetry-from-a-device-to-azure-iot-central-python"></a>Краткое руководство. Отправка данных телеметрии с устройства в Azure IoT Central (Python)

**Область применения**: [разработка приложений для устройств](about-iot-develop.md#device-application-development)

В этом кратком руководстве описан базовый рабочий процесс разработки приложений для устройств Интернета вещей. Сначала вы создадите облачное приложение с помощью Azure IoT Central. Затем вы примените пакет SDK Интернета вещей Azure для Python, чтобы создать имитированное устройство, подключить его к IoT Central и отправить данные телеметрии с устройства в облако. 

## <a name="prerequisites"></a>Предварительные требования
- [Python версии 3.7 и выше](https://www.python.org/downloads/). Сведения о других поддерживаемых версиях Python см. в разделе о [возможностях устройств Azure IoT](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device#azure-iot-device-features).
    
    Чтобы убедиться, что используется актуальная версия Python, выполните команду `python --version`. Если у вас установлены Python 2 и Python 3 и используется окружение Python 3, установите все библиотеки с помощью `pip3`. Выполнение `pip3` гарантирует, что библиотеки будут установлены в среде выполнения Python 3.
    > [!IMPORTANT]
    > В установщике Python выберите действие **Add Python to PATH** (Добавить Python в путь). Если у вас уже установлен Python версии 3.7 или выше, убедитесь, что вы добавили папку установки Python в переменную среды `PATH`.

## <a name="create-an-application"></a>Создание приложения
При работе с этим разделом вы создадите приложение для IoT Central. IoT Central предоставляет платформу и портал для приложений Интернета вещей, которые упрощают и делают экономичнее разработку решений Интернета вещей и управление ими.

Чтобы создать приложение Azure IoT Central, сделайте следующее:
1. Перейдите к [Azure IoT Central](https://apps.azureiotcentral.com/) и выполните вход с личной, рабочей или учебной учетной записью Майкрософт.
1. Перейдите к разделу **Сборка** и выберите элемент **Пользовательские приложения**.
   :::image type="content" source="media/quickstart-send-telemetry-python/iot-central-build.png" alt-text="Стартовая страница IoT Central":::
1. В поле **Имя приложения** введите уникальное имя или подтвердите автоматически созданный вариант.
1. В поле **URL-адрес** введите удобный для вас префикс URL-адреса или подтвердите автоматически созданный префикс URL-адреса.
1. Для параметра **Шаблон приложения** сохраните значение *Пользовательское приложение*. В раскрывающемся списке могут отображаться другие варианты, если в учетной записи есть еще шаблоны.
1. Выберите вариант **тарифного плана**. 
    - Вы можете бесплатно использовать приложение в течение семи дней, выбрав вариант **Бесплатный**. Вы можете перевести приложение с тарифного плана "Бесплатный" на тарифный план "Стандартный", пока не закончился этот срок действия.
    - Кроме того, можно сразу выбрать тарифный план "Стандартный". Если вы выберете тарифный план "Стандартный", отобразятся дополнительные параметры: **Каталог**, **Подписка Azure** и **Расположение**. Дополнительные сведения о ценах см. на [странице цен на Azure IoT Central](https://azure.microsoft.com/pricing/details/iot-central/). 
        - **Каталог** — Azure Active Directory, в котором будет создано приложение. Azure Active Directory содержит удостоверения пользователей, учетные данные и другие сведения об организации. Если у вас нет каталога Azure Active Directory, он будет создан автоматически при создании подписки Azure.
        - **Подписка Azure** позволяет создавать экземпляры служб Azure. IoT Central подготавливает ресурсы в вашей подписке. Если у вас нет подписки Azure, [создайте ее бесплатно](https://azure.microsoft.com/free/). Завершив создание подписки, вернитесь на страницу IoT Central **Новое приложение**. Созданная подписка отобразится здесь в раскрывающемся списке **Подписка Azure**.
        - **Расположение** обозначает [регион Azure](https://azure.microsoft.com/global-infrastructure/geographies/), в котором вы создаете приложение. Выберите регион, расположенный как можно ближе к устройствам, чтобы обеспечить оптимальную производительность. Выбрав расположение для приложения, вы не сможете его изменить.

    :::image type="content" source="media/quickstart-send-telemetry-python/iot-central-pricing.png" alt-text="Диалог создания приложения IoT Central":::
1. Нажмите кнопку **создания**.
    
    Когда IoT Central завершит создание приложения, вы перейдете к панели мониторинга приложения.
    :::image type="content" source="media/quickstart-send-telemetry-python/iot-central-created.png" alt-text="Панель мониторинга для нового приложения IoT Central":::

## <a name="add-a-device"></a>Добавление устройства
При работе с этим разделом вы добавите новое устройство в приложение IoT Central. Устройством здесь называется экземпляр шаблона устройства, который представляет подключаемое к приложению реальное или имитируемое устройство. 

Чтобы создать новое устройство, сделайте следующее:
1. На панели слева выберите элемент **Устройства**, а затем — команду **+Создать**. Откроется диалоговое окно создания устройства.
1. Для параметра **Шаблон устройства** сохраните значение *Не назначено*.

    > [!NOTE]
    > В рамках этого краткого руководства для простоты подключается имитированное устройство, для которого используется шаблон "Не назначено". Вы познакомитесь с шаблонами устройства позднее, по мере использования IoT Central для управления устройствами. Общие сведения о работе с шаблонами устройств см. в статье [Краткое руководство. Добавление имитированного устройства в приложение IoT Central](../iot-central/core/quick-create-simulated-device.md).
1. Задайте понятное **имя устройства** и произвольный **идентификатор устройства**. Можно просто подтвердить автоматически созданные значения.
    :::image type="content" source="media/quickstart-send-telemetry-python/iot-central-create-device.png" alt-text="Диалоговое окно создания устройства IoT Central":::.
1. Нажмите кнопку **создания**.

    Созданное устройство появится в списке **Все устройства**.
    :::image type="content" source="media/quickstart-send-telemetry-python/iot-central-devices-list.png" alt-text="Полный список устройств IoT Central":::.
    
Чтобы получить сведения о подключении для нового устройства, выполните следующие действия:
1. В списке **Все устройства** дважды щелкните имя связанного устройства, чтобы отобразились сведения. 
1. В верхнем меню выберите **Подключение**.

    В диалоговом окне **Подключение устройства** отобразятся сведения о подключении: :::image type="content" source="media/quickstart-send-telemetry-python/iot-central-device-connect.png" alt-text="Сведения о подключении устройства IoT Central":::
1. Скопируйте в надежное расположение указанные ниже значения из диалогового окна **Подключение устройства**. Они потребуются вам при работе со следующим разделом, чтобы подключить устройство к IoT Central.
    * `ID scope`
    * `Device ID`
    * `Primary key`

## <a name="send-messages-and-monitor-telemetry"></a>Отправка сообщений и мониторинг телеметрии
При работе с этим разделом с помощью пакета SDK для Python вы создадите имитированное устройство и отправите данные телеметрии в приложение IoT Central. 

1. Откройте окно терминала, например Windows CMD, PowerShell или Bash (в среде Windows или Linux). В этом терминале вы установите пакет SDK для Python, обновите переменные среды и выполните пример кода на Python.

1. Скопируйте на локальный компьютер [примеры устройств с пакетом SDK Интернета вещей Azure для Python](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples).

    ```console
    git clone https://github.com/Azure/azure-iot-sdk-python
    ```

1. Перейдите к каталогу *azure-iot-sdk-python/azure-iot-device/samples*.

    ```console
    cd azure-iot-sdk-python/azure-iot-device/samples
    ```
1. Установите пакет SDK Интернета вещей Azure для Python

    ```console
    pip install azure-iot-device
    ```

1. Задайте каждую из указанных ниже переменных среды, чтобы имитированное устройство могло подключаться к IoT Central. Для параметров `ID_SCOPE`, `DEVICE_ID` и `DEVICE_KEY` введите значения, сохраненные ранее из диалогового окна *подключения устройства* к IOT Central.

    **CMD (Windows)**

    ```console
    set PROVISIONING_HOST=global.azure-devices-provisioning.net
    ```
    ```console
    set ID_SCOPE=<your ID scope>
    ```
    ```console
    set DEVICE_ID=<your device ID>
    ```
    ```console
    set DEVICE_KEY=<your device's primary key>
    ```

    > [!NOTE]
    > В командной строке Windows CMD не нужно заключать в кавычки строку подключения или другие значения.

    **PowerShell**

    ```azurepowershell
    $env:PROVISIONING_HOST='global.azure-devices-provisioning.net'
    ```
    ```azurepowershell
    $env:ID_SCOPE='<your ID scope>'
    ```
    ```azurepowershell
    $env:DEVICE_ID='<your device ID>'
    ```
    ```azurepowershell
    $env:DEVICE_KEY='<your device's primary key>'
    ```

    **Bash (Linux или Windows)**

    ```bash
    export PROVISIONING_HOST='global.azure-devices-provisioning.net'
    ```
    ```bash
    export ID_SCOPE='<your ID scope>'
    ```
    ```bash
    export DEVICE_ID='<your device ID>'
    ```
    ```bash
    export DEVICE_KEY='<your device's primary key>'
    ```

1. В окне терминала выполните код для примера файла *simple_send_temperature.py. Этот код обращается к имитированному устройству Интернета вещей и отправляет сообщение в IoT Central.

    Чтобы выполнить этот пример Python в окне терминала, используйте следующий код:
    ```console
    python ./simple_send_temperature.py
    ```

    Кроме того, вы можете выполнить код Python из этого примера в интегрированной среде разработки Python:
    ```python
    import asyncio
    import os
    from azure.iot.device.aio import ProvisioningDeviceClient
    from azure.iot.device.aio import IoTHubDeviceClient
    from azure.iot.device import Message
    import uuid
    import json
    import random

    # ensure environment variables are set for your device and IoT Central application credentials
    provisioning_host = os.getenv("PROVISIONING_HOST")
    id_scope = os.getenv("ID_SCOPE")
    registration_id = os.getenv("DEVICE_ID")
    symmetric_key = os.getenv("DEVICE_KEY")

    # allows the user to quit the program from the terminal
    def stdin_listener():
        """
        Listener for quitting the sample
        """
        while True:
            selection = input("Press Q to quit\n")
            if selection == "Q" or selection == "q":
                print("Quitting...")
                break

    async def main():

        # provisions the device to IoT Central-- this uses the Device Provisioning Service behind the scenes
        provisioning_device_client = ProvisioningDeviceClient.create_from_symmetric_key(
            provisioning_host=provisioning_host,
            registration_id=registration_id,
            id_scope=id_scope,
            symmetric_key=symmetric_key,
        )

        registration_result = await provisioning_device_client.register()

        print("The complete registration result is")
        print(registration_result.registration_state)

        if registration_result.status == "assigned":
            print("Your device has been provisioned. It will now begin sending telemetry.")
            device_client = IoTHubDeviceClient.create_from_symmetric_key(
                symmetric_key=symmetric_key,
                hostname=registration_result.registration_state.assigned_hub,
                device_id=registration_result.registration_state.device_id,
            )

            # Connect the client.
            await device_client.connect()

        # Send the current temperature as a telemetry message
        async def send_telemetry():
            print("Sending telemetry for temperature")

            while True:
                current_temp = random.randrange(10, 50)  # Current temperature in Celsius (randomly generated)
                # Send a single temperature report message
                temperature_msg = {"temperature": current_temp}

                msg = Message(json.dumps(temperature_msg))
                msg.content_encoding = "utf-8"
                msg.content_type = "application/json"
                print("Sent message")
                await device_client.send_message(msg)
                await asyncio.sleep(8)

        send_telemetry_task = asyncio.create_task(send_telemetry())

        # Run the stdin listener in the event loop
        loop = asyncio.get_running_loop()
        user_finished = loop.run_in_executor(None, stdin_listener)
        # Wait for user to indicate they are done listening for method calls
        await user_finished

        send_telemetry_task.cancel()
        # Finally, shut down the client
        await device_client.disconnect()

    if __name__ == "__main__":
        asyncio.run(main())

        # If using Python 3.6 or below, use the following code instead of asyncio.run(main()):
        # loop = asyncio.get_event_loop()
        # loop.run_until_complete(main())
        # loop.close()
    ```

Когда код Python отправляет сообщение с устройства в приложение IoT Central, в IoT Central отображаются сведения о нем на вкладке **Необработанные данные** для устройства. Возможно, потребуется обновить эту страницу, чтобы отобразились последние сообщения.

   :::image type="content" source="media/quickstart-send-telemetry-python/iot-central-telemetry-output.png" alt-text="Снимок экрана: необработанные выходные данные на странице IoT Central":::.

Теперь устройство подключено в безопасном режиме и отправляет данные телеметрии в Интернет вещей Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вам больше не нужны ресурсы IoT Central, созданные при работе с этим руководством, их можно удалить с портала IoT Central. Если же вы планируете продолжить работу с другими документами в рамках этого руководства, сохраните созданное приложение для работы с другими примерами.

Чтобы удалить пример приложения Azure IoT Central, а также все его устройства и ресурсы, выполните следующее.
1. Выберите элементы **Администрирование** >  **[имя приложения]** .
1. Выберите команду **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали о базовом рабочем процессе разработки приложений Интернета вещей Azure для безопасного подключения устройства к облаку и отправки данных телеметрии с устройства в облако. С помощью Azure IoT Central вы создали приложение и устройство, а затем с помощью пакета SDK Интернета вещей Azure для Python создали имитированное устройство и отправили данные телеметрии. Кроме того, вы выполнили мониторинг телеметрии с помощью IoT Central.

Теперь изучите пакет SDK для Интернета вещей Azure для Python на примерах приложений.

- [Асинхронные примеры](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-hub-scenarios). Этот каталог содержит асинхронные примеры Python для дополнительных сценариев Центра Интернета вещей.
- [Синхронные примеры.](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/sync-samples) Этот каталог содержит примеры Python для использования в Python 2.7 или в синхронных сценариях совместимости для Python 3.6 и более поздней версии.
- [Примеры IOT Edge](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-edge-scenarios). Этот каталог содержит примеры Python для работы с модулями Edge и подчиненными устройствами.
