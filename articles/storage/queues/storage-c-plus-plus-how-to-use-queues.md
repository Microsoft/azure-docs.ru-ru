---
title: Как использовать хранилище очередей (C++) в службе хранилища Azure
description: Узнайте, как использовать службу хранилища очередей в Azure. Примеры написаны на C++.
author: mhopkins-msft
ms.author: mhopkins
ms.reviewer: dineshm
ms.date: 07/16/2020
ms.topic: how-to
ms.service: storage
ms.subservice: queues
ms.openlocfilehash: 44d64c54049c02b6602f01b97effcc33b03dbcfe
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97591333"
---
# <a name="how-to-use-queue-storage-from-c"></a>Использование хранилища очередей из C++

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>Обзор

В этом руководство будет показано, как выполнять типичные сценарии с помощью службы хранилища очередей Azure. Примеры написаны на C++ и используют [клиентскую библиотеку службы хранилища Azure для C++](https://github.com/Azure/azure-storage-cpp/blob/master/README.md). Здесь описаны такие сценарии, как **вставка**, **просмотр**, **получение** и **удаление** сообщений очереди, а также **создание и удаление очередей**.

> [!NOTE]
> Это руководством предназначено для клиентской библиотеки службы хранилища Azure для C++ версии 1.0.0 и выше. Рекомендуемая версия — клиентская библиотека службы хранилища Azure v 2.2.0, доступная через [NuGet](https://www.nuget.org/packages/wastorage) или [GitHub](https://github.com/Azure/azure-storage-cpp/).

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-c-application"></a>Создание приложения на C++

В этом руководстве будут использоваться компоненты хранилища, которые могут выполняться в приложениях C++.

Для этого необходимо установить клиентскую библиотеку службы хранилища Azure для C++ и создать учетную запись хранения Azure в подписке Azure.

<!-- docutune:casing "Getting Started on Linux" -->

Чтобы установить клиентскую библиотеку службы хранилища Azure для C++, можно использовать следующие методы.

- **Linux:** Следуйте инструкциям, приведенным в [файле сведений о клиентской библиотеке хранилища Azure для C++ README: Начало работы на странице Linux](https://github.com/Azure/azure-storage-cpp#getting-started-on-linux) .
- **Windows:** В Windows в качестве диспетчера зависимостей используйте [vcpkg](https://github.com/microsoft/vcpkg). Выполните инициализацию с помощью [краткого руководства](https://github.com/microsoft/vcpkg#quick-start) `vcpkg` . Затем, чтобы установить библиотеку, используйте следующую команду:

```powershell
.\vcpkg.exe install azure-storage-cpp
```

Руководство по созданию исходного кода и экспорту в NuGet можно найти в файле [сведений](https://github.com/Azure/azure-storage-cpp#download--install) .

## <a name="configure-your-application-to-access-queue-storage"></a>Настройка приложения для доступа к хранилищу очередей

Добавьте следующие инструкции include в начало файла C++, где вы хотите использовать API-интерфейсы службы хранилища Azure для доступа к очередям:

```cpp
#include <was/storage_account.h>
#include <was/queue.h>
```

## <a name="set-up-an-azure-storage-connection-string"></a>Настройка строки подключения к службе хранилища Azure

Клиент службы хранилища Azure использует строку подключения с целью хранения конечных точек и учетных данных для доступа к службам управления данными. При запуске в клиентском приложении необходимо указать строку подключения к хранилищу в следующем формате, используя имя учетной записи хранения и ключ доступа к хранилищу для учетной записи хранения, указанной в [портал Azure](https://portal.azure.com) для `AccountName` `AccountKey` значений и. Сведения о учетных записях хранения и ключах доступа см. в статье [об учетных записях хранения Azure](../common/storage-account-create.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json). В этом примере показано, как объявить статическое поле для размещения строки подключения:

```cpp
// Define the connection-string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key"));
```

Чтобы протестировать приложение на локальном компьютере Windows, можно использовать [эмулятор хранения азурите](../common/storage-use-azurite.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json). Азурите — это служебная программа, имитирующая хранилище BLOB-объектов Azure и хранилище очередей на локальном компьютере разработки. В следующем примере показано, как объявить статическое поле для размещения строки подключения для эмулятора локального хранилища.

```cpp
// Define the connection-string with Azurite.
const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  
```

Сведения о запуске Азурите см. в статье [использование эмулятора азурите для локальной разработки службы хранилища Azure](../common/storage-use-azurite.md).

В приведенных ниже примерах предполагается, что вы использовали одно из этих двух определений для получения строки подключения к хранилищу.

## <a name="retrieve-your-connection-string"></a>Получить строку подключения

Информацию о своей учетной записи хранения можно представить с помощью класса `cloud_storage_account`. Чтобы получить сведения об учетной записи хранения из строки подключения к хранилищу, можно использовать `parse` метод.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);
```

## <a name="how-to-create-a-queue"></a>Практическое руководство. Создание очереди

`cloud_queue_client`Объект позволяет получать ссылочные объекты для очередей. Следующий код создает `cloud_queue_client` объект.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create a queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();
```

Используйте `cloud_queue_client` объект для получения ссылки на очередь, которую необходимо использовать. Очередь можно создать, если она не существует.

```cpp
// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Create the queue if it doesn't already exist.
queue.create_if_not_exists();  
```

## <a name="how-to-insert-a-message-into-a-queue"></a>Практическое руководство. Вставка сообщения в очередь

Чтобы вставить сообщение в существующую очередь, сначала создайте новый объект `cloud_queue_message` . Затем вызовите `add_message` метод. `cloud_queue_message`Можно создать либо из строки (в формате UTF-8), либо в массиве байтов. Ниже приведен код, который создает очередь (если она не существует) и вставляет сообщение `Hello, World` :

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Create the queue if it doesn't already exist.
queue.create_if_not_exists();

// Create a message and add it to the queue.
azure::storage::cloud_queue_message message1(U("Hello, World"));
queue.add_message(message1);  
```

## <a name="how-to-peek-at-the-next-message"></a>Практическое руководство. Просмотр следующего сообщения

Можно просмотреть сообщение в начале очереди, не удаляя его из очереди, вызвав `peek_message` метод.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Peek at the next message.
azure::storage::cloud_queue_message peeked_message = queue.peek_message();

// Output the message content.
std::wcout << U("Peeked message content: ") << peeked_message.content_as_string() << std::endl;
```

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Практическое руководство. Изменение содержимого сообщения в очереди

Вы можете изменить содержимое сообщения непосредственно в очереди. Если сообщение представляет собой рабочую задачу, можно использовать эту функцию для обновления состояния рабочей задачи. Следующий код добавляет новое содержимое в очередь сообщений и продлевает время ожидания видимости еще на 60 секунд. Это сохраняет состояние работы, связанной с данным сообщением, и позволяет клиенту продолжить работу с сообщением на протяжении еще одной минуты. Эту методику можно использовать для контроля многошаговых рабочих процессов в сообщениях очереди без необходимости запуска с начала, если шаг обработки завершается сбоем из-за сбоя оборудования или программного обеспечения. Обычно также сохраняется счетчик повторов; если количество повторов сообщения превысит n раз, его нужно удалить. Это обеспечивает защиту от сообщений, которые инициируют ошибку приложения при каждой попытке обработки.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Get the message from the queue and update the message contents.
// The visibility timeout "0" means make it visible immediately.
// The visibility timeout "60" means the client can get another minute to continue
// working on the message.
azure::storage::cloud_queue_message changed_message = queue.get_message();

changed_message.set_content(U("Changed message"));
queue.update_message(changed_message, std::chrono::seconds(60), true);

// Output the message content.
std::wcout << U("Changed message content: ") << changed_message.content_as_string() << std::endl;  
```

## <a name="how-to-dequeue-the-next-message"></a>Практическое руководство. Удаление следующего сообщения из очереди

Код удаляет сообщение из очереди в два этапа. При вызове `get_message` вы получаете следующее сообщение в очереди. Сообщение, возвращаемое методом `get_message`, становится невидимым для другого кода, считывающего сообщения из этой очереди. Чтобы завершить удаление сообщения из очереди, необходимо также вызвать метод `delete_message` . Этот двухэтапный процесс удаления сообщения позволяет удостовериться, что если коду не удастся обработать сообщение из-за сбоя оборудования или программного обеспечения, другой экземпляр кода сможет получить то же сообщение и повторить попытку. Код вызывается `delete_message` сразу после обработки сообщения.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Get the next message.
azure::storage::cloud_queue_message dequeued_message = queue.get_message();
std::wcout << U("Dequeued message: ") << dequeued_message.content_as_string() << std::endl;

// Delete the message.
queue.delete_message(dequeued_message);
```

## <a name="how-to-use-additional-options-for-dequeuing-messages"></a>Как использовать дополнительные параметры для удаления сообщений из очереди

Способ извлечения сообщения из очереди можно настроить двумя способами. Во-первых, можно получить пакет сообщений (до 32 сообщений). Во-вторых, можно задать более длительное или короткое время ожидания видимости, чтобы предоставить коду больше или меньше времени на полную обработку каждого сообщения. В следующем примере кода метод `get_messages` используется для получения 20 сообщений в одном вызове. Затем он обрабатывает каждое сообщение с помощью цикла `for`. Он также задает время ожидания невидимости 5 минут для каждого сообщения. Обратите внимание, что пять минут начинается для всех сообщений в одно и то же время, поэтому по истечении пяти минут после вызова функции `get_messages` все сообщения, которые не были удалены, снова станут видимыми.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Dequeue some queue messages (maximum 32 at a time) and set their visibility timeout to
// 5 minutes (300 seconds).
azure::storage::queue_request_options options;
azure::storage::operation_context context;

// Retrieve 20 messages from the queue with a visibility timeout of 300 seconds.
std::vector<azure::storage::cloud_queue_message> messages = queue.get_messages(20, std::chrono::seconds(300), options, context);

for (auto it = messages.cbegin(); it != messages.cend(); ++it)
{
    // Display the contents of the message.
    std::wcout << U("Get: ") << it->content_as_string() << std::endl;
}
```

## <a name="how-to-get-the-queue-length"></a>Практическое руководство. Получение длины очереди

Вы можете узнать приблизительное количество сообщений в очереди. `download_attributes`Метод возвращает свойства очереди, включая число сообщений. `approximate_message_count`Метод получает приблизительное количество сообщений в очереди.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Fetch the queue attributes.
queue.download_attributes();

// Retrieve the cached approximate message count.
int cachedMessageCount = queue.approximate_message_count();

// Display number of messages.
std::wcout << U("Number of messages in queue: ") << cachedMessageCount << std::endl;  
```

## <a name="how-to-delete-a-queue"></a>Практическое руководство. Удаление очереди

Чтобы удалить очередь и все сообщения, содержащиеся в ней, вызовите `delete_queue_if_exists` метод для объекта очереди.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// If the queue exists and delete it.
queue.delete_queue_if_exists();  
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы узнали основные сведения о хранилище очередей, перейдите по следующим ссылкам, чтобы узнать больше о службе хранилища Azure.

- [Использование хранилища BLOB-объектов из C++](../blobs/storage-c-plus-plus-how-to-use-blobs.md)
- [Использование табличного хранилища из C++](../../cosmos-db/table-storage-how-to-use-c-plus.md)
- [Перечисление ресурсов хранилища Azure в C++](../common/storage-c-plus-plus-enumeration.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)
- [Справочник по клиентской библиотеке службы хранилища Azure для C++](https://azure.github.io/azure-storage-cpp)
- [Документация по службе хранилища Azure](https://azure.microsoft.com/documentation/services/storage/)
