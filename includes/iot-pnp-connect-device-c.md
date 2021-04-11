---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: f626d6058374d52b5a1fd07bfdb229cfb715ca45
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104612534"
---
В этом кратком руководстве показано, как создать пример приложения устройства IoT Plug and Play, подключить его к Центру Интернета вещей и с помощью обозревателя Интернета вещей Azure просмотреть данные телеметрии, которые он отправляет. Пример приложения написан на языке C и включен в пакет SDK для устройств Azure IoT для C. Разработчик решения может использовать обозреватель Интернета вещей Azure, чтобы ознакомиться с возможностями устройства IoT Plug and Play, не просматривая код устройства.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Операции, описанные в этом кратком руководстве, можно выполнить в Linux или Windows. Команды оболочки в этом кратком руководстве соответствуют требованиям соглашения Linux для разделителей пути "`/`". Если вы используете Windows, не забудьте изменить эти разделители на "`\`".

Необходимые условия зависят от операционной системы:

### <a name="linux"></a>Linux

В этом кратком руководстве предполагается, что вы используете Ubuntu Linux. Действия, описанные в этом кратком руководстве, были протестированы в Ubuntu 18.04.

Для выполнения инструкций, указанных в этом кратком руководстве, в ОС Linux необходимо установить указанное ниже программное обеспечение в локальной среде.

Установите **GCC**, **Git**, **cmake** и все необходимые зависимости с помощью команды `apt-get`:

```sh
sudo apt-get update
sudo apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
```

Убедитесь, что версия `cmake` выше **2.8.12**, а версия **GCC** выше **4.4.7**.

```sh
cmake --version
gcc --version
```

### <a name="windows"></a>Windows

Для работы с этим кратким руководством в Windows необходимо установить следующее программное обеспечение в локальной среде:

* [Visual Studio (Community, Professional или Enterprise)](https://visualstudio.microsoft.com/downloads/). При [установке](/cpp/build/vscpp-step-0-installation?preserve-view=true&view=vs-2019) Visual Studio обязательно добавьте рабочую нагрузку **Разработка классических приложений на C++** .
* [Git](https://git-scm.com/download/).
* [CMake](https://cmake.org/download/).

## <a name="download-the-code"></a>Загрузка кода

С помощью этого краткого руководства вы подготовите среду разработки, которую можно использовать для клонирования и сборки пакета SDK для устройств Центра Интернета вещей Azure для C.

Откройте командную строку в выбранном каталоге. Выполните следующую команду для клонирования репозитория GitHub [пакетов SDK и библиотек Интернета вещей Azure для С](https://github.com/Azure/azure-iot-sdk-c) в это расположение:

```cmd\bash
git clone https://github.com/Azure/azure-iot-sdk-c.git
cd azure-iot-sdk-c
git submodule update --init
```

Выполнение этой операции может занять несколько минут.

## <a name="build-the-code"></a>Сборка кода

Используйте пакет SDK для устройств, чтобы выполнить сборку прилагаемого примера кода:

1. Создайте подкаталог _cmake_ в корневой папке пакета SDK для устройства и перейдите к этой папке:

    ```cmd\bash
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

1. Выполните следующие команды, чтобы создать пакет SDK и примеры:

    ```cmd\bash
    cmake -Duse_prov_client=ON -Dhsm_type_symm_key=ON -Drun_e2e_tests=OFF ..
    cmake --build .
    ```

> [!TIP]
> В Windows можно открыть решение, созданное командой `cmake` в Visual Studio 2019. Откройте файл проекта *azure_iot_sdks.sln* в каталоге _cmake_ и задайте проект **pnp_simple_thermostat** в качестве запускаемого проекта в решении. Теперь можно создать пример в Visual Studio и запустить его в режиме отладки.

## <a name="run-the-device-sample"></a>Запуск примера устройства

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Дополнительные сведения о примере конфигурации см. в [образце файла сведений](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/samples/pnp/readme.md).

Запустите пример приложения в пакете SDK для имитации устройства IoT Plug and Play, отправляющего данные телеметрии в ваш центр Интернета вещей, выполнив указанные ниже действия.

Из папки _cmake_ перейдите к папке, содержащей исполняемый файл, и запустите его:

```bash
# Bash
cd iothub_client/samples/pnp/pnp_simple_thermostat/
./pnp_simple_thermostat
```

```cmd
REM Windows
cd iothub_client\samples\pnp\pnp_simple_thermostat\Debug
.\pnp_simple_thermostat.exe
```

> [!TIP]
> Для трассировки выполнения кода в Visual Studio в Windows добавьте точку останова в функцию `main` в файле _pnp_simple_thermostat.c_.

Теперь устройство готово к получению команд и обновлений свойств и начало отправлять данные телеметрии в концентратор. Продолжите работу примера, после того как выполните следующие действия.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Проверка кода с помощью обозревателя Интернета вещей Azure

После запуска примера клиента устройства используйте обозреватель Интернета вещей Azure, чтобы убедиться, что он работает.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Просмотр кода

В этом примере используется простой терморегулятор IoT Plug and Play. Модель, которую реализует этот пример, не использует [компоненты](../articles/iot-pnp/concepts-modeling-guide.md) IoT Plug and Play. [Файл модели DTDL для терморегулятора](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) определяет данные телеметрии, свойства и команды, выполняемые устройством.

Код устройства использует стандартную функцию для подключения к центру Интернета вещей:

```c
deviceHandle = IoTHubDeviceClient_CreateFromConnectionString(connectionString, MQTT_Protocol)
```

Устройство отправляет идентификатор модели DTDL, которую он реализует в запросе на подключение. Устройство, отправляющее идентификатор модели, — это устройство IoT Plug and Play:

```c
static const char g_ModelId[] = "dtmi:com:example:Thermostat;1";

...

IoTHubDeviceClient_SetOption(deviceHandle, OPTION_MODEL_ID, modelId)
```

Код, который обновляет свойства, обрабатывает команды и отправляет данные телеметрии, идентичен коду устройства, которое не использует соглашения IoT Plug and Play.

Код использует библиотеку Parson для анализа объектов JSON в полезных данных, отправленных из центра Интернета вещей:

```c
// JSON parser
#include "parson.h"
```
