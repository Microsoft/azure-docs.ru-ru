---
title: Мониторинг контейнеров с помощью журналов Azure Monitor
description: Используйте журналы Azure Monitor для мониторинга контейнеров, работающих в кластерах Service Fabric Azure.
author: srrengar
ms.topic: conceptual
ms.date: 02/25/2019
ms.author: srrengar
ms.openlocfilehash: 6539815875b87a0d0f525d7e89464fa7d2505746
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100570200"
---
# <a name="monitor-containers-with-azure-monitor-logs"></a>Мониторинг контейнеров с помощью журналов Azure Monitor
 
В этой статье рассматриваются шаги, необходимые для настройки решения для мониторинга контейнеров Azure Monitor журналов. См. дополнительные сведения о [настройке кластера для сбора событий контейнера](service-fabric-tutorial-monitoring-wincontainers.md). 

[!INCLUDE [log-analytics-agent-note.md](../../includes/log-analytics-agent-note.md)]

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="set-up-the-container-monitoring-solution"></a>Настройка решения для мониторинга контейнера

> [!NOTE]
> Необходимо иметь Azure Monitor журналов, настроенных для кластера, а также агент Log Analytics, развернутый на узлах. В противном случае выполните действия, описанные в разделе [Настройка журналов Azure Monitor](service-fabric-diagnostics-oms-setup.md) и сначала [Добавьте агент log Analytics в кластер](service-fabric-diagnostics-oms-agent.md) .

1. После настройки кластера с Azure Monitor журналами и агентом Log Analytics разверните контейнеры. Прежде чем перейти к следующему шагу, подождите, пока ваши контейнеры развернутся.

2. Найдите *Решение для мониторинга контейнеров* в Azure Marketplace и щелкните ресурс **Решения для мониторинга контейнеров** в категории "Мониторинг и управление".

    ![Добавление решения "Контейнеры"](./media/service-fabric-diagnostics-event-analysis-oms/containers-solution.png)

3. Создайте решение внутри рабочей области, созданной для кластера. При таком изменении агент автоматически активируется для запуска сбора данных Docker в контейнеры. Приблизительно через 15 минут в решении появятся входящие журналы и статистика, как показано на рисунке ниже.

    ![Базовая панель мониторинга Log Analytics](./media/service-fabric-diagnostics-event-analysis-oms/oms-containers-dashboard.png)

Агент включает сбор нескольких журналов, зависящих от контейнера, которые могут запрашиваться в Azure Monitor журналах или использоваться для визуализации индикаторов производительности. Собираются следующие типы журналов.

* ContainerInventory — отображает информацию о местонахождении контейнера, имени и образах.
* ContainerImageInventory — информация о развернутых образах, в том числе их идентификаторы или размеры.
* ContainerLog — журналы конкретных ошибок, журналы Docker (StdOut и т. д.) и другие записи.
* ContainerServiceLog — выполнявшиеся команды управляющей программы Docker.
* Perf — счетчики производительности контейнера, в том числе ЦП, память, сетевой трафик, операции ввода и вывода на диске и пользовательские метрики с компьютеров узла.



## <a name="next-steps"></a>Дальнейшие действия
* Дополнительные сведения о [решении "контейнеры журналов Azure Monitor](../azure-monitor/containers/containers.md)".
* Дополнительные сведения об оркестрации контейнера в Service Fabric см. в статье [Service Fabric и контейнеры](service-fabric-containers-overview.md)
* Ознакомьтесь с функциями [поиска по журналам и запросов к журналам](../azure-monitor/logs/log-query-overview.md), которые являются частью журналов Azure Monitor
* Настройка журналов Azure Monitor для настройки правил [автоматического оповещения](../azure-monitor/alerts/alerts-overview.md) , помогающих в обнаружении и диагностике
