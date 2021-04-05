---
title: Учебник. Обновление встроенного ПО устройства через Центр Интернета вещей | Документация Майкрософт
description: Из этого учебника вы узнаете, как реализовать обновление встроенного ПО устройства, чтобы процесс могло запускать серверное приложение, подключенное к центру Интернета вещей.
services: iot-hub
author: wesmc7777
ms.author: wesmc
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 06/28/2019
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-js
- devx-track-azurecli
ms.openlocfilehash: 3fc257ff192ccb1bb05b233c6ac802696ece0054
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99575726"
---
# <a name="tutorial-implement-a-device-firmware-update-process"></a>Руководство по Реализация обновления встроенного ПО устройства

Вам может потребоваться обновление встроенного ПО устройств, подключенных к Центру Интернета вещей. Например, вы можете добавить новые функции во встроенное ПО или применить исправления безопасности. Во многих сценариях Интернета вещей нецелесообразно физически получать доступ, а затем вручную применять обновления встроенного ПО на устройствах. В этом руководстве показано, как можно удаленно запускать и контролировать процесс обновления встроенного ПО через внутреннее приложение, подключенное к центру.

Чтобы создавать и отслеживать процесс обновления встроенного ПО, в рамках этого руководства внутреннее приложение создает _конфигурацию_ в Центре Интернета вещей. Функция [автоматического управление устройствами](./iot-hub-automatic-device-management.md) Центра Интернета вещей использует эту конфигурацию для обновления наборов _требуемых свойств двойников устройств_ на всех устройствах chiller. В требуемых свойствах указаны подробности о требуемом обновлении встроенного ПО. Во время выполнения процесса обновления встроенного ПО на устройствах chiller они сообщают о своем состоянии внутреннему приложению с помощью _сообщаемых свойств двойников устройств_. Внутреннее приложение может использовать конфигурацию для отслеживания сообщаемых свойств, отправленных с устройства, и процесса обновления встроенного ПО:

![Процесс обновления встроенного ПО](media/tutorial-firmware-update/Process.png)

В этом руководстве выполняются следующие задачи:

> [!div class="checklist"]
> * создание Центра Интернета вещей и добавление тестового устройства в реестр удостоверений;
> * создание конфигурации для определения обновлений встроенного ПО;
> * симуляция процесса обновления встроенного ПО на устройстве;
> * получение обновления состояния устройства в процессе обновления встроенного ПО.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

Примеры приложений, запускаемых в рамках этого краткого руководства, написаны на языке Node.js. Вам потребуется установить Node.js 10 x.x или более поздней версии на компьютере для разработки.

