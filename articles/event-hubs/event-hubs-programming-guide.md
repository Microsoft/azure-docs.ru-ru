---
title: Руководством по программированию для .NET — концентраторы событий Azure (устаревший) | Документация Майкрософт
description: В этой статье приводятся сведения о том, как писать код для службы "Центры событий Azure" с помощью пакета Azure SDK для .NET.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 32c3c05b61d2ee8fc79d7c863ddbe84de5fe7e2b
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102432746"
---
# <a name="net-programming-guide-for-azure-event-hubs-legacy-microsoftazureeventhubs-package"></a>Руководством по программированию .NET для концентраторов событий Azure (устаревший пакет Microsoft. Azure. EventHubs)
В данной статье обсуждаются некоторые распространенные сценарии написания кодов с помощью Центров событий Azure. Предполагается, что вы уже имеете представление о Центрах событий. Общие сведения о Центрах событий см. в статье [Общие сведения о Центрах событий Azure](./event-hubs-about.md).

> [!WARNING]
> Это краткое справочное по для старого пакета **Microsoft. Azure. EventHubs** . Мы рекомендуем [перенести](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md) код, чтобы использовать последний пакет [Azure. Messaging. EventHubs](event-hubs-dotnet-standard-getstarted-send.md) .  


## <a name="event-publishers"></a>Издатели событий

Отправка событий в концентратор событий осуществляется с использованием HTTP POST или через подключение AMQP 1.0. Выбор способа и времени зависит от определенного сценария, к которому выполняется обращение. Подключения AMQP 1.0 измеряются как подключения через посредника по служебной шине. Они больше всего подходят для сценариев с большими объемами сообщений и менее строгими требованиями к задержке, так как такие подключения обеспечивают постоянный канал обмена сообщениями.

При использовании управляемых API .NET основными конструктивными элементами для публикации данных в Центрах событий являются классы [EventHubClient][] и [EventData][]. [EventHubClient][] предоставляет канал связи AMQP, по которому события отправляются в концентратор событий. Класс [EventData][] представляет собой событие и используется для публикации сообщений в концентраторе событий. Этот класс включает тело, некоторые метаданные (свойства) и сведения о заголовке (Системпропертиес) о событии. По мере перемещения объекта [EventData][] через концентратор событий к объекту добавляются другие свойства.

## <a name="get-started"></a>Начало работы
Классы .NET, поддерживающие Центры событий, входят в пакет NuGet [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/). Центр событий можно установить с помощью обозревателя решений Visual Studio или [консоли диспетчера пакетов](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) в Visual Studio. Для этого выполните следующую команду в окне [консоли диспетчера пакетов](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) :

```shell
Install-Package Microsoft.Azure.EventHubs
```

## <a name="create-an-event-hub"></a>Создание концентратора событий

Вы можете использовать портал Azure, службу Azure PowerShell или Azure CLI для создания Центров событий. Подробности см. в статье [Создание пространства имен концентраторов событий и концентратора событий с помощью портала Azure](event-hubs-create.md).

## <a name="create-an-event-hubs-client"></a>Создание клиента Центров событий

Основным классом для взаимодействия с концентраторами событий является класс [Microsoft.Azure.EventHubs.EventHubClient][EventHubClient]. Можно создать экземпляр этого класса, используя метод [CreateFromConnectionString](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createfromconnectionstring), как показано в следующем примере:

```csharp
private const string EventHubConnectionString = "Event Hubs namespace connection string";
private const string EventHubName = "event hub name";

var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
{
    EntityPath = EventHubName

};
eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
```

## <a name="send-events-to-an-event-hub"></a>Отправка событий в концентратор событий

Для отправки событий в концентратор событий создается экземпляр [EventHubClient][], который отправляется асинхронно с помощью метода [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync). Этот метод принимает один параметр экземпляра [EventData][] и синхронно отправляет его в концентратор событий.

## <a name="event-serialization"></a>Сериализация событий

Класс [EventData][] имеет [два перегруженных конструктора](/dotnet/api/microsoft.azure.eventhubs.eventdata.-ctor), которые принимают различные параметры, байты или массив байтов, которые представляют полезные данные событий. При использовании JSON совместно с [EventData][]можно применить метод **Encoding.UTF8.GetBytes()** для получения массива байтов для строки в кодировке JSON. Пример:

```csharp
for (var i = 0; i < numMessagesToSend; i++)
{
    var message = $"Message {i}";
    Console.WriteLine($"Sending message: {message}");
    await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
}
```

## <a name="partition-key"></a>Ключ секции

