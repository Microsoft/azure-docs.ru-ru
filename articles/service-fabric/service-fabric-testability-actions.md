---
title: Имитация сбоев в микрослужбах Azure
description: В этой статье рассматриваются действия по тестированию, доступные в Microsoft Azure Service Fabric.
ms.topic: conceptual
ms.date: 06/07/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: c57f54096a9593f5ab25a5722d3f2d2b9878b511
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100595017"
---
# <a name="testability-actions"></a>Действия, доступные благодаря Testability
Для моделирования ненадежной инфраструктуры платформа Azure Service Fabric предоставляет разработчику несколько способов имитации различных реалистичных ошибок и переходов. Такие действия доступны благодаря компоненту Testability. Эти действия представляют из себя интерфейсы API низкого уровня, которые вызывают определенную ошибку, смену состояния или проверку. Сочетая эти действия, вы можете создать комплексные сценарии тестирования своих служб.

В платформе Service Fabric доступны несколько распространенных сценариев тестирования, которые состоят из этих действий. Настоятельно рекомендуется использовать эти встроенные сценарии, тщательно подобранные для тестирования распространенных смен состояний и ошибок. Тем не менее действия можно использовать для создания настраиваемых сценариев тестирования, отличающихся расширенными возможностями по сравнению со встроенными сценариями или специально настроенных для вашего приложения.

Реализации действий на языке C# представлены в сборке System.Fabric.dll. Модуль PowerShell Service Fabric находится в сборке Microsoft.ServiceFabric.Powershell.dll. В процессе установки среды выполнения устанавливается модуль PowerShell Service Fabric, обеспечивающий легкость использования решений.

## <a name="graceful-vs-ungraceful-fault-actions"></a>Нормальные и ненормальные ошибки
Действия, доступные благодаря Testability, делятся на две основных части.

* Ненормальные ошибки. Эти сбои имитируют ошибки, такие как перезагрузки компьютеров и аварийное завершение процессов. В случае таких ошибок контекст выполнения процесса немедленно прерывается. Это означает, что очистка состояния возможна только после повторного запуска приложения.
* Нормальные ошибки. Эти сбои имитируют нормальные действия, такие как перемещение реплик и операции удаления, инициированные балансировкой нагрузки. В таких случаях служба получает уведомление о закрытии и может выполнить очистку состояния перед завершением работы.

Для повышения качества выполняйте службы и рабочие нагрузки бизнеса при вызове различных нормальных и ненормальных ошибок. При ненормальных ошибках возникают сценарии, при которых процессы служб неожиданно прерываются посреди рабочих процессов. Таким образом можно проверить путь восстановления после восстановления реплики службы платформой Service Fabric. Это поможет проверить согласованность данных и правильность состояния службы после ошибок. Другой набор сбоев (нормальные ошибки) позволяет проверить, правильно ли реагирует служба на перемещение реплики платформой Service Fabric. Это обеспечит тестирование отмены операций в методе RunAsync. Служба должна проверить, правильно ли устанавливаемый маркер отмены сохраняет данные о своем состоянии и выходит из метода RunAsync.

