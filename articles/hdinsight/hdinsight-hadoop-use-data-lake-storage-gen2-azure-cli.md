---
title: Создание Azure HDInsight-Azure Data Lake Storage 2-го поколения-Azure CLI
description: Узнайте, как использовать Azure Data Lake Storage 2-го поколения с кластерами Azure HDInsight с помощью Azure CLI.
author: guyhay
ms.author: guyhay
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020, devx-track-azurecli
ms.date: 09/17/2020
ms.openlocfilehash: bbc1cd27d5c16eddd3aaad748c34445e5017e209
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98945491"
---
# <a name="create-a-cluster-with-data-lake-storage-gen2-using-azure-cli"></a>Создание кластера с Data Lake Storage 2-го поколения помощью Azure CLI

Чтобы создать кластер HDInsight, который использует Data Lake Storage 2-го поколения для хранения, выполните следующие действия.

## <a name="prerequisites"></a>Предварительные условия

- Если вы не знакомы с Azure Data Lake Storage 2-го поколения, ознакомьтесь с [разделом Обзор](hdinsight-hadoop-use-data-lake-storage-gen2.md). 
- Если у вас нет учетной записи Azure, [зарегистрируйтесь для получения бесплатной пробной учетной записи](https://azure.microsoft.com/free/), прежде чем продолжать.
- Выполнить примеры сценариев для интерфейса командной строки можно тремя способами:
    - использовать [Azure Cloud Shell](../cloud-shell/overview.md) с портала Azure (см. следующий раздел).
    - использовать внедренный компонент Azure Cloud Shell с помощью кнопки "Попробуйте!", расположенной в правом верхнем углу каждого блока кода.
    - [установить последнюю версию Azure CLI](/cli/azure/install-azure-cli) (2.0.13 или выше), если вы предпочитаете использовать локальную консоль CLI. Войдите в Azure с помощью `az login` , используя учетную запись, связанную с подпиской Azure, в которой вы хотите развернуть управляемое пользователем удостоверение. Azure CLI. 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Вы можете [скачать пример файла шаблона](https://github.com/Azure-Samples/hdinsight-data-lake-storage-gen2-templates/blob/master/hdinsight-adls-gen2-template.json) и [скачать пример файла параметров](https://github.com/Azure-Samples/hdinsight-data-lake-storage-gen2-templates/blob/master/parameters.json). Перед использованием шаблона и приведенного ниже фрагмента кода Azure CLI Замените следующие заполнители их правильными значениями:

| Заполнитель | Описание |
|---|---|
| `<SUBSCRIPTION_ID>` | Идентификатор подписки Azure |
| `<RESOURCEGROUPNAME>` | Группа ресурсов, в которой вы хотите создать новый кластер и учетную запись хранения. |
| `<MANAGEDIDENTITYNAME>` | Имя управляемого удостоверения, которому будут предоставлены разрешения для учетной записи хранения с Azure Data Lake Storage 2-го поколения. |
| `<STORAGEACCOUNTNAME>` | Новая учетная запись хранения с Azure Data Lake Storage 2-го поколения, которая будет создана. |
| `<FILESYSTEMNAME>`  | Имя файловой системы, которую этот кластер должен использовать в учетной записи хранения. |
| `<CLUSTERNAME>` | Это имя вашего кластера HDInsight. |
| `<PASSWORD>` | Выбранный пароль для входа в кластер с помощью SSH и панели мониторинга Ambari. |

Приведенный ниже фрагмент кода выполняет следующие начальные действия.

1. Выполняет вход в учетную запись Azure.
1. Задает активную подписку, в которой будут выполняться операции создания.
1. Создает новую группу ресурсов для новых действий развертывания.
1. Создание управляемого удостоверения, назначенного пользователем.
1. Добавляет расширение в Azure CLI для использования функций Data Lake Storage 2-го поколения.
1. Создает новую учетную запись хранения с Data Lake Storage 2-го поколения с помощью `--hierarchical-namespace true` флага.

```azurecli
az login
az account set --subscription <SUBSCRIPTION_ID>

# Create resource group
az group create --name <RESOURCEGROUPNAME> --location eastus

# Create managed identity
az identity create -g <RESOURCEGROUPNAME> -n <MANAGEDIDENTITYNAME>

az extension add --name storage-preview

az storage account create --name <STORAGEACCOUNTNAME> \
    --resource-group <RESOURCEGROUPNAME> \
    --location eastus --sku Standard_LRS \
    --kind StorageV2 --hierarchical-namespace true
```

Затем войдите на портал. Добавьте новое назначенное пользователем управляемое удостоверение в роль **участник данных BLOB-объекта хранилища** в учетной записи хранения. Этот шаг описан на шаге 3 в разделе [использование портал Azure](hdinsight-hadoop-use-data-lake-storage-gen2.md).

 > [!IMPORTANT]
 > Убедитесь, что учетной записи хранения назначено пользовательское удостоверение с разрешениями роли **участника данных BLOB-объекта хранилища** . в противном случае создание кластера завершится ошибкой.

```azurecli
az deployment group create --name HDInsightADLSGen2Deployment \
    --resource-group <RESOURCEGROUPNAME> \
    --template-file hdinsight-adls-gen2-template.json \
    --parameters parameters.json
```

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

Вы успешно создали кластер HDInsight. Теперь узнайте, как работать с кластером.

### <a name="apache-spark-clusters"></a>Кластеры Apache Spark

* [Настройка кластеров HDInsight под управлением Linux с помощью действий сценариев](hdinsight-hadoop-customize-cluster-linux.md)
* [Создание автономного приложения с использованием Scala](spark/apache-spark-create-standalone-application.md)
* [Удаленный запуск заданий с помощью Apache Livy в кластере Apache Spark](spark/apache-spark-livy-rest-interface.md)
* [Использование Apache Spark со средствами бизнес-аналитики. Выполнение интерактивного анализа данных с использованием Spark в HDInsight с помощью средств бизнес-аналитики](spark/apache-spark-use-bi-tools.md)
* [Apache Spark и Машинное обучение. Прогнозирование результатов проверки пищевых продуктов с помощью Spark в HDInsight](spark/apache-spark-machine-learning-mllib-ipython.md)

### <a name="apache-hadoop-clusters"></a>Кластеры Apache Hadoop

* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hadoop/hdinsight-use-hive.md)
* [Использование MapReduce в Hadoop в HDInsight](hadoop/hdinsight-use-mapreduce.md)

### <a name="apache-hbase-clusters"></a>Кластеры Apache HBase

* [Начало работы с Apache HBase в HDInsight](hbase/apache-hbase-tutorial-get-started-linux.md)
* [Разработка приложений Java для Apache HBase в HDInsight](hbase/apache-hbase-build-java-maven-linux.md)
