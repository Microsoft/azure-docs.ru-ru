---
title: включить файл
description: включить файл
services: notification-hubs
author: jwargo
ms.service: notification-hubs
ms.topic: include
ms.date: 01/17/2019
ms.author: jowargo
ms.custom: include file
ms.openlocfilehash: 2ec602f056b339a1b1dcb78d6b8d7583aeaf0434
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96009125"
---
1. Войдите на [портал Azure](https://portal.azure.com).

1. Выберите **Все службы** в меню слева и щелкните **Центры уведомлений** в разделе **Мобильный**. Щелкните значок звездочки рядом с именем службы, чтобы добавить ее в раздел **Избранное** в меню слева. После добавления **Центров уведомлений** в **Избранное** выберите их в меню слева.

      ![Портал Azure. Выбор Центров уведомлений](./media/notification-hubs-portal-create-new-hub/all-services-select-notification-hubs.png)

1. На странице **Центры уведомлений** выберите **Добавить** на панели инструментов.

      ![Центры уведомлений. Добавление кнопки панели инструментов](./media/notification-hubs-portal-create-new-hub/add-toolbar-button.png)

1. На странице **Концентратор уведомлений** выполните следующие действия:

    1. Введите имя в поле **Центр уведомлений**.  

    1. Введите имя в поле **Создать пространство имен**. Пространство имен содержит один или несколько центров.

    1. Выберите значение в раскрывающемся списке **Расположение**. Это значение определяет расположение, в котором создается центр.

    1. В поле **Группа ресурсов** выберите существующую группу ресурсов или укажите имя для новой группы ресурсов.

    1. Нажмите кнопку **Создать**.

        ![Портал Azure: настройка свойств концентратора уведомлений](./media/notification-hubs-portal-create-new-hub/notification-hubs-azure-portal-settings.png)

1. Последовательно выберите **Уведомления** (значок колокольчика) и **Перейти к ресурсу**. Вы также можете обновить список на странице **Центры уведомлений** и выбрать свой центр.

      ![Портал Azure: кнопка "Перейти к ресурсу"](./media/notification-hubs-portal-create-new-hub/go-to-notification-hub.png)

1. Выберите **Политики доступа** в списке. Обратите внимание, что станут доступны две строки подключения. Они потребуются позже для обработки push-уведомлений.

      >[!IMPORTANT]
      >*Не* используйте в приложении политику **DefaultFullSharedAccessSignature**. Этот параметр можно использовать только в серверной части.
      >

      ![Портал Azure: строки подключения к концентратору уведомлений](./media/notification-hubs-portal-create-new-hub/notification-hubs-connection-strings-portal.png)
