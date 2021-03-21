---
title: Уведомления Reliable Services
description: Концептуальная документация по Service Fabric Reliable Services уведомлений для диспетчера надежных состояний и надежного словаря
author: mcoskun
ms.topic: conceptual
ms.date: 6/29/2017
ms.author: mcoskun
ms.custom: devx-track-csharp
ms.openlocfilehash: f5b48cc6cca2e143c48ed7bdfc99de936be2a227
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98784585"
---
# <a name="reliable-services-notifications"></a>Уведомления Reliable Services
Уведомления позволяют клиентам отслеживать изменения, которые вносятся в интересующий их объект. Существует два типа объектов, поддерживающих уведомления: *диспетчер надежных состояний* и *надежный словарь*.

Распространенные причины для использования уведомлений:

* Создание материализованных представлений, например вторичных индексов или агрегированных отфильтрованных представлений состояния реплики. К примеру, это может быть отсортированный индекс всех ключей в надежном словаре.
* Отправка данных мониторинга, например количества пользователей, добавленных за последний час.

Уведомления активируются в ходе применения операций. В связи с этим обработка уведомлений должна быть максимально быстрой и в синхронных событиях не должно быть ресурсоемких операций.

## <a name="reliable-state-manager-notifications"></a>Уведомления диспетчера надежных состояний
Диспетчер надежных состояний предоставляет уведомления для следующих событий:

* Транзакция
  * Commit
* Диспетчер состояний:
  * Перестроение
  * добавление надежного состояния;
  * удаление надежного состояния.

Диспетчер надежных состояний отслеживает текущие обрабатываемые транзакции. Как только состояние транзакции меняется, при фиксации этой транзакции срабатывает уведомление.

Диспетчер надежных состояний поддерживает коллекцию надежных состояний, таких как надежный словарь и надежная очередь. При изменении этой коллекции он активирует уведомления. Это может быть добавление или удаление надежного состояния, а также перестроение всей коллекции.
Перестроение коллекции диспетчера надежных состояний происходит в трех случаях.

* Восстановление. При запуске реплики ее предыдущее состояние будет восстановлено с диска. По завершении восстановления активируется событие с экземпляром **NotifyStateManagerChangedEventArgs**, содержащим набор восстановленных надежных состояний.
* Полная копия. Прежде чем реплику можно будет присоединить к набору конфигурации, нужно выполнить ее сборку. В некоторых случаях может потребоваться применить к неактивной вторичной реплике полную копию состояния диспетчера надежных состояний из первичной реплики. Диспетчер надежных состояний для вторичной реплики использует экземпляр **NotifyStateManagerChangedEventArgs**, чтобы активировать событие, содержащее набор надежных состояний, полученный из первичной реплики.
* Восстановление. В сценариях аварийного восстановления состояние реплики можно восстановить из резервной копии с помощью **RestoreAsync**. В таких случаях диспетчер надежных состояний для первичной реплики использует экземпляр **NotifyStateManagerChangedEventArgs**, чтобы активировать событие, содержащее набор надежных состояний, восстановленный из резервной копии.

Для получения уведомлений о транзакциях и (или) уведомлений диспетчера состояний необходимо выполнить регистрацию в диспетчере надежных состояний, используя событие **TransactionChanged** или **StateManagerChanged**. Чаще всего для регистрации в этих обработчиках событий используется конструктор службы с отслеживанием состояния. Если выполнить регистрацию в конструкторе, вы будете получать все уведомления в связи с изменениями в течение всего времени существования **IReliableStateManager**.

```csharp
public MyService(StatefulServiceContext context)
    : base(MyService.EndpointName, context, CreateReliableStateManager(context))
{
    this.StateManager.TransactionChanged += this.OnTransactionChangedHandler;
    this.StateManager.StateManagerChanged += this.OnStateManagerChangedHandler;
}
```

Обработчик событий **TransactionChanged** использует **NotifyTransactionChangedEventArgs** для предоставления сведений о событии. В этих сведениях содержится свойство Action (например, **NotifyTransactionChangedAction.Commit**), указывающее тип изменения, и свойство Transaction, которое содержит ссылку на измененную транзакцию.

> [!NOTE]
> Сейчас события **TransactionChanged** порождаются, только если фиксируется транзакция. В этом случае свойство Action будет иметь значение **NotifyTransactionChangedAction.Commit**. В будущих обновлениях могут быть добавлены события для других типов изменений состояния транзакции. Таким образом, рекомендуется проверять свойство Action и обрабатывать событие, только если оно ожидается.
> 
> 

