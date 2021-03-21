---
title: Защитник-IoT-Micro-Agent и устройство двойников
description: Узнайте о концепции защитника, IoT-Micro-Agent двойников и о том, как они используются в защитнике для Интернета вещей.
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
ms.date: 07/24/2019
ms.author: mlottner
ms.openlocfilehash: 552da329b90b102a13ef53158ec81be87684c1fc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103493192"
---
# <a name="defender-iot-micro-agent"></a>Защитник-IoT-Micro-Agent

В этой статье объясняется, как защитник для Интернета вещей использует двойниковы устройств и модули.

## <a name="device-twins"></a>Двойники устройства

Для созданных в Azure решений Интернета вещей двойники устройств играют ключевую роль в управлении устройствами и автоматизации процессов.

Решение "Defender для Интернета вещей" обеспечивает полную интеграцию с платформой управления устройствами Интернета вещей, позволяя управлять состоянием защиты устройства и использовать существующие возможности управления устройствами. Интеграцию обеспечивает механизм двойников Центра Интернета вещей.

Узнайте больше о концепции [двойниковов устройств](../iot-hub/iot-hub-devguide-device-twins.md) в центре Интернета вещей Azure.

## <a name="defender-iot-micro-agent-twins"></a>Защитник-IoT-Micro-Agent двойников

Защитник для Интернета вещей поддерживает защитник-IoT-Micro-Agent двойника для каждого устройства в службе.
Защитник-IoT-Micro-Agent двойника содержит всю информацию, относящуюся к безопасности устройств для каждого конкретного устройства в решении.
Свойства безопасности устройства хранятся в выделенном защитнике, IoT-Micro-Agent двойника для безопасного обмена данными и для включения обновлений и обслуживания, требующих меньше ресурсов.

Дополнительные сведения о создании, настройке и настройке двойника см. в статье [Создание защитника IOT-Micro-Agent двойника](quickstart-create-security-twin.md) и [Настройка агентов безопасности](how-to-agent-configuration.md) . Дополнительные сведения о концепции модуля двойников в центре Интернета вещей см. в разделе [Основные сведения о модуле двойников](../iot-hub/iot-hub-devguide-module-twins.md) .

## <a name="see-also"></a>См. также раздел

- [Обзор защитника для IoT](overview.md)
- [Развертывание агентов безопасности](how-to-deploy-agent.md)
- [Методы проверки подлинности агента безопасности](concept-security-agent-authentication-methods.md)