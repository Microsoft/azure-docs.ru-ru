---
title: Установка агента C# на устройстве Windows
description: Узнайте, как установить защитник для агента IoT на 32-разрядных или 64-разрядных устройствах Windows.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: e7c7fdd5874dbde5ca304309d0840724cb3872df
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103494535"
---
# <a name="deploy-a-defender-for-iot-c-based-security-agent-for-windows"></a>Развертывание агента безопасности на основе C# для Windows IoT

В этом руководство объясняется, как установить защитник безопасности на основе C# для IoT в Windows.

Из этого руководства вы узнаете, как выполнить следующие задачи:

- Установка
- Проверка развертывания
- Удаление агента
- Диагностика

## <a name="prerequisites"></a>Предварительные требования

Сведения о других платформах и разновидностях агентов см. в разделе Выбор подходящего [агента безопасности](how-to-deploy-agent.md).

1. Права локального администратора на компьютере, на котором вы хотите установить.

1. [Создайте защитник-IOT-Micro-Agent](quickstart-create-security-twin.md) для устройства.

## <a name="installation"></a>Установка

Чтобы установить агент безопасности, используйте следующий рабочий процесс:

1. Установите на устройстве защитник для Windows на C# Agent. Скачайте последнюю версию на свой компьютер из [репозитория GitHub](https://github.com/Azure/Azure-IoT-Security-Agent-CS)для IOT.

1. Извлеките содержимое пакета и перейдите в папку /Install.

1. Откройте Windows PowerShell от имени администратора.
1. Добавьте разрешения на выполнение скрипта Инсталлсекуритяжент, выполнив команду:

    ```
    Unblock-File .\InstallSecurityAgent.ps1
    ```

    затем выполните:

    ```
    .\InstallSecurityAgent.ps1 -Install -aui <authentication identity> -aum <authentication method> -f <file path> -hn <host name> -di <device id> -cl <certificate location kind>
    ```

    Пример:

    ```
    .\InstallSecurityAgent.ps1 -Install -aui Device -aum SymmetricKey -f c:\Temp\Key.txt -hn MyIotHub.azure-devices.net -di Mydevice1 -cl store
    ```

    Дополнительные сведения о параметрах проверки подлинности см. [в разделе Настройка проверки подлинности](concept-security-agent-authentication-methods.md).

Этот сценарий выполняет следующие действия:

* Установка необходимых компонентов.
* Добавляет пользователя службы (с отключенным интерактивным входом).
* Установка агента в качестве **системной службы**.
* Настройка агента с предоставленными параметрами проверки подлинности.

Для получения дополнительной справки используйте команду Get-Help в PowerShell.

Get-Help пример:    ```Get-Help .\InstallSecurityAgent.ps1```

### <a name="verify-deployment-status"></a>Проверка состояния развертывания

Проверьте состояние развертывания агента, выполнив следующую команду:

```sc.exe query "ASC IoT Agent"```

### <a name="uninstall-the-agent"></a>Удаление агента

Чтобы удалить агент, сделайте следующее:

1. Запустите следующий скрипт PowerShell с параметром **-mode**, для которого задано значение **Uninstall**.

    ```
    .\InstallSecurityAgent.ps1 -Uninstall
    ```

## <a name="troubleshooting"></a>Устранение неполадок

Если агент не запускается, включите ведение журнала (по умолчанию ведение журнала *отключено*), чтобы просмотреть дополнительные сведения.

Чтобы включить ведение журнала, сделайте следующее:

1. Откройте файл конфигурации (General.config) для редактирования с помощью стандартного редактора файлов.

1. Измените следующие значения:

   ```xml
   <add key="logLevel" value="Debug" />
   <add key="fileLogLevel" value="Debug"/>
   <add key="diagnosticVerbosityLevel" value="Some" />
   <add key="logFilePath" value="IoTAgentLog.log" />
   ```

    > [!NOTE]
    > Мы рекомендуем **отключить** ведение журнала после устранения неполадок. Если оставить ведение журнала **включенным**, размер файла журнала и использование данных увеличится.

1. Перезапустите агент, выполнив следующую команду в PowerShell или командной строке.

    **PowerShell**

     ```
     Restart-Service "ASC IoT Agent"
     ```

   или диспетчер конфигурации служб

    **CMD**

     ```
     sc.exe stop "ASC IoT Agent"
     sc.exe start "ASC IoT Agent"
     ```

1. Дополнительные сведения о сбое можно просмотреть в файле журнала. Файл журнала будет находиться в рабочем каталоге, в котором выполняется сценарий. 

   Расположение файла журнала: `.\IoTAgentLog.log`.

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь с [обзором](overview.md) службы "защитник для Интернета вещей"
* Дополнительные сведения о защитнике для [архитектуры](architecture.md) IOT
* Включите [службу](quickstart-onboard-iot-hub.md).
* Ознакомьтесь с [вопросами и](resources-frequently-asked-questions.md) ответами
* Ознакомьтесь со сведениями об [оповещениях](concept-security-alerts.md).