Ниже приведен пример обработчика событий **TransactionChanged** .

```csharp
private void OnTransactionChangedHandler(object sender, NotifyTransactionChangedEventArgs e)
{
    if (e.Action == NotifyTransactionChangedAction.Commit)
    {
        this.lastCommitLsn = e.Transaction.CommitSequenceNumber;
        this.lastTransactionId = e.Transaction.TransactionId;

        this.lastCommittedTransactionList.Add(e.Transaction.TransactionId);
    }
}
```

Обработчик событий **StateManagerChanged** использует **NotifyStateManagerChangedEventArgs** для предоставления сведений о событии.
Существуют два подкласса экземпляра **NotifyStateManagerChangedEventArgs**: **NotifyStateManagerRebuildEventArgs** и **NotifyStateManagerSingleEntityChangedEventArgs**.
Свойство Action в **NotifyStateManagerChangedEventArgs** используется для приведения **NotifyStateManagerChangedEventArgs** к правильному подклассу:

* **NotifyStateManagerChangedAction.Rebuild**: **NotifyStateManagerRebuildEventArgs**;
* **NotifyStateManagerChangedAction.Add** и **NotifyStateManagerChangedAction.Remove**: **NotifyStateManagerSingleEntityChangedEventArgs**.

Ниже приведен пример обработчика уведомлений **StateManagerChanged** .

```csharp
public void OnStateManagerChangedHandler(object sender, NotifyStateManagerChangedEventArgs e)
{
    if (e.Action == NotifyStateManagerChangedAction.Rebuild)
    {
        this.ProcessStateManagerRebuildNotification(e);

        return;
    }

    this.ProcessStateManagerSingleEntityNotification(e);
}
```

## <a name="reliable-dictionary-notifications"></a>Уведомления надежного словаря
Надежный словарь предоставляет уведомления для следующих событий:

* Перестроение. Вызывается при восстановлении состояния **ReliableDictionary** из восстановленного или скопированного локального состояния либо из резервной копии.
* Очистка. Вызывается при очистке состояния **ReliableDictionary** с помощью метода **ClearAsync**.
* Добавление. Вызывается при добавлении элемента в **ReliableDictionary**.
* Обновление. Вызывается при обновлении элемента в **IReliableDictionary**.
* Удаление. Вызывается при удалении элемента из **IReliableDictionary**.

Для получения уведомлений надежного словаря нужно выполнить регистрацию обработчика событий **DictionaryChanged** для **IReliableDictionary**. Чаще всего для регистрации этих обработчиков событий используется уведомление о добавлении **ReliableStateManager.StateManagerChanged** .
Если вы выполните регистрацию во время добавления **IReliableDictionary** для **IReliableStateManager**, то не пропустите ни одно уведомление.

```csharp
private void ProcessStateManagerSingleEntityNotification(NotifyStateManagerChangedEventArgs e)
{
    var operation = e as NotifyStateManagerSingleEntityChangedEventArgs;

    if (operation.Action == NotifyStateManagerChangedAction.Add)
    {
        if (operation.ReliableState is IReliableDictionary<TKey, TValue>)
        {
            var dictionary = (IReliableDictionary<TKey, TValue>)operation.ReliableState;
            dictionary.RebuildNotificationAsyncCallback = this.OnDictionaryRebuildNotificationHandlerAsync;
            dictionary.DictionaryChanged += this.OnDictionaryChangedHandler;
        }
    }
}
```

> [!NOTE]
> **ProcessStateManagerSingleEntityNotification** — это пример метода, который вызывается **OnStateManagerChangedHandler** в приведенном выше примере.
> 
> 

Указанный выше код задает интерфейс **IReliableNotificationAsyncCallback**, а также **DictionaryChanged**. Так как **NotifyDictionaryRebuildEventArgs** содержит интерфейс **IAsyncEnumerable**, для которого требуется асинхронное перечисление, уведомления о перестроении активируются посредством **RebuildNotificationAsyncCallback** вместо **OnDictionaryChangedHandler**.

```csharp
public async Task OnDictionaryRebuildNotificationHandlerAsync(
    IReliableDictionary<TKey, TValue> origin,
    NotifyDictionaryRebuildEventArgs<TKey, TValue> rebuildNotification)
{
    this.secondaryIndex.Clear();

    var enumerator = e.State.GetAsyncEnumerator();
    while (await enumerator.MoveNextAsync(CancellationToken.None))
    {
        this.secondaryIndex.Add(enumerator.Current.Key, enumerator.Current.Value);
    }
}
```

