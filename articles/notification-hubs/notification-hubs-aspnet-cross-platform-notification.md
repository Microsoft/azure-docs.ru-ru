---
title: Отправка кроссплатформенных уведомлений пользователям с помощью центров уведомлений Azure (ASP.NET)
description: Узнайте, как использовать шаблоны центров уведомлений для отправки в одном запросе независимых от платформы уведомлений, предназначенных для всех платформ.
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
editor: thsomasu
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: multiple
ms.topic: article
ms.date: 09/14/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 10/02/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 3f8f9357fff4773467dd93046ece8533514eb444
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92313888"
---
# <a name="send-cross-platform-notifications-with-azure-notification-hubs"></a>Отправка межплатформенных уведомлений с помощью концентраторов уведомлений Azure

Это руководство построено на предыдущем учебном курсе, которое [отправляет уведомления конкретным пользователям с помощью центров уведомлений Azure]. В этом учебнике описывается, как отправлять уведомления на все устройства, зарегистрированные для определенного пользователя, прошедшего проверку подлинности. Этот подход требует нескольких запросов на отправку уведомлений каждой поддерживаемой клиентской платформе. Концентраторы уведомлений Azure поддерживают шаблоны, которые позволяют указать, каким образом конкретное устройство должно получать уведомления. Этот метод упрощает отправку кроссплатформенных уведомлений.

В этой статье показано, как использовать шаблоны для отправки уведомлений, нацеленных на все платформы. В этой статье для отправки независимого от платформы уведомления используется один запрос. Более подробные сведения о шаблонах см. в разделе [Общие сведения о концентраторах уведомлений][Templates].

> [!IMPORTANT]
> Windows Phone проекты 8,1 и более ранних версий не поддерживаются в Visual Studio 2019. Дополнительные сведения см. в разделе [Целевая платформа и совместимость для Visual Studio 2019](/visualstudio/releases/2019/compatibility).

> [!NOTE]
> Центры уведомлений позволяют устройству зарегистрировать несколько шаблонов с помощью одного и того же тега. В этом случае входящее сообщение, предназначенное для этого тега, приводит к отправке на устройство нескольких уведомлений, по одному для каждого шаблона. Этот процесс дает возможность отобразить одно сообщение в нескольких визуальных уведомлениях, например в виде значка и в виде всплывающего уведомления в приложении Магазина Windows.

## <a name="send-cross-platform-notifications-using-templates"></a>Отправка кроссплатформенных уведомлений с использованием шаблонов

В этом разделе используется пример кода, созданный в учебнике [Отправка уведомлений для конкретных пользователей с помощью центров уведомлений Azure] . [Полный пример можно скачать на сайте GitHub](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/NotifyUsers).

Чтобы отправлять межплатформенные уведомления с помощью шаблонов, выполните следующие действия.

1. В **Обозреватель решений** Visual Studio разверните папку **Controllers** , а затем откройте файл *RegisterController. CS* .

1. Найдите в методе `Put` блок кода, который создает регистрацию, и замените содержимое `switch` следующим кодом.

    ```csharp
    switch (deviceUpdate.Platform)
    {
        case "mpns":
            var toastTemplate = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                "<wp:Notification xmlns:wp=\"WPNotification\">" +
                    "<wp:Toast>" +
                        "<wp:Text1>$(message)</wp:Text1>" +
                    "</wp:Toast> " +
                "</wp:Notification>";
            registration = new MpnsTemplateRegistrationDescription(deviceUpdate.Handle, toastTemplate);
            break;
        case "wns":
            toastTemplate = @"<toast><visual><binding template=""ToastText01""><text id=""1"">$(message)</text></binding></visual></toast>";
            registration = new WindowsTemplateRegistrationDescription(deviceUpdate.Handle, toastTemplate);
            break;
        case "apns":
            var alertTemplate = "{\"aps\":{\"alert\":\"$(message)\"}}";
            registration = new AppleTemplateRegistrationDescription(deviceUpdate.Handle, alertTemplate);
            break;
        case "fcm":
            var messageTemplate = "{\"data\":{\"message\":\"$(message)\"}}";
            registration = new FcmTemplateRegistrationDescription(deviceUpdate.Handle, messageTemplate);
            break;
        default:
            throw new HttpResponseException(HttpStatusCode.BadRequest);
    }
    ```

    Этот код вызывает метод, используемый на конкретной платформе для регистрации шаблонов вместо собственной регистрации. Регистрация шаблонов является производной от собственной регистрации. Поэтому существующие регистрации изменять не нужно.

1. В **Обозреватель решений** в папке **Controllers** откройте файл **NotificationsController. CS** . Замените метод `Post` следующим кодом:

    ```csharp
    public async Task<HttpResponseMessage> Post()
    {
        var user = HttpContext.Current.User.Identity.Name;
        var userTag = "username:" + user;

        var notification = new Dictionary<string, string> { { "message", "Hello, " + user } };
        await Notifications.Instance.Hub.SendTemplateNotificationAsync(notification, userTag);

        return Request.CreateResponse(HttpStatusCode.OK);
    }
    ```

    Этот код отправляет уведомление на все платформы в одно и то же время. Вы не указываете собственные полезные данные. Центры уведомлений создают и доставляют правильные полезные данные для каждого устройства с указанным значением тега в соответствии с зарегистрированными шаблонами.

1. Повторно опубликуйте проект веб-API.

1. Запустите клиентское приложение еще раз, чтобы убедиться, что регистрация выполнена успешно.

1. При необходимости разверните клиентское приложение на втором устройстве, а затем запустите приложение. Уведомление отображается на каждом из устройств.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы завершили работу с этим руководством, Узнайте больше о концентраторах уведомлений и шаблонах в следующих статьях:

* Сведения о различных сценариях использования шаблонов см. в руководстве [Push-уведомления для конкретных устройств Windows, на которых выполняются универсальная платформа Windows приложения][Use Notification Hubs to send breaking news] .
* Более подробные сведения о шаблонах см. в разделе [Общие сведения о концентраторах уведомлений][Templates].

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[Push to users ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Push to users Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Visual Studio 2012 Express for Windows 8]: https://visualstudio.microsoft.com/downloads/

[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Azure Notification Hubs]: https://go.microsoft.com/fwlink/p/?LinkId=314257
[Отправка уведомлений определенным пользователям с помощью службы "Центры уведомлений Azure"]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Templates]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hub How to for Windows Store]: /previous-versions/azure/azure-services/jj927170(v=azure.100)