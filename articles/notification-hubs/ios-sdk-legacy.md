---
title: Отправка push-уведомлений в iOS с помощью Центров уведомлений Azure и пакета средств разработки для iOS версии 2.0.4
description: Из этого руководства вы узнаете, как отправлять push-уведомления на устройства iOS (версии 2.0.4) с помощью Центров уведомлений Azure и службы push-уведомлений Apple.
author: sethmanheim
ms.author: sethm
ms.date: 06/19/2020
ms.topic: tutorial
ms.service: notification-hubs
ms.reviewer: thsomasu
ms.lastreviewed: 06/01/2020
ms.openlocfilehash: a1017f7c7aa0ede9e3c91acd3dba510618e15fb6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100598349"
---
# <a name="tutorial-send-push-notifications-to-ios-apps-using-azure-notification-hubs-legacy-api"></a>Руководство по отправке push-уведомлений в приложения iOS с помощью Центров уведомлений Azure (устаревший API)

В этом учебнике описано, как использовать Центры уведомлений Azure для отправки push-уведомлений в приложение iOS с помощью пакета SDK для Центров уведомлений Azure для устаревших API Apple.

В этом руководстве рассматриваются следующие действия:

- создание примера приложения iOS;
- подключение приложения iOS к Центрам уведомлений Azure;
- отправка тестовых push-уведомлений;
- проверка получения уведомлений приложением.

