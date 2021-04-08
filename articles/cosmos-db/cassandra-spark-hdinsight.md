---
title: Доступ к API Cassandra для Azure Cosmos DB из Spark в YARN с HDInsight
description: В этой статье описывается, как работать с API Cassandra для Azure Cosmos DB из Spark в YARN с HDInsight.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: e09ac39037024b5c360af4bb6ada26590b142924
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97359601"
---
# <a name="access-azure-cosmos-db-cassandra-api-from-spark-on-yarn-with-hdinsight"></a>Доступ к API Cassandra для Azure Cosmos DB из Spark в YARN с HDInsight
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

В этой статье описывается, как обращаться к API Cassandra для Azure Cosmos DB из Spark в YARN с HDInsight, используя оболочку Spark. HDInsight — это платформа как услуга Hadoop Hortonworks в Azure корпорации Майкрософт, которая использует хранилище объектов для HDFS и поставляется в нескольких конфигурациях, включая решение [Spark](../hdinsight/spark/apache-spark-overview.md).  Хотя в этом документе описывается HDInsight Spark, он подходит для всех дистрибутивов Hadoop.  

## <a name="prerequisites"></a>Предварительные требования

* [Подготовка API Cassandra для Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account)

* [Просмотр основных сведений о подключении к API Cassandra для Azure Cosmos DB](cassandra-spark-generic.md)

* [Подготовка кластера HDInsight Spark](../hdinsight/spark/apache-spark-jupyter-spark-sql.md)

