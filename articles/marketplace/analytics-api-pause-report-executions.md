---
title: Приостановка API выполнения отчетов
description: Этот API используется для приостановки запланированного выполнения коммерческого отчета по анализу Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 39b535278fef42818f572631cfa1cb1f923930a6
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102584035"
---
# <a name="pause-report-executions-api"></a>Приостановка API выполнения отчетов

Этот API при выполнении приостанавливает запланированное выполнение отчетов.

**Синтаксис запроса**

| Метод | Универсальный код ресурса (URI) запроса |
| ------------ | ------------- |
| PUT | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport/pause/{Report ID}` |
|||

**Заголовок запроса**

| Заголовок | Type | Описание |
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

Код ответа: 200, 400, 401, 403, 404, 500 отклик ответа:

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
| `ReportId` | Универсальный уникальный идентификатор (UUID) удаленного отчета |
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
