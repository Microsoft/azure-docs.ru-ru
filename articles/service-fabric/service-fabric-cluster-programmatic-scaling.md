---
title: Программное масштабирование Azure Service Fabric
description: Программное уменьшение и увеличение масштаба кластера Azure Service Fabric в соответствии с пользовательскими триггерами
author: mjrousos
ms.topic: conceptual
ms.date: 01/23/2018
ms.author: mikerou
ms.custom: devx-track-csharp
ms.openlocfilehash: 9f39bd874c1f5a1be42ca1c88e6ea2fe8df22f87
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94648214"
---
# <a name="scale-a-service-fabric-cluster-programmatically"></a>Программное масштабирование кластера Service Fabric 

В основе кластеров Service Fabric кластеры, запущенных в Azure, лежат масштабируемые наборы виртуальных машин.  Вы можете узнать, как выполнять [масштабирование кластеров](./service-fabric-cluster-scale-in-out.md) вручную или с помощью правил автомасштабирования. В этой статье описывается, как управлять учетными данными и масштабировать кластер с помощью свободного пакета SDK для вычислений Azure, что является более сложным сценарием. См. дополнительные сведения о [программных методах выполнения операций](service-fabric-cluster-scaling.md#programmatic-scaling). 


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="manage-credentials"></a>Управление учетными данными
Одна из проблем написания службы для обработки масштабирования заключается в том, что служба должна иметь возможность доступа к ресурсам масштабируемого набора виртуальных машин без интерактивного входа в систему. Доступ к кластеру Service Fabric прост, если служба масштабирования изменяет собственное приложение Service Fabric, но для доступа к масштабируемому набору нужны учетные данные. Чтобы войти в систему, можно использовать [субъект-службу](/cli/azure/create-an-azure-service-principal-azure-cli) , созданный с помощью [Azure CLI](https://github.com/azure/azure-cli).

Чтобы создать субъект-службу, необходимо выполнить следующие действия:

1. Войдите в Azure CLI () в `az login` качестве пользователя с доступом к масштабируемому набору виртуальных машин.
2. Создайте субъект-службу с помощью `az ad sp create-for-rbac`.
    1. Запишите идентификатор приложения (другое название — идентификатор клиента), имя, пароль и клиент для последующего использования.
    2. Также вам понадобится идентификатор подписки, который можно просмотреть с помощью `az account list`.

Библиотека вычислений Fluent может выполнить вход с использованием этих учетных данных, как показано ниже (Обратите внимание, что основные типы Azure Fluent, такие как `IAzure` , находятся в пакете [Microsoft. Azure. Management. Fluent](https://www.nuget.org/packages/Microsoft.Azure.Management.Fluent/) ):

```csharp
var credentials = new AzureCredentials(new ServicePrincipalLoginInformation {
                ClientId = AzureClientId,
                ClientSecret = 
                AzureClientKey }, AzureTenantId, AzureEnvironment.AzureGlobalCloud);
IAzure AzureClient = Azure.Authenticate(credentials).WithSubscription(AzureSubscriptionId);

if (AzureClient?.SubscriptionId == AzureSubscriptionId)
{
    ServiceEventSource.Current.ServiceMessage(Context, "Successfully logged into Azure");
}
else
{
    ServiceEventSource.Current.ServiceMessage(Context, "ERROR: Failed to login to Azure");
}
```

После входа в систему можно запросить подсчет экземпляров масштабируемого набора с помощью `AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId).Capacity`.

## <a name="scaling-out"></a>Развертывание
Используя свободный пакет SDK вычислений для Azure, можно добавлять экземпляры в масштабируемый набор виртуальных машин с помощью всего нескольких вызовов:

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);
var newCapacity = (int)Math.Min(MaximumNodeCount, scaleSet.Capacity + 1);
scaleSet.Update().WithCapacity(newCapacity).Apply(); 
``` 

Кроме того, размер масштабируемого набора виртуальных машин можно изменить с помощью командлетов PowerShell. [`Get-AzVmss`](/powershell/module/az.compute/get-azvmss) может получить объект масштабируемого набора виртуальных машин. Текущая емкость указывается в свойстве `.sku.capacity`. После изменения емкости до требуемого значения масштабируемый набор виртуальных машин в Azure можно обновить с помощью [`Update-AzVmss`](/powershell/module/az.compute/update-azvmss) команды.

Как и при добавлении узла вручную, добавления экземпляра масштабируемого набора должно быть достаточно для запуска нового узла Service Fabric, так как шаблон масштабируемого набора включает расширения для автоматического присоединения новых экземпляров к кластеру Service Fabric. 

## <a name="scaling-in"></a>Уменьшение масштаба

Масштабирование аналогично масштабированию. Фактические изменения масштабируемого набора виртуальных машин практически одинаковы. Но, как было описано ранее, служба Service Fabric автоматически очищает только удаленные узлы с уровнем устойчивости Gold или Silver. Таким образом, при уменьшении масштаба на узле с уровнем устойчивости Bronze необходимо взаимодействовать с кластером Service Fabric для завершения работы узла, который нужно удалить, а затем удалить его состояние.

Подготовка узла к завершению работы предусматривает поиск узла, который нужно удалить (последний добавленный экземпляр масштабируемого набора виртуальных машин), и его отключение. Экземпляры масштабируемого набора виртуальных машин нумеруются в порядке, в котором они добавляются. Поэтому новые узлы можно находить, сравнивая числовые суффиксы в именах узлов (которые соответствует именам базовых экземпляров масштабируемого набора виртуальных машин). 

```csharp
using (var client = new FabricClient())
{
    var mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync())
        .Where(n => n.NodeType.Equals(NodeTypeToScale, StringComparison.OrdinalIgnoreCase))
        .Where(n => n.NodeStatus == System.Fabric.Query.NodeStatus.Up)
        .OrderByDescending(n =>
        {
            var instanceIdIndex = n.NodeName.LastIndexOf("_");
            var instanceIdString = n.NodeName.Substring(instanceIdIndex + 1);
            return int.Parse(instanceIdString);
        })
        .FirstOrDefault();
```

После обнаружения узла, который следует удалить, его можно отключить и удалить с помощью того же экземпляра `FabricClient` и экземпляра `IAzure` более ранней версии.

```csharp
var scaleSet = AzureClient.VirtualMachineScaleSets.GetById(ScaleSetId);

// Remove the node from the Service Fabric cluster
ServiceEventSource.Current.ServiceMessage(Context, $"Disabling node {mostRecentLiveNode.NodeName}");
await client.ClusterManager.DeactivateNodeAsync(mostRecentLiveNode.NodeName, NodeDeactivationIntent.RemoveNode);

// Wait (up to a timeout) for the node to gracefully shutdown
var timeout = TimeSpan.FromMinutes(5);
var waitStart = DateTime.Now;
while ((mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Up || mostRecentLiveNode.NodeStatus == System.Fabric.Query.NodeStatus.Disabling) &&
        DateTime.Now - waitStart < timeout)
{
    mostRecentLiveNode = (await client.QueryManager.GetNodeListAsync()).FirstOrDefault(n => n.NodeName == mostRecentLiveNode.NodeName);
    await Task.Delay(10 * 1000);
}

// Decrement VMSS capacity
var newCapacity = (int)Math.Max(MinimumNodeCount, scaleSet.Capacity - 1); // Check min count 

scaleSet.Update().WithCapacity(newCapacity).Apply(); 
```

Как и при развертывании, можно использовать командлеты PowerShell для изменения емкости масштабируемого набора виртуальных машин, если подход с использованием скриптов предпочтительнее. После удаления экземпляра виртуальной машины можно удалить состояние узла Service Fabric.

```csharp
await client.ClusterManager.RemoveNodeStateAsync(mostRecentLiveNode.NodeName);
```

## <a name="next-steps"></a>Дальнейшие действия

Чтобы внедрить собственную логику автомасштабирования, ознакомьтесь со следующими понятиями и полезными API-интерфейсами:

- [Масштабирование кластера Service Fabric с помощью правил автомасштабирования](./service-fabric-cluster-scale-in-out.md)
- [Библиотеки управления Azure для .NET](https://github.com/Azure/azure-libraries-for-net) (используются для взаимодействия с базовыми наборами масштабируемых наборов виртуальных машин кластера Service Fabric)
- [System.Fabric.FabricClient](/dotnet/api/system.fabric.fabricclient) (для взаимодействия с кластером Service Fabric и его узлами)
