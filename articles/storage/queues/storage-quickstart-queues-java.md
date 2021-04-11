---
title: Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 — Java
description: Узнайте, как использовать клиентскую библиотеку Хранилища очередей Azure версии 12 для Java, чтобы создавать очереди и добавлять в них сообщения, а также считывать и удалять сообщения из очереди. Также здесь описано, как удалить очередь.
author: twooley
ms.author: twooley
ms.date: 12/01/2020
ms.topic: quickstart
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-java
ms.openlocfilehash: 79b10e8281fe8189b490fe97dfe896179c8e64fc
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106275878"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-java"></a>Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 для Java

Приступите к работе с клиентской библиотекой Хранилища очередей Azure версии 12 для Java. Хранилище очередей Azure — это служба хранения большого количества сообщений для последующего извлечения и обработки. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для выполнения базовых задач.

Клиентскую библиотеку Хранилища очередей Azure версии 12 для Java можно использовать для выполнения следующих задач.

- Создание очереди
- Добавление сообщений в очередь
- Просмотр сообщений из очереди
- Обновление сообщений в очереди
- Получение и удаление сообщений из очереди
- Удаление очереди

Дополнительные ресурсы:

- [Справочная документация по API](/java/api/overview/azure/storage-queue-readme)
- [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-queue)
- [Пакет (Maven)](https://mvnrepository.com/artifact/com.azure/azure-storage-queue)
- [Примеры](../common/storage-samples-java.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Предварительные требования

- [Пакет разработчиков Java (JDK)](/java/azure/jdk/) версии 8 или более поздней версии.
- [Apache Maven](https://maven.apache.org/download.cgi)
- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- Учетная запись хранения Azure — [создайте такую учетную запись](../common/storage-account-create.md).

## <a name="setting-up"></a>Настройка

В этом разделе рассматривается подготовка проекта для работы с клиентской библиотекой Хранилища очередей Azure версии 12 для Java.

### <a name="create-the-project"></a>Создание проекта

Создайте приложение Java с именем `queues-quickstart-v12`.

1. В окне консоли (командная строка, PowerShell или Bash) с помощью Maven создайте консольное приложение с именем `queues-quickstart-v12`. Введите следующую команду `mvn`, чтобы создать проект Java hello world.

    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    mvn archetype:generate `
        --define interactiveMode=n `
        --define groupId=com.queues.quickstart `
        --define artifactId=queues-quickstart-v12 `
        --define archetypeArtifactId=maven-archetype-quickstart `
        --define archetypeVersion=1.4
    ```

    # <a name="bash"></a>[Bash](#tab/bash)

    ```bash
    mvn archetype:generate \
        --define interactiveMode=n \
        --define groupId=com.queues.quickstart \
        --define artifactId=queues-quickstart-v12 \
        --define archetypeArtifactId=maven-archetype-quickstart \
        --define archetypeVersion=1.4
    ```

    ---

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
    [INFO] Parameter: groupId, Value: com.queues.quickstart
    [INFO] Parameter: artifactId, Value: queues-quickstart-v12
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.queues.quickstart
    [INFO] Parameter: packageInPathFormat, Value: com/queues/quickstart
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.queues.quickstart
    [INFO] Parameter: groupId, Value: com.queues.quickstart
    [INFO] Parameter: artifactId, Value: queues-quickstart-v12
    [INFO] Project created from Archetype in dir: C:\quickstarts\queues\queues-quickstart-v12
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  6.394 s
    [INFO] Finished at: 2019-12-03T09:58:35-08:00
    [INFO] ------------------------------------------------------------------------
    ```

1. Переключитесь на только что созданный каталог `queues-quickstart-v12`.

   ```console
   cd queues-quickstart-v12
   ```

### <a name="install-the-package"></a>Установка пакета

Откройте файл `pom.xml` в текстовом редакторе. Добавьте приведенный ниже элемент зависимости в группу зависимостей.

```xml
<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-storage-queue</artifactId>
  <version>12.0.1</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Перейдите в каталог `/src/main/java/com/queues/quickstart`.
1. Откройте файл `App.java` в редакторе.
1. Удалите оператор `System.out.println("Hello, world");`.
1. Добавьте директивы `import`.

Вот этот код:

```java
package com.queues.quickstart;

/**
 * Azure Queue Storage client library v12 quickstart
 */
import com.azure.storage.queue.*;
import com.azure.storage.queue.models.*;
import java.io.*;
import java.time.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
    }
}
```

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Объектная модель

Хранилище очередей Azure — это служба для хранения большого количества сообщений. Максимальный размер сообщения в очереди составляет 64 КБ. Очередь может содержать миллионы сообщений вплоть до лимита всей емкости учетной записи хранения. Очереди обычно используются для создания списка невыполненных заданий для асинхронной обработки. В Хранилище очередей предлагается три типа ресурсов:

- учетная запись хранения;
- очередь в учетной записи хранилища;
- сообщения в очереди.

На следующей схеме показана связь между этими ресурсами.

![Схема архитектуры Хранилища очередей](./media/storage-queues-introduction/queue1.png)

Используйте следующие классы Java для взаимодействия с этими ресурсами.

- [`QueueClientBuilder`](/java/api/com.azure.storage.queue.queueclientbuilder): Класс `QueueClientBuilder` настраивает объект `QueueClient` и создает его экземпляр.
- [`QueueServiceClient`](/java/api/com.azure.storage.queue.queueserviceclient): `QueueServiceClient` позволяет управлять всеми очередями в учетной записи хранения.
- [`QueueClient`](/java/api/com.azure.storage.queue.queueclient). Класс `QueueClient` позволяет управлять отдельной очередью и сообщениями в ней.
- [`QueueMessageItem`](/java/api/com.azure.storage.queue.models.queuemessageitem). Класс `QueueMessageItem` представляет отдельные объекты, которые возвращаются при вызове [`ReceiveMessages`](/java/api/com.azure.storage.queue.queueclient.receivemessages) для очереди.

## <a name="code-examples"></a>Примеры кода

В этих примерах фрагментов кода показано, как выполнять следующие действия с помощью клиентской библиотеки Хранилища очередей Azure для Java.

- [Получение строки подключения](#get-the-connection-string)
- [Создание очереди](#create-a-queue)
- [Добавление сообщений в очередь](#add-messages-to-a-queue)
- [Просмотр сообщений из очереди](#peek-at-messages-in-a-queue)
- [Обновление сообщений в очереди](#update-a-message-in-a-queue)
- [Получение и удаление сообщений из очереди](#receive-and-delete-messages-from-a-queue)
- [Удаление очереди](#delete-a-queue)

### <a name="get-the-connection-string"></a>Получение строки подключения

Приведенный ниже код извлекает строку подключения к учетной записи хранения. Эта строка подключения сохраняется в переменной среды, которую вы создали в разделе [Настройка строки подключения хранилища](#configure-your-storage-connection-string).

Добавьте этот код в метод `main`.

```java
System.out.println("Azure Queue Storage client library v12 - Java quickstart sample\n");

// Retrieve the connection string for use with the application. The storage
// connection string is stored in an environment variable on the machine
// running the application called AZURE_STORAGE_CONNECTION_STRING. If the environment variable
// is created after the application is launched in a console or with
// Visual Studio, the shell or application needs to be closed and reloaded
// to take the environment variable into account.
String connectStr = System.getenv("AZURE_STORAGE_CONNECTION_STRING");
```

### <a name="create-a-queue"></a>Создание очереди

Выберите имя для новой очереди. Приведенный ниже код добавляет к имени очереди значение GUID, чтобы сделать это имя уникальным.

> [!IMPORTANT]
> Имя очереди может содержать только строчные буквы, цифры и дефисы и должно начинаться с буквы или цифры. Перед каждым дефисом должен быть знак без дефиса. Количество символов в имени должно быть от 3 до 63. Дополнительные сведения см. в статье о [присвоении имен очередям и метаданным](/rest/api/storageservices/naming-queues-and-metadata).

Создайте экземпляр класса [`QueueClient`](/java/api/com.azure.storage.queue.queueclient). Затем вызовите метод [`Create`](/java/api/com.azure.storage.queue.queueclient.create), чтобы создать очередь в своей учетной записи хранения.

Добавьте следующий код в конец метода `main`.

```java
// Create a unique name for the queue
String queueName = "quickstartqueues-" + java.util.UUID.randomUUID();

System.out.println("Creating queue: " + queueName);

// Instantiate a QueueClient which will be
// used to create and manipulate the queue
QueueClient queueClient = new QueueClientBuilder()
                                .connectionString(connectStr)
                                .queueName(queueName)
                                .buildClient();

// Create the queue
queueClient.create();
```

### <a name="add-messages-to-a-queue"></a>Добавление сообщений в очередь

Следующий фрагмент кода добавляет сообщения в очередь, вызывая метод [`sendMessage`](/java/api/com.azure.storage.queue.queueclient.sendmessage). Он также сохраняет [`SendMessageResult`](/java/api/com.azure.storage.queue.models.sendmessageresult), возвращенный вызовом `sendMessage`. Этот результат используется для обновления сообщения далее в коде программы.

Добавьте следующий код в конец метода `main`.

```java
System.out.println("\nAdding messages to the queue...");

// Send several messages to the queue
queueClient.sendMessage("First message");
queueClient.sendMessage("Second message");

// Save the result so we can update this message later
SendMessageResult result = queueClient.sendMessage("Third message");
```

### <a name="peek-at-messages-in-a-queue"></a>Просмотр сообщений из очереди

Чтобы просмотреть сообщения в очереди, вызовите метод [`peekMessages`](/java/api/com.azure.storage.queue.queueclient.peekmessages). Этот метод извлекает одно или несколько сообщений из начала очереди, не изменяя видимость этих сообщений.

Добавьте следующий код в конец метода `main`.

```java
System.out.println("\nPeek at the messages in the queue...");

// Peek at messages in the queue
queueClient.peekMessages(10, null, null).forEach(
    peekedMessage -> System.out.println("Message: " + peekedMessage.getMessageText()));
```

### <a name="update-a-message-in-a-queue"></a>Обновление сообщений в очереди

Чтобы обновить содержимое сообщения, вызовите метод [`updateMessage`](/java/api/com.azure.storage.queue.queueclient.updatemessage). Этот метод позволяет изменить время видимости сообщения и его содержимое. Содержимое сообщение должно иметь формат строки в кодировке UTF-8 длиной не более 64 КБ. Вместе с новым содержимым сообщения передайте идентификатор сообщения и подтверждение извлечения, используя сохраненный ранее `SendMessageResult`. Идентификатор сообщения и подтверждение извлечения указывают, какое сообщение следует обновить.

```java
System.out.println("\nUpdating the third message in the queue...");

// Update a message using the result that
// was saved when sending the message
queueClient.updateMessage(result.getMessageId(),
                          result.getPopReceipt(),
                          "Third message has been updated",
                          Duration.ofSeconds(1));
```

### <a name="receive-and-delete-messages-from-a-queue"></a>Получение и удаление сообщений из очереди

Чтобы скачать ранее добавленные сообщения, вызовите метод [`receiveMessages`](/java/api/com.azure.storage.queue.queueclient.receivemessages). Также этот пример кода удаляет из очереди сообщения, которые уже получены и обработаны. В нашем примере обработка сводится к выводу сообщения в консоль.

Перед получением и удалением сообщений приложение ожидает ввода от пользователя, вызывая метод `System.console().readLine();`. Перед удалением ресурсов убедитесь на [портале Azure](https://portal.azure.com), что они были правильно созданы. Все сообщения, которые не были удалены явным образом, снова становятся видимыми в очереди для новой попытки их обработать.

Добавьте следующий код в конец метода `main`.

```java
System.out.println("\nPress Enter key to receive messages and delete them from the queue...");
System.console().readLine();

// Get messages from the queue
queueClient.receiveMessages(10).forEach(
    // "Process" the message
    receivedMessage -> {
        System.out.println("Message: " + receivedMessage.getMessageText());

        // Let the service know we're finished with
        // the message and it can be safely deleted.
        queueClient.deleteMessage(receivedMessage.getMessageId(), receivedMessage.getPopReceipt());
    }
);
```

### <a name="delete-a-queue"></a>Удаление очереди

Следующий код очищает созданные приложением ресурсы, удаляя очередь с помощью метода [`Delete`](/java/api/com.azure.storage.queue.queueclient.delete).

Добавьте следующий код в конец метода `main`.

```java
System.out.println("\nPress Enter key to delete the queue...");
System.console().readLine();

// Clean up
System.out.println("Deleting queue: " + queueClient.getQueueName());
queueClient.delete();

System.out.println("Done");
```

## <a name="run-the-code"></a>Выполнение кода

Это приложение создает три сообщения и добавляет их в очередь Azure. Затем код получает список сообщений, извлекает и удаляет их, и наконец удаляет саму очередь.

В окне консоли перейдите к каталогу приложения, выполните его сборку и запустите его.

```console
mvn compile
```

Затем выполните сборку пакета.

```console
mvn package
```

Выполните следующую команду `mvn` для запуска этого приложения.

```console
mvn exec:java -Dexec.mainClass="com.queues.quickstart.App" -Dexec.cleanupDaemonThreads=false
```

Вы должны увидеть выходные данные приложения, как показано ниже.

```output
Azure Queue Storage client library v12 - Java quickstart sample

Adding messages to the queue...

Peek at the messages in the queue...
Message: First message
Message: Second message
Message: Third message

Updating the third message in the queue...

Press Enter key to receive messages and delete them from the queue...

Message: First message
Message: Second message
Message: Third message has been updated

Press Enter key to delete the queue...

Deleting queue: quickstartqueues-fbf58f33-4d5a-41ac-ac0e-1a05d01c7003
Done
```

Когда приложение приостановится перед получением сообщений, проверьте учетную запись хранения на [портале Azure](https://portal.azure.com). Убедитесь, что сообщения находятся в очереди.

Нажмите клавишу `Enter`, чтобы получить и удалить сообщения. При появлении запроса снова нажмите клавишу `Enter`, чтобы удалить очередь и завершить демонстрацию.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создавать очередь и добавлять в нее сообщения из кода на Java. Затем вы изучили процессы вставки, просмотра, получения и удаления сообщений. Наконец, вы узнали, как удалить очередь сообщений.

Руководства, примеры, краткие руководства и другую документацию можно найти здесь:

> [!div class="nextstepaction"]
> [Azure для разработчиков облачных решений Java](/azure/developer/java/)

- Дополнительные примеры приложений для клиентской библиотеки Хранилища очередей Azure версии 12 для Java см. [здесь](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue).
