---
title: Общие сведения о заданиях Центра Интернета вещей Azure | Документация Майкрософт
description: Руководство разработчика. Планирование выполнения заданий на нескольких устройствах, подключенных к Центру Интернета вещей. Задания могут обновлять теги и требуемые свойства, а также вызывать прямые методы на нескольких устройствах.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/06/2019
ms.custom: mqtt
ms.openlocfilehash: 5c14e8cfcbf8df86b0f71d6b12025594d2e648c4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "81730111"
---
# <a name="schedule-jobs-on-multiple-devices"></a>Планирование заданий на нескольких устройствах

Центр Интернета вещей Azure включает ряд стандартных блоков ([свойства и теги двойников устройств](iot-hub-devguide-device-twins.md) и [прямые методы](iot-hub-devguide-direct-methods.md)). Как правило, внутренние приложения позволяют администраторам и операторам устройств массово и в запланированное время обновлять устройства Интернета вещей и взаимодействовать с ними. На запланированное время задания выполняют обновления для двойников устройств и прямых методов для ряда устройств. Например, оператор использует внутреннее приложение, которое инициирует задание перезапуска ряда устройств на третьем этаже в здании №43 в определенное время и отслеживает ход его выполнения. При этом задание не будет нарушать выполнение других операций в здании.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Используйте задания, если необходимо запланировать выполнение задания на ряде устройств, а также настроить отслеживание выполнения действий:

* Обновление требуемых свойств
* Обновление тегов
* Вызов прямых методов

## <a name="job-lifecycle"></a>Жизненный цикл задания

Задания инициируются в серверной части решения, а осуществляются с помощью Центра Интернета вещей. Вы можете инициировать задание, используя обращенный к службе универсальный код ресурса (URI) `PUT https://<iot hub>/jobs/v2/<jobID>?api-version=2018-06-30` и запросить данные о выполнении задания, используя аналогичный URI `GET https://<iot hub>/jobs/v2/<jobID?api-version=2018-06-30`. Чтобы обновить состояние выполняемых заданий после запуска задания, выполните запрос задания.

> [!NOTE]
> При запуске задания в именах и значениях свойств должны содержаться только печатаемые буквенно-цифровые символы US-ASCII, кроме следующих: `$ ( ) < > @ , ; : \ " / [ ] ? = { } SP HT`.

## <a name="jobs-to-execute-direct-methods"></a>Задания для выполнения прямого метода

Приведенный ниже фрагмент запроса HTTPS 1.1 предназначен для выполнения [прямого метода](iot-hub-devguide-direct-methods.md) с использованием задания на ряде устройств:

```
PUT /jobs/v2/<jobId>?api-version=2018-06-30

Authorization: <config.sharedAccessSignature>
Content-Type: application/json; charset=utf-8

{
    "jobId": "<jobId>",
    "type": "scheduleDeviceMethod",
    "cloudToDeviceMethod": {
        "methodName": "<methodName>",
        "payload": <payload>,
        "responseTimeoutInSeconds": methodTimeoutInSeconds
    },
    "queryCondition": "<queryOrDevices>", // query condition
    "startTime": <jobStartTime>,          // as an ISO-8601 date string
    "maxExecutionTimeInSeconds": <maxExecutionTimeInSeconds>
}
```

Условие запроса также может находиться в одном идентификаторе устройства или в списке идентификаторов устройств, как показано ниже.

```
"queryCondition" = "deviceId = 'MyDevice1'"
"queryCondition" = "deviceId IN ['MyDevice1','MyDevice2']"
"queryCondition" = "deviceId IN ['MyDevice1']"
```

В статье [Справочник по языку запросов Центра Интернета вещей для двойников устройств и заданий](iot-hub-devguide-query-language.md) подробно рассматривается язык запросов Центра Интернета вещей.

В следующем фрагменте кода показан запрос и ответ для задания, запланированного на вызов прямого метода с именем testMethod на всех устройствах в Contoso-Hub-1.

```
PUT https://contoso-hub-1.azure-devices.net/jobs/v2/job01?api-version=2018-06-30 HTTP/1.1
Authorization: SharedAccessSignature sr=contoso-hub-1.azure-devices.net&sig=68iv------------------------------------v8Hxalg%3D&se=1556849884&skn=iothubowner
Content-Type: application/json; charset=utf-8
Host: contoso-hub-1.azure-devices.net
Content-Length: 317

{
    "jobId": "job01",
    "type": "scheduleDeviceMethod",
    "cloudToDeviceMethod": {
        "methodName": "testMethod",
        "payload": {},
        "responseTimeoutInSeconds": 30
    },
    "queryCondition": "*", 
    "startTime": "2019-05-04T15:53:00.077Z",
    "maxExecutionTimeInSeconds": 20
}

HTTP/1.1 200 OK
Content-Length: 65
Content-Type: application/json; charset=utf-8
Vary: Origin
Server: Microsoft-HTTPAPI/2.0
Date: Fri, 03 May 2019 01:46:18 GMT

{"jobId":"job01","type":"scheduleDeviceMethod","status":"queued"}
```

## <a name="jobs-to-update-device-twin-properties"></a>Задания для обновления свойств устройств-двойников

В следующем фрагменте показаны сведения по обновлению свойств устройства-двойника с использованием задания в запросе HTTPS 1.1.

```
PUT /jobs/v2/<jobId>?api-version=2018-06-30

Authorization: <config.sharedAccessSignature>
Content-Type: application/json; charset=utf-8

{
    "jobId": "<jobId>",
    "type": "scheduleUpdateTwin",
    "updateTwin": <patch>                 // Valid JSON object
    "queryCondition": "<queryOrDevices>", // query condition
    "startTime": <jobStartTime>,          // as an ISO-8601 date string
    "maxExecutionTimeInSeconds": <maxExecutionTimeInSeconds>
}
```

