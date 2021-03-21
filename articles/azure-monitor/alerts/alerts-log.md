---
title: Создание, просмотр и Управление оповещениями журнала с помощью Azure Monitor | Документация Майкрософт
description: Использование Azure Monitor для создания и просмотра правил генерации оповещений журнала и управления ими
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 8759a539005a38892498c9450d31f97884547db7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102033713"
---
# <a name="create-view-and-manage-log-alerts-using-azure-monitor"></a>Создание и просмотр оповещений журнала, а также управление ими с помощью Azure Monitor

## <a name="overview"></a>Обзор

Оповещения журнала позволяют пользователям использовать [log Analytics](../logs/log-analytics-tutorial.md) запрос для проверки журнала ресурсов каждые заданные частоты и запуска предупреждения на основе результатов. Правила могут запускать одно или несколько действий с помощью [групп действий](./action-groups.md). [Дополнительные сведения о функциональности и терминологии оповещений журнала](./alerts-unified-log.md).

В этой статье показано, как создавать оповещения журналов и управлять ими с помощью Azure Monitor. Правила генерации оповещений определяются тремя компонентами:
- Цель: конкретный ресурс Azure для мониторинга.
- Критерий: логика для вычисления. При достижении этого предупреждения срабатывает.  
- Действие: уведомления или автоматизация — электронная почта, SMS, веб-перехватчик и т. д.

Правила генерации оповещений журнала также можно создать с помощью шаблонов Azure Resource Manager, которые описаны в [отдельной статье](../alerts/alerts-log-create-templates.md).

> [!NOTE]
> Данные журнала из [log Analytics рабочей области](../logs/log-analytics-tutorial.md) можно отправить в хранилище метрик Azure Monitor. Поведение оповещений метрик [отличается от поведения](./alerts-metric-overview.md), которое может быть более желательным в зависимости от данных, с которыми вы работаете. Сведения о том, что и как можно направить журналы в метрики, см. в разделе [метрики оповещений для журналов](./alerts-metric-logs.md).

## <a name="create-a-log-alert-rule-with-the-azure-portal"></a>Создание правила генерации оповещений журнала с помощью портала Azure

Ниже приведены инструкции по началу записи запросов для оповещений.

1. Перейдите к ресурсу, по которому вы хотите включить оповещение. Рекомендуется настроить правила оповещений для нескольких ресурсов, выбрав подписку или область группы ресурсов, когда это возможно. Оповещения на нескольких ресурсах уменьшают затраты и требуют управления несколькими правилами генерации оповещений.
1. В разделе **монитор** выберите **журналы**.
1. Запросите данные журнала, в которых может быть указана проблема. В [разделе с примерами запросов на предупреждения](../logs/example-queries.md) можно узнать, что можно найти или начать [с написания собственного запроса](../logs/log-analytics-tutorial.md). Кроме того, [узнайте, как создать оптимизированные запросы оповещений](alerts-log-query.md).
1. Нажмите кнопку "+ новое правило генерации оповещений", чтобы начать процесс создания предупреждения.

    ![Настройка оповещения в Log Analytics](media/alerts-log/AlertsAnalyticsCreate.png)

