---
title: Отправка и получение событий через Центры событий Azure с помощью Java (последних версий)
description: В статье описано, как создать приложение Java, которое отправляет события или получает их из службы "Центры событий Azure" с помощью пакета сообщений azure-eventhubs последней версии.
ms.topic: quickstart
ms.date: 04/05/2021
ms.custom: devx-track-java
ms.openlocfilehash: bae0d4147fddf57398d494ca7f13c347f892e45e
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106448446"
---
# <a name="use-java-to-send-events-to-or-receive-events-from-azure-event-hubs-azure-messaging-eventhubs"></a>Использование Java для отправки и получения событий в Центрах событий Azure (azure-messaging-eventhubs)
В этом кратком руководстве показано, как отправлять события в концентратор событий и получать события из него с помощью пакета Java **azure-messaging-eventhubs**.

> [!IMPORTANT]
> В этом кратком руководстве используется новый пакет **azure-messaging-eventhubs**. Сведения о том, как отправлять события в концентратор событий и получать события из него с помощью старых пакетов **azure-eventhubs** и **azure-eventhubs-eph**, см. в [этом кратком руководстве](event-hubs-java-get-started-send-legacy.md). 


## <a name="prerequisites"></a>Предварительные требования
Если вы впервые используете Центры событий Azure, ознакомьтесь с [общими сведениями](event-hubs-about.md), прежде чем приступить к работе с этим руководством. 

Для работы с данным руководством необходимо следующее:

