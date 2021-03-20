---
title: Форматированные push-уведомления на основе концентраторов уведомлений Azure
description: Узнайте, как отправлять форматированные push-уведомления в приложение iOS из Azure. Примеры кода написаны на Objective-C и C#.
documentationcenter: ios
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 08/17/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 33626b7aee615d07ef88dd9fbca46e6512e2cafc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90090369"
---
# <a name="azure-notification-hubs-rich-push"></a>Форматированные push-уведомления на основе концентраторов уведомлений Azure

## <a name="overview"></a>Обзор

Чтобы привлечь пользователей с помощью мгновенного форматированного содержимого, приложению может потребоваться не просто обычный текст. Эти уведомления развивают взаимодействие с пользователем и предоставляют такие материалы, как URL-адреса, звуки, изображения и купоны, и многое другое. Это руководство построено в руководстве по [уведомлению пользователей](notification-hubs-aspnet-backend-ios-apple-apns-notification.md) и демонстрирует, как отправлять push-уведомления, включающие полезные данные (например, изображения).

Этот учебник совместим с iOS 7 и 8.

  ![Три снимка экрана: экран приложения с кнопкой отправки push-уведомлений, начальным экраном на устройстве и эмблемой Windows с кнопкой "назад".][IOS1]

На высоком уровне:

1. Серверная часть приложения:
   * Хранит обширные полезные данные (в данном случае это изображение) в серверной базе данных или локальном хранилище.
   * Отправляет идентификатор этого расширенного уведомления на устройство.
2. Приложение на устройстве:
   * Обращается к серверной части, запрашивающей обширные полезные данные, с полученным ИДЕНТИФИКАТОРом.
   * Отправляет пользователям уведомления на устройстве при завершении извлечения данных и отображает полезные данные немедленно, когда пользователь нажмет кнопку для получения дополнительных сведений.

## <a name="webapi-project"></a>Проект WebAPI

1. В Visual Studio откройте проект **AppBackend** , созданный в учебнике [Уведомление пользователей](notification-hubs-aspnet-backend-ios-apple-apns-notification.md) .
2. Получите изображение, которое нужно показать пользователям, и разместите его в папке **img** в каталоге проекта.
3. Щелкните **Показать все файлы** в обозревателе решений и щелкните правой кнопкой мыши папку, чтобы **включить ее в проект**.
4. Выбрав изображение, измените его **действие сборки** в окне **Свойства** на **внедренный ресурс**.

    ![Снимок экрана обозреватель решений. Выбран файл изображения, и в его области свойств в качестве действия сборки отображается внедренный ресурс.][IOS2]
5. В `Notifications.cs` добавьте следующую `using` инструкцию:

    ```csharp
    using System.Reflection;
    ```

6. Замените класс `Notifications` на следующий код. Не забудьте заменить заполнители учетными данными и именем файла образа центра уведомлений:

    ```csharp
    public class Notification {
        public int Id { get; set; }
        // Initial notification message to display to users
        public string Message { get; set; }
        // Type of rich payload (developer-defined)
        public string RichType { get; set; }
        public string Payload { get; set; }
        public bool Read { get; set; }
    }

    public class Notifications {
        public static Notifications Instance = new Notifications();

        private List<Notification> notifications = new List<Notification>();

        public NotificationHubClient Hub { get; set; }

        private Notifications() {
            // Placeholders: replace with the connection string (with full access) for your notification hub and the hub name from the Azure Classics Portal
            Hub = NotificationHubClient.CreateClientFromConnectionString("{conn string with full access}",  "{hub name}");
        }

        public Notification CreateNotification(string message, string richType, string payload) {
            var notification = new Notification() {
                Id = notifications.Count,
                Message = message,
                RichType = richType,
                Payload = payload,
                Read = false
            };

            notifications.Add(notification);

            return notification;
        }

        public Stream ReadImage(int id) {
            var assembly = Assembly.GetExecutingAssembly();
            // Placeholder: image file name (for example, logo.png).
            return assembly.GetManifestResourceStream("AppBackend.img.{logo.png}");
        }
    }
    ```

