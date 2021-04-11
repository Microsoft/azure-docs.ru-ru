---
title: Учебник. Использование MQTT для создания клиента устройства Azure IoT Plug and Play | Документация Майкрософт
description: Учебник. Использование протокола MQTT напрямую для создания клиента устройства IoT Plug and Play без применения пакетов SDK для устройств Azure IoT.
author: ericmitt
ms.author: ericmitt
ms.date: 05/13/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 64ff4615c2b41e341352dce5143d48ec8e6e802a
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106066778"
---
# <a name="tutorial---use-mqtt-to-develop-an-iot-plug-and-play-device-client"></a>Учебник. Использование MQTT для разработки клиента устройства IoT Plug and Play

При возможности в процессе создания клиентов устройств IoT Plug and Play вы должны использовать один из пакетов SDK для устройств Azure IoT. Но в некоторых сценариях, например при ограниченном объеме памяти на устройстве, возможно, потребуется использовать библиотеку MQTT для обмена данными с центром Интернета вещей.

В примере в этом руководстве используется библиотека MQTT [Eclipse Mosquitto](http://mosquitto.org/) и Visual Studio. В инструкциях этого руководства предполагается, что на компьютере разработки вы используете Windows.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Скачайте и выполните сборку библиотеки Eclipse Mosquitto.
> * Измените пример кода MQTT на основе C, чтобы включить на устройстве поддержку IoT Plug and Play.
> * Найдите разделы MQTT, используемые устройством IoT Plug and Play.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](../../includes/iot-pnp-prerequisites.md)]

Для выполнения инструкций из этого учебника в ОС Windows установите в локальной среде Windows такое программное обеспечение:

