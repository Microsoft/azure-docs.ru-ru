---
title: Учебник. Подключение универсального модуля Azure IoT Plug and Play | Документация Майкрософт
description: Учебник. Использование примера кода устройства IoT Plug and Play на C# в универсальном модуле.
author: ericmitt
ms.author: ericmitt
ms.date: 9/22/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: b981352dcfc5d3d2de2d773e2936feb27b6ba3d4
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106064704"
---
# <a name="tutorial-connect-an-iot-plug-and-play-module-c"></a>Руководство по подключению модуля IoT Plug and Play (C#)

В этом руководстве показано, как подключить универсальный [модуль](../iot-hub/iot-hub-devguide-module-twins.md) IoT Plug and Play.

Устройство — это устройство IoT Plug and Play, если оно публикует свой идентификатор модели при подключении к центру Интернета вещей и реализует свойства и методы, описанные в модели DTDL, определяемой идентификатором модели. Дополнительные сведения о том, как устройства используют DTDL и идентификатор модели, см. в статье [Руководство разработчика IoT Plug and Play](./concepts-developer-guide-device.md). Модули используют идентификаторы моделей и модели DTDL одинаковым образом.

Для демонстрации того, как реализовать модуль IoT Plug and Play, в этом учебнике описывается следующее:

> [!div class="checklist"]
> * добавление устройства с помощью модуля я в центр Интернета вещей;
> * преобразование примера устройства термостата на C# в общий модуль;
> * использование пакета SDK службы для взаимодействия с модулем.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [iot-pnp-prerequisites](../../includes/iot-pnp-prerequisites.md)]

Для выполнения инструкций из этого учебника в ОС Windows установите в локальной среде Windows такое программное обеспечение:

