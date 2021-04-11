---
title: Отправка сообщений в разделы Служебной шины Azure с помощью azure-messaging-servicebus
description: В этом кратком руководстве показано, как отправлять сообщения в разделы Служебной шины Azure с помощью пакета azure-messaging-servicebus.
ms.topic: quickstart
ms.tgt_pltfrm: dotnet
ms.date: 03/16/2021
ms.custom: contperf-fy21q3
ms.openlocfilehash: 79eb7783fd3daf546539dd5b9048f4e9f484374f
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106279805"
---
# <a name="send-messages-to-an-azure-service-bus-topic-and-receive-messages-from-subscriptions-to-the-topic-net"></a>Отправка сообщений в раздел Служебной шины Azure и прием сообщений из подписок в разделе (.NET)
В рамках этого руководства создается приложение C# для выполнения следующих задач:

1. Отправите сообщения в раздел служебной шины. 

    Раздел Служебной шины предоставляет конечную точку для отправки сообщений приложениями-отправителями. В разделе может быть одна или несколько подписок. В каждую подписку раздела поступает копия сообщения, отправленного в раздел. В статье [Что такое Служебная шина Azure?](service-bus-messaging-overview.md) вы найдете дополнительные сведения о разделах Служебной шины. 
1. Получение сообщений из подписки раздела. 

    :::image type="content" source="./media/service-bus-messaging-overview/about-service-bus-topic.png" alt-text="Разделы и подписки служебной шины":::

    > [!Important]
    > Для работы с этим кратким руководством используется новый пакет **Azure.Messaging.ServiceBus**. Если вы используете старый пакет Microsoft.Azure.ServiceBus, см. статью об [отправке и получении сообщений с помощью пакета Microsoft.Azure.ServiceBus](service-bus-dotnet-how-to-use-topics-subscriptions-legacy.md).

## <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Для работы с этим учебником требуется учетная запись Azure. Вы можете активировать [преимущества подписчика Visual Studio или MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) или зарегистрироваться для получения [бесплатной учетной записи](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Выполните шаги из этого [краткого руководства](service-bus-quickstart-topics-subscriptions-portal.md), чтобы создать раздел Служебной шины и подписки на него. 

    > [!NOTE]
    > В рамках этого руководства будет использоваться строка подключения к пространству имен, имя раздела и имя одной из подписок на раздел.  
