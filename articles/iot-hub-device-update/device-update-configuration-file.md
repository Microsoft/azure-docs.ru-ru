---
title: Общие сведения об обновлении устройства для файла конфигурации центра Интернета вещей Azure | Документация Майкрософт
description: Общие сведения об обновлении устройства для файла конфигурации центра Интернета вещей Azure.
author: ValOlson
ms.author: valls
ms.date: 2/12/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: 39ecd9d6d0deec942d5c8cce9357c779a4b328d3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101662948"
---
# <a name="device-update-for-iot-hub-configuration-file"></a>Обновление устройства для файла конфигурации центра Интернета вещей

Файл "adu-conf.txt" является дополнительным и может создаваться для управления следующими настройками.

* AzureDeviceUpdateCore:4.ClientMetadata:4.deviceProperties["производитель"]
* AzureDeviceUpdateCore:4.ClientMetadata:4.deviceProperties["модель"]
* DeviceInformation.manufacturer
* DeviceInformation.model
* Строка подключения устройства (если она неизвестна агенту обновления устройств).

## <a name="purpose"></a>Цель
Агент обновления устройств сначала пытается получить значения `manufacturer` и `model` от устройства, которое используется для [уровня интерфейса](device-update-agent-overview.md#the-interface-layer). Если это не получится, агент обновления устройств продолжит поиск файла "adu-conf.txt" и будет использовать значения из этого файла. Если обе попытки поиска окажутся неудачными, агент обновления устройств будет использовать значения, заданные [по умолчанию](https://github.com/Azure/iot-hub-device-update/blob/main/CMakeLists.txt).

В следующих статьях можно получить дополнительные сведения о [Базовом интерфейсе обновления устройств Azure](https://github.com/Azure/iot-hub-device-update/tree/main/src/agent/adu_core_interface) и [Интерфейсе сведений об устройстве](https://github.com/Azure/iot-hub-device-update/tree/main/src/agent/device_info_interface).

## <a name="file-location"></a>Размещение файла

В системе Linux в разделе или на диске с именем `adu` создайте текстовый файл "adu-conf.txt" со следующими полями.

## <a name="list-of-fields"></a>Список полей

|Имя|Описание|
|-----------|--------------------|
|connection_string|Предварительно подготовленная строка подключения, которую может использовать устройство для подключения к центру Интернета вещей. Примечание. Это не требуется, если вы настраиваете агент обновления устройств через [службу удостоверений Azure IoT](https://azure.github.io/iot-identity-service/)|
|aduc_manufacturer|Это значение передается интерфейсом `AzureDeviceUpdateCore:4.ClientMetadata:4` для классификации устройства с целью развертывания необходимых обновлений.|
|aduc_model|Это значение передается интерфейсом `AzureDeviceUpdateCore:4.ClientMetadata:4` для классификации устройства с целью развертывания необходимых обновлений.|
|manufacturer|Это значение передается агентом обновления устройств как часть интерфейса `DeviceInformation`.|
|model|Это значение передается агентом обновления устройств как часть интерфейса `DeviceInformation`.|

## <a name="example-adu-conftxt-file-contents"></a>Пример содержимого файла "adu-conf.txt"

```markdown

connection_string = `HostName=<yourIoTHubName>;DeviceId=<yourDeviceId>;SharedAccessKey=<yourSharedAccessKey>`
aduc_manufacturer = <value to send through `AzureDeviceUpdateCore:4.ClientMetadata:4.deviceProperties["manufacturer"]`
aduc_model = <value to send through `AzureDeviceUpdateCore:4.ClientMetadata:4.deviceProperties["model"]`
manufacturer = <value to send through `DeviceInformation.manufacturer`
model = <value to send through `DeviceInformation.manufacturer`
```
