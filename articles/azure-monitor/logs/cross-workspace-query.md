---
title: Запрос ресурсов с помощью Azure Monitor | Документация Майкрософт
description: Из этой статьи вы узнаете, как выполнять запросы к ресурсам из нескольких рабочих областей и приложения App Insights в вашей подписке.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/22/2020
ms.openlocfilehash: 57ed43b25c9031138a91f0870d316e1ae7a07a5b
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102030973"
---
# <a name="perform-log-query-in-azure-monitor-that-span-across-workspaces-and-apps"></a>Выполнение запроса журнала в Azure Monitor, охватывающем рабочие области и приложения

Журналы Azure Monitor поддерживают запросы к нескольким рабочим областям Log Analytics и приложениям Application Insights в одной группе ресурсов, другой группе ресурсов или другой подписке. Благодаря этому вы можете получить представление данных на уровне системы.

Существует два метода для запроса данных, которые хранятся в нескольких рабочих областях и приложениях:
1. Явно, указав сведения о рабочей области и приложении. Этот метод подробно описан в этой статье.
2. Неявно с помощью [запросов контекста ресурсов](./design-logs-deployment.md#access-mode). При выполнении запроса в контексте определенного ресурса, группы ресурсов или подписки соответствующие данные будут получены из всех рабочих областей, содержащих данные для этих ресурсов. Application Insights данные, хранящиеся в приложениях, не будут получены.

> [!IMPORTANT]
> Если вы используете Application Insights телеметрии [ресурсов на основе рабочей области](../app/create-workspace-resource.md) , она хранится в log Analytics рабочей области со всеми остальными данными журнала. Используйте выражение Workspace () для написания запроса, включающего в себя приложение в нескольких рабочих областях. Для нескольких приложений в одной рабочей области не требуется перекрестный запрос к рабочей области.


## <a name="cross-resource-query-limits"></a>Ограничения запросов между ресурсами 

* Число Application Insightsных ресурсов и Log Analytics рабочих областей, которые можно включить в один запрос, ограничено 100.
* Запросы между несколькими ресурсами не поддерживаются в конструкторе представлений. Вы можете создать запрос в Log Analytics и закрепить его на панели мониторинга Azure для [визуализации запроса журнала](../visualize/tutorial-logs-dashboards.md). 
* Запросы перекрестных ресурсов в оповещениях журнала поддерживаются только в текущем [API счедуледкуерирулес](/rest/api/monitor/scheduledqueryrules). Если вы используете API устаревших Log Analyticsных предупреждений, необходимо [переключиться на текущий API](../alerts/alerts-log-api-switch.md).


## <a name="querying-across-log-analytics-workspaces-and-from-application-insights"></a>Выполнение запросов в рабочих областях Log Analytics и запрос данных из приложения Application Insights
Воспользуйтесь идентификатором [*рабочей области*](../logs/workspace-expression.md) для обращения к другой рабочей области в запросе, а для Application Insights используйте идентификатор [*приложения*](./app-expression.md).  

### <a name="identifying-workspace-resources"></a>Определение ресурсов рабочей области
В приведенных ниже примерах демонстрируются запросы к различным рабочим областям Log Analytics. Эти запросы предназначены для получения итогового количества журналов из таблицы "Обновление" в рабочей области с именем *contosoretail-it*. 

Определить рабочую область можно несколькими способами:

* Имя ресурса — это имя рабочей области в удобном для восприятия формате, которое иногда называется *именем компонента*. 

    `workspace("contosoretail-it").Update | count`

* Полное имя — это полное имя рабочей области, состоящее из имени подписки, группы ресурсов и имени компонента в следующем формате: *subscriptionName/resourceGroup/componentName*. 

    `workspace('contoso/contosoretail/contosoretail-it').Update | count`

    >[!NOTE]
    >Так как имена подписки Azure не являются уникальными, этот идентификатор может быть неоднозначным. 
    >

* Идентификатор рабочей области — это уникальный, неизменяемый идентификатор, присвоенный каждой рабочей области и представленный в виде глобального уникального идентификатора (GUID).

    `workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update | count`

* Идентификатор ресурса Azure — это уникальный идентификатор рабочей области в Azure. Он используется, если имя ресурса является неоднозначным.  Для рабочих областей используется следующий формат: */subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft.OperationalInsights/workspaces/componentName*.  

    Пример.
    ``` 
    workspace("/subscriptions/e427519-5645-8x4e-1v67-3b84b59a1985/resourcegroups/ContosoAzureHQ/providers/Microsoft.OperationalInsights/workspaces/contosoretail-it").Update | count
    ```

### <a name="identifying-an-application"></a>Определение приложения
В приведенных ниже примерах возвращается итоговое количество запросов к приложению *fabrikamapp* в Application Insights. 

Определение приложения в Application Insights можно обеспечить с помощью выражения *app(Identifier)*.  Аргумент *Identifier* указывает приложение с помощью одного из следующих объектов:

* Имя ресурса — это имя приложения в удобном для восприятия формате, которое иногда называется *именем компонента*.  

    `app("fabrikamapp")`

    >[!NOTE]
    >Определение приложения по имени предполагает уникальность во всех доступных подписках. Если у вас есть несколько приложений с таким именем, запрос не выполнится из-за неоднозначности. В этом случае необходимо воспользоваться другим идентификатором.

* Полное имя — имя приложения, состоящее из имени подписки, группы ресурсов и компонента в следующем формате: *subscriptionName/resourceGroup/componentName*. 

    `app("AI-Prototype/Fabrikam/fabrikamapp").requests | count`

     >[!NOTE]
    >Так как имена подписки Azure не являются уникальными, этот идентификатор может быть неоднозначным. 
    >

* Идентификатор — глобальный уникальный идентификатор приложения.

    `app("b459b4f6-912x-46d5-9cb1-b43069212ab4").requests | count`

* Идентификатор ресурса Azure — это уникальный идентификатор приложения в Azure. Он используется, если имя ресурса является неоднозначным. Используется такой формат: */subscriptions/subscriptionId/resourcegroups/resourceGroup/providers/microsoft. Компоненты/OperationalInsights/componentName*.  

    Пример.
    ```
    app("/subscriptions/b459b4f6-912x-46d5-9cb1-b43069212ab4/resourcegroups/Fabrikam/providers/microsoft.insights/components/fabrikamapp").requests | count
    ```

### <a name="performing-a-query-across-multiple-resources"></a>Выполнение запроса для нескольких ресурсов
Вы можете запрашивать несколько ресурсов из любого экземпляра ресурса: это может быть сочетание рабочих областей и приложений.
    
Пример запроса в двух рабочих областях:    

```
union Update, workspace("contosoretail-it").Update, workspace("b459b4u5-912x-46d5-9cb1-p43069212nb4").Update
| where TimeGenerated >= ago(1h)
| where UpdateState == "Needed"
| summarize dcount(Computer) by Classification
```

## <a name="using-cross-resource-query-for-multiple-resources"></a>Использование запроса между ресурсами для нескольких ресурсов
Если использовать запросы между ресурсами, чтобы коррелировать данные из нескольких рабочих областей Log Analytics и ресурсов Application Insights, это усложнит запрос и его будет трудно обработать. Поэтому следует использовать [функции в запросах журнала Azure Monitor](./functions.md), чтобы отделить логику запроса от его области ресурсов, благодаря чему структура запроса становится упрощенной. В приведенном ниже примере показано, как отслеживать несколько ресурсов Application Insights и визуализировать количество неудачных запросов по имени приложения. 

Создайте запрос, как показано ниже, который ссылается на область ресурсов Application Insights. Команда `withsource= SourceApp` позволяет добавить столбец, который определяет имя приложения, отправившего журнал. [Сохраните запрос как функцию](./functions.md#create-a-function) с псевдонимом _applicationsScoping_.

```Kusto
// crossResource function that scopes my Application Insights resources
union withsource= SourceApp
app('Contoso-app1').requests, 
app('Contoso-app2').requests,
app('Contoso-app3').requests,
app('Contoso-app4').requests,
app('Contoso-app5').requests
```



Теперь вы можете [использовать эту функцию](./functions.md#use-a-function) в запросе между ресурсами, как показано ниже. Псевдоним функции _applicationsScoping_ возвращает объединение таблицы запросов из всех определенных приложений. После этого с помощью запроса фильтруются неудачные запросы и визуализируются тенденции по приложению. Оператор _parse_ в этом примере не является обязательным. Оператор позволяет извлечь имя приложения из свойства _SourceApp_.

```Kusto
applicationsScoping 
| where timestamp > ago(12h)
| where success == 'False'
| parse SourceApp with * '(' applicationName ')' * 
| summarize count() by applicationName, bin(timestamp, 1h) 
| render timechart
```

>[!NOTE]
> Этот метод нельзя использовать с оповещениями журнала, так как проверка доступа к ресурсам правила генерации оповещений, включая рабочие области и приложения, выполняется во время создания предупреждения. Добавление новых ресурсов в функцию после создания предупреждения не поддерживается. Если вы предпочитаете использовать функцию для определения области ресурсов в оповещениях журнала, необходимо изменить правило генерации оповещений на портале или с помощью шаблона диспетчер ресурсов, чтобы обновить ресурсы с областью действия. Кроме того, можно включить список ресурсов в запрос оповещения журнала.


![Timechart](media/cross-workspace-query/chart.png)

## <a name="next-steps"></a>Дальнейшие действия

- Просмотрите статью [Анализ данных журнала в Azure Monitor](./log-query-overview.md), где описываются запросы журнала и структуры данных журнала Azure Monitor.