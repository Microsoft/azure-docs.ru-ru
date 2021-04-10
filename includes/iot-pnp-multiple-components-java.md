---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 3bf5ac4e01bca3bfc3cc8720a068bc53830b4747
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104611964"
---
В этом учебнике показано, как создать многокомпонентный пример приложения устройства IoT Plug and Play, подключить его к Центру Интернета вещей и с помощью обозревателя IoT Azure просмотреть телеметрию, которую он отправляет. Пример приложения написан на языке Java и включен в пакет SDK для устройств центра Интернета вещей Azure для Java. Разработчик решения может использовать Azure CLI, чтобы ознакомиться с возможностями устройства IoT Plug and Play, не просматривая код устройства.

В этом руководстве показано, как создать пример приложения устройства IoT Plug and Play с компонентами, подключить его к центру Интернета вещей и с помощью обозревателя Интернета вещей Azure просмотреть сведения, отправляемые в центр. Пример приложения написан на языке Java и включен в пакет SDK для устройств центра Интернета вещей Azure для Java. Разработчик решения может использовать обозреватель Интернета вещей Azure, чтобы ознакомиться с возможностями устройства IoT Plug and Play, не просматривая код устройства.

Изучив это руководство, вы:

> [!div class="checklist"]
> * Скачайте пример кода.
> * Выполните сборку примера кода.
> * Запустите пример приложения устройства и проверьте, подключается ли оно к вашему центру Интернета вещей.
> * Просмотрите исходный код.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Для выполнения инструкций из этого учебника в ОС Windows установите в локальной среде Windows такое программное обеспечение:

* Пакет SDK для Java SE 8. В статье [Долгосрочная поддержка Java для Azure и Azure Stack](/java/azure/jdk/) выберите пункт **Java 8** в разделе **Долгосрочная поддержка**.
* [Apache Maven 3](https://maven.apache.org/download.cgi).

## <a name="download-the-code"></a>Загрузка кода

Если вы ознакомились с [Кратким руководством по подключению примера приложения устройства IoT Plug and Play в Windows к Центру Интернета вещей (Java)](../articles/iot-pnp/quickstart-connect-device.md), значит вы уже клонировали репозиторий.

Откройте командную строку в выбранном каталоге. Выполните следующую команду для клонирования репозитория GitHub [пакетов SDK и библиотек Интернета вещей Azure для Java](https://github.com/Azure/azure-iot-sdk-java) в это расположение:

```cmd
git clone https://github.com/Azure/azure-iot-sdk-java.git
```

Выполнение этой операции может занять несколько минут.

## <a name="build-the-code"></a>Сборка кода

В Windows перейдите к корневой папке клонированного репозитория пакета SDK для Java. Выполните следующую команду, чтобы создать зависимости:

```cmd/sh
mvn install -T 2C -DskipTests
```

## <a name="run-the-device-sample"></a>Запуск примера устройства

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Чтобы запустить пример приложения, перейдите в папку *\device\iot-device-samples\pnp-device-sample\temperature-controller-device-sample* и выполните следующую команду:

```cmd/sh
mvn exec:java -Dexec.mainClass="samples.com.microsoft.azure.sdk.iot.device.TemperatureController"
```

Теперь устройство готово к получению команд и обновлений свойств и начало отправлять данные телеметрии в концентратор. Продолжите работу примера, после того как выполните следующие действия.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Проверка кода с помощью обозревателя Интернета вещей Azure

После запуска примера клиента устройства используйте обозреватель Интернета вещей Azure, чтобы убедиться, что он работает.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Просмотр кода

В этом примере реализуется устройство контроллера температуры IoT Plug and Play. Модель, которую реализует этот пример, использует [несколько компонентов](../articles/iot-pnp/concepts-modeling-guide.md). [Файл модели языка определения Digital Twins (DTDL) для устройства определения температуры](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) определяет телеметрию, свойства и команды, реализуемые устройством.

Код устройства использует стандартный класс `DeviceClient` для подключения к центру Интернета вещей. Устройство отправляет идентификатор модели DTDL, которую он реализует в запросе на подключение. Устройство, отправляющее идентификатор модели, — это устройство IoT Plug and Play:

```java
private static void initializeDeviceClient() throws URISyntaxException, IOException {
    ClientOptions options = new ClientOptions();
    options.setModelId(MODEL_ID);
    deviceClient = new DeviceClient(deviceConnectionString, protocol, options);

    deviceClient.registerConnectionStatusChangeCallback((status, statusChangeReason, throwable, callbackContext) -> {
        log.debug("Connection status change registered: status={}, reason={}", status, statusChangeReason);

        if (throwable != null) {
            log.debug("The connection status change was caused by the following Throwable: {}", throwable.getMessage());
            throwable.printStackTrace();
        }
    }, deviceClient);

    deviceClient.open();
}
```

Идентификатор модели хранится в коде, как показано в следующем фрагменте кода:

```java
private static final String MODEL_ID = "dtmi:com:example:Thermostat;1";
```

После подключения устройства к центру Интернета вещей код начнет регистрировать обработчики команд.

```java
deviceClient.subscribeToDeviceMethod(new MethodCallback(), null, new MethodIotHubEventCallback(), null);
```

На двух компонентах термостата имеются отдельные обработчики для обновления требуемых свойств:

```java
deviceClient.startDeviceTwin(new TwinIotHubEventCallback(), null, new GenericPropertyUpdateCallback(), null);
Map<Property, Pair<TwinPropertyCallBack, Object>> desiredPropertyUpdateCallback = Stream.of(
        new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
                new Property(THERMOSTAT_1, null),
                new Pair<>(new TargetTemperatureUpdateCallback(), THERMOSTAT_1)),
        new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
                new Property(THERMOSTAT_2, null),
                new Pair<>(new TargetTemperatureUpdateCallback(), THERMOSTAT_2))
).collect(Collectors.toMap(AbstractMap.SimpleEntry::getKey, AbstractMap.SimpleEntry::getValue));

deviceClient.subscribeToTwinDesiredProperties(desiredPropertyUpdateCallback);
```

Пример кода отправляет данные телеметрии из каждого компонента термостата:

```java
sendTemperatureReading(THERMOSTAT_1);
sendTemperatureReading(THERMOSTAT_2);
```

Метод `sendTemperatureReading` использует класс `PnpHhelper` для создания сообщений для каждого компонента:

```java
Message message = PnpHelper.createIotHubMessageUtf8(telemetryName, currentTemperature, componentName);
```

Класс `PnpHelper` содержит другие примеры методов, которые можно использовать с многокомпонентной моделью.

Используйте средство обозревателя Интернета вещей Azure, чтобы просматривать данные телеметрии и свойства двух компонентов термостата:

:::image type="content" source="media/iot-pnp-multiple-components-java/multiple-component.png" alt-text="Многокомпонентное устройство в обозревателе Интернета вещей Azure":::

Кроме того, средство обозревателя Интернета вещей Azure можно использовать для вызова команд в одном из двух компонентов термостата или в компоненте по умолчанию.
