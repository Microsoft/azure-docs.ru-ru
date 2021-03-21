---
title: Проверка подлинности на основе маркеров (HTTP/2) для APNS в центрах уведомлений Azure | Документация Майкрософт
description: Узнайте, как использовать новую проверку подлинности маркеров для APNS.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 11/14/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 02/13/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 4f8de6389a04448579672b84e91f0bb4dd0f4ce2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96460469"
---
# <a name="token-based-http2-authentication-for-apns"></a>Проверка подлинности на основе маркеров (HTTP/2) для APNS

## <a name="overview"></a>Обзор

В этой статье объясняется, как использовать новый протокол HTTP/2 APNS с проверкой подлинности на основе маркеров.

Основные преимущества использования нового протокола:

* Создание маркеров относительно просто (по сравнению с сертификатами)
* У токенов отсутствует срок окончания действия. Вы управляете их созданием и отзывом.
* Полезные данные могут иметь размер до 4 КБ.
* Синхронная обратная связь.
* Вы используете последний протокол Apple — сертификаты по-прежнему используют двоичный протокол, отмеченный для устаревания.

Использование этого нового механизма может быть выполнено в два этапа:

* Получите необходимые сведения на портале учетной записи разработчика Apple.
* Настройте центр уведомлений, указав новые сведения.

Центры уведомлений теперь настроены для использования новой системы проверки подлинности с APNS.

Обратите внимание, что если вы выполнили миграцию с использованием учетных данных сертификата для APNS, свойства маркера перезапишут ваш сертификат в нашей системе, но приложение продолжит получать уведомления без проблем.

## <a name="obtaining-authentication-information-from-apple"></a>Получение сведений о проверке подлинности от Apple

Чтобы включить проверку подлинности на основе маркеров, вам потребуются следующие свойства учетной записи разработчика Apple:

### <a name="key-identifier"></a>Идентификатор ключа

Идентификатор ключа можно получить на странице **ключи** в разделе **сертификаты, идентификаторы & профили** в учетной записи разработчика Apple:

![Сертификаты](./media/notification-hubs-push-notification-http2-token-authentification/keys.png)

![Идентификаторы](./media/notification-hubs-push-notification-http2-token-authentification/obtaining-auth-information-from-apple.png)

### <a name="application-identifier-and-application-name"></a>Идентификатор приложения и имя приложения

Имя и идентификатор приложения также доступны на странице **сертификаты, идентификаторы & профилей** в учетной записи разработчика:

![Сертификаты и идентификаторы](./media/notification-hubs-push-notification-http2-token-authentification/app-name.png)

### <a name="configure-via-the-net-sdk-or-the-azure-portal"></a>Настройка с помощью пакета SDK для .NET или портал Azure

Вы можете настроить центр для использования проверки подлинности на основе маркеров с помощью нашего [новейшего клиентского пакета SDK](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs)или в портал Azure. Чтобы включить проверку подлинности на основе маркеров на портале, войдите в портал Azure и перейдите на панель Параметры центра уведомлений **> Apple (APNs)** . Выберите **токен** в свойстве **режим проверки подлинности** , чтобы обновить центр всеми соответствующими свойствами маркера.

![Настройка токена](./media/notification-hubs-push-notification-http2-token-authentification/azure-portal-apns-settings.png)

* Введите свойства, полученные из учетной записи разработчика Apple.
* Выберите режим приложения (**Рабочая** или **Песочница**).
* Нажмите кнопку **сохранить** , чтобы обновить учетные данные APNs.

Учетные данные на основе токенов состоят из следующих полей:

* **Идентификатор ключа**: идентификатор закрытого ключа, созданного на портале разработчика Apple. Например, `2USFGKSKLT` .
* **Идентификатор команды**: также называется префиксом или префиксом приложения. Это идентификатор организации на портале разработчика Apple. Например, `S4V3D7CHJR` .
* **Идентификатор пакета**: также называется идентификатором приложения. Это идентификатор пакета для приложения; Например, `com.example.myapp` . Обратите внимание, что для одного приложения можно использовать только один ключ. Это значение сопоставляется с `apns-topic` заголовком HTTP при отправке уведомления и используется для назначения конкретного приложения. Нельзя явно задать значение `apns-topic` .
* **Token**: также называется "Key" или "закрытый ключ". Это значение берется из файла с расширением. P8, созданного на портале разработчика Apple. Для ключа должно быть включено значение APNS (которое выбирается на портале разработчика Apple при создании ключа). При указании этого значения на портале или API-интерфейсе NH значение должно быть удалено из верхнего или нижнего колонтитула PEM.
* **Конечная точка**: это переключатель в колонке портала концентраторов уведомлений и строковое поле в API. Допустимые значения: `https://api.development.push.apple.com:443/3/device` или `https://api.sandbox.push.apple.com:443/3/device`. Центры уведомлений используют это значение для среды рабочего или песочницы для отправки уведомлений. Это должно совпадать с назначением `aps-environment` в приложении, в противном случае созданные маркеры устройств APNs не соответствуют среде, и уведомления не будут отправлены.

Ниже приведен пример кода, иллюстрирующий правильное использование:

```csharp
NamespaceManager nm = NamespaceManager.CreateFromConnectionString(_endpoint);
string token = "YOUR PRIVATE KEY HERE";
string keyId = "YOUR KEY ID HERE";
string appName = "YOUR APP NAME HERE";
string appId = "YOUR APP ID HERE";
NotificationHubDescription desc = new NotificationHubDescription("PATH TO YOUR HUB");
desc.ApnsCredential = new ApnsCredential(token, keyId, appId, appName);
desc.ApnsCredential.Endpoint = @"https://api.development.push.apple.com:443/3/device";
nm.UpdateNotificationHubAsync(desc);
```

## <a name="next-steps"></a>Дальнейшие действия

* [Создание центра уведомлений Azure с помощью портала Azure](create-notification-hub-portal.md)
* [Настройка концентратора уведомлений в портал Azure](create-notification-hub-portal.md)
