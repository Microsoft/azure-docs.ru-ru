---
title: Функции в запросах журнала Azure Monitor | Документация Майкрософт
description: В этой статье объясняется, как использовать функции для вызова запроса из другого запроса журнала в Azure Monitor.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/31/2020
ms.openlocfilehash: 9b2466bbe061dc147d020104b13da197840fe40a
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102030789"
---
# <a name="using-functions-in-azure-monitor-log-queries"></a>Использование функций в запросах журнала Azure Monitor

Чтобы использовать запрос журнала с другим запросом, сохраните его как функцию. Это позволяет упростить сложные запросы, разбивая их на части, и повторно использовать общий код с несколькими запросами.

## <a name="create-a-function"></a>Создание функции

Создайте функцию с помощью Log Analytics на портале Azure. Для этого щелкните **Сохранить** и укажите сведения из указанной ниже таблицы.

| Параметр | Описание |
|:---|:---|
| Имя           | Отображаемое имя для запроса в **обозревателе запросов**. |
| Сохранить как        | Компонент |
| Псевдоним функции | Короткое имя функции в других запросах. Не может содержать пробелы и должно быть уникальным. |
| Категория       | Категория для организации сохраненных запросов и функций в **обозревателе запросов**. |




## <a name="use-a-function"></a>Использование функции
Используйте функцию, включив ее псевдоним в другой запрос. Она может использоваться как любая другая таблица.

## <a name="function-parameters"></a>Параметры функции 
В функцию можно добавить параметры, чтобы при ее вызове можно было указать значения для определенных переменных. Единственным способом создания функции с параметрами является использование шаблона диспетчер ресурсов. Пример см. [в разделе примеры шаблонов диспетчер ресурсов для запросов журналов в Azure Monitor](./resource-manager-log-queries.md#parameterized-function) .

## <a name="example"></a>Пример
Приведенный ниже пример запроса возвращает все отсутствующие обновления безопасности, о которых сообщалось в последний день. Сохраните этот запрос как функцию с псевдонимом _security_updates_last_day_. 

```Kusto
Update
| where TimeGenerated > ago(1d) 
| where Classification == "Security Updates" 
| where UpdateState == "Needed"
```

Создайте другой запрос, который ссылается на функцию _security_updates_last_day_ для поиска нужных обновлений безопасности, связанных с SQL.

```Kusto
security_updates_last_day | where Title contains "SQL"
```

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с дополнительными уроками о написании запросов журнала Azure Monitor.

- [Работа со строками](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#string-operations)
- [Работа со значениями даты и времени](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#date-and-time-operations)
- [Статистические функции в запросах Log Analytics](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#aggregations)
- [Расширенные статистические функции в запросах Azure Log Analytics](/azure/data-explorer/write-queries#advanced-aggregations)
- [Работа с JSON и структурами данных в запросах Log Analytics](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#json-and-data-structures)
- [Joins](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#joins)
- [Создание графиков](/azure/data-explorer/kusto/query/samples?&pivots=azuremonitor#charts)