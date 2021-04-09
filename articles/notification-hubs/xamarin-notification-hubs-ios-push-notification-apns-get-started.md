---
title: Отправка push-уведомлений в Xamarin с помощью Центров уведомлений Azure | Документация Майкрософт
description: Из этого руководства вы узнаете, как использовать Центры уведомлений Azure для отправки push-уведомлений в приложение Xamarin.iOS.
services: notification-hubs
keywords: push-уведомления для IOS, push-сообщения, push-уведомления, push-сообщение
documentationcenter: xamarin
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-ios
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 01/12/2021
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 05/23/2019
ms.openlocfilehash: ff1e5edad05ebd7157f71ad2e099ea88905be4f3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98221142"
---
# <a name="tutorial-send-push-notifications-to-xamarinios-apps-using-azure-notification-hubs"></a>Руководство по Отправка push-уведомлений в приложения Xamarin.iOS с помощью Центров уведомлений Azure

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

## <a name="overview"></a>Обзор

В этом учебнике показано, как использовать Центры уведомлений Azure для отправки push-уведомлений в приложение на платформе iOS. Вы создадите пустое приложение Xamarin.iOS, получающее push-уведомления с помощью [Службы push-уведомлений Apple (APNs)](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html).

По завершении вы сможете рассылать push-уведомления на все устройства, где запущено ваше приложение, с помощью центра уведомлений. Готовый код доступен в примере [приложения NotificationHubs][GitHub].

В рамках этого руководства создается или обновляется код для выполнения следующих задач:

> [!div class="checklist"]
> * Создание файла запроса на подпись сертификата
> * Регистрация приложения для работы с push-уведомлениями
> * Создание профиля подготовки для приложения
> * настройка push-уведомлений iOS в центре уведомлений;
> * Отправка тестовых push-уведомлений

## <a name="prerequisites"></a>Предварительные требования

