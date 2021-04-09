---
title: Диагностика распространенных сценариев Azure Service Fabric
description: Сведения об устранении неполадок в распространенных сценариях мониторинга и диагностики в приложениях Azure Service Fabric.
ms.topic: article
ms.date: 02/25/2019
ms.openlocfilehash: 3083348948ce97ae9d99098896878d79081a15f0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100580490"
---
# <a name="diagnose-common-scenarios-with-service-fabric"></a>Диагностика распространенных сценариев с помощью Service Fabric

В этой статье показаны общие сценарии, с которыми сталкиваются пользователи в области мониторинга и диагностики с помощью Service Fabric. Представленные сценарии охватывают все три уровня Service Fabric: приложения, кластеры и инфраструктуры. Для каждого сценария каждое решение использует Application Insights, журналы Azure Monitor и средства мониторинга Azure. Шаги в каждом решении позволяют пользователям понять, как использовать Application Insights и журналы Azure Monitor в контексте Service Fabric.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites-and-recommendations"></a>Предварительные требования и рекомендации

Решения в этой статье будут использовать следующие средства. Мы рекомендуем вам настроить следующую конфигурацию:

* [Application Insights в Service Fabric](service-fabric-tutorial-monitoring-aspnet.md).
* [Включение диагностики Azure в кластере](service-fabric-diagnostics-event-aggregation-wad.md).
* [Настройка рабочей области Log Analytics](service-fabric-diagnostics-oms-setup.md)
* [Агент Log Analytics для отслеживания счетчиков производительности](service-fabric-diagnostics-oms-agent.md)

## <a name="how-can-i-see-unhandled-exceptions-in-my-application"></a>Как я могу увидеть необработанные исключения в своем приложении?

1. Перейдите к ресурсу Application Insights, с помощью которого настроено ваше приложение.
2. Щелкните *Поиск* в левом верхнем углу. Затем щелкните фильтр на следующей панели.

    ![Страница "Обзор" в Application Insights](media/service-fabric-diagnostics-common-scenarios/ai-search-filter.png)

3. Вы увидите множество типов событий (трассировки, запросы, пользовательские события). Выберите "Исключение" в качестве фильтра.

    ![Список фильтров AI](media/service-fabric-diagnostics-common-scenarios/ai-filter-list.png)

    Щелкая исключения в списке, вы можете посмотреть более подробные сведения, включая контекст службы, если вы используете пакет SDK Service Fabric Application Insights.

    ![Исключение AI](media/service-fabric-diagnostics-common-scenarios/ai-exception.png)

## <a name="how-do-i-view-which-http-calls-are-used-in-my-services"></a>Как просмотреть HTTP-вызовы, которые используются в моих службах?

1. В том же ресурсе Application Insights вы можете фильтровать запросы вместо исключений и просматривать все сделанные запросы.
2. Если вы используете пакет SDK для Service Fabric Application Insights, вы можете увидеть визуальное представление ваших служб, связанных друг с другом, а также количество успешных и неудачных запросов. В левой части экрана нажмите вкладку "Схема приложений"

    ![Колонка схемы приложения ИИ](media/service-fabric-diagnostics-common-scenarios/app-map-blade.png) ![Схема приложения ИИ](media/service-fabric-diagnostics-common-scenarios/app-map-new.png)

    Дополнительные сведения о схеме приложения см. в документации [по схеме приложения](../azure-monitor/app/app-map.md).

## <a name="how-do-i-create-an-alert-when-a-node-goes-down"></a>Как создать оповещение, когда узел выходит из строя

1. События узла отслеживаются кластером Service Fabric. Перейдите к ресурсу решения Аналитики Service Fabric с именем **ServiceFabric (NameofResourceGroup)**
2. Щелкните график в нижней части колонки под названием "Сводка"

    ![Решение журналов Azure Monitor](media/service-fabric-diagnostics-common-scenarios/oms-solution-azure-portal.png)

