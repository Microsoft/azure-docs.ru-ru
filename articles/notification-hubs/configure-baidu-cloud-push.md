---
title: Настройка Baidu Cloud Push в центрах уведомлений Azure | Документация Майкрософт
description: Узнайте, как настроить параметры Baidu для центра уведомлений Azure.
services: notification-hubs
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.topic: article
ms.date: 03/25/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 03/25/2019
ms.openlocfilehash: 759e35ba353f470ea3abc5f5d4182fa2b2ea0e73
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96003588"
---
# <a name="deprecated-configure-baidu-cloud-push-settings-for-a-notification-hub-in-the-azure-portal"></a>Не рекомендуется: Настройте параметры push-уведомлений Baidu в облаке в портал Azure

В этой статье показано, как настроить параметры push-уведомлений Baidu в облаке Azure с помощью портал Azure.

> [!IMPORTANT]
> Этот учебник не рекомендуется к использованию. 

## <a name="prerequisites"></a>Предварительные условия
Если вы еще не создали центр уведомлений, сделайте это сейчас. Дополнительные сведения см. в статье [Создание центра уведомлений Azure с помощью портала Azure](create-notification-hub-portal.md). 

## <a name="configure-baidu-cloud-push"></a>Настройка push-уведомлений облака Baidu
Следующая процедура позволяет настроить параметры Baidu Cloud Push для центра уведомлений.

1. В портал Azure на странице **Центр уведомлений** выберите **Baidu (Android China)** в меню слева. 
2. В соответствующее поле введите **ключ API**, полученный из консоли Baidu, в проекте службы push-уведомлений облака Baidu. 
3. Введите **секретный ключ**, полученный из консоли Baidu в проекте службы push-уведомлений облака Baidu. 
4. Щелкните **Сохранить**. 

    ![Снимок экрана Центров уведомлений, на котором показана настройка push-уведомлений в Baidu (Android China)](./media/notification-hubs-baidu-get-started/AzureNotificationServicesBaidu.png)

## <a name="next-steps"></a>Дальнейшие действия
Пошаговые инструкции по отправке уведомлений в Baidu с помощью центров уведомлений Azure и Baidu Cloud Push см. в статье [Приступая к работе с центрами уведомлений с помощью Baidu](notification-hubs-baidu-china-android-notifications-get-started.md).
