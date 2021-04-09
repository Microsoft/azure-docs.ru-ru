---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 8d3f35a733a0f78fabc33df857d911ba3cd222f5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102244660"
---
IoT Plug and Play упрощает работу с Интернетом вещей, позволяя взаимодействовать с возможностями устройства без знаний базовой реализации устройства. В этом кратком руководстве показано, как подключиться к устройству IoT Plug and Play, подключенному к решению, а также управлять им с помощью Java.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Для работы с этим кратким руководством в Windows необходимо установить следующее программное обеспечение в локальной среде:

* Пакет SDK для Java SE 8. В статье [Долгосрочная поддержка Java для Azure и Azure Stack](/java/azure/jdk/) выберите пункт **Java 8** в разделе **Долгосрочная поддержка**.
* [Apache Maven 3](https://maven.apache.org/download.cgi).

### <a name="clone-the-sdk-repository-with-the-sample-code"></a>Клонирование репозитория пакета SDK с помощью примера кода

Если вы ознакомились с [Кратким руководством по подключению примера приложения устройства IoT Plug and Play в Windows к Центру Интернета вещей (Java)](../articles/iot-pnp/quickstart-connect-device.md), значит вы уже клонировали репозиторий.

Откройте командную строку в выбранном каталоге. Выполните следующую команду, чтобы клонировать репозиторий GitHub [пакетов SDK для Интернета вещей Microsoft Azure для Java](https://github.com/Azure/azure-iot-sdk-java) в это расположение:

```cmd
git clone https://github.com/Azure/azure-iot-sdk-java.git
```

## <a name="build-and-run-the-sample-device"></a>Сборка и запуск примера устройства

[!INCLUDE [iot-pnp-environment](iot-pnp-environment.md)]

Дополнительные сведения о примере конфигурации см. в [образце файла сведений](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-samples/readme.md).

С помощью этого краткого руководства вы сможете использовать пример термостата, написанный на Java, как устройство IoT Plug and Play. Чтобы запустить пример устройства, сделайте следующее:

1. Откройте окно терминала и перейдите в локальную папку, содержащую пакет SDK Microsoft Azure IoT для Java, клонированный из репозитория GitHub.

1. Это окно терминала используется в качестве терминала **устройства**. Перейдите в корневую папку клонированного репозитория. Установите все зависимости, выполнив следующую команду:

1. Запустите сборку примера приложения устройства с помощью следующей команды.

    ```cmd
    mvn install -T 2C -DskipTests
    ```

1. Чтобы запустить пример приложения устройства, перейдите в папку *device\iot-device-samples\pnp-device-sample\thermostat-device-sample* и выполните следующую команду:

    ```cmd
    mvn exec:java -Dexec.mainClass="samples.com.microsoft.azure.sdk.iot.device.Thermostat"
    ```

Теперь устройство готово к получению команд и обновлений свойств и начало отправлять данные телеметрии в концентратор. Не отключайте пример устройства, после того как выполните следующие действия.

## <a name="run-the-sample-solution"></a>Запуск примера решения

Во время прохождения статьи [Настройка среды для кратких руководств и учебников IoT Plug and Play](../articles/iot-pnp/set-up-environment.md) вы создали две переменные среды, чтобы настроить пример для подключения к центру Интернета вещей и устройству:

* **IOTHUB_CONNECTION_STRING**: строка подключения центра Интернета вещей, которую вы записали ранее.
* **IOTHUB_DEVICE_ID**: `"my-pnp-device"`.

С помощью этого краткого руководства вы примените пример решения Интернета вещей на языке Java для взаимодействия с настроенным примером устройства.

> [!NOTE]
> В этом образце используется пространство имен **com.microsoft.azure.sdk.iot.service** из **клиента службы Центра Интернета вещей**. Дополнительные сведения об интерфейсах API, включая API цифровых двойников, см. в [руководстве для разработчиков служб](../articles/iot-pnp/concepts-developer-guide-service.md).

1. Откройте другое окно терминала, которое будет терминалом **службы**.

1. В клонированном репозитории пакета SDK для Java перейдите в папку *service\iot-service-samples\pnp-service-sample\thermostat-service-sample*.

1. Чтобы запустить пример приложения службы, выполните следующую команду:

    ```cmd
    mvm exec:java -Dexec.mainClass="samples.com.microsoft.azure.sdk.iot.service.Thermostat"
    ```

### <a name="get-device-twin"></a>Получение двойника устройства

В следующем фрагменте кода показано, как получить двойник устройства в службе:

```java
 // Get the twin and retrieve model Id set by Device client.
DeviceTwinDevice twin = new DeviceTwinDevice(deviceId);
twinClient.getTwin(twin);
System.out.println("Model Id of this Twin is: " + twin.getModelId());
```

### <a name="update-a-device-twin"></a>Обновление двойника устройства

В следующем фрагменте кода показано, как использовать *исправление* для обновления свойств с помощью двойника устройства:

```java
String propertyName = "targetTemperature";
double propertyValue = 60.2;
twin.setDesiredProperties(Collections.singleton(new Pair(propertyName, propertyValue)));
twinClient.updateTwin(twin);
```

В выходных данных устройства показано, как устройство отвечает на это обновление свойства.

### <a name="invoke-a-command"></a>Вызов команды

В следующем фрагменте кода показано, как вызвать команду на устройстве:

```java
// The method to invoke for a device without components should be "methodName" as defined in the DTDL.
String methodToInvoke = "getMaxMinReport";
System.out.println("Invoking method: " + methodToInvoke);

Long responseTimeout = TimeUnit.SECONDS.toSeconds(200);
Long connectTimeout = TimeUnit.SECONDS.toSeconds(5);

// Invoke the command.
String commandInput = ZonedDateTime.now(ZoneOffset.UTC).minusMinutes(5).format(DateTimeFormatter.ISO_DATE_TIME);
MethodResult result = methodClient.invoke(deviceId, methodToInvoke, responseTimeout, connectTimeout, commandInput);
if(result == null)
{
    throw new IOException("Method result is null");
}

System.out.println("Method result status is: " + result.getStatus());
```

В выходных данных устройства показано, как устройство отвечает на эту команду.