3. Здесь находится множество графиков и плиток, отображающих различные показатели. Щелкните один из графиков и перейдите к поиску по журналам. Здесь вы можете запросить любые события кластера или счетчики производительности.
4. Введите следующий запрос. Эти идентификаторы событий находятся в справке [по событиям узла](service-fabric-diagnostics-event-generation-operational.md#application-events).

    ```kusto
    ServiceFabricOperationalEvent
    | where EventID >= 25622 and EventID <= 25626
    ```

5. Щелкните "Новое правило оповещения" вверху, и теперь в любое время, когда поступит событие на основе этого запроса, вы получите оповещение в выбранном вами способе взаимодействия.

    ![Новое оповещение журналов Azure Monitor](media/service-fabric-diagnostics-common-scenarios/oms-create-alert.png)

## <a name="how-can-i-be-alerted-of-application-upgrade-rollbacks"></a>Как можно получить уведомления об откате обновления приложений?

1. В том же окне поиска по журналам, как и прежде, введите следующий запрос откатов обновлений. Эти идентификаторы событий находятся в справке [по событиям приложений](service-fabric-diagnostics-event-generation-operational.md#application-events).

    ```kusto
    ServiceFabricOperationalEvent
    | where EventID == 29623 or EventID == 29624
    ```

2. Щелкните "Новое правило оповещения" вверху, и теперь в любое время, когда поступит событие на основе этого запроса, вы получите оповещение.

## <a name="how-do-i-see-container-metrics"></a>Как просмотреть метрики контейнера?

В том же представлении со всеми графиками вы увидите некоторые плитки для производительности ваших контейнеров. Для заполнения этих плиток вам потребуется агент Log Analytics и [решение для мониторинга контейнеров](service-fabric-diagnostics-oms-containers.md).

![Метрики контейнеров Log Analytics](media/service-fabric-diagnostics-common-scenarios/containermetrics.png)

>[!NOTE]
>Для телеметрии инструментирования **внутри** вашего контейнера вам нужно добавить [пакет nuget Application Insights для контейнеров](https://github.com/Microsoft/ApplicationInsights-servicefabric#microsoftapplicationinsightsservicefabric--for-service-fabric-lift-and-shift-scenarios).

## <a name="how-can-i-monitor-performance-counters"></a>Как я могу контролировать счетчики производительности?

1. После того как вы добавили агент Log Analytics в свой кластер, вам нужно добавить определенные счетчики производительности, которые вы хотите отслеживать. Перейдите на страницу рабочей области Log Analytics на портале. На странице решения вкладка рабочей области находится в меню слева.

    ![Вкладка рабочей области Log Analytics](media/service-fabric-diagnostics-common-scenarios/workspacetab.png)

2. После этого на странице рабочего пространства, щелкните "Дополнительные настройки" в том же меню слева.

    ![Дополнительные параметры Log Analytics](media/service-fabric-diagnostics-common-scenarios/advancedsettingsoms.png)

3. Щелкните "Данные" > "Счетчики производительности Windows" ("Данные" > "Счетчики производительности Linux" для компьютеров Linux), чтобы начать сбор конкретных счетчиков с ваших узлов через агент Log Analytics. Вот примеры формата добавления счетчиков

   * `.NET CLR Memory(<ProcessNameHere>)\\# Total committed Bytes`
   * `Processor(_Total)\\% Processor Time`

     В этом кратком руководстве именами процессов являются VotingData и VotingWeb, поэтому отслеживание этих счетчиков будет выглядеть так:

   * `.NET CLR Memory(VotingData)\\# Total committed Bytes`
   * `.NET CLR Memory(VotingWeb)\\# Total committed Bytes`

     ![Счетчики производительности Log Analytics](media/service-fabric-diagnostics-common-scenarios/omsperfcounters.png)

4. Это позволит вам увидеть, как ваша инфраструктура обрабатывает рабочие нагрузки, и установить соответствующие оповещения на основе использования ресурсов. Например, вы можете установить оповещение, если общее использование процессора превышает 90% или опускается ниже 5%. Имя счетчика, которое вы использовали для этого, — % Processor Time. Вы можете сделать это, создав правило оповещения для следующего запроса:

    ```kusto
    Perf | where CounterName == "% Processor Time" and InstanceName == "_Total" | where CounterValue >= 90 or CounterValue <= 5.
    ```

## <a name="how-do-i-track-performance-of-my-reliable-services-and-actors"></a>Как отслеживать работу моих Reliable Services и субъектов?

Чтобы отслеживать эффективность Reliable Services или субъектов в ваших приложениях, вы должны также собирать данные со счетчиков субъекта Service Fabric, метода субъекта, службы и метода службы. Ниже приведены примеры счетчиков производительности Reliable Services и субъектов, с которых необходимо собирать данные.

>[!NOTE]
>В настоящее время агент Log Analytics не может собирать данные со счетчиков производительности Service Fabric, но это могут делать [другие диагностические решения](service-fabric-diagnostics-partners.md)

* `Service Fabric Service(*)\\Average milliseconds per request`
* `Service Fabric Service Method(*)\\Invocations/Sec`
* `Service Fabric Actor(*)\\Average milliseconds per request`
* `Service Fabric Actor Method(*)\\Invocations/Sec`

Проверьте полный список счетчиков производительности по этим ссылкам [Reliable Services](service-fabric-reliable-serviceremoting-diagnostics.md) и [Субъекты](service-fabric-reliable-actors-diagnostics.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Поиск распространенных ошибок активации пакетов кода](./service-fabric-diagnostics-code-package-errors.md)
* [Настройте оповещения в Application Insights](../azure-monitor/alerts/alerts-log.md), чтобы узнавать об изменениях в производительности или характере использования.
* [Интеллектуальное обнаружение в Application Insights](../azure-monitor/app/proactive-diagnostics.md) осуществляет упреждающий анализ данных телеметрии, отправляемых в Application Insights, и предупреждает о потенциальных проблемах с производительностью.
* Узнайте больше об [оповещениях](../azure-monitor/alerts/alerts-overview.md) Azure Monitor, которые помогут в обнаружении и диагностике.
* Для локальных кластеров журналы Azure Monitor предлагают шлюз (прокси-сервер переадресации HTTP), который можно использовать для отправки данных в журналы Azure Monitor. Дополнительные сведения см. в разделе [Подключение компьютеров к журналам Azure Monitor с помощью шлюза Log Analytics без доступа к Интернету](../azure-monitor/agents/gateway.md)
* Ознакомьтесь с функциями [поиска по журналам и запросов к журналам](../azure-monitor/logs/log-query-overview.md), которые являются частью журналов Azure Monitor
* Более подробные сведения о журналах Azure Monitor и их возможностях см. в статье [Что такое журналы Azure Monitor?](../azure-monitor/overview.md)
