---
title: Azure Service Fabric автоматическое масштабирование служб и контейнеров
description: Azure Service Fabric позволяет устанавливать политики автомасштабирования для служб и контейнеров.
author: radicmilos
ms.topic: conceptual
ms.date: 04/17/2018
ms.author: miradic
ms.custom: devx-track-csharp
ms.openlocfilehash: 060bb9dcdd504846c76ab4c782b2857fdddfa394
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91354808"
---
# <a name="introduction-to-auto-scaling"></a>Знакомство с автомасштабированием
Автомасштабирование — это дополнительная возможность службы Service Fabric, позволяющая динамически масштабировать службы на основе нагрузки, предоставляемой службами, или на основе использования ресурсов. Автомасштабирование предоставляет большую эластичность и позволяет подготовить дополнительные экземпляры или разделы службы по запросу. Весь процесс автомасштабирования автоматизирован и прозрачен, и после настройки политик в службе нет необходимости в выполнении масштабирования на уровне службы вручную. Автомасштабирование можно включить во время создания службы или в любое другое время путем обновления службы.

Автомасштабирование необходимо выполнять, если нагрузка на конкретную службу изменяется со временем. Например, такая служба, как шлюз, может масштабироваться на основе количества ресурсов, необходимых для обработки входящих запросов. Давайте рассмотрим пример таких правил масштабирования.
* Если все экземпляры шлюза используют в среднем более двух ядер, масштабируйте службу шлюза, добавив еще один экземпляр. Добавляйте экземпляр каждый час, но в целом не более семи экземпляров.
* Если все экземпляры шлюза в среднем используют менее 0,5 ядра, масштабируйте службу, удаляя один экземпляр. Удаляйте экземпляр каждый час, оставив при этом не менее трех экземпляров.

Автоматическое масштабирование поддерживается как для контейнеров, так и для обычных служб Service Fabric. Чтобы использовать автоматическое масштабирование, вам понадобится среда выполнения Service Fabric версии 6.2 или более поздней. 

В остальной части этой статьи описаны политики масштабирования, способы включения или отключения автомасштабирования, а также приведены примеры использования этой функции.

## <a name="describing-auto-scaling"></a>Описание автомасштабирования
Для каждой службы в кластере Service Fabric можно определить политики автомасштабирования. Каждая политика масштабирования состоит из двух частей:
* **Триггер масштабирования** описывает, когда будет выполняться масштабирование службы. Условия, определенные в триггере, периодически проверяются, чтобы определить, нужно ли масштабировать службу.

* **Механизм масштабирования** описывает способ масштабирования при запуске. Механизм применяется только при выполнении условий триггера.

Все поддерживаемые в настоящее время триггеры работают либо с [логическими метриками нагрузки](service-fabric-cluster-resource-manager-metrics.md), либо с физическими метриками, такими как использование ЦП или памяти. В любом случае Service Fabric будет отслеживать метрики сообщаемой нагрузки и периодически оценивать триггер, чтобы определить, требуется ли масштабирование.

В настоящее время для автомасштабирования поддерживаются два механизма. Первый предназначен для служб без отслеживания состояния или для контейнеров, в которых автомасштабирование выполняется путем добавления или удаления [экземпляров](service-fabric-concepts-replica-lifecycle.md). Для служб с отслеживанием и без отслеживания состояния автомасштабирование можно также выполнять путем добавления или удаления именованных [разделов](service-fabric-concepts-partitioning.md) службы.

> [!NOTE]
> Сейчас поддерживается только одна политика масштабирования на службу и только один триггер масштабирования на политику масштабирования.

## <a name="average-partition-load-trigger-with-instance-based-scaling"></a>Среднее время загрузки триггеров раздела при масштабировании на основе экземпляра
Первый тип триггера зависит от нагрузки экземпляров в разделе службы без отслеживания состояния. Загрузка метрик сначала сглаживается, чтобы получить данные о загрузке для каждого экземпляра раздела, а затем эти значения усредняются по всем экземплярам раздела. Существует три факторов, определяющих масштабирование службы.

