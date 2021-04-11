---
title: 'Подключенный кэш Майкрософт: шлюз Azure IoT Edge с двумя уровнями вложенности с исходящим непроверяемым прокси-сервером | Документация Майкрософт'
titleSuffix: Device Update for Azure IoT Hub
description: 'Руководство "Подключенный кэш Майкрософт: шлюз Azure IoT Edge с двумя уровнями вложенности с исходящим непроверяемым прокси-сервером"'
author: andyriv
ms.author: andyriv
ms.date: 2/16/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: 7facb74cd407c576b2a7b119f19427dcd185f04e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105568823"
---
# <a name="microsoft-connected-cache-preview-deployment-scenario-sample-two-level-nested-azure-iot-edge-gateway-with-outbound-unauthenticated-proxy"></a>Пример сценария развертывания для Подключенного кэша Майкрософт (предварительная версия): шлюз Azure IoT Edge с двумя уровнями вложенности с исходящим непроверяемым прокси-сервером

В соответствии с приведенной ниже схемой в этом сценарии имеются шлюз Azure IoT Edge и нижестоящее устройство Azure IoT Edge, один шлюз Azure IoT Edge является родительским по отношению к другому шлюзу Azure IoT Edge, а также есть прокси-сервер в демилитаризованной зоне ИТ. Ниже приведен пример переменных среды Подключенного кэша Майкрософт (MCC), которые следует задать в пользовательском интерфейсе портала Azure для обоих модулей MCC, развернутых на шлюзах Azure IoT Edge. В этом примере демонстрируется конфигурация с двумя уровнями шлюзов Azure IoT Edge, но Подключенный кэш Майкрософт поддерживает любую глубину вложенности вышестоящих узлов. Параметры создания контейнера MCC такие же, как в предыдущих примерах.

Дополнительные сведения о настройке многоуровневых развертываний шлюзов Azure IoT Edge см. в документации [Подключение нижестоящих устройств IOT Edge — Azure IOT Edge](../iot-edge/how-to-connect-downstream-iot-edge-device.md?preserve-view=true&tabs=azure-portal&view=iotedge-2020-11). Кроме того, обратите внимание: при развертывании Azure IoT Edge, Подключенного кэша Майкрософт и пользовательских модулей все модули должны находиться в одном реестре контейнеров.

На схеме ниже показан следующий сценарий: один шлюз Azure IoT Edge с прямым доступом к ресурсам CDN является родительским для другого шлюза Azure IoT Edge, который выступает в качестве родителя для конечного устройства Azure IoT, например Raspberry Pi. Возможность подключения к ресурсам CDN в Интернете есть только у родительского шлюза Azure IoT Edge, а дочерний шлюз Azure IoT Edge и устройство Azure IoT изолированы от Интернета. 

  :::image type="content" source="media/connected-cache-overview/nested-level-proxy.png" alt-text="Подключенный кэш Майкрософт с уровнями вложенности" lightbox="media/connected-cache-overview/nested-level-proxy.png":::

## <a name="parent-gateway-configuration"></a>Настройка родительского шлюза

1. Добавьте модуль Подключенного кэша Майкрософт в развертывание устройства шлюза Azure IoT Edge в Центре Интернета вещей Azure.
2. Добавьте переменные среды для развертывания. Ниже приведен пример переменных среды.

    **Переменные среды**

    | Имя                 | Значение                                       |
    | ----------------------------- | --------------------------------------------| 
    | CACHE_NODE_ID                 | Ознакомьтесь с описанием переменных среды выше. |
    | CUSTOMER_ID                   | Ознакомьтесь с описанием переменных среды выше. |
    | CUSTOMER_KEY                  | Ознакомьтесь с описанием переменных среды выше. |
    | STORAGE_ *N* _SIZE_GB           | N = 5                                       |
    | CACHEABLE_CUSTOM_1_HOST       | Packagerepo.com:80                          |
    | CACHEABLE_CUSTOM_1_CANONICAL  | Packagerepo.com                             |
    | IS_SUMMARY_ACCESS_UNRESTRICTED| true                                        |
    | UPSTREAM_PROXY                | IP-адрес или полное доменное имя прокси-сервера                     |

3. Добавьте параметры создания контейнера для развертывания. Параметры создания контейнера MCC такие же, как в предыдущем примере. Ниже приведен пример параметров создания контейнера.

### <a name="container-create-options"></a>Параметры создания контейнера

```markdown
{
    "HostConfig": {
        "Binds": [
            "/MicrosoftConnectedCache1/:/nginx/cache1/"
        ],
        "PortBindings": {
            "8081/tcp": [
                {
                    "HostPort": "80"
                }
            ],
            "5000/tcp": [
                {
                    "HostPort": "5100"
                }
            ]
        }
    }
```

## <a name="child-gateway-configuration"></a>Настройка дочернего шлюза

>[!Note]
>Если в вашем собственном частном реестре есть реплицированные контейнеры, используемые в конфигурации, потребуется внести изменения в параметры config.toml и параметры среды выполнения в развертывании модуля. Дополнительные сведения см. в руководстве [Создание иерархии устройств IoT Edge — Azure IoT Edge](../iot-edge/tutorial-nested-iot-edge.md?preserve-view=true&tabs=azure-portal&view=iotedge-2020-11#deploy-modules-to-the-lower-layer-device).

1. Измените путь к образу для агента Edge, как показано в этом примере:

```markdown
[agent]
name = "edgeAgent"
type = "docker"
env = {}
[agent.config]
image = "<parent_device_fqdn_or_ip>:8000/iotedge/azureiotedge-agent:1.2.0-rc2"
auth = {}
```
2. Измените параметры среды выполнения для центра Edge и агента Edge в развертывании Azure IoT Edge, как показано в этом примере:
    
    * В разделе "Центр Edge" в поле образа введите ```$upstream:8000/iotedge/azureiotedge-hub:1.2.0-rc2```.
    * В разделе "Агент Edge" в поле образа введите ```$upstream:8000/iotedge/azureiotedge-agent:1.2.0-rc2```.

3. Добавьте модуль Подключенного кэша Майкрософт в развертывание устройства шлюза Azure IoT Edge в Центре Интернета вещей Azure.

   * Присвойте модулю имя: ```ConnectedCache```.
   * Измените URI образа: ```$upstream:8000/mcc/linux/iot/mcc-ubuntu-iot-amd64:latest```.

4. Добавьте те же переменные среды и параметры создания контейнера, которые используются в развертывании родительского шлюза.

Чтобы проверить, правильно ли функционирует Подключенный кэш Майкрософт, выполните приведенную ниже команду в терминале устройства IoT Edge, в котором размещен модуль, или любого устройства в сети.

```bash
    wget "http://<CHILD Azure IoT Edge Gateway IP>/mscomtest/wuidt.gif?cacheHostOrigin=au.download.windowsupdate.com
```