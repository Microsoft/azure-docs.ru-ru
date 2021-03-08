---
title: включить файл
description: включить файл
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 9/1/2020
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: 0225c948fddf65b9312c689144ecc567a70aa27e
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101750490"
---
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
# Add required client library components from quickstart here

try:
    print('Azure Communication Services - Chat Quickstart')
    # Quickstart code goes here
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="install-client-library"></a>Установка клиентской библиотеки

```console

pip install azure-communication-chat

```

## <a name="object-model"></a>Объектная модель

Следующие классы и интерфейсы реализуют некоторые основные функции клиентской библиотеки чата Служб коммуникации для Python.

| Имя                                  | Описание                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ChatClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения пользователей, а также для отправки уведомлений о вводе и уведомления о прочтении. |

## <a name="create-a-chat-client"></a>Создание клиента чата

Чтобы создать клиент чата, вы будете использовать конечную точку службы коммуникации и `Access Token`, который был создан в рамках предварительных требований. Дополнительные сведения о маркерах доступа пользователей см. [здесь](../../access-tokens.md).

В этом кратком руководстве не рассматривается создание уровня служб для управления маркерами для приложения чата, хотя это и рекомендуется. Дополнительные сведения см. в этой документации об [архитектуре чатов](../../../concepts/chat/concepts.md)

```console
pip install azure-communication-administration
```

```python
from azure.communication.chat import ChatClient, CommunicationTokenCredential, CommunicationTokenRefreshOptions

endpoint = "https://<RESOURCE_NAME>.communication.azure.com"
refresh_options = CommunicationTokenRefreshOptions(<Access Token>)
chat_client = ChatClient(endpoint, CommunicationTokenCredential(refresh_options))
```

## <a name="start-a-chat-thread"></a>Запуск потока чата

Для создания потока чата используйте метод `create_chat_thread`.

