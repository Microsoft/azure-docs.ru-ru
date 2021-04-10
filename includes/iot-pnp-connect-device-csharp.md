---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 11/20/2020
ms.openlocfilehash: 30ea75a2df63fa935314fc103fe1e7e092f655b2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104612126"
---
В этом кратком руководстве показано, как создать пример приложения устройства IoT Plug and Play, подключить его к Центру Интернета вещей и с помощью обозревателя Интернета вещей Azure просмотреть данные телеметрии, которые он отправляет. Пример приложения написан на языке C# и включен в набор примеров для Центра Интернета вещей Azure для C#. Разработчик решения может использовать обозреватель Интернета вещей Azure, чтобы ознакомиться с возможностями устройства IoT Plug and Play, не просматривая код устройства.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](iot-pnp-prerequisites.md)]

Для работы с этим кратким руководством в Windows необходимо установить следующее ПО на компьютер разработки:

* [Visual Studio (Community, Professional или Enterprise)](https://visualstudio.microsoft.com/downloads/).
* [Git](https://git-scm.com/download/).

## <a name="download-the-code"></a>Загрузка кода

С помощью этого краткого руководства вы подготовите среду разработки, которую можно использовать для клонирования и создания репозитория примеров Интернета вещей Azure для C#.

Откройте командную строку в выбранной папке. С помощью следующей команды клонируйте репозиторий [примеров Интернета вещей Microsoft Azure для C# (.NET)](https://github.com/Azure-Samples/azure-iot-samples-csharp) с сайта GitHub в указанное расположение:

```cmd
git clone  https://github.com/Azure-Samples/azure-iot-samples-csharp.git
```

## <a name="build-the-code"></a>Сборка кода

Теперь можно создать пример в Visual Studio и запустить его в режиме отладки.

1. Откройте файл проекта *azure-iot-samples-csharp\iot-hub\Samples\device\PnpDeviceSamples\Thermostat\Thermostat.csproj* в Visual Studio 2019.

1. В Visual Studio последовательно выберите **Проект > Thermostat Properties (Свойства термостата) > Отладка**. Затем добавьте в проект следующие переменные среды:

    | Имя | Значение |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | DPS |
    | IOTHUB_DEVICE_DPS_ENDPOINT | global.azure-devices-provisioning.net |
    | IOTHUB_DEVICE_DPS_ID_SCOPE | Значение, которое записали после завершения [настройки среды](../articles/iot-pnp/set-up-environment.md). |
    | IOTHUB_DEVICE_DPS_DEVICE_ID | my-pnp-device |
    | IOTHUB_DEVICE_DPS_DEVICE_KEY | Значение, которое записали после завершения [настройки среды](../articles/iot-pnp/set-up-environment.md). |

Теперь можно создать пример в Visual Studio и запустить его в режиме отладки.

## <a name="run-the-device-sample"></a>Запуск примера устройства

Для трассировки выполнения кода в Visual Studio в Windows добавьте точку останова в функцию `main` в файле program.cs.

Теперь устройство готово к получению команд и обновлений свойств и начало отправлять данные телеметрии в концентратор. Продолжите работу примера, после того как выполните следующие действия.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Проверка кода с помощью обозревателя Интернета вещей Azure

После запуска примера клиента устройства используйте обозреватель Интернета вещей Azure, чтобы убедиться, что он работает.

[!INCLUDE [iot-pnp-iot-explorer.md](iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Просмотр кода

В этом примере используется простой терморегулятор IoT Plug and Play. Модель, которую реализует этот пример, не использует [компоненты](../articles/iot-pnp/concepts-modeling-guide.md) IoT Plug and Play. [Файл модели языка определения Digital Twins (DTDL) для терморегулятора](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) определяет телеметрию, свойства и команды, реализуемые устройством.

Код устройства подключается к центру Интернета вещей с помощью стандартного метода `CreateFromConnectionString`. Устройство отправляет идентификатор модели DTDL, которую он реализует в запросе на подключение. Устройство, отправляющее идентификатор модели, — это устройство IoT Plug and Play:

```csharp
private static void InitializeDeviceClientAsync()
{
  var options = new ClientOptions
  {
    ModelId = ModelId,
  };
  s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt, options);
  s_deviceClient.SetConnectionStatusChangesHandler((status, reason) =>
  {
     s_logger.LogDebug($"Connection status change registered - status={status}, reason={reason}.");
  });
}
```

Идентификатор модели хранится в коде, как показано в следующем фрагменте кода:

```csharp
private const string ModelId = "dtmi:com:example:Thermostat;1";
```

Код, который обновляет свойства, обрабатывает команды и отправляет данные телеметрии, идентичен коду устройства, которое не использует соглашения IoT Plug and Play.

В примере используется библиотека JSON для анализа объектов JSON в полезных данных, отправленных из центра Интернета вещей:

```csharp
using Newtonsoft.Json;

...

DateTime since = JsonConvert.DeserializeObject<DateTime>(request.DataAsJson);
```