## <a name="testability-actions-list"></a>Список действий, доступных благодаря Testability
| Действие | Описание | Управляемый API | Командлет PowerShell | Нормальная или ненормальная ошибка |
| --- | --- | --- | --- | --- |
| CleanTestState |Удаляет все данные о состоянии тестирования из кластера в случае неправильного завершения работы тестового драйвера. |CleanTestStateAsync |Remove-ServiceFabricTestState |Неприменимо |
| InvokeDataLoss |Приводит к потере данных в разделе службы. |InvokeDataLossAsync |Invoke-ServiceFabricPartitionDataLoss |Нормальная |
| InvokeQuorumLoss |Приводит к потере кворума в указанном разделе службы с отслеживанием состояния. |InvokeQuorumLossAsync |Invoke-ServiceFabricQuorumLoss |Нормальная |
| MovePrimary |Перемещает заданную первичную реплику службы с отслеживанием состояния на указанный узел кластера. |MovePrimaryAsync |Move-ServiceFabricPrimaryReplica |Нормальная |
| MoveSecondary |Перемещая текущую вторичную реплику службы с отслеживанием состояния на другой узел кластера. |MoveSecondaryAsync |Move-ServiceFabricSecondaryReplica |Нормальная |
| RemoveReplica |Имитирует ошибку реплики путем удаления реплики из кластера. Это приведет к закрытию реплики и переходу к роли "None" с удалением всех данных о состоянии из кластера. |RemoveReplicaAsync |Remove-ServiceFabricReplica |Нормальная |
| RestartDeployedCodePackage |Имитирует ошибку процесса пакета кода путем перезапуска пакета кода, развернутого на узле в кластере. Это приводит к прерыванию процесса пакета кода, который перезапустит все реплики пользовательской службы, размещенные в этом процессе. |RestartDeployedCodePackageAsync |Restart-ServiceFabricDeployedCodePackage |Ненормальная |
| RestartNode |Имитирует ошибку узла кластера Service Fabric путем перезапуска узла. |RestartNodeAsync |Restart-ServiceFabricNode |Ненормальная |
| RestartPartition |Имитирует сценарий отключения центра обработки данных или кластера путем перезапуска некоторых или всех реплик раздела. |RestartPartitionAsync |Restart-ServiceFabricPartition |Нормальная |
| RestartReplica |Имитирует ошибку реплики путем перезапуска материализованной реплики в кластере с последующим закрытием и открытием реплики. |RestartReplicaAsync |Restart-ServiceFabricReplica |Нормальная |
| StartNode |Запускает узел в кластере, который уже остановлен. |StartNodeAsync |Start-ServiceFabricNode |Неприменимо |
| StopNode |Имитирует ошибку узла, останавливая его работу в кластере. Узел начнет работу только после вызова StartNode. |StopNodeAsync |Stop-ServiceFabricNode |Ненормальная |
| ValidateApplication |Проверяет доступность и работоспособность всех служб Service Fabric в приложении обычно после инициирования ошибки в системе. |ValidateApplicationAsync |Test-ServiceFabricApplication |Неприменимо |
| ValidateService |Проверяет доступность и работоспособность службы Service Fabric обычно после инициирования ошибки в системе. |ValidateServiceAsync |Test-ServiceFabricService |Неприменимо |

## <a name="running-a-testability-action-using-powershell"></a>Выполнение действия тестирования с помощью PowerShell
В этом учебнике показано, как выполнить действие, доступное благодаря Testability, с помощью PowerShell. Вы узнаете, как выполнить такое действие для тестирования локального (содержащего одно поле) кластера или кластера Azure. Microsoft.Fabric.Powershell.dll — модуль PowerShell Service Fabric — устанавливается автоматически при установке MSI Microsoft Service Fabric. Этот модуль автоматически загружается при открытии командной строки PowerShell.

Разделы учебника

