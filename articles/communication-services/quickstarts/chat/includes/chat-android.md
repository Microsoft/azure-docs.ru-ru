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
ms.openlocfilehash: b2c5237f3f7e949edbfb5486a3a17cc6e0a008a4
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106178501"
---
[!INCLUDE [Public Preview Notice](../../../includes/public-preview-include-chat.md)]

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы нужно сделать следующее:

- Создайте учетную запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Установите [Android Studio](https://developer.android.com/studio). В этом кратком руководстве эта среда будет использоваться для создания приложения Android для установки зависимостей.
- Создайте ресурс Служб коммуникации Azure. Дополнительные сведения см. в статье [Краткое руководство по созданию ресурсов Служб коммуникации и управлению ими](../../create-communication-resource.md). Для работы с этим кратким руководством необходимо записать **конечную точку**.
- Создайте **двух** пользователей Служб коммуникации и выдайте им маркер доступа пользователя (подробнее о [маркере доступа пользователя](../../access-tokens.md)). Обязательно задайте для области значение **chat** и **запишите строку маркера, а также строку userId**. При работе с этим кратким руководством мы создадим поток с первым участником, а затем добавим в поток второго участника.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-android-application"></a>Создание приложения Android

1. Запустите Android Studio и выберите `Create a new project`. 
2. В следующем окне выберите шаблон проекта `Empty Activity`.
3. При выборе параметров введите имя проекта `ChatQuickstart`.
4. Нажмите кнопку Next (Далее) и выберите каталог, в котором нужно создать проект.

### <a name="install-the-libraries"></a>Установка библиотек

Мы будем использовать Gradle для установки необходимых зависимостей Служб коммуникации. В командной строке перейдите в корневой каталог проекта `ChatQuickstart`. Откройте файл build.gradle приложения и добавьте в целевой объект `ChatQuickstart` следующие зависимости:

```
implementation 'com.azure.android:azure-communication-common:1.0.0-beta.8'
implementation 'com.azure.android:azure-communication-chat:1.0.0-beta.8'
```

#### <a name="exclude-meta-files-in-packaging-options-in-root-buildgradle"></a>Исключение файлов метаданных в параметрах упаковки в файле build.gradle из корневой папки
```
android {
   ...
    packagingOptions {
        exclude 'META-INF/DEPENDENCIES'
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/license'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/notice'
        exclude 'META-INF/ASL2.0'
        exclude("META-INF/*.md")
        exclude("META-INF/*.txt")
        exclude("META-INF/*.kotlin_module")
    }
}
```

#### <a name="alternative-to-install-libraries-through-maven"></a>(Альтернативный вариант) Установка библиотек с помощью Maven
Чтобы импортировать библиотеку в проект с помощью системы сборки [Maven](https://maven.apache.org/), добавьте ее в раздел `dependencies` файла приложения `pom.xml`, указав идентификатор артефакта и версию, которая будет использоваться.

```xml
<dependency>
  <groupId>com.azure.android</groupId>
  <artifactId>azure-communication-chat</artifactId>
  <version>1.0.0-beta.8</version>
</dependency>
```


### <a name="setup-the-placeholders"></a>Настройка заполнителей

Откройте файл `MainActivity.java` и измените его. В рамках этого краткого руководства мы добавим наш код в класс `MainActivity` и просмотрим выходные данные в консоли. В этом кратком руководстве не рассматривается создание пользовательского интерфейса. Импортируйте библиотеки `Communication common` и `Communication chat` в начало файла:

```
import com.azure.android.communication.chat.*;
import com.azure.android.communication.common.*;
```

Скопируйте в файл `MainActivity` следующий код:

```java
    private String secondUserId = "<second_user_id>";
    private String threadId = "<thread_id>";
    private String chatMessageId = "<chat_message_id>";
    private final String sdkVersion = "1.0.0-beta.8";
    private static final String APPLICATION_ID = "Chat Quickstart App";
    private static final String SDK_NAME = "azure-communication-com.azure.android.communication.chat";
    private static final String TAG = "--------------Chat Quickstart App-------------";

    private void log(String msg) {
        Log.i(TAG, msg);
        Toast.makeText(this, msg, Toast.LENGTH_LONG).show();
    }
    
   @Override
    protected void onStart() {
        super.onStart();
        try {
            // <CREATE A CHAT CLIENT>

            // <CREATE A CHAT THREAD>

            // <CREATE A CHAT THREAD CLIENT>

            // <SEND A MESSAGE>

            // <ADD A USER>

            // <LIST USERS>

            // <REMOVE A USER>
            
            // <<SEND A TYPING NOTIFICATION>>
            
            // <<SEND A READ RECEIPT>>
               
            // <<LIST READ RECEIPTS>>
        } catch (Exception e){
            System.out.println("Quickstart failed: " + e.getMessage());
        }
    }
```

В следующих шагах мы заменим заполнители примером кода с помощью библиотеки чата Служб коммуникации Azure.


### <a name="create-a-chat-client"></a>Создание клиента чата

Замените комментарий `<CREATE A CHAT CLIENT>` следующим кодом (вставьте операторы import в начало файла):

```java
import com.azure.android.communication.chat.ChatAsyncClient;
import com.azure.android.communication.chat.ChatClientBuilder;
import com.azure.android.core.credential.AccessToken;
import com.azure.android.core.http.HttpHeader;
import com.azure.android.core.http.okhttp.OkHttpAsyncClientProvider;
import com.azure.android.core.http.policy.BearerTokenAuthenticationPolicy;
import com.azure.android.core.http.policy.UserAgentPolicy;

final String endpoint = "https://<resource>.communication.azure.com";
final String userAccessToken = "<user_access_token>";

ChatAsyncClient chatAsyncClient = new ChatClientBuilder()
    .endpoint(endpoint)
    .credentialPolicy(new BearerTokenAuthenticationPolicy((request, callback) ->
        callback.onSuccess(new AccessToken(userAccessToken, OffsetDateTime.now().plusDays(1)))))
    .addPolicy(new UserAgentPolicy(APPLICATION_ID, SDK_NAME, sdkVersion))
    .httpClient(new OkHttpAsyncClientProvider().createInstance())
    .buildAsyncClient();

```

1. С помощью `ChatClientBuilder` создайте экземпляр `ChatAsyncClient` и настройте его.
2. Замените `<resource>` своим ресурсом Служб коммуникации.
3. Замените `<user_access_token>` допустимым маркером доступа Служб коммуникации.

## <a name="object-model"></a>Объектная модель
Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK для чата Служб коммуникации для JavaScript.

| Имя                                   | Описание                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ChatClient/ChatAsyncClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient/ChatThreadAsyncClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения пользователей, отправки уведомлений о вводе и уведомления о прочтении и подписки на чат. |

## <a name="start-a-chat-thread"></a>Запуск потока чата

Мы воспользуемся `ChatAsyncClient`, чтобы создать поток с первым пользователем.

Замените комментарий `<CREATE A CHAT THREAD>` следующим фрагментом кода:

```java
// A list of ChatParticipant to start the thread with.
List<ChatParticipant> participants = new ArrayList<>();
// The communication user ID you created before, required.
String id = "<user_id>";
// The display name for the thread participant.
String displayName = "initial participant";
participants.add(new ChatParticipant()
    .setCommunicationIdentifier(new CommunicationUserIdentifier(id))
    .setDisplayName(displayName));

// The topic for the thread.
final String topic = "General";
// Optional, set a repeat request ID.
final String repeatabilityRequestID = "";
// Options to pass to the create method.
CreateChatThreadOptions createChatThreadOptions = new CreateChatThreadOptions()
    .setTopic(topic)
    .setParticipants(participants)
    .setIdempotencyToken(repeatabilityRequestID);

CreateChatThreadResult createChatThreadResult =
    chatAsyncClient.createChatThread(createChatThreadOptions).get();
ChatThreadProperties chatThreadProperties = createChatThreadResult.getChatThreadProperties();
threadId = chatThreadProperties.getId();

```

Замените `<user_id>` допустимым идентификатором пользователя Служб коммуникации. Мы будем использовать `threadId` из ответа, возвращенного завершающему обработчику, в последующих шагах, поэтому в классе замените `<thread_id>` `threadId`, полученным из этого запроса, и запустите приложение повторно.

## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата

Теперь, когда создан поток чата, получим `ChatThreadAsyncClient` для выполнения операций в потоке. Замените комментарий `<CREATE A CHAT THREAD CLIENT>` следующим фрагментом кода:

```
ChatThreadAsyncClient chatThreadAsyncClient = new ChatThreadClientBuilder()
    .endpoint(endpoint)
    .credentialPolicy(new BearerTokenAuthenticationPolicy((request, callback) ->
        callback.onSuccess(new AccessToken(userAccessToken, OffsetDateTime.now().plusDays(1)))))
    .addPolicy(new UserAgentPolicy(APPLICATION_ID, SDK_NAME, sdkVersion))
    .httpClient(new OkHttpAsyncClientProvider().createInstance())
    .chatThreadId(threadId)
    .buildAsyncClient();

```

## <a name="send-a-message-to-a-chat-thread"></a>Отправка сообщения в поток чата

Убедитесь, что `<thread_id>` заменен допустимым идентификатором потока. Сейчас мы отправим сообщение в этот поток.

Замените комментарий `<SEND A MESSAGE>` следующим фрагментом кода:

```java
// The chat message content, required.
final String content = "Test message 1";
// The display name of the sender, if null (i.e. not specified), an empty name will be set.
final String senderDisplayName = "An important person";
SendChatMessageOptions chatMessageOptions = new SendChatMessageOptions()
    .setType(ChatMessageType.TEXT)
    .setContent(content)
    .setSenderDisplayName(senderDisplayName);

// A string is the response returned from sending a message, it is an id, which is the unique ID of the message.
chatMessageId = chatThreadAsyncClient.sendMessage(chatMessageOptions).get().getId();

```

После получения `chatMessageId` можно заменить `<chat_message_id>` `chatMessageId` для дальнейшего использования метода в кратком руководстве и повторно запустить приложение.

## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

Замените комментарий `<ADD A USER>` следующим фрагментом кода:

```java
// The display name for the thread participant.
displayName = "a new participant";
ChatParticipant participant = new ChatParticipant()
    .setCommunicationIdentifier(new CommunicationUserIdentifier(secondUserId))
    .setDisplayName(secondUserDisplayName);
        
chatThreadAsyncClient.addParticipant(participant);

```

Замените `<second_user_id>` в классе идентификатором добавляемого пользователя Служб коммуникации. 

## <a name="list-users-in-a-thread"></a>Вывод списка пользователей в потоке

Замените комментарий `<LIST USERS>` следующим кодом:

```java
// The maximum number of participants to be returned per page, optional.
final int maxPageSize = 10;

// Skips participants up to a specified position in response.
final int skip = 0;

// Options to pass to the list method.
ListParticipantsOptions listParticipantsOptions = new ListParticipantsOptions()
    .setMaxPageSize(maxPageSize)
    .setSkip(skip);

PagedResponse<ChatParticipant> firstPageWithResponse =
    chatThreadAsyncClient.getParticipantsFirstPageWithResponse(listParticipantsOptions, Context.NONE).get();

for (ChatParticipant participant : firstPageWithResponse.getValue()) {
    // You code to handle participant
}

listParticipantsNextPage(firstPageWithResponse.getContinuationToken(), 2);

```

Вставьте следующий вспомогательный метод в класс:

```java
void listParticipantsNextPage(String continuationToken, int pageNumber) {
if (continuationToken != null) {
    PagedResponse<ChatParticipant> nextPageWithResponse =
        chatThreadAsyncClient.getParticipantsNextPageWithResponse(continuationToken, Context.NONE).get();
        for (ChatParticipant participant : nextPageWithResponse.getValue()) {
            // You code to handle participant
        }
            
        listParticipantsNextPage(nextPageWithResponse.getContinuationToken(), ++pageNumber);
    }
}

```


## <a name="remove-user-from-a-chat-thread"></a>Удаление пользователя из потока чата

Обязательно замените `<second_user_id>` допустимым идентификатором пользователя. Сейчас мы удалим из потока второго пользователя.

Замените комментарий `<REMOVE A USER>` следующим кодом:

```java
// Using the unique ID of the participant.
chatThreadAsyncClient.removeParticipant(new CommunicationUserIdentifier(secondUserId)).get();

```

## <a name="send-a-typing-notification"></a>Отправка уведомления о вводе

Замените комментарий `<SEND A TYPING NOTIFICATION>` следующим кодом:

```java
chatThreadAsyncClient.sendTypingNotification().get();
```

## <a name="send-a-read-receipt"></a>Отправка уведомления о прочтении

Обязательно замените `<chat_message_id>` допустимым идентификатором сообщения чата. Сейчас мы отправим уведомление о прочтении этого сообщения.

Замените комментарий `<SEND A READ RECEIPT>` следующим кодом:

```java
chatThreadAsyncClient.sendReadReceipt(chatMessageId).get();
```

## <a name="list-read-receipts"></a>Получение списка уведомлений о прочтении

Замените комментарий `<READ RECEIPTS>` следующим кодом:

```java
// The maximum number of participants to be returned per page, optional.
maxPageSize = 10;
// Skips participants up to a specified position in response.
skip = 0;
// Options to pass to the list method.
ListReadReceiptOptions listReadReceiptOptions = new ListReadReceiptOptions()
    .setMaxPageSize(maxPageSize)
    .setSkip(skip);

PagedResponse<ChatMessageReadReceipt> firstPageWithResponse =
    chatThreadAsyncClient.getReadReceiptsFirstPageWithResponse(listReadReceiptOptions, Context.NONE).get();

for (ChatMessageReadReceipt readReceipt : firstPageWithResponse.getValue()) {
    // You code to handle readReceipt
}

listReadReceiptsNextPage(firstPageWithResponse.getContinuationToken(), 2);

```

Вставьте следующий вспомогательный метод в класс:
```java
void listReadReceiptsNextPage(String continuationToken, int pageNumber) {
    if (continuationToken != null) {
        PagedResponse<ChatMessageReadReceipt> nextPageWithResponse =
            chatThreadAsyncClient.getReadReceiptsNextPageWithResponse(continuationToken, Context.NONE).get();

        for (ChatMessageReadReceipt readReceipt : nextPageWithResponse.getValue()) {
            // You code to handle readReceipt
        }

        listParticipantsNextPage(nextPageWithResponse.getContinuationToken(), ++pageNumber);
    }
}

```


## <a name="run-the-code"></a>Выполнение кода

В Android Studio нажмите кнопку "Run" (Выполнить), чтобы выполнить сборку и запуск проекта. В консоли можно просмотреть выходные данные кода и средства ведения журнала из ChatClient.
