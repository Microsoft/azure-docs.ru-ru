---
title: Запрос журналов Azure Monitor для мониторинга кластеров Azure HDInsight
description: Узнайте, как выполнять запросы к журналам Azure Monitor для мониторинга заданий, выполняемых в кластере HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/02/2019
ms.openlocfilehash: 3cf97039983ecec44a7c3a32e178fdcf9f9c45ff
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104872189"
---
# <a name="query-azure-monitor-logs-to-monitor-hdinsight-clusters"></a>Мониторинг кластеров HDInsight с помощью запросов к журналам Azure Monitor

Ознакомьтесь с некоторыми базовыми сценариями использования журналов Azure Monitor для мониторинга кластеров Azure HDInsight.

* [Анализ метрик кластера HDInsight](#analyze-hdinsight-cluster-metrics)
* [Создание оповещений о событиях](#create-alerts-for-tracking-events).

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Предварительные требования

Вы должны настроить кластер HDInsight для использования журналов Azure Monitor и добавить в нее решения для мониторинга Azure Monitor журналов, относящиеся к кластеру HDInsight. Инструкции см. в статье [Использование журналов Azure Monitor с кластерами HDInsight](hdinsight-hadoop-oms-log-analytics-tutorial.md).

## <a name="analyze-hdinsight-cluster-metrics"></a>Анализ метрик кластера HDInsight

Узнайте, как выполнить поиск определенных метрик для кластера HDInsight.

1. Откройте рабочую область Log Analytics, которая связана с вашим кластером HDInsight на портале Azure.
1. В разделе **Общие** щелкните **Журналы**.
1. Введите следующий запрос в поле поиска, чтобы найти все метрики всех доступных метрик для всех кластеров HDInsight, настроенных для использования журналов Azure Monitor, а затем выберите **выполнить**. Просмотрите результаты.

    ```kusto
    search *
    ```

    :::image type="content" source="./media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-search-all-metrics.png" alt-text="Apache Ambari Analytics Поиск всех метрик":::

1. В меню слева выберите вкладку **Фильтр** .

1. В разделе **тип** выберите **пульс**. Затем выберите **применить & выполнить**.

    :::image type="content" source="./media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-search-specific-metrics.png" alt-text="Поиск конкретных метрик в log Analytics":::

1. Обратите внимание, что запрос в текстовом поле изменяется на:

    ```kusto
    search *
    | where Type == "Heartbeat"
    ```

1. Более глубокие знания можно получить с помощью параметров, доступных в меню слева. Например:

   - Чтобы просмотреть журналы из определенного узла, выполните следующие действия.

     :::image type="content" source="./media/hdinsight-hadoop-oms-log-analytics-use-queries/log-analytics-specific-node.png" alt-text="Поиск конкретных ошибок output1":::

   - Чтобы просмотреть журналы в определенное время:

     :::image type="content" source="./media/hdinsight-hadoop-oms-log-analytics-use-queries/log-analytics-specific-time.png" alt-text="Поиск конкретных ошибок Output2":::

1. Выберите **применить & выполнить** и проверьте результаты. Также обратите внимание, что запрос был обновлен до:

    ```kusto
    search *
    | where Type == "Heartbeat"
    | where (Computer == "zk2-myhado") and (TimeGenerated == "2019-12-02T23:15:02.69Z" or TimeGenerated == "2019-12-02T23:15:08.07Z" or TimeGenerated == "2019-12-02T21:09:34.787Z")
    ```

### <a name="additional-sample-queries"></a>Дополнительные примеры запросов

Пример запроса, основанный на среднем значении ресурсов, которые используются 10-минутным интервалом, разбитым по имени кластера:

```kusto
search in (metrics_resourcemanager_queue_root_default_CL) * 
| summarize AggregatedValue = avg(UsedAMResourceMB_d) by ClusterName_s, bin(TimeGenerated, 10m)
```

Вместо уточнения по среднему количеству используемых ресурсов можно использовать следующий запрос, чтобы уточнить результаты на основе максимального использования ресурсов в 10-минутном интервале (а также 90-го и 95-го процентиля):

```kusto
search in (metrics_resourcemanager_queue_root_default_CL) * 
| summarize ["max(UsedAMResourceMB_d)"] = max(UsedAMResourceMB_d), ["pct95(UsedAMResourceMB_d)"] = percentile(UsedAMResourceMB_d, 95), ["pct90(UsedAMResourceMB_d)"] = percentile(UsedAMResourceMB_d, 90) by ClusterName_s, bin(TimeGenerated, 10m)
```

## <a name="create-alerts-for-tracking-events"></a>Создание оповещений для отслеживания событий

Первым шагом создания оповещения является получение запроса, на основе которого активируется оповещение. Для создания оповещения можно использовать любой запрос.

1. Откройте рабочую область Log Analytics, которая связана с вашим кластером HDInsight на портале Azure.
1. В разделе **Общие** щелкните **Журналы**.
1. Выполните следующий запрос, для которого нужно создать оповещение, а затем выберите **выполнить**.

    ```kusto
    metrics_resourcemanager_queue_root_default_CL | where AppsFailed_d > 0
    ```

    Запрос вернет список запущенных на кластерах HDInsight приложений, в которых возник сбой.

1. Выберите **новое правило генерации оповещений** в верхней части страницы.

    :::image type="content" source="./media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-create-alert-query.png" alt-text="Новое правило генерации оповещений":::

1. В окне **Создать правило** введите запрос и другие сведения, чтобы создать оповещение, а затем щелкните **Создать правило генерации оповещений**.

    :::image type="content" source="./media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-create-alert.png" alt-text="Определите условие оповещения.":::

### <a name="edit-or-delete-an-existing-alert"></a>Изменить или удалить существующее оповещение

1. Откройте рабочую область Log Analytics на портале Azure.

1. В меню слева в разделе **мониторинг** выберите **оповещения**.

1. В верхней части выберите **Управление правилами оповещений**.

1. Выберите оповещение, которое требуется изменить или удалить.

1. Доступны следующие варианты: **Сохранить**, **Отменить**, **Отключить** и **Удалить**.

    :::image type="content" source="media/hdinsight-hadoop-oms-log-analytics-use-queries/hdinsight-log-analytics-edit-alert.png" alt-text="Предупреждение об удалении журнала Azure Monitor HDInsight":::

Дополнительные сведения см. в статье [Создание и просмотр оповещений метрик, а также управление ими с помощью Azure Monitor](../azure-monitor/alerts/alerts-metric.md).

## <a name="see-also"></a>См. также раздел

* [Начало работы с запросами к журналам Azure Monitor](../azure-monitor/logs/get-started-queries.md)
* [Создание пользовательских представлений с помощью конструктора представлений в Azure Monitor](../azure-monitor/visualize/view-designer.md)
