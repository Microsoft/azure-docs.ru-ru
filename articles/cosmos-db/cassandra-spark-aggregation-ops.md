---
title: Статистические операции для таблиц API Cassandra для Azure Cosmos DB из Spark
description: В этой статье рассматриваются основные статистические операции над таблицами API Cassandra для Azure Cosmos DB из Spark
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: 5939690d3f2c0bb9affa3e2fb425b909b96cf002
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93087627"
---
# <a name="aggregate-operations-on-azure-cosmos-db-cassandra-api-tables-from-spark"></a>Статистические операции для таблиц API Cassandra для Azure Cosmos DB из Spark 
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

В этой статье описаны основные статистические операции над таблицами API Cassandra для Azure Cosmos DB из Spark. 

> [!NOTE]
> Фильтрация на стороне сервера и статистическая обработка на стороне сервера сейчас не поддерживаются в API Cassandra для Azure Cosmos DB.

## <a name="cassandra-api-configuration"></a>Конфигурация API Cassandra

```scala
import org.apache.spark.sql.cassandra._
//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//CosmosDB library for multiple retry
import com.microsoft.azure.cosmosdb.cassandra

//Connection-related
spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")
spark.conf.set("spark.cassandra.connection.port","10350")
spark.conf.set("spark.cassandra.connection.ssl.enabled","true")
spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")
spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")
spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")
//Throughput-related...adjust as needed
spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")
spark.conf.set("spark.cassandra.concurrent.reads", "512")
spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")
```
## <a name="sample-data-generator"></a>Пример генератора данных

```scala
// Generate a simple dataset containing five values
val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887,11.33),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890,22.45),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892,19.83),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893,14.22),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901,12.25)
).toDF("book_id", "book_author", "book_name", "book_pub_year","book_price")

booksDF.write
  .mode("append")
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks", "output.consistency.level" -> "ALL", "ttl" -> "10000000"))
  .save()
```

## <a name="count-operation"></a>Операция подсчета


### <a name="rdd-api"></a>API RRD

```scala
sc.cassandraTable("books_ks", "books").count
```

**Выходные данные:**
```bash
res48: Long = 5
```

### <a name="dataframe-api"></a>API Dataframe

Подсчет кадров данных сейчас не поддерживается.  В следующем примере показано, как выполнить подсчет кадров данных после сохранения кадра данных в память в качестве обходного решения.