- **Подписка Microsoft Azure.** Чтобы использовать службы Azure, в том числе Центры событий Azure, потребуется действующая подписка.  Если у вас еще нет учетной записи Azure, [зарегистрируйтесь для работы с бесплатной пробной версией](https://azure.microsoft.com/free/) или [активируйте преимущества для подписчиков MSDN при создании учетной записи](https://azure.microsoft.com).
- Среда разработки Java. В рамках этого краткого руководства используется [Eclipse](https://www.eclipse.org/). Необходим комплект SDK Java (JDK) версии 8 или более поздней. 
- **Создайте пространство имен Центров событий и концентратор событий**. Первым шагом является использование [портала Azure](https://portal.azure.com) для создания пространства имен типа Центров событий и получение учетных данных управления, необходимых приложению для взаимодействия с концентратором событий. Чтобы создать пространство имен и концентратор событий, выполните инструкции из [этой статьи](event-hubs-create.md). Получите **строку подключения для пространства имен Центров событий**, следуя инструкциям из статьи [Получение строки подключения на портале](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Строка подключения понадобится нам позже в рамках этого краткого руководства.

## <a name="send-events"></a>Отправка событий 
Из этого раздела вы узнаете, как создать приложение Java, которое отправляет события в концентратор событий. 

### <a name="add-reference-to-azure-event-hubs-library"></a>Добавление ссылки на библиотеку Центров событий Azure

Клиентская библиотека Java для Центров событий доступна в [центральном репозитории Maven](https://search.maven.org/search?q=a:azure-messaging-eventhubs). Чтобы сослаться на эту библиотеку, используйте следующее объявление зависимостей в файле проекта Maven:

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-eventhubs</artifactId>
    <version>5.6.0</version>
</dependency>
```

> [!NOTE]
> Обновите версию до последней, опубликованной в репозитории Maven. 

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Написание кода для отправки сообщений в концентратор событий

Следующий пример сначала создает новый проект Maven для приложения консоли или оболочки в избранной среде разработки Java. Добавьте класс с именем `Sender` и добавьте в него следующий код.

> [!IMPORTANT]
> Замените `<Event Hubs namespace connection string>` строками подключения для вашего пространства имен Центров событий. Замените `<Event hub name>` именем пространства имен для концентратора событий. 

```java
import com.azure.messaging.eventhubs.*;
import java.util.Arrays;
import java.util.List;

public class Sender {
    private static final String connectionString = "<Event Hubs namespace connection string>";
    private static final String eventHubName = "<Event hub name>";

    public static void main(String[] args) {
        publishEvents();
    }
}
```
### <a name="add-code-to-publish-events-to-the-event-hub"></a>Добавление кода для публикации событий в концентраторе событий
Добавьте новый метод с именем `publishEvents` в класс `Sender`, как показано далее: 

```java
    /**
     * Code sample for publishing events.
     * @throws IllegalArgumentException if the event data is bigger than max batch size.
     */
    public static void publishEvents() {
        // create a producer client
        EventHubProducerClient producer = new EventHubClientBuilder()
            .connectionString(connectionString, eventHubName)
            .buildProducerClient();

        // sample events in an array
        List<EventData> allEvents = Arrays.asList(new EventData("Foo"), new EventData("Bar"));
        
        // create a batch
        EventDataBatch eventDataBatch = producer.createBatch();

        for (EventData eventData : allEvents) {
            // try to add the event from the array to the batch
            if (!eventDataBatch.tryAdd(eventData)) {
                // if the batch is full, send it and then create a new batch
                producer.send(eventDataBatch);
                eventDataBatch = producer.createBatch();

                // Try to add that event that couldn't fit before.
                if (!eventDataBatch.tryAdd(eventData)) {
                    throw new IllegalArgumentException("Event is too large for an empty batch. Max size: "
                        + eventDataBatch.getMaxSizeInBytes());
                }
            }
        }
        // send the last batch of remaining events
        if (eventDataBatch.getCount() > 0) {
            producer.send(eventDataBatch);
        }
        producer.close();
    }
```

Скомпилируйте программу и убедитесь в отсутствии ошибок. Эту программу вы запустите после запуска программы получателя. 

## <a name="receive-events"></a>Получение событий
Код в этом руководстве основан на [шаблоне EventProcessorClient в GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorBlobCheckpointStoreSample.java). Изучите его, чтобы получить полное представление о рабочем приложении.

> [!WARNING]
> Запустив этот код в Azure Stack Hub, вы получите ошибки выполнения, если не укажете конкретную версию API службы хранилища. Это связано с тем, что пакет SDK для Центров событий использует последний доступный в Azure API службы хранилища Azure, который может быть недоступным на вашей платформе Azure Stack Hub. Azure Stack Hub может поддерживать версию пакета SDK для хранилища BLOB-объектов, отличающуюся от доступных в Azure. Если вы используете Хранилище BLOB-объектов Azure в качестве хранилища контрольных точек, проверьте [поддерживаемую версию API службы хранилища Azure для сборки Azure Stack Hub](/azure-stack/user/azure-stack-acs-differences?#api-version) и включите эту версию в код. 
>
> Например, если вы используете Azure Stack Hub версии 2005, последней доступной версией для службы хранилища будет 2019-02-02. По умолчанию клиентская библиотека пакета SDK для Центров событий использует последнюю доступную версию в Azure (2019-07-07 на момент выпуска пакета SDK). В таком случае кроме действий, описанных в этом разделе, вам нужно добавить код для указания API службы хранилища версии 2019-02-02. Пример нацеливания на определенную версию API службы хранилища см. [на сайте GitHub](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/EventProcessorWithCustomStorageVersion.java). 


### <a name="create-an-azure-storage-and-a-blob-container"></a>Создание службы хранилища Azure и контейнера больших двоичных объектов
В этом кратком руководстве показано, как настроить службу хранилища Azure (Хранилище BLOB-объектов) в качестве хранилища контрольных точек. Контрольные точки позволяют обработчику событий помечать или фиксировать расположение последнего успешно обработанного события в секции. Контрольная точка обычно отмечается в функции, которая обрабатывает события. См. статью [Балансировка нагрузки секций между несколькими экземплярами приложения](event-processor-balance-partition-load.md).

Выполните указанные ниже действия, чтобы создать учетную запись хранения Azure. 

1. [Создайте учетную запись хранения Azure](../storage/common/storage-account-create.md?tabs=azure-portal)
2. [Создание контейнера больших двоичных объектов](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)
3. [Получение строки подключения к учетной записи хранения](../storage/common/storage-configure-connection-string.md).

    Запишите **имя контейнера** и **строку подключения**. Их вы примените далее в коде получения данных. 

### <a name="add-event-hubs-libraries-to-your-java-project"></a>Добавление библиотек Центров событий в проект Java
Добавьте следующие зависимости в файл pom.xml. 

- [azure-messaging-eventhubs](https://search.maven.org/search?q=a:azure-messaging-eventhubs)
- [azure-messaging-eventhubs-checkpointstore-blob](https://search.maven.org/search?q=a:azure-messaging-eventhubs-checkpointstore-blob)

```xml
<dependencies>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-eventhubs</artifactId>
        <version>5.6.0</version>
    </dependency>
    <dependency>
        <groupId>com.azure</groupId>
        <artifactId>azure-messaging-eventhubs-checkpointstore-blob</artifactId>
        <version>1.5.1</version>
    </dependency>
</dependencies>
```

1. Добавьте в начало файла Java следующие инструкции **import**. 

    ```java
    import com.azure.messaging.eventhubs.EventHubClientBuilder;
    import com.azure.messaging.eventhubs.EventProcessorClient;
    import com.azure.messaging.eventhubs.EventProcessorClientBuilder;
    import com.azure.messaging.eventhubs.checkpointstore.blob.BlobCheckpointStore;
    import com.azure.messaging.eventhubs.models.ErrorContext;
    import com.azure.messaging.eventhubs.models.EventContext;
    import com.azure.storage.blob.BlobContainerAsyncClient;
    import com.azure.storage.blob.BlobContainerClientBuilder;
    import java.util.function.Consumer;
    ```
2. Создайте класс с именем `Receiver` и добавьте в него следующие строковые переменные. Замените значения заполнителей своими значениями. 

    > [!IMPORTANT]
    > Замените значения заполнителей своими значениями.
    > - Замените `<Event Hubs namespace connection string>` строками подключения для вашего пространства имен Центров событий. Update 
    > - Замените `<Event hub name>` именем пространства имен для концентратора событий. 
    > - Замените `<Storage connection string>` строками подключения для вашей учетной записи хранения Azure. 
    > - Замените `<Storage container name>` именем контейнера в хранилище больших двоичных объектов Azure. 

    ```java
    private static final String connectionString = "<Event Hubs namespace connection string>";
    private static final String eventHubName = "<Event hub name>";
    private static final String storageConnectionString = "<Storage connection string>";
    private static final String storageContainerName = "<Storage container name>";
    ```
1. Добавьте в класс следующий метод `main`. 

    ```java
    public static void main(String[] args) throws Exception {
        // Create a blob container client that you use later to build an event processor client to receive and process events
        BlobContainerAsyncClient blobContainerAsyncClient = new BlobContainerClientBuilder()
            .connectionString(storageConnectionString) 
            .containerName(storageContainerName) 
            .buildAsyncClient();
    
        // Create a builder object that you will use later to build an event processor client to receive and process events and errors.
        EventProcessorClientBuilder eventProcessorClientBuilder = new EventProcessorClientBuilder()
            .connectionString(connectionString, eventHubName) 
            .consumerGroup(EventHubClientBuilder.DEFAULT_CONSUMER_GROUP_NAME)
            .processEvent(PARTITION_PROCESSOR) 
            .processError(ERROR_HANDLER) 
            .checkpointStore(new BlobCheckpointStore(blobContainerAsyncClient)); 

        // Use the builder object to create an event processor client 
        EventProcessorClient eventProcessorClient = eventProcessorClientBuilder.buildEventProcessorClient();

        System.out.println("Starting event processor");
        eventProcessorClient.start();

        System.out.println("Press enter to stop.");
        System.in.read();

        System.out.println("Stopping event processor");
        eventProcessorClient.stop();
        System.out.println("Event processor stopped.");

        System.out.println("Exiting process");
    }    
    ```
4. Добавьте два вспомогательных метода (`PARTITION_PROCESSOR` и `ERROR_HANDLER`), которые обрабатывают события и ошибки в классе `Receiver`. 

    ```java
    public static final Consumer<EventContext> PARTITION_PROCESSOR = eventContext -> {
        System.out.printf("Processing event from partition %s with sequence number %d with body: %s %n", 
                eventContext.getPartitionContext().getPartitionId(), eventContext.getEventData().getSequenceNumber(), eventContext.getEventData().getBodyAsString());

        if (eventContext.getEventData().getSequenceNumber() % 10 == 0) {
            eventContext.updateCheckpoint();
        }
    };
    
    public static final Consumer<ErrorContext> ERROR_HANDLER = errorContext -> {
        System.out.printf("Error occurred in partition processor for partition %s, %s.%n",
            errorContext.getPartitionContext().getPartitionId(),
            errorContext.getThrowable());
    };    
    ```
3. Скомпилируйте программу и убедитесь в отсутствии ошибок. 

## <a name="run-the-applications"></a>Запуск приложений
1. Сначала запустите приложение **получателя**.
1. Затем запустите приложение **отправителя**. 
1. В окне **получатель** приложения убедитесь, что отображаются события, опубликованные приложением отправителя.

    ```cmd
    Starting event processor
    Press enter to stop.
    Processing event from partition 0 with sequence number 331 with body: Foo 
    Processing event from partition 0 with sequence number 332 with body: Bar 
    ```
1. Нажмите **ENTER** в окне приложения получателя, чтобы прервать работу приложения. 

    ```cmd
    Starting event processor
    Press enter to stop.
    Processing event from partition 0 with sequence number 331 with body: Foo 
    Processing event from partition 0 with sequence number 332 with body: Bar 
    
    Stopping event processor
    Event processor stopped.
    Exiting process
    ```

## <a name="next-steps"></a>Дальнейшие действия
См. следующие примеры на сайте GitHub:

- [Примеры azure-messaging-eventhubs](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples/java/com/azure/messaging/eventhubs)
- [Примеры azure-messaging-eventhubs-checkpointstore-blob](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob).  
