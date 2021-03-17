---
title: Краткое руководство. Использование интерфейса командной строки для создания Управляемого экземпляра Azure для кластера Apache Cassandra
description: Следуйте инструкциям этого краткого руководства, чтобы создать Управляемый экземпляр Azure для кластера Apache Cassandra с помощью Azure CLI.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: quickstart
ms.date: 03/02/2021
ms.openlocfilehash: 86fa7e2e45dacb86b6601b699dca46b1b909fd08
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102424705"
---
# <a name="quickstart-create-an-azure-managed-instance-for-apache-cassandra-cluster-using-azure-cli-preview"></a>Краткое руководство. Создание Управляемого экземпляра Azure для кластера Apache Cassandra с помощью Azure CLI (предварительная версия)

Служба "Управляемый экземпляр Azure для Apache Cassandra" позволяет автоматизировать операции развертывания и масштабирования для управляемых решений Apache Cassandra с открытым кодом для центров обработки данных. Эта служба помогает ускорить гибридные сценарии и сократить текущее обслуживание.

> [!IMPORTANT]
> Служба "Управляемый экземпляр Azure для Apache Cassandra" сейчас предоставляется в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этом кратком руководстве описывается, как использовать команды Azure CLI для создания кластера с Управляемым экземпляром Azure для Apache Cassandra. Кроме того, в нем описано, как создавать центр обработки данных, а также увеличивать и уменьшать масштаб узлов в центре обработки данных.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

* Для работы с этой статьей потребуется Azure CLI 2.12.1 или более поздней версии. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

* [Виртуальная сеть Azure](../virtual-network/virtual-networks-overview.md) с подключением к собственной или локальной среде. Дополнительные сведения о подключении локальных сред к Azure см. в статье [Подключение локальной сети к Azure](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/).

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.


## <a name="create-a-managed-instance-cluster"></a><a id="create-cluster"></a>Создание кластера с управляемым экземпляром

1. Войдите на [портал Azure](https://portal.azure.com/)

1. Укажите идентификатор подписки в Azure CLI:

   ```azurecli-interactive
   az account set -s <Subscription_ID>
   ```

1. Затем создайте виртуальную сеть с выделенной подсетью в группе ресурсов:

   ```azurecli-interactive
   az network vnet create -n <VNet_Name> -l eastus2 -g <Resource_Group_Name> --subnet-name <Subnet Name>
   ```

1. Примените несколько специальных разрешений к виртуальной сети и подсети, которые необходимы для управляемого экземпляра. Выполните команду `az role assignment create`, заменив `<subscription ID>`, `<resource group name>`, `<VNet name>` и `<subnet name>` следующими значениями:

   ```azurecli-interactive
   az role assignment create --assignee e5007d2c-4b13-4a74-9b6a-605d99f03501 --role 4d97b98b-1d4f-4787-a291-c67834d212e7 --scope /subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.Network/virtualNetworks/<VNet name>/subnets/<subnet name>
   ```

   > [!NOTE]
   > Значения `assignee` и `role` в предыдущей команде являются фиксированными идентификаторами субъекта-службы и роли соответственно.

1. Затем создайте кластер в только что созданной виртуальной сети. Выполните следующую команду и убедитесь, что используете значение `Resource ID`, полученное в предыдущей команде, в качестве значения переменной `delegatedManagementSubnetId`:

   ```azurecli-interactive
   resourceGroupName='<Resource_Group_Name>'
   clusterName='<Cluster_Name>'
   location='eastus2'
   delegatedManagementSubnetId='<Resource_ID>'
   initialCassandraAdminPassword='myPassword'
    
   az managed-cassandra cluster create \
      --cluster-name $clusterName \
      --resource-group $resourceGroupName \
      --location $location \
      --delegated-management-subnet-id $delegatedManagementSubnetId \
      --initial-cassandra-admin-password $initialCassandraAdminPassword \
      --debug
   ```

1. Наконец, создайте центр обработки данных для кластера с тремя узлами:

   ```azurecli-interactive
   dataCenterName='dc1'
   dataCenterLocation='eastus2'
   delegatedSubnetId='<Resource_ID>'
    
   az managed-cassandra datacenter create \
      --resource-group $resourceGroupName \
      --cluster-name $clusterName \
      --data-center-name $dataCenterName \
      --data-center-location $dataCenterLocation \
      --delegated-subnet-id $delegatedSubnetId \
      --node-count 3 
   ```

1. После создания центра обработки данных, если в нем требуется вертикально увеличить или вертикально уменьшить масштаб узлов, выполните следующую команду. Измените значение `node-count` параметра на требуемое значение:

   ```azurecli-interactive
   resourceGroupName='<Resource_Group_Name>'
   clusterName='<Cluster Name>'
   dataCenterName='dc1'
   dataCenterLocation='eastus2'
   delegatedSubnetId= '<Resource_ID>'
    
   az managed-cassandra datacenter update \
      --resource-group $resourceGroupName \
      --cluster-name $clusterName \
      --data-center-name $dataCenterName \
      --node-count 9 
   ```

## <a name="connect-to-your-cluster"></a>Подключение к кластеру

В Управляемом экземпляре Azure для Apache Cassandra нельзя создать узлы с общедоступными IP-адресами. Чтобы подключиться к только что созданному кластеру Cassandra, необходимо создать другой ресурс в виртуальной сети. Это может быть приложение или виртуальная машина с установленным средством обработки запросов с открытым кодом [CQLSH](https://cassandra.apache.org/doc/latest/tools/cqlsh.html) от Apache. Для развертывания виртуальной машины Ubuntu можно использовать [шаблон Resource Manager](https://azure.microsoft.com/resources/templates/101-vm-simple-linux/). После развертывания подключитесь к компьютеру с помощью протокола SSH и установите CQLSH, как показано в следующих командах:

```bash
# Install default-jre and default-jdk
sudo apt update
sudo apt install openjdk-8-jdk openjdk-8-jre

# Install the Cassandra libraries in order to get CQLSH:
echo "deb http://www.apache.org/dist/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
sudo apt-get update
sudo apt-get install cassandra

# Export the SSL variables:
export SSL_VERSION=TLSv1_2
export SSL_VALIDATE=false

# Connect to CQLSH (replace <IP> with the private IP addresses of the nodes in your Datacenter):
host=("<IP>" "<IP>" "<IP>")
cqlsh $host 9042 -u cassandra -p cassandra --ssl
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Вы можете удалить группу ресурсов, управляемый экземпляр и все связанные ресурсы, если они больше не нужны, выполнив команду `az group delete`:

```azurecli-interactive
az group delete --name <Resource_Group_Name>
```

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать Управляемый экземпляр Azure для кластера Apache Cassandra с помощью Azure CLI. Теперь можно приступить к работе с кластером:

> [!div class="nextstepaction"]
> [Развертывание управляемого кластера Apache Spark с Azure Databricks](deploy-cluster-databricks.md)
