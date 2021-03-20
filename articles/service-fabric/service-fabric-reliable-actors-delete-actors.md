---
title: Удаление субъектов Azure Service Fabric Actors
description: Узнайте, как вручную и полностью удалить Reliable Actors и их состояние в приложении Service Fabric Azure.
ms.topic: conceptual
ms.date: 03/19/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 16d4ab6a3c155f897cf9212fb1cd6c34d977b9ec
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96574026"
---
# <a name="delete-reliable-actors-and-their-state"></a>Удаление субъектов Reliable Actors и данных их состояния
При удалении деактивированных субъектов удаляется только объект субъекта; из диспетчера состояний субъекта данные не удаляются. При повторной активации субъекта его данные снова становятся доступны в диспетчере состояний. Если субъекты, данные которых хранятся в диспетчере состояний, деактивируются и больше не активируются, может возникнуть необходимость в удалении связанных с ними данных.

В [службе субъектов](service-fabric-reliable-actors-platform.md) имеется функция удаления субъектов с помощью удаленного вызывающего объекта:

```csharp
ActorId actorToDelete = new ActorId(id);

IActorService myActorServiceProxy = ActorServiceProxy.Create(
    new Uri("fabric:/MyApp/MyService"), actorToDelete);

await myActorServiceProxy.DeleteActorAsync(actorToDelete, cancellationToken)
```
```Java
ActorId actorToDelete = new ActorId(id);

ActorService myActorServiceProxy = ActorServiceProxy.create(
    new Uri("fabric:/MyApp/MyService"), actorToDelete);

myActorServiceProxy.deleteActorAsync(actorToDelete);
```

При удалении субъекта происходит следующее (в зависимости от того, были ли он активен в момент удаления):

* **Активный субъект**
  * Субъект удаляется из списка активных субъектов и деактивируется.
  * Состояние субъекта удаляется окончательно.
* **Неактивный субъект**
  * Состояние субъекта удаляется окончательно.

Субъект не может выполнить вызов на собственное удаление при помощи одного из собственных методов. Это обусловлено тем, что нельзя удалить субъект, выполняемый в контексте вызова субъектов, так как для обеспечения однопоточного доступа среда выполнения блокирует вызов субъекта.

Дополнительные сведения о Reliable Actors см. в следующих документах:
* [Таймеры и напоминания субъекта](service-fabric-reliable-actors-timers-reminders.md)
* [События субъекта](service-fabric-reliable-actors-events.md)
* [Повторный вход субъекта](service-fabric-reliable-actors-reentrancy.md)
* [Диагностика и мониторинг производительности в Reliable Actors](service-fabric-reliable-actors-diagnostics.md)
* [Справочная документация по API субъектов](/previous-versions/azure/dn971626(v=azure.100))
* [Пример кода на C#](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started)
* [Пример кода Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

<!--Image references-->
[1]: ./media/service-fabric-reliable-actors-lifecycle/garbage-collection.png
