---
title: включить файл
description: включить файл
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: 146053ffd72b24216bfa86577787727257da2516
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103495428"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [комплект SDK для Java (JDK)](/java/azure/jdk/) версии 8 или более поздней версии.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Развернутый ресурс Служб коммуникации и строка подключения. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- [Маркер доступа пользователя](../../access-tokens.md). Обязательно задайте для области значение chat и запишите строку маркера, а также строку userId.


## <a name="setting-up"></a>Настройка

### <a name="create-a-new-java-application"></a>Создание нового приложения Java

Откройте терминал или командное окно и перейдите в каталог, в котором необходимо создать приложение Java. Выполните приведенную ниже команду, чтобы создать проект Java из шаблона maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

Вы заметите, что цель generate создала каталог с тем же именем, что и у artifactId. В этом каталоге `src/main/java directory` содержит исходный код проекта, каталог `src/test/java` содержит источник теста, а файл pom.xml является объектной моделью проекта или [POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html).

Обновите файл POM приложения, чтобы использовать Java 8 или более поздней версии:

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

### <a name="add-the-package-references-for-the-chat-client-library"></a>Добавление ссылок на пакет для клиентской библиотеки чата

В файле POM добавьте ссылку на пакет `azure-communication-chat` с помощью API-интерфейсов для чатов:

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-chat</artifactId>
    <version>1.0.0-beta.4</version> 
</dependency>
```

Для проверки подлинности клиент должен добавить ссылку на пакет `azure-communication-common`:

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-common</artifactId>
    <version>1.0.0</version> 
</dependency>
```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки чата Служб коммуникации для Java.

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatAsyncClient | Этот класс требуется для реализации всех асинхронных функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения пользователей, а также для отправки уведомлений о вводе и уведомления о прочтении. |
| ChatThreadAsyncClient | Этот класс требуется для реализации всех асинхронных функций потоков чата. Вы получаете экземпляр с помощью ChatAsyncClient и используете его для отправки/получения/обновления/удаления сообщений, добавления/удаления/получения пользователей и отправки уведомлений о вводе и уведомления о прочтении. |

## <a name="create-a-chat-client"></a>Создание клиента чата
Чтобы создать клиент чата, вы будете использовать конечную точку Служб коммуникации и маркер доступа, который был создан на шаге предварительных требований. Маркеры доступа пользователей позволяют создавать клиентские приложения, которые напрямую проходят проверку подлинности в Службах коммуникации Azure. После создания этих маркеров на сервере передайте их обратно клиентскому устройству. Для передачи маркера клиенту чата необходимо использовать класс CommunicationTokenCredential из общей клиентской библиотеки. 

Дополнительные сведения об [архитектуре чатов](../../../concepts/chat/concepts.md)

При добавлении операторов импорта обязательно добавляйте только импорты из пространств имен com.azure.communication.chat и com.azure.communication.chat.models, а не из com.azure.communication.chat.implementation. В файле App.java, созданном с помощью Maven, можно использовать следующий код:

```Java
import com.azure.communication.chat.*;
import com.azure.communication.chat.models.*;
import com.azure.communication.common.*;
import com.azure.core.http.HttpClient;

import java.io.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
        System.out.println("Azure Communication Services - Chat Quickstart");
        
        // Your unique Azure Communication service endpoint
        String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";

        // Create an HttpClient builder of your choice and customize it
        // Use com.azure.core.http.netty.NettyAsyncHttpClientBuilder if that suits your needs
        NettyAsyncHttpClientBuilder yourHttpClientBuilder = new NettyAsyncHttpClientBuilder();
        HttpClient httpClient = yourHttpClientBuilder.build();

        // User access token fetched from your trusted service
        String userAccessToken = "<USER_ACCESS_TOKEN>";

        // Create a CommunicationTokenCredential with the given access token, which is only valid until the token is valid
        CommunicationTokenCredential userCredential = new CommunicationTokenCredential(userAccessToken);

        // Initialize the chat client
        final ChatClientBuilder builder = new ChatClientBuilder();
        builder.endpoint(endpoint)
            .credential(userCredential)
            .httpClient(httpClient);
        ChatClient chatClient = builder.buildClient();
    }
}
```


