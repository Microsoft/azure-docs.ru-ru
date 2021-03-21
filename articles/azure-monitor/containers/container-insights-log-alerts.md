---
title: Журналы оповещений из контейнера "аналитика" | Документация Майкрософт
description: В этой статье описывается создание настраиваемых оповещений журнала для использования памяти и ЦП из аналитики контейнера.
ms.topic: conceptual
ms.date: 01/05/2021
ms.openlocfilehash: 64d499d69194ac338d367ae094e42f4c8af23bef
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101711201"
---
# <a name="how-to-create-log-alerts-from-container-insights"></a>Создание оповещений журнала из аналитики контейнера

Аналитика контейнеров отслеживает производительность рабочих нагрузок контейнера, развернутых в управляемых или самостоятельно управляемых кластерах Kubernetes. Чтобы предупредить о важности, в этой статье описывается создание оповещений на основе журнала для следующих ситуаций с кластерами AKS:

- Если загрузка ЦП или памяти на узлах кластера превышает пороговое значение
- Если загрузка ЦП или памяти для любого контейнера в контроллере превышает пороговое значение по сравнению с ограничением, установленным для соответствующего ресурса
- Число узлов состояния *NotReady*
- Число *неудачных*, *ожидающих*, *неизвестных*, *выполняющихся* или *успешно выполненных* этапов Pod
- Если свободное дисковое пространство на узлах кластера превышает пороговое значение

Чтобы создать оповещение о высокой загрузке ЦП или памяти или нехватке дискового пространства на узлах кластера, используйте предоставленные запросы для создания оповещения метрики или оповещения о измерении метрик. Хотя оповещения метрик имеют меньшую задержку, чем оповещения журнала, оповещения журналов обеспечивают расширенные возможности запросов и более сложные. Запросы на оповещение журнала сравнивают дату и время с текущим оператором *Now* и возвращаясь к одному часу. (В контейнере аналитики хранятся все даты в формате UTC).

Если вы не знакомы с Azure Monitor оповещениями, ознакомьтесь со статьей [Обзор оповещений в Microsoft Azure](../alerts/alerts-overview.md) перед началом работы. Дополнительные сведения об оповещениях, использующих запросы журналов, см. [в разделе оповещения журнала в Azure Monitor](../alerts/alerts-unified-log.md). Дополнительные сведения об оповещениях метрик см. [в разделе оповещения метрик в Azure Monitor](../alerts/alerts-metric-overview.md).

## <a name="resource-utilization-log-search-queries"></a>Запросы поиска по журналам использования ресурсов

