---
title: Архитектура push-уведомлений Enterprise
description: Узнайте, как использовать центры уведомлений Azure в корпоративной среде
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 903023e9-9347-442a-924b-663af85e05c6
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 018315b7ed468e24fb922337848d14703ffdcd4d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89003632"
---
# <a name="enterprise-push-architectural-guidance"></a>Руководство по архитектуре push-уведомлений

Сегодня предприятия постепенно переходят к созданию мобильных приложений для конечных пользователей (внешних) или сотрудников (внутренних). У них есть серверные системы, размещенные на мэйнфреймах, или бизнес-приложения, которые нужно интегрировать в архитектуру приложений для мобильных устройств. В этом руководстве рассказывается об оптимальном подходе к такой интеграции и излагаются рекомендации относительно возможных решений для распространенных сценариев.

Одно из наиболее распространенных требований — возможность отправки push-уведомлений пользователям с помощью мобильных приложений при определенных событиях в серверной системе. Например, клиент банка, которому назначено банковское приложение банка на iPhone, хочет получать уведомления о том, что дебет выполняется выше определенной суммы из учетной записи или сценария интрасети, где сотрудник из финансового отдела, имеющего приложение для утверждения бюджета на Windows Phone хочет получать уведомления при получении запроса на утверждение.

Обслуживание банковского счета и обработка утверждений, как правило, осуществляются в серверной системе, которая должна инициировать отправку push-уведомлений пользователю. Таких серверных систем может быть несколько, а для реализации push-уведомлений, отправка которых инициируется наступлением определенных событий, все они должны быть созданы на основе одной и той же логики. Сложность в том, как интегрировать несколько серверных систем с одной системой push-уведомлений, где конечные пользователи могут подписываться на различные уведомления и даже использовать различные мобильные приложения. Например, в случае интрасети, где одно мобильное приложение может получать уведомления из нескольких таких серверных систем. Серверным системам неизвестна или не должна быть известна семантика или технология системы отправки push-уведомлений, поэтому в таких случаях традиционным решением было внедрение компонента, который опрашивает серверные системы, чтобы определить, не произошло ли то или иное событие, и отвечает за отправку push-уведомлений клиенту.

Лучшим решением является использование служебной шины Azure — модели разделов и подписок, которая обеспечивает простоту и масштабируемость.

Ниже описана общая архитектура решения. Изложенная здесь информация касается общего случая нескольких мобильных приложений, однако она в равной мере применима в случае, если такое приложение только одно.

## <a name="architecture"></a>Architecture

![Схема архитектуры Enterprise, показывающая поток через события, подписки и push-сообщения.][1]

Основной элемент на этой схеме архитектуры — это служебная шина Azure, которая предоставляет модель программирования разделов и подписок (подробнее об этом написано в статье [Как использовать разделы и подписки служебной шины]). В этом случае получатель (как правило, [мобильная служба Azure], инициирующая push-уведомления в мобильные приложения) не получает сообщения непосредственно из серверных систем, а промежуточный уровень абстракции, предоставляемый служебной [шиной Azure], позволяет мобильному серверу получать сообщения из одной или нескольких серверных систем. Раздел "Служебная шина" необходимо создать для каждой из серверных систем, например "Бухгалтерия", "Отдел кадров" и "Финансы", по сути являющиеся "разделами", которые инициируют отправку сообщений в виде push-уведомлений. Серверные системы отправляют сообщения в эти разделы. Мобильный внутренний сервер может подписаться на один или несколько таких разделов, создав подписку на служебную шину. Это дает серверной части мобильной службы возможность получать уведомления от соответствующей серверной системы. Этот сервер продолжает прослушивать сообщения по своим подпискам, и как только сообщение поступает, он отправляет его в виде уведомления в свой центр уведомлений. В конечном итоге центры уведомлений доставляют сообщение в мобильное приложение. Ниже приведен список основных компонентов.

