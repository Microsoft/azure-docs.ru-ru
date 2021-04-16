---
title: Отправка и получение событий через Центры событий Azure с помощью .NET (старая версия)
description: В этой статье описано, как создать приложение .NET Core, которое отправляет события или получает их через службу "Центры событий Azure" с помощью пакета Microsoft.Azure.EventHubs старой версии.
ms.topic: quickstart
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: f289b920b036629e5c518ce3c9e8ec521b1497c3
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107310110"
---
# <a name="send-events-to-or-receive-events-from-azure-event-hubs-using-net-core-microsoftazureeventhubs"></a>Отправка и получение событий с помощью .NET Core в Центрах событий Azure (Microsoft.Azure.EventHubs)
В этом кратком руководстве показано, как отправлять события в концентратор событий и получать события из него с помощью библиотеки .NET Core **Microsoft.Azure.EventHubs**.

> [!WARNING]
> Для работы с этим кратким руководством используется пакет **Microsoft.Azure.EventHubs** старой версии. Краткое руководство, в рамках которого используется библиотека **Azure.Messaging.EventHubs** последней версии, см. в статье об [отправке и получении событий с помощью библиотеки Azure.Messaging.EventHubs](event-hubs-dotnet-standard-getstarted-send.md). Чтобы начать использовать в приложении библиотеку новой версии вместо старой, ознакомьтесь с руководством по [переходу с Microsoft.Azure.EventHubs на Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).

## <a name="prerequisites"></a>Предварительные требования
Если вы впервые используете Центры событий Azure, ознакомьтесь с общими сведениями в [этой статье](event-hubs-about.md), прежде чем приступить к работе с этим руководством. 

Для работы с данным руководством необходимо следующее:

- **Подписка Microsoft Azure.** Чтобы использовать службы Azure, в том числе Центры событий Azure, потребуется действующая подписка.  Если у вас еще нет учетной записи Azure, [зарегистрируйтесь для работы с бесплатной пробной версией](https://azure.microsoft.com/free/) или [активируйте преимущества для подписчиков MSDN при создании учетной записи](https://azure.microsoft.com).
- [Microsoft Visual Studio 2019](https://www.visualstudio.com).
- [Инструментарий Visual Studio 2015 или Visual Studio 2017 для .NET Core](https://www.microsoft.com/net/core). 
- **Создайте пространство имен Центров событий и концентратор событий**. Первым шагом является использование [портала Azure](https://portal.azure.com) для создания пространства имен типа Центров событий и получение учетных данных управления, необходимых приложению для взаимодействия с концентратором событий. Чтобы создать пространство имен и концентратор событий, выполните инструкции из [этой статьи](event-hubs-create.md). Получите **строку подключения для пространства имен концентратора событий**, следуя инструкциям из статьи [Получение строки подключения на портале](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Строка подключения понадобится нам позже в рамках этого краткого руководства.

## <a name="send-events"></a>Отправка событий 
В этом разделе показано, как создать консольное приложение .NET Core, которое отправляет события в концентратор событий. 

> [!NOTE]
> Вы можете скачать это краткое руководство в качестве примера с сайта [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleSender), заменить строки `EventHubConnectionString` и `EventHubName` значениями для своего концентратора событий и выполнить этот пример. Или следуйте инструкциям из этого краткого руководства, чтобы создать собственное решение.


### <a name="create-a-console-application"></a>Создание консольного приложение

Запустите среду Visual Studio. В меню **Файл** последовательно выберите пункты **Создать** и **Проект**. Создайте консольное приложение .NET Core.

![Новый проект](./media/event-hubs-dotnet-standard-getstarted-send/netcoresnd.png)

### <a name="add-the-event-hubs-nuget-package"></a>Добавление пакета NuGet для Центров событий

Добавьте пакет NuGet библиотеки .NET Core [`Microsoft.Azure.EventHubs`](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) в проект, сделав следующее: 

1. Щелкните созданный проект правой кнопкой мыши и выберите **Управление пакетами NuGet**.
2. Откройте вкладку **Обзор**, а затем выполните поиск по фразе Microsoft.Azure.EventHubs и выберите пакет **Microsoft.Azure.EventHubs**. Щелкните **Установить** , чтобы выполнить установку, а затем закройте это диалоговое окно.

### <a name="write-code-to-send-messages-to-the-event-hub"></a>Написание кода для отправки сообщений в концентратор событий

1. Добавьте следующие операторы `using` в начало файла Program.cs.

    ```csharp
    using Microsoft.Azure.EventHubs;
    using System.Text;
    using System.Threading.Tasks;
    ```

2. Добавьте в класс `Program` константы для строки подключения Центров событий и пути сущности (имя отдельного концентратора событий). Замените заполнители в скобках соответствующими значениями, которые были получены при создании концентратора событий. Убедитесь, что `{Event Hubs connection string}` — это строка подключения на уровне пространства имен, а не концентратора событий. 

    ```csharp
    private static EventHubClient eventHubClient;
    private const string EventHubConnectionString = "{Event Hubs connection string}";
    private const string EventHubName = "{Event Hub path/name}";
    ```

3. Добавьте новый метод с именем `MainAsync` в класс `Program`, как показано далее:

    ```csharp
    private static async Task MainAsync(string[] args)
    {
        // Creates an EventHubsConnectionStringBuilder object from the connection string, and sets the EntityPath.
        // Typically, the connection string should have the entity path in it, but this simple scenario
        // uses the connection string from the namespace.
        var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
        {
            EntityPath = EventHubName
        };

        eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

        await SendMessagesToEventHub(100);

        await eventHubClient.CloseAsync();

        Console.WriteLine("Press ENTER to exit.");
        Console.ReadLine();
    }
    ```

4. Добавьте новый метод с именем `SendMessagesToEventHub` в класс `Program`, как показано далее:

    ```csharp
    // Uses the event hub client to send 100 messages to the event hub.
    private static async Task SendMessagesToEventHub(int numMessagesToSend)
    {
        for (var i = 0; i < numMessagesToSend; i++)
        {
            try
            {
                var message = $"Message {i}";
                Console.WriteLine($"Sending message: {message}");
                await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
            }
            catch (Exception exception)
            {
                Console.WriteLine($"{DateTime.Now} > Exception: {exception.Message}");
            }

            await Task.Delay(10);
        }

        Console.WriteLine($"{numMessagesToSend} messages sent.");
    }
    ```

5. В метод `Main` в классе `Program` добавьте следующий код.

    ```csharp
    MainAsync(args).GetAwaiter().GetResult();
    ```

   Вот как будет выглядеть файл Program.cs.

    ```csharp
    namespace SampleSender
    {
        using System;
        using System.Text;
        using System.Threading.Tasks;
        using Microsoft.Azure.EventHubs;

        public class Program
        {
            private static EventHubClient eventHubClient;
            private const string EventHubConnectionString = "{Event Hubs connection string}";
            private const string EventHubName = "{Event Hub path/name}";

            public static void Main(string[] args)
            {
                MainAsync(args).GetAwaiter().GetResult();
            }

            private static async Task MainAsync(string[] args)
            {
                // Creates an EventHubsConnectionStringBuilder object from the connection string, and sets the EntityPath.
                // Typically, the connection string should have the entity path in it, but for the sake of this simple scenario
                // we are using the connection string from the namespace.
                var connectionStringBuilder = new EventHubsConnectionStringBuilder(EventHubConnectionString)
                {
                    EntityPath = EventHubName
                };

                eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());

                await SendMessagesToEventHub(100);

                await eventHubClient.CloseAsync();

                Console.WriteLine("Press ENTER to exit.");
                Console.ReadLine();
            }

            // Uses the event hub client to send 100 messages to the event hub.
            private static async Task SendMessagesToEventHub(int numMessagesToSend)
            {
                for (var i = 0; i < numMessagesToSend; i++)
                {
                    try
                    {
                        var message = $"Message {i}";
                        Console.WriteLine($"Sending message: {message}");
                        await eventHubClient.SendAsync(new EventData(Encoding.UTF8.GetBytes(message)));
                    }
                    catch (Exception exception)
                    {
                        Console.WriteLine($"{DateTime.Now} > Exception: {exception.Message}");
                    }

                    await Task.Delay(10);
                }

                Console.WriteLine($"{numMessagesToSend} messages sent.");
            }
        }
    }
    ```

6. Запустите программу и убедитесь в отсутствии ошибок.

## <a name="receive-events"></a>Получение событий
В этом разделе показано, как создать консольное приложение .NET Core для получения сообщений из концентратора событий с помощью [узла обработчика событий](event-hubs-event-processor-host.md). [Узел обработчика событий](event-hubs-event-processor-host.md) представляет собой класс .NET, который упрощает прием событий из концентраторов событий путем управления постоянными контрольными точками и одновременно принимает сообщения от этих концентраторов событий. С помощью класса EventProcessorHost можно разделить события между несколькими получателями даже в том случае, если они размещены в разных узлах. В этом примере показано, как использовать EventProcessorHost для одного получателя.
> [!NOTE]
> Вы можете скачать это краткое руководство в качестве примера с сайта [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/SampleEphReceiver), заменить строки `EventHubConnectionString`, а также `EventHubName`, `StorageAccountName`, `StorageAccountKey` и `StorageContainerName` значениями для своего концентратора событий и выполнить этот пример. Или следуйте инструкциям из этого руководства, чтобы создать собственное решение.

[!INCLUDE [event-hubs-create-storage](../../includes/event-hubs-create-storage.md)]

### <a name="create-a-console-application"></a>Создание консольного приложение

Запустите среду Visual Studio. В меню **Файл** последовательно выберите пункты **Создать** и **Проект**. Создайте консольное приложение .NET Core.

![Новый проект для получения](./media/event-hubs-dotnet-standard-getstarted-receive-eph/netcorercv.png)

### <a name="add-the-event-hubs-nuget-package"></a>Добавление пакета NuGet для Центров событий

Чтобы добавить в проект пакеты NuGet стандартной библиотеки .NET [**Microsoft.Azure.EventHubs**](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) и [**Microsoft.Azure.EventHubs.Processor**](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor/), выполните следующие действия: 

1. Щелкните созданный проект правой кнопкой мыши и выберите **Управление пакетами NuGet**.
2. Откройте вкладку **Обзор**, а затем выполните поиск по фразе **Microsoft.Azure.EventHubs** и выберите пакет **Microsoft.Azure.EventHubs**. Щелкните **Установить** , чтобы выполнить установку, а затем закройте это диалоговое окно.
3. Повторите шаги 1 и 2 и установите пакет **Microsoft.Azure.EventHubs.Processor**.

### <a name="implement-the-ieventprocessor-interface"></a>Реализация интерфейса IEventProcessor

1. В обозревателе решений щелкните правой кнопкой мыши проект, выберите **Добавить**, а затем щелкните **Класс**. Назовите новый класс **SimpleEventProcessor**.

2. Откройте файл SimpleEventProcessor.cs и добавьте в его начало следующие операторы `using`.

    ```csharp
    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.EventHubs.Processor;
    using System.Threading.Tasks;
    ```

3. Реализовать интерфейс `IEventProcessor`. Замените все содержимое класса `SimpleEventProcessor` следующим кодом:

    ```csharp
    public class SimpleEventProcessor : IEventProcessor
    {
        public Task CloseAsync(PartitionContext context, CloseReason reason)
        {
            Console.WriteLine($"Processor Shutting Down. Partition '{context.PartitionId}', Reason: '{reason}'.");
            return Task.CompletedTask;
        }

        public Task OpenAsync(PartitionContext context)
        {
            Console.WriteLine($"SimpleEventProcessor initialized. Partition: '{context.PartitionId}'");
            return Task.CompletedTask;
        }

        public Task ProcessErrorAsync(PartitionContext context, Exception error)
        {
            Console.WriteLine($"Error on Partition: {context.PartitionId}, Error: {error.Message}");
            return Task.CompletedTask;
        }

        public Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
        {
            foreach (var eventData in messages)
            {
                var data = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
                Console.WriteLine($"Message received. Partition: '{context.PartitionId}', Data: '{data}'");
            }

            return context.CheckpointAsync();
        }
    }
    ```

### <a name="update-the-main-method-to-use-simpleeventprocessor"></a>Обновление метода Main для использования класса SimpleEventProcessor

1. Добавьте следующие операторы `using` в начало файла program.cs.

    ```csharp
    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.EventHubs.Processor;
    using System.Threading.Tasks;
    ```

2. Добавьте в класс `Program` константы для строки подключения концентраторов событий, имени концентратора событий, имени контейнера учетной записи хранения, имени учетной записи хранения и ключа учетной записи хранения. Добавьте следующий код, заменив заполнители соответствующими значениями:

    ```csharp
    private const string EventHubConnectionString = "{Event Hubs connection string}";
    private const string EventHubName = "{Event Hub path/name}";
    private const string StorageContainerName = "{Storage account container name}";
    private const string StorageAccountName = "{Storage account name}";
    private const string StorageAccountKey = "{Storage account key}";

    private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);
    ```   

3. Добавьте новый метод с именем `MainAsync` в класс `Program`, как показано далее:

    ```csharp
    private static async Task MainAsync(string[] args)
    {
        Console.WriteLine("Registering EventProcessor...");

        var eventProcessorHost = new EventProcessorHost(
            EventHubName,
            PartitionReceiver.DefaultConsumerGroupName,
            EventHubConnectionString,
            StorageConnectionString,
            StorageContainerName);

        // Registers the Event Processor Host and starts receiving messages
        await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

        Console.WriteLine("Receiving. Press ENTER to stop worker.");
        Console.ReadLine();

        // Disposes of the Event Processor Host
        await eventProcessorHost.UnregisterEventProcessorAsync();
    }
    ```

3. Добавьте в метод `Main` следующую строку кода:

    ```csharp
    MainAsync(args).GetAwaiter().GetResult();
    ```

    Вот как будет выглядеть файл Program.cs.

    ```csharp
    namespace SampleEphReceiver
    {

        public class Program
        {
            private const string EventHubConnectionString = "{Event Hubs connection string}";
            private const string EventHubName = "{Event Hub path/name}";
            private const string StorageContainerName = "{Storage account container name}";
            private const string StorageAccountName = "{Storage account name}";
            private const string StorageAccountKey = "{Storage account key}";

            private static readonly string StorageConnectionString = string.Format("DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}", StorageAccountName, StorageAccountKey);

            public static void Main(string[] args)
            {
                MainAsync(args).GetAwaiter().GetResult();
            }

            private static async Task MainAsync(string[] args)
            {
                Console.WriteLine("Registering EventProcessor...");

                var eventProcessorHost = new EventProcessorHost(
                    EventHubName,
                    PartitionReceiver.DefaultConsumerGroupName,
                    EventHubConnectionString,
                    StorageConnectionString,
                    StorageContainerName);

                // Registers the Event Processor Host and starts receiving messages
                await eventProcessorHost.RegisterEventProcessorAsync<SimpleEventProcessor>();

                Console.WriteLine("Receiving. Press ENTER to stop worker.");
                Console.ReadLine();

                // Disposes of the Event Processor Host
                await eventProcessorHost.UnregisterEventProcessorAsync();
            }
        }
    }
    ```

4. Запустите программу и убедитесь в отсутствии ошибок.


## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими статьями:

- [Примеры управления доступом на основе ролей в Azure (Azure RBAC)](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). 
    
    В этих примерах используется библиотека **Microsoft.Azure.EventHubs** старой версии, но вы можете легко обновить ее до библиотеки **Azure.Messaging.EventHubs** последней версии. Чтобы начать использовать в примере библиотеку новой версии вместо старой, ознакомьтесь с руководством по [переходу с Microsoft.Azure.EventHubs на Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).
- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Функции и терминология в Центрах событий Azure](event-hubs-features.md)
- [Часто задаваемые вопросы о Центрах событий](event-hubs-faq.yml)


