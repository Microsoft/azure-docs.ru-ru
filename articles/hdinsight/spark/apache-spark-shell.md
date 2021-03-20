---
title: Использование интерактивной оболочки Spark в Azure HDInsight
description: Интерактивная оболочка Spark предоставляет процесс чтения, выполнения и вывода для поочередного выполнения команд Spark и просмотра результатов.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 02/10/2020
ms.openlocfilehash: 324852a967b5de015a9b1e9b465d4b4703e573cb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98929684"
---
# <a name="run-apache-spark-from-the-spark-shell"></a>Запуск Apache Spark из оболочки Spark

Интерактивная оболочка [Apache Spark](https://spark.apache.org/) предоставляет среду REPL (цикл чтения, выполнения и вывода) для поочередного выполнения команд Spark и просмотра результатов. Этот процесс применяется при развертывании и отладке. Spark предоставляет оболочки для каждого из поддерживаемых языков: Scala, Python и R.

## <a name="run-an-apache-spark-shell"></a>Запуск оболочки Apache Spark

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Spark предоставляет оболочки для Scala (Spark-Shell) и Python (pyspark). В сеансе SSH введите *одну* из следующих команд:

    ```bash
    spark-shell

    # Optional configurations
    # spark-shell --num-executors 4 --executor-memory 4g --executor-cores 2 --driver-memory 8g --driver-cores 4
    ```

    ```bash
    pyspark

    # Optional configurations
    # pyspark --num-executors 4 --executor-memory 4g --executor-cores 2 --driver-memory 8g --driver-cores 4
    ```

    Если вы планируете использовать любую необязательную конфигурацию, убедитесь, что вы сначала просматриваете [исключение OutOfMemoryError для Apache Spark](./apache-spark-troubleshoot-outofmemory.md).

1. Несколько простых команд. Выберите соответствующий язык:

    ```spark-shell
    val textFile = spark.read.textFile("/example/data/fruits.txt")
    textFile.first()
    textFile.filter(line => line.contains("apple")).show()
    ```

    ```pyspark
    textFile = spark.read.text("/example/data/fruits.txt")
    textFile.first()
    textFile.filter(textFile.value.contains("apple")).show()
    ```

1. Запросите CSV-файл. Обратите внимание, что язык ниже работает для `spark-shell` и `pyspark` .

    ```scala
    spark.read.csv("/HdiSamples/HdiSamples/SensorSampleData/building/building.csv").show()
    ```

1. Запросите CSV-файл и сохраните результаты в переменной:

    ```spark-shell
    var data = spark.read.format("csv").option("header", "true").option("inferSchema", "true").load("/HdiSamples/HdiSamples/SensorSampleData/building/building.csv")
    ```

    ```pyspark
    data = spark.read.format("csv").option("header", "true").option("inferSchema", "true").load("/HdiSamples/HdiSamples/SensorSampleData/building/building.csv")
    ```

1. Отобразить результаты:

    ```spark-shell
    data.show()
    data.select($"BuildingID", $"Country").show(10)
    ```

    ```pyspark
    data.show()
    data.select("BuildingID", "Country").show(10)
    ```

1. Выход

    ```spark-shell
    :q
    ```

    ```pyspark
    exit()
    ```

## <a name="sparksession-and-sparkcontext-instances"></a>Экземпляры SparkSession и SparkContext

По умолчанию экземпляры SparkSession и SparkContext создаются автоматически при запуске оболочки Spark.

Чтобы получить доступ к экземпляру SparkSession, введите команду `spark`. А чтобы получить доступ к экземпляру SparkContext, введите команду `sc`.

## <a name="important-shell-parameters"></a>Важные параметры оболочки

Команда оболочки Spark ( `spark-shell` или `pyspark` ) поддерживает множество параметров командной строки. Чтобы просмотреть полный список параметров, запустите оболочку Spark с параметром `--help`. Некоторые из этих параметров могут применяться только к `spark-submit` , для которых оболочка Spark заносится в оболочку.

| switch | description; | пример |
| --- | --- | --- |
| --master MASTER_URL | Позволяет задать основной URL-адрес. В HDInsight всегда используется значение `yarn`. | `--master yarn`|
| --jars JAR_LIST | Список разделенных запятыми локальных JAR-файлов, которые добавляются в пути к классам драйвера и исполнителя. В HDInsight этот список состоит из путей к файловой системе по умолчанию в службе хранилища Azure или в Data Lake Storage. | `--jars /path/to/examples.jar` |
| --packages MAVEN_COORDS | Список разделенных запятыми координат Maven для JAR-файлов, которые добавляются в пути к классам драйвера и исполнителя. Выполняет поиск локального репозитория Maven, затем центрального репозитория Maven или же любого дополнительного удаленного репозитория, заданного с помощью параметра `--repositories`. Формат координат: *groupId*:*artifactId*:*version*. | `--packages "com.microsoft.azure:azure-eventhubs:0.14.0"`|
| --py-files LIST | (Только для Python.) Список разделенных запятыми ZIP-, EGG- или PY-файлов, которые размещаются в PYTHONPATH. | `--pyfiles "samples.py"` |

## <a name="next-steps"></a>Дальнейшие действия

- Вводную информацию см. в статье [Общие сведения об Apache Spark в Azure HDInsight](apache-spark-overview.md).
- Сведения о работе со SparkSQL и кластерами Spark см. в статье [Создание кластера Apache Spark в Azure HDInsight](apache-spark-jupyter-spark-sql.md).
- Сведения о создании приложений для обработки потоковых данных с помощью Spark см. в статье [Общие сведения о потоковой передаче Apache Spark](apache-spark-streaming-overview.md).