> [!NOTE]
> Рекомендуется создавать оповещения в масштабе при использовании режима доступа к ресурсам для журналов, которые выполняются на нескольких ресурсах с помощью группы ресурсов или области действия подписки. Предупреждение на шкале уменьшает издержки на управление правилами. Чтобы иметь возможность ориентироваться на ресурсы, включите столбец ИДЕНТИФИКАТОРов ресурсов в результаты. Дополнительные [сведения о разбиении оповещений по измерениям](./alerts-unified-log.md#split-by-alert-dimensions).

### <a name="log-alert-for-log-analytics-and-application-insights"></a>Оповещение журнала для Log Analytics и Application Insights

1. Если синтаксис запроса правильный, то данные журнала для запроса отображаются в виде диаграммы с возможностью настройки периода диаграммы за последние шесть часов до последней недели.
 
    Если результаты запроса содержат сводные данные или специфические для [проекта](/azure/kusto/query/projectoperator) столбцы без времени, на диаграмме отображается одно значение.

    ![Настройка правил оповещений](media/alerts-log/AlertsPreviewAlertLog.png)

1. Выберите диапазон времени, по которому будет оцениваться указанное условие с помощью параметра [**period**](./alerts-unified-log.md#query-time-range) .

1. Оповещения журнала могут основываться на двух типах [**мер**](./alerts-unified-log.md#measure):
    1. **Число результатов** — количество записей, возвращаемых запросом.
    1. **Измерение метрик**  -  *Статистическое значение* , вычисленное с использованием суммирования, сгруппированных по выражениям, выбранным и [ячейками](/azure/kusto/query/binfunction) . Пример:

    ```Kusto
    // Reported errors
    union Event, Syslog // Event table stores Windows event records, Syslog stores Linux records
    | where EventLevelName == "Error" // EventLevelName is used in the Event (Windows) records
    or SeverityLevel== "err" // SeverityLevel is used in Syslog (Linux) records
    | summarize AggregatedValue = count() by Computer, bin(TimeGenerated, 15m)
    ```

1. Для логики оповещения измерений метрик можно дополнительно указать способ [разделения предупреждений по измерениям](./alerts-unified-log.md#split-by-alert-dimensions) с помощью параметра **Aggregate on** . Выражение группирования строк должно быть уникальным и сортированным.

    > [!NOTE]
    > Так как [bin ()](/azure/kusto/query/binfunction) может привести к нечетным интервалам времени, служба оповещений автоматически преобразует функцию [bin ()](/azure/kusto/query/binfunction) в функцию [bin_at ()](/azure/kusto/query/binatfunction) с соответствующим временем во время выполнения, чтобы гарантировать результат с фиксированной точкой.

    > [!NOTE]
    > Разбиение по измерениям предупреждений доступно только для текущего API Счедуледкуерирулес. Если используется устаревший [API-интерфейс предупреждений log Analytics](./api-alerts.md), необходимо будет переключиться. Дополнительные [сведения о переключении](./alerts-log-api-switch.md). Оповещения, ориентированные на ресурсы в масштабе, поддерживаются только в версии API `2020-05-01-preview` и выше.

    ![параметр "Агрегация по"](media/alerts-log/aggregate-on.png)

1. Затем, основываясь на предварительном просмотре, задаете [ **оператор**, **пороговое значение**](./alerts-unified-log.md#threshold-and-operator)и [**частоту**](./alerts-unified-log.md#frequency).

1. Можно также задать [число нарушений, вызывающих срабатывание предупреждения](./alerts-unified-log.md#number-of-violations-to-trigger-alert) , с помощью **общего или последовательного нарушения**.

1. Нажмите кнопку **Готово**. 

1. Определите имя и **Описание** **правила генерации оповещений**, а также выберите **серьезность** предупреждения. Эти сведения используются во всех действиях оповещений. Кроме того, можно отключить правило генерации оповещений при создании, выбрав параметр **включить правило при создании**.

1. Выберите, если вы хотите отключить действия правил в течение определенного времени после запуска оповещения, используйте параметр [**подавлять предупреждения**](./alerts-unified-log.md#state-and-resolving-alerts) . Правило будет выполняться и создавать предупреждения, но действия не будут запускаться для предотвращения шума. Значение для отключения действий должно быть больше, чем частота оповещений, которые должны быть эффективными.

    ![Отключение оповещений в оповещениях журналов](media/alerts-log/AlertsPreviewSuppress.png)

1. Укажите, должно ли правило генерации оповещений активировать одну или несколько [**групп действий**](./action-groups.md#webhook) при выполнении условия оповещения.

    > [!NOTE]
    > Ограничения на действия, которые можно выполнить, см. в разделе [ограничения службы подписки Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md) .  

1. При необходимости можно настроить действия в правилах генерации оповещений журнала:

    - **Настраиваемый субъект электронной почты**: переопределяет *тему электронной почты* для действий с электронной почтой. Нельзя изменять текст сообщения, и это поле **не предназначено для адресов электронной почты**.
    - **Включить пользовательские полезные данные JSON**: переопределяет JSON веб-перехватчика, используемый группами действий, предполагая, что Группа действий содержит действие веб-перехватчика. Дополнительные сведения о [действии веб-перехватчика для оповещений журнала](./alerts-log-webhook.md).

    ![Переопределение действий оповещений журналов](media/alerts-log/AlertsPreviewOverrideLog.png)

1. Если все поля заданы правильно, можно щелкнуть кнопку **создать правило генерации оповещений** и создать оповещение.

    Через несколько минут оповещение включится и будет активироваться, как было описано выше.

    ![Создание правила](media/alerts-log/AlertsPreviewCreate.png)

#### <a name="creating-log-alert-for-log-analytics-and-application-insights-from-the-alerts-management"></a>Создание оповещения журнала для Log Analytics и Application Insights из управления оповещениями

> [!NOTE]
> Создание из управления оповещениями сейчас не поддерживается для журналов с использованием ресурсов

1. На [портале](https://portal.azure.com/)выберите **монитор** и щелкните **оповещения**.

    ![Мониторинг](media/alerts-log/AlertsPreviewMenu.png)

1. Выберите **новое правило генерации оповещений**. 

    ![Добавить оповещение](media/alerts-log/AlertsPreviewOption.png)

1. Появится панель **Создание предупреждения** . Он состоит из четырех частей: 
    - Ресурс, к которому применяется оповещение.
    - Проверяемое условие.
    - Действия, выполняемые, если условие истинно.
    - Сведения для имени и описания оповещения. 

    ![Создание правила](media/alerts-log/AlertsPreviewAdd.png)

1. Нажмите кнопку **выбрать ресурс** . Отфильтруйте, выбрав *подписку*, *тип ресурса* и выберите ресурс. Убедитесь, что в ресурсе доступны журналы.

   ![Выбор ресурса](media/alerts-log/Alert-SelectResourceLog.png)

1. Затем с помощью кнопки добавить **условие** можно просмотреть список параметров сигнала, доступных для ресурса. Выберите **Пользовательский параметр поиска по журналам** .

   ![Выбор ресурса: пользовательский поиск по журналам](media/alerts-log/AlertsPreviewResourceSelectionLog.png)

   > [!NOTE]
   > На портале "оповещения" перечислены сохраненные запросы из Log Analytics и Application Insights и их можно использовать в качестве запросов предупреждений шаблонов.

1. После выбора записи, вставки или изменения запроса на создание предупреждения в поле **поискового запроса** .

1. Перейдите к следующим шагам, описанным в [предыдущем разделе](#log-alert-for-log-analytics-and-application-insights).

### <a name="log-alert-for-all-other-resource-types"></a>Оповещение журнала для всех других типов ресурсов

> [!NOTE]
> В настоящее время дополнительная плата за использование версий API `2020-05-01-preview` и оповещений журнала не взимается.  Цены на функции, доступные в предварительной версии, будут объявлены в будущем и появится уведомление перед началом выставления счетов. Если вы решили продолжить использование новой версии API и оповещений журнала на основе ресурсов по истечении периода уведомления, плата будет взиматься по применимой ставке.

1. Начните с вкладки **условие** :

    1. Убедитесь, что [**мера**](./alerts-unified-log.md#measure), [**тип агрегирования**](./alerts-unified-log.md#aggregation-type)и [**гранулярность статистической обработки**](./alerts-unified-log.md#aggregation-granularity) верны. 
        1. По умолчанию правило подсчитывает количество результатов за последние 5 минут.
        1. При обнаружении сводных результатов запроса правило будет автоматически обновляться в течение нескольких секунд для записи.

    1. При необходимости выберите [разбиение предупреждений по измерениям](./alerts-unified-log.md#split-by-alert-dimensions): 
       - **Столбец идентификаторов ресурсов** выбирается автоматически, если он обнаружен, и изменяет контекст запущенного предупреждения на ресурс записи. 
       - **Столбец Идентификатор ресурса** можно отменить, чтобы инициировать оповещения по подписке или группам ресурсов. Отмена выбора полезна, если результаты запроса основаны на перекрестных ресурсах. Например, запрос, который проверяет, испытывает ли 80% виртуальных машин группы ресурсов высокий уровень загрузки ЦП.
       - Для любого типа числовых или текстовых столбцов с помощью таблицы Dimensions можно также выбрать до шести дополнительных прерываний.
       - Оповещения запускаются отдельно в соответствии с разделением на основе уникальных сочетаний и полезных данных оповещений, содержащих эти сведения.
    
        ![Выбор параметров агрегирования и разбиение](media/alerts-log/select-aggregation-parameters-and-splitting.png)

    1. На диаграмме **предварительного просмотра** результаты оценки запросов отображаются с течением времени. Можно изменить период диаграммы или выбрать другой временной ряд, который привел к уникальному разбиению предупреждений по измерениям.

        ![Предварительный просмотр диаграммы](media/alerts-log/preview-chart.png)

    1. Затем на основе данных для предварительного просмотра задайте **логику оповещения**. [ **Оператор**, **пороговое значение**](./alerts-unified-log.md#threshold-and-operator)и [**периодичность**](./alerts-unified-log.md#frequency).

        ![Предварительный просмотр диаграммы с пороговыми значениями и оповещениями](media/alerts-log/chart-and-alert-logic.png)

    1. При необходимости можно задать [**число нарушений, вызывающих срабатывание предупреждения**](./alerts-unified-log.md#number-of-violations-to-trigger-alert) , в разделе **Дополнительные параметры** .
    
        ![Расширенные параметры](media/alerts-log/advanced-options.png)

1. На вкладке **действия** выберите или создайте необходимые [группы действий](./action-groups.md).

    ![Вкладка "действия"](media/alerts-log/actions-tab.png)

1. На вкладке **сведения** укажите **сведения о правиле генерации оповещений** и **сведения о проекте**. При необходимости можно указать, следует ли не **запускать сейчас**, или [**Отключить действия**](./alerts-unified-log.md#state-and-resolving-alerts) за период после срабатывания правила оповещения.

    > [!NOTE]
    > Правила генерации оповещений журнала в настоящее время не имеют состояния и запускают действие при каждом создании предупреждения, если не определено выключение звука.

    ![Вкладка "Подробные сведения"](media/alerts-log/details-tab.png)

1. На вкладке **теги** задайте необходимые теги в ресурсе правила генерации оповещений.

    ![Вкладка "Теги"](media/alerts-log/tags-tab.png)

1. На вкладке " **Проверка и создание** " будет выполнена проверка и уведомление о любых проблемах. Проверьте и утвердите определение правила.
1. Если все поля верны, нажмите кнопку **создать** и завершите создание правила генерации оповещений. Все оповещения можно просмотреть в окне Управление оповещениями.
 
    ![Вкладка "Проверка и создание"](media/alerts-log/review-and-create-tab.png)

## <a name="view--manage-log-alerts-in-azure-portal"></a>Просмотр оповещений журнала, и управление ими на портале Azure

1. На [портале](https://portal.azure.com/)выберите соответствующий ресурс или службу **мониторинга** . Затем выберите **оповещения** в разделе монитор.

1. В окне Управление оповещениями отображаются все оповещения, которые были запущены. Дополнительные [сведения об управлении оповещениями](alerts-managing-alert-instances.md).

    > [!NOTE]
    > Правила генерации оповещений журнала в настоящее время не имеют [состояния и не разрешаются](./alerts-unified-log.md#state-and-resolving-alerts).

1. Нажмите кнопку **Управление правилами генерации оповещений** на верхней панели, чтобы изменить правила.

    ![ управление правилами генерации оповещений](media/alerts-log/manage-alert-rules.png)

## <a name="managing-log-alerts-using-powershell"></a>Управление оповещениями журнала с помощью PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!NOTE]
> В настоящее время PowerShell не поддерживается в версии API `2020-05-01-preview`

Ниже перечислены командлеты PowerShell, которые можно использовать для управления правилами с помощью [API запланированных правил запросов](/rest/api/monitor/scheduledqueryrules/).

- [New-азсчедуледкуерируле](/powershell/module/az.monitor/new-azscheduledqueryrule) : командлет PowerShell для создания нового правила оповещения журнала.
- Командлет [Set-азсчедуледкуерируле](/powershell/module/az.monitor/set-azscheduledqueryrule) : PowerShell для обновления существующего правила оповещения журнала.
- [New-азсчедуледкуерирулесаурце](/powershell/module/az.monitor/new-azscheduledqueryrulesource) : командлет PowerShell для создания или обновления объекта, указывающего параметры источника для оповещения журнала. Используется в качестве входных данных командлетами [New-азсчедуледкуерируле](/powershell/module/az.monitor/new-azscheduledqueryrule) и [Set-азсчедуледкуерируле](/powershell/module/az.monitor/set-azscheduledqueryrule) .
- [New-азсчедуледкуерирулесчедуле](/powershell/module/az.monitor/new-azscheduledqueryruleschedule): командлет PowerShell для создания или обновления объекта, указывающего параметры расписания для оповещения журнала. Используется в качестве входных данных командлетами [New-азсчедуледкуерируле](/powershell/module/az.monitor/new-azscheduledqueryrule) и [Set-азсчедуледкуерируле](/powershell/module/az.monitor/set-azscheduledqueryrule) .
- [New-азсчедуледкуерирулеалертингактион](/powershell/module/az.monitor/new-azscheduledqueryrulealertingaction) : командлет PowerShell для создания или обновления объекта, указывающего параметры действия для оповещения журнала. Используется в качестве входных данных командлетами [New-азсчедуледкуерируле](/powershell/module/az.monitor/new-azscheduledqueryrule) и [Set-азсчедуледкуерируле](/powershell/module/az.monitor/set-azscheduledqueryrule) .
- [New-азсчедуледкуерирулеазнсактионграуп](/powershell/module/az.monitor/new-azscheduledqueryruleaznsactiongroup) : командлет PowerShell для создания или обновления объекта, указывающего параметры групп действий для оповещения журнала. Используется в качестве входных данных командлетом [New-азсчедуледкуерирулеалертингактион](/powershell/module/az.monitor/new-azscheduledqueryrulealertingaction) .
- [New-азсчедуледкуерирулетригжеркондитион](/powershell/module/az.monitor/new-azscheduledqueryruletriggercondition) : командлет PowerShell для создания или обновления объекта, указывающего параметры условия триггера для оповещения журнала. Используется в качестве входных данных командлетом [New-азсчедуледкуерирулеалертингактион](/powershell/module/az.monitor/new-azscheduledqueryrulealertingaction) .
- [New-азсчедуледкуерирулелогметриктригжер](/powershell/module/az.monitor/new-azscheduledqueryrulelogmetrictrigger) : командлет PowerShell для создания или обновления объекта, указывающего параметры условия триггера метрик для [оповещения журнала типа измерения "измерение](./alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value)". Используется в качестве входных данных командлетом [New-азсчедуледкуерирулетригжеркондитион](/powershell/module/az.monitor/new-azscheduledqueryruletriggercondition) .
- [Get-азсчедуледкуерируле](/powershell/module/az.monitor/get-azscheduledqueryrule) : командлет PowerShell для перечисления существующих правил генерации оповещений журнала или определенного правила оповещения журнала
- [Update-азсчедуледкуерируле](/powershell/module/az.monitor/update-azscheduledqueryrule) : командлет PowerShell для включения или отключения правила генерации оповещений журнала
- [Remove-азсчедуледкуерируле](/powershell/module/az.monitor/remove-azscheduledqueryrule): командлет PowerShell для удаления существующего правила оповещения журнала

> [!NOTE]
> Командлеты PowerShell для Счедуледкуерирулес могут управлять только правилами, созданными в текущем [запланированном API правил запросов](/rest/api/monitor/scheduledqueryrules/). Правила генерации оповещений журнала, созданные с помощью устаревшего [log Analytics API предупреждений](./api-alerts.md) , можно управлять только с помощью PowerShell только после [переключения на API запланированных правил запросов](../alerts/alerts-log-api-switch.md).

Ниже приведены примеры действий по созданию правила генерации оповещений журнала с помощью PowerShell.

```powershell
$source = New-AzScheduledQueryRuleSource -Query 'Heartbeat | summarize AggregatedValue = count() by bin(TimeGenerated, 5m), _ResourceId' -DataSourceId "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/contosoRG/providers/microsoft.OperationalInsights/workspaces/servicews"

$schedule = New-AzScheduledQueryRuleSchedule -FrequencyInMinutes 15 -TimeWindowInMinutes 30

$metricTrigger = New-AzScheduledQueryRuleLogMetricTrigger -ThresholdOperator "GreaterThan" -Threshold 2 -MetricTriggerType "Consecutive" -MetricColumn "_ResourceId"

$triggerCondition = New-AzScheduledQueryRuleTriggerCondition -ThresholdOperator "LessThan" -Threshold 5 -MetricTrigger $metricTrigger

$aznsActionGroup = New-AzScheduledQueryRuleAznsActionGroup -ActionGroup "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/contosoRG/providers/microsoft.insights/actiongroups/sampleAG" -EmailSubject "Custom email subject" -CustomWebhookPayload "{ `"alert`":`"#alertrulename`", `"IncludeSearchResults`":true }"

$alertingAction = New-AzScheduledQueryRuleAlertingAction -AznsAction $aznsActionGroup -Severity "3" -Trigger $triggerCondition

New-AzScheduledQueryRule -ResourceGroupName "contosoRG" -Location "Region Name for your Application Insights App or Log Analytics Workspace" -Action $alertingAction -Enabled $true -Description "Alert description" -Schedule $schedule -Source $source -Name "Alert Name"
```

Ниже приведены примеры действий по созданию правила генерации оповещений журнала с помощью PowerShell с запросами перекрестных ресурсов.

```powershell
$authorized = @ ("/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/contosoRG/providers/microsoft.OperationalInsights/workspaces/servicewsCrossExample", "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/contosoRG/providers/microsoft.insights/components/serviceAppInsights")

$source = New-AzScheduledQueryRuleSource -Query 'Heartbeat | summarize AggregatedValue = count() by bin(TimeGenerated, 5m), _ResourceId' -DataSourceId "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/contosoRG/providers/microsoft.OperationalInsights/workspaces/servicews" -AuthorizedResource $authorized

$schedule = New-AzScheduledQueryRuleSchedule -FrequencyInMinutes 15 -TimeWindowInMinutes 30

$metricTrigger = New-AzScheduledQueryRuleLogMetricTrigger -ThresholdOperator "GreaterThan" -Threshold 2 -MetricTriggerType "Consecutive" -MetricColumn "_ResourceId"

$triggerCondition = New-AzScheduledQueryRuleTriggerCondition -ThresholdOperator "LessThan" -Threshold 5 -MetricTrigger $metricTrigger

$aznsActionGroup = New-AzScheduledQueryRuleAznsActionGroup -ActionGroup "/subscriptions/a123d7efg-123c-1234-5678-a12bc3defgh4/resourceGroups/contosoRG/providers/microsoft.insights/actiongroups/sampleAG" -EmailSubject "Custom email subject" -CustomWebhookPayload "{ `"alert`":`"#alertrulename`", `"IncludeSearchResults`":true }"

$alertingAction = New-AzScheduledQueryRuleAlertingAction -AznsAction $aznsActionGroup -Severity "3" -Trigger $triggerCondition
New-AzScheduledQueryRule -ResourceGroupName "contosoRG" -Location "Region Name for your Application Insights App or Log Analytics Workspace" -Action $alertingAction -Enabled $true -Description "Alert description" -Schedule $schedule -Source $source -Name "Alert Name" 
```

Вы также можете создать оповещение журнала с помощью файлов [шаблона и параметров](./alerts-log-create-templates.md) с помощью PowerShell:

```powershell
Connect-AzAccount

Select-AzSubscription -SubscriptionName <yourSubscriptionName>

New-AzResourceGroupDeployment -Name AlertDeployment -ResourceGroupName ResourceGroupofTargetResource `
  -TemplateFile mylogalerttemplate.json -TemplateParameterFile mylogalerttemplate.parameters.json
```

## <a name="managing-log-alerts-using-cli"></a>Управление оповещениями журнала с помощью интерфейса командной строки

> [!NOTE]
> Поддержка Azure CLI доступна только для версии API Счедуледкуерирулес `2020-05-01-preview` и более поздних версий. Версия API предыдущей может использовать интерфейс командной строки Azure Resource Manager с шаблонами, как описано ниже. При использовании устаревшего [API log Analytics предупреждений](./api-alerts.md)необходимо переключиться на использование интерфейса командной строки. Дополнительные [сведения о переключении](./alerts-log-api-switch.md).

В предыдущих разделах были описаны способы создания, просмотра и управления правилами генерации оповещений журнала с помощью портал Azure. В этом разделе вы узнаете, как сделать то же самое с помощью кроссплатформенного [Azure CLI](/cli/azure/get-started-with-azure-cli). Самый быстрый способ начать использовать Azure CLI посредством [Azure Cloud Shell](../../cloud-shell/overview.md). В этой статье мы будем использовать Cloud Shell.

1. Перейдите в портал Azure выберите **Cloud Shell**.

1. В командной строке вы можете использовать команды с параметром ``--help``, чтобы узнать больше о команде и о том, как ее использовать. Например, следующая команда отображает список команд, доступных для создания, просмотра и управления оповещениями журнала:

    ```azurecli
    az monitor scheduled-query --help
    ```

1. Можно создать правило оповещения журнала, отслеживающее количество ошибок системных событий.

    ```azurecli
    az monitor scheduled-query create -g {ResourceGroup} -n {nameofthealert} --scopes {vm_id} --condition "count \'union Event, Syslog | where TimeGenerated > ago(1h) | where EventLevelName == \"Error\" or SeverityLevel== \"err\"\' > 2" --description {descriptionofthealert}
    ```

1. Вы можете просмотреть все оповещения журнала в группе ресурсов с помощью следующей команды:

    ```azurecli
    az monitor scheduled-query list -g {ResourceGroup}
    ```

1. Сведения о конкретном правиле генерации оповещений журнала можно просмотреть с помощью имени или идентификатора ресурса правила:

    ```azurecli
    az monitor scheduled-query show -g {ResourceGroup} -n {AlertRuleName}
    ```

    ```azurecli
    az monitor scheduled-query show --ids {RuleResourceId}
    ```

1. Вы можете отключить правило генерации оповещений журнала с помощью следующей команды:

    ```azurecli
    az monitor scheduled-query update -g {ResourceGroup} -n {AlertRuleName} --enabled false
    ```

1. Правило оповещения журнала можно удалить с помощью следующей команды:

    ```azurecli
    az monitor scheduled-query delete -g {ResourceGroup} -n {AlertRuleName}
    ```

Azure Resource Manager CLI можно также использовать с файлами [шаблонов](./alerts-log-create-templates.md) :

```azurecli
az login

az deployment group create \
    --name AlertDeployment \
    --resource-group ResourceGroupofTargetResource \
    --template-file mylogalerttemplate.json \
    --parameters @mylogalerttemplate.parameters.json
```

При успешном создании возвращается 201. При успешном обновлении возвращается 200.

## <a name="next-steps"></a>Дальнейшие действия

* Сведения об [оповещениях журнала](./alerts-unified-log.md).
* Создание оповещений журнала с помощью [шаблонов Azure Resource Manager](./alerts-log-create-templates.md).
* Общие сведения о [действиях веб-перехватчика для оповещений журнала](./alerts-log-webhook.md).
* Дополнительные сведения о [запросах журналов](../logs/log-query-overview.md).
