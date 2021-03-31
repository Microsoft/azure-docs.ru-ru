---
title: Отправка push-уведомлений на конкретные телефоны Windows с помощью Центров уведомлений Azure | Документация Майкрософт
description: Из этого руководства вы узнаете, как использовать Центры уведомлений Azure для отправки push-уведомлений на конкретные (не все сразу) устройства под управлением Windows Phone 8 или Windows Phone 8.1, зарегистрированные в серверной части приложения.
services: notification-hubs
documentationcenter: windows
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 42726bf5-cc82-438d-9eaa-238da3322d80
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows-phone
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: 67700ec643a27f8ae5c581fe9de8e3295a1925e5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88998243"
---
# <a name="tutorial-send-push-notifications-to-specific-windows-phones-using-azure-notification-hubs"></a>Руководство. Отправка push-уведомлений на конкретные телефоны Windows с помощью Центров уведомлений Azure

[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

В этом руководстве показано, как использовать Центры уведомлений Azure для отправки push-уведомлений на конкретные устройства под управлением Windows Phone 8 или Windows Phone 8.1. Если вы намерены использовать платформу Windows Phone 8.1 (без Silverlight), перейдите к версии этого руководства для [Windows Universal](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).

Для поддержки этого сценария вам необходимо включить один или несколько *тегов* при регистрации в Центре уведомлений. Если уведомления отправляются на тег, их получают все устройства, зарегистрированные для данного тега. Дополнительные сведения о тегах см. в статье [Маршрутизация и выражения тегов](notification-hubs-tags-segment-push-message.md).

> [!NOTE]
> Центры уведомлений Windows Phone SDK не поддерживают использование службы push-уведомлений Windows (WNS) с приложениями Windows Phone 8.1 Silverlight. Использование WNS (вместо MPNS) с приложениями Windows Phone 8.1 Silverlight описано в статье [Центр уведомлений: учебник для Windows Phone Silverlight] на примере REST API.

В этом руководстве описано следующее.

> [!div class="checklist"]
> * Добавление возможности выбора категорий в мобильное приложение
> * регистрация для получения уведомлений с тегами;
> * Отправка уведомлений с тегами
> * Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

Выполните инструкции из статьи [Начало работы с Центрами уведомлений Azure для приложений Windows Phone](notification-hubs-windows-mobile-push-notifications-mpns.md). При работе с этим руководством вы внесете в мобильное приложение ряд изменений, позволяющих зарегистрироваться в интересующих вас категориях экстренных новостей и получать push-уведомления только для этих категорий.

## <a name="add-category-selection-to-the-mobile-app"></a>Добавление возможности выбора категорий в мобильное приложение

Прежде всего необходимо добавить элементы пользовательского интерфейса для имеющейся главной страницы, позволяющие пользователю выбирать категории для регистрации. Выбранные пользователем категории хранятся на устройстве. При запуске приложения в концентраторе уведомлений создается регистрация устройства с выбранными категориями, представленными в форме тегов.

1. Откройте файл проекта `MainPage.xaml`, а затем замените элементы `Grid` с именами `TitlePanel` и `ContentPanel` следующим кодом:

    ```xml
    <StackPanel x:Name="TitlePanel" Grid.Row="0" Margin="12,17,0,28">
        <TextBlock Text="Breaking News" Style="{StaticResource PhoneTextNormalStyle}" Margin="12,0"/>
        <TextBlock Text="Categories" Margin="9,-7,0,0" Style="{StaticResource PhoneTextTitle1Style}"/>
    </StackPanel>

    <Grid Name="ContentPanel" Grid.Row="1" Margin="12,0,12,0">
        <Grid.RowDefinitions>
            <RowDefinition Height="auto"/>
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
            <RowDefinition Height="auto" />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition />
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <CheckBox Name="WorldCheckBox" Grid.Row="0" Grid.Column="0">World</CheckBox>
        <CheckBox Name="PoliticsCheckBox" Grid.Row="1" Grid.Column="0">Politics</CheckBox>
        <CheckBox Name="BusinessCheckBox" Grid.Row="2" Grid.Column="0">Business</CheckBox>
        <CheckBox Name="TechnologyCheckBox" Grid.Row="0" Grid.Column="1">Technology</CheckBox>
        <CheckBox Name="ScienceCheckBox" Grid.Row="1" Grid.Column="1">Science</CheckBox>
        <CheckBox Name="SportsCheckBox" Grid.Row="2" Grid.Column="1">Sports</CheckBox>
        <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="3" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click" />
    </Grid>
    ```
2. Добавьте в проект класс с именем `Notifications`. Добавьте к определению этого класса модификатор `public`. Затем добавьте в новый файл следующие инструкции `using`.

    ```csharp
    using Microsoft.Phone.Notification;
    using Microsoft.WindowsAzure.Messaging;
    using System.IO.IsolatedStorage;
    using System.Windows;
    ```
3. Скопируйте в новый класс `Notifications` приведенный ниже код:

    ```csharp
    private NotificationHub hub;

    // Registration task to complete registration in the ChannelUriUpdated event handler
    private TaskCompletionSource<Registration> registrationTask;

    public Notifications(string hubName, string listenConnectionString)
    {
        hub = new NotificationHub(hubName, listenConnectionString);
    }

    public IEnumerable<string> RetrieveCategories()
    {
        var categories = (string)IsolatedStorageSettings.ApplicationSettings["categories"];
        return categories != null ? categories.Split(',') : new string[0];
    }

    public async Task<Registration> StoreCategoriesAndSubscribe(IEnumerable<string> categories)
    {
        var categoriesAsString = string.Join(",", categories);
        var settings = IsolatedStorageSettings.ApplicationSettings;
        if (!settings.Contains("categories"))
        {
            settings.Add("categories", categoriesAsString);
        }
        else
        {
            settings["categories"] = categoriesAsString;
        }
        settings.Save();

        return await SubscribeToCategories();
    }

    public async Task<Registration> SubscribeToCategories()
    {
        registrationTask = new TaskCompletionSource<Registration>();

        var channel = HttpNotificationChannel.Find("MyPushChannel");

        if (channel == null)
        {
            channel = new HttpNotificationChannel("MyPushChannel");
            channel.Open();
            channel.BindToShellToast();
            channel.ChannelUriUpdated += channel_ChannelUriUpdated;

            // This is optional, used to receive notifications while the app is running.
            channel.ShellToastNotificationReceived += channel_ShellToastNotificationReceived;
        }

        // If channel.ChannelUri is not null, complete the registrationTask here.  
        // If it is null, the registrationTask will be completed in the ChannelUriUpdated event handler.
        if (channel.ChannelUri != null)
        {
            await RegisterTemplate(channel.ChannelUri);
        }

        return await registrationTask.Task;
    }

    async void channel_ChannelUriUpdated(object sender, NotificationChannelUriEventArgs e)
    {
        await RegisterTemplate(e.ChannelUri);
    }

    async Task<Registration> RegisterTemplate(Uri channelUri)
    {
        // Using a template registration to support notifications across platforms.
        // Any template notifications that contain messageParam and a corresponding tag expression
        // will be delivered for this registration.

        const string templateBodyMPNS = "<wp:Notification xmlns:wp=\"WPNotification\">" +
                                            "<wp:Toast>" +
                                                "<wp:Text1>$(messageParam)</wp:Text1>" +
                                            "</wp:Toast>" +
                                        "</wp:Notification>";

        // The stored categories tags are passed with the template registration.

        registrationTask.SetResult(await hub.RegisterTemplateAsync(channelUri.ToString(),
            templateBodyMPNS, "simpleMPNSTemplateExample", this.RetrieveCategories()));

        return await registrationTask.Task;
    }

    // This is optional. It is used to receive notifications while the app is running.
    void channel_ShellToastNotificationReceived(object sender, NotificationEventArgs e)
    {
        StringBuilder message = new StringBuilder();
        string relativeUri = string.Empty;

        message.AppendFormat("Received Toast {0}:\n", DateTime.Now.ToShortTimeString());

        // Parse out the information that was part of the message.
        foreach (string key in e.Collection.Keys)
        {
            message.AppendFormat("{0}: {1}\n", key, e.Collection[key]);

            if (string.Compare(
                key,
                "wp:Param",
                System.Globalization.CultureInfo.InvariantCulture,
                System.Globalization.CompareOptions.IgnoreCase) == 0)
            {
                relativeUri = e.Collection[key];
            }
        }

        // Display a dialog of all the fields in the toast.
        System.Windows.Deployment.Current.Dispatcher.BeginInvoke(() =>
        {
            MessageBox.Show(message.ToString());
        });
    }
    ```

    Этот класс использует изолированное хранилище для хранения категорий новостей, которые должно получать данное устройство. Он также содержит методы регистрации в этих категориях с помощью [шаблонной](notification-hubs-templates-cross-platform-push-messages.md) регистрации уведомлений.
4. В файле `App.xaml.cs` проекта добавьте следующее свойство в класс `App`: Замените заполнители `<hub name>` и `<connection string with listen access>` именем центра уведомлений и строкой подключения для *DefaultListenSharedAccessSignature*, полученными ранее.

    ```csharp
    public Notifications notifications = new Notifications("<hub name>", "<connection string with listen access>");
    ```

   > [!NOTE]
   > Так как учетные данные, которые распространяются с помощью клиентского приложения, обычно небезопасны, с помощью вашего клиентского приложения следует распространять только ключ для доступа к прослушиванию. Доступ к прослушиванию позволяет приложению регистрироваться для использования уведомлений, однако при этом нельзя изменять имеющиеся регистрации и отправлять уведомления. Для отправки уведомлений и изменения существующих регистраций используется ключ полного доступа в защищенной серверной службе.

5. В файле `MainPage.xaml.cs` добавьте следующую строку:

    ```csharp
    using Windows.UI.Popups;
    ```
6. В файле проекта MainPage.xaml.cs добавьте следующий метод:

    ```csharp
    private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
    {
        var categories = new HashSet<string>();
        if (WorldCheckBox.IsChecked == true) categories.Add("World");
        if (PoliticsCheckBox.IsChecked == true) categories.Add("Politics");
        if (BusinessCheckBox.IsChecked == true) categories.Add("Business");
        if (TechnologyCheckBox.IsChecked == true) categories.Add("Technology");
        if (ScienceCheckBox.IsChecked == true) categories.Add("Science");
        if (SportsCheckBox.IsChecked == true) categories.Add("Sports");

        var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(categories);

        MessageBox.Show("Subscribed to: " + string.Join(",", categories) + " on registration id : " +
            result.RegistrationId);
    }
    ```

    Этот метод создает список категорий и использует класс `Notifications` для хранения списка в локальном хранилище и регистрации соответствующих тегов в концентраторе уведомлений. При изменении категорий регистрация создается заново с новыми категориями.

Ваше приложение теперь может сохранять набор категорий в локальном хранилище на устройстве и регистрироваться в центре уведомлений всякий раз, когда пользователь изменяет выбранные категории.

## <a name="register-for-notifications"></a>Регистрация для использования уведомлений

Эти действия позволяют зарегистрироваться в концентраторе уведомлений при запуске с использованием категорий, сохраненных в локальном хранилище.

> [!NOTE]
> Поскольку URI канала, назначенный службой push-уведомлений Windows (MPNS), может измениться в любое время, следует регулярно производить регистрацию для использования уведомлений, чтобы предотвратить сбои уведомлений. В этом примере регистрация для использования уведомлений осуществляется при каждом запуске приложения. Для тех приложений, которые запускаются часто, более одного раза в день, возможно, лучше пропустить регистрацию, чтобы сэкономить трафик, если с момента прошлой регистрации прошло меньше суток.

1. Откройте файл App.xaml.cs, добавьте модификатор `async` в метод `Application_Launching` и замените код регистрации центров уведомлений, добавленный при работе с руководством по [началу работы с Центрами уведомлений], следующей строкой кода:

    ```csharp
    private async void Application_Launching(object sender, LaunchingEventArgs e)
    {
        var result = await notifications.SubscribeToCategories();

        if (result != null)
            System.Windows.Deployment.Current.Dispatcher.BeginInvoke(() =>
            {
                MessageBox.Show("Registration Id :" + result.RegistrationId, "Registered", MessageBoxButton.OK);
            });
    }
    ```

    Этот код гарантирует, что при каждом запуске приложения категории извлекаются из локального хранилища и для них запрашивается регистрация.
2. В файле проекта MainPage.xaml.cs добавьте следующий код, который выполняет метод `OnNavigatedTo`:

    ```csharp
    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        var categories = ((App)Application.Current).notifications.RetrieveCategories();

        if (categories.Contains("World")) WorldCheckBox.IsChecked = true;
        if (categories.Contains("Politics")) PoliticsCheckBox.IsChecked = true;
        if (categories.Contains("Business")) BusinessCheckBox.IsChecked = true;
        if (categories.Contains("Technology")) TechnologyCheckBox.IsChecked = true;
        if (categories.Contains("Science")) ScienceCheckBox.IsChecked = true;
        if (categories.Contains("Sports")) SportsCheckBox.IsChecked = true;
    }
    ```

    Этот код обновляет главную страницу в зависимости от состояния ранее сохраненных категорий.

Теперь приложение готово и может сохранять набор категорий в локальном хранилище устройств и использовать его для регистрации в концентраторе уведомлений всякий раз, когда пользователь изменяет выбранные категории. Теперь определите серверную часть, которая может отправлять в это приложение уведомления по категориям.

## <a name="send-tagged-notifications"></a>Отправка уведомлений с тегами

[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="test-the-app"></a>Тестирование приложения

1. В Visual Studio нажмите клавишу F5, чтобы скомпилировать и запустить приложение.

    ![Мобильное приложение с категориями][1]

    В пользовательском интерфейсе присутствует набор переключателей, позволяющий выбрать категории для подписки.
2. Включите переключатели одной или нескольких категорий, затем нажмите **Подписаться**.

    Приложение преобразует выбранные категории в теги и запрашивает у концентратора уведомлений новую регистрацию устройств для выбранных тегов. Зарегистрированные категории возвращаются и отображаются в диалоговом окне.

    ![Сообщение об успешной подписке][2]
3. Получив подтверждение об успешной подписки на категории, запустите консольное приложение для отправки уведомлений по каждой категории. Убедитесь, что получены только уведомления для категорий, на которые вы подписаны.

    ![Сообщение уведомления][3]

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как отправлять push-уведомления на определенные устройства, при регистрации которых указаны теги. Сведения о том, как отправлять push-уведомления конкретным пользователям с несколькими устройствами, см. в следующем руководстве: 

> [!div class="nextstepaction"]
>[Отправка push-уведомлений определенным пользователям](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md)

<!-- Anchors. -->
[Add category selection to the app]: #adding-categories
[Register for notifications]: #register
[Send notifications from your back-end]: #send
[Run the app and generate notifications]: #test-app
[Next Steps]: #next-steps

<!-- Images. -->
[1]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-breakingnews.png
[2]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-registration.png
[3]: ./media/notification-hubs-windows-phone-send-breaking-news/notification-hub-toast.png

<!-- URLs.-->
[Приступая к работе с центрами уведомлений]: notification-hubs-windows-mobile-push-notifications-mpns.md
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for Windows Phone]: ??
