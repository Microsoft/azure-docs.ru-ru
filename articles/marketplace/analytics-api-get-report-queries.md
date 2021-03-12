---
title: Получение API запросов отчетов
description: Этот API используется для получения всех запросов, доступных для использования в коммерческих отчетах аналитики Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: e2be43e8402e5179fb62d810fe7b9f41e704c49d
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102584095"
---
# <a name="get-report-queries-api"></a>Получение API запросов отчетов

API получения отчетов о запросах получает все запросы, доступные для использования в отчетах. По умолчанию получает все системные и пользовательские запросы.

**Синтаксис запроса**

| **Метод** | **Универсальный код ресурса (URI) запроса** |
| --- | --- |
| GET | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledQueries?queryId={QueryID}&queryName={QueryName}&includeSystemQueries={include_system_queries}&includeOnlySystemQueries={include_only_system_queries}` |
|||

**Заголовок запроса**

| **Header** | **Тип** | **Описание** |
| --- | --- | --- |
| Авторизация | строка | Обязательный. Маркер доступа Azure Active Directory (Azure AD) в форме `Bearer <token>` |
| Content-Type | строка | `Application/JSON` |
||||

**Параметр пути**

Нет

**Параметр запроса**

| **Имя параметра** | **Тип** | **Обязательно** | **Описание** |
| --- | --- | --- | --- |
| `queryId` | Строка | Нет | Фильтр для получения сведений только о запросах с ИДЕНТИФИКАТОРом, заданным в аргументе |
| `queryName` | Строка | Нет | Фильтр для получения сведений только о запросах с именем, указанным в аргументе |
| `IncludeSystemQueries` | Логическое | Нет | Включить стандартные системные запросы в ответ |
| `IncludeOnlySystemQueries` | Логическое | Нет | Включать в ответ только системные запросы |
|||||

**Полезные данные запроса**

Нет

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
      "QueryId": "string",
      "Name": "string",
      "Description": "string",
      "Query": "string",
      "Type": "string",
      "User": "string",
      "CreatedTime": "string",
      "ModifiedTime": "string"
    }
  ],
  "TotalCount": 0,
  "Message": "string",
  "StatusCode": 0
}
```

**Словарь терминов**

В этой таблице описаны основные определения элементов в ответе.

| **Параметр** | **Описание** |
| --- | --- |
| `QueryId` | Уникальный UUID запроса |
| `Name` | Имя, присвоенное запросу во время создания запроса |
| `Description` | Описание, заданное при создании запроса |
| `Query` | Строка запроса отчета |
| `Type` | Задайте для значение `userDefined` для созданных пользователем запросов и `system` для предопределенных системных запросов. |
| `User` | Идентификатор пользователя, создавшего запрос |
| `CreatedTime` | Время создания запроса |
| `TotalCount` | Число наборов данных в массиве значений |
| `StatusCode` | Код результата. Возможные значения: 200, 400, 401, 403, 500 |
|||
