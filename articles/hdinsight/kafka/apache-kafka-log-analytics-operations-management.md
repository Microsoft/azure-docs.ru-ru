---
title: Журналы Azure Monitor для Apache Kafka Azure HDInsight
description: Узнайте, как использовать журналы Azure Monitor для анализа журналов из кластера Apache Kafka в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 02/17/2020
ms.openlocfilehash: 51301bd38bf0700ce42ef33a47b9e763da8d4ed6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100595289"
---
# <a name="analyze-logs-for-apache-kafka-on-hdinsight"></a>Анализ журналов для Apache Kafka в HDInsight

Узнайте, как использовать журналы Azure Monitor для анализа журналов, созданных Apache Kafka в HDInsight.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="logs-location"></a>Расположение журналов

Журналы Apache Kafka в кластере находятся в папке `/var/log/kafka` . Журналы Kafka не сохраняются и не сохраняются в течение жизненного цикла кластера, независимо от того, используются ли управляемые диски. В следующей таблице показаны доступные журналы.

|Журнал |Описание |
|---|---|
|Kafka. out|stdout и stderr процесса Kafka. В этом файле находятся журналы запуска и завершения работы Kafka.|
|Server. log|Основной журнал сервера Kafka. Все журналы Kafka Broker находятся здесь.|
|контроллер. log|Журналы контроллера, если брокер работает как контроллер.|
|StateChange. log|Все события изменения состояния в брокеры регистрируются в этом файле.|
|Кафка-ГК. log|Статистика сбора мусора Kafka.|

## <a name="enable-azure-monitor-logs-for-apache-kafka"></a>Включение Azure Monitor журналов для Apache Kafka

Действия по включению журналов Azure Monitor для HDInsight одинаковы для всех кластеров HDInsight. Чтобы узнать, как создать и настроить необходимые службы, воспользуйтесь ссылками ниже.

1. Создание рабочей области Log Analytics. Дополнительные сведения см. в разделе [журналы в Azure Monitor](../../azure-monitor/logs/data-platform-logs.md) документе.

2. Создание Kafka в кластере HDInsight. Дополнительные сведения см. в документе [Приступая к работе с Apache Kafka (предварительная версия) в HDInsight](apache-kafka-get-started.md).

3. Настройте кластер Kafka для использования журналов Azure Monitor. Дополнительные сведения см. в разделе [Использование журналов Azure Monitor для мониторинга документа HDInsight](../hdinsight-hadoop-oms-log-analytics-tutorial.md) .

> [!IMPORTANT]  
> Доступ к данным для журналов Azure Monitor может занять около 20 минут.

## <a name="query-logs"></a>Журналы запросов

1. В [портал Azure](https://portal.azure.com)выберите рабочую область log Analytics.

2. В меню слева в разделе **Общие** выберите **журналы**. Здесь можно выполнять поиск данных, собранных из Kafka. Введите запрос в окне запроса и нажмите кнопку **выполнить**. Ниже приведены некоторые примеры поисковых запросов:

* Использование диска:

    ```kusto
    Perf
    | where ObjectName == "Logical Disk" and CounterName == "Free Megabytes" and InstanceName == "_Total" and ((Computer startswith_cs "hn" and Computer contains_cs "-") or (Computer startswith_cs "wn" and Computer contains_cs "-")) 
    | summarize AggregatedValue = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
    ```

* Загрузка ЦП:

    ```kusto
    Perf 
    | where CounterName == "% Processor Time" and InstanceName == "_Total" and ((Computer startswith_cs "hn" and Computer contains_cs "-") or (Computer startswith_cs "wn" and Computer contains_cs "-")) 
    | summarize AggregatedValue = avg(CounterValue) by Computer, bin(TimeGenerated, 1h)
    ```

* Входящих сообщений в секунду: (замените `your_kafka_cluster_name` именем кластера.)

    ```kusto
    metrics_kafka_CL 
    | where ClusterName_s == "your_kafka_cluster_name" and InstanceName_s == "kafka-BrokerTopicMetrics-MessagesInPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_MessagesInPerSec_Count_value_d) by HostName_s, bin(TimeGenerated, 1h)
    ```

* Входящих байт в секунду: (замените `wn0-kafka` именем узла рабочего узла.)

    ```kusto
    metrics_kafka_CL 
    | where HostName_s == "wn0-kafka" and InstanceName_s == "kafka-BrokerTopicMetrics-BytesInPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_BytesInPerSec_Count_value_d) by bin(TimeGenerated, 1h)
    ```

* Исходящих байт в секунду: (замените `your_kafka_cluster_name` именем кластера.)

    ```kusto
    metrics_kafka_CL 
    | where ClusterName_s == "your_kafka_cluster_name" and InstanceName_s == "kafka-BrokerTopicMetrics-BytesOutPerSec-Count" 
    | summarize AggregatedValue = avg(kafka_BrokerTopicMetrics_BytesOutPerSec_Count_value_d) by bin(TimeGenerated, 1h)
    ```

    Кроме того, вы можете ввести `*` для поиска всех типов данных журнала. В настоящее время для запросов доступны следующие журналы:

    | Тип журнала | Описание |
    | ---- | ---- |
    | log\_kafkaserver\_CL | Брокер Kafka, server.log |
    | log\_kafkacontroller\_CL | Брокер Kafka, controller.log |
    | metrics\_kafka\_CL | Метрики Kafka JMX |

    ![Использование ЦП в службе log Analytics для Apache Kafka](./media/apache-kafka-log-analytics-operations-management/apache-kafka-cpu-usage.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Azure Monitor см. в статьях [обзор Azure Monitor](../../azure-monitor/overview.md)и [запрос Azure Monitor журналов для мониторинга кластеров HDInsight](../hdinsight-hadoop-oms-log-analytics-use-queries.md).

Дополнительные сведения о работе с Apache Kafka см. в следующих документах:

* [Зеркальное отображение Apache Kafka в кластерах HDInsight](apache-kafka-mirroring.md)
* [Увеличение масштаба Apache Kafka в HDInsight](apache-kafka-scalability.md)
* [Использование потоковой передачи Apache Spark (DStream) с Apache Kafka](../hdinsight-apache-spark-with-kafka.md)
* [Использование структурированной потоковой передачи Apache Spark с Apache Kafka](../hdinsight-apache-kafka-spark-structured-streaming.md)