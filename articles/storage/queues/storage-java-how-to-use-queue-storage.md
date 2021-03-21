---
title: Использование хранилища очередей из Java в службе хранилища Azure
description: Узнайте, как использовать хранилище очередей для создания и удаления очередей. Узнайте, как вставлять, просматривать, получать и удалять сообщения с помощью клиентской библиотеки службы хранилища Azure для Java.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 08/19/2020
ms.topic: how-to
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-java
ms.openlocfilehash: 997a37ac4252813abf1b35877cd34e192ec3e2ae
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97585723"
---
# <a name="how-to-use-queue-storage-from-java"></a>Использование хранилища очередей из Java

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

## <a name="overview"></a>Обзор

В этом руководство показано, как выполнять код для распространенных сценариев с помощью службы хранилища очередей Azure. Примеры написаны на Java и используют [пакет SDK службы хранилища Azure для Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage). К таким сценариям относятся **Вставка**, **Просмотр**, **Получение** и **Удаление** сообщений очереди. Также рассматривается код для **создания** и **удаления** очередей. Дополнительные сведения об очередях см. в разделе [Дальнейшие действия](#next-steps).

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-java-application"></a>Создание приложения Java

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Сначала убедитесь, что ваша система разработки соответствует предварительным требованиям, перечисленным в [клиентской библиотеке хранилища очередей Azure версии 12 для Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-queue).

Чтобы создать приложение Java с именем `queues-how-to-v12` :

1. В окне консоли (командная строка, PowerShell или Bash) с помощью Maven создайте консольное приложение с именем `queues-how-to-v12`. Введите следующую команду `mvn`, чтобы создать проект Java hello world.

   ```bash
    mvn archetype:generate \
        --define interactiveMode=n \
        --define groupId=com.queues.howto \
        --define artifactId=queues-howto-v12 \
        --define archetypeArtifactId=maven-archetype-quickstart \
        --define archetypeVersion=1.4
   ```

   ```powershell
    mvn archetype:generate `
        --define interactiveMode=n `
        --define groupId=com.queues.howto `
        --define artifactId=queues-howto-v12 `
        --define archetypeArtifactId=maven-archetype-quickstart `
        --define archetypeVersion=1.4
    ```

1. Примерный результат создания проекта показан ниже.

    ```console
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------< org.apache.maven:standalone-pom >-------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] --------------------------------[ pom ]---------------------------------
    [INFO]
    [INFO] >>> maven-archetype-plugin:3.1.2:generate (default-cli) > generate-sources @ standalone-pom >>>
    [INFO]
    [INFO] <<< maven-archetype-plugin:3.1.2:generate (default-cli) < generate-sources @ standalone-pom <<<
    [INFO]
    [INFO]
    [INFO] --- maven-archetype-plugin:3.1.2:generate (default-cli) @ standalone-pom ---
    [INFO] Generating project in Batch mode
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: maven-archetype-quickstart:1.4
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.queues.howto
    [INFO] Parameter: artifactId, Value: queues-howto-v12
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.queues.howto
    [INFO] Parameter: packageInPathFormat, Value: com/queues/howto
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.queues.howto
    [INFO] Parameter: groupId, Value: com.queues.howto
    [INFO] Parameter: artifactId, Value: queues-howto-v12
    [INFO] Project created from Archetype in dir: C:\queues\queues-howto-v12
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  6.775 s
    [INFO] Finished at: 2020-08-17T15:27:31-07:00
    [INFO] ------------------------------------------------------------------------
        ```

1. Switch to the newly created `queues-howto-v12` directory.

   ```console
   cd queues-howto-v12
   ```

### <a name="install-the-package"></a>Установка пакета

Откройте файл `pom.xml` в текстовом редакторе. Добавьте приведенный ниже элемент зависимости в группу зависимостей.

```xml
<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-storage-queue</artifactId>
  <version>12.6.0</version>
