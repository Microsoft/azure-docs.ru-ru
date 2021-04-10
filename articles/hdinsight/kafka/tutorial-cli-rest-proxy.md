---
title: Руководство. Создание кластера Apache Kafka с поддержкой прокси-сервера REST в HDInsight с помощью Azure CLI
description: Сведения о том, как выполнять в Azure HDInsight операции Apache Kafka с помощью прокси-сервера REST для Kafka.
ms.service: hdinsight
ms.topic: tutorial
ms.date: 02/27/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: ff11b8461b483f5a66df19bb1b108a1fe1168fb9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98944030"
---
# <a name="tutorial-create-an-apache-kafka-rest-proxy-enabled-cluster-in-hdinsight-using-azure-cli"></a>Руководство. Создание кластера Apache Kafka с поддержкой прокси-сервера REST в HDInsight с помощью Azure CLI

Из этого учебника вы узнаете, как создать в Azure HDInsight кластер Apache Kafka [с поддержкой прокси-сервера REST](./rest-proxy.md) с помощью интерфейса командной строки (CLI) Azure. Azure HDInsight — это управляемая комплексная служба аналитики с открытым кодом, предназначенная для предприятий. Apache Kafka — это распределенная платформа потоковой передачи с открытым кодом. Она часто используется как брокер сообщений, предоставляя такие же функциональные возможности, как и очередь сообщений типа "публикация-подписка". Прокси-сервер REST для Kafka позволяет взаимодействовать с кластером Kafka через [REST API](/rest/api/hdinsight-kafka-rest-proxy/) по протоколу HTTP. Azure CLI — это кроссплатформенный интерфейс командной строки от Майкрософт для управления ресурсами Azure.

API Apache Kafka доступен только ресурсам, размещенным в той же виртуальной сети. Вы можете обращаться к кластеру напрямую по протоколу SSH. Чтобы подключить к Apache Kafka другие службы, сети или виртуальные машины, необходимо сначала создать виртуальную сеть, а затем создать в ней эти ресурсы. Дополнительные сведения см. в статье [Подключение к Kafka в HDInsight с помощью виртуальной сети Azure](./apache-kafka-connect-vpn-gateway.md).

Из этого руководства вы узнаете:

> [!div class="checklist"]
> * необходимые условия для прокси-сервера REST для Kafka;
> * как создать кластер Apache Kafka с помощью Azure CLI.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Приложение, зарегистрированное в AAD. Клиентские приложения, которые вы создаете для взаимодействия с прокси-сервером REST для Kafka, будут использовать эти значения идентификатора и секрета приложения для проверки подлинности в Azure. Подробнее см. статью [о регистрации приложения на платформе удостоверений Майкрософт](../../active-directory/develop/quickstart-register-app.md).

* Группа безопасности Azure AD, в которой приложение зарегистрировано в качестве участника. Эта группа безопасности будет использоваться для управления тем, какие приложения могут взаимодействовать с прокси-сервером REST. Ознакомьтесь со статьей [Создание простой группы и добавление в нее участников с помощью Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md), чтобы получить сведения о создании групп AAD.

* Azure CLI. Убедитесь, что у вас установлена версия 2.0.79 или более поздняя. Подробнее см. статью [Установка Azure CLI](/cli/azure/install-azure-cli).

## <a name="create-an-apache-kafka-cluster"></a>Создание кластера Apache Kafka

1. Войдите в подписку Azure.

    ```azurecli
    az login
    
    # If you have multiple subscriptions, set the one to use
    # az account set --subscription "SUBSCRIPTIONID"
    ```

1. Задайте переменные среды. В этом учебнике используются переменные, характерные для Bash. В случае других сред потребуются минимальные изменения.

    |Переменная | Описание |
    |---|---|
    |имя_группы_ресурсов|Замените RESOURCEGROUPNAME именем новой группы ресурсов.|
    |location|Замените LOCATION регионом, в котором будет создан кластер. Чтобы получить список допустимых расположений, выполните команду `az account list-locations`.|
    |clusterName|Замените CLUSTERNAME глобально уникальным именем нового кластера.|
    |storageAccount|Замените STORAGEACCOUNTNAME именем новой учетной записи хранения.|
    |httpPassword|Замените PASSWORD паролем для учетной записи **admin** в кластере.|
    |sshPassword|Замените PASSWORD паролем для учетной записи **sshuser** для оболочки SSH.|
    |securityGroupName|Замените SECURITYGROUPNAME именем группы безопасности AAD клиента для прокси-сервера REST для Kafka. Эта переменная будет передана в параметре `--kafka-client-group-name` оператору `az-hdinsight-create`.|
    |securityGroupID|Замените SECURITYGROUPID идентификатором группы безопасности AAD клиента для прокси-сервера REST для Kafka. Эта переменная будет передана в параметре `--kafka-client-group-id` оператору `az-hdinsight-create`.|
    |storageContainer|Контейнер хранилища, который будет использоваться кластером. Его не нужно изменять для этого учебника. Эта переменная получит имя кластера.|
    |workernodeCount|Количество рабочих ролей в кластере. Его не нужно изменять для этого учебника. Чтобы гарантировать высокую доступность, Kafka требуется три рабочих узла или более.|
    |clusterType|Тип кластера HDInsight. Его не нужно изменять для этого учебника.|
    |clusterVersion|Версия кластера HDInsight. Ее не нужно изменять для этого учебника. Прокси-серверу REST для Kafka требуется версия кластера не ниже 4.0.|
    |componentVersion|Версия Kafka. Ее не нужно изменять для этого учебника. Прокси-серверу REST для Kafka требуется версия компонента не ниже 2.1.|

    Присвойте переменным нужные значения. Затем введите команды командной строки, чтобы задать переменные среды.

    ```azurecli
    export resourceGroupName=RESOURCEGROUPNAME
    export location=LOCATION
    export clusterName=CLUSTERNAME
    export storageAccount=STORAGEACCOUNTNAME
    export httpPassword='PASSWORD'
    export sshPassword='PASSWORD'
    export securityGroupName=SECURITYGROUPNAME
    export securityGroupID=SECURITYGROUPID

    export storageContainer=$(echo $clusterName | tr "[:upper:]" "[:lower:]")
    export workernodeCount=3
    export clusterType=kafka
    export clusterVersion=4.0
    export componentVersion=kafka=2.1
    ```

