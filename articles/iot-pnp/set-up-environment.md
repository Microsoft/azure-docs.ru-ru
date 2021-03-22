---
title: Краткое руководство. Настройка ресурсов Интернета вещей, требуемых для IoT Plug and Play | Документация Майкрософт
description: Краткое руководство. Создание центра Интернета вещей и экземпляра Службы подготовки устройств для использования с краткими руководствами и учебниками по IoT Plug and Play.
author: dominicbetts
ms.author: dobett
ms.date: 08/11/2020
ms.topic: quickstart
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 5446f8481439dabffe67f9b00b5f65b1da495746
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101742659"
---
# <a name="quickstart-set-up-your-environment-for-the-iot-plug-and-play-quickstarts-and-tutorials"></a>Краткое руководство. Настройка среды для работы с краткими руководствами и учебниками по IoT Plug and Play

Перед началом работы с краткими руководствами и учебниками по IoT Plug and Play необходимо настроить центр Интернета вещей и Службу подготовки устройств (DPS) в подписке Azure. Кроме того, вам понадобятся локальные копии файлов модели, используемых в примерах приложений, и обозреватель Интернета вещей Azure.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

Чтобы не устанавливать Azure CLI локально, можно использовать Azure Cloud Shell для настройки облачных служб.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resources"></a>Создание ресурсов

Создайте группу ресурсов Azure для ресурсов:

```azurecli-interactive
az group create --name my-pnp-resourcegroup --location centralus
```

Создайте Центр Интернета вещей. Следующая команда использует имя `my-pnp-hub` в качестве примера имени создаваемого центра Интернета вещей. Выберите уникальное имя центра Интернета вещей, которое будет использоваться вместо `my-pnp-hub`:

```azurecli-interactive
az iot hub create --name my-pnp-hub --resource-group my-pnp-resourcegroup --sku F1 --partition-count 2
```

Создайте экземпляр DPS. Следующая команда использует имя `my-pnp-dps` в качестве примера имени создаваемого экземпляра DPS. Выберите уникальное имя экземпляра DPS, которое будет использоваться вместо `my-pnp-dps`:

```azurecli-interactive
az iot dps create --name my-pnp-dps --resource-group my-pnp-resourcegroup
```

Чтобы связать экземпляр DPS с центром Интернета вещей, используйте следующие команды. Замените `my-pnp-dps` и `my-pnp-hub` уникальными именами, которые вы выбрали ранее:

```azurecli-interactive
hubConnectionString=$(az iot hub connection-string show -n my-pnp-hub --key primary --query connectionString -o tsv)
az iot dps linked-hub create --dps-name my-pnp-dps --resource-group my-pnp-resourcegroup --location centralus --connection-string $hubConnectionString
```

## <a name="retrieve-the-settings"></a>Получение параметров

В некоторых кратких руководствах и учебниках используется строка подключения для центра Интернета вещей. Кроме того, строка подключения требуется при настройке обозревателя Интернета вещей Azure. Получите строку подключения и запишите ее. Замените `my-pnp-hub` уникальным именем своего центра Интернета вещей:

```azurecli-interactive
az iot hub connection-string show -n my-pnp-hub --key primary --query connectionString
```

В большинстве кратких руководств и учебников используется *область идентификаторов* конфигурации DPS. Получите область идентификаторов и запишите ее. Замените `my-pnp-dps` уникальным именем своего экземпляра DPS:

```azurecli-interactive
az iot dps show --name my-pnp-dps --query properties.idScope
```

Во всех кратких руководствах и учебниках используется регистрация устройств DPS. Используйте следующую команду, чтобы создать *регистрацию отдельных устройств* `my-pnp-device` в экземпляре DPS. Замените `my-pnp-dps` уникальным именем своего экземпляра DPS. Запишите значения идентификатора регистрации и первичного ключа для использования в кратких руководствах и учебниках:

```azurecli-interactive
az iot dps enrollment create --attestation-type symmetrickey --dps-name my-pnp-dps --resource-group my-pnp-resourcegroup --enrollment-id my-pnp-device --device-id my-pnp-device --query '{registrationID:registrationId,primaryKey:attestation.symmetricKey.primaryKey}'
```

## <a name="create-environment-variables"></a>Создание переменных среды

Создайте пять переменных среды, чтобы настроить примеры для кратких руководств и учебников. Так вы сможете использовать Службу подготовки устройств (DPS) для подключения к центру Интернета вещей:

