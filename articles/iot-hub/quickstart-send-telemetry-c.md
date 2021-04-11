---
title: Краткое руководство. Отправка данных телеметрии в Центр Интернета вещей Azure (C) | Документация Майкрософт
description: 'В этом кратком руководстве мы будем использовать два примера приложений C: для отправки имитированных данных телеметрии в Центр Интернета вещей и для чтения данных телеметрии из Центра Интернета вещей для последующей обработки в облаке.'
author: wesmc7777
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
- devx-track-azurecli
ms.date: 04/10/2019
ms.author: wesmc
ms.openlocfilehash: 6cc20a362ceebe56beb0d2baa48ea479d3b322a7
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106067651"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-and-read-it-with-a-back-end-application-c"></a>Краткое руководство. Отправка данных телеметрии из устройства в Центр Интернета вещей и их чтение с помощью внутреннего приложения (C)

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

Центр Интернета вещей — это служба Azure, которая позволяет получать большие объемы телеметрии с ваших устройств Центра Интернета вещей в облаке на хранение или обработку. В рамках этого краткого руководства вы отправите данные телеметрии из приложения имитированного устройства через Центр Интернета вещей во внутреннее приложение для обработки.

В этом кратком руководстве используется пример приложения C из [пакета SDK для устройств Azure IoT для C](iot-hub-device-sdk-c-intro.md) для отправки данных телеметрии в центр Интернета вещей. Пакеты SDK для устройств Azure IoT написаны в соответствии со стандартом [ANSI C (C99)](https://wikipedia.org/wiki/C99) для обеспечения переносимости и совместимости с широким диапазоном платформ. Прежде чем запустить пример кода, вы создадите центр Интернета вещей и зарегистрируете в нем имитированное устройство.

Эта статья написана для Windows. Но вы можете выполнить задачи из этого краткого руководства и на платформе Linux.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* Установите [Visual Studio 2019](https://www.visualstudio.com/vs/) с включенной рабочей нагрузкой [Разработка классических приложений на C++](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/).

* Установите последнюю версию [Git](https://git-scm.com/download/).

* Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этом кратком руководстве использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="prepare-the-development-environment"></a>Подготовка среды разработки

При работе с этим кратким руководством вы будете использовать [пакет SDK для устройств Azure IoT для C](iot-hub-device-sdk-c-intro.md). 

Для указанных ниже сред можно использовать этот пакет SDK, установив следующие пакеты и библиотеки:

* **Linux**: доступны пакеты apt-get для Ubuntu 16.04 и 18.04 на основе следующих архитектур ЦП: amd64, arm64, armhf и i386. Дополнительные сведения см. в разделе [Using apt-get to create a C device client project on Ubuntu](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/ubuntu_apt-get_sample_setup.md) (Создание проекта клиента для устройств на C в Ubuntu с помощью apt-get).

* **mbed**: для разработчиков, создающих приложения для устройств на платформе mbed, мы опубликовали библиотеку и примеры, которые помогут всего за несколько минут приступить к работе с Центром Интернета вещей Azure. Дополнительные сведения см. в разделе [Use the mbed library](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#mbed) (Использование библиотеки mbed).

* **Arduino**: если вы разрабатываете приложения на платформе Arduino, то вы можете использовать библиотеку Azure IoT, доступную в диспетчере библиотек интегрированной среды разработки Arduino. Дополнительные сведения см. в разделе [The Azure IoT Hub library for Arduino](https://github.com/azure/azure-iot-arduino) (Библиотека Центра Интернета вещей Azure для Arduino).

* **iOS**: пакет SDK для устройств центра Интернета вещей доступен в CocoaPods для разработки для устройств Mac и iOS. Дополнительные сведения можно найти в разделе [iOS Samples for Microsoft Azure IoT](https://cocoapods.org/pods/AzureIoTHubClient) (Примеры iOS для Центра Интернета вещей Microsoft Azure).

Тем не менее с помощью этого краткого руководства вы подготовите среду разработки, которая используется для клонирования и сборки [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c) с сайта GitHub. Пакет SDK на сайте GitHub содержит пример кода, используемый в этом кратком руководстве.

1. Скачайте [систему сборки CMake](https://cmake.org/download/).

    **Перед** установкой `CMake` очень важно установить на компьютер необходимые компоненты Visual Studio (Visual Studio с рабочей нагрузкой "Разработка классических приложений на C++"). После установки компонентов и проверки загрузки установите систему сборки CMake.

2. Найдите имя тега для [последнего выпуска](https://github.com/Azure/azure-iot-sdk-c/releases/latest) пакета SDK.

3. Откройте командную строку или оболочку Git Bash. Выполните приведенные ниже команды, чтобы клонировать репозиторий GitHub с последним выпуском [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c). Используйте найденный тег в качестве значения для параметра `-b`:

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Выполнение этой операции может занять несколько минут.

4. Создайте подкаталог `cmake` в корневом каталоге репозитория Git и перейдите в эту папку. В каталоге `azure-iot-sdk-c` выполните следующие команды:

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Выполните приведенную ниже команду, чтобы создать версию пакета SDK для используемой клиентской платформы разработки. Эта команда также создает решение Visual Studio для имитированного устройства в каталоге `cmake`.

    ```cmd
    cmake ..
    ```

    Если `cmake` не удается найти компилятор C++, во время выполнения предыдущей команды могут возникнуть ошибки сборки. В этом случае попробуйте, выполнить эту команду в [командной строке Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs). 

    После успешного создания последние несколько строк выходных данных будут выглядеть следующим образом:

    ```cmd/sh
    $ cmake ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Регистрация устройства

Устройство должно быть зарегистрировано в Центре Интернета вещей, прежде чем оно сможет подключиться. В этом разделе описано, как зарегистрировать имитированное устройство с помощью [расширения Интернета вещей](/cli/azure/ext/azure-iot/iot) для Azure Cloud Shell.

1. Выполните приведенные ниже команды в Azure Cloud Shell, чтобы создать удостоверение устройства.

   **YourIoTHubName**. Замените этот заполнитель именем вашего центра Интернета вещей.

   **MyCDevice**. Это имя регистрируемого устройства. Рекомендуется использовать **MyCDevice**, как показано ниже. Если вы выбрали другое имя для устройства, используйте его при работе с этим руководством и обновите имя устройства в примерах приложений перед их запуском.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyCDevice
    ```

2. Выполните следующую команду в Azure Cloud Shell, чтобы получить _строку подключения_ зарегистрированного устройства:

   **YourIoTHubName**. Замените этот заполнитель именем вашего центра Интернета вещей.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name {YourIoTHubName} --device-id MyCDevice --output table
    ```

    Запишите строку подключения устройства, которая выглядит так:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyCDevice;SharedAccessKey={YourSharedAccessKey}`

    Это значение понадобится позже при работе с этим кратким руководством.

## <a name="send-simulated-telemetry"></a>Отправка имитированной телеметрии

Приложение имитированного устройства подключается к конечной точке конкретного устройства в центре Интернета вещей и отправляет имитированную телеметрию в виде строки.

1. В текстовом редакторе откройте исходный файл iothub_convenience_sample.c и просмотрите пример кода для отправки данных телеметрии. Файл находится в следующем расположении в рабочем каталоге, в который вы клонировали пакет SDK для Azure IoT C:

    ```
    azure-iot-sdk-c\iothub_client\samples\iothub_convenience_sample\iothub_convenience_sample.c
    ```

2. Найдите объявление константы `connectionString`.

    ```C
    /* Paste in your device connection string  */
    static const char* connectionString = "[device connection string]";
    ```

    Замените значение константы `connectionString` записанной ранее строкой подключения к устройству. Затем сохраните изменения, внесенные в файл **iothub_convenience_sample.c**.

3. В окне терминала на локальном компьютере перейдите в каталог проекта *iothub_convenience_sample* в каталоге CMake, созданном в пакете SDK Интернета вещей Azure для C. Введите следующие команды из рабочего каталога:

    ```cmd/sh
    cd azure-iot-sdk-c/cmake/iothub_client/samples/iothub_convenience_sample
    ```

4. Запустите CMake в окне терминала на локальном компьютере, чтобы выполнить сборку примера с обновленным значением `connectionString`:

    ```cmd/sh
    cmake --build . --target iothub_convenience_sample --config Debug
    ```

5. Запустите приложение имитированного устройства, выполнив в окне терминала на локальном компьютере следующие команды:

    ```cmd/sh
    Debug\iothub_convenience_sample.exe
    ```

    На следующем снимке экрана показан пример выходных данных, когда приложение имитированного устройства отправляет данные телеметрии в центр Интернета вещей.

    ![Запуск виртуального устройства](media/quickstart-send-telemetry-c/simulated-device-app.png)

## <a name="read-the-telemetry-from-your-hub"></a>Чтение данных телеметрии из концентратора

В этом разделе описано, как использовать Azure Cloud Shell с [расширением Интернета вещей](/cli/azure/ext/azure-iot/iot) для мониторинга сообщений, отправляемых имитированным устройством.

1. С помощью Azure Cloud Shell выполните следующую команду для установки подключения к центру Интернета вещей и чтения поступающих из него сообщений:

   **YourIoTHubName**. Замените этот заполнитель именем вашего Центра Интернета вещей.

    ```azurecli-interactive
    az iot hub monitor-events --hub-name {YourIoTHubName} --output table
    ```

    ![Чтение сообщений устройства с помощью Azure CLI](media/quickstart-send-telemetry-c/read-device-to-cloud-messages-app.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим кратким руководством вы настроили Центр Интернета вещей, зарегистрировали устройство, отправили имитированные данные телеметрии в центр с помощью приложения C, а также считали данные телеметрии из центра, используя Azure Cloud Shell.

Чтобы узнать больше о разработке с помощью пакета SDK Центра Интернета вещей Azure для C, перейдите к следующему практическому руководству:

> [!div class="nextstepaction"]
> [Разработка с использованием пакета SDK для Центра Интернета вещей для C](iot-hub-devguide-develop-for-constrained-devices.md)
