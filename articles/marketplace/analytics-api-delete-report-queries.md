---
title: API удаления запросов отчетов
description: Этот API используется для удаления пользовательских запросов к аналитике коммерческого рынка.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 4fc3479f1e35970a97684396a7a2e0c0c2582128
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102584165"
---
# <a name="delete-report-queries-api"></a>API удаления запросов отчетов

Этот API удаляет пользовательские запросы.

**Синтаксис запроса**

| **Метод** | **Универсальный код ресурса (URI) запроса** |
| --- | --- |
| DELETE | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledQueries/{queryId}` |

**Заголовок запроса**

| **Header** | **Тип** | **Описание** |
| --- | --- | --- |
| Авторизация | строка | Обязательный. Маркер доступа Azure Active Directory (Azure AD) в форме `Bearer <token>` |
| Content-Type | строка | `Application/JSON` |

**Параметр пути**

| **Имя параметра** | **Тип** | **Описание** |
| --- | --- | --- |
| `queryId` | string | Фильтр для получения сведений только о запросах с ИДЕНТИФИКАТОРом, заданным в этом аргументе |

**Параметр запроса**

Нет

**Полезные данные запроса**

Нет

**Словарь терминов**

Нет

**Ответ**

Полезные данные ответа структурированы следующим образом в формате JSON.

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

В этой таблице перечислены основные определения элементов в ответе.

| **Параметр** | **Описание** |
| --- | --- |
| `QueryId` | Уникальный UUID удаленного запроса. |
| `Name` | Имя удаленного запроса |
| `Description` | Описание удаленного запроса |
| `Query` | Строка запроса отчета об удаленном запросе |
| `Type` | Пользователяопределенные |
| `User` | Идентификатор пользователя, создавшего запрос |
| `CreatedTime` | Время создания запроса |
| `ModifiedTime` | NULL |
| `TotalCount` | Число наборов данных в массиве значений |
| `StatusCode` | Код результата. Возможные значения: 200, 400, 401, 403, 500 |
