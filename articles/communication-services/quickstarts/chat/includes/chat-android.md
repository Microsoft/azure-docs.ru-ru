---
title: включить файл
description: включить файл
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 2/16/2020
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: 021abce5c6cd83257ad65f529833848d8f14f534
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101750747"
---
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
implementation 'com.azure.android:azure-communication-common:1.0.0-beta.6'
implementation 'com.azure.android:azure-communication-chat:1.0.0-beta.6'
```

#### <a name="exclude-meta-files-in-packaging-options"></a>Исключение метафайлов в параметрах упаковки
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

В Android Studio щелкните "sync now" (Синхронизировать сейчас).

#### <a name="alternative-to-install-libraries-through-maven"></a>(Альтернативный вариант) Установка библиотек с помощью Maven
Чтобы импортировать библиотеку в проект с помощью системы сборки [Maven](https://maven.apache.org/), добавьте ее в раздел `dependencies` файла приложения `pom.xml`, указав идентификатор артефакта и версию, которая будет использоваться.

```xml
<dependency>
  <groupId>com.azure.android</groupId>
  <artifactId>azure-communication-chat</artifactId>
  <version>1.0.0-beta.6</version>
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
    private String second_user_id = "<second_user_id>";
    private String threadId = "<thread_id>";
    private String chatMessageId = "<chat_message_id>";
    private final String sdkVersion = "1.0.0-beta.6";
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
import com.azure.android.communication.chat.ChatClient;
import com.azure.android.core.http.HttpHeader;

final String endpoint = "https://<resource>.communication.azure.com";
final String userAccessToken = "<user_access_token>";

ChatAsyncClient client = new ChatAsyncClient.Builder()
        .endpoint(endpoint)
        .credentialInterceptor(chain -> chain.proceed(chain.request()
                .newBuilder()
                .header(HttpHeader.AUTHORIZATION, "Bearer " + userAccessToken)
                .build()))
        .build();
```

1. С помощью `ChatAsyncClient.Builder` создайте экземпляр `ChatAsyncClient` и настройте его.
2. Замените `<resource>` своим ресурсом Служб коммуникации.
3. Замените `<user_access_token>` допустимым маркером доступа Служб коммуникации.

## <a name="object-model"></a>Объектная модель
Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки чата Служб коммуникации для JavaScript.

| Имя                                   | Описание                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ChatClient/ChatAsyncClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient/ChatThreadAsyncClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения пользователей, отправки уведомлений о вводе и уведомления о прочтении и подписки на чат. |

## <a name="start-a-chat-thread"></a>Запуск потока чата

Мы воспользуемся `ChatClient`, чтобы создать поток с первым пользователем.

Замените комментарий `<CREATE A CHAT THREAD>` следующим фрагментом кода:

```java
//  The list of ChatParticipant to be added to the thread.
List<ChatParticipant> participants = new ArrayList<>();
// The communication user ID you created before, required.
String id = "<user_id>";
// The display name for the thread participant.
String displayName = "initial participant";
participants.add(new ChatParticipant()
        .setId(id)
        .setDisplayName(displayName)
);

// The topic for the thread.
final String topic = "General";
// The model to pass to the create method.
CreateChatThreadRequest thread = new CreateChatThreadRequest()
  .setTopic(topic)
  .setParticipants(participants);

// optional, set a repeat request ID
final String repeatabilityRequestID = "";

client.createChatThread(thread, repeatabilityRequestID, new Callback<CreateChatThreadResult>() {
      public void onSuccess(CreateChatThreadResult result, okhttp3.Response response) {
              ChatThread chatThread = result.getChatThread();
              threadId = chatThread.getId();
              // take further action
              Log.i(TAG, "threadId: " + threadId);
      }

      public void onFailure(Throwable throwable, okhttp3.Response response) {
              // Handle error.
              Log.e(TAG, throwable.getMessage());
      }
});
```

Замените `<user_id>` допустимым идентификатором пользователя Служб коммуникации. Мы будем использовать `threadId` из ответа, возвращенного завершающему обработчику, в последующих шагах, поэтому в классе замените `<thread_id>` `threadId`, полученным из этого запроса, и запустите приложение повторно.

## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата

Теперь, когда создан поток чата, получим `ChatThreadClient` для выполнения операций в потоке. Замените комментарий `<CREATE A CHAT THREAD CLIENT>` следующим фрагментом кода:

```
ChatThreadAsyncClient threadClient =
        new ChatThreadAsyncClient.Builder()
                .endpoint(endpoint)
                .credentialInterceptor(chain -> chain.proceed(chain.request()
                    .newBuilder()
                    .header(HttpHeader.AUTHORIZATION, "Bearer " + userAccessToken)
                    .build()))
                .build();
