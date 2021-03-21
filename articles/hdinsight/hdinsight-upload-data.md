---
title: Отправка данных для заданий Apache Hadoop в HDInsight
description: Узнайте, как отправлять данные для заданий Apache Hadoop в HDInsight и получать к ним доступ. Используйте классический интерфейс командной строки Azure, Обозреватель службы хранилища Azure, Azure PowerShell, командную строку Hadoop или Sqoop.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdiseo17may2017,seoapr2020
ms.date: 04/27/2020
ms.openlocfilehash: a8d21b88298b6092c869b89d48fe4c259c2365b6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98931341"
---
# <a name="upload-data-for-apache-hadoop-jobs-in-hdinsight"></a>Отправка данных для заданий Apache Hadoop в HDInsight

HDInsight предоставляет распределенную файловую систему Hadoop (HDFS) в службе хранилища Azure и Azure Data Lake Storage. Это хранилище включает Gen1 и Gen2. Службы хранилища Azure и Data Lake Storage 1-го поколения и Gen2 разработаны как расширения HDFS. Они позволяют полному набору компонентов в среде Hadoop взаимодействовать непосредственно с данными, которыми он управляет. Служба хранилища Azure, Data Lake Storage 1-го поколения и Gen2 — это разные файловые системы. Системы оптимизированы для хранения данных и вычислений на этих данных. Дополнительные сведения о преимуществах использования службы хранилища Azure см. в статье [Использование службы хранилища Azure с HDInsight](hdinsight-hadoop-use-blob-storage.md). См. также [использование Data Lake Storage 1-го поколения с hdinsight](hdinsight-hadoop-use-data-lake-storage-gen1.md)и [Использование Data Lake Storage 2-го поколения с hdinsight](hdinsight-hadoop-use-data-lake-storage-gen2.md).

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы необходимо ознакомиться со следующими требованиями:

* Кластер Azure HDInsight. Инструкции см. в статье Начало [работы с Azure HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md).
* Изучите следующие статьи:
    * [Использование службы хранилища Azure с HDInsight](hdinsight-hadoop-use-blob-storage.md)
    * [Использование Data Lake Store с кластерами Azure HDInsight](hdinsight-hadoop-use-data-lake-storage-gen1.md)
    * [Использование хранилища Azure Data Lake Storage Gen2 (предварительная версия) с кластерами Azure HDInsight](hdinsight-hadoop-use-data-lake-storage-gen2.md)  

## <a name="upload-data-to-azure-storage"></a>Отправка данных в службу хранилища Azure

### <a name="utilities"></a>Служебные программы

Корпорация Майкрософт предоставляет следующие служебные программы для работы со службой хранилища Azure:

