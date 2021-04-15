---
title: Включить файл
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
ms.openlocfilehash: e5b5433be4a95a9df9d3b3527473c3004d24acac
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107327288"
---
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно.
- [комплект SDK для Java (JDK)](https://docs.microsoft.com/azure/developer/java/fundamentals/java-jdk-install) версии 8 или более поздней версии.
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

### <a name="add-the-package-references-for-the-chat-sdk"></a>Добавление ссылок на пакет для пакета SDK для чата

В файле POM добавьте ссылку на пакет `azure-communication-chat` с помощью API-интерфейсов для чатов:

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-chat</artifactId>
    <version>1.0.0</version>
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

Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK для чата Служб коммуникации Azure для Java.

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatAsyncClient | Этот класс требуется для реализации всех асинхронных функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения пользователей, а также для отправки уведомлений о вводе и уведомления о прочтении. |
| ChatThreadAsyncClient | Этот класс требуется для реализации всех асинхронных функций потоков чата. Вы получаете экземпляр с помощью ChatAsyncClient и используете его для отправки/получения/обновления/удаления сообщений, добавления/удаления/получения пользователей и отправки уведомлений о вводе и уведомления о прочтении. |

## <a name="create-a-chat-client"></a>Создание клиента чата
Чтобы создать клиент чата, вы будете использовать конечную точку Служб коммуникации и маркер доступа, который был создан на шаге предварительных требований. Маркеры доступа пользователей позволяют создавать клиентские приложения, которые напрямую проходят проверку подлинности в Службах коммуникации Azure. После создания этих маркеров на сервере передайте их обратно клиентскому устройству. Для передачи маркера клиенту чата необходимо использовать класс CommunicationTokenCredential из общего пакета SDK.

Дополнительные сведения об [архитектуре чатов](../../../concepts/chat/concepts.md)

При добавлении операторов импорта обязательно добавляйте только импорты из пространств имен com.azure.communication.chat и com.azure.communication.chat.models, а не из com.azure.communication.chat.implementation. В файле App.java, созданном с помощью Maven, можно использовать следующий код:

```Java
package com.communication.quickstart;

import com.azure.communication.chat.*;
import com.azure.communication.chat.models.*;
import com.azure.communication.common.*;
import com.azure.core.http.rest.PagedIterable;

import java.io.*;
import java.util.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
        System.out.println("Azure Communication Services - Chat Quickstart");

        // Your unique Azure Communication service endpoint
        String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";

        // User access token fetched from your trusted service
        String userAccessToken = "<USER_ACCESS_TOKEN>";

        // Create a CommunicationTokenCredential with the given access token, which is only valid until the token is valid
        CommunicationTokenCredential userCredential = new CommunicationTokenCredential(userAccessToken);

        // Initialize the chat client
        final ChatClientBuilder builder = new ChatClientBuilder();
        builder.endpoint(endpoint)
            .credential(userCredential);
        ChatClient chatClient = builder.buildClient();
    }
}
```

## <a name="start-a-chat-thread"></a>Запуск потока чата

Для создания потока чата используйте метод `createChatThread`.
`createChatThreadOptions` используется для описания запроса потока.

- Для указания раздела в этом чате используйте параметр `topic` конструктора. Раздел можно обновить после создания потока чата с помощью функции `UpdateThread`.
- Используйте `participants`, чтобы получить список участников потока, добавляемых в поток. `ChatParticipant` принимает пользователя, которого вы создали при изучении краткого руководства о [маркере доступа пользователя](../../access-tokens.md).

`CreateChatThreadResult` — это ответ, возвращаемый при создании потока чата.
Он содержит метод `getChatThread()`, возвращающий объект `ChatThread`, который можно использовать для получения клиента потока, из которого можно получить `ChatThreadClient` для выполнения операций в созданном потоке: добавление участников, отправка сообщения и т. д. Объект `ChatThread` также содержит метод `getId()`, который получает уникальный идентификатор потока.

```Java
CommunicationUserIdentifier identity1 = new CommunicationUserIdentifier("<USER_1_ID>");
CommunicationUserIdentifier identity2 = new CommunicationUserIdentifier("<USER_2_ID>");

ChatParticipant firstThreadParticipant = new ChatParticipant()
    .setCommunicationIdentifier(identity1)
    .setDisplayName("Participant Display Name 1");

ChatParticipant secondThreadParticipant = new ChatParticipant()
    .setCommunicationIdentifier(identity2)
    .setDisplayName("Participant Display Name 2");

CreateChatThreadOptions createChatThreadOptions = new CreateChatThreadOptions("Topic")
    .addParticipant(firstThreadParticipant)
    .addParticipant(secondThreadParticipant);

CreateChatThreadResult result = chatClient.createChatThread(createChatThreadOptions);
String chatThreadId = result.getChatThread().getId();
```

## <a name="list-chat-threads"></a>Перечисление потоков чата

Для получения списка существующих потоков чата используйте метод `listChatThreads`.

```java
PagedIterable<ChatThreadItem> chatThreads = chatClient.listChatThreads();

chatThreads.forEach(chatThread -> {
    System.out.printf("ChatThread id is %s.\n", chatThread.getId());
});
```

## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата

Метод `getChatThreadClient` возвращает клиент потока для имеющегося потока. Его можно использовать для выполнения операций в созданном потоке: добавления участников, отправки сообщения и т. д. `chatThreadId` — это уникальный идентификатор существующего потока чата.

```Java
ChatThreadClient chatThreadClient = chatClient.getChatThreadClient(chatThreadId);
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

## <a name="receive-chat-messages-from-a-chat-thread"></a>Получение сообщений из потока чата

Вы можете получать сообщения чата, выполняя опрос метода `listMessages` на клиенте потока чата через определенные интервалы.

```Java
chatThreadClient.listMessages().forEach(message -> {
    System.out.printf("Message id is %s.\n", message.getId());
});
```

`listMessages` возвращает последнюю версию сообщения, включая все изменения или удаления, произошедшие с сообщением, с помощью .editMessage() и .deleteMessage(). Для удаленных сообщений `chatMessage.getDeletedOn()` возвращает значение даты и времени, указывающее, когда это сообщение было удалено. Для измененных сообщений `chatMessage.getEditedOn()` возвращает значение даты и времени, указывающее, когда это сообщение было изменено. Доступ к исходному времени создания сообщения можно получить с помощью `chatMessage.getCreatedOn()`, который также можно использовать для упорядочения сообщений.

Дополнительные сведения о типах сообщений см. в разделе [Типы сообщений](../../../concepts/chat/concepts.md#message-types).

## <a name="send-read-receipt"></a>Отправка уведомления о прочтении

Для публикации события уведомления о прочтении в потоке от имени пользователя используйте метод `sendReadReceipt`.
`chatMessageId` — это уникальный идентификатор считанного сообщения в чате.

```Java
String chatMessageId = message.getId();
chatThreadClient.sendReadReceipt(chatMessageId);
```

## <a name="list-chat-participants"></a>Перечисление участников чата

Для получения страничной коллекции, содержащей участников потока чата, идентифицируемого значением chatThreadId, используйте `listParticipants`.

```Java
PagedIterable<ChatParticipant> chatParticipantsResponse = chatThreadClient.listParticipants();
chatParticipantsResponse.forEach(chatParticipant -> {
    System.out.printf("Participant id is %s.\n", ((CommunicationUserIdentifier) chatParticipant.getCommunicationIdentifier()).getId());
});
```

## <a name="add-a-user-as-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

После создания потока чата можно добавлять и удалять пользователей. Добавляя пользователей, вы предоставляете им доступ для отправки сообщений в поток чата, а также добавления или удаления других участников. Необходимо начать с получения нового маркера доступа и удостоверения для этого пользователя. Прежде чем вызывать метод addParticipants, убедитесь, что вы получили новый маркер доступа и удостоверение для этого пользователя. Пользователю потребуется маркер доступа для инициализации своего клиента чата.

Для добавления участников в поток используйте метод `addParticipants`.

- `communicationIdentifier` (обязательно) — это идентификатор CommunicationIdentifier, созданный вами с помощью CommunicationIdentityClient при изучении краткого руководства по работе с [пользовательским маркером доступа](../../access-tokens.md).
- `displayName` (необязательно) — это отображаемое имя для участника потока.
- `shareHistoryTime` (необязательно) — это время, в течение которого участнику предоставляется доступ к журналу чата. Чтобы предоставить общий доступ к журналу с момента запуска потока чата, установите для этого свойства любую дату, равную или меньше времени создания потока. Чтобы в журнале отображались только записи с момента добавления участника, задайте для свойства текущую дату. Чтобы предоставить общий доступ к части журнала, задайте для него требуемую дату.

```Java
List<ChatParticipant> participants = new ArrayList<ChatParticipant>();

CommunicationUserIdentifier identity3 = new CommunicationUserIdentifier("<USER_3_ID>");
CommunicationUserIdentifier identity4 = new CommunicationUserIdentifier("<USER_4_ID>");

ChatParticipant thirdThreadParticipant = new ChatParticipant()
    .setCommunicationIdentifier(identity3)
    .setDisplayName("Display Name 3");

ChatParticipant fourthThreadParticipant = new ChatParticipant()
    .setCommunicationIdentifier(identity4)
    .setDisplayName("Display Name 4");

participants.add(thirdThreadParticipant);
participants.add(fourthThreadParticipant);

chatThreadClient.addParticipants(participants);
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
