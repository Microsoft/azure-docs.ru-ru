---
title: Защитник для защитника IoT — IoT-Micro-Agent для IoT Edge
description: Узнайте об архитектуре и возможностях защитника Azure для защитника IoT — IoT-Micro-Agent для IoT Edge.
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
ms.openlocfilehash: 4b7377f74e24660945555c112fed4d77d2c5ddcc
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103495952"
---
# <a name="azure-defender-for-iot-edge-defender-iot-micro-agent"></a>Защитник Azure для IoT Edge защитника — IoT-Micro-Agent

[Azure IOT Edge](../iot-edge/index.yml) предоставляет мощные возможности для управления и выполнения бизнес-процессов на границе.
Ключевая часть, которую IoT Edge играет в средах IoT, делает ее особенно привлекательной для вредоносных субъектов.

Защитник для защитника IoT — IoT-Micro-Agent предоставляет комплексное решение по обеспечению безопасности для устройств IoT Edge.
Модуль "защитник для Интернета вещей" собирает, объединяет и анализирует необработанные данные безопасности из операционной системы и системы контейнеров в виде рекомендаций и оповещений о безопасности.

Подобно Защитнику для агентов безопасности IoT для устройств IoT, защитник для IoT Edge модуля легко настраивается через его модуль двойника.
Дополнительные сведения см. в разделе [Настройка агента](how-to-agent-configuration.md) .

Защитник для защитника IoT — IoT-Micro-Agent для IoT Edge предоставляет следующие возможности:

- Собирает необработанные события безопасности из базовой операционной системы (Linux) и IoT Edge системы контейнеров.

  Дополнительные сведения о доступных собирающих собираются данных см. в статье [Конфигурация агента для центра Интернета вещей (защитник](how-to-agent-configuration.md) ).

- Анализ манифестов развертывания IoT Edge.

- Выполняет статистическую обработку необработанных событий безопасности в сообщениях, отправляемых через [центр IOT Edge](../iot-edge/iot-edge-runtime.md#iot-edge-hub).

- Удалите конфигурацию, используя защитник-IoT-Micro-Agent двойника.

  Дополнительные сведения см. в статье [Настройка защитника для агента IOT](how-to-agent-configuration.md) .

Защитник для защитника IoT — IoT-Micro-Agent для IoT Edge работает в привилегированном режиме в IoT Edge.
Привилегированный режим необходим, чтобы модуль можно было отслеживать операционную систему, а также другие модули IoT Edge.

## <a name="module-supported-platforms"></a>Поддерживаемые модули платформы

Защитник для защитника IoT — IoT-Micro-Agent для IoT Edge в настоящее время доступен только для Linux.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали об архитектуре и возможностях защитника для защитника IoT — IoT-Micro-Agent для IoT Edge.

Чтобы продолжить знакомство с защитником для развертывания IoT, используйте следующие статьи:

- Развертывание [защитника — IOT-Micro-Agent для IOT Edge](how-to-deploy-edge.md)
- Узнайте, как [настроить защитника — IOT-Micro-Agent](how-to-agent-configuration.md)
- Обзор защитника для [защитника IOT для горизонта IOT](resources-manage-proprietary-protocols.md)
- Узнайте, как [включить защитник для службы IOT в центре Интернета вещей](quickstart-onboard-iot-hub.md)
- Дополнительные сведения о службе из [защитника для Интернета вещей: вопросы и ответы](resources-frequently-asked-questions.md)