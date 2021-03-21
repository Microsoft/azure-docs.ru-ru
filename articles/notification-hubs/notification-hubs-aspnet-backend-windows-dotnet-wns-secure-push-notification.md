---
title: Безопасное push-уведомление центра уведомлений Azure для Windows
description: Узнайте, как отправлять безопасные push-уведомления в Azure. Примеры кода написаны на C# с использованием API .NET.
author: sethmanheim
manager: femila
editor: thsomasu
services: notification-hubs
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: windows
ms.devlang: dotnet
ms.topic: article
ms.date: 09/14/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 01/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 98e587103e63cd5cc26eab5b00864d00e0b9007f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96019432"
---
# <a name="send-secure-push-notifications-from-azure-notification-hubs"></a>Отправка безопасных push-уведомлений из центров уведомлений Azure

> [!div class="op_single_selector"]
> * [Windows Universal](notification-hubs-aspnet-backend-windows-dotnet-wns-secure-push-notification.md)
> * [iOS](notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md)
> * [Android](notification-hubs-aspnet-backend-android-secure-google-gcm-push-notification.md)

## <a name="overview"></a>Обзор

Поддержка push-уведомлений в Microsoft Azure позволяет получить доступ к простой, многоплатформенной и масштабируемой инфраструктуре для отправки push-уведомлений. Она значительно упрощает реализацию push-уведомлений как для объекта-получателя, так и для корпоративных приложений для мобильных платформ.

Из-за ограничений, связанных с правовыми нормами или обеспечением безопасности, иногда в уведомлении могут присутствовать данные, которые нельзя передать через стандартную инфраструктуру push-уведомлений. В этом учебнике рассказывается о том, как реализовать этот принцип при отправке важной информации через защищенное соединение с проверкой подлинности, установленное между устройством клиента и серверной частью приложения.

На высоком уровне поток можно представить следующим образом.

1. Серверная часть приложения:
   * Сохраняет полезную нагрузку в базе данных серверной части.
   * Отправляет идентификатор этого уведомления устройству (защищаемые сведения не передаются).
2. Приложение на устройстве при получении уведомления:
   * Устройство связывается с серверной частью и запрашивает полезную нагрузку.
   * Приложение может показывать полезную нагрузку в виде уведомления на устройстве.

Стоит отметить: в предыдущем потоке (и в этом учебнике) мы предположили, что устройство сохраняет маркер проверки подлинности в локальном хранилище после входа пользователя. Таким образом обеспечивается удобство работы, так как с помощью этого маркера устройство способно получать безопасные полезные данные уведомлений. Если приложение не сохраняет маркеры проверки подлинности на устройстве, или если истек срок действия маркеров, приложение устройства, после получения уведомления, должно отобразить общее уведомление, предлагая пользователю запустить приложение. Затем приложение выполняет проверку подлинности пользователя и отображает полезную нагрузку уведомления.

В этом руководстве показано, как безопасно отправить push-уведомление. Руководство построено в руководстве по [уведомлению пользователей](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md) , поэтому сначала необходимо выполнить действия, описанные в этом руководстве.

> [!NOTE]
> В этом учебнике предполагается, что вы создали и настроили центр уведомлений, как описано в разделе [Отправка уведомлений в универсальная платформа Windows приложения](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).
> Кроме того, обратите внимание, что для Windows Phone 8.1 требуются учетные данные Windows (не Windows Phone) и что фоновые задачи не работают на Windows Phone 8.0 и в Silverlight 8.1. При работе в приложениями из Магазина Windows уведомления можно получать через фоновую задачу только в том случае, если включен экран блокировки приложения (установите флажок в Appmanifest).

[!INCLUDE [notification-hubs-aspnet-backend-securepush](../../includes/notification-hubs-aspnet-backend-securepush.md)]

## <a name="modify-the-windows-phone-project"></a>Изменение проекта Windows Phone

