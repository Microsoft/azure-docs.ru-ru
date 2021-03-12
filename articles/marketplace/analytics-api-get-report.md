---
title: Получить API отчета
description: Этот API используется для получения аналитических отчетов, которые были запланированы в центре партнеров.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 3383af447f40ea984bce9cbc956f22ee6c5af200
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102584046"
---
# <a name="get-report-api"></a>Получить API отчета

Этот API возвращает все отчеты, которые были запланированы.

**Синтаксис запроса**

| **Метод** | **Универсальный код ресурса (URI) запроса** |
| --- | --- |
| GET | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport?reportId={Report ID}&reportName={Report Name}&queryId={Query ID} ` |

**Заголовок запроса**

| **Header** | **Тип** | **Описание** |
| --- | --- | --- |
| Авторизация | строка | Обязательный. Маркер доступа Azure Active Directory (Azure AD) в форме `Bearer <token>` |
| Content-Type | строка | `Application/JSON` |

**Параметр пути**

Нет

**Параметр запроса**

| **Имя параметра** | **Обязательное** | **Тип** | **Описание** |
| --- | --- | --- | --- |
| `reportId` | Нет | строка | Фильтр для получения сведений только об отчетах с `reportId` указанным в этом аргументе |
| `reportName` | Нет | строка | Фильтр для получения сведений только об отчетах с `reportName` указанным в этом аргументе |
| `queryId` | Нет | Логическое | Включить стандартные системные запросы в ответ |

**Словарь терминов**

Нет

**Ответ**

Полезные данные ответа структурированы в формате JSON следующим образом:

Код ответа: 200, 400, 401, 403, 404, 500

Полезные данные ответа:

```json
{
  "Value": [
    {
      "ReportId": "string",
      "ReportName": "string",
      "Description": "string",
      "QueryId": "string",
      "Query": "string",
      "User": "string",
      "CreatedTime": "string",
      "ModifiedTime": "string",
      "StartTime": "string",
      "ReportStatus": "string",
      "RecurrenceInterval": 0,
      " RecurrenceCount": 0,
      "CallbackUrl": "string",
      "Format": "string"
    }
  ],
  "TotalCount": 0,
  "Message": "string",
  "StatusCode": 0
}
```

**Словарь терминов**

В этой таблице перечислены основные определения элементов в ответе.

| **Параметр** | **Описание** |
| --- | --- |
| `ReportId` | Уникальный UUID созданного отчета |
| `ReportName` | Имя, присвоенное отчету в полезных данных запроса |
| `Description` | Описание, заданное при создании отчета |
| `QueryId` | Идентификатор запроса, переданный во время создания отчета |
| `Query` | Текст запроса, который будет выполнен для этого отчета |
| `User` | Идентификатор пользователя, используемый для создания отчета |
| `CreatedTime` | Время создания отчета. Формат времени — гггг-мм-ddTHH: мм: ссZ |
| `ModifiedTime` | Время последнего изменения отчета. Формат времени — гггг-мм-ddTHH: мм: ссZ |
| `StartTime` | Начнется выполнение. Формат времени — гггг-мм-ddTHH: мм: ссZ |
| `ReportStatus` | Состояние выполнения отчета. Возможные значения: приостановлено, активно и неактивно. |
| `RecurrenceInterval` | Интервал повторения, указанный при создании отчета |
| `RecurrenceCount` | Число повторений, указанное при создании отчета |
| `CallbackUrl` | URL-адрес обратного вызова, указанный в запросе |
| `Format` | Формат файлов отчетов |
