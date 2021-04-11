---
title: Краткое руководство. Отправка данных телеметрии в Центр Интернета вещей Azure с помощью Java
description: 'В этом кратком руководстве мы будем использовать два примера приложений Java: для отправки имитированных данных телеметрии в Центр Интернета вещей и для чтения данных телеметрии из Центра Интернета вещей для последующей обработки в облаке.'
author: wesmc7777
manager: lizross
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: quickstart
ms.custom:
- mvc
- seo-java-august2019
- seo-java-september2019
- mqtt
- devx-track-java
- devx-track-azurecli
ms.date: 01/27/2021
ms.openlocfilehash: bc5a6583cc6f47a986379cc071a056751dede59c
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106066965"
---
# <a name="quickstart-send-telemetry-to-an-azure-iot-hub-and-read-it-with-a-java-application"></a>Краткое руководство. Отправка данных телеметрии в Центр Интернета вещей и их чтение с помощью приложения Java

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

В этом руководстве описано, как отправлять данные телеметрии в Центр Интернета вещей Azure и считывать их с помощью приложения Java. Центр Интернета вещей — это служба Azure, которая позволяет получать большие объемы телеметрии с ваших устройств Центра Интернета вещей в облаке на хранение или обработку. В этом кратком руководстве используется два заранее разработанных приложения Java: одно для отправки данных телеметрии, а другое для чтения телеметрии из центра. Прежде чем запускать эти приложения, создайте Центр Интернета вещей и зарегистрируйте устройство в центре.

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* Пакет SDK для Java SE 8. В статье [Долгосрочная поддержка Java для Azure и Azure Stack](/java/azure/jdk/) выберите пункт **Java 8** в разделе **Долгосрочная поддержка**.

    Текущую версию Java на компьютере, на котором ведется разработка, можно проверить, используя следующую команду:

    ```cmd/sh
    java -version
    ```

