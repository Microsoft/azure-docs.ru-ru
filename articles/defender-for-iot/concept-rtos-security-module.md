---
title: Концептуальное описание основ защитника IoT-Micro-Agent для Azure RTO
description: Изучите основные понятия и рабочие процессы защитника IoT-Micro-Agent для Azure RTO.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 04a499f1feae630d3436c75ae2081413789c0ca3
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103494240"
---
# <a name="defender-iot-micro-agent-for-azure-rtos-preview"></a>Защитник-IoT-Micro-Agent для Azure RTO (Предварительная версия)

Используйте эту статью, чтобы получить более полное представление о службе "защитник-IoT-Micro-Agent для Azure RTO", включая функции и преимущества, а также ссылки на соответствующие конфигурации и справочные ресурсы. 

## <a name="azure-rtos-iot-defender-iot-micro-agent"></a>Защитник Azure RTO IoT — IoT-Micro-Agent

Защитник-IoT-Micro-Agent для Azure RTO предоставляет комплексное решение по обеспечению безопасности для устройств Azure RTO в рамках предложения Неткс Duo. В рамках предложения Неткс Duo Azure RTO поставляется с встроенной службой "защитник Azure IoT — IoT-Micro-Agent" и предоставляет сведения об общих угрозах для устройств операционной системы в реальном времени после активации. 

Защитник-IoT-Micro-Agent для Azure RTO работает в фоновом режиме и обеспечивает удобство работы пользователей при отправке сообщений безопасности с использованием уникальных подключений каждого клиента к центру Интернета вещей. Защитник-IoT-Micro-Agent для Azure RTO включен по умолчанию.  

## <a name="azure-rtos-netx-duo"></a>ОСРВ Azure NetX Duo

Azure RTO Неткс Duo — это расширенный, промышленно-ориентированный сетевой стек TCP/IP, разработанный специально для глубоко встраиваемых приложений в режиме реального времени и Интернета вещей. Azure RTO Неткс Duo — это сетевой стек с двумя IPv4 и IPv6, предоставляющий широкий набор протоколов, включая безопасность и облако. Узнайте больше о решениях [Azure RTO Неткс Duo](/azure/rtos/netx-duo/) .

Модуль предоставляет следующие возможности:

- **Обнаружение вредоносных сетевых действий**
- **Базовые показатели поведения устройства на основе настраиваемых оповещений**
- **Улучшение безопасности устройств санации**

## <a name="defender-iot-micro-agent-for-azure-rtos-architecture"></a>Архитектура RTO для защитника IoT-Micro-Agent для Azure

Защитник-IoT-Micro-Agent для Azure RTO инициализируется платформой по промежуточного слоя Интернета вещей Azure и использует клиенты центра Интернета вещей для отправки телеметрии безопасности в центр.

:::image type="content" source="media/architecture/security-module-state-diagram.png" alt-text="Защитник Интернета вещей Azure — IoT-Micro-Agent состояние и поток информации":::

Защитник-IoT-Micro-Agent для Azure RTO отслеживает следующие действия и сведения об устройстве с помощью трех средств телекопирования:
- Сетевая активность устройства **TCP**, **UDP** и **ICM**
- Сведения о системе в виде версий **среадкс** и **неткс Duo**
- События пульса

Каждый сборщик связан с группой приоритетов, а каждая группа приоритетов имеет свой собственный интервал с возможными значениями **низкий**, **средний** и **высокий**. Интервалы влияют на интервал времени, в течение которого данные собираются и отправляются.

Каждый интервал времени можно настроить и включить и отключить соединители IoT для дальнейшей [настройки решения](how-to-azure-rtos-security-module.md). 

## <a name="supported-security-alerts-and-recommendations"></a>Поддерживаемые оповещения и рекомендации по безопасности

Защитник-IoT-Micro-Agent для Azure RTO поддерживает определенные оповещения и рекомендации системы безопасности. Обязательно [Изучите и настройте соответствующие значения оповещений и рекомендаций](concept-rtos-security-alerts-recommendations.md) для службы после завершения начальной настройки.

## <a name="ready-to-begin"></a>Готовы начать?

Защитник-IoT-Micro-Agent для Azure RTO предоставляется в качестве бесплатной загрузки для устройств IoT. Служба "защитник для облачных служб Интернета вещей" доступна с 30-дневной пробной версией для каждой подписки Azure. [Скачайте защитник-IOT-Micro-Agent сейчас](https://github.com/azure-rtos/azure-iot-preview/releases) и приступим к работе. 

## <a name="next-steps"></a>Следующие шаги

- Приступая к работе с защитником IoT-Micro-Agent для RTO [предварительных требований и установки](quickstart-azure-rtos-security-module.md)Azure.
- Дополнительные сведения об [оповещениях безопасности и рекомендациях по защите](concept-rtos-security-alerts-recommendations.md)от защитника IOT-Micro-Agent для Azure RTO. 
- Используйте [Справочный API-интерфейс](azure-rtos-security-module-api.md)защитника IOT-Micro-Agent для Azure RTO.