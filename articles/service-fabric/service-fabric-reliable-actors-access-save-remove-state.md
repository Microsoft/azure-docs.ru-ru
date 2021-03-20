---
title: Управление состоянием Service Fabric Azure
description: Сведения о доступе, сохранении и удалении состояния для надежного субъекта Azure Service Fabric и рекомендации при проектировании приложения.
ms.topic: conceptual
ms.date: 03/19/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 1e796232719342883efc2aa5dd377d586f3039ad
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96571323"
---
# <a name="access-save-and-remove-reliable-actors-state"></a>Доступ к состоянию Reliable Actors, а аткже его сохранение и удаление
[Reliable Actors](service-fabric-reliable-actors-introduction.md) — это однопотоковые объекты для инкапсуляции логики и состояния, а также для поддержки надежного состояния. Каждый экземпляр субъекта имеет собственный [диспетчер состояний](service-fabric-reliable-actors-state-management.md): структура данных, подобная словарю, которая надежно хранит пары "ключ-значение". Диспетчер состояний — это оболочка вокруг поставщика состояний. Его можно использовать для хранения данных независимо от того, какой [параметр сохраняемости](service-fabric-reliable-actors-state-management.md#state-persistence-and-replication) используется.

Ключи диспетчера состояний должны быть строками. Значения являются универсальными и могут быть любого типа, в том числе пользовательского. Значения, хранящиеся в диспетчере состояний, должны быть сериализуемыми контрактом данных, так как они могут передаваться по сети другим узлам во время репликации и быть записаны на диск в зависимости от параметра сохранения состояния субъекта.

Диспетчер состояний предоставляет общие методы словаря для управления состоянием, аналогичные используемым в надежном словаре.

Дополнительные сведения см. в [рекомендациях по управлению состоянием субъекта](service-fabric-reliable-actors-state-management.md#best-practices).

## <a name="access-state"></a>Доступ к состоянию
Доступ к состоянию можно получить через диспетчер состояний по ключу. Методы диспетчера состояний асинхронны, так как они могут потребовать дискового ввода-вывода, если у субъектов сохраняемое состояние. При первом доступе объекты состояния кэшируются в памяти. Повторные операции доступа к объектам обращаются к объектам непосредственно из памяти и возвращают данные синхронно без дискового ввода-вывода или асинхронного переключения контекста. Объект состояния удаляется из кэша в следующих случаях:

* Метод субъекта вызывает необработанное исключение после извлечения объекта из диспетчера состояний.
* Субъект повторно активируется после отключения или из-за сбоя.
* Поставщик состояния записывает состояние на диск. Поведение зависит от реализации поставщика состояний. Поставщик состояний по умолчанию для параметра `Persisted` применяет такое поведение.

Получить состояние можно с помощью стандартной операции *Get*, которая порождает исключение `KeyNotFoundException`(C#) или `NoSuchElementException`(Java), если для ключа запись не существует.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<int> GetCountAsync()
    {
        return this.StateManager.GetStateAsync<int>("MyState");
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture<Integer> getCountAsync()
    {
        return this.stateManager().getStateAsync("MyState");
    }
}
```

Получить состояние можно также используя метод *TryGet*. Он не вызывает исключение, если для ключа запись не существует.

```csharp
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task<int> GetCountAsync()
    {
        ConditionalValue<int> result = await this.StateManager.TryGetStateAsync<int>("MyState");
        if (result.HasValue)
        {
            return result.Value;
        }

        return 0;
    }
}
```
```Java
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture<Integer> getCountAsync()
    {
        return this.stateManager().<Integer>tryGetStateAsync("MyState").thenApply(result -> {
            if (result.hasValue()) {
                return result.getValue();
            } else {
                return 0;
            });
    }
}
```

## <a name="save-state"></a>Сохранение состояния
Методы извлечения диспетчера состояний возвращают ссылку на объект в локальной памяти. Изменение этого объекта только в локальной памяти не приводит к его длительному сохранению. Когда объект извлекается из диспетчера состояний и изменяется, его необходимо снова вставить в диспетчер состояний для постоянного сохранения.

Состояние можно вставить с помощью безусловной операции *Set*, которая эквивалента синтаксису `dictionary["key"] = value`.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task SetCountAsync(int value)
    {
        return this.StateManager.SetStateAsync<int>("MyState", value);
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture setCountAsync(int value)
    {
        return this.stateManager().setStateAsync("MyState", value);
    }
}
```

Состояние можно добавить с помощью метода *Add*, который порождает исключение `InvalidOperationException`(C#) или `IllegalStateException`(Java) при попытке добавить ключ, который уже существует.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task AddCountAsync(int value)
    {
        return this.StateManager.AddStateAsync<int>("MyState", value);
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture addCountAsync(int value)
    {
        return this.stateManager().addOrUpdateStateAsync("MyState", value, (key, old_value) -> old_value + value);
    }
}
```

Состояние также можно добавить с помощью метода *TryAdd*, который не вызывает исключение при попытке добавить уже существующий ключ.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task AddCountAsync(int value)
    {
        bool result = await this.StateManager.TryAddStateAsync<int>("MyState", value);

        if (result)
        {
            // Added successfully!
        }
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture addCountAsync(int value)
    {
        return this.stateManager().tryAddStateAsync("MyState", value).thenApply((result)->{
            if(result)
            {
                // Added successfully!
            }
        });
    }
}
```

В конце метода субъекта диспетчер состояний автоматически сохраняет все значения, которые были добавлены или изменены с помощью операции вставки или обновления. "Сохранение" может включать в себя сохранение на диск и репликацию в зависимости от используемых параметров. Значения, которые не были изменены, не сохраняются и не реплицируются. Если значения не были изменены, операция сохранения не выполняет никаких действий. При сбое сохранения измененное состояние удаляется и загружается исходное состояние.

Состояние также можно сохранить вручную, вызвав метод `SaveStateAsync` в базовом субъекте:

```csharp
async Task IMyActor.SetCountAsync(int count)
{
    await this.StateManager.AddOrUpdateStateAsync("count", count, (key, value) => count > value ? count : value);

    await this.SaveStateAsync();
}
```
```Java
interface MyActor {
    CompletableFuture setCountAsync(int count)
    {
        this.stateManager().addOrUpdateStateAsync("count", count, (key, value) -> count > value ? count : value).thenApply();

        this.stateManager().saveStateAsync().thenApply();
    }
}
```

## <a name="remove-state"></a>Удаление состояния
Состояние можно удалить из диспетчера состояний субъекта без возможности восстановления, вызвав метод *Remove*. Этот метод вызывает исключение `KeyNotFoundException`(C#) или `NoSuchElementException`(Java) при попытке удалить ключ, который не существует.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task RemoveCountAsync()
    {
        return this.StateManager.RemoveStateAsync("MyState");
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture removeCountAsync()
    {
        return this.stateManager().removeStateAsync("MyState");
    }
}
```

Состояние также можно удалить без возможности восстановления с помощью метода *TryRemove*, который не вызывает исключение при попытке удалить несуществующий ключ.

```csharp
[StatePersistence(StatePersistence.Persisted)]
class MyActor : Actor, IMyActor
{
    public MyActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public async Task RemoveCountAsync()
    {
        bool result = await this.StateManager.TryRemoveStateAsync("MyState");

        if (result)
        {
            // State removed!
        }
    }
}
```
```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
class MyActorImpl extends FabricActor implements  MyActor
{
    public MyActorImpl(ActorService actorService, ActorId actorId)
    {
        super(actorService, actorId);
    }

    public CompletableFuture removeCountAsync()
    {
        return this.stateManager().tryRemoveStateAsync("MyState").thenApply((result)->{
            if(result)
            {
                // State removed!
            }
        });
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия

Состояние, которое хранится в Reliable Actors, должно быть сериализовано перед записью на диск и реплицировано для обеспечения высокого уровня доступности. Узнайте больше о [сериализации типа субъекта](service-fabric-reliable-actors-notes-on-actor-type-serialization.md).

Затем ознакомьтесь с [диагностикой и мониторингом производительности субъекта](service-fabric-reliable-actors-diagnostics.md).
