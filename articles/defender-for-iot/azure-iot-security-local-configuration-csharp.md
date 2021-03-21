---
title: Локальная конфигурация агента безопасности IoT для защитника (C#)
description: Узнайте больше о защитнике для службы безопасности IoT, локальном файле конфигурации агента безопасности для C#.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: elazark
manager: rkarlin
editor: ''
ms.devlang: na
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/08/2020
ms.author: v-ekrieg
ms.openlocfilehash: 14237894a38725ee07deca3b5f46cbe0ecfce4cd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103493226"
---
# <a name="understanding-the-local-configuration-file-c-agent"></a>Основные сведения о локальном файле конфигурации (агент C#)

Агент безопасности для защитника IoT использует конфигурации из локального файла конфигурации.

Агент безопасности считывает файл конфигурации один раз после запуска агента. Конфигурации, найденные в локальном файле конфигурации, содержат конфигурацию проверки подлинности и другие конфигурации, связанные с агентом.

Агент безопасности C# использует несколько файлов конфигурации:

- Конфигурации, связанные с **General.config** агента.
- **Authentication.config** — конфигурация, связанная с аутентификацией (включая сведения о проверке подлинности).
- **SecurityIotInterface.config** — конфигурации, связанные с IOT.

Файлы конфигурации содержат конфигурацию по умолчанию. Конфигурация проверки подлинности заполняется во время установки агента, и изменения в файле конфигурации вносятся при перезапуске агента.

## <a name="configuration-file-location"></a>Расположение файла конфигурации

Для Linux:

- Файлы конфигурации операционной системы находятся в папке `/var/ASCIoTAgent` .

Для Windows:

- Файлы конфигурации операционной системы находятся в каталоге агента безопасности.

### <a name="generalconfig-configurations"></a>Конфигурации General.config

| Имя конфигурации | Возможные значения | Сведения |
|:-----------|:---------------|:--------|
| agentId | Код GUID | Уникальный идентификатор агента |
| реадремотеконфигуратионтимеаут | TimeSpan | Период времени для выборки удаленной конфигурации из центра Интернета вещей. Если агент не может получить конфигурацию в течение заданного времени, операция завершится с истечением времени ожидания.|
| счедулеринтервал | TimeSpan | Внутренний интервал планировщика. |
| продуцеринтервал | TimeSpan | Интервал рабочей роли производителя события. |
| консумеринтервал | TimeSpan | Интервал рабочего процесса потребителя событий. |
| хигхприоритикуеуесизеперцентаже | 0 < число < 1 | Часть общего кэша, выделенная для сообщений с высоким приоритетом. |
| LogLevel | "OFF", "неустранимые", "Error", "warning", "Information", "Debug"  | Сообщения журнала, равные и превышающие эту серьезность, регистрируются в консоли отладки (системный журнал в Linux). |
| филелоглевел |  "OFF", "неустранимые", "Error", "warning", "Information", "Debug"| Сообщения журнала, равные и превышающие эту серьезность, записываются в файл (системный журнал в Linux). |
| диагностиквербоситилевел | «None», «some», «ALL», | Уровень детализации событий диагностики. None — диагностические события не отправляются. Отправляются только диагностические события с высокой важностью. Все — все журналы также отправляются как диагностические события. |
| logFilePath | Путь к файлу | Если Филелоглевел >, журналы записываются в этот файл. |
| дефаултевентприорити | "Высокий", "низкий", "Выкл" | Приоритет события по умолчанию. |

### <a name="generalconfig-example"></a>Пример General.config

```xml
<?xml version="1.0" encoding="utf-8"?>
<General>
  <add key="agentId" value="da00006c-dae9-4273-9abc-bcb7b7b4a987" />
  <add key="readRemoteConfigurationTimeout" value="00:00:30" />
  <add key="schedulerInterval" value="00:00:01" />
  <add key="producerInterval" value="00:02:00" />
  <add key="consumerInterval" value="00:02:00" />
  <add key="highPriorityQueueSizePercentage" value="0.5" />
  <add key="logLevel" value="Information" />
  <add key="fileLogLevel" value="Off"/>
  <add key="diagnosticVerbosityLevel" value="Some" />
  <add key="logFilePath" value="IotAgentLog.log" />
  <add key="defaultEventPriority" value="Low"/>
</General>
```

### <a name="authenticationconfig"></a>Authentication.config

| Имя конфигурации | Возможные значения | Сведения |
|:-----------|:---------------|:--------|
| moduleName | строка | Имя удостоверения защитника IoT-Micro-Agent. Это имя должно соответствовать имени удостоверения модуля на устройстве. |
| deviceId | строка | ИДЕНТИФИКАТОР устройства (зарегистрированный в центре Интернета вещей Azure). |
| счедулеринтервал | Строка TimeSpan | Внутренний интервал планировщика. |
| гатевайхостнаме | строка | Имя узла центра Интернета вещей Azure. Обычно <My-Hub>. azure-devices.net |
| filePath | String — путь к файлу | Путь к файлу, содержащему секрет проверки подлинности.|
| тип | "SymmetricKey", "SelfSignedCertificate" | Секрет пользователя для проверки подлинности. Выберите *SymmetricKey* , если секрет пользователя является симметричным ключом, выберите *самозаверяющий сертификат* , если секрет является самозаверяющим сертификатом. |
| удостоверение | "DPS", "Module", "Device" | Удостоверение проверки подлинности — DP, если проверка подлинности выполняется с помощью DPS, модуля, если проверка подлинности выполняется с помощью учетных данных модуля, или устройства, если проверка подлинности выполняется
| цертификателокатионкинд |  "Локальный_файл", "Store" | Локальный_файл если сертификат хранится в файле, сохраните его, если сертификат находится в хранилище сертификатов. |
| idScope | строка | Область ИДЕНТИФИКАТОРов DPS |
| registrationId | строка  | ИДЕНТИФИКАТОР регистрации устройства DPS. |
|

### <a name="authenticationconfig-example"></a>Пример Authentication.config

```xml
<?xml version="1.0" encoding="utf-8"?>
<Authentication>
  <add key="moduleName" value="azureiotsecurity"/>
  <add key="deviceId" value="d1"/>
  <add key="gatewayHostname" value=""/>
  <add key="filePath" value="c:\p-dps-d1.pfx"/>
  <add key="type" value="SelfSignedCertificate" />                     <!-- SymmetricKey, SelfSignedCertificate-->
  <add key="identity" value="DPS" />                 <!-- Device, Module, DPS -->
  <add key="certificateLocationKind" value="LocalFile" />  <!-- LocalFile, Store -->
  <add key="idScope" value="0ne0005335B"/>
  <add key="registrationId" value="d1"/>
</Authentication>
```

### <a name="securityiotinterfaceconfig"></a>SecurityIotInterface.config

| Имя конфигурации | Возможные значения | Сведения |
|:-----------|:---------------|:--------|
| transportType | "AMPQ" "Mqtt" | Тип транспорта центра Интернета вещей. |
|

### <a name="securityiotinterfaceconfig-example"></a>Пример SecurityIotInterface.config

```xml
<ExternalInterface>
  <add key="facadeType"  value="Microsoft.Azure.Security.IoT.Agent.Common.SecurityIoTHubInterface, Security.Common" />
  <add key="transportType" value="Amqp"/>
</ExternalInterface>
```

## <a name="next-steps"></a>Следующие шаги

- Ознакомьтесь с [обзором](overview.md) службы "защитник для Интернета вещей"
- Дополнительные сведения о защитнике для [архитектуры](architecture.md) IOT
- Включение защитника для [службы](quickstart-onboard-iot-hub.md) IOT
- [Вопросы и ответы](resources-frequently-asked-questions.md) о службе "защитник для Интернета вещей"
- Узнайте, как получить доступ к [необработанным данным безопасности](how-to-security-data-access.md).
- Общие сведения о [рекомендациях](concept-recommendations.md)
- Общие сведения об [оповещениях](concept-security-alerts.md) системы безопасности
