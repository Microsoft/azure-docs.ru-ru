---
title: Управление кластерами Azure HDInsight с помощью Azure CLI
description: Узнайте, как использовать Azure CLI для управления кластерами Azure HDInsight. К типам кластеров относятся Apache Hadoop, Spark, HBase, Kafka, Интерактивные запросы и службы ML.
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,hdiseo17may2017, devx-track-azurecli
ms.date: 02/26/2020
ms.openlocfilehash: b17c5a2abc036c16ff3ce36b81428f9149e36b4b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98942853"
---
# <a name="manage-azure-hdinsight-clusters-using-azure-cli"></a>Управление кластерами Azure HDInsight с помощью Azure CLI

[!INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

Узнайте, как использовать [Azure CLI](/cli/azure/) для управления кластерами Azure HDInsight. Azure CLI — это кроссплатформенный интерфейс командной строки от Майкрософт для управления ресурсами Azure.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Azure CLI. Если вы еще не установили Azure CLI, обратитесь к статье [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

* Кластер Apache Hadoop в HDInsight. Ознакомьтесь со статьей [Краткое руководство. Использование Apache Hadoop и Apache Hive в Azure HDInsight с шаблоном Resource Manager](hadoop/apache-hadoop-linux-tutorial-get-started.md).

## <a name="connect-to-azure"></a>Подключение к Azure

Войдите в подписку Azure. Если вы планируете использовать Azure Cloud Shell, щелкните **Попробовать** в правом верхнем углу блока кода. В противном случае введите следующую команду:

```azurecli-interactive
az login

# If you have multiple subscriptions, set the one to use
# az account set --subscription "SUBSCRIPTIONID"
```

## <a name="list-clusters"></a>список кластеров

Список кластеров можно получить с помощью команды [AZ hdinsight List](/cli/azure/hdinsight#az-hdinsight-list) . Измените приведенные ниже команды, заменив `RESOURCE_GROUP_NAME` именем своей группы ресурсов, а затем введите команды:

```azurecli-interactive
# List all clusters in the current subscription
az hdinsight list

# List only cluster name and its resource group
az hdinsight list --query "[].{Cluster:name, ResourceGroup:resourceGroup}" --output table

# List all cluster for your resource group
az hdinsight list --resource-group RESOURCE_GROUP_NAME

# List all cluster names for your resource group
az hdinsight list --resource-group RESOURCE_GROUP_NAME --query "[].{clusterName:name}" --output table
```

## <a name="show-cluster"></a>Отображение кластеров

Чтобы отобразить сведения об указанном кластере, используйте команду [AZ hdinsight демонстрация](/cli/azure/hdinsight#az-hdinsight-show) . Измените приведенную ниже команду, заменив `RESOURCE_GROUP_NAME` и `CLUSTER_NAME` соответствующей информацией, а затем введите команду:

```azurecli-interactive
az hdinsight show --resource-group RESOURCE_GROUP_NAME --name CLUSTER_NAME
```

## <a name="delete-clusters"></a>Удаление кластеров

Чтобы удалить указанный кластер, используйте команду [AZ hdinsight Delete](/cli/azure/hdinsight#az-hdinsight-delete) . Измените приведенную ниже команду, заменив `RESOURCE_GROUP_NAME` и `CLUSTER_NAME` соответствующей информацией, а затем введите команду:

```azurecli-interactive
az hdinsight delete --resource-group RESOURCE_GROUP_NAME --name CLUSTER_NAME
```

Можно также удалить кластер, удалив группу ресурсов, которая содержит этот кластер. Обратите внимание, что это приведет к удалению всех ресурсов в группе, включая учетную запись хранения по умолчанию.

```azurecli-interactive
az group delete --name RESOURCE_GROUP_NAME
```

## <a name="scale-clusters"></a>Масштабирование кластеров

Чтобы изменить размер указанного кластера HDInsight до указанного размера, используйте команду [AZ hdinsight изменить](/cli/azure/hdinsight#az-hdinsight-resize) размер. Измените приведенную ниже команду, заменив `RESOURCE_GROUP_NAME` и `CLUSTER_NAME` соответствующими данными. Замените `WORKERNODE_COUNT` требуемым числом рабочих узлов для кластера. Дополнительные сведения о масштабировании кластеров см. в статье [масштабирование кластеров HDInsight](./hdinsight-scaling-best-practices.md). Введите команду:

```azurecli-interactive
az hdinsight resize --resource-group RESOURCE_GROUP_NAME --name CLUSTER_NAME --workernode-count WORKERNODE_COUNT
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как выполнять различные задачи администрирования кластера HDInsight. Дополнительные сведения см. в следующих статьях:

* [Управление кластерами Apache Hadoop в HDInsight с помощью портала Azure](hdinsight-administer-use-portal-linux.md)
* [Управление кластерами Hadoop в HDInsight с помощью Azure PowerShell](hdinsight-administer-use-powershell.md)
* [Приступая к работе с Azure HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Приступая к работе с Azure CLI](/cli/azure/get-started-with-azure-cli)
