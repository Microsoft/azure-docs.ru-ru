---
title: Reliable Actors примечания о сериализации типов субъектов
description: В этой статье приведены сведения о базовых требованиях к определению сериализуемых классов, которые можно использовать для определения интерфейсов и состояний Reliable Actors в Service Fabric.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: 2a1118e394046b217a288663659a2c910098e992
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96576015"
---
# <a name="notes-on-service-fabric-reliable-actors-type-serialization"></a>Примечания о сериализации типов надежных субъектов Service Fabric
Аргументы всех методов, типы результатов задач, возвращаемых каждым методом в интерфейсе субъекта, и объекты, хранящиеся в диспетчере состояний субъекта, должны быть [сериализуемыми контрактом данных](/dotnet/framework/wcf/feature-details/types-supported-by-the-data-contract-serializer). Это также относится к аргументам методов, определенных в [интерфейсах событий субъекта](service-fabric-reliable-actors-events.md). (Методы интерфейсов для событий субъектов всегда возвращают значение void.)

## <a name="custom-data-types"></a>Пользовательские типы данных
В этом примере приведенный ниже интерфейс субъекта определяет метод, возвращающий пользовательский тип данных `VoicemailBox`.

```csharp
public interface IVoiceMailBoxActor : IActor
{
    Task<VoicemailBox> GetMailBoxAsync();
}
```

```Java
public interface VoiceMailBoxActor extends Actor
{
    CompletableFuture<VoicemailBox> getMailBoxAsync();
}
```

Этот интерфейс реализован субъектом, который использует диспетчер состояния для хранения объекта `VoicemailBox`.

```csharp
[StatePersistence(StatePersistence.Persisted)]
public class VoiceMailBoxActor : Actor, IVoicemailBoxActor
{
    public VoiceMailBoxActor(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<VoicemailBox> GetMailboxAsync()
    {
        return this.StateManager.GetStateAsync<VoicemailBox>("Mailbox");
    }
}

```

```Java
@StatePersistenceAttribute(statePersistence = StatePersistence.Persisted)
public class VoiceMailBoxActorImpl extends FabricActor implements VoicemailBoxActor
{
    public VoiceMailBoxActorImpl(ActorService actorService, ActorId actorId)
    {
         super(actorService, actorId);
    }

    public CompletableFuture<VoicemailBox> getMailBoxAsync()
    {
         return this.stateManager().getStateAsync("Mailbox");
    }
}

```

В этом примере объект `VoicemailBox` сериализуется в следующих случаях.

* Объект передается между экземпляром субъекта и вызывающим объектом.
* Объект сохраняется в диспетчере состояния, где он сохраняется на диске и реплицируется на другие узлы.

Платформа Reliable Actors использует сериализацию DataContract. Таким образом, пользовательские объекты данных и их члены должны быть снабжены атрибутами **DataContract** и **DataMember** соответственно.

```csharp
[DataContract]
public class Voicemail
{
    [DataMember]
    public Guid Id { get; set; }

    [DataMember]
    public string Message { get; set; }

    [DataMember]
    public DateTime ReceivedAt { get; set; }
}
```
```Java
public class Voicemail implements Serializable
{
    private static final long serialVersionUID = 42L;

    private UUID id;                    //getUUID() and setUUID()

    private String message;             //getMessage() and setMessage()

    private GregorianCalendar receivedAt; //getReceivedAt() and setReceivedAt()
}
```


```csharp
[DataContract]
public class VoicemailBox
{
    public VoicemailBox()
    {
        this.MessageList = new List<Voicemail>();
    }

    [DataMember]
    public List<Voicemail> MessageList { get; set; }

    [DataMember]
    public string Greeting { get; set; }
}
```
```Java
public class VoicemailBox implements Serializable
{
    static final long serialVersionUID = 42L;
    
    public VoicemailBox()
    {
        this.messageList = new ArrayList<Voicemail>();
    }

    private List<Voicemail> messageList;   //getMessageList() and setMessageList()

    private String greeting;               //getGreeting() and setGreeting()
}
```


## <a name="next-steps"></a>Дальнейшие действия
* [Жизненный цикл субъекта и сборка мусора](service-fabric-reliable-actors-lifecycle.md)
* [Таймеры и напоминания субъекта](service-fabric-reliable-actors-timers-reminders.md)
* [События субъекта](service-fabric-reliable-actors-events.md)
* [Повторный вход субъекта](service-fabric-reliable-actors-reentrancy.md)
* [Полиморфизм субъекта и объектно-ориентированные шаблоны проектирования](service-fabric-reliable-actors-polymorphism.md)
* [Диагностика и мониторинг производительности в Reliable Actors](service-fabric-reliable-actors-diagnostics.md)