Node.js, предназначенный для нескольких платформ, можно скачать здесь: [nodejs.org](https://nodejs.org).

Текущую версию Node.js на компьютере, на котором ведется разработка, можно проверить, используя следующую команду:

```cmd/sh
node --version
```

Скачайте пример проекта Node.js по адресу https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip и извлеките ZIP-архив.

Убедитесь, что в брандмауэре открыт порт 8883. Пример устройства в этом руководстве использует протокол MQTT, который передает данные через порт 8883. В некоторых корпоративных и академических сетях этот порт может быть заблокирован. Дополнительные сведения и способы устранения этой проблемы см. в разделе о [подключении к Центру Интернета вещей по протоколу MQTT](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="set-up-azure-resources"></a>Настройка ресурсов Azure

Для выполнения задач из этого руководства в подписке Azure должен содержаться Центр Интернета вещей с устройством, добавленным в реестр удостоверений устройств. С помощью записи в реестре удостоверений устройств имитированное устройство, которое запускается в рамках этого руководства, подключается к центру.

Если в вашей подписке еще не настроен Центр Интернета вещей, можно сделать это с помощью приведенного ниже скрипта CLI. В этом скрипте для Центра Интернета вещей используется имя **tutorial-iot-hub**. При запуске замените его своим уникальным именем. С помощью скрипта создается группа ресурсов и центр в регионе **Центральная часть США**. Вы можете использовать регион, который находится ближе к вам. Скрипт получает строку подключения для службы Центра Интернета вещей. Эта строка используется в примере внутреннего приложения для подключения к Центру Интернета вещей:

```azurecli-interactive
hubname=tutorial-iot-hub
location=centralus

# Install the IoT extension if it's not already installed
az extension add --name azure-iot

# Create a resource group
az group create --name tutorial-iot-hub-rg --location $location

# Create a free-tier IoT Hub. You can have only one free IoT Hub per subscription. Free tier hubs can have only 2 partitions.
az iot hub create --name $hubname --location $location --resource-group tutorial-iot-hub-rg --partition-count 2 --sku F1

# Make a note of the service connection string, you need it later
az iot hub connection-string show --name $hubname --policy-name service -o table

```

В этом руководстве используется имитированное устройство с именем **MyTwinDevice**. С помощью следующего скрипта это устройство добавляется в реестр удостоверений, задает значение тега и извлекает строку подключения:

```azurecli-interactive
# Set the name of your IoT hub
hubname=tutorial-iot-hub

# Create the device in the identity registry
az iot hub device-identity create --device-id MyFirmwareUpdateDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg

# Add a device type tag
az iot hub device-twin update --device-id MyFirmwareUpdateDevice --hub-name $hubname --set tags='{"device type":"chiller"}'

# Retrieve the device connection string, you need this later
az iot hub device-identity connection-string show --device-id MyFirmwareUpdateDevice --hub-name $hubname --resource-group tutorial-iot-hub-rg -o table

```

> [!TIP]
> При запуске этих команд в командной строке Windows или Powershell ознакомьтесь со страницей [azure-iot-cli-extension tips](https://github.com/Azure/azure-iot-cli-extension/wiki/Tips), чтобы получить сведения о том, как взять в кавычки строки JSON.

## <a name="start-the-firmware-update"></a>Запуск обновления встроенного ПО

Чтобы начать процесс обновления встроенного ПО на всех устройствах chiller с меткой **devicetype**, создайте в серверном приложении конфигурацию [автоматического управления устройствами](iot-hub-automatic-device-management.md#create-a-configuration). Из этого раздела вы узнаете, как выполнять следующие действия:

* Создание конфигурации из внутреннего приложения.
* Мониторинг задания до его завершения.

### <a name="use-desired-properties-to-start-the-firmware-upgrade-from-the-back-end-application"></a>Использование требуемых свойств для запуска обновления встроенного ПО с внутреннего приложения

Чтобы просмотреть код внутреннего приложения, создающего конфигурацию, перейдите к папке **iot-hub/Tutorials/FirmwareUpdate** в скачанном примере проекта Node.js. Затем в текстовом редакторе откройте файл ServiceClient.js.

Внутреннее приложение создает следующую конфигурацию:

[!code-javascript[Automatic device management configuration](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=configuration "Automatic device management configuration")]

Конфигурация содержит следующие разделы:

* `content` указывает требуемые свойства встроенного ПО, отправляемые на выбранные устройства.
* `metrics` указывает выполняемые запросы, сообщающие о состоянии обновления встроенного ПО.
* `targetCondition` выбирает устройства для получения обновления встроенного ПО.
* `priorty` задает приоритет этой конфигурации относительно других конфигураций.

Внутреннее приложение использует следующий код для создания конфигурации, задающей требуемые свойства:

[!code-javascript[Create configuration](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=createConfiguration "Create configuration")]

После создания конфигурации приложение будет отслеживать обновление встроенного ПО:

[!code-javascript[Monitor firmware update](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/ServiceClient.js?name=monitorConfiguration "Monitor firmware update")]

Конфигурация сообщает о двух типах метрик:

* Системные метрики, которые сообщают, сколько устройств являются целевыми и на скольких устройствах было применено обновление.
* Настраиваемые метрики, создаваемые с помощью запросов, добавляемых в конфигурацию. В этом руководстве запросы сообщают о том, сколько устройств находятся на каждом шаге процесса обновления.

### <a name="respond-to-the-firmware-upgrade-request-on-the-device"></a>Ответ на запрос обновления встроенного ПО на устройстве

Чтобы просмотреть код имитируемого устройства, который обрабатывает требуемые свойства встроенного ПО, отправленные из внутреннего приложения, перейдите к папке **iot-hub/Tutorials/FirmwareUpdate** в скачанном примере проекта Node.js. Затем в текстовом редакторе откройте файл SimulatedDevice.js.

Приложение имитированного устройства создает обработчик для обновлений требуемых свойств **properties.desired.firmware** в двойнике устройств. В обработчике производятся некоторые базовые проверки требуемых свойств перед запуском процесса обновления:

[!code-javascript[Handle desired property update](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=initiateUpdate "Handle desired property update")]

## <a name="update-the-firmware"></a>Обновление встроенного ПО

Функция **initiateFirmwareUpdateFlow** выполняет обновление. Эта функция использует функцию **waterfall** для последовательного выполнения этапов процесса обновления. В этом примере обновление встроенного ПО состоит из четырех этапов. На первом этапе скачивается образ, на втором этапе происходит его проверка с помощью контрольной суммы, на третьем этапе образ применяется, а на последнем выполняется перезагрузка устройства:

[!code-javascript[Firmware update flow](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=firmwareupdateflow "Firmware update flow")]

Во время процесса обновления имитируемое устройство сообщает о ходе выполнения с помощью сообщаемых свойств:

[!code-javascript[Reported properties](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=reportedProperties "Reported properties")]

В следующем фрагменте кода показана реализация этапа скачивания. Во время этапа скачивания имитируемое устройство использует переданные свойства для отправки информации о состоянии во внутреннее приложение:

[!code-javascript[Download image phase](~/iot-samples-node/iot-hub/Tutorials/FirmwareUpdate/SimulatedDevice.js?name=downloadimagephase "Download image phase")]

## <a name="run-the-sample"></a>Запуск примера

В этом разделе вы запускаете два примера приложений для наблюдения за тем, как внутреннее приложение создает конфигурацию для управления процессом обновления встроенного ПО на имитируемом устройстве.

Чтобы запустить имитированное устройство и внутренние приложения, нужны строки подключения для устройства и службы. Вы записали строки подключения при создании ресурсов в начале работы с этим руководством.

Чтобы запустить приложение имитированного устройства, откройте окно оболочки или командной строки и перейдите к папке **iot-hub/Tutorials/FirmwareUpdate** в скачанном проекте Node.js. Затем выполните следующие команды:

```cmd/sh
npm install
node SimulatedDevice.js "{your device connection string}"
```

Чтобы запустить внутреннее приложение, откройте еще одно окно оболочки или командной строки. Затем перейдите к папке **iot-hub/Tutorials/FirmwareUpdate** в скачанном проекте Node.js. Затем выполните следующие команды:

```cmd/sh
npm install
node ServiceClient.js "{your service connection string}"
```

На следующем снимке экрана отображаются выходные данные приложения имитированного устройства и показано, как оно реагирует на обновление требуемых свойств встроенного ПО из встроенного приложения:

![Виртуальное устройство](./media/tutorial-firmware-update/SimulatedDevice.png)

На следующем снимке экрана отображаются выходные данные внутреннего приложения и показано, как создается конфигурация для обновления требуемых свойств встроенного ПО:

![Снимок экрана: выходные данные внутреннего приложения.](./media/tutorial-firmware-update/BackEnd1.png)

На следующем снимке экрана отображаются выходные данные внутреннего приложения и показано, как отслеживаются метрики обновления встроенного ПО с имитированного устройства:

![Внутреннее приложение](./media/tutorial-firmware-update/BackEnd2.png)

Так как автоматические конфигурации устройств запускаются во время создания и затем каждые пять минут, каждое обновление состояния, передаваемое серверному приложению, может не отображаться. Метрики также можно просмотреть на портале в разделе **Automatic device management (Автоматическое управление устройствами) -> IoT device configuration (Конфигурация устройства Интернета вещей)** Центра Интернета вещей:

![Просмотр конфигурации на портале](./media/tutorial-firmware-update/portalview.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Не удаляйте группу ресурсов и Центр Интернета вещей, так как они понадобятся вам при работе со следующим руководством.

Если вам больше не требуется Центр Интернета вещей, удалите его и группу ресурсов на портале. Для этого выберите группу ресурсов **tutorial-iot-hub-rg**, содержащую Центр Интернета вещей, и щелкните **Удалить**.

Или используйте CLI:

```azurecli-interactive
# Delete your resource group and its contents
az group delete --name tutorial-iot-hub-rg
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как реализовать процесс обновления встроенного ПО для подключенных устройств. Перейдите к следующему руководству, чтобы узнать, как применять инструменты на портале Центра Интернета вещей Azure и команды Azure CLI для проверки возможности подключения устройств.

> [!div class="nextstepaction"]
> [Проверка подключения к центру Интернета вещей с помощью имитированного устройства](tutorial-connectivity.md)
