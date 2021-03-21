---
title: Подключение Apache Spark к Azure Cosmos DB
description: Сведения о соединителе Spark для Azure Cosmos DB, который позволяет подключать Apache Spark к Azure Cosmos DB.
author: tknandu
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/21/2019
ms.author: ramkris
ms.openlocfilehash: 06498a27b95a72148497efd2d1e600d802414359
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359563"
---
# <a name="accelerate-big-data-analytics-by-using-the-apache-spark-to-azure-cosmos-db-connector"></a>Ускорение аналитики больших данных с помощью соединителя Apache Spark для Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Задания [Spark](https://spark.apache.org/) можно выполнять с данными, хранящимися в Azure Cosmos DB с помощью соединителя Cosmos DB Spark. Cosmos можно использовать для обработки пакетной и потоковой передачи, а также как уровень обслуживания для доступа с низкой задержкой.

Вы можете использовать соединитель с [Azure Databricks](https://azure.microsoft.com/services/databricks) или [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/), который предоставляет управляемые кластеры Spark в Azure. В следующей таблице показаны поддерживаемые версии Spark.

| Компонент | Версия |
|---------|-------|
| Apache Spark | 2.4. x, 2.3. x, 2.2. x и 2.1. x |
| Scala | 2,11 |
| Версия среды выполнения Azure Databricks | > 3.4 |

> [!WARNING]
> Этот соединитель поддерживает API ядра (SQL) Azure Cosmos DB.
> Для Cosmos DB для API-интерфейса MongoDB используйте [соединитель MongoDB Spark](https://docs.mongodb.com/spark-connector/master/).
> Для Cosmos DB API Cassandra используйте [соединитель Spark Cassandra](https://github.com/datastax/spark-cassandra-connector).

> [!IMPORTANT]
> Соединитель Azure Cosmos DB Spark в настоящее время не поддерживается для учетных записей без [сервера](serverless.md) . Это будет решено, так как бессерверное предложение станет общедоступным.

## <a name="quickstart"></a>Краткое руководство

* Выполните действия, описанные в статье Начало [работы с пакетом SDK для Java](./create-sql-api-java.md) , чтобы настроить учетную запись Cosmos DB и заполнить некоторые данные.
* Выполните действия, описанные в разделе [Azure Databricks Приступая к работе](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal) , чтобы настроить Azure Databricks рабочую область и кластер.
* Теперь можно создавать новые записные книжки и импортировать библиотеку соединителей Cosmos DB. Для получения дополнительных сведений о настройке рабочей области перейдите к разделу [Работа с соединителем Cosmos DB](#bk_working_with_connector) .
* В следующем разделе приведены фрагменты кода, посвященные чтению и записи с помощью соединителя.

### <a name="batch-reads-from-cosmos-db"></a>Пакетная операция чтения из Cosmos DB

В следующем фрагменте кода показано, как создать кадр данных Spark для чтения из Cosmos DB в PySpark.

```python
# Read Configuration
readConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_pcoll",
    "query_custom": "SELECT c.date, c.delay, c.distance, c.origin, c.destination FROM c WHERE c.origin = 'SEA'" // Optional
}

# Connect via azure-cosmosdb-spark to create Spark DataFrame
flights = spark.read.format(
    "com.microsoft.azure.cosmosdb.spark").options(**readConfig).load()
flights.count()
```

И тот же фрагмент кода в Scala:

```scala
// Import Necessary Libraries
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark._
import com.microsoft.azure.cosmosdb.spark.config.Config

// Read Configuration
val readConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_pcoll",
  "query_custom" -> "SELECT c.date, c.delay, c.distance, c.origin, c.destination FROM c WHERE c.origin = 'SEA'" // Optional
))

// Connect via azure-cosmosdb-spark to create Spark DataFrame
val flights = spark.read.cosmosDB(readConfig)
flights.count()
```

### <a name="batch-writes-to-cosmos-db"></a>Пакетная запись в Cosmos DB

В следующем фрагменте кода показано, как записать кадр данных в Cosmos DB в PySpark.

```python
# Write configuration
writeConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_fromsea",
    "Upsert": "true"
}

# Write to Cosmos DB from the flights DataFrame
flights.write.mode("overwrite").format("com.microsoft.azure.cosmosdb.spark").options(
    **writeConfig).save()
```

И тот же фрагмент кода в Scala:

```scala
// Write configuration

val writeConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_fromsea",
  "Upsert" -> "true"
))

// Write to Cosmos DB from the flights DataFrame
import org.apache.spark.sql.SaveMode
flights.write.mode(SaveMode.Overwrite).cosmosDB(writeConfig)
```

### <a name="streaming-reads-from-cosmos-db"></a>Потоковая передача операций чтения из Cosmos DB

В следующем фрагменте кода показано, как подключиться к веб-каналу изменений Azure Cosmos DB и прочитать его из него.

```python
# Read Configuration
readConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_pcoll",
    "ReadChangeFeed": "true",
    "ChangeFeedQueryName": "Departure-Delays",
    "ChangeFeedStartFromTheBeginning": "false",
    "InferStreamSchema": "true",
    "ChangeFeedCheckpointLocation": "dbfs:/Departure-Delays"
}


# Open a read stream to the Cosmos DB Change Feed via azure-cosmosdb-spark to create Spark DataFrame
changes = (spark
           .readStream
           .format("com.microsoft.azure.cosmosdb.spark.streaming.CosmosDBSourceProvider")
           .options(**readConfig)
           .load())
```
И тот же фрагмент кода в Scala:

```scala
// Import Necessary Libraries
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark._
import com.microsoft.azure.cosmosdb.spark.config.Config

// Read Configuration
val readConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_pcoll",
  "ReadChangeFeed" -> "true",
  "ChangeFeedQueryName" -> "Departure-Delays",
  "ChangeFeedStartFromTheBeginning" -> "false",
  "InferStreamSchema" -> "true",
  "ChangeFeedCheckpointLocation" -> "dbfs:/Departure-Delays"
))

// Open a read stream to the Cosmos DB Change Feed via azure-cosmosdb-spark to create Spark DataFrame
val df = spark.readStream.format(classOf[CosmosDBSourceProvider].getName).options(readConfig).load()
```

### <a name="streaming-writes-to-cosmos-db"></a>Потоковая запись в Cosmos DB

В следующем фрагменте кода показано, как записать кадр данных в Cosmos DB в PySpark.

```python
# Write configuration
writeConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_fromsea",
    "Upsert": "true",
    "WritingBatchSize": "500",
    "CheckpointLocation": "/checkpointlocation_write1"
}

# Write to Cosmos DB from the flights DataFrame
changeFeed = (changes
              .writeStream
              .format("com.microsoft.azure.cosmosdb.spark.streaming.CosmosDBSinkProvider")
              .outputMode("append")
              .options(**writeconfig)
              .start())
```

И тот же фрагмент кода в Scala:

```scala
// Write configuration

val writeConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_fromsea",
  "Upsert" -> "true",
  "WritingBatchSize" -> "500",
  "CheckpointLocation" -> "/checkpointlocation_write1"
))

// Write to Cosmos DB from the flights DataFrame
df
.writeStream
.format(classOf[CosmosDBSinkProvider].getName)
.options(writeConfig)
.start()
```
Дополнительные фрагменты кода и сквозные примеры см. в разделе [Jupyter](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples/notebooks).

## <a name="working-with-the-connector"></a><a name="bk_working_with_connector"></a> Работа с соединителем

Вы можете создать соединитель из источника в GitHub или скачать Uber jar из Maven по ссылкам ниже.

| Spark | Scala | Последняя версия |
|---|---|---|
| 2.4.0 | 2,11 | [Azure-cosmosdb-spark_lkg_version](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG)
| 2.3.0 | 2,11 | [Azure-cosmosdb-spark_2.3.0 _2.11_1.3.3](https://search.maven.org/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.3.0_2.11/1.3.3/jar)
| 2.2.0 | 2,11 | [Azure-cosmosdb-spark_2.2.0 _2.11_1.1.1](https://search.maven.org/#artifactdetails%7Ccom.microsoft.azure%7Cazure-cosmosdb-spark_2.2.0_2.11%7C1.1.1%7Cjar)
| 2.1.0 | 2,11 | [Azure-cosmosdb-spark_2.1.0 _2.11_1.2.2](https://search.maven.org/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.1.0_2.11/1.2.2/jar)

### <a name="using-databricks-notebooks"></a>Использование записных книжек с модулями

Создайте библиотеку с помощью рабочей области "сведения о модулях", следуя указаниям в руководстве по Azure Databricks > [использовании соединителя Azure Cosmos DB Spark](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/cosmosdb-connector.html) .

> [!NOTE]
> **Использование страницы соединителя Azure Cosmos DB Spark** в настоящее время устарело. Вместо того чтобы загружать шесть отдельных JAR в шесть разных библиотек, можно скачать JAR-файл Uber из Maven в [Azure-cosmosdb spark_lkg_version](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG) и установить этот JAR-файл или библиотеку.
> 

### <a name="using-spark-cli"></a>Использование Spark-CLI

Для работы с соединителем с помощью Spark-CLI (т. е `spark-shell` .,, `pyspark` `spark-submit` ) можно использовать `--packages` параметр с [координатами Maven](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.4.0_2.11)соединителя.

```sh
spark-shell --master yarn --packages "com.microsoft.azure:azure-cosmosdb-spark_2.4.0_2.11:1.4.0"

```

### <a name="using-jupyter-notebooks"></a>Использование записных книжек Jupyter

Если вы используете Jupyter Notebook в HDInsight, можно использовать ячейку Spark-Magic `%%configure` для указания координат Maven соединителя.

```python
{ "name":"Spark-to-Cosmos_DB_Connector",
  "conf": {
    "spark.jars.packages": "com.microsoft.azure:azure-cosmosdb-spark_2.4.0_2.11:1.4.0",
    "spark.jars.excludes": "org.scala-lang:scala-reflect"
   }
   ...
}
```

> Обратите внимание, что включение `spark.jars.excludes` относится только к удалению потенциальных конфликтов между соединителем, Apache Spark и Livy.

### <a name="build-the-connector"></a>Создание соединителя

Сейчас этот проект соединителя использует `maven` так, чтобы выполнять сборку без зависимостей, можно запустить:

```sh
mvn clean package
```

## <a name="working-with-our-samples"></a>Работа с нашими примерами

В [репозитории Cosmos DB Spark GitHub](https://github.com/Azure/azure-cosmosdb-spark) есть следующие примеры записных книжек и сценариев, которые можно использовать.

* **Производительность в реальном времени с помощью Spark и Cosmos DB (Сиэтл)** [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/On-Time%20Flight%20Performance%20with%20Spark%20and%20Cosmos%20DB%20-%20Seattle.ipynb)  |  [HTML](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/On-Time%20Flight%20Performance%20with%20Spark%20and%20Cosmos%20DB%20-%20Seattle.html): подключение Spark к Cosmos DB с помощью службы HDInsight Jupyter Notebook для демонстрации Spark SQL, граффрамес и прогнозирования задержек рейсов с помощью конвейеров машинного обучения.
* **Источник Twitter с Apache Spark и Azure Cosmos DB веб-канал изменений**: [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Twitter%20with%20Spark%20and%20Azure%20Cosmos%20DB%20Change%20Feed.ipynb)  |  [HTML](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Twitter%20with%20Spark%20and%20Azure%20Cosmos%20DB%20Change%20Feed.html)
* **Использование Apache Spark для запроса Cosmos DB графов**: [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Using%20Apache%20Spark%20to%20query%20Cosmos%20DB%20Graphs.ipynb)  |  [HTML](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Using%20Apache%20Spark%20to%20query%20Cosmos%20DB%20Graphs.html)
* **[Подключение Azure Databricks к Azure Cosmos DB](https://docs.databricks.com/spark/latest/data-sources/azure/cosmosdb-connector.html)** с помощью `azure-cosmosdb-spark` .  Ссылка здесь также является Azure Databricksной версией [записной книжки для повышения производительности](https://github.com/dennyglee/databricks/tree/master/notebooks/Users/denny%40databricks.com/azure-databricks).
* **[Лямбда-архитектура с Azure Cosmos DB и HDInsight (Apache Spark)](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/readme.md)**. Вы можете сократить эксплуатационные расходы на поддержку конвейеров больших данных с помощью Cosmos DB и Spark.

## <a name="more-information"></a>Дополнительные сведения

На вики-сайте есть дополнительные сведения, в `azure-cosmosdb-spark` [](https://github.com/Azure/azure-cosmosdb-spark/wiki) том числе:

* [Azure Cosmos DB Guide для пользователя соединителя Spark](https://github.com/Azure/azure-documentdb-spark/wiki/Azure-Cosmos-DB-Spark-Connector-User-Guide)
* [Примеры агрегатов](https://github.com/Azure/azure-documentdb-spark/wiki/Aggregations-Examples)

### <a name="configuration-and-setup"></a>Настройка и установка

* [Конфигурация соединителя Spark](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuration-references)
* [Установка соединителя Spark в Cosmos DB](https://github.com/Azure/azure-documentdb-spark/wiki/Spark-to-Cosmos-DB-Connector-Setup) (выполняется)
* [Настройка Power BI прямого запроса к Azure Cosmos DB через Apache Spark (HDI)](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuring-Power-BI-Direct-Query-to-Azure-Cosmos-DB-via-Apache-Spark-(HDI))

### <a name="troubleshooting"></a>Устранение неполадок

* [Использование агрегатов Cosmos DB](https://github.com/Azure/azure-documentdb-spark/wiki/Troubleshooting:-Using-Cosmos-DB-Aggregates)
* [Известные проблемы](https://github.com/Azure/azure-cosmosdb-spark/wiki/Known-Issues)

### <a name="performance"></a>Производительность

* [Советы по производительности .NET](https://github.com/Azure/azure-cosmosdb-spark/wiki/Performance-tips)
* [Тестовые запуски запросов](https://github.com/Azure/azure-documentdb-spark/wiki/Query-Test-Runs)
* [Написание тестовых запусков](https://github.com/Azure/azure-cosmosdb-spark/wiki/Writing-Test-Runs)

### <a name="change-feed"></a>Канал изменений

* [Потоковая обработка изменений с помощью Azure Cosmos DB веб-канала изменений и Apache Spark](https://github.com/Azure/azure-cosmosdb-spark/wiki/Stream-Processing-Changes-using-Azure-Cosmos-DB-Change-Feed-and-Apache-Spark)
* [Демонстрации веб-канала изменений](https://github.com/Azure/azure-cosmosdb-spark/wiki/Change-Feed-demos)
* [Демонстрации структурированного потока](https://github.com/Azure/azure-cosmosdb-spark/wiki/Structured-Stream-demos)

### <a name="monitoring"></a>Мониторинг

* [Мониторинг заданий Spark с помощью Application Insights](https://github.com/Azure/azure-cosmosdb-spark/tree/2.3/samples/monitoring)

## <a name="next-steps"></a>Дальнейшие действия

Если у вас еще нет соединителя Spark для Azure Cosmos DB, скачайте его из репозитория GitHub [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark). Изучите следующие дополнительные ресурсы в репозитории:

* [Примеры агрегирований](https://github.com/Azure/azure-cosmosdb-spark/wiki/Aggregations-Examples)
* [Примеры сценариев и записных книжек](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples)

Дополнительные сведения см. в [руководстве по Apache Spark SQL, таблицам и наборам данных](https://spark.apache.org/docs/latest/sql-programming-guide.html), а также статье [Краткое руководство по созданию кластера Spark в HDInsight с помощью шаблона](../hdinsight/spark/apache-spark-jupyter-spark-sql.md).