---
title: Служебная шина Azure — Просмотр сообщений
description: Просмотр и просмотр сообщений служебной шины позволяет клиенту служебной шины Azure перечислять все сообщения в очереди или подписке.
ms.topic: article
ms.date: 11/11/2020
ms.openlocfilehash: c52c9c967d4eada1a931e188ed4d25f7691cfb91
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94553647"
---
# <a name="message-browsing"></a>Просмотр сообщений

Просмотр и просмотр сообщений позволяет клиенту служебной шины перечислять все сообщения в очереди или подписке для диагностики и отладки.

Операция просмотра в очереди возвращает все сообщения в очереди, а не только те из них, которые доступны для немедленного получения с `Receive()` или `OnMessage()` цикла. Свойство `State` каждого сообщения указывает, что сообщение активно (доступно для получения), [отложено](message-deferral.md) или [запланировано](message-sequencing.md). Операция просмотра в подписке возвращает все сообщения, кроме запланированных сообщений в журнале сообщений о подписке. 

Использованные и просроченные сообщения очищаются с помощью асинхронного запуска "сборки мусора". Этот шаг может не произойти сразу после истечения срока действия сообщений. Поэтому `Peek` может возвращать сообщения с уже истекшим сроком действия. Эти сообщения будут удалены или недоставлены, когда операция получения вызывается в очереди или подписке в следующий раз. Учитывайте это поведение при попытке восстановить отложенные сообщения из очереди. Сообщение с истекшим сроком действия больше не подходит для обычного извлечения другими способами, даже если оно возвращается методом Peek. Возврат этих сообщений — это средство диагностики, отражающее текущее состояние журнала.

Peek также возвращает сообщения, которые были заблокированы и в настоящее время обрабатываются другими получателями. Однако, поскольку метод Peek возвращает отключенный моментальный снимок, состояние блокировки сообщения не может наблюдать за просматриваемыми сообщениями. Свойства [LockedUntilUtc](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.lockeduntilutc) и [LockToken](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.locktoken#Microsoft_Azure_ServiceBus_Message_SystemPropertiesCollection_LockToken) создают исключение [InvalidOperationException](/dotnet/api/system.invalidoperationexception) , когда приложение пытается прочитать их.

## <a name="peek-apis"></a>Интерфейсы API просмотра

Методы [Peek/пикасинк](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.peekasync#Microsoft_Azure_ServiceBus_Core_MessageReceiver_PeekAsync) и [PeekBatch/пикбатчасинк](/dotnet/api/microsoft.servicebus.messaging.queueclient.peekbatchasync#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatchAsync_System_Int64_System_Int32_) существуют в клиентских библиотеках .NET и Java, а также в объектах-получателях: **MessageReceiver**, **MessageSession**. Просмотр работает на очередях, подписках и соответствующих очередях недоставленных сообщений.

При повторном вызове метода **Просмотр** перечисляет все сообщения в журнале очереди или подписки по порядку от наименьшего доступного порядкового номера к высшему. Это порядок, в котором сообщения были поставлены в очередь, а не порядок, в котором сообщения могут быть получены в конечном итоге.

[PeekBatch](/dotnet/api/microsoft.servicebus.messaging.queueclient.peekbatch#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatch_System_Int32_) получает несколько сообщений и возвращает их в виде перечисления. Если доступных сообщений нет, объект перечисления является пустым, а не содержащим значения NULL.

Кроме того, можно использовать перегрузку метода с [SequenceNumber](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.sequencenumber#Microsoft_Azure_ServiceBus_Message_SystemPropertiesCollection_SequenceNumber) , с которой следует начать, а затем вызвать перегрузку метода без параметров для дальнейшего перечисления. **PeekBatch** работает аналогично, но получает набор сообщений за раз.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об обмене сообщениями через служебную шину см. в следующих статьях:

* [Очереди, разделы и подписки служебной шины](service-bus-queues-topics-subscriptions.md)
* [Начало работы с очередями служебной шины](service-bus-dotnet-get-started-with-queues.md)
* [Как использовать разделы и подписки служебной шины](service-bus-dotnet-how-to-use-topics-subscriptions.md)