* **IOTHUB_DEVICE_SECURITY_TYPE**: значение `DPS`;
* **IOTHUB_DEVICE_DPS_ID_SCOPE**: область идентификатора DPS, которую вы записали ранее;
* **IOTHUB_DEVICE_DPS_DEVICE_ID**: значение `my-pnp-device`;
* **IOTHUB_DEVICE_DPS_DEVICE_KEY**: первичный ключ регистрации, который вы записали ранее;
* **IOTHUB_DEVICE_DPS_ENDPOINT**: значение `global.azure-devices-provisioning.net`.

Примерам службы требуются следующие переменные среды, чтобы определить центр и устройство для подключения:

* **IOTHUB_CONNECTION_STRING**: строка подключения центра Интернета вещей, которую вы записали ранее.
* **IOTHUB_DEVICE_ID**: `my-pnp-device`.

Например, в оболочке командной системы Linux Bash:

```bash
export IOTHUB_DEVICE_SECURITY_TYPE="DPS"
export IOTHUB_DEVICE_DPS_ID_SCOPE="<Your ID scope>"
export IOTHUB_DEVICE_DPS_DEVICE_ID="my-pnp-device"
export IOTHUB_DEVICE_DPS_DEVICE_KEY="<Your enrolment primary key>"
export IOTHUB_DEVICE_DPS_ENDPOINT="global.azure-devices-provisioning.net"
export IOTHUB_CONNECTION_STRING="<Your IoT hub connection string>"
export IOTHUB_DEVICE_ID="my-pnp-device"
```

Например, в командной строке Windows:

```cmd
set IOTHUB_DEVICE_SECURITY_TYPE=DPS
set IOTHUB_DEVICE_DPS_ID_SCOPE=<Your ID scope>
set IOTHUB_DEVICE_DPS_DEVICE_ID=my-pnp-device
set IOTHUB_DEVICE_DPS_DEVICE_KEY=<Your enrolment primary key>
set IOTHUB_DEVICE_DPS_ENDPOINT=global.azure-devices-provisioning.net
set IOTHUB_CONNECTION_STRING=<Your IoT hub connection string>
set IOTHUB_DEVICE_ID=my-pnp-device
```

## <a name="download-the-model-files"></a>Скачивание файлов модели

В кратких руководствах и учебниках используются примеры файлов модели для контроллера температуры и устройств термостата. Чтобы скачать примеры файлов модели, сделайте следующее:

1. Создайте на локальном компьютере папку *models*.

1. Щелкните правой кнопкой мыши [TemperatureController.json](https://raw.githubusercontent.com/Azure/opendigitaltwins-dtdl/master/DTDL/v2/samples/TemperatureController.json) и сохраните этот JSON-файл в папку *models*.

1. Щелкните правой кнопкой мыши [Thermostat.json](https://raw.githubusercontent.com/Azure/opendigitaltwins-dtdl/master/DTDL/v2/samples/Thermostat.json) и сохраните этот JSON-файл в папку *models*.

## <a name="install-the-azure-iot-explorer"></a>Установка обозревателя Azure IoT

В кратких руководствах и учебниках используется **обозреватель Интернета вещей Azure**. Перейдите к [выпускам обозревателя Интернета вещей Azure](https://github.com/Azure/azure-iot-explorer/releases) и разверните список ресурсов для самого последнего выпуска. Скачайте и установите последнюю версию приложения для вашей операционной системы.

При первом запуске средства вам будет предложено ввести строку подключения центра Интернета вещей. Используйте строку подключения, которую вы записали ранее.

Настройте средство для использования файлов модели, которые вы скачали ранее. На домашней странице средства выберите **IoT Plug and Play Settings** (Параметры IoT Plug and Play) и **+ Add > Local folder** (+ Добавить > Локальная папка). Выберите папку *models*, которую вы создали ранее. Затем щелкните **Save** (Сохранить), чтобы сохранить параметры.

Дополнительные сведения см. в статье [Установка и использование обозревателя Интернета вещей Azure](howto-use-iot-explorer.md).

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете использовать центр Интернета вещей и экземпляр DPS для всех кратких руководств и учебников по IoT Plug and Play — вам нужно только выполнить действия, описанные в этой статье. По завершении работы ресурсы можно удалить из подписки с помощью следующей команды:

```azurecli-interactive
az group delete --name my-pnp-resourcegroup
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили среду, вы можете обратиться к одному из кратких руководств или учебников, например:

> [!div class="nextstepaction"]
> [Подключение примера приложения устройства IoT Plug and Play к Центру Интернета вещей](quickstart-connect-device.md)
