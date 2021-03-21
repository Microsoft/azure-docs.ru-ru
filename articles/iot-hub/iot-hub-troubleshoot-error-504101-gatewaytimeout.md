---
title: Устранение неполадок центра Интернета вещей Azure 504101 Гатевайтимеаут
description: Сведения об исправлении ошибки 504101 Гатевайтимеаут
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.custom: amqp
ms.openlocfilehash: 373acc30ed652a7f540e840dfad5eeeda65ca179
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81759555"
---
# <a name="504101-gatewaytimeout"></a>504101 GatewayTimeout

В этой статье описываются причины и решения для ошибок **504101 гатевайтимеаут** .

## <a name="symptoms"></a>Симптомы

При попытке вызвать прямой метод из центра Интернета вещей на устройстве происходит сбой запроса с ошибкой **504101 гатевайтимеаут**.

## <a name="cause"></a>Причина

### <a name="cause-1"></a>Причина 1

Центр Интернета вещей обнаружил ошибку и не смог подтвердить, завершился ли прямой метод до истечения времени ожидания.

### <a name="cause-2"></a>Причина 2

При использовании более ранней версии пакета SDK для C# для Azure IoT (<1.19.0) связь AMQP между устройством и центром Интернета вещей может быть удалена без вмешательства пользователя из-за ошибки.

## <a name="solution"></a>Решение

### <a name="solution-1"></a>Решение 1

Выполните повторную попытку.

### <a name="solution-2"></a>Решение 2

Обновите пакет SDK для C# для Azure IOT до последней версии.