* [Visual Studio (Community, Professional или Enterprise)](https://visualstudio.microsoft.com/downloads/).
* [Git](https://git-scm.com/download/).

Используйте средство Обозревателя Интернета вещей Azure, чтобы добавить новое устройство с именем **my-module-device** в центр Интернета вещей.

Добавьте модуль с именем **my-module** в **my-module-device**:

1. В средстве Обозревателя Интернета вещей Azure перейдите к устройству **my-module-device**.

1. Выберите **Удостоверение модуля**, а затем выберите **Добавить**.

1. Введите **my-module** в качестве имени удостоверения модуля и нажмите кнопку **Сохранить**.

1. В списке удостоверений модулей выберите **my-module**. Затем скопируйте основную строку подключения. Строка подключения модуля понадобится позже.

1. Перейдите на вкладку **Двойник модуля** и обратите внимание, что на ней нет требуемых или наблюдаемых свойств:

    ```json
    {
      "deviceId": "my-module-device",
      "moduleId": "my-module",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag": "NjgzMzQ1MzQ1",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00Z",
      "connectionState": "Disconnected",
      "lastActivityTime": "0001-01-01T00:00:00Z",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "modelId": "",
      "version": 2,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "0001-01-01T00:00:00Z"
          },
          "$version": 1
          },
          "reported": {
            "$metadata": {
              "$lastUpdated": "0001-01-01T00:00:00Z"
          },
          "$version": 1
        }
      }
    }
    ```

## <a name="download-the-code"></a>Загрузка кода

Если вы еще не сделали этого, клонируйте репозиторий пакета SDK C# для устройств Центра Интернета вещей Azure на локальный компьютер:

Откройте командную строку в выбранной папке. Выполните следующую команду, чтобы клонировать репозиторий GitHub [Примеров C# для Интернета вещей Azure](https://github.com/Azure-Samples/azure-iot-samples-csharp) в следующее расположение:

```cmd
git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git
```

## <a name="prepare-the-project"></a>Подготовка проекта

Чтобы открыть и подготовить пример проекта, выполните следующие действия.

1. Откройте файл проекта *azure-iot-sdk-csharp\iot-hub\Samples\device\PnpDeviceSamples\Thermostat\Thermostat.csproj* в Visual Studio 2019.

1. В Visual Studio последовательно выберите **Проект > Thermostat Properties (Свойства термостата) > Отладка**. Затем добавьте в проект следующие переменные среды:

    | Имя | Значение |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | connectionString |
    | IOTHUB_MODULE_CONNECTION_STRING | Строка подключения модуля, подготовленная ранее |

    Дополнительные сведения о примере конфигурации см. в [образце файла сведений](https://github.com/Azure-Samples/azure-iot-samples-csharp/blob/master/iot-hub/Samples/device/PnpDeviceSamples/readme.md).

## <a name="modify-the-code"></a>вносит изменения в код;

Чтобы изменить код для работы в качестве модуля, а не устройства, выполните следующие действия.

1. В Visual Studio откройте файл *Parameter.cs* и измените строку, которая задает переменную **PrimaryConnectionString**, следующим образом:

    ```csharp
    public string PrimaryConnectionString { get; set; } = Environment.GetEnvironmentVariable("IOTHUB_MODULE_CONNECTION_STRING");
    ```

1. В Visual Studio откройте файл *Program.cs* и замените семь экземпляров класса `DeviceClient` классом `ModuleClient`.

    > [!TIP]
    > Воспользуйтесь поиском и заменой в Visual Studio с параметрами **Match case** (Учитывать регистр) и **Match whole word** (Слово целиком), чтобы заменить `DeviceClient` на `ModuleClient`.

1. В Visual Studio откройте файл *Thermostat.cs* и замените оба экземпляра класса `DeviceClient` классом `ModuleClient` следующим образом.

1. Сохраните измененные файлы.

Если запустить код и использовать Обозреватель Интернета вещей Azure для просмотра обновленного модуля двойника, появится обновленный двойник устройства с идентификатором модели и свойством наблюдаемого модуля:

```json
{
  "deviceId": "my-module-device",
  "moduleId": "my-mod",
  "etag": "AAAAAAAAAAE=",
  "deviceEtag": "NjgzMzQ1MzQ1",
  "status": "enabled",
  "statusUpdateTime": "0001-01-01T00:00:00Z",
  "connectionState": "Connected",
  "lastActivityTime": "0001-01-01T00:00:00Z",
  "cloudToDeviceMessageCount": 0,
  "authenticationType": "sas",
  "x509Thumbprint": {
    "primaryThumbprint": null,
    "secondaryThumbprint": null
  },
  "modelId": "dtmi:com:example:Thermostat;1",
  "version": 3,
  "properties": {
    "desired": {
      "$metadata": {
        "$lastUpdated": "0001-01-01T00:00:00Z"
      },
      "$version": 1
    },
    "reported": {
      "maxTempSinceLastReboot": 5,
      "$metadata": {
        "$lastUpdated": "2020-09-28T08:53:45.9956637Z",
        "maxTempSinceLastReboot": {
          "$lastUpdated": "2020-09-28T08:53:45.9956637Z"
        }
      },
      "$version": 2
    }
  }
}
```

## <a name="interact-with-a-device-module"></a>Взаимодействие с модулем устройства

Пакеты SDK для служб позволяют получить идентификатор модели подключенных устройств и модулей IoT Plug and Play. Пакеты SDK для служб можно использовать для задания перезаписываемых свойств и команд вызова:

1. В другом экземпляре Visual Studio откройте проект *azure-iot-sdk-csharp\iot-hub\Samples\service\PnpServiceSamples\Thermostat\Thermostat.csproj*.

1. В Visual Studio последовательно выберите **Проект > Thermostat Properties (Свойства термостата) > Отладка**. Затем добавьте в проект следующие переменные среды:

    | Имя | Значение |
    | ---- | ----- |
    | IOTHUB_DEVICE_ID | my-module-device |
    | IOTHUB_CONNECTION_STRING | Значение, которое записали после завершения [настройки среды](set-up-environment.md). |

    > [!TIP]
    > Строку подключения центра Интернета вещей также можно найти в средстве Обозревателя Интернета вещей Azure.

1. Откройте файл *Program.cs* и измените строку вызова команды следующим образом:

    ```csharp
    CloudToDeviceMethodResult result = await s_serviceClient.InvokeDeviceMethodAsync(s_deviceId, "my-module", commandInvocation);
    ```

1. В файле *Program.cs* измените строку, которая получает двойник устройства, следующим образом:

    ```csharp
    Twin twin = await s_registryManager.GetTwinAsync(s_deviceId, "my-module");
    ```

1. Убедитесь, что образец клиента модуля все еще работает, а затем запустите этот пример службы. В выходных данных примера службы показан идентификатор модели из двойника устройства, а также вызов команды:

    ```cmd
    [09/28/2020 10:52:55]dbug: Thermostat.Program[0]
      Initialize the service client.
    [09/28/2020 10:52:55]dbug: Thermostat.Program[0]
      Get Twin model Id and Update Twin
    [09/28/2020 10:52:59]dbug: Thermostat.Program[0]
      Model Id of this Twin is: dtmi:com:example:Thermostat;1
    [09/28/2020 10:52:59]dbug: Thermostat.Program[0]
      Invoke a command
    [09/28/2020 10:53:00]dbug: Thermostat.Program[0]
      Command getMaxMinReport invocation result status is: 200
    ```

    Выходные данные клиента модуля содержат ответ обработчика команд:

    ```cmd
    [09/28/2020 10:53:00]dbug: Thermostat.ThermostatSample[0]
      Command: Received - Generating max, min and avg temperature report since 28/09/2020 10:52:55.
    [09/28/2020 10:53:00]dbug: Thermostat.ThermostatSample[0]
      Command: MaxMinReport since 28/09/2020 10:52:55: maxTemp=25.4, minTemp=25.4, avgTemp=25.4, startTime=28/09/2020 10:52:56, endTime=28/09/2020 10:52:56
    ```

## <a name="convert-to-an-iot-edge-module"></a>Преобразование в модуль IoT Edge

Чтобы преобразовать этот пример для работы в качестве модуля IoT Edge IoT Plug and Play, необходимо контейнеризовать приложение. Дополнительные изменения кода не требуются. Переменная среды строки подключения инжектируется средой выполнения IoT Edge при запуске. Узнать больше можно в статье [Use Visual Studio 2019 to develop and debug modules for Azure IoT Edge](../iot-edge/how-to-visual-studio-develop-module.md) (Разработка и отладка модулей для Azure IoT Edge с помощью Visual Studio 2019).

Дополнительные сведения о развертывании контейнеризованного модуля см. в следующих статьях:

* [Запуск Azure IoT Edge на виртуальных машинах Ubuntu](../iot-edge/how-to-install-iot-edge-ubuntuvm.md)
* [Установка среды выполнения Azure IoT Edge в системах Linux на основе Debian](../iot-edge/how-to-install-iot-edge.md)

Средство Обозревателя Интернета вещей Azure можно использовать для просмотра следующих компонентов:

* Идентификатор модели устройства IoT Edge в двойнике модуля.
* Данные телеметрии с устройства IoT Edge.
* Свойство двойника модуля IoT Edge обновляет активацию уведомлений IoT Plug and Play.
* Модуль IoT Edge реагирует на команды IoT Plug and Play.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-pnp-clean-resources](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как подключить устройство IoT Plug and Play с модулями к центру Интернета вещей. Дополнительные сведения о моделях устройства IoT Plug and Play см. в статье

> [!div class="nextstepaction"]
> [Руководство разработчика IoT Plug and Play](./concepts-developer-guide-device.md)