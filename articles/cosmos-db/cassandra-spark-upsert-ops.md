---
title: Операции upsert с данными в API Cassandra для Azure Cosmos DB из Spark
description: В этой статье рассказывается, как выполнять операции upsert в таблицы в API Cassandra для Azure Cosmos DB из Spark.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 09/24/2018
ms.openlocfilehash: e65d526db2861bc0d30a40651bbf9cf46111eaf5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93073202"
---
# <a name="upsert-data-into-azure-cosmos-db-cassandra-api-from-spark"></a>Операции upsert с данными в API Cassandra для Azure Cosmos DB из Spark
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

В этой статье описывается выполнение операций upsert с данными в API Cassandra для Azure Cosmos DB из Spark.

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

### <a name="create-a-dataframe"></a>Создание кадра данных 

```scala
// (1) Update: Changing author name to include prefix of "Sir"
// (2) Insert: adding a new book

val booksUpsertDF = Seq(
    ("b00001", "Sir Arthur Conan Doyle", "A study in scarlet", 1887),
    ("b00023", "Sir Arthur Conan Doyle", "A sign of four", 1890),
    ("b01001", "Sir Arthur Conan Doyle", "The adventures of Sherlock Holmes", 1892),
    ("b00501", "Sir Arthur Conan Doyle", "The memoirs of Sherlock Holmes", 1893),
    ("b00300", "Sir Arthur Conan Doyle", "The hounds of Baskerville", 1901),
    ("b09999", "Sir Arthur Conan Doyle", "The return of Sherlock Holmes", 1905)
    ).toDF("book_id", "book_author", "book_name", "book_pub_year")
booksUpsertDF.show()
```

### <a name="upsert-data"></a>Операция upsert с данными

```scala
// Upsert is no different from create
booksUpsertDF.write
  .mode("append")
  .format("org.apache.spark.sql.cassandra")
  .options(Map( "table" -> "books", "keyspace" -> "books_ks"))
  .save()
```

### <a name="update-data"></a>Обновление данных

```scala
//This runs on the driver, leverage only for one off updates
cdbConnector.withSessionDo(session => session.execute("update books_ks.books set book_price=99.33 where book_id ='b00300';"))
```

## <a name="rdd-api"></a>API RRD
> [!NOTE]
> Операция upsert из API RRD аналогична операции создания 

## <a name="next-steps"></a>Дальнейшие действия

Для выполнения других операций с данными, хранящимися в таблицах API Cassandra Azure Cosmos DB, ознакомьтесь со следующими статьями.
 
* [Операции удаления](cassandra-spark-delete-ops.md)
* [Операции агрегирования](cassandra-spark-aggregation-ops.md)
* [Операции копирования таблиц](cassandra-spark-table-copy-ops.md)
