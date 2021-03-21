---
title: Общие сведения об обновлении устройства для агента Центра Интернета вещей Azure | Документация Майкрософт
description: Общие сведения об обновлении устройства для агента Центра Интернета вещей Azure.
author: ValOlson
ms.author: valls
ms.date: 2/12/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: e932238849baf267983fb3ca1ebb082db169d9fd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101680102"
---
# <a name="device-update-for-iot-hub-agent-overview"></a>Обзор Обновления устройств для агента Центра Интернета вещей

Агент обновления устройства состоит из двух концептуальных уровней:

* Уровень интерфейса строится на основе [Azure IoT Самонастраивающийся (PnP)](https://docs.microsoft.com/azure/iot-pnp/overview-iot-plug-and-play) , позволяющей передавать сообщения между агентом обновления устройства и службами обновления устройств.
* Уровень платформы отвечает за высокоуровневые действия по обновлению загрузки, установки и применения, которые могут быть платформой или конкретными устройствами.

:::image type="content" source="media/understand-device-update/client-agent-reference-implementations.png" alt-text="Реализации агентов." lightbox="media/understand-device-update/client-agent-reference-implementations.png":::

## <a name="the-interface-layer"></a>Уровень интерфейса

Уровень интерфейса состоит из [интерфейса аду Core](https://github.com/Azure/iot-hub-device-update/tree/main/src/agent/adu_core_interface) и [интерфейса сведений об устройстве](https://github.com/Azure/iot-hub-device-update/tree/main/src/agent/device_info_interface).

Эти интерфейсы используют файл конфигурации для значений по умолчанию. Значения по умолчанию включают aduc_manufacturer и aduc_model для интерфейса Азуредевицеупдатекоре и модели и производителя для интерфейса Девицеинформатион. Дополнительные [сведения](device-update-configuration-file.md) см. в файле конфигурации.

### <a name="adu-core-interface"></a>Интерфейс аду Core

Интерфейс "аду Core" является основным каналом связи между агентом обновления устройства и службами. Дополнительные [сведения](device-update-plug-and-play.md#adu-core-interface) об этом интерфейсе.

### <a name="device-information-interface"></a>Интерфейс сведений об устройстве

Интерфейс сведений об устройстве используется для реализации `Azure IoT PnP DeviceInformation` интерфейса. Дополнительные [сведения](device-update-plug-and-play.md#device-information-interface) об этом интерфейсе.

## <a name="the-platform-layer"></a>Уровень платформы

Существует две реализации уровня платформы. Уровень платформы имитатора имеет простую реализацию действий обновления и используется для быстрого тестирования и оценки обновления устройства для служб и установки центра Интернета вещей. Когда агент обновления устройства строится на уровне платформы имитатора, мы будем называть его агентом обновления устройства симулятора или просто симулятором. Дополнительные [сведения](https://github.com/Azure/iot-hub-device-update/blob/main/docs/agent-reference/how-to-run-agent.md) об использовании агента имитатора. Уровень платформы Linux интегрируется с [оптимизацией доставки](https://github.com/microsoft/do-client) для загрузки и используется в нашем эталонном образе Raspberry Pi и на всех клиентах, работающих в системах Linux.

### <a name="simulator-platform-layer"></a>Уровень платформы имитатора

Реализация уровня платформы имитатора может быть найдена в `src/platform_layers/simulator_platform_layer` и может использоваться для тестирования и оценки обновления устройства для центра Интернета вещей.  Многие действия в реализации "симулятор" предназначены для уменьшения физических изменений, чтобы поэкспериментировать с обновлением устройства для центра Интернета вещей.  Сквозное обновление можно выполнить с помощью этого уровня платформы. Дополнительные [сведения](https://github.com/Azure/iot-hub-device-update/blob/main/docs/agent-reference/how-to-run-agent.md) о запуске агента имитатора.

### <a name="linux-platform-layer"></a>Уровень платформы Linux

Реализация уровня платформы Linux может быть найдена в `src/platform_layers/linux_platform_layer` и интегрируется с [клиентом оптимизации доставки](https://github.com/microsoft/do-client/releases) для загрузки и используется в нашем эталонном образе Raspberry Pi и на всех клиентах, работающих в системах Linux.

Этот слой может интегрироваться с разными обработчиками обновлений для реализации установщика. Например, `SWUpdate` обработчик обновления вызывает скрипт оболочки для вызова `SWUpdate` исполняемого файла для выполнения обновления.

## <a name="update-handlers"></a>Обработчики обновлений

Обработчики обновлений — это компоненты, обрабатывающие части обновления содержимого или конкретного установщика. Реализации обработчика обновлений находятся в `src/content_handlers` .

### <a name="simulator-update-handler"></a>Обработчик обновлений симулятора

Обработчик обновлений симуляторов используется на уровне платформы имитатора и может использоваться с уровнем платформы Linux для имитации взаимодействия с обработчиком содержимого. Обработчик обновлений симуляторов реализует API обработчика обновлений с практически без операций. Реализацию обработчика обновлений симулятора можно найти ниже:
* [Симулятор обновления образов](https://github.com/Azure/iot-hub-device-update/blob/main/src/content_handlers/swupdate_handler/inc/aduc/swupdate_simulator_handler.hpp)
* [Симулятор обновления пакета apt](https://github.com/Azure/iot-hub-device-update/blob/main/src/content_handlers/apt_handler/inc/aduc/apt_simulator_handler.hpp)

Примечание. поле Инсталледкритериа в интерфейсе PnP Азуредевицеупдатекоре должно быть хэшем SHA256 содержимого. Это тот же хэш, который содержится в [объекте импорта манифеста](import-update.md#create-device-update-import-manifest). Дополнительные [сведения](device-update-plug-and-play.md) `installedCriteria` и `AzureDeviceUpdateCore` интерфейс.

### <a name="swupdate-update-handler"></a>`SWUpdate` Обработчик обновлений

`SWUpdate`Обработчик обновлений "интегрируется с `SWUpdate` исполняемым файлом командной строки и другими командами оболочки для реализации обновлений A/B специально для эталонного образа Raspberry Pi. Найдите новейшее эталонное изображение Raspberry Pi [здесь](https://github.com/Azure/iot-hub-device-update/releases). `SWUpdate`Обработчик обновлений реализован в [src/content_handlers/swupdate_content_handler](https://github.com/Azure/iot-hub-device-update/tree/main/src/content_handlers/swupdate_handler).

### <a name="apt-update-handler"></a>Обработчик обновлений APT

Обработчик обновлений APT обрабатывает манифест обновления, относящийся к APT, и вызывает APT для установки или обновления указанных пакетов Debian.

## <a name="self-update-device-update-agent"></a>Самостоятельное обновление агента обновления устройства

Агент обновления устройства и его зависимости можно обновить с помощью обновления устройства для конвейера центра Интернета вещей. Если вы используете обновление на основе образа, включите в новый образ последнюю версию агента обновления устройства. Если вы используете обновление на основе пакета, включите агент обновления устройства и требуемую версию в манифест APT, как и любой другой пакет. Дополнительные [сведения](device-update-apt-manifest.md) о манифесте APT. Установленную версию агента обновления устройства и агента оптимизации доставки можно проверить в разделе Свойства устройства в [двойника устройства IOT](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-device-twins). Дополнительные [сведения о свойствах устройств см. в разделе интерфейс аду Core](device-update-plug-and-play.md#device-properties).

## <a name="next-steps"></a>Дальнейшие действия
[Общие сведения о файле конфигурации агента обновления устройства](device-update-configuration-file.md)