* _Понижение порога загрузки_ — это значение, которое определяет, когда служба будет **масштабироваться в**. Если среднее время загрузки всех экземпляров разделов ниже этого значения, служба свернется.
* _Верхний предел загрузки_ — это значение, определяющее, когда служба будет **масштабироваться**. Если средняя нагрузка всех экземпляров секции выше этого значения, то служба будет масштабироваться.
* _Интервал масштабирования_ определяет, как часто будет проверяться триггер. После проверки триггера, если необходимо масштабирование, будет применен соответствующий механизм. Если масштабирование не требуется, никакие действия выполняться не будут. В обоих случаях до истечения интервала масштабирования триггер снова проверяться не будет.

Этот триггер можно использовать только со службами без отслеживания состояния (контейнерами без отслеживания состояния или службами Service Fabric). В случае, когда служба имеет несколько разделов, триггер оценивается для каждого раздела отдельно и к каждому разделу независимо применяется указанный механизм. Следовательно, в этом случае возможно, что некоторые разделы службы будут развернуты, некоторые — свернуты, а некоторые не будут масштабироваться одновременно, в зависимости от их нагрузки.

PartitionInstanceCountScaleMechanism — это единственный механизм, который можно использовать с этим триггером. Существует три фактора, определяющие способ применения механизма.
* _Приращение масштаба_ определяет количество экземпляров, которые будут добавлены или удалены после запуска механизма.
* _Максимальное число экземпляров_ определяет верхний предел для масштабирования. Если число экземпляров раздела достигает этого предела, служба не будет масштабироваться, независимо от нагрузки. Этот предел можно опустить, указав значение -1, и в этом случае служба будет масштабироваться (пределом является количество доступных в кластере узлов).
* _Минимальное число экземпляров_ определяет нижнюю границу масштабирования. Если число экземпляров раздела достигает этого предела, служба не будет свертываться, независимо от загрузки.

## <a name="setting-auto-scaling-policy-for-instance-based-scaling"></a>Настройка политики автоматического масштабирования для масштабирования на основе экземпляра

### <a name="using-application-manifest"></a>Использование манифеста приложения
``` xml
<LoadMetrics>
<LoadMetric Name="MetricB" Weight="High"/>
</LoadMetrics>
<ServiceScalingPolicies>
<ScalingPolicy>
    <AveragePartitionLoadScalingTrigger MetricName="MetricB" LowerLoadThreshold="1" UpperLoadThreshold="2" ScaleIntervalInSeconds="100"/>
    <InstanceCountScalingMechanism MinInstanceCount="3" MaxInstanceCount="4" ScaleIncrement="1"/>
</ScalingPolicy>
</ServiceScalingPolicies>
```
### <a name="using-c-apis"></a>Использование программных интерфейсов C#
```csharp
FabricClient fabricClient = new FabricClient();
StatelessServiceDescription serviceDescription = new StatelessServiceDescription();
//set up the rest of the ServiceDescription
AveragePartitionLoadScalingTrigger trigger = new AveragePartitionLoadScalingTrigger();
PartitionInstanceCountScaleMechanism mechanism = new PartitionInstanceCountScaleMechanism();
mechanism.MaxInstanceCount = 3;
mechanism.MinInstanceCount = 1;
mechanism.ScaleIncrement = 1;
trigger.MetricName = "servicefabric:/_CpuCores";
trigger.ScaleInterval = TimeSpan.FromMinutes(20);
trigger.LowerLoadThreshold = 1.0;
trigger.UpperLoadThreshold = 2.0;
ScalingPolicyDescription policy = new ScalingPolicyDescription(mechanism, trigger);
serviceDescription.ScalingPolicies.Add(policy);
//as we are using scaling on a resource this must be exclusive service
//also resource monitor service needs to be enabled
serviceDescription.ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```
### <a name="using-powershell"></a>Использование PowerShell
```posh
$mechanism = New-Object -TypeName System.Fabric.Description.PartitionInstanceCountScaleMechanism
$mechanism.MinInstanceCount = 1
$mechanism.MaxInstanceCount = 6
$mechanism.ScaleIncrement = 2
$trigger = New-Object -TypeName System.Fabric.Description.AveragePartitionLoadScalingTrigger
$trigger.MetricName = "servicefabric:/_CpuCores"
$trigger.LowerLoadThreshold = 0.3
$trigger.UpperLoadThreshold = 0.8
$trigger.ScaleInterval = New-TimeSpan -Minutes 10
$scalingpolicy = New-Object -TypeName System.Fabric.Description.ScalingPolicyDescription
$scalingpolicy.ScalingMechanism = $mechanism
$scalingpolicy.ScalingTrigger = $trigger
$scalingpolicies = New-Object 'System.Collections.Generic.List[System.Fabric.Description.ScalingPolicyDescription]'
$scalingpolicies.Add($scalingpolicy)
#as we are using scaling on a resource this must be exclusive service
#also resource monitor service needs to be enabled
Update-ServiceFabricService -Stateless -ServiceName "fabric:/AppName/ServiceName" -ScalingPolicies $scalingpolicies
```

