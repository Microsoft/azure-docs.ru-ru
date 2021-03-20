---
title: Создание службы на основе субъектов в Azure Service Fabric
description: Узнайте, как создать, выполнить отладку и развернуть первую службу на основе субъекта на C#, используя Reliable Actors в Service Fabric.
ms.topic: conceptual
ms.date: 07/10/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 225ccb67153a33ed47af68ebb1549dce37426278
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96573467"
---
# <a name="getting-started-with-reliable-actors"></a>Приступая к работе с Reliable Actors
> [!div class="op_single_selector"]
> * [C# в Windows](service-fabric-reliable-actors-get-started.md)
> * [Java в Linux](./service-fabric-create-your-first-linux-application-with-java.md)

В этой статье описывается создание и отладка простого приложения Reliable Actor в Visual Studio. Дополнительные сведения о Reliable Actors см. в статье [Общие сведения о надежных субъектах Service Fabric](service-fabric-reliable-actors-introduction.md).

## <a name="prerequisites"></a>Предварительные условия

Приступая к работе, убедитесь, что на вашем компьютере установлена и настроена среда разработки Service Fabric, включая Visual Studio. Дополнительные сведения см. в разделе [Подготовка среды разработки в Linux](service-fabric-get-started.md).

## <a name="create-a-new-project-in-visual-studio"></a>Создание проекта в Visual Studio

Запустите Visual Studio 2019 или более поздней версии с правами администратора, а затем создайте новый проект **приложения Service Fabric** .

![Средства Service Fabric для Visual Studio — новый проект][1]

В следующем диалоговом окне выберите **службу субъектов** в разделе **.NET Core 2,0** и введите имя службы.

![Шаблоны проекта Service Fabric][5]

Структура созданного проекта приведена ниже.

![Структура проекта Service Fabric][2]

## <a name="examine-the-solution"></a>Изучение решения

Это решение содержит три проекта:

* **Проект приложения (MyApplication)**. В этом проекте упакованы все службы для последующего развертывания. Проект содержит файл *ApplicationManifest.xml* и сценарии PowerShell для управления приложением.

* **Проект интерфейса (HelloWorld.Interfaces)**. Этот проект содержит определение интерфейса субъекта. Интерфейсы субъекта могут определяться в любом проекте с любым именем.  Интерфейс определяет контракт субъекта, который совместно используется реализацией субъекта и клиентами, вызывающими этот субъект.  Так как проекты клиента могут зависеть от него, обычно имеет смысл определить интерфейс в сборке, отличной от реализации субъекта.

* **Проект службы субъекта (HelloWorld)**. Этот проект используется для определения службы Service Fabric, в которой будет размещен субъект. Он содержит реализацию субъекта, *HelloWorld.cs*. Реализация субъекта — это класс, производный от базового типа `Actor`. Он реализует интерфейсы, определенные в проекте *MyActor.Interfaces*. Класс актера также должен реализовать конструктор, принимающий экземпляр `ActorService` и `ActorId` и передающий их в базовый класс `Actor`.
    
    Этот проект также содержит файл *Program.cs*, который регистрирует классы субъекта в среде выполнения Service Fabric с помощью `ActorRuntime.RegisterActorAsync<T>()`. Класс `HelloWorld` уже зарегистрирован. Все дополнительные реализации субъектов, добавляемые в проект, также должны быть зарегистрированы в методе `Main()`.

## <a name="customize-the-helloworld-actor"></a>Настройка субъекта HelloWorld

Шаблон проекта определяет некоторые методы в интерфейсе `IHelloWorld` и реализует их в реализации субъекта `HelloWorld`.  Замените эти методы таким образом, чтобы служба субъекта возвращала простую строку "Hello World".

В проекте *HelloWorld.Interfaces* в файле *IHelloWorld.cs* замените определение интерфейса следующим образом.

```csharp
public interface IHelloWorld : IActor
{
    Task<string> GetHelloWorldAsync();
}
```

В проекте **HelloWorld** в файле **HelloWorld.cs** замените все определение класса следующим образом.

```csharp
[StatePersistence(StatePersistence.Persisted)]
internal class HelloWorld : Actor, IHelloWorld
{
    public HelloWorld(ActorService actorService, ActorId actorId)
        : base(actorService, actorId)
    {
    }

    public Task<string> GetHelloWorldAsync()
    {
        return Task.FromResult("Hello from my reliable actor!");
    }
}
```

Нажмите клавиши **CTRL+SHIFT+B**, чтобы выполнить сборку проекта и убедиться, что все скомпилировалось.

## <a name="add-a-client"></a>Добавление клиента

Создайте простое консольное приложение для вызова службы субъекта.

1. Щелкните решение правой кнопкой мыши в Обозреватель решений > **Добавить**  >  **Новый проект..**..

2. В разделе типов проектов **.NET Core** выберите **Консольное приложение (.NET Core)**.  Назовите проект *ActorClient*.
    
    ![Диалоговое окно "Добавление проекта"][6]    
    
    > [!NOTE]
    > Как правило, консольное приложение не используется в качестве клиента в Service Fabric, но оно является удобным примером для отладки и тестирования с помощью локального кластера Service Fabric.

3. Консольное приложение должно быть 64-разрядным, чтобы обеспечить совместимость с проектом интерфейса и другими зависимостями.  В обозревателе решений щелкните правой кнопкой мыши проект **ActorClient** и выберите **Свойства**.  На вкладке **Сборка** для параметра **Целевая платформа** выберите значение **x64**.
    
    ![Свойства сборки][8]

4. Для проекта клиента требуется пакет NuGet для Reliable Actors.  Щелкните **Сервис** диспетчер  >  **пакетов NuGet**  >  **консоль диспетчера пакетов**.  В консоли диспетчера пакетов введите следующую команду.
    
    ```powershell
    Install-Package Microsoft.ServiceFabric.Actors -IncludePrerelease -ProjectName ActorClient
    ```

    Пакет NuGet и все его зависимости будут установлены в проект ActorClient.

5. В проекте клиента также требуется ссылка на проект интерфейсов.  В проекте ActorClient щелкните правой кнопкой мыши элемент **зависимости** и выберите команду **Добавить ссылку на проект...**.  Выберите **проекты > решение** (если оно еще не выбрано), а затем установите флажок рядом с **HelloWorld. interfaces**.  Нажмите кнопку **OK**.
    
    ![Диалоговое окно "Добавление ссылки"][7]

6. В проекте ActorClient замените все содержимое *Program.cs* следующим кодом.
    
    ```csharp
    using System;
    using System.Threading.Tasks;
    using Microsoft.ServiceFabric.Actors;
    using Microsoft.ServiceFabric.Actors.Client;
    using HelloWorld.Interfaces;
    
    namespace ActorClient
    {
        class Program
        {
            static void Main(string[] args)
            {
                IHelloWorld actor = ActorProxy.Create<IHelloWorld>(ActorId.CreateRandom(), new Uri("fabric:/MyApplication/HelloWorldActorService"));
                Task<string> retval = actor.GetHelloWorldAsync();
                Console.Write(retval.Result);
                Console.ReadLine();
            }
        }
    }
    ```

## <a name="running-and-debugging"></a>Выполнение и отладка

Нажмите клавишу **F5**, чтобы выполнить сборку приложения, развернуть и запустить его локально в кластере Service Fabric разработки.  Вы можете следить за ходом развертывания в окне **Вывод** .

![Окно вывода отладки Service Fabric][3]

Если результат содержит текст *The application is ready* (Приложение готово), то службу можно проверить с помощью приложения ActorClient.  В обозревателе решений щелкните правой кнопкой мыши проект **ActorClient** и выберите **Отладка** > **Запустить новый экземпляр**.  Приложение командной строки должно отобразить выходные данные из службы субъекта.

![Выходные данные приложения][9]

> [!TIP]
> В среде выполнения субъектов Service Fabric предусмотрены некоторые [события и счетчики производительности, связанные с методами субъекта](service-fabric-reliable-actors-diagnostics.md#actor-method-events-and-performance-counters). Они полезны при диагностике и мониторинге производительности.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте больше о том, [как Reliable Actors использует платформу Service Fabric](service-fabric-reliable-actors-platform.md).


[1]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject.PNG
[2]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-projectstructure.PNG
[3]: ./media/service-fabric-reliable-actors-get-started/debugging-output.PNG
[4]: ./media/service-fabric-reliable-actors-get-started/vs-context-menu.png
[5]: ./media/service-fabric-reliable-actors-get-started/reliable-actors-newproject1.PNG
[6]: ./media/service-fabric-reliable-actors-get-started/new-console-app.png
[7]: ./media/service-fabric-reliable-actors-get-started/add-reference.png
[8]: ./media/service-fabric-reliable-actors-get-started/build-props.png
[9]: ./media/service-fabric-reliable-actors-get-started/app-output.png
