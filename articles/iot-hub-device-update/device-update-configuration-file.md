---
title: Общие сведения об обновлении устройства для файла конфигурации центра Интернета вещей Azure | Документация Майкрософт
description: Сведения об обновлении устройства для файла конфигурации центра Интернета вещей Azure.
author: ValOlson
ms.author: valls
ms.date: 2/12/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: 39ecd9d6d0deec942d5c8cce9357c779a4b328d3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101662948"
---
# <a name="device-update-for-iot-hub-configuration-file"></a>Обновление устройства для файла конфигурации центра Интернета вещей

"adu-conf.txt" — это необязательный файл, который можно создать для управления следующими конфигурациями.

* Азуредевицеупдатекоре: 4. Клиентметадата: 4. Девицепропертиес ["производитель"]
* Азуредевицеупдатекоре: 4. Клиентметадата: 4. Девицепропертиес ["Model"]
* Девицеинформатион. Manufacturer
* Девицеинформатион. Model
* Строка подключения устройства (если она неизвестна агентом обновления устройства).

## <a name="purpose"></a>Назначение
Агент обновления устройства сначала попытается получить `manufacturer` `model` значения и с устройства, которое будет использоваться для [уровня интерфейса](device-update-agent-overview.md#the-interface-layer). Если это не удается, агент обновления устройства будет далее искать файл "adu-conf.txt" и использовать значения из него. Если обе попытки не выполнены, агент обновления устройства будет использовать значения [по умолчанию](https://github.com/Azure/iot-hub-device-update/blob/main/CMakeLists.txt) .

Дополнительные сведения об [интерфейсе аду Core](https://github.com/Azure/iot-hub-device-update/tree/main/src/agent/adu_core_interface) и [интерфейсе сведений об устройстве](https://github.com/Azure/iot-hub-device-update/tree/main/src/agent/device_info_interface).

## <a name="file-location"></a>Размещение файла

В системе Linux в разделе или диске `adu` с именем создайте текстовый файл с именем "adu-conf.txt" со следующими полями.

## <a name="list-of-fields"></a>Список полей

|Имя|Описание|
|-----------|--------------------|
|connection_string|Предварительно подготовленная строка подключения, которую устройство может использовать для подключения к центру Интернета вещей. Примечание. Если вы подготавливаете агент обновления устройства через [службу удостоверений Azure IOT](https://azure.github.io/iot-identity-service/) , это не обязательно.|
|aduc_manufacturer|Сообщается `AzureDeviceUpdateCore:4.ClientMetadata:4` интерфейсу для классификации устройства, предназначенного для развертывания обновлений.|
|aduc_model|Сообщается `AzureDeviceUpdateCore:4.ClientMetadata:4` интерфейсу для классификации устройства, предназначенного для развертывания обновлений.|
|manufacturer|Сообщается агентом обновления устройства как часть `DeviceInformation` интерфейса.|
|model|Сообщается агентом обновления устройства как часть `DeviceInformation` интерфейса.|

## <a name="example-adu-conftxt-file-contents"></a>Пример содержимого файла "adu-conf.txt"

```markdown

connection_string = `HostName=<yourIoTHubName>;DeviceId=<yourDeviceId>;SharedAccessKey=<yourSharedAccessKey>`
aduc_manufacturer = <value to send through `AzureDeviceUpdateCore:4.ClientMetadata:4.deviceProperties["manufacturer"]`
aduc_model = <value to send through `AzureDeviceUpdateCore:4.ClientMetadata:4.deviceProperties["model"]`
manufacturer = <value to send through `DeviceInformation.manufacturer`
model = <value to send through `DeviceInformation.manufacturer`
```