> [!NOTE]
> Если вы не знакомы с секциями, см. [эту статью](event-hubs-features.md#partitions). 

При отправке данных событий можно указать значение, которое хэшируется для создания назначения секции. Укажите свойство секции, используя [PartitionSender.PartitionID](/dotnet/api/microsoft.azure.eventhubs.partitionsender.partitionid). Кроме того, решение об использовании секций подразумевает выбор между доступностью и целостностью. Дополнительные сведения см. в разделе [доступность и согласованность](event-hubs-availability-and-consistency.md).

## <a name="batch-event-send-operations"></a>Пакетные операции отправки событий

Пакетная отправка событий позволяет повысить пропускную способность. Вы можете использовать API [CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch) для создания пакета, в который можно позже добавить объекты данных для вызова [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync).

Размер одного пакета не должен превышать 1 МБ для события. Кроме того, каждое сообщение в пакете использует один и тот же идентификатор издателя. Отправитель должен убедиться, что размер пакета не превышает максимальный размер события. Если размер превышен, возникает ошибка **отправки**. Можно использовать вспомогательный метод [EventHubClient.CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch), чтобы убедиться, что пакет не превысит 1 МБ. Вы получаете пустой пакет [EventDataBatch](/dotnet/api/microsoft.azure.eventhubs.eventdatabatch) из API [CreateBatch](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createbatch), а затем с помощью [TryAdd](/dotnet/api/microsoft.azure.eventhubs.eventdatabatch.tryadd) добавляете события для создания пакета. 

## <a name="send-asynchronously-and-send-at-scale"></a>Асинхронная отправка и отправка в нужном масштабе

События можно отправлять в концентратор событий асинхронно. Асинхронная отправка увеличивает частоту, с которой клиент может отправлять события. [SendAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync) возвращает объект [Task](/dotnet/api/system.threading.tasks.task). На стороне клиента можно использовать класс [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) для управления параметрами повторного выполнения попыток клиентом.

## <a name="event-consumers"></a>Получатели событий
Класс [EventProcessorHost][] обрабатывает данные из Центров событий. Эту реализацию следует использовать при создании модулей чтения событий на платформе .NET. [EventProcessorHost][] предоставляет потокобезопасную многопроцессную среду безопасного выполнения для реализаций обработчиков событий. Эта среда также предоставляет средства управления контрольными точками и аренды секций.

Чтобы воспользоваться классом [EventProcessorHost][], можно реализовать интерфейс [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor). Этот интерфейс содержит четыре метода:

* [OpenAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.openasync)
* [CloseAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.closeasync)
* [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync)
* [ProcessErrorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processerrorasync);

Чтобы начать обработку событий, создайте экземпляр [EventProcessorHost][], указав соответствующие параметры для концентратора событий. Пример:

> [!NOTE]
> EventProcessorHost и связанные с ним классы предоставляются в пакете **Microsoft. Azure. EventHubs. Processor** . Добавьте пакет в проект Visual Studio, следуя инструкциям в [этой статье](event-hubs-dotnet-framework-getstarted-send.md#add-the-event-hubs-nuget-package) или выполнив следующую команду в окне [консоли диспетчера пакетов](https://docs.nuget.org/docs/start-here/using-the-package-manager-console) : `Install-Package Microsoft.Azure.EventHubs.Processor` .

```csharp
var eventProcessorHost = new EventProcessorHost(
        EventHubName,
        PartitionReceiver.DefaultConsumerGroupName,
        EventHubConnectionString,
        StorageConnectionString,
        StorageContainerName);
```

Затем вызовите [регистеревентпроцессорасинк](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync) , чтобы зарегистрировать реализацию [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) в среде выполнения:

```csharp
await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();
```

На этом этапе узел попытается получить аренду для каждой секции в концентраторе событий, используя "жадный" алгоритм. Эти аренды будут продолжаться в течение заданного промежутка времени, после чего их необходимо продлить. По мере перехода новых узлов (в нашем случае это рабочие экземпляры) в оперативный режим они размещают резервирования аренды и со временем нагрузка смещается между узлами при каждой попытке получения дополнительной аренды.

![Узел обработчика событий](./media/event-hubs-programming-guide/IC759863.png)

Со временем устанавливается равновесие. Такая динамическая возможность позволяет применить к потребителям автоматическое масштабирование как вверх, так и вниз на основе использования ресурсов ЦП. Так как в Центрах событий не реализован прямой механизм подсчета сообщений, среднее использование ресурсов ЦП зачастую является оптимальным для измерения масштабирования серверной части или потребителя. Если издатели начинают публиковать больше событий, чем могут обработать потребители, увеличение использования ресурсов ЦП потребителями можно использовать для автоматического масштабирования на основе числа экземпляров исполнителей.

Класс [EventProcessorHost][] также реализует механизм создания контрольных точек на основе службы хранилища Azure. Этот механизм хранит смещение для каждой секции, чтобы каждый потребитель мог определить последнюю контрольную точку от предыдущего потребителя. По мере перехода секций между узлами перенос нагрузки осуществляется за счет механизма синхронизации.

## <a name="publisher-revocation"></a>Отзыв издателя

В дополнение к расширенным функциям, выполняемым узлом обработчика событий, служба концентраторов событий включает [отзыв издателя](/rest/api/eventhub/revoke-publisher) , чтобы запретить конкретным издателям отправлять события в концентратор событий. Эти функции полезны, если маркер издателя скомпрометирован или после обновления программного обеспечения эти функции перестали работать должным образом. В этих случаях для идентификатора издателя, который является частью маркера SAS, можно заблокировать возможность публикации событий.

> [!NOTE]
> В настоящее время эта функция поддерживается только REST API ([отзыв издателя](/rest/api/eventhub/revoke-publisher)).


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о сценариях Центров событий см. в разделах, ссылки на которые указаны ниже.

* [Общие сведения об API Центров событий](./event-hubs-samples.md)
* [Что такое концентраторы событий](./event-hubs-about.md)
* [Доступность и согласованность в Центрах событий](event-hubs-availability-and-consistency.md)
* [Справочник по API узла обработчика событий](/dotnet/api/microsoft.servicebus.messaging.eventprocessorhost)

[NamespaceManager]: /dotnet/api/microsoft.servicebus.namespacemanager
[EventHubClient]: /dotnet/api/microsoft.azure.eventhubs.eventhubclient
[EventData]: /dotnet/api/microsoft.azure.eventhubs.eventdata
[CreateEventHubIfNotExists]: /dotnet/api/microsoft.servicebus.namespacemanager.createeventhubifnotexists
[PartitionKey]: /dotnet/api/microsoft.servicebus.messaging.eventdata#Microsoft_ServiceBus_Messaging_EventData_PartitionKey
[EventProcessorHost]: /dotnet/api/microsoft.azure.eventhubs.processor
