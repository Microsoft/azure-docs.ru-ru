---
title: Отправка push-уведомлений конкретным пользователям с помощью концентраторов уведомлений Azure | Документация Майкрософт
description: Узнайте, как отправлять push-уведомления конкретным пользователям iOS с помощью центров уведомлений Azure.
documentationcenter: ios
author: sethmanheim
manager: femila
services: notification-hubs
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 08/07/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: 167c666c536ee33531fd069dbd1edb530331a9f3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96016963"
---
# <a name="tutorial-send-push-notifications-to-specific-users-using-azure-notification-hubs"></a>Руководство. Отправка push-уведомлений конкретным пользователям с помощью центров уведомлений Azure

[!INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]

В этом учебнике показано, как использовать концентраторы уведомлений Azure для отправки push-уведомлений пользователю определенного приложения на конкретном устройстве. Серверная часть ASP.NET WebAPI используется для проверки подлинности клиентов и создания уведомлений, как показано в разделе руководства [Управление регистрацией из серверной части](notification-hubs-push-notification-registration-management.md#registration-management-from-a-backend).

При работе с этим руководством вы выполните следующие задачи:

> [!div class="checklist"]
> * Создание проекта веб-API
> * Аутентификация клиентов в серверной части веб-API.
> * Регистрация для получения уведомлений с помощью серверной части веб-API.
> * Отправка уведомлений из серверной части веб-API.
> * Публикация новой серверной части веб-API.
> * Изменение приложения iOS
> * Тестирование приложения

## <a name="prerequisites"></a>Предварительные требования

В этом руководстве предполагается, что вы создали и настроили центр уведомлений, как описано в статье [Отправка push-уведомлений в приложения iOS с помощью центров уведомлений Azure](ios-sdk-get-started.md). Это руководство также необходимо изучить перед переходом к руководству [Безопасные push-уведомления для концентраторов уведомлений Azure](notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md).
Если вы хотите использовать мобильные приложения в качестве внутренней службы, см. статью [Добавление push-уведомлений в приложение iOS](/previous-versions/azure/app-service-mobile/app-service-mobile-ios-get-started-push).

[!INCLUDE [notification-hubs-aspnet-backend-notifyusers](../../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## <a name="modify-your-ios-app"></a>Изменение приложения iOS

1. Откройте приложение просмотра одной страницы, созданное в руководстве [Отправка push-уведомлений в приложения iOS с помощью центров уведомлений Azure](ios-sdk-get-started.md) .

   > [!NOTE]
   > В этом разделе подразумевается, что вы настроили свой проект, указав пустое имя организации. Если это не так, добавьте в начало названия организации все имена классов.

2. В файл `Main.storyboard` добавьте компоненты из библиотеки объектов, показанные на снимке экрана.

    ![Изменение раскадровки в конструкторе интерфейса Xcode][1]

   * **Имя пользователя**: текстовое поле UITextField с замещающим текстом *Enter Username*, расположенное под меткой отправки результатов, справа, слева и сверху ограниченное полями.
   * **Пароль**: текстовое поле UITextField с замещающим текстом *Enter Password*, расположенное под текстовым полем имени пользователя, справа, слева и сверху ограниченное полями. Отметьте параметр **Защищенный ввод текста** в инспекторе атрибутов в разделе *Символ вывода*.
   * **Вход**: кнопка UIButton с меткой, которая расположена под текстовым полем пароля. Снимите флажок **Включить** в инспекторе атрибутов в разделе *Управление содержимым*.
   * **WNS**: метка и переключатель для включения отправки уведомлений службы уведомлений Windows, если она установлена в центре. См. руководство [Начало работы с центрами уведомлений для приложений универсальной платформы Windows](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).
   * **GCM**: метка и переключатель для включения отправки уведомлений в службу Google Cloud Messaging, если она установлена в центре. См. руководство [Отправка push-уведомлений в приложения Android с помощью центров уведомлений Azure](notification-hubs-android-push-notification-google-gcm-get-started.md).
   * **APNS**: метка и переключатель для включения отправки уведомлений в службу уведомлений платформы Apple Platform Notification Service.
   * **Имя получателя**: текстовое поле UITextField с замещающим текстом *Введите имя получателя*, расположенное непосредственно под меткой GCM, справа, слева и сверху ограниченное полями.

     Некоторые компоненты были добавлены в учебнике [Отправка push-уведомлений в приложения iOS с помощью центров уведомлений Azure](ios-sdk-get-started.md) .

3. **Перетащите указатель** мыши от компонентов в представлении к `ViewController.h` и добавьте следующие новые розетки:

    ```objc
    @property (weak, nonatomic) IBOutlet UITextField *UsernameField;
    @property (weak, nonatomic) IBOutlet UITextField *PasswordField;
    @property (weak, nonatomic) IBOutlet UITextField *RecipientField;
    @property (weak, nonatomic) IBOutlet UITextField *NotificationField;

    // Used to enable the buttons on the UI
    @property (weak, nonatomic) IBOutlet UIButton *LogInButton;
    @property (weak, nonatomic) IBOutlet UIButton *SendNotificationButton;

    // Used to enabled sending notifications across platforms
    @property (weak, nonatomic) IBOutlet UISwitch *WNSSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *GCMSwitch;
    @property (weak, nonatomic) IBOutlet UISwitch *APNSSwitch;

    - (IBAction)LogInAction:(id)sender;
    ```

4. В файл `ViewController.h` добавьте следующее `#define` после импорта операторов. Замените заполнитель `<Your backend endpoint>` URL-адресом назначения, который использовался для развертывания серверной части приложения в предыдущем разделе. Например, `http://your_backend.azurewebsites.net`:

    ```objc
    #define BACKEND_ENDPOINT @"<Your backend endpoint>"
    ```

5. В проекте создайте класс Cocoa Touch с именем `RegisterClient` для взаимодействия с серверной частью ASP.NET. Создайте класс, наследующий `NSObject`. Затем добавьте следующий код в `RegisterClient.h` :

    ```objc
    @interface RegisterClient : NSObject

    @property (strong, nonatomic) NSString* authenticationHeader;

    -(void) registerWithDeviceToken:(NSData*)token tags:(NSSet*)tags
        andCompletion:(void(^)(NSError*))completion;

    -(instancetype) initWithEndpoint:(NSString*)Endpoint;

    @end
    ```

6. В файле `RegisterClient.m` измените раздел `@interface` следующим образом.

    ```objc
    @interface RegisterClient ()

    @property (strong, nonatomic) NSURLSession* session;
    @property (strong, nonatomic) NSURLSession* endpoint;

    -(void) tryToRegisterWithDeviceToken:(NSData*)token tags:(NSSet*)tags retry:(BOOL)retry
                andCompletion:(void(^)(NSError*))completion;
    -(void) retrieveOrRequestRegistrationIdWithDeviceToken:(NSString*)token
                completion:(void(^)(NSString*, NSError*))completion;
    -(void) upsertRegistrationWithRegistrationId:(NSString*)registrationId deviceToken:(NSString*)token
                tags:(NSSet*)tags andCompletion:(void(^)(NSURLResponse*, NSError*))completion;

    @end
    ```

7. Замените раздел `@implementation` в RegisterClient.m следующим кодом.

    ```objc
    @implementation RegisterClient

    // Globals used by RegisterClient
    NSString *const RegistrationIdLocalStorageKey = @"RegistrationId";

    -(instancetype) initWithEndpoint:(NSString*)Endpoint
    {
        self = [super init];
        if (self) {
            NSURLSessionConfiguration* config = [NSURLSessionConfiguration defaultSessionConfiguration];
            _session = [NSURLSession sessionWithConfiguration:config delegate:nil delegateQueue:nil];
            _endpoint = Endpoint;
        }
        return self;
    }

    -(void) registerWithDeviceToken:(NSData*)token tags:(NSSet*)tags
                andCompletion:(void(^)(NSError*))completion
    {
        [self tryToRegisterWithDeviceToken:token tags:tags retry:YES andCompletion:completion];
    }

    -(void) tryToRegisterWithDeviceToken:(NSData*)token tags:(NSSet*)tags retry:(BOOL)retry
                andCompletion:(void(^)(NSError*))completion
    {
        NSSet* tagsSet = tags?tags:[[NSSet alloc] init];

        NSString *deviceTokenString = [[token description]
            stringByTrimmingCharactersInSet:[NSCharacterSet characterSetWithCharactersInString:@"<>"]];
        deviceTokenString = [[deviceTokenString stringByReplacingOccurrencesOfString:@" " withString:@""]
                                uppercaseString];

        [self retrieveOrRequestRegistrationIdWithDeviceToken: deviceTokenString
            completion:^(NSString* registrationId, NSError *error) {
            NSLog(@"regId: %@", registrationId);
            if (error) {
                completion(error);
                return;
            }

            [self upsertRegistrationWithRegistrationId:registrationId deviceToken:deviceTokenString
                tags:tagsSet andCompletion:^(NSURLResponse * response, NSError *error) {
                if (error) {
                    completion(error);
                    return;
                }

                NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*)response;
                if (httpResponse.statusCode == 200) {
                    completion(nil);
                } else if (httpResponse.statusCode == 410 && retry) {
                    [self tryToRegisterWithDeviceToken:token tags:tags retry:NO andCompletion:completion];
                } else {
                    NSLog(@"Registration error with response status: %ld", (long)httpResponse.statusCode);

                    completion([NSError errorWithDomain:@"Registration" code:httpResponse.statusCode
                                userInfo:nil]);
                }

            }];
        }];
    }

    -(void) upsertRegistrationWithRegistrationId:(NSString*)registrationId deviceToken:(NSData*)token
                tags:(NSSet*)tags andCompletion:(void(^)(NSURLResponse*, NSError*))completion
    {
        NSDictionary* deviceRegistration = @{@"Platform" : @"apns", @"Handle": token,
                                                @"Tags": [tags allObjects]};
        NSData* jsonData = [NSJSONSerialization dataWithJSONObject:deviceRegistration
                            options:NSJSONWritingPrettyPrinted error:nil];

        NSLog(@"JSON registration: %@", [[NSString alloc] initWithData:jsonData
                                            encoding:NSUTF8StringEncoding]);

        NSString* endpoint = [NSString stringWithFormat:@"%@/api/register/%@", _endpoint,
                                registrationId];
        NSURL* requestURL = [NSURL URLWithString:endpoint];
        NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
        [request setHTTPMethod:@"PUT"];
        [request setHTTPBody:jsonData];
        NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
                                                self.authenticationHeader];
        [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];
        [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];

        NSURLSessionDataTask* dataTask = [self.session dataTaskWithRequest:request
            completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
        {
            if (!error)
            {
                completion(response, error);
            }
            else
            {
                NSLog(@"Error request: %@", error);
                completion(nil, error);
            }
        }];
        [dataTask resume];
    }

    -(void) retrieveOrRequestRegistrationIdWithDeviceToken:(NSString*)token
                completion:(void(^)(NSString*, NSError*))completion
    {
        NSString* registrationId = [[NSUserDefaults standardUserDefaults]
                                    objectForKey:RegistrationIdLocalStorageKey];

        if (registrationId)
        {
            completion(registrationId, nil);
            return;
        }

        // request new one & save
        NSURL* requestURL = [NSURL URLWithString:[NSString stringWithFormat:@"%@/api/register?handle=%@",
                                _endpoint, token]];
        NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
        [request setHTTPMethod:@"POST"];
        NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
                                                self.authenticationHeader];
        [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

        NSURLSessionDataTask* dataTask = [self.session dataTaskWithRequest:request
            completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
        {
            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
            if (!error && httpResponse.statusCode == 200)
            {
                NSString* registrationId = [[NSString alloc] initWithData:data
                    encoding:NSUTF8StringEncoding];

                // remove quotes
                registrationId = [registrationId substringWithRange:NSMakeRange(1,
                                    [registrationId length]-2)];

                [[NSUserDefaults standardUserDefaults] setObject:registrationId
                    forKey:RegistrationIdLocalStorageKey];
                [[NSUserDefaults standardUserDefaults] synchronize];

                completion(registrationId, nil);
            }
            else
            {
                NSLog(@"Error status: %ld, request: %@", (long)httpResponse.statusCode, error);
                if (error)
                    completion(nil, error);
                else {
                    completion(nil, [NSError errorWithDomain:@"Registration" code:httpResponse.statusCode
                                userInfo:nil]);
                }
            }
        }];
        [dataTask resume];
    }

    @end
    ```

    В этом коде реализуется логика, описываемая в статье [Регистрация из серверной части приложения](notification-hubs-push-notification-registration-management.md#registration-management-from-a-backend), при этом используется NSURLSession для выполнения вызовов REST к серверной части приложения и NSUserDefaults — для локального сохранения идентификатора registrationId, возвращаемого центром уведомлений.

    Для нормальной работы этого класса должно быть правильно задано свойство `authorizationHeader`. Это свойство установлено классом `ViewController` после выполнения входа.

8. В `ViewController.h` добавьте инструкцию `#import` для `RegisterClient.h`. Затем добавьте объявление для токена устройства и ссылку на экземпляр `RegisterClient` в разделе `@interface`.

    ```objc
    #import "RegisterClient.h"

    @property (strong, nonatomic) NSData* deviceToken;
    @property (strong, nonatomic) RegisterClient* registerClient;
    ```

9. Добавьте в ViewController.m объявление частного метода в разделе `@interface` .

    ```objc
    @interface ViewController () <UITextFieldDelegate, NSURLConnectionDataDelegate, NSXMLParserDelegate>

    // create the Authorization header to perform Basic authentication with your app back-end
    -(void) createAndSetAuthenticationHeaderWithUsername:(NSString*)username
                    AndPassword:(NSString*)password;

    @end
    ```

    > [!NOTE]
    > Этот фрагмент не является безопасной схемой проверки подлинности. Следует заменить реализацию `createAndSetAuthenticationHeaderWithUsername:AndPassword:` своим собственным механизмом проверки подлинности, формирующим маркер проверки подлинности, который используется классом клиента регистра, например, OAuth, Active Directory.

10. Затем в разделе `@implementation` в `ViewController.m` добавьте приведенный ниже код, который задает маркер устройства и заголовок аутентификации.

    ```objc
    -(void) setDeviceToken: (NSData*) deviceToken
    {
        _deviceToken = deviceToken;
        self.LogInButton.enabled = YES;
    }

    -(void) createAndSetAuthenticationHeaderWithUsername:(NSString*)username
                    AndPassword:(NSString*)password;
    {
        NSString* headerValue = [NSString stringWithFormat:@"%@:%@", username, password];

        NSData* encodedData = [[headerValue dataUsingEncoding:NSUTF8StringEncoding] base64EncodedDataWithOptions:NSDataBase64EncodingEndLineWithCarriageReturn];

        self.registerClient.authenticationHeader = [[NSString alloc] initWithData:encodedData
                                                    encoding:NSUTF8StringEncoding];
    }

    -(BOOL)textFieldShouldReturn:(UITextField *)textField
    {
        [textField resignFirstResponder];
        return YES;
    }
    ```

    Обратите внимание на то, как Настройка маркера устройства включает кнопку **Вход** . Это происходит по той причине, что контроллер представления, являясь частью действия входа, регистрирует push-уведомления в серверной части приложения. Вы не хотите, чтобы действие **журнала** было доступно, пока маркер устройства не будет настроен правильно. Можно разделить вход и push-регистрацию, если первая операция выполняется раньше второй.

11. В ViewController. m используйте следующие фрагменты кода, чтобы реализовать метод действия для кнопки **входа в систему** и метод отправки сообщения уведомления с помощью серверной части ASP.NET.

    ```objc
    - (IBAction)LogInAction:(id)sender {
        // create authentication header and set it in register client
        NSString* username = self.UsernameField.text;
        NSString* password = self.PasswordField.text;

        [self createAndSetAuthenticationHeaderWithUsername:username AndPassword:password];

        __weak ViewController* selfie = self;
        [self.registerClient registerWithDeviceToken:self.deviceToken tags:nil
            andCompletion:^(NSError* error) {
            if (!error) {
                dispatch_async(dispatch_get_main_queue(),
                ^{
                    selfie.SendNotificationButton.enabled = YES;
                    [self MessageBox:@"Success" message:@"Registered successfully!"];
                });
            }
        }];
    }

    - (void)SendNotificationASPNETBackend:(NSString*)pns UsernameTag:(NSString*)usernameTag
                Message:(NSString*)message
    {
        NSURLSession* session = [NSURLSession
            sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:nil
            delegateQueue:nil];

        // Pass the pns and username tag as parameters with the REST URL to the ASP.NET backend
        NSURL* requestURL = [NSURL URLWithString:[NSString
            stringWithFormat:@"%@/api/notifications?pns=%@&to_tag=%@", BACKEND_ENDPOINT, pns,
            usernameTag]];

        NSMutableURLRequest* request = [NSMutableURLRequest requestWithURL:requestURL];
        [request setHTTPMethod:@"POST"];

        // Get the mock authenticationheader from the register client
        NSString* authorizationHeaderValue = [NSString stringWithFormat:@"Basic %@",
            self.registerClient.authenticationHeader];
        [request setValue:authorizationHeaderValue forHTTPHeaderField:@"Authorization"];

        //Add the notification message body
        [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];
        [request setHTTPBody:[message dataUsingEncoding:NSUTF8StringEncoding]];

        // Execute the send notification REST API on the ASP.NET Backend
        NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
            completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
        {
            NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
            if (error || httpResponse.statusCode != 200)
            {
                NSString* status = [NSString stringWithFormat:@"Error Status for %@: %d\nError: %@\n",
                                    pns, httpResponse.statusCode, error];
                dispatch_async(dispatch_get_main_queue(),
                ^{
                    // Append text because all 3 PNS calls may also have information to view
                    [self.sendResults setText:[self.sendResults.text stringByAppendingString:status]];
                });
                NSLog(status);
            }

            if (data != NULL)
            {
                xmlParser = [[NSXMLParser alloc] initWithData:data];
                [xmlParser setDelegate:self];
                [xmlParser parse];
            }
        }];
        [dataTask resume];
    }
    ```

12. Обновите действие для кнопки **Отправить уведомление** для использования серверной части ASP.NET и отправки любой PNS, разрешенной коммутатором.

    ```objc
    - (IBAction)SendNotificationMessage:(id)sender
    {
        //[self SendNotificationRESTAPI];
        [self SendToEnabledPlatforms];
    }

    -(void)SendToEnabledPlatforms
    {
        NSString* json = [NSString stringWithFormat:@"\"%@\"",self.notificationMessage.text];

        [self.sendResults setText:@""];

        if ([self.WNSSwitch isOn])
            [self SendNotificationASPNETBackend:@"wns" UsernameTag:self.RecipientField.text Message:json];

        if ([self.GCMSwitch isOn])
            [self SendNotificationASPNETBackend:@"gcm" UsernameTag:self.RecipientField.text Message:json];

        if ([self.APNSSwitch isOn])
            [self SendNotificationASPNETBackend:@"apns" UsernameTag:self.RecipientField.text Message:json];
    }
    ```

13. В функции `ViewDidLoad` добавьте следующий код, чтобы создать экземпляр `RegisterClient` и задать делегат для текстовых полей.

    ```objc
    self.UsernameField.delegate = self;
    self.PasswordField.delegate = self;
    self.RecipientField.delegate = self;
    self.registerClient = [[RegisterClient alloc] initWithEndpoint:BACKEND_ENDPOINT];
    ```

14. Теперь удалите в `AppDelegate.m` все содержимое метода `application:didRegisterForPushNotificationWithDeviceToken:` и замените его следующим содержимым (чтобы убедиться, что контроллер представления содержит последний маркер устройства, полученный из APNs).

    ```objc
    // Add import to the top of the file
    #import "ViewController.h"

    - (void)application:(UIApplication *)application
                didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
    {
        ViewController* rvc = (ViewController*) self.window.rootViewController;
        rvc.deviceToken = deviceToken;
    }
    ```

15. Наконец, убедитесь, что в `AppDelegate.m` есть следующий метод.

    ```objc
    - (void)application:(UIApplication *)application didReceiveRemoteNotification: (NSDictionary *)userInfo {
        NSLog(@"%@", userInfo);
        [self MessageBox:@"Notification" message:[[userInfo objectForKey:@"aps"] valueForKey:@"alert"]];
    }
    ```

## <a name="test-the-application"></a>Тестирование приложения

1. В XCode запустите приложение на физическом устройстве под управлением iOS (push-уведомления не будут работать в симуляторе).
2. В пользовательском интерфейсе приложения iOS введите одинаковое значение для имени пользователя и пароля. Затем нажмите кнопку **войти**.

    ![Тестирование приложения iOS][2]

3. Должно отобразиться всплывающее окно с сообщением об успешной регистрации. Нажмите кнопку **OK**.

    ![Отображение уведомления о тестировании iOS][3]

4. В текстовом поле \**Тег имени получателя* введите тег имени получателя, используемый при регистрации с другого устройства.
5. Введите сообщение уведомления и нажмите кнопку **Отправить уведомление**. Сообщения уведомления могут получать только устройства с зарегистрированным тегом имени получателя. Сообщения отправляются только таким пользователям.

    ![Отправка уведомлений тестирования iOS с тегами][4]

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как отправлять push-уведомления определенным пользователям, с регистрацией которых связаны теги. Чтобы узнать, как отправлять push-уведомления на основе расположения, перейдите к следующему руководству:

> [!div class="nextstepaction"]
>[Отправка push-уведомлений с определением геозон с помощью Центров уведомлений Azure и Bing Spatial Data](notification-hubs-push-bing-spatial-data-geofencing-notification.md)

[1]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-interface.png
[2]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-enter-user-pwd.png
[3]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-registered.png
[4]: ./media/notification-hubs-aspnet-backend-ios-notify-users/notification-hubs-ios-notify-users-enter-msg.png
