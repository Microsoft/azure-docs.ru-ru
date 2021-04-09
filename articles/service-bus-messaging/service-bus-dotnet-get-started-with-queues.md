---
title: Начало работы с очередями Служебной шины Azure (Azure.Messaging.ServiceBus)
description: В рамках этого руководства вы создадите приложение .NET Core на C# для отправки и получения сообщений через очередь Служебной шины.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 11/13/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: ec3f53e6f69614028c013efa5f0e6852cbc3f8ae
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98631646"
---
# <a name="send-messages-to-and-receive-messages-from-azure-service-bus-queues-net"></a>Отправка сообщений в очереди Служебной шины и получение сообщений из них (.NET)
В рамках этого руководства вы создадите консольное приложение .NET Core для отправки и получения сообщений через очередь Служебной шины с помощью пакета **Azure.Messaging.ServiceBus**. 

> [!Important]
> Для работы с этим кратким руководством используется новый пакет Azure.Messaging.ServiceBus. Краткое руководство, при работе с которым используется старый пакет Microsoft.Azure.ServiceBus, можно найти в статье об [отправке и получении событий с помощью пакета Microsoft.Azure.ServiceBus](service-bus-dotnet-get-started-with-queues-legacy.md).

## <a name="prerequisites"></a>Предварительные требования