## <a name="average-service-load-trigger-with-partition-based-scaling"></a>Среднее время загрузки триггеров службы при масштабировании на основе раздела
Второй триггер основан на загрузке всех разделов одной службы. Загрузка метрик сначала сглаживается, чтобы получить данные о загрузке для каждой реплики или экземпляра раздела. Для службы с отслеживанием состояния загрузка раздела считается загрузкой первичной реплики, тогда как для служб без отслеживания состояния загрузка раздела — это среднее время загрузки всех экземпляров раздела. Это средние значения для всех разделов службы. Они используются для запуска автомасштабирования. Как и в предыдущем случае, три фактора определяют, когда будет масштабироваться служба:

* _Понижение порога загрузки_ — это значение, которое определяет, когда служба будет **масштабироваться в**. Если среднее время загрузки всех разделов службы ниже этого значения, служба свернется.
* _Верхний предел загрузки_ — это значение, определяющее, когда служба будет **масштабироваться**. Если средняя загрузка всех секций службы выше этого значения, то служба будет масштабироваться.
* _Интервал масштабирования_ определяет, как часто будет проверяться триггер. После проверки триггера, если необходимо масштабирование, будет применен соответствующий механизм. Если масштабирование не требуется, никакие действия выполняться не будут. В обоих случаях до истечения интервала масштабирования триггер снова проверяться не будет.

Этот триггер может использоваться со службами с отслеживанием и без отслеживания состояния. AddRemoveIncrementalNamedPartitionScalingMechanism — это единственный механизм, который можно использовать с этим триггером. При развертывании службы добавляется новый раздел, а при свертывании — один из имеющихся разделов удаляется. Существуют ограничения, которые необходимо учитывать при создании или обновлении службы. Если эти условия не будут выполнены, создание или обновление закончится ошибкой.
* Для службы должна использоваться именованная схема секционирования.
* Имена разделов должны состоять из последовательных целых чисел, например 0, 1...
* Первый раздел должен иметь следующее имя: "0".

Например, если служба изначально создана с тремя разделами, единственные допустимые имена разделов — "0", "1" и "2".

Фактическая операция автомасштабирования также будет учитывать эту схему именования.
* Если текущие разделы службы называются "0", "1" и "2", для выполнения развертывания будет добавлен раздел с именем "3".
* Если текущие разделы службы называются "0", "1" и "2", раздел с именем "2" будет удален для выполнения свертывания.

Как и в случае с механизмом, выполняющим масштабирование путем добавления или удаления экземпляров, способ применения механизма определяют три параметра.
* _Приращение масштаба_ определяет количество разделов, которые будут добавлены или удалены после запуска механизма.
* _Максимальное число разделов_ определяет верхний предел для масштабирования. Если число разделов службы достигает этого предела, служба не будет развертываться, независимо от нагрузки. Этот предел можно опустить, указав значение -1, и в этом случае служба будет масштабироваться (пределом является значение фактической загрузки кластера).
* _Минимальное число экземпляров_ определяет нижнюю границу масштабирования. Если число разделов службы достигает этого предела, служба не будет свертываться, независимо от нагрузки.

> [!WARNING] 
> Если AddRemoveIncrementalNamedPartitionScalingMechanism используется со службами с отслеживанием состояния, разделы в Service Fabric добавляются или удаляются **без уведомления либо предупреждения**. Если механизм масштабирования запущен, перераспределение разделов данных не будет выполняться. В случае операции масштабирования новые секции будут пустыми, и в случае операции масштабирования **Секция будет удалена вместе со всеми данными, которые он содержит**.

## <a name="setting-auto-scaling-policy-for-partition-based-scaling"></a>Настройка политики автоматического масштабирования для масштабирования на основе секционирования