> [!NOTE]
> Для свойства *упдатетвин* требуется допустимое совпадение ETag. Например, `etag="*"` .

В следующем фрагменте кода показан запрос и ответ для задания, запланированного на обновление свойств двойникаа устройства для Test-device в Contoso-Hub-1:

```
PUT https://contoso-hub-1.azure-devices.net/jobs/v2/job02?api-version=2018-06-30 HTTP/1.1
Authorization: SharedAccessSignature sr=contoso-hub-1.azure-devices.net&sig=BN0U-------------------------------------RuA%3D&se=1556925787&skn=iothubowner
Content-Type: application/json; charset=utf-8
Host: contoso-hub-1.azure-devices.net
Content-Length: 339

{
    "jobId": "job02",
    "type": "scheduleUpdateTwin",
    "updateTwin": {
      "properties": {
        "desired": {
          "test1": "value1"
        }
      },
     "etag": "*"
     },
    "queryCondition": "deviceId = 'test-device'",
    "startTime": "2019-05-08T12:19:56.868Z",
    "maxExecutionTimeInSeconds": 20
}

HTTP/1.1 200 OK
Content-Length: 63
Content-Type: application/json; charset=utf-8
Vary: Origin
Server: Microsoft-HTTPAPI/2.0
Date: Fri, 03 May 2019 22:45:13 GMT

{"jobId":"job02","type":"scheduleUpdateTwin","status":"queued"}
```

## <a name="querying-for-progress-on-jobs"></a>Запрос на отслеживание выполнения заданий

Следующий фрагмент кода HTTPS 1.1 используется для получения сведений о выполнении заданий с помощью запросов:

```
GET /jobs/v2/query?api-version=2018-06-30[&jobType=<jobType>][&jobStatus=<jobStatus>][&pageSize=<pageSize>][&continuationToken=<continuationToken>]

Authorization: <config.sharedAccessSignature>
Content-Type: application/json; charset=utf-8
```

Объект continuationToken получен в рамках ответа.

Вы можете выполнять запросы для получения состояния выполнения заданий на каждом устройстве, используя [язык запросов Центра Интернета вещей для двойников устройств, заданий и маршрутизации сообщений](iot-hub-devguide-query-language.md).

## <a name="jobs-properties"></a>Свойства заданий

В таблице ниже содержится список свойств, которые можно использовать при выполнении запросов на задания и их результаты, а также описание этих свойств.

| Свойство | Описание |
| --- | --- |
| **jobId** |Идентификатор задания, указанный в приложении. |
| **startTime** |Время начала задания (ISO-8601), указанное в приложении. |
| **Завершения** |Дата завершения задания (ISO-8601), указанная в Центре Интернета вещей. Она допустима только после перехода задания в завершенное состояние. |
| **type** |Типы заданий: |
| | **счедулеупдатетвин**: задание, используемое для обновления набора требуемых свойств или тегов. |
| | **счедуледевицемесод**: задание, используемое для вызова метода устройства в наборе двойниковов устройства. |
| **status** |Текущее состояние задания. Возможные значения состояния: |
| | **pending** — задание запланировано и ожидает действий от службы заданий. |
| | **scheduled** — задание запланировано на будущее. |
| | **running** — задание сейчас активно. |
| | **canceled** — задание было отменено. |
| | **failed** — произошел сбой задания. |
| | **completed** — задание завершено. |
| **deviceJobStatistics** |Статистика задания. |
| | свойства **deviceJobStatistics** : |
| | **deviceJobStatistics.deviceCount**: число устройств в задании. |
| | **deviceJobStatistics.failedCount**: количество устройств, на которых произошел сбой задания. |
| | **deviceJobStatistics.succeededCount**: количество устройств, на которых задание выполнено успешно. |
| | **deviceJobStatistics.runningCount**: количество устройств, на которых сейчас выполняется задание. |
| | **deviceJobStatistics.pendingCount**: количество устройств, на которых сейчас ожидается выполнение задания. |

### <a name="additional-reference-material"></a>Дополнительные справочные материалы

Другие справочные статьи в руководстве разработчика для Центра Интернета вещей:

* В [справочнике по конечным точкам Центра Интернета вещей](iot-hub-devguide-endpoints.md) содержатся сведения о конечных точках, которые каждый центр Интернета вещей предоставляет для операций управления и среды выполнения.

* Статья [Руководство. Квоты и регулирование в Центре Интернета вещей](iot-hub-devguide-quotas-throttling.md) содержит сведения о квотах, применимых к службе Центра Интернета вещей, и ожидаемом поведении регулирования при использовании службы.

* В статье о [пакетах SDK для устройств и служб Azure IoT](iot-hub-devguide-sdks.md) указаны различные языковые пакеты SDK, которые можно использовать при разработке приложений для устройств и служб, взаимодействующих с Центром Интернета вещей.

* [Язык запросов Центра Интернета вещей для двойников устройств, заданий и маршрутизации сообщений](iot-hub-devguide-query-language.md) описывает языковые запросы Центра Интернета вещей. Используйте этот язык запросов для получения сведений о двойниках устройств и заданиях из Центра Интернета вещей.

* Статья [Взаимодействие с Центром Интернета вещей с помощью протокола MQTT](iot-hub-mqtt-support.md) содержит дополнительные сведения о поддержке протокола MQTT в Центре Интернета вещей.

## <a name="next-steps"></a>Дальнейшие действия

Чтобы применить некоторые основные понятия, описанные в этой статье, просмотрите следующие руководства по Центру Интернета вещей:

* [Планирование и трансляция заданий](iot-hub-node-node-schedule-jobs.md)