* [Apache Maven 3](https://maven.apache.org/download.cgi).

    Текущую версию Maven на компьютере, на котором ведется разработка, можно проверить, используя следующую команду:

    ```cmd/sh
    mvn --version
    ```

* Скачайте или клонируйте репозиторий azure-iot-samples-java с помощью кнопки **Code** (Код) на странице репозитория [azure-iot-samples-java](https://github.com/Azure-Samples/azure-iot-samples-java). 

    В этой статье используются примеры [simulated-device](https://github.com/Azure-Samples/azure-iot-samples-java/tree/master/iot-hub/Quickstarts/simulated-device) и [read-d2c-messages](https://github.com/Azure-Samples/azure-iot-samples-java/tree/master/iot-hub/Quickstarts/read-d2c-messages) из этого репозитория.

* Порт 8883, открытый в брандмауэре. Пример устройства в этом кратком руководстве использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Регистрация устройства

Устройство должно быть зарегистрировано в Центре Интернета вещей, прежде чем оно сможет подключиться. В этом кратком руководстве для регистрации имитируемого устройства используется Azure Cloud Shell.

1. Выполните приведенные ниже команды в Azure Cloud Shell, чтобы создать удостоверение устройства.

   **YourIoTHubName**. Замените этот заполнитель именем вашего центра Интернета вещей.

   **MyJavaDevice**. Это имя регистрируемого устройства. Рекомендуется использовать **MyJavaDevice**, как показано ниже. Если вы выбрали другое имя для устройства, используйте его при работе с этим руководством и обновите имя устройства в примерах приложений перед их запуском.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyJavaDevice
    ```

2. Выполните следующую команду в Azure Cloud Shell, чтобы получить _строку подключения_ зарегистрированного устройства:

    **YourIoTHubName**. Замените этот заполнитель именем вашего центра Интернета вещей.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name {YourIoTHubName} --device-id MyJavaDevice --output table
    ```

    Запишите строку подключения устройства, которая выглядит так:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyJavaDevice;SharedAccessKey={YourSharedAccessKey}`

    Это значение понадобится позже при работе с этим кратким руководством.

3. Вам также понадобится _конечная точка, совместимая с Центрами событий_, _путь, совместимый с Центрами событий_, и _первичный ключ службы_ из Центра Интернета вещей, чтобы подключить внутреннее приложение к Центру Интернета вещей и получить сообщения. Следующие команды позволяют получить эти значения для Центра Интернета вещей:

     **YourIoTHubName**. Замените этот заполнитель именем вашего центра Интернета вещей.

    ```azurecli-interactive
    az iot hub show --query properties.eventHubEndpoints.events.endpoint --name {YourIoTHubName}

    az iot hub show --query properties.eventHubEndpoints.events.path --name {YourIoTHubName}

    az iot hub policy show --name service --query primaryKey --hub-name {YourIoTHubName}
    ```

    Запишите эти три значения, они понадобятся позже при работе с этим кратким руководством.

## <a name="send-simulated-telemetry"></a>Отправка имитированной телеметрии

Приложение имитированного устройства подключается к конечной точке конкретного устройства Центра Интернета вещей и отправляет имитированную телеметрию температуры и влажности.

1. В окне терминала на локальном компьютере перейдите в корневую папку примера проекта Java. Затем перейдите в папку **iot-hub\Quickstarts\simulated-device**.

2. Откройте файл **src/main/java/com/microsoft/docs/iothub/samples/SimulatedDevice.java** в любом текстовом редакторе.

    Замените значение переменной `connString` записанной ранее строкой подключения к устройству. Сохраните изменения в файле **SimulatedDevice.java**.

    ```java
    public class SimulatedDevice {
      // The device connection string to authenticate the device with your IoT hub.
      // Using the Azure CLI:
      // az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyJavaDevice --output table

      //private static String connString = "{Your device connection string here}";    
      private static String connString = "HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyJavaDevice;SharedAccessKey={YourSharedAccessKey}";    
     ```

3. Установите необходимые библиотеки и создайте приложение имитированного устройства, выполнив в окне локального терминала следующие команды:

    ```cmd/sh
    mvn clean package
    ```

4. Запустите приложение имитированного устройства, выполнив в окне терминала на локальном компьютере следующие команды:

    ```cmd/sh
    java -jar target/simulated-device-1.0.0-with-deps.jar
    ```

    На следующем снимке экрана показан пример выходных данных, когда приложение имитированного устройства отправляет данные телеметрии в Центр Интернета вещей:

    ![Данные телеметрии, переданные устройством в центр Интернета вещей](media/quickstart-send-telemetry-java/simulated-device.png)

## <a name="read-the-telemetry-from-your-hub"></a>Чтение данных телеметрии из концентратора

Внутреннее приложение подключается к конечной точке на стороне службы **События** в Центре Интернета вещей. Приложение получает сообщения с устройства в облако, отправленные с имитированного устройства. Внутреннее приложение Центра Интернета вещей обычно запускается в облаке, чтобы получать сообщения с устройства в облако и обрабатывать их.

1. В другом окне терминала на локальном компьютере перейдите в корневую папку примера проекта Java. Затем перейдите в папку **iot-hub\Quickstarts\read-d2c-messages**.

2. Откройте файл **src/main/java/com/microsoft/docs/iothub/samples/ReadDeviceToCloudMessages.java** в любом текстовом редакторе. Обновите указанные ниже переменные и сохраните изменения в файле.

    | Переменная | Значение |
    | -------- | ----------- |
    | `EVENT_HUBS_COMPATIBLE_ENDPOINT` | Замените значение переменной записанной ранее конечной точкой, совместимой с Центрами событий. |
    | `EVENT_HUBS_COMPATIBLE_PATH`     | Замените значение переменной записанным ранее путем, совместимым с Центрами событий. |
    | `IOT_HUB_SAS_KEY`                | Замените значение переменной записанным ранее первичным ключом службы. |

    ```java
    public class ReadDeviceToCloudMessages {
    
      private static final String EH_COMPATIBLE_CONNECTION_STRING_FORMAT = "Endpoint=%s/;EntityPath=%s;"
          + "SharedAccessKeyName=%s;SharedAccessKey=%s";
    
      // az iot hub show --query properties.eventHubEndpoints.events.endpoint --name {your IoT Hub name}
      private static final String EVENT_HUBS_COMPATIBLE_ENDPOINT = "{your Event Hubs compatible endpoint}";
    
      // az iot hub show --query properties.eventHubEndpoints.events.path --name {your IoT Hub name}
      private static final String EVENT_HUBS_COMPATIBLE_PATH = "{your Event Hubs compatible name}";
    
      // az iot hub policy show --name service --query primaryKey --hub-name {your IoT Hub name}
      private static final String IOT_HUB_SAS_KEY = "{your service primary key}";    
    ```


3. Установите необходимые библиотеки и создайте внутреннее приложение, выполнив в окне терминала на локальном компьютере следующие команды:

    ```cmd/sh
    mvn clean package
    ```

4. Запустите внутреннее приложение, выполнив в окне терминала на локальном компьютере следующие команды:

    ```cmd/sh
    java -jar target/read-d2c-messages-1.0.0-with-deps.jar
    ```

    На следующем снимке экрана показан пример выходных данных, когда внутреннее приложение получает данные телеметрии, отправленные в концентратор имитированным устройством:

    ![Выходные данные после получения серверным приложением данных телеметрии, отправляемых в центр Интернета вещей](media/quickstart-send-telemetry-java/read-device-to-cloud.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы настроили Центр Интернета вещей, зарегистрировали устройство, отправили имитированные данные телеметрии в центр с помощью приложения Java, а также считали данные телеметрии из центра, используя простое внутреннее приложение.

Чтобы узнать, как управлять имитированным устройством из внутреннего приложения, перейдите к следующему краткому руководству.

> [!div class="nextstepaction"]
> [Краткое руководство. Управление подключенным к Центру Интернета вещей устройством](quickstart-control-device-java.md)
