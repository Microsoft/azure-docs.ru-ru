---
title: Вызов Chaos в кластерах Service Fabric
description: Управление Chaos в кластере с помощью интерфейсов API службы внесения ошибок и анализа кластера.
ms.topic: conceptual
ms.date: 02/05/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 72b8f7e9e4934b516f843ae8bc9bb7adc1c349ec
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101720516"
---
# <a name="induce-controlled-chaos-in-service-fabric-clusters"></a>Вызов контролируемого хаоса в кластерах Service Fabric
Крупномасштабные распределенные системы, такие как облачные инфраструктуры, ненадежны по своей сути. Azure Service Fabric позволяет разработчикам создавать надежные распределенные службы на основе ненадежной инфраструктуры. Чтобы писать надежные распределенные службы на основе ненадежной инфраструктуры, разработчики должны иметь возможность проверять стабильность работы своих служб, когда для базовой ненадежной инфраструктуры выполняются сложные переходы состояния из-за ошибок.

[Служба внесения ошибок и анализа кластера](./service-fabric-testability-overview.md) (также называемая службой анализа ошибок) позволяет разработчикам вызывать ошибки для тестирования работы служб. Эти целевые имитируемые ошибки, например [перезапуск секции](/powershell/module/servicefabric/start-servicefabricpartitionrestart), помогают выполнять самые распространенные переходы между состояниями. При этом целевым имитируемым ошибкам по определению свойственна погрешность, поэтому могут происходить ошибки, которые появляются только в трудной для прогнозирования, длинной и сложной последовательности переходов между состояниями. Для неискаженного тестирования без можно использовать Chaos.

Chaos моделирует в кластере периодические сбои, чередуя как нормальные, так и ненормальные ошибки, на протяжении долгого периода времени. Нормальная ошибка должна включать набор вызовов API Service Fabric. Например, перезапуск реплики является нормальной ошибкой, поскольку включает закрытие реплики с последующим открытием. Также Chaos применяет такие нормальные ошибки, как удаление реплики и перемещение первичной или вторичной реплики. Завершение процесса, например перезапуск узла или перезапуск пакета кода, считается ненормальной ошибкой. 

Настроив частоту и тип ошибок в инструменте Chaos, его можно запускать или останавливать через API-интерфейсы C#, PowerShell или REST API, чтобы моделировать сбои в работе кластера и службы. Можно настроить выполнение Chaos в течение указанного периода времени (например, на один час), после чего Chaos останавливается автоматически. Также можно в любой момент вызывать API StopChaos (C#, Powershell или REST) для его остановки.

> [!NOTE]
> Текущая версия Chaos вызывает только безопасные ошибки, то есть при отсутствии внешних ошибок исключена вероятность потери кворума или данных.
>

