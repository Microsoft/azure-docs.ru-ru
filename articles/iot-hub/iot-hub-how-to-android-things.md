---
title: Разработка на платформе Android Things с помощью пакетов SDK для Центра Интернета вещей Azure | Документация Майкрософт
description: Руководство для разработчиков. Дополнительные сведения о разработке на платформе Android Things с помощью пакетов SDK для Центра Интернета вещей Azure.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/30/2019
ms.author: robinsh
ms.openlocfilehash: 6008ca1549629bcbb582f38de2ab1b453cd73a99
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94843763"
---
# <a name="develop-for-android-things-platform-using-azure-iot-sdks"></a>Разработка на платформе Android Things с помощью пакетов SDK для Интернета вещей Azure

[Пакеты SDK для Центра Интернета вещей Azure](./iot-hub-devguide-sdks.md) обеспечивают поддержку первого уровня для популярных платформ (Windows, Linux, OSX, MBED) и мобильных платформ (Android и iOS).  Как часть наших обязательств по расширению выбора и гибкости развертываний в Центре Интернета вещей пакет SDK для Java также поддерживает платформу [Android Things](https://developer.android.com/things/).  Разработчики могут пользоваться преимуществами операционной системы Android Things на стороне устройства, используя [Центр Интернета вещей Azure](about-iot-hub.md) в качестве центрального концентратора сообщений, который масштабируется до миллионов одновременно подключенных устройств.

В этом руководстве описано, как создать приложение на стороне устройства на платформе Android Things при помощи пакета SDK Java для Центра Интернета вещей Azure.

## <a name="prerequisites"></a>Предварительные условия

* Оборудование, поддерживаемое платформой Android Things, под управлением операционной системы Android Things.  Чтобы выполнить инструкции по установке операционной системы Android Things, см. документацию по Android Things [здесь](https://developer.android.com/things/get-started/kits#flash-at).  Убедитесь, что устройство под управлением Android Things подключено к Интернету и подключены все необходимые дополнительные устройства, например, клавиатура, дисплей и мышь.  В этом руководстве используется Raspberry Pi 3.

* Последняя версия [Android Studio](https://developer.android.com/studio/).

* Последняя версия [Git](https://git-scm.com/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Регистрация устройства

Устройство должно быть зарегистрировано в Центре Интернета вещей, прежде чем оно сможет подключиться. В этом кратком руководстве для регистрации имитируемого устройства используется Azure Cloud Shell.

1. Выполните приведенные ниже команды в Azure Cloud Shell, чтобы добавить расширение CLI Центра Интернета вещей и создать удостоверение устройства.

   **YourIoTHubName.** Замените этот заполнитель именем центра Интернета вещей.

   **Мяндроидсингсдевице** : это имя, заданное для зарегистрированного устройства. Используйте MyAndroidThingsDevice, как показано ниже. Если вы выбрали другое имя для устройства, используйте его при работе с этим руководством и обновите имя устройства в примерах приложений перед их запуском.

    ```azurecli-interactive
    az extension add --name azure-iot
    az iot hub device-identity create --hub-name YourIoTHubName --device-id MyAndroidThingsDevice
    ```

2. Выполните следующие команды в Azure Cloud Shell, чтобы получить *строку подключения устройства* для только что зарегистрированного устройства. Замените `YourIoTHubName` под именем, выбранным для центра Интернета вещей.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name YourIoTHubName --device-id MyAndroidThingsDevice --output table
    ```

    Запишите строку подключения устройства, которая выглядит так:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyAndroidThingsDevice;SharedAccessKey={YourSharedAccessKey}`

    Это значение понадобится позже в рамках этого краткого руководства.

## <a name="building-an-android-things-application"></a>Создание приложения Android Things

1. Первым шагом в создании приложения Android Things является подключение к устройствам Android Things. Подключите устройство Android Things к дисплею, а затем настройте подключение к Интернету. Для Android Things предоставляется [документация](https://developer.android.com/things/get-started/kits) по подключению к Wi-Fi. После подключения к Интернету запишите IP-адрес, указанный в разделе Networks (Сети).

2. Подключитесь к устройству Android Things с помощью средства [Android Debug Bridge](https://developer.android.com/studio/command-line/adb) с использованием записанного ранее IP-адреса. Еще раз проверьте подключение, выполнив следующую команду в окне терминала. Устройства должны отображаться как подключенные.

   ```
   adb devices
   ```

3. Скачайте наш пример для Android или Android Things из [репозитория](https://github.com/Azure-Samples/azure-iot-samples-java) или используйте Git.

   ```
   git clone https://github.com/Azure-Samples/azure-iot-samples-java.git
   ```

4. В Android Studio откройте проект Android, находящийся в папке \azure-iot-samples-java\iot-hub\Samples\device\AndroidSample.

5. Откройте файл gradle.properties и замените Device_connection_string строкой подключения устройства, записанной ранее.
 
6. Щелкните Run — Debug (Запуск — отладка) и выберите устройство Android Things, чтобы развернуть на нем этот код.

7. После успешного запуска приложения вы увидите, что оно запущено на вашем устройстве Android Things. Этот пример приложения отправляет случайно сгенерированные показатели температуры.

## <a name="read-the-telemetry-from-your-hub"></a>Чтение данных телеметрии из концентратора

Вы можете просматривать данные с помощью центра Интернета вещей по мере их получения. Расширение CLI для Центра Интернета вещей позволяет подключить конечную точку сервера **События** к Центру Интернета вещей. Расширение получает сообщения с устройства в облако, отправленные с имитированного устройства. Внутреннее приложение Центра Интернета вещей обычно запускается в облаке, чтобы получать сообщения с устройства в облако и обрабатывать их.

Выполните следующие команды в Azure Cloud Shell, заменив `YourIoTHubName` на имя вашего центра Интернета вещей:

```azurecli-interactive
az iot hub monitor-events --device-id MyAndroidThingsDevice --hub-name YourIoTHubName
```

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о том, как управлять подключениями и надежным обменом сообщениями с помощью пакетов SDK центра Интернета вещей, см. в [этой статье](iot-hub-reliability-features-in-sdks.md).
* Дополнительные сведения см. в статье [Разработка для мобильных устройств с помощью пакетов SDK для Центра Интернета вещей Azure](iot-hub-how-to-develop-for-mobile-devices.md).
* [Поддержка платформ пакетов SDK для устройств Azure IoT](iot-hub-device-sdk-platform-support.md)