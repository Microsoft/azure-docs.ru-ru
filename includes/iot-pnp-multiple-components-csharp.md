---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: cd87978f9ec34e103ede869360858c5633a8c6ec
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104611996"
---
В этом руководстве показано, как создать пример приложения устройства IoT Plug and Play с компонентами, подключить его к центру Интернета вещей и с помощью обозревателя Интернета вещей Azure просмотреть сведения, отправляемые в центр. Пример приложения написан на языке C# и включен в пакет SDK для устройств центра Интернета вещей Azure для C#. Разработчик решения может использовать обозреватель Интернета вещей Azure, чтобы ознакомиться с возможностями устройства IoT Plug and Play, не просматривая код устройства.

Изучив это руководство, вы:

> [!div class="checklist"]
> * Скачайте пример кода.
> * Выполните сборку примера кода.
> * Запустите пример приложения устройства и проверьте, подключается ли оно к вашему центру Интернета вещей.
> * Просмотрите исходный код.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Для выполнения инструкций из этого учебника в ОС Windows установите в локальной среде Windows такое программное обеспечение:

* [Visual Studio (Community, Professional или Enterprise)](https://visualstudio.microsoft.com/downloads/).
* [Git](https://git-scm.com/download/).

### <a name="clone-the-sdk-repository-with-the-sample-code"></a>Клонирование репозитория пакета SDK с помощью примера кода

Если вы ознакомились с [Кратким руководством по подключению примера приложения устройства IoT Plug and Play в Windows к Центру Интернета вещей (C#)](../articles/iot-pnp/quickstart-connect-device.md), значит вы уже клонировали репозиторий.

Клонируйте примеры из репозитория GitHub с примерами для Интернета вещей Azure на C#. Откройте командную строку в выбранной папке. С помощью следующей команды клонируйте репозиторий примеров [Интернета вещей Microsoft Azure для .NET](https://github.com/Azure-Samples/azure-iot-samples-csharp) с сайта GitHub.

```cmd
git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git
```

## <a name="run-the-sample-device"></a>Запуск примера устройства

С помощью этого краткого руководства вы сможете использовать пример устройства котроллера температуры, написанный на C#, как устройство IoT Plug and Play. Чтобы запустить пример устройства, сделайте следующее:

1. Откройте файл проекта *azure-iot-samples-csharp\iot-hub\Samples\device\PnpDeviceSamples\TemperatureController\TemperatureController.csproj* в Visual Studio 2019.

1. В Visual Studio последовательно выберите **Проект > TemperatureController Properties (Свойства контроллера температуры) > Отладка**. Затем добавьте в проект следующие переменные среды:

    | Имя | Значение |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | DPS |
    | IOTHUB_DEVICE_DPS_ENDPOINT | global.azure-devices-provisioning.net |
    | IOTHUB_DEVICE_DPS_ID_SCOPE | Значение, которое записали после завершения [настройки среды](../articles/iot-pnp/set-up-environment.md). |
    | IOTHUB_DEVICE_DPS_DEVICE_ID | my-pnp-device |
    | IOTHUB_DEVICE_DPS_DEVICE_KEY | Значение, которое записали после завершения [настройки среды](../articles/iot-pnp/set-up-environment.md). |


1. Теперь можно создать пример в Visual Studio и запустить его в режиме отладки.

1. Отобразится сообщение о том, что устройство отправило определенные сведения и находится в сети. Эти сообщения означают, что устройство начало отправлять в концентратор данные телеметрии и теперь готово получать команды и обновления свойств. Не закрывайте этот экземпляр Visual Studio, он понадобится вам, чтобы проверить работу примера службы.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Проверка кода с помощью обозревателя Интернета вещей Azure

После запуска примера клиента устройства используйте обозреватель Интернета вещей Azure, чтобы убедиться, что он работает.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Просмотр кода

В этом примере реализуется устройство контроллера температуры IoT Plug and Play. Модель, которую реализует этот пример, использует [несколько компонентов](../articles/iot-pnp/concepts-modeling-guide.md). [Файл модели языка определения Digital Twins (DTDL) для устройства определения температуры](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) определяет телеметрию, свойства и команды, реализуемые устройством.

Код устройства подключается к центру Интернета вещей с помощью стандартного метода `CreateFromConnectionString`. Устройство отправляет идентификатор модели DTDL, которую он реализует в запросе на подключение. Устройство, отправляющее идентификатор модели, — это устройство IoT Plug and Play:

```csharp
private static DeviceClient InitializeDeviceClient(string hostname, IAuthenticationMethod authenticationMethod)
{
    var options = new ClientOptions
    {
        ModelId = ModelId,
    };

    var deviceClient = DeviceClient.Create(hostname, authenticationMethod, TransportType.Mqtt, options);
    deviceClient.SetConnectionStatusChangesHandler((status, reason) =>
    {
        s_logger.LogDebug($"Connection status change registered - status={status}, reason={reason}.");
    });

    return deviceClient;
}
```

Идентификатор модели хранится в коде, как показано в следующем фрагменте кода:

```csharp
private const string ModelId = "dtmi:com:example:TemperatureController;1";
```

После подключения устройства к центру Интернета вещей код начнет регистрировать обработчики команд. Команда `reboot` определяется в компоненте по умолчанию. Команда `getMaxMinReport` указана в каждом из двух компонентов термостата:

```csharp
await _deviceClient.SetMethodHandlerAsync("reboot", HandleRebootCommandAsync, _deviceClient, cancellationToken);
await _deviceClient.SetMethodHandlerAsync("thermostat1*getMaxMinReport", HandleMaxMinReportCommandAsync, Thermostat1, cancellationToken);
await _deviceClient.SetMethodHandlerAsync("thermostat2*getMaxMinReport", HandleMaxMinReportCommandAsync, Thermostat2, cancellationToken);

```

На двух компонентах термостата имеются отдельные обработчики для обновления требуемых свойств:

```csharp
_desiredPropertyUpdateCallbacks.Add(Thermostat1, TargetTemperatureUpdateCallbackAsync);
_desiredPropertyUpdateCallbacks.Add(Thermostat2, TargetTemperatureUpdateCallbackAsync);

```

Пример кода отправляет данные телеметрии из каждого компонента термостата:

```csharp
await SendTemperatureAsync(Thermostat1, cancellationToken);
await SendTemperatureAsync(Thermostat2, cancellationToken);
```

Метод `SendTemperatureTelemetryAsync` использует класс `PnpHhelper` для создания сообщений для каждого компонента:

```csharp
using Message msg = PnpHelper.CreateIothubMessageUtf8(telemetryName, JsonConvert.SerializeObject(currentTemperature), componentName);
```

Класс `PnpHelper` содержит другие примеры методов, которые можно использовать с многокомпонентной моделью.

Используйте средство обозревателя Интернета вещей Azure, чтобы просматривать данные телеметрии и свойства двух компонентов термостата:

:::image type="content" source="media/iot-pnp-multiple-components-csharp/multiple-component.png" alt-text="Многокомпонентное устройство в обозревателе Интернета вещей Azure":::

Кроме того, средство обозревателя Интернета вещей Azure можно использовать для вызова команд в одном из двух компонентов термостата или в компоненте по умолчанию.
