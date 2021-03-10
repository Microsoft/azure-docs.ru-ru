---
title: Удаление API отчета
description: Используйте этот API, чтобы удалить все записи отчета и выполнения отчетов для коммерческих отчетов Аналитики Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 7c39f8bc0db44f1d8aa885969ca09d90b0dcd332
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102584115"
---
# <a name="delete-report-api"></a>Удаление API отчета

При выполнении этот API удаляет все записи отчетов и выполнения отчетов.

**Синтаксис запроса**

| Метод | Универсальный код ресурса (URI) запроса |
| ------------ | ------------- |
| DELETE | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport/{Report ID}` |
|||

**Заголовок запроса**

| Заголовок | Type | Описание |
| ------------ | ------------- | ------------- |
| Авторизация | строка | Обязательный. Маркер доступа Azure AD в виде `Bearer <token>` |
| Тип содержимого | строка | `Application/JSON` |
||||

**Параметр пути**

Нет

**Параметр запроса**

| Имя параметра | Обязательно | строка | Описание |
| ------------ | ------------- | ------------- | ------------- |
| `reportId` | Да | строка | Идентификатор изменяемого отчета |
|||||

**Словарь терминов**

Нет

**Ответ**

Полезные данные ответа структурированы следующим образом:

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
| `ReportId` | Уникальный UUID удаленного отчета |
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
