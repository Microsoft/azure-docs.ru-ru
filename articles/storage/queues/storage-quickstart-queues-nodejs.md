---
title: Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 — JavaScript
description: Узнайте, как использовать клиентскую библиотеку Хранилища очередей Azure версии 12 для JavaScript, чтобы создавать очереди и добавлять в них сообщения, а также считывать и удалять сообщения из очереди. Также здесь описано, как удалить очередь.
author: twooley
ms.author: twooley
ms.date: 12/13/2019
ms.topic: quickstart
ms.service: storage
ms.subservice: queues
ms.custom:
- devx-track-js
- mode-api
ms.openlocfilehash: 943678debc32116ff7a2e54810c4edcf2d331bd6
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/16/2021
ms.locfileid: "107534432"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-javascript"></a>Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 для JavaScript

Приступите к работе с клиентской библиотекой Хранилища очередей Azure версии 12 для JavaScript. Хранилище очередей Azure — это служба хранения большого количества сообщений для последующего извлечения и обработки. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для выполнения базовых задач.

Клиентскую библиотеку Хранилища очередей Azure версии 12 для JavaScript можно использовать для выполнения следующих задач.

- Создание очереди
- Добавление сообщений в очередь
- Просмотр сообщений из очереди
- Обновление сообщений в очереди
- Получение сообщений из очереди
- Удаление сообщений из очереди
- Удаление очереди

Дополнительные ресурсы:

- [Справочная документация по API](/javascript/api/@azure/storage-queue/)
- [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-queue)
- [Пакет (npm)](https://www.npmjs.com/package/@azure/storage-queue)
- [Примеры](../common/storage-samples-javascript.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- Учетная запись хранения Azure — [создайте такую учетную запись](../common/storage-account-create.md).
- Последняя версия [Node.js](https://nodejs.org/en/download/) для вашей операционной системы.

## <a name="setting-up"></a>Настройка

В этом разделе рассматривается подготовка проекта для работы с клиентской библиотекой Хранилища очередей Azure версии 12 для JavaScript.

### <a name="create-the-project"></a>Создание проекта

Создайте приложение Node.js с именем `queues-quickstart-v12`.

1. В окне консоли (командная строка, PowerShell или Bash) создайте каталог для проекта.

    ```console
    mkdir queues-quickstart-v12
    ```

1. Переключитесь на только что созданный каталог `queues-quickstart-v12`.

    ```console
    cd queues-quickstart-v12
    ```

1. Создайте текстовый файл `package.json`. Этот файл определяет проект Node.js. Сохраните этот файл в каталоге `queues-quickstart-v12`. Вот содержимое этого файла:

    ```json
    {
        "name": "queues-quickstart-v12",
        "version": "1.0.0",
        "description": "Use the @azure/storage-queue SDK version 12 to interact with Azure Queue storage",
        "main": "queues-quickstart-v12.js",
        "scripts": {
            "start": "node queues-quickstart-v12.js"
        },
        "author": "Your Name",
        "license": "MIT",
        "dependencies": {
            "@azure/storage-queue": "^12.0.0",
            "@types/dotenv": "^4.0.3",
            "dotenv": "^6.0.0"
        }
    }
    ```

    Вы можете указать собственное имя в поле `author`, если хотите.

### <a name="install-the-package"></a>Установка пакета

Оставаясь в каталоге `queues-quickstart-v12`, установите пакет клиентской библиотеки Хранилища очередей Azure для JavaScript с помощью команды `npm install`.

```console
npm install
```

Эта команда считывает файл `package.json` и устанавливает пакет клиентской библиотеки Хранилища очередей Azure версии 12 для JavaScript и все библиотеки, от которых она зависит.

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Откройте другой новый текстовый файл в редакторе кода.
1. Добавьте в него вызовы `require` для загрузки модулей Azure и Node.js.
1. Создайте структуру программы, включая самую простую обработку исключений.

    Вот этот код:

    ```javascript
    const { QueueClient } = require("@azure/storage-queue");
    const uuidv1 = require("uuid/v1");

    async function main() {
        console.log("Azure Queue Storage client library v12 - JavaScript quickstart sample");
        // Quick start code goes here
    }

    main().then(() => console.log("\nDone")).catch((ex) => console.log(ex.message));

    ```

1. Сохраните новый файл как `queues-quickstart-v12.js` в каталоге `queues-quickstart-v12`.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Объектная модель

Хранилище очередей Azure — это служба для хранения большого количества сообщений. Максимальный размер сообщения в очереди составляет 64 КБ. Очередь может содержать миллионы сообщений вплоть до лимита всей емкости учетной записи хранения. Очереди обычно используются для создания списка невыполненных заданий для асинхронной обработки. В Хранилище очередей предлагается три типа ресурсов:

- учетная запись хранения;
- очередь в учетной записи хранилища;
- сообщения в очереди.

На следующей схеме показана связь между этими ресурсами.

![Схема архитектуры Хранилища очередей](./media/storage-queues-introduction/queue1.png)

Используйте следующие классы JavaScript для взаимодействия с этими ресурсами.

- [`QueueServiceClient`](/javascript/api/@azure/storage-queue/queueserviceclient). `QueueServiceClient` позволяет управлять всеми очередями в учетной записи хранения.
- [`QueueClient`](/javascript/api/@azure/storage-queue/queueclient). Класс `QueueClient` позволяет управлять отдельной очередью и сообщениями в ней.
- [`QueueMessage`](/javascript/api/@azure/storage-queue/queuemessage). Класс `QueueMessage` представляет отдельные объекты, которые возвращаются при вызове [`ReceiveMessages`](/javascript/api/@azure/storage-queue/queueclient#receivemessages-queuereceivemessageoptions-) для очереди.

## <a name="code-examples"></a>Примеры кода

В этих примерах фрагментов кода показано, как выполнять следующие действия с помощью клиентской библиотеки Хранилища очередей Azure для JavaScript.

- [Получение строки подключения](#get-the-connection-string)
- [Создание очереди](#create-a-queue)
- [Добавление сообщений в очередь](#add-messages-to-a-queue)
- [Просмотр сообщений из очереди](#peek-at-messages-in-a-queue)
- [Обновление сообщений в очереди](#update-a-message-in-a-queue)
- [Получение сообщений из очереди](#receive-messages-from-a-queue)
- [Удаление сообщений из очереди](#delete-messages-from-a-queue)
- [Удаление очереди](#delete-a-queue)

### <a name="get-the-connection-string"></a>Получение строки подключения

Приведенный ниже код извлекает строку подключения для учетной записи хранения из переменной среды, созданной при работе с разделом [Настройка строки подключения хранилища](#configure-your-storage-connection-string).

Добавьте этот код в функцию `main`.

```javascript
// Retrieve the connection string for use with the application. The storage
// connection string is stored in an environment variable on the machine
// running the application called AZURE_STORAGE_CONNECTION_STRING. If the
// environment variable is created after the application is launched in a
// console or with Visual Studio, the shell or application needs to be
// closed and reloaded to take the environment variable into account.
const AZURE_STORAGE_CONNECTION_STRING = process.env.AZURE_STORAGE_CONNECTION_STRING;
```

### <a name="create-a-queue"></a>Создание очереди

Выберите имя для новой очереди. Приведенный ниже код добавляет к имени очереди значение UUID, чтобы сделать это имя уникальным.

> [!IMPORTANT]
> Имя очереди может содержать только строчные буквы, цифры и дефисы и должно начинаться с буквы или цифры. Перед каждым дефисом должен быть знак без дефиса. Количество символов в имени должно быть от 3 до 63. Дополнительные сведения см. в статье [Именование очередей и метаданных](/rest/api/storageservices/naming-queues-and-metadata).

Создайте экземпляр класса [`QueueClient`](/javascript/api/@azure/storage-queue/queueclient). Затем вызовите метод [`create`](/javascript/api/@azure/storage-queue/queueclient#create-queuecreateoptions-), чтобы создать очередь в своей учетной записи хранения.

Добавьте следующий код в конец функции `main`.

```javascript
// Create a unique name for the queue
const queueName = "quickstart" + uuidv1();

console.log("\nCreating queue...");
console.log("\t", queueName);

// Instantiate a QueueClient which will be used to create and manipulate a queue
const queueClient = new QueueClient(AZURE_STORAGE_CONNECTION_STRING, queueName);

// Create the queue
const createQueueResponse = await queueClient.create();
console.log("Queue created, requestId:", createQueueResponse.requestId);
```

### <a name="add-messages-to-a-queue"></a>Добавление сообщений в очередь

Следующий фрагмент кода добавляет сообщения в очередь, вызывая метод [`sendMessage`](/javascript/api/@azure/storage-queue/queueclient#sendmessage-string--queuesendmessageoptions-). Он также сохраняет объект [`QueueMessage`](/javascript/api/@azure/storage-queue/queuemessage), полученный из третьего вызова `sendMessage`. Возвращенный `sendMessageResponse` используется для обновления содержимого сообщения далее в коде программы.

Добавьте следующий код в конец функции `main`.

```javascript
console.log("\nAdding messages to the queue...");

// Send several messages to the queue
await queueClient.sendMessage("First message");
await queueClient.sendMessage("Second message");
const sendMessageResponse = await queueClient.sendMessage("Third message");

console.log("Messages added, requestId:", sendMessageResponse.requestId);
```

### <a name="peek-at-messages-in-a-queue"></a>Просмотр сообщений из очереди

Чтобы просмотреть сообщения в очереди, вызовите метод [`peekMessages`](/javascript/api/@azure/storage-queue/queueclient#peekmessages-queuepeekmessagesoptions-). Этот метод извлекает одно или несколько сообщений из начала очереди, не изменяя видимость этих сообщений.

Добавьте следующий код в конец функции `main`.

```javascript
console.log("\nPeek at the messages in the queue...");

// Peek at messages in the queue
const peekedMessages = await queueClient.peekMessages({ numberOfMessages : 5 });

for (i = 0; i < peekedMessages.peekedMessageItems.length; i++) {
    // Display the peeked message
    console.log("\t", peekedMessages.peekedMessageItems[i].messageText);
}
```

### <a name="update-a-message-in-a-queue"></a>Обновление сообщений в очереди

Чтобы обновить содержимое сообщения, вызовите метод [`updateMessage`](/javascript/api/@azure/storage-queue/queueclient#updatemessage-string--string--string--undefined---number--queueupdatemessageoptions-). Этот метод позволяет изменить время видимости сообщения и его содержимое. Содержимое сообщение должно иметь формат строки в кодировке UTF-8 длиной не более 64 КБ. Вместе с новым содержимым передайте `messageId` и `popReceipt` из ответа, сохраненного ранее в коде программы. Свойства `sendMessageResponse` указывают, какое сообщение следует обновить.

```javascript
console.log("\nUpdating the third message in the queue...");

// Update a message using the response saved when calling sendMessage earlier
updateMessageResponse = await queueClient.updateMessage(
    sendMessageResponse.messageId,
    sendMessageResponse.popReceipt,
    "Third message has been updated"
);

console.log("Message updated, requestId:", updateMessageResponse.requestId);
```

### <a name="receive-messages-from-a-queue"></a>Получение сообщений из очереди

Чтобы скачать ранее добавленные сообщения, вызовите метод [`receiveMessages`](/javascript/api/@azure/storage-queue/queueclient#receivemessages-queuereceivemessageoptions-). В поле `numberOfMessages` передайте максимальное число получаемых сообщений для этого вызова.

Добавьте следующий код в конец функции `main`.

```javascript
console.log("\nReceiving messages from the queue...");

// Get messages from the queue
const receivedMessagesResponse = await queueClient.receiveMessages({ numberOfMessages : 5 });

console.log("Messages received, requestId:", receivedMessagesResponse.requestId);
```

### <a name="delete-messages-from-a-queue"></a>Удаление сообщений из очереди

Удалите из очереди сообщения, которые уже получены и обработаны. В нашем примере обработка сводится к выводу сообщения в консоль.

Чтобы удалить сообщения, вызовите метод [`deleteMessage`](/javascript/api/@azure/storage-queue/queueclient#deletemessage-string--string--queuedeletemessageoptions-). Все сообщения, которые не были удалены явным образом, снова становятся видимыми в очереди для новой попытки их обработать.

Добавьте следующий код в конец функции `main`.

```javascript
// 'Process' and delete messages from the queue
for (i = 0; i < receivedMessagesResponse.receivedMessageItems.length; i++) {
    receivedMessage = receivedMessagesResponse.receivedMessageItems[i];

    // 'Process' the message
    console.log("\tProcessing:", receivedMessage.messageText);

    // Delete the message
    const deleteMessageResponse = await queueClient.deleteMessage(
        receivedMessage.messageId,
        receivedMessage.popReceipt
    );
    console.log("\tMessage deleted, requestId:", deleteMessageResponse.requestId);
}
```

### <a name="delete-a-queue"></a>Удаление очереди

Следующий код очищает созданные приложением ресурсы, удаляя очередь с помощью метода [`delete`](/javascript/api/@azure/storage-queue/queueclient#delete-queuedeleteoptions-).

Добавьте этот код в конец функции `main` и сохраните файл.

```javascript
// Delete the queue
console.log("\nDeleting queue...");
const deleteQueueResponse = await queueClient.delete();
console.log("Queue deleted, requestId:", deleteQueueResponse.requestId);
```

## <a name="run-the-code"></a>Выполнение кода

Это приложение создает три сообщения и добавляет их в очередь Azure. Затем код получает список сообщений, извлекает и удаляет их, и наконец удаляет саму очередь.

В окне консоли перейдите в каталог, который содержит файл `queues-quickstart-v12.js`, а затем запустите приложение, выполнив следующую команду `node`.

```console
node queues-quickstart-v12.js
```

Вы должны увидеть выходные данные приложения, как показано ниже.

```output
Azure Queue Storage client library v12 - JavaScript quickstart sample

Creating queue...
         quickstartc095d120-1d04-11ea-af30-090ee231305f
Queue created, requestId: 5c0bc94c-6003-011b-7c11-b13d06000000

Adding messages to the queue...
Messages added, requestId: a0390321-8003-001e-0311-b18f2c000000

Peek at the messages in the queue...
         First message
         Second message
         Third message

Updating the third message in the queue...
Message updated, requestId: cb172c9a-5003-001c-2911-b18dd6000000

Receiving messages from the queue...
Messages received, requestId: a039036f-8003-001e-4811-b18f2c000000
        Processing: First message
        Message deleted, requestId: 4a65b82b-d003-00a7-5411-b16c22000000
        Processing: Second message
        Message deleted, requestId: 4f0b2958-c003-0030-2a11-b10feb000000
        Processing: Third message has been updated
        Message deleted, requestId: 6c978fcb-5003-00b6-2711-b15b39000000

Deleting queue...
Queue deleted, requestId: 5c0bca05-6003-011b-1e11-b13d06000000

Done
```

Пошагово выполните код в отладчике и просматривайте результаты на [портале Azure](https://portal.azure.com). Проверьте учетную запись хранения и убедитесь, что сообщения в очереди создаются и удаляются.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создавать очередь и добавлять в нее сообщения из кода на JavaScript. Затем вы изучили процессы вставки, просмотра, получения и удаления сообщений. Наконец, вы узнали, как удалить очередь сообщений.

Руководства, примеры, краткие руководства и другую документацию можно найти здесь:

> [!div class="nextstepaction"]
> [Документация по работе с Azure для JavaScript](/azure/developer/javascript/).

- Дополнительные сведения см. в документации по [клиентской библиотеке Хранилища очередей Azure для JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-queue).
- Дополнительные примеры приложений для клиентской библиотеки Хранилища очередей Azure версии 12 для JavaScript см. [здесь](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/storage/storage-queue/samples).