1. [Создайте группу ресурсов](/cli/azure/group#az-group-create), введя следующую команду:

    ```azurecli
     az group create \
        --location $location \
        --name $resourceGroupName
    ```

1. [Создайте учетную запись хранения Azure](/cli/azure/storage/account#az-storage-account-create), введя следующую команду:

    ```azurecli
    # Note: kind BlobStorage is not available as the default storage account.
    az storage account create \
        --name $storageAccount \
        --resource-group $resourceGroupName \
        --https-only true \
        --kind StorageV2 \
        --location $location \
        --sku Standard_LRS
    ```

1. [Извлеките первичный ключ](/cli/azure/storage/account/keys#az-storage-account-keys-list) из учетной записи хранения Azure и сохраните его в переменной, введя следующую команду:

    ```azurecli
    export storageAccountKey=$(az storage account keys list \
        --account-name $storageAccount \
        --resource-group $resourceGroupName \
        --query [0].value -o tsv)
    ```

1. [Создайте контейнер хранилища Azure](/cli/azure/storage/container#az-storage-container-create), введя следующую команду:

    ```azurecli
    az storage container create \
        --name $storageContainer \
        --account-key $storageAccountKey \
        --account-name $storageAccount
    ```

1. [Создайте кластер HDInsight](/cli/azure/hdinsight#az-hdinsight-create). Прежде чем вводить эту команду, укажите следующие параметры.

    1. Обязательные параметры для кластеров Kafka:

        |Параметр | Описание|
        |---|---|
        |--type|Необходимо задать значение **Kafka**.|
        |--workernode-data-disks-per-node|Количество дисков данных, используемых для каждого рабочего узла. Kafka поддерживается в HDInsight только с дисками данных. В этом учебнике используется значение **2**.|

    1. Обязательные параметры для прокси-сервера REST для Kafka:

        |Параметр | Описание|
        |---|---|
        |--kafka-management-node-size|Размер узла. В этом руководстве используется значение **Standard_D4_v2**.|
        |--kafka-client-group-id|Идентификатор группы безопасности AAD клиента для прокси-сервера REST для Kafka. Это значение передается из переменной **$securityGroupID**.|
        |--kafka-client-group-name|Имя группы безопасности AAD клиента для прокси-сервера REST для Kafka. Это значение передается из переменной **$securityGroupName**.|
        |--version|Требуется версия кластера HDInsight не ниже 4.0. Это значение передается из переменной **$clusterVersion**.|
        |--component-version|Требуется версия Kafka не ниже 2.1. Это значение передается из переменной **$componentVersion**.|
    
        Если вы хотите создать кластер без прокси-сервера REST, исключите параметры `--kafka-management-node-size`, `--kafka-client-group-id` и `--kafka-client-group-name` из команды `az hdinsight create`.

    1. Если у вас уже есть виртуальная сеть, добавьте параметры `--vnet-name` и `--subnet` с нужными значениями.

    Введите следующую команду, чтобы создать кластер.

    ```azurecli
    az hdinsight create \
        --name $clusterName \
        --resource-group $resourceGroupName \
        --type $clusterType \
        --component-version $componentVersion \
        --http-password $httpPassword \
        --http-user admin \
        --location $location \
        --ssh-password $sshPassword \
        --ssh-user sshuser \
        --storage-account $storageAccount \
        --storage-account-key $storageAccountKey \
        --storage-container $storageContainer \
        --version $clusterVersion \
        --workernode-count $workernodeCount \
        --workernode-data-disks-per-node 2 \
        --kafka-management-node-size "Standard_D4_v2" \
        --kafka-client-group-id $securityGroupID \
        --kafka-client-group-name "$securityGroupName"
    ```

    Процесс создания кластера может занять несколько минут. обычно около 15 минут.

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы с этой статьей кластер можно удалить. В случае с HDInsight ваши данные хранятся в службе хранилища Azure, что позволяет безопасно удалить неиспользуемый кластер. Плата за кластеры HDInsight взимается, даже когда они не используются. Так как затраты на кластер во много раз превышают затраты на хранилище, экономически целесообразно удалять неиспользуемые кластеры.

Введите все следующие команды или некоторые из них, чтобы удалить ресурсы:

```azurecli
# Remove cluster
az hdinsight delete \
    --name $clusterName \
    --resource-group $resourceGroupName

# Remove storage container
az storage container delete \
    --account-name $storageAccount  \
    --name $storageContainer

# Remove storage account
az storage account delete \
    --name $storageAccount  \
    --resource-group $resourceGroupName

# Remove resource group
az group delete \
    --name $resourceGroupName
```

## <a name="next-steps"></a>Дальнейшие действия

Итак, вы успешно создали в Azure HDInsight кластер с поддержкой прокси-сервера REST для Apache Kafka с помощью Azure CLI. Теперь вы можете взаимодействовать с этим прокси-сервером REST, используя код Python:

> [!div class="nextstepaction"]
> [Создание примера приложения](./rest-proxy.md#client-application-sample)