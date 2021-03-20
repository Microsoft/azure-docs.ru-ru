---
title: Настройка Cлужба push-уведомлений Apple в центрах уведомлений Azure | Документация Майкрософт
description: Узнайте, как настроить центр уведомлений Azure с помощью параметров Cлужба push-уведомлений Apple (APNS).
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.topic: article
ms.date: 06/22/2020
ms.author: sethm
ms.reviewer: thsomasu
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: 63c7e0c9569428b55420911f253deee52ce440cb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85255404"
---
# <a name="configure-apple-push-notification-service-settings-for-a-notification-hub-in-the-azure-portal"></a>Настройка параметров Cлужба push-уведомлений Apple для центра уведомлений в портал Azure

В этой статье показано, как настроить параметры Cлужба push-уведомлений Apple (APNS) для центра уведомлений Azure с помощью портал Azure.

## <a name="prerequisites"></a>Предварительные условия

Если вы еще не создали центр уведомлений, сделайте это сейчас. Дополнительные сведения см. в статье [Создание центра уведомлений Azure с помощью портала Azure](create-notification-hub-portal.md).

## <a name="configure-apple-push-notification-service"></a>Настройка Cлужба push-уведомлений Apple

В следующей процедуре приведены действия по настройке параметров Cлужба push-уведомлений Apple (APNS) для центра уведомлений.

1. В портал Azure на странице **Центр уведомлений** выберите **Apple (APNs)** в меню слева.

1. В разделе **Режим проверки подлинности** выберите **Сертификат** или **Токен**.

   - Если вы выбрали **Сертификат**:
      - Щелкните значок файла и выберите файл *.p12*, который нужно отправить.
      - Введите пароль.
      - Выберите режим **Песочница**. Если нужно отправлять push-уведомления пользователям, которые приобрели приложение в магазине, выберите режим **Рабочий**.

     ![Снимок экрана настройки сертификата APNS на портале Azure](./media/configure-apple-push-notification-service/notification-hubs-apple-config-cert.png)

   - Если вы выбрали **Токен**:
      - Введите значения для параметров **Идентификатор ключа**, **Идентификатор набора**, **Идентификатор команды** и **Токен**.
      - Выберите режим **Песочница**. Если нужно отправлять push-уведомления пользователям, которые приобрели приложение в магазине, выберите режим **Рабочий**.

     ![Снимок экрана настройки токена APNS на портале Azure](./media/configure-apple-push-notification-service/notification-hubs-apple-config-token.png)

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по отправке уведомлений на устройства iOS см. в следующей статье: [Отправка push-уведомлений в приложения iOS с помощью концентраторов уведомлений Azure](ios-sdk-get-started.md).
