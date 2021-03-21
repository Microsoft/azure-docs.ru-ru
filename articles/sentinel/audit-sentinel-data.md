---
title: Аудит запросов и действий Sentinel Azure | Документация Майкрософт
description: В этой статье описывается, как выполнять аудит запросов и действий, выполняемых в Azure Sentinel.
services: sentinel
documentationcenter: na
author: batamig
manager: rkarlin
editor: ''
ms.assetid: 9b4c8e38-c986-4223-aa24-a71b01cb15ae
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/03/2021
ms.author: bagol
ms.openlocfilehash: a8ea32d84da521c8a1af926c6cb5e26bc2738de2
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102054983"
---
# <a name="audit-azure-sentinel-queries-and-activities"></a>Аудит запросов и действий Azure Sentinel

В этой статье описывается, как просмотреть данные аудита для выполнения запросов и действий, выполняемых в рабочей области "Sentinel" Azure, например для внутренних и внешних требований к соответствию в рабочей области "операции безопасности" (SOC).

Azure Sentinel предоставляет доступ к следующим:

- Таблица **AzureActivity** , в которой содержатся сведения обо всех действиях, выполненных в Azure Sentinel, таких как изменение правил оповещений. Таблица **AzureActivity** не регистрирует определенные данные запроса. Дополнительные сведения см. в статье [Аудит с помощью журналов действий Azure](#auditing-with-azure-activity-logs).

- Таблица **лакуерилогс** , в которой содержатся подробные сведения о запросах, выполняемых в log Analytics, включая запросы, выполняемые из Azure Sentinel. Дополнительные сведения см. в разделе [Audit with лакуерилогс](#auditing-with-laquerylogs).

> [!TIP]
> В дополнение к ручным запросам, описанным в этой статье, Azure Sentinel предоставляет встроенную книгу, помогающую выполнять аудит действий в среде SOC.
>
> В области **книги** Sentinel Azure найдите книгу **аудита рабочей области** .



## <a name="auditing-with-azure-activity-logs"></a>Аудит с помощью журналов действий Azure

Журналы аудита Sentinel Azure хранятся в [журналах действий Azure](../azure-monitor/essentials/platform-logs-overview.md), где таблица **AzureActivity** включает все действия, выполненные в рабочей области Sentinel.

При аудите действий в среде SOC с помощью Azure Sentinel можно использовать таблицу **AzureActivity** .

**Для запроса таблицы AzureActivity**:

1. Подключите источник данных [действий Azure](connect-azure-activity.md) , чтобы начать потоковую передачу событий аудита в новую таблицу на экране **журналов** с именем AzureActivity.

1. Затем запросите данные с помощью ККЛ, как в любой другой таблице.

    Таблица **AzureActivity** содержит данные из многих служб, включая метку Azure. Чтобы отфильтровать только данные из Azure Sentinel, запустите запрос, используя следующий код:

    ```kql
     AzureActivity
    | where OperationNameValue startswith "MICROSOFT.SECURITYINSIGHTS"
    ```

    Например, чтобы узнать, кто был последним пользователем для изменения определенного правила аналитики, используйте следующий запрос (заменив `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` идентификатором правила, которое требуется проверить):

    ```kusto
    AzureActivity
    | where OperationNameValue startswith "MICROSOFT.SECURITYINSIGHTS/ALERTRULES/WRITE"
    | where Properties contains "alertRules/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    | project Caller , TimeGenerated , Properties
    ```

Добавьте дополнительные параметры в запрос для дальнейшего изучения таблицы **азуреактивитиес** в зависимости от того, что нужно сообщить. В следующих разделах представлены другие примеры запросов для использования при аудите с использованием данных таблицы **AzureActivity** . 

Дополнительные сведения см. [в разделе данные Sentinel Azure, включаемые в журналы действий Azure](#azure-sentinel-data-included-in-azure-activity-logs).

### <a name="find-all-actions-taken-by-a-specific-user-in-the-last-24-hours"></a>Поиск всех действий, выполненных конкретным пользователем за последние 24 часа

В следующем запросе таблицы **AzureActivity** перечислены все действия, выполняемые конкретным пользователем Azure AD за последние 24 часа.

```kql
AzureActivity
| where OperationNameValue contains "SecurityInsights"
| where Caller == "[AzureAD username]"
| where TimeGenerated > ago(1d)
```

### <a name="find-all-delete-operations"></a>Найти все операции удаления

В следующем запросе таблицы **AzureActivity** перечислены все операции удаления, выполненные в рабочей области "Sentinel" Azure.

```kql
AzureActivity
| where OperationNameValue contains "SecurityInsights"
| where OperationName contains "Delete"
| where ActivityStatusValue contains "Succeeded"
| project TimeGenerated, Caller, OperationName
``` 


### <a name="azure-sentinel-data-included-in-azure-activity-logs"></a>Данные Sentinel Azure, включаемые в журналы действий Azure
 
Журналы аудита Sentinel Azure хранятся в [журналах действий Azure](../azure-monitor/essentials/platform-logs-overview.md)и включают следующие типы данных:

|Операция  |Типы сведений  |
|---------|---------|
|**Создано**     |Правила оповещения <br> Комментарии к регистру <br>Комментарии к инциденту <br>Сохраненные поисковые запросы<br>ватчлистс    <br>Workbooks     |
|**Удалено**     |Правила оповещения <br>Закладки <br>Соединители данных <br>Инциденты <br>Сохраненные поисковые запросы <br>Параметры <br>Отчеты по аналитике угроз <br>ватчлистс      <br>Workbooks <br>Рабочий процесс  |
|**Обновленные возможности**     |  Правила оповещения<br>Закладки <br> Случаи <br> Соединители данных <br>Инциденты <br>Комментарии к инциденту <br>Отчеты по аналитике угроз <br> Workbooks <br>Рабочий процесс       |
|     |         |

Вы также можете использовать журналы действий Azure для проверки прав и лицензий пользователей. 

Например, в следующей таблице перечислены выбранные операции, найденные в журналах действий Azure, с конкретным ресурсом, из которого извлекаются данные журнала.

|Имя операции|    Тип ресурса|
|----|----|
|Создать или обновить книгу  |Microsoft. Insights/книги|
|Удалить книгу    |Microsoft. Insights/книги|
|Задание рабочего процесса   |Microsoft.Logic/workflows|
|Удалить рабочий процесс    |Microsoft.Logic/workflows|
|Создание сохраненного поискового запроса    |Microsoft. OperationalInsights/workspaces/Саведсеарчес|
|Удалить сохраненный поиск    |Microsoft. OperationalInsights/workspaces/Саведсеарчес|
|Обновление правил генерации оповещений |Microsoft. Секуритинсигхтс/alertRules|
|Удаление правил генерации оповещений |Microsoft. Секуритинсигхтс/alertRules|
|Обновление действий ответа для правила генерации оповещений |Microsoft. Секуритинсигхтс/alertRules/действия|
|Удаление действий ответа на правило генерации оповещений |Microsoft. Секуритинсигхтс/alertRules/действия|
|Обновить закладки   |Microsoft. Секуритинсигхтс/закладки|
|Удалить закладки   |Microsoft. Секуритинсигхтс/закладки|
|Обновление вариантов   |Microsoft. Секуритинсигхтс/cases|
|Исследование вариантов обновления  |Microsoft. Секуритинсигхтс/cases/расследования|
|Создание комментариев к вариантам   |Microsoft. Секуритинсигхтс/cases/Comments|
|Обновление соединителей данных |Microsoft. Секуритинсигхтс/Connects|
|Удаление соединителей данных |Microsoft. Секуритинсигхтс/Connects|
|Параметры обновления    |Microsoft. Секуритинсигхтс/параметры|
| | |

Дополнительные сведения см. в статье [схема событий журнала действий Azure](/azure/azure-monitor/essentials/activity-log-schema).


## <a name="auditing-with-laquerylogs"></a>Аудит с помощью Лакуерилогс

В таблице **лакуерилогс** содержатся сведения о запросах журнала, выполняемых в log Analytics. Так как Log Analytics используется как базовое хранилище данных Sentinel Azure, вы можете настроить систему для получения данных Лакуерилогс в рабочей области Azure Sentinel.

Данные Лакуерилогс включают следующие сведения:

- При выполнении запросов
- Кто запустил запросы в Log Analytics
- Какое средство использовалось для выполнения запросов в Log Analytics, например в Azure Sentinel
- Сами тексты запросов
- Данные о производительности при каждом выполнении запроса

> [!NOTE]
> - Таблица **лакуерилогс** содержит только те запросы, которые были выполнены в колонке журналы в Azure Sentinel. Он не включает запросы, выполняемые правилами аналитики по расписанию, с помощью **графа расследования** **или страницы Поиск** маркеров Azure.
> - Может возникнуть небольшая задержка между временем выполнения запроса и заполнением данных в таблице **лакуерилогс** . Мы рекомендуем подождать около 5 минут, чтобы запросить таблицу **лакуерилогс** для данных аудита.
>


**Для запроса таблицы лакуерилогс**:

1. Таблица **лакуерилогс** по умолчанию не включена в рабочей области log Analytics. Чтобы использовать данные **лакуерилогс** при аудите в Azure Sentinel, сначала включите **лакуерилогс** в области **параметров диагностики** рабочей области log Analytics.

    Дополнительные сведения см. [в разделе Аудит запросов в журналах Azure Monitor](/azure/azure-monitor/logs/query-audit).


1. Затем запросите данные с помощью ККЛ, как в любой другой таблице.

    Например, в следующем запросе показано количество запросов, выполненных за последнюю неделю, за каждый день.

    ```kql
    LAQueryLogs
    | where TimeGenerated > ago(7d)
    | summarize events_count=count() by bin(TimeGenerated, 1d)
    ```

В следующих разделах приведены дополнительные примеры запросов для выполнения в таблице **лакуерилогс** при аудите действий в среде SoC с помощью Sentinel Azure.

### <a name="the-number-of-queries-run-where-the-response-wasnt-ok"></a>Число запусков запросов, для которых ответ не был "ОК"

В следующем запросе таблицы **лакуерилогс** показано количество запущенных запросов, где было получено любое значение, кроме HTTP-ответа от **200 ОК** . Например, это число будет включать запросы, которые не удалось запустить.

```kql
LAQueryLogs
| where ResponseCode != 200 
| count 
```

### <a name="show-users-for-cpu-intensive-queries"></a>Отображение пользователей для запросов с интенсивным использованием ЦП

В следующем запросе таблицы **лакуерилогс** перечислены пользователи, которые запускали большинство ресурсоемких запросов, в зависимости от используемого ЦП и длительности выполнения запроса.

```kql
LAQueryLogs
|summarize arg_max(StatsCPUTimeMs, *) by AADClientId
| extend User = AADEmail, QueryRunTime = StatsCPUTimeMs
| project User, QueryRunTime, QueryText
| order by QueryRunTime desc
```

### <a name="show-users-who-ran-the-most-queries-in-the-past-week"></a>Отображение пользователей, которые запускали большинство запросов за последнюю неделю

В следующем запросе таблицы **лакуерилогс** перечислены пользователи, которые запускали большинство запросов за последнюю неделю.

```kql
LAQueryLogs
| where TimeGenerated > ago(7d)
| summarize events_count=count() by AADEmail
| extend UserPrincipalName = AADEmail, Queries = events_count
| join kind= leftouter (
    SigninLogs)
    on UserPrincipalName
| project UserDisplayName, UserPrincipalName, Queries
| summarize arg_max(Queries, *) by UserPrincipalName
| sort by Queries desc
```

## <a name="configuring-alerts-for-azure-sentinel-activities"></a>Настройка оповещений для действий Sentinel Azure

Для создания упреждающих оповещений может потребоваться использовать ресурсы аудита Sentinel Azure.

Например, если в рабочей области Sentinel Azure есть конфиденциальные таблицы, используйте следующий запрос, чтобы уведомить вас каждый раз, когда эти таблицы будут опрошены:

```kql
LAQueryLogs
| where QueryText contains "[Name of sensitive table]"
| where TimeGenerated > ago(1d)
| extend User = AADEmail, Query = QueryText
| project User, Query
```


## <a name="next-steps"></a>Следующие шаги

В Azure Sentinel используйте книгу **аудита рабочей области** для аудита действий в среде SoC.

Дополнительные сведения см. в разделе [учебник. Визуализация и мониторинг данных](tutorial-monitor-your-data.md).
