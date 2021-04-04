---
title: Краткое руководство. Взаимодействие с приложением устройства на языке C# с помощью потоков устройств Центра Интернета вещей Azure
description: В этом кратком руководстве выполняются два примера приложений C#, взаимодействующих через поток устройств, установленный при помощи центра Интернета вещей.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: 3eb65db27e5b96f4b12973154bc860a2ab3df020
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98624614"
---
# <a name="quickstart-communicate-to-a-device-application-in-c-via-iot-hub-device-streams-preview"></a>Краткое руководство. Обмен данными с приложением устройства с помощью C# и потоков устройств Центра Интернета вещей (предварительная версия)

[!INCLUDE [iot-hub-quickstarts-3-selector](../../includes/iot-hub-quickstarts-3-selector.md)]

Центр Интернета вещей Azure поддерживает потоки устройств, которые сейчас доступны в режиме [предварительной версии](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[Потоки устройств Центра Интернета вещей](./iot-hub-device-streams-overview.md) позволяют службам и приложениям устройств безопасным и подходящим методом обмениваться данными с брандмауэром. Это краткое руководство включает в себя две программы C#, использующие преимущества потоков устройств для обмена данными (вывод на экран).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* Предварительная версия потоков устройств сейчас поддерживается только в центрах Интернета вещей, созданных в следующих регионах:
  * Центральная часть США
  * Центральная часть США (EUAP)
  * Северная Европа
  * Юго-Восточная Азия

* Два примера приложений, запускаемые в рамках этого краткого руководства, написаны на языке C#. На компьютере, на котором ведется разработка, необходимо установить пакет SDK для .NET Core версии 2.1.0 или более поздней.

    Скачайте [пакет SDK для .NET Core с поддержкой нескольких платформ](https://www.microsoft.com/net/download/all).

    Проверьте текущую версию C# на компьютере, на котором ведется разработка, используя следующую команду:

    ```
    dotnet --version
    ```

* [Скачайте примеры Azure IoT на C#](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip) и извлеките ZIP-архив. Он понадобится на стороне устройства и на стороне службы.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Регистрация устройства

Устройство должно быть зарегистрировано в Центре Интернета вещей, прежде чем оно сможет подключиться. При изучении этого раздела для регистрации имитированного устройства используется Azure Cloud Shell.

1. Чтобы создать удостоверение устройства, выполните приведенные ниже команды в Cloud Shell.

   > [!NOTE]
   > * Замените заполнитель *YourIoTHubName* именем созданного центра Интернета вещей.
   > * Для имени регистрируемого устройства рекомендуется использовать имя *MyDevice*, как показано в примере. Если вы выбрали другое имя для устройства, используйте его при работе с этой статьей и обновите имя устройства в примерах приложений перед их запуском.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

1. Выполните следующую команду в Azure Cloud Shell, чтобы получить *строку подключения* зарегистрированного устройства.

   > [!NOTE]
   > Замените заполнитель *YourIoTHubName* именем созданного центра Интернета вещей.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name {YourIoTHubName} --device-id MyDevice --output table
    ```

    Запишите возвращенную строку подключения к устройству для последующего использования в этом кратком руководстве. Это должно выглядеть следующим образом:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyDevice;SharedAccessKey={YourSharedAccessKey}`

3. Понадобится также *строка подключения к службе* из Центра Интернета вещей, чтобы включить приложение на стороне службы для подключения к Центру Интернета вещей и установить потоки устройств. Следующая команда получает это значение для Центра Интернета вещей:

   > [!NOTE]
   > Замените заполнитель *YourIoTHubName* именем созданного центра Интернета вещей.

    ```azurecli-interactive
    az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
    ```

    Запишите возвращенную строку подключения к службе для последующего использования в этом кратком руководстве. Это должно выглядеть следующим образом:

   `"HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"`

## <a name="communicate-between-the-device-and-the-service-via-device-streams"></a>Обмен данными между устройством и службой через потоки устройств

В рамках этого раздела вы запустите приложения на стороне устройства и на стороне службы, а также настроите обмен данными между двумя этими приложениями.

### <a name="run-the-service-side-application"></a>Запуск приложения на стороне службы

В окне терминала на локальном компьютере перейдите к каталогу `iot-hub/Quickstarts/device-streams-echo/service` в распакованной папке проекта. Держите следующие сведения под рукой:

| Имя параметра | Значение параметра |
|----------------|-----------------|
| `ServiceConnectionString` | Строка подключения к службе центра Интернета вещей. |
| `MyDevice` | Идентификатор устройства, созданного ранее. |

Выполните компиляцию и запуск кода с помощью следующих команд:

```
cd ./iot-hub/Quickstarts/device-streams-echo/service/

# Build the application
dotnet build

# Run the application
# In Linux or macOS
dotnet run "{ServiceConnectionString}" "MyDevice"

# In Windows
dotnet run {ServiceConnectionString} MyDevice
```
Приложение будет ожидать, пока приложение устройства станет доступным.

> [!NOTE]
> Происходит превышение времени ожидания, если приложения на стороне устройства не отвечает вовремя.

### <a name="run-the-device-side-application"></a>Запуск приложения на стороне устройства

В другом окне терминала на локальном компьютере перейдите к каталогу `iot-hub/Quickstarts/device-streams-echo/device` в распакованной папке проекта. Держите следующие сведения под рукой:

| Имя параметра | Значение параметра |
|----------------|-----------------|
| `DeviceConnectionString` | Строка подключения к устройству Центра Интернета вещей. |

Выполните компиляцию и запуск кода с помощью следующих команд:

```
cd ./iot-hub/Quickstarts/device-streams-echo/device/

# Build the application
dotnet build

# Run the application
# In Linux or macOS
dotnet run "{DeviceConnectionString}"

# In Windows
dotnet run {DeviceConnectionString}
```

В конце последнего шага приложение на стороне службы инициирует поток на ваше устройство. После установления потока приложение отправляет строковый буфер в службу через поток. В этом примере приложение на стороне службы просто возвращает те же данные устройству, демонстрируя успешную двустороннюю связь между двумя приложениями.

Выходные данные консоли на стороне устройства:

![Выходные данные консоли на стороне устройства](./media/quickstart-device-streams-echo-csharp/device-console-output.png)

Выходные данные консоли на стороне службы.

![Выходные данные консоли на стороне службы](./media/quickstart-device-streams-echo-csharp/service-console-output.png)

Трафик, передаваемый по потоку, туннелируется через центр Интернета вещей, а не напрямую. См. подробнее о [преимуществах потоков устройств](./iot-hub-device-streams-overview.md#benefits).

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-hub-quickstarts-clean-up-resources-device-streams](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы настроили центр Интернета вещей, зарегистрировали устройство, установили поток устройств между приложением C# на стороне устройства и службы и использовали поток для обмена данными между приложениями.

Дополнительные сведения о потоках устройств см. в следующей статье:

> [!div class="nextstepaction"]
> [IoT Hub Device Streams (preview)](./iot-hub-device-streams-overview.md) (Потоки устройств (предварительная версия))
