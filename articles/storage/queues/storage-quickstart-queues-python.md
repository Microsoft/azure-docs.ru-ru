---
title: Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 — Python
description: Узнайте, как использовать клиентскую библиотеку Хранилища очередей Azure версии 12 для Python, чтобы создавать очереди и добавлять в них сообщения, а также считывать и удалять сообщения из очереди. Также здесь описано, как удалить очередь.
author: twooley
ms.author: twooley
ms.date: 12/10/2019
ms.topic: quickstart
ms.service: storage
ms.subservice: queues
ms.custom:
- devx-track-python
- mode-api
ms.openlocfilehash: 68f68c32b160757e82d59b1dd5ee4847a06ec698
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107534351"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-python"></a>Краткое руководство. Клиентская библиотека Хранилища очередей Azure версии 12 для Python

Начните работу с клиентской библиотекой Хранилища очередей Azure версии 12 для Python. Хранилище очередей Azure — это служба хранения большого количества сообщений для последующего извлечения и обработки. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для выполнения базовых задач.

Клиентская библиотека Хранилища очередей Azure версии 12 для Python позволяет выполнять следующие задачи.

- Создание очереди
- Добавление сообщений в очередь
- Просмотр сообщений из очереди
- Обновление сообщений в очереди
- Получение сообщений из очереди
- Удаление сообщений из очереди
- Удаление очереди

Дополнительные ресурсы:

