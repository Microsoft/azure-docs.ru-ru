---
title: Рекомендации по управлению кластером — Azure HDInsight
description: Ознакомьтесь с рекомендациями по управлению кластерами HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 12/02/2019
ms.openlocfilehash: 7ab113a9c7ab03a477f387c5599b55d67c99c6fd
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102433902"
---
# <a name="hdinsight-cluster-management-best-practices"></a>Рекомендации по управлению кластерами HDInsight

Ознакомьтесь с рекомендациями по управлению кластерами HDInsight.

## <a name="how-do-i-create-hdinsight-clusters"></a>Разделы справки создать кластеры HDInsight?

| Параметр | Документы |
|---|---|
| Фабрика данных Azure | [Создание кластеров Apache Hadoop в HDInsight по запросу с помощью Фабрики данных Azure](./hdinsight-hadoop-create-linux-clusters-adf.md) |
| Пользовательский шаблон диспетчер ресурсов | [Создание кластеров Apache Hadoop в HDInsight с помощью шаблонов Resource Manager](./hdinsight-hadoop-create-linux-clusters-arm-templates.md) |
| Шаблоны быстрого запуска | [Шаблоны быстрого запуска HDInsight](https://azure.microsoft.com/resources/templates/?term=hdinsight) |
| Примеры Azure | [Примеры Azure HDInsight](/samples/browse/?products=azure-hdinsight) |
| Портал Azure | [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](./spark/apache-spark-intellij-tool-plugin.md) |
| Azure CLI | [Создание кластеров HDInsight с помощью интерфейса командной строки Azure](./hdinsight-hadoop-create-linux-clusters-azure-cli.md) |
| Azure PowerShell | [Создание кластеров под управлением Linux в HDInsight с помощью Azure PowerShell](./hdinsight-hadoop-create-linux-clusters-azure-powershell.md) |
| cURL | [Создание кластеров Apache Hadoop с помощью REST API Azure](./hdinsight-hadoop-create-linux-clusters-curl-rest.md) |
| Пакеты SDK (.NET, Python, Java) | [.NET](/dotnet/api/overview/azure/hdinsight), [Python](/python/api/overview/azure/hdinsight?preserve-view=true&view=azure-python), [Java](/java/api/overview/azure/hdinsight), [Go](./hdinsight-go-sdk-overview.md) |

> [!Note]
> Если вы создаете кластер и повторно используете имя кластера из ранее созданного кластера, перед созданием кластера дождитесь завершения предыдущего удаления кластера.

## <a name="how-do-i-customize-hdinsight-clusters"></a>Разделы справки настроить кластеры HDInsight?

| Параметр | Документы |
|---|---|
| Действия сценария | [Настройка кластеров Azure HDInsight с помощью действий сценариев](./hdinsight-hadoop-customize-cluster-linux.md) |
| Бутстрэп | [Настройка кластеров HDInsight с помощью начальной загрузки](./hdinsight-hadoop-customize-cluster-bootstrap.md) |
| Внешние метахранилища | [Использование внешних хранилищ метаданных в Azure HDInsight](./hdinsight-use-external-metadata-stores.md) |
| Пользовательская база данных Ambari DB | [Настройка кластеров HDInsight с помощью настраиваемой базы данных Ambari DB](./hdinsight-custom-ambari-db.md) |

## <a name="what-are-some-errors-i-might-face-when-creating-clusters"></a>Какие ошибки я могу столкнуться при создании кластеров?

| Ошибка | Дополнительные сведения |
|---|---|
| Нет квоты | Существуют квоты на количество ядер, которые можно создать для подписки в каждом регионе. Дополнительные сведения см. в разделе [планирование емкости: квоты](./hdinsight-capacity-planning.md). |
| Больше нет доступных IP-адресов | Каждая виртуальная сеть имеет ограниченное количество IP-адресов. При создании кластера HDInsight каждый узел (включая узлы Zookeeper и шлюза) использует некоторые из этих выделенных IP-адресов. Если все IP-адреса используются, возникнет эта ошибка.  |
| Правила группы безопасности сети (NSG) не разрешают взаимодействие с поставщиками ресурсов HDInsight | Если вы используете группы безопасности сети или определяемые пользователем маршруты (определяемые пользователем маршруты) для управления входящим трафиком к кластеру HDInsight, необходимо убедиться, что кластер может взаимодействовать с критически важными службами работоспособности и управления Azure. Дополнительные сведения см. в статье [теги службы группы безопасности сети (NSG) для Azure HDInsight](./hdinsight-service-tags.md) . |
| Повторное использование имени кластера | При использовании имени кластера, которое использовалось ранее, необходимо подождать X минут перед повторной созданием кластера. В противном случае вы увидите сообщение о том, что ресурс уже существует. |

## <a name="how-do-i-manage-running-hdinsight-clusters"></a>Разделы справки управлять работающими кластерами HDInsight?

| Параметр | Документы |
|---|---|
| Автомасштабирование | [Автоматическое масштабирование кластеров Azure HDInsight](./hdinsight-autoscale-clusters.md) |
| Масштабирование вручную | [Масштабирование кластеров Azure HDInsight](./hdinsight-scaling-best-practices.md) |
| Мониторинг с помощью Ambari| [Мониторинг производительности кластера в Azure HDInsight](./hdinsight-key-scenarios-to-monitor.md) |
| Мониторинг с помощью журналов Azure Monitor | [Мониторинг кластеров HDInsight с помощью журналов Azure Monitor](./hdinsight-hadoop-oms-log-analytics-tutorial.md) |
| Проблемы службы, плановое техническое обслуживание, рекомендации по безопасности & работоспособности | [Подписка на предупреждения о работоспособности службы, относящиеся к подписке](../service-health/alerts-activity-log-service-notifications-portal.md) |


## <a name="how-do-i-check-on-deleted-hdinsight-clusters"></a>Разделы справки проверить удаленные кластеры HDInsight?

### <a name="azure-monitor-logs"></a>Журналы Azure Monitor

Для мониторинга удаленных кластеров можно использовать следующий запрос с журналами Azure Monitor.

```loganalytics
AzureActivity
| where ResourceProvider == "Microsoft.HDInsight" and (OperationName == "Create or Update Cluster" or OperationName == "Delete Cluster") and ActivityStatus == "Succeeded"
```

## <a name="next-steps"></a>Дальнейшие действия

* [Планирование загрузки кластеров HDInsight](./hdinsight-capacity-planning.md)
* [Каковы используемые по умолчанию и рекомендуемые конфигурации узлов для Azure HDInsight?](./hdinsight-supported-node-configuration.md)