| Инструмент | Linux | OS X | Windows |
| --- |:---:|:---:|:---:|
| [Портал Azure](../storage/blobs/storage-quickstart-blobs-portal.md) |✔ |✔ |✔ |
| [Azure CLI](../storage/blobs/storage-quickstart-blobs-cli.md) |✔ |✔ |✔ |
| [Azure PowerShell](../storage/blobs/storage-quickstart-blobs-powershell.md) | | |✔ |
| [AzCopy](../storage/common/storage-use-azcopy-v10.md) |✔ | |✔ |
| [Команда Hadoop](#hadoop-command-line) |✔ |✔ |✔ |

> [!NOTE]  
> Команда Hadoop доступна только в кластере HDInsight. Она допускает загрузку данных только из локальной файловой системы в службу хранилища Azure.  

### <a name="hadoop-command-line"></a>Командная строка Hadoop

Командная строка Hadoop полезна только для хранения данных в большом двоичном объекте службы хранилища Azure, когда данные уже присутствуют на головном узле кластера.

Чтобы использовать команду Hadoop, необходимо сначала подключиться к головного узла с помощью [SSH или](hdinsight-hadoop-linux-use-ssh-unix.md)выполнив ввод.

После подключения можно использовать следующий синтаксис для отправки файла в хранилище.

```bash
hadoop fs -copyFromLocal <localFilePath> <storageFilePath>
```

Например: `hadoop fs -copyFromLocal data.txt /example/data/data.txt`

Так как файловая система по умолчанию для HDInsight находится в службе хранилища Azure,/ексампле/Дата/data.txt на самом деле находится в службе хранилища Azure. Можно также использовать следующую ссылку на файл:

`wasbs:///example/data/data.txt`

или диспетчер конфигурации служб

`wasbs://<ContainerName>@<StorageAccountName>.blob.core.windows.net/example/data/davinci.txt`

Список других команд Hadoop, работающих с файлами, см. в разделе [https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html)

> [!WARNING]  
> В кластерах Apache HBase при записи данных используется размер блока по умолчанию (256 КБ). Это подходит, когда используются API HBase или REST API. Но при попытке записать более 12 ГБ данных с помощью команды `hadoop` или `hdfs dfs` возникнет ошибка. Дополнительные сведения см. [в статье исключение хранилища для записи в BLOB-объекте](hdinsight-troubleshoot-hdfs.md#storage-exception-for-write-on-blob).

### <a name="graphical-clients"></a>Графические клиенты

Существуют также несколько приложений, которые предоставляют графический интерфейс для работы с хранилищем Azure. В следующей таблице приведен список некоторых из таких приложений:

| Клиент | Linux | OS X | Windows |
| --- |:---:|:---:|:---:|
| [Microsoft Visual Studio Tools для HDInsight](hadoop/apache-hadoop-visual-studio-tools-get-started.md#explore-linked-resources) |✔ |✔ |✔ |
| [Обозреватель службы хранилища Azure](../storage/blobs/storage-quickstart-blobs-storage-explorer.md) |✔ |✔ |✔ |
| [`Cerulea`](https://www.cerebrata.com/products/cerulean/features/azure-storage) | | |✔ |
| [CloudXplorer;](https://clumsyleaf.com/products/cloudxplorer) | | |✔ |
| [CloudBerry Explorer для Microsoft Azure](https://www.cloudberrylab.com/free-microsoft-azure-explorer.aspx) | | |✔ |
| [Cyberduck](https://cyberduck.io/) | |✔ |✔ |

## <a name="mount-azure-storage-as-local-drive"></a>Подключение службы хранилища Azure в качестве локального диска

Подробные сведения см. в статье [Mount Azure Blob Storage as Local Drive](/archive/blogs/bigdatasupport/mount-azure-blob-storage-as-local-drive) (Подключение хранилища BLOB-объектов Azure в качестве локального диска).

## <a name="upload-using-services"></a>Отправка с помощью служб

### <a name="azure-data-factory"></a>Фабрика данных Azure

Служба фабрики данных Azure — это полностью управляемая служба для составления данных: службы хранилища, обработки и перемещения в оптимизированные, адаптируемые и надежные конвейеры производства данных.

|Тип хранилища|Документация|
|----|----|
|Хранилище BLOB-объектов Azure|[Копирование данных в хранилище BLOB-объектов Azure и обратно с помощью фабрики данных Azure](../data-factory/connector-azure-blob-storage.md)|
|Хранилище Azure Data Lake Storage 1-го поколения|[Копирование данных в Azure Data Lake Storage Gen1 и из него с помощью фабрики данных Azure](../data-factory/connector-azure-data-lake-store.md)|
|Azure Data Lake Storage 2-го поколения |[Загрузка данных в Azure Data Lake Storage 2-го поколения с помощью Фабрики данных Azure](../data-factory/load-azure-data-lake-storage-gen2.md)|

### <a name="apache-sqoop"></a>Apache Sqoop

Sqoop — это средство, предназначенное для передачи данных между Hadoop и реляционными базами данных. Используйте его для импорта данных из системы управления реляционными базами данных (РСУБД), например SQL Server, MySQL или Oracle. Затем в распределенную файловую систему Hadoop (HDFS). Преобразуйте данные в Hadoop с помощью MapReduce или Hive, а затем экспортируйте данные обратно в реляционную СУБД.

Дополнительные сведения см. в разделе [Использование Sqoop с Hadoop в HDInsight](hadoop/hdinsight-use-sqoop.md).

### <a name="development-sdks"></a>Пакеты SDK для разработки

Доступ к службе хранилища Azure также можно получить с помощью пакета Azure SDK со следующих языков программирования:

* .NET
* Java
* Node.js
* PHP
* Python
* Ruby

Подробные сведения об установке пакетов SDK для Azure см. на странице [Загрузки](https://azure.microsoft.com/downloads/).

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы понимаете, как получать данные в HDInsight, ознакомьтесь со следующими статьями, чтобы изучить анализ:

* [Приступая к работе с Azure HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Отправка заданий Apache Hadoop программными средствами](hadoop/submit-apache-hadoop-jobs-programmatically.md)
* [Использование Hive и HiveQL с Hadoop в HDInsight для анализа примера файла Apache log4j](hadoop/hdinsight-use-hive.md)