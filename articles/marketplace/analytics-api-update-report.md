---
title: API обновления отчета
description: Используйте этот API для отчета о параметрах для отчетов по аналитике коммерческих рынков.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 38680eb291417ded4c2f93539e8d1ae091b1d560
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102583928"
---
# <a name="update-report-api"></a>API обновления отчета

Этот API помогает изменить параметр отчета.

**Синтаксис запроса**

| Метод | Универсальный код ресурса (URI) запроса |
| ------------ | ------------- |
| PUT | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport/{Report ID}` |
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

**Полезные данные запроса**

```json
{
  "ReportName": "string",
  "Description": "string",
  "StartTime": "string",
  "RecurrenceInterval": 0,
  "RecurrenceCount": 0,
  "Format": "string",
  "CallbackUrl": "string"
}
```

**Словарь терминов**

В этой таблице перечислены основные определения элементов в полезных данных запроса.

| Параметр | Обязательно | Описание | Допустимые значения |
| ------------ | ------------- | ------------- | ------------- |
| `ReportName` | Да | Имя, присваиваемое отчету | Строка |
| `Description` | Нет | Описание созданного отчета | строка |
| `StartTime` | Да | Метка времени, после которой начнется создание отчета | Строка |
| `RecurrenceInterval` | Нет | Периодичность создания отчета в часах. Минимальное значение равно 4 | Целое число |
| `RecurrenceCount` | Нет | Число создаваемых отчетов. Значение по умолчанию неопределенное | Целое число |
| `Format` | Да | Формат файла экспортируемого файла. Значение по умолчанию — CSV. | CSV/TSV |
| `CallbackUrl` | Да | URL-адрес обратного вызова HTTPS, вызываемый при создании отчета | строка |
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
| `ReportId` | Универсальный уникальный идентификатор (UUID) обновляемого отчета |
| `ReportName` | Имя, присвоенное отчету в полезных данных запроса |
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
