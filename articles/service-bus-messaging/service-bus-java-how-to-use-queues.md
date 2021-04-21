---
title: Использование очередей служебной шины Azure с Java (azure-messaging-servicebus)
description: В этом руководстве показано, как использовать Java для отправки и получения сообщений через очередь Служебной шины Azure. Вы используете новый пакет azure-messaging-servicebus.
ms.date: 02/13/2021
ms.topic: quickstart
ms.devlang: Java
ms.custom:
- seo-java-july2019
- seo-java-august2019
- seo-java-september2019
- devx-track-java
- mode-api
ms.openlocfilehash: e3998e812bb921ff8bea5e83199bc4e142ed2e83
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107533428"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-java"></a>Отправка сообщений в очереди Служебной шины и получение сообщений из них (Java)
В этом кратком руководстве описывается создание приложения Java для отправки сообщений в очередь служебной шины Azure и получения сообщений из нее. 

> [!IMPORTANT]
> Для работы с этим кратким руководством используется новый пакет azure-messaging-servicebus. Сведения об использовании пакета прежней версии см. в кратком руководстве по [отправке и получению сообщений с помощью azure-servicebus](service-bus-java-how-to-use-queues-legacy.md).


## <a name="prerequisites"></a>Предварительные требования
- Подписка Azure. Для работы с этим учебником требуется учетная запись Azure. Вы можете активировать [преимущества подписчика MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Если у вас нет подходящей очереди служебной шины, [создайте ее с помощью портала Azure](service-bus-quickstart-portal.md). Запишите **строку подключения** к пространству имен Служебной шины и имя созданной **очереди**.
- Установите [пакет Azure SDK для][Azure SDK for Java]Java. При использовании Eclipse можно установить [набор средств Azure для Eclipse][Azure Toolkit for Eclipse], включающий в себя пакет Azure SDK для Java. Затем можно добавить **библиотеки Microsoft Azure для Java** в проект. Если вы используете IntelliJ, см. статью [Установка набора средств Azure для IntelliJ](/azure/developer/java/toolkit-for-intellij/installation). 


## <a name="send-messages-to-a-queue"></a>Отправка сообщений в очередь
В рамках этого раздела вы создадите консольный проект Java и добавите код для отправки сообщений в созданную вами ранее очередь. 

### <a name="create-a-java-console-project"></a>Создание консольного проекта Java
Создайте проект Java с помощью Eclipse или инструмента по своему усмотрению. 

### <a name="configure-your-application-to-use-service-bus"></a>Настройка приложения для использования служебной шины
Добавьте ссылку на библиотеки Azure Core и Служебной шины Azure. 

Если вы используете Eclipse и создали консольное приложение Java, преобразуйте проект Java в Maven. Для этого щелкните правой кнопкой мыши проект в окне **обозревателя пакетов** и выберите **Настроить** -> **Преобразовать в проект Maven**. Затем добавьте зависимости в эти две библиотеки, как показано в следующем примере.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.myorg.sbusquickstarts</groupId>
    <artifactId>sbustopicqs</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <build>
        <sourceDirectory>src</sourceDirectory>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <release>15</release>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-core</artifactId>
            <version>1.13.0</version>
        </dependency>
        <dependency>
            <groupId>com.azure</groupId>
            <artifactId>azure-messaging-servicebus</artifactId>
            <version>7.0.2</version>
        </dependency>
    </dependencies>
</project>
```

### <a name="add-code-to-send-messages-to-the-queue"></a>Добавление кода для отправки сообщений в очередь
1. Добавьте в раздел Java-файла следующие инструкции `import`. 

    ```java
    import com.azure.messaging.servicebus.*;
    
    import java.util.concurrent.CountDownLatch;
    import java.util.concurrent.TimeUnit;
    import java.util.Arrays;
    import java.util.List;
    ```    
5. В классе, определите переменные для хранения строки подключения и имени очереди, как показано ниже. 

    ```java
    static String connectionString = "<NAMESPACE CONNECTION STRING>";
    static String queueName = "<QUEUE NAME>";    
    ```

    Замените `<NAMESPACE CONNECTION STRING>` строкой подключения к пространству имен Служебной шины. Затем замените `<QUEUE NAME>` именем очереди.
3. Добавьте метод с именем `sendMessage`, в класс для отправки одного сообщения в очередь. 

    ```java
    static void sendMessage()
    {
        // create a Service Bus Sender client for the queue 
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .queueName(queueName)
                .buildClient();
        
        // send one message to the queue
        senderClient.sendMessage(new ServiceBusMessage("Hello, World!"));
        System.out.println("Sent a single message to the queue: " + queueName);        
    }
    ```
1. Добавьте метод с именем `createMessages` в класс, чтобы создать список сообщений. Как правило, эти сообщения поступают из различных частей приложения. Здесь мы создадим список примеров сообщений.

    ```java
    static List<ServiceBusMessage> createMessages()
    {
        // create a list of messages and return it to the caller
        ServiceBusMessage[] messages = {
                new ServiceBusMessage("First message"),
                new ServiceBusMessage("Second message"),
                new ServiceBusMessage("Third message")
        };
        return Arrays.asList(messages);
    }
    ```
1. Добавьте метод с именем `sendMessageBatch` для отправки сообщений в созданную вами очередь. Этот метод создает `ServiceBusSenderClient` для очереди, вызывает метод `createMessages` для получения списка сообщений, подготавливает один или несколько пакетов и отправляет пакеты в очередь. 

    ```java
    static void sendMessageBatch()
    {
        // create a Service Bus Sender client for the queue 
        ServiceBusSenderClient senderClient = new ServiceBusClientBuilder()
                .connectionString(connectionString)
                .sender()
                .queueName(queueName)
                .buildClient();

        // Creates an ServiceBusMessageBatch where the ServiceBus.
        ServiceBusMessageBatch messageBatch = senderClient.createMessageBatch();        
        
        // create a list of messages
        List<ServiceBusMessage> listOfMessages = createMessages();
        
        // We try to add as many messages as a batch can fit based on the maximum size and send to Service Bus when
        // the batch can hold no more messages. Create a new batch for next set of messages and repeat until all
        // messages are sent.        
        for (ServiceBusMessage message : listOfMessages) {
            if (messageBatch.tryAddMessage(message)) {
                continue;
            }

            // The batch is full, so we create a new batch and send the batch.
            senderClient.sendMessages(messageBatch);
            System.out.println("Sent a batch of messages to the queue: " + queueName);
            
            // create a new batch
            messageBatch = senderClient.createMessageBatch();

            // Add that message that we couldn't before.
            if (!messageBatch.tryAddMessage(message)) {
                System.err.printf("Message is too large for an empty batch. Skipping. Max size: %s.", messageBatch.getMaxSizeInBytes());
            }
        }
        
        if (messageBatch.getCount() > 0) {
            senderClient.sendMessages(messageBatch);
            System.out.println("Sent a batch of messages to the queue: " + queueName);
        }

        //close the client
        senderClient.close();
    }
    ```

## <a name="receive-messages-from-a-queue"></a>Получение сообщений из очереди
В рамках этого раздела вы добавите код для получения сообщений из очереди. 

1. Добавьте метод с именем `receiveMessages` для получения сообщений из очереди. Этот метод создает `ServiceBusProcessorClient` для очереди, указывая один обработчик для обработки сообщений и другой — для обработки ошибок. Затем он запускает процессор, ждет несколько секунд, выводит полученные сообщения, а затем останавливает и закрывает процессор.

    > [!IMPORTANT]
    > Замените `QueueTest` в `QueueTest::processMessage` в коде именем класса. 

    ```java
    // handles received messages
    static void receiveMessages() throws InterruptedException
    {
        CountDownLatch countdownLatch = new CountDownLatch(1);

        // Create an instance of the processor through the ServiceBusClientBuilder
        ServiceBusProcessorClient processorClient = new ServiceBusClientBuilder()
            .connectionString(connectionString)
            .processor()
            .queueName(queueName)
            .processMessage(QueueTest::processMessage)
            .processError(context -> processError(context, countdownLatch))
            .buildProcessorClient();

        System.out.println("Starting the processor");
        processorClient.start();

        TimeUnit.SECONDS.sleep(10);
        System.out.println("Stopping and closing the processor");
        processorClient.close();        
    }   
    ```
2. Добавьте метод `processMessage` для обработки сообщения, полученного от подписки Служебной шины. 

    ```java
    private static void processMessage(ServiceBusReceivedMessageContext context) {
        ServiceBusReceivedMessage message = context.getMessage();
        System.out.printf("Processing message. Session: %s, Sequence #: %s. Contents: %s%n", message.getMessageId(),
            message.getSequenceNumber(), message.getBody());
    }    
    ```
3. Добавьте метод `processError` для обработки сообщений об ошибках.

    ```java
    private static void processError(ServiceBusErrorContext context, CountDownLatch countdownLatch) {
        System.out.printf("Error when receiving messages from namespace: '%s'. Entity: '%s'%n",
            context.getFullyQualifiedNamespace(), context.getEntityPath());

        if (!(context.getException() instanceof ServiceBusException)) {
            System.out.printf("Non-ServiceBusException occurred: %s%n", context.getException());
            return;
        }

        ServiceBusException exception = (ServiceBusException) context.getException();
        ServiceBusFailureReason reason = exception.getReason();

        if (reason == ServiceBusFailureReason.MESSAGING_ENTITY_DISABLED
            || reason == ServiceBusFailureReason.MESSAGING_ENTITY_NOT_FOUND
            || reason == ServiceBusFailureReason.UNAUTHORIZED) {
            System.out.printf("An unrecoverable error occurred. Stopping processing with reason %s: %s%n",
                reason, exception.getMessage());

            countdownLatch.countDown();
        } else if (reason == ServiceBusFailureReason.MESSAGE_LOCK_LOST) {
            System.out.printf("Message lock lost for message: %s%n", context.getException());
        } else if (reason == ServiceBusFailureReason.SERVICE_BUSY) {
            try {
                // Choosing an arbitrary amount of time to wait until trying again.
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                System.err.println("Unable to sleep for period of time");
            }
        } else {
            System.out.printf("Error source %s, reason %s, message: %s%n", context.getErrorSource(),
                reason, context.getException());
        }
    }  
    ```
2. Обновите метод `main`, чтобы вызвать методы `sendMessage`, `sendMessageBatch` и `receiveMessages` и вызвать `InterruptedException`.     

    ```java
    public static void main(String[] args) throws InterruptedException {        
        sendMessage();
        sendMessageBatch();
        receiveMessages();
    }   
    ```

## <a name="run-the-app"></a>Запустите приложение
При запуске приложения в окне консоли отображаются следующие сообщения. 

```console
Sent a single message to the queue: myqueue
Sent a batch of messages to the queue: myqueue
Starting the processor
Processing message. Session: 88d961dd801f449e9c3e0f8a5393a527, Sequence #: 1. Contents: Hello, World!
Processing message. Session: e90c8d9039ce403bbe1d0ec7038033a0, Sequence #: 2. Contents: First message
Processing message. Session: 311a216a560c47d184f9831984e6ac1d, Sequence #: 3. Contents: Second message
Processing message. Session: f9a871be07414baf9505f2c3d466c4ab, Sequence #: 4. Contents: Third message
Stopping and closing the processor
```

На странице **Обзор** для пространства имен служебной шины в портале Azure можно видеть количество **входящих** и **исходящих** сообщений. Чтобы просмотреть последние значения, возможно, потребуется подождать около минуты, а затем обновить страницу. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png" alt-text="Количество входящих и исходящих сообщений" lightbox="./media/service-bus-java-how-to-use-queues/overview-incoming-outgoing-messages.png":::

Выберите очередь на этой странице **Обзор**, чтобы перейти на страницу **Очередь служебной шины**. На этой странице отображаются количество **входящих** и **исходящих** сообщений. Вы также можете увидеть другие сведения, например **текущий размер** очереди, **максимальный размер**, **количество активных сообщений** и т. д. 

:::image type="content" source="./media/service-bus-java-how-to-use-queues/queue-details.png" alt-text="Сведения об очереди" lightbox="./media/service-bus-java-how-to-use-queues/queue-details.png":::



## <a name="next-steps"></a>Next Steps
Ознакомьтесь со следующими примерами и документацией:

- [Клиентская библиотека Служебной шины Azure для Java: файл сведений](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/servicebus/azure-messaging-servicebus/README.md)
- [Примеры на GitHub](/samples/azure/azure-sdk-for-java/servicebus-samples/)
- [Справочник по API Java](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-messaging-servicebus/7.0.0/index.html)

[Azure SDK for Java]: /azure/developer/java/sdk/get-started
[Azure Toolkit for Eclipse]: /azure/developer/java/toolkit-for-eclipse/installation
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
