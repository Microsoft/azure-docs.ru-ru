---
title: Настройка и настройка защитника — IoT-Micro-Agent для Azure RTO
description: Узнайте, как настроить и настроить защитник — IoT-Micro-Agent для Azure RTO.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: shhazam-ms
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/07/2021
ms.author: shhazam
ms.openlocfilehash: 874a783763882a28f2fe7078e3a264d09107808a
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103495102"
---
# <a name="configure-and-customize-defender-iot-micro-agent-for-azure-rtos-preview"></a>Настройка и настройка защитника — IoT-Micro-Agent для Azure RTO (Предварительная версия)

В этой статье описывается, как настроить защитник-IoT-Micro-Agent для устройства Azure RTO в соответствии с требованиями к сети, пропускной способности и памяти.

В каталоге необходимо выбрать целевой файл распространения с `*.dist` расширением `netxduo/addons/azure_iot/azure_iot_security_module/configs` .  

При использовании среды компиляции CMak необходимо установить параметр командной строки в `IOT_SECURITY_MODULE_DIST_TARGET` значение для выбранного значения. Например, `-DIOT_SECURITY_MODULE_DIST_TARGET=RTOS_BASE`.

В ИАР или другой среде компиляции без CMak необходимо добавить `netxduo/addons/azure_iot/azure_iot_security_module/inc/configs/<target distribution>/` путь к всем известным включаемым путям. Например, `netxduo/addons/azure_iot/azure_iot_security_module/inc/configs/RTOS_BASE`.

Используйте следующий файл, чтобы настроить поведение устройства.

**нетксдуо/надстройки/azure_iot/azure_iot_security_module/инк/конфигс/ \<target distribution> /asc_config. h**

В среде компиляции CMak необходимо изменить конфигурацию по умолчанию, изменив `netxduo/addons/azure_iot/azure_iot_security_module/configs/<target distribution>.dist` файл. Используйте следующий формат CMak `set(ASC_XXX ON)` или следующий файл `netxduo/addons/azure_iot/azure_iot_security_module/inc/configs/<target distribution>/asc_config.h` для всех других сред. Например, `#define ASC_XXX`.

Поведение каждой конфигурации по умолчанию предоставляется в следующих таблицах: 

## <a name="general"></a>Общие сведения

| Имя | Тип | По умолчанию | Сведения |
| - | - | - | - |
| ASC_SECURITY_MODULE_ID | Строковый | защитник-IOT-Micro-Agent | Уникальный идентификатор устройства.  |
| SECURITY_MODULE_VERSION_ (ОСНОВНОЙ) (ДОПОЛНИТЕЛЬНЫЙ) (ИСПРАВЛЕНИЕ)  | Число | 3.2.1 | Версия. |
| ASC_SECURITY_MODULE_SEND_MESSAGE_RETRY_TIME  | Число  | 3 | Время, в течение которого защитник-IoT-Micro-Agent будет отсылать сообщение безопасности после сбоя. (в секундах) |
| ASC_SECURITY_MODULE_PENDING_TIME  | Число | 300 | Время ожидания защитника IoT-Micro-Agent (в секундах). При превышении времени состояние изменится на Suspend. |

## <a name="collection"></a>Коллекция

| Имя | Тип | По умолчанию | Сведения |
| - | - | - | - |
| ASC_FIRST_COLLECTION_INTERVAL | Число  | 30  | Смещение интервала сбора для запуска сборщика. Во время запуска значение будет добавлено в коллекцию системы, чтобы избежать одновременной отправки сообщений с нескольких устройств.  |
| ASC_HIGH_PRIORITY_INTERVAL | Число | 10 | Интервал группы с высоким приоритетом сборщика (в секундах). |
| ASC_MEDIUM_PRIORITY_INTERVAL | Число | 30 | Интервал группы среднего приоритета сборщика (в секундах). |
| ASC_LOW_PRIORITY_INTERVAL | Число | 145 440  | Интервал группы с низким приоритетом сборщика (в секундах). |

#### <a name="collector-network-activity"></a>Активность сети сборщика

Чтобы настроить конфигурацию сетевой активности сборщика, используйте следующую команду:

| Имя | Тип | По умолчанию | Сведения |
| - | - | - | - |
| ASC_COLLECTOR_NETWORK_ACTIVITY_TCP_DISABLED | Логическое | false | Фильтрует `TCP` сетевую активность. |
| ASC_COLLECTOR_NETWORK_ACTIVITY_UDP_DISABLED | Логическое | false | Фильтрует `UDP` события активности сети. |
| ASC_COLLECTOR_NETWORK_ACTIVITY_ICMP_DISABLED | Логическое | false | Фильтрует `ICMP` события активности сети. |
| ASC_COLLECTOR_NETWORK_ACTIVITY_CAPTURE_UNICAST_ONLY | Логическое | Да | Захватывает только одноадресные входящие пакеты. Если задано значение false, он также будет записывать как широковещательные, так и многоадресные рассылки. |
| ASC_COLLECTOR_NETWORK_ACTIVITY_SEND_EMPTY_EVENTS  | Логическое  | false  | Отправляет пустые события сборщика. |
| ASC_COLLECTOR_NETWORK_ACTIVITY_MAX_IPV4_OBJECTS_IN_CACHE | Число | 64 | Максимальное число событий сети IPv4 для хранения в памяти. |
| ASC_COLLECTOR_NETWORK_ACTIVITY_MAX_IPV6_OBJECTS_IN_CACHE | Число | 64  | Максимальное число событий сети IPv6 для хранения в памяти. |

### <a name="collectors"></a>Сборщики
| Имя | Тип | По умолчанию | Сведения |
| - | - | - | - |
| ASC_COLLECTOR_HEARTBEAT_ENABLED | Логическое | ON | Включает сборщик пульса. |
| ASC_COLLECTOR_NETWORK_ACTIVITY_ENABLED  | Логическое | ON | Включает сборщик активности сети. |
| ASC_COLLECTOR_SYSTEM_INFORMATION_ENABLED | Логическое | ON | Включает сборщик системных сведений.  |

Другие флаги конфигурации являются расширенными и имеют неподдерживаемые функции. Обратитесь в службу поддержки, чтобы изменить это, или для получения дополнительных сведений.
 
## <a name="supported-security-alerts-and-recommendations"></a>Поддерживаемые оповещения и рекомендации по безопасности

Защитник-IoT-Micro-Agent для Azure RTO поддерживает определенные оповещения и рекомендации системы безопасности. Обязательно [Проверьте и настройте соответствующие значения оповещений и рекомендаций](concept-rtos-security-alerts-recommendations.md) для вашей службы.

## <a name="log-analytics-optional"></a>Log Analytics (необязательно)

Вы можете включить и настроить Log Analytics для изучения событий и действий устройства. Дополнительные сведения см. в статье о настройке и использовании [log Analytics со службой "защитник для Интернета вещей"](how-to-security-data-access.md#log-analytics) . 

## <a name="next-steps"></a>Дальнейшие действия


- Обзор и настройка защитника — IoT-Micro-Agent для [оповещений системы безопасности Azure RTO и рекомендации](concept-rtos-security-alerts-recommendations.md)
- При необходимости см. [сведения об API-интерфейсе защитника IOT-Micro-Agent для Azure RTO](azure-rtos-security-module-api.md) .
