---
title: Использование списков видео к просмотру Azure Sentinel
description: В этой статье описывается, как использовать Azure Sentinel ватчлистс для исследования угроз, импорта бизнес-данных, создания списков разрешений и обогащения данных событий.
services: sentinel
author: yelevin
ms.author: yelevin
ms.assetid: 1721d0da-c91e-4c96-82de-5c7458df566b
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: conceptual
ms.custom: mvc
ms.date: 09/06/2020
ms.openlocfilehash: 97509b878fb5e0cb28bddc5d1b58c21b32c34675
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99555652"
---
# <a name="use-azure-sentinel-watchlists"></a>Использование списков видео к просмотру Azure Sentinel

> [!IMPORTANT]
> Функция ватчлистс сейчас доступна в **предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

Azure Sentinel ватчлистс включает сбор данных из внешних источников данных для корреляции с событиями в среде Azure Sentinel. После создания можно использовать ватчлистс в поиске, правилах обнаружения, поиске угроз и отклике модули PlayBook. Ватчлистс хранятся в рабочей области Sentinel Azure в виде пар "имя-значение" и кэшируются для обеспечения оптимальной производительности запросов и низкой задержки.

Распространенные сценарии использования ватчлистс:

- Оперативное **исследование угроз** и реагирование на инциденты с помощью быстрого импорта IP-адресов, хэшей файлов и других данных из файлов CSV. После импорта можно использовать пары "имя-значение" списка воспроизведения для соединений и фильтров в правилах оповещений, поиске угроз, книгах, записных книжках и общих запросах.

- **Импорт бизнес-данных** в виде списка воспроизведения. Например, можно импортировать списки пользователей с привилегированным доступом к системе или уволенных сотрудников, а затем использовать списка воспроизведения для создания списков разрешений и запретов, используемых для обнаружения пользователей в сети и предотвращения их входа в сеть.

- **Сокращение выносливости предупреждений**. Создайте списки разрешений для подавления оповещений от группы пользователей, например пользователей с разрешенных IP-адресов, которые выполняют задачи, которые обычно активируют оповещение, и предотвращают появление предупреждений о неблагоприятных событиях.

- **Обогащение данных о событиях**. Используйте ватчлистс, чтобы расширить данные событий с помощью сочетаний «имя-значение», полученных из внешних источников данных.

## <a name="create-a-new-watchlist"></a>Создание нового списка воспроизведения

1. В портал Azure перейдите к конфигурации **Sentinel Azure**  >    >  **списка воспроизведения** , а затем выберите **Добавить новый**.

    > [!div class="mx-imgBorder"]
    > ![создать списка воспроизведения](./media/watchlists/sentinel-watchlist-new.png)

1. На странице **Общие** укажите имя, описание и псевдоним для списка воспроизведения, а затем нажмите кнопку **Далее**.

    > [!div class="mx-imgBorder"]
    > ![Страница "Общие" списка воспроизведения](./media/watchlists/sentinel-watchlist-general.png)

1. На странице **источник** выберите тип набора данных, отправьте файл, а затем нажмите кнопку **Далее**.

    :::image type="content" source="./media/watchlists/sentinel-watchlist-source.png" alt-text="Исходная страница списка воспроизведения" lightbox="./media/watchlists/sentinel-watchlist-source.png":::

    > [!NOTE]
    >
    > Передача файлов в настоящее время ограничена файлами размером до 3,8 МБ.

1. Просмотрите сведения, убедитесь, что они верны, а затем нажмите кнопку **создать**.

    > [!div class="mx-imgBorder"]
    > ![Страница проверки списка воспроизведения](./media/watchlists/sentinel-watchlist-review.png)

    После создания списка воспроизведения появляется уведомление.

    :::image type="content" source="./media/watchlists/sentinel-watchlist-complete.png" alt-text="уведомление об успешном создании списка воспроизведения" lightbox="./media/watchlists/sentinel-watchlist-complete.png":::

## <a name="use-watchlists-in-queries"></a>Использование ватчлистс в запросах

1. В портал Azure перейдите в раздел Конфигурация **Sentinel Azure**  >    >  **списка воспроизведения**, выберите списка воспроизведения, который требуется использовать, а затем щелкните **Просмотр в log Analytics**.

    :::image type="content" source="./media/watchlists/sentinel-watchlist-queries-list.png" alt-text="Использование ватчлистс в запросах" lightbox="./media/watchlists/sentinel-watchlist-queries-list.png":::

1. Элементы в списка воспроизведения автоматически извлекаются для запроса и отображаются на вкладке **результаты** . В приведенном ниже примере показаны результаты извлечения полей **ServerName** и **IPAddress** .

    > [!NOTE]
    > Отметка времени в запросах будет пропускаться как в пользовательском интерфейсе запроса, так и в запланированных предупреждениях.

    :::image type="content" source="./media/watchlists/sentinel-watchlist-queries-fields.png" alt-text="запросы с полями списка воспроизведения" lightbox="./media/watchlists/sentinel-watchlist-queries-fields.png":::
    
1. Вы можете запросить данные в любой таблице для данных из списка воспроизведения, рассматривая списка воспроизведения как таблицу для соединений и уточняющих запросов.

    ```kusto
    Heartbeat
    | lookup kind=leftouter _GetWatchlist('IPlist') 
     on $left.ComputerIP == $right.IPAddress
    ```
    :::image type="content" source="./media/watchlists/sentinel-watchlist-queries-join.png" alt-text="запросы к списка воспроизведения как подстановка":::

## <a name="use-watchlists-in-analytics-rules"></a>Использование ватчлистс в правилах аналитики

Чтобы использовать ватчлистс в правилах аналитики, в портал Azure перейдите в раздел   >    >  **аналитика** конфигурации Sentinel Azure и создайте правило с помощью `_GetWatchlist('<watchlist>')` функции в запросе.

1. В этом примере создайте списка воспроизведения с именем "ипватчлист" со следующими значениями:

    :::image type="content" source="./media/watchlists/create-watchlist.png" alt-text="список четырех элементов для списка воспроизведения":::

    :::image type="content" source="./media/watchlists/sentinel-watchlist-new-2.png" alt-text="Создание списка воспроизведения с четырьмя элементами":::

1. Затем создайте правило аналитики.  В этом примере мы включаем события только с IP-адресов в списка воспроизведения:

    ```kusto
    //Watchlist as a variable
    let watchlist = (_GetWatchlist('ipwatchlist') | project IPAddress);
    Heartbeat
    | where ComputerIP in (watchlist)
    ```
    ```kusto
    //Watchlist inline with the query
    Heartbeat
    | where ComputerIP in ( 
        (_GetWatchlist('ipwatchlist')
        | project IPAddress)
    )
    ```

:::image type="content" source="./media/watchlists/sentinel-watchlist-analytics-rule-2.png" alt-text="Использование ватчлистс в правилах аналитики":::

## <a name="view-list-of-watchlists-aliases"></a>Просмотреть список псевдонимов ватчлистс

Чтобы получить список псевдонимов списка воспроизведения, в портал Azure перейдите в раздел Общие журналы **Azure Sentinel**  >  **General**  >  и выполните следующий запрос: `_GetWatchlistAlias` . Список псевдонимов можно просмотреть на вкладке **результаты** .

> [!div class="mx-imgBorder"]
> ![ватчлистс списка](./media/watchlists/sentinel-watchlist-alias.png)

## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как использовать ватчлистс в Azure Sentinel для обогащения данных и улучшения исследований. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](./tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
