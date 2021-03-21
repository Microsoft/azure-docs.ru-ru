---
title: Пересылка данных задания службы автоматизации Azure в журналы Azure Monitor
description: В этой статье рассказывается, как отправлять данные о состоянии задания и потоки заданий runbook в журналы Azure Monitor.
services: automation
ms.subservice: process-automation
ms.date: 09/02/2020
ms.topic: conceptual
ms.openlocfilehash: 4199c5576662eee1dd6cedc388440a71e21f8b74
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100581208"
---
# <a name="forward-azure-automation-job-data-to-azure-monitor-logs"></a>Пересылка данных задания службы автоматизации Azure в журналы Azure Monitor

Служба автоматизации Azure может отправлять состояние задания runbook и потоки заданий в рабочую область Log Analytics. Этот процесс не предполагает связывания с рабочей областью и является полностью независимым. На портале Azure или с помощью PowerShell можно просмотреть журналы заданий и потоки заданий для отдельных заданий. Это дает возможность выполнять простые исследования. С помощью журналов Azure Monitor вы можете:

* получить информацию о состоянии заданий службы автоматизации;
* активировать отправку электронного сообщения или оповещения в соответствии с состоянием задания runbook (например, сбой или приостановка);
* создавать сложные запросы для потоков заданий;
* коррелировать задания и учетные записи службы автоматизации;
* с помощью пользовательских представлений и поисковых запросов визуализировать результаты выполнения runbook, состояние задания runbook и другие ключевые индикаторы или метрики.

## <a name="prerequisites"></a>Предварительные требования

Чтобы начать отправку журналов службы автоматизации в журналы Azure Monitor, необходимо следующее:

* Последний выпуск [Azure PowerShell](/powershell/azure/).

* Рабочая область Log Analytics и ее идентификатор ресурса. Дополнительные сведения см. в статье [Начало работы с журналами Azure Monitor](../azure-monitor/overview.md).

* Идентификатор ресурса учетной записи службы автоматизации Azure.

## <a name="how-to-find-resource-ids"></a>Как найти идентификаторы ресурсов

1. Выполните следующую команду, чтобы получить идентификатор ресурса для учетной записи службы автоматизации Azure:

    ```powershell-interactive
    # Find the ResourceId for the Automation account
    Get-AzResource -ResourceType "Microsoft.Automation/automationAccounts"
    ```

2. Скопируйте значение **ResourceId**.

3. Используйте следующую команду, чтобы найти идентификатор ресурса для рабочей области Log Analytics:

    ```powershell-interactive
    # Find the ResourceId for the Log Analytics workspace
    Get-AzResource -ResourceType "Microsoft.OperationalInsights/workspaces"
    ```

4. Скопируйте значение **ResourceId**.

Чтобы получить результаты из определенной группы ресурсов, включите `-ResourceGroupName` параметр. Дополнительные сведения см. в разделе [Get-азресаурце](/powershell/module/az.resources/get-azresource).

Если в выходных данных предыдущих команд указано несколько учетных записей службы автоматизации или рабочих областей, вы можете выяснить имя и другие свойства для получения полного идентификатора ресурса учетной записи службы автоматизации, выполнив следующие действия:

