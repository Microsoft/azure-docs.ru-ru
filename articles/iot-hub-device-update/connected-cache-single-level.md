---
title: Примеры сценариев развертывания Подключенного кэша Майкрософт (предварительная версия) | Документация Майкрософт
titleSuffix: Device Update for Azure IoT Hub
description: Учебники с примерами сценариев развертывания Подключенного кэша Майкрософт (предварительная версия).
author: andyriv
ms.author: andyriv
ms.date: 2/16/2021
ms.topic: tutorial
ms.service: iot-hub-device-update
ms.openlocfilehash: ae07926d7d8c768170e945e916367bee41999571
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101658877"
---
# <a name="microsoft-connected-cache-preview-deployment-scenario-samples"></a>Примеры сценариев развертывания Подключенного кэша Майкрософт (предварительная версия)

## <a name="single-level-azure-iot-edge-gateway-no-proxy"></a>Одноуровневый шлюз Azure IoT Edge без прокси-сервера

На приведенной ниже схеме показан сценарий, в котором используется шлюз Azure IoT Edge с прямым доступом к ресурсам CDN, а также конечное устройство Интернета вещей Azure, например Raspberry PI, которое является изолированным от Интернета дочерним устройством для шлюза Azure IoT Edge. 

  :::image type="content" source="media/connected-cache-overview/disconnected-device-update.png" alt-text="Обновление отключенного устройства Подключенного кэша Майкрософт" lightbox="media/connected-cache-overview/disconnected-device-update.png":::

1. Добавьте модуль Подключенного кэша Майкрософт в развертывание устройства шлюза Azure IoT Edge в Центре Интернета вещей Azure (дополнительные сведения о том, как получить этот модуль: `MCC concepts`).
2. Добавьте переменные среды для развертывания. Ниже приведен пример переменных среды.

    **Переменные среды**
    
    | Имя                 | Значение                                       |
    | ----------------------------- | --------------------------------------------| 
    | CACHE_NODE_ID                 | Ознакомьтесь с описанием переменных среды выше. |
    | CUSTOMER_ID                   | Ознакомьтесь с описанием переменных среды выше. |
    | CUSTOMER_KEY                  | Ознакомьтесь с описанием переменных среды выше. |
    | STORAGE_ *N* _SIZE_GB           | N = 5                                       |

3. Добавьте параметры создания контейнера для развертывания. Ниже приведен пример параметров создания контейнера.

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

Чтобы проверить, правильно ли функционирует Подключенный кэш Майкрософт, выполните приведенную ниже команду в терминале на устройстве IoT Edge, в котором размещен модуль, или на любом устройстве в сети.

```bash
    wget "http://<IOT Edge Gateway IP>/mscomtest/wuidt.gif?cacheHostOrigin=au.download.windowsupdate.com
```

## <a name="single-level-azure-iot-edge-gateway-with-outbound-unauthenticated-proxy"></a>Одноуровневый шлюз Azure IoT Edge с исходящим прокси-сервером без аутентификации

В этом сценарии используется шлюз Azure IoT Edge с доступом к ресурсам CDN через исходящий прокси-сервер без аутентификации. Подключенный кэш Майкрософт настроен для кэширования содержимого из пользовательского репозитория, а сводный отчет доступен для просмотра всем пользователям в сети. Ниже приведен пример переменных среды MCC, которые задаются.

  :::image type="content" source="media/connected-cache-overview/single-level-proxy.png" alt-text="Одноуровневый прокси-сервер Подключенного кэша Майкрософт" lightbox="media/connected-cache-overview/single-level-proxy.png":::

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

```json
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

Чтобы проверить, правильно ли функционирует Подключенный кэш Майкрософт, выполните приведенную ниже команду в терминале на устройстве Azure IoT Edge, в котором размещен модуль, или на любом устройстве в сети.

```bash
    wget "http://<Azure IOT Edge Gateway IP>/mscomtest/wuidt.gif?cacheHostOrigin=au.download.windowsupdate.com
```
