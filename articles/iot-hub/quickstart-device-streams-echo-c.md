---
title: Краткое руководство. Взаимодействие с приложением устройства на языке C с помощью потоков устройств Центра Интернета вещей Azure
description: С помощью этого краткого руководства вы запустите приложение C на стороне устройства, которое обменивается данными с устройством Интернета вещей через поток устройств.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli
ms.date: 08/20/2019
ms.author: robinsh
ms.openlocfilehash: fd179bf17f481ae84f768d619c6ddc0b9afab280
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102202558"
---
# <a name="quickstart-communicate-to-a-device-application-in-c-via-iot-hub-device-streams-preview"></a>Краткое руководство. Взаимодействие с приложением устройства в C с помощью потоков устройств Центра Интернета вещей (предварительная версия)

[!INCLUDE [iot-hub-quickstarts-3-selector](../../includes/iot-hub-quickstarts-3-selector.md)]

Центр Интернета вещей Azure поддерживает потоки устройств, которые сейчас доступны в режиме [предварительной версии](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[Потоки устройств Центра Интернета вещей](iot-hub-device-streams-overview.md) позволяют службам и приложениям устройств безопасным и подходящим методом обмениваться данными с брандмауэром. На этапе предварительной версии пакет SDK для C поддерживает только потоки устройств на стороне устройства. В результате это краткое руководство охватывает инструкции по запуску приложения только на стороне устройства. Подробные сведения о том, как запустить соответствующее приложение на стороне службы, см. в следующих статьях:

* [Обмен данными с приложениями устройств в C# с помощью потоков устройств Центра Интернета вещей](./quickstart-device-streams-echo-csharp.md)

* [Взаимодействие с приложением устройства в Node.js с помощью потоков устройств Центра Интернета вещей (предварительная версия)](./quickstart-device-streams-echo-nodejs.md)

Приложение C на стороне устройства, используемое в этом кратком руководстве, имеет следующие функции:

* установка потока устройств к устройству Интернета вещей;

* получение данных, отправленных приложением со стороны службы, и отправка их обратно.

Код демонстрирует процесс инициализации потока устройств, а также его использование для отправки и получения данных.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Вам потребуется сделать следующее:

* Установите [Visual Studio 2019](https://www.visualstudio.com/vs/) с включенной рабочей нагрузкой **Разработка классических приложений на C++**.

* Установите последнюю версию [Git](https://git-scm.com/download/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

Предварительная версия потоков устройств сейчас поддерживается только в центрах Интернета вещей, созданных в следующих регионах:

  * Центральная часть США
  * Центральная часть США (EUAP)
  * Северная Европа
  * Юго-Восточная Азия

## <a name="prepare-the-development-environment"></a>Подготовка среды разработки

При работе с этим кратким руководством используйте [пакет SDK для устройств Azure IoT для C](iot-hub-device-sdk-c-intro.md). Вы подготовите среду разработки, которая используется для клонирования и сборки [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c) с сайта GitHub. Пакет SDK на сайте GitHub содержит пример кода, используемый в этом кратком руководстве.

   > [!NOTE]
   > Перед началом этой процедуры убедитесь, что Visual Studio устанавливается вместе с рабочей нагрузкой **Разработка классических приложений на C++**.

1. Установите [систему сборки CMake](https://cmake.org/download/), как описано на странице скачивания.

1. Откройте командную строку или оболочку Git Bash. Выполните следующие команды для клонирования репозитория GitHub [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c).

    ```cmd/sh
    git clone -b public-preview https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Это должно занять несколько минут.

1. Создайте подкаталог *cmake* в корневом каталоге репозитория git и перейдите в эту папку. В каталоге *azure-iot-sdk-c* выполните следующие команды:

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

1. Выполните приведенную ниже команду из каталога *cmake*, чтобы создать версию пакета SDK для используемой клиентской платформы разработки.

   * В Linux:

      ```bash
      cmake ..
      make -j
      ```

   * В Windows откройте [командную строку разработчика для Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs). Выполните команду для своей версии Visual Studio. В этом кратком руководстве используется Visual Studio 2019. Эти команды также создают решение Visual Studio для имитированного устройства в каталоге *cmake*.

      ```cmd
      rem For VS2015
      cmake .. -G "Visual Studio 14 2015"

      rem Or for VS2017
      cmake .. -G "Visual Studio 15 2017"

      rem Or for VS2019
      cmake .. -G "Visual Studio 16 2019"

      rem Then build the project
      cmake --build . -- /m /p:Configuration=Release
      ```

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Регистрация устройства

Устройство должно быть зарегистрировано в центре Интернета вещей, прежде чем оно сможет подключиться. В этом разделе описано, как зарегистрировать имитированное устройство с помощью [расширения Интернета вещей](/cli/azure/ext/azure-iot/iot) для Azure Cloud Shell.

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

## <a name="communicate-between-the-device-and-the-service-via-device-streams"></a>Обмен данными между устройством и службой через потоки устройств

В рамках этого раздела вы запустите приложения на стороне устройства и на стороне службы, а также настроите обмен данными между двумя этими приложениями.

### <a name="run-the-device-side-application"></a>Запуск приложения на стороне устройства

Для запуска приложения на стороне устройства выполните следующие шаги:

1. Укажите учетные данные устройства, отредактировав исходный файл **iothub_client_c2d_streaming_sample.c** в папке `iothub_client/samples/iothub_client_c2d_streaming_sample` и добавив строку подключения к устройству.

   ```C
   /* Paste in your iothub connection string  */
   static const char* connectionString = "{DeviceConnectionString}";
   ```

1. Скомпилируйте код, выполнив следующие команды:

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_sample
   make -j
   ```

   ```cmd
   rem In Windows
   rem Go to the cmake folder at the root of repo
   cmake --build . -- /m /p:Configuration=Release
   ```

1. Запустите скомпилированную программу:

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_sample
   ./iothub_client_c2d_streaming_sample
   ```

   ```cmd
   rem In Windows
   rem Go to the sample's release folder cmake\iothub_client\samples\iothub_client_c2d_streaming_sample\Release
   iothub_client_c2d_streaming_sample.exe
   ```

### <a name="run-the-service-side-application"></a>Запуск приложения на стороне службы

Как упоминалось ранее, пакет SDK для C центра Интернета вещей поддерживает потоки устройств только на стороне устройства. Чтобы скомпилировать и запустить сопутствующее приложение на стороне службы, выполните инструкции, приведенные в одном из следующих кратких руководств.

* [Взаимодействие с приложениями устройств в C# с помощью потоков устройств Центра Интернета вещей (предварительная версия)](./quickstart-device-streams-echo-csharp.md)

* [Взаимодействие с приложением устройства в Node.js с помощью потоков устройств Центра Интернета вещей (предварительная версия)](./quickstart-device-streams-echo-nodejs.md)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-hub-quickstarts-clean-up-resources-device-streams](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы настроили центр Интернета вещей, зарегистрировали устройство, установили поток устройств между приложением C на стороне устройства и еще одним приложением на стороне службы, а затем использовали поток для обмена данными между приложениями.

Дополнительные сведения о потоках устройств см. в следующей статье:

> [!div class="nextstepaction"]
> [IoT Hub Device Streams (preview)](./iot-hub-device-streams-overview.md) (Потоки устройств (предварительная версия))