1. Войдите на [портал Azure](https://portal.azure.com).
1. На портале Azure выберите учетную запись службы автоматизации на странице **Учетные записи службы автоматизации**.
1. На странице выбранной учетной записи службы автоматизации в разделе **Настройки учетной записи** выберите пункт **Свойства**.
1. На странице **Свойства** найдите сведения, показанные ниже.

    ![Свойства учетной записи службы автоматизации](media/automation-manage-send-joblogs-log-analytics/automation-account-properties.png).

## <a name="configure-diagnostic-settings"></a>Настройка параметров диагностики

Параметры диагностики службы автоматизации поддерживают перенаправление следующих журналов платформы и данных метрик:

* JobLogs
* JobStreams
* дскнодестатус
* Метрики — всего заданий, всего запущенных компьютеров развертывания обновлений, всего запусков развертывания обновлений

Чтобы начать отправку журналов автоматизации в журналы Azure Monitor, ознакомьтесь с разделом [Создание параметров диагностики](../azure-monitor/essentials/diagnostic-settings.md) , чтобы понять, какие функции и методы доступны для настройки параметров диагностики для отправки журналов платформы.

## <a name="azure-monitor-log-records"></a>Записи журнала Azure Monitor

Диагностика службы автоматизации Azure создает в журналах Azure Monitor записи двух типов с тегом `AzureDiagnostics`. В таблицах следующих разделов приведены примеры записей, создаваемых службой автоматизации Azure, и типы данных, которые отображаются в результатах поиска по журналам.

### <a name="job-logs"></a>Журналы заданий

| Свойство | Описание |
| --- | --- |
| TimeGenerated |Дата и время выполнения задания Runbook. |
| RunbookName_s |Имя Runbook. |
| Caller_s |Вызывающая сторона, которая запустила эту операцию. Допустимые значения: электронный адрес или system для запланированных заданий. |
| Tenant_g | GUID, идентифицирующий клиент для вызывающего объекта. |
| JobId_g |GUID, идентифицирующий задание runbook. |
| ResultType |Состояние задания Runbook. Возможны следующие значения:<br>Новое<br>- Создано<br>Started<br>- Остановлена<br>Приостановлено<br>Сбой<br>Завершено |
| Категория | Классификация типа данных. Для службы автоматизации значением является JobLogs. |
| OperationName | Тип операции, выполняемой в Azure. Для службы автоматизации значением является Job. |
| Ресурс | Имя учетной записи службы автоматизации |
| SourceSystem | Система, которую используют журналы Azure Monitor для сбора данных. Для системы диагностики Azure здесь всегда указано значение "Azure". |
| ResultDescription |Состояние результата задания runbook. Возможны следующие значения:<br>— Job is started;<br>— Job Failed;<br>- Job Completed. |
| CorrelationId |GUID корреляции для задания runbook. |
| ResourceId |Указывает идентификатор ресурса для учетной записи службы автоматизации Azure, к которой относится этот runbook. |
| SubscriptionId | GUID подписки Azure для учетной записи службы автоматизации. |
| ResourceGroup | Имя группы ресурсов для учетной записи службы автоматизации. |
| ResourceProvider | Поставщик ресурса. Имеет значение MICROSOFT.AUTOMATION. |
| ResourceType | Тип ресурса. Имеет значение AUTOMATIONACCOUNTS. |

### <a name="job-streams"></a>Потоки заданий
| Свойство | Описание |
| --- | --- |
| TimeGenerated |Дата и время выполнения задания Runbook. |
| RunbookName_s |Имя Runbook. |
| Caller_s |Вызывающая сторона, которая запустила эту операцию. Допустимые значения: электронный адрес или system для запланированных заданий. |
| StreamType_s |Тип потока задания. Возможны следующие значения:<br>ход выполнения<br>Выходные данные<br>Предупреждение<br>error<br>debug<br>- Verbose. |
| Tenant_g | GUID, идентифицирующий клиент для вызывающего объекта. |
| JobId_g |GUID, идентифицирующий задание runbook. |
| ResultType |Состояние задания Runbook. Возможны следующие значения:<br>- In Progress |
| Категория | Классификация типа данных. Для службы автоматизации значением является JobStreams. |
| OperationName | Тип операции, выполняемой в Azure. Для службы автоматизации значением является Job. |
| Ресурс | Имя учетной записи службы автоматизации. |
| SourceSystem | Система, которую используют журналы Azure Monitor для сбора данных. Для системы диагностики Azure здесь всегда указано значение "Azure". |
| ResultDescription |Описание, в котором указан выходной поток этого runbook. |
| CorrelationId |GUID корреляции для задания runbook. |
| ResourceId |Указывает идентификатор ресурса для учетной записи службы автоматизации Azure, к которой относится этот runbook. |
| SubscriptionId | GUID подписки Azure для учетной записи службы автоматизации. |
| ResourceGroup | Имя группы ресурсов для учетной записи службы автоматизации. |
| ResourceProvider | Поставщик ресурса. Имеет значение MICROSOFT.AUTOMATION. |
| ResourceType | Тип ресурса. Имеет значение AUTOMATIONACCOUNTS. |

## <a name="view-automation-logs-in-azure-monitor-logs"></a>Просмотр журналов службы автоматизации в журналах Azure Monitor

Теперь, когда вы начали отправлять потоки заданий службы автоматизации в журналы Azure Monitor, давайте посмотрим, что можно делать с этими журналами в Azure Monitor журналах.

Чтобы просмотреть журналы, выполните следующий запрос: `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION"`

### <a name="send-an-email-when-a-runbook-job-fails-or-suspends"></a>Отправка электронного сообщения при сбое или приостановке задания Runbook

В следующих шагах показано, как настроить в Azure Monitor оповещения о том, что с заданием runbook что-то пошло не так.

Чтобы создать правило генерации оповещений, для начала создайте поиск по журналам для записей заданий runbook, по которым вам требуется оповещение. Щелкните кнопку **Оповещение**, чтобы создать и настроить правило генерации оповещений.

1. На странице обзорных сведений о рабочей области Log Analytics щелкните **Просмотр журналов**.

2. Создайте запрос поиска по журналам для нужного оповещения, введя следующий текст в поле запроса: `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and (ResultType == "Failed" or ResultType == "Suspended")`<br><br>Вы также можете использовать группирование по имени runbook с помощью `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and (ResultType == "Failed" or ResultType == "Suspended") | summarize AggregatedValue = count() by RunbookName_s`

   Если вы настроили для рабочей области журналы из более чем одной учетной записи службы автоматизации или подписки, то можете группировать оповещения по подписке или учетной записи службы автоматизации. Имя учетной записи службы автоматизации можно найти в поле `Resource` в результатах поиска `JobLogs`.

3. Чтобы открыть экран **Создание правила**, щелкните **Новое правило генерации оповещений** в верхней части страницы. Дополнительные сведения о параметрах настройки оповещения см. в статье [Оповещения журнала в Azure Monitor. Интерфейс оповещений](../azure-monitor/alerts/alerts-unified-log.md).

### <a name="find-all-jobs-that-have-completed-with-errors"></a>Поиск всех заданий, завершенных с ошибками

Помимо оповещений о сбоях можно узнать, когда задание Runbook вызывает устранимую ошибку. В этих случаях PowerShell создает поток ошибок, но устранимые ошибки не приводят к приостановке или сбою задания.

1. В рабочей области Log Analytics щелкните **Журналы**.

2. В поле поискового запроса введите `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and StreamType_s == "Error" | summarize AggregatedValue = count() by JobId_g`.

3. Нажмите кнопку **Поиск**.

### <a name="view-job-streams-for-a-job"></a>Просмотр потоков заданий для задания

При отладке задания вам могут пригодиться потоки заданий. Следующий запрос показывает все потоки для одного задания с идентификатором GUID `2ebd22ea-e05e-4eb9-9d76-d73cbd4356e0` :

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and JobId_g == "2ebd22ea-e05e-4eb9-9d76-d73cbd4356e0"
| sort by TimeGenerated asc
| project ResultDescription
```

### <a name="view-historical-job-status"></a>Просмотр хронологии состояния задания

Наконец, вам может понадобиться визуализировать временную линию по журналу заданий. Для поиска для поиска состояния заданий по прошествии времени можно использовать приведенный ниже запрос.

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and ResultType != "started"
| summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h)
```

![Диаграмма хронологии состояния задания в Log Analytics](media/automation-manage-send-joblogs-log-analytics/historical-job-status-chart.png)

### <a name="filter-job-status-output-converted-into-a-json-object"></a>Выходные данные состояния задания фильтрации, преобразованные в объект JSON

Недавно мы изменили поведение записи данных журнала автоматизации в `AzureDiagnostics` таблицу в службе log Analytics, где они больше не разбивают свойства JSON на отдельные поля. Если вы настроили модуль Runbook для форматирования объектов в выходном потоке в формате JSON в виде отдельных столбцов, необходимо перенастроить запросы для анализа этого поля в объект JSON, чтобы получить доступ к этим свойствам. Это достигается с помощью [parseJSON](/azure/data-explorer/kusto/query/samples?pivots=#parsejson) для доступа к определенному элементу JSON в известном пути.

Например, модуль Runbook форматирует свойство *ресултдескриптион* в выходном потоке в формате JSON с несколькими полями. Чтобы найти состояние заданий, которые находятся в состоянии сбоя, как указано в поле **Status (состояние**), используйте следующий пример запроса для поиска *ресултдескриптион* с состоянием **Failed**:

```kusto
AzureDiagnostics
| where Category == 'JobStreams'
| extend jsonResourceDescription = parse_json(ResultDescription)
| where jsonResourceDescription.Status == 'Failed'
```

![Log Analytics формат JSON потока заданий с предысторией](media/automation-manage-send-joblogs-log-analytics/job-status-format-json.png)

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы узнать, как создавать поисковые запросы и просматривать журналы заданий службы автоматизации на платформе журналов Azure Monitor, ознакомьтесь со статьей [Поиск в журналах Azure Monitor](../azure-monitor/logs/log-query-overview.md).
* Чтобы узнать больше о создании и получении выходных данных и сообщений об ошибках для runbook, см. статью [Отслеживание выходных данных runbook](automation-runbook-output-and-messages.md).
* Дополнительные сведения о выполнении runbook, отслеживании заданий runbook и другие технические подробности см. в статье [Выполнение runbook в службе автоматизации Azure](automation-runbook-execution.md).
* Чтобы узнать больше о журналах Azure Monitor и источниках сбора данных, см. статью [Общие сведения о сборе данных службы хранилища Azure на платформе журналов Azure Monitor](../azure-monitor/essentials/resource-logs.md#send-to-log-analytics-workspace).
* В устранении неполадок с Log Analytics вам может помочь раздел [Почему Log Analytics больше не собирает данные](../azure-monitor/logs/manage-cost-storage.md#troubleshooting-why-log-analytics-is-no-longer-collecting-data).