---
title: Отслеживание оценки безопасности в центре безопасности Azure
description: Узнайте о нескольких способах доступа и контроля оценки безопасности в центре безопасности Azure.
author: memildin
ms.author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 02/25/2021
ms.openlocfilehash: 5efc48d348e9cfceab590bcfba8c621e7721376f
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102107964"
---
# <a name="access-and-track-your-secure-score"></a>Доступ и контроль оценки безопасности

Вы можете найти общую оценку безопасности, а также оценку по подписке с помощью портал Azure или программно, как описано в следующих разделах:

> [!TIP]
> Подробное описание расчета оценок см. [в разделе вычисления — понимание оценки](secure-score-security-controls.md#calculations---understanding-your-score).

## <a name="get-your-secure-score-from-the-portal"></a>Получение оценки безопасности на портале

Центр безопасности Отображает оценочный рейтинг на портале: это первая Основная плитка страницы обзор центра безопасности. При выборе этой плитки вы переходите на страницу выделенной оценки безопасности, где вы увидите, что оценка разбита по подписке. Выберите одну подписку, чтобы просмотреть подробный список приоритетных рекомендаций и потенциальное влияние, устранения их на оценку подписки. 

Итак, Оценка безопасности показана в следующих расположениях на страницах портала центра безопасности.

- В плитке **Обзор** центра безопасности (основная панель мониторинга):

    :::image type="content" source="./media/secure-score-security-controls/score-on-main-dashboard.png" alt-text="Оценка безопасности на панели мониторинга центра безопасности":::

- На странице выделенная **Оценка безопасности** можно увидеть оценку безопасности подписки и групп управления.

    :::image type="content" source="./media/secure-score-security-controls/score-on-dedicated-dashboard.png" alt-text="Оценка безопасности подписок на странице &quot;Оценка безопасности&quot; центра безопасности":::

    :::image type="content" source="./media/secure-score-security-controls/secure-score-management-groups.png" alt-text="Оценка безопасности для групп управления на странице &quot;Оценка безопасности&quot; центра безопасности":::

    > [!NOTE]
    > Все группы управления, для которых у вас нет необходимых разрешений, будут отображать их оценку "ограничено". 

- В верхней части страницы **рекомендации** :

    :::image type="content" source="./media/secure-score-security-controls/score-on-recommendations-page.png" alt-text="Оценка безопасности на странице рекомендаций центра безопасности":::

## <a name="get-your-secure-score-from-the-rest-api"></a>Получение оценки безопасности из REST API

Вы можете получить доступ к оценке через API оценки безопасности. Методы этого API позволяют гибко выполнять запросы к данным и создавать собственные механизмы создания отчетов об оценках безопасности за разные периоды. Например, можно использовать [API для оценки безопасности](/rest/api/securitycenter/securescores) , чтобы получить оценку для конкретной подписки. Кроме того, можно использовать [API управления оценками](/rest/api/securitycenter/securescorecontrols) безопасности для перечисления элементов управления безопасностью и текущей оценки подписок.

![Получение одной безопасной оценки через API](media/secure-score-security-controls/single-secure-score-via-api.png)

Примеры средств, построенных на основе API оценки безопасности, см. [в области "Оценка безопасности" сообщества GitHub](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score). 

## <a name="get-your-secure-score-from-azure-resource-graph"></a>Получение оценки безопасности из графа ресурсов Azure

Граф ресурсов Azure обеспечивает мгновенный доступ к сведениям о ресурсах в облачных средах с помощью надежных возможностей фильтрации, группировки и сортировки. Это быстрый и эффективный способ запросить сведения по подпискам Azure программно или с помощью портала Azure. [Узнайте больше об Azure Resource Graph.](../governance/resource-graph/index.yml)

Чтобы получить доступ к оценкам безопасности нескольких подписок с помощью графа ресурсов Azure, сделайте следующее:

1. В портал Azure откройте **Обозреватель графа ресурсов Azure**.

    :::image type="content" source="./media/security-center-identity-access/opening-resource-graph-explorer.png" alt-text="Страница рекомендаций по запуску обозревателя графа ресурсов Azure * *" :::

1. Введите запрос Kusto (в приведенных ниже примерах см. руководство).

    - Этот запрос возвращает идентификатор подписки, текущую оценку в точках и в процентах и максимальную оценку подписки. 

        ```kusto
        SecurityResources 
        | where type == 'microsoft.security/securescores' 
        | extend current = properties.score.current, max = todouble(properties.score.max)
        | project subscriptionId, current, max, percentage = ((current / max)*100)
        ```

    - Этот запрос возвращает состояние всех элементов управления безопасностью. Для каждого элемента управления вы получите число неработоспособных ресурсов, текущую оценку и максимальную оценку. 

        ```kusto
        SecurityResources 
        | where type == 'microsoft.security/securescores/securescorecontrols'
        | extend SecureControl = properties.displayName, unhealthy = properties.unhealthyResourceCount, currentscore = properties.score.current, maxscore = properties.score.max
        | project SecureControl , unhealthy, currentscore, maxscore
        ```

1. Выберите **выполнить запрос**.


## <a name="tracking-your-secure-score-over-time"></a>Отслеживание оценки безопасности с течением времени

### <a name="secure-score-over-time-report-in-workbooks-page"></a>Отчет о безопасности с оценками по времени на странице книг

Страница книги центра безопасности содержит готовый отчет для визуального отслеживания оценок подписок, элементов управления безопасностью и многого другого. Дополнительные сведения см. в статье [Создание полнофункциональных интерактивных отчетов по данным центра безопасности](custom-dashboards-azure-workbooks.md).

:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-snip.png" alt-text="Раздел из коллекции книг центра безопасности Azure с отчетом о безопасности с учетом времени":::

### <a name="power-bi-pro-dashboards"></a>Панели мониторинга Power BI Pro

Если вы Power BI пользователя с учетной записью Pro, вы можете использовать **оценку безопасности** с помощью Power BI панели мониторинга, чтобы отслеживанию безопасной оценки с течением времени и изучения любых изменений.

> [!TIP]
> Вы можете найти эту панель мониторинга, а также другие средства для программной работы с более безопасными показателями в выделенной области сообщества центра безопасности Azure на сайте GitHub: https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score

На панели мониторинга содержатся следующие два отчета, помогающие анализировать состояние безопасности.

- **Сводка по ресурсам** . предоставляет сводные данные о работоспособности ресурсов.
- **Сводка по показателям безопасности** . предоставляет сводные данные о ходе выполнения оценки. Используйте диаграмму "безопасность оценки по времени на подписку" для просмотра изменений в оценке. Если вы заметили значительное изменение в оценке, проверьте наличие изменений в таблице "Обнаруженные изменения, которые могут повлиять на вашу оценку", чтобы внести изменения, которые могли привести к внесению изменений. В этой таблице представлены удаленные ресурсы, развернутые ресурсы или ресурсы, для которых в соответствии с рекомендациями изменилось состояние безопасности.

:::image type="content" source="./media/secure-score-security-controls/power-bi-secure-score-dashboard.png" alt-text="Необязательная Оценка безопасности по времени Power BI панели мониторинга для отслеживания оценки безопасности с течением времени и изучения изменений.":::


## <a name="next-steps"></a>Дальнейшие действия

В этой статье описывается, как получить и отвести оценку безопасности. Связанные материалы см. в следующих статьях:

- [Узнайте о различных элементах рекомендации](security-center-recommendations.md)
- [Узнайте, как выполнять рекомендации](security-center-remediate-recommendations.md)
- [Просмотр средств на основе GitHub для программной работы с помощью оценки безопасности](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score)