7. В `NotificationsController.cs` Переопределите `NotificationsController` с помощью следующего кода. При этом на устройство будет отправлен первоначальный идентификатор уведомления с богатыми возможностями без уведомления и разрешено получение образа на стороне клиента:

    ```csharp
    // Return http response with image binary
    public HttpResponseMessage Get(int id) {
        var stream = Notifications.Instance.ReadImage(id);

        var result = new HttpResponseMessage(HttpStatusCode.OK);
        result.Content = new StreamContent(stream);
        // Switch in your image extension for "png"
        result.Content.Headers.ContentType = new System.Net.Http.Headers.MediaTypeHeaderValue("image/{png}");

        return result;
    }

    // Create rich notification and send initial silent notification (containing id) to client
    public async Task<HttpResponseMessage> Post() {
        // Replace the placeholder with image file name
        var richNotificationInTheBackend = Notifications.Instance.CreateNotification("Check this image out!", "img",  "{logo.png}");

        var usernameTag = "username:" + HttpContext.Current.User.Identity.Name;

        // Silent notification with content available
        var aboutUser = "{\"aps\": {\"content-available\": 1, \"sound\":\"\"}, \"richId\": \"" + richNotificationInTheBackend.Id.ToString() + "\",  \"richMessage\": \"" + richNotificationInTheBackend.Message + "\", \"richType\": \"" + richNotificationInTheBackend.RichType + "\"}";

        // Send notification to apns
        await Notifications.Instance.Hub.SendAppleNativeNotificationAsync(aboutUser, usernameTag);

        return Request.CreateResponse(HttpStatusCode.OK);
    }
    ```

8. Теперь повторно разверните это приложение на веб-сайте Azure, чтобы сделать его доступным со всех устройств. Щелкните правой кнопкой мыши проект **AppBackend** и нажмите кнопку **Опубликовать**.
9. Выберите **веб-сайт Azure** в качестве цели публикации. Войдите с помощью учетной записи Azure и выберите существующий или новый веб-сайт и запишите значение свойства **URL-адрес назначения** на вкладке **Подключение** . Далее в этом руководстве мы будем называть этот URL-адрес *конечной точкой серверной части* . Нажмите кнопку **Опубликовать**.

## <a name="modify-the-ios-project"></a>Внесение изменений в проект iOS

Теперь, когда Серверная часть приложения изменена для отправки только *идентификатора* уведомления, измените приложение iOS таким образом, чтобы оно ОБРАБАТЫВАЛ этот идентификатор, и получите Расширенное сообщение из серверной части:

1. Откройте проект iOS и включите удаленные уведомления. Для этого перейдите в целевое основное приложение в разделе **Targets** (Целевые объекты).
2. Выберите **возможности**, включить **фоновые режимы** и установите флажок **Удаленные уведомления** .

    ![Снимок экрана проекта iOS, отображающего экран возможностей. Фоновые режимы включены, флажок Удаленные уведомления установлен.][IOS3]
3. Откройте `Main.storyboard` и убедитесь, что у вас есть контроллер представления (в этом учебнике это называется контроллером домашнего представления) из учебника [уведомление пользователя](notification-hubs-aspnet-backend-ios-apple-apns-notification.md) .
4. Добавьте **контроллер навигации** в раскадровку и перетащите контроллер представления Home, чтобы сделать его **корневым представлением** навигации. Убедитесь, что параметр **является начальным контроллером представления** в инспекторе атрибутов выбран только для контроллера навигации.
5. Добавьте **контроллер представления** в раскадровку и добавьте **представление изображений**. Эту страницу пользователи увидят, когда, чтобы узнать больше, щелкнут на уведомление. Раскадровка должна выглядеть следующим образом:

    ![Снимок экрана раскадровки. Отображаются три экрана приложения: представление навигации, представление домашней страницы и представление изображений.][IOS4]
6. Щелкните **контроллер домашнего представления** в раскадровке и убедитесь, что он имеет **хомевиевконтроллер** в качестве своего **пользовательского класса** и **идентификатора раскадровки** в инспекторе удостоверений.
7. Сделайте то же самое для контроллера представления изображений, как **imageViewController**.
8. Затем создайте новый класс контроллера представления с именем **imageViewController** , чтобы обрабатывал только что созданный пользовательский интерфейс.
9. В **imageViewController. h** добавьте следующий код в объявления интерфейса контроллера. Удерживая клавишу Control, перетащите представление изображения раскадровки в эти свойства, чтобы связать их:

    ```objc
    @property (weak, nonatomic) IBOutlet UIImageView *myImage;
    @property (strong) UIImage* imagePayload;
    ```

10. В `imageViewController.m` добавьте следующий код в конце `viewDidload`.

    ```objc
    // Display the UI Image in UI Image View
    [self.myImage setImage:self.imagePayload];
    ```

11. В файле `AppDelegate.m` импортируйте созданный контроллер образа.

    ```objc
    #import "imageViewController.h"
    ```

