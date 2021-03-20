---
title: Моделирование ошибок в приложениях Service Fabric Azure
description: Узнайте, как защитить службы Azure Service Fabric от корректного и нельготного сбоя.
ms.topic: conceptual
ms.date: 06/15/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: c714ae30c64ea073cbac521eac5e15a8d968b7ea
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91531316"
---
# <a name="simulate-failures-during-service-workloads"></a>Моделирование ошибок во время рабочих нагрузок службы
Сценарии тестирования в Azure Service Fabric позволяют разработчикам не беспокоиться об обработке отдельных ошибок. Однако есть сценарии, в которых может потребоваться явное чередование клиентской рабочей нагрузки и сбоев. Чередование клиентской рабочей нагрузки и сбоев обеспечивает выполнение службой необходимых действий в случае сбоя. Учитывая уровень управления тестированием, эти сбои могут наблюдаться именно в точках выполнения рабочей нагрузки. Вызывая таким образом ошибки при различных состояниях приложения, можно находить ошибки и улучшить качество.

## <a name="sample-custom-scenario"></a>Пример пользовательского сценария
В этом тесте показан сценарий чередования рабочей нагрузки бизнеса с [нормальными и ненормальными сбоями](service-fabric-testability-actions.md#graceful-vs-ungraceful-fault-actions). Для получения оптимальных результатов сбои следует вызывать во время выполнения операций службы или вычислений.

Рассмотрим пример службы, которая предоставляет четыре рабочих нагрузки (А, Б, В и Г). Каждая из них соответствует набору рабочих процессов и может представлять из себя вычисления, хранилище или их сочетание. Для простоты рабочие нагрузки в нашем примере абстрагированы. Различные сбои, выполненные в этом примере:

* RestartNode: ненормальный сбой для моделирования перезагрузки компьютера.
* RestartDeployedCodePackage: ненормальный сбой для моделирования аварийного завершения хост-процесса службы.
* RemoveReplica: нормальный сбой для моделирования удаления реплики.
* MovePrimary: нормальный сбой для моделирования перемещения реплик, активированного балансировщиком нагрузки Service Fabric.

```csharp
// Add a reference to System.Fabric.Testability.dll and System.Fabric.dll.

using System;
using System.Fabric;
using System.Fabric.Testability.Scenario;
using System.Threading;
using System.Threading.Tasks;

class Test
{
    public static int Main(string[] args)
    {
        // Replace these strings with the actual version for your cluster and application.
        string clusterConnection = "localhost:19000";
        Uri applicationName = new Uri("fabric:/samples/PersistentToDoListApp");
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");

        Console.WriteLine("Starting Workload Test...");
        try
        {
            RunTestAsync(clusterConnection, applicationName, serviceName).Wait();
        }
        catch (AggregateException ae)
        {
            Console.WriteLine("Workload Test failed: ");
            foreach (Exception ex in ae.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("Workload Test completed successfully.");
        return 0;
    }

    public enum ServiceWorkloads
    {
        A,
        B,
        C,
        D
    }

    public enum ServiceFabricFaults
    {
        RestartNode,
        RestartCodePackage,
        RemoveReplica,
        MovePrimary,
    }

    public static async Task RunTestAsync(string clusterConnection, Uri applicationName, Uri serviceName)
    {
        // Create FabricClient with connection and security information here.
        FabricClient fabricClient = new FabricClient(clusterConnection);
        // Maximum time to wait for a service to stabilize.
        TimeSpan maxServiceStabilizationTime = TimeSpan.FromSeconds(120);

        // How many loops of faults you want to execute.
        uint testLoopCount = 20;
        Random random = new Random();

        for (var i = 0; i < testLoopCount; ++i)
        {
            var workload = SelectRandomValue<ServiceWorkloads>(random);
            // Start the workload.
            var workloadTask = RunWorkloadAsync(workload);

            // While the task is running, induce faults into the service. They can be ungraceful faults like
            // RestartNode and RestartDeployedCodePackage or graceful faults like RemoveReplica or MovePrimary.
            var fault = SelectRandomValue<ServiceFabricFaults>(random);

            // Create a replica selector, which will select a primary replica from the given service to test.
            var replicaSelector = ReplicaSelector.PrimaryOf(PartitionSelector.RandomOf(serviceName));
            // Run the selected random fault.
            await RunFaultAsync(applicationName, fault, replicaSelector, fabricClient);
            // Validate the health and stability of the service.
            await fabricClient.TestManager.ValidateServiceAsync(serviceName, maxServiceStabilizationTime);

            // Wait for the workload to finish successfully.
            await workloadTask;
        }
    }

    private static async Task RunFaultAsync(Uri applicationName, ServiceFabricFaults fault, ReplicaSelector selector, FabricClient client)
    {
        switch (fault)
        {
            case ServiceFabricFaults.RestartNode:
                await client.FaultManager.RestartNodeAsync(selector, CompletionMode.Verify);
                break;
            case ServiceFabricFaults.RestartCodePackage:
                await client.FaultManager.RestartDeployedCodePackageAsync(applicationName, selector, CompletionMode.Verify);
                break;
            case ServiceFabricFaults.RemoveReplica:
                await client.FaultManager.RemoveReplicaAsync(selector, CompletionMode.Verify, false);
                break;
            case ServiceFabricFaults.MovePrimary:
                await client.FaultManager.MovePrimaryAsync(selector.PartitionSelector);
                break;
        }
    }

    private static Task RunWorkloadAsync(ServiceWorkloads workload)
    {
        throw new NotImplementedException();
        // This is where you trigger and complete your service workload.
        // Note that the faults induced while your service workload is running will
        // fault the primary service. Hence, you will need to reconnect to complete or check
        // the status of the workload.
    }

    private static T SelectRandomValue<T>(Random random)
    {
        Array values = Enum.GetValues(typeof(T));
        T workload = (T)values.GetValue(random.Next(values.Length));
        return workload;
    }
}
```
