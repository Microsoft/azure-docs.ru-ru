---
title: Работа с Reliable Collections
description: Ознакомьтесь с рекомендациями по работе с надежными коллекциями в приложении Service Fabric Azure.
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: 908f24cc22c969a34a513ff3fd3ceaa788420620
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98787881"
---
# <a name="working-with-reliable-collections"></a>Работа с Reliable Collections
Платформа Service Fabric предлагает модель программирования с отслеживанием состояния, которая доступна разработчикам .NET через API-интерфейсы Reliable Collections. В частности, Service Fabric предоставляет такие классы, как Reliable Dictionaries (далее — надежные словари) и Reliable Queues (далее — надежные очереди). При использовании этих классов состояние секционируется (для масштабируемости), реплицируется (для доступности) и обрабатывается в ходе транзакции в секции (для семантики ACID). Давайте взглянем на типичное использование объекта надежного словаря и посмотрим, что он реально делает.

```csharp
try
{
   // Create a new Transaction object for this partition
   using (ITransaction tx = base.StateManager.CreateTransaction())
   {
      // AddAsync takes key's write lock; if >4 secs, TimeoutException
      // Key & value put in temp dictionary (read your own writes),
      // serialized, redo/undo record is logged & sent to secondary replicas
      await m_dic.AddAsync(tx, key, value, cancellationToken);

      // CommitAsync sends Commit record to log & secondary replicas
      // After quorum responds, all locks released
      await tx.CommitAsync();
   }
   // If CommitAsync isn't called, Dispose sends Abort
   // record to log & all locks released
}
catch (TimeoutException)
{
   // choose how to handle the situation where you couldn't get a lock on the file because it was 
   // already in use. You might delay and retry the operation
   await Task.Delay(100);
}
```

Для всех операций с объектами надежных словарей (за исключением операции ClearAsync, которая является обратимой) требуется объект ITransaction. Этот объект связан с ним и все изменения, которые вы пытаетесь внести в любой надежный словарь и (или) надежные объекты очереди в пределах одной секции. Чтобы получить объект ITransaction, вызовите метод секционирования Статеманажер'с CreateTransaction.

В приведенном выше коде объект ITransaction передается в метод метод AddAsync надежного словаря. Внутри методов словаря, которые принимают ключ, устанавливают блокировки на чтение или запись, связанные с ключом. Если метод изменяет значение ключа, метод принимает блокировку на запись для ключа, и если метод считывает только из значения ключа, то для ключа выполняется блокировка чтения. Так как метод AddAsync изменяет значение ключа на новое, переданное значение, принимается блокировка записи ключа. Таким образом, если два (или более) потока пытаются одновременно добавить значения с тем же ключом, один поток получит блокировку записи, а остальные потоки заблокируются. По умолчанию методы блокируются до четырех секунд для получения блокировки. Через четыре секунды методы создают исключение TimeoutException. Существуют перегрузки методов, позволяющие передать явное значение времени ожидания, если вы предпочитаете.

Для этого нужно написать код, который будет реагировать на исключение TimeoutException, перехватывая его и повторяя всю операцию (как показано в приведенном выше коде). В этом простом коде вызывается задача. Задержка при каждом выполнении передается в 100 миллисекунд. Но на самом деле, возможно, будет лучше использовать экспоненциальную задержку.

После получения блокировки метод AddAsync добавляет ссылки на объект ключа и значения для внутренних временных словарей, связанных с объектом ITransaction. Это делается для обеспечения семантики, которая позволяет владельцу считывать собственные записи. То есть после вызова метод AddAsync последующий вызов Трижетвалуеасинк с помощью того же объекта ITransaction возвратит значение, даже если транзакция еще не зафиксирована.

> [!NOTE]
> Вызов Трижетвалуеасинк с новой транзакцией вернет ссылку на последнее зафиксированное значение. Не изменяйте эту ссылку напрямую, так как это обходит механизм для сохранения и репликации изменений. Рекомендуется сделать значения только для чтения, чтобы единственный способ изменить значение ключа — через API-интерфейсы надежного словаря.

Затем метод AddAsync сериализует объекты ключа и значения в массивы байтов, а также добавляет эти массивы байтов в файл журнала на локальном узле. Наконец, метод AddAsync отправляет массивы байтов во все вторичные реплики, чтобы реплики имели ту же информацию о ключе и значении. Хотя информация о ключе и значении записана в файл журнала, эти сведения не считаются частью словаря, пока не будет зафиксирована связанная с ними транзакция.

