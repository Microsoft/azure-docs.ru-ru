---
title: Обновление информации о запасах с помощью портала Azure, разделов и подписок
description: Из этого руководства вы узнаете, как отправлять и получать сообщения из раздела и подписки, а также как добавлять и использовать правила фильтрации с помощью .NET
author: spelluru
ms.author: spelluru
ms.date: 10/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: d0a94f346f9d3cf7a05a1ca6e1b37d4d008f3e75
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102179669"
---
# <a name="tutorial-update-inventory-using-azure-portal-and-topicssubscriptions"></a>Руководство. Обновление информации о запасах с помощью портала Azure, разделов и подписок

Служебная шина Microsoft Azure — это мультитенантная облачная служба для обмена сообщениями, которая позволяет пересылать информацию между приложениями и службами. Асинхронная работа обеспечивает гибкий обмен сообщениями через брокер и обработку сообщений в порядке поступления, а также возможность публикации и подписки. В этом руководстве объясняется, как использовать подписки и темы Служебной шины Azure для обновления списка розничных товаров через каналы публикации (подписки) с помощью портала Azure и .NET.

В этом руководстве описано следующее.
> [!div class="checklist"]
> * создание раздела служебной шины и одной или нескольких подписок на этот раздел с помощью портала Azure;
> * добавление фильтров раздела в коде .NET;
> * создание двух сообщений с разным содержимым;
> * отправка сообщений и проверка их поступления в ожидаемые подписки;
> * получение сообщений из подписок.

Такой сценарий, например, отлично подходит для обновления сведений об ассортименте товаров в нескольких магазинах розничной торговли. В этом сценарии каждый магазин или группа магазинов получают сообщения, позволяющие им обновить ассортимент. В этом руководстве показано, как реализовать этот сценарий с помощью подписок и фильтров. Сначала вы создадите раздел с тремя подписками, добавите в него правила и фильтры, а затем отправите и получите сообщения через разделы и подписки.

![Раздел](./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png)

Если у вас еще нет подписки Azure, вы можете создать [бесплатная учетная запись][], прежде чем начинать работу.

## <a name="prerequisites"></a>предварительные требования

Для работы с этим руководством вам потребуются:

- [Visual Studio 2017 с обновлением 3 (версия 15.3, 26730.01)](https://www.visualstudio.com/vs) или более новая версия.
- [Пакет SDK для .NET Core](https://www.microsoft.com/net/download/windows) версии 2.0 или более новой.

## <a name="service-bus-topics-and-subscriptions"></a>Разделы и подписки служебной шины

Каждая [подписка на раздел](service-bus-messaging-overview.md#topics) может получать копию любого сообщения. Разделы полностью совместимы с очередями служебной шины на уровнях протоколов и семантики. Разделы служебной шины поддерживают широкий набор правил отбора и условий фильтра, а также позволяют создать действия для присвоения или изменения свойств сообщения. Каждый раз, когда срабатывает правило, создается новое сообщение. Чтобы узнать больше о правилах, фильтрах и действиях, перейдите по [этой ссылке](topic-filters.md).

[!INCLUDE [service-bus-create-namespace-portal](../../includes/service-bus-create-namespace-portal.md)]

[!INCLUDE [service-bus-create-topics-three-subscriptions-portal](../../includes/service-bus-create-topics-three-subscriptions-portal.md)]



## <a name="create-filter-rules-on-subscriptions"></a>Создание правил фильтрации по подпискам

Итак, вы уже подготовили пространство имен, разделы и подписки, также у вас есть необходимые учетные данные, а значит вы готовы создать правила фильтрации для этих подписок, чтобы отправлять и получать сообщения. Этот код можно изучить в [папке с примером на GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/azure-servicebus/TopicFilters).

## <a name="send-and-receive-messages"></a>Отправка и получение сообщений

Чтобы выполнить этот код, сделайте следующее:

1. В командной строке или в командной строке PowerShell выполните следующую команду, которая клонирует [репозиторий GitHub для Служебной шины](https://github.com/Azure/azure-service-bus/):

   ```shell
   git clone https://github.com/Azure/azure-service-bus.git
   ```

2. Перейдите к папке `azure-service-bus\samples\DotNet\Azure.Messaging.ServiceBus\BasicSendReceiveTutorialwithFilters` с примерами.

3. Найдите строку подключения, которую вы скопировали в Блокнот ранее в разделе "Получение учетных данных управления" этого руководства. Также здесь потребуется имя раздела, который вы создали в предыдущем разделе.

4. В командной строке введите следующую команду:

   ```shell
   dotnet build
   ```

5. Перейдите в папку `BasicSendReceiveTutorialwithFilters\bin\Debug\netcoreapp3.1`.

6. Введите приведенную ниже команду, чтобы запустить программу. Не забудьте ввести вместо `myConnectionString` значение, которое вы получили ранее, а вместо `myTopicName` — имя созданного раздела:

   ```shell
   dotnet BasicSendReceiveTutorialwithFilters.dll -ConnectionString "myConnectionString" -TopicName "myTopicName"
   ``` 
7. Следуйте инструкциям в консоли, чтобы выбрать процедуру создания фильтра. При создании фильтров необходимо также удалить стандартные фильтры. Если вы используете PowerShell или интерфейс командной строки, стандартный фильтр удалять не требуется, но при выполнении операций в коде это сделать необходимо. Команды консоли 1 и 3 позволяют управлять фильтрами для ранее созданных подписок:

   - Выполните команду 1, чтобы удалить стандартные фильтры.
   - Выполните команду 2, чтобы добавить собственные фильтры.
   - Выполните команду 3, если нужно удалить свои фильтры. Обратите внимание, что это действие не восстанавливает стандартные фильтры.

     ![Пример выходных данных команды 2](./media/service-bus-tutorial-topics-subscriptions-portal/create-rules.png)

8. Завершив создание фильтров, вы можете отправлять сообщения. Нажмите клавишу 4 и наблюдайте, как в раздел отправляются 10 сообщений:

    ![Передача выходных данных](./media/service-bus-tutorial-topics-subscriptions-portal/send-output.png)

9. Нажмите клавишу 5 и наблюдайте за получением этих сообщений. Если не все 10 сообщений будут успешно получены, нажмите клавишу "M" для отображения меню и снова нажмите 5.

    ![Выходные данные операции получения](./media/service-bus-tutorial-topics-subscriptions-portal/receive-output.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите пространство имен и раздел, если они вам больше не нужны. Для этого выберите эти ресурсы на портале и щелкните **Удалить**.

## <a name="understand-the-sample-code"></a>Разбор примера кода

Этот раздел содержит дополнительные сведения о работе этого примера кода.

### <a name="get-connection-string-and-topic"></a>Получение строки подключения и раздела

Прежде всего этот код объявляет набор переменных, которые управляют выполнением остальных частей программы.

```csharp
string ServiceBusConnectionString;
string TopicName;

static string[] Subscriptions = { "S1", "S2", "S3" };
static IDictionary<string, string[]> SubscriptionFilters = new Dictionary<string, string[]> {
    { "S1", new[] { "StoreId IN('Store1', 'Store2', 'Store3')", "StoreId = 'Store4'"} },
    { "S2", new[] { "sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'" } },
    { "S3", new[] { "sys.To NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8')" } }
};
// You can have only have one action per rule and this sample code supports only one action for the first filter, which is used to create the first rule. 
static IDictionary<string, string> SubscriptionAction = new Dictionary<string, string> {
    { "S1", "" },
    { "S2", "" },
    { "S3", "SET sys.Label = 'SalesEvent'"  }
};
static string[] Store = { "Store1", "Store2", "Store3", "Store4", "Store5", "Store6", "Store7", "Store8", "Store9", "Store10" };
static string SysField = "sys.To";
static string CustomField = "StoreId";
static int NrOfMessagesPerStore = 1; // Send at least 1.
```

Строка подключения и имя раздела передаются через параметры командной строки, как показано здесь, и затем считываются в метод `Main()`:

```csharp
static void Main(string[] args)
{
    string ServiceBusConnectionString = "";
    string TopicName = "";

    for (int i = 0; i < args.Length; i++)
    {
        if (args[i] == "-ConnectionString")
        {
            Console.WriteLine($"ConnectionString: {args[i + 1]}");
            ServiceBusConnectionString = args[i + 1]; // Alternatively enter your connection string here.
        }
        else if (args[i] == "-TopicName")
        {
            Console.WriteLine($"TopicName: {args[i + 1]}");
            TopicName = args[i + 1]; // Alternatively enter your queue name here.
        }
    }

    if (ServiceBusConnectionString != "" && TopicName != "")
    {
        Program P = StartProgram(ServiceBusConnectionString, TopicName);
        P.PresentMenu().GetAwaiter().GetResult();
    }
    else
    {
        Console.WriteLine("Specify -Connectionstring and -TopicName to execute the example.");
        Console.ReadKey();
    }
}
```

### <a name="remove-default-filters"></a>Удаление стандартных фильтров

При создании подписки в Служебной шине создается стандартный фильтр для каждой подписки. Этот фильтр позволяет получать каждое сообщение, отправленное в раздел. Если вы намерены использовать пользовательские фильтры, этот стандартный фильтр можно удалить, как показано в следующем коде:

```csharp
private async Task RemoveDefaultFilters()
{
    Console.WriteLine($"Starting to remove default filters.");

    try
    {
        var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
        foreach (var subscription in Subscriptions)
        {
            await client.DeleteRuleAsync(TopicName, subscription, CreateRuleOptions.DefaultRuleName);
            Console.WriteLine($"Default filter for {subscription} has been removed.");
        }

        Console.WriteLine("All default Rules have been removed.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="create-filters"></a>Создание фильтров

В следующем коде добавляются пользовательские фильтры, которые описаны в этом руководстве:

```csharp
private async Task CreateCustomFilters()
{
    try
    {
        for (int i = 0; i < Subscriptions.Length; i++)
        {
            var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
            string[] filters = SubscriptionFilters[Subscriptions[i]];
            if (filters[0] != "")
            {
                int count = 0;
                foreach (var myFilter in filters)
                {
                    count++;

                    string action = SubscriptionAction[Subscriptions[i]];
                    if (action != "")
                    {
                        await client.CreateRuleAsync(TopicName, Subscriptions[i], new CreateRuleOptions
                        {
                            Filter = new SqlRuleFilter(myFilter),
                            Action = new SqlRuleAction(action),
                            Name = $"MyRule{count}"
                        });
                    }
                    else
                    {
                        await client.CreateRuleAsync(TopicName, Subscriptions[i], new CreateRuleOptions
                        {
                            Filter = new SqlRuleFilter(myFilter),
                            Name = $"MyRule{count}"
                        });
                    }
                }
            }

            Console.WriteLine($"Filters and actions for {Subscriptions[i]} have been created.");
        }

        Console.WriteLine("All filters and actions have been created.\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }

    await PresentMenu();
}
```

### <a name="remove-your-custom-created-filters"></a>Удаление созданных пользовательских фильтров

Если нужно удалить из подписки все фильтры, воспользуйтесь следующим примером кода:

```csharp
private async Task CleanUpCustomFilters()
{
    foreach (var subscription in Subscriptions)
    {
        try
        {
            var client = new ServiceBusAdministrationClient(ServiceBusConnectionString);
            IAsyncEnumerator<RuleProperties> rules = client.GetRulesAsync(TopicName, subscription).GetAsyncEnumerator();
            while (await rules.MoveNextAsync())
            {
                await client.DeleteRuleAsync(TopicName, subscription, rules.Current.Name);
                Console.WriteLine($"Rule {rules.Current.Name} has been removed.");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
    Console.WriteLine("All default filters have been removed.\n");

    await PresentMenu();
}
```

### <a name="send-messages"></a>Отправка сообщений

Отправка сообщений в раздел происходит так же, как и отправка сообщений в очередь. В этом примере показано, как отправлять сообщения, используя список задач и асинхронную обработку:

```csharp
public async Task SendMessages()
{
    try
    {
        await using var client = new ServiceBusClient(ServiceBusConnectionString);
        var taskList = new List<Task>();
        for (int i = 0; i < Store.Length; i++)
        {
            taskList.Add(SendItems(client, Store[i]));
        }

        await Task.WhenAll(taskList);
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }
    Console.WriteLine("\nAll messages sent.\n");
}

private async Task SendItems(ServiceBusClient client, string store)
{
    // create the sender
    ServiceBusSender tc = client.CreateSender(TopicName);

    for (int i = 0; i < NrOfMessagesPerStore; i++)
    {
        Random r = new Random();
        Item item = new Item(r.Next(5), r.Next(5), r.Next(5));

        // Note the extension class which is serializing an deserializing messages
        ServiceBusMessage message = item.AsMessage();
        message.To = store;
        message.ApplicationProperties.Add("StoreId", store);
        message.ApplicationProperties.Add("Price", item.GetPrice().ToString());
        message.ApplicationProperties.Add("Color", item.GetColor());
        message.ApplicationProperties.Add("Category", item.GetItemCategory());

        await tc.SendMessageAsync(message);
        Console.WriteLine($"Sent item to Store {store}. Price={item.GetPrice()}, Color={item.GetColor()}, Category={item.GetItemCategory()}"); ;
    }
}
```

### <a name="receive-messages"></a>Получение сообщений

Сообщения принимаются обратно через список задач. Этот код использует стратегию пакетной обработки. Пакетную обработку можно применять как для отправки, так и для получения. Но в нашем примере реализовано только пакетное получение. Для реальных систем этот цикл обычно не прерывается, а повторяется неограниченно долго и с более длительным периодом ожидания, например в одну минуту. На протяжении этого периода поддерживается подключение к брокеру. Когда появится новое сообщение, программа немедленно возвращает их и создает новый вызов для получения данных. Такой подход называется методом *продолжительного опроса*. Еще чаще применяется средство переноса полученных данных, которое реализовано в [этом кратком руководстве](service-bus-quickstart-portal.md) и еще нескольких примерах в репозитории.

```csharp
public async Task Receive()
{
    var taskList = new List<Task>();
    for (var i = 0; i < Subscriptions.Length; i++)
    {
        taskList.Add(this.ReceiveMessages(Subscriptions[i]));
    }

    await Task.WhenAll(taskList);
}

private async Task ReceiveMessages(string subscription)
{
    await using var client = new ServiceBusClient(ServiceBusConnectionString);
    ServiceBusReceiver receiver = client.CreateReceiver(TopicName, subscription);

    // In reality you would not break out of the loop like in this example but would keep looping. The receiver keeps the connection open
    // to the broker for the specified amount of seconds and the broker returns messages as soon as they arrive. The client then initiates
    // a new connection. So in reality you would not want to break out of the loop. 
    // Also note that the code shows how to batch receive, which you would do for performance reasons. For convenience you can also always
    // use the regular receive pump which we show in our Quick Start and in other github samples.
    while (true)
    {
        try
        {
            //IList<Message> messages = await receiver.ReceiveAsync(10, TimeSpan.FromSeconds(2));
            // Note the extension class which is serializing an deserializing messages and testing messages is null or 0.
            // If you think you did not receive all messages, just press M and receive again via the menu.
            IReadOnlyList<ServiceBusReceivedMessage> messages = await receiver.ReceiveMessagesAsync(maxMessages: 100);

            if (messages.Any())
            {
                foreach (ServiceBusReceivedMessage message in messages)
                {
                    lock (Console.Out)
                    {
                        Item item = message.As<Item>();
                        IReadOnlyDictionary<string, object> myApplicationProperties = message.ApplicationProperties;
                        Console.WriteLine($"StoreId={myApplicationProperties["StoreId"]}");
                        if (message.Subject != null)
                        {
                            Console.WriteLine($"Subject={message.Subject}");
                        }
                        Console.WriteLine(
                            $"Item data: Price={item.GetPrice()}, Color={item.GetColor()}, Category={item.GetItemCategory()}");
                    }

                    await receiver.CompleteMessageAsync(message);
                }
            }
            else
            {
                break;
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
}
```

> [!NOTE]
> Вы можете управлять ресурсами служебной шины с помощью [обозревателя служебной шины](https://github.com/paolosalvatori/ServiceBusExplorer/). Обозреватель служебной шины позволяет без труда подключаться к пространству имен служебной шины и управлять сущностями обмена сообщениями. Средство предоставляет дополнительные возможности, например функции импорта и экспорта или возможность проверять разделы, очереди, подписки, службы ретрансляции, центры уведомлений и концентраторы событий. 

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим руководством вы подготовили ресурсы с помощью портала Azure, а затем отправили и получили сообщения через раздел Служебной шины и подписки на него. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * создание раздела служебной шины и одной или нескольких подписок на этот раздел с помощью портала Azure;
> * добавление фильтров раздела в коде .NET;
> * создание двух сообщений с разным содержимым;
> * отправка сообщений и проверка их поступления в ожидаемые подписки;
> * получение сообщений из подписок.

Чтобы изучить дополнительные примеры отправки и получения сообщений, перейдите к [описанию примеров для служебной шины на GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted).

Следующее руководство содержит дополнительные сведения об использовании возможностей публикации и подписки в cлужебной шине Azure.

> [!div class="nextstepaction"]
> [Реагирование на события с помощью службы "Сетка событий"](service-bus-to-event-grid-integration-example.md)

[бесплатная учетная запись]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Azure portal]: https://portal.azure.com/

[connection-string]: ./media/service-bus-tutorial-topics-subscriptions-portal/connection-string.png
[service-bus-flow]: ./media/service-bus-tutorial-topics-subscriptions-portal/about-service-bus-topic.png