> [!NOTE]
> В приведенном выше коде в ходе обработки уведомлений о перестроении сначала очищается поддерживаемое агрегированное состояние. Так как надежная коллекция перестраивается с новым состоянием, то все прежние уведомления несущественны.
> 
> 

Обработчик событий **DictionaryChanged** использует **NotifyDictionaryChangedEventArgs** для предоставления сведений о событии.
**NotifyDictionaryChangedEventArgs** . Свойство Action в **NotifyDictionaryChangedEventArgs** используется для приведения **NotifyDictionaryChangedEventArgs** к правильному подклассу:

* **NotifyDictionaryChangedAction.Rebuild**: **NotifyDictionaryRebuildEventArgs**;
* **NotifyDictionaryChangedAction.Clear**: **NotifyDictionaryClearEventArgs**;
* **NotifyDictionaryChangedAction. Add**: **нотифидиктионаритемаддедевентаргс**
* **NotifyDictionaryChangedAction.Update**: **NotifyDictionaryItemUpdatedEventArgs**;
* **NotifyDictionaryChangedAction.Remove**: **NotifyDictionaryItemRemovedEventArgs**.

```csharp
public void OnDictionaryChangedHandler(object sender, NotifyDictionaryChangedEventArgs<TKey, TValue> e)
{
    switch (e.Action)
    {
        case NotifyDictionaryChangedAction.Clear:
            var clearEvent = e as NotifyDictionaryClearEventArgs<TKey, TValue>;
            this.ProcessClearNotification(clearEvent);
            return;

        case NotifyDictionaryChangedAction.Add:
            var addEvent = e as NotifyDictionaryItemAddedEventArgs<TKey, TValue>;
            this.ProcessAddNotification(addEvent);
            return;

        case NotifyDictionaryChangedAction.Update:
            var updateEvent = e as NotifyDictionaryItemUpdatedEventArgs<TKey, TValue>;
            this.ProcessUpdateNotification(updateEvent);
            return;

        case NotifyDictionaryChangedAction.Remove:
            var deleteEvent = e as NotifyDictionaryItemRemovedEventArgs<TKey, TValue>;
            this.ProcessRemoveNotification(deleteEvent);
            return;

        default:
            break;
    }
}
```

## <a name="recommendations"></a>Рекомендации
* *Завершайте* события уведомлений максимально быстро.
* *Не* выполняйте какие-либо ресурсоемкие операции (например, операции ввода-вывода) в составе синхронных событий.
* *Проверяйте* тип действия перед обработкой события. Новые типы действий могут быть добавлены в будущем.

При этом нужно помнить о следующем:

* Уведомления срабатывают в ходе выполнения операции. Например, уведомление о восстановлении срабатывает на последнем этапе восстановления. Восстановление не будет завершено, пока это уведомление не будет обработано.
* Так как уведомления активируются при применении операций, клиенты будут видеть только уведомления для локально зафиксированных операций. Обратите внимание, что так как гарантируется только локальная фиксация операций (другими словами, ведение их журнала), то возможность в дальнейшем отменить их может быть как доступна, так и недоступна.
* На пути повтора для каждой примененной операции активируется одно уведомление. Это означает, что если транзакция T1 содержит операции Create(X), Delete(X), Create(X), то вы получите одно уведомление о создании X, одно — об удалении и еще одно — о создании, причем в указанном порядке.
* Для транзакций, которые содержат несколько операций, эти операции будут применены в порядке, в котором они были получены от пользователя в первичной реплике.
* При обработке хода выполнения с результатом false некоторые операции могут быть отменены. Для таких операций отмены будут создаваться уведомления и выполнен откат состояния реплики до стабильный точки. Одно важное отличие уведомлений об отмене — события с повторяющимися ключами агрегируются. Например, если упомянутая выше транзакция T1 отменяется, то пользователь увидит одно уведомление для операции Delete(X).

## <a name="next-steps"></a>Дальнейшие действия
* [Надежные коллекции](service-fabric-work-with-reliable-collections.md)
* [Быстрый запуск Reliable Services](service-fabric-reliable-services-quick-start.md)
* [Архивация и восстановление (аварийное восстановление) надежных служб](service-fabric-reliable-services-backup-restore.md)
* [Справочник разработчика по надежным коллекциям](/dotnet/api/microsoft.servicefabric.data.collections#microsoft_servicefabric_data_collections)