### <a name="using-application-manifest"></a>Использование манифеста приложения
``` xml
<NamedPartition>
    <Partition Name="0" />
</NamedPartition>
<ServiceScalingPolicies>
    <ScalingPolicy>
        <AverageServiceLoadScalingTrigger MetricName="servicefabric:/_MemoryInMB" LowerLoadThreshold="300" UpperLoadThreshold="500" ScaleIntervalInSeconds="600"/>
        <AddRemoveIncrementalNamedPartitionScalingMechanism MinPartitionCount="1" MaxPartitionCount="3" ScaleIncrement="1"/>
    </ScalingPolicy>
</ServiceScalingPolicies>
```
### <a name="using-c-apis"></a>Использование программных интерфейсов C#
```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceUpdateDescription serviceUpdate = new StatefulServiceUpdateDescription();
AveragePartitionLoadScalingTrigger trigger = new AverageServiceLoadScalingTrigger();
PartitionInstanceCountScaleMechanism mechanism = new AddRemoveIncrementalNamedPartitionScalingMechanism();
mechanism.MaxPartitionCount = 4;
mechanism.MinPartitionCount = 1;
mechanism.ScaleIncrement = 1;
//expecting that the service already has metric NumberOfConnections
trigger.MetricName = "NumberOfConnections";
trigger.ScaleInterval = TimeSpan.FromMinutes(15);
trigger.LowerLoadThreshold = 10000;
trigger.UpperLoadThreshold = 20000;
ScalingPolicyDescription policy = new ScalingPolicyDescription(mechanism, trigger);
serviceUpdate.ScalingPolicies = new List<ScalingPolicyDescription>;
serviceUpdate.ScalingPolicies.Add(policy);
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/AppName/ServiceName"), serviceUpdate);
```
### <a name="using-powershell"></a>Использование PowerShell
```posh
$mechanism = New-Object -TypeName System.Fabric.Description.AddRemoveIncrementalNamedPartitionScalingMechanism
$mechanism.MinPartitionCount = 1
$mechanism.MaxPartitionCount = 3
$mechanism.ScaleIncrement = 2
$trigger = New-Object -TypeName System.Fabric.Description.AverageServiceLoadScalingTrigger
$trigger.MetricName = "servicefabric:/_MemoryInMB"
$trigger.LowerLoadThreshold = 5000
$trigger.UpperLoadThreshold = 10000
$trigger.ScaleInterval = New-TimeSpan -Minutes 25
$scalingpolicy = New-Object -TypeName System.Fabric.Description.ScalingPolicyDescription
$scalingpolicy.ScalingMechanism = $mechanism
$scalingpolicy.ScalingTrigger = $trigger
$scalingpolicies = New-Object 'System.Collections.Generic.List[System.Fabric.Description.ScalingPolicyDescription]'
$scalingpolicies.Add($scalingpolicy)
#as we are using scaling on a resource this must be exclusive service
#also resource monitor service needs to be enabled
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -TargetReplicaSetSize 3 -MinReplicaSetSize 2 -HasPersistedState true -PartitionNames @("0","1") -ServicePackageActivationMode ExclusiveProcess -ScalingPolicies $scalingpolicies
```

## <a name="auto-scaling-based-on-resources"></a>Автомасштабирование на основе ресурсов

Чтобы позволить службе мониторинга ресурсов масштабироваться на основе фактических ресурсов, сделайте следующее:

``` json
"fabricSettings": [
...      
],
"addonFeatures": [
    "ResourceMonitorService"
],
```
Существуют две метрики, которые представляют фактические физические ресурсы. Один из них — servicefabric:/_CpuCores, представляющий собой фактическое использование ЦП (поэтому представляет собой половину ядра), а другой — servicefabric:/_MemoryInMB, который представляет собой использование памяти в МБ.
Служба ResourceMonitorService отвечает за отслеживание использования ЦП и памяти служб обслуживания пользователей. Эта служба будет применять взвешенное скользящее среднее для учета потенциальных кратковременных пиков. Мониторинг ресурсов поддерживается для контейнерных и неконтейнерных приложений в Windows, а также для контейнерных приложений — в Linux. Автомасштабирование ресурсов доступно только для служб, активированных в [модели с монопольным процессом](service-fabric-hosting-model.md#exclusive-process-model).

## <a name="next-steps"></a>Дальнейшие действия
[Масштабирование в Service Fabric](service-fabric-concepts-scalability.md).
