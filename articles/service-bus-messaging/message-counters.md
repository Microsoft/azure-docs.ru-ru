---
title: Служебная шина Azure — число сообщений
description: Получите количество сообщений, хранящихся в очередях и подписках, с помощью Azure Resource Manager и API-интерфейсов служебной шины Azure NamespaceManager.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: d0e1a7a5c6eb0b281b4e6ac08135f41f28ecbec8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85341279"
---
# <a name="message-counters"></a>Счетчики сообщений

Можно получить количество сообщений в очереди и подписке с помощью интерфейсов API [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) Azure Resource Manager и служебной шины, доступных в пакете SDK для .NET Framework.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

С помощью PowerShell это количество можно получить следующим образом.

```powershell
(Get-AzServiceBusQueue -ResourceGroup mygrp -NamespaceName myns -QueueName myqueue).CountDetails
```

## <a name="message-count-details"></a>Сведения о количестве сообщений

Знание числа активных сообщений помогает определить, создает ли очередь невыполненную работу, которая требует больше ресурсов для обработки, чем развернуто в настоящее время. В классе [MessageCountDetails](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails) доступны следующие данные счетчиков.

-   [ActiveMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.activemessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_ActiveMessageCount): число сообщений в очереди или подписке, которые находятся в активном состоянии и готовы к доставке.
-   [DeadLetterMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.deadlettermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_DeadLetterMessageCount): число сообщений в очереди недоставленных сообщений.
-   [ScheduledMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.scheduledmessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_ScheduledMessageCount): число запланированных сообщений.
-   [TransferDeadLetterMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.transferdeadlettermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_TransferDeadLetterMessageCount): число сообщений, которые не удалось перенести в другую очередь или раздел и которые были перемещены в очередь недоставленных сообщений для передачи.
-   [TransferMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.transfermessagecount#Microsoft_ServiceBus_Messaging_MessageCountDetails_TransferMessageCount): число сообщений, ожидающих передачи в другую очередь или раздел.

Если приложению требуется масштабировать ресурсы в зависимости от длины очереди, оно должно делать это размеренно. Получение данных счетчиков сообщений является ресурсоемкой операцией в брокере обмена сообщениями, и частое ее выполнение напрямую негативно влияет на производительность сущности.

> [!NOTE]
> Сообщения, отправляемые в раздел служебной шины, перенаправляются в подписки этого раздела. Таким образом, число активных сообщений в самом разделе равно 0, так как эти сообщения были успешно переданы в подписку. Получите количество сообщений в подписке и убедитесь, что оно больше 0. Даже если вы видите сообщения в подписке, они фактически хранятся в хранилище, принадлежащем разделу. 

Если взглянуть на подписки, то они бы имели ненулевое число сообщений (которое добавляет до 323MB пространства для всей сущности).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об обмене сообщениями через служебную шину см. в следующих статьях:

* [Очереди, разделы и подписки служебной шины](service-bus-queues-topics-subscriptions.md)
* [Начало работы с очередями служебной шины](service-bus-dotnet-get-started-with-queues.md)
* [Как использовать разделы и подписки служебной шины](service-bus-dotnet-how-to-use-topics-subscriptions.md)
