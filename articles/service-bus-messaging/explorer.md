---
title: Использование Azure Service Bus Explorer для выполнения операций с данными в Служебной шине (предварительная версия)
description: Эта статья содержит сведения об использовании Azure Service Bus Explorer на портале для доступа к данным Служебной шины Azure.
ms.topic: conceptual
ms.date: 01/20/2020
ms.openlocfilehash: 0b5274c492a1dfb2523c52d7aea2b7ebf8eae675
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101738962"
---
# <a name="use-service-bus-explorer-to-perform-data-operations-on-service-bus-preview"></a>Использование Service Bus Explorer для выполнения операций с данными в Служебной шине (предварительная версия)

## <a name="overview"></a>Обзор

Служебная шина Azure позволяет клиентским приложениям отправителя и получателя отделить свою бизнес-логику с помощью привычной семантики "точка —точка" (очередь) и "публикация — подписка" (раздел — подписка).

Операции, выполняемые в пространстве имен Служебной шины Azure, бывают двух типов: 
   * Операции управления: создание, обновление, удаление пространства имен служебной шины, очередей, разделов и подписок.
   * Операции с данными: отправка и получение сообщений в очередях, разделах и подписках.

Azure Service Bus Explorer расширяет функциональные возможности портала за пределы операций управления для поддержки операций с данными (отправка, получение, просмотр) в очередях, разделах и подписках (и их вложенных сущностях недоставленных сообщений) прямо из самого портала Azure.

> [!NOTE]
> В этой статье описаны функциональные возможности Azure Service Bus Explorer, используемые на портале Azure.
>
> Средство Azure Service Bus Explorer ***не*** является средством [Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer), принадлежащим сообществу OSS.
>

## <a name="prerequisites"></a>Предварительные требования

Для использования средства обозревателя служебной шины необходимо выполнить следующие задачи. 

- Подготавливает пространство имен служебной шины Azure.
- Создайте очередь для отправки и получения сообщений из или раздела с подпиской для проверки функциональности. Сведения о создании очередей, разделов и подписок см. в следующих статьях: 
    - [Краткое руководство. Создание очередей](service-bus-quickstart-portal.md)
    - [Краткое руководство. Создание разделов](service-bus-quickstart-topics-subscriptions-portal.md)
