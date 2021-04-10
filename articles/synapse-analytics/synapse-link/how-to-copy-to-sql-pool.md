---
title: Копирование Synapse Link для данных Azure Cosmos DB в выделенный пул SQL с помощью Apache Spark
description: Загрузка данных в кадр данных Spark, их курирование и загрузка в таблицу выделенного пула SQL
services: synapse-analytics
author: Rodrigossz
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 08/10/2020
ms.author: rosouz
ms.reviewer: jrasnick
ms.custom: cosmos-db
ms.openlocfilehash: ff04d43e72d4eca9800b330d5a4721ba951c85f9
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105627648"
---
# <a name="copy-data-from-azure-cosmos-db-into-a-dedicated-sql-pool-using-apache-spark"></a>Копирование данных из Azure Cosmos DB в выделенный пул SQL с помощью Apache Spark

С помощью Azure Synapse Link для Azure Cosmos DB пользователи могут запускать аналитику почти в реальном времени по операционным данным в Azure Cosmos DB. Однако в некоторых ситуациях определенные данные необходимо агрегировать и обогащать для обслуживания пользователей хранилища данных. Для курирования и экспорта данных Synapse Link достаточно всего нескольких ячеек в записной книжке.

## <a name="prerequisites"></a>Предварительные требования
* [Подготовьте рабочую область Synapse](../quickstart-create-workspace.md), используя:
    * [бессерверный пул Apache Spark;](../quickstart-create-apache-spark-pool-studio.md)
    * [выделенный пул SQL.](../quickstart-create-sql-pool-studio.md)
* [Подготовьте учетную запись Cosmos DB, используя контейнер HTAP с данными](../../cosmos-db/configure-synapse-link.md)
* [Подключите контейнер HTAP Azure Cosmos DB к рабочей области](./how-to-connect-synapse-link-cosmos-db.md)
* [Настройте надлежащим образом импорт данных в выделенный пул SQL из Spark.](../spark/synapse-spark-sql-pool-import-export.md)

## <a name="steps"></a>Шаги
В этом руководстве показано, как подключиться к аналитическому хранилищу. Описанные здесь действия не влияют на хранилище транзакций (для их выполнения единицы запросов не потребляются). Мы выполним следующие действия:
1. Считывание контейнера HTAP Cosmos DB в кадр данных Spark
2. Агрегирование результатов в новый кадр данных
3. Прием данных в выделенный пул SQL

[![Передача из Spark в SQL 1](../media/synapse-link-spark-to-sql/synapse-spark-to-sql.png)](../media/synapse-link-spark-to-sql/synapse-spark-to-sql.png#lightbox)

## <a name="data"></a>Данные
В этом примере мы используем контейнер HTAP с именем **RetailSales**. Это часть связанной службы с именем **ConnectedData** со следующей схемой:
* _rid: string (nullable = true)
* _ts: long (nullable = true)
* logQuantity: double (nullable = true)
* productCode: string (nullable = true)
* quantity: long (nullable = true)
* price: long (nullable = true)
* id: string (nullable = true)
* advertising: long (nullable = true)
* storeId: long (nullable = true)
* weekStarting: long (nullable = true)
* _etag: string (nullable = true)

Для отчетности мы будем агрегировать продажи (*quantity*, *revenue* (price x quantity) по *productCode* и *weekStarting*. Наконец, мы экспортируем эти данные в таблицу выделенного пула SQL с именем **dbo.productsales**.

## <a name="configure-a-spark-notebook"></a>Настройка записной книжки Spark
Создайте записную книжку Spark, используя в качестве основного языка с Scala на Spark (Scala). Мы используем для сеанса заданный по умолчанию параметр записной книжки.

## <a name="read-the-data-in-spark"></a>Чтение данных в Spark
С помощью Spark считайте контейнер HTAP Cosmos DB в кадр данных в первой ячейке.

```java
val df_olap = spark.read.format("cosmos.olap").
    option("spark.synapse.linkedService", "ConnectedData").
    option("spark.cosmos.container", "RetailSales").
    load()
```

## <a name="aggregate-the-results-in-a-new-dataframe"></a>Агрегирование результатов в новый кадр данных

Во второй ячейке будут выполнятся преобразование и статистические вычисления, которые необходимо выполнить для нового кадра данных до его загрузки в базу данных выделенного пула SQL.

```java
// Select relevant columns and create revenue
val df_olap_step1 = df_olap.select("productCode","weekStarting","quantity","price").withColumn("revenue",col("quantity")*col("price"))
//Aggregate revenue, quantity sold and avg. price by week and product ID
val df_olap_aggr = df_olap_step1.groupBy("productCode","weekStarting").agg(sum("quantity") as "Sum_quantity",sum("revenue") as "Sum_revenue").
    withColumn("AvgPrice",col("Sum_revenue")/col("Sum_quantity"))
```

## <a name="load-the-results-into-a-dedicated-sql-pool"></a>Загрузка результатов в выделенный пул SQL

В третьей ячейке данные загружаются в выделенный пул SQL. При этом автоматически создается временная внешняя таблица, внешний источник данных и формат внешнего файла, которые будут удалены после завершения задания.

```java
df_olap_aggr.write.sqlanalytics("userpool.dbo.productsales", Constants.INTERNAL)
```

## <a name="query-the-results-with-sql"></a>Запрос результатов с помощью SQL

Вы можете запросить результат, используя простой SQL-запрос, например так, как в следующем скрипте SQL:
```sql
SELECT  [productCode]
,[weekStarting]
,[Sum_quantity]
,[Sum_revenue]
,[AvgPrice]
 FROM [dbo].[productsales]
```

В ответ на запрос будут представлены следующие результаты в режиме диаграммы: [![Передача из Spark в SQL 2](../media/synapse-link-spark-to-sql/sql-script-spark-sql.png)](../media/synapse-link-spark-to-sql/sql-script-spark-sql.png#lightbox)

## <a name="next-steps"></a>Дальнейшие шаги
* [Отправка запросов в аналитическое хранилище Azure Cosmos DB с помощью Apache Spark](./how-to-query-analytical-store-spark.md)