</dependency>
```

# <a name="java-v8"></a>[V8 Java](#tab/java8)

Сначала убедитесь, что система разработки соответствует требованиям, перечисленным в [пакете SDK службы хранилища Azure для Java V8](https://github.com/azure/azure-storage-java). Следуйте инструкциям по скачиванию и установке библиотек службы хранилища Azure для Java. Затем можно создать приложение Java, используя примеры, приведенные в этой статье.

---

## <a name="configure-your-application-to-access-queue-storage"></a>Настройка приложения для доступа к хранилищу очередей

Добавьте следующие операторы импорта в начало файла Java, где вы хотите использовать API-интерфейсы службы хранилища Azure для доступа к очередям:

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_ImportStatements":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

```java
// Include the following imports to use queue APIs.
import com.microsoft.azure.storage.*;
import com.microsoft.azure.storage.queue.*;
```

---

## <a name="set-up-an-azure-storage-connection-string"></a>Настройка строки подключения к службе хранилища Azure

Клиент службы хранилища Azure использует строку подключения к хранилищу для доступа к службам управления данными. Получите имя и первичный ключ доступа для учетной записи хранения, указанной в [портал Azure](https://portal.azure.com). Используйте их в качестве `AccountName` `AccountKey` значений и в строке подключения. В этом примере показано, как объявить статическое поле для размещения строки подключения:

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_ConnectionString":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

```java
// Define the connection-string with your values.
final String storageConnectionString =
    "DefaultEndpointsProtocol=https;" +
    "AccountName=your_storage_account;" +
    "AccountKey=your_storage_account_key";
```

Эту строку можно сохранить в файле конфигурации службы с именем `ServiceConfiguration.cscfg` . Для приложения, выполняющегося в роли Microsoft Azure, необходимо получить доступ к строке подключения, вызвав метод `RoleEnvironment.getConfigurationSettings` . Ниже приведен пример получения строки подключения из `Setting` элемента с именем `StorageConnectionString` .

```java
// Retrieve storage account from connection-string.
String storageConnectionString =
    RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");
