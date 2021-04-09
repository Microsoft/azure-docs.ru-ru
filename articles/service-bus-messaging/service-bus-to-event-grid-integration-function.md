---
title: Обработка событий Служебной шины с помощью службы "Сетка событий" и Функций Azure
description: В этой статье показано, как обрабатывать события Служебной шины с помощью службы "Сетка событий", используя Функции Azure.
documentationcenter: .net
author: spelluru
ms.topic: tutorial
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-csharp
ms.openlocfilehash: afc0a5bf9b83363d1f4baab955b55148fe3a8498
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "95818484"
---
# <a name="tutorial-respond-to-azure-service-bus-events-received-via-azure-event-grid-by-using-azure-functions"></a>Руководство. Реагирование на события Служебной шины Azure, получаемые через службу "Сетка событий Azure", с помощью Функций Azure
Узнайте, как отвечать на события служебной шины Azure, получаемые через Сетку событий Azure с помощью Функций Azure и Azure Logic Apps. 

В этом руководстве описано следующее:
> [!div class="checklist"]
> * Создание пространства имен служебной шины
> * Подготовка примера приложения для отправки сообщений
> * Отправка сообщений в раздел служебной шины
> * Получение сообщений с помощью Logic Apps
> * Настройка проверочной функции в Azure
> * Подключение функции и пространства имен с помощью службы "Сетка событий"
> * Получение сообщений с помощью службы "Функции Azure"

[!INCLUDE [service-bus-event-grid-prerequisites](../../includes/service-bus-event-grid-prerequisites.md)]