## <a name="start-a-chat-thread"></a>Запуск потока чата

Для создания потока чата используйте метод `createChatThread`.
`createChatThreadOptions` используется для описания запроса потока.

- Для указания раздела в этом чате используйте `topic`. Раздел можно обновить после создания потока чата с помощью функции `UpdateThread`.
- Используйте `participants`, чтобы получить список участников потока, добавляемых в поток. `ChatParticipant` принимает пользователя, которого вы создали при изучении краткого руководства о [маркере доступа пользователя](../../access-tokens.md).

Ответ `chatThreadClient`, который используется для выполнения операций с созданным потоком чата для добавления участников в поток, отправки, удаления сообщения и т. д. Он содержит свойство `chatThreadId`, которое является уникальным идентификатором потока чата. Свойство доступно для общедоступного метода .getChatThreadId().

```Java
List<ChatParticipant> participants = new ArrayList<ChatParticipant>();

ChatParticipant firstThreadParticipant = new ChatParticipant()
    .setCommunicationIdentifier(firstUser)
    .setDisplayName("Participant Display Name 1");
    
ChatParticipant secondThreadParticipant = new ChatParticipant()
    .setCommunicationIdentifier(secondUser)
    .setDisplayName("Participant Display Name 2");

participants.add(firstThreadParticipant);
participants.add(secondThreadParticipant);

CreateChatThreadOptions createChatThreadOptions = new CreateChatThreadOptions()
    .setTopic("Topic")
    .setParticipants(participants);
ChatThreadClient chatThreadClient = chatClient.createChatThread(createChatThreadOptions);
String chatThreadId = chatThreadClient.getChatThreadId();
```

## <a name="send-a-message-to-a-chat-thread"></a>Отправка сообщения в поток чата

Используйте метод `sendMessage` для отправки сообщения в созданный вами поток, идентифицируемый с помощью chatThreadId.
`sendChatMessageOptions` используется для описания запроса сообщения чата.

- Используйте `content`, чтобы указать содержимое сообщения в чате.
- Используйте `type` для указания типа содержимого сообщения в чате, текста или HTML.
- Используйте `senderDisplayName`, чтобы указать отображаемое имя отправителя.

`sendChatMessageResult` ответа содержит `id`, который является уникальным идентификатором сообщения.

```Java
SendChatMessageOptions sendChatMessageOptions = new SendChatMessageOptions()
    .setContent("Message content")
    .setType(ChatMessageType.TEXT)
    .setSenderDisplayName("Sender Display Name");

SendChatMessageResult sendChatMessageResult = chatThreadClient.sendMessage(sendChatMessageOptions);
String chatMessageId = sendChatMessageResult.getId();
```


## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата

Метод `getChatThreadClient` возвращает клиент потока для имеющегося потока. Его можно использовать для выполнения операций в созданном потоке: добавления участников, отправки сообщения и т. д. `chatThreadId` — это уникальный идентификатор существующего потока чата.

