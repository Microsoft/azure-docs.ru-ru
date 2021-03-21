---
title: Рекомендации по повышению производительности с помощью служебной шины Azure
description: В этой статье описывается использование служебной шины Azure для оптимизации производительности при обмене сообщениями в брокере.
ms.topic: article
ms.date: 03/09/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 10435f74cfb7c87ccb28b64e1b3f136add1dc927
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102561880"
---
# <a name="best-practices-for-performance-improvements-using-service-bus-messaging"></a>Рекомендации по повышению производительности с помощью обмена сообщениями через служебную шину

В этой статье описано использование служебной шины Azure для оптимизации производительности при обмене сообщениями через брокер. В первой части этой статьи описываются различные механизмы повышения производительности. Вторая часть предоставляет рекомендации по использованию служебной шины таким образом, чтобы обеспечить наилучшую производительность в определенном сценарии.

В этой статье термин "клиент" означает любую сущность, которая обращается к служебной шине. Клиент может быть как отправителем, так и получателем. Термин "sender" используется для клиента очереди служебной шины или клиента раздела, который отправляет сообщения в очередь служебной шины или раздел. Термин "получатель" относится к клиенту очереди служебной шины или клиенту подписки, который получает сообщения из очереди или подписки служебной шины.

## <a name="protocols"></a>Протоколы
Служебная шина позволяет клиентам отправлять и получать сообщения с помощью трех протоколов.

1. Протокол AMQP
2. Протокол SBMP
3. Протокол HTTP

