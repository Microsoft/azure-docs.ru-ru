---
title: Отправка push-уведомлений на определенные устройства с iOS с помощью концентраторов уведомлений Azure | Документация Майкрософт
description: Из этого руководства вы узнаете, как отправлять push-уведомления на определенные устройства iOS с помощью службы "Центры уведомлений Azure".
services: notification-hubs
documentationcenter: ios
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: article
ms.date: 11/07/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 11/07/2019
ms.openlocfilehash: 2cb979491e247a4d44b9ae9ae27c433fb3f436d1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100579232"
---
# <a name="tutorial-send-push-notifications-to-specific-ios-devices-using-azure-notification-hubs"></a>Руководство. Отправка push-уведомлений на определенные устройства с iOS с помощью центров уведомлений Azure

[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

## <a name="overview"></a>Обзор

В этом руководстве показано, как использовать службу "Центры уведомлений Azure" для рассылки уведомлений об экстренных новостях в приложение iOS. По завершении вы сможете зарегистрироваться в интересующих вас категориях экстренных новостей и получать push-уведомления только для этих категорий. Этот сценарий часто используется во многих приложениях, которые отправляют уведомления определенным группам пользователей, подтвердившим интерес к этим уведомлениям. Типичные примеры: средства чтения RSS, приложения для любителей музыки и т. д.

Широковещательные сценарии реализуются путем включения одного или нескольких *тегов* при создании регистрации в концентраторе уведомлений. Если уведомления отправляются по тегу, то их получают устройства, зарегистрированные для данного тега. Поскольку теги представляют собой обычные строки, их не нужно подготавливать заранее. Дополнительные сведения о тегах см. в статье [Маршрутизация и выражения тегов](notification-hubs-tags-segment-push-message.md).

При работе с этим руководством вы выполните следующие задачи:

> [!div class="checklist"]
> * добавление возможности выбора категорий в приложение;
> * Отправка уведомлений с тегами
> * отправка уведомлений с устройства;
> * Запуск приложения и создание уведомлений

## <a name="prerequisites"></a>Предварительные условия

В этом разделе используется приложение, созданное при выполнении заданий в статье [Руководство по отправке push-уведомлений в приложения iOS с помощью Центров уведомлений Azure][get-started]. Прежде чем приступить к этому руководству, вы должны изучить раздел [Руководство по отправке push-уведомлений в приложения iOS с помощью Центров уведомлений Azure][get-started].

## <a name="add-category-selection-to-the-app"></a>Добавление возможности выбора категорий в приложение

Прежде всего, необходимо добавить элементы пользовательского интерфейса для имеющейся раскадровки, позволяющие пользователю выбирать категории для регистрации. Выбранные пользователем категории хранятся на устройстве. При запуске приложения в концентраторе уведомлений создается регистрация устройства с выбранными категориями, представленными в форме тегов.

1. В вашем **MainStoryboard_iPhone.storyboard** добавьте следующие компоненты из библиотеки объектов:

   * метка с текстом "Экстренные новости";
   * метки с текстами категории "Мир", "Политика", "Бизнес", "Технология", "Наука", "Спорт";
   * Шесть переключателей, по одному в каждой категории. Задайте для каждого переключателя **Состояние** по умолчанию **Off** (Откл.).
   * Одна кнопка с надписью «Подписка».

     Раскадровка должна выглядеть следующим образом:

     ![Конструктор интерфейса Xcode][3]

2. В редакторе помощника создайте выходы для всех переключателей и назовите их WorldSwitch, PoliticsSwitch, BusinessSwitch, TechnologySwitch, ScienceSwitch, SportsSwitch.

3. Создайте действие для кнопки `subscribe`. Ваш `ViewController.h` должен содержать следующий код.

    ```objc
    @property (weak, nonatomic) IBOutlet UISwitch *WorldSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *PoliticsSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *BusinessSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *TechnologySwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *ScienceSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *SportsSwitch;

    - (IBAction)subscribe:(id)sender;
    ```

4. Создайте класс **Cocoa Touch Class** с именем `Notifications`. Скопируйте следующий код в интерфейсную часть файла Notifications.h:

    ```objc
    @property NSData* deviceToken;

    - (id)initWithConnectionString:(NSString*)listenConnectionString HubName:(NSString*)hubName;

    - (void)storeCategoriesAndSubscribeWithCategories:(NSArray*)categories
                completion:(void (^)(NSError* error))completion;

    - (NSSet*)retrieveCategories;

    - (void)subscribeWithCategories:(NSSet*)categories completion:(void (^)(NSError *))completion;
    ```

5. Добавьте следующую директиву импорта в Notifications.m:

    ```objc
    #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
    ```

6. Скопируйте следующий код в раздел файла Notifications.m, в котором запрограммирована реализация.

    ```objc
    SBNotificationHub* hub;

    - (id)initWithConnectionString:(NSString*)listenConnectionString HubName:(NSString*)hubName{

        hub = [[SBNotificationHub alloc] initWithConnectionString:listenConnectionString
                                    notificationHubPath:hubName];

        return self;
    }

    - (void)storeCategoriesAndSubscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
        [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];

        [self subscribeWithCategories:categories completion:completion];
    }

    - (NSSet*)retrieveCategories {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

        NSArray* categories = [defaults stringArrayForKey:@"BreakingNewsCategories"];

        if (!categories) return [[NSSet alloc] init];
        return [[NSSet alloc] initWithArray:categories];
    }

    - (void)subscribeWithCategories:(NSSet *)categories completion:(void (^)(NSError *))completion
    {
        NSString* templateBodyAPNS = @"{\"aps\":{\"alert\":\"$(messageParam)\"}}";

        [hub registerTemplateWithDeviceToken:self.deviceToken name:@"simpleAPNSTemplate" 
            jsonBodyTemplate:templateBodyAPNS expiryTemplate:@"0" tags:categories completion:completion];
    }
    ```

    Этот класс использует локальное хранилище для хранения и извлечения категорий новостей, которые получает данное устройство. Он также содержит метод регистрации этих категорий с помощью [шаблонной](notification-hubs-templates-cross-platform-push-messages.md) регистрации.

7. В файле `AppDelegate.h` добавьте оператор импорта для `Notifications.h` и свойство для экземпляра класса `Notifications`.

    ```objc
    #import "Notifications.h"

    @property (nonatomic) Notifications* notifications;
    ```

8. В методе `didFinishLaunchingWithOptions` в `AppDelegate.m` добавьте в начало метода код для инициализации экземпляра уведомления.  
    В `HUBNAME` и `HUBLISTENACCESS` (определены в `hubinfo.h`) заполнители `<hub name>` и `<connection string with listen access>` необходимо заменить именем центра уведомлений и строкой подключения для *DefaultListenSharedAccessSignature*, полученными раньше.

    ```objc
    self.notifications = [[Notifications alloc] initWithConnectionString:HUBLISTENACCESS HubName:HUBNAME];
    ```

    > [!NOTE]
    > Так как учетные данные, которые распространяются с помощью клиентского приложения, обычно небезопасны, с помощью вашего клиентского приложения следует распространять только ключ для доступа к прослушиванию. Доступ к прослушиванию позволяет приложению регистрироваться для использования уведомлений, однако при этом нельзя изменять имеющиеся регистрации и отправлять уведомления. Для отправки уведомлений и изменения существующих регистраций используется ключ полного доступа в защищенной серверной службе.

9. В методе `didRegisterForRemoteNotificationsWithDeviceToken` в `AppDelegate.m` замените код в методе следующим кодом, чтобы переместить маркер устройства в класс `notifications`. Класс `notifications` выполняет регистрацию для уведомлений с категориями. Если пользователь меняет выбранные категории, для их обновления будет вызван метод `subscribeWithCategories` в ответ на действие кнопки **subscribe**.

    > [!NOTE]
    > Так как маркер устройства, назначенный Cлужба push-уведомлений Apple (APNS), может быть изменен в любое время, следует регулярно регистрироваться для получения уведомлений, чтобы избежать сбоев уведомлений. В этом примере регистрация для использования уведомлений осуществляется при каждом запуске приложения. Для тех приложений, которые запускаются часто, более одного раза в день, возможно, лучше пропустить регистрацию, чтобы сэкономить трафик, если с момента прошлой регистрации прошло меньше суток.

    ```objc
    self.notifications.deviceToken = deviceToken;

    // Retrieves the categories from local storage and requests a registration for these categories
    // each time the app starts and performs a registration.

    NSSet* categories = [self.notifications retrieveCategories];
    [self.notifications subscribeWithCategories:categories completion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error registering for notifications: %@", error);
        }
    }];
    ```

    На этом этапе в методе `didRegisterForRemoteNotificationsWithDeviceToken` не должно быть никакого другого кода.

10. Изучив статью [Руководство по отправке push-уведомлений в приложения iOS с помощью Центров уведомлений Azure][get-started], в файле `AppDelegate.m` должны присутствовать указанные методы. В противном случае добавьте их.

    ```objc
    - (void)MessageBox:(NSString *)title message:(NSString *)messageText
    {

        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title message:messageText delegate:self
            cancelButtonTitle:@"OK" otherButtonTitles: nil];
        [alert show];
    }

    - (void)application:(UIApplication *)application didReceiveRemoteNotification:
       (NSDictionary *)userInfo {
       NSLog(@"%@", userInfo);
       [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
     }
    ```

    Этот метод обрабатывает уведомления, полученные при запуске приложения, отображая простой **UIAlert**.

11. В `ViewController.m` добавьте оператор `import` для `AppDelegate.h` и скопируйте следующий код в созданный с помощью XCode метод `subscribe`. Этот код обновляет регистрацию уведомлений для использования тегов новой категории, которые пользователь выбрал в пользовательском интерфейсе.

    ```objc
    #import "Notifications.h"

    NSMutableArray* categories = [[NSMutableArray alloc] init];

    if (self.WorldSwitch.isOn) [categories addObject:@"World"];
    if (self.PoliticsSwitch.isOn) [categories addObject:@"Politics"];
    if (self.BusinessSwitch.isOn) [categories addObject:@"Business"];
    if (self.TechnologySwitch.isOn) [categories addObject:@"Technology"];
    if (self.ScienceSwitch.isOn) [categories addObject:@"Science"];
    if (self.SportsSwitch.isOn) [categories addObject:@"Sports"];

    Notifications* notifications = [(AppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

    [notifications storeCategoriesAndSubscribeWithCategories:categories completion: ^(NSError* error) {
        if (!error) {
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:"Notification" message:"Subscribed" delegate:self
            cancelButtonTitle:@"OK" otherButtonTitles: nil];
            [alert show];
        } else {
            NSLog(@"Error subscribing: %@", error);
        }
    }];
    ```

    Этот метод создает `NSMutableArray` категорий и использует класс `Notifications` для хранения списка в локальном хранилище и регистрации соответствующих тегов в концентраторе уведомлений. При изменении категорий регистрация создается заново с новыми категориями.

12. В файле `ViewController.m` добавьте следующий код в метод `viewDidLoad`, чтобы задать пользовательский интерфейс на основе ранее сохраненных категорий.

    ```objc
    // This updates the UI on startup based on the status of previously saved categories.

    Notifications* notifications = [(AppDelegate*)[[UIApplication sharedApplication]delegate] notifications];

    NSSet* categories = [notifications retrieveCategories];

    if ([categories containsObject:@"World"]) self.WorldSwitch.on = true;
    if ([categories containsObject:@"Politics"]) self.PoliticsSwitch.on = true;
    if ([categories containsObject:@"Business"]) self.BusinessSwitch.on = true;
    if ([categories containsObject:@"Technology"]) self.TechnologySwitch.on = true;
    if ([categories containsObject:@"Science"]) self.ScienceSwitch.on = true;
    if ([categories containsObject:@"Sports"]) self.SportsSwitch.on = true;
    ```

Теперь приложение может сохранять набор категорий в локальном хранилище устройства и использовать его для регистрации в концентраторе уведомлений всякий раз при запуске приложения. Пользователь может изменить выбранные категории в среде выполнения и щелкнуть метод `subscribe`, чтобы обновить регистрацию устройства. Далее вы можете обновить приложение для отправки уведомлений об экстренных новостях непосредственно в само приложение.

## <a name="optional-send-tagged-notifications"></a>(Необязательно.) Отправка уведомлений с тегами

Если у вас нет доступа к Visual Studio, можно перейти к следующему разделу и отправлять уведомления из самого приложения. Вы также можете отправлять правильные шаблонные уведомления с [портала Azure] с помощью вкладки "Отладка" для центра уведомлений.

[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="optional-send-notifications-from-the-device"></a>(Необязательно.) Отправка уведомлений с устройства

Как правило, уведомления отправляются серверной службой, но вы можете отправлять уведомления об экстренных новостях непосредственно из приложения. Для этого обновите метод `SendNotificationRESTAPI`, определенный в руководстве [по началу работы со службой "Центры уведомлений"][get-started].

1. В файле `ViewController.m` обновите метод `SendNotificationRESTAPI`, как показано ниже, чтобы он принимал параметр для тега категории и отправлял правильное [шаблонное](notification-hubs-templates-cross-platform-push-messages.md) уведомление.

    ```objc
    - (void)SendNotificationRESTAPI:(NSString*)categoryTag
    {
        NSURLSession* session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration
                                    defaultSessionConfiguration] delegate:nil delegateQueue:nil];

        NSString *json;

        // Construct the messages REST endpoint
        NSURL* url = [NSURL URLWithString:[NSString stringWithFormat:@"%@%@/messages/%@", HubEndpoint,
                                            HUBNAME, API_VERSION]];

        // Generated the token to be used in the authorization header.
        NSString* authorizationToken = [self generateSasToken:[url absoluteString]];

        //Create the request to add the template notification message to the hub
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
        [request setHTTPMethod:@"POST"];

        // Add the category as a tag
        [request setValue:categoryTag forHTTPHeaderField:@"ServiceBusNotification-Tags"];

        // Template notification
        json = [NSString stringWithFormat:@"{\"messageParam\":\"Breaking %@ News : %@\"}",
                categoryTag, self.notificationMessage.text];

        // Signify template notification format
        [request setValue:@"template" forHTTPHeaderField:@"ServiceBusNotification-Format"];

        // JSON Content-Type
        [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];

        //Authenticate the notification message POST request with the SaS token
        [request setValue:authorizationToken forHTTPHeaderField:@"Authorization"];

        //Add the notification message body
        [request setHTTPBody:[json dataUsingEncoding:NSUTF8StringEncoding]];

        // Send the REST request
        NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
                    completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
            {
            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
                if (error || httpResponse.statusCode != 200)
                {
                    NSLog(@"\nError status: %d\nError: %@", httpResponse.statusCode, error);
                }
                if (data != NULL)
                {
                    //xmlParser = [[NSXMLParser alloc] initWithData:data];
                    //[xmlParser setDelegate:self];
                    //[xmlParser parse];
                }
            }];

        [dataTask resume];
    }
    ```

2. В файле `ViewController.m` обновите действие `Send Notification`, как показано в указанном ниже коде. Таким образом, он будет отправлять уведомления на несколько платформ, используя отдельно каждый тег.

    ```objc
    - (IBAction)SendNotificationMessage:(id)sender
    {
        self.sendResults.text = @"";

        NSArray* categories = [NSArray arrayWithObjects: @"World", @"Politics", @"Business",
                                @"Technology", @"Science", @"Sports", nil];

        // Lets send the message as breaking news for each category to WNS, FCM, and APNS
        // using a template.
        for(NSString* category in categories)
        {
            [self SendNotificationRESTAPI:category];
        }
    }
    ```

3. Повторно создайте проект и убедитесь, что у вас не возникли ошибки сборки.

## <a name="run-the-app-and-generate-notifications"></a>Запуск приложения и создание уведомлений

1. Нажмите кнопку Запуск для построения проекта, после чего запустите приложение. Чтобы подписаться на некоторые экстренные новости, нажмите кнопку **Подписаться** . В появившемся диалоговом окне отобразятся те уведомления, на которые была настроена подписка.

    ![Пример уведомления в iOS][1]

    Если выбрано **Подписка**, приложение преобразует выбранные категории в теги и запрашивает у концентратора уведомлений новую регистрацию устройств для выбранных тегов.

2. Введите сообщение, отправляемое в качестве экстренных новостей, и нажмите кнопку **Отправить уведомление** . Можно также запустить консольное приложение .NET для создания уведомлений.

    ![Настройка параметров уведомлений в iOS][2]

3. Каждое устройство с подпиской на экстренные новости получает отправленные вами уведомления об экстренных новостях.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы отправили широковещательные уведомления на конкретные устройства iOS, зарегистрированные для получения уведомлений по категориям. Чтобы узнать, как отправлять локализованные push-уведомления, перейдите к следующему руководству.

> [!div class="nextstepaction"]
>[Отправка локализованных push-уведомлений](notification-hubs-ios-xplat-localized-apns-push-notification.md)

<!-- Images. -->
[1]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-subscribed.png
[2]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios1.png
[3]: ./media/notification-hubs-ios-send-breaking-news/notification-hub-breakingnews-ios2.png

<!-- URLs. -->
[How To: Service Bus Notification Hubs (iOS Apps)]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-ios-xplat-localized-apns-push-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-ios-notify-users.md
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/dn530749(v=azure.100)
[Notification Hubs How-To for iOS]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[get-started]: ios-sdk-get-started.md
[Портал Azure]: https://portal.azure.com