* **Подписка Azure**. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
* последняя версия [Xcode][Install Xcode]
* устройство под управлением iOS 10 (или более поздней версии);
* [программе для разработчиков на платформе Apple](https://developer.apple.com/programs/) .
* [Visual Studio для Mac]
  
  > [!NOTE]
  > Из-за требований к конфигурации push-уведомлений для устройств iOS развертывание и тестирование примера приложения необходимо выполнять на физическом устройстве iOS (iPhone или iPad), а не в симуляторе.

Изучение этого руководства важно для понимания всех других руководств, посвященных Центрам уведомлений для приложений Xamarin.iOS.

[!INCLUDE [Notification Hubs Enable Apple Push Notifications](../../includes/notification-hubs-enable-apple-push-notifications.md)]

## <a name="connect-your-app-to-the-notification-hub"></a>Подключение приложения к центру уведомлений

### <a name="create-a-new-project"></a>Создание нового проекта

1. В Visual Studio создайте проект iOS и выберите шаблон **Приложение одного представления**, а затем нажмите кнопку **Далее**.

     ![Выбор типа приложения в Visual Studio][31]

2. Введите имя приложения и идентификатор организации, затем нажмите кнопку **Далее** и **Создать**.

3. В представлении решения дважды щелкните *Info.plist* и убедитесь, что идентификатор пакета соответствует указанному при создании профиля подготовки. Для этого просмотрите сведения в разделе **Идентификатор**. В разделе **Подписи** укажите учетную запись разработчика в поле **Команда**, установите флажок Automatically manage signing (Автоматическое управление подписями) и убедитесь, что сертификат для подписи и профиль подготовки заданы автоматически.

    ![Настройка приложения iOS в Visual Studio][32]

4. В представлении "Решение" дважды щелкните файл `Entitlements.plist` и убедитесь, что установлен флажок **Включить push-уведомления**.

    ![Настройка прав iOS в Visual Studio][33]

5. Добавьте пакет службы сообщений Azure. В представлении решения щелкните проект правой кнопкой мыши и последовательно выберите **Добавить** > **Add NuGet Packages** (Добавить пакеты NuGet). Выполните поиск пакета **Xamarin.Azure.NotificationHubs.iOS** и добавьте его в проект.

6. Добавьте новый файл в класс и присвойте ему имя `Constants.cs`. Добавьте указанные далее переменные и замените буквенные заполнители строк на `hubname` и `DefaultListenSharedAccessSignature`, записанные ранее.

    ```csharp
    // Azure app-specific connection string and hub path
    public const string ListenConnectionString = "<Azure DefaultListenSharedAccess Connection String>";
    public const string NotificationHubName = "<Azure Notification Hub Name>";
    ```

7. В `AppDelegate.cs` добавьте следующую инструкцию.

    ```csharp
    using WindowsAzure.Messaging.NotificationHubs;
    using UserNotifications
    ```

8. Создайте реализацию `MSNotificationHubDelegate` в `AppDelegate.cs`:

    ```csharp
    public class AzureNotificationHubListener : MSNotificationHubDelegate
    {
        public override void DidReceivePushNotification(MSNotificationHub notificationHub, MSNotificationHubMessage message)
        {

        }
    }
    ```

9. В файле `AppDelegate.cs` обновите метод `FinishedLaunching()`, как в приведенном ниже коде:

    ```csharp
    public override bool FinishedLaunching(UIApplication application, NSDictionary launchOptions)
    {
        // Set the Message listener
        MSNotificationHub.SetDelegate(new AzureNotificationHubListener());
        
        // Start the SDK
        MSNotificationHub.Start(ListenConnectionString, NotificationHubName);

        return true;
    }
    ```

10. В файле `AppDelegate.cs` реализуйте метод `DidReceivePushNotification` для класса `AzureNotificationHubListener`:

    ```csharp
    public override void DidReceivePushNotification(MSNotificationHub notificationHub, MSNotificationHubMessage message)
    {
        // This sample assumes { aps: { alert: { title: "Hello", body: "World" } } }
        var alertTitle = message.Title ?? "Notification";
        var alertBody = message.Body;

        var myAlert = UIAlertController.Create(alertTitle, alertBody, UIAlertControllerStyle.Alert);
        myAlert.AddAction(UIAlertAction.Create("OK", UIAlertActionStyle.Default, null));
        UIApplication.SharedApplication.KeyWindow.RootViewController.PresentViewController(myAlert, true, null);
    }
    ```

11. Запустите приложение на устройстве.

## <a name="send-test-push-notifications"></a>Отправка тестовых push-уведомлений

Можно проверить, поступают ли в приложение уведомления, с помощью параметра *Тестовая отправка* на [Портал Azure]. Этот параметр позволяет отправить на устройство тестовое push-уведомление.

![Портал Azure — тестовая отправка][30]

Push-уведомления обычно отправляются во внутренней службе, например мобильных приложениях или службе ASP.NET, с помощью совместимой библиотеки. Если для серверной части библиотека недоступна, для отправки уведомлений также можно напрямую использовать REST API.

## <a name="next-steps"></a>Следующие шаги

В рамках этого руководства вы отправили широковещательные уведомления на все устройства iOS, зарегистрированные в серверной части. Сведения о том, как отправлять push-уведомления на конкретные устройства iOS, см. в следующем руководстве:

> [!div class="nextstepaction"]
>[Руководство по отправке push-уведомлений на конкретные устройства Android с помощью Центров уведомлений Azure и Google Cloud Messaging](notification-hubs-ios-xplat-segmented-apns-push-notification.md)

<!-- Images. -->
[6]: ./media/notification-hubs-ios-get-started/notification-hubs-apple-config.png
[7]: ./media/notification-hubs-ios-get-started/notification-hubs-apple-config-cert.png
[213]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-create-console-app.png
[215]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-scheduler1.png
[216]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-scheduler2.png
[30]: ./media/notification-hubs-ios-get-started/notification-hubs-test-send.png
[31]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-create-ios-app.png
[32]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-app-settings.png
[33]: ./media/partner-xamarin-notification-hubs-ios-get-started/notification-hub-entitlements-settings.png

<!-- URLs. -->
[Install Xcode]: https://go.microsoft.com/fwLink/p/?LinkID=266532
[iOS Provisioning Portal]: https://go.microsoft.com/fwlink/p/?LinkId=272456
[Visual Studio для Mac]: https://visualstudio.microsoft.com/vs/mac/
[Local and Push Notification Programming Guide]: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/HandlingRemoteNotifications.html#//apple_ref/doc/uid/TP40008194-CH6-SW1
[Apple Push Notification Service]: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html
[Apple Push Notification Service fwlink]: https://go.microsoft.com/fwlink/p/?LinkId=272584
[GitHub]: https://github.com/xamarin/mobile-samples/tree/master/Azure/NotificationHubs
[Портал Azure]: https://portal.azure.com
