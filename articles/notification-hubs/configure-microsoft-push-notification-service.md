---
title: Настройка службы push-уведомлений Майкрософт в центрах уведомлений Azure | Документация Майкрософт
description: Узнайте, как настроить параметры службы push-уведомлений Майкрософт для центра уведомлений Azure.
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
ms.openlocfilehash: 6784331dccd779390ace7dc632d54fe54aedc458
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87760997"
---
# <a name="configure-microsoft-push-notification-service-mpns-settings-in-the-azure-portal"></a>Настройка параметров службы push-уведомлений Майкрософт (MPNS) в портал Azure

В этой статье показано, как настроить параметры службы push-уведомлений Майкрософт (MPNS) для центра уведомлений Azure с помощью портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Если вы еще не создали центр уведомлений, сделайте это сейчас. Дополнительные сведения см. в статье [Создание центра уведомлений Azure с помощью портала Azure](create-notification-hub-portal.md).

## <a name="configure-microsoft-push-notification-service-mpns"></a>Настройка службы push-уведомлений Майкрософт (MPNS)

В следующей процедуре описывается настройка параметров службы push-уведомлений Майкрософт (MPNS) для центра уведомлений.

1. В портал Azure на странице **Центр уведомлений** выберите **Windows Phone (MPNS)** в меню слева.
2. Включите push-уведомления с проверкой подлинности или без нее:

   а. Чтобы включить push-уведомления без проверки подлинности, выберите **Включить push-уведомления без проверки подлинности** > **Сохранить**.

      ![Снимок экрана, на котором показано, как включить push-уведомления без проверки подлинности](./media/notification-hubs-windows-phone-get-started/azure-portal-unauth.png)

   b. Чтобы включить push-уведомления с проверкой подлинности, сделайте следующее:
      * На панели инструментов щелкните **Отправка сертификата**.
      * Щелкните значок файла, а затем выберите файл сертификата.
      * Введите пароль для сертификата.
      * Щелкните **ОК**.
      * На странице **Windows Phone(MPNS)** выберите **Сохранить**.

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по отправке уведомлений на Windows Phone устройства с помощью центров уведомлений Azure и службы push-уведомлений Майкрософт (MPNS) см. в разделе [Отправка push-уведомлений для Windows Phone приложений с помощью концентраторов уведомлений](notification-hubs-windows-mobile-push-notifications-mpns.md).
