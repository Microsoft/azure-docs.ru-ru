---
title: Аналитика SQL Azure решение в Azure Monitor | Документация Майкрософт
description: Как решение "Аналитика SQL Azure" поможет вам управлять базами данных SQL Azure
ms.topic: conceptual
author: danimir
ms.author: danil
ms.date: 09/19/2020
ms.reviewer: carlrab
ms.openlocfilehash: 54ef88e65925ba9c7e9fe2e44ef0c76fbc9ceb04
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101717491"
---
# <a name="monitor-azure-sql-database-using-azure-sql-analytics-preview"></a>Мониторинг базы данных SQL Azure с помощью решения "Аналитика SQL Azure" (предварительная версия)

![Символ службы "Аналитика SQL Azure"](./media/azure-sql/azure-sql-symbol.png)

Аналитика SQL Azure — это расширенное решение мониторинга облака для мониторинга производительности всех баз данных SQL Azure в масштабе и нескольких подписок в одном представлении. Аналитика SQL Azure собирает и визуализирует ключевые метрики производительности со встроенной аналитикой для устранения неполадок с производительностью.

Используя собранные метрики, можно создавать настраиваемые правила и предупреждения мониторинга. Аналитика SQL Azure помогает выявление проблем на каждом уровне стека приложений. Он использует метрики диагностики Azure вместе с Azure Monitor представлениями, чтобы представлять данные обо всех базах данных SQL Azure в одной Log Analytics рабочей области. Azure Monitor помогает собираются, сопоставляются и визуализируются структурированные и неструктурированные данные.

Практические сведения об использовании решения "Аналитика SQL Azure" и типичные сценарии его применения приведены в видео:

