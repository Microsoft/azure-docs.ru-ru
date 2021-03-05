---
title: Создание и развертывание моста самонастраивающийся IoT | Документация Майкрософт
description: Найдите компоненты моста самонастраивающийся IoT. Узнайте, как запускать его на устройствах Интернета вещей, шлюзах и в качестве модуля IoT Edge.
author: usivagna
ms.author: ugans
ms.date: 1/20/2021
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: c89427f83600d3b8091d5293b1757fa6f1a15ef1
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102202490"
---
# <a name="build-and-deploy-the-iot-plug-and-play-bridge"></a>Создание и развертывание моста самонастраивающийся IoT

[Мост Интернета вещей Самонастраивающийся](concepts-iot-pnp-bridge.md#iot-plug-and-play-bridge-architecture) позволяет подключать существующие устройства, подключенные к шлюзу, к центру Интернета вещей. С помощью моста можно сопоставлять интерфейсы Интернета вещей самонастраивающийся с подключенными устройствами. Интерфейс Интернета вещей самонастраивающийся определяет данные телеметрии, которые отправляет устройство, свойства, синхронизированные между устройством и облаком, а также команды, на которые отвечает устройство. Вы можете установить и настроить приложение моста с открытым исходным кодом в шлюзах Windows или Linux. Кроме того, мост можно запустить как модуль среды выполнения Azure IoT Edge.

В этой статье подробно описывается, как:

- Настройка моста.
- Создание и запуск моста в различных средах.

Простой пример, демонстрирующий использование моста, см. в статье [Подключение примера моста Самонастраивающийся для Интернета вещей, работающего в Linux или Windows, в центр Интернета вещей](howto-use-iot-pnp-bridge.md).

Руководства и примеры в этой статье предполагают знание [Azure Digital двойников](../digital-twins/overview.md) и [IOT Самонастраивающийся](overview-iot-plug-and-play.md).

## <a name="general-prerequisites"></a>Основные обязательные требования

### <a name="supported-os-platforms"></a>Поддерживаемые платформы ОС

Поддерживаются следующие платформы и версии ОС:

|Платформа  |Поддерживаемые версии  |
|---------|---------|
|быть под управлением ОС Windows 10; |     Поддерживаются все номера SKU Windows. Например: IoT Корпоративная, сервер, Настольный компьютер, IoT Core. *Для функций мониторинга работоспособности камеры рекомендуется использовать 20H1 или более позднюю сборку. Все остальные функции доступны во всех сборках Windows 10.*  |
|Linux     |Протестированные и поддерживаемые в Ubuntu 18,04, но не тестировали функциональность других дистрибутивов.         |
||

### <a name="hardware"></a>Оборудование

- Любая аппаратная платформа, поддерживающая указанные выше номера SKU и версии ОС.
- Устройства с последовательными портами, USB, Bluetooth, камеры и датчики поддерживаются изначально. Мост Интернета вещей самонастраивающийся можно расширить для поддержки любого настраиваемого периферийного устройства или датчика.

### <a name="azure-iot-products-and-tools"></a>Продукты и инструменты Azure IoT

- **Центр Интернета вещей Azure** — для подключения устройства к требуется [центр Интернета вещей Azure](../iot-hub/index.yml) в подписке Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу. Если у вас нет центра Интернета вещей, [следуйте этим инструкциям, чтобы создать его](../iot-hub/iot-hub-create-using-cli.md). Поддержка самонастраивающийся IoT не включена в центры Интернета вещей уровня Basic.
- **Azure IOT Explorer** . для взаимодействия с вашим устройством Самонастраивающийся IOT можно использовать средство Azure IOT Explorer. [Скачайте и установите последний выпуск обозревателя Интернета вещей Azure](./howto-use-iot-explorer.md) для вашей операционной системы. Настройте средство Azure IoT Explorer для подключения к центру Интернета вещей и найдите определения моделей в папке *пнпбридже\докс\счемас* в клонированном центре **Интернета вещей-Plug-and-Play-Bridge** .

## <a name="configure-a-bridge"></a>Настройка моста

Существует два способа настройки моста.

- Если мост работает изначально на устройстве Интернета вещей или в шлюзе, используйте файл конфигурации. В этом сценарии может использоваться компьютер под управлением Linux или Windows.
- Если мост выполняется как модуль IoT Edge в среде выполнения IoT Edge, используйте требуемое свойство модуля двойника. В этом сценарии предполагается, что среда выполнения IoT Edge размещается в среде Linux.

### <a name="configuration-file"></a>Файл конфигурации

Если самонастраивающийся Bridge работает на устройстве или в шлюзе Интернета вещей, он использует файл конфигурации для выполнения следующих действий.

- Получение сведений о подключении IoT Central или центра Интернета вещей.
- Настройте устройства, для которых публикуются интерфейсы Интернета вещей самонастраивающийся.

Для передачи файла конфигурации в мост используйте один из следующих параметров:

- Передайте путь к файлу конфигурации в качестве параметра командной строки для исполняемого файла моста.
- Используйте существующий файл *config.js* в папке *пнпбридже\кмаке\ pnpbridge_x86 \срк\пнпбридже\самплес\консоле* .

[Схема файла конфигурации](https://github.com/Azure/iot-plug-and-play-bridge/blob/master/pnpbridge/src/pnpbridge/src/pnpbridge_config_schema.json) доступна в репозитории GitHub.

> [!TIP]
> При изменении файла конфигурации моста в VS Code можно использовать этот файл схемы для проверки файла.

### <a name="iot-edge-module-configuration"></a>Конфигурация модуля IoT Edge

Когда мост запускается как модуль IoT Edge в среде выполнения IoT Edge, файл конфигурации отправляется из облака в качестве обновления `PnpBridgeConfig` требуемого свойства. Мост ожидает обновления этого свойства перед настройкой адаптеров и компонентов.

## <a name="build-and-run-the-bridge-on-an-iot-device-or-gateway"></a>Создание и запуск моста на устройстве или шлюзе Интернета вещей

| Платформа | Поддерживается |
| :-----------: | :-----------: |
| Windows |  Да |
| Linux | Да |

### <a name="prerequisites"></a>Предварительные условия

Для выполнения этого раздела необходимо установить на локальном компьютере следующее программное обеспечение:

- Среда разработки, поддерживающая компиляцию C++, например [Visual Studio (Community, Professional или Enterprise)](https://visualstudio.microsoft.com/downloads/) . при установке Visual Studio необходимо включить компонент диспетчера пакетов NuGet и рабочую нагрузку " **Разработка классических приложений на C++** ".
- [CMAK](https://cmake.org/download/) — при установке CMAK выберите параметр **Добавить CMAK в системный путь**.
- Если вы собираетесь на Windows, вам также потребуется загрузить [пакет SDK для windows 17763](https://developer.microsoft.com/windows/downloads/windows-10-sdk).

### <a name="source-code"></a>Исходный код

Клонировать репозиторий [моста Самонастраивающийся IOT](https://github.com/Azure/iot-plug-and-play-bridge) на локальный компьютер:

```console
git clone https://github.com/Azure/iot-plug-and-play-bridge.git

cd iot-plug-and-play-bridge

git submodule update --init --recursive
```

Ожидание выполнения предыдущей команды займет несколько минут.

> [!NOTE]
> Если проблемы возникают в случае сбоя в `git submodule update` Windows, попробуйте выполнить следующую команду, чтобы устранить проблему: `git config --system core.longpaths true` .

### <a name="build-the-bridge-on-windows"></a>Создание моста в Windows

Откройте **Командная строка РАЗРАБОТЧИКА VS 2019** и перейдите в папку, содержащую клонированный репозиторий, и выполните следующие команды:

```console
cd pnpbridge\scripts\windows

build.cmd
```

Ожидание выполнения предыдущей команды займет несколько минут.

При необходимости можно открыть созданный файл решения *пнпбридже\кмаке\ pnpbridge_x86 \ azure_iot_pnp_bridge. sln* в Visual Studio для редактирования, перестроения и отладки кода.

> [!TIP]
> Этот проект использует CMAK для создания файлов проекта. Любые изменения, вносимые в проект в Visual Studio, могут быть потеряны, если соответствующие файлы CMAK не обновляются.

### <a name="build-the-bridge-on-linux"></a>Создание моста в Linux

С помощью оболочки bash перейдите в папку, содержащую клонированный репозиторий, и выполните следующие команды:

```bash
cd scripts/linux

./setup.sh

./build.sh
```

### <a name="configure-the-generic-sensors"></a>Настройка универсальных датчиков

Измените следующие параметры в `pnp_bridge_connection_parameters` узле *config.jsдля* файла в папке *ИОТ-плуг-Анд-Плай-бридже\пнпбридже\кмаке\ pnpbridge_x86 \срк\пнпбридже\самплес\консоле* в Windows или в папке *IOT-Plug-and-Play-bridge/пнпбридже/CMAK/pnpbridge_linux \ src/pnpbridge/Samples/Console* в Windows:

Если вы используете строку подключения для подключения к центру Интернета вещей:

```JSON
{
  "connection_parameters": {
    "connection_type" : "connection_string",
    "connection_string" : "dtmi:com:example:RootPnpBridgeSampleDevice;1",
    "root_interface_model_id": "[To fill in]",
    "auth_parameters": {
      "auth_type": "symmetric_key",
      "symmetric_key": "[To fill in]"
    }
  }
}
```

> [!TIP]
> `symmetric_key`Значение должно соответствовать ключу SAS в строке подключения.

Если вы используете службу DPS для подключения к центру Интернета вещей или IoT Central приложении:

```JSON
{
  "connection_parameters": {
    "connection_type" : "dps",
    "root_interface_model_id": "dtmi:com:example:RootPnpBridgeSampleDevice;1",
    "auth_parameters" : {
      "auth_type" : "symmetric_key",
      "symmetric_key" : "[DEVICE KEY]"
    },
    "dps_parameters" : {
      "global_prov_uri" : "[GLOBAL PROVISIONING URI] - typically it is global.azure-devices-provisioning.net",
      "id_scope": "[IoT Central / IoT Hub ID SCOPE]",
      "device_id": "[DEVICE ID]"
    }
  }
}
```

Просмотрите оставшуюся часть файла конфигурации, чтобы узнать, какие компоненты интерфейса и глобальные параметры настроены в этом образце.

### <a name="start-the-bridge-in-windows"></a>Запуск моста в Windows

Запустите мост, запустив его в командной строке:

```console
cd iot-plug-and-play-bridge\pnpbridge\cmake\pnpbridge_x86\src\pnpbridge\samples\console

Debug\pnpbridge_bin.exe
```

> [!TIP]
> Чтобы использовать файл конфигурации из другого расположения, передайте его путь в качестве параметра командной строки при запуске моста.
  
> [!TIP]
> При наличии встроенной камеры или USB-камеры, подключенной к компьютеру, можно запустить приложение, использующее камеру, например встроенное приложение для **камеры** . Когда приложение **камеры** работает, в выходных данных консоли Bridge отображается статистика мониторинга, а частота кадров — через интерфейс Digital двойника в центр Интернета вещей.

## <a name="build-and-run-the-bridge-as-an-iot-edge-module"></a>Создание и запуск моста как модуля IoT Edge

| Платформа | Поддерживается |
| :-----------: | :-----------: |
| Windows |  Нет |
| Linux | Да |

### <a name="prerequisites"></a>Предварительные условия

Для работы с этим разделом вам понадобится центр Интернета вещей Azure уровня "бесплатный" или "Стандартный". Сведения о том, как создать центр Интернета вещей, см. в статье [Создание центра Интернета](../iot-hub/iot-hub-create-through-portal.md)вещей.

В действиях, описанных в этом разделе, предполагается, что на компьютере с Windows 10 установлена следующая среда разработки. Эти средства позволяют создавать и развертывать модуль IoT Edge на устройстве IoT Edge.

- Подсистема Windows для Linux (WSL) 2 под управлением Ubuntu 18,04 LTS. Дополнительные сведения см. в статье [подсистема Windows для Linux](/windows/wsl/install-win10), посвященная установке Windows 10.
- DOCKER Desktop для Windows, настроенный для использования WSL 2. Дополнительные сведения см. в разделе [сервер DOCKER Desktop WSL 2](https://docs.docker.com/docker-for-windows/wsl/).
- [Visual Studio Code установлен в среде Windows](https://code.visualstudio.com/docs/setup/windows) с установленными следующими тремя расширениями:

  - [Пакет расширений удаленной разработки](https://aka.ms/vscode-remote/download/extension) — это расширение позволяет создавать и запускать код в WSL 2.
  - [DOCKER для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) — это расширение должно быть включено в среде VS Code **WSL: Ubuntu-18,04** .
  - [Средства Azure IOT для Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) — это расширение должно быть включено в среде VS Code **WSL: Ubuntu-18,04** .

- Выполните следующие команды в среде WSL 2, чтобы установить необходимые средства сборки:

  ```bash
  sudo apt-get update
  sudo apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
  ```

- [Azure CLI](/cli/azure/install-azure-cli-apt) , установленный в среде WSL 2 для управления ресурсами Azure.

  > [!TIP]
  > При желании можно выполнить `az` команды в [Azure Cloud Shell](https://shell.azure.com/) , где предварительно установлен интерфейс командной строки.

### <a name="create-an-iot-edge-device"></a>Создание устройства IoT Edge

Эти команды создают IoT Edge устройство, работающее на виртуальной машине Azure. Запуск устройства IoT Edge в виртуальной машине полезно для тестирования развертывания, если у вас нет доступа к реальному устройству.

Чтобы создать IoT Edge регистрации устройств в центре Интернета вещей, выполните следующие команды в среде WSL 2. `az login`Для входа в подписку Azure используйте команду:

```azurecli
az iot hub device-identity create --device-id bridge-edge-device --edge-enabled true --hub-name {your IoT hub name}
```

Чтобы создать виртуальную машину Azure с установленной средой выполнения IoT Edge, выполните следующие команды. Обновите заполнители с подходящими значениями:

```azurecli
az group create --name bridge-edge-resources --location eastus
az deployment group create \
--resource-group bridge-edge-resources \
--template-uri "https://aka.ms/iotedge-vm-deploy" \
--parameters dnsLabelPrefix='{unique DNS name for virtual machine}' \
--parameters adminUsername='{admin user name}' \
--parameters deviceConnectionString=$(az iot hub device-identity connection-string show --device-id bridge-edge-device --hub-name {your IoT hub name} -o tsv) \
--parameters authenticationType='password' \
--parameters adminPasswordOrKey="{admin password for virtual machine}"
```

Теперь у вас есть среда выполнения IoT Edge, выполняющаяся на виртуальной машине. Чтобы убедиться, что на устройстве работают **$edgeAgent** и **$edgeHub** , можно использовать следующую команду:

```azurecli
az iot hub module-identity list --device-id bridge-edge-device -o table --hub-name {your IoT hub name}
```

> [!CAUTION]
> При выполнении этой виртуальной машины выставляется счет. Не забудьте завершить его работу, если вы не используете его, и удалить его после завершения работы.

### <a name="download-the-source-code"></a>Скачивание исходного кода

В следующих шагах создается образ DOCKER в среде WSL 2. Чтобы клонировать репозиторий **IOT-Plug-and-Play-Bridge** , выполните следующие команды в оболочке bash WSL 2 в подходящей папке:

```bash
git clone https://github.com/Azure/iot-plug-and-play-bridge.git

cd iot-plug-and-play-bridge

git submodule update --init --recursive
```

Ожидание выполнения предыдущей команды займет несколько минут.

### <a name="update-the-dockerfile"></a>Обновление *Dockerfile*

Запустите VS Code, откройте палитру команд, введите *Remote WSL: Open Folder в WSL*, а затем откройте папку *IOT-Plug-and-Play-Bridge* , содержащую клонированный репозиторий.

Откройте файл *pnpbridge\Dockerfile.AMD64* . Измените определения переменных среды следующим образом:

```dockerfile
ENV PNP_BRIDGE_ROOT_MODEL_ID="dtmi:com:example:RootPnpBridgeSampleDevice;1"
ENV PNP_BRIDGE_HUB_TRACING_ENABLED="false"
ENV IOTEDGE_WORKLOADURI="something"
```

> [!TIP]
> Чтобы получить строку подключения устройства, можно использовать следующую команду: `az iot hub device-identity connection-string show --device-id bridge-edge-device --hub-name {your IoT hub name}`

Существуют примеры *файлы dockerfile* для других архитектур.

### <a name="build-the-iot-plug-and-play-bridge-module-image"></a>Создание образа модуля моста Интернета вещей самонастраивающийся

В VS Code откройте палитру команд, введите реестры *docker: Войдите в DOCKER CLI*, а затем выберите подписку Azure и реестр контейнеров. Эта команда позволяет DOCKER подключиться к реестру контейнеров и отправить образ модуля.

Откройте файл *пнпбридже/module.jsв* файле. Добавьте в `{your container registry name}.azurecr.io/iotpnpbridge` качестве репозитория образов контейнеров и в `v1` качестве версии.

В VS Code щелкните правой кнопкой мыши файл *пнпбридже/module.js* в представлении **проводника** , выберите **Сборка и отправка IOT Edge образ модуля**, а затем выберите **AMD64** в качестве платформы.

В VS Code в представлении **DOCKER** можно просмотреть содержимое реестра контейнеров, которое теперь включает образ модуля **иотпнпбридже: v1-AMD64** .

### <a name="create-a-container-registry"></a>Создание реестра контейнеров

Устройство IoT Edge скачивает свои образы модулей из реестра контейнеров. В этом примере используется реестр контейнеров Azure.

Создайте реестр контейнеров Azure в группе ресурсов **Bridge-ребра-Resources** . Затем включите административный доступ к реестру контейнеров и получите учетные данные, необходимые устройству IoT Edge для загрузки образов модулей:

```azurecli
az acr create -g bridge-edge-resources --sku Basic -n {your container registry name}
az acr update --admin-enabled true -n {your container registry name}
az acr credential show -n {your container registry name}
```

### <a name="create-the-deployment-manifest"></a>Создание манифеста развертывания

Манифест развертывания IoT Edge указывает модули и значения конфигурации для развертывания на устройстве IoT Edge.

В VS Code откройте файл *пнпбридже/deployment.template.jsв* файле:

- Обновите `registryCredentials` со значениями из предыдущей команды. `address`Значение выглядит так `{your container registry name}.azurecr.io` :.
- Обновите `image` значение для `ModulePnpBridge` . Образ модуля выглядит следующим образом: `{your container registry}.azurecr.io/iotpnpbridge:v1-amd64` .
- Замените на `PnPBridgeConfig` следующий JSON, чтобы настроить адаптер обнаружения CO2:

  ```json
  "PnpBridgeConfig": {
    "pnp_bridge_interface_components": [
      {
        "_comment": "Component 1 - Modbus Device",
        "pnp_bridge_component_name": "Co2Detector1",
        "pnp_bridge_adapter_id": "modbus-pnp-interface",
        "pnp_bridge_adapter_config": {
          "unit_id": 1,
          "tcp": {
            "host": "10.159.29.2",
            "port": 502
          },
          "modbus_identity": "DL679"
        }
      }
    ],
    "pnp_bridge_adapter_global_configs": {
      "modbus-pnp-interface": {
        "DL679": {
          "telemetry": {
            "co2": {
              "startAddress": "40001",
              "length": 1,
              "dataType": "integer",
              "defaultFrequency": 5000,
              "conversionCoefficient": 1
            },
            "temperature": {
              "startAddress": "40003",
              "length": 1,
              "dataType": "decimal",
              "defaultFrequency": 5000,
              "conversionCoefficient": 0.01
            }
          },
          "properties": {
            "firmwareVersion": {
              "startAddress": "40482",
              "length": 1,
              "dataType": "hexstring",
              "defaultFrequency": 60000,
              "conversionCoefficient": 1,
              "access": 1
            },
            "modelName": {
              "startAddress": "40483",
              "length": 2,
              "dataType": "string",
              "defaultFrequency": 60000,
              "conversionCoefficient": 1,
              "access": 1
            },
            "alarmStatus_co2": {
              "startAddress": "00305",
              "length": 1,
              "dataType": "boolean",
              "defaultFrequency": 1000,
              "conversionCoefficient": 1,
              "access": 1
            },
            "alarmThreshold_co2": {
              "startAddress": "40225",
              "length": 1,
              "dataType": "integer",
              "defaultFrequency": 30000,
              "conversionCoefficient": 1,
              "access": 2
            }
          },
          "commands": {
            "clearAlarm_co2": {
              "startAddress": "00305",
              "length": 1,
              "dataType": "flag",
              "conversionCoefficient": 1
            }
          }
        }
      }
    }
  }
  ```

  Сохраните изменения.

В VS Code щелкните правой кнопкой мыши файл *пнпбридже/deployment.template.js* в представлении **проводника** и выберите **создать IOT Edge манифест развертывания**. Эта команда создает *пнпбридже/config/deployment.amd64.jsв* манифесте развертывания.

### <a name="deploy-the-bridge-to-your-iot-edge-device"></a>Развертывание моста на устройстве IoT Edge

В VS Code откройте палитру команд, введите *центр Интернета вещей Azure: выберите центр Интернета вещей*, а затем выберите подписку и центр Интернета вещей.

В VS Code щелкните правой кнопкой мыши файл *пнпбридже/config/deployment.amd64.js* в представлении **проводника** , выберите **Создать развертывание для одного устройства** и выберите **Bridge-ребр-устройство**.

Чтобы просмотреть состояние модулей на устройстве, выполните следующую команду:

```azurecli
az iot hub module-identity list --device-id bridge-edge-device -o table --hub-name {your IoT hub name}
```

Список запущенных модулей теперь включает модуль **модулепнпбридже** , настроенный для использования адаптера обнаружения CO2.

### <a name="tidy-up"></a>Удаление ненужных элементов

Чтобы удалить виртуальную машину и реестр контейнеров из подписки Azure, выполните следующую команду:

```azurecli
az group delete -n bridge-edge-resources
```

## <a name="folder-structure"></a>Структура папок

*пнпбридже\депс\азуре-ИОТ-СДК-к-ПНП*: подмодули Git, содержащие пакет SDK для устройств Azure IOT для c SDK.

*пнпбридже\скриптс*: создание скриптов.

*пнпбридже\срк*: исходный код для моста IOT Самонастраивающийся.

*пнпбридже\срк\адаптерс*: исходный код для различных адаптеров моста интернета вещей Самонастраивающийся.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Bridge самонастраивающийся центра Интернета вещей см. в репозитории GitHub [Самонастраивающийся Bridge](https://github.com/Azure/iot-plug-and-play-bridge) .