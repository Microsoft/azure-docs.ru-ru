---
title: Начало работы с разделами и подписками служебной шины Azure | Документация Майкрософт
description: Написание консольного приложения C# .NET Core, которое использует обмен сообщениями служебной шины для разделов и подписок.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 09/02/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 28e94cdb0df0a18b41f4c8a0ded362b50df6dcac
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100652978"
---
# <a name="get-started-with-service-bus-topics"></a>Начало работы с разделами служебной шины
В этом руководстве рассматриваются следующие действия:

1. Написание консольного приложения .NET Core для отправки набора сообщений в раздел.
2. Написание консольного приложения .NET Core для получения этих сообщений из подписки.

> [!WARNING]
> Для работы с этим кратким руководством используются пакеты Microsoft.Azure.ServiceBus прежних версий. Сведения об использовании последнего пакета см. в кратком руководстве по [отправке и получению сообщений с помощью пакета Azure.Messaging.ServiceBus](service-bus-dotnet-how-to-use-topics-subscriptions.md). Чтобы начать использовать в приложении новый пакет вместо старого, ознакомьтесь с руководством по [переходу с Microsoft.Azure.ServiceBus на Azure.Messaging.ServiceBus](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/servicebus/Azure.Messaging.ServiceBus/MigrationGuide.md). 

## <a name="prerequisites"></a>Предварительные требования