- [Справочная документация по API](/python/api/azure-storage-queue/index)
- [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-queue)
- [Пакет (индекс пакета Python)](https://pypi.org/project/azure-storage-queue/)
- [Примеры](../common/storage-samples-python.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- Учетная запись хранения Azure — [создайте такую учетную запись](../common/storage-account-create.md).
- [Python](https://www.python.org/downloads/) 2.7, 3.6 или более поздней версии для вашей операционной системы.

## <a name="setting-up"></a>Настройка

В этом разделе показано, как подготовить проект для работы с клиентской библиотекой Хранилища очередей Azure версии 12 для Python.

### <a name="create-the-project"></a>Создание проекта

Создайте приложение Python с именем `queues-quickstart-v12`.

1. В окне консоли (командная строка, PowerShell или Bash) создайте каталог для проекта.

    ```console
    mkdir queues-quickstart-v12
    ```

1. Переключитесь на только что созданный каталог `queues-quickstart-v12`.

    ```console
    cd queues-quickstart-v12
    ```

### <a name="install-the-package"></a>Установка пакета

Чтобы установить пакет клиентской библиотеки Хранилища очередей Azure для Python, выполните команду `pip install`.

```console
pip install azure-storage-queue
```

Эта команда устанавливает пакет клиентской библиотеки Хранилища очередей Azure для Python и все библиотеки, от которых она зависит. В нашем случае это только основная библиотека Azure для Python.

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

1. Откройте новый текстовый файл в редакторе кода.
1. Добавьте в него операторы `import`.
1. Создайте структуру программы, включая самую простую обработку исключений.

    Вот этот код:

    ```python
    import os, uuid
    from azure.storage.queue import QueueServiceClient, QueueClient, QueueMessage

    try:
        print("Azure Queue storage v12 - Python quickstart sample")
        # Quick start code goes here
    except Exception as ex:
        print('Exception:')
        print(ex)

    ```

1. Сохраните новый файл как `queues-quickstart-v12.py` в каталоге `queues-quickstart-v12`.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Объектная модель

Хранилище очередей Azure — это служба для хранения большого количества сообщений. Максимальный размер сообщения в очереди составляет 64 КБ. Очередь может содержать миллионы сообщений вплоть до лимита всей емкости учетной записи хранения. Очереди обычно используются для создания списка невыполненных заданий для асинхронной обработки. В Хранилище очередей предлагается три типа ресурсов:

- учетная запись хранения;
- очередь в учетной записи хранилища;
- сообщения в очереди.

На следующей схеме показана связь между этими ресурсами.

![Схема архитектуры Хранилища очередей](./media/storage-queues-introduction/queue1.png)

Используйте следующие классы Python для взаимодействия с этими ресурсами.

- [`QueueServiceClient`](/python/api/azure-storage-queue/azure.storage.queue.queueserviceclient). `QueueServiceClient` позволяет управлять всеми очередями в учетной записи хранения.
- [`QueueClient`](/python/api/azure-storage-queue/azure.storage.queue.queueclient). Класс `QueueClient` позволяет управлять отдельной очередью и сообщениями в ней.
- [`QueueMessage`](/python/api/azure-storage-queue/azure.storage.queue.queuemessage). Класс `QueueMessage` представляет отдельные объекты, которые возвращаются при вызове [`receive_messages`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#receive-messages---kwargs-) для очереди.

## <a name="code-examples"></a>Примеры кода

В этих примерах фрагментов кода показано, как выполнять следующие действия с помощью клиентской библиотеки Хранилища очередей Azure для Python.

- [Получение строки подключения](#get-the-connection-string)
- [Создание очереди](#create-a-queue)
- [Добавление сообщений в очередь](#add-messages-to-a-queue)
- [Просмотр сообщений из очереди](#peek-at-messages-in-a-queue)
- [Обновление сообщений в очереди](#update-a-message-in-a-queue)
- [Получение сообщений из очереди](#receive-messages-from-a-queue)
- [Удаление сообщений из очереди](#delete-messages-from-a-queue)
- [Удаление очереди](#delete-a-queue)

### <a name="get-the-connection-string"></a>Получение строки подключения

Приведенный ниже код извлекает строку подключения к учетной записи хранения. Эта строка подключения сохраняется в переменной среды, которую вы создали в разделе [Настройка строки подключения хранилища](#configure-your-storage-connection-string).

Добавьте этот код в блок `try`.

```python
    # Retrieve the connection string for use with the application. The storage
    # connection string is stored in an environment variable on the machine
    # running the application called AZURE_STORAGE_CONNECTION_STRING. If the
    # environment variable is created after the application is launched in a
    # console or with Visual Studio, the shell or application needs to be
    # closed and reloaded to take the environment variable into account.
    connect_str = os.getenv('AZURE_STORAGE_CONNECTION_STRING')
```

### <a name="create-a-queue"></a>Создание очереди

Выберите имя для новой очереди. Приведенный ниже код добавляет к имени очереди значение UUID, чтобы сделать это имя уникальным.

> [!IMPORTANT]
> Имя очереди может содержать только строчные буквы, цифры и дефисы и должно начинаться с буквы или цифры. Перед каждым дефисом должен быть знак без дефиса. Количество символов в имени должно быть от 3 до 63. Дополнительные сведения см. в статье [Именование очередей и метаданных](/rest/api/storageservices/naming-queues-and-metadata).

Создайте экземпляр класса [`QueueClient`](/python/api/azure-storage-queue/azure.storage.queue.queueclient). Затем вызовите метод [`create_queue`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#create-queue---kwargs-), чтобы создать очередь в своей учетной записи хранения.

Добавьте следующий код в конец блока `try`.

```python
    # Create a unique name for the queue
    queue_name = "quickstartqueues-" + str(uuid.uuid4())

    print("Creating queue: " + queue_name)

    # Instantiate a QueueClient which will be
    # used to create and manipulate the queue
    queue_client = QueueClient.from_connection_string(connect_str, queue_name)

    # Create the queue
    queue_client.create_queue()
```

### <a name="add-messages-to-a-queue"></a>Добавление сообщений в очередь

Следующий фрагмент кода добавляет сообщения в очередь, вызывая метод [`send_message`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#send-message-content----kwargs-). Он также сохраняет объект [`QueueMessage`](/python/api/azure-storage-queue/azure.storage.queue.queuemessage), полученный из третьего вызова `send_message`. Этот `saved_message` используется для обновления содержимого сообщения далее в коде программы.

Добавьте следующий код в конец блока `try`.

```python
    print("\nAdding messages to the queue...")

    # Send several messages to the queue
    queue_client.send_message(u"First message")
    queue_client.send_message(u"Second message")
    saved_message = queue_client.send_message(u"Third message")
```

### <a name="peek-at-messages-in-a-queue"></a>Просмотр сообщений из очереди

Чтобы просмотреть сообщения в очереди, вызовите метод [`peek_messages`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#peek-messages-max-messages-none----kwargs-). Этот метод извлекает одно или несколько сообщений из начала очереди, не изменяя видимость этих сообщений.

Добавьте следующий код в конец блока `try`.

```python
    print("\nPeek at the messages in the queue...")

    # Peek at messages in the queue
    peeked_messages = queue_client.peek_messages(max_messages=5)

    for peeked_message in peeked_messages:
        # Display the message
        print("Message: " + peeked_message.content)
```

### <a name="update-a-message-in-a-queue"></a>Обновление сообщений в очереди

Чтобы обновить содержимое сообщения, вызовите метод [`update_message`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#update-message-message--pop-receipt-none--content-none----kwargs-). Этот метод позволяет изменить время видимости сообщения и его содержимое. Содержимое сообщение должно иметь формат строки в кодировке UTF-8 длиной не более 64 КБ. Вместе с новым содержимым передайте значения из сообщения, сохраненного ранее в коде программы. Значения `saved_message` указывают, какое сообщение следует обновить.

```python
    print("\nUpdating the third message in the queue...")

    # Update a message using the message saved when calling send_message earlier
    queue_client.update_message(saved_message, pop_receipt=saved_message.pop_receipt, \
        content="Third message has been updated")
```

### <a name="receive-messages-from-a-queue"></a>Получение сообщений из очереди

Чтобы скачать ранее добавленные сообщения, вызовите метод [`receive_messages`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#receive-messages---kwargs-).

Добавьте следующий код в конец блока `try`.

```python
    print("\nReceiving messages from the queue...")

    # Get messages from the queue
    messages = queue_client.receive_messages(messages_per_page=5)
```

### <a name="delete-messages-from-a-queue"></a>Удаление сообщений из очереди

Удалите из очереди сообщения, которые уже получены и обработаны. В нашем примере обработка сводится к выводу сообщения в консоль.

Перед обработкой и удалением сообщений приложение ожидает ввода от пользователя, вызывая метод `input`. Перед удалением ресурсов убедитесь на [портале Azure](https://portal.azure.com), что они были правильно созданы. Все сообщения, которые не были удалены явным образом, снова становятся видимыми в очереди для новой попытки их обработать.

Добавьте следующий код в конец блока `try`.

```python
    print("\nPress Enter key to 'process' messages and delete them from the queue...")
    input()

    for msg_batch in messages.by_page():
            for msg in msg_batch:
                # "Process" the message
                print(msg.content)
                # Let the service know we're finished with
                # the message and it can be safely deleted.
                queue_client.delete_message(msg)
```

### <a name="delete-a-queue"></a>Удаление очереди

Следующий код очищает созданные приложением ресурсы, удаляя очередь с помощью метода [`delete_queue`](/python/api/azure-storage-queue/azure.storage.queue.queueclient#delete-queue---kwargs-).

Добавьте этот код в конец блока `try` и сохраните файл.

```python
    print("\nPress Enter key to delete the queue...")
    input()

    # Clean up
    print("Deleting queue...")
    queue_client.delete_queue()

    print("Done")
```

## <a name="run-the-code"></a>Выполнение кода

Это приложение создает три сообщения и добавляет их в очередь Azure. Затем код получает список сообщений, извлекает и удаляет их, и наконец удаляет саму очередь.

В окне консоли перейдите в каталог, который содержит файл `queues-quickstart-v12.py`, а затем запустите приложение, выполнив следующую команду `python`.

```console
python queues-quickstart-v12.py
```

Вы должны увидеть выходные данные приложения, как показано ниже.

```output
Azure Queue Storage client library v12 - Python quickstart sample
Creating queue: quickstartqueues-cac365be-7ce6-4065-bd65-3756ea052cb8

Adding messages to the queue...

Peek at the messages in the queue...
Message: First message
Message: Second message
Message: Third message

Updating the third message in the queue...

Receiving messages from the queue...

Press Enter key to 'process' messages and delete them from the queue...

First message
Second message
Third message has been updated

Press Enter key to delete the queue...

Deleting queue...
Done
```

Когда приложение приостановится перед получением сообщений, проверьте учетную запись хранения на [портале Azure](https://portal.azure.com). Убедитесь, что сообщения находятся в очереди.

Нажмите клавишу `Enter`, чтобы получить и удалить сообщения. При появлении запроса снова нажмите клавишу `Enter`, чтобы удалить очередь и завершить демонстрацию.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создавать очередь и добавлять в нее сообщения из кода на Python. Затем вы изучили процессы вставки, просмотра, получения и удаления сообщений. Наконец, вы узнали, как удалить очередь сообщений.

Руководства, примеры, краткие руководства и другую документацию можно найти здесь:

> [!div class="nextstepaction"]
> [Azure for Python developers](/azure/python/) (Azure для разработчиков Python).

- Чтобы узнать больше, ознакомьтесь с [библиотеками службы хранилища Azure для Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage).
- Дополнительные примеры приложений для клиентской библиотеки Хранилища очередей Azure версии 12 для Python см. [здесь](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-queue/samples).
