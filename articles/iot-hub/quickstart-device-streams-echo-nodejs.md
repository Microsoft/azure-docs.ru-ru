---
title: Краткое руководство. Взаимодействие с приложением устройства в Node.js с помощью потоков устройств Центра Интернета вещей Azure
description: В этом кратком руководстве будет выполняться приложение Node.js на стороне службы, которое обменивается данными с устройством IoT через поток устройств.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: quickstart
ms.custom: mvc, devx-track-js, devx-track-azurecli
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: 335014f032162866e4780bf1294ddcd108b4fd03
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98624394"
---
# <a name="quickstart-communicate-to-a-device-application-in-nodejs-via-iot-hub-device-streams-preview"></a>Краткое руководство. Взаимодействие с приложением устройства в Node.js с помощью потоков устройств Центра Интернета вещей (предварительная версия)

[!INCLUDE [iot-hub-quickstarts-3-selector](../../includes/iot-hub-quickstarts-3-selector.md)]

В этом кратком руководстве показано, как запустить приложение на стороне службы и настроить связь между устройством и службой, используя потоки устройств. Потоки устройств Центра Интернета вещей Azure позволяют службам и приложениям устройств безопасным и подходящим методом обмениваться данными с брандмауэром. На этапе общедоступной предварительной версии пакет SDK для Node.js поддерживает только потоки устройств на стороне службы. В результате это краткое руководство охватывает только инструкции по запуску приложения на стороне службы.

## <a name="prerequisites"></a>Предварительные требования

