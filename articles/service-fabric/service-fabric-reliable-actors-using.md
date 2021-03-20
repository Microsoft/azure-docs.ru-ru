---
title: Реализация функций в Azure Service Fabric Actors
description: В статье описано, как написать собственную службу субъекта, которая реализует функции на уровне службы так же, как это делается при наследовании StatefulService.
ms.topic: conceptual
ms.date: 03/19/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: d39ec93e0ad03d6c860bae9d0790e860c95457a5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96575573"
---
# <a name="implement-service-level-features-in-your-actor-service"></a>Реализация функций уровня службы в службе субъектов

Как указано в описании[слоев службы](service-fabric-reliable-actors-platform.md#service-layering), сама служба субъекта является надежной службой. Вы можете написать собственную службу, производную от службы `ActorService`, а также реализовать функции уровня службы таким же образом, как при наследовании службы с отслеживанием состояния, например:

- резервное копирование и восстановление службы;
- общие функции для всех субъектов, например автоматическое выключение;
- вызовы удаленных взаимодействий для самой службы субъектов, а также для любого отдельного субъекта.

## <a name="use-the-actor-service"></a>Использование службы субъектов

Экземпляры субъектов имеют доступ к службе субъектов, в которой они выполняются. Через службу субъектов они могут программным образом получить контекст службы, включая идентификатор секции, имя службы, имя приложения и другие данные конкретной платформы Azure Service Fabric:

```csharp
Task MyActorMethod()
{
    Guid partitionId = this.ActorService.Context.PartitionId;
    string serviceTypeName = this.ActorService.Context.ServiceTypeName;
    Uri serviceInstanceName = this.ActorService.Context.ServiceName;
    string applicationInstanceName = this.ActorService.Context.CodePackageActivationContext.ApplicationName;
}
```
```Java
CompletableFuture<?> MyActorMethod()
{
    UUID partitionId = this.getActorService().getServiceContext().getPartitionId();
    String serviceTypeName = this.getActorService().getServiceContext().getServiceTypeName();
    URI serviceInstanceName = this.getActorService().getServiceContext().getServiceName();
    String applicationInstanceName = this.getActorService().getServiceContext().getCodePackageActivationContext().getApplicationName();
}
```

Как и все службы Reliable Services, служба субъектов должна быть зарегистрирована в среде выполнения Service Fabric с указанием типа службы. Чтобы служба субъектов выполняла экземпляры вашей службы, тип вашего субъекта также нужно зарегистрировать в службе субъектов. Метод регистрации `ActorRuntime` выполняет это действие для субъектов. В самом простом случае можно зарегистрировать тип субъекта, и служба субъектов будет использовать параметры по умолчанию.

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>().GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```

Можно также сформировать саму службу субъектов, воспользовавшись лямбда-выражением, предоставляемым методом регистрации. Затем можно настроить службу субъектов и явно сформировать экземпляры субъектов. Вы можете внедрить зависимости для субъекта через конструктор.

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>(
            (context, actorType) => new ActorService(context, actorType, () => new MyActor()))
            .GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```
```Java
static class Program
{
    private static void Main()
    {
      ActorRuntime.registerActorAsync(
              MyActor.class,
              (context, actorTypeInfo) -> new FabricActorService(context, actorTypeInfo),
              timeout);

        Thread.sleep(Long.MAX_VALUE);
    }
}
```

## <a name="actor-service-methods"></a>Методы службы субъектов

Служба субъектов реализует метод `IActorService` (C#) или `ActorService` (Java), который, в свою очередь, реализует метод `IService` (C#) или `Service` (Java). Этот интерфейс используется удаленными взаимодействиями служб Reliable Services, что позволяет вызывать удаленные процедуры в методах службы. Он содержит методы уровня службы, которые можно вызывать удаленно с помощью удаленных взаимодействий служб. Его можно использовать для субъектов [перечисления](service-fabric-reliable-actors-enumerate.md) и [удаления](service-fabric-reliable-actors-delete-actors.md).


## <a name="custom-actor-service"></a>Пользовательская служба субъектов

С помощью лямбда-выражения для регистрации субъектов можно зарегистрировать собственную пользовательскую службу субъектов, производную от `ActorService` (C#) и `FabricActorService` (Java), и реализовать в ней собственные функции уровня службы. Для этого нужно написать класс службы, наследующий класс `ActorService` (C#) или `FabricActorService` (Java). Пользовательская служба субъектов наследует все функции среды выполнения субъектов от класса `ActorService` (C#) или `FabricActorService` (Java), и может использоваться для реализации собственных методов.

```csharp
class MyActorService : ActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, Func<ActorBase> newActor)
        : base(context, typeInfo, newActor)
    { }
}
```
```Java
class MyActorService extends FabricActorService
{
    public MyActorService(StatefulServiceContext context, ActorTypeInformation typeInfo, BiFunction<FabricActorService, ActorId, ActorBase> newActor)
    {
         super(context, typeInfo, newActor);
    }
}
```

```csharp
static class Program
{
    private static void Main()
    {
        ActorRuntime.RegisterActorAsync<MyActor>(
            (context, actorType) => new MyActorService(context, actorType, () => new MyActor()))
            .GetAwaiter().GetResult();

        Thread.Sleep(Timeout.Infinite);
    }
}
```
```Java
public class Program
{
    public static void main(String[] args)
    {
        ActorRuntime.registerActorAsync(
                MyActor.class,
                (context, actorTypeInfo) -> new FabricActorService(context, actorTypeInfo),
                timeout);
        Thread.sleep(Long.MAX_VALUE);
    }
}
```

## <a name="implement-actor-backup-and-restore"></a>Реализация резервного копирования и восстановления субъектов

Пользовательская служба субъектов предоставляет метод для резервного копирования данных субъектов с использованием прослушивателя удаленных взаимодействий, уже присутствующего в `ActorService`. См. дополнительные сведения о [резервном копировании и восстановлении субъектов](service-fabric-reliable-actors-backup-and-restore.md).

## <a name="actor-that-uses-a-remoting-v2-interface-compatible-stack"></a>Субъект, использующий стек удаленного взаимодействия версии 2 (с совместимым интерфейсом)

Стек удаленного взаимодействия версии 2 (с совместимым интерфейсом, известный как V2_1) обладает всеми функциями стека удаленного взаимодействия версии 2. Его интерфейс совместим со стеком удаленного взаимодействия версии 1, но не обладает обратной совместимостью с версиями 1 и (или) 2. Чтобы перейти с V1 на V2_1 без влияния на доступность службы, выполните действия, описанные в следующем разделе.

Чтобы использовать стек удаленного взаимодействия версии 2_1, требуется внести следующие изменения.

1. Добавьте приведенный ниже атрибут сборки в интерфейсы субъекта.
  
   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
   ```

2. Выполните сборку и обновление проектов службы и клиента субъекта, чтобы начать использование стека версии 2.

### <a name="actor-service-upgrade-to-remoting-v2-interface-compatible-stack-without-affecting-service-availability"></a>Обновление службы субъекта для использования стека удаленного взаимодействия версии 2 (с совместимым интерфейсом) без влияния на доступность службы

Это изменение является двухэтапным обновлением. Соблюдайте порядок действий в этой процедуре.

1. Добавьте приведенный ниже атрибут сборки в интерфейсы субъекта. Этот атрибут запустит два прослушивателя для службы субъекта: прослушиватель версии 1 (имеющийся) и прослушиватель версии 2_1. Обновите службу субъекта, внеся в нее это изменение.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V1|RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
   ```

2. Обновите клиенты субъекта после завершения предыдущего обновления.
   Это гарантирует, что прокси-сервер субъекта будет использовать стек удаленного взаимодействия версии 2_1.

3. Этот шаг является необязательным. Измените приведенный выше атрибут, чтобы удалить прослушиватель версии 1.

    ```csharp
    [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2_1,RemotingClientVersion = RemotingClientVersion.V2_1)]
    ```

## <a name="actor-that-uses-the-remoting-v2-stack"></a>Субъект, использующий стек удаленного взаимодействия версии 2

Теперь с помощью пакета NuGet 2.8 пользователи могут использовать стек удаленного взаимодействия версии 2, который лучше работает и предоставляет такие функции, как настраиваемая сериализация. Стек удаленного взаимодействия версии 2 не обеспечивает обратную совместимость с имеющимся стеком удаленного взаимодействия (который теперь называется стеком удаленного взаимодействия версии 1).

Для использования стека удаленного взаимодействия версии 2 требуется внести следующие изменения.

1. Добавьте приведенный ниже атрибут сборки в интерфейсы субъекта.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
   ```

2. Выполните сборку и обновление проектов службы и клиента субъекта, чтобы начать использование стека версии 2.

### <a name="upgrade-the-actor-service-to-the-remoting-v2-stack-without-affecting-service-availability"></a>Обновление службы субъекта для использования стека удаленного взаимодействия версии 2 без влияния на доступность службы

Это изменение является двухэтапным обновлением. Соблюдайте порядок действий в этой процедуре.

1. Добавьте приведенный ниже атрибут сборки в интерфейсы субъекта. Этот атрибут запустит два прослушивателя для службы субъекта: прослушиватель версии 1 (имеющийся) и прослушиватель версии 2. Обновите службу субъекта, внеся в нее это изменение.

   ```csharp
   [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V1|RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
   ```

2. Обновите клиенты субъекта после завершения предыдущего обновления.
   Это гарантирует, что прокси-сервер субъекта будет использовать стек удаленного взаимодействия версии 2.

3. Этот шаг является необязательным. Измените приведенный выше атрибут, чтобы удалить прослушиватель версии 1.

    ```csharp
    [assembly:FabricTransportActorRemotingProvider(RemotingListenerVersion = RemotingListenerVersion.V2,RemotingClientVersion = RemotingClientVersion.V2)]
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Управление состоянием субъекта](service-fabric-reliable-actors-state-management.md)
* [Жизненный цикл субъекта и сборка мусора](service-fabric-reliable-actors-lifecycle.md)
* [Справочная документация по API субъектов](/previous-versions/azure/dn971626(v=azure.100))
* [Пример кода .NET](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Пример кода Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-platform/actor-service.png
[2]: ./media/service-fabric-reliable-actors-platform/app-deployment-scripts.png
[3]: ./media/service-fabric-reliable-actors-platform/actor-partition-info.png
[4]: ./media/service-fabric-reliable-actors-platform/actor-replica-role.png
[5]: ./media/service-fabric-reliable-actors-introduction/distribution.png
