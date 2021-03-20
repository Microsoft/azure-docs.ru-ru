---
title: Решения службы хранилища Azure для служб ML в HDInsight — Azure
description: Сведения о разных вариантах хранилищ, доступных для служб машинного обучения в HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 01/02/2020
ms.openlocfilehash: ddc48025de164ff68fb539a293e06bae09171742
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98943909"
---
# <a name="azure-storage-solutions-for-ml-services-on-azure-hdinsight"></a>Решения службы хранилища Azure для служб ML в Azure HDInsight

Службы ML в HDInsight могут использовать различные решения хранилища для сохранения данных, кода или объектов, содержащих результаты анализа. Эти решения включают следующие варианты.

- [Хранилище BLOB-объектов Azure](https://azure.microsoft.com/services/storage/blobs/)
- [Azure Data Lake Storage 1-го поколения](https://azure.microsoft.com/services/storage/data-lake-storage/)
- [Хранилище файлов Azure](https://azure.microsoft.com/services/storage/files/)

Кроме того, из кластера HDInsight вы можете обращаться к нескольким учетным записям хранения или контейнерам Azure. Хранилище файлов Azure — это удобный вариант хранения данных для использования на пограничном узле, который позволяет подключать файловый ресурс службы хранилища Azure к, например, файловой системе Linux. Общие папки Azure можно подключить и использовать на любом устройстве с поддерживаемой операционной системой, например Windows или Linux.

При создании кластера Apache Hadoop в HDInsight необходимо указать учетную запись **хранилища BLOB-объектов Azure** или **Data Lake Storage 1-го поколения**. Определенный контейнер хранилища в этой учетной записи используется для хранения файловой системы создаваемого кластера, например распределенной файловой системы Hadoop (HDFS). Дополнительные сведения и рекомендации см. в следующих статьях:

- [Использование хранилища BLOB-объектов Azure с HDInsight](../hdinsight-hadoop-use-blob-storage.md)
- [Использование Data Lake Storage 1-го поколения с кластерами Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen1.md)

## <a name="use-azure-blob-storage-accounts-with-ml-services-cluster"></a>Использование учетных записей хранения BLOB-объектов Azure с кластером служб машинного обучения

Ниже объясняется, как получить доступ к данным и операциям кластера служб машинного обучения с помощью дополнительной учетной записи, если при создании кластера этих служб вы указали несколько учетных записей хранения. Предположим, что у вас есть две учетные записи хранения (**storage1** для контейнера по умолчанию **container1** и **storage2** для **container2**).

> [!WARNING]  
> Для повышения производительности кластер HDInsight создается в том же центре обработки данных, где находится заданная вами основная учетная запись хранения. Использование учетной записи хранения, расположение которой отличается от расположения кластера HDInsight, не поддерживается.

### <a name="use-the-default-storage-with-ml-services-on-hdinsight"></a>Использование хранилища по умолчанию для служб машинного обучения в HDInsight

1. С помощью клиента SSH подключитесь к граничному узлу кластера. Сведения об использовании протокола SSH с кластерами HDInsight см. в статье [Подключение к HDInsight (Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md).
  
2. Скопируйте пример файла mysamplefile.csv в каталог /share.

    ```bash
    hadoop fs –mkdir /share
    hadoop fs –copyFromLocal mycsv.scv /share
    ```

3. Подключитесь к R Studio или другой консоли R, введите код R, чтобы задать узлу имени **значение по умолчанию** и расположение файла, к которому нужно получить доступ.  

    ```R
    myNameNode <- "default"
    myPort <- 0

    #Location of the data:  
    bigDataDirRoot <- "/share"  

    #Define Spark compute context:
    mySparkCluster <- RxSpark(nameNode=myNameNode, consoleOutput=TRUE)

    #Set compute context:
    rxSetComputeContext(mySparkCluster)

    #Define the Hadoop Distributed File System (HDFS) file system:
    hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

    #Specify the input file to analyze in HDFS:
    inputFile <-file.path(bigDataDirRoot,"mysamplefile.csv")
    ```

Все ссылки на каталоги и файлы указывают на учетную запись хранения `wasbs://container1@storage1.blob.core.windows.net`. Это **учетная запись хранения по умолчанию**, связанная с кластером HDInsight.

### <a name="use-the-additional-storage-with-ml-services-on-hdinsight"></a>Использование дополнительного хранилища для служб машинного обучения в HDInsight

Теперь мы рассмотрим ситуацию, в которой вам нужно обработать файл с именем mysamplefile1.csv, расположенный в каталоге /private контейнера **container2** в учетной записи хранения **storage2**.

В коде R измените ссылку на узел имени так, чтобы она указывала на учетную запись хранения **storage2** .

```R
myNameNode <- "wasbs://container2@storage2.blob.core.windows.net"
myPort <- 0

#Location of the data:
bigDataDirRoot <- "/private"

#Define Spark compute context:
mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

#Set compute context:
rxSetComputeContext(mySparkCluster)

#Define HDFS file system:
hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

#Specify the input file to analyze in HDFS:
inputFile <-file.path(bigDataDirRoot,"mysamplefile1.csv")
```

Теперь все ссылки на каталоги и файлы указывают на учетную запись хранения `wasbs://container2@storage2.blob.core.windows.net`. Это указанный вами **узел имени**.

Настройте `/user/RevoShare/<SSH username>` каталог в **storage2** следующим образом:

```bash
hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user
hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user/RevoShare
hadoop fs -mkdir wasbs://container2@storage2.blob.core.windows.net/user/RevoShare/<RDP username>
```

## <a name="use-azure-data-lake-storage-gen1-with-ml-services-cluster"></a>Использование Azure Data Lake Storage 1-го поколения с кластером служб ML

Чтобы использовать Data Lake Storage 1-го поколения с кластером HDInsight, необходимо предоставить кластеру доступ к каждой Azure Data Lake Storage 1-го поколения, которую вы хотите использовать. Инструкции по использованию портал Azure для создания кластера HDInsight с Azure Data Lake Storage 1-го поколения в качестве хранилища по умолчанию или дополнительного хранилища см. в статье [Создание кластера hdinsight с Data Lake Storage 1-го поколения с помощью портал Azure](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md).

Затем вы можете использовать такое хранилище в скрипте R примерно так же, как дополнительную учетную запись хранения Azure, как описано в предыдущей процедуре.

### <a name="add-cluster-access-to-your-azure-data-lake-storage-gen1"></a>Добавление доступа к кластеру в Azure Data Lake Storage 1-го поколения

Доступ к Data Lake Storage 1-го поколения осуществляется с помощью субъекта-службы Azure Active Directory (Azure AD), связанного с кластером HDInsight.

1. При создании кластера HDInsight выберите пункт **Удостоверение кластера Azure AD** на вкладке **источник данных** .

2. В диалоговом окне **Удостоверение кластера Azure AD** в разделе **Выбор субъекта-службы AD** выберите **создать**.

Присвоив имя субъекту-службе и создав для него пароль, щелкните **Управление доступом ADLS**, чтобы связать субъект-службу с Data Lake Storage.

Можно также добавить доступ к кластеру для одной или нескольких Data Lake учетных записей Gen1 хранилища после создания кластера. Откройте запись портал Azure Data Lake Storage 1-го поколения и перейдите к **обозреватель данных > доступ > добавить**.

### <a name="how-to-access-data-lake-storage-gen1-from-ml-services-on-hdinsight"></a>Доступ к Data Lake Storage 1-го поколения из служб машинного обучения в HDInsight

Когда вы получите доступ к Data Lake Storage 1-го поколения, вы можете использовать хранилище в кластере служб ML в HDInsight так же, как и для дополнительной учетной записи хранения Azure. Единственное отличие заключается в том, что префикс **wasbs://** изменяется на **ADL://** следующим образом:

```R
# Point to the ADL Storage (e.g. ADLtest)
myNameNode <- "adl://rkadl1.azuredatalakestore.net"
myPort <- 0

# Location of the data (assumes a /share directory on the ADL account)
bigDataDirRoot <- "/share"  

# Define Spark compute context
mySparkCluster <- RxSpark(consoleOutput=TRUE, nameNode=myNameNode, port=myPort)

# Set compute context
rxSetComputeContext(mySparkCluster)

# Define HDFS file system
hdfsFS <- RxHdfsFileSystem(hostName=myNameNode, port=myPort)

# Specify the input file in HDFS to analyze
inputFile <-file.path(bigDataDirRoot,"mysamplefile.csv")
```

Следующие команды используются для настройки Data Lake Storage 1-го поколения с помощью каталога RevoShare и добавления файла Sample. CSV из предыдущего примера:

```bash
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare
hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/user/RevoShare/<user>

hadoop fs -mkdir adl://rkadl1.azuredatalakestore.net/share

hadoop fs -copyFromLocal /usr/lib64/R Server-7.4.1/library/RevoScaleR/SampleData/mysamplefile.csv adl://rkadl1.azuredatalakestore.net/share

hadoop fs –ls adl://rkadl1.azuredatalakestore.net/share
```

## <a name="use-azure-file-storage-with-ml-services-on-hdinsight"></a>Использование хранилища файлов Azure для служб машинного обучения в HDInsight

Существует также удобный вариант хранения данных для использования в пограничном узле, называемом [службой файлов Azure](https://azure.microsoft.com/services/storage/files/). Они позволяют подключить общую папку службы хранения Azure к файловой системе Linux. Этот вариант может быть удобен для хранения файлов данных, скриптов R и объектов результатов, которые могут понадобиться позже, особенно если на граничном узле есть смысл использовать собственную файловую систему, а не HDFS.

Основное преимущество файлов Azure заключается в том, что общие папки можно подключить и использовать на любом устройстве с поддерживаемой операционной системой, например Windows или Linux. Например, их можно использовать в другом кластере HDInsight, принадлежащем вам или коллеге, в виртуальной машине Azure, или даже в локальной системе. Дополнительные сведения см. в разделе:

- [Использование хранилища файлов Azure с Linux](../../storage/files/storage-how-to-use-files-linux.md)
- [Использование хранилища файлов Azure в Windows](../../storage/files/storage-dotnet-how-to-use-files.md)

## <a name="next-steps"></a>Дальнейшие действия

- [Основные сведения о кластере служб машинного обучения в HDInsight](r-server-overview.md)
- [Варианты контекста вычислений для кластера служб машинного обучения в HDInsight](r-server-compute-contexts.md)
- [Использование Azure Data Lake Storage Gen2 с кластерами Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md)
