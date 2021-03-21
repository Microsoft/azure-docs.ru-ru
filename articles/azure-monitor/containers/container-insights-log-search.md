---
title: Запрос журналов из аналитики контейнера | Документация Майкрософт
description: Аналитика контейнера собирает метрики и данные журнала. в этой статье описываются записи и примеры запросов.
ms.topic: conceptual
ms.date: 03/03/2021
ms.openlocfilehash: c2b7331255e1109f27f89a84d66e25eb07a20569
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102201385"
---
# <a name="how-to-query-logs-from-container-insights"></a>Запрос журналов из аналитики контейнера

Эта аналитика собирает метрики производительности, данные инвентаризации и сведения о состоянии работоспособности из узлов контейнеров и контейнеров. Данные собираются каждые три минуты и пересылаются в рабочую область Log Analytics в Azure Monitor. Эти данные доступны для [запроса](../logs/log-query-overview.md) в Azure Monitor. Эти данные используются в различных сценариях, таких как планирование миграции, анализ емкости, обнаружение и устранение проблем с производительностью по требованию.

## <a name="container-records"></a>Записи контейнеров

В следующей таблице представлены сведения о записях, собираемых службой "аналитика контейнеров". Список описаний столбцов см. в справочнике по таблицам [контаинеринвентори](/azure/azure-monitor/reference/tables/containerinventory) и [контаинерлог](/azure/azure-monitor/reference/tables/containerlog) .

| Данные | Источник данных | Тип данных | Поля |
|------|-------------|-----------|--------|
| Список контейнеров | kubelet | `ContainerInventory` | TimeGenerated, Computer, Name, Контаинерхостнаме, Image, Имажетаг, Контаинерстате, ExitCode, Енвиронментвар, Command, время создания, Стартедтиме, Финишедтиме, Sourcesystem имеет значение, ContainerID, ImageID |
| Журнал контейнеров | Docker | `ContainerLog` | TimeGenerated, Computer, Image ID, Name, Ложентрисаурце, LogEntry, Sourcesystem имеет значение, ContainerID |
| Список узлов контейнеров | API KUBE | `ContainerNodeInventory`| TimeGenerated, Computer, ClassName_s, DockerVersion_s, OperatingSystem_s, Volume_s, Network_s, NodeRole_s, OrchestratorType_s, InstanceID_g, SourceSystem|
| Список модулей pod в кластере Kubernetes | API KUBE | `KubePodInventory` | TimeGenerated, Computer, ClusterId, Контаинеркреатионтиместамп, Подуид, Подкреатионтиместамп, Контаинеррестарткаунт, Подрестарткаунт, PodStartTime, ContainerStartTime, ServiceName, ControllerKind, ControllerName, ContainerStatus, ContainerStatusReason, ContainerID, ContainerName, Name, PodLabel, Namespace, PodStatus, имя_кластера, модуля, Sourcesystem имеет значение |
| Список узлов кластера Kubernetes | API KUBE | `KubeNodeInventory` | TimeGenerated, Computer, ClusterName, ClusterId, LastTransitionTimeReady, Labels, Status, KubeletVersion, KubeProxyVersion, CreationTimeStamp, SourceSystem | 
|Инвентаризация постоянных томов в кластере Kubernetes |API KUBE |`KubePVInventory` | TimeGenerated, Пвнаме, ПвкапаЦитибитес, Пвкнаме, Пвкнамеспаце, Пвстатус, Пвакцессмодес, Пвтипе, Пвтипеинфо, PVStorageClassName, PVCreationTimestamp, ClusterId, имя_кластера, _ResourceId, Sourcesystem имеет значение |
| События Kubernetes. | API KUBE | `KubeEvents` | TimeGenerated, Computer, ClusterId_s, FirstSeen_t, LastSeen_t, Count_d, ObjectKind_s, Namespace_s, Name_s, Reason_s, Type_s, TimeGenerated_s, SourceComponent_s, ClusterName_s, Message,  SourceSystem | 
| Службы в кластере Kubernetes | API KUBE | `KubeServices` | TimeGenerated, ServiceName_s, Namespace_s, SelectorLabels_s, ClusterId_s, ClusterName_s, ClusterIP_s, ServiceType_s, SourceSystem | 
| Метрики производительности для узлов кластера Kubernetes | Метрики использования получены от cAdvisor и от ограничений API KUBE. | `Perf \| where ObjectName == "K8SNode"` | Computer, ObjectName, CounterName &#40;Кпуаллокатабленанокорес, Меморяллокатаблебитес, КпукапаЦитинанокорес, МеморикапаЦитибитес, memoryRssBytes, cpuUsageNanoCores, memoryWorkingsetBytes, restartTimeEpoch&#41;, CounterValue, TimeGenerated, CounterPath, Sourcesystem имеет значение | 
| Метрики производительности контейнеров кластера Kubernetes | Метрики использования получены от cAdvisor и от ограничений API KUBE. | `Perf \| where ObjectName == "K8SContainer"` | CounterName &#40;Кпурекуестнанокорес, Меморирекуестбитес, Кпулимитнанокорес, Мемориворкингсетбитес, restartTimeEpoch, cpuUsageNanoCores, memoryRssBytes&#41;, CounterValue, TimeGenerated, CounterPath, Sourcesystem имеет значение | 
| Пользовательские метрики ||`InsightsMetrics` | Компьютер, имя, пространство имен, источник, Sourcesystem имеет значение, теги<sup>1</sup>, timegenerated, тип, ва, _ResourceId | 

