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
ms.openlocfilehash: 31704e705b828cc0070e3b79f5d527cfa9deb0c3
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107386860"
---
[!INCLUDE [Public Preview Notice](../../../includes/public-preview-include-chat.md)]

## <a name="prerequisites"></a>Предварительные требования
Перед началом работы нужно сделать следующее:

- Создайте учетную запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- установите [Python](https://www.python.org/downloads/);
- Создайте ресурс Служб коммуникации Azure. Дополнительные сведения см. в статье [Краткое руководство по созданию ресурсов Служб коммуникации и управлению ими](../../create-communication-resource.md). Для работы с этим кратким руководством необходимо записать **конечную точку**.
- [Маркер доступа пользователя](../../access-tokens.md). Обязательно задайте для области значение chat и запишите строку маркера, а также строку userId.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-python-application"></a>Создание приложения Python

Откройте терминал или командное окно, создайте каталог для своего приложения и перейдите к нему.

```console
mkdir chat-quickstart && cd chat-quickstart
```

Используйте текстовый редактор, чтобы создать файл с именем **start-chat.py** в корневом каталоге проекта и добавить структуру для программы, включая базовую обработку исключений. В следующих разделах показано, как добавить в этот файл весь исходный код для примера из этого краткого руководства.

```python
import os
# Add required SDK components from quickstart here

try:
    print('Azure Communication Services - Chat Quickstart')
    # Quickstart code goes here
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="install-sdk"></a>Установка пакета SDK

```console

pip install azure-communication-chat

```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK для чата Служб коммуникации Azure для Python.

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения пользователей, а также для отправки уведомлений о вводе и уведомления о прочтении. |

## <a name="create-a-chat-client"></a>Создание клиента чата

Чтобы создать клиент чата, вы будете использовать конечную точку службы коммуникации и `Access Token`, который был создан в рамках предварительных требований. Дополнительные сведения о маркерах доступа пользователей см. [здесь](../../access-tokens.md).

В этом кратком руководстве не рассматривается создание уровня служб для управления маркерами для приложения чата, хотя это и рекомендуется. Дополнительные сведения см. в этой документации об [архитектуре чатов](../../../concepts/chat/concepts.md)

```console
pip install azure-communication-identity
```

```python
from azure.communication.chat import ChatClient, CommunicationTokenCredential

endpoint = "https://<RESOURCE_NAME>.communication.azure.com"
chat_client = ChatClient(endpoint, CommunicationTokenCredential("<Access Token>"))
```

## <a name="start-a-chat-thread"></a>Запуск потока чата

Для создания потока чата используйте метод `create_chat_thread`.

- Для указания раздела в этом чате используйте `topic`. Раздел можно обновить после создания потока чата с помощью функции `update_thread`.
- Используйте `thread_participants`, чтобы получить список `ChatParticipant`, добавляемых в поток чата. `ChatParticipant` принимает объект `CommunicationUserIdentifier` типа `user`, который был получен после [создания пользователя](../../access-tokens.md#create-an-identity).

`CreateChatThreadResult` является результатом, возвращаемым при создании потока, который можно использовать для получения `id` созданного потока чата. Затем этот `id` можно использовать, чтобы получить объект `ChatThreadClient` с помощью метода `get_chat_thread_client`. `ChatThreadClient` можно использовать для выполнения других операций чата в этом потоке чата.

```python
topic="test topic"

create_chat_thread_result = chat_client.create_chat_thread(topic)
chat_thread_client = chat_client.get_chat_thread_client(create_chat_thread_result.chat_thread.id)
```

## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата
Метод `get_chat_thread_client` возвращает клиент потока для имеющегося потока. Его можно использовать для выполнения операций в созданном потоке: добавления участников, отправки сообщения и т. д. Идентификатор thread_id — это уникальный идентификатор имеющегося потока чата.

`ChatThreadClient` можно использовать для выполнения других операций чата в этом потоке чата.

```python
thread_id = create_chat_thread_result.chat_thread.id
chat_thread_client = chat_client.get_chat_thread_client(thread_id)
```


## <a name="list-all-chat-threads"></a>Перечисление всех потоков чата
Метод `list_chat_threads` возвращает итератор типа `ChatThreadItem`. Его можно использовать для перечисления всех потоков чата.

- Используйте `start_time` для указания самой ранней точки во времени, вплоть до которой будут получены потоки чата.
- Используйте `results_per_page`, чтобы указать максимальное число возвращаемых потоков чата для каждой страницы.

Итератор `[ChatThreadItem]` является ответом, возвращенным из списка потоков.

```python
from datetime import datetime, timedelta

start_time = datetime.utcnow() - timedelta(days=2)

chat_threads = chat_client.list_chat_threads(results_per_page=5, start_time=start_time)
for chat_thread_item_page in chat_threads.by_page():
    for chat_thread_item in chat_thread_item_page:
        print(chat_thread_item)
        print('Chat Thread Id: ', chat_thread_item.id)
```


## <a name="send-a-message-to-a-chat-thread"></a>Отправка сообщения в поток чата

Используйте метод `send_message` для отправки сообщения в созданный вами поток чата, идентифицируемый с помощью thread_id.

- Используйте `content`, чтобы указать содержимое сообщения в чате.
- Используйте `chat_message_type`, чтобы указать тип содержимого сообщения. Возможные значения: "Text" и "HTML". Если этот параметр не задан, по умолчанию используется значение "Text".
- Используйте `sender_display_name`, чтобы указать отображаемое имя отправителя.

`SendChatMessageResult` — ответ, возвращаемый при отправке сообщения. Содержит идентификатор, который является уникальным идентификатором сообщения.

```python
from azure.communication.chat import ChatMessageType

topic = "test topic"
create_chat_thread_result = chat_client.create_chat_thread(topic)
thread_id = create_chat_thread_result.chat_thread.id
chat_thread_client = chat_client.get_chat_thread_client(create_chat_thread_result.chat_thread.id)


content='hello world'
sender_display_name='sender name'

# specify chat message type with pre-built enumerations
send_message_result_w_enum = chat_thread_client.send_message(content=content, sender_display_name=sender_display_name, chat_message_type=ChatMessageType.TEXT)
print("Message sent: id: ", send_message_result_w_enum.id)
```


## <a name="receive-chat-messages-from-a-chat-thread"></a>Получение сообщений из потока чата

Кроме того, можно получать сообщения чата, выполняя опрос метода `list_messages` через определенные интервалы.

- Используйте `results_per_page`, чтобы указать максимальное число возвращаемых сообщений для каждой страницы.
- Используйте `start_time` для указания самой ранней точки во времени, вплоть до которой будут получены сообщения.

Итератор `[ChatMessage]` является ответом, возвращенным из списка сообщений.

```python
from datetime import datetime, timedelta

start_time = datetime.utcnow() - timedelta(days=1)

chat_messages = chat_thread_client.list_messages(results_per_page=1, start_time=start_time)
for chat_message_page in chat_messages.by_page():
    for chat_message in chat_message_page:
        print("ChatMessage: Id=", chat_message.id, "; Content=", chat_message.content.message)
```

`list_messages` возвращает последнюю версию сообщения, включая все изменения или удаления, произошедшие с сообщением, с помощью `update_message` и `delete_message`. Для удаленных сообщений `ChatMessage.deleted_on` возвращает значение даты и времени, указывающее, когда это сообщение было удалено. Для измененных сообщений `ChatMessage.edited_on` возвращает значение даты и времени, указывающее, когда это сообщение было изменено. Доступ к исходному времени создания сообщения можно получить с помощью `ChatMessage.created_on`, который можно использовать для упорядочения сообщений.

`list_messages` возвращает различные типы сообщений, которые могут быть идентифицированы с помощью `ChatMessage.type`. 

Дополнительные сведения о типах сообщений см. в разделе [Типы сообщений](../../../concepts/chat/concepts.md#message-types).

## <a name="send-read-receipt"></a>Отправка уведомления о прочтении
Метод `send_read_receipt` можно использовать для публикации события уведомления о прочтении в потоке от имени пользователя.

- Используйте `message_id`, чтобы указать идентификатор последнего сообщения, прочитанного текущим пользователем.

```python
content='hello world'

send_message_result = chat_thread_client.send_message(content)
chat_thread_client.send_read_receipt(message_id=send_message_result.id)
```


## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

После создания потока чата можно добавлять и удалять пользователей. Добавляя пользователей, вы предоставляете им возможность отправлять сообщения в поток чата, а также добавлять или удалять других участников. Перед вызовом метода `add_participants` убедитесь, что вы получили новый маркер доступа и удостоверение для этого пользователя. Пользователю потребуется маркер доступа для инициализации своего клиента чата.

В поток чата также можно добавить одного или нескольких пользователей с помощью метода `add_participants`, если для всех пользователей доступен новый маркер доступа и идентификатор.

возвращается `list(tuple(ChatParticipant, CommunicationError))`; Если участник успешно добавлен, ожидается пустой список. Если произошла ошибка при добавлении участника, список заполняется сбойными участниками вместе с возникшей ошибкой.

```python
from azure.communication.identity import CommunicationIdentityClient
from azure.communication.chat import ChatParticipant
from datetime import datetime

# create 2 users
identity_client = CommunicationIdentityClient.from_connection_string('<connection_string>')
new_users = [identity_client.create_user() for i in range(2)]

# # conversely, you can also add an existing user to a chat thread; provided the user_id is known
# from azure.communication.identity import CommunicationUserIdentifier
#
# user_id = 'some user id'
# user_display_name = "Wilma Flinstone"
# new_user = CommunicationUserIdentifier(user_id)
# participant = ChatParticipant(
#     user=new_user,
#     display_name=user_display_name,
#     share_history_time=datetime.utcnow())

participants = []
for _user in new_users:
  chat_thread_participant = ChatParticipant(
    user=_user,
    display_name='Fred Flinstone',
    share_history_time=datetime.utcnow()
  ) 
  participants.append(chat_thread_participant) 

response = chat_thread_client.add_participants(participants)

def decide_to_retry(error, **kwargs):
    """
    Insert some custom logic to decide if retry is applicable based on error
    """
    return True

# verify if all users has been successfully added or not
# in case of partial failures, you can retry to add all the failed participants 
retry = [p for p, e in response if decide_to_retry(e)]
if retry:
    chat_thread_client.add_participants(retry)
```


## <a name="list-thread-participants-in-a-chat-thread"></a>Перечисление участников в беседе в чате

Аналогично тому, как выполняется добавление участников, вы можете перечислить их в потоке.

Используйте `list_participants`, чтобы получить участников потока.
- Используйте `results_per_page` (необязательно), чтобы указать максимальное число участников, возвращаемых на каждой странице.
- Используйте `skip` (необязательно), чтобы пропустить участников вплоть до указанной позиции в ответе.

Итератор `[ChatParticipant]` является ответом, возвращаемым из списка участников.

```python
chat_thread_participants = chat_thread_client.list_participants()
for chat_thread_participant_page in chat_thread_participants.by_page():
    for chat_thread_participant in chat_thread_participant_page:
        print("ChatParticipant: ", chat_thread_participant)
```

## <a name="run-the-code"></a>Выполнение кода

Запустите приложение из каталога приложения с помощью команды `python`.

```console
python start-chat.py
```
