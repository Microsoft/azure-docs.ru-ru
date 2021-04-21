---
title: Использование разделов и подписок Служебной шины Azure с помощью пакета azure-servicebus версии 7.0.0 для Python
description: Из этой статьи вы узнаете, как с помощью Python отправлять сообщения в раздел и получать сообщения из подписки.
documentationcenter: python
author: spelluru
ms.author: spelluru
ms.date: 11/18/2020
ms.topic: quickstart
ms.devlang: python
ms.custom:
- devx-track-python
- mode-api
ms.openlocfilehash: 49e80e277c6df5372341293861d5bda0580f3e8c
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107537167"
---
# <a name="send-messages-to-an-azure-service-bus-topic-and-receive-messages-from-subscriptions-to-the-topic-python"></a>Отправка сообщений в раздел Служебной шины Azure и прием сообщений из подписок в разделе (Python)
Из этой статьи вы узнаете, как с помощью Python отправлять сообщения в раздел служебной шины и получать сообщения из подписки в разделе. 

## <a name="prerequisites"></a>Предварительные требования
- Подписка Azure. Вы можете активировать [преимущества подписчика Visual Studio или MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Выполните шаги из [краткого руководства по созданию раздела Служебной шины и подписок на него с помощью портала Azure](service-bus-quickstart-topics-subscriptions-portal.md). Запишите строку подключения, имя раздела и имя подписки. При работе с этим кратким руководством вы будете использовать только одну подписку. 
- Python 2.7 или более поздней версии с установленным пакетом [Azure SDK для Python] [пакет Azure для Python]. Дополнительные сведения см. в [руководстве по установке Python](/azure/developer/python/azure-sdk-install).

## <a name="send-messages-to-a-topic"></a>Отправка сообщений в раздел

1. Добавьте в файл следующие операторы импорта. 

    ```python
    from azure.servicebus import ServiceBusClient, ServiceBusMessage
    ```
2. Добавьте следующие константы. 

    ```python
    CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
    TOPIC_NAME = "<TOPIC NAME>"
    SUBSCRIPTION_NAME = "<SUBSCRIPTION NAME>"
    ```
    
    > [!IMPORTANT]
    > - Замените `<NAMESPACE CONNECTION STRING>` строками подключения для вашего пространства имен.
    > - Затем замените `<TOPIC NAME>` именем раздела.
    > - Замените `<SUBSCRIPTION NAME>` именем подписки на раздел. 
3. Добавьте метод для отправки одного сообщения.

    ```python
    def send_single_message(sender):
        # create a Service Bus message
        message = ServiceBusMessage("Single Message")
        # send the message to the topic
        sender.send_messages(message)
        print("Sent a single message")
    ```

    Отправитель — это объект, который выступает в качестве клиента для созданного раздела. Вы создадите его позже и отправите в качестве аргумента этой функции. 
4. Добавьте метод для отправки списка сообщений.

    ```python
    def send_a_list_of_messages(sender):
        # create a list of messages
        messages = [ServiceBusMessage("Message in list") for _ in range(5)]
        # send the list of messages to the topic
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
        # send the batch of messages to the topic
        sender.send_messages(batch_message)
        print("Sent a batch of 10 messages")
    ```
6. Создайте клиента служебной шины, а затем объект отправителя раздела для отправки сообщений.

    ```python
    # create a Service Bus client using the connection string
    servicebus_client = ServiceBusClient.from_connection_string(conn_str=CONNECTION_STR, logging_enable=True)
    with servicebus_client:
        # get a Topic Sender object to send messages to the topic
        sender = servicebus_client.get_topic_sender(topic_name=TOPIC_NAME)
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
 
## <a name="receive-messages-from-a-subscription"></a>Получение сообщений из подписки
Добавьте приведенный ниже код инструкции print. Этот код будет непрерывно получать новые сообщения, пока в течение 5 (`max_wait_time`) секунд не будет получено ни одного нового сообщения. 

```python
with servicebus_client:
    # get the Subscription Receiver object for the subscription    
    receiver = servicebus_client.get_subscription_receiver(topic_name=TOPIC_NAME, subscription_name=SUBSCRIPTION_NAME, max_wait_time=5)
    with receiver:
        for msg in receiver:
            print("Received: " + str(msg))
            # complete the message so that the message is removed from the subscription
            receiver.complete_message(msg)
```

## <a name="full-code"></a>Полный код

```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage

CONNECTION_STR = "<NAMESPACE CONNECTION STRING>"
TOPIC_NAME = "<TOPIC NAME>"
SUBSCRIPTION_NAME = "<SUBSCRIPTION NAME>"

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
    sender = servicebus_client.get_topic_sender(topic_name=TOPIC_NAME)
    with sender:
        send_single_message(sender)
        send_a_list_of_messages(sender)
        send_batch_message(sender)

print("Done sending messages")
print("-----------------------")

with servicebus_client:
    receiver = servicebus_client.get_subscription_receiver(topic_name=TOPIC_NAME, subscription_name=SUBSCRIPTION_NAME, max_wait_time=5)
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

Выберите раздел в нижней области, чтобы просмотреть страницу **Раздел служебной шины** своего раздела. На этой странице вы увидите три входящих и три исходящих сообщения в диаграмме **Сообщения**. 

:::image type="content" source="./media/service-bus-python-how-to-use-topics-subscriptions/topic-page-portal.png" alt-text="Экран &quot;Входящие и исходящие сообщения&quot;":::

При выборе подписки на этой странице вы будете перенаправлены на страницу **Service Bus Subscription** (Подписка служебной шины). Также на этой странице можно увидеть количество активных и недоставленных сообщений и многое другое. В этом примере получены все сообщения, поэтому количество активных сообщений равно нулю. 

:::image type="content" source="./media/service-bus-python-how-to-use-topics-subscriptions/active-message-count.png" alt-text="Экран &quot;Количество активных сообщений&quot;":::

Если закомментировать код получения, вы увидите, что количество активных сообщений будет равно 16. 

:::image type="content" source="./media/service-bus-python-how-to-use-topics-subscriptions/active-message-count-2.png" alt-text="Экран &quot;Количество не полученных активных сообщений&quot;":::

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими примерами и документацией: 

- [Клиентская библиотека Служебной шины Azure для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus)
- [Примеры.](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/servicebus/azure-servicebus/samples) 
    - В папке **sync_samples** приведены примеры, демонстрирующие взаимодействие со служебной шиной в синхронном режиме. Проходя краткое руководство, вы использовали этот метод. 
    - В папке **async_samples** приведены примеры, демонстрирующие взаимодействие со служебной шиной в асинхронном режиме. 
- [Справочная документация по использованию пакета azure-servicebus](/python/api/azure-servicebus/azure.servicebus?preserve-view=true)