1. Серверные системы (бизнес-системы или устаревшие системы)
   * Создает раздел Service Bus
   * Отправляет сообщение
1. Серверная часть мобильной службы
   * Создает подписку на службу
   * Получает сообщение (из серверной системы)
   * Отправляет уведомление клиентам (через центр уведомлений Azure)
1. Мобильное приложение
   * Получает и отображает уведомление

### <a name="benefits"></a>Преимущества

1. Разделение на получателя (мобильное приложение или служба через центр уведомлений) и отправителя (серверные системы) позволяет интегрировать дополнительные серверные системы с минимальными изменениями.
1. Упрощается ситуация, когда необходимо, чтобы уведомления о событиях в одной или нескольких серверных системах получало несколько мобильных приложений.  

## <a name="sample"></a>Образец

### <a name="prerequisites"></a>Предварительные требования

Чтобы ознакомиться с основными понятиями и общими процедурами создания и настройки, необходимо изучить следующие руководства.

1. [Начало работы с разделами служебной шины]. В этом руководстве подробно описывается работа с разделами и подписками служебной шины, создание пространства имен для хранения разделов и подписок, отправка им сообщений и получение сообщений от них.
2. [Начало работы с Центрами уведомлений для приложений универсальной платформы Windows]. В этом руководстве рассказывается о том, как настроить приложение Магазина Windows и использовать центры уведомлений для регистрации и последующего получения уведомлений.

### <a name="sample-code"></a>Пример кода

Полный пример кода доступен в [коллекции примеров центра уведомлений]. Он состоит из трех компонентов:

1. **EnterprisePushBackendSystem**

    а. Этот проект использует пакет NuGet **WindowsAzure.ServiceBus** и основан на материале, изложенном в статье [Начало работы с разделами служебной шины].

    b. Это простое консольное приложение на C# для моделирования бизнес-систем, которое инициирует доставку сообщений мобильному приложению.

    ```csharp
    static void Main(string[] args)
    {
        string connectionString =
            CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

        // Create the topic
        CreateTopic(connectionString);

        // Send message
        SendMessage(connectionString);
    }
    ```

    c. `CreateTopic` используется для создания раздела служебной шины.

    ```csharp
    public static void CreateTopic(string connectionString)
    {
        // Create the topic if it does not exist already

        var namespaceManager =
            NamespaceManager.CreateFromConnectionString(connectionString);

        if (!namespaceManager.TopicExists(sampleTopic))
        {
            namespaceManager.CreateTopic(sampleTopic);
        }
    }
    ```

    d. `SendMessage` используется для отправки сообщений в этот раздел служебной шины. Данный код просто периодически отправляет набор случайных сообщений в этот раздел. Как правило, эту функцию выполняет серверная система, которая отправляет сообщения при наступлении события.

    ```csharp
    public static void SendMessage(string connectionString)
    {
        TopicClient client =
            TopicClient.CreateFromConnectionString(connectionString, sampleTopic);

        // Sends random messages every 10 seconds to the topic
        string[] messages =
        {
            "Employee Id '{0}' has joined.",
            "Employee Id '{0}' has left.",
            "Employee Id '{0}' has switched to a different team."
        };

        while (true)
        {
            Random rnd = new Random();
            string employeeId = rnd.Next(10000, 99999).ToString();
            string notification = String.Format(messages[rnd.Next(0,messages.Length)], employeeId);

            // Send Notification
            BrokeredMessage message = new BrokeredMessage(notification);
            client.Send(message);

            Console.WriteLine("{0} Message sent - '{1}'", DateTime.Now, notification);

            System.Threading.Thread.Sleep(new TimeSpan(0, 0, 10));
        }
    }
    ```
