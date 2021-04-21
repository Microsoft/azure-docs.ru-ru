---
title: включить файл
description: включить файл
services: service-bus-messaging
author: axisc
ms.service: service-bus-messaging
ms.topic: include
ms.date: 6/9/2020
ms.author: aschhab
ms.custom: include file
ms.openlocfilehash: 574507fcc6a3c05919c441bd6d0ec9c573d4b6ae
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "100652616"
---
В следующей таблице перечислены функции службы сообщений Java (JMS), поддерживаемые в настоящее время служебной шиной Azure. В ней также указаны неподдерживаемые функции.


| Компонент | API |Состояние |
|---|---|---|
| Очереди   | <ul> <li> JMSContext.createQueue( строка queueName) </li> </ul>| **Поддерживается** |
| Разделы   | <ul> <li> JMSContext.createTopic( строка topicName) </li> </ul>| **Поддерживается** |
| Временные очереди |<ul> <li> JMSContext.createTemporaryQueue() </li> </ul>| **Поддерживается** |
| Временные разделы |<ul> <li> JMSContext.createTemporaryTopic() </li> </ul>| **Поддерживается** |
| Создатель сообщений /<br/> JMSProducer |<ul> <li> JMSContext.createProducer() </li> </ul>| **Поддерживается** |
| Браузеры очередей |<ul> <li> JMSContext.createBrowser(Queue queue) </li> <li> JMSContext.createBrowser(Queue queue, строка messageSelector) </li> </ul> | **Поддерживается** |
| Получатель сообщений/ <br/> JMSConsumer | <ul> <li> JMSContext.createConsumer( Destination destination) </li> <li> JMSContext.createConsumer( Destination destination, строка messageSelector) </li> <li> JMSContext.createConsumer( Destination destination, строка messageSelector, логическая величина noLocal)</li> </ul>  <br/> noLocal в настоящее время не поддерживается | **Поддерживается** |
| Общие устойчивые подписки | <ul> <li> JMSContext.createSharedDurableConsumer(Topic topic, строка name) </li> <li> JMSContext.createSharedDurableConsumer(Topic topic, строка name, строка messageSelector) </li> </ul>| **Поддерживается**|
| Необщие устойчивые подписки | <ul> <li> JMSContext.createDurableConsumer(Topic topic, строка name) </li> <li> createDurableConsumer(Topic topic, строка name, строка messageSelector, логическая величина noLocal) </li> </ul> <br/> noLocal в настоящее время не поддерживается, для этой величины следует задать значение "false" | **Поддерживается** |
| Общие неустойчивые подписки |<ul> <li> JMSContext.createSharedConsumer(Topic topic, строка sharedSubscriptionName) </li> <li> JMSContext.createSharedConsumer(Topic topic, строка sharedSubscriptionName, строка messageSelector) </li> </ul> | **Поддерживается** |
| Необщие неустойчивые подписки |<ul> <li> JMSContext.createConsumer(Destination destination) </li> <li> JMSContext.createConsumer( Destination destination, строка messageSelector) </li> <li> JMSContext.createConsumer( Destination destination, строка messageSelector, логическая величина noLocal) </li> </ul> <br/> noLocal в настоящее время не поддерживается, для этой величины следует задать значение "false" | **Поддерживается** |
| Селекторы сообщений | зависит от созданного получателя | **Поддерживается** |
| Delivery Delay (запланированные сообщения) | <ul> <li> JMSProducer.setDeliveryDelay( long deliveryDelay) </li> </ul>|**Поддерживается**|
| Создание сообщения |<ul> <li> JMSContext.createMessage() </li> <li> JMSContext.createBytesMessage() </li> <li> JMSContext.createMapMessage() </li> <li> JMSContext.createObjectMessage( Serializable object) </li> <li> JMSContext.createStreamMessage() </li> <li> JMSContext.createTextMessage() </li> <li> JMSContext.createTextMessage( строка text) </li> </ul>| **Поддерживается** |
| Транзакции между объектами |<ul> <li> Connection.createSession(true, Session.SESSION_TRANSACTED) </li> </ul> | **Поддерживается** |
| Распределенные транзакции || Не поддерживается |
