---
title: Отправка push-уведомлений на конкретные устройства (универсальная платформа Windows) | Документация Майкрософт
description: Использование Центров уведомлений Azure с тегами в регистрации для отправки экстренных новостей в приложение универсальной платформы Windows.
services: notification-hubs
documentationcenter: windows
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 994d2eed-f62e-433c-bf65-4afebf1c0561
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 09/30/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 03/22/2019
ms.openlocfilehash: 2c77eba69fd914e8ecc7d08a1b16f61ceefe101b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92320573"
---
# <a name="tutorial-send-notifications-to-specific-devices-running-universal-windows-platform-applications"></a>Руководство. Отправка уведомлений на конкретные устройства с запущенными приложениями универсальной платформы Windows

[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

## <a name="overview"></a>Обзор

В этом учебнике показано, как использовать службу "Центры уведомлений Azure" для рассылки уведомлений об экстренных новостях. Здесь рассматриваются приложения для Магазина Windows или Windows Phone 8.1 (не Silverlight). Если вы используете Windows Phone 8.1 Silverlight, см. статью [Push notifications to specific Windows Phone devices by using Azure Notification Hubs](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md) (Руководство по отправке push-уведомлений на конкретные устройства Windows Phone с помощью Центров уведомлений Azure).

Из этого учебника вы узнаете, как с помощью Центров уведомлений Azure отправлять push-уведомления на конкретные устройства Windows с запущенными приложениями универсальной платформы Windows. После завершения работы с этим учебником вы сможете зарегистрироваться для получения экстренных новостей тех категорий, которые вас интересуют. Вы будете получать push-уведомления только для этих категорий.

Широковещательные сценарии можно включить, добавив один или несколько *тегов* при создании регистрации в центре уведомлений. Если уведомления отправляются на тег, их получают все устройства, зарегистрированные для данного тега. Дополнительные сведения см. в статье [Routing and tag expressions](notification-hubs-tags-segment-push-message.md) (Маршрутизация и выражения тегов).

> [!NOTE]
> Проекты для Магазина Windows и проекты Windows Phone 8.1 и более ранней версии не поддерживаются в Visual Studio 2019. Дополнительные сведения см. в статье [Целевая платформа и совместимость для Visual Studio 2019](/visualstudio/releases/2019/compatibility).

В этом учебнике описано, как выполнять такие задачи:

> [!div class="checklist"]
> * Добавление возможности выбора категорий в мобильное приложение
> * Регистрация для использования уведомлений
> * Отправка уведомлений с тегами
> * Запуск приложения и создание уведомлений

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы с этим руководством выполните инструкции из статьи [Начало работы с Центрами уведомлений для приложений универсальной платформы Windows][get-started].  

## <a name="add-category-selection-to-the-app"></a>Добавление возможности выбора категорий в приложение

Прежде всего необходимо добавить элементы пользовательского интерфейса для имеющейся главной страницы, чтобы пользователь имел возможность выбирать категории для регистрации. Выбранные категории хранятся на устройстве. При запуске приложения в центре уведомлений создается регистрация устройства с выбранными категориями, представленными в форме тегов.

1. Откройте файл проекта *MainPage.xaml* и скопируйте следующий код в элемент `Grid`:

    ```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="1" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="2" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="3" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="1" Grid.Column="1" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="2" Grid.Column="1" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="3" Grid.Column="1" HorizontalAlignment="Center"/>
        <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click"/>
    </Grid>
    ```

1. Щелкните правой кнопкой мыши проект в **обозревателе решений** и выберите пункт **Добавить** > **Класс**. На вкладке **Добавление нового элемента** присвойте классу имя *Уведомления* и нажмите кнопку **Добавить**. При необходимости добавьте к определению этого класса модификатор `public`.

1. Затем добавьте в новый файл следующие инструкции `using`.

    ```csharp
    using Windows.Networking.PushNotifications;
    using Microsoft.WindowsAzure.Messaging;
    using Windows.Storage;
    using System.Threading.Tasks;
    ```

1. Скопируйте в новый класс `Notifications` приведенный ниже код:

    ```csharp
    private NotificationHub hub;

    public Notifications(string hubName, string listenConnectionString)
    {
        hub = new NotificationHub(hubName, listenConnectionString);
    }

    public async Task<Registration> StoreCategoriesAndSubscribe(IEnumerable<string> categories)
    {
        ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
        return await SubscribeToCategories(categories);
    }

    public IEnumerable<string> RetrieveCategories()
    {
        var categories = (string) ApplicationData.Current.LocalSettings.Values["categories"];
        return categories != null ? categories.Split(','): new string[0];
    }

    public async Task<Registration> SubscribeToCategories(IEnumerable<string> categories = null)
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        if (categories == null)
        {
            categories = RetrieveCategories();
        }

        // Using a template registration to support notifications across platforms.
        // Any template notifications that contain messageParam and a corresponding tag expression
        // will be delivered for this registration.

        const string templateBodyWNS = "<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(messageParam)</text></binding></visual></toast>";

        return await hub.RegisterTemplateAsync(channel.Uri, templateBodyWNS, "simpleWNSTemplateExample",
                categories);
    }
    ```

    Этот класс использует локальное хранилище для хранения категорий новостей, которые данное устройство должно получать. Вместо вызова метода `RegisterNativeAsync` вызовите `RegisterTemplateAsync` для регистрации в категориях с помощью шаблонной регистрации.

    Если вы намерены зарегистрировать несколько шаблонов, введите имя шаблона (например, *simpleWNSTemplateExample*). Назовите шаблоны таким образом, чтобы можно было обновить или удалить их. Вы можете зарегистрировать несколько шаблонов: один для получения всплывающих уведомлений и один для плиток.

    >[!NOTE]
    > Центры уведомлений позволяют устройству зарегистрировать несколько шаблонов с помощью одного и того же тега. В этом случае входящее сообщение, предназначенное для этого тега, приводит к отправке на устройство нескольких уведомлений, по одному для каждого шаблона. Этот процесс дает возможность отобразить одно сообщение в нескольких визуальных уведомлениях, например в виде значка и в виде всплывающего уведомления в приложении Магазина Windows.

    Дополнительные сведения см. в статье [Шаблоны](notification-hubs-templates-cross-platform-push-messages.md).

1. В файле проекта *App.xaml.cs* добавьте следующее свойство в класс `App`:

    ```csharp
    public Notifications notifications = new Notifications("<hub name>", "<connection string with listen access>");
    ```

    Это свойство используется для создания экземпляра `Notifications` и доступа к нему.

    В коде замените заполнители `<hub name>` и `<connection string with listen access>` именем центра уведомлений и строкой подключения для **DefaultListenSharedAccessSignature**, полученными ранее.

   > [!NOTE]
   > Так как учетные данные, которые распределены с помощью клиентского приложения, обычно небезопасны, следует распространять только ключ для доступа к *прослушиванию*. Доступ к прослушиванию позволяет приложению регистрироваться для использования уведомлений, однако при этом нельзя изменять имеющиеся регистрации и отправлять уведомления. Для отправки уведомлений и изменения существующих регистраций используется ключ полного доступа в защищенной серверной службе.

1. В файле *MainPage.xaml.cs* добавьте следующую строку:

    ```csharp
    using Windows.UI.Popups;
    ```

1. В файле *MainPage.xaml.cs* добавьте следующий метод:

    ```csharp
    private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
    {
        var categories = new HashSet<string>();
        if (WorldToggle.IsOn) categories.Add("World");
        if (PoliticsToggle.IsOn) categories.Add("Politics");
        if (BusinessToggle.IsOn) categories.Add("Business");
        if (TechnologyToggle.IsOn) categories.Add("Technology");
        if (ScienceToggle.IsOn) categories.Add("Science");
        if (SportsToggle.IsOn) categories.Add("Sports");

        var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(categories);

        var dialog = new MessageDialog("Subscribed to: " + string.Join(",", categories) + " on registration Id: " + result.RegistrationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    ```

    Этот метод создает список категорий и использует класс `Notifications` для хранения списка в локальном хранилище. Он также регистрирует соответствующие теги в центре уведомлений. При изменении категорий регистрация создается заново с новыми категориями.

Теперь приложение может сохранять набор категорий в локальном хранилище устройства. Приложение регистрируется в центре уведомлений всякий раз, когда пользователи выбирают другую категорию.

## <a name="register-for-notifications"></a>Регистрация для использования уведомлений

В этом разделе приведены действия, позволяющие зарегистрироваться в центре уведомлений при запуске с использованием категорий, сохраненных в локальном хранилище.

> [!NOTE]
> Так как универсальный код ресурса (URI) канала, назначенный службой уведомлений Windows (WNS), может измениться в любое время, следует регулярно производить регистрацию для использования уведомлений, чтобы предотвратить сбои уведомлений. В этом примере регистрация для использования уведомлений осуществляется при каждом запуске приложения. Для тех приложений, которые запускаются часто (более одного раза в день), возможно, лучше пропустить регистрацию, чтобы не изменять пропускную способность, если с момента прошлой регистрации прошло меньше суток.

1. Откройте файл `notifications`App.xaml.cs *и обновите метод*, чтобы использовать класс `InitNotificationsAsync` для подписки по категориям.

    ```csharp
    // *** Remove or comment out these lines ***
    //var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();
    //var hub = new NotificationHub("your hub name", "your listen connection string");
    //var result = await hub.RegisterNativeAsync(channel.Uri);

    var result = await notifications.SubscribeToCategories();
    ```

    Этот процесс гарантирует, что при каждом запуске приложения оно извлекает категории из локального хранилища. Затем он запрашивает регистрацию этих категорий. Вы создали метод `InitNotificationsAsync` при работе с учебником об [отправке уведомлений в приложения универсальной платформы Windows с использованием Центров уведомлений Azure][get-started].
2. В файле проекта *MainPage.xaml.cs* добавьте следующий код в метод `OnNavigatedTo`:

    ```csharp
    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        var categories = ((App)Application.Current).notifications.RetrieveCategories();

        if (categories.Contains("World")) WorldToggle.IsOn = true;
        if (categories.Contains("Politics")) PoliticsToggle.IsOn = true;
        if (categories.Contains("Business")) BusinessToggle.IsOn = true;
        if (categories.Contains("Technology")) TechnologyToggle.IsOn = true;
        if (categories.Contains("Science")) ScienceToggle.IsOn = true;
        if (categories.Contains("Sports")) SportsToggle.IsOn = true;
    }
    ```

    В соответствии с этим кодом главная страница обновляется в зависимости от состояния ранее сохраненных категорий.

Приложение готово. Теперь приложение может сохранять набор категорий в локальном хранилище устройства. Когда пользователи изменяют выбор категорий, сохраненные категории используются для регистрации в центре уведомлений. В следующем разделе мы определим серверную часть, которая может отправлять уведомления категорий в это приложение.

## <a name="run-the-uwp-app"></a>Запуск приложения UWP

1. В Visual Studio нажмите клавишу F5, чтобы скомпилировать и запустить приложение. В пользовательском интерфейсе присутствует набор переключателей, позволяющий выбрать категории для подписки.

   ![Приложение экстренных новостей](./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-breaking-news.png)

1. Включите переключатели одной или нескольких категорий, а затем выберите **Подписаться**.

   Приложение преобразует выбранные категории в теги и запрашивает у концентратора уведомлений новую регистрацию устройств для выбранных тегов. Приложение отображает зарегистрированные категории в диалоговом окне.

    ![Переключатели категорий и кнопка подписки](./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-toast.png)

## <a name="create-a-console-app-to-send-tagged-notifications"></a>Создание консольного приложения для отправки уведомлений с тегами

[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="run-the-console-app-to-send-tagged-notifications"></a>Запуск консольного приложения для отправки уведомлений с тегами

Запустите приложение, созданное в предыдущем разделе. Уведомления для выбранных категорий отображаются в виде всплывающих уведомлений.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы рассмотрели, как производить рассылку экстренных новостей по категориям. Приложение серверной части отправляет уведомления с тегами на устройства, зарегистрированные для получения уведомлений с этим тегом. Сведения о том, как отправлять push-уведомления конкретным пользователям, независимо от того, какое устройство они используют, см. в следующем учебнике:

> [!div class="nextstepaction"]
> [Руководство по отправке локализованных уведомлений в приложения Windows с помощью Центров уведомлений Azure](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)

<!-- Anchors. -->
[Add category selection to the app]: #adding-categories
[Register for notifications]: #register
[Send notifications from your back-end]: #send
[Run the app and generate notifications]: #test-app
[Next Steps]: #next-steps

<!-- URLs.-->
[get-started]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started/
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for Windows Store]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[wns object]: /previous-versions/azure/reference/jj860484(v=azure.100)