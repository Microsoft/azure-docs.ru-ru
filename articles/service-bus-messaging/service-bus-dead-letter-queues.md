---
title: Очереди недоставленных сообщений служебной шины | Документация Майкрософт
description: В статье рассматриваются очереди недоставленных сообщений в служебной шине Azure. Очереди служебной шины и подписки на разделы предоставляют дополнительную очередь, которая называется очередью недоставленных сообщений.
ms.topic: article
ms.date: 06/23/2020
ms.custom: fasttrack-edit, devx-track-csharp
ms.openlocfilehash: ad62f946584071e7ce6fd55f48b5f7ee8db44a2f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98630104"
---
# <a name="overview-of-service-bus-dead-letter-queues"></a>Обзор очередей недоставленных сообщений служебной шины

Очереди служебной шины Azure и подписки на разделы предоставляют дополнительную очередь, которая называется *очередью недоставленных сообщений*. Очередь недоставленных сообщений не нужно создавать явным образом. Ее нельзя удалить или как-либо управлять ею отдельно от основной сущности.

В этой статье рассматриваются очереди недоставленных сообщений в служебной шине. Большую часть статьи иллюстрирует [пример для очередей недоставленных сообщений](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/DeadletterQueue), доступный на сайте GitHub.
 
## <a name="the-dead-letter-queue"></a>Очередь недоставленных сообщений

Очередь недоставленных сообщений предназначена для хранения сообщений, которые невозможно доставить ни одному адресату либо не удалось обработать. Сообщение из очереди DLQ можно извлечь и изучить. Приложение с помощью оператора может исправить проблемы и повторно отправить это сообщение, зарегистрировать ошибку и предпринять действие по исправлению. 

С точки зрения интерфейса API и протокола очередь DLQ очень похожа на другие очереди за исключением того, что сообщения могут отправляться только посредством операции недоставленного сообщения родительской сущности. Кроме того, в ней не отслеживается время жизни и из нее нельзя удалить сообщение. Очередь недоставленных сообщений полностью поддерживает доставку с блокировкой для просмотра и транзакционные операции.

Автоматическая очистка очереди DLQ не выполняется. Сообщения остаются в очереди DLQ, пока вы явно не извлечете их оттуда и не вызовете метод [Complete()](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) для недоставленного сообщения.

## <a name="dlq-message-count"></a>Количество сообщений в очереди недоставленных сообщений
Нет возможности получить количество сообщений в очереди недоставленных сообщений на уровне разделов. Это обусловлено тем, что эти сообщения не сохраняются на уровне разделов, если в служебной шине не происходило внутренних ошибок. Вместо этого, когда отправитель отправляет сообщение в раздел, оно за несколько миллисекунд передается в подписку этого раздела и уже не находится на уровне раздела. Это означает, что сообщения нужно просматривать в очереди недоставленных сообщений, сопоставленной с подпиской нужного раздела. В следующем примере **Service Bus Explorer** показывает, что в очереди недоставленных сообщений для подписки test1 есть 62 сообщения. 

![Количество сообщений в очереди недоставленных сообщений](./media/service-bus-dead-letter-queues/dead-letter-queue-message-count.png)

