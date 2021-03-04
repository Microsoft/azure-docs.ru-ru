---
title: API модуля безопасности для ОСРВ Azure
description: Справочный API модуля безопасности для Azure RTO.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/07/2020
ms.author: mlottner
ms.openlocfilehash: cec28f9290808836ec2dfd334b23fe8c76df03fc
ms.sourcegitcommit: dac05f662ac353c1c7c5294399fca2a99b4f89c8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102120068"
---
# <a name="security-module-for-azure-rtos-api"></a>API модуля безопасности для ОСРВ Azure 

Этот API предназначен для использования с модулем безопасности только для Azure RTO. Дополнительные ресурсы см. в разделе [модуль безопасности для Azure RTO GitHub Resource](https://github.com/azure-rtos/azure-iot-preview/releases). 

## <a name="enable-security-module-for-azure-rtos"></a>Включение модуля безопасности для Azure RTO

**nx_azure_iot_security_module_enable**

### <a name="prototype"></a>Прототип

```c
UINT nx_azure_iot_security_module_enable(NX_AZURE_IOT *nx_azure_iot_ptr);
```

### <a name="description"></a>Описание

Эта подпрограммы включает подсистему модуля безопасности Интернета вещей Azure. Внутренний конечный автомат управляет сбором событий безопасности и отправляет их в центр Интернета вещей Azure. Для управления сбором данных необходим только один экземпляр NX_AZURE_IOT_SECURITY_MODULE.

### <a name="parameters"></a>Параметры

| Имя | Описание |
|---------|---------|
| nx_azure_iot_ptr [in]    | Указатель на объект `NX_AZURE_IOT`.  |

### <a name="return-values"></a>Возвращаемые значения

|Возвращаемые значения  |Описание |
|---------|---------|
|NX_AZURE_IOT_SUCCESS|   Модуль безопасности IoT Azure успешно включен.     |
|NX_AZURE_IOT_FAILURE   |  Не удалось включить модуль безопасности IoT Azure из-за внутренней ошибки.    |
|NX_AZURE_IOT_INVALID_PARAMETER   |  Для модуля безопасности требуется допустимый экземпляр #NX_AZURE_IOT.      |

### <a name="allowed-from"></a>Разрешено из

Потоки

## <a name="disable-azure-iot-security-module"></a>Отключение модуля безопасности Azure IoT

**nx_azure_iot_security_module_disable**


### <a name="prototype"></a>Прототип

```c
UINT nx_azure_iot_security_module_disable(NX_AZURE_IOT *nx_azure_iot_ptr);
```

### <a name="description"></a>Описание

Эта подпрограммы отключает подсистему модуля безопасности Интернета вещей Azure.

### <a name="parameters"></a>Параметры

| Имя | Описание |
|---------|---------|
| nx_azure_iot_ptr [in]    | Указатель на объект `NX_AZURE_IOT`. Если значение NULL, то одноэлементный экземпляр отключен. |

### <a name="return-values"></a>Возвращаемые значения

|Возвращаемые значения  |Описание |
|---------|---------|
|NX_AZURE_IOT_SUCCESS     |   Успешно, если модуль безопасности IoT Azure успешно отключен.      |
|NX_AZURE_IOT_INVALID_PARAMETER   |  Экземпляр центра Интернета вещей Azure отличается от одноэлементного составного экземпляра.       |
|NX_AZURE_IOT_FAILURE    |  Не удалось отключить модуль безопасности IoT Azure из-за внутренней ошибки.       |

### <a name="allowed-from"></a>Разрешено из

Потоки


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о том, как приступить к работе с модулем безопасности Azure RTO, см. в следующих статьях:

- Ознакомьтесь с [обзором](iot-security-azure-rtos.md)модуля безопасности защитника для IOT RTO.