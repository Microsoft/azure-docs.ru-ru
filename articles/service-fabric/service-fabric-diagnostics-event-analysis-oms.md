---
title: Анализ событий Azure Service Fabric с помощью журналов Azure Monitor
description: Сведения о визуализации и анализе событий с помощью журналов Azure Monitor для мониторинга и диагностики кластеров Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 02/21/2019
ms.author: srrengar
ms.openlocfilehash: 678e96c9b5611cd6d72abd7c4582691b55ffe479
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100581551"
---
# <a name="event-analysis-and-visualization-with-azure-monitor-logs"></a>Анализ событий и визуализация с помощью журналов Azure Monitor
 Azure Monitor собирает и анализирует данные телеметрии от приложений и служб, размещенных в облаке, и предоставляет средства анализа, с помощью которых вы сможете максимально увеличить их доступность и производительность. В этой статье описано, как выполнять запросы в журналах Azure Monitor, чтобы получить подробные сведения и устранить неполадки, происходящие в кластере. Рассматриваются следующие распространенные вопросы:

* Как устранить неполадки событий работоспособности?
* Как узнать, что узел вышел из строя?
* Как узнать, запущены или остановлены ли службы приложения?

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="overview-of-the-log-analytics-workspace"></a>Обзор рабочей области Log Analytics

>[!NOTE] 
>Хранилище диагностики по умолчанию включено в кластере во время создания, но вам нужно настроить рабочую область Log Analytics для чтения из хранилища диагностики.

Журналы Azure Monitor собирает данные из управляемых ресурсов, включая таблицу службы хранилища Azure или агент, и сохраняют их в центральном репозитории. Эти данные далее могут использоваться для анализа, оповещения, визуализации или последующего экспорта. Журналы Azure Monitor поддерживают события, данные производительности или любые другие пользовательские данные. Ознакомьтесь [с инструкциями по настройке расширения диагностики для агрегирования событий](service-fabric-diagnostics-event-aggregation-wad.md) и [шагами по созданию log Analytics рабочей области для считывания данных из событий в хранилище](service-fabric-diagnostics-oms-setup.md) , чтобы убедиться, что данные переносятся в журналы Azure Monitor.

После получения данных журналами Azure Monitor в Azure есть несколько *решений для мониторинга* , которые являются предустановленными решениями или операционными панелями для мониторинга входящих данных, настроенных для нескольких сценариев. К ним относятся *Аналитика Service Fabric* и *Контейнеры* — два наиболее важных решения для диагностики и мониторинга кластеров Service Fabric. В этой статье описывается использование решения "Аналитика Service Fabric", созданного с помощью рабочей области.

## <a name="access-the-service-fabric-analytics-solution"></a>Получение доступа к решению "Аналитика Service Fabric"

