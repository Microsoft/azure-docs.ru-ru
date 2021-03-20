---
title: Пользовательские схемы отслеживания для сообщений B2B
description: Создание настраиваемых схем отслеживания для отслеживания сообщений B2B в Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 01/01/2020
ms.openlocfilehash: c82f9cbfaf2e23ddaa5e4b05f4aac4795d3e16a9
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "76903054"
---
# <a name="create-custom-tracking-schemas-that-monitor-end-to-end-workflows-in-azure-logic-a"></a>Создание настраиваемых схем отслеживания, отслеживающих сквозные рабочие процессы в Azure Logic A

Azure Logic Apps имеет встроенные возможности отслеживания, которые можно включить для частей рабочего процесса. Однако можно настроить пользовательское отслеживание, которое регистрирует события от начала до конца рабочих процессов, например рабочие процессы, которые включают приложение логики, BizTalk Server, SQL Server или любой другой слой. В этой статье приведен настраиваемый код, который можно использовать в уровнях за пределами приложения логики.

## <a name="custom-tracking-schema"></a>Настраиваемая схема отслеживания

```json
{
   "sourceType": "",
   "source": {
      "workflow": {
         "systemId": ""
      },
      "runInstance": {
         "runId": ""
      },
      "operation": {
         "operationName": "",
         "repeatItemScopeName": "",
         "repeatItemIndex": ,
         "trackingId": "",
         "correlationId": "",
         "clientRequestId": ""
      }
   },
   "events": [
      {
         "eventLevel": "",
         "eventTime": "",
         "recordType": "",
         "record": {}
      }
   ]
}
```

| Свойство | Обязательно | Тип | Описание |
|----------|----------|------|-------------|
| sourceType | Да | Строка | Тип источника запуска с этими разрешенными значениями: `Microsoft.Logic/workflows` , `custom` |
| source | Да | String или JToken | Если исходный тип — `Microsoft.Logic/workflows` , сведения об источнике должны следовать этой схеме. Если тип источника — `custom` , то схема является JToken. |
| systemId | Да | Строка | Идентификатор системы приложения логики |
| runId | Да | Строка | Идентификатор запуска приложения логики |
| operationName | Да | Строка | Имя операции, например действие или триггер |
| repeatItemScopeName | Да | Строка | Повторение имени элемента, если действие находится внутри `foreach` `until` цикла или |
| repeatItemIndex | Да | Целое число | Указывает, что действие находится внутри `foreach` цикла или `until` и является повторяющимся индексом элемента. |
| trackingId | нет | Строка | ИДЕНТИФИКАТОР отслеживания для корреляции сообщений |
| correlationId | нет | Строка | Идентификатор корреляции для корреляции сообщений |
| clientRequestId | нет | Строка | Клиент может заполнить это свойство для корреляции сообщений. |
| eventLevel | Да | Строка | Уровень события |
| eventTime | Да | Дата/время | Время события в формате UTC: *гггг-мм-ddThh: mm: SS. 00000z)* |
| recordType | Да | Строка | Тип записи Track только с этим разрешенным значением: `custom` |
| запись | Да | JToken | Пользовательский тип записи только с форматом JToken |
|||||

## <a name="b2b-protocol-tracking-schemas"></a>Схемы отслеживания для протоколов B2B

Сведения о схемах отслеживания для протоколов B2B см. в следующих статьях:

* [Схемы отслеживания AS2](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)
* [Схемы отслеживания X12](logic-apps-track-integration-account-x12-tracking-schema.md)

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [мониторинге сообщений B2B с помощью журналов Azure Monitor](../logic-apps/monitor-b2b-messages-log-analytics.md)