В приведенном выше коде вызов CommitAsync фиксирует все операции транзакции. В частности, этот вызов добавляет в файл журнала на локальном узле сведения о фиксации, а также отправляет запись о фиксации во все вторичные реплики. После получения ответа от большинства реплик все изменения данных считаются постоянными, и все блокировки, связанные с ключами, которые обрабатывались с помощью объекта ITransaction, отменяются. Теперь другие потоки или транзакции могут обрабатывать те же ключи и их значения.

Если метод CommitAsync не вызывается (обычно из-за исключения), объект ITransaction ликвидируется. При удалении незафиксированного объекта ITransaction Service Fabric добавляет сведения о прерывании в файл журнала локального узла и ничего не нужно отправлять ни на одну вторичную реплику. Затем отменяются все блокировки, связанные с ключами, которые обрабатывались с помощью транзакции.

## <a name="volatile-reliable-collections"></a>Постоянные надежные коллекции 
В некоторых рабочих нагрузках, например в реплицированном кэше, можно допускать случайную потери данных. Предотвращение сохранения данных на диск может обеспечить более высокую задержку и пропускную способность при записи в надежные словари. Компромисс отсутствия сохраняемости заключается в том, что при потере кворума произойдет полная потери данных. Так как потери кворума происходит редко, при повышении производительности может возникать редкий риск потери данных для этих рабочих нагрузок.

