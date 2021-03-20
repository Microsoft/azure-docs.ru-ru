---
title: Мониторинг кластеров Azure HDInsight с помощью журналов Azure Monitor
description: Узнайте, как использовать журналы Azure Monitor для мониторинга заданий, выполняемых в кластере HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020, devx-track-azurecli, devx-track-azurepowershell
ms.date: 05/13/2020
ms.openlocfilehash: b81a2021cabefd9587d448aa236e7e87bc254741
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100590824"
---
# <a name="use-azure-monitor-logs-to-monitor-hdinsight-clusters"></a>Мониторинг кластеров HDInsight с помощью журналов Azure Monitor

Узнайте, как использовать журналы Azure Monitor для мониторинга операций кластера Hadoop в HDInsight. А также узнайте, как добавить решение для мониторинга HDInsight.

[Журналы Azure Monitor](../azure-monitor/logs/log-query-overview.md) — это служба Azure Monitor, которая наблюдает за облачными и локальными средами. Мониторинг помогает поддерживать их доступность и производительность. Эта служба собирает данные, создаваемые ресурсами в облачных и локальных средах, а также другими средствами мониторинга. На основе этих данных служба предоставляет аналитические сведения, используя несколько источников.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Рабочая область Log Analytics. Рабочую область можно представить как уникальную среду журналов Azure Monitor с собственным репозиторием данных, источниками данных и решениями. С инструкциями можно ознакомиться в разделе [Создание рабочей области](../azure-monitor/vm/quick-collect-azurevm.md#create-a-workspace).

* Кластер Azure HDInsight. Сейчас вы можете использовать журналы Azure Monitor со следующими типами кластеров HDInsight:

  * Hadoop
  * HBase
  * Интерактивный запрос
  * Kafka
  * Spark
  * Storm

  Инструкции по созданию кластера HDInsight см. в руководстве по [началу работы с Azure HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md).  

* При использовании PowerShell вам потребуется [модуль Az](/powershell/azure/). Убедитесь, что у вас установлена последняя версия. При необходимости выполните командлет `Update-Module -Name Az`.

* Если вы хотите использовать Azure CLI и еще не установили его, обратитесь к разделу [Установка Azure CLI](/cli/azure/install-azure-cli).

> [!NOTE]  
> Для повышения производительности рекомендуется разместить кластер HDInsight и рабочую область Log Analytics в одном регионе. Журналы Azure Monitor доступны не во всех регионах Azure.

## <a name="enable-azure-monitor-using-the-portal"></a>Включение Azure Monitor с помощью портала

В этом разделе вы настроите существующий кластер HDInsight Hadoop, чтобы использовать рабочую область Azure Log Analytics для мониторинга заданий, журналов отладки и пр.

1. На [портале Azure](https://portal.azure.com/) выберите свой кластер. Кластер откроется на новой странице портала.

1. Слева в разделе **Мониторинг** выберите **Azure Monitor**.

1. В главном представлении в разделе **Интеграция Azure Monitor** выберите **Включить**.

1. В раскрывающемся списке **Выберите рабочую область** выберите существующую рабочую область Log Analytics.

1. Щелкните **Сохранить**.  Параметр сохраняется в течение нескольких секунд.

    ![Включение мониторинга кластеров HDInsight](./media/hdinsight-hadoop-oms-log-analytics-tutorial/azure-portal-monitoring.png "Включение мониторинга кластеров HDInsight")

## <a name="enable-azure-monitor-using-azure-powershell"></a>Включение Azure Monitor с помощью Azure PowerShell

Журналы Azure Monitor можно включить с помощью командлета модуля Az Azure PowerShell [Enable-AzHDInsightMonitoring](/powershell/module/az.hdinsight/enable-azhdinsightmonitoring).

```powershell
# Enter user information
$resourceGroup = "<your-resource-group>"
$cluster = "<your-cluster>"
$LAW = "<your-Log-Analytics-workspace>"
# End of user input

# obtain workspace id for defined Log Analytics workspace
$WorkspaceId = (Get-AzOperationalInsightsWorkspace `
                    -ResourceGroupName $resourceGroup `
                    -Name $LAW).CustomerId

# obtain primary key for defined Log Analytics workspace
$PrimaryKey = (Get-AzOperationalInsightsWorkspace `
                    -ResourceGroupName $resourceGroup `
                    -Name $LAW | Get-AzOperationalInsightsWorkspaceSharedKeys).PrimarySharedKey

# Enables monitoring and relevant logs will be sent to the specified workspace.
Enable-AzHDInsightMonitoring `
    -ResourceGroupName $resourceGroup `
    -Name $cluster `
    -WorkspaceId $WorkspaceId `
    -PrimaryKey $PrimaryKey

# Gets the status of monitoring installation on the cluster.
Get-AzHDInsightMonitoring `
    -ResourceGroupName $resourceGroup `
    -Name $cluster
```

Для отключения используйте командлет [Disable-AzHDInsightMonitoring](/powershell/module/az.hdinsight/disable-azhdinsightmonitoring).

```powershell
Disable-AzHDInsightMonitoring -Name "<your-cluster>"
```

## <a name="enable-azure-monitor-using-azure-cli"></a>Включение Azure Monitor с помощью Azure CLI

Вы можете включить журналы Azure Monitor с помощью `[az hdinsight monitor enable` команды Azure CLI] (/кли/Азуре/хдинсигхт/монитор # AZ-hdinsight-Monitor-Enable).

```azurecli
# set variables
export resourceGroup=RESOURCEGROUPNAME
export cluster=CLUSTERNAME
export LAW=LOGANALYTICSWORKSPACENAME

# Enable the Azure Monitor logs integration on an HDInsight cluster.
az hdinsight monitor enable --name $cluster --resource-group $resourceGroup --workspace $LAW

# Get the status of Azure Monitor logs integration on an HDInsight cluster.
az hdinsight monitor show --name $cluster --resource-group $resourceGroup
```

Для отключения используйте команду [`az hdinsight monitor disable`](/cli/azure/hdinsight/monitor#az-hdinsight-monitor-disable).

```azurecli
az hdinsight monitor disable --name $cluster --resource-group $resourceGroup
```

## <a name="install-hdinsight-cluster-management-solutions"></a>Установка решений по управлению кластерами HDInsight

HDInsight предоставляет решения по управлению кластерами, которые можно добавлять в журналы Azure Monitor. [Решения по управлению](../azure-monitor/insights/solutions.md) расширяют функциональные возможности журналов Azure Monitor, предоставляя дополнительные средства анализа и обработки данных. Эти решения собирают важные метрики производительности из кластеров HDInsight. Кроме того, они предоставляют средства для поиска метрик. Кроме того, они позволяют использовать визуализации и панели мониторинга для большинства типов кластеров, которые поддерживаются в HDInsight. На основе этих метрик можно создавать пользовательские правила мониторинга и оповещения.

Доступные решения HDInsight:

* Мониторинг HDInsight Hadoop
* HDInsight HBase Monitoring
* мониторинг HDInsight Interactive Query;
* Мониторинг Kafka HDInsight
* Мониторинг HDInsight Spark
* Мониторинг HDInsight Storm

Инструкции для решений по управлению см. в разделе [Решения по управлению в Azure](../azure-monitor/insights/solutions.md#install-a-monitoring-solution). Чтобы поэкспериментировать, установите решение для мониторинга Hadoop в HDInsight. После установки вы увидите элемент **HDInsightHadoop** в разделе **Сводка**. Выберите элемент **HDInsightHadoop**. Решение HDInsightHadoop будет выглядеть примерно следующим образом:

![Решение HDInsight для мониторинга](media/hdinsight-hadoop-oms-log-analytics-tutorial/hdinsight-oms-hdinsight-hadoop-monitoring-solution.png)

Так как это новый кластер, в отчете не будут отображаться никакие действия.

## <a name="configuring-performance-counters"></a>Настройка счетчиков производительности

Azure Monitor поддерживает сбор и анализ метрик производительности для узлов в кластере. Дополнительные сведения см. в разделе [Источники данных о производительности Windows и Linux в Azure Monitor](../azure-monitor/agents/data-sources-performance-counters.md#linux-performance-counters).

## <a name="cluster-auditing"></a>Аудит кластеров

HDInsight поддерживает аудит кластеров с помощью журналов Azure Monitor, импортируя следующие типы журналов:

* `log_gateway_audit_CL` — эта таблица содержит журналы аудита с узлов шлюза кластера, которые показывают успешные и неудачные попытки входа;
* `log_auth_CL` — эта таблица содержит журналы SSH с успешными и неудачными попытками входа;
* `log_ambari_audit_CL` — эта таблица содержит журналы аудита из Ambari;
* `log_ranger_audti_CL` — эта таблица содержит журналы аудита из Apache Ranger в кластерах ESP.

## <a name="next-steps"></a>Дальнейшие действия

* [Мониторинг кластеров HDInsight с помощью запросов к журналам Azure Monitor](hdinsight-hadoop-oms-log-analytics-use-queries.md)
* [Как отслеживать доступность кластера с помощью журналов Apache Ambari и Azure Monitor](./hdinsight-cluster-availability.md)