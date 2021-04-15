---
title: Отправка и получение событий через Центры событий Azure с помощью Java (прежних версий)
description: В статье описано, как создать приложение Java, которое отправляет события или получает из службы "Центры событий Azure" с помощью пакета azure-eventhubs старой версии.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-java
ms.openlocfilehash: b9bbd5d4290261ec96755f02a6179a7cde267d82
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107309413"
---
# <a name="use-java-to-send-events-to-or-receive-events-from-azure-event-hubs-azure-eventhubs"></a>Использование Java для отправки и получения событий в Центрах событий Azure (azure-eventhubs)

В этом кратком руководстве показано, как отправлять события в концентратор событий и получать события из него с помощью пакета Java **azure-eventhubs**.

> [!WARNING]
> В этом руководстве используются старые пакеты **azure-eventhubs** и **azure-eventhubs-eph**. Краткое руководство, в рамках которого используется последний пакет **azure-messaging-eventhubs**, см. в статье об [отправке и получении событий с помощью azure-messaging-eventhubs](event-hubs-java-get-started-send.md). Чтобы начать использовать в приложении новый пакет вместо старого, ознакомьтесь с руководством по [переходу с azure-eventhubs на azure-messaging-eventhubs](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/migration-guide.md). 

## <a name="prerequisites"></a>Предварительные требования

Если вы впервые используете Центры событий Azure, ознакомьтесь с общими сведениями в [этой статье](event-hubs-about.md), прежде чем приступить к работе с этим руководством. 

Для работы с данным руководством необходимо следующее:

- **Подписка Microsoft Azure.** Чтобы использовать службы Azure, в том числе Центры событий Azure, потребуется действующая подписка.  Если у вас еще нет учетной записи Azure, [зарегистрируйтесь для работы с бесплатной пробной версией](https://azure.microsoft.com/free/) или [активируйте преимущества для подписчиков MSDN при создании учетной записи](https://azure.microsoft.com).
- Среда разработки Java. В рамках этого краткого руководства используется [Eclipse](https://www.eclipse.org/).
- **Создайте пространство имен Центров событий и концентратор событий**. Первым шагом является использование [портала Azure](https://portal.azure.com) для создания пространства имен типа Центров событий и получение учетных данных управления, необходимых приложению для взаимодействия с концентратором событий. Чтобы создать пространство имен и концентратор событий, выполните инструкции из [этой статьи](event-hubs-create.md). Затем получите значение ключа доступа для концентратора событий, следуя инструкциям в приведенных ниже статьях. [Получение строки подключения на портале](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Используйте ключ доступа в коде, который вы напишете далее в рамках этого краткого руководства. Имя ключа по умолчанию: **RootManageSharedAccessKey**.

## <a name="send-events"></a>Отправка событий 
Из этого раздела вы узнаете, как создать приложение Java, которое отправляет события в концентратор событий. 

> [!NOTE]
> Вы можете скачать это краткое руководство в качестве примера с сайта [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/SimpleSend), заменить строки `EventHubConnectionString` и `EventHubName` значениями для своего концентратора событий и выполнить этот пример. Или следуйте инструкциям из этого краткого руководства, чтобы создать собственное решение.

### <a name="add-reference-to-azure-event-hubs-library"></a>Добавление ссылки на библиотеку Центров событий Azure

Клиентскую библиотеку Java для Центров событий можно использовать в проектах Maven из [центрального репозитория Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22). Чтобы сослаться на эту библиотеку, используйте следующее объявление зависимостей в файле проекта Maven:

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs</artifactId>
    <version>2.2.0</version>
</dependency>
```

Для различных типов сред сборки можно явно получить последние выпущенные JAR-файлы из [центрального репозитория Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs%22).  

Если используется простой издатель событий, импортируйте пакет *com.microsoft.azure.eventhubs* для клиентских классов Центров событий и пакет *com.microsoft.azure.servicebus* для служебных классов, таких как общие исключения, которые используются совместно с клиентом обмена сообщениями служебной шины Azure. 

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Написание кода для отправки сообщений в концентратор событий

Следующий пример сначала создает новый проект Maven для приложения консоли или оболочки в избранной среде разработки Java. Добавьте класс с именем `SimpleSend` и добавьте в него следующий код.

```java
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.microsoft.azure.eventhubs.ConnectionStringBuilder;
import com.microsoft.azure.eventhubs.EventData;
import com.microsoft.azure.eventhubs.EventHubClient;
import com.microsoft.azure.eventhubs.EventHubException;

import java.io.IOException;
import java.nio.charset.Charset;
import java.time.Instant;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;

public class SimpleSend {

    public static void main(String[] args)
            throws EventHubException, ExecutionException, InterruptedException, IOException {

    }
 }
```

### <a name="construct-connection-string"></a>Создание строки подключения

Чтобы создать значение строки подключения, используйте класс ConnectionStringBuilder. Затем передайте это значение в экземпляр клиента службы "Центры событий". Замените заполнители значениями, которые получены при создании пространства имен и концентратора событий:

```java
        final ConnectionStringBuilder connStr = new ConnectionStringBuilder()
                .setNamespaceName("<EVENTHUB NAMESPACE") 
                .setEventHubName("EVENT HUB")
                .setSasKeyName("RootManageSharedAccessKey")
                .setSasKey("SHARED ACCESS KEY");
```

### <a name="write-code-to-send-events"></a>Написание кода для отправки событий

Создайте одиночное событие, преобразовав строку в байтовую кодировку UTF-8. Затем создайте экземпляр клиента Центров событий из строки подключения и отправьте сообщение.   

```java 
        final Gson gson = new GsonBuilder().create();

        // The Executor handles all asynchronous tasks and this is passed to the EventHubClient instance.
        // This enables the user to segregate their thread pool based on the work load.
        // This pool can then be shared across multiple EventHubClient instances.
        // The following sample uses a single thread executor, as there is only one EventHubClient instance,
        // handling different flavors of ingestion to Event Hubs here.
        final ScheduledExecutorService executorService = Executors.newScheduledThreadPool(4);

        // Each EventHubClient instance spins up a new TCP/TLS connection, which is expensive.
        // It is always a best practice to reuse these instances. The following sample shows this.
        final EventHubClient ehClient = EventHubClient.createSync(connStr.toString(), executorService);

        try {
            for (int i = 0; i < 10; i++) {

                String payload = "Message " + Integer.toString(i);
                byte[] payloadBytes = gson.toJson(payload).getBytes(Charset.defaultCharset());
                EventData sendEvent = EventData.create(payloadBytes);

                // Send - not tied to any partition
                // Event Hubs service will round-robin the events across all Event Hubs partitions.
                // This is the recommended & most reliable way to send to Event Hubs.
                ehClient.sendSync(sendEvent);
            }

            System.out.println(Instant.now() + ": Send Complete...");
            System.out.println("Press Enter to stop.");
            System.in.read();
        } finally {
            ehClient.closeSync();
            executorService.shutdown();
        }

``` 

Скомпилируйте и запустите программу и убедитесь в отсутствии ошибок.

Поздравляем! Теперь вы можете отправлять сообщения в концентратор событий.

### <a name="appendix-how-messages-are-routed-to-eventhub-partitions"></a>Дополнение. Способ маршрутизации сообщений к разделам концентратора событий

Прежде чем сообщения будут получены потребителями, они должны быть опубликованы издателями в разделах. При синхронной публикации сообщения в концентраторе событий с помощью метода sendSync() в объекте com.microsoft.azure.eventhubs.EventHubClient сообщение может быть отправлено в определенную секцию или распределиться по всем доступным разделам циклически в зависимости от того, указан ли ключ раздела или нет.

Когда указана строка, представляющая ключ раздела, ключ будет хэширован, чтобы определить, в какой раздел нужно отправить событие.

Если ключ раздела не задан, сообщения будут циклически отправляться во все доступные разделы.

```java
// Serialize the event into bytes
byte[] payloadBytes = gson.toJson(messagePayload).getBytes(Charset.defaultCharset());

// Use the bytes to construct an {@link EventData} object
EventData sendEvent = EventData.create(payloadBytes);

// Transmits the event to event hub without a partition key
// If a partition key is not set, then we will round-robin to all topic partitions
eventHubClient.sendSync(sendEvent);

//  the partitionKey will be hash'ed to determine the partitionId to send the eventData to.
eventHubClient.sendSync(sendEvent, partitionKey);

// close the client at the end of your program
eventHubClient.closeSync();

```

## <a name="receive-events"></a>Получение событий
Код в этом руководстве основан на [коде EventProcessorSample в GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/Java/Basic/EventProcessorSample). Изучите его, чтобы получить полное представление о рабочем приложении.

### <a name="receive-messages-with-eventprocessorhost-in-java"></a>Прием сообщений через EventProcessorHost в Java

**EventProcessorHost** — это класс Java, который упрощает прием событий от концентраторов событий путем управления постоянными контрольными точками и параллельно принимает сообщения от этих концентраторов событий. С помощью класса EventProcessorHost можно разделить события между несколькими получателями, даже если они размещены на разных узлах. В этом примере показано, как использовать EventProcessorHost для одного получателя.

### <a name="create-a-storage-account"></a>Создание учетной записи хранения

Для использования EventProcessorHost требуется [учетная запись хранения Azure][учетная запись хранения Azure]:

1. Войдите на [портал Azure](https://portal.azure.com) и щелкните **Создать ресурс** в левой части экрана.
2. Выберите **Хранилище**, а затем **Учетная запись хранения**. В окне **Создание учетной записи хранения** введите имя учетной записи хранения. Заполните остальные поля, выберите нужный регион, а затем **Создать**.

    ![Создание учетной записи хранения на портале Azure](./media/event-hubs-dotnet-framework-getstarted-receive-eph/create-azure-storage-account.png)

3. Выберите созданную учетную запись хранения, а затем **Ключи доступа**:

    ![Получение ключей доступа на портале Azure](./media/event-hubs-dotnet-framework-getstarted-receive-eph/select-azure-storage-access-keys.png)

    Скопируйте значение key1 во временное расположение. Оно будет использоваться далее в этом руководстве.

### <a name="create-a-java-project-using-the-eventprocessor-host"></a>Создание проекта Java с помощью EventProcessorHost

Клиентская библиотека Java для Центров событий доступна для использования в проектах из [центрального репозитория Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22). Ссылаться на нее можно, используя следующее объявление зависимости в файле проекта Maven: 

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs</artifactId>
    <version>2.2.0</version>
</dependency>
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-eventhubs-eph</artifactId>
    <version>2.4.0</version>
</dependency>
```

Для различных типов сред сборки можно явно получить последние выпущенные JAR-файлы из [центрального репозитория Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-eventhubs-eph%22).

1. Следующий пример сначала создает новый проект Maven для приложения консоли или оболочки в избранной среде разработки Java. Класс называется `ErrorNotificationHandler`.     

    ```java
    import java.util.function.Consumer;
    import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;

    public class ErrorNotificationHandler implements Consumer<ExceptionReceivedEventArgs>
    {
        @Override
        public void accept(ExceptionReceivedEventArgs t)
        {
            System.out.println("SAMPLE: Host " + t.getHostname() + " received general error notification during " + t.getAction() + ": " + t.getException().toString());
        }
    }
    ```
2. Создайте новый класс с именем `EventProcessorSample`с помощью следующего кода: Замените значения заполнителей значениями, которые использовались при создании концентратора событий и учетной записи хранения:

   ```java
   package com.microsoft.azure.eventhubs.samples.eventprocessorsample;

   import com.microsoft.azure.eventhubs.ConnectionStringBuilder;
   import com.microsoft.azure.eventhubs.EventData;
   import com.microsoft.azure.eventprocessorhost.CloseReason;
   import com.microsoft.azure.eventprocessorhost.EventProcessorHost;
   import com.microsoft.azure.eventprocessorhost.EventProcessorOptions;
   import com.microsoft.azure.eventprocessorhost.ExceptionReceivedEventArgs;
   import com.microsoft.azure.eventprocessorhost.IEventProcessor;
   import com.microsoft.azure.eventprocessorhost.PartitionContext;

   import java.util.concurrent.ExecutionException;
   import java.util.function.Consumer;

   public class EventProcessorSample
   {
       public static void main(String args[]) throws InterruptedException, ExecutionException
       {
           String consumerGroupName = "$Default";
           String namespaceName = "----NamespaceName----";
           String eventHubName = "----EventHubName----";
           String sasKeyName = "----SharedAccessSignatureKeyName----";
           String sasKey = "----SharedAccessSignatureKey----";
           String storageConnectionString = "----AzureStorageConnectionString----";
           String storageContainerName = "----StorageContainerName----";
           String hostNamePrefix = "----HostNamePrefix----";

           ConnectionStringBuilder eventHubConnectionString = new ConnectionStringBuilder()
                .setNamespaceName(namespaceName)
                .setEventHubName(eventHubName)
                .setSasKeyName(sasKeyName)
                .setSasKey(sasKey);

           EventProcessorHost host = new EventProcessorHost(
                EventProcessorHost.createHostName(hostNamePrefix),
                eventHubName,
                consumerGroupName,
                eventHubConnectionString.toString(),
                storageConnectionString,
                storageContainerName);

           System.out.println("Registering host named " + host.getHostName());
           EventProcessorOptions options = new EventProcessorOptions();
           options.setExceptionNotification(new ErrorNotificationHandler());

           host.registerEventProcessor(EventProcessor.class, options)
           .whenComplete((unused, e) ->
           {
               if (e != null)
               {
                   System.out.println("Failure while registering: " + e.toString());
                   if (e.getCause() != null)
                   {
                       System.out.println("Inner exception: " + e.getCause().toString());
                   }
               }
           })
           .thenAccept((unused) ->
           {
               System.out.println("Press enter to stop.");
                 try 
               {
                   System.in.read();
               }
               catch (Exception e)
               {
                   System.out.println("Keyboard read failed: " + e.toString());
               }
           })
           .thenCompose((unused) ->
            {
                return host.unregisterEventProcessor();
           })
           .exceptionally((e) ->
           {
               System.out.println("Failure while unregistering: " + e.toString());
               if (e.getCause() != null)
               {
                   System.out.println("Inner exception: " + e.getCause().toString());
               }
               return null;
           })
           .get(); // Wait for everything to finish before exiting main!

           System.out.println("End of sample");
       }
   }
   ```
3. Создайте еще один класс с именем `EventProcessor` при помощи следующего кода:

    ```java
    public static class EventProcessor implements IEventProcessor
    {
        private int checkpointBatchingCount = 0;

        // OnOpen is called when a new event processor instance is created by the host. 
        @Override
        public void onOpen(PartitionContext context) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is opening");
        }

        // OnClose is called when an event processor instance is being shut down. 
        @Override
        public void onClose(PartitionContext context, CloseReason reason) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " is closing for reason " + reason.toString());
        }

        // onError is called when an error occurs in EventProcessorHost code that is tied to this partition, such as a receiver failure.
        @Override
        public void onError(PartitionContext context, Throwable error)
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " onError: " + error.toString());
        }

        // onEvents is called when events are received on this partition of the Event Hub. 
        @Override
        public void onEvents(PartitionContext context, Iterable<EventData> events) throws Exception
        {
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " got event batch");
            int eventCount = 0;
            for (EventData data : events)
            {
                try
                {
                    System.out.println("SAMPLE (" + context.getPartitionId() + "," + data.getSystemProperties().getOffset() + "," +
                            data.getSystemProperties().getSequenceNumber() + "): " + new String(data.getBytes(), "UTF8"));
                    eventCount++;

                    // Checkpointing persists the current position in the event stream for this partition and means that the next
                    // time any host opens an event processor on this event hub+consumer group+partition combination, it will start
                    // receiving at the event after this one. 
                    this.checkpointBatchingCount++;
                    if ((checkpointBatchingCount % 5) == 0)
                    {
                        System.out.println("SAMPLE: Partition " + context.getPartitionId() + " checkpointing at " +
                               data.getSystemProperties().getOffset() + "," + data.getSystemProperties().getSequenceNumber());
                        // Checkpoints are created asynchronously. It is important to wait for the result of checkpointing
                        // before exiting onEvents or before creating the next checkpoint, to detect errors and to ensure proper ordering.
                        context.checkpoint(data).get();
                    }
                }
                catch (Exception e)
                {
                    System.out.println("Processing failed for an event: " + e.toString());
                }
            }
            System.out.println("SAMPLE: Partition " + context.getPartitionId() + " batch size was " + eventCount + " for host " + context.getOwner());
        }
    }
    ```

В данном учебнике используется один экземпляр EventProcessorHost. Чтобы увеличить пропускную способность, можно запустить несколько экземпляров EventProcessorHost, желательно на разных компьютерах.  Кроме того, такое решение обеспечит избыточность. В этом случае различные экземпляры автоматически координируются друг с другом для распределения нагрузки полученных событий. Если каждый из нескольких получателей должен обрабатывать *все* события, то необходимо использовать понятие **ConsumerGroup** . При получении события от разных компьютеров может оказаться полезным указать имена экземпляров EventProcessorHost в компьютерах (или ролях), где они развернуты.

### <a name="publishing-messages-to-eventhub"></a>Публикация сообщений в EventHub

Прежде чем сообщения будут получены потребителями, они должны быть опубликованы издателями в разделах. Следует отметить, что при синхронной публикации сообщения в концентраторе событий с помощью метода sendSync() в объекте com.microsoft.azure.eventhubs.EventHubClient сообщение может быть отправлено в определенный раздел или распределиться по всем доступным разделам циклически, в зависимости от того, указан ли ключ раздела.

Если указана строка, представляющая ключ раздела, раздел для отправки события определяется по хэшу ключа.

Если ключ раздела не задан, сообщения будут отправляются поочередно во все доступные разделы.

```java
// Serialize the event into bytes
byte[] payloadBytes = gson.toJson(messagePayload).getBytes(Charset.defaultCharset());

// Use the bytes to construct an {@link EventData} object
EventData sendEvent = EventData.create(payloadBytes);

// Transmits the event to event hub without a partition key
// If a partition key is not set, then we will round-robin to all topic partitions
eventHubClient.sendSync(sendEvent);

//  the partitionKey will be hash'ed to determine the partitionId to send the eventData to.
eventHubClient.sendSync(sendEvent, partitionKey);

```

### <a name="implementing-a-custom-checkpointmanager-for-eventprocessorhost-eph"></a>Реализация пользовательского CheckpointManager для EventProcessorHost (EPH)

API предоставляет механизм реализации пользовательского диспетчера контрольных точек для сценариев, где реализация по умолчанию несовместима с вашим вариантом использования.

Диспетчер контрольных точек по умолчанию использует хранилище BLOB-объектов, но вы можете переопределить диспетчер контрольных точек собственной реализацией и использовать любое хранилище для резервного копирования в этой реализации диспетчера контрольных точек.

Создайте класс, который реализует интерфейс com.microsoft.azure.eventprocessorhost.ICheckpointManager.

Используйте свою реализацию пользовательского диспетчера контрольных точек (com.microsoft.azure.eventprocessorhost.ICheckpointManager).

В реализации можно переопределить механизм создания контрольных точек по умолчанию и реализовать свои собственные контрольные точки на основе хранилища данных (например SQL Server, CosmosDB и кэш Azure для Redis). Мы рекомендуем, чтобы хранилище, используемое для резервного копирования в вашей реализации диспетчера контрольных точек, было доступно всем экземплярам EPH, которые обрабатывают события для группы потребителей.

Вы можете использовать любое хранилище данных, которое доступно в вашей среде.

Класс com.microsoft.azure.eventprocessorhost.EventProcessorHost предоставляет два конструктора, которые позволяют переопределить диспетчер контрольных точек для EventProcessorHost.

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими статьями: 

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Функции и терминология в Центрах событий Azure](event-hubs-features.md)
- [Часто задаваемые вопросы о Центрах событий](event-hubs-faq.yml)