* [Выполнение действия для кластера с одним полем](#run-an-action-against-a-one-box-cluster)
* [Выполнение действия для кластера Azure](#run-an-action-against-an-azure-cluster)

### <a name="run-an-action-against-a-one-box-cluster"></a>Выполнение действия для кластера с одним полем
Чтобы выполнить действие, доступное благодаря Testability, для тестирования локального кластера, сначала необходимо подключиться к кластеру и открыть командную строку PowerShell в режиме администратора. Рассмотрим действие **Restart-ServiceFabricNode** .

```powershell
Restart-ServiceFabricNode -NodeName Node1 -CompletionMode DoNotVerify
```

Здесь действие **Restart-ServiceFabricNode** выполняется на узле с именем Node1. Режим завершения указывает, что не нужно проверять успешность выполнения действия по перезапуску узла. Указание для режима завершения значения Verify приведет к проверке успешности выполнения действия по перезапуску. Вместо того чтобы использовать имя узла, вы можете следующим образом указать узел с помощью ключа раздела и типа реплики:

```powershell
Restart-ServiceFabricNode -ReplicaKindPrimary  -PartitionKindNamed -PartitionKey Partition3 -CompletionMode Verify
```


```powershell
$connection = "localhost:19000"
$nodeName = "Node1"

Connect-ServiceFabricCluster $connection
Restart-ServiceFabricNode -NodeName $nodeName -CompletionMode DoNotVerify
```

**Restart-ServiceFabricNode** следует использовать для перезапуска узла Service Fabric в кластере. Это остановит процесс Fabric.exe, что приведет к перезапуску всей системной службы и реплик пользовательской службы, размещенных на этом узле. Использование этого интерфейса API для тестирования службы помогает обнаружить ошибки в путях восстановления при отработке отказа. Это позволяет имитировать ошибки узлов в кластере.

На следующем снимке экрана приведена команда Testability **Restart-ServiceFabricNode** в действии.

![Снимок экрана: выполнение команды Restart-ServiceFabricNode в PowerShell.](media/service-fabric-testability-actions/Restart-ServiceFabricNode.png)

Выходные данные первой команды **Get-ServiceFabricNode** (командлет из модуля Service Fabric PowerShell) показывают, что в локальном кластере есть пять узлов с именами от Node.1 до Node.5. После выполнения командлета **Restart-ServiceFabricNode** (действия, доступного благодаря Testability) на узле с именем Node.4 мы увидим, что время работы узла сброшено.

### <a name="run-an-action-against-an-azure-cluster"></a>Выполнение действия для кластера Azure
Выполнение действия, доступного благодаря Testability, в кластере Azure с помощью PowerShell аналогично выполнению этого действия в локальном кластере. Единственная разница в том, что перед выполнением действия необходимо подключиться не к локальному кластеру, а к кластеру Azure.

## <a name="running-a-testability-action-using-c35"></a>Выполнение действия, доступного благодаря Testability, с помощью C&#35;
Чтобы выполнить такое действие с использованием языка C#, сначала необходимо подключиться к кластеру с помощью FabricClient. Затем следует получить параметры, необходимые для выполнения действия. Для выполнения одного действия можно использовать различные параметры.
Один из способов выполнить действие RestartServiceFabricNode — используя сведения об узле (имя узла и идентификатор экземпляра узла) в кластере.

```csharp
RestartNodeAsync(nodeName, nodeInstanceId, completeMode, operationTimeout, CancellationToken.None)
```

Описание параметров

* **Режим завершения** указывает, что режиму не нужно проверять успешность выполнения действия по перезапуску. Указание для режима завершения значения Verify приведет к проверке успешности выполнения действия по перезапуску.  
* **OperationTimeout** задает количество времени для выполнения операции до создания исключения TimeoutException.
* **CancellationToken** позволяет отменить ожидающий вызов.

Вместо того чтобы использовать имя узла, вы можете указать узел с помощью ключа раздела и типа реплики.

Дополнительные сведения см. в PartitionSelector и ReplicaSelector.

```csharp
// Add a reference to System.Fabric.Testability.dll and System.Fabric.dll
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Fabric.Testability;
using System.Fabric;
using System.Threading;
using System.Numerics;

class Test
{
    public static int Main(string[] args)
    {
        string clusterConnection = "localhost:19000";
        Uri serviceName = new Uri("fabric:/samples/PersistentToDoListApp/PersistentToDoListService");
        string nodeName = "N0040";
        BigInteger nodeInstanceId = 130743013389060139;

        Console.WriteLine("Starting RestartNode test");
        try
        {
            //Restart the node by using ReplicaSelector
            RestartNodeAsync(clusterConnection, serviceName).Wait();

            //Another way to restart node is by using nodeName and nodeInstanceId
            RestartNodeAsync(clusterConnection, nodeName, nodeInstanceId).Wait();
        }
        catch (AggregateException exAgg)
        {
            Console.WriteLine("RestartNode did not complete: ");
            foreach (Exception ex in exAgg.InnerExceptions)
            {
                if (ex is FabricException)
                {
                    Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
                }
            }
            return -1;
        }

        Console.WriteLine("RestartNode completed.");
        return 0;
    }

    static async Task RestartNodeAsync(string clusterConnection, Uri serviceName)
    {
        PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);
        ReplicaSelector primaryofReplicaSelector = ReplicaSelector.PrimaryOf(randomPartitionSelector);

        // Create FabricClient with connection and security information here
        FabricClient fabricclient = new FabricClient(clusterConnection);
        await fabricclient.FaultManager.RestartNodeAsync(primaryofReplicaSelector, CompletionMode.Verify);
    }

    static async Task RestartNodeAsync(string clusterConnection, string nodeName, BigInteger nodeInstanceId)
    {
        // Create FabricClient with connection and security information here
        FabricClient fabricclient = new FabricClient(clusterConnection);
        await fabricclient.FaultManager.RestartNodeAsync(nodeName, nodeInstanceId, CompletionMode.Verify);
    }
}
```

## <a name="partitionselector-and-replicaselector"></a>PartitionSelector и ReplicaSelector
### <a name="partitionselector"></a>PartitionSelector
PartitionSelector — это вспомогательное средство, представленное в модуле Testability, позволяющее выбрать определенный раздел, который следует протестировать. Это приложение можно использовать для выбора определенного раздела, если вам заранее известен идентификатор раздела. Кроме того, можно указать ключ раздела, чтобы операция сопоставила идентификатор раздела внутренне. Кроме того, можно выбрать случайный раздел.

Для использования этой возможности создайте объект PartitionSelector и выберите раздел с помощью одного из методов Select \*, а затем передайте объект PartitionSelector в интерфейс API, которому он требуется. Если выбор не сделан, по умолчанию выбирается случайный раздел.

```csharp
Uri serviceName = new Uri("fabric:/samples/InMemoryToDoListApp/InMemoryToDoListService");
Guid partitionIdGuid = new Guid("8fb7ebcc-56ee-4862-9cc0-7c6421e68829");
string partitionName = "Partition1";
Int64 partitionKeyUniformInt64 = 1;

// Select a random partition
PartitionSelector randomPartitionSelector = PartitionSelector.RandomOf(serviceName);

// Select a partition based on ID
PartitionSelector partitionSelectorById = PartitionSelector.PartitionIdOf(serviceName, partitionIdGuid);

// Select a partition based on name
PartitionSelector namedPartitionSelector = PartitionSelector.PartitionKeyOf(serviceName, partitionName);

// Select a partition based on partition key
PartitionSelector uniformIntPartitionSelector = PartitionSelector.PartitionKeyOf(serviceName, partitionKeyUniformInt64);
```

### <a name="replicaselector"></a>ReplicaSelector
ReplicaSelector — это вспомогательное средство, представленное в модуле Testability, позволяющее выбрать реплику, которую следует протестировать. Это приложение можно использовать для выбора реплики, если вам заранее известен идентификатор реплики. Кроме того, вы можете выбрать первичную или случайную вторичную реплику. ReplicaSelector создается на основе объекта PartitionSelector, поэтому необходимо выбрать как реплику, так и раздел, в котором вы хотите выполнить операцию по тестированию.

Для использования этого вспомогательного метода создайте объект ReplicaSelector и укажите способ выбора реплики и раздела. Затем этот объект можно передать в интерфейс API, которому он требуется. Если выбор не сделан, по умолчанию выбираются случайная реплика и случайный раздел.

```csharp
Guid partitionIdGuid = new Guid("8fb7ebcc-56ee-4862-9cc0-7c6421e68829");
PartitionSelector partitionSelector = PartitionSelector.PartitionIdOf(serviceName, partitionIdGuid);
long replicaId = 130559876481875498;

// Select a random replica
ReplicaSelector randomReplicaSelector = ReplicaSelector.RandomOf(partitionSelector);

// Select the primary replica
ReplicaSelector primaryReplicaSelector = ReplicaSelector.PrimaryOf(partitionSelector);

// Select the replica by ID
ReplicaSelector replicaByIdSelector = ReplicaSelector.ReplicaIdOf(partitionSelector, replicaId);

// Select a random secondary replica
ReplicaSelector secondaryReplicaSelector = ReplicaSelector.RandomSecondaryOf(partitionSelector);
```

## <a name="next-steps"></a>Дальнейшие действия
* [Сценарии Testability](service-fabric-testability-scenarios.md)
* Тестирование службы
  * [Моделирование ошибок во время рабочих нагрузок службы](service-fabric-testability-workload-tests.md)
  * [Ошибки обмена данными между службами](service-fabric-testability-scenarios-service-communication.md)

