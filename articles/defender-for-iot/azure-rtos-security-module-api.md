---
title: Защитник — IoT-Micro-Agent для Azure RTO API
description: Справочный API для защитника IoT-Micro-Agent для Azure RTO.
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
ms.openlocfilehash: 9a285636ba10f7ca0668f597d0e9016cff73494a
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103494297"
---
# <a name="defender-iot-micro-agent-for-azure-rtos-api-preview"></a>Защитник-IoT-Micro-Agent для Azure RTO API (Предварительная версия)

Этот API предназначен только для использования с защитником-IoT-Micro-Agent для Azure RTO. Дополнительные ресурсы см. в разделе [защитник-IOT-Micro-Agent для Azure RTO GitHub](https://github.com/azure-rtos/azure-iot-preview/releases). 

## <a name="enable-defender-iot-micro-agent-for-azure-rtos"></a>Включение защитника — IoT-Micro-Agent для Azure RTO

**nx_azure_iot_security_module_enable**

### <a name="prototype"></a>Прототип

```c
UINT nx_azure_iot_security_module_enable(NX_AZURE_IOT *nx_azure_iot_ptr);
```

### <a name="description"></a>Описание

Эта подсистема включает подсистему "защитник Интернета вещей Azure — IoT-Micro-Agent". Внутренний конечный автомат управляет сбором событий безопасности и отправляет их в центр Интернета вещей Azure. Для управления сбором данных необходим только один экземпляр NX_AZURE_IOT_SECURITY_MODULE.

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

## <a name="disable-azure-iot-defender-iot-micro-agent"></a>Отключение защитника Интернета вещей Azure — IoT-Micro-Agent

**nx_azure_iot_security_module_disable**


### <a name="prototype"></a>Прототип

```c
UINT nx_azure_iot_security_module_disable(NX_AZURE_IOT *nx_azure_iot_ptr);
```

### <a name="description"></a>Описание

Эта подпрограммы отключает подсистему "защитник Интернета вещей Azure — IoT-Micro-Agent".

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

Дополнительные сведения о том, как приступить к работе с Azure RTO Defender — IoT-Micro-Agent, см. в следующих статьях:

- Ознакомьтесь с [обзором](iot-security-azure-rtos.md)защитника RTO защитника IOT-Micro-Agent.
