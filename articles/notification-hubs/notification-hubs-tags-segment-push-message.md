---
title: Маршрутизация и выражения тегов в центрах уведомлений Azure
description: Узнайте, как маршрутизировать и отмечать выражения для центров уведомлений Azure.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 0fffb3bb-8ed8-4e0f-89e8-0de24a47f644
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 12/09/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 12/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 4ff8225522e79e2be40682fb5e4823777dde2aa0
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88998309"
---
# <a name="routing-and-tag-expressions"></a>Маршрутизация и выражения тегов

## <a name="overview"></a>Обзор

Выражения тегов позволяют выбирать определенные целевые наборы устройств, а точнее регистраций, при отправке push-уведомлений через центры уведомлений.

## <a name="targeting-specific-registrations"></a>Выбор регистраций

Единственный способ выбрать определенные регистрации для получения уведомлений — это связать с ними теги, а затем указывать эти теги в качестве назначения. Как описано в разделе [Управление регистрацией](notification-hubs-push-notification-registration-management.md)для получения push-уведомлений, приложение должно зарегистрировать маркер устройства в центре уведомлений. Когда приложение создает регистрацию в центре уведомлений, серверная часть приложения может отправлять в нее push-уведомления. Серверная часть приложения выбирает целевые регистрации для отправки определенных уведомлений следующим образом.

1. **Широковещательная рассылка**: уведомление получают все регистрации в центре уведомлений.
2. **По тегу**: уведомление получают все регистрации, которые содержат указанный тег.
3. **По выражению тега**: уведомление получают все регистрации, набор тегов которых соответствует указанному выражению.

## <a name="tags"></a>Теги

Тег может быть любой строкой длиной до 120 символов, содержащей буквенно-цифровые и следующие символы: " `_` ", " `@` ", "", "", "", "" `#` `.` `:` `-` . В следующем примере показано приложение, из которого можно получать всплывающие уведомления об определенных музыкальных группах. В этом сценарии простой способ маршрутизации уведомлений — регистрация регистраций с тегами, представляющими различные зоны, как показано на следующем рисунке.

![Общие сведения о тегах](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags.png)

На рисунке сообщение с тегом **Beatles** достигает только планшета, зарегистрированного в теге **Beatles**.

Дополнительные сведения о создании регистраций по тегам см. в статье [Управление регистрацией](notification-hubs-push-notification-registration-management.md).

Можно отправлять уведомления для тегов, используя методы отправки уведомлений класса `Microsoft.Azure.NotificationHubs.NotificationHubClient` из пакета SDK [центров уведомлений Microsoft Azure](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/) . Также можно использовать Node.js или интерфейсы API REST push-уведомлений.  Ниже приведен пример с использованием пакета SDK.

```csharp
Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;

// Windows 8.1 / Windows Phone 8.1
var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
"You requested a Beatles notification</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, "Beatles");

// Windows 10
toast = @"<toast><visual><binding template=""ToastGeneric""><text id=""1"">" +
"You requested a Wailers notification</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, "Wailers");
```

Теги не должны быть предварительно подготовлены и могут ссылаться на несколько концепций конкретного приложения. Например, пользователи приложения из этого примера могут оставлять свои комментарии по группам и хотят получать всплывающие уведомления не только о комментариях по своим любимым группам, но и обо всех комментариях от своих друзей, независимо от того, какую группу они комментируют. На следующем рисунке показан пример этого сценария.

![Теги друзья](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags2.png)

В этом примере Алиса заинтересована в обновлении Beatles, а Боб заинтересован в обновлении Ваилерс. Боб также заинтересован в комментариях Чарли, и Чарли заинтересован в Ваилерс. При отправке уведомления для комментария Чарли к Beatles концентраторы уведомлений отправляют его Алисе и Бобу.

Хотя можно закодировать несколько проблем в тегах (например, `band_Beatles` или `follows_Charlie` ), теги являются простыми строками, а не свойствами со значениями. Регистрация совпадает только с присутствием или отсутствием определенного тега.

Полное пошаговое руководство по использованию тегов для отправки сообщений по группам интересов см. в статье [Использование Центров уведомлений для передачи экстренных новостей](notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md).

> [!NOTE]
> Центры уведомлений Azure поддерживают не более 60 тегов для каждой регистрации.

## <a name="using-tags-to-target-users"></a>Использование тегов для выбора конечных пользователей

Другим способом использования тегов является обнаружение всех устройств, связанных с определенным пользователем. Вы можете пометить регистрацию тегом, который содержит идентификатор пользователя, как показано на следующем рисунке:

![Теги пользователей](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags3.png)

На рисунке показано, что сообщение помечается на `user_Alice` все устройства, помеченные как `user_Alice` .

## <a name="tag-expressions"></a>Выражения тегов

Бывают случаи, когда уведомления должны ориентироваться на набор регистраций, идентифицируемый не одним тегом, а с помощью логического выражения, использующего Теги.

Рассмотрим спортивное приложение, которое отправляет напоминание всем пользователям в Бостоне об игре между командами Red Sox и Cardinals. Если в клиентском приложении регистрируются теги об интересе к командам и месту, то уведомление должно отправляться всем пользователям в Бостоне, интересующимся Red Sox или Cardinals. Это условие можно задать с помощью следующего логического выражения:

```csharp
(follows_RedSox || follows_Cardinals) && location_Boston
```

![Выражения тегов](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags4.png)

Выражения тегов поддерживают общие логические операторы, такие как `AND` ( `&&` ), `OR` ( `||` ) и `NOT` ( `!` ); они также могут содержать круглые скобки. Выражения тегов, использующие только `OR` операторы, могут ссылаться на 20 тегов; выражение с `AND` операторами, но ни один `OR` оператор не может ссылаться на 10 тегов; в противном случае выражения тегов ограничены 6 тегами.

Ниже приведен пример отправки уведомлений с использованием выражений тегов с помощью пакета SDK:

```csharp
Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;

String userTag = "(location_Boston && !follows_Cardinals)";

// Windows 8.1 / Windows Phone 8.1
var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
"You want info on the Red Sox</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);

// Windows 10
toast = @"<toast><visual><binding template=""ToastGeneric""><text id=""1"">" +
"You want info on the Red Sox</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
```