1. В проекте **NotifyUserWindowsPhone** добавьте следующий код в App.xaml.cs, чтобы зарегистрировать фоновую задачу push-уведомления. В конце метода `OnLaunched()` добавьте следующую строку кода:

    ```csharp
    RegisterBackgroundTask();
    ```

2. Находясь в App.xaml.cs, добавьте следующий код сразу после метода `OnLaunched()` :

    ```csharp
    private async void RegisterBackgroundTask()
    {
        if (!Windows.ApplicationModel.Background.BackgroundTaskRegistration.AllTasks.Any(i => i.Value.Name == "PushBackgroundTask"))
        {
            var result = await BackgroundExecutionManager.RequestAccessAsync();
            var builder = new BackgroundTaskBuilder();

            builder.Name = "PushBackgroundTask";
            builder.TaskEntryPoint = typeof(PushBackgroundComponent.PushBackgroundTask).FullName;
            builder.SetTrigger(new Windows.ApplicationModel.Background.PushNotificationTrigger());
            BackgroundTaskRegistration task = builder.Register();
        }
    }
    ```

3. Добавьте в начало файла App.xaml.cs следующие операторы `using` :

    ```csharp
    using Windows.Networking.PushNotifications;
    using Windows.ApplicationModel.Background;
    ```

4. В меню **Файл** Visual Studio выберите **Сохранить все**.

## <a name="create-the-push-background-component"></a>Создание фонового компонента push-уведомлений

Следующий шаг заключается в создании фонового компонента push-уведомления.

