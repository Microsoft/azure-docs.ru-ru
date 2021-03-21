---
title: Создание кластеров Apache Hadoop с помощью Azure CLI Azure HDInsight
description: Узнайте, как создавать кластеры Azure HDInsight с помощью кросс-платформенных Azure CLI.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-azurecli
ms.date: 02/03/2020
ms.openlocfilehash: 9028d85346611341afec0d0598f27a77e4f37fdf
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101715502"
---
# <a name="create-hdinsight-clusters-using-the-azure-cli"></a>Создание кластеров HDInsight с помощью интерфейса командной строки Azure

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]

Действия, описанные в этом документе, пошаговые инструкции по созданию кластера HDInsight 3,6 с помощью Azure CLI.

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="create-a-cluster"></a>Создание кластера

1. Войдите в подписку Azure. Если вы планируете использовать Azure Cloud Shell, щелкните **Попробовать** в правом верхнем углу блока кода. В противном случае введите следующую команду:

    ```azurecli-interactive
    az login

    # If you have multiple subscriptions, set the one to use
    # az account set --subscription "SUBSCRIPTIONID"
    ```

2. Задайте переменные среды. Использование переменных в этой статье основано на bash. В случае других сред потребуются минимальные изменения. Полный список возможных параметров для создания кластера см. в разделе [AZ-hdinsight-Create](/cli/azure/hdinsight#az-hdinsight-create) .

    |Параметр | Описание |
    |---|---|
    |`--workernode-count`| Число рабочих узлов в кластере. В этой статье используется переменная в `clusterSizeInNodes` качестве значения, переданного в `--workernode-count` . |
    |`--version`| Версия кластера HDInsight. В этой статье используется переменная в `clusterVersion` качестве значения, переданного в `--version` . См. также: [Поддерживаемые версии HDInsight](./hdinsight-component-versioning.md#supported-hdinsight-versions).|
    |`--type`| Тип кластера HDInsight, например Hadoop, интерактивехиве, HBase, Kafka, множество, Spark, Rserver, млсервицес.  В этой статье используется переменная в `clusterType` качестве значения, переданного в `--type` . См. также: [типы кластеров и конфигурация](./hdinsight-hadoop-provision-linux-clusters.md#cluster-type).|
    |`--component-version`|Версии различных компонентов Hadoop в формате "Component = версия", разделенных пробелами. В этой статье используется переменная в `componentVersion` качестве значения, переданного в `--component-version` . См. также: [компоненты Hadoop](./hdinsight-component-versioning.md).|

    Замените `RESOURCEGROUPNAME` , `LOCATION` , `CLUSTERNAME` , `STORAGEACCOUNTNAME` и на `PASSWORD` нужные значения. При необходимости измените значения для других переменных. Затем введите команды интерфейса командной строки.

    ```azurecli-interactive
    export resourceGroupName=RESOURCEGROUPNAME
    export location=LOCATION
    export clusterName=CLUSTERNAME
    export AZURE_STORAGE_ACCOUNT=STORAGEACCOUNTNAME
    export httpCredential='PASSWORD'
    export sshCredentials='PASSWORD'

    export AZURE_STORAGE_CONTAINER=$clusterName
    export clusterSizeInNodes=1
    export clusterVersion=3.6
    export clusterType=hadoop
    export componentVersion=Hadoop=2.7
    ```

3. [Создайте группу ресурсов](/cli/azure/group#az-group-create) , введя следующую команду:

    ```azurecli-interactive
    az group create \
        --location $location \
        --name $resourceGroupName
    ```

    Чтобы получить список допустимых расположений, используйте `az account list-locations` команду, а затем используйте одно из расположений из `name` значения.

4. [Создайте учетную запись хранения Azure](/cli/azure/storage/account#az-storage-account-create) , введя следующую команду:

    ```azurecli-interactive
    # Note: kind BlobStorage is not available as the default storage account.
    az storage account create \
        --name $AZURE_STORAGE_ACCOUNT \
        --resource-group $resourceGroupName \
        --https-only true \
        --kind StorageV2 \
        --location $location \
        --sku Standard_LRS
    ```

5. [Извлеките первичный ключ из учетной записи хранения Azure](/cli/azure/storage/account/keys#az-storage-account-keys-list) и сохраните его в переменной, введя следующую команду:

    ```azurecli-interactive
    export AZURE_STORAGE_KEY=$(az storage account keys list \
        --account-name $AZURE_STORAGE_ACCOUNT \
        --resource-group $resourceGroupName \
        --query [0].value -o tsv)
    ```

6. [Создайте контейнер службы хранилища Azure](/cli/azure/storage/container#az-storage-container-create) , введя следующую команду:

    ```azurecli-interactive
    az storage container create \
        --name $AZURE_STORAGE_CONTAINER \
        --account-key $AZURE_STORAGE_KEY \
        --account-name $AZURE_STORAGE_ACCOUNT
    ```

7. [Создайте кластер HDInsight](/cli/azure/hdinsight#az-hdinsight-create) , введя следующую команду:

    ```azurecli-interactive
    az hdinsight create \
        --name $clusterName \
        --resource-group $resourceGroupName \
        --type $clusterType \
        --component-version $componentVersion \
        --http-password $httpCredential \
        --http-user admin \
        --location $location \
        --workernode-count $clusterSizeInNodes \
        --ssh-password $sshCredentials \
        --ssh-user sshuser \
        --storage-account $AZURE_STORAGE_ACCOUNT \
        --storage-account-key $AZURE_STORAGE_KEY \
        --storage-container $AZURE_STORAGE_CONTAINER \
        --version $clusterVersion
    ```

    > [!IMPORTANT]  
    > Кластеры HDInsight бывают разных типов, которые соответствуют рабочей нагрузке или технологии, для которой предназначен кластер. Создать кластер, в котором бы объединились несколько типов, например Storm и HBase, нельзя.

    Процесс создания кластера может занять несколько минут. обычно около 15 минут.

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этой статьей кластер можно удалить. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Так как затраты на кластер во много раз превышают затраты на хранилище, экономически целесообразно удалять неиспользуемые кластеры.

Введите все следующие команды или некоторые из них, чтобы удалить ресурсы:

```azurecli-interactive
# Remove cluster
az hdinsight delete \
    --name $clusterName \
    --resource-group $resourceGroupName

# Remove storage container
az storage container delete \
    --account-name $AZURE_STORAGE_ACCOUNT \
    --name $AZURE_STORAGE_CONTAINER

# Remove storage account
az storage account delete \
    --name $AZURE_STORAGE_ACCOUNT \
    --resource-group $resourceGroupName

# Remove resource group
az group delete \
    --name $resourceGroupName
```

## <a name="troubleshoot"></a>Диагностика

Если при создании кластеров HDInsight возникли проблемы, см. раздел [Создание кластеров](./hdinsight-hadoop-customize-cluster-linux.md#access-control).

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы успешно создали кластер HDInsight с помощью Azure CLI, используйте следующую команду, чтобы узнать, как работать с кластером.

### <a name="apache-hadoop-clusters"></a>Кластеры Apache Hadoop

* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hadoop/hdinsight-use-hive.md)
* [Использование MapReduce в Hadoop в HDInsight](hadoop/hdinsight-use-mapreduce.md)

### <a name="apache-hbase-clusters"></a>Кластеры Apache HBase

* [Начало работы с Apache HBase в HDInsight](hbase/apache-hbase-tutorial-get-started-linux.md)
* [Разработка приложений Java для Apache HBase в HDInsight](hbase/apache-hbase-build-java-maven-linux.md)

### <a name="apache-storm-clusters"></a>Кластеры Apache Storm

* [Создание топологий для Apache Storm в HDInsight на языке Java](storm/apache-storm-develop-java-topology.md)
* [Использование компонентов Python в Apache Storm в HDInsight](storm/apache-storm-develop-python-topology.md)
* [Развертывание и мониторинг топологии с помощью Apache Storm в HDInsight.](storm/apache-storm-deploy-monitor-topology-linux.md)
