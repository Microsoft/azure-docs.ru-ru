---
title: 'Создание первого приложения Service Fabric на языке C #'
description: Вводные сведения о создании приложения Service Fabric Microsoft Azure со службами с отслеживанием состояния и без него.
ms.topic: conceptual
ms.date: 07/10/2019
ms.custom: sfrev, devx-track-csharp
ms.openlocfilehash: 45341c98a40cbcabfa8b96f2016f02f1755fe2b3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791533"
---
# <a name="get-started-with-reliable-services"></a>Приступая к работе с надежными службами

> [!div class="op_single_selector"]
> * [C# в Windows](service-fabric-reliable-services-quick-start.md)
> * [Java в Linux](service-fabric-reliable-services-quick-start-java.md)

Приложение Azure Service Fabric содержит одну или несколько служб, которые выполняют код. В этом руководстве показано, как создавать приложения Service Fabric с отслеживанием состояния и без его учета с помощью служб [Reliable Services](service-fabric-reliable-services-introduction.md).  

## <a name="basic-concepts"></a>Основные понятия

Чтобы приступить к работе с Reliable Services, необходимо ознакомиться только с несколькими основными понятиями.

* **Тип службы**. Это ваша реализация службы. Ее определяет создаваемый вами класс, который расширяет `StatelessService`, и любой другой код или зависимости, используемые в нем, а также имя и номер версии.
* **Именованный экземпляр службы**. Для запуска службы создаются именованные экземпляры вашего типа службы, что во многом похоже на создание экземпляров объектов типа класса. Экземпляр службы имеет имя в формате универсального кода ресурса (URI), использующее шаблон fabric:/. Пример: fabric:/MyApp/MyService.
* **Узел службы**. Создаваемые экземпляры службы должны выполняться в хост-процессе. Узел службы — это просто процесс, в котором можно запускать экземпляры вашей службы.
* **Регистрации службы**. Регистрация объединяет все элементы. Тип службы должен быть зарегистрирован в среде выполнения Service Fabric на узле службы, что позволит Service Fabric создавать экземпляры службы для запуска.  

## <a name="create-a-stateless-service"></a>Создание службы без отслеживания состояния

Сейчас в облачных приложениях обычно используются службы без отслеживания состояния. Термин "без отслеживания состояния" означает, что сама служба не содержит данные, которым требуется надежное хранение или обеспечение высокого уровня доступности. Когда экземпляр службы без отслеживания состояния завершает работу, все данные о его внутреннем состоянии будут утрачены. Для этого типа службы состояние должно быть сохранено во внешнем хранилище, например в таблицах Azure или базе данных SQL, чтобы обеспечить высокую доступность и надежность.

Запустите Visual Studio 2017 или Visual Studio 2019 с правами администратора и создайте новый проект Service Fabric приложения с именем *HelloWorld*:

![Создание нового приложения Service Fabric с помощью диалогового окна "Создание проекта"](media/service-fabric-reliable-services-quick-start/hello-stateless-NewProject.png)

Затем создайте проект службы без отслеживания состояния с помощью **.NET Core 2,0** с именем *HelloWorldStateless*:

![Во втором диалоговом окне создайте проект службы без отслеживания состояния](media/service-fabric-reliable-services-quick-start/hello-stateless-NewProject2.png)

Теперь решение содержит два проекта:

* *HelloWorld*. Это проект *приложения*, который содержит ваши *службы*. В нем также содержится манифест приложения, описывающий приложение, и ряд скриптов PowerShell для развертывания приложения.
* *HelloWorldStateless*. Это проект службы. Он содержит реализацию службы без отслеживания состояния.

## <a name="implement-the-service"></a>Реализация службы

Откройте файл **HelloWorldStateless.cs** в проекте службы. В Service Fabric служба может выполнять любую бизнес-логику. API службы предоставляет две точки входа для кода.

* Вызывается метод *RunAsync* с открытой точкой входа, в котором можно начать выполнение любой рабочей нагрузки, например длительных вычислений.

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    ...
}
```

* Точка входа для обмена данными, к которой можно подключить любой стек связи, например ASP.NET Core. С ее помощью вы можете получать запросы от пользователей и других служб.

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    ...
}
```

