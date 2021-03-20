---
title: Устранение неполадок центра Интернета вещей Azure 404103 Девиценотонлине
description: Сведения об исправлении ошибки 404103 Девиценотонлине
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: troubleshooting
ms.date: 01/30/2020
ms.author: jlian
ms.openlocfilehash: e648428f924cfc33421c8591c41f7ac85b05a033
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "76960819"
---
# <a name="404103-devicenotonline"></a>404103 DeviceNotOnline

В этой статье описываются причины и решения для ошибок **404103 девиценотонлине** .

## <a name="symptoms"></a>Симптомы

Прямой метод для устройства завершается ошибкой **404103 девиценотонлине** , даже если устройство находится в режиме «в сети». 

## <a name="cause"></a>Причина

Если известно, что устройство находится в сети и по-прежнему получает ошибку, вероятно, это вызвано тем, что обратный вызов прямого метода не зарегистрирован на устройстве.

## <a name="solution"></a>Решение

Чтобы правильно настроить устройство для обратных вызовов прямого метода, см. раздел [Работа с прямым методом на устройстве](iot-hub-devguide-direct-methods.md#handle-a-direct-method-on-a-device).