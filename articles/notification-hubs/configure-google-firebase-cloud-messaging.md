---
title: Настройка облачных сообщений Google Firebase в центрах уведомлений Azure | Документация Майкрософт
description: Узнайте, как настроить центр уведомлений Azure с помощью Google Firebase Cloud Messaging Settings.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.topic: article
ms.date: 08/04/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: 0da103c11e2412108535ca322917632f5d95559d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96003571"
---
# <a name="configure-google-firebase-settings-for-a-notification-hub-in-the-azure-portal"></a>Настройка параметров Google Firebase для центра уведомлений в портал Azure

В этой статье показано, как настроить параметры Google Firebase Cloud Messaging (FCM) для центра уведомлений Azure с помощью портал Azure.  

## <a name="prerequisites"></a>Предварительные условия

Если вы еще не создали центр уведомлений, сделайте это сейчас. Дополнительные сведения см. в статье [Создание центра уведомлений Azure с помощью портала Azure](create-notification-hub-portal.md).

## <a name="configure-google-firebase-cloud-messaging-fcm"></a>Настройка Google Firebase Cloud Messaging (FCM)

В следующей процедуре описана процедура настройки параметров Google Firebase Cloud Messaging (FCM) для центра уведомлений.

1. В портал Azure на странице **Центр уведомлений** выберите **Google (gcm/FCM)** в меню слева.
2. В соответствующее поле вставьте **ключ API** для проекта FCM, сохраненного ранее.
3. Щелкните **Сохранить**.

   ![Снимок экрана, на котором показано, как настроить Центры уведомлений для Google FCM](./media/notification-hubs-android-push-notification-google-fcm-get-started/fcm-server-key.png)

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по отправке уведомлений на устройства Android с помощью концентраторов уведомлений Azure и Google Firebase Cloud Messaging см. в статье [Отправка push-уведомлений на устройства Android с помощью концентраторов уведомлений и Google FCM](notification-hubs-android-push-notification-google-fcm-get-started.md).