<sup>1</sup> свойство *Tags* представляет [несколько измерений](../essentials/data-platform-metrics.md#multi-dimensional-metrics) для соответствующей метрики. Дополнительные сведения о метриках, собираемых и хранящихся в `InsightsMetrics` таблице, а также описание свойств записи см. в разделе [инсигхтсметрикс Overview](https://github.com/microsoft/OMS-docker/blob/vishwa/june19agentrel/docs/InsightsMetrics.md).

## <a name="search-logs-to-analyze-data"></a>Поиск по журналам для анализа данных

Журналы Azure Monitor могут помочь в поиске тенденций, диагностике узких мест, прогнозировании или корреляции данных, которые помогут определить, оптимально ли выполняется текущая конфигурация кластера. Вам доступны предопределенные запросы поиска по журналам, которые можно использовать без изменений или настроить для получения сведений нужным вам образом.

Анализ данных в рабочей области можно выполнять в интерактивном режиме, выбрав в раскрывающемся списке **Просмотр в аналитике** параметр **Просмотреть журналы событий Kubernetes** или **Просмотреть журналы контейнеров** . Страница **поиска по журналам** появится в правой области страницы портала Azure, которую вы использовали.

![Анализ данных в Log Analytics](./media/container-insights-analyze/container-health-log-search-example.png)

Контейнеры выводят журналы, перенаправляемые в рабочую область — STDOUT и STDERR. Так как Azure Monitor является мониторингом управляемого Azure Kubernetes (AKS), KUBE-System не собирается сегодня из-за большого объема созданных данных. 

### <a name="example-log-search-queries"></a>Примеры запросов для поиска по журналам

При создании запросов часто бывает полезно начать с одного-двух примеров, внося затем в них изменения в соответствии со своими требованиями. Можно поэкспериментировать с приведенными ниже примерами запросов, чтобы научиться создавать более сложные запросы.

### <a name="list-all-of-a-containers-lifecycle-information"></a>Вывод всех сведений о жизненном цикле контейнера

```kusto
ContainerInventory
| project Computer, Name, Image, ImageTag, ContainerState, CreatedTime, StartedTime, FinishedTime
| render table
```

### <a name="kubernetes-events"></a>События Kubernetes

``` kusto
KubeEvents_CL
| where not(isempty(Namespace_s))
| sort by TimeGenerated desc
| render table
```
### <a name="image-inventory"></a>Инвентаризация образа

``` kusto
ContainerImageInventory
| summarize AggregatedValue = count() by Image, ImageTag, Running
```

### <a name="container-cpu"></a>Ресурсы ЦП контейнера.

**Выберите параметр Показать график**

``` kusto
Perf
| where ObjectName == "K8SContainer" and CounterName == "cpuUsageNanoCores" 
| summarize AvgCPUUsageNanoCores = avg(CounterValue) by bin(TimeGenerated, 30m), InstanceName 
```

### <a name="container-memory"></a>Память контейнера

**Выберите параметр Показать график**

```kusto
Perf
| where ObjectName == "K8SContainer" and CounterName == "memoryRssBytes"
| summarize AvgUsedRssMemoryBytes = avg(CounterValue) by bin(TimeGenerated, 30m), InstanceName
```

### <a name="requests-per-minute-with-custom-metrics"></a>Количество запросов в минуту с пользовательскими метриками

```kusto
InsightsMetrics
| where Name == "requests_count"
| summarize Val=any(Val) by TimeGenerated=bin(TimeGenerated, 1m)
| sort by TimeGenerated asc<br> &#124; project RequestsPerMinute = Val - prev(Val), TimeGenerated
| render barchart 
```

## <a name="query-prometheus-metrics-data"></a>Запрос данных метрик Prometheus

В следующем примере показан запрос метрик Prometheus, отображающий операции чтения с диска в секунду на диск для каждого узла.

```
InsightsMetrics
| where Namespace == 'container.azm.ms/diskio'
| where TimeGenerated > ago(1h)
| where Name == 'reads'
| extend Tags = todynamic(Tags)
| extend HostName = tostring(Tags.hostName), Device = Tags.name
| extend NodeDisk = strcat(Device, "/", HostName)
| order by NodeDisk asc, TimeGenerated asc
| serialize
| extend PrevVal = iif(prev(NodeDisk) != NodeDisk, 0.0, prev(Val)), PrevTimeGenerated = iif(prev(NodeDisk) != NodeDisk, datetime(null), prev(TimeGenerated))
| where isnotnull(PrevTimeGenerated) and PrevTimeGenerated != TimeGenerated
| extend Rate = iif(PrevVal > Val, Val / (datetime_diff('Second', TimeGenerated, PrevTimeGenerated) * 1), iif(PrevVal == Val, 0.0, (Val - PrevVal) / (datetime_diff('Second', TimeGenerated, PrevTimeGenerated) * 1)))
| where isnotnull(Rate)
| project TimeGenerated, NodeDisk, Rate
| render timechart

```

Чтобы просмотреть метрики Prometheus, которые Azure Monitor отфильтрованы по пространству имен, укажите "Prometheus". Ниже приведен пример запроса для просмотра метрик Prometheus из `default` пространства имен kubernetes.

```
InsightsMetrics 
| where Namespace == "prometheus"
| extend tags=parse_json(Tags)
| summarize count() by Name
```

Данные Prometheus также можно запрашивать напрямую по имени.

```
InsightsMetrics 
| where Namespace == "prometheus"
| where Name contains "some_prometheus_metric"
```

### <a name="query-config-or-scraping-errors"></a>Настройка запроса или ошибки брака

Чтобы исследовать ошибки конфигурации или потери данных, следующий пример запроса возвращает информационные события из `KubeMonAgentEvents` таблицы.

```
KubeMonAgentEvents | where Level != "Info" 
```

В выходных данных отобразятся результаты, аналогичные приведенным в следующем примере:

![Регистрация результатов запроса информационных событий от агента](./media/container-insights-log-search/log-query-example-kubeagent-events.png)

## <a name="next-steps"></a>Дальнейшие действия

Контейнерная аналитика не включает предопределенный набор предупреждений. Ознакомьтесь с разработкой [оповещений о производительности с помощью Container Insights](./container-insights-log-alerts.md) , чтобы узнать, как создавать Рекомендуемые оповещения для высокой загрузки ЦП и памяти для поддержки DevOps или рабочих процессов и процедур.