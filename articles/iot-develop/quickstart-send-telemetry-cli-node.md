---
title: Краткое руководство по отправке телеметрии устройства в Центр Интернета вещей Azure (Node.js)
description: Из этого краткого руководства вы узнаете, как отправлять данные телеметрии с устройства в центр Интернета вещей с помощью пакета SDK для устройств Центра Интернета вещей Azure для Node.js.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.devlang: node
ms.topic: quickstart
ms.date: 03/25/2021
ms.openlocfilehash: 3d42ac814678136c2f6342cd1064e3c3ff394507
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107777245"
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

1. Перейдите в каталог *azure-iot-sdk-node/device/samples/pnp*:

    ```console
    cd azure-iot-sdk-node/device/samples/pnp
    ```

1. Установите пакет SDK для Интернета вещей Azure для Node.js и необходимые зависимости:

    ```console
    npm install
    ```

    Эта команда устанавливает необходимые зависимости, как указано в файле *package.json* в каталоге примеров устройств.

1. Задайте обе указанные ниже переменные среды, чтобы имитированное устройство могло подключаться к Интернету вещей Azure.
    * Задайте переменную среды с именем `IOTHUB_DEVICE_CONNECTION_STRING`. В качестве значения переменной укажите строку подключения устройства, сохраненную при работе с предыдущим разделом.
    * Задайте переменную среды с именем `IOTHUB_DEVICE_SECURITY_TYPE`. В качестве значения переменной используйте строку литерала `connectionString`.

    **Windows (CMD)**

    ```console
    set IOTHUB_DEVICE_CONNECTION_STRING=<your connection string here>
    ```
    ```console
    set IOTHUB_DEVICE_SECURITY_TYPE=connectionString
    ```

    > [!NOTE]
    > В Windows CMD строковые значения для каждой переменной не берутся в кавычки.

    **PowerShell**

    ```azurepowershell
    $env:IOTHUB_DEVICE_CONNECTION_STRING='<your connection string here>'
    ```
    ```azurepowershell
    $env:IOTHUB_DEVICE_SECURITY_TYPE='connectionString'
    ```

    **Bash (Linux или Windows)**

    ```bash
    export IOTHUB_DEVICE_CONNECTION_STRING="<your connection string here>"
    ```
    ```bash
    export IOTHUB_DEVICE_SECURITY_TYPE="connectionString"
    ```
1. В открытой оболочке CLI выполните команду [az iot hub monitor-event](/cli/azure/ext/azure-iot/iot/hub#ext-azure-iot-az-iot-hub-monitor-events), чтобы начать мониторинг событий на имитированном устройстве Интернета вещей.  Сообщения о событиях будут выводиться в терминале по мере поступления.

    ```azurecli
    az iot hub monitor-events --output table --hub-name {YourIoTHubName}
    ```

1. В окне терминала Node.js выполните код для установленного примера файла *simple_thermostat.js*. Этот код обращается к имитированному устройству Интернета вещей и отправляет сообщение в центр Интернета вещей.

    Чтобы выполнить пример Node.js из терминала, используйте следующий код:
    ```console
    node ./simple_thermostat.js
    ```
    > [!NOTE]
    > В этом примере кода используется технология Azure IoT Plug and Play, которая позволяет интегрировать интеллектуальные устройства в решения без какой-либо настройки вручную.  По умолчанию большинство примеров в этой документации используют IoT Plug and Play. Дополнительные сведения о преимуществах технологии IoT Plug and Play, а также о сценариях ее использования или неприменимости см. в статье [Общие сведения об IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md).

Когда код Node.js отправляет смоделированное сообщение телеметрии с устройства в центр Интернета вещей, в оболочке CLI для мониторинга событий отображается сообщение:

```output
Starting event monitor, use ctrl-c to stop...
event:
  component: ''
  interface: dtmi:com:example:Thermostat;1
  module: ''
  origin: <your device ID>
  payload:
    temperature: 36.87027777131555
```

Теперь устройство безопасно подключено и отправляет данные телеметрии в Центр Интернета вещей Azure.

## <a name="clean-up-resources"></a>Очистка ресурсов
Если вам больше не нужны ресурсы Azure, созданные в этом кратком руководстве, используйте Azure CLI, чтобы удалить их.

> [!IMPORTANT]
> Удаление группы ресурсов — процесс необратимый. Группа ресурсов и все содержащиеся в ней ресурсы удаляются без возможности восстановления. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. 

Удаление группы ресурсов по имени:
1. Выполните команду [az group delete](/cli/azure/group#az_group_delete). При этом будут удалены созданные группа ресурсов, центр Интернета вещей и регистрация устройства.

    ```azurecli
    az group delete --name MyResourceGroup
    ```
1. Выполните команду [az group list](/cli/azure/group#az_group_list), чтобы подтвердить удаление группы ресурсов.  

    ```azurecli
    az group list
    ```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали о базовом рабочем процессе разработки приложений Интернета вещей Azure для безопасного подключения устройства к облаку и отправки данных телеметрии с устройства в облако. Вы создали центр Интернета вещей и имитированное устройство с помощью Azure CLI, а затем с помощью пакета SDK для Интернета вещей Azure для Node.js получили доступ к устройству и отправили данные телеметрии в центр Интернета вещей. 

Теперь изучите пакет SDK для Интернета вещей Azure для Node.js на примерах приложений.

- [Другие примеры Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples). Этот каталог содержит дополнительные примеры из репозитория SDK для Node.js для демонстрации сценариев Центра Интернета вещей.