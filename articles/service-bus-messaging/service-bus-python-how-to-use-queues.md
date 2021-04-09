---
title: Использование очередей Служебной шины Azure с помощью пакета azure-servicebus версии 7.0.0 для Python
description: Из этой статьи вы узнаете, как с помощью Python отправлять сообщения в очереди Служебной шины Azure и получать сообщения из них.
author: spelluru
documentationcenter: python
ms.devlang: python
ms.topic: quickstart
ms.date: 11/18/2020
ms.author: spelluru
ms.custom: seo-python-october2019, devx-track-python
ms.openlocfilehash: 0553062032a58ec9eb9cf3c474ee7c8f19fc544d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98631561"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-python"></a>Отправка сообщений в очереди Служебной шины Azure и получение сообщений из них (Python)
Из этой статьи вы узнаете, как с помощью Python отправлять сообщения в очереди Служебной шины Azure и получать сообщения из них. 

## <a name="prerequisites"></a>Предварительные требования
- Подписка Azure. Вы можете активировать [преимущества подписчика Visual Studio или MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Если у вас нет подходящей очереди служебной шины, [создайте ее с помощью портала Azure](service-bus-quickstart-portal.md). Запишите **строку подключения** к пространству имен служебной шины и имя созданной **очереди**.
- Python 2.7 или более поздней версии с установленным пакетом [Служебной шины Azure для Python](https://pypi.python.org/pypi/azure-servicebus). Дополнительные сведения см. в [руководстве по установке Python](/azure/developer/python/azure-sdk-install). 

## <a name="send-messages-to-a-queue"></a>Отправка сообщений в очередь

1. Добавьте в файл следующие операторы импорта. 

    ```python
    from azure.servicebus import ServiceBusClient, ServiceBusMessage
    ```
2. Добавьте следующие константы. 

    ```python
    CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
    QUEUE_NAME = "<QUEUE NAME>"
    ```

    > [!IMPORTANT]
    > - Замените `<NAMESPACE CONNECTION STRING>` строкой подключения для пространства имен Служебной шины.
    > - Замените `<QUEUE NAME>` именем очереди. 
3. Добавьте метод для отправки одного сообщения.

    ```python
    def send_single_message(sender):
        # create a Service Bus message
        message = ServiceBusMessage("Single Message")
        # send the message to the queue
        sender.send_messages(message)
        print("Sent a single message")
    ```

    Отправитель — это объект, который выступает в качестве клиента для созданной очереди. Вы создадите его позже и отправите в качестве аргумента этой функции. 
4. Добавьте метод для отправки списка сообщений.

    ```python
    def send_a_list_of_messages(sender):
        # create a list of messages
        messages = [ServiceBusMessage("Message in list") for _ in range(5)]
        # send the list of messages to the queue
        sender.send_messages(messages)
        print("Sent a list of 5 messages")
    ```
5. Добавьте метод для отправки пакета сообщений.

    ```python
    def send_batch_message(sender):
        # create a batch of messages
        batch_message = sender.create_message_batch()
        for _ in range(10):
            try:
                # add a message to the batch
                batch_message.add_message(ServiceBusMessage("Message inside a ServiceBusMessageBatch"))
            except ValueError:
                # ServiceBusMessageBatch object reaches max_size.
                # New ServiceBusMessageBatch object can be created here to send more data.
                break
        # send the batch of messages to the queue
        sender.send_messages(batch_message)
        print("Sent a batch of 10 messages")
    ```
6. Создайте клиента служебной шины, а затем объект отправителя очереди для отправки сообщений.

    ```python
    # create a Service Bus client using the connection string
    servicebus_client = ServiceBusClient.from_connection_string(conn_str=CONNECTION_STR, logging_enable=True)
    with servicebus_client:
        # get a Queue Sender object to send messages to the queue
        sender = servicebus_client.get_queue_sender(queue_name=QUEUE_NAME)
        with sender:
            # send one message        
            send_single_message(sender)
            # send a list of messages
            send_a_list_of_messages(sender)
            # send a batch of messages
            send_batch_message(sender)
    
    print("Done sending messages")
    print("-----------------------")
    ```
 
## <a name="receive-messages-from-a-queue"></a>Получение сообщений из очереди
Добавьте приведенный ниже код инструкции print. Этот код будет непрерывно получать новые сообщения, пока в течение 5 (`max_wait_time`) секунд не будет получено ни одного нового сообщения. 

```python
with servicebus_client:
    # get the Queue Receiver object for the queue
    receiver = servicebus_client.get_queue_receiver(queue_name=QUEUE_NAME, max_wait_time=5)
    with receiver:
        for msg in receiver:
            print("Received: " + str(msg))
            # complete the message so that the message is removed from the queue
            receiver.complete_message(msg)
```

## <a name="full-code"></a>Полный код

```python
# import os
from azure.servicebus import ServiceBusClient, ServiceBusMessage

CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
QUEUE_NAME = "<QUEUE NAME>"

def send_single_message(sender):
    message = ServiceBusMessage("Single Message")
    sender.send_messages(message)
    print("Sent a single message")

def send_a_list_of_messages(sender):
    messages = [ServiceBusMessage("Message in list") for _ in range(5)]
    sender.send_messages(messages)
    print("Sent a list of 5 messages")

def send_batch_message(sender):
    batch_message = sender.create_message_batch()
    for _ in range(10):
        try:
            batch_message.add_message(ServiceBusMessage("Message inside a ServiceBusMessageBatch"))
        except ValueError:
            # ServiceBusMessageBatch object reaches max_size.
            # New ServiceBusMessageBatch object can be created here to send more data.
            break
    sender.send_messages(batch_message)
    print("Sent a batch of 10 messages")

servicebus_client = ServiceBusClient.from_connection_string(conn_str=CONNECTION_STR, logging_enable=True)

with servicebus_client:
    sender = servicebus_client.get_queue_sender(queue_name=QUEUE_NAME)
    with sender:
        send_single_message(sender)
        send_a_list_of_messages(sender)
        send_batch_message(sender)

print("Done sending messages")
print("-----------------------")

with servicebus_client:
    receiver = servicebus_client.get_queue_receiver(queue_name=QUEUE_NAME, max_wait_time=5)
    with receiver:
        for msg in receiver:
            print("Received: " + str(msg))
            receiver.complete_message(msg)
```

## <a name="run-the-app"></a>Запустите приложение
При запуске приложения должны отобразиться следующие выходные данные: 

```console
Sent a single message
Sent a list of 5 messages
Sent a batch of 10 messages
Done sending messages
-----------------------
Received: Single Message
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message in list
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
Received: Message inside a ServiceBusMessageBatch
```

Перейдите к пространству имен служебной шины на портале Azure. На странице **Обзор** убедитесь, что количество **входящих** и **исходящих** сообщений равно 16. Если количество не отображается, обновите страницу, подождав несколько минут. 

:::image type="content" source="./media/service-bus-python-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Количество входящих и исходящих сообщений":::

Выберите очередь на этой странице **Обзор**, чтобы перейти на страницу **Очередь служебной шины**. На этой странице также отображается количество **входящих** и **исходящих** сообщений. Вы также можете увидеть другие сведения, например **текущий размер** очереди и **количество активных сообщений**. 

:::image type="content" source="./media/service-bus-python-how-to-use-queues/queue-details.png" alt-text="Сведения об очереди":::


## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими примерами и документацией: 

- [Клиентская библиотека Служебной шины Azure для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus)
- [Примеры.](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus/samples) 
    - В папке **sync_samples** приведены примеры, демонстрирующие взаимодействие со служебной шиной в синхронном режиме. Проходя краткое руководство, вы использовали этот метод. 
    - В папке **async_samples** приведены примеры, демонстрирующие взаимодействие со служебной шиной в асинхронном режиме. 
- [Справочная документация по использованию пакета azure-servicebus](/python/api/azure-servicebus/azure.servicebus?preserve-view=true)