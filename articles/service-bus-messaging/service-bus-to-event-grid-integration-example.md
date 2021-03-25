---
title: Обработка событий службы "Служебная шина" с помощью службы "Сетка событий" с использованием Azure Logic Apps
description: В этой статье содержатся инструкции по обработке событий служебной шины с помощью службы "Сетка событий" и Azure Logic Apps.
documentationcenter: .net
author: spelluru
ms.topic: tutorial
ms.date: 10/16/2020
ms.author: spelluru
ms.custom: devx-track-csharp
ms.openlocfilehash: 93375f6047fbe4eda2132e024dab0e067e83ccf1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95999069"
---
# <a name="tutorial-respond-to-azure-service-bus-events-received-via-azure-event-grid-by-using-azure-logic-apps"></a>Руководство по реагированию на события Служебной шины Azure, получаемые через Сетку событий Azure с помощью Azure Logic Apps
Узнайте, как отвечать на события служебной шины Azure, получаемые через Сетку событий Azure с помощью Azure Logic Apps. 

[!INCLUDE [service-bus-event-grid-prerequisites](../../includes/service-bus-event-grid-prerequisites.md)]

## <a name="receive-messages-by-using-logic-apps"></a>Получение сообщений с помощью Logic Apps
На этом этапе вы создадите приложение логики Azure, которое получает события служебной шины через службу "Сетка событий Azure". 

1. Создайте приложение логики на портале Azure.
    1. Щелкните **+ Создать ресурс**, выберите **Интеграция**, а затем — **Logic App**. 
    2. На странице **Приложение логики — создание** укажите **имя** приложения логики.
    3. Выберите свою **подписку Azure**. 
    4. Щелкните **Использовать существующую** для **группы ресурсов** и выберите группу ресурсов, которая использовалась для других ресурсов (например, Функции Azure или пространство имен служебной шины), созданных ранее. 
    5. Укажите **расположение** приложения логики. 
    6. Выберите **Review + Create** (Просмотреть и создать). 
    1. На странице **Просмотр и создание** выберите **Создать**, чтобы создать приложение логики. 
1. На странице **Конструктор Logic Apps** щелкните **Пустое приложение логики** в разделе **Шаблоны**. 
1. В конструкторе сделайте следующее:
    1. Введите **Сетка событий** для поиска. 
    2. Выберите **При возникновении события ресурса — Сетка событий Azure**. 

        ![Конструктор Logic Apps — выбор триггера Сетки событий](./media/service-bus-to-event-grid-integration-example/logic-apps-event-grid-trigger.png)
4. Щелкните **Войти**, введите свои учетные данные Azure и выберите **Разрешить доступ**. 
5. На странице **При возникновении события ресурса** сделайте следующее:
    1. Выберите подписку Azure. 
    2. Для параметра **Тип ресурса** выберите значение **Microsoft.ServiceBus.Namespaces**. 
    3. Для параметра **Имя ресурса** выберите пространство имен служебной шины. 
    4. Щелкните **Добавить параметр** и выберите **Фильтр суффиксов**. 
    5. В разделе **Фильтр суффиксов** укажите имя вашей второй подписки раздела служебной шины. 
        ![Конструктор Logic Apps — настройка события](./media/service-bus-to-event-grid-integration-example/logic-app-configure-event.png)
6. В конструкторе щелкните **+ Новый шаг** и сделайте следующее:
    1. Введите **Служебная шина** для поиска.
    2. Выберите **Служебная шина** в списке. 
    3. Выберите **Получить сообщения** в списке **Действия**. 
    4. Выберите **Получение сообщений из подписки раздела (с блокировкой при извлечении)** . 

        ![Конструктор Logic Apps — действие получения сообщений](./media/service-bus-to-event-grid-integration-example/service-bus-get-messages-step.png)
    5. Укажите **имя подключения**. Пример: **Получение сообщений из подписки раздела**. Затем выберите пространство имен служебной шины. 

        ![Конструктор Logic Apps — выбор пространства имен служебной шины](./media/service-bus-to-event-grid-integration-example/logic-apps-select-namespace.png) 
    6. Выберите **RootManageSharedAccessKey** и щелкните **Создать**.

        ![Конструктор Logic Apps — выбор общего ключа доступа](./media/service-bus-to-event-grid-integration-example/logic-app-shared-access-key.png) 
    8. Выберите **раздел** и **подписку**. 
    
        ![Снимок экрана, на котором показано, где выбрать раздел и подписку.](./media/service-bus-to-event-grid-integration-example/logic-app-select-topic-subscription.png)
