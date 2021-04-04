---
title: Управление потенциальными клиентами в Marketo — коммерческой платформе Майкрософт
description: Узнайте, как настроить систему CRM для управления потенциальными клиентами из Microsoft AppSource и Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 03/30/2020
ms.openlocfilehash: cbd12481312d310add15bf3d41e21e9c416f1c39
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94491081"
---
# <a name="use-marketo-to-manage-commercial-marketplace-leads"></a>Использование Marketo для управления потенциальными клиентами на коммерческой платформе

В этой статье описывается, как настроить систему CRM Marketo для обработки потенциальных клиентов своих предложений в Microsoft AppSource и Azure Marketplace.

## <a name="set-up-your-marketo-crm-system"></a>Настройка системы CRM Marketo

1. Войдите в Marketo.

1. Выберите **Разработка студии**.

    ![Разработка студии Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-1.png)

1.  Выберите **Новая форма**.

    ![Новая форма Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-2.png)

1.  Заполните необходимые поля в диалоговом окне **Новая форма**, а затем выберите **Создать**.

    ![Создание новой формы Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-3.png)

1.  На странице **Сведения о поле** выберите **Готово**.

    ![Завершение формы Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-4.png)

1.  Подтвердите и закройте окно.

1. На вкладке **MarketplaceLeadBacked** выберите **Внедрить код**. 

    ![Код внедрения Marketo](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-6.png)

1. Код внедрения Marketo отображает код, аналогичный приведенному ниже.

    ```html
    <form id="mktoForm_1179"></form>
    <script>MktoForms2.loadForm("("//app-ys12.marketo.com", "123-PQR-789", 1179);</script>
    ```

1. Скопируйте значения для следующих полей, показанных в форме кода внедрения. Эти значения используются для настройки получения потенциальных клиентов в предложении на следующем этапе. Используйте следующий пример в качестве руководства для получения необходимых идентификаторов из примера кода внедрения Marketo.

    - Идентификатор сервера = **ys12**
    - Идентификатор Munchkin = **123-PQR-789**
    - Идентификатор формы = **1179**

    Еще один способ определить эти значения:

    - Идентификатор сервера указан в URL-адресе экземпляра Marketo, например `serverID.marketo.com`.
    - Получите идентификатор Munchkin для своей подписки в меню **Администратор** > **Munchkin** в поле **Идентификатор учетной записи Munchkin** или в первой части дочернего домена узла Marketo REST API: `https://{Munchkin ID}.mktorest.com`.
    - Идентификатор формы — это идентификатор формы кода внедрения, созданной на шаге 7 для отправки потенциальных клиентов из Marketplace.

## <a name="configure-your-offer-to-send-leads-to-marketo"></a>Настройка предложения для отправки потенциальных клиентов в Marketo

После того как все будет готово к настройке сведений об управлении потенциальными клиентами для вашего предложения на портале публикации, выполните следующие действия. 

1. Войдите в [Центр партнеров](https://partner.microsoft.com/dashboard/home).

1. Выберите свое предложение и перейдите на вкладку **Настройка предложения**.

1. В разделе **Потенциальные клиенты** выберите **Подключить**.

    :::image type="content" source="./media/commercial-marketplace-lead-management-instructions-marketo/customer-leads.png" alt-text="Потенциальные клиенты":::

1. Во всплывающем окне **Сведения о подключении** в качестве **назначения потенциального клиента** выберите **Marketo**.

    ![Выберите назначение потенциального клиента](./media/commercial-marketplace-lead-management-instructions-marketo/choose-lead-destination.png)

1. Укажите **идентификатор сервера**, **идентификатор учетной записи Munchkin** и **идентификатор формы**.

    > [!NOTE]
    > Прежде чем вы сможете получать сведения о потенциальных клиентах для своего предложения необходимо завершить настройку его остальной части и опубликовать его. 

1. В разделе **Контактный адрес электронной почты** введите адреса электронной почты сотрудников организации, которые должны получать уведомления по электронной почте при обнаружении нового потенциального клиента. Можно указать несколько адресов электронной почты, разделяя их точкой с запятой.

1. Щелкните **ОК**.

   Чтобы убедиться, что вы успешно подключились к месту назначения потенциальных клиентов, выберите **Проверить**. При успешном подключении вы получите тестового потенциального клиента в месте назначения потенциальных клиентов.

   ![Всплывающее окно сведений о подключении](./media/commercial-marketplace-lead-management-instructions-marketo/marketo-connection-details.png)
