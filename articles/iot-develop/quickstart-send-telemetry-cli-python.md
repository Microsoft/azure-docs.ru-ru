---
title: Краткое руководство по отправке телеметрии устройства в Центр Интернета вещей Azure (Python)
description: Из этого краткого руководства вы узнаете, как отправлять данные телеметрии с устройства в центр Интернета вещей с помощью пакета SDK для устройств Центра Интернета вещей Azure для Python.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: python
ms.topic: quickstart
ms.date: 01/11/2021
ms.openlocfilehash: d73f8eeb7b69440f8db67d0b95b40ed6258ee8e7
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102201793"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-azure-iot-hub-python"></a>Краткое руководство. Отправка данных телеметрии с устройства в Центр Интернета вещей Azure (Python)

**Область применения**: [разработка приложений для устройств](about-iot-develop.md#device-application-development)

В этом кратком руководстве описан базовый рабочий процесс разработки приложений для устройств Интернета вещей. Вы создадите центр Интернета вещей Azure и устройство с помощью Azure CLI, а затем с помощью пакета SDK для Интернета вещей Azure для Python создадите имитированное устройство клиента и отправите данные телеметрии в центр Интернета вещей. 

## <a name="prerequisites"></a>Предварительные требования
- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Azure CLI. Вы можете выполнить все команды в этом кратком руководстве, используя Azure Cloud Shell, интерактивную оболочку CLI, которая работает в браузере. При использовании Cloud Shell вам не нужно ничего устанавливать. Если вы решили использовать CLI локально, для выполнения инструкций, приведенных в этом руководстве вам потребуется Azure CLI 2.0.76 или более поздней версии. Чтобы узнать версию, выполните команду az --version. Чтобы выполнить установку или обновление, см. сведения в статье [Установка Azure CLI]( /cli/azure/install-azure-cli).
- [Python версии 3.7 и выше](https://www.python.org/downloads/). Сведения о других поддерживаемых версиях Python см. в разделе о [возможностях устройств Azure IoT](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device#azure-iot-device-features).
    
    Чтобы убедиться, что используется актуальная версия Python, выполните команду `python --version`. Если у вас установлены Python 2 и Python 3 и используется окружение Python 3, установите все библиотеки с помощью `pip3`. Благодаря этому библиотеки будут установлены в среде выполнения Python 3.
    > [!IMPORTANT]
    > В установщике Python выберите действие **Add Python to PATH** (Добавить Python в путь). Если у вас уже установлен Python версии 3.7 или выше, убедитесь, что вы добавили папку установки Python в переменную среды `PATH`.

[!INCLUDE [iot-hub-include-create-hub-cli](../../includes/iot-hub-include-create-hub-cli.md)]

## <a name="use-the-python-sdk-to-send-messages"></a>Использование пакета SDK для Python для отправки сообщений
Используя сведения этого раздела вы научитесь отправлять сообщения с имитированного устройства в центр Интернета вещей с помощью пакета SDK для Python.

1. Откройте новое окно терминала. Этот терминал будет использоваться для установки пакета SDK для Python и работы с примером кода Python. Теперь у вас должно быть открыто два терминала: тот, который вы только что открыли для работы с Python, и оболочка CLI, которая использовалась в предыдущих разделах для ввода команд Azure CLI.       

1. Скопируйте [примеры устройств с пакетом SDK для Центра Интернета вещей Azure для Python](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples) на локальный компьютер:

    ```console
    git clone https://github.com/Azure/azure-iot-sdk-python
    ```

    перейдите к каталогу *azure-iot-sdk-python/azure-iot-device/samples*:

    ```console
    cd azure-iot-sdk-python/azure-iot-device/samples
    ```
1. Установка пакета SDK Интернета вещей Azure для Python

    ```console
    pip install azure-iot-device
    ```
1. Задайте строку подключения устройства как переменную среды с именем `IOTHUB_DEVICE_CONNECTION_STRING`. Это строка, полученная в предыдущем разделе после создания имитированного устройства Python.

    **Windows (CMD)**

    ```console
    set IOTHUB_DEVICE_CONNECTION_STRING=<your connection string here>
    ```

    > [!NOTE]
    > В Windows CMD строка подключения не берется в кавычки.

    **Linux (Bash)**

    ```bash
    export IOTHUB_DEVICE_CONNECTION_STRING="<your connection string here>"
    ```

1. В открытой оболочке CLI выполните команду [az iot hub monitor-event](/cli/azure/ext/azure-iot/iot/hub#ext-azure-iot-az-iot-hub-monitor-events), чтобы начать мониторинг событий на имитированном устройстве Интернета вещей.  Сообщения о событиях будут выводиться в терминале по мере поступления.

    ```azurecli
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```

1. В окне терминала Python выполните код для установленного примера файла *simple_send_message.py*. Этот код обращается к имитированному устройству Интернета вещей и отправляет сообщение в центр Интернета вещей.

    Чтобы выполнить этот пример Python в окне терминала, используйте следующий код:
    ```console
    python ./simple_send_message.py
    ```

    Кроме того, вы можете выполнить код Python из этого примера в интегрированной среде разработки Python:
    ```python
    import os
    import asyncio
    from azure.iot.device.aio import IoTHubDeviceClient


    async def main():
        # Fetch the connection string from an environment variable
        conn_str = os.getenv("IOTHUB_DEVICE_CONNECTION_STRING")

        # Create instance of the device client using the authentication provider
        device_client = IoTHubDeviceClient.create_from_connection_string(conn_str)

        # Connect the device client.
        await device_client.connect()

        # Send a single message
        print("Sending message...")
        await device_client.send_message("This is a message that is being sent")
        print("Message successfully sent!")

        # finally, disconnect
        await device_client.disconnect()


    if __name__ == "__main__":
        asyncio.run(main())
    ```

Когда код Python отправляет сообщение с устройства в центр Интернета вещей, в оболочке CLI для мониторинга событий отображается сообщение:

```output
Starting event monitor, use ctrl-c to stop...
event:
origin: <your Device name>
payload: This is a message that is being sent
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
Из этого краткого руководства вы узнали о базовом рабочем процессе разработки приложений Интернета вещей Azure для безопасного подключения устройства к облаку и отправки данных телеметрии с устройства в облако. Вы создали центр Интернета вещей и устройство с помощью Azure CLI, а затем с помощью пакета SDK для Интернета вещей Azure для Python создали имитированное устройство и отправили данные телеметрии в центр Интернета вещей. 

Теперь изучите пакет SDK для Интернета вещей Azure для Python на примерах приложений.

- [Асинхронные примеры](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-hub-scenarios). Этот каталог содержит асинхронные примеры Python для дополнительных сценариев Центра Интернета вещей.
- [Синхронные примеры](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/sync-samples). Этот каталог содержит примеры Python для использования в Python 2.7 или в синхронных сценариях совместимости для Python 3.5+.
- [Примеры IOT Edge](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-edge-scenarios). Этот каталог содержит примеры Python для работы с модулями Edge и подчиненными устройствами.