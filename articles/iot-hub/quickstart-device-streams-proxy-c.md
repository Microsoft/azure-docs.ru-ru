---
title: Краткое руководство. Потоки устройств Центра Интернета вещей в C для SSH и RDP
description: В этом кратком руководстве будет выполняться запуск примера приложения C, которое выступает в качестве прокси-приложения, для реализации сценариев SSH и RDP через потоки устройств центра Интернета вещей.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom: references_regions, devx-track-azurecli
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: c9c09914e30bdeae809924bc4256c773f42abd52
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/14/2021
ms.locfileid: "107479946"
---
# <a name="quickstart-enable-ssh-and-rdp-over-an-iot-hub-device-stream-by-using-a-c-proxy-application-preview"></a>Краткое руководство. Подключение по протоколу SSH и RDP через поток устройств центра Интернета вещей с помощью прокси-приложения C (предварительная версия)

[!INCLUDE [iot-hub-quickstarts-4-selector](../../includes/iot-hub-quickstarts-4-selector.md)]

Центр Интернета вещей Azure поддерживает потоки устройств, которые сейчас доступны в режиме [предварительной версии](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[Потоки устройств Центра Интернета вещей](./iot-hub-device-streams-overview.md) позволяют службам и приложениям устройств безопасным и подходящим методом обмениваться данными с брандмауэром. Общие сведения о настройке cм. на странице [Пример локального прокси-приложения](./iot-hub-device-streams-overview.md#local-proxy-sample-for-ssh-or-rdp).

В этом кратком руководстве описывается настройка для туннелирования трафика Secure Shell (SSH) (используя порт 22) через потоки устройств. Настройка для трафика протокола удаленного рабочего стола (RDP) выполняется аналогично и требует простого изменения конфигурации. Так как потоки устройств не зависят от приложений и протоколов, вы можете изменить это краткое руководство для размещения других типов трафика приложений.

## <a name="prerequisites"></a>Предварительные требования

* Предварительная версия потоков устройств сейчас поддерживается только в центрах Интернета вещей, созданных в следующих регионах:

  * Центральная часть США
  * Центральная часть США (EUAP)
  * Северная Европа
  * Юго-Восточная Азия

* Установите [Visual Studio 2019](https://www.visualstudio.com/vs/) с включенной рабочей нагрузкой [Разработка классических приложений на C++](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/).
* Установите последнюю версию [Git](https://git-scm.com/download/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="how-it-works"></a>Принцип работы

На рисунке, приведенном ниже, показано, как программы локального прокси-сервера устройств и служб обеспечивают сквозное подключение между клиентом SSH и процессами управляющей программы SSH. На этапе предварительной версии пакет SDK для C поддерживает только потоки устройств на стороне устройства. В результате это краткое руководство охватывает инструкции по запуску только приложения прокси-устройства на стороне службы. Чтобы скомпилировать и запустить сопутствующее приложение на стороне службы, выполните инструкции, приведенные в одном из следующих кратких руководств.

* [SSH/RDP через потоки устройств Центра Интернета вещей с помощью прокси-приложения на C#](./quickstart-device-streams-proxy-csharp.md)
* [SSH/RDP через потоки устройств Центра Интернета вещей с помощью прокси-приложения на NodeJS](./quickstart-device-streams-proxy-nodejs.md)

![Настройка локального прокси-приложения](./media/quickstart-device-streams-proxy-c/device-stream-proxy-diagram.png)

1. Локальное прокси-приложение службы подключается к центру Интернета вещей и запускает поток устройства на целевое устройство.

2. Локальный прокси-сервер устройства завершает подтверждение инициации потоковой передачи и устанавливает сквозной потоковый туннель через конечную точку потоковой передачи центра Интернета вещей на стороне службы.

3. Локальный прокси-сервер устройства подключается к управляющей программе SSH, ожидая передачи данных к устройству через порт 22. Этот параметр можно настроить, используя инструкции, описанные в разделе "Запуск приложения локального прокси-сервера устройства".

4. Локальный прокси-сервер службы ожидает новые подключения SSH от пользователя, прослушивая назначенный порт, который в этом случае является портом 2222. Этот параметр можно настроить, используя инструкции, описанные в разделе "Запуск приложения локального прокси-сервера устройства". Когда пользователь подключается через клиент SSH, туннель позволяет передавать трафик приложения SSH между клиентом SSH и программами сервера.

> [!NOTE]
> Трафик SSH, передаваемый по потоку устройств, будет туннелироваться через конечную точку потоковой передачи центра Интернета вещей, а не напрямую между службой и устройством. Дополнительные сведения см. в [списке преимуществ использования потоков устройств центра Интернета вещей](iot-hub-device-streams-overview.md#benefits). Кроме того, на рисунке показана управляющая программа SSH, запущенная на том же устройстве (или компьютере), что и локальный прокси-сервер устройства. В этом кратком руководстве предоставление IP-адреса управляющей программы SSH позволяет локальному прокси-серверу устройства и управляющей программе работать на разных компьютерах.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prepare-the-development-environment"></a>Подготовка среды разработки

При работе с этим кратким руководством используйте [пакет SDK для устройств Azure IoT для C](iot-hub-device-sdk-c-intro.md). Вы подготовите среду разработки, которая используется для клонирования и сборки [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c) с сайта GitHub. Пакет SDK на сайте GitHub содержит пример кода, используемый в этом кратком руководстве.

1. Скачайте [систему сборки CMake](https://cmake.org/download/).

    *Перед* установкой CMake важно установить на компьютер необходимые компоненты Visual Studio (Visual Studio и рабочую нагрузку *Разработка классических приложений на C++*). После установки компонентов и проверки загрузки вы можете установить систему сборки CMake.

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

   * В Windows выполните приведенные ниже команды в Командной строке разработчика Visual Studio 2015 или 2017. Эта команда также создает решение Visual Studio для имитированного устройства в каталоге *cmake*.

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

Устройство должно быть зарегистрировано в Центре Интернета вещей, прежде чем оно сможет подключиться. В этом разделе описано, как зарегистрировать имитированное устройство с помощью [расширения Интернета вещей](/cli/azure/ext/azure-iot/iot) для Azure Cloud Shell.

1. Чтобы создать удостоверение устройства, выполните приведенные ниже команды в Cloud Shell.

   > [!NOTE]
   > * Замените заполнитель *YourIoTHubName* именем созданного центра Интернета вещей.
   > * Для имени регистрируемого устройства рекомендуется использовать имя *MyDevice*, как показано в примере. Если вы выбрали другое имя для устройства, используйте его при работе с этой статьей и обновите имя устройства в примерах приложений перед их запуском.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

1. Выполните следующие команды в Cloud Shell, чтобы получить *строку подключения* зарегистрированного устройства.

   > [!NOTE]
   > Замените заполнитель *YourIoTHubName* именем созданного центра Интернета вещей.

    ```azurecli-interactive
    az iot hub device-identity connection-string show --hub-name {YourIoTHubName} --device-id MyDevice --output table
    ```

    Запишите возвращенную строку подключения к устройству для последующего использования в этом кратком руководстве. Это должно выглядеть следующим образом:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyDevice;SharedAccessKey={YourSharedAccessKey}`

## <a name="ssh-to-a-device-via-device-streams"></a>SSH-подключение к устройству через потоки устройств

В этом разделе показано, как установить сквозной поток для туннелирования трафика SSH.

### <a name="run-the-device-local-proxy-application"></a>Запуск приложения локального прокси-сервера устройства

1. Отредактируйте исходный файл **iothub_client_c2d_streaming_proxy_sample.c** в папке `iothub_client/samples/iothub_client_c2d_streaming_proxy_sample` и укажите строку подключения к устройству, IP-адрес или имя узла целевого устройства, а также порт SSH 22.

   ```C
   /* Paste in your device connection string  */
   static const char* connectionString = "{DeviceConnectionString}";
   static const char* localHost = "{IP/Host of your target machine}"; // Address of the local server to connect to.
   static const size_t localPort = 22; // Port of the local server to connect to.
   ```

1. Компиляция образца

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_proxy_sample
   make -j
   ```

   ```cmd
   rem In Windows
   rem Go to cmake at root of repository
   cmake --build . -- /m /p:Configuration=Release
   ```

1. Запустите скомпилированную программу на устройстве:

   ```bash
   # In Linux
   # Go to the sample's folder cmake/iothub_client/samples/iothub_client_c2d_streaming_proxy_sample
   ./iothub_client_c2d_streaming_proxy_sample
   ```

   ```cmd
   rem In Windows
   rem Go to the sample's release folder cmake\iothub_client\samples\iothub_client_c2d_streaming_proxy_sample\Release
   iothub_client_c2d_streaming_proxy_sample.exe
   ```

### <a name="run-the-service-local-proxy-application"></a>Запуск приложения локального прокси-сервера службы

Как обсуждалось в разделе "Принцип работы", для создания сквозного потока для туннелирования трафика SSH требуется локальное прокси-приложение на каждом конце (как на службе, так и на устройстве). На этапе предварительной версии пакет SDK для центра Интернета вещей для C поддерживает только потоки устройств на стороне устройства. Чтобы создать и запустить локальный прокси-сервер службы, следуйте инструкциям в одном из следующих кратких руководств:

   * [SSH/RDP через потоки устройств Центра Интернета вещей с помощью прокси-приложения на C#](./quickstart-device-streams-proxy-csharp.md)
   * [SSH/RDP через потоки устройств Центра Интернета вещей с помощью прокси-приложения на Node.js](./quickstart-device-streams-proxy-nodejs.md)

### <a name="establish-an-ssh-session"></a>Установка сеанса SSH

После запуска локальных прокси-серверов устройств и служб используйте клиентскую программу SSH и подключитесь к локальному прокси-серверу службы через порт 2222 (вместо подключения напрямую через управляющую программу SSH).

```cmd/sh
ssh {username}@localhost -p 2222
```

На этом этапе в окне входа в SSH можно ввести учетные данные.

На следующем рисунке показан вывод на консоль на локальном прокси-сервере устройства, который подключается к управляющей программе SSH по адресу `IP_address:22`.

![Вывод локального прокси-приложения на устройстве](./media/quickstart-device-streams-proxy-c/device-console-output.png)

На рисунке ниже показаны выходные данные консоли клиентской программы SSH. Клиент SSH связывается с управляющей программой SSH, подключаясь к порту 22, который прослушивается локальным прокси-сервером службы.

![Вывод клиента SSH](./media/quickstart-device-streams-proxy-csharp/ssh-console-output.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Дальнейшие действия

В рамках этого краткого руководства вы настроили центр Интернета вещей, зарегистрировали устройство, развернули программу локальных прокси-серверов устройств и служб, чтобы установить потоковую передачу устройств через центр Интернета вещей, а также использовали прокси-серверы для туннелирования трафика SSH.

Дополнительные сведения о потоках устройств см. в следующей статье:

> [!div class="nextstepaction"]
> [IoT Hub Device Streams (preview)](./iot-hub-device-streams-overview.md) (Потоки устройств (предварительная версия))