```

## <a name="send-a-message-to-a-chat-thread"></a>Отправка сообщения в поток чата

Убедитесь, что `<thread_id>` заменен допустимым идентификатором потока. Сейчас мы отправим сообщение в этот поток.

Замените комментарий `<SEND A MESSAGE>` следующим фрагментом кода:

```java
// The chat message content, required.
final String content = "Test message 1";
// The display name of the sender, if null (i.e. not specified), an empty name will be set.
final String senderDisplayName = "An important person";
SendChatMessageRequest message = new SendChatMessageRequest()
        .setType(ChatMessageType.TEXT)
        .setContent(content)
        .setSenderDisplayName(senderDisplayName);

threadClient.sendChatMessage(threadId, message, new Callback<String>() {
        @Override
        public void onSuccess(String messageId, okhttp3.Response response) {
                // A string is the response returned from sending a message, it is an id,
                // which is the unique ID of the message.
                chatMessageId = messageId;
                // Take further action.
                Log.i(TAG, "chatMessageId: " + chatMessageId);
        }

        @Override
        public void onFailure(Throwable throwable, okhttp3.Response response) {
                // Handle error.
                Log.e(TAG, throwable.getMessage());
        }
});
```

После получения `chatMessageId` можно заменить `<chat_message_id>` `chatMessageId` для дальнейшего использования метода в кратком руководстве и повторно запустить приложение.

## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

Замените комментарий `<ADD A USER>` следующим фрагментом кода:

```java
//  The list of ChatParticipant to be added to the thread.
participants = new ArrayList<>();
// The display name for the thread participant.
displayName = "a new participant";
participants.add(new ChatParticipant().setId(second_user_id).setDisplayName(secondUserDisplayName));
// The model to pass to the add method.
AddChatParticipantsRequest addParticipantsRequest = new AddChatParticipantsRequest()
  .setParticipants(participants);

threadClient.addChatParticipants(threadId, addParticipantsRequest, new Callback<AddChatParticipantsResult>() {
        @Override
        public void onSuccess(AddChatParticipantsResult result, okhttp3.Response response) {
                // Take further action.
                Log.i(TAG, "add chat participants success");
        }
        
        @Override
        public void onFailure(Throwable throwable, okhttp3.Response response) {
                // Handle error.
                Log.e(TAG, throwable.getMessage());
        }
});
```

Замените `<second_user_id>` в классе идентификатором добавляемого пользователя Служб коммуникации. 

## <a name="list-users-in-a-thread"></a>Вывод списка пользователей в потоке

Замените комментарий `<LIST USERS>` следующим кодом:

```java
// The maximum number of participants to be returned per page, optional.
final int maxPageSize = 10;

// Skips participants up to a specified position in response.
final int skip = 0;

threadClient.listChatParticipantsPages(threadId,
    maxPageSize,
    skip,
    new Callback<AsyncPagedDataCollection<ChatParticipant, Page<ChatParticipant>>>() {
        @Override
        public void onSuccess(AsyncPagedDataCollection<ChatParticipant, Page<ChatParticipant>> pageCollection,
            okhttp3.Response response) {
                // pageCollection enables enumerating list of chat participants.
                pageCollection.getFirstPage(new Callback<Page<ChatParticipant>>() {
                    @Override
                    public void onSuccess(Page<ChatParticipant> firstPage, okhttp3.Response response) {
                        for (ChatParticipant participant : firstPage.getItems()) {
                            // Take further action.
                            Log.i(TAG, "participant: " + participant.getDisplayName());
                        }
                        listChatParticipantsNext(firstPage.getNextPageId(), pageCollection);
                    }

                    @Override
                    public void onFailure(Throwable throwable, okhttp3.Response response) {
                        // Handle error.
                        Log.e(TAG, throwable.getMessage());
                    }
                });

                @Override
                public void onFailure(Throwable throwable, okhttp3.Response response) {
                        // Handle error.
                        Log.e(TAG, throwable.getMessage());
                }
                });
        }

        @Override
        public void onFailure(Throwable throwable, okhttp3.Response response) {
                // Handle error.
                Log.e(TAG, throwable.getMessage());
        }
});
```

Вставьте следующий вспомогательный метод в класс:

```java
void listChatParticipantsNext(String nextLink, AsyncPagedDataCollection<ChatParticipant, Page<ChatParticipant>> pageCollection) {
    if (nextLink != null) {
        pageCollection.getPage(nextLink, new Callback<Page<ChatParticipant>>() {
            @Override
            public void onSuccess(Page<ChatParticipant> nextPage, Response response) {
                for (ChatParticipant participant : nextPage.getItems()) {
                    // Take further action.
                    Log.i(TAG, "participant: " + participant.getDisplayName());
                }
                if (nextPage.getPageId() != null) {
                    listChatParticipantsNext(nextPage.getPageId(), pageCollection);
                }
            }

            @Override
            public void onFailure(Throwable throwable, Response response) {
                Log.e(TAG, throwable.getMessage());
            }
        });
    }
}
```


## <a name="remove-user-from-a-chat-thread"></a>Удаление пользователя из потока чата

Обязательно замените `<second_user_id>` допустимым идентификатором пользователя. Сейчас мы удалим из потока второго пользователя.

Замените комментарий `<REMOVE A USER>` следующим кодом:

```java
threadClient.removeChatParticipant(threadId, second_user_id, new Callback<Void>() {
    @Override
    public void onSuccess(Void result, okhttp3.Response response) {
        // Take further action.
        Log.i(TAG, "remove a user successfully");
    }

    @Override
    public void onFailure(Throwable throwable, okhttp3.Response response) {
        // Handle error.
        Log.e(TAG, throwable.getMessage());
    }
});
```

## <a name="send-a-typing-notification"></a>Отправка уведомления о вводе

Замените комментарий `<SEND A TYPING NOTIFICATION>` следующим кодом:

```java
threadClient.sendTypingNotification(threadId, new Callback<Void>() {
    @Override
    public void onSuccess(Void result, Response response) {
        Log.i(TAG, "send a typing notification successfully");
    }

    @Override
    public void onFailure(Throwable throwable, Response response) {
        Log.e(TAG, throwable.getMessage());
    }
});
```

## <a name="send-a-read-receipt"></a>Отправка уведомления о прочтении

Обязательно замените `<chat_message_id>` допустимым идентификатором сообщения чата. Сейчас мы отправим уведомление о прочтении этого сообщения.

Замените комментарий `<SEND A READ RECEIPT>` следующим кодом:

```java
SendReadReceiptRequest readReceipt = new SendReadReceiptRequest()
    .setChatMessageId(chatMessageId);
