---
title: Обновление устройства для сетевых требований центра Интернета вещей | Документация Майкрософт
description: Обновление устройства для центра Интернета вещей использует различные сетевые порты для разных целей.
author: philmea
ms.author: philmea
ms.date: 1/11/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: 0512308fbaa0a725c6ecca573c70c90d8c04e247
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105558386"
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

Дополнительные [сведения](../iot-hub/iot-hub-devguide-protocols.md) о текущем списке поддерживаемых протоколов.