## <a name="additional-prerequisites"></a>Дополнительные требования
[Установите Visual Studio 2019](https://www.visualstudio.com/vs) с рабочей нагрузкой **Разработка для Azure**. Эта рабочая нагрузка включает **инструменты Функций Azure**, требуемые для создания, сборки и развертывания проектов Функций Azure в Visual Studio. 

## <a name="deploy-the-function-app"></a>Развертывание приложения-функции 

>[!NOTE]
> Дополнительные сведения см. в статье [Разработка Функций Azure с помощью Visual Studio](../azure-functions/functions-develop-vs.md).

1. Откройте файл **ReceiveMessagesOnEvent.cs** в проекте **FunctionApp1** решения **SBEventGridIntegration.sln**. 
1. Замените `<SERCICE BUS NAMESPACE - CONNECTION STRING>` строкой подключения к пространству имен Служебной шины. Эта строка должна совпадать с той, которая использовалась в файле **Program.cs** проекта **MessageSender** в том же решении. 
1. Щелкните правой кнопкой мыши **FunctionApp1** и выберите **Опубликовать**. 
1. На странице **Публикация** выберите **Запустить**. Хотя описанные действия могут отличаться, процесс публикации должен быть похожим. 
1. В мастере **публикации** на странице **Целевой объект** выберите **Azure** для параметра **Целевой объект**. 
1. На странице **Указанный целевой объект** выберите **Приложение-функция Azure (Windows)** . 
1. На странице **Экземпляр функций** выберите **Создать новую функцию Azure**. 
1. На странице **Приложение-функция (Windows)** выполните следующие действия:
    1. Введите **имя** приложения-функции.
    1. Выберите **подписку** Azure.
    1. Выберите существующую **группу ресурсов** или создайте другую. Для работы с этим руководством выберите группу ресурсов с пространством имен Служебной шины. 
    1. Выберите **типа плана** для Службы приложений.
    1. Выберите **расположение**. Выберите то же расположение, что и у пространства имен Служебной шины. 
    1. Выберите существующую **службу хранилища Azure** или создайте новую **учетную запись хранения**, которая будет использоваться приложением Функций. 
    1. Щелкните **Создать**, чтобы создать приложение Функций. 
1. Вернитесь на страницу **Экземпляр Функций** мастера **публикации** и щелкните **Готово**. 
1. На странице **Публикация** в Visual Studio выберите **Опубликовать**, чтобы опубликовать приложение Функций в Azure. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/publish-function-app.png" alt-text="Публикация приложения Функций":::    
1. В окне **Вывод** просмотрите сообщения, связанные со сборкой и публикацией, и убедитесь, что все в порядке. 
1. Теперь на странице **Публикация** щелкните **Управление на портале Azure**. 
1. На портале Azure на странице **Приложение-функция** выберите **Функции** в меню слева. Должны отобразиться две функции: 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/functions.png" alt-text="Две функции для обработчика триггера Сетки событий и триггера HTTP":::

    > [!NOTE]
    > `EventGridTriggerFunction` использует [триггер Сетки событий](../azure-functions/functions-bindings-event-grid-trigger.md), а `HTTPTriggerFunction` использует [триггер HTTP](../azure-functions/functions-bindings-http-webhook-trigger.md).
1. Выберите **EventGridTriggerFunction** в списке. Рекомендуется использовать триггер Сетки событий с Функциями Azure, так как его использование связано с несколькими преимуществами по сравнению с использованием триггера HTTP. Дополнительные сведения см. в статье [Использование функции Azure как обработчика событий Сетки событий](../event-grid/handler-functions.md).
1. На странице **Функция** для **EventGridTriggerFunction** выберите **Монитор** в меню слева. 
1. Выберите **Настроить**, чтобы настроить Application Insights для записи журнала вызовов. Эта страница используется для наблюдения за выполнением функций при получении событий Служебной шины из службы "Сетка событий". 
1. На странице **Application Insights** укажите имя и **расположение** ресурса, а затем щелкните **ОК**. 
1. Выберите функцию **EventGridTriggerFunction** вверху (меню навигации), чтобы вернуться на страницу **Функция**. 
1. Убедитесь, что вы находитесь на странице **Монитор**. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/monitor-before.png" alt-text="Страница монитора для функции перед вызовами":::
    
    Не закрывайте эту страницу в браузере. Вы обновите ее позже, чтобы увидеть вызовы для этой функции.

## <a name="connect-the-function-and-namespace-via-event-grid"></a>Подключение функции и пространства имен с помощью службы "Сетка событий"
В этом разделе вы свяжете функцию и пространство имен служебной шины с помощью портала Azure. 

Чтобы создать подписку на Сетку событий Azure, сделайте следующее:

1. На портале Azure перейдите к пространству имен и выберите в области слева **События**. На правой панели откроется окно пространства имен с двумя подписками на Сетку событий. 
    
    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/service-bus-events-page.png" alt-text="Служебная шина — страница событий":::
2. Выберите **+ Подписка на события** на панели инструментов. 
3. На странице **Создание подписки на события** сделайте следующее:
    1. Укажите **имя** подписки. 
    2. Введите **имя** **системного раздела**. Системные разделы — это разделы, созданные для ресурсов Azure, таких как учетная запись хранения Azure и служебная шина Azure. См. [общие сведения о системных разделах](../event-grid/system-topics.md).
    2. Выберите **Функции Azure** для параметра **Тип конечной точки** и щелкните **Выбрать конечную точку**. 

        ![Служебная шина — подписка на Сетку событий](./media/service-bus-to-event-grid-integration-example/event-grid-subscription-page.png)
    3. На странице **Выбор функции Azure** выберите подписку, группу ресурсов, приложение-функцию, слот и функцию, а затем щелкните **Подтвердить выбор**. 

        ![Функция — выбор конечной точки](./media/service-bus-to-event-grid-integration-example/function-select-endpoint.png)
    4. На странице **Создать подписку на события** перейдите на вкладку **Фильтры** и выполните следующие действия:
        1. Выберите **Включить фильтрацию тем**
        2. Введите имя подписки в раздел Служебной шины (**S1**), созданный ранее.
        3. Нажмите кнопку **Создать**. 

            ![Фильтр событий подписки](./media/service-bus-to-event-grid-integration-example/event-subscription-filter.png)
4. Перейдите на вкладку **Подписки на события** на странице **События** и убедитесь, что в списке отображается подписка на события.

    ![Подписка на события в списке](./media/service-bus-to-event-grid-integration-example/event-subscription-in-list.png)

## <a name="monitor-the-functions-app"></a>Мониторинг приложения-функции
Сообщения, отправленные ранее в раздел Служебной шины, перенаправляются в подписку (S1). Сетка событий перенаправляет сообщения в подписке в функцию Azure. Теперь вы подтвердите, что функция была вызвана, и просмотрите записанные информационные сообщения. 

1. На странице приложения-функции Azure перейдите на вкладку **Монитор**, если она еще не открыта. Вы должны увидеть запись для каждого сообщения, опубликованного в разделе Служебной шины. Если записи не отображаются, обновите страницу через несколько минут. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/function-monitor.png" alt-text="Страница монитора для функции после вызовов":::
2. Выберите вызов в списке, чтобы просмотреть подробные сведения. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/invocation-details.png" alt-text="Сведения о вызове функции" lightbox="./media/service-bus-to-event-grid-integration-example/invocation-details.png":::

    Просмотреть сведения журнала по мере отправки сообщений можно также на вкладке **Журналы** на странице **Монитор**. Сообщения в журнале могут появляться с задержкой до нескольких минут. 

    :::image type="content" source="./media/service-bus-to-event-grid-integration-example/function-logs.png" alt-text="Журналы функций" lightbox="./media/service-bus-to-event-grid-integration-example/function-logs.png":::

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