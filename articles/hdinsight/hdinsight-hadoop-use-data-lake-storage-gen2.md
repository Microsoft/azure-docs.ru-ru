---
title: Использование Azure Data Lake Storage 2-го поколения с кластерами Azure HDInsight
description: Узнайте, как использовать Azure Data Lake Storage 2-го поколения с кластерами Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,seoapr2020, devx-track-azurecli
ms.date: 04/24/2020
ms.openlocfilehash: 2bbfbd2d953ea663453f0092ff366e95f6dd5ea7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98945376"
---
# <a name="use-azure-data-lake-storage-gen2-with-azure-hdinsight-clusters"></a>Использование Azure Data Lake Storage 2-го поколения с кластерами Azure HDInsight

[Azure Data Lake Storage 2-го поколения](../storage/blobs/data-lake-storage-introduction.md) — это служба облачного хранилища, выделенная для аналитики больших данных, созданная на основе [хранилища BLOB-объектов Azure](../storage/blobs/storage-blobs-introduction.md). Data Lake Storage 2-го поколения сочетает возможности хранилища BLOB-объектов Azure и Azure Data Lake Storage 1-го поколения. Итоговая служба предлагает функции из Azure Data Lake Storage 1-го поколения, включая семантику файловой системы, безопасность на уровне каталога и защиту на уровне файлов, а также адаптируемость. Наряду с низкими затратами, многоуровневое хранилище, высокая доступность и возможности аварийного восстановления из хранилища BLOB-объектов Azure.

Полное сравнение параметров создания кластера с помощью Data Lake Storage 2-го поколения см. в статье [Сравнение вариантов хранения для использования с кластерами Azure HDInsight](hdinsight-hadoop-compare-storage-options.md).

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="data-lake-storage-gen2-availability"></a>Доступность Azure Data Lake Storage 2-го поколения

Data Lake Storage 2-го поколения доступен в качестве хранилища практически для всех типов кластеров Azure HDInsight как по умолчанию, так и в дополнительной учетной записи хранения. Однако HBase может иметь только одну учетную запись с Data Lake Storage 2-го поколения.

> [!Note]  
> Выбрав Data Lake Storage 2-го поколения в качестве **основного типа хранилища**, вы не сможете выбрать Data Lake Storage 1-го поколения в качестве дополнительного хранилища.

## <a name="create-hdinsight-clusters-using-data-lake-storage-gen2"></a>Создание кластеров HDInsight с помощью Data Lake Storage 2-го поколения

Используйте следующие ссылки, чтобы получить подробные инструкции по созданию кластеров HDInsight с доступом к Data Lake Storage 2-го поколения.

* [Использование портала](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2-portal.md)
* [Использование Azure CLI](../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2-azure-cli.md)
* Сейчас PowerShell не поддерживается для создания кластера HDInsight с Azure Data Lake Storage 2-го поколения.

## <a name="access-control-for-data-lake-storage-gen2-in-hdinsight"></a>Контроль доступа для Data Lake Storage 2-го поколения в HDInsight

### <a name="what-kinds-of-permissions-does-data-lake-storage-gen2-support"></a>Какие типы разрешений поддерживаются в Azure Data Lake Storage 2-го поколения?

Data Lake Storage 2-го поколения использует модель управления доступом, поддерживающую управление доступом на основе ролей Azure (Azure RBAC) и списки управления доступом (ACL), такие как POSIX. Data Lake Storage 1-го поколения поддерживает списки управления доступом только для управления доступом к данным.

Azure RBAC использует назначения ролей для эффективного применения наборов разрешений для пользователей, групп и субъектов-служб для ресурсов Azure. Как правило, ресурсы Azure ограничены ресурсами верхнего уровня (например, учетными записями хранилища BLOB-объектов Azure). Для хранилища BLOB-объектов Azure, а также Data Lake Storage 2-го поколения этот механизм был расширен до ресурса файловой системы.