1. Подписка Azure. Для работы с этим учебником требуется учетная запись Azure. Вы можете активировать [преимущества подписчика Visual Studio или MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
2. Выполните шаги из [краткого руководства по Создание раздела служебной шины и подписок на него с помощью портала Azure](service-bus-quickstart-topics-subscriptions-portal.md). Так вы сделаете следующее:
    1. Создайте **пространство имен** Служебной шины.
    2. Получите **строку подключения**.
    3. Создайте **раздел** в пространстве имен.
    4. Создайте **одну подписку** в разделе пространства имен.
3. [Visual Studio 2017 с обновлением 3 (версия 15.3, 26730.01)](https://www.visualstudio.com/vs) или более новая версия.
4. [Пакет SDK для .NET Core](https://www.microsoft.com/net/download/windows) версии 2.0 или более новой.
 
## <a name="send-messages-to-the-topic"></a>Отправка сообщений в раздел

Для отправки сообщений в раздел создайте консольное приложение C# с помощью Visual Studio.

### <a name="create-a-console-application"></a>Создание консольного приложение

Откройте Visual Studio и создайте проект **Консольное приложение (.NET Core)**.

### <a name="add-the-service-bus-nuget-package"></a>Получение пакета NuGet для служебной шины

1. Щелкните созданный проект правой кнопкой мыши и выберите **Управление пакетами NuGet**.
2. Откройте вкладку **Обзор**, выполните поиск по фразе **[Microsoft.Azure.ServiceBus](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus/)** и выберите элемент **Microsoft.Azure.ServiceBus**. Щелкните **Установить** , чтобы выполнить установку, а затем закройте это диалоговое окно.
   
    ![Установка пакета NuGet][nuget-pkg]

### <a name="write-code-to-send-messages-to-the-topic"></a>Написание кода для отправки сообщений в раздел

1. В файл Program.cs добавьте следующие операторы `using` в начале определения пространства имен перед объявлением класса.
   
    ```csharp
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.ServiceBus;
    ```

2. В классе `Program` объявите приведенные ниже переменные. Укажите для переменной `ServiceBusConnectionString` строку подключения, полученную при создании пространства имен, и задайте для `TopicName` имя раздела, которое использовалось при его создании.
   
    ```csharp
    const string ServiceBusConnectionString = "<your_connection_string>";
    const string TopicName = "<your_topic_name>";
    static ITopicClient topicClient;
    ``` 

3. Замените метод `Main()` следующим **асинхронным** методом `Main`, который асинхронно отправляет сообщения с помощью метода SendMessagesAsync (его вы добавите на следующем шаге). 

    ```csharp
    public static async Task Main(string[] args)
    {
        const int numberOfMessages = 10;
        topicClient = new TopicClient(ServiceBusConnectionString, TopicName);

        Console.WriteLine("======================================================");
        Console.WriteLine("Press ENTER key to exit after sending all the messages.");
        Console.WriteLine("======================================================");

        // Send messages.
        await SendMessagesAsync(numberOfMessages);

        Console.ReadKey();

        await topicClient.CloseAsync();
    }
    ```
5. Сразу после метода `Main` добавьте следующий метод `SendMessagesAsync()`, который позволяет отправлять такое число сообщений, как задано для параметра `numberOfMessagesToSend` (текущее значение — 10):

    ```csharp
    static async Task SendMessagesAsync(int numberOfMessagesToSend)
    {
        try
        {
            for (var i = 0; i < numberOfMessagesToSend; i++)
            {
                // Create a new message to send to the topic.
                string messageBody = $"Message {i}";
                var message = new Message(Encoding.UTF8.GetBytes(messageBody));

                // Write the body of the message to the console.
                Console.WriteLine($"Sending message: {messageBody}");

                // Send the message to the topic.
                await topicClient.SendAsync(message);
            }
        }
        catch (Exception exception)
        {
            Console.WriteLine($"{DateTime.Now} :: Exception: {exception.Message}");
        }
    }
    ```
  
6. Вот как будет выглядеть файл Program.cs отправителя:
   
    ```csharp
    namespace CoreSenderApp
    {
        using System;
        using System.Text;
        using System.Threading;
        using System.Threading.Tasks;
        using Microsoft.Azure.ServiceBus;

        class Program
        {
            const string ServiceBusConnectionString = "<your_connection_string>";
            const string TopicName = "<your_topic_name>";
            static ITopicClient topicClient;

            public static async Task Main(string[] args)
            {
                const int numberOfMessages = 10;
                topicClient = new TopicClient(ServiceBusConnectionString, TopicName);
    
                Console.WriteLine("======================================================");
                Console.WriteLine("Press ENTER key to exit after sending all the messages.");
                Console.WriteLine("======================================================");
    
                // Send messages.
                await SendMessagesAsync(numberOfMessages);
    
                Console.ReadKey();
    
                await topicClient.CloseAsync();
            }

            static async Task SendMessagesAsync(int numberOfMessagesToSend)
            {
                try
                {
                    for (var i = 0; i < numberOfMessagesToSend; i++)
                    {
                        // Create a new message to send to the topic
                        string messageBody = $"Message {i}";
                        var message = new Message(Encoding.UTF8.GetBytes(messageBody));

                        // Write the body of the message to the console
                        Console.WriteLine($"Sending message: {messageBody}");

                        // Send the message to the topic
                        await topicClient.SendAsync(message);
                    }
                }
                catch (Exception exception)
                {
                    Console.WriteLine($"{DateTime.Now} :: Exception: {exception.Message}");
                }
            }
        }
    }
    ```

3. Запустите программу и перейдите на портал Azure. В окне **Обзор** пространства имен щелкните имя раздела. Отобразится экран раздела **Основное**. Обратите внимание, что теперь для подписки внизу окна параметр **Число сообщений** должен иметь значение **10**. Каждый раз при запуске приложения отправителя без получения сообщений (как указано в следующем разделе) это значение увеличивается на 10. А при каждом добавлении сообщения в раздел текущий размер раздела увеличивается, как и значение параметра **Текущий** в колонке **Основное**.
   
      ![Размер сообщения][topic-message]

## <a name="receive-messages-from-the-subscription"></a>Получение сообщений из подписки

Чтобы получить отправленные сообщения, создайте другое консольное приложение .NET Core и установите пакет NuGet **Microsoft.Azure.ServiceBus**, как для предыдущего приложения отправителя.

### <a name="write-code-to-receive-messages-from-the-subscription"></a>Написание кода для получения сообщений из подписки

1. В файл Program.cs добавьте следующие операторы `using` в начале определения пространства имен перед объявлением класса.
   
    ```csharp
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.ServiceBus;
    ```

2. В классе `Program` объявите приведенные ниже переменные. Укажите для переменной `ServiceBusConnectionString` строку подключения, полученную при создании пространства имен, и задайте для `TopicName` имя раздела, которое использовалось при его создании, а для `SubscriptionName` задайте имя подписки на раздел, которое использовалось при ее создании.
   
    ```csharp
    const string ServiceBusConnectionString = "<your_connection_string>";
    const string TopicName = "<your_topic_name>";
    const string SubscriptionName = "<your_subscription_name>";
    static ISubscriptionClient subscriptionClient;
    ```

3. Замените метод `Main()` приведенным ниже **асинхронным** методом `Main`. Он вызывает метод `RegisterOnMessageHandlerAndReceiveMessages()`, который вы добавите на следующем шаге. 

    ```csharp
    public static async Task Main(string[] args)
    {    
        subscriptionClient = new SubscriptionClient(ServiceBusConnectionString, TopicName, SubscriptionName);

        Console.WriteLine("======================================================");
        Console.WriteLine("Press ENTER key to exit after receiving all the messages.");
        Console.WriteLine("======================================================");

        // Register subscription message handler and receive messages in a loop
        RegisterOnMessageHandlerAndReceiveMessages();

        Console.ReadKey();

        await subscriptionClient.CloseAsync();    
    }
   ```
5. Сразу после метода `Main()` добавьте следующий метод, который позволяет зарегистрировать обработчик сообщений и получать сообщения, отправленные приложением-отправителем:

    ```csharp
    static void RegisterOnMessageHandlerAndReceiveMessages()
    {
        // Configure the message handler options in terms of exception handling, number of concurrent messages to deliver, etc.
        var messageHandlerOptions = new MessageHandlerOptions(ExceptionReceivedHandler)
        {
            // Maximum number of concurrent calls to the callback ProcessMessagesAsync(), set to 1 for simplicity.
            // Set it according to how many messages the application wants to process in parallel.
            MaxConcurrentCalls = 1,

            // Indicates whether the message pump should automatically complete the messages after returning from user callback.
            // False below indicates the complete operation is handled by the user callback as in ProcessMessagesAsync().
            AutoComplete = false
        };

        // Register the function that processes messages.
        subscriptionClient.RegisterMessageHandler(ProcessMessagesAsync, messageHandlerOptions);
    }
    ```    

6. Сразу после предыдущего метода добавьте следующий метод `ProcessMessagesAsync()` для обработки полученных сообщений:
 
    ```csharp
    static async Task ProcessMessagesAsync(Message message, CancellationToken token)
    {
        // Process the message.
        Console.WriteLine($"Received message: SequenceNumber:{message.SystemProperties.SequenceNumber} Body:{Encoding.UTF8.GetString(message.Body)}");

        // Complete the message so that it is not received again.
        // This can be done only if the subscriptionClient is created in ReceiveMode.PeekLock mode (which is the default).
        await subscriptionClient.CompleteAsync(message.SystemProperties.LockToken);

        // Note: Use the cancellationToken passed as necessary to determine if the subscriptionClient has already been closed.
        // If subscriptionClient has already been closed, you can choose to not call CompleteAsync() or AbandonAsync() etc.
        // to avoid unnecessary exceptions.
    }
    ```

7. Наконец, добавьте следующий метод для обработки исключений, которые могут возникнуть:
 
    ```csharp
    // Use this handler to examine the exceptions received on the message pump.
    static Task ExceptionReceivedHandler(ExceptionReceivedEventArgs exceptionReceivedEventArgs)
    {
        Console.WriteLine($"Message handler encountered an exception {exceptionReceivedEventArgs.Exception}.");
        var context = exceptionReceivedEventArgs.ExceptionReceivedContext;
        Console.WriteLine("Exception context for troubleshooting:");
        Console.WriteLine($"- Endpoint: {context.Endpoint}");
        Console.WriteLine($"- Entity Path: {context.EntityPath}");
        Console.WriteLine($"- Executing Action: {context.Action}");
        return Task.CompletedTask;
    }    
    ```    

8. Вот как будет выглядеть файл Program.cs получателя:
   
    ```csharp
    namespace CoreReceiverApp
    {
        using System;
        using System.Text;
        using System.Threading;
        using System.Threading.Tasks;
        using Microsoft.Azure.ServiceBus;

        class Program
        {
            const string ServiceBusConnectionString = "<your_connection_string>";
            const string TopicName = "<your_topic_name>";
            const string SubscriptionName = "<your_subscription_name>";
            static ISubscriptionClient subscriptionClient;

            public static async Task Main(string[] args)
            {    
                subscriptionClient = new SubscriptionClient(ServiceBusConnectionString, TopicName, SubscriptionName);
        
                Console.WriteLine("======================================================");
                Console.WriteLine("Press ENTER key to exit after receiving all the messages.");
                Console.WriteLine("======================================================");
        
                // Register subscription message handler and receive messages in a loop
                RegisterOnMessageHandlerAndReceiveMessages();
        
                Console.ReadKey();
        
                await subscriptionClient.CloseAsync();    
            }

            static void RegisterOnMessageHandlerAndReceiveMessages()
            {
                // Configure the message handler options in terms of exception handling, number of concurrent messages to deliver, etc.
                var messageHandlerOptions = new MessageHandlerOptions(ExceptionReceivedHandler)
                {
                    // Maximum number of concurrent calls to the callback ProcessMessagesAsync(), set to 1 for simplicity.
                    // Set it according to how many messages the application wants to process in parallel.
                    MaxConcurrentCalls = 1,

                    // Indicates whether MessagePump should automatically complete the messages after returning from User Callback.
                    // False below indicates the Complete will be handled by the User Callback as in `ProcessMessagesAsync` below.
                    AutoComplete = false
                };

                // Register the function that processes messages.
                subscriptionClient.RegisterMessageHandler(ProcessMessagesAsync, messageHandlerOptions);
            }

            static async Task ProcessMessagesAsync(Message message, CancellationToken token)
            {
                // Process the message.
                Console.WriteLine($"Received message: SequenceNumber:{message.SystemProperties.SequenceNumber} Body:{Encoding.UTF8.GetString(message.Body)}");

                // Complete the message so that it is not received again.
                // This can be done only if the subscriptionClient is created in ReceiveMode.PeekLock mode (which is the default).
                await subscriptionClient.CompleteAsync(message.SystemProperties.LockToken);

                // Note: Use the cancellationToken passed as necessary to determine if the subscriptionClient has already been closed.
                // If subscriptionClient has already been closed, you can choose to not call CompleteAsync() or AbandonAsync() etc.
                // to avoid unnecessary exceptions.
            }

            static Task ExceptionReceivedHandler(ExceptionReceivedEventArgs exceptionReceivedEventArgs)
            {
                Console.WriteLine($"Message handler encountered an exception {exceptionReceivedEventArgs.Exception}.");
                var context = exceptionReceivedEventArgs.ExceptionReceivedContext;
                Console.WriteLine("Exception context for troubleshooting:");
                Console.WriteLine($"- Endpoint: {context.Endpoint}");
                Console.WriteLine($"- Entity Path: {context.EntityPath}");
                Console.WriteLine($"- Executing Action: {context.Action}");
                return Task.CompletedTask;
            }
        }
    }
    ```
9. Запустите программу и перейдите на портал еще раз. Обратите внимание, что параметры **Число сообщений** и **Текущий** имеют значение **0**.
   
    ![Длина раздела][topic-message-receive]

Поздравляем! С помощью библиотеки .NET Standard вы только что создали раздел и подписку, отправили 10 сообщений и получили их.

> [!NOTE]
> Вы можете управлять ресурсами служебной шины с помощью [обозревателя служебной шины](https://github.com/paolosalvatori/ServiceBusExplorer/). Обозреватель служебной шины позволяет без труда подключаться к пространству имен служебной шины и управлять сущностями обмена сообщениями. Средство предоставляет дополнительные возможности, например функции импорта и экспорта или возможность проверять разделы, очереди, подписки, службы ретрансляции, центры уведомлений и концентраторы событий. 

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [примерами в репозитории GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples) для служебной шины, демонстрирующими расширенные возможности обмена сообщениями служебной шины.

<!--Image references-->

[nuget-pkg]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/nuget-package.png
[topic-message]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/topic-message.png
[topic-message-receive]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/topic-message-receive.png
[createtopic1]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/create-topic1.png
[createtopic2]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/create-topic2.png
[createtopic3]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/create-topic3.png
[createtopic4]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/create-topic4.png
[github-samples]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples
[azure-portal]: https://portal.azure.com