На [портале Azure](https://portal.azure.com)перейдите к группе ресурсов, в которой вы создали аналитика Service Fabric решение.

Выберите ресурс **ServiceFabric \<nameOfOMSWorkspace\>**.

В `Summary` вы увидите плитки в форме графа для каждого включенного решения, включая одну для Service Fabric. Щелкните граф **Service Fabric**, чтобы продолжить решение службы Fabric Analytics.

![Решение Service Fabric](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_summary.PNG)

На приведенном ниже рисунке показана домашняя страница решения "Аналитика Service Fabric". На этой домашней странице представлена информация, касающаяся работы вашего кластера.

![Снимок экрана, на котором показана домашняя страница решения Аналитика Service Fabric.](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_solution.PNG)

 Если включить диагностику во время создания кластера, можно просмотреть такие события: 

* [События кластера Service Fabric](service-fabric-diagnostics-event-generation-operational.md).
* [События модели программирования на основе Reliable Actors](service-fabric-reliable-actors-diagnostics.md).
* [События модели программирования на основе Reliable Services](service-fabric-reliable-services-diagnostics.md).

>[!NOTE]
>Помимо готовых событий Service Fabric, можно собирать дополнительные системные события. Для этого [обновите файл конфигурации расширения диагностики](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations).

## <a name="view-service-fabric-events-including-actions-on-nodes"></a>Просмотр событий Service Fabric, включая действия на узлах

На странице аналитики Service Fabric щелкните диаграмму для **События Service Fabric**.

![Операционный канал решения Service Fabric](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_events_selection.png)

Щелкните **Список**, чтобы просмотреть список событий. Здесь вы увидите все собранные системные события. Эти данные взяты из таблицы **WADServiceFabricSystemEventsTable** в учетной записи хранения Azure, а события служб Reliable Services и субъектов Reliable Actors, которые показаны далее, также взяты из этих соответствующих таблиц.
    
![Операционный канал запроса](media/service-fabric-diagnostics-event-analysis-oms/oms_service_fabric_events.png)

Кроме того, чтобы найти необходимые данные, можно щелкнуть значок лупы в левой части экрана и воспользоваться языком запросов Kusto. Например, чтобы найти все действия, выполняемые в узлах кластера, можно использовать приведенный ниже запрос. Идентификаторы событий, используемые ниже, можно найти в [справочнике по событиям операционного канала](service-fabric-diagnostics-event-generation-operational.md).

```kusto
ServiceFabricOperationalEvent
| where EventId < 25627 and EventId > 25619 
```

Вы можете запрашивать много дополнительных полей (например, конкретные узлы (Компьютер) и системную службу (TaskName)).

## <a name="view-service-fabric-reliable-service-and-actor-events"></a>Просмотр событий служб Reliable Services и субъектов Reliable Actors в Service Fabric

На странице аналитики Service Fabric щелкните диаграмму для **Reliable Services**.

![Reliable Services в решении Service Fabric](media/service-fabric-diagnostics-event-analysis-oms/oms_reliable_services_events_selection.png)

Щелкните **Список**, чтобы просмотреть список событий. Здесь можно просмотреть события из служб Reliable Services. Когда служба RunAsync запускается и завершается, могут отображаться различные события. Обычно это происходит при развертывании и обновлении. 

![Запрос к Reliable Services](media/service-fabric-diagnostics-event-analysis-oms/oms_reliable_service_events.png)

Аналогичным образом можно просмотреть события субъектов Reliable Actors. Чтобы настроить более подробные события для субъектов Reliable Actors, необходимо изменить `scheduledTransferKeywordFilter` в конфигурации для расширения диагностики (см. ниже). Сведения об этих значениях можно найти в [справочнике по событиям субъектов Reliable Actors](service-fabric-reliable-actors-diagnostics.md#keywords).

```json
"EtwEventSourceProviderConfiguration": [
                {
                    "provider": "Microsoft-ServiceFabric-Actors",
                    "scheduledTransferKeywordFilter": "1",
                    "scheduledTransferPeriod": "PT5M",
                    "DefaultEvents": {
                    "eventDestination": "ServiceFabricReliableActorEventTable"
                    }
                },
```

Язык запросов Kusto предоставляет широкие возможности. Вы можете выполнить другой полезный запрос, чтобы узнать, какие узлы создают больше событий. Запрос на снимке экрана, приведенном ниже, показывает операционное событие Service Fabric агрегированное с конкретной службой и узлом.

![События запросов на каждом узле](media/service-fabric-diagnostics-event-analysis-oms/oms_kusto_query.png)

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы включить мониторинг инфраструктуры (т. е. счетчики производительности), перейдите к разделу о [добавлении агента Log Analytics](service-fabric-diagnostics-oms-agent.md). Агент собирает счетчики производительности и добавляет их в имеющуюся рабочую область.
* Для локальных кластеров Azure Monitor журналы предлагают шлюз (прокси-сервер HTTP Forward), который можно использовать для отправки данных в журналы Azure Monitor. Дополнительные сведения см. в статье [Подключение компьютеров без доступа к Интернету к журналам Azure Monitor с помощью шлюза log Analytics](../azure-monitor/agents/gateway.md).
* Настройте [автоматические оповещения](../azure-monitor/alerts/alerts-overview.md), которые помогают выполнять обнаружение и диагностику.
* Ознакомьтесь с функциями [поиска по журналам и запросов к журналам](../azure-monitor/logs/log-query-overview.md), которые являются частью журналов Azure Monitor.
* Более подробный обзор журналов Azure Monitor и их предложений см. в статье [что такое Azure Monitor журналов?](../azure-monitor/overview.md).