Дополнительные сведения о разрешениях для файлов с помощью Azure RBAC см. в статье [Управление доступом на основе ролей в Azure (Azure RBAC)](../storage/blobs/data-lake-storage-access-control-model.md#role-based-access-control).

Дополнительные сведения о разрешениях для файлов с помощью ACL см. [в разделе списки управления доступом к файлам и каталогам](../storage/blobs/data-lake-storage-access-control.md).

### <a name="how-do-i-control-access-to-my-data-in-data-lake-storage-gen2"></a>Разделы справки управлять доступом к моим данным в Data Lake Storage 2-го поколения?

Доступ к файлам в Data Lake Storage 2-го поколения управляется кластером HDInsight с помощью управляемых удостоверений. Управляемое удостоверение — это удостоверение, зарегистрированное в Azure Active Directory (Azure AD), учетные данные которых управляются Azure. При использовании управляемых удостоверений не нужно регистрировать субъекты-службы в Azure AD. Или сохранить учетные данные, например сертификаты.

Службы Azure имеют два типа управляемых удостоверений: назначенные системой и назначенные пользователем. HDInsight использует назначенные пользователем управляемые удостоверения для доступа к Data Lake Storage 2-го поколения. Создается `user-assigned managed identity` как автономный ресурс Azure. При этом Azure создает удостоверение в доверенном клиенте Azure AD используемой подписки. После создания удостоверения оно может быть назначено одному или нескольким экземплярам служб Azure.

Управление жизненным циклом назначаемого пользователем удостоверения осуществляется отдельно от жизненного цикла экземпляров служб Azure, для которых оно назначено. Дополнительные сведения об управляемых удостоверениях см. в статье [что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md).

### <a name="how-do-i-set-permissions-for-azure-ad-users-to-query-data-in-data-lake-storage-gen2-by-using-hive-or-other-services"></a>Разделы справки задать разрешения для пользователей Azure AD на запрос данных в Data Lake Storage 2-го поколения с помощью Hive или других служб?

Чтобы задать разрешения для запроса данных для пользователей, используйте группы безопасности Azure AD в качестве назначенного участника в списке ACL. Не присваивайте напрямую разрешения на доступ к файлам отдельным пользователям или субъектам-службам. С помощью групп безопасности Azure AD можно управлять потоком разрешений, добавлять и удалять пользователей или субъекты-службы без повторного применения списков ACL ко всей структуре каталогов. Нужно просто добавить пользователей в соответствующую группу безопасности Azure AD или удалить их из нее. Списки управления доступом не наследуются, поэтому для повторного применения ACL требуется обновление списка управления доступом для каждого файла и подкаталога.

## <a name="access-files-from-the-cluster"></a>Доступ к файлам из кластера

Существует несколько способов доступа к файлам в Data Lake Storage 2-го поколения из кластера HDInsight.

* **С помощью полного доменного имени**. При таком подходе необходимо указать полный путь к файлу, к которому требуется доступ.

    ```
    abfs://<containername>@<accountname>.dfs.core.windows.net/<file.path>/
    ```

* **Используя сокращенный формат пути**. При таком подходе путь до корня кластера заменяется на следующий:

    ```
    abfs:///<file.path>/
    ```

* **С помощью относительного пути**. При таком подходе указывается только относительный путь к файлу, к которому требуется доступ.

    ```
    /<file.path>/
    ```

### <a name="data-access-examples"></a>Примеры доступа к данным

Примеры основаны на [SSH-подключении](./hdinsight-hadoop-linux-use-ssh-unix.md) к головному узлу кластера. В примерах используются все три схемы URI. Замените `CONTAINERNAME` и `STORAGEACCOUNT` соответствующими значениями

#### <a name="a-few-hdfs-commands"></a>Несколько команд HDFS

1. Создайте файл в локальном хранилище.

    ```bash
    touch testFile.txt
    ```

1. Создайте каталог в хранилище кластера.

    ```bash
    hdfs dfs -mkdir abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -mkdir abfs:///sampledata2/
    hdfs dfs -mkdir /sampledata3/
    ```

1. Скопируйте данные из локального хранилища в хранилище кластера.

    ```bash
    hdfs dfs -copyFromLocal testFile.txt  abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -copyFromLocal testFile.txt  abfs:///sampledata2/
    hdfs dfs -copyFromLocal testFile.txt  /sampledata3/
    ```

1. Перечислите содержимое каталога в хранилище кластера.

    ```bash
    hdfs dfs -ls abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/sampledata1/
    hdfs dfs -ls abfs:///sampledata2/
    hdfs dfs -ls /sampledata3/
    ```

#### <a name="creating-a-hive-table"></a>Создание таблицы Hive

Для наглядности показаны три расположения файлов. Для фактического выполнения используйте только одну из записей `LOCATION`.

```hql
DROP TABLE myTable;
CREATE EXTERNAL TABLE myTable (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE
LOCATION 'abfs://CONTAINERNAME@STORAGEACCOUNT.dfs.core.windows.net/example/data/';
LOCATION 'abfs:///example/data/';
LOCATION '/example/data/';
```

## <a name="next-steps"></a>Дальнейшие действия

* [Azure HDInsight integration with Data Lake Storage Gen2 preview - ACL and security update](https://azure.microsoft.com/blog/azure-hdinsight-integration-with-data-lake-storage-gen-2-preview-acl-and-security-update/) (Интеграция Azure HDInsight с Data Lake Storage 2-го поколения (предварительная версия) — обновление списка управления доступом и системы безопасности)
* [Общие сведения об Azure Data Lake Storage 2-го поколения](../storage/blobs/data-lake-storage-introduction.md)
* [Руководство. Извлечение, преобразование и загрузка данных с помощью интерактивного запроса в Azure HDInsight](./interactive-query/interactive-query-tutorial-analyze-flight-data.md)