- Для указания раздела в этом чате используйте `topic`. Раздел можно обновить после создания потока чата с помощью функции `update_thread`.
- Используйте `thread_participants`, чтобы получить список `ChatThreadParticipant`, добавляемых в поток чата. `ChatThreadParticipant` принимает объект `CommunicationUserIdentifier` типа `user`, который был получен после [создания пользователя](../../access-tokens.md#create-an-identity).
- Используйте `repeatability_request_id`, чтобы указать повторяемость запроса. Клиент может сделать запрос несколько раз с одним и тем же Repeatability-Request-ID и получить соответствующий ответ, при этом серверу не нужно выполнять запрос несколько раз.

Ответ `chat_thread_client` используется для выполнения операций с созданным потоком чата, например для добавления участников в поток, отправки, удаления сообщения и т. д. Он содержит свойство `thread_id`, которое является уникальным идентификатором потока чата.

#### <a name="without-repeatability-request-id"></a>Без идентификатора Repeatability-Request-ID
```python
from datetime import datetime
from azure.communication.chat import ChatThreadParticipant

topic="test topic"
participants = [ChatThreadParticipant(
    user=user,
    display_name='name',
    share_history_time=datetime.utcnow()
)]

chat_thread_client = chat_client.create_chat_thread(topic, participants)
```

#### <a name="with-repeatability-request-id"></a>С идентификатором Repeatability-Request-ID
```python
from datetime import datetime
from azure.communication.chat import ChatThreadParticipant

topic="test topic"
participants = [ChatThreadParticipant(
    user=user,
    display_name='name',
    share_history_time=datetime.utcnow()
)]

repeatability_request_id = 'b66d6031-fdcc-41df-8306-e524c9f226b8' # some unique identifier
chat_thread_client = chat_client.create_chat_thread(topic, participants, repeatability_request_id)
```

## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата
Метод `get_chat_thread_client` возвращает клиент потока для имеющегося потока. Его можно использовать для выполнения операций в созданном потоке: добавления участников, отправки сообщения и т. д. Идентификатор thread_id — это уникальный идентификатор имеющегося потока чата.

```python
thread_id = chat_thread_client.thread_id
chat_thread_client = chat_client.get_chat_thread_client(thread_id)
```

## <a name="list-all-chat-threads"></a>Перечисление всех потоков чата
Метод `list_chat_threads` возвращает итератор типа `ChatThreadInfo`. Его можно использовать для перечисления всех потоков чата.

- Используйте `start_time` для указания самой ранней точки во времени, вплоть до которой будут получены потоки чата.
- Используйте `results_per_page`, чтобы указать максимальное число возвращаемых потоков чата для каждой страницы.

```python
from datetime import datetime, timedelta
import pytz

start_time = datetime.utcnow() - timedelta(days=2)
start_time = start_time.replace(tzinfo=pytz.utc)
chat_thread_infos = chat_client.list_chat_threads(results_per_page=5, start_time=start_time)

for chat_thread_info_page in chat_thread_infos.by_page():
    for chat_thread_info in chat_thread_info_page:
        # Iterate over all chat threads
        print("thread id:", chat_thread_info.id)
```

## <a name="delete-a-chat-thread"></a>Удаление беседы в чате
Объект `delete_chat_thread` используется для удаления потока чата.

- Используйте `thread_id` для указания thread_id существующего потока чата, который необходимо удалить

```python
thread_id = chat_thread_client.thread_id
chat_client.delete_chat_thread(thread_id)
```

## <a name="send-a-message-to-a-chat-thread"></a>Отправка сообщения в поток чата

Используйте метод `send_message` для отправки сообщения в созданный вами поток чата, идентифицируемый с помощью thread_id.

- Используйте `content`, чтобы указать содержимое сообщения в чате.
- Используйте `chat_message_type`, чтобы указать тип содержимого сообщения. Возможные значения: "Text" и "HTML". Если этот параметр не задан, по умолчанию используется значение "Text".
- Используйте `sender_display_name`, чтобы указать отображаемое имя отправителя.

Ответ — это идентификатор типа `str`, то есть уникальный идентификатор сообщения.

#### <a name="message-type-not-specified"></a>Тип сообщения не указан
```python
chat_thread_client = chat_client.create_chat_thread(topic, participants)

content='hello world'
sender_display_name='sender name'

send_message_result_id = chat_thread_client.send_message(content=content, sender_display_name=sender_display_name)
```

#### <a name="message-type-specified"></a>Тип сообщения указан
```python
from azure.communication.chat import ChatMessageType

content='hello world'
sender_display_name='sender name'

# specify chat message type with pre-built enumerations
send_message_result_id_w_enum = chat_thread_client.send_message(content=content, sender_display_name=sender_display_name, chat_message_type=ChatMessageType.TEXT)

# specify chat message type as string
send_message_result_id_w_str = chat_thread_client.send_message(content=content, sender_display_name=sender_display_name, chat_message_type='text')
```

## <a name="get-a-specific-chat-message-from-a-chat-thread"></a>Получение определенного сообщения чата из потока чата
Функцию `get_message` можно использовать для получения определенного сообщения, идентифицируемого с помощью message_id

- Используйте `message_id`, чтобы указать идентификатор сообщения.

Ответ типа `ChatMessage` содержит все сведения, связанные с одним сообщением.

```python
message_id = send_message_result_id
chat_message = chat_thread_client.get_message(message_id)
```

## <a name="receive-chat-messages-from-a-chat-thread"></a>Получение сообщений из потока чата

Кроме того, можно получать сообщения чата, выполняя опрос метода `list_messages` через определенные интервалы.

- Используйте `results_per_page`, чтобы указать максимальное число возвращаемых сообщений для каждой страницы.
- Используйте `start_time` для указания самой ранней точки во времени, вплоть до которой будут получены сообщения.

```python
chat_messages = chat_thread_client.list_messages(results_per_page=1, start_time=start_time)
for chat_message_page in chat_messages.by_page():
    for chat_message in chat_message_page:
        print('ChatMessage: ', chat_message)
        print('ChatMessage: ', chat_message.content.message)
```

`list_messages` возвращает последнюю версию сообщения, включая все изменения или удаления, произошедшие с сообщением, с помощью `update_message` и `delete_message`. Для удаленных сообщений `ChatMessage.deleted_on` возвращает значение даты и времени, указывающее, когда это сообщение было удалено. Для измененных сообщений `ChatMessage.edited_on` возвращает значение даты и времени, указывающее, когда это сообщение было изменено. Доступ к исходному времени создания сообщения можно получить с помощью `ChatMessage.created_on`, который можно использовать для упорядочения сообщений.

`list_messages` возвращает различные типы сообщений, которые могут быть идентифицированы с помощью `ChatMessage.type`. Это следующие типы:

- `ChatMessageType.TEXT`. Обычное сообщение чата, отправленное участником потока.

- `ChatMessageType.HTML`. HTML-сообщение чата, отправленное участником потока.

- `ChatMessageType.TOPIC_UPDATED`: системное сообщение, указывающее на изменение темы.

- `ChatMessageType.PARTICIPANT_ADDED`. Системное сообщение о том, что один или несколько участников были добавлены в поток чата.

- `ChatMessageType.PARTICIPANT_REMOVED`. Системное сообщение о том, что участник был удален из потока чата.

Дополнительные сведения см. в разделе о [типах сообщений](../../../concepts/chat/concepts.md#message-types).

## <a name="update-topic-of-a-chat-thread"></a>Обновление темы потока чата
Можно обновить тему потока чата с помощью метода `update_topic`

```python
topic = "updated thread topic"
chat_thread_client.update_topic(topic=topic)
updated_topic = chat_client.get_chat_thread(chat_thread_client.thread_id).topic
print('Updated topic: ', updated_topic)
```

## <a name="update-a-message"></a>Обновление сообщения
Можно обновить содержимое существующего сообщения с помощью метода `update_message`, идентифицируемого с помощью message_id

- Используйте `message_id`, чтобы указать message_id
- Используйте `content`, чтобы указать новое содержимое сообщения

```python
content = 'Hello world!'
send_message_result_id = chat_thread_client.send_message(content=content, sender_display_name=sender_display_name)

content = 'Hello! I am updated content'
chat_thread_client.update_message(message_id=send_message_result_id, content=content)

chat_message = chat_thread_client.get_message(send_message_result_id)
print('Updated message content: ', chat_message.content.message)
```

## <a name="send-read-receipt-for-a-message"></a>Отправка уведомления о прочтении сообщения
Метод `send_read_receipt` можно использовать для публикации события уведомления о прочтении в потоке от имени пользователя.

- Используйте `message_id`, чтобы указать идентификатор последнего сообщения, прочитанного текущим пользователем.

```python
message_id=send_message_result_id
chat_thread_client.send_read_receipt(message_id=message_id)
```

## <a name="list-read-receipts-for-a-chat-thread"></a>Получение списка уведомлений о прочтении для потока чата
Метод `list_read_receipts` можно использовать для получения уведомлений о прочтении для потока.

- Используйте `results_per_page`, чтобы указать максимальное количество возвращаемых уведомлений о прочтении сообщений чата для каждой страницы.
- Используйте `skip`, чтобы указать пропуск уведомлений о прочтении сообщений чата вплоть до указанной позиции в ответе.

```python
read_receipts = chat_thread_client.list_read_receipts(results_per_page=2, skip=0)

for read_receipt_page in read_receipts.by_page():
    for read_receipt in read_receipt_page:
        print('ChatMessageReadReceipt: ', read_receipt)
```

## <a name="send-typing-notification"></a>Отправка уведомления о вводе
Метод `send_typing_notification` можно использовать для публикации события уведомления о вводе в потоке от имени пользователя.

```python
chat_thread_client.send_typing_notification()
```

## <a name="delete-message"></a>Удаление сообщения
Метод `delete_message` можно использовать для удаления сообщения, идентифицируемого с помощью message_id

- Используйте `message_id`, чтобы указать message_id

```python
message_id=send_message_result_id
chat_thread_client.delete_message(message_id=message_id)
```

## <a name="add-a-user-as-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

После создания потока чата можно добавлять и удалять пользователей. Добавляя пользователей, вы предоставляете им возможность отправлять сообщения в поток чата, а также добавлять или удалять других участников. Перед вызовом метода `add_participant` убедитесь, что вы получили новый маркер доступа и удостоверение для этого пользователя. Пользователю потребуется маркер доступа для инициализации своего клиента чата.

Используйте метод `add_participant`, чтобы добавить участников в поток, идентифицируемый с помощью thread_id.

- Используйте `thread_participant`, чтобы указать участника, который будет добавлен в поток чата;
- Свойство `user` (обязательное) — это значение `CommunicationUserIdentifier`, полученное `CommunicationIdentityClient` при [создании пользователя](../../access-tokens.md#create-an-identity).
- `display_name` (необязательно) — это отображаемое имя для участника потока.
- `share_history_time` (необязательно) — это время, в течение которого участнику предоставляется доступ к журналу чата. Чтобы предоставить общий доступ к журналу с момента запуска потока чата, установите для этого свойства любую дату, равную или меньше времени создания потока. Чтобы в журнале отображались только записи с момента добавления участника, задайте для свойства текущую дату. Чтобы предоставить общий доступ к части журнала, задайте для него промежуточную дату.

```python
new_user = identity_client.create_user()

from azure.communication.chat import ChatThreadParticipant
from datetime import datetime

new_chat_thread_participant = ChatThreadParticipant(
    user=new_user,
    display_name='name',
    share_history_time=datetime.utcnow())

chat_thread_client.add_participant(new_chat_thread_participant)
```

В поток чата также можно добавить несколько пользователей с помощью метода `add_participants`, если для всех пользователей доступен новый маркер доступа и идентификатор.

```python
from azure.communication.chat import ChatThreadParticipant
from datetime import datetime

new_chat_thread_participant = ChatThreadParticipant(
        user=self.new_user,
        display_name='name',
        share_history_time=datetime.utcnow())
thread_participants = [new_chat_thread_participant] # instead of passing a single participant, you can pass a list of participants
chat_thread_client.add_participants(thread_participants)
```


## <a name="remove-user-from-a-chat-thread"></a>Удаление пользователя из потока чата

Аналогично тому, как выполняется добавление участников, вы можете удалять их из потока. Чтобы удалить участника, отследите идентификаторы добавленных участников.

Используйте метод `remove_participant`, чтобы удалить участников из потока, идентифицируемого с помощью threadId.
- `user` — это объект `CommunicationUserIdentifier`, который будет удален из потока.

```python
chat_thread_client.remove_participant(new_user)
```

## <a name="run-the-code"></a>Выполнение кода

Запустите приложение из каталога приложения с помощью команды `python`.

```console
python start-chat.py
```
