---
title: Настройка службы push-уведомлений Windows в центрах уведомлений Azure | Документация Майкрософт
description: Узнайте, как настроить параметры службы push-уведомлений Windows для центра уведомлений Azure.
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
ms.openlocfilehash: 62d73a23fff3bd15d873a579d451b2f91509b7fe
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87758736"
---
# <a name="configure-windows-push-notification-service-settings-in-the-azure-portal"></a>Настройка параметров службы push-уведомлений Windows в портал Azure

В этой статье показано, как настроить параметры службы уведомлений Windows (WNS) для центра уведомлений Azure с помощью портал Azure.  

## <a name="prerequisites"></a>Предварительные требования

Если вы еще не создали центр уведомлений, сделайте это сейчас. Дополнительные сведения см. в статье [Создание центра уведомлений Azure с помощью портала Azure](create-notification-hub-portal.md).

## <a name="configure-windows-push-notification-service-wns"></a>Настройка службы push-уведомлений Windows (WNS)

В следующей процедуре описаны действия по настройке параметров службы push-уведомлений Windows (WNS) для центра уведомлений.

1. В портал Azure на странице **Центр уведомлений** выберите **Windows (WNS)** в меню слева.
2. Введите значения для **идентификатора безопасности пакета** и **ключа безопасности**.
3. Щелкните **Сохранить**.

   ![Снимок экрана, на котором показаны поля "ИД безопасности пакета" и "Ключ безопасности"](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-configure-wns.png)

## <a name="next-steps"></a>Дальнейшие действия

Пошаговые инструкции по отправке push-уведомлений для универсальная платформа Windows приложений с помощью центров уведомлений Azure и службы push-уведомлений Windows (WNS) см. в статье [Отправка уведомлений в приложения UWP с помощью концентраторов уведомлений Azure](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).
