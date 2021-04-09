---
title: Общие сведения об использовании Linux Foundation Delta Lake в Apache Spark для Azure Synapse Analytics
description: Узнайте, как использовать Delta Lake в Apache Spark для Azure Synapse Analytics при создании и использовании таблиц со свойствами ACID.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.reviewer: euang
ms.topic: overview
ms.subservice: spark
ms.date: 07/28/2020
ms.author: euang
ms.custom: devx-track-csharp
zone_pivot_groups: programming-languages-spark-all-minus-sql
ms.openlocfilehash: 1e0dfd597e7f445eeba6cef332d8ea12b27dc3a2
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101676309"
---
# <a name="linux-foundation-delta-lake-overview"></a>Общие сведения об Linux Foundation Delta Lake

Эта статья для большей ясности была адаптирована по сравнению с [оригиналом](https://docs.delta.io/latest/quick-start.html). Эта статья поможет вам быстро ознакомиться с основными возможностями [Delta Lake](https://delta.io). В этой статье приводятся фрагменты кода, демонстрирующие чтение и запись в таблицах Delta Lake с использованием интерактивных, пакетных и потоковых запросов. Фрагменты кода также доступны в наборе записных книжек [PySpark здесь](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/PySpark/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20Python.ipynb), [Scala здесь](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/Scala/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20Scala.ipynb)и [C# здесь](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/Spark.NET%20C%23/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20CSharp.ipynb).

Здесь будут рассматриваться:

* Создание таблицы
* Чтение данных
* Обновление данных таблицы
* Перезапись данных таблицы
* Условное обновление без перезаписи
* Чтение более старых версий данных с помощью перехода по времени
* Запись потока данных в таблицу
* Чтение потока изменений из таблицы
* Поддержка SQL

## <a name="configuration"></a>Конфигурация

Обязательно измените приведенные ниже сведения в соответствии со своей средой.

:::zone pivot = "programming-language-python"

```python
import random

session_id = random.randint(0,1000000)
delta_table_path = "/delta/delta-table-{0}".format(session_id)

delta_table_path
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var sessionId = (new Random()).Next(10000000);
var deltaTablePath = $"/delta/delta-table-{sessionId}";

deltaTablePath
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val sessionId = scala.util.Random.nextInt(1000000)
val deltaTablePath = s"/delta/delta-table-$sessionId";
```

::: zone-end

Результат будет иметь такой вид:

"/delta/delta-table-335323"

## <a name="create-a-table"></a>Создание таблицы

Чтобы создать таблицу Delta Lake, запишите DataFrame в формате delta. Вы можете изменить формат Parquet, CSV, JSON и т. д. на delta.

В приведенном ниже коде показано, как создать новую таблицу Delta Lake с помощью схемы, выводимой из DataFrame.

:::zone pivot = "programming-language-python"

```python
data = spark.range(0,5)
data.show()
data.write.format("delta").save(delta_table_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var data = spark.Range(0,5);
data.Show();
data.Write().Format("delta").Save(deltaTablePath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val data = spark.range(0, 5)
data.show
data.write.format("delta").save(deltaTablePath)
```

::: zone-end

Результат будет иметь такой вид:

| ID|
|---|
|  0|
|  1|
|  2|
|  3|
|  4|

## <a name="read-data"></a>Чтение данных

Чтение данных в таблице Delta Lake осуществляется путем указания пути к файлам и формата delta.

:::zone pivot = "programming-language-python"

```python
df = spark.read.format("delta").load(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var df = spark.Read().Format("delta").Load(deltaTablePath);
df.Show()
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val df = spark.read.format("delta").load(deltaTablePath)
df.show()
```

::: zone-end

Результат будет иметь такой вид:

| ID|
|---|
|  1|
|  3|
|  4|
|  0|
|  2|

Порядок вывода результатов отличается от приведенного выше, где порядок заранее не был определен явно.

## <a name="update-table-data"></a>Обновление данных таблицы

Delta Lake поддерживает несколько операций по изменению таблиц с помощью стандартных API-интерфейсов DataFrame — это одно из наибольших улучшений, добавленных форматом delta. В следующем примере выполняется пакетное задание для перезаписи данных в таблице.

:::zone pivot = "programming-language-python"

```python
data = spark.range(5,10)
data.write.format("delta").mode("overwrite").save(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var data = spark.Range(5,10);
data.Write().Format("delta").Mode("overwrite").Save(deltaTablePath);
df.Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val data = spark.range(5, 10)
data.write.format("delta").mode("overwrite").save(deltaTablePath)
df.show()
```

::: zone-end

Результат будет иметь такой вид:

| ID|
|---|
|  7|
|  8|
|  5|
|  9|
|  6|

Здесь можно увидеть, что все пять записей были обновлены для хранения новых значений.

## <a name="save-as-catalog-tables"></a>Сохранение как таблиц в каталоге

Delta Lake может выполнять запись в управляемые или внешние таблицы каталога.

:::zone pivot = "programming-language-python"

```python
data.write.format("delta").saveAsTable("ManagedDeltaTable")
spark.sql("CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '{0}'".format(delta_table_path))
spark.sql("SHOW TABLES").show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
data.Write().Format("delta").SaveAsTable("ManagedDeltaTable");
spark.Sql($"CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '{deltaTablePath}'");
spark.Sql("SHOW TABLES").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
data.write.format("delta").saveAsTable("ManagedDeltaTable")
spark.sql(s"CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '$deltaTablePath'")
spark.sql("SHOW TABLES").show
```

::: zone-end

Результат будет иметь такой вид:

|База данных|         tableName|isTemporary|
|--------|------------------|-----------|
| default|externaldeltatable|      false|
| default| manageddeltatable|      false|

С помощью этого кода вы создали новую таблицу в каталоге на основе существующей таблицы данных, называемой управляемой таблицей. Затем вы определили в каталоге новую внешнюю таблицу, которая использует существующее расположение, называемое внешней таблицей. В выходных данных можно увидеть, что обе таблицы, независимо от того, как они были созданы, представлены в каталоге.

Теперь можно просмотреть расширенные свойства обеих таблиц.

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE EXTENDED ManagedDeltaTable").show(truncate=False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql("DESCRIBE EXTENDED ManagedDeltaTable").Show(truncate: 0);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql("DESCRIBE EXTENDED ManagedDeltaTable").show(truncate=false)
```

::: zone-end

Результат будет иметь такой вид:

|col_name                    |Тип данных                                                                                                    |comment|
|----------------------------|-------------------------------------------------------------------------------------------------------------|-------|
|идентификатор                          |BIGINT                                                                                                       |null   |
|                            |                                                                                                             |       |
|Подробные сведения о таблице  |                                                                                                             |       |
|База данных                    |default                                                                                                      |       |
|Таблица                       |manageddeltatable                                                                                            |       |
|Владелец                       |trusted-service-user                                                                                         |       |
|Время создания                |Sat Apr 25 00:35:34 UTC 2020                                                                                 |       |
|Последний доступ                 |Thu Jan 01 00:00:00 UTC 1970                                                                                 |       |
|Автор                  |Spark 2.4.4.2.6.99.201-11401300                                                                              |       |
|Тип                        |MANAGED                                                                                                      |       |
|Поставщик                    |delta                                                                                                        |       |
|Свойства таблицы            |[transient_lastDdlTime=1587774934]                                                                           |       |
|Статистика                  |2407 байт                                                                                                   |       |
|Расположение                    |abfss://data@<data lake>.dfs.core.windows.net/synapse/workspaces/<workspace name>/warehouse/manageddeltatable|       |
|Библиотека Serde               |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                                                           |       |
|InputFormat                 |org.apache.hadoop.mapred.SequenceFileInputFormat                                                             |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat                                                    |       |
|Свойства хранилища          |[serialization.format=1]                                                                                     |       |

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE EXTENDED ExternalDeltaTable").show(truncate=False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql("DESCRIBE EXTENDED ExternalDeltaTable").Show(truncate: 0);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql("DESCRIBE EXTENDED ExternalDeltaTable").show(truncate=false)
```

::: zone-end

Результат будет иметь такой вид:

|col_name                    |Тип данных                                                             |comment|
|----------------------------|----------------------------------------------------------------------|-------|
|идентификатор                          |BIGINT                                                                |null   |
|                            |                                                                      |       |
|Подробные сведения о таблице  |                                                                      |       |
|База данных                    |default                                                               |       |
|Таблица                       |externaldeltatable                                                    |       |
|Владелец                       |trusted-service-user                                                  |       |
|Время создания                |Sat Apr 25 00:35:38 UTC 2020                                          |       |
|Последний доступ                 |Thu Jan 01 00:00:00 UTC 1970                                          |       |
|Автор                  |Spark 2.4.4.2.6.99.201-11401300                                       |       |
|Тип                        |EXTERNAL                                                              |       |
|Поставщик                    |DELTA                                                                 |       |
|Свойства таблицы            |[transient_lastDdlTime=1587774938]                                    |       |
|Расположение                    |abfss://data@<data lake>.dfs.core.windows.net/delta/delta-table-587152|       |
|Библиотека Serde               |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                    |       |
|InputFormat                 |org.apache.hadoop.mapred.SequenceFileInputFormat                      |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat             |       |
|Свойства хранилища          |[serialization.format=1]                                              |       |

## <a name="conditional-update-without-overwrite"></a>Условное обновление без перезаписи

Delta Lake предоставляет программные интерфейсы API для условного обновления, удаления и слияния (обычно эта операция называется upsert) данных в таблицах.

:::zone pivot = "programming-language-python"

```python
from delta.tables import *
from pyspark.sql.functions import *

delta_table = DeltaTable.forPath(spark, delta_table_path)

delta_table.update(
  condition = expr("id % 2 == 0"),
  set = { "id": expr("id + 100") })
delta_table.toDF().show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
using Microsoft.Spark.Extensions.Delta;
using Microsoft.Spark.Extensions.Delta.Tables;
using Microsoft.Spark.Sql;
using static Microsoft.Spark.Sql.Functions;

var deltaTable = DeltaTable.ForPath(deltaTablePath);

deltaTable.Update(
  condition: Expr("id % 2 == 0"),
  set: new Dictionary<string, Column>(){{ "id", Expr("id + 100") }});
deltaTable.ToDF().Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
import io.delta.tables._
import org.apache.spark.sql.functions._

val deltaTable = DeltaTable.forPath(deltaTablePath)

// Update every even value by adding 100 to it
deltaTable.update(
  condition = expr("id % 2 == 0"),
  set = Map("id" -> expr("id + 100")))
deltaTable.toDF.show
```

::: zone-end

Результат будет иметь такой вид:

| ID|
|---|
|106|
|108|
|  5|
|  7|
|  9|

Здесь вы только что добавили 100 к каждому четному идентификатору.

:::zone pivot = "programming-language-python"

```python
delta_table.delete("id % 2 == 0")
delta_table.toDF().show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.Delete(condition: Expr("id % 2 == 0"));
deltaTable.ToDF().Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.delete(condition = expr("id % 2 == 0"))
deltaTable.toDF.show
```

::: zone-end

Результат будет иметь такой вид:

| ID|
|---|
|  5|
|  7|
|  9|

Обратите внимание, что каждая четная строка была удалена.

:::zone pivot = "programming-language-python"

```python
new_data = spark.range(0,20).alias("newData")

delta_table.alias("oldData")\
    .merge(new_data.alias("newData"), "oldData.id = newData.id")\
    .whenMatchedUpdate(set = { "id": lit("-1")})\
    .whenNotMatchedInsert(values = { "id": col("newData.id") })\
    .execute()

delta_table.toDF().show(100)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var newData = spark.Range(20).As("newData");

deltaTable
    .As("oldData")
    .Merge(newData, "oldData.id = newData.id")
    .WhenMatched()
        .Update(new Dictionary<string, Column>() {{"id", Lit("-1")}})
    .WhenNotMatched()
        .Insert(new Dictionary<string, Column>() {{"id", Col("newData.id")}})
    .Execute();

deltaTable.ToDF().Show(100);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val newData = spark.range(0, 20).toDF

deltaTable.as("oldData").
  merge(
    newData.as("newData"),
    "oldData.id = newData.id").
  whenMatched.
  update(Map("id" -> lit(-1))).
  whenNotMatched.
  insert(Map("id" -> col("newData.id"))).
  execute()

deltaTable.toDF.show()
```

::: zone-end

Результат будет иметь такой вид:

| ID|
|---|
| 18|
| 15|
| 19|
|  2|
|  1|
|  6|
|  8|
|  3|
| -1|
| 10|
| 13|
|  0|
| 16|
|  4|
| -1|
| 12|
| 11|
| 14|
| -1|
| 17|

Здесь есть сочетание существующих данных. Существующим данным было присвоено значение-1 в пути кода обновления (WhenMatched). Были также добавлены новые данные, созданные в верхней части фрагмента и добавленные с помощью пути кода вставки (WhenNotMatched).

### <a name="history"></a>Журнал

Delta Lake имеет возможность разрешать поиск в журнале таблицы. То есть изменения, внесенные в базовую разностную таблицу. В следующей ячейке показано, насколько просто проверять журнал.

:::zone pivot = "programming-language-python"

```python
delta_table.history().show(20, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.History().Show(20, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.history.show(false)
```

::: zone-end

Результат будет иметь такой вид:

|version|          TIMESTAMP|userId|userName|Операция|                                                operationParameters| задание|записная книжка|clusterId|readVersion|isolationLevel|isBlindAppend|
|-------|-------------------|------|--------|---------|-------------------------------------------------------------------|----|--------|---------|-----------|--------------|-------------|
|      4|2020-04-25 00:36:27|  null|    null|    MERGE|                       [predicate -> (oldData.`ID` = newData.`ID`)]|null|    null|     null|          3|          null|        false|
|      3|2020-04-25 00:36:08|  null|    null|   DELETE|[predicate -> ["((`ID` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|null|    null|     null|          2|          null|        false|
|      2|2020-04-25 00:35:51|  null|    null|   UPDATE| [predicate -> ((ID#744L % cast(2 as bigint)) = cast(0 as bigint))]|null|    null|     null|          1|          null|        false|
|      1|2020-04-25 00:35:05|  null|    null|    WRITE|                             [mode -> Overwrite, partitionBy -> []]|null|    null|     null|          0|          null|        false|
|      0|2020-04-25 00:34:34|  null|    null|    WRITE|                         [mode -> ErrorIfExists, partitionBy -> []]|null|    null|     null|       null|          null|         true|

Здесь можно увидеть все изменения, внесенные в приведенные выше фрагменты кода.

## <a name="read-older-versions-of-data-using-time-travel"></a>Чтение более старых версий данных с помощью перехода по времени

Можно выполнить запрос к предыдущим моментальным снимкам таблицы Delta Lake, используя функцию, которая называется переходом по времени. Если требуется получить доступ к данным, которые были перезаписаны, можно запросить моментальный снимок таблицы, прежде чем будет перезаписан первый набор данных с помощью параметра versionAsOf.

После выполнения приведенной ниже ячейки вы должны увидеть первый набор данных перед тем, как он будет перезаписан. Переход по времени — это чрезвычайно эффективная функция, которая использует преимущества журнала транзакции Delta Lake для доступа к данным, которые больше не находятся в таблице. Удаление параметра версии 0 (или указание версии 1) позволит снова просматривать новые данные. Дополнительные сведения см. в разделе [Запрос старого моментального снимка таблицы](https://docs.delta.io/latest/delta-batch.html#deltatimetravel).

:::zone pivot = "programming-language-python"

```python
df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var df = spark.Read().Format("delta").Option("versionAsOf", 0).Load(deltaTablePath);
df.Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val df = spark.read.format("delta").option("versionAsOf", 0).load(deltaTablePath)
df.show()
```

::: zone-end

Результат будет иметь такой вид:

| ID|
|---|
|  0|
|  1|
|  4|
|  3|
|  2|

Здесь можно увидеть, что вы перешли к наиболее ранней версии данных.

## <a name="write-a-stream-of-data-to-a-table"></a>Запись потока данных в таблицу

Кроме того, запись в таблицу Delta Lake можно выполнять с помощью структурированного потока Spark. Журнал транзакций Delta Lake гарантирует ровно одну обработку, даже если в таблице имеются другие потоки или пакетные запросы, выполняющиеся параллельно. По умолчанию потоки выполняются в режиме добавления, когда добавляются новые записи в таблицу.

Дополнительные сведения об интеграции Delta Lake со структурированными потоками см. в статье [Потоковые операции чтения и записи в таблице](https://docs.delta.io/latest/delta-streaming.html).

Вот что мы делаем в ячейках далее:

* Ячейка 30. Показаны только что добавленные данные
* Ячейка 31. Журнал проверки
* Ячейка 32. Завершение задания структурированного потока
* Ячейка 33. Журнал проверки <-- можно видеть, что добавление остановлено

Сначала необходимо настроить простое задание потоковой передачи Spark для создания последовательности и сделать задание записи в таблицу Delta Lake.

:::zone pivot = "programming-language-python"

```python
streaming_df = spark.readStream.format("rate").load()
stream = streaming_df\
    .selectExpr("value as id")\
    .writeStream\
    .format("delta")\
    .option("checkpointLocation", "/tmp/checkpoint-{0}".format(session_id))\
    .start(delta_table_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var streamingDf = spark.ReadStream().Format("rate").Load();
var stream = streamingDf.SelectExpr("value as id").WriteStream().Format("delta").Option("checkpointLocation", $"/tmp/checkpoint-{sessionId}").Start(deltaTablePath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val streamingDf = spark.readStream.format("rate").load()
val stream = streamingDf.select($"value" as "id").writeStream.format("delta").option("checkpointLocation", s"/tmp/checkpoint-$sessionId").start(deltaTablePath)
```

::: zone-end

## <a name="read-a-stream-of-changes-from-a-table"></a>Чтение потока изменений из таблицы

Пока поток записывается в таблицу Delta Lake, можно также читать данные из этой таблицы в качестве источника потоковой передачи. Например, можно запустить другой потоковый запрос, который выводит все изменения, внесенные в таблицу Delta Lake.

:::zone pivot = "programming-language-python"

```python
delta_table.toDF().sort(col("id").desc()).show(100)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.ToDF().Sort(Col("id").Desc()).Show(100);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.toDF.sort($"id".desc).show
```

::: zone-end

Результат будет иметь такой вид:

| ID|
|---|
| 19|
| 18|
| 17|
| 16|
| 15|
| 14|
| 13|
| 12|
| 11|
| 10|
|  8|
|  6|
|  4|
|  3|
|  2|
|  1|
|  0|
| -1|
| -1|
| -1|

:::zone pivot = "programming-language-python"

```python
delta_table.history().drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").show(20, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.History().Drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").Show(20, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.history.show
```

::: zone-end

Результат будет иметь такой вид:

|version|          TIMESTAMP|       Операция|                                                                  operationParameters|readVersion|
|-------|-------------------|----------------|-------------------------------------------------------------------------------------|-----------|
|      5|2020-04-25 00:37:09|ПОТОКОВАЯ ПЕРЕДАЧА ОБНОВЛЕНИЯ|[outputMode -> Append, queryId -> d26b4f8a-7e5a-44f2-a5fb-23a7bd02aef7, epochId -> 0]|          4|
|      4|2020-04-25 00:36:27|           MERGE|                                         [predicate -> (oldData.`id` = newData.`id`)]|          3|
|      3|2020-04-25 00:36:08|          DELETE|                  [predicate -> ["((`id` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|          2|
|      2|2020-04-25 00:35:51|          UPDATE|                   [predicate -> ((id#744L % cast(2 as bigint)) = cast(0 as bigint))]|          1|
|      1|2020-04-25 00:35:05|           WRITE|                                               [mode -> Overwrite, partitionBy -> []]|          0|
|      0|2020-04-25 00:34:34|           WRITE|                                           [mode -> ErrorIfExists, partitionBy -> []]|       null|

Здесь отбрасываются некоторые малоинтересные столбцы, чтобы упростить представление журнала.

:::zone pivot = "programming-language-python"

```python
stream.stop()
delta_table.history().drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").show(100, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
stream.Stop();
deltaTable.History().Drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").Show(100, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
stream.stop
deltaTable.history.show
```

Результат будет иметь такой вид:

|version|          TIMESTAMP|       Операция|                                                                  operationParameters|readVersion|
|-------|-------------------|----------------|-------------------------------------------------------------------------------------|-----------|
|      5|2020-04-25 00:37:09|ПОТОКОВАЯ ПЕРЕДАЧА ОБНОВЛЕНИЯ|[outputMode -> Append, queryId -> d26b4f8a-7e5a-44f2-a5fb-23a7bd02aef7, epochId -> 0]|          4|
|      4|2020-04-25 00:36:27|           MERGE|                                         [predicate -> (oldData.`id` = newData.`id`)]|          3|
|      3|2020-04-25 00:36:08|          DELETE|                  [predicate -> ["((`id` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|          2|
|      2|2020-04-25 00:35:51|          UPDATE|                   [predicate -> ((id#744L % cast(2 as bigint)) = cast(0 as bigint))]|          1|
|      1|2020-04-25 00:35:05|           WRITE|                                               [mode -> Overwrite, partitionBy -> []]|          0|
|      0|2020-04-25 00:34:34|           WRITE|                                           [mode -> ErrorIfExists, partitionBy -> []]|       null|

::: zone-end

## <a name="convert-parquet-to-delta"></a>Преобразование формата Parquet в Delta

Преобразование из формата Parquet в формат delta можно выполнить зразу на месте.

Здесь будет выполняться проверка того, представлена ли существующая таблица в формате delta.
:::zone pivot = "programming-language-python"

```python
parquet_path = "/parquet/parquet-table-{0}".format(session_id)
data = spark.range(0,5)
data.write.parquet(parquet_path)
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var parquetPath = $"/parquet/parquet-table-{sessionId}";
var data = spark.Range(0,5);
data.Write().Parquet(parquetPath);
DeltaTable.IsDeltaTable(parquetPath)
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val parquetPath = s"/parquet/parquet-table-$sessionId"
val data = spark.range(0,5)
data.write.parquet(parquetPath)
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

Результат будет иметь такой вид:

Неверно

Далее будет выполняться преобразование данных в формат delta и проверка их работоспособности.

:::zone pivot = "programming-language-python"

```python
DeltaTable.convertToDelta(spark, "parquet.`{0}`".format(parquet_path))
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
DeltaTable.ConvertToDelta(spark, $"parquet.`{parquetPath}`");
DeltaTable.IsDeltaTable(parquetPath)
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
DeltaTable.convertToDelta(spark, s"parquet.`$parquetPath`")
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

Результат будет иметь такой вид:

Верно

## <a name="sql-support"></a>Поддержка SQL

Delta поддерживает служебные команды для таблиц с помощью SQL. SQL позволяет выполнять следующие операции.

* Получение журнала DeltaTable
* Опорожнять DeltaTable
* Преобразование файла Parquet в формат delta

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE HISTORY delta.`{0}`".format(delta_table_path)).show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql($"DESCRIBE HISTORY delta.`{deltaTablePath}`").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql(s"DESCRIBE HISTORY delta.`$deltaTablePath`").show()
```

::: zone-end

Результат будет иметь такой вид:

|version|          TIMESTAMP|userId|userName|       Операция| operationParameters| задание|записная книжка|clusterId|readVersion|isolationLevel|isBlindAppend|
|-------|-------------------|------|--------|----------------|--------------------|----|--------|---------|-----------|--------------|-------------|
|      5|2020-04-25 00:37:09|  null|    null|ПОТОКОВАЯ ПЕРЕДАЧА ОБНОВЛЕНИЯ|[outputMode -> Ap...|null|    null|     null|          4|          null|         true|
|      4|2020-04-25 00:36:27|  null|    null|           MERGE|[predicate -> (ol...|null|    null|     null|          3|          null|        false|
|      3|2020-04-25 00:36:08|  null|    null|          DELETE|[predicate -> ["(...|null|    null|     null|          2|          null|        false|
|      2|2020-04-25 00:35:51|  null|    null|          UPDATE|[predicate -> ((i...|null|    null|     null|          1|          null|        false|
|      1|2020-04-25 00:35:05|  null|    null|           WRITE|[mode -> Overwrit...|null|    null|     null|          0|          null|        false|
|      0|2020-04-25 00:34:34|  null|    null|           WRITE|[mode -> ErrorIfE...|null|    null|     null|       null|          null|         true|

:::zone pivot = "programming-language-python"

```python
spark.sql("VACUUM delta.`{0}`".format(delta_table_path)).show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql($"VACUUM delta.`{deltaTablePath}`").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql(s"VACUUM delta.`$deltaTablePath`").show()
```

::: zone-end

Результат будет иметь такой вид:

|                path|
|--------------------|
|abfss://data@arca...|

Здесь выполнится проверка, что таблица не в формате delta, а затем она преобразует ее в формат delta с помощью Spark SQL и будет получено подтверждение, что она была преобразована правильно.

:::zone pivot = "programming-language-python"

```python
parquet_id = random.randint(0,1000)
parquet_path = "/parquet/parquet-table-{0}-{1}".format(session_id, parquet_path)
data = spark.range(0,5)
data.write.parquet(parquet_path)
DeltaTable.isDeltaTable(spark, parquet_path)
spark.sql("CONVERT TO DELTA parquet.`{0}`".format(parquet_path))
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var parquetId =  (new Random()).Next(10000000);
var parquetPath = $"/parquet/parquet-table-{sessionId}-{parquetId}";
var data = spark.Range(0,5);
data.Write().Parquet(parquetPath);
DeltaTable.IsDeltaTable(parquetPath);
spark.Sql($"CONVERT TO DELTA parquet.`{parquetPath}`");
DeltaTable.IsDeltaTable(parquetPath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val parquetId = scala.util.Random.nextInt(1000)
val parquetPath = s"/parquet/parquet-table-$sessionId-$parquetId"
val data = spark.range(0,5)
data.write.parquet(parquetPath)
DeltaTable.isDeltaTable(parquetPath)
spark.sql(s"CONVERT TO DELTA parquet.`$parquetPath`")
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

Результат будет иметь такой вид:

Верно

Более полную документацию см. на [странице документации по Delta Lake](https://docs.delta.io/latest/delta-intro.html).

Дополнительные сведения см. на странице [проекта Delta Lake](https://github.com/delta-io/delta).

## <a name="next-steps"></a>Дальнейшие действия

* [Документация по .NET для Apache Spark](/dotnet/spark)
* [Azure Synapse Analytics](../index.yml)