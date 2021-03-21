---
title: Управление ресурсами службы "Управляемый экземпляр Azure для ресурсов Apache Cassandra" с помощью Azure CLI
description: Сведения об общих командах для автоматизации управления Управляемый экземпляр Azure для Apache Cassandra с помощью Azure CLI.
author: TheovanKraay
ms.service: managed-instance-apache-cassandra
ms.topic: how-to
ms.date: 03/15/2021
ms.author: thvankra
ms.openlocfilehash: 3e44625d23a302c58ea065a4fc3ecec5605e60b9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103564530"
---
# <a name="manage-azure-managed-instance-for-apache-cassandra-resources-using-azure-cli-preview"></a>Управление Управляемый экземпляр Azure для ресурсов Apache Cassandra с помощью Azure CLI (Предварительная версия)

В этой статье описаны распространенные команды для автоматизации управления Управляемый экземпляр Azure для кластеров Apache Cassandra с помощью Azure CLI.

> [!IMPORTANT]
> Служба "Управляемый экземпляр Azure для Apache Cassandra" сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

> [!IMPORTANT]
> Для работы с этой статьей требуется Azure CLI версии 2.17.1 или выше. Если вы используете Azure Cloud Shell, последняя версия уже установлена.
>
> Управление Управляемый экземпляр Azure для ресурсов Apache Cassandra невозможно переименовать, так как это нарушает работу Azure Resource Manager с URI ресурсов.

## <a name="azure-managed-instance-for-apache-cassandra-clusters"></a>Управляемый экземпляр Azure для кластеров Apache Cassandra

В следующих разделах показано, как управлять Управляемый экземпляр Azure для кластеров Apache Cassandra, включая:

* [Создание кластера управляемых экземпляров](#create-cluster)
* [Удаление кластера управляемых экземпляров](#delete-cluster)
* [Получение сведений о кластере](#get-cluster-details)
* [Получение состояния узла кластера](#get-cluster-status)
* [Вывод списка кластеров по группе ресурсов](#list-clusters-resource-group)
* [Список кластеров по ИДЕНТИФИКАТОРу подписки](#list-clusters-subscription)

### <a name="create-a-managed-instance-cluster"></a><a id="create-cluster"></a>Создание кластера с управляемым экземпляром

Создайте Управляемый экземпляр Azure для кластера Apache Cassandra с помощью команды [AZ Managed-Cassandra Cluster Create](/cli/azure/ext/cosmosdb-preview/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_cluster_create) .

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
location='West US'
delegatedManagementSubnetId='/subscriptions/<subscription id>/resourceGroups/customer-vnet-rg/providers/Microsoft.Network/virtualNetworks/customer-vnet/subnets/management'
initialCassandraAdminPassword='myPassword'

# You can override the cluster name if the original name is not legal for an Azure resource.
# overrideClusterName='ClusterNameIllegalForAzureResource'
# the default Cassandra version is v3.11

az managed-cassandra cluster create \
    --cluster-name $clusterName \
    --resource-group $resourceGroupName \
    --location $location \
    --delegated-management-subnet-id $delegatedManagementSubnetId \
    --initial-cassandra-admin-password $initialCassandraAdminPassword \
```

### <a name="delete-a-managed-instance-cluster"></a><a id="delete-cluster"></a>Удаление кластера управляемых экземпляров

Удалите кластер с помощью команды [AZ Managed-Cassandra кластера Delete](/cli/azure/ext/cosmosdb-preview/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_cluster_delete) :

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'

az managed-cassandra cluster delete \
    --cluster-name $clusterName \
    --resource-group $resourceGroupName
```

### <a name="get-the-cluster-details"></a><a id="get-cluster-details"></a>Получение сведений о кластере

Получите сведения о кластере с помощью команды [AZ Managed-Cassandra Cluster](/cli/azure/ext/cosmosdb-preview/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_cluster_show) .

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'

az managed-cassandra cluster show \
    --cluster-name $clusterName \
    --resource-group $resourceGroupName
```

### <a name="get-the-cluster-node-status"></a><a id="get-cluster-status"></a>Получение состояния узла кластера

Получите сведения о кластере с помощью команды [AZ Managed-Cassandra кластера-Status](/cli/azure/ext/cosmosdb-preview/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_cluster_node_status) :

```azurecli-interactive
clusterName='cassandra-hybrid-cluster'
resourceGroupName='MyResourceGroup'

az managed-cassandra cluster node-status \
    --cluster-name $clusterName \
    --resource-group $resourceGroupName
```

### <a name="list-the-clusters-by-resource-group"></a><a id="list-clusters-resource-group"></a>Список кластеров по группе ресурсов

Выведите список кластеров по группе ресурсов с помощью команды [AZ Managed-Cassandra Cluster List](/cli/azure/ext/cosmosdb-preview/managed-cassandra/cluster?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_cluster_list) :

```azurecli-interactive
subscriptionId='MySubscriptionId'
resourceGroupName='MyResourceGroup'

az managed-cassandra cluster list\
    --resource-group $resourceGroupName
```

### <a name="list-clusters-by-subscription-id"></a><a id="list-clusters-subscription"></a>Список кластеров по ИДЕНТИФИКАТОРу подписки

Вычислите кластеры по ИДЕНТИФИКАТОРу подписки с помощью команды [AZ Managed-Cassandra Cluster List](/cli/azure/ext/cosmosdb-preview/managed-cassandra?view=azure-cli-latest&preserve-view=true) :

```azurecli-interactive
# set your subscription id
az account set -s <subscription id>

az managed-cassandra cluster list
```

## <a name="the-managed-instance-datacenters"></a><a id="managed-instance-datacenter"></a>Центры обработки данных управляемого экземпляра

В следующих разделах показано, как управлять Управляемый экземпляр Azure для центров обработки данных Apache Cassandra, включая:

* [Создание центра обработки данных](#create-datacenter)
* [Удаление центра обработки данных](#delete-datacenter)
* [Получение сведений о центре обработки данных](#get-datacenter-details)
* [Обновление или масштабирование центра обработки данных](#update-datacenter)
* [Получение центров обработки данных в кластере](#get-datacenters-cluster)

### <a name="create-a-datacenter"></a><a id="create-datacenter"></a>Создание центра обработки данных

Создайте центр обработки данных с помощью команды [AZ Managed-Cassandra Datacenter Create](/cli/azure/ext/cosmosdb-preview/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_datacenter_create) :

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'
dataCenterLocation='eastus2'
delegatedSubnetId='/subscriptions/<Subscription_ID>/resourceGroups/customer-vnet-rg/providers/Microsoft.Network/virtualNetworks/customer-vnet/subnets/dc1-subnet'

az managed-cassandra datacenter create \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName \
    --data-center-location $dataCenterLocation \
    --delegated-subnet-id $delegatedSubnetId \
    --node-count 3 
```

### <a name="delete-a-datacenter"></a><a id="delete-datacenter"></a>Удаление центра обработки данных

Удалите центр обработки данных с помощью команды [AZ Managed-Cassandra Datacenter Delete](/cli/azure/ext/cosmosdb-preview/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_datacenter_delete) :

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'

az managed-cassandra datacenter delete \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName 
```

### <a name="get-datacenter-details"></a><a id="get-datacenter-details"></a>Получение сведений о центре обработки данных

Получите сведения о центре обработки данных с помощью команды [AZ Managed-Cassandra Datacenter демонстрация](/cli/azure/ext/cosmosdb-preview/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_datacenter_show) .

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'

az managed-cassandra datacenter show \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName 
```

### <a name="update-or-scale-a-datacenter"></a><a id="update-datacenter"></a>Обновление или масштабирование центра обработки данных

Обновите или масштабируйте центр обработки данных (для масштабирования значения nodeCount изменения) с помощью команды [AZ Managed-Cassandra Datacenter Update](/cli/azure/ext/cosmosdb-preview/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_datacenter_update) :

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'
dataCenterName='dc1'
dataCenterLocation='eastus'
delegatedSubnetId= '/subscriptions/<Subscription_ID>/resourceGroups/customer-vnet-rg/providers/Microsoft.Network/virtualNetworks/customer-vnet/subnets/dc1-subnet'

az managed-cassandra datacenter update \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName \
    --data-center-name $dataCenterName \
    --node-count 13 
```

### <a name="get-the-datacenters-in-a-cluster"></a><a id="get-datacenters-cluster"></a>Получение центров обработки данных в кластере

Получение центров обработки данных в кластере с помощью команды [AZ Managed-Cassandra Datacenter List](/cli/azure/ext/cosmosdb-preview/managed-cassandra/datacenter?view=azure-cli-latest&preserve-view=true#ext_cosmosdb_preview_az_managed_cassandra_datacenter_list) .

```azurecli-interactive
resourceGroupName='MyResourceGroup'
clusterName='cassandra-hybrid-cluster'

az managed-cassandra datacenter list \
    --resource-group $resourceGroupName \
    --cluster-name $clusterName
```

## <a name="next-steps"></a>Следующие шаги

* [Создание кластера управляемых экземпляров на портале Azure](create-cluster-portal.md)
* [Развертывание управляемого кластера Apache Spark с Azure Databricks](deploy-cluster-databricks.md)