В этом учебнике рассматривается метод точки входа `RunAsync()` . Используя его, вы можете сразу же начать выполнение кода.
Шаблон проекта включает пример реализации `RunAsync()` , который увеличивает показания счетчика.

> [!NOTE]
> Дополнительные сведения о работе с стеком связи см. в статье [взаимодействие служб с ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md)

### <a name="runasync"></a>RunAsync

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following sample code with your own logic
    //       or remove this RunAsync override if it's not needed in your service.

    long iterations = 0;

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        ServiceEventSource.Current.ServiceMessage(this.Context, "Working-{0}", ++iterations);

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
}
```

Платформа вызывает этот метод, когда экземпляр службы размещен и готов к выполнению. Для службы без отслеживания состояния это означает, что экземпляр службы открыт. Токен отмены предоставляется для координации закрытия экземпляра вашей службы. В Service Fabric этот цикл открытия и закрытия экземпляра службы может выполняться много раз за все время существования службы. Это может происходить по ряду причин.

* Система перемещает экземпляры служб для распределения ресурсов.
* В коде возникают ошибки.
* Выполняется обновление приложения или системы.
* Возникает аппаратный сбой.

Этой оркестрацией управляет система для сохранения высокой доступности и правильного баланса службы.

Реализация `RunAsync()` не должна использовать синхронную блокировку. Ваша реализация RunAsync должна вернуть задачу или дождаться завершения любых длительных или блокирующих операций, чтобы продолжить выполнение. Обратите внимание, что в цикле `while(true)` в предыдущем примере для возвращения задачи используется `await Task.Delay()`. Если для рабочей нагрузки требуется синхронная блокировка, то в своей реализации `RunAsync` следует запланировать новую задачу с `Task.Run()`.

Отмена рабочей нагрузки является совместным действием, координированным предоставленным токеном отмены. Чтобы продолжать работу, система должна дождаться завершения задачи (успешного завершения, отмены или завершения вследствие ошибки). Когда система запрашивает отмену задачи, важно как можно быстрее подтвердить токен отмены, завершить все выполняющиеся операции и выйти из `RunAsync()` .

В этом примере службы без отслеживания состояния счетчик хранится в локальной переменной. Но так как это служба без отслеживания состояния, хранимое значение существует только для текущего жизненного цикла экземпляра службы. Если службу переместить или перезапустить, значение будет утеряно.

## <a name="create-a-stateful-service"></a>Создание службы с отслеживанием состояния

Service Fabric представляет новый вид службы с отслеживанием состояния. Служба с отслеживанием состояния может надежно поддерживать состояние в самой службе (совмещая с кодом, который его использует). Service Fabric делает состояние высокодоступным без необходимости сохранения состояния во внешнее хранилище.

Чтобы преобразовать значение счетчика без учета состояния в высокодоступное и постоянное (даже при перемещении или перезапуске службы), нужно использовать службу с отслеживанием состояния.

В том же приложении *HelloWorld* можно добавить новую службу, щелкнув правой кнопкой мыши ссылки на службы в проекте приложения и выбрав **Добавить > Новая служба Service Fabric**.

![Добавление службы в приложение Service Fabric](media/service-fabric-reliable-services-quick-start/hello-stateful-NewService.png)

Выберите **.NET Core 2,0-> службу с отслеживанием состояния** и назовите ее *HelloWorldStateful*. Нажмите кнопку **OK**.

![Создание службы с отслеживанием состояния в Service Fabric с помощью диалогового окна "Создание проекта"](media/service-fabric-reliable-services-quick-start/hello-stateful-NewProject.png)

Теперь в вашем приложении должно быть две службы: служба без отслеживания состояния *HelloWorldStateless* и служба с отслеживанием состояния *HelloWorldStateful*.

Служба с отслеживанием состояния имеет такие же точки входа, как и служба без отслеживания состояния. Основное различие заключается в доступности *поставщика состояний* , который может надежно хранить состояние. Service Fabric поставляется с реализацией поставщика состояний, называемой [Надежные коллекции](service-fabric-reliable-services-reliable-collections.md), которая позволяет создавать реплицированные структуры данных с помощью диспетчера надежных состояний. Служба Reliable Service с отслеживанием состояния использует этот поставщик состояний по умолчанию.

Откройте **HelloWorldStateful.cs** в службе *HelloWorldStateful*, содержащей следующий метод RunAsync.

```csharp
protected override async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following sample code with your own logic
    //       or remove this RunAsync override if it's not needed in your service.

    var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");

    while (true)
    {
        cancellationToken.ThrowIfCancellationRequested();

        using (var tx = this.StateManager.CreateTransaction())
        {
            var result = await myDictionary.TryGetValueAsync(tx, "Counter");

            ServiceEventSource.Current.ServiceMessage(this.Context, "Current Counter Value: {0}",
                result.HasValue ? result.Value.ToString() : "Value does not exist.");

            await myDictionary.AddOrUpdateAsync(tx, "Counter", 0, (key, value) => ++value);

            // If an exception is thrown before calling CommitAsync, the transaction aborts, all changes are
            // discarded, and nothing is saved to the secondary replicas.
            await tx.CommitAsync();
        }

        await Task.Delay(TimeSpan.FromSeconds(1), cancellationToken);
    }