12. Добавьте раздел интерфейса со следующим объявлением:

    ```objc
    @interface AppDelegate ()

    @property UIImage* imagePayload;
    @property NSDictionary* userInfo;
    @property BOOL iOS8;

    // Obtain content from backend with notification id
    - (void)retrieveRichImageWithId:(int)richId completion: (void(^)(NSError*)) completion;

    // Redirect to Image View Controller after notification interaction
    - (void)redirectToImageViewWithImage: (UIImage *)img;

    @end
    ```

13. Убедитесь, что в `AppDelegate` ваше приложение регистрирует автоматические уведомления в `application: didFinishLaunchingWithOptions`.

    ```objc
    // Software version
    self.iOS8 = [[UIApplication sharedApplication] respondsToSelector:@selector(registerUserNotificationSettings:)] && [[UIApplication sharedApplication] respondsToSelector:@selector(registerForRemoteNotifications)];

    // Register for remote notifications for iOS8 and previous versions
    if (self.iOS8) {
        NSLog(@"This device is running with iOS8.");

        // Action
        UIMutableUserNotificationAction *richPushAction = [[UIMutableUserNotificationAction alloc] init];
        richPushAction.identifier = @"richPushMore";
        richPushAction.activationMode = UIUserNotificationActivationModeForeground;
        richPushAction.authenticationRequired = NO;
        richPushAction.title = @"More";

        // Notification category
        UIMutableUserNotificationCategory* richPushCategory = [[UIMutableUserNotificationCategory alloc] init];
        richPushCategory.identifier = @"richPush";
        [richPushCategory setActions:@[richPushAction] forContext:UIUserNotificationActionContextDefault];

        // Notification categories
        NSSet* richPushCategories = [NSSet setWithObjects:richPushCategory, nil];

        UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeSound |
                                                UIUserNotificationTypeAlert |
                                                UIUserNotificationTypeBadge
                                                                                    categories:richPushCategories];

        [[UIApplication sharedApplication] registerUserNotificationSettings:settings];
        [[UIApplication sharedApplication] registerForRemoteNotifications];

    }
    else {
        // Previous iOS versions
        NSLog(@"This device is running with iOS7 or earlier versions.");

        [[UIApplication sharedApplication] registerForRemoteNotificationTypes: UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeSound | UIRemoteNotificationTypeNewsstandContentAvailability];
    }

    return YES;
    ```

14. Замените следующую реализацию на, `application:didRegisterForRemoteNotificationsWithDeviceToken` чтобы изменения в интерфейсе раскадровки вступили в силу:

    ```objc
    // Access navigation controller which is at the root of window
    UINavigationController *nc = (UINavigationController *)self.window.rootViewController;
    // Get home view controller from stack on navigation controller
    homeViewController *hvc = (homeViewController *)[nc.viewControllers objectAtIndex:0];
    hvc.deviceToken = deviceToken;
    ```

15. Затем добавьте следующие методы в `AppDelegate.m`, чтобы получить изображения из конечной точки, и отправьте локальное уведомление после получения. Обязательно замените заполнитель `{backend endpoint}` на конечную точку серверной части:

    ```objc
    NSString *const GetNotificationEndpoint = @"{backend endpoint}/api/notifications";

    // Helper: retrieve notification content from backend with rich notification id
    - (void)retrieveRichImageWithId:(int)richId completion: (void(^)(NSError*)) completion {
        UINavigationController *nc = (UINavigationController *)self.window.rootViewController;
        homeViewController *hvc = (homeViewController *)[nc.viewControllers objectAtIndex:0];
        NSString* authenticationHeader = hvc.registerClient.authenticationHeader;
        // Check if authenticated
        if (!authenticationHeader) return;

        NSURLSession* session = [NSURLSession
                                sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]
                                delegate:nil
                                delegateQueue:nil];

        NSURL* requestURL = [NSURL URLWithString:[NSString stringWithFormat:@"%@/%d", GetNotificationEndpoint, richId]];
        NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
        [request setHTTPMethod:@"GET"];
        NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@", authenticationHeader];
        [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

        NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {

            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
            if (!error && httpResponse.statusCode == 200) {
                // From NSData to UIImage
                self.imagePayload = [UIImage imageWithData:data];

                completion(nil);
            }
            else {
                NSLog(@"Error status: %ld, request: %@", (long)httpResponse.statusCode, error);
                if (error)
                    completion(error);
                else {
                    completion([NSError errorWithDomain:@"APICall" code:httpResponse.statusCode userInfo:nil]);
                }
            }
        }];
        [dataTask resume];
    }

    // Handle silent push notifications when id is sent from backend
    - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))handler {
        self.userInfo = userInfo;
        int richId = [[self.userInfo objectForKey:@"richId"] intValue];
        NSString* richType = [self.userInfo objectForKey:@"richType"];

        // Retrieve image data
        if ([richType isEqualToString:@"img"]) {  
            [self retrieveRichImageWithId:richId completion:^(NSError* error) {
                if (!error){
                    // Send local notification
                    UILocalNotification* localNotification = [[UILocalNotification alloc] init];

                    // "5" is arbitrary here to give you enough time to quit out of the app and receive push notifications
                    localNotification.fireDate = [NSDate dateWithTimeIntervalSinceNow:5];
                    localNotification.userInfo = self.userInfo;
                    localNotification.alertBody = [self.userInfo objectForKey:@"richMessage"];
                    localNotification.timeZone = [NSTimeZone defaultTimeZone];

                    // iOS8 categories
                    if (self.iOS8) {
                        localNotification.category = @"richPush";
                    }

                    [[UIApplication sharedApplication] scheduleLocalNotification:localNotification];

                    handler(UIBackgroundFetchResultNewData);
                }
                else{
                    handler(UIBackgroundFetchResultFailed);
                }
            }];
        }
        // Add "else if" here to handle more types of rich content such as url, sound files, etc.
    }
    ```