* Выполненные инструкции из руководства [Обмен данными с приложениями устройств на C с помощью потоков устройств Центра Интернета вещей](./quickstart-device-streams-echo-c.md) или [Обмен данными с приложениями устройств на C# с помощью потоков устройств Центра Интернета вещей](./quickstart-device-streams-echo-csharp.md).

* Учетная запись Azure с активной подпиской. [Создайте бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Node.js версии 10 и выше](https://nodejs.org).

    Текущую версию Node.js на компьютере, на котором ведется разработка, можно проверить, используя следующую команду:

    ```cmd/sh
    node --version
    ```

* [Пример проекта Node.js](https://github.com/Azure-Samples/azure-iot-samples-node/archive/streams-preview.zip).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

Центр Интернета вещей Microsoft Azure поддерживает потоки устройств, которые сейчас доступны в режиме [предварительной версии](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

> [!IMPORTANT]
> Предварительная версия потоков устройств сейчас поддерживается только в Центрах Интернета вещей, созданных в следующих регионах.
>
> * Центральная часть США
> * Центральная часть США (EUAP)
> * Северная Европа
> * Юго-Восточная Азия

## <a name="create-an-iot-hub"></a>Создание Центра Интернета вещей

Если вы закончили работу с предыдущим [руководством по отправке данных телеметрии с устройства в Центр Интернета вещей](quickstart-send-telemetry-node.md), этот шаг можно пропустить.

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Регистрация устройства

Если вы закончили работу с предыдущим [руководством по отправке данных телеметрии с устройства в Центр Интернета вещей](quickstart-send-telemetry-node.md), этот шаг можно пропустить.

Устройство должно быть зарегистрировано в Центре Интернета вещей, прежде чем оно сможет подключиться. В этом кратком руководстве для регистрации имитируемого устройства используется Azure Cloud Shell.

1. Выполните приведенные ниже команды в Azure Cloud Shell, чтобы создать удостоверение устройства.

   **YourIoTHubName**. Замените этот заполнитель именем вашего центра Интернета вещей.

   **MyDevice**. Это имя регистрируемого устройства. Рекомендуется использовать **MyDevice**, как показано ниже. Если вы выбрали другое имя для устройства, используйте его при работе с этим руководством и обновите имя устройства в примерах приложений перед их запуском.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

2. Чтобы разрешить внутреннему приложению подключаться к Центру Интернета вещей и получать сообщения, вам необходима *строка подключения к службе*. Следующая команда извлекает строку подключения службы для Центра Интернета вещей:

    **YourIoTHubName**. Замените этот заполнитель именем вашего центра Интернета вещей.

    ```azurecli-interactive
    az iot hub connection-string show --policy-name service --name {YourIoTHubName} --output table
    ```

    Запишите возвращенную строку подключения к службе для последующего использования в этом кратком руководстве. Это должно выглядеть следующим образом:

   `"HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"`

## <a name="communicate-between-device-and-service-via-device-streams"></a>Обмен данными между устройством и службой через потоки устройств

В рамках этого раздела вы запустите приложения на стороне устройства и на стороне службы, а также настроите обмен данными между двумя этими приложениями.

### <a name="run-the-device-side-application"></a>Запуск приложения на стороне устройства

Как упоминалось ранее, пакет SDK для Node.js Центра Интернета вещей поддерживает только потоки устройств на стороне службы. Для приложения на стороне устройства используйте одну из сопутствующих программ устройства, которая доступна в этих кратких руководствах:

* [Обмен данными с приложениями устройств в C с помощью потоков устройств Центра Интернета вещей](./quickstart-device-streams-echo-c.md)

* [Обмен данными с приложениями устройств в C# с помощью потоков устройств Центра Интернета вещей](./quickstart-device-streams-echo-csharp.md)

Прежде чем перейти к следующему шагу, убедитесь, что приложение на стороне устройства запущено.

### <a name="run-the-service-side-application"></a>Запуск приложения на стороне службы

Приложение Node.js на стороне службы, используемое в этом кратком руководстве, имеет следующие функции:

* создание потока устройств к устройству IoT;
* считывание введенной в командную строку информации и отправка в приложение устройства, которое отправит ее обратно.

Код продемонстрирует процесс инициализации потока устройств, а также его использование для отправки и получения данных.

Если приложение на стороне устройства запущено, выполните следующие действия в окне терминала на локальном компьютере, чтобы запустить приложение на стороне службы в Node.js.

* Укажите свои учетные данные службы и идентификатор устройства в качестве переменных среды.
 
   ```cmd/sh
   # In Linux
   export IOTHUB_CONNECTION_STRING="{ServiceConnectionString}"
   export STREAMING_TARGET_DEVICE="MyDevice"

   # In Windows
   SET IOTHUB_CONNECTION_STRING={ServiceConnectionString}
   SET STREAMING_TARGET_DEVICE=MyDevice
   ```
  
   Измените заполнитель ServiceConnectionString, чтобы он соответствовал строке подключения к службе, и **MyDevice**, чтобы он соответствовал коду устройства, если вы указали другое имя.

* Перейдите к папке `Quickstarts/device-streams-service` в распакованной папке проекта и запустите образец, используя узел.

   ```cmd/sh
   cd azure-iot-samples-node-streams-preview/iot-hub/Quickstarts/device-streams-service
    
   # Install the preview service SDK, and other dependencies
   npm install azure-iothub@streams-preview
   npm install

   node echo.js
   ```

После последнего шага программа на стороне службы инициирует поток к устройству и после установления соединения отправит службе через поток строковый буфер. В этом примере программа на стороне службы просто считывает данные из `stdin`, введенные в окне терминала, и отправляет на устройство, которое затем возвращает их обратно. Это демонстрирует успешную двунаправленную связь между приложениями.

![Выходные данные консоли на стороне службы](./media/quickstart-device-streams-echo-nodejs/service-console-output.png)

Теперь можно завершить выполнение программы, повторно нажав клавишу ВВОД.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [iot-hub-quickstarts-clean-up-resources-device-streams](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы настроили Центр Интернета вещей, зарегистрировали устройство, установили поток устройств между приложением на стороне устройства и службы и использовали поток для обмена данными между приложениями.

Используйте приведенные ниже ссылки для получения дополнительных сведений о потоках устройства:

> [!div class="nextstepaction"]
> [IoT Hub Device Streams (preview)](./iot-hub-device-streams-overview.md) (Потоки устройств (предварительная версия))
