---
title: Разработка модульных тестов для служб с отслеживанием состояния
description: Дополнительные сведения о модульном тестировании в Azure Service Fabric для служб с отслеживанием состояния и особенностях, которые следует учитывать при разработке.
ms.topic: conceptual
ms.date: 09/04/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 287c0544daa3c44d91fd336b502c496b9b4bb266
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89011401"
---
# <a name="create-unit-tests-for-stateful-services"></a>Создание модульных тестов для служб с отслеживанием состояния
Модульное тестирование служб с отслеживанием состояния в Service Fabric помогает выявить некоторые распространенные ошибки, которые необязательно будут обнаружены обычным приложением или модульным тестированием домена. При разработке модульных тестов для служб с отслеживанием состояния существуют некоторые особенности, которые следует иметь в виду.

1. Каждая реплика выполняет код приложения в разном контексте. Если служба использует три реплики, то код службы выполняется параллельно на трех узлах в разном контексте или роли.
2. Состояние, сохраненное в рамках службы с отслеживанием состояния, должно быть согласовано во всех репликах. Диспетчер состояний и надежные коллекции обеспечивают такую согласованность по умолчанию. Тем не менее для состояния, выполняющегося в памяти, будет требоваться управление с помощью кода приложения.
3. Роль каждой реплики в определенный момент во время выполнения в кластере будет меняться. Вторичная реплика станет первичной в случае недоступности или перегруженности узла, на котором размещена первичная реплика. Это естественное поведение для Service Fabric, поэтому службы должны планировать, что в конечном итоге будет выполнен переход в другую роль.

В этой статье предполагается, что вы уже ознакомились со статьей [Модульное тестирование служб с отслеживанием состояния в Service Fabric](service-fabric-concepts-unit-testing.md).

