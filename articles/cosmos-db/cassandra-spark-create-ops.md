---
title: Создание или вставка данных в Azure Cosmos DB API Cassandra из Spark
description: В этой статье рассказывается, как вставить образец данных в таблицы API Cassandra для Azure Cosmos DB.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: 3ec44d20b763a98683d9b947c94ad6be75180113
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93092259"
---
# <a name="createinsert-data-into-azure-cosmos-db-cassandra-api-from-spark"></a>Создание или добавление данных в API Cassandra для Azure Cosmos DB из Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]
 
В этой статье рассказывается, как вставлить образец данных в таблицу API Cassandra для Azure Cosmos DB из Spark.

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
## <a name="dataframe-api"></a>API Dataframe

### <a name="create-a-dataframe-with-sample-data"></a>Создание кадра данных с образцом данных

```scala
// Generate a dataframe containing five records
val booksDF = Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901)
).toDF("book_id", "book_author", "book_name", "book_pub_year")

//Review schema
booksDF.printSchema

//Print
booksDF.show
```

> [!NOTE]
> Функциональная возможность "Создать, если не существует" на уровне строк пока не поддерживается.

### <a name="persist-to-azure-cosmos-db-cassandra-api"></a>Сохранение данных в API Cassandra для Azure Cosmos DB

При сохранении данных, можно также задать время существования и параметры политики согласованности, как показано в следующем примере.

```scala
//Persist
booksDF.write
  .mode("append")
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks", "output.consistency.level" -> "ALL", "ttl" -> "10000000"))
  .save()
```

> [!NOTE]
> Уровень столбца TTL пока не поддерживается.

#### <a name="validate-in-cqlsh"></a>Проверка в cqlsh

```sql
use books_ks;
select * from books;
```

## <a name="resilient-distributed-database-rdd-api"></a>API устойчивой распределенной базы данных (RDD)

### <a name="create-a-rdd-with-sample-data"></a>Создание базы данных с образцами данных
```scala
//Delete records created in the previous section 
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("delete from books_ks.books where book_id in ('b00300','b00001','b00023','b00501','b09999','b01001','b00999','b03999','b02999');"))

//Create RDD
val booksRDD = sc.parallelize(Seq(
   ("b00001", "Arthur Conan Doyle", "A study in scarlet", 1887),
   ("b00023", "Arthur Conan Doyle", "A sign of four", 1890),
   ("b01001", "Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
   ("b00501", "Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
   ("b00300", "Arthur Conan Doyle", "The hounds of Baskerville", 1901)
))

//Review
booksRDD.take(2).foreach(println)
```

> [!NOTE]
> Функциональная возможность "Создать, если не существует" пока не поддерживается.

### <a name="persist-to-azure-cosmos-db-cassandra-api"></a>Сохранение данных в API Cassandra для Azure Cosmos DB

При сохранении данных в Cassandra API, можно также задать время существования и параметры политики согласованности, как показано в следующем примере.

```scala
import com.datastax.spark.connector.writer._

//Persist
booksRDD.saveToCassandra("books_ks", "books", SomeColumns("book_id", "book_author", "book_name", "book_pub_year"),writeConf = WriteConf(ttl = TTLOption.constant(900000),consistencyLevel = ConsistencyLevel.ALL))
```

#### <a name="validate-in-cqlsh"></a>Проверка в cqlsh

```sql
use books_ks;
select * from books;
```

## <a name="next-steps"></a>Дальнейшие действия

После того как данные вставлены в таблицу API Cassandra для Azure Cosmos DB, ознакомьтесь со следующими статьями, чтобы узнать, как выполнять другие операции над хранимыми данными в API Cassandra для Cosmos DB.
 
* [Операции чтения](cassandra-spark-read-ops.md)
* [Операции upsert](cassandra-spark-upsert-ops.md)
* [Операции удаления](cassandra-spark-delete-ops.md)
* [Операции агрегатных вычислений](cassandra-spark-aggregation-ops.md)
* [Операции копирования таблиц](cassandra-spark-table-copy-ops.md)

