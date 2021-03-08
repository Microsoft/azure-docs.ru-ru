---
title: Выбор и развертывание агентов безопасности
description: Узнайте, как выбрать и развернуть защитник для агентов безопасности IoT на устройствах IoT.
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
ms.date: 07/23/2019
ms.author: mlottner
ms.openlocfilehash: 89ecf26778433a3f075f76905183f3fe866cb5db
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102447271"
---
# <a name="select-and-deploy-a-security-agent-on-your-iot-device"></a>Выбор и развертывание агента безопасности на устройстве IoT

Защитник для Интернета вещей предоставляет эталонные архитектуры для агентов безопасности, которые отслеживают и собираются данные с устройств IoT.
Дополнительные сведения см. в разделе [Эталонная архитектура агента безопасности](security-agent-architecture.md).

Агенты разрабатываются в виде проектов с открытым исходным кодом и доступны в двух разновидностях: <br> [C](https://aka.ms/iot-security-github-c)и [C#](https://aka.ms/iot-security-github-cs).

Вы узнаете, как выполнять следующие задачи:
- Сравнение разновидностей агента безопасности
- Обнаружение поддерживаемых платформ агентов
- Выберите подходящую версию агента для своего решения

## <a name="understand-security-agent-options"></a>Общие сведения о параметрах агента безопасности

Каждый защитник для версии агента безопасности IoT предлагает один и тот же набор функций и поддерживает аналогичные параметры конфигурации.

У агента безопасности на основе C есть меньше памяти, и это идеальный вариант для устройств с меньшим количеством доступных ресурсов.

|     | Агент безопасности на основе C | Агент безопасности на основе C# |
| --- | ----------- | --------- |
| **Открытый исходный код** | Доступно в рамках [лицензии MIT](https://en.wikipedia.org/wiki/MIT_License) в [GitHub](https://aka.ms/iot-security-github-c) | Доступно в рамках [лицензии MIT](https://en.wikipedia.org/wiki/MIT_License) в [GitHub](https://aka.ms/iot-security-github-cs) |
| **Язык разработки**    | C | C# |
| **Поддерживаемые платформы Windows** | Нет | Да |
| **Предварительные требования для Windows** | --- | [WMI](/windows/desktop/wmisdk/) |
| **Поддерживаемые платформы Linux?** | Да, x64 и x86 | Да, только x64 |
| **Предварительные требования для Linux** | libunwind8, libcurl3, UUID — среда выполнения, аудит, аудиспд — подключаемые модули | libunwind8, libcurl3, UUID-Runtime, audited, аудиспд-plugins, sudo, netstat, iptables |
| **Место на диске** | 10,5 МБ | 90 МБ |
| **Объем памяти (в среднем)** | 5,5 МБ | 33 МБ |
| **[Проверка подлинности](concept-security-agent-authentication-methods.md) в центре Интернета вещей** | Да | Да |
| **[Сбор](how-to-agent-configuration.md#supported-security-events) данных безопасности** | Да | Да |
| **Агрегация событий** | Да | Да |
| **Удаленная настройка через [модуль безопасности двойника](concept-security-module.md)** | Да | Да |

## <a name="security-agent-installation-guidelines"></a>Рекомендации по установке агента безопасности

Для **Windows**: сценарий установки SecurityAgent.ps1 должен быть выполнен из окна администратора PowerShell.

Для **Linux**: InstallSecurityAgent.sh должен быть запущен от имени суперпользователя. Рекомендуется выполнить префикс "Sudo" для команды установки.

## <a name="choose-an-agent-flavor"></a>Выберите разновидность агента

Чтобы выбрать правильный агент, ответьте на следующие вопросы о своих устройствах IoT:

- Используете ли вы _Windows Server_ или _Windows IOT Core_?

    [Развертывание агента безопасности на основе C# для Windows](how-to-deploy-windows-cs.md).

- Используете ли вы дистрибутив Linux с архитектурой x86?

    [Развертывание агента безопасности на основе C для Linux](how-to-deploy-linux-c.md).

- Используете ли вы дистрибутив Linux с архитектурой x64?

    Можно использовать обе разновидности агента. <br>
    [Развертывание агента безопасности на основе C для Linux](how-to-deploy-linux-c.md) и/или [развертывание агента безопасности на основе C# для Linux](how-to-deploy-linux-cs.md).

Обе разновидности агента предлагают один и тот же набор функций и поддерживают аналогичные параметры конфигурации.
Дополнительные сведения см. в разделе [Сравнение агента безопасности](how-to-deploy-agent.md#understand-security-agent-options) .

## <a name="supported-platforms"></a>Поддерживаемые платформы

В следующем списке перечислены все поддерживаемые в настоящее время платформы.

|Защитник для агента IoT |Операционная система |Архитектура |
|--------------|------------|--------------|
|C|Ubuntu 16.04 |    X64|
|C|Ubuntu 18.04 |    x64, ARMv7|
|C|Debian 9 |    x64, x86|
|C#|Ubuntu 16.04     |X64|
|C#|Ubuntu 18.04    |x64, ARMv7|
|C#|Debian 9    |X64|
|C#|Windows Server 2016|    X64|
|C#|Windows 10 IoT Базовая, сборка 17763    |X64|
|

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать больше о параметрах конфигурации, перейдите к руководству по настройке агента.
> [!div class="nextstepaction"]
> [Руководство по настройке агента](./how-to-agent-configuration.md)