В настоящее время временная поддержка доступна только для надежных словарей и надежных очередей, а не Релиаблеконкурренткуеуес. Ознакомьтесь со списком [предостережений](service-fabric-reliable-services-reliable-collections-guidelines.md#volatile-reliable-collections) , чтобы сообщить о том, следует ли использовать временные коллекции.

Чтобы включить временную поддержку в службе, установите ```HasPersistedState``` флаг в объявлении типа службы ```false``` , например так:
```xml
<StatefulServiceType ServiceTypeName="MyServiceType" HasPersistedState="false" />
```

>[!NOTE]
>Существующие сохраненные службы нельзя сделать постоянными, и наоборот. Если вы хотите сделать это, необходимо удалить существующую службу, а затем развернуть ее с помощью обновленного флага. Это означает, что вы должны быть готовы к полной потери данных, если вы хотите изменить ```HasPersistedState``` флаг. 

## <a name="common-pitfalls-and-how-to-avoid-them"></a>Типичные проблемы и способы их решения
Теперь, когда вы понимаете, как работают надежные коллекции, давайте взглянем на некоторые распространенные ненужные способы их использования. Взгляните на этот код.

```csharp
using (ITransaction tx = StateManager.CreateTransaction())
{
   // AddAsync serializes the name/user, logs the bytes,
   // & sends the bytes to the secondary replicas.
   await m_dic.AddAsync(tx, name, user);

   // The line below updates the property's value in memory only; the
   // new value is NOT serialized, logged, & sent to secondary replicas.
   user.LastLogin = DateTime.UtcNow;  // Corruption!

   await tx.CommitAsync();
}
```

Если вы работаете с обычным словарем .NET, вы можете добавить в словарь ключ и значение, а затем изменить значение свойства (например, LastLogin). Но этот код не будет правильно работать с надежным словарем. Как вы помните, вызов метода AddAsync сериализует объекты ключей и значений в массивы байтов, сохраняет эти массивы в локальный файл, а также отправляет их во вторичные реплики. При последующем изменении свойства значение свойства изменяется только в памяти; Он не влияет на локальный файл или данные, отправляемые репликам. В случае сбоя процесса выдается память. Если запускается новый процесс или основной репликой становится другая реплика, доступным остается только старое значение свойства.

Допустить описанную выше ошибку очень просто. А узнаете вы о ней, только когда произойдет сбой. Чтобы получить правильный код, достаточно поменять местами две строки:


```csharp
using (ITransaction tx = StateManager.CreateTransaction())
{
   user.LastLogin = DateTime.UtcNow;  // Do this BEFORE calling AddAsync
   await m_dic.AddAsync(tx, name, user);
   await tx.CommitAsync();
}
```

Вот другой пример, демонстрирующий распространенную ошибку.

```csharp
using (ITransaction tx = StateManager.CreateTransaction())
{
   // Use the user's name to look up their data
   ConditionalValue<User> user = await m_dic.TryGetValueAsync(tx, name);

   // The user exists in the dictionary, update one of their properties.
   if (user.HasValue)
   {
      // The line below updates the property's value in memory only; the
      // new value is NOT serialized, logged, & sent to secondary replicas.
      user.Value.LastLogin = DateTime.UtcNow; // Corruption!
      await tx.CommitAsync();
   }
}
```

Опять же, если используются обычные словари .NET, приведенный выше код работает нормально. Это обычная ситуация — разработчик использует ключ для поиска значений. Если значение существует, разработчик изменяет значение свойства. Но если используются API-интерфейсы Reliable Collections, в коде возникает та же проблема. **Не изменяйте объект, если он отправлен в Reliable Collections.**

Чтобы правильно обновить значение в API, нужно получить ссылку на существующее значение и считать объект, на который указывает эта ссылка, неизменяемым. Теперь создайте новый объект, который будет точной копией исходного объекта. Вы можете изменить состояние этого нового объекта и записать новый объект в коллекцию, чтобы он был сериализован в массив байтов, добавлен в локальный файл и отправлен в реплику. После фиксации изменений объекты в памяти, локальный файл и все реплики будут в одинаковом состоянии. Теперь все хорошо.

Следующий пример кода показывает, как правильно обновить значение в коллекции Reliable Collections.

```csharp
using (ITransaction tx = StateManager.CreateTransaction())
{
   // Use the user's name to look up their data
   ConditionalValue<User> currentUser = await m_dic.TryGetValueAsync(tx, name);

   // The user exists in the dictionary, update one of their properties.
   if (currentUser.HasValue)
   {
      // Create new user object with the same state as the current user object.
      // NOTE: This must be a deep copy; not a shallow copy. Specifically, only
      // immutable state can be shared by currentUser & updatedUser object graphs.
      User updatedUser = new User(currentUser);

      // In the new object, modify any properties you desire
      updatedUser.LastLogin = DateTime.UtcNow;

      // Update the key's value to the updateUser info
      await m_dic.SetValue(tx, name, updatedUser);
      await tx.CommitAsync();
   }
}
```

## <a name="define-immutable-data-types-to-prevent-programmer-error"></a>Определение неизменяемых типов данных для предотвращения ошибок разработчика
В идеале мы хотим, чтобы компилятор отправлял сообщения об ошибках при случайном создании кода, изменяющего состояние объекта, который вы должны считать неизменяемыми. Но компилятор C# не может это сделать. Таким образом, чтобы избежать потенциальных ошибок, мы настоятельно рекомендуем определять типы, используемые в Reliable Collections, как неизменяемые. Это означает, что вы будете использовать основные типы значений (например, числа [Int32, UInt64 и т. д.], DateTime, Guid, TimeSpan и т. п). Вы также можете использовать строку. Свойства коллекции лучше не использовать, так как их частая сериализация и десериализация может снизить производительность. Однако если вы хотите использовать свойства коллекции, мы настоятельно рекомендуем использовать. Библиотека неизменяемых коллекций NET ([System. Collections. unmutable](https://www.nuget.org/packages/System.Collections.Immutable/)). Эта библиотека доступна для загрузки из https://nuget.org . Мы также советуем запечатывать классы и делать поля доступны только для чтения, когда это возможно.

Тип UserInfo ниже демонстрирует, как определить неизменяемый тип, чтобы воспользоваться преимуществами упомянутых выше рекомендаций.

```csharp
[DataContract]
// If you don't seal, you must ensure that any derived classes are also immutable
public sealed class UserInfo
{
   private static readonly IEnumerable<ItemId> NoBids = ImmutableList<ItemId>.Empty;

   public UserInfo(String email, IEnumerable<ItemId> itemsBidding = null) 
   {
      Email = email;
      ItemsBidding = (itemsBidding == null) ? NoBids : itemsBidding.ToImmutableList();
   }

   [OnDeserialized]
   private void OnDeserialized(StreamingContext context)
   {
      // Convert the deserialized collection to an immutable collection
      ItemsBidding = ItemsBidding.ToImmutableList();
   }

   [DataMember]
   public readonly String Email;

   // Ideally, this would be a readonly field but it can't be because OnDeserialized
   // has to set it. So instead, the getter is public and the setter is private.
   [DataMember]
   public IEnumerable<ItemId> ItemsBidding { get; private set; }

   // Since each UserInfo object is immutable, we add a new ItemId to the ItemsBidding
   // collection by creating a new immutable UserInfo object with the added ItemId.
   public UserInfo AddItemBidding(ItemId itemId)
   {
      return new UserInfo(Email, ((ImmutableList<ItemId>)ItemsBidding).Add(itemId));
   }
}
```

Тип ItemId также является неизменяемым, как показано ниже.

```csharp
[DataContract]
public struct ItemId
{
   [DataMember] public readonly String Seller;
   [DataMember] public readonly String ItemName;
   public ItemId(String seller, String itemName)
   {
      Seller = seller;
      ItemName = itemName;
   }
}
```

## <a name="schema-versioning-upgrades"></a>Управление версиями схемы (обновления)
На внутреннем уровне надежные коллекции сериализуются объекты с помощью. DataContractSerializer из сети. Сериализованные объекты сохраняются на локальном диске первичной реплики, а также передаются во вторичные реплики. По мере развития вашей службы, скорее всего, потребуется изменить тип данных (схемы), необходимый службе. Управлять версиями данных необходимо предельно осторожно. Первое и самое главное: у вас должна быть возможность десериализовать старые данные. Это означает, что код десериализации должен быть неограниченно обратно совместимым: версия 333 кода службы должна поддерживать обработку данных, которые версия 1 кода службы разместила в надежной коллекции 5 лет назад.

Кроме того, код службы обновляет один домен обновления одновременно. Таким образом, во время обновления одновременно работают две различные версии кода службы. Проследите, чтобы новая версия кода службы не использовала новую схему, так как старые версии кода службы могут не работать с новой схемой. По возможности следует разрабатывать каждую новую версию службы так, чтобы она была совместима с версией 1. Это означает, что версия 1 кода службы должна игнорировать все элементы схемы, которые она явно не обрабатывает. Однако он должен иметь возможность сохранять любые данные, которые он не знает, и записывать его при обновлении ключа или значения словаря.

> [!WARNING]
> Хотя можно изменить схему ключа, необходимо убедиться, что хэш-код и алгоритмы равенства ключа являются стабильными. Если вы измените работу одного из этих алгоритмов, вы никогда больше не сможете искать ключ в надежном словаре.
> Строки .NET можно использовать в качестве ключа, но использовать саму строку в качестве ключа — не используйте результат String. GetHashCode в качестве ключа.

Кроме того, можете выполнить так называемое двухэтапное обновление. С помощью двухэтапного обновления вы обновляете службу с версии v1 на v2: v2 содержит код, который знает, как работать с новым изменением схемы, но этот код не выполняется. Когда код версии 2 считывает данные версии 1, он работает с этими данными, а затем записывает их. Затем, когда обновление завершится на всех доменах обновления, вы можете сообщить запущенным экземплярам версии 2, что обновление завершено. (Один из способов сообщить об этом — развернуть обновление конфигурации. это то, что делает это в два этапа обновления.) Теперь экземпляры v2 могут считывать данные версии 1, преобразовывать их в данные v2, обрабатывать их и записывать в виде данных v2. Когда другие экземпляры считывают данные версии 2, им не нужно преобразовывать их. Они могут работать с ними и записывать как данные версии 2.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о создании контрактов данных с прямой совместимостью см. в статье [Контракты данных, совместимые с любыми будущими изменениями](/dotnet/framework/wcf/feature-details/forward-compatible-data-contracts).

Рекомендации по управлению версиями контрактов данных см. в статье [Управление версиями контракта данных](/dotnet/framework/wcf/feature-details/data-contract-versioning).

Указания по реализации контрактов данных, которые не зависят от версий, см. в статье [Обратные вызовы сериализации, независимые от версий](/dotnet/framework/wcf/feature-details/version-tolerant-serialization-callbacks).

Указания по предоставлению структуры данных, которые могут взаимодействовать в нескольких версиях, см. в статье [Интерфейс IExtensibleDataObject](/dotnet/api/system.runtime.serialization.iextensibledataobject).