```Java
String chatThreadId = "Id";
ChatThread chatThread = chatClient.getChatThread(chatThreadId);
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Получение сообщений из потока чата

Вы можете получать сообщения чата, выполняя опрос метода `listMessages` на клиенте потока чата через определенные интервалы.

```Java
chatThreadClient.listMessages().iterableByPage().forEach(resp -> {
    System.out.printf("Response headers are %s. Url %s  and status code %d %n", resp.getHeaders(),
        resp.getRequest().getUrl(), resp.getStatusCode());
    resp.getItems().forEach(message -> {
        System.out.printf("Message id is %s.", message.getId());
    });
});
```

`listMessages` возвращает последнюю версию сообщения, включая все изменения или удаления, произошедшие с сообщением, с помощью .editMessage() и .deleteMessage(). Для удаленных сообщений `chatMessage.getDeletedOn()` возвращает значение даты и времени, указывающее, когда это сообщение было удалено. Для измененных сообщений `chatMessage.getEditedOn()` возвращает значение даты и времени, указывающее, когда это сообщение было изменено. Доступ к исходному времени создания сообщения можно получить с помощью `chatMessage.getCreatedOn()`, который также можно использовать для упорядочения сообщений.

`listMessages` возвращает различные типы сообщений, которые могут быть идентифицированы с помощью `chatMessage.getType()`. Это следующие типы:

- `text`. Обычное сообщение чата, отправленное участником потока.

- `html`. HTML-сообщение чата, отправленное участником потока.

- `topicUpdated`: системное сообщение, указывающее на изменение темы.

- `participantAdded`. Системное сообщение о том, что один или несколько участников были добавлены в поток чата.

- `participantRemoved`. Системное сообщение о том, что участник был удален из потока чата.

Дополнительные сведения см. в разделе о [типах сообщений](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

После создания потока чата можно добавлять и удалять пользователей. Добавляя пользователей, вы предоставляете им доступ для отправки сообщений в поток чата, а также добавления или удаления других участников. Необходимо начать с получения нового маркера доступа и удостоверения для этого пользователя. Прежде чем вызывать метод addParticipants, убедитесь, что вы получили новый маркер доступа и удостоверение для этого пользователя. Пользователю потребуется маркер доступа для инициализации своего клиента чата.

Используйте метод `addParticipants`, чтобы добавить участников в определяемый threadId поток.

- Используйте `listParticipants`, чтобы получить список участников, добавляемых в поток чата.
- `communicationIdentifier` (обязательно) — это идентификатор CommunicationIdentifier, созданный вами с помощью CommunicationIdentityClient при изучении краткого руководства по работе с [пользовательским маркером доступа](../../access-tokens.md).
- `display_name` (необязательно) — это отображаемое имя для участника потока.
- `share_history_time` (необязательно) — это время, в течение которого участнику предоставляется доступ к журналу чата. Чтобы предоставить общий доступ к журналу с момента запуска потока чата, установите для этого свойства любую дату, равную или меньше времени создания потока. Чтобы в журнале отображались только записи с момента добавления участника, задайте для свойства текущую дату. Чтобы предоставить общий доступ к части журнала, задайте для него требуемую дату.

```Java
List<ChatParticipant> participants = new ArrayList<ChatParticipant>();

ChatParticipant firstThreadParticipant = new ChatParticipant()
    .setCommunicationIdentifier(identity1)
    .setDisplayName("Display Name 1");

ChatParticipant secondThreadParticipant = new ChatParticipant()
    .setCommunicationIdentifier(identity2)
    .setDisplayName("Display Name 2");

participants.add(firstThreadParticipant);
participants.add(secondThreadParticipant);

AddChatParticipantsOptions addChatParticipantsOptions = new AddChatParticipantsOptions()
    .setParticipants(participants);
chatThreadClient.addParticipants(addChatParticipantsOptions);
```

## <a name="remove-participant-from-a-chat-thread"></a>Удаление участника из потока чата

Аналогично тому, как выполняется добавление участников в поток, вы можете удалять их из потока чата. Для этого необходимо отследить идентификаторы добавленных участников.

Используйте `removeParticipant`, где `identifier` — это созданный идентификатор CommunicationIdentifier.

```Java
chatThreadClient.removeParticipant(identity);
```

## <a name="run-the-code"></a>Выполнение кода

Перейдите в каталог, содержащий файл *pom.xml*, и скомпилируйте проект с помощью следующей команды `mvn`.

```console
mvn compile
```

Затем выполните сборку пакета.

```console
mvn package
```

Выполните следующую команду `mvn` для запуска приложения.

```console
mvn exec:java -Dexec.mainClass="com.communication.quickstart.App" -Dexec.cleanupDaemonThreads=false
```