16. Обработайте предыдущее локальное уведомление, открыв контроллер представления изображений в `AppDelegate.m` со следующими методами.

    ```objc
    // Helper: redirect users to image view controller
    - (void)redirectToImageViewWithImage: (UIImage *)img {
        UINavigationController *navigationController = (UINavigationController*) self.window.rootViewController;
        UIStoryboard *mainStoryboard = [UIStoryboard storyboardWithName:@"Main" bundle: nil];
        imageViewController *imgViewController = [mainStoryboard instantiateViewControllerWithIdentifier: @"imageViewController"];
        // Pass data/image to image view controller
        imgViewController.imagePayload = img;

        // Redirect
        [navigationController pushViewController:imgViewController animated:YES];
    }

    // Handle local notification sent above in didReceiveRemoteNotification
    - (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification {
        if (application.applicationState == UIApplicationStateActive) {
            // Show in-app alert with an extra "more" button
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:notification.alertBody delegate:self cancelButtonTitle:@"OK" otherButtonTitles:@"More", nil];
            [alert show];
        }
        // App becomes active from user's tap on notification
        else {
            [self redirectToImageViewWithImage:self.imagePayload];
        }
    }

    // Handle buttons in in-app alerts and redirect with data/image
    - (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex {
        // Handle "more" button
        if (buttonIndex == 1)
        {
            [self redirectToImageViewWithImage:self.imagePayload];
        }
        // Add "else if" here to handle more buttons
    }

    // Handle notification setting actions in iOS8
    - (void)application:(UIApplication *)application handleActionWithIdentifier:(NSString *)identifier forLocalNotification:(UILocalNotification *)notification completionHandler:(void (^)())completionHandler {
        // Handle richPush related buttons
        if ([identifier isEqualToString:@"richPushMore"]) {
            [self redirectToImageViewWithImage:self.imagePayload];
        }
        completionHandler();
    }
    ```

## <a name="run-the-application"></a>Запуск приложения

1. В XCode запустите приложение на физическом устройстве под управлением iOS (push-уведомления не будут работать в симуляторе).
2. В пользовательском интерфейсе приложения iOS введите имя пользователя и пароль, которые применяются для проверки подлинности, и щелкните **Вход**.
3. Щелкните **Отправить push-уведомление** , после чего в приложении появится предупреждение. Если нажать **Еще**, появится изображение, добавленное в серверную часть приложения.
4. Также можно щелкнуть **Отправить push-уведомление** и сразу же нажать кнопку домашней страницы устройства. Через несколько секунд вы получите push-уведомление. Если коснуться его или нажать «Еще», вы перейдете в приложение и увидите изображение.

[IOS1]: ./media/notification-hubs-aspnet-backend-ios-rich-push/rich-push-ios-1.png
[IOS2]: ./media/notification-hubs-aspnet-backend-ios-rich-push/rich-push-ios-2.png
[IOS3]: ./media/notification-hubs-aspnet-backend-ios-rich-push/rich-push-ios-3.png
[IOS4]: ./media/notification-hubs-aspnet-backend-ios-rich-push/rich-push-ios-4.png