- [Visual Studio 2019](https://www.visualstudio.com/vs)
- Подписка Azure. Для работы с этим учебником требуется учетная запись Azure. Вы можете активировать [преимущества подписчика MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Если у вас нет подходящей очереди служебной шины, [создайте ее с помощью портала Azure](service-bus-quickstart-portal.md). Запишите **строку подключения** к пространству имен Служебной шины и имя созданной **очереди**.

## <a name="send-messages-to-a-queue"></a>Отправка сообщений в очередь
В рамках этого краткого руководства вы создадите консольное приложение .NET Core на C# для отправки сообщений в очередь.

### <a name="create-a-console-application"></a>Создание консольного приложение
Откройте Visual Studio и создайте проект **Консольное приложение (.NET Core)** для C#. 

### <a name="add-the-service-bus-nuget-package"></a>Получение пакета NuGet для служебной шины

1. Щелкните созданный проект правой кнопкой мыши и выберите **Управление пакетами NuGet**.
1. Нажмите кнопку **Обзор**. Найдите и выберите элемент **[Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/)** .
1. Выберите элемент **Установить**, чтобы завершить установку, и закройте диспетчер пакетов NuGet.

### <a name="add-code-to-send-messages-to-the-queue"></a>Добавление кода для отправки сообщений в очередь

1. В файле *Program.cs* добавьте следующие инструкции `using` в начале определения пространства имен перед объявлением класса:

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    
    using Azure.Messaging.ServiceBus;
    ```

1. В классе `Program` объявите следующие переменные:

    ```csharp
        static string connectionString = "<NAMESPACE CONNECTION STRING>";
        static string queueName = "<QUEUE NAME>";
    ```

    Введите строку подключения для пространства имен в виде переменной `connectionString`. Введите имя очереди.

1. Сразу после метода `Main()` добавьте следующий метод `SendMessagesAsync()`, который отправляет сообщение.

    ```csharp
        static async Task SendMessageAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a sender for the queue 
                ServiceBusSender sender = client.CreateSender(queueName);

                // create a message that we can send
                ServiceBusMessage message = new ServiceBusMessage("Hello world!");

                // send the message
                await sender.SendMessageAsync(message);
                Console.WriteLine($"Sent a single message to the queue: {queueName}");
            }
        }
    ```
1. Добавьте метод с именем `CreateMessages`, чтобы создать очередь сообщений (очередь .NET) для класса `Program`. Как правило, эти сообщения поступают из различных частей приложения. Здесь мы создадим очередь примеров сообщений.

    ```csharp
        static Queue<ServiceBusMessage> CreateMessages()
        {
            // create a queue containing the messages and return it to the caller
            Queue<ServiceBusMessage> messages = new Queue<ServiceBusMessage>();
            messages.Enqueue(new ServiceBusMessage("First message in the batch"));
            messages.Enqueue(new ServiceBusMessage("Second message in the batch"));
            messages.Enqueue(new ServiceBusMessage("Third message in the batch"));
            return messages;
        }
    ```
1. Добавьте метод с именем `SendMessageBatchAsync` в класс `Program` и добавьте приведенный ниже код. Этот метод позволяет принять очередь сообщений и подготовить один или несколько пакетов для отправки в очередь Служебной шины. 

    ```csharp
        static async Task SendMessageBatchAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a sender for the queue 
                ServiceBusSender sender = client.CreateSender(queueName);

                // get the messages to be sent to the Service Bus queue
                Queue<ServiceBusMessage> messages = CreateMessages();

                // total number of messages to be sent to the Service Bus queue
                int messageCount = messages.Count;

                // while all messages are not sent to the Service Bus queue
                while (messages.Count > 0)
                {
                    // start a new batch 
                    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();

                    // add the first message to the batch
                    if (messageBatch.TryAddMessage(messages.Peek()))
                    {
                        // dequeue the message from the .NET queue once the message is added to the batch
                        messages.Dequeue();
                    }
                    else
                    {
                        // if the first message can't fit, then it is too large for the batch
                        throw new Exception($"Message {messageCount - messages.Count} is too large and cannot be sent.");
                    }

                    // add as many messages as possible to the current batch
                    while (messages.Count > 0 && messageBatch.TryAddMessage(messages.Peek()))
                    {
                        // dequeue the message from the .NET queue as it has been added to the batch
                        messages.Dequeue();
                    }
        
                    // now, send the batch
                    await sender.SendMessagesAsync(messageBatch);
        
                    // if there are any remaining messages in the .NET queue, the while loop repeats 
                }

                Console.WriteLine($"Sent a batch of {messageCount} messages to the topic: {queueName}");
            }
        }
    ```
1. Замените метод `Main()` приведенным ниже **асинхронным** методом `Main`. Он вызывает оба метода для отправки в очередь одного сообщения и пакета сообщений. 

    ```csharp
        static async Task Main()
        {
            // send a message to the queue
            await SendMessageAsync();

            // send a batch of messages to the queue
            await SendMessageBatchAsync();
        }
    ```
5. Запустите приложение. Отобразятся примерно такие сообщения, как указано ниже. 

    ```console
    Sent a single message to the queue: myqueue
    Sent a batch of messages to the queue: myqueue
    ```       
1. На портале Azure выполните следующие действия:
    1. Перейдите к пространству имен Служебной шины. 
    1. На странице **Обзор** выберите очередь в центральной области снизу. 
    1. Обратите внимание на значения в разделе **Основное**.

    :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/sent-messages-essentials.png" alt-text="Полученные сообщения с указанием количества и размера" lightbox="./media/service-bus-dotnet-get-started-with-queues/sent-messages-essentials.png":::

    Обратите внимание на следующие значения:
    - Значение параметра **Количество активных сообщений** для очереди теперь равно **4**. Каждый раз при запуске этого приложения-отправителя без получения сообщений это значение увеличивается на 4.
    - При каждом добавлении сообщений в очередь ее текущий размер увеличивается, как и значение параметра **Текущий** в колонке **Основное**.
    - На диаграмме **Сообщения** в разделе **Метрики** в нижней части страницы можно увидеть, что в очередь поступило четыре входящих сообщения. 

## <a name="receive-messages-from-a-queue"></a>Получение сообщений из очереди
В рамках этого раздела вы добавите код для получения сообщений из очереди.

1. Добавьте следующие методы в класс `Program` для обработки сообщений и любых ошибок: 

    ```csharp
        // handle received messages
        static async Task MessageHandler(ProcessMessageEventArgs args)
        {
            string body = args.Message.Body.ToString();
            Console.WriteLine($"Received: {body}");

            // complete the message. messages is deleted from the queue. 
            await args.CompleteMessageAsync(args.Message);
        }

        // handle any errors when receiving messages
        static Task ErrorHandler(ProcessErrorEventArgs args)
        {
            Console.WriteLine(args.Exception.ToString());
            return Task.CompletedTask;
        }
    ```
1. Добавьте метод с именем `ReceiveMessagesAsync` в класс `Program` и добавьте приведенный ниже код для получения сообщений. 

    ```csharp
        static async Task ReceiveMessagesAsync()
        {
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a processor that we can use to process the messages
                ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());

                // add handler to process messages
                processor.ProcessMessageAsync += MessageHandler;

                // add handler to process any errors
                processor.ProcessErrorAsync += ErrorHandler;

                // start processing 
                await processor.StartProcessingAsync();

                Console.WriteLine("Wait for a minute and then press any key to end the processing");
                Console.ReadKey();

                // stop processing 
                Console.WriteLine("\nStopping the receiver...");
                await processor.StopProcessingAsync();
                Console.WriteLine("Stopped receiving messages");
            }
        }
    ```
1. Добавьте вызов метода `ReceiveMessagesAsync` из метода `Main`. Закомментируйте метод `SendMessagesAsync`, если нужно проверить только получение сообщений. В противном случае в очередь будут отправлены еще четыре сообщения. 

    ```csharp
        static async Task Main()
        {
            // send a message to the queue
            await SendMessageAsync();

            // send a batch of messages to the queue
            await SendMessageBatchAsync();

            // receive message from the queue
            await ReceiveMessagesAsync();
        }
    ```

## <a name="run-the-app"></a>Запустите приложение
Запустите приложение. Подождите минуту, а затем нажмите любую клавишу, чтобы остановить получение сообщений. В окне консоли должны отобразиться следующие выходные данные (клавиша ПРОБЕЛ на клавиатуре). 

```console
Sent a single message to the queue: myqueue
Sent a batch of messages to the queue: myqueue
Wait for a minute and then press any key to end the processing
Received: Hello world!
Received: First message in the batch
Received: Second message in the batch
Received: Third message in the batch
Received: Hello world!
Received: First message in the batch
Received: Second message in the batch
Received: Third message in the batch

Stopping the receiver...
Stopped receiving messages
```

Снова просмотрите сведения на портале. 

- Значения параметров **Количество активных сообщений** и **Текущий** теперь равны **0**.
- На диаграмме **Сообщения** в разделе **Метрики** в нижней части страницы можно увидеть, что в очередь поступило восемь входящих и восемь исходящих сообщений. 

    :::image type="content" source="./media/service-bus-dotnet-get-started-with-queues/queue-messages-size-final.png" alt-text="Активные сообщения и их размер после получения" lightbox="./media/service-bus-dotnet-get-started-with-queues/queue-messages-size-final.png":::

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими примерами и документацией:

- [Клиентская библиотека Служебной шины Azure для .NET: файл сведений](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus)
- [Примеры на GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus/samples)
- [Справочник по API .NET](/dotnet/api/azure.messaging.servicebus?preserve-view=true)