```

---

В следующих примерах предполагается, что у вас есть `String` объект, содержащий строку подключения к хранилищу.

## <a name="how-to-create-a-queue"></a>Практическое руководство. Создание очереди

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

`QueueClient`Объект содержит операции для взаимодействия с очередью. Следующий код создает `QueueClient` объект. Используйте `QueueClient` объект для создания очереди, которую вы хотите использовать.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_CreateQueue":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

`CloudQueueClient`Объект позволяет получать ссылочные объекты для очередей. Следующий код создает `CloudQueueClient` объект, предоставляющий ссылку на очередь, которую необходимо использовать. Очередь можно создать, если она не существует.

> [!NOTE]
> Существуют другие способы создания `CloudStorageAccount` объектов. Дополнительные сведения см `CloudStorageAccount` . в [справочнике по пакету SDK для клиента службы хранилища Azure](https://azure.github.io/azure-sdk-for-java/storage.html).

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

   // Create the queue client.
   CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

   // Retrieve a reference to a queue.
   CloudQueue queue = queueClient.getQueueReference("myqueue");

   // Create the queue if it doesn't already exist.
   queue.createIfNotExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-add-a-message-to-a-queue"></a>Практическое руководство. Добавление сообщения в очередь

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Чтобы вставить сообщение в существующую очередь, вызовите `sendMessage` метод. Сообщение может быть либо строкой (в формате UTF-8), либо массивом байтов. Ниже приведен код, который отправляет строковое сообщение в очередь.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_AddMessage":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

Чтобы вставить сообщение в существующую очередь, сначала создайте новый объект `CloudQueueMessage` . Затем вызовите `addMessage` метод. `CloudQueueMessage`Можно создать либо из строки (в формате UTF-8), либо в массиве байтов. Ниже приведен код, создающий очередь (если она не существует) и вставляет сообщение `Hello, World` .

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Create the queue if it doesn't already exist.
    queue.createIfNotExists();

    // Create a message and add it to the queue.
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    queue.addMessage(message);
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-peek-at-the-next-message"></a>Практическое руководство. Просмотр следующего сообщения

Вы можете просматривать сообщение в начале очереди, не удаляя его из очереди, вызывая `peekMessage` .

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_PeekMessage":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Peek at the next message.
    CloudQueueMessage peekedMessage = queue.peekMessage();

    // Output the message value.
    if (peekedMessage != null)
    {
      System.out.println(peekedMessage.getMessageContentAsString());
   }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Практическое руководство. Изменение содержимого сообщения в очереди

Вы можете изменить содержимое сообщения непосредственно в очереди. Если сообщение представляет собой рабочую задачу, можно использовать эту функцию для обновления состояния. Следующий код обновляет сообщение очереди с новым содержимым и устанавливает время ожидания видимости для продления еще 30 секунд. Расширение времени ожидания видимости позволяет клиенту продолжать работу с сообщением через 30 секунд. Также можно сократить число повторных попыток. Если сообщение повторяется более *n* раз, его необходимо удалить. Этот сценарий защищает от сообщения, которое вызывает ошибку приложения при каждой обработке.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Следующий пример кода выполняет поиск в очереди сообщений, находит первое содержимое сообщения, соответствующее строке поиска, изменяет содержимое сообщения и завершает работу.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_UpdateSearchMessage":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

Следующий пример кода выполняет поиск в очереди сообщений, находит первое содержимое сообщения, которое соответствует `Hello, world` , изменяет содержимое сообщения и завершает работу.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // The maximum number of messages that can be retrieved is 32.
    final int MAX_NUMBER_OF_MESSAGES_TO_PEEK = 32;

    // Loop through the messages in the queue.
    for (CloudQueueMessage message : queue.retrieveMessages(MAX_NUMBER_OF_MESSAGES_TO_PEEK,1,null,null))
    {
        // Check for a specific string.
        if (message.getMessageContentAsString().equals("Hello, World"))
        {
            // Modify the content of the first matching message.
            message.setMessageContent("Updated contents.");
            // Set it to be visible in 30 seconds.
            EnumSet<MessageUpdateFields> updateFields =
                EnumSet.of(MessageUpdateFields.CONTENT,
                MessageUpdateFields.VISIBILITY);
            // Update the message.
            queue.updateMessage(message, 30, updateFields, null, null);
            break;
        }
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

Следующий пример кода обновляет только первое видимое сообщение в очереди.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_UpdateFirstMessage":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Retrieve the first visible message in the queue.
    CloudQueueMessage message = queue.retrieveMessage();

    if (message != null)
    {
        // Modify the message content.
        message.setMessageContent("Updated contents.");
        // Set it to be visible in 60 seconds.
        EnumSet<MessageUpdateFields> updateFields =
            EnumSet.of(MessageUpdateFields.CONTENT,
            MessageUpdateFields.VISIBILITY);
        // Update the message.
        queue.updateMessage(message, 60, updateFields, null, null);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-get-the-queue-length"></a>Практическое руководство. Получение длины очереди

Вы можете узнать приблизительное количество сообщений в очереди.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

`getProperties`Метод возвращает несколько значений, включая количество сообщений, находящихся в очереди в данный момент. Число является приблизительным, так как сообщения могут добавляться или удаляться после запроса. `getApproximateMessageCount`Метод возвращает последнее значение, полученное при вызове метода `getProperties` , без вызова хранилища очередей.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_GetQueueLength":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

`downloadAttributes`Метод извлекает несколько значений, включая количество сообщений, находящихся в очереди в данный момент. Число является приблизительным, так как сообщения могут добавляться или удаляться после запроса. `getApproximateMessageCount`Метод возвращает последнее значение, полученное при вызове метода `downloadAttributes` , без вызова хранилища очередей.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
       CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

   // Download the approximate message count from the server.
    queue.downloadAttributes();

    // Retrieve the newly cached approximate message count.
    long cachedMessageCount = queue.getApproximateMessageCount();

    // Display the queue length.
    System.out.println(String.format("Queue length: %d", cachedMessageCount));
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-dequeue-the-next-message"></a>Практическое руководство. Удаление следующего сообщения из очереди

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Код удаляет сообщение из очереди в два этапа. При вызове `receiveMessage` вы получаете следующее сообщение в очереди. Сообщение, возвращаемое методом `receiveMessage`, становится невидимым для другого кода, считывающего сообщения из этой очереди. По умолчанию это сообщение остается невидимым в течение 30 секунд. Чтобы завершить удаление сообщения из очереди, необходимо также вызвать метод `deleteMessage` . Если код не может обработать сообщение, этот двухэтапный процесс гарантирует, что вы получите то же сообщение и повторите попытку. Код вызывается `deleteMessage` сразу после обработки сообщения.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_DequeueMessage":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

Код удаляет сообщение из очереди в два этапа. При вызове `retrieveMessage` вы получаете следующее сообщение в очереди. Сообщение, возвращаемое методом `retrieveMessage`, становится невидимым для другого кода, считывающего сообщения из этой очереди. По умолчанию это сообщение остается невидимым в течение 30 секунд. Чтобы завершить удаление сообщения из очереди, необходимо также вызвать метод `deleteMessage` . Если код не может обработать сообщение, этот двухэтапный процесс гарантирует, что вы получите то же сообщение и повторите попытку. Код вызывается `deleteMessage` сразу после обработки сообщения.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Retrieve the first visible message in the queue.
    CloudQueueMessage retrievedMessage = queue.retrieveMessage();

    if (retrievedMessage != null)
    {
        // Process the message in less than 30 seconds, and then delete the message.
        queue.deleteMessage(retrievedMessage);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="additional-options-for-dequeuing-messages"></a>Дополнительные варианты удаления сообщений из очереди

Извлечение сообщений из очереди можно настроить двумя способами. Сначала получите пакет сообщений (до 32). Во-вторых, задайте более длительное или короткое время ожидания невидимости, что позволяет коду больше или меньше времени на полную обработку каждого сообщения.

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

В следующем примере кода метод `receiveMessages` используется для получения 20 сообщений в одном вызове. Затем он обрабатывает каждое сообщение с помощью цикла `for`. Пример также задает время ожидания невидимости в размере пяти минут (300 секунд) для каждого сообщения. Время ожидания начинается для всех сообщений одновременно. Если после вызова функции прошло пять минут `receiveMessages` , все сообщения, не удаленные, снова станут видимыми.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_DequeueMessages":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

В следующем примере кода метод `retrieveMessages` используется для получения 20 сообщений в одном вызове. Затем он обрабатывает каждое сообщение с помощью цикла `for`. Пример также задает время ожидания невидимости в размере пяти минут (300 секунд) для каждого сообщения. Время ожидания начинается для всех сообщений одновременно. Если после вызова функции прошло пять минут `retrieveMessages` , все сообщения, не удаленные, снова станут видимыми.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Retrieve 20 messages from the queue with a visibility timeout of 300 seconds.
    for (CloudQueueMessage message : queue.retrieveMessages(20, 300, null, null)) {
        // Do processing for all messages in less than 5 minutes,
        // deleting each message after processing.
        queue.deleteMessage(message);
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-list-the-queues"></a>Как перечислять очереди

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Чтобы получить список текущих очередей, вызовите `QueueServiceClient.listQueues()` метод, который вернет коллекцию `QueueItem` объектов.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_ListQueues":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

Чтобы получить список текущих очередей, вызовите `CloudQueueClient.listQueues()` метод, который вернет коллекцию `CloudQueue` объектов.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient =
        storageAccount.createCloudQueueClient();

    // Loop through the collection of queues.
    for (CloudQueue queue : queueClient.listQueues())
    {
        // Output each queue name.
        System.out.println(queue.getName());
    }
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

## <a name="how-to-delete-a-queue"></a>Практическое руководство. Удаление очереди

# <a name="java-v12"></a>[Версия для Java 12](#tab/java)

Чтобы удалить очередь и все сообщения, содержащиеся в ней, вызовите `delete` метод для `QueueClient` объекта.

:::code language="java" source="~/azure-storage-snippets/queues/howto/java/java-v12/src/main/java/com/queues/howto/App.java" id="Snippet_DeleteMessageQueue":::

# <a name="java-v8"></a>[V8 Java](#tab/java8)

Чтобы удалить очередь и все сообщения, содержащиеся в ней, вызовите `deleteIfExists` метод для `CloudQueue` объекта.

```java
try
{
    // Retrieve storage account from connection-string.
    CloudStorageAccount storageAccount =
        CloudStorageAccount.parse(storageConnectionString);

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // Delete the queue if it exists.
    queue.deleteIfExists();
}
catch (Exception e)
{
    // Output the stack trace.
    e.printStackTrace();
}
```

---

[!INCLUDE [storage-check-out-samples-java](../../../includes/storage-check-out-samples-java.md)]

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали основные сведения о хранилище очередей, перейдите по следующим ссылкам, чтобы узнать о более сложных задачах хранилища.

- [Пакет SDK для службы хранилища Azure для Java](https://github.com/Azure/Azure-SDK-for-Java)
- [Справочник по пакету SDK для клиента службы хранилища Azure](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage)
- [REST API служб хранилища Azure](/rest/api/storageservices/)
- [Блог команды разработчиков службы хранилища Azure](https://techcommunity.Microsoft.com/t5/Azure-storage/bg-p/azurestorageblog)