Выберите [вариант хранения]( https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#which-storage-level-to-choose) из доступных вариантов, чтобы не столкнуться с проблемами нехватки памяти:

* MEMORY_ONLY: это вариант хранилища по умолчанию. Хранит RDD как десериализованные объекты Java на виртуальной машине Java. Если RDD не помещается в памяти, некоторые секции не будут кэшироваться, и они будут вычисляться повторно в режиме реального времени каждый раз, когда требуется.

* MEMORY_AND_DISK: хранит RDD как десериализованные объекты Java на виртуальной машине Java. Если RDD не помещается в памяти, секции, которые не помещаются в памяти, будут сохранены на диске, и когда они потребуются, они будут считаны из расположения, в котором хранятся.

* MEMORY_ONLY_SER (Java/Scala): хранит RDD как сериализованные объекты Java — в однобайтовом массиве для каждой секции. Этот вариант позволяет сэкономить пространство по сравнению с десериализованными объектами, особенно при использовании быстрого сериализатора, однако для чтения таких объектов более интенсивно используется ЦП.

* MEMORY_AND_DISK_SER (Java/Scala): этот вариант похож на MEMORY_ONLY_SER, единственное различие состоит в том, что секции, которые не помещаются на диске, сбрасываются, вместо того чтобы вычислять их повторно, когда они необходимы.

* DISK_ONLY: хранит секции RDD только на диске.

* MEMORY_ONLY_2, MEMORY_AND_DISK_2…: то же, что и для уровней, указанных выше, однако каждая секция реплицируется на два узла кластера.

* OFF_HEAP (экспериментальная функция): аналогично MEMORY_ONLY_SER, однако данные сохраняются в памяти вне кучи, и память вне кучи требуется предварительно включить. 

```scala
//Workaround
import org.apache.spark.storage.StorageLevel

//Read from source
val readBooksDF = spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()

//Explain plan
readBooksDF.explain

//Materialize the dataframe
readBooksDF.persist(StorageLevel.MEMORY_ONLY)

//Subsequent execution against this DF hits the cache 
readBooksDF.count

//Persist as temporary view
readBooksDF.createOrReplaceTempView("books_vw")
```

### <a name="sql"></a>SQL

```sql
select * from books_vw;
select count(*) from books_vw where book_pub_year > 1900;
select count(book_id) from books_vw;
select book_author, count(*) as count from books_vw group by book_author;
select count(*) from books_vw;
```

## <a name="average-operation"></a>Операция усреднения

### <a name="rdd-api"></a>API RRD

```scala
sc.cassandraTable("books_ks", "books").select("book_price").as((c: Double) => c).mean
```

**Выходные данные:**
```
res24: Double = 16.016000175476073
```

### <a name="dataframe-api"></a>API Dataframe

```scala
spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()
  .select("book_price")
  .agg(avg("book_price"))
  .show
```

**Выходные данные:**
```
+------------------+
|   avg(book_price)|
+------------------+
|16.016000175476073|
+------------------+
```

### <a name="sql"></a>SQL

```sql
select avg(book_price) from books_vw;
```
**Выходные данные:**
```
16.016000175476073
```

## <a name="min-operation"></a>Операция определения минимума

### <a name="rdd-api"></a>API RRD

```scala
sc.cassandraTable("books_ks", "books").select("book_price").as((c: Float) => c).min
```

**Выходные данные:**
```
res31: Float = 11.33
```

### <a name="dataframe-api"></a>API Dataframe

```scala
spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()
  .select("book_id","book_price")
  .agg(min("book_price"))
  .show
```

**Выходные данные:**
```
+---------------+
|min(book_price)|
+---------------+
|          11.33|
+---------------+
```

### <a name="sql"></a>SQL

```sql
select min(book_price) from books_vw;
```

**Выходные данные:**
```
11.33
```

## <a name="max-operation"></a>Операция определения максимума

### <a name="rdd-api"></a>API RRD

```scala
sc.cassandraTable("books_ks", "books").select("book_price").as((c: Float) => c).max
```

### <a name="dataframe-api"></a>API Dataframe

```scala 
spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()
  .select("book_price")
  .agg(max("book_price"))
  .show
```

**Выходные данные:**
```
+---------------+
|max(book_price)|
+---------------+
|          22.45|
+---------------+
```

### <a name="sql"></a>SQL

```sql
select max(book_price) from books_vw;
```
**Выходные данные:**
```
22.45
```

## <a name="sum-operation"></a>Операция суммирования

### <a name="rdd-api"></a>API RRD

```scala
sc.cassandraTable("books_ks", "books").select("book_price").as((c: Float) => c).sum
```

**Выходные данные:**
```
res46: Double = 80.08000087738037
```

### <a name="dataframe-api"></a>API Dataframe

```scala
spark
  .read
  .cassandraFormat("books", "books_ks", "")
  .load()
  .select("book_price")
  .agg(sum("book_price"))
  .show
```
**Выходные данные:**
```
+-----------------+
|  sum(book_price)|
+-----------------+
|80.08000087738037|
+-----------------+
```

### <a name="sql"></a>SQL

```sql
select sum(book_price) from books_vw;
```

**Выходные данные:**
```
80.08000087738037
```

## <a name="top-or-comparable-operation"></a>Операция определения максимального значения или сравнения

### <a name="rdd-api"></a>API RRD

```scala
val readCalcTopRDD = sc.cassandraTable("books_ks", "books").select("book_name","book_price").sortBy(_.getFloat(1), false)
readCalcTopRDD.zipWithIndex.filter(_._2 < 3).collect.foreach(println)
//delivers the first top n items without collecting the rdd to the driver.
```
**Выходные данные:**
```
(CassandraRow{book_name: A sign of four, book_price: 22.45},0)
(CassandraRow{book_name: The adventures of Sherlock Holmes, book_price: 19.83},1)
(CassandraRow{book_name: The memoirs of Sherlock Holmes, book_price: 14.22},2)
readCalcTopRDD: org.apache.spark.rdd.RDD[com.datastax.spark.connector.CassandraRow] = MapPartitionsRDD[430] at sortBy at command-2371828989676374:1
```
### <a name="dataframe-api"></a>API Dataframe

```scala
import org.apache.spark.sql.functions._

val readBooksDF = spark.read.format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .load
  .select("book_name","book_price")
  .orderBy(desc("book_price"))
  .limit(3)

//Explain plan
readBooksDF.explain

//Top
readBooksDF.show
```

**Выходные данные:**
```
== Physical Plan ==
TakeOrderedAndProject(limit=3, orderBy=[book_price#1840 DESC NULLS LAST], output=[book_name#1839,book_price#1840])
+- *(1) Scan org.apache.spark.sql.cassandra.CassandraSourceRelation@29cd5f58 [book_name#1839,book_price#1840] PushedFilters: [], ReadSchema: struct<book_name:string,book_price:float>
+--------------------+----------+
|           book_name|book_price|
+--------------------+----------+
|      A sign of four|     22.45|
|The adventures of...|     19.83|
|The memoirs of Sh...|     14.22|
+--------------------+----------+

import org.apache.spark.sql.functions._
readBooksDF: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [book_name: string, book_price: float]
```

### <a name="sql"></a>SQL

```sql
select book_name,book_price from books_vw order by book_price desc limit 3;
```

## <a name="next-steps"></a>Дальнейшие действия

Для выполнения операций копирования таблицы обратитесь к разделу:

* [Операции копирования таблиц](cassandra-spark-table-copy-ops.md)