>[!VIDEO https://www.youtube.com/embed/j-NDkN4GIzg]
>

## <a name="connected-sources"></a>Подключенные источники

Аналитика SQL Azure — это облачное решение для мониторинга, которое поддерживает потоковую передачу диагностических данных диагностики для всех ваших баз. Поскольку Аналитика SQL Azure не использует агенты для подключения к Azure Monitor, он не поддерживает мониторинг SQL Server, размещенных локально или на виртуальных машинах.

| Подключенный источник | Поддерживается | Описание |
| --- | --- | --- |
| [Параметры диагностики](../essentials/diagnostic-settings.md) | **Да** | Данные метрик и журналов Azure отправляются в журналы Azure Monitor непосредственно в Azure. |
| [Учетная запись хранения Azure](../essentials/resource-logs.md#send-to-log-analytics-workspace) | Нет | Azure Monitor не считывает данные из учетной записи хранения. |
| [Агенты Windows](../agents/agent-windows.md) | Нет | Непосредственные агенты Windows не используются Аналитика SQL Azure. |
| [Агенты Linux](../vm/quick-collect-linux-computer.md) | Нет | Непосредственные агенты Linux не используются Аналитика SQL Azure. |
| [Группа управления System Center Operations Manager](../agents/om-agents.md) | Нет | Прямое подключение от агента Operations Manager к Azure Monitor не используется Аналитика SQL Azure. |

## <a name="azure-sql-analytics-options"></a>Параметры Аналитика SQL Azure

В приведенной ниже таблице описаны поддерживаемые варианты для двух версий панели мониторинга Аналитика SQL Azure, одна для базы данных SQL Azure, а другая — для баз данных Azure SQL Управляемый экземпляр.

| Аналитика SQL Azure, параметр | Описание | Поддержка базы данных SQL | Поддержка Управляемого экземпляра SQL |
| --- | ------- | ----- | ----- |
| Resource by type (Ресурсы по типу) | Перспектива, в которой представлено число всех отслеживаемых ресурсов. | Да | Да |
| Аналитика | Предоставляет подробные сведения о производительности Intelligent Insights в иерархическом виде. | Да | Да |
| ошибки | Предоставляет подробные данные об ошибках SQL, возникших в базах данных, в иерархическом виде. | Да | Да |
| Время ожидания | Предоставляет подробные данные о времени ожидания SQL в базах данных в иерархическом виде. | Да | Нет |
| Blockings (Блокировки) | Предоставляет подробные данные о блокировках SQL в базах данных в иерархическом виде. | Да | Нет |
| Database waits (Время ожидания базы данных) | Предоставляет подробную статистику времени ожидания SQL на уровне базы данных в иерархическом виде. Включает сводку по общему времени ожидания и времени ожидания для каждого типа ожидания. |Да | Нет |
| Query duration (Длительность запросов) | Предоставляет подробную статистику о выполнении запросов, такую как продолжительность запроса, загрузку ЦП, число операций ввода-вывода данных, число операций ввода-вывода журнала, в иерархическом виде. | Да | Да |
| Query waits (Время ожидания запроса) | Предоставляет подробную статистику времени ожидании запросов по категории ожидания в иерархическом виде. | Да | Да |

## <a name="configuration"></a>Конфигурация

Используйте процесс, описанный в статье [добавление Azure Monitor решений из коллекция решений](./solutions.md) , чтобы добавить аналитика SQL Azure (Предварительная версия) в рабочую область log Analytics.

### <a name="configure-azure-sql-database-to-stream-diagnostics-telemetry"></a>Настройка базы данных SQL Azure для передачи телеметрии диагностики

После создания решения Аналитика SQL Azure в рабочей области необходимо **настроить каждый** ресурс, который необходимо отслеживать, для потоковой передачи данных диагностики для аналитика SQL Azure. Выполните подробные инструкции на этой странице:

- Включите система диагностики Azure для базы данных, чтобы [передавать данные телеметрии диагностики в аналитика SQL Azure](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md).

Приведенная выше страница также содержит инструкции по включению поддержки для наблюдения за несколькими подписками Azure из одной рабочей области службы "Аналитика SQL Azure", выступающей в качестве единой панели.

## <a name="using-azure-sql-analytics"></a>Использование Аналитика SQL Azure

При добавлении Аналитика SQL Azure в рабочую область плитка Аналитика SQL Azure добавляется в рабочую область и отображается в разделе Обзор. Щелкните ссылку Просмотреть сводку, чтобы загрузить содержимое плитки.

![Плитка сводки Аналитика SQL Azure](./media/azure-sql/azure-sql-sol-tile-01.png)

После загрузки на плитке отображается количество баз данных и эластичных пулов в базе данных SQL и экземплярах и базах данных экземпляров в SQL Управляемый экземпляр, из которых Аналитика SQL Azure получает данные телеметрии диагностики.

![Элемент служб анализа SQL Azure](./media/azure-sql/azure-sql-sol-tile-02.png)

Аналитика SQL Azure предоставляет два отдельных представления — одно для мониторинга базы данных SQL, а другое — для наблюдения за Управляемый экземпляр SQL.

Чтобы просмотреть панель мониторинга Аналитика SQL Azure наблюдения для базы данных SQL, щелкните верхнюю часть плитки. Чтобы просмотреть панель мониторинга Аналитика SQL Azure мониторинг для SQL Управляемый экземпляр, щелкните нижнюю часть плитки.

### <a name="viewing-azure-sql-analytics-data"></a>Просмотр данных службы "Аналитика SQL Azure"

На панели мониторинга отображаются общие сведения обо всех базах данных, мониторинг которых осуществляется через различные перспективы. Для работы с разными перспективами необходимо включить соответствующие метрики или журналы в ресурсах SQL для потоковой передачи в Log Analytics рабочую область.

Если некоторые метрики или журналы не передаются в Azure Monitor, плитки в Аналитика SQL Azure не заполняются данными мониторинга.

### <a name="sql-database-view"></a>Представление базы данных SQL

После выбора плитки "Аналитика SQL Azure" для базы данных отобразится панель мониторинга.

![Снимок экрана, показывающий панель мониторинга.](./media/azure-sql/azure-sql-sol-overview.png)

При выборе любой плитки открывается подробный отчет о конкретной перспективе. Выбрав перспективу, вы увидите подробный отчет.

![Снимок экрана, показывающий детализированный отчет на конкретную перспективу.](./media/azure-sql/azure-sql-sol-metrics.png)

Каждая перспектива в этом представлении предоставляет сводные данные по уровням подписки, сервера, эластичного пула и базы данных. Кроме того, справа в каждой перспективе отображается отчет о ней. Выбор подписки, сервера, пула или базы данных из списка позволяет продолжить подробное изучение.

### <a name="sql-managed-instance-view"></a>Представление SQL Управляемый экземпляр

После выбора плитки "Аналитика SQL Azure" для базы данных отобразится панель мониторинга.

![Обзор службы "Аналитика SQL Azure"](./media/azure-sql/azure-sql-sol-overview-mi.png)

При выборе любой плитки открывается подробный отчет о конкретной перспективе. Выбрав перспективу, вы увидите подробный отчет.

При выборе представления SQL Управляемый экземпляр отображаются сведения об использовании экземпляра, базах данных экземпляра и телеметрии о запросах, выполняемых в управляемом экземпляре.

![Время ожидания службы "Аналитика SQL Azure"](./media/azure-sql/azure-sql-sol-metrics-mi.png)

### <a name="intelligent-insights-report"></a>Отчет Intelligent Insights

[Intelligent Insights](../../azure-sql/database/intelligent-insights-overview.md) для Базы данных SQL Azure позволяет узнать о производительности всех баз данных SQL Azure. Все собранные данные Intelligent Insights можно визуализировать в перспективе Intelligent Insights, а также получить к ним доступ.

![Аналитические сведения службы "Аналитика SQL Azure"](./media/azure-sql/azure-sql-sol-insights.png)

### <a name="elastic-pools-and-database-reports"></a>Эластичные пулы и отчеты базы данных

Как эластичные пулы, так и базы данных имеют собственные отчеты, отображающие все данные, собранные для ресурса за указанное время.

![База данных службы "Аналитика SQL Azure"](./media/azure-sql/azure-sql-sol-database.png)

![Эластичный пул SQL Azure](./media/azure-sql/azure-sql-sol-pool.png)

### <a name="query-reports"></a>Отчеты о запросах

С помощью запроса длительность и время ожидания запроса можно сопоставить производительность любого запроса в отчете. В этом отчете сравнивается производительность запросов в различных базах данных. Кроме того, он упрощает точное определение баз данных с высокой и низкой производительностью запросов.

![Запросы службы "Аналитика SQL Azure"](./media/azure-sql/azure-sql-sol-queries.png)

## <a name="permissions"></a>Разрешения

Для использования службы "Аналитика SQL Azure" пользователям нужно предоставить минимальное разрешение ​​роли читателя в Azure. Однако эта роль не позволяет им видеть текст запроса или выполнять какие-либо действия по автоматической настройке. Более разрешающие роли в Azure, которые позволяют использовать Аналитика SQL Azure для полного экстента, — владелец, участник, участник базы данных SQL или участник SQL Server. Вы также можете создать на портале пользовательскую роль с определенными разрешениями, необходимыми только для использования службы "Аналитика SQL Azure", и без доступа к управлению другими ресурсами.

### <a name="creating-a-custom-role-in-portal"></a>Создание пользовательской роли на портале

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Некоторые организации применяют строгие механизмы контроля разрешений в Azure, поэтому мы предоставляем следующий скрипт PowerShell, который позволяет создать на портале Azure пользовательскую роль "Оператор мониторинга Аналитики SQL" с минимальными разрешениями на чтение и запись, необходимыми для использования службы "Аналитика SQL Azure" в полной мере.

Замените {SubscriptionId} в приведенном ниже скрипте идентификатором подписки Azure и выполните скрипт, войдя в систему от имени роли владельца или участника в Azure.

   ```powershell
    Connect-AzAccount
    Select-AzSubscription {SubscriptionId}
    $role = Get-AzRoleDefinition -Name Reader
    $role.Name = "SQL Analytics Monitoring Operator"
    $role.Description = "Lets you monitor database performance with Azure SQL Analytics as a reader. Does not allow change of resources."
    $role.IsCustom = $true
    $role.Actions.Add("Microsoft.SQL/servers/databases/read");
    $role.Actions.Add("Microsoft.SQL/servers/databases/topQueries/queryText/*");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/write");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/recommendedActions/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/advisors/recommendedActions/write");
    $role.Actions.Add("Microsoft.Sql/servers/databases/automaticTuning/read");
    $role.Actions.Add("Microsoft.Sql/servers/databases/automaticTuning/write");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/read");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/write");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/recommendedActions/read");
    $role.Actions.Add("Microsoft.Sql/servers/advisors/recommendedActions/write");
    $role.Actions.Add("Microsoft.Resources/deployments/write");
    $role.AssignableScopes = "/subscriptions/{SubscriptionId}"
    New-AzRoleDefinition $role
   ```

После создания роли назначьте ее каждому пользователю, которому необходимо предоставить пользовательские разрешения на использование службы "Аналитика SQL Azure".

## <a name="analyze-data-and-create-alerts"></a>Анализ данных и создание оповещений

Анализ данных в службе "Аналитика SQL Azure" основан на [языке Log Analytics](../logs/get-started-queries.md), который используется для пользовательских запросов и отчетов. Найдите описание доступных данных, полученных из ресурса базы данных, для отправки пользовательских запросов к [доступным метрикам и журналам](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md#metrics-and-logs-available).

Автоматическое оповещение в Аналитика SQL Azure основано на написании Log Analytics запроса, который запускает оповещение при выполнении условия. Ниже приведены несколько примеров Log Analytics запросов, на основе которых можно настроить оповещения в Аналитика SQL Azure.

### <a name="creating-alerts-for-azure-sql-database"></a>Создание оповещений для Базы данных SQL Azure

[Оповещения можно легко создать](../alerts/alerts-metric.md) с помощью данных, поступающих из ресурсов службы "База данных SQL Azure". Вот несколько полезных [запросов журналов](../logs/log-query-overview.md), которые можно использовать с оповещениями журналов:

#### <a name="high-cpu"></a>Высокая загрузка ЦП

```
AzureMetrics
| where ResourceProvider=="MICROSOFT.SQL"
| where ResourceId contains "/DATABASES/"
| where MetricName=="cpu_percent"
| summarize AggregatedValue = max(Maximum) by bin(TimeGenerated, 5m)
| render timechart
```

> [!NOTE]
>
> - Предварительное требование для настройки этого предупреждения заключается в том, что отслеживаемые базы данных передаются базовыми метриками для Аналитика SQL Azure.
> - Замените значение MetricName cpu_percent на dtu_consumption_percent, чтобы получить высокие результаты DTU.

#### <a name="high-cpu-on-elastic-pools"></a>Высокая загрузка ЦП в эластичных пулах

```
AzureMetrics
| where ResourceProvider=="MICROSOFT.SQL"
| where ResourceId contains "/ELASTICPOOLS/"
| where MetricName=="cpu_percent"
| summarize AggregatedValue = max(Maximum) by bin(TimeGenerated, 5m)
| render timechart
```

> [!NOTE]
>
> - Предварительное требование для настройки этого предупреждения заключается в том, что отслеживаемые базы данных передаются базовыми метриками для Аналитика SQL Azure.
> - Замените значение MetricName cpu_percent на dtu_consumption_percent, чтобы получить высокие результаты DTU.

#### <a name="storage-in-average-above-95-in-the-last-1-hr"></a>Хранилище в среднем выше 95% за последние 1 ч

```
let time_range = 1h;
let storage_threshold = 95;
AzureMetrics
| where ResourceId contains "/DATABASES/"
| where MetricName == "storage_percent"
| summarize max_storage = max(Average) by ResourceId, bin(TimeGenerated, time_range)
| where max_storage > storage_threshold
| distinct ResourceId
```

> [!NOTE]
>
> - Предварительное требование для настройки этого предупреждения заключается в том, что отслеживаемые базы данных передаются базовыми метриками для Аналитика SQL Azure.
> - Для этого запроса требуется такая настройка правила генерации оповещений, чтобы оповещение запускалось при наличии результатов (> 0) запроса, обозначающих, что условие существует в некоторых базах данных. В выходных данных содержится список ресурсов базы данных, которые превышают порог storage_threshold в рамках определенного time_range.
> - В выходных данных содержится список ресурсов базы данных, которые превышают порог storage_threshold в рамках определенного time_range.

#### <a name="alert-on-intelligent-insights"></a>Оповещение об Intelligent Insights

> [!IMPORTANT]
> Если база данных работает хорошо и Intelligent Insights не были созданы, этот запрос завершится с сообщением об ошибке: не удалось разрешить скалярное выражение с именем "rootCauseAnalysis_s". Это поведение ожидается во всех случаях, когда отсутствует интеллектуальная аналитика для базы данных.

```
let alert_run_interval = 1h;
let insights_string = "hitting its CPU limits";
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active"
| where TimeGenerated > ago(alert_run_interval)
| where rootCauseAnalysis_s contains insights_string
| distinct ResourceId
```

> [!NOTE]
>
> - Предварительное требование для настройки этого предупреждения заключается в том, что мониторинг журнала диагностики SQLInsights Streaming databases в Аналитика SQL Azure.
> - Для этого запроса также требуется настроить правило генерации оповещений, которое будет выполняться с той же частотой, что и alert_run_interval, чтобы избежать повторяющихся результатов. Правило должно быть настроено на то, чтобы запускать оповещение, когда для запроса существуют результаты (> 0).
> - Настройте alert_run_interval, чтобы указать диапазон времени для проверки наличия условия в базах данных, настроенных для потоковой передачи журнала SQLInsights в Аналитика SQL Azure.
> - Настройте insights_string, чтобы записывать текст анализа первопричин, представленный в экземпляре аналитических сведений. Это тот же текст, который отображается в пользовательском интерфейсе Аналитика SQL Azure, который можно использовать из существующих аналитических данных. Кроме того, приведенный ниже запрос можно использовать для просмотра текста всех аналитических сведений, созданных в подписке. В выходных данных запроса содержатся различные строки, которые можно использовать для настройки оповещений в Insights.

```
AzureDiagnostics
| where Category == "SQLInsights" and status_s == "Active"
| distinct rootCauseAnalysis_s
```

### <a name="creating-alerts-for-sql-managed-instance"></a>Создание предупреждений для SQL Управляемый экземпляр

#### <a name="storage-is-above-90"></a>Хранилище превышает 90%

```
let storage_percentage_threshold = 90;
AzureDiagnostics
| where Category =="ResourceUsageStats"
| summarize (TimeGenerated, calculated_storage_percentage) = arg_max(TimeGenerated, todouble(storage_space_used_mb_s) *100 / todouble (reserved_storage_mb_s))
   by ResourceId
| where calculated_storage_percentage > storage_percentage_threshold
```

> [!NOTE]
>
> - Предварительное требование для настройки этого предупреждения заключается в том, что отслеживаемый управляемый экземпляр использует потоковую передачу Ресаурцеусажестатс журнала с включенным для Аналитика SQL Azure.
> - Для этого запроса необходимо настроить правило генерации оповещений на срабатывание предупреждения, если в запросе есть результаты (> 0). Это означает, что условие существует в управляемом экземпляре. Выходные данные — процент использования хранилища в управляемом экземпляре.

#### <a name="cpu-average-consumption-is-above-95-in-the-last-1-hr"></a>Среднее потребление ресурсов ЦП превышает 95% за последние 1 ч

```
let cpu_percentage_threshold = 95;
let time_threshold = ago(1h);
AzureDiagnostics
| where Category == "ResourceUsageStats" and TimeGenerated > time_threshold
| summarize avg_cpu = max(todouble(avg_cpu_percent_s)) by ResourceId
| where avg_cpu > cpu_percentage_threshold
```

> [!NOTE]
>
> - Предварительное требование для настройки этого предупреждения заключается в том, что отслеживаемый управляемый экземпляр использует потоковую передачу Ресаурцеусажестатс журнала с включенным для Аналитика SQL Azure.
> - Для этого запроса необходимо настроить правило генерации оповещений на срабатывание предупреждения, если в запросе есть результаты (> 0). Это означает, что условие существует в управляемом экземпляре. В выходных данных указывается среднее использование ЦП в течение заданного периода на управляемом экземпляре.

### <a name="pricing"></a>Цены

Хотя Аналитика SQL Azure свободно для использования, использование данных телеметрии диагностики выше количества свободных единиц, выделяемых за каждый месяц, применяется, см. [log Analytics цены](https://azure.microsoft.com/pricing/details/monitor). Бесплатные единицы приема данных позволяют осуществлять бесплатный мониторинг нескольких баз данных каждый месяц. Большее количество активных баз данных с более требовательными рабочими нагрузками применяет больше данных и бездействующих баз данных. Вы можете легко отслеживать потребление данных в Аналитика SQL Azure, выбрав рабочую область OMS в меню навигации Аналитика SQL Azure, а затем выбрав использование и предполагаемые затраты.

## <a name="next-steps"></a>Дальнейшие действия

- Используйте [запросы журналов](../logs/log-query-overview.md) в Azure Monitor для просмотра подробных данных SQL Azure.
- [Создавайте пользовательские панели мониторинга](../visualize/tutorial-logs-dashboards.md), отображающие данные SQL Azure.
- [Создавайте оповещения](../alerts/alerts-overview.md), предупреждающие о возникновении определенных событий SQL Azure.

