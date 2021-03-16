---
title: Агенты безопасности
description: Приступите к изучению, настройке, развертыванию и использованию защитника Azure для агентов службы безопасности IoT на устройствах IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: shhazam-ms
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 1/24/2021
ms.author: shhazam
ms.openlocfilehash: 3b98013eab1ae8d21b9da7c1a4460551dc363c80
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103487774"
---
# <a name="get-started-with-azure-defender-for-iot-device-micro-agents"></a>Приступая к работе с защитником Azure для устройств IoT Micro Agent

Защитник для агентов безопасности IoT предлагает расширенные возможности безопасности, такие как мониторинг рекомендаций по настройке операционной системы. Управление защитой угроз в поле устройства и обеспечение безопасности с помощью одной службы.

Защитники для агентов безопасности IoT обрабатывали сбор необработанных событий из операционной системы устройства, агрегирование событий для снижения затрат и настройку с помощью модуля устройства двойника. Сообщения безопасности отправляются через центр Интернета вещей в защитник для служб аналитики IoT.

Используйте следующий рабочий процесс, чтобы развернуть и проверить защитник для агентов безопасности IoT:

1. [Включите Защитник для службы IOT в центре Интернета вещей](quickstart-onboard-iot-hub.md).

1. Если в центре Интернета вещей нет зарегистрированных устройств, [Зарегистрируйте новое устройство](../iot-accelerators/iot-accelerators-device-simulation-overview.md).

1. [Создайте модуль дефендериотмикроажент двойника](quickstart-create-micro-agent-module-twin.md) для своих устройств.

1. Чтобы установить агент на виртуальном устройстве Azure, а не при установке на реальном устройстве, следует выполнить [Запуск новой виртуальной машины Azure](../virtual-machines/linux/quick-create-portal.md) в доступной зоне.

1. [Разверните защитник для агента безопасности IOT](how-to-deploy-linux-cs.md) на устройстве IOT или на новой виртуальной машине.

1. Следуйте инструкциям по [trigger_events](https://aka.ms/iot-security-github-trigger-events) для запуска основного события ОС.

1. Проверьте наличие рекомендаций в защитнике для Интернета вещей в ответ на сбой проверки базового плана ОС на предыдущем шаге. Начните проверку через 30 минут после выполнения сценария.

## <a name="next-steps"></a>Дальнейшие действия

- Настройка [решения](quickstart-configure-your-solution.md)
- [Создание защитника — IoT-Micro-Agents](quickstart-create-security-twin.md)
- Настройка [пользовательских оповещений](quickstart-create-custom-alerts.md)
- [Развертывание агента безопасности](how-to-deploy-agent.md)