2. **ReceiveAndSendNotification**

    а. В этом проекте используются *WindowsAzure. servicebus* и **Microsoft. Web. веб-задания. Публикация** пакетов NuGet и основана на [программировании на основе публикации и подменю служебной шины].

    b. Приведенное ниже консольное приложение запускается в качестве [веб-задания Azure], так как оно должно выполняться непрерывно для ожидания передачи сообщений из бизнес-систем и серверных систем. Это приложение входит в состав серверной части мобильного приложения.

    ```csharp
    static void Main(string[] args)
    {
        string connectionString =
                 CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

        // Create the subscription that receives messages
        CreateSubscription(connectionString);

        // Receive message
        ReceiveMessageAndSendNotification(connectionString);
    }
    ```

    c. `CreateSubscription` используется для создания подписки служебной шины для раздела, в который серверная система отправляет сообщения. В зависимости от бизнес-сценария этот компонент создает одну или несколько подписок на соответствующие разделы (например, одни будут получать сообщения из системы "Отдел кадров", другие — из системы "Финансы" и т. д.)

    ```csharp
    static void CreateSubscription(string connectionString)
    {
        // Create the subscription if it does not exist already
        var namespaceManager =
            NamespaceManager.CreateFromConnectionString(connectionString);

        if (!namespaceManager.SubscriptionExists(sampleTopic, sampleSubscription))
        {
            namespaceManager.CreateSubscription(sampleTopic, sampleSubscription);
        }
    }
    ```

    d. `ReceiveMessageAndSendNotification` используется для чтения сообщения из раздела с использованием подписки. Если операция считывания выполнена успешно, создается уведомление (в этом примере сценария — собственное всплывающее уведомление Windows), которое необходимо отправить в мобильное приложение с помощью Центров уведомлений Azure.

    ```csharp
    static void ReceiveMessageAndSendNotification(string connectionString)
    {
        // Initialize the Notification Hub
        string hubConnectionString = CloudConfigurationManager.GetSetting
                ("Microsoft.NotificationHub.ConnectionString");
        hub = NotificationHubClient.CreateClientFromConnectionString
                (hubConnectionString, "enterprisepushservicehub");

        SubscriptionClient Client =
            SubscriptionClient.CreateFromConnectionString
                    (connectionString, sampleTopic, sampleSubscription);

        Client.Receive();

        // Continuously process messages received from the subscription
        while (true)
        {
            BrokeredMessage message = Client.Receive();
            var toastMessage = @"<toast><visual><binding template=""ToastText01""><text id=""1"">{messagepayload}</text></binding></visual></toast>";

            if (message != null)
            {
                try
                {
                    Console.WriteLine(message.MessageId);
                    Console.WriteLine(message.SequenceNumber);
                    string messageBody = message.GetBody<string>();
                    Console.WriteLine("Body: " + messageBody + "\n");

                    toastMessage = toastMessage.Replace("{messagepayload}", messageBody);
                    SendNotificationAsync(toastMessage);

                    // Remove message from subscription
                    message.Complete();
                }
                catch (Exception)
                {
                    // Indicate a problem, unlock message in subscription
                    message.Abandon();
                }
            }
        }
    }
    static async void SendNotificationAsync(string message)
    {
        await hub.SendWindowsNativeNotificationAsync(message);
    }
    ```

    д) Чтобы опубликовать это приложение в виде **веб-задания**, щелкните правой кнопкой мыши решение в Visual Studio и выберите **Publish as WebJob** (Опубликовать как веб-задание).

    ![Снимок экрана параметров щелчка правой кнопкой мыши, отображаемых с помощью команды "опубликовать как веб-задание Azure", описанной в Red.][2]

    е) Выберите профиль публикации и создайте новый веб-сайт Azure для размещения веб-задания (если он еще не существует), а когда веб-сайт будет создан, щелкните **Опубликовать**.

    :::image type="complex" source="./media/notification-hubs-enterprise-push-architecture/PublishAsWebJob.png" alt-text="Снимок экрана, показывающий рабочий процесс создания сайта в Azure.":::
    Снимок экрана: диалоговое окно "Публикация веб-сайта" с выбранным параметром Microsoft Azure веб-сайтов, зеленая стрелка, указывающая на диалоговое окно "Выбор существующего веб-сайта" с новым параметром, выделенным красным цветом, и зеленая стрелка, указывающая на диалоговое окно "Создание сайта в Microsoft Azure" с именем сайта и параметрами создания, выделенными красным цветом.
    :::image-end:::

    ж. Настройте непрерывное выполнение задания, чтобы при входе на [портал Azure] отображалось примерно следующее:

    ![Снимок экрана портала Azure с отображенными веб-заданиями Enterprise push-уведомлений, а также значениями имени, расписания и журналов, выделенными красным цветом.][4]

