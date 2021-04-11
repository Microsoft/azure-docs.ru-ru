---
title: Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 — .NET
description: Сведения о том, как использовать клиентскую библиотеку Хранилища очередей Azure версии 12 для .NET, чтобы создавать очереди и добавлять в них сообщения. Далее вы узнаете, как читать и удалять сообщения из очереди. Также здесь описано, как удалить очередь.
author: twooley
ms.author: twooley
ms.date: 07/24/2020
ms.topic: quickstart
ms.service: storage
ms.subservice: queues
ms.custom: devx-track-csharp
ms.openlocfilehash: 2c4af7afd96f6c503f35cdb4896c3b49d4ef6ed3
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106276235"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-net"></a>Краткое руководство. Использование клиентской библиотеки Хранилища очередей Azure версии 12 для .NET

Приступите к работе с клиентской библиотекой Хранилища очередей Azure версии 12 для .NET. Хранилище очередей Azure — это служба хранения большого количества сообщений для последующего извлечения и обработки. Выполните приведенные здесь действия, чтобы установить пакет и протестировать пример кода для выполнения базовых задач.

Клиентскую библиотеку Хранилища очередей Azure версии 12 для .NET можно использовать для следующих задач:

- Создание очереди
- Добавление сообщений в очередь
- Просмотр сообщений из очереди
- Обновление сообщений в очереди
- Получение сообщений из очереди
- Удаление сообщений из очереди
- Удаление очереди

Дополнительные ресурсы:

- [Справочная документация по API](/dotnet/api/azure.storage.queues)
- [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Queues)
- [Пакет (NuGet)](https://www.nuget.org/packages/Azure.Storage.Queues/12.0.0)
- [Примеры](../common/storage-samples-dotnet.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/).
- Учетная запись хранения Azure — [создайте такую учетную запись](../common/storage-account-create.md).
- [Пакет SDK для .NET Core](https://dotnet.microsoft.com/download/dotnet-core) последней версии для вашей операционной системы. Обязательно получите пакет SDK, а не среду выполнения.

## <a name="setting-up"></a>Настройка

В этом разделе рассматривается подготовка проекта для работы с клиентской библиотекой Хранилища очередей Azure версии 12 для .NET.

### <a name="create-the-project"></a>Создание проекта

Создайте приложение .NET Core с именем `QueuesQuickstartV12`.

1. В окне консоли (cmd, PowerShell или Bash) выполните команду `dotnet new`, чтобы создать консольное приложение с именем `QueuesQuickstartV12`. Эта команда создает простой проект Hello World на языке C# с одним файлом исходного кода: `Program.cs`.

   ```console
   dotnet new console -n QueuesQuickstartV12
   ```

1. Переключитесь на только что созданный каталог `QueuesQuickstartV12`.

   ```console
   cd QueuesQuickstartV12
   ```

### <a name="install-the-package"></a>Установка пакета

Оставаясь в каталоге приложения, установите пакет клиентской библиотеки Хранилища очередей Azure для .NET с помощью команды `dotnet add package`.

```console
dotnet add package Azure.Storage.Queues
```

### <a name="set-up-the-app-framework"></a>Настройка платформы приложения

Из каталога проекта:

1. Откройте файл `Program.cs` в редакторе.
1. Удалите инструкцию `Console.WriteLine("Hello, World");`.
1. Добавьте директивы `using`.
1. Обновите объявление метода `Main` для [поддержки асинхронного кода](/dotnet/csharp/whats-new/csharp-7#async-main).

Вот этот код:

```csharp
using Azure;
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;
using System;
using System.Threading.Tasks;

namespace QueuesQuickstartV12
{
    class Program
    {
        static async Task Main(string[] args)
        {
        }
    }
}
```

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Объектная модель

Хранилище очередей Azure — это служба для хранения большого количества сообщений. Максимальный размер сообщения в очереди составляет 64 КБ. Очередь может содержать миллионы сообщений вплоть до лимита всей емкости учетной записи хранения. Очереди обычно используются для создания списка невыполненных заданий для асинхронной обработки. В Хранилище очередей предлагается три типа ресурсов:

- учетная запись хранения;
- очередь в учетной записи хранилища;
- сообщения в очереди.

На следующей схеме показана связь между этими ресурсами.

![Схема архитектуры Хранилища очередей](./media/storage-queues-introduction/queue1.png)

Используйте следующие классы .NET для взаимодействия с этими ресурсами:

- [`QueueServiceClient`](/dotnet/api/azure.storage.queues.queueserviceclient). `QueueServiceClient` позволяет управлять всеми очередями в учетной записи хранения.
- [`QueueClient`](/dotnet/api/azure.storage.queues.queueclient). Класс `QueueClient` позволяет управлять отдельной очередью и сообщениями в ней.
- [`QueueMessage`](/dotnet/api/azure.storage.queues.models.queuemessage). Класс `QueueMessage` представляет отдельные объекты, которые возвращаются при вызове [`ReceiveMessages`](/dotnet/api/azure.storage.queues.queueclient.receivemessages) для очереди.

## <a name="code-examples"></a>Примеры кода

В этих примерах фрагментов кода показано, как выполнять следующие действия с помощью клиентской библиотеки Хранилища очередей Azure для .NET:

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

Добавьте этот код в метод `Main`.

```csharp
Console.WriteLine("Azure Queue Storage client library v12 - .NET quickstart sample\n");

// Retrieve the connection string for use with the application. The storage
// connection string is stored in an environment variable called
// AZURE_STORAGE_CONNECTION_STRING on the machine running the application.
// If the environment variable is created after the application is launched
// in a console or with Visual Studio, the shell or application needs to be
// closed and reloaded to take the environment variable into account.
string connectionString = Environment.GetEnvironmentVariable("AZURE_STORAGE_CONNECTION_STRING");
```

### <a name="create-a-queue"></a>Создание очереди

Выберите имя для новой очереди. Приведенный ниже код добавляет к имени очереди значение GUID, чтобы сделать это имя уникальным.

> [!IMPORTANT]
> Имя очереди может содержать только строчные буквы, цифры и дефисы и должно начинаться с буквы или цифры. Перед каждым дефисом должен быть знак без дефиса. Количество символов в имени должно быть от 3 до 63. Дополнительные сведения см. в статье [Именование очередей и метаданных](/rest/api/storageservices/naming-queues-and-metadata).

Создайте экземпляр класса [`QueueClient`](/dotnet/api/azure.storage.queues.queueclient). Затем вызовите метод [`CreateAsync`](/dotnet/api/azure.storage.queues.queueclient.createasync), чтобы создать очередь в своей учетной записи хранения.

Добавьте следующий код в конец метода `Main`.

```csharp
// Create a unique name for the queue
string queueName = "quickstartqueues-" + Guid.NewGuid().ToString();

Console.WriteLine($"Creating queue: {queueName}");

// Instantiate a QueueClient which will be
// used to create and manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

// Create the queue
await queueClient.CreateAsync();
```

### <a name="add-messages-to-a-queue"></a>Добавление сообщений в очередь

Приведенный ниже фрагмент кода асинхронно добавляет сообщения в очередь, вызывая метод [`SendMessageAsync`](/dotnet/api/azure.storage.queues.queueclient.sendmessageasync). Он также сохраняет [`SendReceipt`](/dotnet/api/azure.storage.queues.models.sendreceipt), возвращенный вызовом `SendMessageAsync`. Это подтверждение используется для обновления сообщения далее в коде программы.

Добавьте следующий код в конец метода `Main`.

```csharp
Console.WriteLine("\nAdding messages to the queue...");

// Send several messages to the queue
await queueClient.SendMessageAsync("First message");
await queueClient.SendMessageAsync("Second message");

// Save the receipt so we can update this message later
SendReceipt receipt = await queueClient.SendMessageAsync("Third message");
```

### <a name="peek-at-messages-in-a-queue"></a>Просмотр сообщений из очереди

Чтобы просмотреть сообщения в очереди, вызовите метод [`PeekMessagesAsync`](/dotnet/api/azure.storage.queues.queueclient.peekmessagesasync). Этот метод извлекает одно или несколько сообщений из начала очереди, не изменяя видимость этих сообщений.

Добавьте следующий код в конец метода `Main`.

```csharp
Console.WriteLine("\nPeek at the messages in the queue...");

// Peek at messages in the queue
PeekedMessage[] peekedMessages = await queueClient.PeekMessagesAsync(maxMessages: 10);

foreach (PeekedMessage peekedMessage in peekedMessages)
{
    // Display the message
    Console.WriteLine($"Message: {peekedMessage.MessageText}");
}
```

### <a name="update-a-message-in-a-queue"></a>Обновление сообщений в очереди

Чтобы обновить содержимое сообщения, вызовите метод [`UpdateMessageAsync`](/dotnet/api/azure.storage.queues.queueclient.updatemessageasync). Этот метод позволяет изменить время видимости сообщения и его содержимое. Содержимое сообщение должно иметь формат строки в кодировке UTF-8 длиной не более 64 КБ. Вместе с новым содержимым сообщения передайте значения из `SendReceipt`, сохраненного ранее в коде программы. Значения `SendReceipt` указывают, какое сообщение следует обновить.

```csharp
Console.WriteLine("\nUpdating the third message in the queue...");

// Update a message using the saved receipt from sending the message
await queueClient.UpdateMessageAsync(receipt.MessageId, receipt.PopReceipt, "Third message has been updated");
```

### <a name="receive-messages-from-a-queue"></a>Получение сообщений из очереди

Чтобы скачать ранее добавленные сообщения, вызовите метод [`ReceiveMessagesAsync`](/dotnet/api/azure.storage.queues.queueclient.receivemessagesasync).

Добавьте следующий код в конец метода `Main`.

```csharp
Console.WriteLine("\nReceiving messages from the queue...");

// Get messages from the queue
QueueMessage[] messages = await queueClient.ReceiveMessagesAsync(maxMessages: 10);
```

### <a name="delete-messages-from-a-queue"></a>Удаление сообщений из очереди

Удалите из очереди сообщения, которые уже обработаны. В нашем примере обработка сводится к выводу сообщения в консоль.

Перед обработкой и удалением сообщений приложение ожидает ввода от пользователя, вызывая метод `Console.ReadLine`. Перед удалением ресурсов убедитесь на [портале Azure](https://portal.azure.com), что они были правильно созданы. Все сообщения, которые не были удалены явным образом, снова становятся видимыми в очереди для новой попытки их обработать.

Добавьте следующий код в конец метода `Main`.

```csharp
Console.WriteLine("\nPress Enter key to 'process' messages and delete them from the queue...");
Console.ReadLine();

// Process and delete messages from the queue
foreach (QueueMessage message in messages)
{
    // "Process" the message
    Console.WriteLine($"Message: {message.MessageText}");

    // Let the service know we're finished with
    // the message and it can be safely deleted.
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}
```

### <a name="delete-a-queue"></a>Удаление очереди

Следующий код очищает созданные приложением ресурсы, удаляя очередь с помощью метода [`DeleteAsync`](/dotnet/api/azure.storage.queues.queueclient.deleteasync).

Добавьте следующий код в конец метода `Main`.

```csharp
Console.WriteLine("\nPress Enter key to delete the queue...");
Console.ReadLine();

// Clean up
Console.WriteLine($"Deleting queue: {queueClient.Name}");
await queueClient.DeleteAsync();

Console.WriteLine("Done");
```

## <a name="run-the-code"></a>Выполнение кода

Это приложение создает три сообщения и добавляет их в очередь Azure. Затем код получает список сообщений, извлекает и удаляет их, и наконец удаляет саму очередь.

В окне консоли перейдите к каталогу приложения, выполните его сборку и запустите его.

```console
dotnet build
```

```console
dotnet run
```

Вы должны увидеть выходные данные приложения, как показано ниже.

```output
Azure Queue Storage client library v12 - .NET quickstart sample

Creating queue: quickstartqueues-5c72da2c-30cc-4f09-b05c-a95d9da52af2

Adding messages to the queue...

Peek at the messages in the queue...
Message: First message
Message: Second message
Message: Third message

Updating the third message in the queue...

Receiving messages from the queue...

Press Enter key to 'process' messages and delete them from the queue...

Message: First message
Message: Second message
Message: Third message has been updated

Press Enter key to delete the queue...

Deleting queue: quickstartqueues-5c72da2c-30cc-4f09-b05c-a95d9da52af2
Done
```

Когда приложение приостановится перед получением сообщений, проверьте учетную запись хранения на [портале Azure](https://portal.azure.com). Убедитесь, что сообщения находятся в очереди.

Нажмите клавишу `Enter`, чтобы получить и удалить сообщения. При появлении запроса снова нажмите клавишу `Enter`, чтобы удалить очередь и завершить демонстрацию.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создавать очередь и добавлять в нее сообщения из асинхронного кода на .NET. Затем вы изучили процессы вставки, просмотра, получения и удаления сообщений. Наконец, вы узнали, как удалить очередь сообщений.

Руководства, примеры, краткие руководства и другую документацию можно найти здесь:

> [!div class="nextstepaction"]
> [Azure для разработчиков .NET и .NET Core](/dotnet/azure/)

- Чтобы узнать больше, ознакомьтесь с [библиотеками службы хранилища Azure для .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage).
- Дополнительные примеры приложений для клиентской библиотеки Хранилища очередей Azure для .NET см. [здесь](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Queues/samples).
- Дополнительные сведения о .NET Core см. в статье [Get started with .NET in 10 minutes](https://www.microsoft.com/net/learn/get-started/) (Быстрое начало работы с .NET).
