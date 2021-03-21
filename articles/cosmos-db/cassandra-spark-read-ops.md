---
title: Чтения табличных данных API Cassandra с помощью Spark
titleSufix: Azure Cosmos DB
description: В этой статье рассказывается, как читать данные из таблиц API Cassandra в Azure Cosmos DB.
author: TheovanKraay
ms.author: thvankra
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 06/02/2020
ms.custom: seodec18
ms.openlocfilehash: ceede96cbf3be12a6129e27d34e318e4c4163458
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93073502"
---
# <a name="read-data-from-azure-cosmos-db-cassandra-api-tables-using-spark"></a>Чтения данных из таблиц API Cassandra в Azure Cosmos DB с помощью Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

 В этой статье рассказывается, как читать данные, сохраненные в API Cassandra для Azure Cosmos DB, из Spark.

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

### <a name="read-table-using-sessionreadformat-command"></a>Чтение таблицы с помощью команды session.read.format

```scala
val readBooksDF = sqlContext
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .load

readBooksDF.explain
readBooksDF.show
```
### <a name="read-table-using-sparkreadcassandraformat"></a>Чтение таблицы, с помощью spark.read.cassandraFormat 

```scala
val readBooksDF = spark.read.cassandraFormat("books", "books_ks", "").load()
```

### <a name="read-specific-columns-in-table"></a>Чтение определенных столбцов в таблице

```scala
val readBooksDF = spark
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .load
  .select("book_name","book_author", "book_pub_year")

readBooksDF.printSchema
readBooksDF.explain
readBooksDF.show
```

### <a name="apply-filters"></a>Применение фильтров

Вы можете отправить предикаты в базу данных, чтобы обеспечить более оптимизированные запросы Spark. Предикат — это условие в запросе, возвращающее значение true или false, которое обычно находится в предложении WHERE. Предикат принудительно раскрывает данные в запросе к базе данных, уменьшая количество записей, полученных из базы данных, и повышая производительность запросов. По умолчанию API набора данных Spark автоматически передает допустимые предложения WHERE в базу данных. 

```scala
val df = spark.read.cassandraFormat("books", "books_ks").load
df.explain
val dfWithPushdown = df.filter(df("book_pub_year") > 1891)
dfWithPushdown.explain

readBooksDF.printSchema
readBooksDF.explain
readBooksDF.show
```

Раздел Пушедфилтерс физического плана включает фильтр push-уведомлений GreaterThan. 

:::image type="content" source="./media/cassandra-spark-read-ops/pushdown-predicates.png" alt-text="Partition":::

## <a name="rdd-api"></a>API RRD

### <a name="read-table"></a>Чтение таблицы
```scala
val bookRDD = sc.cassandraTable("books_ks", "books")
bookRDD.take(5).foreach(println)
```

### <a name="read-specific-columns-in-table"></a>Чтение определенных столбцов в таблице

```scala
val booksRDD = sc.cassandraTable("books_ks", "books").select("book_id","book_name").cache
booksRDD.take(5).foreach(println)
```

## <a name="sql-views"></a>Представления SQL 

### <a name="create-a-temporary-view-from-a-dataframe"></a>Создание временного представления из кадра данных

```scala
spark
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .load.createOrReplaceTempView("books_vw")
```

### <a name="run-queries-against-the-view"></a>Выполнение запросов к представлению

```sql
select * from books_vw where book_pub_year > 1891
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные статьи по работе с API Cassandra для Azure Cosmos DB из Spark:
 
 * [Операции upsert](cassandra-spark-upsert-ops.md)
 * [Операции удаления](cassandra-spark-delete-ops.md)
 * [Операции агрегатных вычислений](cassandra-spark-aggregation-ops.md)
 * [Операции копирования таблиц](cassandra-spark-table-copy-ops.md)

