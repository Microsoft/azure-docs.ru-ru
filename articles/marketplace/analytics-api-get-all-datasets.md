---
title: API получения всех наборов данных
description: Этот API используется для получения всех доступных наборов данных для коммерческой аналитики Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 23aac2c94ffd909ca132cbc481998b9eda317156
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102583942"
---
# <a name="get-all-datasets-api"></a>API получения всех наборов данных

API получения всех наборов данных получает все доступные наборы данных. Наборы данных. список таблиц, столбцов, метрик и временных диапазонов.

**Синтаксис запроса**

| **Метод** | **Универсальный код ресурса (URI) запроса** |
| --- | --- |
| GET | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledDataset?datasetName={Dataset Name}` |

**Заголовок запроса**

| **Header** | **Тип** | **Описание** |
| --- | --- | --- |
| Авторизация | строка | Обязательный. Маркер доступа Azure Active Directory (Azure AD) в форме `Bearer <token>` |
| Content-Type | строка | `Application/JSON` |

**Параметр пути**

Нет

**Параметр запроса**

| **Имя параметра** | **Тип** | **Обязательно** | **Описание** |
| --- | --- | --- | --- |
| `datasetName` | Строка | Нет | Фильтр для получения сведений только об одном наборе данных |

**Полезные данные запроса**

Нет

**Словарь терминов**

Нет

**Ответ**

Полезные данные ответа структурированы следующим образом:

Код ответа: 200, 400, 401, 403, 404, 500

Пример полезных данных ответа:

```json
{
   "Value":[
      {
         "DatasetName ":"string",
         "SelectableColumns":[
            "string"
         ],
         "AvailableMetrics":[
            "string"
         ],
         "AvailableDateRanges ":[
            "string"
         ]
      }
   ],
   "TotalCount":int,
   "Message":"<Error Message>",
   "StatusCode": int
}
```

**Словарь терминов**

В этой таблице определяются ключевые элементы в ответе:

| **Параметр** | **Описание** |
| --- | --- |
| `DatasetName` | Имя набора данных, определяемого этим объектом массива |
| `SelectableColumns` | Необработанные столбцы, которые могут быть указаны в столбцах SELECT |
| `AvailableMetrics` | Имена столбцов агрегирования и метрики, которые могут быть указаны в столбцах SELECT |
| `AvailableDateRanges` | Диапазон дат, который может использоваться в запросах отчета для набора данных |
| `NextLink` | Ссылка на следующую страницу, если данные разбиты на страницы |
| `TotalCount` | Число наборов данных в массиве значений |
| `StatusCode` | Код результата. Возможные значения: 200, 400, 401, 403, 500 |
