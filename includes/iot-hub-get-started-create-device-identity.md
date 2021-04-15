---
title: Включить файл
description: включить файл
services: iot-hub
author: dominicbetts
ms.service: iot-hub
ms.topic: include
ms.date: 09/07/2018
ms.author: dobett
ms.custom: include file, devx-track-azurecli
ms.openlocfilehash: b1a863498603006e37ab98b838ffd426b962d788
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92755741"
---
В этом разделе рассматривается создание удостоверения устройства для этой статьи с помощью Azure CLI. Идентификаторы устройств чувствительны к регистру.

1. Откройте [Azure Cloud Shell](https://shell.azure.com/).

1. В Azure Cloud Shell выполните следующую команду, чтобы установить расширение Интернета вещей Microsoft Azure для Azure CLI:

    ```azurecli-interactive
    az extension add --name azure-iot
    ```

2. Создайте удостоверение устройства с именем `myDeviceId` и получите строку подключения устройства с помощью следующих команд:

    ```azurecli-interactive
    az iot hub device-identity create --device-id myDeviceId --hub-name {Your IoT Hub name}
    az iot hub device-identity show-connection-string --device-id myDeviceId --hub-name {Your IoT Hub name} -o table
    ```

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

Запишите строку подключения устройства из результата. Строка подключения этого устройства используется приложением устройства для подключения к Центру Интернета вещей в качестве устройства.

<!-- images and links -->