AMQP является наиболее эффективным, так как поддерживает подключение к служебной шине. Он также реализует [пакетную обработку](#batching-store-access) и [предзагрузку](#prefetching). Если явно не указано иное, в этой статье предполагается использование протокола AMQP или SBMP.

> [!IMPORTANT]
> SBMP доступен только для платформа .NET Framework. AMQP используется по умолчанию для .NET Standard.

## <a name="choosing-the-appropriate-service-bus-net-sdk"></a>Выбор соответствующего пакета SDK для .NET служебной шины
Существует три поддерживаемых пакета SDK для .NET служебной шины Azure. Их API-интерфейсы похожи, и их можно запутать. Сведения о принятии решений см. в следующей таблице. Пакет SDK для Azure. Messaging. ServiceBus является последним, и мы рекомендуем использовать его поверх других пакетов SDK. Пакеты SDK Azure. Messaging. ServiceBus и Microsoft. Azure. ServiceBus являются современными, производительными и совместимыми с различными платформами. Кроме того, они поддерживают AMQP через WebSockets и являются частью коллекции SDK Azure .NET для проектов с открытым кодом.

| Пакет NuGet | Основные пространства имен | Минимальная платформа (-ы) | Протокол(-ы) |
|---------------|----------------------|---------------------|-------------|
| [Azure. Messaging. ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus) | `Azure.Messaging.ServiceBus`<br>`Azure.Messaging.ServiceBus.Administration` | .NET Core 2.0;<br>.NET Framework 4.6.1<br>Mono 5.4<br>Xamarin.iOS 10.14<br>Xamarin.Mac 3.8<br>Xamarin.Android 8.0;<br>универсальная платформа Windows 10.0.16299. | AMQP<br>HTTP |
| [Microsoft.Azure.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/) | `Microsoft.Azure.ServiceBus`<br>`Microsoft.Azure.ServiceBus.Management` | .NET Core 2.0;<br>.NET Framework 4.6.1<br>Mono 5.4<br>Xamarin.iOS 10.14<br>Xamarin.Mac 3.8<br>Xamarin.Android 8.0;<br>универсальная платформа Windows 10.0.16299. | AMQP<br>HTTP |
| [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus) | `Microsoft.ServiceBus`<br>`Microsoft.ServiceBus.Messaging` | .NET Framework 4.6.1 | AMQP<br>SBMP<br>HTTP |

Дополнительные сведения о минимальной .NET Standard поддержки платформ см. в разделе [Поддержка реализации .NET](/dotnet/standard/net-standard#net-implementation-support).

## <a name="reusing-factories-and-clients"></a>Повторное использование фабрик и клиентов
# <a name="azuremessagingservicebus-sdk"></a>[Пакет SDK для Azure. Messaging. ServiceBus](#tab/net-standard-sdk-2)
Объекты служебной шины, взаимодействующие со службой, такие как [сервицебусклиент](/dotnet/api/azure.messaging.servicebus.servicebusclient), [сервицебуссендер](/dotnet/api/azure.messaging.servicebus.servicebussender), [сервицебусрецеивер](/dotnet/api/azure.messaging.servicebus.servicebusreceiver)и [сервицебуспроцессор](/dotnet/api/azure.messaging.servicebus.servicebusprocessor), должны быть зарегистрированы для внедрения зависимостей как одноэлементные (или создаются один раз и совместно используются). Сервицебусклиент можно зарегистрировать для внедрения зависимостей с помощью [сервицебусклиентбуилдерекстенсионс](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/servicebus/Azure.Messaging.ServiceBus/src/Compatibility/ServiceBusClientBuilderExtensions.cs). 

Не рекомендуется закрывать или удалять эти объекты после отправки или получения каждого сообщения. Закрытие или удаление объектов, зависящих от сущности (Сервицебуссендер/получатель/процессор), приводит к разрыву ссылки на службу служебной шины. Удаление Сервицебусклиент приведет к разрыву подключения к службе служебной шины. 

# <a name="microsoftazureservicebus-sdk"></a>[Пакет SDK Microsoft. Azure. ServiceBus](#tab/net-standard-sdk)

Клиентские объекты служебной шины, такие как реализации [`IQueueClient`][QueueClient] или [`IMessageSender`][MessageSender] , должны быть зарегистрированы для внедрения зависимостей в виде Singleton-элементов (или создается один раз и совместно используются). Мы не рекомендуем закрывать фабрики сообщений, очереди, разделы или клиенты подписки после отправки сообщения, а затем создавать их повторно при отправке следующего сообщения. Закрытие фабрики обмена сообщениями приводит к удалению подключения к службе служебной шины. При повторном создании фабрики устанавливается новое соединение. 

# <a name="windowsazureservicebus-sdk"></a>[Пакет SDK для WindowsAzure. ServiceBus](#tab/net-framework-sdk)

Клиентские объекты служебной шины, такие как `QueueClient` или `MessageSender` , создаются с помощью объекта [MessagingFactory][MessagingFactory] , который также обеспечивает внутреннее управление соединениями. Мы не рекомендуем закрывать фабрики сообщений, очереди, разделы или клиенты подписки после отправки сообщения, а затем создавать их повторно при отправке следующего сообщения. При закрытии фабрики обмена сообщениями удаляется подключение к службе служебной шины, а при повторном создании фабрики устанавливается новое подключение. 

---

Следующее примечание относится ко всем пакетам SDK:

> [!NOTE]
> Установка подключения является ресурсоемкой операцией, которую можно исключить, если повторно использовать одну и ту же фабрику и клиентские объекты для нескольких операций. Вы можете безопасно использовать объекты клиента для параллельных асинхронных операций из нескольких потоков.

## <a name="concurrent-operations"></a>Параллельные операции
Такие операции, как отправка, получение, удаление и т. д., занимают некоторое время. В этот раз включается время, в течение которого служба служебной шины обрабатывает операцию и задержку запроса и ответа. Чтобы увеличить количество операций в единицу времени, необходимо выполнять их параллельно.

Клиент планирует параллельные операции, выполняя **асинхронные** операции. Следующий запрос запускается до завершения предыдущего запроса. Вот пример фрагмента кода асинхронной операции отправки:

# <a name="azuremessagingservicebus-sdk"></a>[Пакет SDK для Azure. Messaging. ServiceBus](#tab/net-standard-sdk-2)
```csharp
var messageOne = new ServiceBusMessage(body);
var messageTwo = new ServiceBusMessage(body);

var sendFirstMessageTask =
    sender.SendMessageAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    sender.SendMessageAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");

```

# <a name="microsoftazureservicebus-sdk"></a>[Пакет SDK Microsoft. Azure. ServiceBus](#tab/net-standard-sdk)

```csharp
var messageOne = new Message(body);
var messageTwo = new Message(body);

var sendFirstMessageTask =
    queueClient.SendAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    queueClient.SendAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");
```

# <a name="windowsazureservicebus-sdk"></a>[Пакет SDK для WindowsAzure. ServiceBus](#tab/net-framework-sdk)

```csharp
var messageOne = new BrokeredMessage(body);
var messageTwo = new BrokeredMessage(body);

var sendFirstMessageTask =
    queueClient.SendAsync(messageOne).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #1");
    });
var sendSecondMessageTask =
    queueClient.SendAsync(messageTwo).ContinueWith(_ =>
    {
        Console.WriteLine("Sent message #2");
    });

await Task.WhenAll(sendFirstMessageTask, sendSecondMessageTask);
Console.WriteLine("All messages sent");
```

---

Вот пример асинхронной операции получения.

# <a name="azuremessagingservicebus-sdk"></a>[Пакет SDK для Azure. Messaging. ServiceBus](#tab/net-standard-sdk-2)

```csharp
var client = new ServiceBusClient(connectionString);
var options = new ServiceBusProcessorOptions 
{

      AutoCompleteMessages = false,
      MaxConcurrentCalls = 20
};
await using ServiceBusProcessor processor = client.CreateProcessor(queueName,options);
processor.ProcessMessageAsync += MessageHandler;
processor.ProcessErrorAsync += ErrorHandler;

static Task ErrorHandler(ProcessErrorEventArgs args)
{
    Console.WriteLine(args.Exception);
    return Task.CompletedTask;
};

static async Task MessageHandler(ProcessMessageEventArgs args)
{
    Console.WriteLine("Handle message");
    await args.CompleteMessageAsync(args.Message);
}

await processor.StartProcessingAsync();
```

# <a name="microsoftazureservicebus-sdk"></a>[Пакет SDK Microsoft. Azure. ServiceBus](#tab/net-standard-sdk)

Полные <a href="https://github.com/Azure/azure-service-bus/blob/master/samples/DotNet/Microsoft.Azure.ServiceBus/SendersReceiversWithQueues" target="_blank"> <span class="docon docon-navigate-external x-hidden-focus"></span> примеры исходного кода </a>см. в репозитории GitHub:

```csharp
var receiver = new MessageReceiver(connectionString, queueName, ReceiveMode.PeekLock);

static Task LogErrorAsync(Exception exception)
{
    Console.WriteLine(exception);
    return Task.CompletedTask;
};

receiver.RegisterMessageHandler(
    async (message, cancellationToken) =>
    {
        Console.WriteLine("Handle message");
        await receiver.CompleteAsync(message.SystemProperties.LockToken);
    },
    new MessageHandlerOptions(e => LogErrorAsync(e.Exception))
    {
        AutoComplete = false,
        MaxConcurrentCalls = 20
    });
```

`MessageReceiver`Экземпляр объекта создается с помощью строки подключения, имени очереди и режима получения просмотра. Затем `receiver` экземпляр используется для регистрации обработчика сообщений.

# <a name="windowsazureservicebus-sdk"></a>[Пакет SDK для WindowsAzure. ServiceBus](#tab/net-framework-sdk)

Полные <a href="https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/SendersReceiversWithQueues" target="_blank"> <span class="docon docon-navigate-external x-hidden-focus"></span> примеры исходного кода </a>см. в репозитории GitHub:

```csharp
var factory = MessagingFactory.CreateFromConnectionString(connectionString);
var receiver = await factory.CreateMessageReceiverAsync(queueName, ReceiveMode.PeekLock);

// Register the handler to receive messages asynchronously
receiver.OnMessageAsync(
    async message =>
    {
        Console.WriteLine("Handle message");
        await message.CompleteAsync();
    },
    new OnMessageOptions
    {
        AutoComplete = false,
        MaxConcurrentCalls = 20
    });
```

`MessagingFactory`Создает `factory` объект из строки подключения. Экземпляр создает экземпляр `factory` `MessageReceiver` . Затем `receiver` экземпляр используется для регистрации обработчика сообщений.

---

## <a name="receive-mode"></a>Режим получения

При создании клиента очереди или подписки вы можете выбрать режим получения: *блокировка для просмотра* или *получение и удаление*. Режим получения по умолчанию — `PeekLock`. При работе в режиме по умолчанию клиент отправляет запрос на получение сообщения из служебной шины. После получения сообщения клиент отправляет запрос на завершение сообщения.

При установке режима получения `ReceiveAndDelete` оба действия объединяются в одном запросе. В результате этих шагов уменьшается общее количество операций, что может повысить общую пропускную способность для сообщений. Однако такое повышение производительности сопряжено с риском потери сообщений.

Служебная шина не поддерживает транзакции для операций приема и удаления. Кроме того, семантика просмотра-блокировки необходима для сценариев, в которых клиент хочет отложить или [недоставленное](service-bus-dead-letter-queues.md) сообщение.

## <a name="client-side-batching"></a>Пакетная обработка на стороне клиента

Пакетная обработка на стороне клиента позволяет клиенту очереди или раздела задержать отправку сообщения на определенный период времени. Если в течение этого времени клиент будет отправлять дополнительные сообщения, они будут переданы в одном пакете. Кроме того, пакетная обработка на стороне клиента очереди или подписки позволяет объединить несколько запросов на **завершение** в один. Пакетная обработка может использоваться только для асинхронных операций **отправки** и **завершения**. Синхронные операции отправляются в службу служебной шины незамедлительно. Пакетная обработка не выполняется для операций просмотра или получения, а также не выполняется пакетной обработки на разных клиентах.

# <a name="azuremessagingservicebus-sdk"></a>[Пакет SDK для Azure. Messaging. ServiceBus](#tab/net-standard-sdk-2)
Функции пакетной обработки для пакета SDK для .NET Standard пока не предоставляют свойство для управления.

# <a name="microsoftazureservicebus-sdk"></a>[Пакет SDK Microsoft. Azure. ServiceBus](#tab/net-standard-sdk)

Функции пакетной обработки для пакета SDK для .NET Standard пока не предоставляют свойство для управления.

# <a name="windowsazureservicebus-sdk"></a>[Пакет SDK для WindowsAzure. ServiceBus](#tab/net-framework-sdk)

По умолчанию интервал пакетной обработки в клиенте составляет 20 мс. Интервал пакетной обработки можно изменить с помощью свойства [BatchFlushInterval][BatchFlushInterval], которое задается перед созданием фабрики обмена сообщениями. Этот параметр влияет на все клиенты, создаваемые этой фабрикой.

Чтобы отключить пакетную обработку, задайте для свойства [BatchFlushInterval][BatchFlushInterval] значение **TimeSpan.Zero**. Пример:

```csharp
var settings = new MessagingFactorySettings
{
    NetMessagingTransportSettings =
    {
        BatchFlushInterval = TimeSpan.Zero
    }
};
var factory = MessagingFactory.Create(namespaceUri, settings);
```

Пакетная обработка не влияет на число оплачиваемых операций обмена сообщениями и доступна только для клиентского протокола служебной шины с помощью библиотеки [Microsoft. servicebus. Messaging](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) . Протокол HTTP не поддерживает пакетную обработку.

> [!NOTE]
> Параметр `BatchFlushInterval` гарантирует, что Пакетная обработка будет неявной с точки зрения приложения. т. е. приложение создает `SendAsync` и `CompleteAsync` вызывает и не выполняет определенные вызовы пакетов.
>
> Явную пакетную обработку на стороне клиента можно реализовать, используя следующий вызов метода:
> ```csharp
> Task SendBatchAsync(IEnumerable<BrokeredMessage> messages);
> ```
> Здесь общий размер сообщений должен быть меньше, чем максимальный размер, поддерживаемый ценовой категорией.

---

## <a name="batching-store-access"></a>Пакетный доступ к хранилищу

Чтобы увеличить пропускную способность очереди, раздела или подписки, служебная шина объединяет несколько сообщений в пакеты, когда записывает их в свое внутреннее хранилище. 

- При включении пакетной обработки в очереди запись сообщений в хранилище и удаление сообщений из хранилища будут пакетированы. 
- При включении пакетной обработки в разделе выполняется пакетирование записи сообщений в хранилище. 
- При включении пакетной обработки в подписке удаляются сообщения из хранилища. 
- Если для сущности включен пакетный доступ к хранилищу, служебная шина задерживает операцию записи хранилища для этой сущности до 20 мс.

> [!NOTE]
> При пакетной обработке нет опасности потерять сообщения, даже если по истечении 20 мс такой обработки происходит сбой службы "Служебная шина".

Последующие операции с хранилищем, выполняемые в течение этого периода, добавляются в пакет. Пакетный доступ к хранилищу влияет только на операции **отправки** и **завершения** . операции получения не затрагиваются. Пакетный доступ к хранилищу является свойством сущности. Пакетная обработка применяется ко всем сущностям, для которых включен пакетный доступ к хранилищу.

При создании новой очереди, раздела или подписки пакетный доступ к хранилищу включен по умолчанию.


# <a name="azuremessagingservicebus-sdk"></a>[Пакет SDK для Azure. Messaging. ServiceBus](#tab/net-standard-sdk-2)
Для отключения пакетного доступа к хранилищу необходим экземпляр `ServiceBusAdministrationClient` . Создайте `CreateQueueOptions` из описания очереди, которое задает `EnableBatchedOperations` для свойства значение `false` .

```csharp
var options = new CreateQueueOptions(path)
{
    EnableBatchedOperations = false
};
var queue = await administrationClient.CreateQueueAsync(options);
```


# <a name="microsoftazureservicebus-sdk"></a>[Пакет SDK Microsoft. Azure. ServiceBus](#tab/net-standard-sdk)

Для отключения пакетного доступа к хранилищу необходим экземпляр `ManagementClient` . Создайте очередь из описания очереди, которая задает `EnableBatchedOperations` для свойства значение `false` .

```csharp
var queueDescription = new QueueDescription(path)
{
    EnableBatchedOperations = false
};
var queue = await managementClient.CreateQueueAsync(queueDescription);
```

Дополнительные сведения см. в следующих статьях:
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.management.queuedescription.enablebatchedoperations" target="_blank">`Microsoft.Azure.ServiceBus.Management.QueueDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.management.subscriptiondescription.enablebatchedoperations" target="_blank">`Microsoft.Azure.ServiceBus.Management.SubscriptionDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.management.topicdescription.enablebatchedoperations" target="_blank">`Microsoft.Azure.ServiceBus.Management.TopicDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

# <a name="windowsazureservicebus-sdk"></a>[Пакет SDK для WindowsAzure. ServiceBus](#tab/net-framework-sdk)

Для отключения пакетного доступа к хранилищу необходим экземпляр `NamespaceManager` . Создайте очередь из описания очереди, которая задает `EnableBatchedOperations` для свойства значение `false` .

```csharp
var queueDescription = new QueueDescription(path)
{
    EnableBatchedOperations = false
};
var queue = namespaceManager.CreateQueue(queueDescription);
```

Дополнительные сведения см. в следующих статьях:
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription.enablebatchedoperations" target="_blank">`Microsoft.ServiceBus.Messaging.QueueDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.enablebatchedoperations" target="_blank">`Microsoft.ServiceBus.Messaging.SubscriptionDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.topicdescription.enablebatchedoperations" target="_blank">`Microsoft.ServiceBus.Messaging.TopicDescription.EnableBatchedOperations` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

---

Пакетный доступ к хранилищу не влияет на количество оплачиваемых операций обмена сообщениями. Это свойство очереди, раздела или подписки. Он не зависит от режима приема и протокола, используемого между клиентом и службой служебной шины.

## <a name="prefetching"></a>Предварительная выборка

[Предварительная выборка](service-bus-prefetch.md) позволяет клиенту очереди или подписки загружать дополнительные сообщения из службы при получении сообщений. Клиент сохраняет эти сообщения в локальном кэше. Размер кэша определяется `QueueClient.PrefetchCount` `SubscriptionClient.PrefetchCount` свойствами или. В каждом клиенте с включенной предварительной выборкой создается собственный кэш. Кэш не используется несколькими клиентами. Если клиент запускает операцию Receive и кэш пуст, служба передает пакет сообщений. Размер пакета равен размеру кэша или 256 КБ (в зависимости от того, какое значение меньше). Если клиент запускает операцию получения, а кэш содержит сообщение, сообщение берется из кэша.

При использовании предварительной выборки сообщений служба блокирует выбранное сообщение. После блокировки сообщение с предвыборкой не может быть получено другим получателем. Если получателю не удается завершить сообщение до истечения срока действия блокировки, сообщение станет доступным другим получателям. Копия предварительно выбранного сообщения остается в кэше. Если получатель обратится к кэшированной копии, срок действия которой истек, то при попытке завершить это сообщение будет создано исключение. По умолчанию период блокировки сообщения составляет 60 секунд. Это значение можно увеличить до 5 минут. Чтобы предотвратить использование просроченных сообщений, задайте размер кэша меньше, чем количество сообщений, которые может использовать клиент в течение интервала времени ожидания блокировки.

При использовании срока блокировки по умолчанию в 60 секунд для `PrefetchCount` хорошо подходит значение, в 20 раз превышающее максимальную скорость обработки для всех получателей фабрики. Например, фабрика создает трех получателей, каждый из которых может обработать до 10 сообщений в секунду. Число предвыборки не должно превышать 20 X 3 X 10 = 600. По умолчанию `PrefetchCount` имеет значение 0, что означает, что никакие дополнительные сообщения из службы не извлекаться.

Предварительная выборка сообщений увеличивает совокупную пропускную способность очереди или подписки, так как уменьшает общее количество операций с сообщениями (так называемых круговых путей). Тем не менее выборка первого сообщения займет больше времени (из-за увеличенного размера сообщения). Получение предварительно выбранных сообщений из кэша будет выполняться быстрее, так как эти сообщения уже скачаны клиентом.

Свойство срока жизни сообщения проверяется сервером в момент отправки сообщения клиенту. Клиент не проверяет свойство TTL сообщения при получении сообщения. Вместо этого сообщение может быть получено, даже если срок жизни сообщения истек, пока сообщение было кэшировано клиентом.

Предвыборка не влияет на число оплачиваемых операций обмена сообщениями и доступна только для протокола клиента служебной шины. Протокол HTTP не поддерживает предзагрузку. Предварительная выборка доступна как для синхронных, так и для асинхронных операций получения.

# <a name="azuremessagingservicebus-sdk"></a>[Пакет SDK для Azure. Messaging. ServiceBus](#tab/net-standard-sdk-2)
Дополнительные сведения см. в следующих `PrefetchCount` свойствах:

- [Сервицебусрецеивер. PrefetchCount](/dotnet/api/azure.messaging.servicebus.servicebusreceiver.prefetchcount)
- [Сервицебуспроцессор. PrefetchCount](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.prefetchcount)

Значения этих свойств можно задать в [сервицебусрецеивероптионс](/dotnet/api/azure.messaging.servicebus.servicebusreceiveroptions) или [сервицебуспроцессороптионс](/dotnet/api/azure.messaging.servicebus.servicebusprocessoroptions).

# <a name="microsoftazureservicebus-sdk"></a>[Пакет SDK Microsoft. Azure. ServiceBus](#tab/net-standard-sdk)

Дополнительные сведения см. в следующих `PrefetchCount` свойствах:

* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount" target="_blank">`Microsoft.Azure.ServiceBus.QueueClient.PrefetchCount` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.azure.servicebus.subscriptionclient.prefetchcount" target="_blank">`Microsoft.Azure.ServiceBus.SubscriptionClient.PrefetchCount` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

# <a name="windowsazureservicebus-sdk"></a>[Пакет SDK для WindowsAzure. ServiceBus](#tab/net-framework-sdk)

Дополнительные сведения см. в следующих `PrefetchCount` свойствах:

* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient.prefetchcount" target="_blank">`Microsoft.ServiceBus.Messaging.QueueClient.PrefetchCount` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
* <a href="https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.subscriptionclient.prefetchcount" target="_blank">`Microsoft.ServiceBus.Messaging.SubscriptionClient.PrefetchCount` <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

---

## <a name="prefetching-and-receivebatch"></a>Упреждающая выборка и Рецеивебатч
Хотя основные понятия, связанные с выборке нескольких сообщений, имеют схожую семантику для обработки сообщений в пакете ( `ReceiveBatch` ), существуют некоторые небольшие отличия, которые следует учитывать при совместном использовании этих подходов.

Предварительная выборка — это конфигурация (или режим) на клиенте ( `QueueClient` и `SubscriptionClient` ), `ReceiveBatch` которая является операцией (с семантикой запросов и ответов).

При совместном использовании этих подходов рассмотрим следующие случаи.

* Предвыборка должна быть больше или равна количеству сообщений, из которых ожидается получение `ReceiveBatch` .
* При выборке может быть не более n/3 раз больше количества сообщений, обрабатываемых в секунду, где n — продолжительность блокировки по умолчанию.

Существуют некоторые трудности с беспроблемным подходом, т. е. поддержание максимального числа предварительных выборок, поскольку предполагает, что сообщение заблокировано определенным получателем. Рекомендуется испробовать значения предварительного выборки между пороговыми значениями, упомянутыми выше, и определить, что помещается.

## <a name="multiple-queues"></a>Несколько очередей

Если одна очередь или раздел не может справиться с ожидаемым, используйте несколько сущностей обмена сообщениями. При использовании нескольких сущностей создайте выделенный клиент для каждой сущности. Не используйте один клиент для всех сущностей.

## <a name="development-and-testing-features"></a>Возможности для разработки и тестирования

> [!NOTE]
> Этот раздел применим только к пакету SDK для WindowsAzure. ServiceBus, так как Microsoft. Azure. ServiceBus и Azure. Messaging. ServiceBus не предоставляют эту функцию.

В служебной шине есть один компонент, используемый специально для разработки, который **никогда не должен использоваться в рабочих конфигурациях**: [`TopicDescription.EnableFilteringMessagesBeforePublishing`][TopicDescription.EnableFiltering] .

При добавлении к разделу новых правил или фильтров можно использовать [`TopicDescription.EnableFilteringMessagesBeforePublishing`][TopicDescription.EnableFiltering] для проверки правильности работы нового критерия фильтра.

## <a name="scenarios"></a>Сценарии

Далее описываются стандартные сценарии обмена сообщениями и приводятся рекомендации по настройкам служебной шины. Пропускная способность бывает небольшой (менее 1 сообщения в секунду), средней (1 сообщение в секунду или больше, но не более 100 сообщений в секунду) и высокой (100 сообщений в секунду или больше). Количество клиентов бывает небольшим (5 и меньше), средним (более 5, но не более 20) и большим (более 20).

### <a name="high-throughput-queue"></a>Очередь с высокой пропускной способностью

Цель: максимально повысить пропускную способность одной очереди. Количество отправителей и получателей: небольшое.

* Чтобы увеличить общую скорость отправки сообщений в очередь, используйте несколько фабрик обмена сообщениями для создания отправителей. Для каждого отправителя используйте асинхронные операции или разделение на потоки.
* Чтобы увеличить общую скорость получения сообщений из очереди, используйте несколько фабрик обмена сообщениями для создания получателей.
* Используйте асинхронные операции, чтобы воспользоваться преимуществами пакетной обработки на стороне клиента.
* Установите интервал пакетной обработки на 50 мс, чтобы уменьшить число передач по протоколу клиента служебной шины. Если используется несколько отправителей, увеличьте интервал пакетной обработки до 100 мс.
* Не отключайте пакетный доступ к хранилищу. Этот доступ повысит общую скорость записи сообщений в очередь.
* Для количества элементов предварительной выборки установите значение, в 20 раз превышающее максимальную скорость обработки всех получателей фабрики. Это снизит число передач по протоколу клиента служебной шины.

### <a name="multiple-high-throughput-queues"></a>Несколько очередей с высокой пропускной способностью

Цель: максимально повысить общую пропускную способность нескольких очередей. Пропускная способность одной очереди: средняя или высокая.

Чтобы максимально повысить пропускную способность нескольких очередей, примените настройки, максимально повышающие пропускную способность одной очереди. Кроме того, используйте разные фабрики для создания клиентов, которые отправляют или получают из разных очередей.

### <a name="low-latency-queue"></a>Очередь с небольшой задержкой

Цель: сократите задержку очереди или раздела. Количество отправителей и получателей: небольшое. Пропускная способность очереди: низкая или средняя.

* Отключите пакетную обработку на стороне клиента. Клиент будет немедленно отправлять сообщения.
* Отключите пакетный доступ к хранилищу. Служба будет немедленно записывать сообщения в хранилище.
* При использовании одного клиента измените количество элементов предварительной выборки на значение, в 20 раз превышающее скорость обработки получателя. Если в очередь одновременно поступает несколько сообщений, протокол клиента служебной шины передаст их все одновременно. Когда клиент получит следующее сообщение, оно уже будет в локальном кэше. Размер кэша должен быть небольшим.
* При использовании нескольких клиентов установите количество элементов предварительной выборки на 0. В результате этого второй клиент сможет получить второе сообщение, пока первый клиент будет обрабатывать первое сообщение.

### <a name="queue-with-a-large-number-of-senders"></a>Очередь с большим числом отправителей

Цель: максимально повысить пропускную способность очереди или раздела с большим количеством отправителей. Каждый отправитель отправляет сообщения со средней скоростью. Количество получателей: небольшое.

Служебная шина позволяет до 1000 одновременных подключений к сущности обмена сообщениями. Это ограничение применяется на уровне пространства имен, а очереди, разделы или подписки ограничены ограничением количества одновременных подключений на пространство имен. В случае очередей это значение распределяется между отправителями и получателями. Если отправителям требуются все 1000 подключений, замените очередь одним разделом и одной подпиской. Раздел принимает до 1000 одновременных подключений от отправителей. Подписка принимает дополнительные 1000 параллельных подключений от получателей. Если количество параллельных отправителей превышает 1000, то они должны отправлять сообщения на протокол служебной шины по протоколу HTTP.

Чтобы повысить пропускную способность, выполните следующие действия.

* Если каждый отправитель находится в другом процессе, используйте только одну фабрику на процесс.
* Используйте асинхронные операции, чтобы воспользоваться преимуществами пакетной обработки на стороне клиента.
* Установите интервал пакетной обработки на 20 мс, чтобы уменьшить число передач по протоколу клиента служебной шины.
* Не отключайте пакетный доступ к хранилищу. Этот доступ повысит общую скорость записи сообщений в очередь или раздел.
* Для количества элементов предварительной выборки установите значение, в 20 раз превышающее максимальную скорость обработки всех получателей фабрики. Это снизит число передач по протоколу клиента служебной шины.

### <a name="queue-with-a-large-number-of-receivers"></a>Очередь с большим числом получателей

Цель: максимально повысить скорость получения в очереди или подписке с большим количеством получателей. Каждый получатель получает сообщения со средней скоростью. Количество отправителей: небольшое.

В служебной шине можно открыть до 1000 параллельных подключений к сущности. Если для очереди требуется более 1000 получателей, замените очередь одним разделом и несколькими подписками. Каждая подписка поддерживает до 1000 параллельных подключений. Кроме того, получатели могут обращаться к очереди через протокол HTTP.

Чтобы повысить пропускную способность, следуйте приведенным ниже рекомендациям.

* Если каждый получатель находится в другом процессе, используйте только одну фабрику для каждого процесса.
* Получатели могут использовать синхронные и асинхронные операции. Учитывая среднюю скорость получения отдельного получателя, пакетная обработка на стороне клиента полного запроса не влияет на пропускную способность получателя.
* Не отключайте пакетный доступ к хранилищу. Это снизит общую нагрузку на сущность. Кроме того, это понизит общую скорость записи сообщений в очередь или раздел.
* Для количества элементов предварительной выборки установите небольшое значение (например, PrefetchCount = 10). Это позволит избежать ситуации, когда одни получатели простаивают, а другие получают множество кэшированных сообщений.

### <a name="topic-with-a-few-subscriptions"></a>Раздел с несколько подписок

Цель: максимально увеличить пропускную способность раздела с помощью нескольких подписок. Сообщение получается несколькими подписками. Это означает, что общая скорость получения во всех подписках будет больше, чем скорость отправки. Количество отправителей: небольшое. Получателей в каждой подписке так же немного.

Чтобы повысить пропускную способность, следуйте приведенным ниже рекомендациям.

* Чтобы увеличить общую скорость отправки сообщений в раздел, используйте несколько фабрик обмена сообщениями для создания отправителей. Для каждого отправителя используйте асинхронные операции или разделение на потоки.
* Чтобы увеличить общую скорость получения сообщений из подписки, используйте несколько фабрик обмена сообщениями для создания получателей. Для каждого получателя используйте асинхронные операции или разделение на потоки.
* Используйте асинхронные операции, чтобы воспользоваться преимуществами пакетной обработки на стороне клиента.
* Установите интервал пакетной обработки на 20 мс, чтобы уменьшить число передач по протоколу клиента служебной шины.
* Не отключайте пакетный доступ к хранилищу. Это повысит общую скорость записи сообщений в раздел.
* Для количества элементов предварительной выборки установите значение, в 20 раз превышающее максимальную скорость обработки всех получателей фабрики. Это снизит число передач по протоколу клиента служебной шины.

### <a name="topic-with-a-large-number-of-subscriptions"></a>Раздел с большим количеством подписок

Цель: максимально повысить пропускную способность раздела с большим количеством подписок. Сообщение направляется нескольким подпискам. Это означает, что общая скорость получения во всех подписках будет значительно больше, чем скорость отправки. Количество отправителей: небольшое. Получателей в каждой подписке так же немного.

Разделы с большим количеством подписок обычно обеспечивают низкую общую пропускную способность, если все сообщения направляются во все подписки. Это связано с тем, что каждое сообщение получается много раз, и все сообщения в разделе и все его подписки хранятся в одном хранилище. Предполагается, что количество отправителей и количество получателей на подписку мало. Служебная шина поддерживает до 2000 подписок на раздел.

Чтобы максимально повысить пропускную способность, попробуйте выполнить следующие шаги:

* Используйте асинхронные операции, чтобы воспользоваться преимуществами пакетной обработки на стороне клиента.
* Установите интервал пакетной обработки на 20 мс, чтобы уменьшить число передач по протоколу клиента служебной шины.
* Не отключайте пакетный доступ к хранилищу. Это повысит общую скорость записи сообщений в раздел.
* Для количества элементов предварительной выборки установите значение, в 20 раз превышающее ожидаемую скорость получения в секундах. Это снизит число передач по протоколу клиента служебной шины.

<!-- .NET Standard SDK, Microsoft.Azure.ServiceBus -->
[QueueClient]: /dotnet/api/microsoft.azure.servicebus.queueclient
[MessageSender]: /dotnet/api/microsoft.azure.servicebus.core.messagesender

<!-- .NET Framework SDK, Microsoft.Azure.ServiceBus -->
[MessagingFactory]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[BatchFlushInterval]: /dotnet/api/microsoft.servicebus.messaging.messagesender.batchflushinterval
[ForcePersistence]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage.forcepersistence
[EnablePartitioning]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.enablepartitioning
[TopicDescription.EnableFiltering]: /dotnet/api/microsoft.servicebus.messaging.topicdescription.enablefilteringmessagesbeforepublishing

<!-- Local links -->
[Partitioned messaging entities]: service-bus-partitioning.md