Готовый код для этого руководства вы можете скачать на сайте [GitHub](https://github.com/Azure/azure-notificationhubs-ios/tree/main/SampleNHAppLegacyObjC).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам потребуется:

- компьютер Mac с [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), а также действительный сертификат разработчика, установленный в цепочку ключей;
- устройство iPhone или iPad под управлением iOS 10 или более поздней версии;
- регистрация физического устройства на [портале Apple](https://developer.apple.com/) и настройка связи с сертификатом.

Прежде чем продолжить, обязательно пройдите предыдущее руководство по началу работы с [Центрами уведомлений Azure для приложений iOS](ios-sdk-get-started.md), чтобы создать и настроить учетные данные для отправки push-уведомлений в центре уведомлений. Вы сможете выполнить следующие действия, даже если у вас нет опыта разработки в iOS.

> [!NOTE]
> В соответствии с требованиями к настройке push-уведомлений необходимо развернуть push-уведомления и протестировать их на физическом устройстве под управлением iOS (iPhone или iPad), но не в эмуляторе iOS.

## <a name="connect-your-ios-app-to-notification-hubs"></a>Подключение приложения iOS к центрам уведомлений

1. В XCode создайте новый проект iOS и выберите шаблон **Single View Application** (Приложение с одним представлением).

   :::image type="content" source="media/ios-sdk/image1.png" alt-text="Выбор шаблона":::

2. При настройке параметров нового проекта используйте те же **имя продукта** и **идентификатор организации**, которые вы использовали при указании идентификатора пакета на портале разработчиков Apple.

3. В навигаторе проекта выберите имя проекта в разделе **Цели**, а затем вкладку **Signing & Capabilities** (Подписывание и возможности). Обязательно выберите соответствующую **команду** для учетной записи разработчика Apple. Среда XCode должна автоматически извлечь профиль подготовки, который вы создали ранее, используя ваш идентификатор пакета.

   Если новый профиль подготовки, созданный в Xcode, не отображается, обновите профили для идентификатора подписи. В строке меню щелкните **Xcode** выберите **Preferences** (Настройки), откройте вкладку **Account** (Учетная запись), нажмите кнопку **View Details** (Просмотреть сведения), щелкните свой идентификатор подписи, а затем нажмите кнопку Refresh (Обновить) в нижнем правом углу.

   :::image type="content" source="media/ios-sdk/image2.png" alt-text="Подробнее":::

4. На вкладке **Signing & Capabilities** (Подписывание и возможности) выберите **+ Capability** (+ Возможность). Дважды щелкните **Push-уведомления**, чтобы включить их.

   :::image type="content" source="media/ios-sdk/image3.png" alt-text="Возможность":::

5. Добавьте модули пакета SDK для Центров уведомлений Azure.

   Вы можете интегрировать пакет SDK для Центров уведомлений Azure в свое приложение с помощью [Cocoapods](https://cocoapods.org/) или вручную, добавив двоичные файлы в свой проект.

   - Интеграция с помощью Cocoapods. Добавьте следующие зависимости в файл pod, чтобы включить пакет средств разработки Центров уведомлений Azure в приложение.

      ```ruby
      pod 'AzureNotificationHubs-iOS'
      ```

      - Запустите "pod install", чтобы установить только что определенный объект pod, затем откройте файл с расширением .xcworkspace.

         Если при выполнении pod install возникла ошибка типа **Unable to find a specification for AzureNotificationHubs-iOS** (Не удается найти спецификацию для AzureNotificationHubs-iOS), выполните `pod repo update`, чтобы получить последние объекты pod из репозитория Cocoapods, и еще раз выполните pod install.

   - Интеграция с помощью Carthage. Добавьте в Cartfile следующие зависимости, чтобы включить пакет средств разработки Центров уведомлений Azure в свое приложение:

      ```ruby
      github "Azure/azure-notificationhubs-ios"
      ```

      - Затем обновите зависимости сборки:

      ```shell
      $ carthage update
      ```

      Дополнительные сведения об использовании Carthage см. в [репозитории GitHub](https://github.com/Carthage/Carthage).

   - Интеграция путем копирования двоичных файлов в проект. Для интеграции вы можете просто скопировать двоичные файлы в проект, как описано ниже.

        - Скачайте и распакуйте платформу для [пакета средств разработки Центров уведомлений Azure](https://github.com/Azure/azure-notificationhubs-android/releases), которая предоставляется в виде ZIP-файла.

        - В XCode щелкните проект правой кнопкой мыши и выберите параметр **Add Files to** (Добавить файлы в), чтобы добавить папку **WindowsAzureMessaging.framework** в проект XCode. Нажмите кнопку **Options** (Параметры), установите флажок **Copy items if needed** (Копировать элементы при необходимости) и нажмите кнопку **Add** (Добавить).

          :::image type="content" source="media/ios-sdk/image4.png" alt-text="Добавление платформы":::

6. Добавьте сведения для подключения к Центрам уведомлений Azure в соответствующем разделе `<string></string>`.  Замените строковые литералы`--HUB-NAME--` и `--CONNECTION-STRING--`, выполняющие роль заполнителей, именем центра и значением **DefaultListenSharedAccessSignature**, которые вы получили ранее на портале:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>HUB_NAME</key>
        <string>--HUB-NAME--</string>
        <key>CONNECTION_STRING</key>
        <string>--CONNECTION-STRING--</string>
    </dict>
    </plist>
    ```

7. Откройте файл проекта **AppDelegate.h** и замените его содержимое следующим кодом:

    ```objc
    #import <UIKit/UIKit.h>
    #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
    #import <UserNotifications/UserNotifications.h> 

    @interface AppDelegate : UIResponder <UIApplicationDelegate,   UNUserNotificationCenterDelegate>

    @property (strong, nonatomic) UIWindow *window;

    - (void)handleRegister;
    - (void)handleUnregister;

    @end
    ```

8. В файле проекта **AppDelegate.m** добавьте следующие операторы `import`:

    ```objc
    #import "NotificationDetailViewController.h"
    ```

12. Кроме того, добавьте в файле **AppDelegate.m** следующую строку кода в метод `didFinishLaunchingWithOptions` с учетом вашей версии iOS. Этот код регистрирует маркер вашего устройства в APNs.

    ```objc
    [[UNUserNotificationCenter currentNotificationCenter] setDelegate:self];
    ```

13. В том же файле **AppDelegate.m** замените весь код после `didFinishLaunchingWithOptions` следующим:

    ```objc
    // Tells the app that a remote notification arrived that indicates there is data to be fetched.

    - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler {
        NSLog(@"Received remote (silent) notification");
        [self logNotificationDetails:userInfo];
        
        //
        // Let the system know the silent notification has been processed.
        //
        completionHandler(UIBackgroundFetchResultNoData);
    }

    // Tells the delegate that the app successfully registered with Apple Push Notification service (APNs).

    - (void) application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
        NSMutableSet *tags = [[NSMutableSet alloc] init];
        
        // Load and parse stored tags
        NSString *unparsedTags = [[NSUserDefaults standardUserDefaults] valueForKey:NHUserDefaultTags];
        if (unparsedTags.length > 0) {
            NSArray *tagsArray = [unparsedTags componentsSeparatedByString: @","];
            [tags addObjectsFromArray:tagsArray];
        }
        
        // Register the device with the Notification Hub.
        // If the device has not already been registered, this will create the registration.
        // If the device has already been registered, this will update the existing registration.
        //
        SBNotificationHub* hub = [self getNotificationHub];
        [hub registerNativeWithDeviceToken:deviceToken tags:tags completion:^(NSError* error) {
            if (error != nil) {
                NSLog(@"Error registering for notifications: %@", error);
            } else {
                [self showAlert:@"Registered" withTitle:@"Registration Status"];
            }
        }];
    }

    // UNUserNotificationCenterDelegate methods
    //
    // Asks the delegate how to handle a notification that arrived while the app was running in the  foreground.

    - (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler {
        NSLog(@"Received notification while the application is in the foreground");
        
        // The system calls this delegate method when the app is in the foreground. This allows the app to handle the notification
        // itself (and potentially modify the default system behavior).
        
        // Handle the notification by displaying custom UI.
        //
        [self showNotification:notification.request.content.userInfo];
        
        // Use 'options' to specify which default behaviors to enable.
        // - UNNotificationPresentationOptionsBadge: Apply the notification's badge value to the app’s icon.
        // - UNNotificationPresentationOptionList: Show in the Notification Center
        // - UNNotificationPresentationOptionsSound: Play the sound associated with the notification.
        //
        completionHandler(UNNotificationPresentationOptionsBadge | UNNotificationPresentationOptionsSound);
    }

    // Asks the delegate to process the user's response to a delivered notification.
    //

    - (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)(void))completionHandler {
        NSLog(@"Received notification while the application is in the background");
        
        // The system calls this delegate method when the user taps or responds to the system notification.
        
        // Handle the notification response by displaying custom UI
        //
        [self showNotification:response.notification.request.content.userInfo];
        
        // Let the system know the response has been processed.
        //
        completionHandler();
    }

    // App logic and helpers

    - (SBNotificationHub *)getNotificationHub {
        NSString *path = [[NSBundle mainBundle] pathForResource:@"DevSettings" ofType:@"plist"];
        NSDictionary *configValues = [NSDictionary dictionaryWithContentsOfFile:path];
        
        NSString *connectionString = [configValues objectForKey:@"CONNECTION_STRING"];
        NSString *hubName = [configValues objectForKey:@"HUB_NAME"];
        
        return [[SBNotificationHub alloc] initWithConnectionString:connectionString notificationHubPath:hubName];
    }

    - (void)handleRegister {
        UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
        
        UNAuthorizationOptions options =  UNAuthorizationOptionAlert | UNAuthorizationOptionSound | UNAuthorizationOptionBadge;
        [center requestAuthorizationWithOptions:(options) completionHandler:^(BOOL granted, NSError * _Nullable error) {
            if (error != nil) {
                NSLog(@"Error requesting for authorization: %@", error);
            }

            if (granted) {
                NSLog(@"Authorization granted");
            }
        }];
        [[UIApplication sharedApplication] registerForRemoteNotifications];
    }

    - (void)handleUnregister {
        //
        // Unregister the device with the Notification Hub.
        //
        SBNotificationHub *hub = [self getNotificationHub];
        [hub unregisterNativeWithCompletion:^(NSError* error) {
            if (error != nil) {
                NSLog(@"Error unregistering for push: %@", error);
            } else {
                [self showAlert:@"Unregistered" withTitle:@"Registration Status"];
            }
        }];
    }

    - (void)logNotificationDetails:(NSDictionary *)userInfo {
        if (userInfo != nil) {
            UIApplicationState state = [UIApplication sharedApplication].applicationState;
            BOOL background = state != UIApplicationStateActive;
            NSLog(@"Received %@notification: \n%@", background ? @"(background) " : @"", userInfo);
        }
    }

    - (void)showAlert:(NSString *)message withTitle:(NSString *)title {
        if (title == nil) {
            title = @"Alert";
        }

        UIAlertController *alert = [UIAlertController alertControllerWithTitle:title message:message preferredStyle:UIAlertControllerStyleAlert];
        [alert addAction:[UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleDefault handler:nil]];
        [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:alert animated:YES completion:nil];
    }

    - (void)showNotification:(NSDictionary *)userInfo {
        [self logNotificationDetails:userInfo];
        
        NotificationDetailViewController *notificationDetail = [[NotificationDetailViewController alloc] initWithUserInfo:userInfo];
        [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:notificationDetail animated:YES completion:nil];
    }

    @end
    ```

    Этот код подключается к центру уведомлений, используя сведения о соединении, указанные в проекте **Constants.h**. Затем он передает маркер устройства в центр уведомлений, чтобы тот мог отправлять уведомления.

### <a name="create-notificationdetailviewcontroller-header-file"></a>Создание файла заголовка NotificationDetailViewController

1. Аналогично предыдущим инструкциям добавьте другой файл заголовка **NotificationDetailViewController.h**. Замените содержимое нового файла приведенным ниже кодом:

   ```objc
   #import <UIKit/UIKit.h>

   NS_ASSUME_NONNULL_BEGIN

   @interface NotificationDetailViewController : UIViewController

   @property (strong, nonatomic) IBOutlet UILabel *titleLabel;
   @property (strong, nonatomic) IBOutlet UILabel *bodyLabel;
   @property (strong, nonatomic) IBOutlet UIButton *dismissButton;

   @property (strong, nonatomic) NSDictionary *userInfo;

   - (id)initWithUserInfo:(NSDictionary *)userInfo;

   @end

   NS_ASSUME_NONNULL_END
   ```

2. Добавьте файл реализации **NotificationDetailViewController.m**. Замените содержимое этого файла следующим кодом, который реализует методы UIViewController:

   ```objc
   #import "NotificationDetailViewController.h"

   @interface NotificationDetailViewController ()

   @end

   @implementation NotificationDetailViewController

   - (id)initWithUserInfo:(NSDictionary *)userInfo {
        self = [super initWithNibName:@"NotificationDetail" bundle:nil];
        if (self) {
            _userInfo = userInfo;
        }
        return self;
   }

   - (void)viewDidLayoutSubviews {
        [self.titleLabel sizeToFit];
        [self.bodyLabel sizeToFit];
   }

   - (void)viewDidLoad {
        [super viewDidLoad];
        
        NSString *title = nil;
        NSString *body = nil;
        
        NSDictionary *aps = [_userInfo valueForKey:@"aps"];
        NSObject *alertObject = [aps valueForKey:@"alert"];
        if (alertObject != nil) {
            if ([alertObject isKindOfClass:[NSDictionary class]]) {
                NSDictionary *alertDict = (NSDictionary *)alertObject;
                title = [alertDict valueForKey:@"title"];
                body = [alertObject valueForKey:@"body"];
            } else if ([alertObject isKindOfClass:[NSString class]]) {
                body = (NSString *)alertObject;
            } else {
                NSLog(@"Unable to parse notification content. Unexpected format: %@", alertObject);
            }
        }
        
        if (title == nil) {
            title = @"<unset>";
        }
        
        if (body == nil) {
            body = @"<unset>";
        }
        
        self.titleLabel.text = title;
        self.bodyLabel.text = body;
   }

   - (IBAction)handleDismiss:(id)sender {
        [self dismissViewControllerAnimated:YES completion:nil];
   }

   @end
   ```

### <a name="viewcontroller"></a>ViewController

1. В файле проекта **ViewController.h** добавьте следующие инструкции `import`:

   ```objc
   #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
   #import <UserNotifications/UserNotifications.h>
   ```

2. Кроме того, в **ViewController.h** добавьте следующие объявления свойств после объявления `@interface`:

   ```objc
   @property (strong, nonatomic) IBOutlet UITextField *tagsTextField;
   @property (strong, nonatomic) IBOutlet UIButton *registerButton;
   @property (strong, nonatomic) IBOutlet UIButton *unregisterButton;
   ```

3. Замените содержимое файла реализации проекта **ViewController.m** следующим кодом:

   ```objc
   #import "ViewController.h"
   #import "AppDelegate.h"

    static NSString *const kNHUserDefaultTags = @"notification_tags";

   @interface ViewController ()

   @end

   @implementation ViewController

   // UIViewController methods

   - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        // Simple method to dismiss keyboard when user taps outside of the UITextField.
        [self.view endEditing:YES];
   }

   - (void)viewDidLoad {
        [super viewDidLoad];
        
        // Load raw tags text from storage and initialize the text field
        self.tagsTextField.text = [[NSUserDefaults standardUserDefaults] valueForKey:kNHUserDefaultTags];
   }

   - (IBAction)handleRegister:(id)sender {
        // Save raw tags text in storage
        [[NSUserDefaults standardUserDefaults] setValue:self.tagsTextField.text forKey:kNHUserDefaultTags];
        
        // Delegate processing the register action to the app delegate.
        [[[UIApplication sharedApplication] delegate] performSelector:@selector(handleRegister)];
   }

   - (IBAction)handleUnregister:(id)sender {
        [[[UIApplication sharedApplication] delegate] performSelector:@selector(handleUnregister)];
   }

   @end
   ```

4. Выполните сборку приложения и запустите его на устройстве, чтобы убедиться в отсутствии сбоев.

## <a name="send-test-push-notifications"></a>Отправка тестовых push-уведомлений

Можно проверить, поступают ли в приложение уведомления, с помощью параметра **Тестовая отправка** на [Портал Azure](https://portal.azure.com/). Этот параметр позволяет отправить на устройство тестовое push-уведомление.

:::image type="content" source="media/ios-sdk/image6.png" alt-text="Тестовая отправка":::

Push-уведомления обычно отправляются во внутренней службе, например мобильных приложениях или службе ASP.NET, с помощью совместимой библиотеки. Если для серверной части библиотека недоступна, для отправки уведомлений также можно напрямую использовать REST API.

Ниже приведен список других учебников, касающихся отправки уведомлений:

- Мобильные приложения Azure. Пример отправки уведомлений с сервера мобильных приложений, интегрированного с Центрами уведомлений, см. в статье [Add Push Notifications to your iOS App](/previous-versions/azure/app-service-mobile/app-service-mobile-ios-get-started-push) (Добавление push-уведомлений в приложение iOS).
- ASP.NET: [Отправка push-уведомлений определенным пользователям с помощью Центров уведомлений](notification-hubs-aspnet-backend-ios-apple-apns-notification.md).
- Пакет SDK для Центров уведомлений Azure для Java: сведения об отправке уведомлений Java см. в статье [Использование концентраторов уведомлений из Java](notification-hubs-java-push-notification-tutorial.md). Было протестировано в Eclipse для разработки для Android.
- PHP: [Использование Центров уведомлений из PHP](notification-hubs-php-push-notification-tutorial.md).

## <a name="verify-that-your-app-receives-push-notifications"></a>Проверка получения push-уведомлений приложением

Для тестирования push-уведомлений в iOS необходимо развернуть приложение на физическом устройстве под управлением iOS. Отправка push-уведомлений Apple через эмулятор iOS невозможна.

1. Запустите приложение и убедитесь, что оно успешно зарегистрировано, а затем нажмите кнопку **ОК**.

   :::image type="content" source="media/ios-sdk/image7.png" alt-text="Зарегистрировать":::

2. Теперь отправьте тестовое push-уведомление с [портала Azure](https://portal.azure.com/), как описано в предыдущем разделе.

3. Push-уведомление отправляется на все устройства, зарегистрированные для получения уведомлений от используемого центра уведомлений.

   :::image type="content" source="media/ios-sdk/image8.png" alt-text="Тестовая отправка":::

## <a name="next-steps"></a>Дальнейшие действия

В этом простом примере мы отправляем push-уведомления на все зарегистрированные устройства iOS. Сведения о том, как отправлять push-уведомления на конкретные устройства iOS, см. в следующем руководстве:

[Руководство. Отправка push-уведомлений на конкретные устройства](notification-hubs-ios-xplat-segmented-apns-push-notification.md)

Дополнительные сведения см. в следующих статьях:

- [Общие сведения о Центрах уведомлений Azure](notification-hubs-push-notification-overview.md)
- [Использование интерфейса REST центров уведомлений](/rest/api/notificationhubs/)
- [Пакет средств разработки Центров уведомлений для операций серверной части](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/)
- [Пакет средств разработки для Центров уведомлений на GitHub](https://github.com/Azure/azure-notificationhubs)
- [Регистрация в серверной части приложения](notification-hubs-ios-aspnet-register-user-from-backend-to-push-notification.md)
- [Управление регистрацией](notification-hubs-push-notification-registration-management.md)
- [Работа с тегами](notification-hubs-tags-segment-push-message.md)
- [Работа с пользовательскими шаблонами](notification-hubs-templates-cross-platform-push-messages.md)
- [Управление доступом к служебной шине с помощью подписанных URL-адресов](../service-bus-messaging/service-bus-sas.md)
- [Программное создание маркеров SAS](/rest/api/eventhub/generate-sas-token)
- [Безопасность в Apple: Common Crypto](https://developer.apple.com/security/)
- [Формат времени UNIX Epoch](https://en.wikipedia.org/wiki/Unix_time)
- [Код проверки подлинности сообщений с помощью хэш-функций](https://en.wikipedia.org/wiki/HMAC)