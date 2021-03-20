---
title: Отправка локализованных push-уведомлений в iOS с помощью концентраторов уведомлений Azure | Документация Майкрософт
description: Узнайте, как отправлять локализованные уведомления на устройства iOS с помощью службы "Центры уведомлений Azure".
services: notification-hubs
documentationcenter: ios
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 484914b5-e081-4a05-a84a-798bbd89d428
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: a78d3a76e2b13a120e9e744e181c95bfcb330f27
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92313907"
---
# <a name="tutorial-send-localized-push-notifications-to-ios-using-azure-notification-hubs"></a>Руководство. Отправка локализованных push-уведомлений в iOS с помощью центров уведомлений Azure

> [!div class="op_single_selector"]
> * [C# в Магазине Windows](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
> * [iOS](notification-hubs-ios-xplat-localized-apns-push-notification.md)

В этом руководстве показано, как использовать функцию [шаблонов](notification-hubs-templates-cross-platform-push-messages.md) службы "Центры уведомлений Azure" для рассылки уведомлений об экстренных новостях, локализованных для языка и устройства. В этом руководстве вы начинаете с приложения iOS, созданного в [статье Использование концентраторов уведомлений для отправки экстренных новостей]. По завершении вы можете зарегистрироваться для получения интересующих категорий, указать язык, на котором будут отправляться уведомления, и получать push-уведомления только для выбранных категорий на этом языке.

Этот сценарий состоит из двух частей:

* приложение iOS позволяет клиентским устройствам указывать язык и подписываться на различные категории экстренных новостей;
* сервер рассылает уведомления, используя функции **тегов** и **шаблонов** службы "Центры уведомлений Azure".

При работе с этим руководством вы выполните следующие задачи:

> [!div class="checklist"]
> * обновление пользовательского интерфейса приложения;
> * Выполнение сборки приложения iOS
> * отправка локализованных шаблонных уведомлений из консольного приложения .NET;
> * отправка локализованных шаблонных уведомлений с устройства.

## <a name="overview"></a>Обзор

В [статье Использование концентраторов уведомлений для отправки экстренных новостей]вы создали приложение, которое использовало **теги** для подписки на уведомления для различных категорий новостей. Однако многие приложения ориентированы на несколько рынков и требуют локализации. Это означает, что само содержимое уведомлений должно быть локализовано и доставлено в правильный набор устройств. В этом руководстве демонстрируются способы использования шаблонов службы "Центры уведомлений", которые позволяют легко доставлять локализованные уведомления об экстренных новостях.

> [!NOTE]
> Один из способов отправки локализованных уведомлений — создание нескольких версий каждого тега. Например, для поддержки английского, французского и китайского языков понадобится три разных тега для мировых новостей: world_en, world_fr и world_ch. Затем необходимо отправить локализованную версию мировых новостей по каждому из этих тегов. В этом разделе мы используем шаблоны во избежание избыточного количества тегов и необходимости отправки нескольких сообщений.

Шаблоны позволяют указать, каким образом конкретное устройство должно получить уведомление. Шаблон указывает точный формат полезных данных путем обращения к свойствам, которые являются частью сообщения, отправленного сервером вашего приложение. В нашем случае отправляется независимое от языка сообщение, которое содержит все поддерживаемые языки.

```json
{
    "News_English": "...",
    "News_French": "...",
    "News_Mandarin": "..."
}
```

Затем следует убедиться, что устройство зарегистрировано с использованием шаблона, который ссылается на нужное свойство. Например, приложение iOS, которое требуется зарегистрировать для французских новостей, использует следующую синтаксическую конструкцию для регистрации.

```json
{
    aps: {
        alert: "$(News_French)"
    }
}
```

Дополнительные сведения о шаблонах см. в статье [Шаблоны](notification-hubs-templates-cross-platform-push-messages.md).

## <a name="prerequisites"></a>Предварительные условия

* Выполните задания руководства [Отправка push-уведомлений на определенные устройства iOS с помощью инфраструктуры "Центры уведомлений Azure"](notification-hubs-ios-xplat-segmented-apns-push-notification.md) и подготовьте код, так как он используется в текущем руководстве.
* Visual Studio 2019 является необязательным.

## <a name="update-the-app-user-interface"></a>обновление пользовательского интерфейса приложения;

В этом разделе вы измените приложение Breaking News, созданное в разделе [Использование концентраторов уведомлений для передачи экстренных новостей] для отправки локализованных экстренных новостей с помощью шаблонов.

В `MainStoryboard_iPhone.storyboard` добавьте сегментированный элемент управления на трех языках: Английский, французский и мандаринский диалект.

![Создание раскадровки пользовательского интерфейса для iOS][13]

Затем обязательно добавьте IBOutlet в свой файл ViewController.h, как показано на рисунке ниже.

![Создание выходов для переключателей][14]

## <a name="build-the-ios-app"></a>Выполнение сборки приложения iOS

1. В `Notification.h` добавьте `retrieveLocale` метод и измените методы Store и Subscribe, как показано в следующем коде:

    ```objc
    - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet*) categories completion: (void (^)(NSError* error))completion;

    - (void) subscribeWithLocale:(int) locale categories:(NSSet*) categories completion:(void (^)(NSError *))completion;

    - (NSSet*) retrieveCategories;

    - (int) retrieveLocale;
    ```
    В `Notification.m` измените метод `storeCategoriesAndSubscribe` путем добавления параметра `locale` и сохранения его в качестве пользователя по умолчанию.

    ```objc
    - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
        [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];
        [defaults setInteger:locale forKey:@"BreakingNewsLocale"];
        [defaults synchronize];

        [self subscribeWithLocale: locale categories:categories completion:completion];
    }
    ```

    Затем измените метод *subscribe* , чтобы включить языковой стандарт:

    ```objc
    - (void) subscribeWithLocale: (int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion{
        SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:@"<connection string>" notificationHubPath:@"<hub name>"];

        NSString* localeString;
        switch (locale) {
            case 0:
                localeString = @"English";
                break;
            case 1:
                localeString = @"French";
                break;
            case 2:
                localeString = @"Mandarin";
                break;
        }

        NSString* template = [NSString stringWithFormat:@"{\"aps\":{\"alert\":\"$(News_%@)\"},\"inAppMessage\":\"$(News_%@)\"}", localeString, localeString];

        [hub registerTemplateWithDeviceToken:self.deviceToken name:@"localizednewsTemplate" jsonBodyTemplate:template expiryTemplate:@"0" tags:categories completion:completion];
    }
    ```

    Используйте метод `registerTemplateWithDeviceToken` вместо `registerNativeWithDeviceToken`. При регистрации шаблона вам необходимо предоставить шаблон json, а также имя шаблона (так как для приложения может возникнуть необходимость зарегистрировать различные шаблоны). Убедитесь, что ваши категории регистрируются как теги, поскольку вы хотите получать уведомления об этих новостях.

    Добавьте метод для извлечения языкового стандарта из пользовательских параметров по умолчанию.

    ```objc
    - (int) retrieveLocale {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

        int locale = [defaults integerForKey:@"BreakingNewsLocale"];

        return locale < 0?0:locale;
    }
    ```

2. После изменения класса `Notifications` необходимо убедиться, что `ViewController` использует новый `UISegmentControl`. Добавьте следующую строку в метод `viewDidLoad`, чтобы убедиться в том, что отображается языковой стандарт, выбранный в данный момент.

    ```objc
    self.Locale.selectedSegmentIndex = [notifications retrieveLocale];
    ```

    Затем в своем методе `subscribe` измените вызов `storeCategoriesAndSubscribe` на следующий код.

    ```objc
    [notifications storeCategoriesAndSubscribeWithLocale: self.Locale.selectedSegmentIndex categories:[NSSet setWithArray:categories] completion: ^(NSError* error) {
        if (!error) {
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
                                    @"Subscribed!" delegate:nil cancelButtonTitle:
                                    @"OK" otherButtonTitles:nil, nil];
            [alert show];
        } else {
            NSLog(@"Error subscribing: %@", error);
        }
    }];
    ```

3. И наконец, необходимо обновить метод `didRegisterForRemoteNotificationsWithDeviceToken` в методе AppDelegate.m, чтобы правильно обновить регистрацию при запуске приложения. Замените вызов метода уведомлений `subscribe` следующим кодом.

    ```obj-c
    NSSet* categories = [self.notifications retrieveCategories];
    int locale = [self.notifications retrieveLocale];
    [self.notifications subscribeWithLocale: locale categories:categories completion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error registering for notifications: %@", error);
        }
    }];
    ```

## <a name="optional-send-localized-template-notifications-from-net-console-app"></a>(Необязательно.) Отправка локализованных шаблонных уведомлений из консольного приложения .NET

[!INCLUDE [notification-hubs-localized-back-end](../../includes/notification-hubs-localized-back-end.md)]

## <a name="optional-send-localized-template-notifications-from-the-device"></a>(Необязательно.) Отправка локализованных шаблонных уведомлений с устройства

Предположим, у вас нет доступа к Visual Studio или вы просто хотите проверить отправку локализованных шаблонных уведомлений непосредственно из приложения на устройстве. Вы можете добавить параметры локализованных шаблонов в метод `SendNotificationRESTAPI`, определенный в предыдущем руководстве.

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
    json = [NSString stringWithFormat:@"{\"messageParam\":\"Breaking %@ News : %@\","
            \"News_English\":\"Breaking %@ News in English : %@\","
            \"News_French\":\"Breaking %@ News in French : %@\","
            \"News_Mandarin\":\"Breaking %@ News in Mandarin : %@\","
            categoryTag, self.notificationMessage.text,
            categoryTag, self.notificationMessage.text,  // insert English localized news here
            categoryTag, self.notificationMessage.text,  // insert French localized news here
            categoryTag, self.notificationMessage.text]; // insert Mandarin localized news here

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

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы отправили локализованные уведомления на устройства iOS. Чтобы узнать, как отправлять push-уведомления конкретным пользователям приложений iOS, перейдите к следующему руководству.

> [!div class="nextstepaction"]
>[Отправка push-уведомлений определенным пользователям](notification-hubs-aspnet-backend-ios-apple-apns-notification.md)

<!-- Images. -->
[13]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized1.png
[14]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized2.png

<!-- URLs. -->
[How To: Service Bus Notification Hubs (iOS Apps)]: /previous-versions/azure/reference/dn223264(v=azure.100)
[Использование концентраторов уведомлений для передачи экстренных новостей]: notification-hubs-ios-xplat-segmented-apns-push-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs: ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Notify users with Notification Hubs: Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-ios
[Get started with authentication]: /develop/mobile/tutorials/get-started-with-users-ios
[Get started with push notifications]: /develop/mobile/tutorials/get-started-with-push-ios
[Push notifications to app users]: /develop/mobile/tutorials/push-notifications-to-users-ios
[Authorize users with scripts]: /develop/mobile/tutorials/authorize-users-in-scripts-ios
[JavaScript and HTML]: ../get-started-with-push-js.md
[Windows Developer Preview registration steps for Mobile Services]: ../mobile-services-windows-developer-preview-registration.md
[wns object]: /previous-versions/azure/reference/jj860484(v=azure.100)
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for iOS]: /previous-versions/azure/reference/dn223264(v=azure.100)