---
title: Общие сведения об обработке транзакций в служебной шине Azure
description: В этой статье приводятся общие сведения об обработке транзакций и функции отправки через служебную шину Azure.
ms.topic: article
ms.date: 03/03/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: e2848f41d5557584b0f1a197b548a00a4aef1564
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102183749"
---
# <a name="overview-of-service-bus-transaction-processing"></a>Обзор обработки транзакций в служебной шине

Эта статья описывает возможности служебной шины Microsoft Azure по работе с транзакциями. Большая часть обсуждения иллюстрируется [AMQP транзакциями с помощью служебной шины](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/TransactionsAndSendVia/TransactionsAndSendVia/AMQPTransactionsSendVia). Эта статья рассматривает лишь обработку транзакций и функцию *отправить через* в служебной шине, но пример атомарных транзакций образец гораздо масштабнее и сложнее.

> [!NOTE]
> Базовый уровень служебной шины не поддерживает транзакции. Уровни "Стандартный" и "Премиум" поддерживают транзакции. Различия между этими ценовыми категориями приведены на странице [цен на Служебную шину](https://azure.microsoft.com/pricing/details/service-bus/).

## <a name="transactions-in-service-bus"></a>Транзакции в служебной шине

*Транзакция* группирует две или более операций в *область выполнения*. По своей природе такая транзакция должна обеспечивать либо успешное, либо неудачное выполнение всех относящихся к данной группе операций. В этом отношении транзакция выступает в качестве единого целого, что часто называется *атомарностью*.

Служебная шина является брокером для транзакционных сообщений и гарантирует целостность всех внутренних операций в соответствии с хранилищами сообщений. Все передачи сообщений внутри служебной шины, например перемещение сообщений в [очередь недоставленных сообщений](service-bus-dead-letter-queues.md) или [автоматическая пересылка](service-bus-auto-forwarding.md) сообщений между сущностями, являются транзакционными. Таким образом, когда служебная шина принимает сообщение, оно уже сохранено и помечено порядковым номером. Начиная с этого момента любые передачи сообщений в служебной шине, являются скоординированными операциями между сущностями и не приведут ни к потере (успех исходной части, сбой целевой части), ни к дублированию (сбой исходной части, успех целевой части) сообщения.

Служебная шина поддерживает операции группирования относительно одной сущности обмена сообщениями (очереди, раздела, подписки) в области транзакции. Например, можно отправить несколько сообщений в одну очередь из области транзакций, и сообщение будет зафиксировано в журнале очереди только после успешного завершения транзакции.

## <a name="operations-within-a-transaction-scope"></a>Операции в области транзакций

Далее приведены операции, которые могут выполняться в области транзакций.

* **[QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient), [MessageSender](/dotnet/api/microsoft.azure.servicebus.core.messagesender), [TopicClient](/dotnet/api/microsoft.azure.servicebus.topicclient)**: `Send` , `SendAsync` , `SendBatch` ,`SendBatchAsync`
* **[BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage)**: `Complete` , `CompleteAsync` , `Abandon` , `AbandonAsync` , `Deadletter` , `DeadletterAsync` , `Defer` , `DeferAsync` , `RenewLock` , `RenewLockAsync` 

Операции получения не указаны, так как предполагается, что приложение получает сообщения с помощью режима [ReceiveMode.PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode), внутри некоторого цикла получения или с помощью с обратного вызова [OnMessage](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessage) и только затем открывает область транзакций для обработки сообщения.

Обработка сообщения (выполнено, прервано, отложено, не доставлено) происходит в пределах области и зависит от общего результата транзакции.

## <a name="transfers-and-send-via"></a>Передачи и функция "отправить через"

Чтобы включить транзакционную передачи вам образа данных из очереди или раздела на процессор, а затем в другую очередь или раздел, служебная шина поддерживает *передачи*. При выполнении операции передачи отправитель сначала отправляет сообщение в *очередь передачи или раздел*, а очередь передачи или раздел немедленно перемещает сообщение в предполагаемую очередь или раздел назначения, используя ту же надежную реализацию передачи, которую использует функция перемотки. Сообщение никогда не фиксируется в журнале пересылки или разделе так, чтобы оно стало видимым для потребителей очереди или пользователя раздела.

Мощь этой транзакционной возможности становится очевидной, когда очередь передачи или сам раздел является источником входных сообщений отправителя. Иными словами, служебная шина может передавать сообщение в очередь назначения или раздел «через» в очередь или раздел «перемещение», выполняя операцию (или отложить или недоставленную) для входного сообщения, всего за одну атомарную операцию. 

Если необходимо получить из подписки раздела, а затем отправить в очередь или раздел в той же транзакции, то сущность передачи должна быть темой. В этом сценарии вы запускаете область транзакций в разделе, получаете из подписки с в области действия транзакции и отправляете его через раздел передача в очередь или назначение раздела. 

### <a name="see-it-in-code"></a>Пример кода

Для настройки таких передач вы создаете отправитель сообщений, сориентированный на очередь назначения через очередь передачи. Также есть получатель, который извлекает сообщения из той же очереди. Пример:

```csharp
var connection = new ServiceBusConnection(connectionString);

var sender = new MessageSender(connection, QueueName);
var receiver = new MessageReceiver(connection, QueueName);
```

Затем простая транзакция использует эти элементы, как показано в следующем примере. Чтобы ознакомиться с полным примером, обратитесь к [исходному коду на сайте GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/TransactionsAndSendVia/TransactionsAndSendVia/AMQPTransactionsSendVia):

```csharp
var receivedMessage = await receiver.ReceiveAsync();

using (var ts = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    try
    {
        // do some processing
        if (receivedMessage != null)
            await receiver.CompleteAsync(receivedMessage.SystemProperties.LockToken);

        var myMsgBody = new MyMessage
        {
            Name = "Some name",
            Address = "Some street address",
            ZipCode = "Some zip code"
        };

        // send message
        var message = myMsgBody.AsMessage();
        await sender.SendAsync(message).ConfigureAwait(false);
        Console.WriteLine("Message has been sent");

        // complete the transaction
        ts.Complete();
    }
    catch (Exception ex)
    {
        // This rolls back send and complete in case an exception happens
        ts.Dispose();
        Console.WriteLine(ex.ToString());
    }
}
```

## <a name="timeout"></a>Время ожидания
Время ожидания транзакции истекло через 2 минуты. Таймер транзакции запускается, когда начинается первая операция в транзакции. 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об очередях служебной шины см. в следующих статьях:

* [Как использовать очереди служебной шины](service-bus-dotnet-get-started-with-queues.md)
* [Объединение в цепочки сущностей служебной шины с помощью автоматической переадресации](service-bus-auto-forwarding.md)
* [Пример с пересылкой](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/AutoForward)
* [Пример выполнения атомарных транзакций с помощью служебной шины](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/AtomicTransactions)
* [Сравнение службы очередей Azure и службы очередей служебной шины](service-bus-azure-and-service-bus-queues-compared-contrasted.md)