Запросы в этом разделе поддерживают каждый сценарий создания предупреждений. Они используются на шаге 7 раздела [Создание предупреждения](#create-an-alert-rule) этой статьи.

Следующий запрос вычисляет среднюю загрузку ЦП как среднее использование ЦП для узлов членов каждую минуту.  

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let capacityCounterName = 'cpuCapacityNanoCores';
let usageCounterName = 'cpuUsageNanoCores';
KubeNodeInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
// cluster filter would go here if multiple clusters are reporting to the same Log Analytics workspace
| distinct ClusterName, Computer
| join hint.strategy=shuffle (
  Perf
  | where TimeGenerated < endDateTime
  | where TimeGenerated >= startDateTime
  | where ObjectName == 'K8SNode'
  | where CounterName == capacityCounterName
  | summarize LimitValue = max(CounterValue) by Computer, CounterName, bin(TimeGenerated, trendBinSize)
  | project Computer, CapacityStartTime = TimeGenerated, CapacityEndTime = TimeGenerated + trendBinSize, LimitValue
) on Computer
| join kind=inner hint.strategy=shuffle (
  Perf
  | where TimeGenerated < endDateTime + trendBinSize
  | where TimeGenerated >= startDateTime - trendBinSize
  | where ObjectName == 'K8SNode'
  | where CounterName == usageCounterName
  | project Computer, UsageValue = CounterValue, TimeGenerated
) on Computer
| where TimeGenerated >= CapacityStartTime and TimeGenerated < CapacityEndTime
| project ClusterName, Computer, TimeGenerated, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize AggregatedValue = avg(UsagePercent) by bin(TimeGenerated, trendBinSize), ClusterName
```

Следующий запрос вычисляет среднее использование памяти как среднее использование памяти узлами элементов каждую минуту.

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let capacityCounterName = 'memoryCapacityBytes';
let usageCounterName = 'memoryRssBytes';
KubeNodeInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
// cluster filter would go here if multiple clusters are reporting to the same Log Analytics workspace
| distinct ClusterName, Computer
| join hint.strategy=shuffle (
  Perf
  | where TimeGenerated < endDateTime
  | where TimeGenerated >= startDateTime
  | where ObjectName == 'K8SNode'
  | where CounterName == capacityCounterName
  | summarize LimitValue = max(CounterValue) by Computer, CounterName, bin(TimeGenerated, trendBinSize)
  | project Computer, CapacityStartTime = TimeGenerated, CapacityEndTime = TimeGenerated + trendBinSize, LimitValue
) on Computer
| join kind=inner hint.strategy=shuffle (
  Perf
  | where TimeGenerated < endDateTime + trendBinSize
  | where TimeGenerated >= startDateTime - trendBinSize
  | where ObjectName == 'K8SNode'
  | where CounterName == usageCounterName
  | project Computer, UsageValue = CounterValue, TimeGenerated
) on Computer
| where TimeGenerated >= CapacityStartTime and TimeGenerated < CapacityEndTime
| project ClusterName, Computer, TimeGenerated, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize AggregatedValue = avg(UsagePercent) by bin(TimeGenerated, trendBinSize), ClusterName
```
>[!IMPORTANT]
>В следующих запросах используются значения заполнителей \<your-cluster-name> и \<your-controller-name> для представления кластера и контроллера. При настройке оповещений замените их значениями, характерными для вашей среды.

Следующий запрос вычисляет среднюю загрузку ЦП для всех контейнеров в контроллере как среднее использование ЦП каждым экземпляром контейнера в контроллере каждую минуту. Измерение представляет собой процентную долю от пределов, установленных для контейнера.

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let capacityCounterName = 'cpuLimitNanoCores';
let usageCounterName = 'cpuUsageNanoCores';
let clusterName = '<your-cluster-name>';
let controllerName = '<your-controller-name>';
KubePodInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
| where ClusterName == clusterName
| where ControllerName == controllerName
| extend InstanceName = strcat(ClusterId, '/', ContainerName),
         ContainerName = strcat(controllerName, '/', tostring(split(ContainerName, '/')[1]))
| distinct Computer, InstanceName, ContainerName
| join hint.strategy=shuffle (
    Perf
    | where TimeGenerated < endDateTime
    | where TimeGenerated >= startDateTime
    | where ObjectName == 'K8SContainer'
    | where CounterName == capacityCounterName
    | summarize LimitValue = max(CounterValue) by Computer, InstanceName, bin(TimeGenerated, trendBinSize)
    | project Computer, InstanceName, LimitStartTime = TimeGenerated, LimitEndTime = TimeGenerated + trendBinSize, LimitValue
) on Computer, InstanceName
| join kind=inner hint.strategy=shuffle (
    Perf
    | where TimeGenerated < endDateTime + trendBinSize
    | where TimeGenerated >= startDateTime - trendBinSize
    | where ObjectName == 'K8SContainer'
    | where CounterName == usageCounterName
    | project Computer, InstanceName, UsageValue = CounterValue, TimeGenerated
) on Computer, InstanceName
| where TimeGenerated >= LimitStartTime and TimeGenerated < LimitEndTime
| project Computer, ContainerName, TimeGenerated, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize AggregatedValue = avg(UsagePercent) by bin(TimeGenerated, trendBinSize) , ContainerName
```

Следующий запрос вычисляет среднее использование памяти для всех контейнеров в контроллере как среднее использование памяти каждым экземпляром контейнера в контроллере каждую минуту. Измерение представляет собой процентную долю от пределов, установленных для контейнера.

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let capacityCounterName = 'memoryLimitBytes';
let usageCounterName = 'memoryRssBytes';
let clusterName = '<your-cluster-name>';
let controllerName = '<your-controller-name>';
KubePodInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
| where ClusterName == clusterName
| where ControllerName == controllerName
| extend InstanceName = strcat(ClusterId, '/', ContainerName),
         ContainerName = strcat(controllerName, '/', tostring(split(ContainerName, '/')[1]))
| distinct Computer, InstanceName, ContainerName
| join hint.strategy=shuffle (
    Perf
    | where TimeGenerated < endDateTime
    | where TimeGenerated >= startDateTime
    | where ObjectName == 'K8SContainer'
    | where CounterName == capacityCounterName
    | summarize LimitValue = max(CounterValue) by Computer, InstanceName, bin(TimeGenerated, trendBinSize)
    | project Computer, InstanceName, LimitStartTime = TimeGenerated, LimitEndTime = TimeGenerated + trendBinSize, LimitValue
) on Computer, InstanceName
| join kind=inner hint.strategy=shuffle (
    Perf
    | where TimeGenerated < endDateTime + trendBinSize
    | where TimeGenerated >= startDateTime - trendBinSize
    | where ObjectName == 'K8SContainer'
    | where CounterName == usageCounterName
    | project Computer, InstanceName, UsageValue = CounterValue, TimeGenerated
) on Computer, InstanceName
| where TimeGenerated >= LimitStartTime and TimeGenerated < LimitEndTime
| project Computer, ContainerName, TimeGenerated, UsagePercent = UsageValue * 100.0 / LimitValue
| summarize AggregatedValue = avg(UsagePercent) by bin(TimeGenerated, trendBinSize) , ContainerName
```

Следующий запрос возвращает все узлы и счетчики, имеющие состояние *Ready* и *NotReady*.

```kusto
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
let clusterName = '<your-cluster-name>';
KubeNodeInventory
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
| distinct ClusterName, Computer, TimeGenerated
| summarize ClusterSnapshotCount = count() by bin(TimeGenerated, trendBinSize), ClusterName, Computer
| join hint.strategy=broadcast kind=inner (
    KubeNodeInventory
    | where TimeGenerated < endDateTime
    | where TimeGenerated >= startDateTime
    | summarize TotalCount = count(), ReadyCount = sumif(1, Status contains ('Ready'))
                by ClusterName, Computer,  bin(TimeGenerated, trendBinSize)
    | extend NotReadyCount = TotalCount - ReadyCount
) on ClusterName, Computer, TimeGenerated
| project   TimeGenerated,
            ClusterName,
            Computer,
            ReadyCount = todouble(ReadyCount) / ClusterSnapshotCount,
            NotReadyCount = todouble(NotReadyCount) / ClusterSnapshotCount
| order by ClusterName asc, Computer asc, TimeGenerated desc
```
Следующий запрос возвращает количество этапов Pod на основе всех фаз: *сбой*, *Ожидание*, *неизвестный*, *выполнен* или *успешно*.  

```kusto
let endDateTime = now(); 
let startDateTime = ago(1h);
let trendBinSize = 1m;
let clusterName = '<your-cluster-name>';
KubePodInventory
    | where TimeGenerated < endDateTime
    | where TimeGenerated >= startDateTime
    | where ClusterName == clusterName
    | distinct ClusterName, TimeGenerated
    | summarize ClusterSnapshotCount = count() by bin(TimeGenerated, trendBinSize), ClusterName
    | join hint.strategy=broadcast (
        KubePodInventory
        | where TimeGenerated < endDateTime
        | where TimeGenerated >= startDateTime
        | summarize PodStatus=any(PodStatus) by TimeGenerated, PodUid, ClusterName
        | summarize TotalCount = count(),
                    PendingCount = sumif(1, PodStatus =~ 'Pending'),
                    RunningCount = sumif(1, PodStatus =~ 'Running'),
                    SucceededCount = sumif(1, PodStatus =~ 'Succeeded'),
                    FailedCount = sumif(1, PodStatus =~ 'Failed')
                by ClusterName, bin(TimeGenerated, trendBinSize)
    ) on ClusterName, TimeGenerated
    | extend UnknownCount = TotalCount - PendingCount - RunningCount - SucceededCount - FailedCount
    | project TimeGenerated,
              TotalCount = todouble(TotalCount) / ClusterSnapshotCount,
              PendingCount = todouble(PendingCount) / ClusterSnapshotCount,
              RunningCount = todouble(RunningCount) / ClusterSnapshotCount,
              SucceededCount = todouble(SucceededCount) / ClusterSnapshotCount,
              FailedCount = todouble(FailedCount) / ClusterSnapshotCount,
              UnknownCount = todouble(UnknownCount) / ClusterSnapshotCount
| summarize AggregatedValue = avg(PendingCount) by bin(TimeGenerated, trendBinSize)
```

>[!NOTE]
>Чтобы предупредить на определенных стадиях Pod, таких как *Ожидание*, *сбой* или *неизвестно*, измените последнюю строку запроса. Например, чтобы оповещать об использовании *FailedCount* , выполните следующие действия. <br/>`| summarize AggregatedValue = avg(FailedCount) by bin(TimeGenerated, trendBinSize)`

Следующий запрос возвращает дисковые узлы кластера, размер которых превышает 90% свободного места. Чтобы получить идентификатор кластера, сначала выполните следующий запрос и скопируйте значение из `ClusterId` Свойства:

```kusto
InsightsMetrics
| extend Tags = todynamic(Tags)            
| project ClusterId = Tags['container.azm.ms/clusterId']   
| distinct tostring(ClusterId)   
``` 

```kusto
let clusterId = '<cluster-id>';
let endDateTime = now();
let startDateTime = ago(1h);
let trendBinSize = 1m;
InsightsMetrics
| where TimeGenerated < endDateTime
| where TimeGenerated >= startDateTime
| where Origin == 'container.azm.ms/telegraf'            
| where Namespace == 'container.azm.ms/disk'            
| extend Tags = todynamic(Tags)            
| project TimeGenerated, ClusterId = Tags['container.azm.ms/clusterId'], Computer = tostring(Tags.hostName), Device = tostring(Tags.device), Path = tostring(Tags.path), DiskMetricName = Name, DiskMetricValue = Val   
| where ClusterId =~ clusterId       
| where DiskMetricName == 'used_percent'
| summarize AggregatedValue = max(DiskMetricValue) by bin(TimeGenerated, trendBinSize)
| where AggregatedValue >= 90
```

## <a name="create-an-alert-rule"></a>Создание правила оповещения

В этом разделе рассматривается создание правила генерации оповещений измерения метрик с помощью данных о производительности из контейнера аналитика. Этот базовый процесс можно использовать с различными запросами журнала для оповещения о различных счетчиках производительности. Используйте один из запросов поиска по журналам, предоставленных ранее, чтобы начать с. Сведения о создании с помощью шаблона ARM см. в статье [примеры создания оповещений журнала с помощью шаблона ресурсов Azure](../alerts/alerts-log-create-templates.md).

>[!NOTE]
>Следующая процедура создания правила оповещения для использования ресурсов контейнера требует переключения на новый API предупреждений журнала, как описано в разделе [Переключение предпочтений API для оповещений журнала](../alerts/alerts-log-api-switch.md).
>

1. Войдите на [портал Azure](https://portal.azure.com).
2. На портале Azure найдите и выберите **Рабочие области Log Analytics**.
3. В списке рабочих областей Log Analytics выберите рабочую область, поддерживающую аналитику контейнера. 
4. На панели слева выберите **журналы** , чтобы открыть страницу Azure Monitor журналы. Эта страница используется для записи и выполнения запросов журнала Azure.
5. На странице **журналы** вставьте один из указанных выше [запросов](#resource-utilization-log-search-queries) в поле **поисковый запрос** , а затем выберите **выполнить** , чтобы проверить результаты. Если не выполнить этот шаг, параметр **+ создать оповещение** будет недоступен для выбора.
6. Выберите **+ создать оповещение** , чтобы создать оповещение журнала.
7. В разделе **условие** выберите **каждый раз, \<logic undefined> когда пользовательский поиск по журналам** заранее задает условие пользовательского журнала. Тип сигнала **поиска пользовательских журналов** выбирается автоматически, так как мы создаем правило оповещения непосредственно на странице журналов Azure Monitor.  
8. Вставьте один из [запросов](#resource-utilization-log-search-queries) , приведенных выше, в поле **поискового запроса** .
9. Настройте оповещение следующим образом.

    1. В раскрывающемся списке **На основе** выберите **Измерение метрик**. Измерение метрики создает предупреждение для каждого объекта в запросе, значение которого превышает указанный порог.
    1. В поле **условие** выберите **больше** и введите **75** в качестве начального **порогового значения** для оповещений об использовании ЦП и памяти. В поле оповещение о нехватке места на диске введите **90**. Или введите другое значение, соответствующее вашим критериям.
    1. В разделе **триггер оповещений на основе** выберите пункт **последовательные нарушения**. В раскрывающемся списке выберите **больше** и введите **2**.
    1. Чтобы настроить оповещение о загрузке ЦП или памяти контейнера, в разделе **Статистическая обработка** выберите **ContainerName**. Чтобы настроить оповещение о низком диске узла кластера, выберите **ClusterId**.
    1. В разделе **Оценка на основе** установите значение **периода** **60 минут**. Правило будет выполняться каждые 5 минут и вернет записи, которые были созданы за последний час с текущего времени. Задание периода времени для расширенных учетных записей для возможной задержки данных. Это также гарантирует, что запрос возвращает данные, чтобы избежать ложного отрицательного значения, при котором предупреждение никогда не срабатывает.

10. Нажмите кнопку **Готово** , чтобы завершить правило генерации оповещений.
11. Введите имя в поле **имя правила генерации оповещений** . Укажите **Описание** , которое предоставляет сведения о предупреждении. И выберите подходящий уровень серьезности из предоставленных параметров.
12. Чтобы немедленно активировать правило генерации оповещений, примите значение по умолчанию для параметра **включить правило при создании**.
13. Выберите существующую **группу действий** или создайте новую. Этот шаг обеспечивает выполнение одних и тех же действий при каждом запуске оповещения. Настройте в зависимости от того, как ИТ или группа DevOps управляют инцидентами.
14. Выберите **создать правило генерации оповещений** , чтобы завершить правило генерации оповещений. Оно начнет выполняться немедленно.

## <a name="next-steps"></a>Дальнейшие действия

- Просмотрите [примеры запросов журналов](container-insights-log-search.md#search-logs-to-analyze-data) , чтобы просмотреть предварительно определенные запросы и примеры для проверки или настройки предупреждений, визуализации или анализа кластеров.

- Дополнительные сведения о Azure Monitor и мониторинге других аспектов кластера Kubernetes см. в статье [Просмотр производительности кластера Kubernetes](container-insights-analyze.md) и [Просмотр работоспособности кластера Kubernetes](./container-insights-overview.md).