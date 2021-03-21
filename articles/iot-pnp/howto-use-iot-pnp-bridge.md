---
title: Как подключить пример моста IoT самонастраивающийся, работающий в Linux или Windows, в центр Интернета вещей | Документация Майкрософт
description: Создавайте и запускайте мост IoT самонастраивающийся в Linux или Windows, который подключается к центру Интернета вещей. С помощью обозревателя Интернета вещей Azure просматривайте сведения, отправленные устройством в центр.
author: usivagna
ms.author: ugans
ms.date: 12/11/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 9bcf256b6144702254bbff4a57e5ff402abaa962
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99834108"
---
# <a name="how-to-connect-an--iot-plug-and-play-bridge-sample-running-on-linux-or-windows-to-iot-hub"></a>Как подключить пример моста IoT самонастраивающийся под управлением Linux или Windows в центр Интернета вещей

В этой статье показано, как создать пример адаптера окружающей среды для центра Интернета вещей самонастраивающийся, подключить его к центру Интернета вещей и использовать средство Azure IoT Explorer для просмотра отправляемых данных телеметрии. Мост IoT самонастраивающийся написан на языке C и включает пакет SDK для устройств Azure IoT для C. По завершении работы с этим руководством вы сможете запустить мост IoT самонастраивающийся и просмотреть данные телеметрии отчетов в ИТ в Azure IoT Explorer:

:::image type="content" source="media/concepts-iot-pnp-bridge/iot-pnp-bridge-explorer-telemetry.png" alt-text="Снимок экрана, показывающий обозреватель Интернета вещей Azure с таблицей данных телеметрии (влажности, температура) из центра Интернета вещей самонастраивающийся Bridge.":::

## <a name="prerequisites"></a>Предварительные условия

Пример можно запустить в статье в Windows или Linux. Команды оболочки, приведенные в этом пошаговом руководство, соответствуют соглашению Windows для разделителей пути " `\` ". Если вы используете Linux, обязательно замените эти разделители для " `/` ".

### <a name="azure-iot-explorer"></a>Обозреватель Интернета вещей Azure

Для взаимодействия с примером устройства во второй части этой статьи используется средство **Azure IOT Explorer** . [Скачайте и установите последний выпуск обозревателя Интернета вещей Azure](./howto-use-iot-explorer.md) для вашей операционной системы.

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Выполните следующую команду, чтобы получить _строку подключения центра Интернета вещей_ для центра. Запишите эту строку подключения, которая будет использоваться позже в этой статье:

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