```

### <a name="runasync"></a>RunAsync

`RunAsync()` работает одинаково в службах с отслеживанием и без отслеживания состояния. Однако в службе с отслеживанием состояния платформа выполняет дополнительные действия от вашего имени перед выполнением `RunAsync()`. Эти действия могут включать подготовку диспетчера надежных состояний и коллекций Reliable Collections к использованию.

### <a name="reliable-collections-and-the-reliable-state-manager"></a>Reliable Collections и диспетчер надежных состояний

```csharp
var myDictionary = await this.StateManager.GetOrAddAsync<IReliableDictionary<string, long>>("myDictionary");
```

[IReliableDictionary](/dotnet/api/microsoft.servicefabric.data.collections.ireliabledictionary-2#microsoft_servicefabric_data_collections_ireliabledictionary_2) — это реализация словаря, которая позволяет надежно хранить состояние службы. В Service Fabric с помощью Reliable Collections можно хранить данные непосредственно в службе без использования внешнего постоянного хранилища. Надежные коллекции Reliable Collections делают данные высокодоступными. Service Fabric выполняет эту задачу, автоматически создавая несколько *реплик* службы и управляя ими. Кроме того, платформа предоставляет API-интерфейс, который упрощает управление этими репликами и переходы между их состояниями.

В Reliable Collections можно хранить любые типы объектов .NET, включая пользовательские типы. Необходимо только учитывать следующее.

* Service Fabric делает состояние высокодоступным, *реплицируя* его между узлами и сохраняя на локальном диске, а служба Reliable Collections сохраняет ваши данные на локальном диске каждой реплики. Это означает, что все объекты, хранящиеся в Reliable Collections, должны поддерживать *сериализацию*. По умолчанию в Reliable Collections для сериализации используется [DataContract](/dotnet/api/system.runtime.serialization.datacontractattribute). Поэтому при использовании сериализатора по умолчанию убедитесь, что ваши типы [поддерживаются сериализатором контрактов данных](/dotnet/framework/wcf/feature-details/types-supported-by-the-data-contract-serializer).
* Когда вы зафиксируете транзакции в Reliable Collections, объекты реплицируются и становятся высокодоступными. Объекты, сохраненные в Reliable Collections, хранятся в локальной памяти вашей службы. Это означает, что у вас есть локальная ссылка на объект.
  
   Очень важно не изменять локальные экземпляры этих объектов без обновления коллекции в транзакции, так как эти изменения не реплицируются автоматически. Необходимо повторно вставить объект обратно в словарь или использовать один из методов *Update* в словаре.

Диспетчер надежных состояний автоматически управляет коллекциями Reliable Collections. Просто запросите у диспетчера надежных состояний имя коллекции в любое время и в любом расположении службы, и вы гарантированно получите ссылку. Мы не рекомендуем сохранять ссылки на экземпляры коллекции в переменные или свойства членов класса. Необходимо внимательно следить, чтобы ссылка указывала на экземпляр на протяжении всего жизненного цикла службы. Диспетчер надежных состояний выполняет за вас эту работу. Кроме того, он оптимизирован для повторных визитов.

### <a name="transactional-and-asynchronous-operations"></a>Транзакционные и асинхронные операции

```csharp
using (ITransaction tx = this.StateManager.CreateTransaction())
{
    var result = await myDictionary.TryGetValueAsync(tx, "Counter-1");

    await myDictionary.AddOrUpdateAsync(tx, "Counter-1", 0, (k, v) => ++v);

    await tx.CommitAsync();
}
```

Надежные коллекции имеют множество операций, которые `System.Collections.Generic` выполняют их и `System.Collections.Concurrent` аналоги, за исключением запросов, интегрированных с языком LINQ. Операции в надежных коллекциях являются асинхронными. Это вызвано тем, что операции записи с Reliable Collections выполняют операции ввода-вывода для репликации и сохранения данных на диске.

Операции Reliable Collections являются *транзакционными*, так что вы можете сохранять согласованное состояние между несколькими Reliable Collections и операциями. Например, вы можете исключить рабочий элемент из надежной очереди, выполнить над ним какую-либо операцию и сохранить результат в надежном словаре, и это все в пределах одной транзакции. Такая операция называется атомарной. Предусматривается, что либо вся операция завершится успешно, либо она будет полностью отменена. Если ошибка возникнет после выведения элемента из очереди, но до сохранения результата, будет выполнен откат всей транзакции, а элемент останется в очереди для обработки.

## <a name="run-the-application"></a>Выполнение приложения
Теперь вернемся к приложению *HelloWorld* . Теперь вы можете построить и развернуть свои службы. Когда вы нажмете клавишу **F5**, приложение будет построено и развернуто в вашем локальном кластере.

Запустив службы, вы сможете просмотреть созданные события трассировки событий Windows в окне **События диагностики** . Обратите внимание, что в приложении отображаются события как из службы без отслеживания состояния, так и из службы с отслеживанием состояния. Поток можно приостановить, нажав кнопку **Приостановить** . Затем можно просмотреть подробные сведения о сообщении, развернув его.

> [!NOTE]
> Перед запуском приложения убедитесь, что кластер локальной разработки запущен. Сведения о настройке локальной среды см. в [руководстве по началу работы](service-fabric-get-started.md).
> 
> 

![Просмотр событий диагностики в Visual Studio](media/service-fabric-reliable-services-quick-start/hello-stateful-Output.png)

## <a name="next-steps"></a>Дальнейшие действия
[Отладка приложения Service Fabric с помощью Visual Studio](service-fabric-debugging-your-application.md)

[Приступая к работе со службами веб-API Service Fabric с саморазмещением OWIN](./service-fabric-reliable-services-communication-aspnetcore.md)

[Дополнительные сведения о надежных коллекциях](service-fabric-reliable-services-reliable-collections.md)

[Развертывание приложения](service-fabric-deploy-remove-applications.md)

[Обновление приложения](service-fabric-application-upgrade.md)

[Справочник разработчика по надежным службам](/previous-versions/azure/dn706529(v=azure.100))
