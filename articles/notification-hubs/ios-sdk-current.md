---
title: Отправка push-уведомлений в iOS с помощью Центров уведомлений Azure и пакета средств разработки для iOS версии 3.0.0 (предварительная версия 1)
description: Из этого руководства вы узнаете, как отправлять push-уведомления на устройства iOS (версия 3.0.0) с помощью Центров уведомлений Azure (версия 3.0.0, предварительная версия 1) и службы push-уведомлений Apple.
author: sethmanheim
ms.author: sethm
ms.date: 06/19/2020
ms.topic: tutorial
ms.service: notification-hubs
ms.reviewer: thsomasu
ms.lastreviewed: 06/01/2020
ms.openlocfilehash: f905bfa830bfc78caa6ebb02ae49d4839168367b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100598246"
---
# <a name="tutorial-send-push-notifications-to-ios-apps-using-azure-notification-hubs-sdk-for-apple"></a>Руководство по отправке push-уведомлений в приложения iOS с помощью пакета SKD для Центров уведомлений Azure для Apple

В этом учебнике описано, как использовать Центры уведомлений Azure для отправки push-уведомлений в приложение iOS с помощью пакета SDK для Центров уведомлений Azure для Apple.

В этом руководстве рассматриваются следующие действия:

- создание примера приложения iOS;
- подключение приложения iOS к Центрам уведомлений Azure;
- отправка тестовых push-уведомлений;
- проверка получения уведомлений приложением.

Готовый код для этого руководства вы можете скачать на сайте [GitHub](https://github.com/Azure/azure-notificationhubs-ios/tree/main/SampleNHAppObjC).

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

   - Интеграция путем копирования двоичных файлов в проект.

      Для интеграции вы можете просто скопировать двоичные файлы в проект, как описано ниже.

        - Скачайте и распакуйте платформу для [пакета средств разработки Центров уведомлений Azure](https://github.com/Azure/azure-notificationhubs-iOS/releases/), которая предоставляется в виде ZIP-файла.

        - В XCode щелкните проект правой кнопкой мыши и выберите параметр **Add Files to** (Добавить файлы в), чтобы добавить папку **WindowsAzureMessaging.framework** в проект XCode. Нажмите кнопку **Options** (Параметры), установите флажок **Copy items if needed** (Копировать элементы при необходимости) и нажмите кнопку **Add** (Добавить).

          :::image type="content" source="media/ios-sdk/image4.png" alt-text="Добавление платформы":::

6. Добавьте или измените файл с именем **DevSettings.plist**, который содержит два свойства: `CONNECTION_STRING` для строки подключения к Центру уведомлений Azure и `HUB_NAME` для имени Центра уведомлений Azure.

7. Добавьте сведения для подключения к Центрам уведомлений Azure в соответствующем разделе `<string></string>`.  Замените строковые литералы`--HUB-NAME--` и `--CONNECTION-STRING--`, выполняющие роль заполнителей, именем центра и значением **DefaultListenSharedAccessSignature**, которые вы получили ранее на портале:

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

8. В том же файле **AppDelegate.m** замените весь код после `didFinishLaunchingWithOptions` следующим:

    ```objc
    #import <WindowsAzureMessaging/WindowsAzureMessaging.h>
    #import <UserNotifications/UserNotifications.h>

    // Extend the AppDelegate to listen for messages using MSNotificationHubDelegate and User Notification Center
    @interface AppDelegate () <MSNotificationHubDelegate>

    @end

    @implementation AppDelegate
    
    @synthesize notificationPresentationCompletionHandler;
    @synthesize notificationResponseCompletionHandler;

    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

        NSString *path = [[NSBundle mainBundle] pathForResource:@"DevSettings" ofType:@"plist"];
        NSDictionary *configValues = [NSDictionary dictionaryWithContentsOfFile:path];
        
        NSString *connectionString = [configValues objectForKey:@"CONNECTION_STRING"];
        NSString *hubName = [configValues objectForKey:@"HUB_NAME"];

        if([connectionString length] != 0 && [hubName length] != 0) {
            [[UNUserNotificationCenter currentNotificationCenter] setDelegate:self];
            [MSNotificationHub setDelegate:self];
            [MSNotificationHub initWithConnectionString:connectionString withHubName:hubName];
    
            return YES;
        }

        NSLog(@"Please setup CONNECTION_STRING and HUB_NAME in DevSettings.plist and restart application");

        exit(-1);
    }

    - (void)notificationHub:(MSNotificationHub *)notificationHub didReceivePushNotification:(MSNotificationHubMessage *)message {
        // Send message using NSNotificationCenter with the message
        NSDictionary *userInfo = [NSDictionary dictionaryWithObject:message forKey:@"message"];
        [[NSNotificationCenter defaultCenter] postNotificationName:@"MessageReceived" object:nil userInfo:userInfo];
    }

    @end
    ```

    Этот код подключается к центру уведомлений, используя сведения о подключении, указанные в файле **DevSettings.plist**. Затем он передает маркер устройства в центр уведомлений, чтобы тот мог отправлять уведомления.

### <a name="create-notificationdetailviewcontroller-header-file"></a>Создание файла заголовка NotificationDetailViewController

1. Так же, как вы сделали это ранее, добавьте другой файл заголовка с именем **SetupViewController.h**. Замените содержимое нового файла приведенным ниже кодом:

   ```objc
   #import <UIKit/UIKit.h>

   NS_ASSUME_NONNULL_BEGIN

   @interface SetupViewController : UIViewController

   @end

   NS_ASSUME_NONNULL_END
   ```

2. Добавьте файл реализации **SetupViewController.m**. Замените содержимое этого файла следующим кодом, который реализует методы UIViewController:

   ```objc
   #import "SetupViewController.h"

    static NSString *const kNHMessageReceived = @"MessageReceived";
    
    @interface SetupViewController ()
    
    @end

    @implementation SetupViewController

    - (void)viewDidLoad {
        [super viewDidLoad];
        
        // Listen for messages using NSNotificationCenter
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(didReceivePushNotification:) name:kNHMessageReceived object:nil];
    }

    - (void)dealloc {
        // Clean up subscription to NSNotificationCenter
        [[NSNotificationCenter defaultCenter] removeObserver:self name:kNHMessageReceived object:nil];
    }

    - (void)didReceivePushNotification:(NSNotification *)notification {
        MSNotificationHubMessage *message = [notification.userInfo objectForKey:@"message"];

        // Create UI Alert controller with message title and body
        UIAlertController *alertController = [UIAlertController alertControllerWithTitle:message.title
                             message:message.body
                      preferredStyle:UIAlertControllerStyleAlert];
        [alertController addAction:[UIAlertAction actionWithTitle:@"OK" style:UIAlertActionStyleCancel handler:nil]];
        [self presentViewController:alertController animated:YES completion:nil];
        
        // Dismiss after 2 seconds
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [alertController dismissViewControllerAnimated:YES completion: nil];
        });

    }

    @end
   ```

3. Выполните сборку приложения и запустите его на устройстве, чтобы убедиться в отсутствии сбоев.

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