3. **EnterprisePushMobileApp**

    а. Это приложение Магазина Windows, которое будет получать всплывающие уведомления из задания WebJob, выполняющегося в составе серверной части мобильной службы, и отображать это уведомление. Оно основано на материалах [руководства по центрам уведомлений для Windows Universal].  

    b. Убедитесь, что в приложении включено получение всплывающих уведомлений.

    c. Убедитесь, что при запуске приложения вызывается следующий код регистрации Центров уведомлений (после замены значений `HubName` и `DefaultListenSharedAccessSignature`).

    ```csharp
    private async void InitNotificationsAsync()
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        var hub = new NotificationHub("[HubName]", "[DefaultListenSharedAccessSignature]");
        var result = await hub.RegisterNativeAsync(channel.Uri);

        // Displays the registration ID so you know it was successful
        if (result.RegistrationId != null)
        {
            var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
    }
    ```

### <a name="running-the-sample"></a>Выполнение примера

1. Убедитесь, что веб-задание успешно запущено и для него запланировано непрерывное выполнение.
2. Запустите **ентерприсепушмобилеапп**, который запускает приложение для Магазина Windows.
3. Запустите консольное приложение **EnterprisePushBackendSystem**, которое имитирует серверную часть бизнес-системы. Оно начнет отправлять сообщения. Вы должны увидеть всплывающие уведомления, как на рисунке ниже.

    ![Снимок экрана консоли, на которой работает приложение системы push-уведомлений Enterprise и сообщение, отправленное приложением.][5]

4. Первоначально сообщения были отправлены в разделы служебной шины, которые отслеживаются по подпискам служебной шины в веб-задании. При получении сообщения было создано уведомление, которое затем было отправлено в мобильное приложение. Чтобы убедиться в правильности выполнения обработки, можно просмотреть журналы веб-заданий, перейдя по ссылке "Журналы" на [портале Azure] для соответствующего веб-задания:

    ![Снимок экрана диалогового окна "сведения о непрерывном веб-задании" с сообщением, которое выделено красным цветом.][6]

<!-- Images -->
[1]: ./media/notification-hubs-enterprise-push-architecture/architecture.png
[2]: ./media/notification-hubs-enterprise-push-architecture/WebJobsContextMenu.png
[3]: ./media/notification-hubs-enterprise-push-architecture/PublishAsWebJob.png
[4]: ./media/notification-hubs-enterprise-push-architecture/WebJob.png
[5]: ./media/notification-hubs-enterprise-push-architecture/Notifications.png
[6]: ./media/notification-hubs-enterprise-push-architecture/WebJobsLog.png

<!-- Links -->
[коллекции примеров центра уведомлений]: https://github.com/Azure/azure-notificationhubs-samples
[Мобильная служба Azure]: https://azure.microsoft.com/documentation/services/mobile-services/
[служебной шине Azure]: ../service-bus-messaging/service-bus-messaging-overview.md
[Использование разделов и подписок Service Bus]: ../service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions.md
[веб-задания Azure]: ../app-service/webjobs-create.md
[Учебник по центрам уведомлений для Windows Universal]: ./notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Портал Azure]: https://portal.azure.com/