- Убедитесь, что вы являетесь членом одной из этих ролей в пространстве имен: 
    - [Владелец данных служебной шины](../role-based-access-control/built-in-roles.md#azure-service-bus-data-owner) 
    - [Участник](../role-based-access-control/built-in-roles.md#contributor) 
    - [Владелец](../role-based-access-control/built-in-roles.md#owner)


## <a name="using-the-service-bus-explorer"></a>Использование Service Bus Explorer

Чтобы использовать Azure Service Bus Explorer, необходимо перейти в пространство имен Служебной шины, в котором вы хотите выполнять операции отправки, просмотра и получения.

Если вы хотите выполнить операции с очередью, выберите **Очереди** в меню навигации. Если вы хотите выполнять операции с разделом (и связанными с ним подписками), выберите **Разделы**. 

:::image type="content" source="./media/service-bus-explorer/queue-topics-left-navigation.png" alt-text="Выбор сущности":::

Выбрав **Очереди** или **Разделы**, выберите конкретную очередь или раздел.

Затем выберите элемент **Service Bus Explorer (предварительная версия)** в меню навигации слева.

:::image type="content" source="./media/service-bus-explorer/left-navigation-menu-selected.png" alt-text="Меню навигации Service Bus Explorer":::

### <a name="sending-a-message-to-a-queue-or-topic"></a>Отправление сообщения в очередь или раздел

Чтобы отправить сообщение в **очередь** или **раздел**, щелкните вкладку **_Отправить_** в обозревателе служебной шины.

Чтобы создать здесь сообщение, сделайте следующее: 

1. Задайте для параметра **Тип содержимого** значение Text/Plain, Application/Xml или Application/Json.
2. Добавьте **содержимое** сообщения. Убедитесь, что оно соответствует заданному ранее полю **Тип содержимого**.
3. Задайте **Дополнительные свойства** (необязательно). Сюда входят идентификатор корреляции, идентификатор сообщения, метка, ReplyTo, срок жизни (TTL) и время плановой постановки в очередь (для запланированных сообщений).
4. Задайте **Пользовательские свойства**. Это могут быть любые свойства пользователя, заданные для ключа словаря.

После составления сообщения нажмите кнопку "Отправить".

:::image type="content" source="./media/service-bus-explorer/send-experience.png" alt-text="Создание сообщения":::

После успешного завершения операции отправки: 

* При отправке в очередь значение счетчика метрик **Активные сообщения** будет увеличено.

    :::image type="content" source="./media/service-bus-explorer/queue-after-send-metrics.png" alt-text="QueueAfterSendMetrics":::

* При отправке в раздел значение счетчика метрик **Активные сообщения** будет увеличено для подписки, в которую было направлено сообщение.

    :::image type="content" source="./media/service-bus-explorer/topic-after-send-metrics.png" alt-text="TopicAfterSendMetrics":::

### <a name="receiving-a-message-from-a-queue"></a>Получение сообщения из очереди

Функция получения в Service Bus Explorer разрешает получение одного сообщения за раз. Операция получения выполняется с использованием режима **ReceiveAndDelete**.

> [!IMPORTANT]
> Обратите внимание, что операция получения, выполняемая Service Bus Explorer, — это ***получение с удалением***, т. е. сообщение удаляется из очереди, если оно отображается в средстве Service Bus Explorer.
>
> Чтобы просмотреть сообщения, не удаляя их из очереди, рассмотрите возможность использования функции ***просмотра***.
>

Чтобы получить сообщение из очереди (или ее подочереди недоставленных сообщений), сделайте следующее: 

1. Перейдите на вкладку ***Получение*** в Service Bus Explorer.
2. Проверьте метрики, чтобы узнать, есть ли для получения **активные сообщения** или **недоставленные сообщения**.

    :::image type="content" source="./media/service-bus-explorer/queue-after-send-metrics.png" alt-text="QueueAfterSendMetrics":::

3. Выберите подочередь ***Queue** _ или _ *_недоставлен_**.

    :::image type="content" source="./media/service-bus-explorer/queue-or-deadletter.png" alt-text="QueueOrDeadletter":::

4. Нажмите кнопку "**получить** _", а затем — "*_Да"_**, чтобы подтвердить операцию получения и удаления.


После успешного выполнения операции получения сведения о сообщении отобразятся в сетке как показано ниже. Вы можете выбрать сообщение из сетки, чтобы отобразить сведения о нем.

:::image type="content" source="./media/service-bus-explorer/receive-message-from-queue-2.png" alt-text="Снимок экрана: окно &quot;очереди&quot; в обозревателе служебной шины Azure со сведениями о сообщении, отображаемым для активного сообщения, выбранного в очереди.":::


### <a name="peeking-a-message-from-a-queue"></a>Просмотр сообщения из очереди

Благодаря функциональности просмотра в Service Bus Explorer можно просматривать первые 32 сообщения в очереди или очереди недоставленных сообщений.

1. Чтобы просмотреть сообщение в очереди, выберите вкладку ***Обзор*** в окне Service Bus Explorer.

    :::image type="content" source="./media/service-bus-explorer/peek-tab-selected.png" alt-text="PeekTab":::

2. Проверьте метрики, чтобы узнать, есть ли для просмотра **активные сообщения** или **недоставленные сообщения**.

    :::image type="content" source="./media/service-bus-explorer/queue-after-send-metrics.png" alt-text="QueueAfterSendMetrics":::

3. Затем выберите подочередь ***Queue** _ или _ *_недоставлен_**.

    :::image type="content" source="./media/service-bus-explorer/queue-or-deadletter.png" alt-text="QueueOrDeadletter":::

4. Нажмите кнопку ***Обзор***. 

После завершения операции просмотра в сетке будут отображаться до 32 сообщений, как показано ниже. Чтобы просмотреть подробные сведения о конкретном сообщении, выберите его в сетке. 

:::image type="content" source="./media/service-bus-explorer/peek-message-from-queue-2.png" alt-text="PeekMessageFromQueue":::

> [!NOTE]
>
> Так как операция просмотра выполняется без удаления, сообщения **не будут** удаляться из очереди.
>

### <a name="receiving-a-message-from-a-subscription"></a>Получение сообщения из подписки

Как и в случае с очередью, операцию ***получения*** можно выполнить для подписки (или ее сущности "Невостребованное"). Но так как подписка находится в контексте раздела, операция получения выполняется путем перехода в Service Bus Explorer для определенного раздела.

> [!IMPORTANT]
> Обратите внимание, что операция получения, выполняемая Service Bus Explorer, — это ***получение с удалением***, т. е. сообщение удаляется из очереди, если оно отображается в средстве Service Bus Explorer.
>
> Чтобы просмотреть сообщения, не удаляя их из очереди, рассмотрите возможность использования функции ***просмотра***.
>

1. Перейдите на вкладку ***получить** _ и выберите в раскрывающемся списке конкретную *_подписку_** *.

    :::image type="content" source="./media/service-bus-explorer/receive-subscription-tab-selected.png" alt-text="ReceiveTabSelected":::

2. Выберите подсущность ***Подписка** _ или _ *_недоставленный_**.

    :::image type="content" source="./media/service-bus-explorer/subscription-or-deadletter.png" alt-text="SubscriptionOrDeadletter":::

3. Нажмите кнопку "**получить** _", а затем — "*_Да"_**, чтобы подтвердить операцию получения и удаления.

После успешного выполнения операции получения в сетке как показано ниже отобразится полученное сообщение. Щелкните это сообщение, чтобы просмотреть подробные сведения о нем.

:::image type="content" source="./media/service-bus-explorer/receive-message-from-subscription.png" alt-text="Снимок экрана: вкладка &quot;получить&quot; в обозревателе служебной шины Azure со сведениями о сообщении для активного сообщения, которое было получено.":::

### <a name="peeking-a-message-from-a-subscription"></a>Просмотр сообщения из подписки

Чтобы просто просмотреть сообщения в подписке или ее подсущности недоставленных сообщений, можно также использовать функцию ***просмотра***.

1. Перейдите на вкладку ***Peek** _ и выберите в раскрывающемся списке конкретную *_подписку_** *.

    :::image type="content" source="./media/service-bus-explorer/peek-subscription-tab-selected.png" alt-text="PeekTabSelected":::

2. Выберите подсущность ***Подписка** _ или _ *_недоставленный_**.

    :::image type="content" source="./media/service-bus-explorer/subscription-or-deadletter.png" alt-text="SubscriptionOrDeadletter":::

3. Нажмите кнопку ***Обзор***.

После завершения операции просмотра в сетке будут отображаться до 32 сообщений, как показано ниже. Чтобы просмотреть подробные сведения о конкретном сообщении, выберите его в сетке. 

:::image type="content" source="./media/service-bus-explorer/peek-message-from-subscription.png" alt-text="PeekMessageFromSubscription":::

> [!NOTE]
>
> Так как операция просмотра выполняется без удаления, сообщения **не будут** удаляться из очереди.
>

## <a name="next-steps"></a>Next Steps

   * Ознакомьтесь с дополнительными сведениями об [очередях](service-bus-queues-topics-subscriptions.md#queues) и [разделах](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) Служебной шины Azure.
   * Ознакомьтесь с дополнительными сведениями о создании [очередей Служебной шины Azure с помощью портала Azure](service-bus-quickstart-portal.md).
   * Ознакомьтесь с дополнительными сведениями о создании [разделов и подписок Служебной шины Azure с помощью портала Azure](service-bus-quickstart-topics-subscriptions-portal.md).