threadClient.sendChatReadReceipt(threadId, readReceipt, new Callback<Void>() {
    @Override
    public void onSuccess(Void result, Response response) {
        Log.i(TAG, "send a read receipt successfully");
    }

    @Override
    public void onFailure(Throwable throwable, Response response) {
        Log.e(TAG, throwable.getMessage());
    }
});
```

## <a name="list-read-receipts"></a>Получение списка уведомлений о прочтении

Замените комментарий `<READ RECEIPTS>` следующим кодом:

```java
// The maximum number of participants to be returned per page, optional.
maxPageSize = 10;
// Skips participants up to a specified position in response.
skip = 0;
threadClient.listChatReadReceiptsPages(threadId,
    maxPageSize,
    skip,
    new Callback<AsyncPagedDataCollection<ChatMessageReadReceipt, Page<ChatMessageReadReceipt>>>() {
        @Override
        public void onSuccess(AsyncPagedDataCollection<ChatMessageReadReceipt, Page<ChatMessageReadReceipt>> pageCollection,
                              Response response) {
            // pageCollection enables enumerating list of chat participants.
            pageCollection.getFirstPage(new Callback<Page<ChatMessageReadReceipt>>() {
                @Override
                public void onSuccess(Page<ChatMessageReadReceipt> firstPage, Response response) {
                    for (ChatMessageReadReceipt receipt : firstPage.getItems()) {
                        Log.i(TAG, "receipt: " + receipt.getChatMessageId());
                    }
                    listChatReadReceiptsNext(firstPage.getNextPageId(), pageCollection);
                }

                @Override
                public void onFailure(Throwable throwable, Response response) {
                    Log.e(TAG, throwable.getMessage());
                }
            });
        }

        @Override
        public void onFailure(Throwable throwable, Response response) {
            Log.e(TAG, throwable.getMessage());
        }
});
```

Вставьте следующий вспомогательный метод в класс:
```java
void listChatReadReceiptsNext(String nextLink, AsyncPagedDataCollection<ChatMessageReadReceipt, Page<ChatMessageReadReceipt>> pageCollection) {
    if (nextLink != null) {
        pageCollection.getPage(nextLink, new Callback<Page<ChatMessageReadReceipt>>() {
            @Override
            public void onSuccess(Page<ChatMessageReadReceipt> nextPage, Response response) {
                for (ChatMessageReadReceipt receipt : nextPage.getItems()) {
                    Log.i(TAG, "receipt: " + receipt.getChatMessageId());
                }
                if (nextPage.getPageId() != null) {
                    listChatReadReceiptsNext(nextPage.getPageId(), pageCollection);
                }
            }

            @Override
            public void onFailure(Throwable throwable, Response response) {
                Log.e(TAG, throwable.getMessage());
            }
        });
    }
}

```


## <a name="run-the-code"></a>Выполнение кода

В Android Studio нажмите кнопку "Run" (Выполнить), чтобы выполнить сборку и запуск проекта. В консоли можно просмотреть выходные данные кода и средства ведения журнала из ChatClient.