* [Visual Studio (Community, Professional или Enterprise).](https://visualstudio.microsoft.com/downloads/) При [установке](/cpp/build/vscpp-step-0-installation?preserve-view=true&view=vs-2019) Visual Studio обязательно добавьте рабочую нагрузку **Разработка классических приложений на C++** .
* [Git](https://git-scm.com/download/);
* [CMake](https://cmake.org/download/).

Используйте средство *обозревателя Интернета вещей Azure* для добавления нового устройства к Центру Интернета вещей. После завершения работы со статьей [Настройка среды для кратких руководств и учебников IoT Plug and Play](set-up-environment.md) вы настроите центр Интернета вещей и средство обозревателя Интернета вещей Azure:

1. Запустите **обозреватель Интернета вещей Azure**.
1. На странице **Центры Интернета вещей** выберите **View devices in this hub** (Просмотр устройств в этом центре).
1. На странице **Устройства** выберите **+ Создать**.
1. Создайте устройство с именем *my-mqtt-device*, которое использует автоматически сформированный симметричный ключ.
1. На странице **Device identity** (Удостоверение устройства) разверните **Connection string with SAS token** (Строка подключения с маркером SAS).
1. Выберите **Первичный ключ** для использования в качестве **симметричного ключа**, установите срок действия 60 минут и щелкните **Создать**.
1. Скопируйте созданную **строку подключения маркера SAS**, это значение будет использоваться позже в этом учебнике.

## <a name="clone-sample-repo"></a>Клонирование примера репозитория

Используйте следующую команду, чтобы клонировать пример репозитория в подходящее расположение на локальном компьютере:

```cmd
git clone https://github.com/Azure-Samples/IoTMQTTSample.git
```

## <a name="install-mqtt-library"></a>Установка библиотеки MQTT

С помощью средства `vcpkg` скачайте библиотеку Eclipse Mosquitto и выполните ее сборку.

Используйте следующую команду, чтобы клонировать репозиторий **Vcpkg** в подходящее расположение на локальном компьютере:

```cmd
git clone https://github.com/Microsoft/vcpkg.git
```

Используйте следующие команды, чтобы подготовить среду `vcpkg`. При выполнении `vcpkg integrate install` вам потребуется командная строка с повышенными привилегиями:

```cmd
cd vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg integrate install
```

Используйте следующую команду, чтобы скачать библиотеку Eclipse Mosquitto и выполнить ее сборку:

```cmd
.\vcpkg install mosquitto:x64-windows
```

## <a name="migrate-the-sample-to-iot-plug-and-play"></a>Перенос примера в IoT Plug and Play

### <a name="review-the-non-iot-plug-and-play-sample-code"></a>Проверка примера кода без данных IoT Plug and Play

Внесите в код данные о своем центре Интернета вещей и устройстве, прежде чем выполнять его сборку и запуск.

Чтобы просмотреть пример кода в Visual Studio, откройте файл решения *MQTTWin32.sln* в папке *IoTMQTTSample\src\Windows*.

В **Обозревателе решений** щелкните правой кнопкой мыши проект **TelemetryMQTTWin32** и выберите **Назначить запускаемым проектом**.

В проекте **TelemetryMQTTWin32** откройте исходный файл **MQTT_Mosquitto.cpp**. Внесите в определения данных о подключениях сведения об устройстве, которые вы записали ранее. Замените заполнители строк маркера для:

* Идентификатора `IOTHUBNAME` на имя центра Интернета вещей.
* Идентификатора `DEVICEID` на `my-mqtt-device`.
* Идентификатора `PWD` на правильную часть созданной для устройства строки подключения маркера SAS. Используйте часть строки подключения от `SharedAccessSignature sr=` и до конца.

Убедитесь, что код работает правильно, запустив обозреватель Интернета вещей Azure и начав прослушивание телеметрии.

Запустите приложение (CTRL+F5), и через несколько секунд вы увидите выходные данные, аналогичные следующим:

:::image type="content" source="media/tutorial-use-mqtt/mqtt-sample-output.png" alt-text="Выходные данные примера приложения MQTT":::

В обозревателе Интернета вещей Azure вы увидите, что устройство не является устройством IoT Plug and Play:

:::image type="content" source="media/tutorial-use-mqtt/non-pnp-iot-explorer.png" alt-text="Устройство, не относящееся к IoT Plug and Play, в обозревателе Интернета вещей Azure":::

### <a name="make-the-device-an-iot-plug-and-play-device"></a>Включение IoT Plug and Play для устройства

Устройство IoT Plug and Play должно соответствовать ряду простых соглашений. Если устройство отправляет идентификатор модели при подключении, оно становится устройством IoT Plug and Play.

В этом примере вы добавите идентификатор модели к пакету подключения MQTT. Вы передадите идентификатор модели в качестве параметра querystring в `USERNAME` и измените `api-version` на `2020-09-30`:

```c
// computed Host Username and Topic
//#define USERNAME IOTHUBNAME ".azure-devices.net/" DEVICEID "/?api-version=2018-06-30"
#define USERNAME IOTHUBNAME ".azure-devices.net/" DEVICEID "/?api-version=2020-09-30&model-id=dtmi:com:example:Thermostat;1"
#define PORT 8883
#define HOST IOTHUBNAME //".azure-devices.net"
#define TOPIC "devices/" DEVICEID "/messages/events/"
```

Повторно выполните сборку и запустите пример.

Двойник устройства теперь включает идентификатор модели:

:::image type="content" source="media/tutorial-use-mqtt/model-id-iot-explorer.png" alt-text="Просмотр идентификатора модели в обозревателе Azure IoT":::

Теперь вы можете перейти к компоненту IoT Plug and Play:

:::image type="content" source="media/tutorial-use-mqtt/components-iot-explorer.png" alt-text="Просмотр компонентов в обозревателе Интернета вещей Azure":::

Теперь вы можете изменить код устройства для реализации телеметрии, свойств и команд, определенных в вашей модели. Пример реализации устройства термостата с использованием библиотеки Mosquitto см. на странице [Использование MQTT PnP с Центром Интернета вещей Azure без пакета SDK для Интернета вещей в Windows](https://github.com/Azure-Samples/IoTMQTTSample/tree/master/src/Windows/PnPMQTTWin32) на сайте GitHub.

> [!NOTE]
>Клиент использует файл корневого сертификата `IoTHubRootCA_Baltimore.pem` для проверки удостоверения центра Интернета вещей, к которому он подключается.

### <a name="mqtt-topics"></a>Темы MQTT

Приведенные ниже определения предназначены для тем MQTT, которые устройство использует для отправки данных в центр Интернета вещей. Дополнительные сведения см. в статье [Взаимодействие с Центром Интернета вещей с помощью протокола MQTT](../iot-hub/iot-hub-mqtt-support.md):

* `DEVICE_CAPABILITIES_MESSAGE` определяет тему, которую устройство использует для передачи данных о реализуемых интерфейсах.
* `DEVICETWIN_PATCH_MESSAGE` определяет тему, которую устройство использует для передачи данных об обновлениях свойств в центр Интернета вещей.
* `DEVICE_TELEMETRY_MESSAGE` определяет тему, которую устройство использует для отправки телеметрии в центр Интернета вещей.

Дополнительные сведения об MQTT см. на странице с [примерами MQTT для Интернета вещей Azure](https://github.com/Azure-Samples/IoTMQTTSample/) в репозитории GitHub.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-pnp-clean-resources](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как изменить клиент устройства MQTT для соответствия соглашениям IoT Plug and Play. Дополнительные сведения о поддержке Центра Интернета вещей для протокола MQTT см. в статье:

> [!div class="nextstepaction"]
> [Взаимодействие с Центром Интернета вещей с помощью протокола MQTT](../iot-hub/iot-hub-mqtt-support.md)