## <a name="the-servicefabricmocks-library"></a>Библиотека ServiceFabric.Mocks
Начиная с версии 3.3.0, [ServiceFabric.Mocks](https://www.nuget.org/packages/ServiceFabric.Mocks/) предоставляет API для макетирования как оркестрации реплик, так и управления состоянием. Библиотека будет использоваться в примерах.

[NuGet](https://www.nuget.org/packages/ServiceFabric.Mocks/) 
 [GitHub](https://github.com/loekd/ServiceFabric.Mocks)

*ServiceFabric. макеты не принадлежат или не обслуживаются корпорацией Майкрософт. Однако в настоящее время это рекомендуемая библиотека Майкрософт для модульного тестирования служб с отслеживанием состояния.*

## <a name="set-up-the-mock-orchestration-and-state"></a>Настройка макетирования оркестрации и состояния
Как часть сегмента упорядочения теста будут созданы набор макетирования реплик и диспетчер состояний. Затем набор реплик будет выполнять события создания экземпляра тестируемой службы для каждой реплики. Он также будет выполнять события жизненного цикла, такие как `OnChangeRole` и `RunAsync`. Диспетчер макетирования состояний обеспечит, что любая операция, выполняемая через диспетчер состояний, будет запускаться и храниться так, как если бы это был диспетчер фактических состояний.

1. Создайте делегат фабрики служб, который создает экземпляр тестируемой службы. Это должно быть аналогично или так же, как обратный вызов фабрики служб, который обычно находится в `Program.cs` для службы или субъекта Service Fabric. Для этого необходима следующая сигнатура:
   ```csharp
   MyStatefulService CreateMyStatefulService(StatefulServiceContext context, IReliableStateManagerReplica2 stateManager)
   ```
2. Создайте экземпляр класса `MockReliableStateManager`. Он будет макетировать все взаимодействия с диспетчером состояний.
3. Создайте экземпляр `MockStatefulServiceReplicaSet<TStatefulService>`, где `TStatefulService` — это тип службы, которая тестируется. Для этого потребуется делегат, созданный на шаге № 1, и диспетчер состояний, созданный на шаге № 2.
4. Добавьте реплики в набор реплик. Укажите роль (например, первичная, активная вторичная, неактивная вторичная) и идентификатор реплики.
   > Сохраните идентификаторы реплик! Они будут использоваться во время частей выполнения и подтверждения модульного теста.

```csharp
//service factory to instruct how to create the service instance
var serviceFactory = (StatefulServiceContext context, IReliableStateManagerReplica2 stateManager) => new MyStatefulService(context, stateManager);
//instantiate a new mock state manager
var stateManager = new MockReliableStateManager();
//instantiate a new replica set with the service factory and state manager
var replicaSet = new MockStatefulServiceReplicaSet<MyStatefulService>(CreateStatefulService, stateManager);
//add a new Primary replica with id 1
await replicaSet.AddReplicaAsync(ReplicaRole.Primary, 1);
//add a new ActiveSecondary replica with id 2
await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 2);
//add a second ActiveSecondary replica with id 3
await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 3);
```

## <a name="execute-service-requests"></a>Выполнение запросов на обслуживание
Запросы на обслуживание могут быть выполнены в конкретной реплике с использованием удобных свойств и поиска.
```csharp
const string stateName = "test";
var payload = new Payload(StatePayload);

//execute a request on the primary replica using
await replicaSet.Primary.ServiceInstance.InsertAsync(stateName, payload);

//execute a request against replica with id 2
await replicaSet[2].ServiceInstance.InsertAsync(stateName, payload);

//execute a request against one of the active secondary replicas
await replicaSet.FirstActiveSecondary.InsertAsync(stateName, payload);
```

## <a name="execute-a-service-move"></a>Выполнение перемещения службы
Набор макетирования реплик предоставляет несколько удобных методов для активирования различных типов перемещения службы.
```csharp
//promote the first active secondary to primary
replicaSet.PromoteNewReplicaToPrimaryAsync();
//promote the secondary with replica id 4 to primary
replicaSet.PromoteNewReplicaToPrimaryAsync(4);

//promote the first idle secondary to an active secondary
PromoteIdleSecondaryToActiveSecondaryAsync();
//promote idle secondary with replica id 4 to active secondary
PromoteIdleSecondaryToActiveSecondaryAsync(4);

//add a new replica with randomly assigned replica id and promote it to primary
PromoteNewReplicaToPrimaryAsync()
//add a new replica with replica id 4 and promote it to primary
PromoteNewReplicaToPrimaryAsync(4)
```

## <a name="putting-it-all-together"></a>Сборка
Следующий тест демонстрирует настройку набора реплик из трех узлов и проверку того, что данные доступны со вторичной реплики после изменения роли. Может возникнуть типичная проблема, если данные, добавленные во время `InsertAsync`, были сохранены в памяти или в надежной коллекции без выполнения `CommitAsync`. В любом случае вторичная реплика не будет синхронизирована с первичной. Это может привести к несогласованности ответов после перемещения службы.

```csharp
[TestMethod]
public async Task TestServiceState_InMemoryState_PromoteActiveSecondary()
{
    var stateManager = new MockReliableStateManager();
    var replicaSet = new MockStatefulServiceReplicaSet<MyStatefulService>(CreateStatefulService, stateManager);
    await replicaSet.AddReplicaAsync(ReplicaRole.Primary, 1);
    await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 2);
    await replicaSet.AddReplicaAsync(ReplicaRole.ActiveSecondary, 3);

    const string stateName = "test";
    var payload = new Payload(StatePayload);

    //insert data
    await replicaSet.Primary.ServiceInstance.InsertAsync(stateName, payload);
    //promote one of the secondaries to primary
    await replicaSet.PromoteActiveSecondaryToPrimaryAsync(2);
    //get data
    var payloads = (await replicaSet.Primary.ServiceInstance.GetPayloadsAsync()).ToList();

    //data should match what was inserted against the primary
    Assert.IsTrue(payloads.Count == 1);
    Assert.IsTrue(payloads[0].Content == payload.Content);

    //verify the data was saved against the reliable dictionary
    var dictionary = await StateManager.GetOrAddAsync<IReliableDictionary<string, Payload>>(MyStatefulService.StateManagerDictionaryKey);
    using(var tx = StateManager.CreateTransaction())
    {
        var payload = await dictionary.TryGetValue(stateName);
        Assert.IsTrue(payload.HasValue);
        Assert.IsTrue(payload.Value.Content == payload.Content);
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о тестировании см. в статьях [Сценарии тестирования платформы Service Fabric: обмен данными между службами](service-fabric-testability-scenarios-service-communication.md) и [Вызов контролируемого хаоса в кластерах Service Fabric](service-fabric-controlled-chaos.md).