* [Просмотр примеров кода для работы с API Cassandra](cassandra-spark-generic.md#next-steps)

* [Использование при необходимости cqlsh для проверки](cassandra-spark-generic.md#connecting-to-azure-cosmos-db-cassandra-api-from-spark)

* **Настройка API Cassandra в Spark2**. Для инициализации соединителя Spark для Cassandra как части контекста Spark необходимы сведения о подключении Cassandra. При запуске Jupyter Notebook сеанс и контекст Spark уже инициализированы. Не рекомендуется останавливать запуск и повторно инициализировать контекст Spark, если только он не выполнен с использованием наборов конфигурации, заданных для запуска Jupyter Notebook по умолчанию для HDInsight. Одним из способов обхода является добавление сведений об экземпляре Cassandra непосредственно в конфигурацию службы Ambari, Spark2. Это требуется выполнить один раз для каждого кластера, на котором нужно перезапустить службу Spark2.
 
  1. Перейдите к службе Ambari, Spark2 и выберите конфигурации.

  2. Затем перейдите к пользовательской конфигурации spark2-defaults и добавьте новое свойство с приведенным ниже значением, после чего перезапустите службу Spark2.

  ```scala
  spark.cassandra.connection.host=YOUR_COSMOSDB_ACCOUNT_NAME.cassandra.cosmosdb.azure.com<br>
  spark.cassandra.connection.port=10350<br>
  spark.cassandra.connection.ssl.enabled=true<br>
  spark.cassandra.auth.username=YOUR_COSMOSDB_ACCOUNT_NAME<br>
  spark.cassandra.auth.password=YOUR_COSMOSDB_KEY<br>
  ```

## <a name="access-azure-cosmos-db-cassandra-api-from-spark-shell"></a>Доступ к API Cassandra для Azure Cosmos DB из оболочки Spark

Оболочка Spark используется для тестирования и исследования.

* Запустите оболочку Spark с обязательными зависимостями Maven, совместимыми с версией Spark кластера.

  ```scala
  spark-shell --packages "com.datastax.spark:spark-cassandra-connector_2.11:2.3.0,com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0"
  ```

* Выполните несколько операций DDL и DML.

  ```scala
  import org.apache.spark.rdd.RDD
  import org.apache.spark.{SparkConf, SparkContext}

  import spark.implicits._
  import org.apache.spark.sql.functions._
  import org.apache.spark.sql.Column
  import org.apache.spark.sql.types.{StructType, StructField, StringType, IntegerType,LongType,FloatType,DoubleType, TimestampType}
  import org.apache.spark.sql.cassandra._

  //Spark connector
  import com.datastax.spark.connector._
  import com.datastax.spark.connector.cql.CassandraConnector

  //CosmosDB library for multiple retry
  import com.microsoft.azure.cosmosdb.cassandra

  // Specify connection factory for Cassandra
  spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

  // Parallelism and throughput configs
  spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
  spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
  spark.conf.set("spark.cassandra.output.concurrent.writes", "100")
  spark.conf.set("spark.cassandra.concurrent.reads", "512")
  spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
  spark.conf.set("spark.cassandra.connection.keep_alive_ms", "60000000") //Increase this number as needed
  ```

* Выполните операции CRUD.

  ```scala
  //1) Create table if it does not exist
  val cdbConnector = CassandraConnector(sc)
  cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks.books(book_id TEXT PRIMARY KEY,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT) WITH cosmosdb_provisioned_throughput=4000;"))

  //2) Delete data from potential prior runs
  cdbConnector.withSessionDo(session => session.execute("DELETE FROM books_ks.books WHERE book_id IN ('b00300','b00001','b00023','b00501','b09999','b01001','b00999','b03999','b02999','b000009');"))

  //3) Generate a few rows
  val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887,11.33),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890,22.45),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892,19.83),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893,14.22),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901,12.25)
  ).toDF("book_id", "book_author", "book_name", "book_pub_year","book_price")

  //4) Persist
  booksDF.write.mode("append").format("org.apache.spark.sql.cassandra").options(Map( "table" -> "books", "keyspace" -> "books_ks", "output.consistency.level" -> "ALL", "ttl" -> "10000000")).save()

  //5) Read the data in the table
  spark.read.format("org.apache.spark.sql.cassandra").options(Map( "table" -> "books", "keyspace" -> "books_ks")).load.show
  ```

## <a name="access-azure-cosmos-db-cassandra-api-from-jupyter-notebooks"></a>Доступ к API Cassandra для Azure Cosmos DB из элементов Jupyter Notebook

HDInsight Spark включает в себя службы Zeppelin и Jupyter Notebook. Это веб-среды Notebook, которые поддерживают Scala и Python. Элементы Notebook отлично подходят для интерактивной исследовательской аналитики и совместной работы, но не предназначены для оперативных или производственных процессов.

Приведенные ниже элементы Jupyter Notebook можно передать в кластер HDInsight Spark и обеспечить готовые примеры работы с API Cassandra для Azure Cosmos DB. Обязательно просмотрите первый элемент Notebook `1.0-ReadMe.ipynb`, чтобы проверить конфигурацию службы Spark для подключения к API Cassandra для Azure Cosmos DB.

Скачайте эти элементы Notebook из раздела [azure-cosmos-db-cassandra-api-spark-notebooks-jupyter](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-notebooks-jupyter/blob/main/scala/) на компьютер.
  
### <a name="how-to-upload"></a>Передача
При запуске Jupyter перейдите к Scala. Сначала создайте каталог, а затем передайте в него элементы Notebook. Кнопка "Upload" (Передать) находится вверху справа.  

### <a name="how-to-run"></a>Запуск
Последовательно запустите элементы Notebook и их ячейки.  В верхней части каждого элемента Notebook нажмите кнопку "Run" (Запустить), чтобы выполнить все ячейки, или нажмите клавиши SHIFT+ВВОД для каждой ячейки.

## <a name="access-with-azure-cosmos-db-cassandra-api-from-your-spark-scala-program"></a>Доступ к API Cassandra для Azure Cosmos DB из программы Spark Scala

Для автоматизации процессов в рабочей среде программы Spark отправляются в кластер с помощью [spark-submit](https://spark.apache.org/docs/latest/submitting-applications.html).

## <a name="next-steps"></a>Дальнейшие действия

* [Руководство. Создание приложения Scala Maven для Spark в HDInsight с помощью IntelliJ](../hdinsight/spark/apache-spark-create-standalone-application.md)

* [Подключение к API Cassandra для Azure Cosmos DB из программ Spark Scala](https://github.com/Azure-Samples/azure-cosmos-db-cassandra-api-spark-connector-sample/blob/main/src/main/scala/com/microsoft/azure/cosmosdb/cassandra/SampleCosmosDBApp.scala)

* [Полный список примеров кода для работы с API Cassandra](cassandra-spark-generic.md)