Во время выполнения Chaos создает различные события, которые записывают состояние выполнения в определенный момент времени. Например, ExecutingFaultsEvent содержит все сбои, которые Chaos выполняет в этой итерации. ValidationFailedEvent содержит сведения о сбое проверки (проблемы работоспособности и стабильности), обнаруженном во время проверки кластера. Для получения отчета о выполнении Chaos можно вызывать API GetChaosReportAsync (C#, Powershell или REST). Эти события сохраняются в [надежном словаре](./service-fabric-reliable-services-reliable-collections.md), для которого настроена политика усечения, диктуемая двумя конфигурациями: **MaxStoredChaosEventCount** (значение по умолчанию — 25 000) и **StoredActionCleanupIntervalInSeconds** (значение по умолчанию — 3600). Каждые *StoredActionCleanupIntervalInSeconds* Chaos выполняет проверку. В результате проверки определенное число последних событий (*MaxStoredChaosEventCount*) удаляется из надежного словаря.

## <a name="faults-induced-in-chaos"></a>Ошибки, вызываемые Chaos
Chaos генерирует сбои во всем кластере Service Fabric и вмещает сбои за месяцы или годы работы всего в несколько часов тестирования. Сочетание ошибок с чередованием с высокой частотой возникновения сбоев позволяет находить проблемы, которые иначе были бы упущены. Это упражнение Chaos обеспечивает значительное улучшение качества кода службы.

Chaos вызывает ошибки следующих категорий:

* Перезапуск узла
* Перезапуск развернутого пакета кода
* Удаление реплики
* Перезапуск реплики
* перемещение первичной реплики (настраивается);
* перемещение вторичной реплики (настраивается).

Chaos выполняет несколько итераций. Каждая из них включает в себя сбои и проверку кластера в течение указанного периода времени. Вы можете настроить время, затрачиваемое на стабилизацию работы и проверку кластера для успешной его работы. В случае обнаружения сбоя при проверке кластера Chaos создает и сохраняет событие ValidationFailedEvent с меткой времени в формате UTC и подробными сведениями об ошибке. Например, рассмотрим экземпляр Chaos, для которого настроено выполнение в течение одного часа и не более трех одновременных сбоев. Chaos вызывает три сбоя и проверяет работоспособность кластера. Затем он выполняет итерацию предыдущего шага, пока выполнение не будет явным образом остановлено посредством API StopChaosAsync или через один час. Если при какой-либо итерации кластер становится неработоспособным, то есть его работа не стабилизируется и не восстанавливается в течение заданного времени (MaxClusterStabilizationTimeout), Chaos создает событие ValidationFailedEvent. Это событие указывает на наличие ошибки, которую необходимо дополнительно изучить.

Чтобы узнать, какие сбои Chaos, можно использовать API GetChaosReport (PowerShell, C# или остальное). API возвращает следующий сегмент отчета Chaos, основанного на переданном маркере продолжения или диапазоне времени. Можно указать ContinuationToken для получения следующего сегмента отчета Chaos или указать диапазон времени с помощью параметров StartTimeUtc и EndTimeUtc, но нельзя указать ContinuationToken и диапазон времени одновременно. При наличии более 100 событий Chaos отчет Chaos возвращается по сегментам, каждый из которых содержит не более 100 событий.

## <a name="important-configuration-options"></a>Важные параметры конфигурации
* **TimeToRun**. Общее время выполнения Chaos до успешного завершения. Вы можете остановить выполнение Chaos до истечения периода TimeToRun с помощью API StopChaos.

* **MaxClusterStabilizationTimeout**. Максимальный период ожидания возврата кластера в работоспособное состояние до создания события ValidationFailedEvent. Это нужно, чтобы уменьшить нагрузку на кластер, пока его работоспособность восстанавливается. Выполняемые проверки:
  * работоспособен ли кластер;
  * работоспособна ли служба;
  * достигнут ли заданный размер целевого набора реплик для секции службы;
  * отсутствуют ли реплики InBuild.
* **MaxConcurrentFaults**. Максимальное количество одновременных сбоев, вызываемых при каждой итерации. Чем выше число, тем более агрессивно ведет себя Chaos, а комбинации отработок отказа и переходов состояния, которые проходит кластер, становятся более сложными. 

> [!NOTE]
> Независимо от значения *MaxConcurrentFaults* при отсутствии внешних ошибок Chaos исключает вероятность потери кворума или данных.
>

* **EnableMoveReplicaFaults**. Включает или отключает сбои, приводящие к перемещению первичных или вторичных реплик. Эти ошибки включены по умолчанию.
* **WaitTimeBetweenIterations**. Время ожидания между итерациями. То есть время, на которое Chaos приостанавливает работу после выполнения цикла ошибок и завершения соответствующей проверки работоспособности кластера. Чем выше значение этого параметра, тем ниже средняя скорость внедрения ошибки.
* **WaitTimeBetweenFaults**. Время ожидания между двумя последовательными сбоями в одной итерации. Чем выше значение, тем меньше степень параллелизма (или перекрывания) в ошибках.
* **ClusterHealthPolicy**. Политика работоспособности кластера используется для проверки работоспособности кластера между итерациями Chaos. Если работоспособность кластера находится в состоянии ошибки или непредвиденного исключения во время выполнения ошибки, Chaos ожидает 30 минут до следующей проверки работоспособности, чтобы дать кластеру время восстановить работу.
* **Context**. Коллекция (string, string) пар "ключ —значение". Для записи сведений о запуске Chaos можно использовать карту. Максимальное число таких пар — 100. Каждая строка (ключ или значение) может содержать не более 4095 символов. Эту карту настраивает стартер запуска Chaos, чтобы при необходимости сохранять контекст конкретного выполнения.
* **ChaosTargetFilter.** Этот фильтр можно использовать, чтобы направлять ошибки Chaos только в узлы определенного типа или только в определенные экземпляры приложения. Если фильтр ChaosTargetFilter не используется, Chaos создает ошибки во всех сущностях кластера. Если фильтр ChaosTargetFilter используется, Chaos создает ошибки только в тех сущностях, которые соответствуют параметрам ChaosTargetFilter. NodeTypeInclusionList и ApplicationInclusionList допускают только семантику объединения. Другими словами, вы не сможете использовать пересечение условий NodeTypeInclusionList и ApplicationInclusionList. Например, нельзя создать правило "создавать ошибки в этом приложении, только если оно размещено в узле этого типа". Если правило NodeTypeInclusionList или ApplicationInclusionList включают определенную сущность, ее нельзя исключить с помощью ChaosTargetFilter. В приложении X могут создаваться ошибки Chaos, даже если оно не входит в список ApplicationInclusionList, если в определенный момент это приложение будет выполняться в узле типа Y, включенного в список NodeTypeInclusionList. Если NodeTypeInclusionList и ApplicationInclusionList одновременно не указаны или имеют значение NULL, создается исключение ArgumentException.
    * **NodeTypeInclusionList.** Это список типов узлов, которые нужно включать в ошибки Chaos. Для узлов этого типа применяются все типы ошибок (перезапуск узла, перезапуск пакета кода, удаление реплики, перезапуск реплики, перемещение первичной и вторичной реплик). Если определенный тип узла (скажем, тип узла X) не включен в список NodeTypeInclusionList, в узле типа X никогда не создаются ошибки уровня узла (например, перезапуск узла), но могут создаваться ошибки уровня пакета кода или реплики, если в узле типа X в определенный момент будет выполняться приложение из списка ApplicationInclusionList. В этот список можно включить не более 100 имен типов узлов. Чтобы увеличить это число, измените параметр конфигурации MaxNumberOfNodeTypesInChaosTargetFilter.
    * **ApplicationInclusionList**. Это список URI приложений, которые нужно включать в ошибки Chaos. Все реплики, относящиеся к службам этих приложений, становятся доступными для ошибок реплики Chaos (перезапуск реплики, удаление реплики, перемещения первичной или вторичной реплики). Chaos может перезапустить пакет кода, только если в нем размещены реплики указанных приложений. Если приложение не включено в этот список, для него иногда могут создаваться ошибки Chaos, если в определенный момент это приложение будет выполняться в узле определенного типа, включенного в список NodeTypeInclusionList. Но если, например, приложение X привязано к типу узла Y посредством ограничений размещения и при этом приложение X отсутствует в списке ApplicationInclusionList и тип узла Y отсутствует в списке NodeTypeInclusionList, то для такого приложения X никогда не будут применяться ошибки. В этот список можно включить не более 1000 имен приложений. Чтобы увеличить это число, измените параметр конфигурации MaxNumberOfApplicationsInChaosTargetFilter.

## <a name="how-to-run-chaos"></a>Выполнение Chaos

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Fabric;

using System.Diagnostics;
using System.Fabric.Chaos.DataStructures;

class Program
{
    private class ChaosEventComparer : IEqualityComparer<ChaosEvent>
    {
        public bool Equals(ChaosEvent x, ChaosEvent y)
        {
            return x.TimeStampUtc.Equals(y.TimeStampUtc);
        }
        public int GetHashCode(ChaosEvent obj)
        {
            return obj.TimeStampUtc.GetHashCode();
        }
    }

    static void Main(string[] args)
    {
        var clusterConnectionString = "localhost:19000";
        using (var client = new FabricClient(clusterConnectionString))
        {
            var startTimeUtc = DateTime.UtcNow;

            // The maximum amount of time to wait for all cluster entities to become stable and healthy. 
            // Chaos executes in iterations and at the start of each iteration it validates the health of cluster
            // entities. 
            // During validation if a cluster entity is not stable and healthy within
            // MaxClusterStabilizationTimeoutInSeconds, Chaos generates a validation failed event.
            var maxClusterStabilizationTimeout = TimeSpan.FromSeconds(30.0);

            var timeToRun = TimeSpan.FromMinutes(60.0);

            // MaxConcurrentFaults is the maximum number of concurrent faults induced per iteration. 
            // Chaos executes in iterations and two consecutive iterations are separated by a validation phase.
            // The higher the concurrency, the more aggressive the injection of faults -- inducing more complex
            // series of states to uncover bugs.
            // The recommendation is to start with a value of 2 or 3 and to exercise caution while moving up.
            var maxConcurrentFaults = 3;

            // Describes a map, which is a collection of (string, string) type key-value pairs. The map can be
            // used to record information about the Chaos run. There cannot be more than 100 such pairs and
            // each string (key or value) can be at most 4095 characters long.
            // This map is set by the starter of the Chaos run to optionally store the context about the specific run.
            var startContext = new Dictionary<string, string>{{"ReasonForStart", "Testing"}};

            // Time-separation (in seconds) between two consecutive iterations of Chaos. The larger the value, the
            // lower the fault injection rate.
            var waitTimeBetweenIterations = TimeSpan.FromSeconds(10);

            // Wait time (in seconds) between consecutive faults within a single iteration.
            // The larger the value, the lower the overlapping between faults and the simpler the sequence of
            // state transitions that the cluster goes through. 
            // The recommendation is to start with a value between 1 and 5 and exercise caution while moving up.
            var waitTimeBetweenFaults = TimeSpan.Zero;

            // Passed-in cluster health policy is used to validate health of the cluster in between Chaos iterations. 
            var clusterHealthPolicy = new ClusterHealthPolicy
            {
                ConsiderWarningAsError = false,
                MaxPercentUnhealthyApplications = 100,
                MaxPercentUnhealthyNodes = 100
            };

            // All types of faults, restart node, restart code package, restart replica, move primary replica,
            // and move secondary replica will happen for nodes of type 'FrontEndType'
            var nodetypeInclusionList = new List<string> { "FrontEndType"};

            // In addition to the faults included by nodetypeInclusionList,
            // restart code package, restart replica, move primary replica, move secondary replica faults will
            // happen for 'fabric:/TestApp2' even if a replica or code package from 'fabric:/TestApp2' is residing
            // on a node which is not of type included in nodeypeInclusionList.
            var applicationInclusionList = new List<string> { "fabric:/TestApp2" };

            // List of cluster entities to target for Chaos faults.
            var chaosTargetFilter = new ChaosTargetFilter
            {
                NodeTypeInclusionList = nodetypeInclusionList,
                ApplicationInclusionList = applicationInclusionList
            };

            var parameters = new ChaosParameters(
                maxClusterStabilizationTimeout,
                maxConcurrentFaults,
                true, /* EnableMoveReplicaFault */
                timeToRun,
                startContext,
                waitTimeBetweenIterations,
                waitTimeBetweenFaults,
                clusterHealthPolicy) {ChaosTargetFilter = chaosTargetFilter};

            try
            {
                client.TestManager.StartChaosAsync(parameters).GetAwaiter().GetResult();
            }
            catch (FabricChaosAlreadyRunningException)
            {
                Console.WriteLine("An instance of Chaos is already running in the cluster.");
            }

            var filter = new ChaosReportFilter(startTimeUtc, DateTime.MaxValue);

            var eventSet = new HashSet<ChaosEvent>(new ChaosEventComparer());

            string continuationToken = null;

            while (true)
            {
                ChaosReport report;
                try
                {
                    report = string.IsNullOrEmpty(continuationToken)
                        ? client.TestManager.GetChaosReportAsync(filter).GetAwaiter().GetResult()
                        : client.TestManager.GetChaosReportAsync(continuationToken).GetAwaiter().GetResult();
                }
                catch (Exception e)
                {
                    if (e is FabricTransientException)
                    {
                        Console.WriteLine("A transient exception happened: '{0}'", e);
                    }
                    else if(e is TimeoutException)
                    {
                        Console.WriteLine("A timeout exception happened: '{0}'", e);
                    }
                    else
                    {
                        throw;
                    }

                    Task.Delay(TimeSpan.FromSeconds(1.0)).GetAwaiter().GetResult();
                    continue;
                }

                continuationToken = report.ContinuationToken;

                foreach (var chaosEvent in report.History)
                {
                    if (eventSet.Add(chaosEvent))
                    {
                        Console.WriteLine(chaosEvent);
                    }
                }

                // When Chaos stops, a StoppedEvent is created.
                // If a StoppedEvent is found, exit the loop.
                var lastEvent = report.History.LastOrDefault();

                if (lastEvent is StoppedEvent)
                {
                    break;
                }

                Task.Delay(TimeSpan.FromSeconds(1.0)).GetAwaiter().GetResult();
            }
        }
    }
}
```

```powershell
$clusterConnectionString = "localhost:19000"
$timeToRunMinute = 60

# The maximum amount of time to wait for all cluster entities to become stable and healthy.
# Chaos executes in iterations and at the start of each iteration it validates the health of cluster entities.
# During validation if a cluster entity is not stable and healthy within MaxClusterStabilizationTimeoutInSeconds,
# Chaos generates a validation failed event.
$maxClusterStabilizationTimeSecs = 30

# MaxConcurrentFaults is the maximum number of concurrent faults induced per iteration.
# Chaos executes in iterations and two consecutive iterations are separated by a validation phase.
# The higher the concurrency, the more aggressive the injection of faults -- inducing more complex series of
# states to uncover bugs.
# The recommendation is to start with a value of 2 or 3 and to exercise caution while moving up.
$maxConcurrentFaults = 3

# Time-separation (in seconds) between two consecutive iterations of Chaos. The larger the value, the lower the
# fault injection rate.
$waitTimeBetweenIterationsSec = 10

# Wait time (in seconds) between consecutive faults within a single iteration.
# The larger the value, the lower the overlapping between faults and the simpler the sequence of state
# transitions that the cluster goes through.
# The recommendation is to start with a value between 1 and 5 and exercise caution while moving up.
$waitTimeBetweenFaultsSec = 0

# Passed-in cluster health policy is used to validate health of the cluster in between Chaos iterations. 
$clusterHealthPolicy = new-object -TypeName System.Fabric.Health.ClusterHealthPolicy
$clusterHealthPolicy.MaxPercentUnhealthyNodes = 100
$clusterHealthPolicy.MaxPercentUnhealthyApplications = 100
$clusterHealthPolicy.ConsiderWarningAsError = $False

# Describes a map, which is a collection of (string, string) type key-value pairs. The map can be used to record
# information about the Chaos run.
# There cannot be more than 100 such pairs and each string (key or value) can be at most 4095 characters long.
# This map is set by the starter of the Chaos run to optionally store the context about the specific run.
$context = @{"ReasonForStart" = "Testing"}

#List of cluster entities to target for Chaos faults.
$chaosTargetFilter = new-object -TypeName System.Fabric.Chaos.DataStructures.ChaosTargetFilter
$chaosTargetFilter.NodeTypeInclusionList = new-object -TypeName "System.Collections.Generic.List[String]"

# All types of faults, restart node, restart code package, restart replica, move primary replica, and move
# secondary replica will happen for nodes of type 'FrontEndType'
$chaosTargetFilter.NodeTypeInclusionList.AddRange( [string[]]@("FrontEndType") )
$chaosTargetFilter.ApplicationInclusionList = new-object -TypeName "System.Collections.Generic.List[String]"

# In addition to the faults included by nodetypeInclusionList, 
# restart code package, restart replica, move primary replica, move secondary replica faults will happen for
# 'fabric:/TestApp2' even if a replica or code package from 'fabric:/TestApp2' is residing on a node which is
# not of type included in nodeypeInclusionList.
$chaosTargetFilter.ApplicationInclusionList.Add("fabric:/TestApp2")

Connect-ServiceFabricCluster $clusterConnectionString

$events = @{}
$now = [System.DateTime]::UtcNow

Start-ServiceFabricChaos -TimeToRunMinute $timeToRunMinute -MaxConcurrentFaults $maxConcurrentFaults -MaxClusterStabilizationTimeoutSec $maxClusterStabilizationTimeSecs -EnableMoveReplicaFaults -WaitTimeBetweenIterationsSec $waitTimeBetweenIterationsSec -WaitTimeBetweenFaultsSec $waitTimeBetweenFaultsSec -ClusterHealthPolicy $clusterHealthPolicy -ChaosTargetFilter $chaosTargetFilter

while($true)
{
    $stopped = $false
    $report = Get-ServiceFabricChaosReport -StartTimeUtc $now -EndTimeUtc ([System.DateTime]::MaxValue)

    foreach ($e in $report.History) {

        if(-Not ($events.Contains($e.TimeStampUtc.Ticks)))
        {
            $events.Add($e.TimeStampUtc.Ticks, $e)
            if($e -is [System.Fabric.Chaos.DataStructures.ValidationFailedEvent])
            {
                Write-Host -BackgroundColor White -ForegroundColor Red $e
            }
            else
            {
                Write-Host $e
                # When Chaos stops, a StoppedEvent is created.
                # If a StoppedEvent is found, exit the loop.
                if($e -is [System.Fabric.Chaos.DataStructures.StoppedEvent])
                {
                    return
                }
            }
        }
    }

    Start-Sleep -Seconds 1
}
```
