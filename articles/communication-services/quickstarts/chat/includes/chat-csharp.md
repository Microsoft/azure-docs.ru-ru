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
ms.openlocfilehash: 4c8bd66dde54ff90ea2191fba58f10c87c45cf68
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105958281"
---
## <a name="prerequisites"></a>Предварительные требования
Перед началом работы нужно сделать следующее:
- Создайте учетную запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- установить [Visual Studio](https://visualstudio.microsoft.com/downloads/);
- Создайте ресурс Служб коммуникации Azure. Дополнительные сведения см. в статье [Краткое руководство по созданию ресурсов Служб коммуникации и управлению ими](../../create-communication-resource.md). Для работы с этим кратким руководством необходимо записать **конечную точку**.
- [Маркер доступа пользователя](../../access-tokens.md). Обязательно задайте для области значение chat и запишите строку маркера, а также строку userId.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-c-application"></a>Создание нового приложения C#

В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `ChatQuickstart`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: **Program.cs**.

```console
dotnet new console -o ChatQuickstart
```

Измените каталог на только что созданную папку приложения и выполните команду `dotnet build`, чтобы скомпилировать приложение.

```console
cd ChatQuickstart
dotnet build
```

### <a name="install-the-package"></a>Установка пакета

Установка пакета SDK для чата Служб коммуникации Azure для .NET

```PowerShell
dotnet add package Azure.Communication.Chat --version 1.0.0
```

## <a name="object-model"></a>Объектная модель

Указанные ниже классы реализуют некоторые основные функции пакета SDK для чата Служб коммуникации Azure для C#.

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения участников, а также для отправки уведомлений о вводе и уведомления о прочтении. |

## <a name="create-a-chat-client"></a>Создание клиента чата

Чтобы создать клиент чата, вы будете использовать конечную точку Службы коммуникации и маркер доступа, который был создан в рамках предварительных требований. Чтобы создать пользователя и выдать маркер для передачи клиенту чата, необходимо использовать класс `CommunicationIdentityClient` из пакета SDK для удостоверений.

Дополнительные сведения о маркерах доступа пользователей см. [здесь](../../access-tokens.md).

В этом кратком руководстве не рассматривается создание уровня служб для управления маркерами для приложения чата, хотя это и рекомендуется. Дополнительные сведения об [архитектуре чатов](../../../concepts/chat/concepts.md)

Скопируйте приведенные ниже фрагменты кода и вставьте их в исходный файл **Program.CS**.
```csharp
using Azure;
using Azure.Communication;
using Azure.Communication.Chat;
using System;

namespace ChatQuickstart
{
    class Program
    {
        static async System.Threading.Tasks.Task Main(string[] args)
        {
            // Your unique Azure Communication service endpoint
            Uri endpoint = new Uri("https://<RESOURCE_NAME>.communication.azure.com");

            CommunicationTokenCredential communicationTokenCredential = new CommunicationTokenCredential(<Access_Token>);
            ChatClient chatClient = new ChatClient(endpoint, communicationTokenCredential);
        }
    }
}
```

## <a name="start-a-chat-thread"></a>Запуск потока чата

Для создания потока чата используйте метод `createChatThread` в ChatClient
- Для указания раздела в этом чате используйте `topic`. Раздел можно обновить после создания потока чата с помощью функции `UpdateTopic`.
- Используйте свойство `participants` для передачи списка объектов `ChatParticipant`, добавляемых в поток чата. Объект `ChatParticipant` инициализирован с помощью объекта `CommunicationIdentifier`. `CommunicationIdentifier` должен иметь тип `CommunicationUserIdentifier`, `MicrosoftTeamsUserIdentifier` или `PhoneNumberIdentifier`. Чтобы получить объект `CommunicationIdentifier`, необходимо передать идентификатор доступа, созданный с помощью приведенной инструкции для [создания пользователя](../../access-tokens.md#create-an-identity)

Объект ответа из метода `createChatThread` содержит подробные сведения о `chatThread`. Для взаимодействия с операциями потока чата, такими как добавление участников, отправка сообщения, удаление сообщения и т. д., необходимо создать экземпляр клиента `chatThreadClient` с помощью метода `GetChatThreadClient` в клиенте `ChatClient`.

```csharp
var chatParticipant = new ChatParticipant(identifier: new CommunicationUserIdentifier(id: "<Access_ID>"))
{
    DisplayName = "UserDisplayName"
};
CreateChatThreadResult createChatThreadResult = await chatClient.CreateChatThreadAsync(topic: "Hello world!", participants: new[] { chatParticipant });
ChatThreadClient chatThreadClient = chatClient.GetChatThreadClient(threadId: createChatThreadResult.ChatThread.Id);
string threadId = chatThreadClient.Id;
```

## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата
Метод `GetChatThreadClient` возвращает клиент потока для имеющегося потока. Его можно использовать для выполнения операций в созданном потоке: добавления участников, отправки сообщения и т. д. thread_id — уникальный идентификатор имеющегося потока чата.

```csharp
string threadId = "<THREAD_ID>";
ChatThreadClient chatThreadClient = chatClient.GetChatThreadClient(threadId: threadId);
```

## <a name="list-all-chat-threads"></a>Перечисление всех потоков чата
Чтобы получить все потоки чата, в которых участвует пользователь, используйте `GetChatThreads`.

```csharp
AsyncPageable<ChatThreadItem> chatThreadItems = chatClient.GetChatThreadsAsync();
await foreach (ChatThreadItem chatThreadItem in chatThreadItems)
{
    Console.WriteLine($"{ chatThreadItem.Id}");
}
```

## <a name="send-a-message-to-a-chat-thread"></a>Отправка сообщения в поток чата

Для отправки сообщения в поток используйте `SendMessage`.

- Используйте `content`, чтобы указать содержимое сообщения, если это необходимо.
- Используйте `type`, чтобы указать тип содержимого сообщения, например "Text" или "HTML". Если этот параметр не задан, используется тип "Text".
- Используйте `senderDisplayName`, чтобы указать отображаемое имя отправителя. Если этот параметр не задан, используется пустая строка.

```csharp
SendChatMessageResult sendChatMessageResult = await chatThreadClient.SendMessageAsync(content:"hello world", type: ChatMessageType.Text);
string messageId = sendChatMessageResult.Id;
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Получение сообщений из потока чата

Вы можете получать сообщения чата, выполняя опрос метода `GetMessages` на клиенте потока чата через определенные интервалы.

```csharp
AsyncPageable<ChatMessage> allMessages = chatThreadClient.GetMessagesAsync();
await foreach (ChatMessage message in allMessages)
{
    Console.WriteLine($"{message.Id}:{message.Content.Message}");
}
```

`GetMessages` принимает необязательный параметр `DateTimeOffset`. Если указано это смещение, то вы будете получать сообщения, которые были получены, обновлены или удалены после него. Обратите внимание, что сообщения, полученные до смещения времени, но исправлены или удалены после него, также будут возвращены.

`GetMessages` возвращает последнюю версию сообщения, включая все изменения или удаления, произошедшие с сообщением, с помощью `UpdateMessage` и `DeleteMessage`. Для удаленных сообщений `chatMessage.DeletedOn` возвращает значение даты и времени, указывающее, когда это сообщение было удалено. Для измененных сообщений `chatMessage.EditedOn` возвращает значение даты и времени, указывающее, когда это сообщение было изменено. Доступ к исходному времени создания сообщения можно получить с помощью `chatMessage.CreatedOn`, который также можно использовать для упорядочения сообщений.

`GetMessages` возвращает различные типы сообщений, которые могут быть идентифицированы с помощью `chatMessage.Type`. Это следующие типы:

- `Text`: обычное сообщение чата, отправляемое участником потока.

- `Html`: форматированное текстовое сообщение. Обратите внимание, что сейчас пользователи Служб коммуникации не могут отправить сообщения с форматированным текстом. Этот тип сообщений поддерживается в сообщениях, отправленных от пользователей Teams пользователям Служб коммуникации в сценариях взаимодействия с Teams.

- `TopicUpdated`: системное сообщение, указывающее на изменение темы. (только чтение)

- `ParticipantAdded`: системное сообщение о том, что один или несколько участников были добавлены в поток чата. (только чтение)

- `ParticipantRemoved`. Системное сообщение о том, что участник был удален из потока чата.

Дополнительные сведения см. в разделе о [типах сообщений](../../../concepts/chat/concepts.md#message-types).

## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

После создания потока можно добавлять и удалять пользователей. Добавляя пользователей, вы предоставляете им возможность отправлять сообщения в поток, а также добавлять или удалять других участников. Перед вызовом `AddParticipants` убедитесь, что вы получили новый маркер доступа и удостоверение для этого пользователя. Пользователю потребуется маркер доступа для инициализации своего клиента чата.

Используйте `AddParticipants` для добавления одного или нескольких участников в поток чата. Ниже приведены поддерживаемые атрибуты для каждого участника потока:
- `communicationUser` (обязательно) — это идентификатор участника потока.
- `displayName` (необязательно) — это отображаемое имя для участника потока.
- `shareHistoryTime` (необязательно) — это время, начиная с которого участнику предоставляется доступ к журналу чата.

```csharp
var josh = new CommunicationUserIdentifier(id: "<Access_ID_For_Josh>");
var gloria = new CommunicationUserIdentifier(id: "<Access_ID_For_Gloria>");
var amy = new CommunicationUserIdentifier(id: "<Access_ID_For_Amy>");

var participants = new[]
{
    new ChatParticipant(josh) { DisplayName = "Josh" },
    new ChatParticipant(gloria) { DisplayName = "Gloria" },
    new ChatParticipant(amy) { DisplayName = "Amy" }
};

await chatThreadClient.AddParticipantsAsync(participants: participants);
```

## <a name="get-thread-participants"></a>Получение участников потока

Используйте `GetParticipants` для получения участников потока чата.

```csharp
AsyncPageable<ChatParticipant> allParticipants = chatThreadClient.GetParticipantsAsync();
await foreach (ChatParticipant participant in allParticipants)
{
    Console.WriteLine($"{((CommunicationUserIdentifier)participant.User).Id}:{participant.DisplayName}:{participant.ShareHistoryTime}");
}
```

## <a name="send-read-receipt"></a>Отправка уведомления о прочтении

Используйте `SendReadReceipt`, чтобы уведомить других участников о том, что пользователь прочитал сообщение.

```csharp
await chatThreadClient.SendReadReceiptAsync(messageId: messageId);
```

## <a name="run-the-code"></a>Выполнение кода

Запустите приложение из каталога приложения с помощью команды `dotnet run`.

```console
dotnet run
```