- [Visual Studio 2019](https://www.visualstudio.com/vs). 
 
## <a name="send-messages-to-a-topic"></a>Отправка сообщений в раздел
При работе с этим разделом вы создадите консольное приложение .NET Core в Visual Studio и добавите код для отправки сообщений в созданный вами раздел Служебной шины. 

### <a name="create-a-console-application"></a>Создание консольного приложение
Создайте консольное приложение .NET Core в Visual Studio. 

1. Запустите Visual Studio.  
1. Если отображается страница **Начало работы**, выберите элемент **Создать новый проект**. 
1. На странице **Создание нового проекта** выполните следующие действия: 
    1. В качестве языка программирования выберите **C#** . 
    1. В качестве типа проекта выберите **Консоль**. 
    1. Выберите в списке шаблонов вариант **Консольное приложение (.NET Core)** . 
    1. Затем нажмите кнопку **Далее**. 
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/create-console-project.png" alt-text="Создание проекта консольного приложения":::
1. На странице **Настроить новый проект** выполните следующие инструкции: 
    1. Введите **имя проекта** в соответствующее поле. 
    1. В поле **Расположение** выберите расположение для файлов проекта и решения. 
    1. Введите **имя решения** в соответствующее поле. В решении Visual Studio может быть один или более проектов. В рамках этого краткого руководства решение будет содержать только один проект. 
    1. Выберите **Создать**, чтобы создать проект. 
            
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/create-console-project-2.png" alt-text="Ввод имени и расположения проекта и решения":::    


### <a name="add-the-service-bus-nuget-package"></a>Получение пакета NuGet для служебной шины
1. Щелкните созданный проект правой кнопкой мыши и выберите **Управление пакетами NuGet**.

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/manage-nuget-packages-menu.png" alt-text="Меню управления пакетами NuGet":::        
1. Перейдите на вкладку **Обзор**.
1. Найдите и выберите элемент **[Azure.Messaging.ServiceBus](https://www.nuget.org/packages/Azure.Messaging.ServiceBus/)** .
1. Выберите **Установить** для завершения установки.

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/select-service-bus-package.png" alt-text="Выбор пакета NuGet Служебной шины.":::
5. Если отображается диалоговое окно **Просмотр изменений**, нажмите кнопку **ОК**, чтобы продолжить. 
1. Если отображается страница **Принятие условий лицензионного соглашения**, выберите вариант **Принимаю**, чтобы продолжить. 
    

### <a name="add-code-to-send-messages-to-the-topic"></a>Добавление кода для отправки сообщений в раздел 

1. В окне **Обозревателя решений** дважды щелкните файл **Program.cs**, чтобы открыть его в редакторе. 
1. Добавьте следующие инструкции `using` в начале определения пространства имен перед объявлением класса:
   
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Azure.Messaging.ServiceBus;
    ```
1. В классе `Program` над функцией `Main` объявите следующие переменные:

    ```csharp
        static string connectionString = "<NAMESPACE CONNECTION STRING>";
        static string topicName = "<SERVICE BUS TOPIC NAME>";
        static string subscriptionName = "<SERVICE BUS - TOPIC SUBSCRIPTION NAME>";
    ```

    Измените следующие значения:
    - `<NAMESPACE CONNECTION STRING>` на строку подключения к пространству имен Служебной шины;
    - `<TOPIC NAME>` на имя раздела;
    - `<SUBSCRIPTION NAME>` на имя подписки.

### <a name="send-a-single-message-to-the-topic"></a>Отправка одного сообщения в раздел
Добавьте метод с именем `SendMessageToTopicAsync` в класс `Program` выше или ниже метода `Main`. Этот метод позволяет отправить одно сообщение в раздел.

```csharp
    static async Task SendMessageToTopicAsync()
    {
        // create a Service Bus client 
        await using (ServiceBusClient client = new ServiceBusClient(connectionString))
        {
            // create a sender for the topic
            ServiceBusSender sender = client.CreateSender(topicName);
            await sender.SendMessageAsync(new ServiceBusMessage("Hello, World!"));
            Console.WriteLine($"Sent a single message to the topic: {topicName}");
        }
    }
```

Метод отвечает за следующие действия: 
1. Создание объекта [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) с использованием строки подключения к пространству имен. 
1. Создание объекта [ServiceBusSender](/dotnet/api/azure.messaging.servicebus.servicebussender) для указанного раздела Служебной шины с помощью объекта `ServiceBusClient`. На этом этапе используется метод [ServiceBusClient.CreateSender](/dotnet/api/azure.messaging.servicebus.servicebusclient.createsender).
1. Затем метод отправляет одно тестовое сообщение в раздел Служебной шины с помощью метода [ServiceBusSender.SendMessageAsync](/dotnet/api/azure.messaging.servicebus.servicebussender.sendmessageasync). 

### <a name="send-a-batch-of-messages-to-the-topic"></a>Отправка пакета сообщений в раздел
1. Добавьте метод с именем `CreateMessages`, чтобы создать очередь сообщений (очередь .NET, а не Служебной шины) для класса `Program`. Как правило, эти сообщения поступают из различных частей приложения. Здесь мы создадим очередь примеров сообщений. Если вы не знакомы с очередями .NET, см. статью о [Queue.Enqueue](/dotnet/api/system.collections.queue.enqueue).

    ```csharp
        static Queue<ServiceBusMessage> CreateMessages()
        {
            // create a queue containing the messages and return it to the caller
            Queue<ServiceBusMessage> messages = new Queue<ServiceBusMessage>();
            messages.Enqueue(new ServiceBusMessage("First message"));
            messages.Enqueue(new ServiceBusMessage("Second message"));
            messages.Enqueue(new ServiceBusMessage("Third message"));
            return messages;
        }
    ```
1. Добавьте метод с именем `SendMessageBatchAsync` в класс `Program` и добавьте приведенный ниже код. Этот метод позволяет принять очередь сообщений и подготовить один или несколько пакетов для отправки в раздел Служебной шины. 

    ```csharp
        static async Task SendMessageBatchToTopicAsync()
        {
            // create a Service Bus client 
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {

                // create a sender for the topic 
                ServiceBusSender sender = client.CreateSender(topicName);

                // get the messages to be sent to the Service Bus topic
                Queue<ServiceBusMessage> messages = CreateMessages();

                // total number of messages to be sent to the Service Bus topic
                int messageCount = messages.Count;

                // while all messages are not sent to the Service Bus topic
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

                Console.WriteLine($"Sent a batch of {messageCount} messages to the topic: {topicName}");
            }
        }
    ```    

    Вот важные шаги из кода:
    1. Создание объекта [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) с использованием строки подключения к пространству имен. 
    1. Вызов метода [CreateSender](/dotnet/api/azure.messaging.servicebus.servicebusclient.createsender) для объекта `ServiceBusClient`. Это позволяет создать объект [ServiceBusSender](/dotnet/api/azure.messaging.servicebus.servicebussender) для указанного раздела Служебной шины. 
    1. Вызов вспомогательного метода `GetMessages` для получения очереди сообщений, отправляемых в раздел Служебной шины. 
    1. Создание [ServiceBusMessageBatch](/dotnet/api/azure.messaging.servicebus.servicebusmessagebatch) с использованием [ServiceBusSender.CreateMessageBatchAsync](/dotnet/api/azure.messaging.servicebus.servicebussender.createmessagebatchasync).
    1. Добавление сообщений в пакет с помощью [ServiceBusMessageBatch.TryAddMessage](/dotnet/api/azure.messaging.servicebus.servicebusmessagebatch.tryaddmessage). По мере добавления сообщений в пакет они удаляются из очереди .NET. 
    1. Отправка пакета сообщений в раздел Служебной шины с помощью метода [ServiceBusSender.SendMessagesAsync](/dotnet/api/azure.messaging.servicebus.servicebussender.sendmessagesasync).

### <a name="update-the-main-method"></a>Обновление метода Main
Замените метод `Main()` приведенным ниже **асинхронным** методом `Main`. Он вызывает оба метода для отправки в раздел одного сообщения и пакета сообщений.  

```csharp
    static async Task Main()
    {
        // send a single message to the topic
        await SendMessageToTopicAsync();

        // send a batch of messages to the topic
        await SendMessageBatchToTopicAsync();
    }
```

### <a name="test-the-app-to-send-messages-to-the-topic"></a>Тестирование приложения для отправки сообщений в раздел
1. Запустите приложение. Вы должны увидеть следующий результат:

    ```console
    Sent a single message to the topic: mytopic
    Sent a batch of 3 messages to the topic: mytopic
    ```
1. На портале Azure выполните следующие действия:
    1. Перейдите к пространству имен Служебной шины. 
    1. В центральной области в нижней части страницы **Обзор** перейдите на вкладку **Разделы** и выберите раздел Служебной шины. В приведенном ниже примере это `mytopic`.
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/select-topic.png" alt-text="Выбор раздела":::
    1. В нижней части страницы **раздела Служебной шины** на диаграмме **Сообщения** в разделе **Метрики** можно увидеть, что в раздел поступило четыре входящих сообщения. Если значение не отображается, подождите несколько минут и обновите страницу, чтобы просмотреть обновленную диаграмму. 

        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/sent-messages-essentials.png" alt-text="Сообщения, отправленные в раздел" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/sent-messages-essentials.png":::
    4. Выберите подписку в области снизу. В приведенном ниже примере это **S1**. На странице **подписки на Служебную шину** для параметра **Количество активных сообщений** отображается значение **4**. В подписку поступили четыре сообщения, которые вы отправили в раздел, но получатель их еще не открыл. 
    
        :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page.png" alt-text="Сообщения, поступившие в подписку" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page.png":::
    

## <a name="receive-messages-from-a-subscription"></a>Получение сообщений из подписки

1. Добавьте следующие методы в класс `Program` для обработки сообщений и любых ошибок: 

    ```csharp
        static async Task MessageHandler(ProcessMessageEventArgs args)
        {
            string body = args.Message.Body.ToString();
            Console.WriteLine($"Received: {body} from subscription: {subscriptionName}");

            // complete the message. messages is deleted from the queue. 
            await args.CompleteMessageAsync(args.Message);
        }

        static Task ErrorHandler(ProcessErrorEventArgs args)
        {
            Console.WriteLine(args.Exception.ToString());
            return Task.CompletedTask;
        }
    ```
1. Добавьте в класс `Program` следующий метод `ReceiveMessagesFromSubscriptionAsync`:

    ```csharp
        static async Task ReceiveMessagesFromSubscriptionAsync()
        {
            await using (ServiceBusClient client = new ServiceBusClient(connectionString))
            {
                // create a processor that we can use to process the messages
                ServiceBusProcessor processor = client.CreateProcessor(topicName, subscriptionName, new ServiceBusProcessorOptions());

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

    Вот важные шаги из кода:
    1. Создание объекта [ServiceBusClient](/dotnet/api/azure.messaging.servicebus.servicebusclient) с использованием строки подключения к пространству имен. 
    1. Вызов метода [CreateProcessor](/dotnet/api/azure.messaging.servicebus.servicebusclient.createprocessor) для объекта `ServiceBusClient`. Это позволяет создать объект [ServiceBusProcessor](/dotnet/api/azure.messaging.servicebus.servicebusprocessor) для указанной комбинации раздела и подписки Служебной шины. 
    1. Указание обработчиков для событий [ProcessMessageAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.processmessageasync) и [ProcessErrorAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.processerrorasync) объекта `ServiceBusProcessor`. 
    1. Запуск обработки сообщений путем вызова [StartProcessingAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.startprocessingasync) для объекта `ServiceBusProcessor`. 
    1. Вызов [StopProcessingAsync](/dotnet/api/azure.messaging.servicebus.servicebusprocessor.stopprocessingasync) для объекта `ServiceBusProcessor` при нажатии пользователем кнопки для завершения обработки. 
1. Добавьте в метод `Main` вызов метода `ReceiveMessagesFromSubscriptionAsync`. Закомментируйте метод `SendMessagesToTopicAsync`, если нужно проверить только получение сообщений. В противном случае в раздел будут отправлены еще четыре сообщения. 

    ```csharp
        static async Task Main()
        {
            // send a message to the topic
            await SendMessageToTopicAsync();

            // send a batch of messages to the topic
            await SendMessageBatchToTopicAsync();

            // receive messages from the subscription
            await ReceiveMessagesFromSubscriptionAsync();
        }
    ```
## <a name="run-the-app"></a>Запустите приложение
Запустите приложение. Подождите минуту, а затем нажмите любую клавишу, чтобы остановить получение сообщений. В окне консоли должны отобразиться следующие выходные данные (клавиша ПРОБЕЛ на клавиатуре). 

```console
Sent a single message to the topic: mytopic
Sent a batch of 3 messages to the topic: mytopic
Wait for a minute and then press any key to end the processing
Received: Hello, World! from subscription: mysub
Received: First message from subscription: mysub
Received: Second message from subscription: mysub
Received: Third message from subscription: mysub
Received: Hello, World! from subscription: mysub
Received: First message from subscription: mysub
Received: Second message from subscription: mysub
Received: Third message from subscription: mysub

Stopping the receiver...
Stopped receiving messages
```

Снова просмотрите сведения на портале. 

- На странице **раздела Служебной шины** на диаграмме **Сообщения** можно увидеть, что в раздел поступило восемь входящих и восемь исходящих сообщений. Если эти числа не отображаются, подождите несколько минут и обновите страницу, чтобы просмотреть обновленную диаграмму. 

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/messages-size-final.png" alt-text="Отправленные и полученные сообщения" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/messages-size-final.png":::
- На странице **подписки на Служебную шину** для параметра **Количество активных сообщений** отображается нулевое значение. Это означает, что получатель получил сообщения из этой подписки и обработал их. 

    :::image type="content" source="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page-final.png" alt-text="Финальное число активных сообщений в подписке" lightbox="./media/service-bus-dotnet-how-to-use-topics-subscriptions/subscription-page-final.png":::
    


## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими примерами и документацией:

- [Клиентская библиотека Служебной шины Azure для .NET: файл сведений](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus)
- [Примеры .NET для Служебной шины Azure на сайте GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/servicebus/Azure.Messaging.ServiceBus/samples)
- [Справочник по API .NET](/dotnet/api/azure.messaging.servicebus?preserve-view=true)