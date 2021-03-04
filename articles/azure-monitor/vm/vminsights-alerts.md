---
title: Оповещения из VM Insights
description: Описание создания правил генерации оповещений на основе данных о производительности, собираемых службой "аналитика виртуальных машин".
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/10/2020
ms.openlocfilehash: 06c58b7081ed68724a3c907f8fe76dcf5f7b8057
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102046811"
---
# <a name="how-to-create-alerts-from-vm-insights"></a>Создание оповещений из VM Insights
[Оповещения в Azure Monitor](../alerts/alerts-overview.md) заранее уведомляют Вас о интересных данных и закономерностях в данных мониторинга. В базе данных VM Insights не включены предварительно настроенные правила генерации оповещений, но вы можете создавать собственные данные на основе собираемых. В этой статье приводятся рекомендации по созданию правил генерации оповещений, включая набор образцов запросов.

> [!IMPORTANT]
> Предупреждения, описанные в этой статье, основываются на запросах журналов от данных, собранных VM Insights. Это отличается от предупреждений, созданных [Azure Monitor для работоспособности гостевой виртуальной машины](vminsights-health-overview.md) , которая в настоящее время доступна в общедоступной предварительной версии. Так как эта функция приближается к общедоступной, руководство по оповещениям будет объединено.


## <a name="alert-rule-types"></a>Типы правил генерации оповещений
Azure Monitor имеет [различные типы правил генерации оповещений](../alerts/alerts-overview.md#what-you-can-alert-on) на основе данных, используемых для создания предупреждения. Все данные, собираемые VM Insights, хранятся в журналах Azure Monitor, которые поддерживают [оповещения журнала](../alerts/alerts-log.md). В настоящее время вы не можете использовать [оповещения метрик](../alerts/alerts-log.md) с данными о производительности, собранными из Application Insights, так как данные не собираются в метрики Azure Monitor. Чтобы получить данные для оповещений метрик, установите [расширение системы диагностики](../agents/diagnostics-extension-overview.md) для виртуальных машин Windows или [Агент Telegraf](../essentials/collect-custom-metrics-linux-telegraf.md) для виртуальных машин Linux, чтобы получить данные о производительности в метриках.

Существует два типа оповещений журнала в Azure Monitor:

- [Оповещение о количестве результатов](../alerts/alerts-unified-log.md#count-of-the-results-table-rows) создает одно предупреждение, когда запрос возвращает хотя бы указанное число записей. Они идеально подходят для нецифровых данных, таких как события Windows и syslog, собираемые [агентом log Analytics](../agents/log-analytics-agent.md) или для анализа тенденций производительности на нескольких компьютерах.
- [Оповещения измерения метрик](../alerts/alerts-unified-log.md#calculation-of-measure-based-on-a-numeric-column-such-as-cpu-counter-value) создают отдельное оповещение для каждой записи в запросе со значением, превышающим пороговое значение, определенное в правиле генерации оповещений. Эти правила генерации оповещений идеально подходят для данных о производительности, собираемых службой "аналитика виртуальных машин", так как они могут создавать отдельные предупреждения для каждого компьютера.


## <a name="alert-rule-walkthrough"></a>Пошаговое руководство по правилу оповещения
В этом разделе рассматривается создание правила генерации оповещений измерения метрик с использованием данных о производительности из VM Insights. Этот базовый процесс можно использовать с различными запросами журнала для оповещения о различных счетчиках производительности.

Начните с создания нового правила генерации оповещений, следуя процедуре [Создание, просмотр и Управление оповещениями журнала с помощью Azure Monitor](../alerts/alerts-log.md). Для **ресурса** выберите рабочую область log Analytics, в которой Azure Monitor виртуальные машины используются в вашей подписке. Так как целевой ресурс для правил генерации оповещений журнала всегда является Log Analytics рабочей областью, запрос журнала должен включать любой фильтр для определенных виртуальных машин или масштабируемых наборов виртуальных машин. 

Для **условия** правила генерации оповещений используйте один из запросов в [разделе ниже](#sample-alert-queries) в качестве **поискового запроса**. Запрос должен возвращать числовое свойство с именем *AggregatedValue*. Он должен суммировать данные по компьютерам, чтобы можно было создать отдельное оповещение для каждой виртуальной машины, которая превышает пороговое значение.

В **логике оповещений** выберите **измерение метрик** , а затем укажите **пороговое значение**. В поле **активировать оповещение на основе** укажите, сколько раз должно быть превышено пороговое значение, прежде чем будет создано оповещение. Например, вы, вероятно, не следите за тем, что процессор превышает пороговое значение, а затем возвращается к обычному, но вы следите за тем, чтобы он продолжал превысить пороговое значение для нескольких последовательных измерений.

В разделе, **основанном на оценке** , определяется, как часто выполняется запрос, и окно времени для запроса. В приведенном ниже примере запрос будет выполняться каждые 15 минут и оценивать значения производительности, собранные за предыдущие 15 минут.


![Правило генерации оповещений для измерения метрик](media/vminsights-alerts/metric-measurement-alert.png)

## <a name="sample-alert-queries"></a>Примеры запросов предупреждений
Следующие запросы можно использовать с правилом измерения метрик с помощью данных о производительности, собираемых службой "аналитика VM". Каждый из них обобщает данные по компьютерам, чтобы создать оповещение для каждого компьютера со значением, превышающим пороговое значение.

### <a name="cpu-utilization"></a>загрузка ЦП;

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "Processor" and Name == "UtilizationPercentage" 
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="available-memory-in-mb"></a>Доступная память (МБ)

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Memory" and Name == "AvailableMB"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="available-memory-in-percentage"></a>Объем доступной памяти в процентах

```kusto
InsightsMetrics 
| where Origin == "vm.azm.ms" 
| where Namespace == "Memory" and Name == "AvailableMB" 
| extend TotalMemory = toreal(todynamic(Tags)["vm.azm.ms/memorySizeMB"])
| extend AvailableMemoryPercentage = (toreal(Val) / TotalMemory) * 100.0 
| summarize AggregatedValue = avg(AvailableMemoryPercentage) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="logical-disk-used---all-disks-on-each-computer"></a>Используемый логический диск — все диски на каждом компьютере

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "LogicalDisk" and Name == "FreeSpacePercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="logical-disk-used---individual-disks"></a>Используемый логический диск — отдельные диски

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "LogicalDisk" and Name == "FreeSpacePercentage"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, Disk
```

### <a name="logical-disk-iops"></a>Логические диски, операций ввода-вывода

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "LogicalDisk" and Name == "TransfersPerSecond"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m) ), Computer, _ResourceId, Disk
```

### <a name="logical-disk-data-rate"></a>Скорость данных логического диска

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "LogicalDisk" and Name == "BytesPerSecond"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m) , Computer, _ResourceId, Disk
```

### <a name="network-interfaces-bytes-received---all-interfaces"></a>Получено байт сетевых интерфейсов — все интерфейсы

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "ReadBytesPerSecond"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="network-interfaces-bytes-received---individual-interfaces"></a>Полученные байты сетевых интерфейсов — индивидуальные интерфейсы

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "ReadBytesPerSecond"
| extend NetworkInterface=tostring(todynamic(Tags)["vm.azm.ms/networkDeviceId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, NetworkInterface
```

### <a name="network-interfaces-bytes-sent---all-interfaces"></a>Отправлено байт сетевых интерфейсов-все интерфейсы

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "WriteBytesPerSecond"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="network-interfaces-bytes-sent---individual-interfaces"></a>Отправлено байт сетевых интерфейсов — индивидуальные интерфейсы

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "WriteBytesPerSecond"
| extend NetworkInterface=tostring(todynamic(Tags)["vm.azm.ms/networkDeviceId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, NetworkInterface
```

### <a name="virtual-machine-scale-set"></a>Набор масштабирования виртуальных машин
Измените с помощью идентификатора подписки, группы ресурсов и имени масштабируемого набора виртуальных машин.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachineScaleSets/my-vm-scaleset" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId
```

### <a name="specific-virtual-machine"></a>Конкретная виртуальная машина
Измените с помощью идентификатора подписки, группы ресурсов и имени виртуальной машины.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId =~ "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/my-vm" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m)
```

### <a name="cpu-utilization-for-all-compute-resources-in-a-subscription"></a>Загрузка ЦП для всех вычислительных ресурсов в подписке
Измените с помощью идентификатора подписки.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" and (_ResourceId contains "/providers/Microsoft.Compute/virtualMachines/" or _ResourceId contains "/providers/Microsoft.Compute/virtualMachineScaleSets/")
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId
```

### <a name="cpu-utilization-for-all-compute-resources-in-a-resource-group"></a>Загрузка ЦП для всех вычислительных ресурсов в группе ресурсов
Измените с помощью идентификатора подписки и группы ресурсов.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/"
or _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachineScaleSets/" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId

```

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения об [оповещениях см. в Azure Monitor](../alerts/alerts-overview.md).
- Дополнительные сведения о [запросах журналов с помощью данных из VM Insights](vminsights-log-search.md).
