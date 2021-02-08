---
title: Исследование подозрительного устройства
description: В этом разделе описывается, как с помощью защитника для Интернета вещей исследовать подозрительные устройства Интернета вещей, используя Log Analytics.
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
ms.date: 09/04/2020
ms.author: mlottner
ms.openlocfilehash: a7b51138abe6d8e97f55ceae11d4cf13b9ebc136
ms.sourcegitcommit: 2501fe97400e16f4008449abd1dd6e000973a174
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2021
ms.locfileid: "99820608"
---
# <a name="investigate-a-suspicious-iot-device"></a>Исследование подозрительных устройств Интернета вещей

Защитник для оповещений службы Интернета вещей предоставляет четкие указания, когда устройства Интернета вещей могут быть вовлечены в подозрительные действия или когда есть признаки возникновения нарушения безопасности устройства.

В этом руководство вы можете использовать рекомендации по исследованию, которые помогут определить потенциальные риски для Организации, решить, как исправляться, и найти лучшие способы предотвращения подобных атак в будущем.

> [!div class="checklist"]
> * Поиск данных об устройстве
> * Анализ с использованием запросов kql

## <a name="how-can-i-access-my-data"></a>Как получить доступ к моим данным?

По умолчанию защитник для Интернета вещей хранит ваши оповещения и рекомендации по безопасности в рабочей области Log Analytics. Можно также хранить необработанные данные системы безопасности.

Чтобы выбрать рабочую область Log Analytics для хранения данных, выполните следующие действия.

1. Откройте Центр Интернета вещей.
1. В разделе **Безопасность** выберите **Параметры**, а затем выберите **сбор данных**.
1. Измените сведения о конфигурации рабочей области Log Analytics.
1. Щелкните **Save** (Сохранить).

После этого для доступа к данным, хранящимся в рабочей области Log Analytics, сделайте следующее:

1. Выберите и щелкните защитник для оповещения IoT в центре Интернета вещей.
1. Щелкните **Further investigation** (Дальнейшие исследования).
1. Выберите **To see which devices have this alert click here and view the DeviceId column** (Чтобы узнать, какие устройства имеют это предупреждение, щелкните здесь и просмотрите столбец DeviceId).

## <a name="investigation-steps-for-suspicious-iot-devices"></a>Исследование подозрительных устройств Интернета вещей

Чтобы просмотреть аналитические сведения и необработанные данные об устройствах IoT, перейдите в рабочую область Log Analytics, [чтобы получить доступ к данным](#how-can-i-access-my-data).

Ознакомьтесь с примерами запросов ККЛ ниже, чтобы приступить к исследованию оповещений и действий на устройстве.

### <a name="related-alerts"></a>Связанные предупреждения

Чтобы узнать, были ли в это же время активированы другие оповещения, используйте следующий запрос kql:

  ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityAlert
  | where ExtendedProperties contains device and ResourceId contains tolower(hub)
  | project TimeGenerated, AlertName, AlertSeverity, Description, ExtendedProperties
  ```

### <a name="users-with-access"></a>Пользователи с доступом

Чтобы узнать, какие пользователи имеют доступ к этому устройству, используйте следующий запрос kql:

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "LocalUsers"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     GroupNames=extractjson("$.GroupNames", EventDetails, typeof(string)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string))
  | summarize FirstObserved=min(TimestampLocal) by GroupNames, UserName
 ```
Используйте следующие данные для исследования:

- Какие пользователи имеют доступ к устройству?
- Имеют ли пользователи с доступом ожидаемые уровни разрешений?

### <a name="open-ports"></a>Открытие портов

Чтобы узнать, какие порты на устройстве в настоящее время используются или использовались, используйте следующий запрос ККЛ:

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ListeningPorts"
     and extractjson("$.LocalPort", EventDetails, typeof(int)) <= 1024 // avoid short-lived TCP ports (Ephemeral)
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Protocol=extractjson("$.Protocol", EventDetails, typeof(string)),
     LocalAddress=extractjson("$.LocalAddress", EventDetails, typeof(string)),
     LocalPort=extractjson("$.LocalPort", EventDetails, typeof(int)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     RemotePort=extractjson("$.RemotePort", EventDetails, typeof(string))
  | summarize MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), AllowedRemoteIPAddress=makeset(RemoteAddress), AllowedRemotePort=makeset(RemotePort) by Protocol, LocalPort
 ```

Используйте следующие данные для исследования:

- Какие прослушивающие сокеты сейчас активны на устройстве?
- Разрешены ли прослушиваемые сокеты, активные в данный момент?
- Есть ли подозрительные удаленные адреса, подключенные к устройству?

### <a name="user-logins"></a>Имена входа пользователей

Чтобы найти пользователей, выполнивших вход на устройство, используйте следующий запрос ККЛ:

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "Login"
     // filter out local, invalid and failed logins
     and EventDetails contains "RemoteAddress"
     and EventDetails !contains '"RemoteAddress":"127.0.0.1"'
     and EventDetails !contains '"UserName":"(invalid user)"'
     and EventDetails !contains '"UserName":"(unknown user)"'
     //and EventDetails !contains '"Result":"Fail"'
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string)),
     LoginHandler=extractjson("$.Executable", EventDetails, typeof(string)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     Result=extractjson("$.Result", EventDetails, typeof(string))
  | summarize CntLoginAttempts=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), CntIPAddress=dcount(RemoteAddress), IPAddress=makeset(RemoteAddress) by UserName, Result, LoginHandler
 ```

Используйте следующие результаты запроса для исследования:

- Какие пользователи входили на устройство?
- Должны ли пользователи, вошедшие в систему, входить в систему?
- Подключались ли вошедшие пользователи с ожидаемых или неожидаемых IP-адресов?

### <a name="process-list"></a>Список процессов

Чтобы узнать, соответствует ли список процессов ожидаемым, используйте следующий запрос ККЛ:

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ProcessCreate"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Executable=extractjson("$.Executable", EventDetails, typeof(string)),
     UserId=extractjson("$.UserId", EventDetails, typeof(string)),
     CommandLine=extractjson("$.CommandLine", EventDetails, typeof(string))
  | join kind=leftouter (
     // user UserId details
     SecurityIoTRawEvent
     | where
        DeviceId == device and AssociatedResourceId contains tolower(hub)
        and RawEventName == "LocalUsers"
     | project
        UserId=extractjson("$.UserId", EventDetails, typeof(string)),
        UserName=extractjson("$.UserName", EventDetails, typeof(string))
     | distinct UserId, UserName
  ) on UserId
  | extend UserIdName = strcat("Id:", UserId, ", Name:", UserName)
  | summarize CntExecutions=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), ExecutingUsers=makeset(UserIdName), ExecutionCommandLines=makeset(CommandLine) by Executable
```

Используйте следующие результаты запроса для исследования:

- Выполнялись ли подозрительные процессы на устройстве?
- Запускались ли такие процессы санкционированным пользователем?
- Вводились ли в командную строку правильные и ожидаемые аргументы?

## <a name="next-steps"></a>Следующие шаги

После изучения устройства и получения сведений о рисках вы можете [настроить получаемые оповещения](quickstart-create-custom-alerts.md), чтобы повысить уровень безопасности решения Интернета вещей. Если у вас еще нет агента безопасности устройства, вы можете [развернуть его](how-to-deploy-agent.md) или [изменить его конфигурацию](how-to-agent-configuration.md) для улучшения результатов.