Выполните указанную ниже команду, чтобы получить _строку подключения устройства_, добавленного в центр. Запишите эту строку подключения, которая будет использоваться позже в этой статье:

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output table
```

## <a name="download-and-run-the-bridge"></a>Скачивание и запуск моста

В этой статье у вас есть два варианта запуска моста. Вы можете:

- Скачайте готовый исполняемый файл и запустите его, как описано в этом разделе.
- Скачайте исходный код, а затем [выполните сборку и запуск моста](#build-and-run-the-bridge) , как описано в следующем разделе.

Чтобы скачать и запустить мост, выполните следующие действия.

1. Перейдите на [страницу Самонастраивающийся выпусков](https://github.com/Azure/iot-plug-and-play-bridge/releases)IOT.
1. Скачайте предварительно созданный исполняемый файл для операционной системы: **pnpbridge_bin.exe** для Windows или **pnpbridge_bin** для Linux.
1. Скачайте пример [config.jsв](https://raw.githubusercontent.com/Azure/iot-plug-and-play-bridge/master/pnpbridge/src/adapters/samples/environmental_sensor/config.json) файле конфигурации для примера "датчик окружающей среды". Убедитесь, что файл конфигурации находится в той же папке, что и исполняемый объект.
1. Измените *config.jsв* файле:

    - Добавьте `connection-string` значение, которое является _строкой подключения устройства_ , которое вы записали ранее.
    - Добавьте `symmetric_key` значение, которое является значением общего ключа доступа, из _строки подключения устройства_.
    - Замените `root_interface_model_id` значение на `dtmi:com:example:PnpBridgeEnvironmentalSensor;1` .

    Первый раздел файла *config.jsв* файле теперь выглядит как следующий фрагмент кода:

    ```json
    {
      "$schema": "../../../pnpbridge/src/pnpbridge_config_schema.json",
      "pnp_bridge_connection_parameters": {
        "connection_type" : "connection_string",
        "connection_string" : "HostName=youriothub.azure-devices.net;DeviceId=yourdevice;SharedAccessKey=TTrz8fR7ylHKt7DC/e/e2xocCa5VIcq5x9iQKxKFVa8=",
        "root_interface_model_id": "dtmi:com:example:PnpBridgeEnvironmentalSensor;1",
        "auth_parameters": {
            "auth_type": "symmetric_key",
            "symmetric_key": "TTrz8fR7ylHKt7DC/e/e2xocCa5VIcq5x9iQKxKFVa8="
        },
    ```

1. Запустите исполняемый файл в среде командной строки. Мост создает выходные данные, которые выглядят следующим образом:

    ```output
    c:\temp\temp-bridge>dir
     Volume in drive C is OSDisk
     Volume Serial Number is 38F7-DA4A
    
     Directory of c:\temp\temp-bridge
    
    10/12/2020  12:24    <DIR>          .
    10/12/2020  12:24    <DIR>          ..
    08/12/2020  15:26             1,216 config.json
    10/12/2020  12:21         3,617,280 pnpbridge_bin.exe
                   2 File(s)      3,618,496 bytes
                   2 Dir(s)  12,999,147,520 bytes free
    
    c:\temp\temp-bridge>pnpbridge_bin.exe
    Info:
     -- Press Ctrl+C to stop PnpBridge
    
    Info: Using default configuration location
    Info: Starting Azure PnpBridge
    Info: Pnp Bridge is running as am IoT egde device.
    Info: Pnp Bridge creation succeeded.
    Info: Connection_type is [connection_string]
    Info: Tracing is disabled
    Info: WARNING: SharedAccessKey is included in connection string. Ignoring symmetric_key in config file.
    Info: IoT Edge Device configuration initialized successfully
    Info: Building Pnp Bridge Adapter Manager, Adapters & Components
    Info: Adapter with identity environment-sensor-sample-pnp-adapter does not have any associated global parameters. Proceeding with adapter creation.
    Info: Pnp Adapter with adapter ID environment-sensor-sample-pnp-adapter has been created.
    Info: Pnp Adapter Manager created successfully.
    Info: Pnp components created successfully.
    Info: Pnp components built in model successfully.
    Info: Connected to Azure IoT Hub
    Info: Environmental Sensor: Starting Pnp Component
    Info: IoTHub client call to _SendReportedState succeeded
    Info: Environmental Sensor Adapter:: Sending device information property to IoTHub. propertyName=state, propertyValue=true
    Info: Pnp components started successfully.
    ```

## <a name="build-and-run-the-bridge"></a>Сборка и запуск моста

Если вы предпочитаете самостоятельно создать исполняемый файл, можно скачать исходный код и скрипты сборки.

Откройте командную строку в выбранной папке. Выполните следующую команду, чтобы клонировать репозиторий Azure [IoT Самонастраивающийся Bridge](https://github.com/Azure/iot-plug-and-play-bridge) в это расположение:

```cmd
git clone https://github.com/Azure/iot-plug-and-play-bridge.git
```

После клонирования репозитория обновите подмодули. Подмодули включают пакет SDK для Azure IoT для C:

```cmd
cd iot-plug-and-play-bridge
git submodule update --init --recursive
```

Выполнение этой операции может занять несколько минут.

> [!TIP]
> Если возникли проблемы при сбое обновления модуля подсистемы клонирования Git, это известная проблема с путями файлов Windows. Чтобы устранить эту проблему, можно попробовать выполнить следующую команду: `git config --system core.longpaths true`

Необходимые условия для создания моста отличаются операционной системой.

### <a name="windows"></a>Windows

Чтобы создать мост IoT самонастраивающийся в Windows, установите следующее программное обеспечение:

* [Visual Studio (Community, Professional или Enterprise)](https://visualstudio.microsoft.com/downloads/). При [установке](/cpp/build/vscpp-step-0-installation?preserve-view=true&view=vs-2019) Visual Studio обязательно добавьте рабочую нагрузку **Разработка классических приложений на C++** .
* [Git](https://git-scm.com/download/).
* [CMake](https://cmake.org/download/).

### <a name="linux"></a>Linux

В этой статье предполагается, что вы используете Ubuntu Linux. Действия, описанные в этой статье, были протестированы с помощью Ubuntu 18,04.

Чтобы создать мост IoT самонастраивающийся в Linux, установите **GCC**, **Git**, **CMAK** и все необходимые зависимости с помощью `apt-get` команды:

```sh
sudo apt-get update
sudo apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
```

Убедитесь, что версия `cmake` выше **2.8.12**, а версия **GCC** выше **4.4.7**.

```sh
cmake --version
gcc --version
```

### <a name="build-the-iot-plug-and-play-bridge"></a>Создание моста самонастраивающийся IoT

Перейдите в папку *пнпбридже* в каталоге репозитория.

Для Windows выполните следующие действия в [Командная строка разработчика для Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs):

```cmd
cd scripts\windows
build.cmd
```

Аналогичным образом для Linux выполните следующие действия:

```bash
cd scripts/linux
./setup.sh
./build.sh
```

>[!TIP]
>В Windows можно открыть решение, созданное с помощью команды CMAK в Visual Studio 2019. Откройте файл проекта *azure_iot_pnp_bridge. sln* в каталоге CMAK и задайте проект *pnpbridge_bin* в качестве запускаемого проекта в решении. Теперь можно создать пример в Visual Studio и запустить его в режиме отладки.

### <a name="edit-the-configuration-file"></a>Изменение файла конфигурации

Дополнительные сведения о файлах конфигурации см. в [документе Основные понятия о bridge Самонастраивающийся для центра Интернета вещей](concepts-iot-pnp-bridge.md).

Откройте файл *iot-plug-and-play-bridge\pnpbridge\src\adapters\samples\environmental_sensor\config.js* в текстовом редакторе.

- Добавьте `connection-string` значение, которое является _строкой подключения устройства_ , которое вы записали ранее.
- Добавьте `symmetric_key` значение, которое является значением общего ключа доступа, из _строки подключения устройства_.
- Замените `root_interface_model_id` значение на `dtmi:com:example:PnpBridgeEnvironmentalSensor;1` .

Первый раздел файла *config.jsв* файле теперь выглядит как следующий фрагмент кода:

```json
{
  "$schema": "../../../pnpbridge/src/pnpbridge_config_schema.json",
  "pnp_bridge_connection_parameters": {
    "connection_type" : "connection_string",
    "connection_string" : "HostName=youriothub.azure-devices.net;DeviceId=yourdevice;SharedAccessKey=TTrz8fR7ylHKt7DC/e/e2xocCa5VIcq5x9iQKxKFVa8=",
    "root_interface_model_id": "dtmi:com:example:PnpBridgeEnvironmentalSensor;1",
    "auth_parameters": {
        "auth_type": "symmetric_key",
        "symmetric_key": "TTrz8fR7ylHKt7DC/e/e2xocCa5VIcq5x9iQKxKFVa8="
    },
```

### <a name="run-the-iot-plug-and-play-bridge"></a>Запуск моста самонастраивающийся IoT

Запустите пример датчика среды для центра Интернета вещей самонастраивающийся. Параметр — это путь к `config.json` файлу, измененному в предыдущем разделе.

```cmd
REM Windows
cd iot-plug-and-play-bridge\pnpbridge\cmake\pnpbridge_x86\src\pnpbridge\samples\console
Debug\pnpbridge_bin.exe ..\..\..\..\..\..\src\adapters\samples\environmental_sensor\config.json
```

Мост создает выходные данные, которые выглядят следующим образом:

```output
c:\temp>cd iot-plug-and-play-bridge\pnpbridge\cmake\pnpbridge_x86\src\pnpbridge\samples\console

c:\temp\iot-plug-and-play-bridge\pnpbridge\cmake\pnpbridge_x86\src\pnpbridge\samples\console>Debug\pnpbridge_bin.exe ..\..\..\..\..\..\src\adapters\samples\environmental_sensor\config.json
Info:
 -- Press Ctrl+C to stop PnpBridge

Info: Using configuration from specified file path: ..\..\..\..\..\..\src\adapters\samples\environmental_sensor\config.json
Info: Starting Azure PnpBridge
Info: Pnp Bridge is running as am IoT egde device.
Info: Pnp Bridge creation succeeded.
Info: Connection_type is [connection_string]
Info: Tracing is disabled
Info: WARNING: SharedAccessKey is included in connection string. Ignoring symmetric_key in config file.
Info: IoT Edge Device configuration initialized successfully
Info: Building Pnp Bridge Adapter Manager, Adapters & Components
Info: Adapter with identity environment-sensor-sample-pnp-adapter does not have any associated global parameters. Proceeding with adapter creation.
Info: Pnp Adapter with adapter ID environment-sensor-sample-pnp-adapter has been created.
Info: Pnp Adapter Manager created successfully.
Info: Pnp components created successfully.
Info: Pnp components built in model successfully.
Info: Connected to Azure IoT Hub
Info: Environmental Sensor: Starting Pnp Component
Info: IoTHub client call to _SendReportedState succeeded
Info: Environmental Sensor Adapter:: Sending device information property to IoTHub. propertyName=state, propertyValue=true
Info: Pnp components started successfully.
Info: IoTHub client call to _SendEventAsync succeeded
Info: PnpBridge_PnpBridgeStateTelemetryCallback called, result=0, telemetry=PnpBridge configuration complete
Info: Processing property update for the device or module twin
Info: Environmental Sensor Adapter:: Successfully delivered telemetry message for <environmentalSensor>
```

Используйте следующие команды для запуска моста в Linux:

```bash
cd iot-plug-and-play-bridge/pnpbridge/cmake/pnpbridge_x86/src/pnpbridge/samples/console
./pnpbridge_bin ../../../../../../src/adapters/samples/environmental_sensor/config.json
```

## <a name="download-the-model-files"></a>Скачивание файлов модели

Azure IoT Explorer можно использовать позже для просмотра устройства при подключении к центру Интернета вещей. В обозревателе Azure IoT требуется локальная копия файла модели, соответствующая **идентификатору модели** , который отправляет устройство. Файл модели позволяет обозревателю Интернета вещей отображать данные телеметрии, свойства и команды, реализуемые устройством.

Чтобы скачать модели для Azure IoT Explorer, сделайте следующее:

1. Создайте на локальном компьютере папку *models*.
1. Сохраните [EnvironmentalSensor.js](https://raw.githubusercontent.com/Azure/iot-plug-and-play-bridge/master/pnpbridge/docs/schemas/EnvironmentalSensor.json) в папке *Models* , созданной на предыдущем шаге.
1. Если открыть этот файл модели в текстовом редакторе, то можно увидеть, что модель определяет компонент с помощью в `dtmi:com:example:PnpBridgeEnvironmentalSensor;1` качестве идентификатора. Это тот же идентификатор модели, который использовался в *config.js* файле.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Проверка кода с помощью обозревателя Интернета вещей Azure

После запуска моста используйте средство Azure IoT Explorer, чтобы проверить его работоспособность. Вы видите данные телеметрии, свойства и команды, определенные в `dtmi:com:example:PnpBridgeEnvironmentalSensor;1` модели.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как подключить устройство самонастраивающийся IoT к центру Интернета вещей. Дополнительные сведения о создании решения, взаимодействующего с устройствами IoT Plug and Play, см. в следующей статье:

* [Что такое мост IoT самонастраивающийся](./concepts-iot-pnp-bridge.md)
* [Создание, развертывание и расширение моста самонастраивающийся IoT](howto-build-deploy-extend-pnp-bridge.md)
* [Мост IoT самонастраивающийся на GitHub](https://github.com/Azure/iot-plug-and-play-bridge)
