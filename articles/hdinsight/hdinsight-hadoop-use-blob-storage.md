---
title: Запрос данных из HDFS-совместимой службы хранилища Azure в Azure HDInsight
description: Узнайте, как запрашивать данные из службы хранилища Azure и Azure Data Lake Storage для сохранения результатов анализа.
ms.service: hdinsight
ms.topic: how-to
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: cedc0ff1b3c2aa64f32445eabc800748a753981d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98945422"
---
# <a name="use-azure-storage-with-azure-hdinsight-clusters"></a>Использование службы хранилища Azure с кластерами Azure HDInsight

Данные можно хранить в [хранилище BLOB-объектов Azure](../storage/common/storage-introduction.md), [Azure Data Lake Storage 1-го поколения](../data-lake-store/data-lake-store-overview.md)или [Azure Data Lake Storage 2-го поколения](../storage/blobs/data-lake-storage-introduction.md). Также можно использовать сочетание этих ресурсов. Данные варианты хранилищ позволяют безопасно и без потери пользовательских данных удалять используемые для расчетов кластеры HDInsight.

В Apache Hadoop поддерживается концепция файловой системы по умолчанию. Файловая система по умолчанию подразумевает использование центра сертификации и схемы по умолчанию. Она также может использоваться для разрешения относительных путей. При создании кластера HDInsight в качестве файловой системы по умолчанию можно указать контейнер больших двоичных объектов в службе хранилища Azure. В HDInsight 3,6 можно выбрать хранилище BLOB-объектов Azure или Azure Data Lake Storage 1-го поколения/Azure Data Lake Storage 2-го поколения в качестве системы файлов по умолчанию с некоторыми исключениями. Сведения о поддержке использования Data Lake Storage 1-го поколения как по умолчанию, так и по связанному хранилищу см. в разделе [доступность кластера HDInsight](./hdinsight-hadoop-use-data-lake-storage-gen1.md#availability-for-hdinsight-clusters).

Из этой статьи вы узнаете, как служба хранилища Azure работает с кластерами HDInsight. 
* Сведения о том, как Data Lake Storage 1-го поколения работает с кластерами HDInsight, см. в статье [использование Azure Data Lake Storage 1-го поколения кластеров Azure hdinsight](./hdinsight-hadoop-use-data-lake-storage-gen1.md).
* сведения о том, как Data Lake Storage 2-го поколения работает с кластерами HDInsight, см. в статье [использование Azure Data Lake Storage 2-го поколения кластеров Azure hdinsight](./hdinsight-hadoop-use-data-lake-storage-gen2.md).
* Дополнительные сведения о создании кластера HDInsight см. в статье [Установка кластеров в HDInsight с использованием Hadoop, Spark, Kafka и других технологий](./hdinsight-hadoop-provision-linux-clusters.md).

> [!IMPORTANT]  
> Тип учетной записи хранения **BlobStorage** можно использовать только как дополнительное хранилище кластеров HDInsight.

| Тип учетной записи хранения | Поддерживаемые службы | Поддерживаемые уровни производительности |Неподдерживаемые уровни производительности| Поддерживаемые уровни доступа |
|----------------------|--------------------|-----------------------------|---|------------------------|
| StorageV2 (учетная запись общего назначения версии 2)  | BLOB-объект     | Standard                    |Premium| Горячий, холодный или архивный\*   |
| Учетная запись хранения общего назначения версии 1   | BLOB-объект     | Standard                    |Premium| Недоступно                    |
| BlobStorage                    | BLOB-объект     | Standard                    |Premium| Горячий, холодный или архивный\*   |

Применяемый по умолчанию контейнер больших двоичных объектов не рекомендуется использовать для хранения бизнес-данных. Чтобы сократить затраты на хранение, контейнер больших двоичных объектов по умолчанию рекомендуется удалять после каждого использования. Контейнер по умолчанию содержит журналы приложений и системный журнал. Обязательно извлеките эти журналы перед удалением контейнера.

Совместное использование одного контейнера больших двоичных объектов в качестве файловой системы по умолчанию для нескольких кластеров не поддерживается.

> [!NOTE]  
> Архивный уровень доступа — это автономный уровень с задержкой извлечения в несколько часов, который не рекомендуется использовать вместе с HDInsight. Дополнительные сведения см. в разделе [Архивный уровень доступа](../storage/blobs/storage-blob-storage-tiers.md#archive-access-tier).

## <a name="access-files-from-within-cluster"></a>Доступ к файлам из кластера

Существует несколько способов доступа к файлам в Data Lake Storage из кластера HDInsight. Эта схема URI предоставляет как незашифрованный доступ с префиксом *wasb:* , так и доступ с использованием TLS-шифрования с *wasbs*. Мы рекомендуем использовать *wasbs* всегда, когда это возможно, даже при обращении к данным, которые хранятся в том же регионе Azure.

* **С помощью полного доменного имени**. При таком подходе необходимо указать полный путь к файлу, к которому требуется доступ.

    ```
    wasb://<containername>@<accountname>.blob.core.windows.net/<file.path>/
    wasbs://<containername>@<accountname>.blob.core.windows.net/<file.path>/
    ```

* **Используя сокращенный формат пути**. При таком подходе путь до корня кластера заменяется на следующий:

    ```
    wasb:///<file.path>/
    wasbs:///<file.path>/
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
    hdfs dfs -mkdir wasbs://CONTAINERNAME@STORAGEACCOUNT.blob.core.windows.net/sampledata1/
    hdfs dfs -mkdir wasbs:///sampledata2/
    hdfs dfs -mkdir /sampledata3/
    ```

1. Скопируйте данные из локального хранилища в хранилище кластера.

    ```bash
    hdfs dfs -copyFromLocal testFile.txt  wasbs://CONTAINERNAME@STORAGEACCOUNT.blob.core.windows.net/sampledata1/
    hdfs dfs -copyFromLocal testFile.txt  wasbs:///sampledata2/
    hdfs dfs -copyFromLocal testFile.txt  /sampledata3/
    ```

1. Перечислите содержимое каталога в хранилище кластера.

    ```bash
    hdfs dfs -ls wasbs://CONTAINERNAME@STORAGEACCOUNT.blob.core.windows.net/sampledata1/
    hdfs dfs -ls wasbs:///sampledata2/
    hdfs dfs -ls /sampledata3/
    ```

> [!NOTE]  
> При работе с большими двоичными объектами вне HDInsight большинство программ не распознают формат WASB и вместо этого ожидают формат базового пути, например `example/jars/hadoop-mapreduce-examples.jar`.

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
LOCATION 'wasbs://CONTAINERNAME@STORAGEACCOUNT.blob.core.windows.net/example/data/';
LOCATION 'wasbs:///example/data/';
LOCATION '/example/data/';
```

## <a name="access-files-from-outside-cluster"></a>Доступ к файлам из внешнего кластера

Корпорация Майкрософт предоставляет следующие инструменты для работы со службой хранилища Azure:

| Инструмент | Linux | OS X | Windows |
| --- |:---:|:---:|:---:|
| [Портал Azure](../storage/blobs/storage-quickstart-blobs-portal.md) |✔ |✔ |✔ |
| [Azure CLI](../storage/blobs/storage-quickstart-blobs-cli.md) |✔ |✔ |✔ |
| [Azure PowerShell](../storage/blobs/storage-quickstart-blobs-powershell.md) | | |✔ |
| [AzCopy](../storage/common/storage-use-azcopy-v10.md) |✔ | |✔ |

## <a name="identify-storage-path-from-ambari"></a>Указание пути к хранилищу из Ambari

* Чтобы указать полный путь к настроенному хранилищу по умолчанию, перейдите в следующий раздел:

    **HDFS** > **Configs** (Конфигурации) и введите `fs.defaultFS` в поле фильтра.

* Чтобы проверить, настроено ли хранилище wasb как дополнительное, перейдите в следующий раздел:

    **HDFS** > **Configs** (Конфигурации) и введите `blob.core.windows.net` в поле фильтра.

Сведения о том, как получить путь с помощью REST API Ambari, см. в разделе [Получение хранилища по умолчанию](./hdinsight-hadoop-manage-ambari-rest-api.md#get-the-default-storage).

## <a name="blob-containers"></a>Контейнеры больших двоичных объектов

Чтобы использовать большие двоичные объекты, сначала создайте [учетную запись службы хранилища Azure](../storage/common/storage-account-create.md). В рамках этого шага укажите регион Azure, в котором создается учетная запись хранения. Кластер и учетная запись хранения должны размещаться в одном регионе. База данных SQL Server хранилища метаданных Hive и база данных SQL Server хранилища метаданных Apache Oozie должны располагаться в одном регионе.

Где бы ни находился созданный BLOB-объект, он принадлежит контейнеру в вашей учетной записи хранения Azure. Этот контейнер может быть существующим BLOB-объектом, созданным за пределами HDInsight. Или это может быть контейнер, созданный для кластера HDInsight.

Стандартный контейнер больших двоичных объектов хранит сведения о кластере, включая журналы заданий. Не используйте стандартный контейнер BLOB-объектов с несколькими кластерами HDInsight. Это действие может привести к искажению истории заданий. Для каждого кластера рекомендуется использовать отдельный контейнер. Общие данные следует поместить в связанную учетную запись хранения, указанную для всех соответствующих кластеров, а не в учетную запись хранения по умолчанию. Дополнительные сведения о настройке связанных учетных записей хранения см. в статье [Создание кластеров HDInsight](hdinsight-hadoop-provision-linux-clusters.md). Тем не менее, вы можете повторно использовать контейнер хранения по умолчанию после удаления исходного кластера HDInsight. Для кластеров HBase можно сохранить все данные и схемы таблицы HBase, создав кластер HBase с помощью контейнера больших двоичных объектов по умолчанию, который используется удаленным кластером HBase.

[!INCLUDE [secure-transfer-enabled-storage-account](../../includes/hdinsight-secure-transfer.md)]

## <a name="use-additional-storage-accounts"></a>Использование дополнительных учетных записей хранения

При создании кластера HDInsight укажите учетную запись хранения Azure, которую необходимо с ним связать. Кроме того, в процессе создания или после создания кластера можно добавить дополнительные учетные записи хранения из той же или других подписок Azure. Инструкции по добавлению дополнительных учетных записей хранения см. в статье [Создание кластеров Hadoop в HDInsight](hdinsight-hadoop-provision-linux-clusters.md).

> [!WARNING]  
> Использование дополнительной учетной записи хранения, расположение которой отличается от расположения кластера HDInsight, не поддерживается.

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как использовать HDFS-совместимую службу хранилища Azure с HDInsight. В этом хранилище можно создавать адаптируемые, долгосрочные решения для получения данных архивирования, а также использовать HDInsight для разблокирования информации внутри хранимых структурированных и неструктурированных данных.

Дополнительные сведения см. в разделе:

* [Краткое руководство. Создание кластера Apache Hadoop](hadoop/apache-hadoop-linux-create-cluster-get-started-portal.md)
* [Учебник. Создание кластеров HDInsight](hdinsight-hadoop-provision-linux-clusters.md)
* [Использование Azure Data Lake Storage Gen2 с кластерами Azure HDInsight](hdinsight-hadoop-use-data-lake-storage-gen2.md)
* [Отправка данных в HDInsight](hdinsight-upload-data.md)
* [Руководство. Извлечение, преобразование и загрузка данных с помощью интерактивного запроса в Azure HDInsight](./interactive-query/interactive-query-tutorial-analyze-flight-data.md)
* [Использование подписанных URL-адресов хранилища Azure для ограничения доступа к данным с помощью HDInsight](hdinsight-storage-sharedaccesssignature-permissions.md)