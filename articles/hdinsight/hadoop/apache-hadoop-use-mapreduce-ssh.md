---
title: Подключение MapReduce и SSH с Apache Hadoop — Azure HDInsight
description: Узнайте, как использовать SSH для выполнения заданий MapReduce с помощью Apache Hadoop в HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 01/10/2020
ms.openlocfilehash: 991b91d1feee185d17bbf01266e0392f347e1a66
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98939633"
---
# <a name="use-mapreduce-with-apache-hadoop-on-hdinsight-with-ssh"></a>Использование MapReduce с Apache Hadoop в HDInsight с помощью SSH

[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

Узнайте, как отправлять задания MapReduce в HDInsight с помощью подключения SSH.

> [!NOTE]
> Если вы уже знаете, как использовать серверы Apache Hadoop на платформе Linux, но не знакомы с HDInsight, ознакомьтесь со статьей [Сведения об использовании HDInsight в Linux](../hdinsight-hadoop-linux-information.md).

## <a name="prerequisites"></a>Предварительные требования

Кластер Apache Hadoop в HDInsight. См. [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](../hdinsight-hadoop-create-linux-clusters-portal.md).

## <a name="use-hadoop-commands"></a>Использование команд Hadoop

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. После подключения к кластеру HDInsight используйте следующую команду, чтобы запустить задание MapReduce:

    ```bash
    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/WordCountOutput
    ```

    Эта команда запускает класс `wordcount`, который содержится в файле `hadoop-mapreduce-examples.jar`. Она использует документ `/example/data/gutenberg/davinci.txt` в качестве входных данных, а выходные данные хранятся в `/example/data/WordCountOutput`.

    > [!NOTE]
    > Дополнительную информацию об этом задании MapReduce и примере данных см. в статье [Использование MapReduce в Apache Hadoop в HDInsight](hdinsight-use-mapreduce.md).

    Задание выдает информацию о ходе обработки, а по завершении задания возвращает информацию, аналогичную приведенной ниже:

    ```output
    File Input Format Counters
    Bytes Read=1395666
    File Output Format Counters
    Bytes Written=337623
    ```

1. По завершении задания воспользуйтесь следующей командой, чтобы просмотреть выходные файлы:

    ```bash
    hdfs dfs -ls /example/data/WordCountOutput
    ```

    Эта команда отображает два файла — `_SUCCESS` и `part-r-00000`. Файл `part-r-00000` содержит выходные данные этого задания.

    > [!NOTE]  
    > Некоторые задания MapReduce могут разделять результаты на несколько файлов **part-r-№№№№№** . В этом случае используйте суффикс №№№№№, чтобы определить порядок файлов.

1. Чтобы просмотреть выходные данные, используйте следующую команду:

    ```bash
    hdfs dfs -cat /example/data/WordCountOutput/part-r-00000
    ```

    Она отобразит список слов, содержащихся в файле **wasb://example/data/gutenberg/davinci.txt**, а также количество вхождений каждого из них. Ниже приведен пример данных, содержащихся в файле.

    ```output
    wreathed        3
    wreathing       1
    wreaths         1
    wrecked         3
    wrenching       1
    wretched        6
    wriggling       1
    ```

## <a name="next-steps"></a>Дальнейшие действия

Как видите, команды Hadoop позволяют с легкостью выполнять задания MapReduce в кластере HDInsight и просматривать выходные данные задания. Дополнительная информация о других способах работы с Hadoop в HDInsight:

* [Использование MapReduce в Hadoop в HDInsight](hdinsight-use-mapreduce.md)
* [Использование Apache Hive с Apache Hadoop в HDInsight](hdinsight-use-hive.md)