1. В обозревателе решений щелкните правой кнопкой мыши узел верхнего уровня решения (в данном случае — **Solution SecurePush**), а затем нажмите кнопку **Добавить**, после чего щелкните **Создать проект**.
2. Разверните пункт **Приложения Магазина**, затем щелкните **Приложения Windows Phone** и выберите **Компонент среды выполнения Windows (Windows Phone)**. Присвойте проекту имя **PushBackgroundComponent**, а затем нажмите кнопку **ОК**, чтобы создать проект.

    ![Снимок экрана: диалоговое окно "Добавление нового проекта" с выделенным параметром среда выполнения Windows компонент (Windows Phone) Visual C#.][12]
3. В обозревателе решений щелкните правой кнопкой мыши проект **PushBackgroundComponent (Windows Phone 8.1)**, а затем щелкните **Добавить** и выберите **Класс**. Присвойте новому классу имя `PushBackgroundTask.cs`. Щелкните кнопку **Добавить** , чтобы создать класс.
4. Замените все содержимое определения пространства имен `PushBackgroundComponent` следующим кодом, заменив заполнитель `{back-end endpoint}` конечной точкой серверной части, полученной при развертывании серверной части:

    ```csharp
    public sealed class Notification
        {
            public int Id { get; set; }
            public string Payload { get; set; }
            public bool Read { get; set; }
        }

        public sealed class PushBackgroundTask : IBackgroundTask
        {
            private string GET_URL = "{back-end endpoint}/api/notifications/";

            async void IBackgroundTask.Run(IBackgroundTaskInstance taskInstance)
            {
                // Store the content received from the notification so it can be retrieved from the UI.
                RawNotification raw = (RawNotification)taskInstance.TriggerDetails;
                var notificationId = raw.Content;

                // retrieve content
                BackgroundTaskDeferral deferral = taskInstance.GetDeferral();
                var httpClient = new HttpClient();
                var settings = ApplicationData.Current.LocalSettings.Values;
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

                var notificationString = await httpClient.GetStringAsync(GET_URL + notificationId);

                var notification = JsonConvert.DeserializeObject<Notification>(notificationString);

                ShowToast(notification);

                deferral.Complete();
            }

            private void ShowToast(Notification notification)
            {
                ToastTemplateType toastTemplate = ToastTemplateType.ToastText01;
                XmlDocument toastXml = ToastNotificationManager.GetTemplateContent(toastTemplate);
                XmlNodeList toastTextElements = toastXml.GetElementsByTagName("text");
                toastTextElements[0].AppendChild(toastXml.CreateTextNode(notification.Payload));
                ToastNotification toast = new ToastNotification(toastXml);
                ToastNotificationManager.CreateToastNotifier().Show(toast);
            }
        }
    ```

5. В обозревателе решений щелкните правой кнопкой мыши проект **PushBackgroundComponent (Windows Phone 8.1)**, а затем щелкните **Управление пакетами NuGet**.
6. В левой части окна выберите **В сети**.
7. В текстовом поле **Поиск** введите **Клиент HTTP**.
8. В списке результатов выберите **Клиентские библиотеки Microsoft HTTP** и нажмите **Установить**. Выполните установку.
9. Вернитесь к полю NuGet **Поиск** и введите **Json.net**. Установите пакет **Json.NET** , затем закройте окно диспетчера пакетов NuGet.
10. Добавьте следующие инструкции `using` в начало файла `PushBackgroundTask.cs`.

    ```csharp
    using Windows.ApplicationModel.Background;
    using Windows.Networking.PushNotifications;
    using System.Net.Http;
    using Windows.Storage;
    using System.Net.Http.Headers;
    using Newtonsoft.Json;
    using Windows.UI.Notifications;
    using Windows.Data.Xml.Dom;
    ```

11. В обозреватель решений в проекте **проекте notifyuserwindowsphone (Windows Phone 8,1)** щелкните правой кнопкой мыши элемент **ссылки** и выберите команду **Добавить ссылку...**. В диалоговом окне Диспетчер ссылок установите флажок **PushBackgroundComponent** и нажмите кнопку **ОК**.
12. В обозревателе решений дважды щелкните **Package.appxmanifest** в проекте **NotifyUserWindowsPhone (Windows Phone 8.1)**. В поле **Уведомления** установите для параметра **Всплывающие уведомления** значение **Да**.

    ![Снимок экрана обозреватель решений окна, в котором основное внимание уделяется свойству Package. appxmanifest с параметром, имеющим значение "Да", выделенным красным цветом.][3]
13. Находясь в **Package.appxmanifest**, откройте меню **Объявления** вверху. В раскрывающемся списке **Доступные объявления** щелкните **Фоновые задачи** и затем нажмите кнопку **Добавить**.
14. В **Package.appxmanifest** в разделе **Свойства** включите параметр **Push-уведомление**.
15. В **Package.appxmanifest** в области **Параметры приложения** введите **PushBackgroundComponent.PushBackgroundTask** в поле **Точка входа**.

    ![Снимок экрана обозреватель решений окна, которое посвящено пакету. appxmanifest с доступными объявлениями, поддерживаемыми объявлениями, Push-уведомлениями и параметрами точки входа, выделенными красным цветом.][13]
16. В меню **файл** выберите команду **сохранить все**.

## <a name="run-the-application"></a>Выполнение приложения

Для запуска приложения выполните следующие действия:

1. В Visual Studio запустите приложение веб-API **AppBackend** . Отобразится веб-страница ASP.NET.
2. В Visual Studio запустите приложение **NotifyUserWindowsPhone (Windows Phone 8.1)** для Windows Phone. Эмулятор Windows Phone запустится и автоматически загрузит приложение.
3. В пользовательском интерфейсе приложения **NotifyUserWindowsPhone** введите имя пользователя и пароль. Это могут быть любые совпадающие строки.
4. В пользовательском интерфейсе приложения **NotifyUserWindowsPhone** щелкните **Log in and register** (Вход и регистрация). Затем нажмите **Отправить push-уведомление**.

[3]: ./media/notification-hubs-aspnet-backend-windows-dotnet-secure-push/notification-hubs-secure-push3.png
[12]: ./media/notification-hubs-aspnet-backend-windows-dotnet-secure-push/notification-hubs-secure-push12.png
[13]: ./media/notification-hubs-aspnet-backend-windows-dotnet-secure-push/notification-hubs-secure-push13.png
