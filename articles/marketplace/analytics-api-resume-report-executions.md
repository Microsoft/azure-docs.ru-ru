---
title: API возобновления выполнения отчетов
description: Этот API используется для возобновления запланированного выполнения приостановленного отчета аналитики коммерческого рынка.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 4a11783b28352cb62c5a3c0d38e45dcdc47a8d86
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102583939"
---
# <a name="resume-report-executions-api"></a>API возобновления выполнения отчетов

Этот API при выполнении возобновляет запланированное выполнение приостановленного отчета аналитики коммерческого рынка.

**Синтаксис запроса**

| Метод | Универсальный код ресурса (URI) запроса |
| ------------ | ------------- |
| PUT | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport/resume/{reportId}` |
|||

**Заголовок запроса**

| Header | Type | Описание |
| ------------ | ------------- | ------------- |
| Авторизация | строка | Обязательный. Маркер доступа Azure Active Directory (Azure AD) в форме `Bearer <token>` |
| Content-Type | строка | `Application/JSON` |
||||

**Параметр пути**

Нет

**Параметр запроса**

| Имя параметра | Обязательно | Тип | Описание |
| ------------ | ------------- | ------------- | ------------- |
| `reportId` | Да | строка | Идентификатор изменяемого отчета |
|||||

**Словарь терминов**

Нет

**Ответ**

Полезные данные ответа структурированы следующим образом в формате JSON:

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
      "RecurrenceCount": 0,
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

| Параметр | Описание |
| ------------ | ------------- |
| `ReportId` | Универсальный уникальный идентификатор (UUID) возобновляемого отчета |
| `ReportName` | Имя, присвоенное отчету во время создания |
| `Description` | Описание, заданное при создании отчета |
| `QueryId` | Идентификатор запроса, переданный во время создания отчета |
| `Query` | Текст запроса, который будет выполнен для этого отчета |
| `User` | Идентификатор пользователя, используемый для создания отчета |
| `CreatedTime` | Время создания отчета. Формат времени — гггг-мм-ddTHH: мм: ссZ |
| `ModifiedTime` | Время последнего изменения отчета. Формат времени — гггг-мм-ddTHH: мм: ссZ |
| `StartTime` | Время, когда начнется выполнение отчета. Формат времени — гггг-мм-ddTHH: мм: ссZ |
| `ReportStatus` | Состояние выполнения отчета. Возможные значения: приостановлено, активно и неактивно. |
| `RecurrenceInterval` | Интервал повторения, указанный при создании отчета |
| `RecurrenceCount` | Число повторений, указанное при создании отчета |
| `CallbackUrl` | URL-адрес обратного вызова, указанный в запросе |
| `Format` | Формат файлов отчетов |
|||
