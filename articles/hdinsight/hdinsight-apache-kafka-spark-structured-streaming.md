---
title: Руководство по Потоковая передача Apache Spark c Apache Kafka в Azure HDInsight
description: Узнайте об использовании потоковой передачи Apache Spark для двунаправленного обмена данными с Apache Kafka. В этом учебнике показано, как выполнить потоковую передачу данных, используя записную книжку Jupyter Notebook из Spark в HDInsight.
ms.service: hdinsight
ms.topic: tutorial
ms.custom: hdinsightactive,seodec18,seoapr2020
ms.date: 04/22/2020
ms.openlocfilehash: 72c82e8f425b05dde37352225dd7167b089ba48a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104868466"
---
# <a name="tutorial-use-apache-spark-structured-streaming-with-apache-kafka-on-hdinsight"></a>Руководство по Использование структурированной потоковой передачи Apache Spark с Apache Kafka в HDInsight

В этом руководстве показано, как использовать [структурированную потоковую передачу Apache Spark](https://spark.apache.org/docs/latest/structured-streaming-programming-guide), чтобы считывать и записывать данные с использованием [Apache Kafka](./kafka/apache-kafka-introduction.md) в Azure HDInsight.

Структурированная потоковая передача Spark — это механизм обработки потока, встроенный в Spark SQL. Он позволяет выражать потоковые вычисления так же, как пакетные вычисления статических данных.  

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Использование шаблона Azure Resource Manager для создания кластеров.
> * Использование структурированной потоковой передачи Spark с Kafka.

Выполнив инструкции, не забудьте удалить кластеры, чтобы избежать ненужных расходов.

## <a name="prerequisites"></a>Предварительные требования

* jq — обработчик командной строки JSON.  См. раздел [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/).

* Опыт работы с [записными книжками Jupyter](https://jupyter.org/) и Spark в HDInsight. Дополнительные сведения см. в статье [Руководство. Загрузка данных и выполнение запросов в кластере Apache Spark в Azure HDInsight](spark/apache-spark-load-data-run-query.md).

* Опыт работы с языком программирования Scala. Код, используемый в этом руководстве, написан на языке Scala.

* Опыт создания разделов Kafka. Дополнительные сведения см. в документе [Краткое руководство по созданию Apache Kafka в кластере HDInsight](kafka/apache-kafka-get-started.md).

> [!IMPORTANT]  
> Чтобы выполнить действия, описанные в этом документе, необходимо иметь группу ресурсов Azure, которая содержит кластеры Spark и Kafka в HDInsight. Оба этих кластера находятся в виртуальной сети Azure, что позволяет кластеру Spark напрямую обмениваться данными с кластером Kafka.
>
> Для удобства в этом документе есть ссылка на шаблон, с помощью которого можно создать все необходимые ресурсы Azure.
>
> Дополнительные сведения об использовании HDInsight в виртуальной сети см. в статье о[планирование виртуальной сети для Azure HDInsight](hdinsight-plan-virtual-network-deployment.md).

## <a name="structured-streaming-with-apache-kafka"></a>Структурированная потоковая передача с помощью Apache Kafka

Структурированная потоковая передача Spark — это механизм обработки потока, встроенный в ядро Spark SQL. При использовании структурированной потоковой передачи можно писать запросы потоковой передачи так же, как и пакетные запросы.

В следующих фрагментах кода демонстрируется чтение данных из Kafka и их сохранение в файле. Первый фрагмент кода — это пакетная операция, а второй — операция потоковой передачи.

```scala
// Read a batch from Kafka
val kafkaDF = spark.read.format("kafka")
                .option("kafka.bootstrap.servers", kafkaBrokers)
                .option("subscribe", kafkaTopic)
                .option("startingOffsets", "earliest")
                .load()

// Select data and write to file
kafkaDF.select(from_json(col("value").cast("string"), schema) as "trip")
                .write
                .format("parquet")
                .option("path","/example/batchtripdata")
                .option("checkpointLocation", "/batchcheckpoint")
                .save()
```

```scala
// Stream from Kafka
val kafkaStreamDF = spark.readStream.format("kafka")
                .option("kafka.bootstrap.servers", kafkaBrokers)
                .option("subscribe", kafkaTopic)
                .option("startingOffsets", "earliest")
                .load()

// Select data from the stream and write to file
kafkaStreamDF.select(from_json(col("value").cast("string"), schema) as "trip")
                .writeStream
                .format("parquet")
                .option("path","/example/streamingtripdata")
                .option("checkpointLocation", "/streamcheckpoint")
                .start.awaitTermination(30000)
```

В обоих фрагментах кода данные считываются из Kafka и записываются в файл. Ниже показаны различия в примерах.

| Пакетная служба Azure | Потоковая передача |
| --- | --- |
| `read` | `readStream` |
| `write` | `writeStream` |
| `save` | `start` |

Операция потоковой передачи также использует `awaitTermination(30000)`, что останавливает потоковую передачу спустя 30 000 мс.

Чтобы использовать структурированную потоковую передачу с помощью Kafka, ваш проект должен иметь зависимость в пакете `org.apache.spark : spark-sql-kafka-0-10_2.11`. Версия этого пакета должна соответствовать версии Spark в HDInsight. Для Spark версии 2.2.0 (доступна в HDInsight 3.6) можно найти сведения о зависимости для разных типов проекта по адресу [https://search.maven.org/#artifactdetails%7Corg.apache.spark%7Cspark-sql-kafka-0-10_2.11%7C2.2.0%7Cjar](https://search.maven.org/#artifactdetails%7Corg.apache.spark%7Cspark-sql-kafka-0-10_2.11%7C2.2.0%7Cjar).

Для приложения Jupyter Notebook, используемого в рамках этого руководства, следующая ячейка передает зависимость пакета:

```
%%configure -f
{
    "conf": {
        "spark.jars.packages": "org.apache.spark:spark-sql-kafka-0-10_2.11:2.2.0",
        "spark.jars.excludes": "org.scala-lang:scala-reflect,org.apache.spark:spark-tags_2.11"
    }
}
```

## <a name="create-the-clusters"></a>Создание кластеров

Apache Kafka в HDInsight не предоставляет доступ к брокерам Kafka через общедоступный сегмент Интернета. Все ресурсы, которые использует Kafka, должны находиться в одной виртуальной сети Azure. В рамках этого руководства кластеры Kafka и Spark расположены в одной и той же виртуальной сети Azure.

На следующей схеме показано, как происходит обмен данными между Spark и Kafka:

:::image type="content" source="./media/hdinsight-apache-kafka-spark-structured-streaming/apache-spark-kafka-vnet.png" alt-text="Схема кластеров Spark и Kafka в виртуальной сети Azure" border="false":::

> [!NOTE]  
> Служба Kafka ограничена обменом данными в пределах виртуальной сети. Другие службы в кластере, например SSH и Ambari, могут быть доступны через Интернет. Дополнительные сведения об общих портах, доступных в HDInsight, см. в статье [Порты и универсальные коды ресурсов (URI), используемые кластерами HDInsight](hdinsight-hadoop-port-settings-for-services.md).

Чтобы создать виртуальную сеть Azure, а затем создать кластеры Kafka и Spark в ее пределах, выполните такие действия:

1. Нажмите эту кнопку, чтобы войти в Azure и открыть шаблон на портале Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fhdinsight-spark-kafka-structured-streaming%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apache-kafka-spark-structured-streaming/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Шаблон Azure Resource Manager доступен по адресу **https://raw.githubusercontent.com/Azure-Samples/hdinsight-spark-kafka-structured-streaming/master/azuredeploy.json** .

    Этот шаблон создает следующие ресурсы:

   * Kafka в кластере HDInsight 3.6;
   * Spark 2.2.0 в кластере HDInsight 3.6;
   * виртуальную сеть Azure, содержащую кластеры HDInsight.

     > [!IMPORTANT]  
     > Записная книжка структурированной потоковой передачи, используемая в этом руководстве, требует Spark 2.2.0 в HDInsight 3.6. Если используется более ранняя версия Spark в HDInsight, возникнут ошибки при использовании этой записной книжки.

2. Заполните раздел **Настроенный шаблон**, используя следующие сведения:

    | Параметр | Значение |
    | --- | --- |
    | Подписка | Ваша подписка Azure. |
    | Группа ресурсов | Группа ресурсов, в которой содержатся ресурсы. |
    | Расположение | Регион Azure, в котором создаются ресурсы. |
    | Spark Cluster Name (Имя кластера Spark) | Имя кластера Spark. Первые шесть символов должны отличаться от имени кластера Kafka. |
    | Kafka Cluster Name (Имя кластера Kafka) | Имя кластера Kafka. Первые шесть символов должны отличаться от имени кластера Spark. |
    | Имя пользователя для входа в кластер | Имя администратора кластеров. |
    | Пароль для входа в кластер | Пароль администратора кластеров. |
    | Имя пользователя SSH | Пользователь SSH, который создается для кластеров. |
    | Пароль SSH | Пароль пользователя SSH. |

    :::image type="content" source="./media/hdinsight-apache-kafka-spark-structured-streaming/spark-kafka-template.png" alt-text="Снимок экрана с настроенным шаблоном":::

3. Прочтите **условия использования** и установите флажок **Я принимаю указанные выше условия**.

4. Щелкните **Приобрести**.

> [!NOTE]  
> Создание кластеров может занять до 20 минут.

## <a name="use-spark-structured-streaming"></a>Использование структурированной потоковой передачи Spark

В этом примере показано, как использовать структурированную потоковую передачу Spark с Kafka в HDInsight. Для этого примера используются данные о поездках в такси, предоставленные сайтом города Нью-Йорк.  Набор данных, используемых в этой записной книжке, взят на странице [2016 Green Taxi Trip Data](https://data.cityofnewyork.us/Transportation/2016-Green-Taxi-Trip-Data/hvrh-b6nb).

1. Выполните сбор информации об узле. Воспользуйтесь приведенными ниже командами curl и [jq](https://stedolan.github.io/jq/), чтобы получить сведения об узлах Kafka ZooKeeper и брокера. Эти команды предназначены для командной строки Windows. Команды для других сред будут немного отличаться. Замените `KafkaCluster` именем кластера Kafka, а `KafkaPassword` — паролем для входа в кластер. Также замените `C:\HDI\jq-win64.exe` фактическим путем к папке установки jq. Введите команды в командной строке Windows и сохраните выходные данные для использования в следующих шагах.

    ```cmd
    REM Enter cluster name in lowercase

    set CLUSTERNAME=KafkaCluster
    set PASSWORD=KafkaPassword

    curl -u admin:%PASSWORD% -G "https://%CLUSTERNAME%.azurehdinsight.net/api/v1/clusters/%CLUSTERNAME%/services/ZOOKEEPER/components/ZOOKEEPER_SERVER" | C:\HDI\jq-win64.exe -r "["""\(.host_components[].HostRoles.host_name):2181"""] | join(""",""")"

    curl -u admin:%PASSWORD% -G "https://%CLUSTERNAME%.azurehdinsight.net/api/v1/clusters/%CLUSTERNAME%/services/KAFKA/components/KAFKA_BROKER" | C:\HDI\jq-win64.exe -r "["""\(.host_components[].HostRoles.host_name):9092"""] | join(""",""")"
    ```

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net/jupyter`, где `CLUSTERNAME` — это имя вашего кластера. При появлении запроса введите имя администратора кластера и пароль, которые использовались при создании кластера.

1. Выберите **New > Spark** (Создать > Spark), чтобы создать записную книжку.

1. При потоковой передаче данных Spark использует микропакеты, то есть данные передаются в пакетах, после чего такие пакеты обрабатываются исполнителями. Если время ожидания простоя исполнителя меньше времени, затрачиваемого на обработку пакета, исполнители будут постоянно добавляться и удаляться. Если время ожидания простоя исполнителя превышает длительность передачи пакета, такой исполнитель не будет удаляться. Поэтому **мы рекомендуем отключить динамическое распределение, задав для параметра spark.dynamicAllocation.enabled значение false, при запуске приложений с потоковой передачей данных**.

    Загрузите пакеты, используемые Notebook, указав следующие сведения в ячейке Notebook. Выполните команду, нажав клавиши **CTRL+ВВОД**.

    ```configuration
    %%configure -f
    {
        "conf": {
            "spark.jars.packages": "org.apache.spark:spark-sql-kafka-0-10_2.11:2.2.0",
            "spark.jars.excludes": "org.scala-lang:scala-reflect,org.apache.spark:spark-tags_2.11",
            "spark.dynamicAllocation.enabled": false
        }
    }
    ```

1. Создайте раздел Kafka. Измените указанную ниже команду, заменив `YOUR_ZOOKEEPER_HOSTS` сведениями об узле Zookeeper, которые вы извлекли на первом шаге. Введите измененную команду в Jupyter Notebook, чтобы создать раздел `tripdata`.

    ```scala
    %%bash
    export KafkaZookeepers="YOUR_ZOOKEEPER_HOSTS"

    /usr/hdp/current/kafka-broker/bin/kafka-topics.sh --create --replication-factor 3 --partitions 8 --topic tripdata --zookeeper $KafkaZookeepers
    ```

1. Получите данные о поездках в такси. Введите команду в следующей ячейке, чтобы загрузить данные о поездках в такси в Нью-Йорке. Данные загружаются в таблицу данных, которая потом отображается в виде выходных данных ячейки.

    ```scala
    import spark.implicits._

    // Load the data from the New York City Taxi data REST API for 2016 Green Taxi Trip Data
    val url="https://data.cityofnewyork.us/resource/pqfs-mqru.json"
    val result = scala.io.Source.fromURL(url).mkString

    // Create a dataframe from the JSON data
    val taxiDF = spark.read.json(Seq(result).toDS)

    // Display the dataframe containing trip data
    taxiDF.show()
    ```

1. Укажите сведения об узлах брокера Kafka. Замените `YOUR_KAFKA_BROKER_HOSTS` информацией об узлах брокера, которую вы извлекли на первом шаге.  Введите измененную команду в следующей ячейке Jupyter Notebook.

    ```scala
    // The Kafka broker hosts and topic used to write to Kafka
    val kafkaBrokers="YOUR_KAFKA_BROKER_HOSTS"
    val kafkaTopic="tripdata"

    println("Finished setting Kafka broker and topic configuration.")
    ```

1. Отправьте данные в Kafka. В приведенной ниже команде поле `vendorid` используется в качестве значения ключа для сообщения Kafka. Kafka использует ключ при секционировании данных. Все поля хранятся в сообщении Kafka в виде строковых значений JSON. Введите следующую команду в Jupyter, чтобы сохранить данные в Kafka с помощью пакетного запроса.

    ```scala
    // Select the vendorid as the key and save the JSON string as the value.
    val query = taxiDF.selectExpr("CAST(vendorid AS STRING) as key", "to_JSON(struct(*)) AS value").write.format("kafka").option("kafka.bootstrap.servers", kafkaBrokers).option("topic", kafkaTopic).save()

    println("Data sent to Kafka")
    ```

1. Объявите схему. В приведенной ниже команде показано, как использовать схему при считывании данных JSON из Kafka. Введите команду в следующую ячейку Jupyter.

    ```scala
    // Import bits useed for declaring schemas and working with JSON data
    import org.apache.spark.sql._
    import org.apache.spark.sql.types._
    import org.apache.spark.sql.functions._

    // Define a schema for the data
    val schema = (new StructType).add("dropoff_latitude", StringType).add("dropoff_longitude", StringType).add("extra", StringType).add("fare_amount", StringType).add("improvement_surcharge", StringType).add("lpep_dropoff_datetime", StringType).add("lpep_pickup_datetime", StringType).add("mta_tax", StringType).add("passenger_count", StringType).add("payment_type", StringType).add("pickup_latitude", StringType).add("pickup_longitude", StringType).add("ratecodeid", StringType).add("store_and_fwd_flag", StringType).add("tip_amount", StringType).add("tolls_amount", StringType).add("total_amount", StringType).add("trip_distance", StringType).add("trip_type", StringType).add("vendorid", StringType)
    // Reproduced here for readability
    //val schema = (new StructType)
    //   .add("dropoff_latitude", StringType)
    //   .add("dropoff_longitude", StringType)
    //   .add("extra", StringType)
    //   .add("fare_amount", StringType)
    //   .add("improvement_surcharge", StringType)
    //   .add("lpep_dropoff_datetime", StringType)
    //   .add("lpep_pickup_datetime", StringType)
    //   .add("mta_tax", StringType)
    //   .add("passenger_count", StringType)
    //   .add("payment_type", StringType)
    //   .add("pickup_latitude", StringType)
    //   .add("pickup_longitude", StringType)
    //   .add("ratecodeid", StringType)
    //   .add("store_and_fwd_flag", StringType)
    //   .add("tip_amount", StringType)
    //   .add("tolls_amount", StringType)
    //   .add("total_amount", StringType)
    //   .add("trip_distance", StringType)
    //   .add("trip_type", StringType)
    //   .add("vendorid", StringType)

    println("Schema declared")
    ```

1. Выберите данные и начните потоковую передачу. Следующая команда извлекает данные из Kafka с помощью пакетного запроса. Затем запишите результаты в HDFS в кластере Spark. В этом примере `select` получает сообщение (значение поля) из Kafka и применяет схему к нему. Затем данные записываются в HDFS (WASB или ADL) в формате Parquet. Введите команду в следующую ячейку Jupyter.

    ```scala
    // Read a batch from Kafka
    val kafkaDF = spark.read.format("kafka").option("kafka.bootstrap.servers", kafkaBrokers).option("subscribe", kafkaTopic).option("startingOffsets", "earliest").load()

    // Select data and write to file
    val query = kafkaDF.select(from_json(col("value").cast("string"), schema) as "trip").write.format("parquet").option("path","/example/batchtripdata").option("checkpointLocation", "/batchcheckpoint").save()

    println("Wrote data to file")
    ```

1. Чтобы проверить, созданы ли файлы, введите команду в следующую ячейку Jupyter. Отобразится список файлов из каталога `/example/batchtripdata`.

    ```scala
    %%bash
    hdfs dfs -ls /example/batchtripdata
    ```

1. В предыдущем примере использовался пакетный запрос. В следующей команде показано, как сделать то же самое, используя запрос потоковой передачи. Введите команду в следующую ячейку Jupyter.

    ```scala
    // Stream from Kafka
    val kafkaStreamDF = spark.readStream.format("kafka").option("kafka.bootstrap.servers", kafkaBrokers).option("subscribe", kafkaTopic).option("startingOffsets", "earliest").load()

    // Select data from the stream and write to file
    kafkaStreamDF.select(from_json(col("value").cast("string"), schema) as "trip").writeStream.format("parquet").option("path","/example/streamingtripdata").option("checkpointLocation", "/streamcheckpoint").start.awaitTermination(30000)
    println("Wrote data to file")
    ```

1. Выполните следующую ячейку, чтобы проверить, записаны ли файлы в ответ на запрос потоковой передачи.

    ```scala
    %%bash
    hdfs dfs -ls /example/streamingtripdata
    ```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить ресурсы, созданные при работе с этим руководством, удалите группу ресурсов. При удалении группы ресурсов также удаляются связанные с ней кластер HDInsight и другие ресурсы.

Чтобы удалить группу ресурсов с помощью портала Azure, сделайте следующее:

1. На [портале Azure](https://portal.azure.com/) разверните меню слева, чтобы открыть меню служб, а затем выберите __Группы ресурсов__, чтобы просмотреть список групп ресурсов.
2. Найдите группу ресурсов, которую нужно удалить, и щелкните правой кнопкой мыши кнопку __Дополнительно__ (…) справа от списка.
3. Выберите __Удалить группу ресурсов__ и подтвердите выбор.

> [!WARNING]  
> Начисление оплаты начинается после создания кластера HDInsight и прекращается только после его удаления. Кластеры оплачиваются поминутно, поэтому всегда следует удалять кластер, когда он больше не нужен.
>
> При удалении кластера Kafka в HDInsight удаляются все данные, хранящиеся в Kafka.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как использовать структурированную потоковую передачу Apache Spark. Запись и чтение данных из Apache Kafka в HDInsight. Воспользуйтесь следующей ссылкой, чтобы узнать, как использовать Apache Storm с Kafka.

> [!div class="nextstepaction"]
> [Использование Apache Storm с Apache Kafka](hdinsight-apache-storm-with-kafka.md)