7. Щелкните **+ Новый шаг** и сделайте следующее: 
    1. Выберите **служебную шину**.
    2. Выберите **Завершить сообщение в подписке раздела** в списке действий. 
    3. Выберите **раздел** служебной шины.
    4. Выберите вторую **подписку** на раздел.
    5. Для параметра **Маркер блокировки сообщения** выберите значение **Маркер блокировки** в списке **Динамическое содержимое**. 

        ![Экран "Конструктор Logic Apps — завершение сообщения"](./media/service-bus-to-event-grid-integration-example/logic-app-complete-message.png)
8. Выберите **Сохранить** на панели инструментов в конструкторе Logic Apps, чтобы сохранить приложение логики. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/save-logic-app.png" alt-text="Экран &quot;Сохранение приложения логики&quot;":::
1. Если вы еще не отправили тестовые сообщения в раздел, выполните инструкции по [отправке сообщений в раздел служебной шины](#send-messages-to-the-service-bus-topic). 
1. Перейдите на страницу **Обзор** приложения-логики. Выполнение приложения логики можно отследить в **журнале выполнения** для отправленных сообщений. Запуск приложения логики может занять несколько минут. Щелкните **Обновить** на панели инструментов, чтобы обновить страницу. 

    ![Конструктор Logic Apps — выполнение приложения логики](./media/service-bus-to-event-grid-integration-example/logic-app-runs.png)
1. Выберите "Запуск приложения логики", чтобы просмотреть подробные сведения. Обратите внимание, что приложение обработало 5 сообщений в цикле. 
    
    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/logic-app-run-details.png" alt-text="Экран &quot;Сведения о выполнении приложения логики&quot;":::    

## <a name="troubleshoot"></a>Диагностика
Если после ожидания и обновления вызовы не отображаются, выполните следующие действия: 

1. Убедитесь, что сообщения отправлены в раздел Служебной шины. Взгляните на счетчик **входящих сообщений** на странице **Раздел служебной шины**. Так как в нашем примере приложение **MessageSender** запущено дважды, отображаются 10 сообщений (по 5 для каждого запуска).

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/topic-incoming-messages.png" alt-text="Страница &quot;Раздел служебной шины&quot; — входящие сообщения":::    
1. Убедитесь, что на странице "Подписка служебной шины" нет **активных сообщений**. 
    Если на этой странице нет событий, убедитесь, что на странице **Service Bus Subscription** (Подписка служебной шины) не отображено **число активных сообщений**. Если там отображается число выше нуля, это значит, что сообщения в подписке по каким-то причинам не пересылаются в функцию обработчика (обработчик подписки на события). Убедитесь, что вы правильно настроили подписку на события. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/subscription-active-message-count.png" alt-text="Число активных сообщений в подписке Служебной шины":::    
1. Кроме того, на странице **События** в пространстве имен Cлужебной шины можно просмотреть **доставленные события**. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/event-subscription-page.png" alt-text="Страница событий — доставленные события" lightbox="./media/service-bus-to-event-grid-integration-example/invocation-details.png":::
1. Вы также можете заметить, что события доставляются на страницу **Подписка на событие**. Чтобы перейти на эту страницу, выберите подписку на события на странице **События**. 
    
    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/event-subscription-delivered-events.png" alt-text="Страница подписки на события — доставленные события":::
## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о службе "Сетка событий" Azure см. [здесь](../event-grid/index.yml).
* Дополнительные сведения о решении "Функции Azure" см. [здесь](../azure-functions/index.yml).
* Дополнительные сведения о компоненте Logic Apps службы приложений Azure см. [здесь](../logic-apps/index.yml).
* Дополнительные сведения о [служебной шине Azure](/azure/service-bus/).


[2]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid2.png
[3]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid3.png
[7]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid7.png
[8]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid8.png
[9]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid9.png
[10]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid10.png
[11]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid11.png
[12]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12.png
[12-1]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12-1.png
[12-2]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid12-2.png
[13]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid13.png
[14]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid14.png
[15]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid15.png
[16]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid16.png
[17]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid17.png
[18]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgrid18.png
[20]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal.png
[21]: ./media/service-bus-to-event-grid-integration-example/sbtoeventgridportal2.png