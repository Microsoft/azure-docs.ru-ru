---
title: Обновление устройства для сетевых требований центра Интернета вещей | Документация Майкрософт
description: Обновление устройства для центра Интернета вещей использует различные сетевые порты для разных целей.
author: philmea
ms.author: philmea
ms.date: 1/11/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: e72ff144a56f44ccaa695b7dab328e42052fce39
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101680091"
---
# <a name="ports-used-with-device-update-for-iot-hub"></a>Порты, используемые с обновлением устройства для центра Интернета вещей
АДУ использует различные сетевые порты для разных целей.

## <a name="default-ports"></a>Порты по умолчанию

Назначение|Номер порта |
---|---
Скачать из сетей или CDN  | 80 (протокол HTTP)
Скачать из MCC/CDN | 80 (протокол HTTP)
Подключение агента аду к центру Интернета вещей Azure  | 8883 (протокол MQTT)

## <a name="use-azure-iot-hub-supported-protocols"></a>Использование поддерживаемых протоколов в центре Интернета вещей Azure
Агент аду можно изменить для использования любого из поддерживаемых протоколов центра Интернета вещей Azure.

Дополнительные [сведения](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-protocols#:~:text=Table%202%20%20%20,%201%20more%20rows) о текущем списке поддерживаемых протоколов.