Количество сообщений в очереди недоставленных сообщений можно узнать с помощью команды Azure CLI [`az servicebus topic subscription show`](/cli/azure/servicebus/topic/subscription#az-servicebus-topic-subscription-show). 

## <a name="moving-messages-to-the-dlq"></a>Перемещение сообщений в очередь DLQ

В служебной шине есть несколько действий, которые приводят к отправке сообщений в очередь DLQ из самой системы обработки сообщений. Сообщения также могут явно перемещаться в очередь DLQ самим приложением. 

Когда брокер перемещает сообщение, к нему добавляется два свойства в момент вызова брокером своей внутренней версии метода [DeadLetter](/dotnet/api/microsoft.azure.servicebus.queueclient.deadletterasync) для сообщения: `DeadLetterReason` и `DeadLetterErrorDescription`.

Приложения могут определять свои собственные коды для свойства `DeadLetterReason`, однако система задает следующие значения.

| DeadLetterReason | DeadLetterErrorDescription |
| --- | --- |
|HeaderSizeExceeded |Превышен максимальный размер для этого потока. |
|TTLExpiredException |Время жизни сообщения истекло, и оно было перемещено в очередь недоставленных. Дополнительные сведения см. в разделе [Превышение значения TimeToLive](#exceeding-timetolive). |
|Идентификатор сеанса имеет значение NULL. |Сущность, поддерживающая сеансы, не допускает использование сообщений, идентификатор сеанса которых имеет значение null. |
|MaxTransferHopCountExceeded | Максимальное количество допустимых прыжков при пересылке между очередями. Этот параметр имеет значение 4. |
| MaxDeliveryCountExceededExceptionMessage | Сообщение не удалось обработать после максимально возможного количества попыток доставки. Дополнительные сведения см. в разделе [Превышено значение MaxDeliveryCount](#exceeding-maxdeliverycount). |

## <a name="exceeding-maxdeliverycount"></a>Превышено значение MaxDeliveryCount

Очереди и подписки имеют свойства [QueueDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) и [SubscriptionDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.maxdeliverycount) соответственно. Значение по умолчанию — 10. Всякий раз, когда сообщение доставляется с блокировкой ([ReceiveMode.PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode)), но затем от него явно отказываются или срок действия блокировки истекает, значение счетчика [BrokeredMessage.DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) увеличивается. Когда значение [DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) превышает [MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount), сообщение перемещается в очередь DLQ с указанием кода причины `MaxDeliveryCountExceeded`.

Это поведение нельзя отключить, но можно задать для параметра [MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) большое значение.

## <a name="exceeding-timetolive"></a>Превышение значения TimeToLive

Когда свойство [QueueDescription.EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.queuedescription) или [SubscriptionDescription.EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription) имеет значение **true** (по умолчанию — **false**), все просроченные сообщения перемещаются в очередь DLQ с указанием кода причины `TTLExpiredException`.

Сообщения с истекшим сроком действия удаляются и перемещаются в очередь DLQ, когда по крайней мере один активный получатель получает из основной очереди или подписки, а [отложенные сообщения](./message-deferral.md) также не удаляются и не перемещаются в очередь недоставленных сообщений по истечении срока их действия. Эти поведения предназначены для разработки.

## <a name="errors-while-processing-subscription-rules"></a>Ошибки при обработке правил подписки

Когда для подписки активировано свойство [SubscriptionDescription.EnableDeadLetteringOnFilterEvaluationExceptions](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription), любые ошибки, возникающие при исполнении правила SQL-фильтрации подписки, фиксируются в очереди DLQ вместе с сообщением, вызвавшим ошибку. Не используйте этот параметр в рабочей среде, в которой не все типы сообщений имеют подписчиков.

## <a name="application-level-dead-lettering"></a>Недоставленные сообщения на уровне приложения

Помимо системных функций работы с недоставленными сообщениями, приложения могут использовать очередь DLQ для явного отклонения недопустимых сообщений. Сюда могут относиться сообщения, которые невозможно нормально обработать из-за каких-либо системных проблем, сообщения с полезными данными неправильного формата или сообщения, не прошедшие проверку подлинности при использовании какой-либо схемы безопасности на уровне сообщений.

## <a name="dead-lettering-in-forwardto-or-sendvia-scenarios"></a>Недоставка сообщений в сценариях ForwardTo или SendVia

Сообщения отправляются в очередь недоставленных сообщений при следующих условиях:

- Сообщение проходит более 4 очередей или разделов, которые [связаны друг с другом](service-bus-auto-forwarding.md).
- Целевая очередь (или раздел) отключена или удалена.
- Очередь или раздел назначения превышает максимальный размер сущности.

Для получения этих недоставленных сообщений можно создать получатель с помощью вспомогательного метода [FormatTransferDeadletterPath](/dotnet/api/microsoft.azure.servicebus.entitynamehelper.formattransferdeadletterpath).

## <a name="example"></a>Пример

Следующий фрагмент кода создает получатель сообщений. В цикле получения для основной очереди код получает сообщение с помощью метода [Receive(TimeSpan.Zero)](/dotnet/api/microsoft.servicebus.messaging.messagereceiver), который запрашивает у брокера мгновенный возврат любых доступных сообщений или возврат без результата. Если код получает сообщение, он немедленно отказывается от него, что ведет к увеличению `DeliveryCount`. Когда система перемещает сообщение в очередь DLQ, основная очередь оказывается пуста и осуществляется выход из цикла, так как [ReceiveAsync](/dotnet/api/microsoft.servicebus.messaging.messagereceiver) возвращает значение **null**.

```csharp
var receiver = await receiverFactory.CreateMessageReceiverAsync(queueName, ReceiveMode.PeekLock);
while(true)
{
    var msg = await receiver.ReceiveAsync(TimeSpan.Zero);
    if (msg != null)
    {
        Console.WriteLine("Picked up message; DeliveryCount {0}", msg.DeliveryCount);
        await msg.AbandonAsync();
    }
    else
    {
        break;
    }
}
```

## <a name="path-to-the-dead-letter-queue"></a>Путь к очереди недоставленных сообщений
Вы можете обратиться к очереди недоставленных сообщений с помощью следующего синтаксиса:

```
<queue path>/$deadletterqueue
<topic path>/Subscriptions/<subscription path>/$deadletterqueue
```

Если вы используете пакет SDK для .NET, можно получить путь к очереди недоставленных сообщений с помощью метода SubscriptionClient.FormatDeadLetterPath(). Этот метод принимает имя раздела, имя подписки и суффиксы для **/$DeadLetterQueue**.


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об очередях служебной шины см. в следующих статьях:

* [Начало работы с очередями служебной шины](service-bus-dotnet-get-started-with-queues.md)
* [Сравнение службы очередей Azure и службы очередей служебной шины](service-bus-azure-and-service-bus-queues-compared-contrasted.md)

