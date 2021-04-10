---
title: Операции Apache Spark, поддерживаемые Hive Warehouse Connector в Azure HDInsight
description: Узнайте о различных возможностях Hive Warehouse Connector в Azure HDInsight.
author: nis-goel
ms.author: nisgoel
ms.service: hdinsight
ms.topic: how-to
ms.date: 05/22/2020
ms.openlocfilehash: 1799aff8bff96d404ddcbefbf58a5f5014cdba6a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104871594"
---
# <a name="apache-spark-operations-supported-by-hive-warehouse-connector-in-azure-hdinsight"></a>Операции Apache Spark, поддерживаемые Hive Warehouse Connector в Azure HDInsight

В этой статье показаны операции на основе Spark, поддерживаемые Hive Warehouse Connector (HWC). Все представленные ниже примеры будут выполняться в оболочке Apache Spark.

## <a name="prerequisite"></a>Предварительные требования

Выполните инструкции из статьи [Настройка Hive Warehouse Connector](./apache-hive-warehouse-connector.md#hive-warehouse-connector-setup).

## <a name="getting-started"></a>Начало работы

Чтобы начать сеанс spark-shell, выполните указанные ниже действия.

1. С помощью команды [ssh command](../hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру Apache Spark. Измените приведенную ниже команду, заменив CLUSTERNAME именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. В сеансе SSH выполните следующую команду, чтобы отметить версию `hive-warehouse-connector-assembly`:

    ```bash
    ls /usr/hdp/current/hive_warehouse_connector
    ```

1. Измените приведенный ниже код, указав версию `hive-warehouse-connector-assembly`, указанную выше. Затем выполните команду, чтобы запустить оболочку Spark:

    ```bash
    spark-shell --master yarn \
    --jars /usr/hdp/current/hive_warehouse_connector/hive-warehouse-connector-assembly-<STACK_VERSION>.jar \
    --conf spark.security.credentials.hiveserver2.enabled=false
    ```

1. После запуска оболочки Spark можно запустить экземпляр Hive Warehouse Connector с помощью следующих команд:

    ```scala
    import com.hortonworks.hwc.HiveWarehouseSession
    val hive = HiveWarehouseSession.session(spark).build()
    ```

## <a name="creating-spark-dataframes-using-hive-queries"></a>Создание кадров данных Spark с помощью запросов Hive

Результаты всех запросов, использующих библиотеку HWC, возвращаются в виде кадров данных. В следующих примерах показано, как создать базовый запрос Hive.

```scala
hive.setDatabase("default")
val df = hive.executeQuery("select * from hivesampletable")
df.filter("state = 'Colorado'").show()
```

Результаты запроса представляют собой фрагменты данных Spark, которые можно использовать с библиотеками Spark, такими как MLIB и SparkSQL.

## <a name="writing-out-spark-dataframes-to-hive-tables"></a>Запись кадров данных Spark в таблицы Hive

Spark изначально не поддерживает запись в управляемые Hive таблицы ACID. Однако с помощью HWC можно записать в таблицу Hive любой кадр данных. Эта возможность демонстрируется в следующем примере.

1. Создайте таблицу с именем `sampletable_colorado` и укажите ее столбцы с помощью следующей команды:

    ```scala
    hive.createTable("sampletable_colorado").column("clientid","string").column("querytime","string").column("market","string").column("deviceplatform","string").column("devicemake","string").column("devicemodel","string").column("state","string").column("country","string").column("querydwelltime","double").column("sessionid","bigint").column("sessionpagevieworder","bigint").create()
    ```

1. Отфильтруйте таблицу `hivesampletable`, где столбец `state` равен `Colorado`. Этот запрос Hive возвращает кадр данных Spark, который сохраняется в таблице Hive `sampletable_colorado` с помощью функции `write`.

    ```scala
    hive.table("hivesampletable").filter("state = 'Colorado'").write.format("com.hortonworks.spark.sql.hive.llap.HiveWarehouseConnector").mode("append").option("table","sampletable_colorado").save()
    ```

1. Просмотрите результаты с помощью следующей команды:

    ```scala
    hive.table("sampletable_colorado").show()
    ```
    
    :::image type="content" source="./media/apache-hive-warehouse-connector/hive-warehouse-connector-show-hive-table.png" alt-text="Отображение таблицы Hive с помощью Hive Warehouse Connector" border="true":::


## <a name="structured-streaming-writes"></a>Операции записи посредством структурированной потоковой передачи

С помощью Hive Warehouse Connector можно использовать потоковую передачу Spark для записи данных в таблицы Hive.

> [!IMPORTANT]
> Операции записи посредством структурированной потоковой передачи не поддерживаются в кластерах Spark 4.0 с поддержкой ESP.

Для принятия данных из потока Spark через порт localhost 9999 в таблицу Hive посредством Hive Warehouse Connector выполните указанные ниже действия.

1. В открытой оболочке Spark запустите поток Spark с помощью следующей команды:

    ```scala
    val lines = spark.readStream.format("socket").option("host", "localhost").option("port",9999).load()
    ```

1. Создайте данные для созданного потока Spark, выполнив указанные ниже действия.
    1. Откройте второй сеанс SSH в том же кластере Spark.
    1. В командной строке введите `nc -lk 9999`. Эта команда использует служебную программу netcat для отправки данных из командной строки на указанный порт.

1. Вернитесь к первому сеансу SSH и создайте таблицу Hive для хранения данных потоковой передачи. В оболочке Spark введите следующую команду:

    ```scala
    hive.createTable("stream_table").column("value","string").create()
    ```

1. Затем запишите данные потоковой передачи в созданную таблицу с помощью следующей команды:

    ```scala
    lines.filter("value = 'HiveSpark'").writeStream.format("com.hortonworks.spark.sql.hive.llap.streaming.HiveStreamingDataSource").option("database", "default").option("table","stream_table").option("metastoreUri",spark.conf.get("spark.datasource.hive.warehouse.metastoreUri")).option("checkpointLocation","/tmp/checkpoint1").start()
    ```

    >[!Important]
    > В настоящее время параметры `metastoreUri` и `database` должны задаваться вручную из-за известной проблемы в Apache Spark. Дополнительные сведения об этой проблеме см. на странице [SPARK-25460](https://issues.apache.org/jira/browse/SPARK-25460).

1. Вернитесь ко второму сеансу SSH и введите следующие значения:

    ```bash
    foo
    HiveSpark
    bar
    ```

1. Вернитесь к первому сеансу SSH и обратите внимание на кратковременные действия. Для просмотра данных используйте следующую команду:

    ```scala
    hive.table("stream_table").show()
    ```

Нажмите клавиши **CTRL+C**, чтобы остановить работу netcat во втором сеансе SSH. Используйте `:q`, чтобы выйти из оболочки Spark в первом сеансе SSH.

## <a name="next-steps"></a>Дальнейшие действия

* [Интеграция HWC с Apache Spark и Apache Hive](./apache-hive-warehouse-connector.md)
* [Use Interactive Query with HDInsight](./apache-interactive-query-get-started.md) (Использование Interactive Query в HDInsight)
* [Интеграция HWC с Apache Zeppelin](./apache-hive-warehouse-connector-zeppelin.md)
