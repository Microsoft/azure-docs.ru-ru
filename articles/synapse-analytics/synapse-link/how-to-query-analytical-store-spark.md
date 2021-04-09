---
title: Взаимодействие с Azure Cosmos DB с помощью Apache Spark в Azure Synapse Link
description: Взаимодействие с Azure Cosmos DB с помощью Apache Spark в Azure Synapse Link
services: synapse-analytics
author: Rodrigossz
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 09/15/2020
ms.author: rosouz
ms.reviewer: jrasnick
ms.custom: cosmos-db
ms.openlocfilehash: 4a8367ea41ea96d8a412af965346684737d190fe
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105627580"
---
# <a name="interact-with-azure-cosmos-db-using-apache-spark-in-azure-synapse-link"></a>Взаимодействие с Azure Cosmos DB с помощью Apache Spark в Azure Synapse Link

Из этой статьи вы узнаете, как взаимодействовать с Azure Cosmos DB с помощью Synapse Apache Spark. Благодаря полной поддержке Scala, Python, SparkSQL и C# Synapse Apache Spark является центральным элементом для аналитики, инжиниринга, обработки и анализа данных, а также для сценариев исследования данных в [Azure Synapse Link для Azure Cosmos DB](../../cosmos-db/synapse-link.md).

При взаимодействии с Azure Cosmos DB поддерживаются следующие возможности.
* Synapse Apache Spark позволяет анализировать данные в контейнерах Azure Cosmos DB, которые поддерживают Azure Synapse Link практически в реальном времени, не влияя на производительность транзакционных рабочих нагрузок. Для запроса [аналитического хранилища](../../cosmos-db/analytical-store-introduction.md) Azure Cosmos DB из Spark доступны следующие два варианта:
    + Загрузка в кадр данных Spark.
    + Создание таблицы Spark
* Synapse Apache Spark также позволяет принимать данные в Azure Cosmos DB. Важно отметить, что данные всегда поступают в контейнеры Azure Cosmos DB через хранилище транзакций. Если включена поддержка Synapse Link, все новые операции вставки, обновления и удаления автоматически синхронизируются с аналитическим хранилищем.
* Synapse Apache Spark также поддерживает структурированную потоковую передачу Spark в Azure Cosmos DB в качестве источника и приемника. 

В следующих разделах излагается синтаксис функций, описанных выше. Жесты в рабочей области Azure Synapse Analytics разработаны с целью предоставления простого готового интерфейса для начала работы. Чтобы просмотреть жесты, щелкните правой кнопкой мыши контейнер Azure Cosmos DB на вкладке **Данные** рабочей области Synapse. С помощью жестов можно быстро создать код и скорректировать его в соответствии с потребностями. Жесты также идеально подходят для обнаружения данных одним щелчком мыши.

> [!IMPORTANT]
> Вы должны учитывать некоторые ограничения в аналитической схеме, которые могут привести к непредвиденному поведению в операциях загрузки данных.
> Например, в аналитической схеме доступны только первые 1000 свойств из транзакционной схемы, свойства с пробелами недоступны и т. д. Если вы столкнулись с неожиданными результатами, проверьте [ограничения схемы аналитического хранилища](../../cosmos-db/analytical-store-introduction.md#schema-constraints) для получения дополнительных сведений.

## <a name="query-azure-cosmos-db-analytical-store"></a>Запрос аналитического хранилища Azure Cosmos DB

Прежде чем вы узнаете о двух возможных вариантах для запроса аналитического хранилища Azure Cosmos DB, загрузки в Spark DataFrame и создании таблицы Spark, следует ознакомиться с различиями в работе. Это позволит выбрать вариант, который лучше всего соответствует вашим потребностям.

Различия заключаются в том, должны ли изменения базовых данных в контейнере Azure Cosmos DB автоматически отражаться в анализе, выполненном в Spark. При регистрации кадра данных Spark или создании таблицы Spark для аналитического хранилища контейнера метаданные, связанные с текущим моментальным снимком данных в аналитическом хранилище, извлекаются в Spark. Это позволяет выполнить эффективное включение последующего анализа. Важно отметить, что так как Spark следует политике отложенной оценки, если только не вызвано действие в кадре данных Spark или не будет выполнен запрос SparkSQL в таблице Spark, фактические данные не будут получены из аналитического хранилища базового контейнера.

В случае **загрузки в кадр данных Spark** извлеченные метаданные кэшируются в течение времени существования сеанса Spark. Поэтому последующие действия, вызываемые в кадре данных, оцениваются по моментальному снимку аналитического хранилища на момент создания кадра данных.

С другой стороны, в случае **создания таблицы Spark** метаданные состояния аналитического хранилища не кэшируются в Spark и повторно загружаются при каждом выполнении запроса SparkSQL в таблице Spark.

Таким образом, вы можете выполнить загрузку в кадр данных Spark или создать таблицу Spark в зависимости от того, хотите ли вы оценить ваш анализ Spark по фиксированному снимку аналитического хранилища или по последнему снимку аналитического хранилища соответственно.

> [!NOTE]
> Чтобы запросить API Azure Cosmos DB учетных записей Mongo DB, ознакомьтесь с [представлением полноценной схемы](../../cosmos-db/analytical-store-introduction.md#analytical-schema) в аналитическом хранилище и используемыми именами расширенных свойств.

### <a name="load-to-spark-dataframe"></a>Загрузка в кадр данных Spark.

В этом примере вы создадите кадр данных Spark, который указывает на аналитическое хранилище Azure Cosmos DB. Затем можно выполнить дополнительный анализ, вызвав действия Spark для кадра данных. Эта операция не влияет на хранилище транзакций.

Синтаксис на языке **Python** будет следующим:
```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

df = spark.read.format("cosmos.olap")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .load()
```

Эквивалентный синтаксис на языке **Scala** будет следующим:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val df_olap = spark.read.format("cosmos.olap").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>").
    load()
```

### <a name="create-spark-table"></a>Создание таблицы Spark

В этом примере вы создадите таблицу Spark, которая указывает на аналитическое хранилище Azure Cosmos DB. Затем можно выполнить дополнительный анализ, вызвав запросы SparkSQL к таблице. Эта операция никак не влияет на хранилище транзакций и не влечет за собой перемещение данных. Если вы решили удалить эту таблицу Spark, это никак не повлияет на базовый контейнер Azure Cosmos DB и соответствующее аналитическое хранилище. 

Этот сценарий удобен для повторного использования таблиц Spark с применением средств сторонних разработчиков и предоставления доступа к базовым данным в среде выполнения.

Синтаксис для создания таблицы Spark выглядит следующим образом:
```sql
%%sql
-- To select a preferred list of regions in a multi-region Azure Cosmos DB account, add spark.cosmos.preferredRegions '<Region1>,<Region2>' in the config options

create table call_center using cosmos.olap options (
    spark.synapse.linkedService '<enter linked service name>',
    spark.cosmos.container '<enter container name>'
)
```

> [!NOTE]
> Если у вас есть сценарии, в которых схема базового контейнера Azure Cosmos DB изменяется с течением времени, и если вы хотите, чтобы обновленная схема автоматически отражалась в запросах к таблице Spark, в параметрах таблицы Spark установите для параметра `spark.cosmos.autoSchemaMerge` значение `true`.


## <a name="write-spark-dataframe-to-azure-cosmos-db-container"></a>Запись кадра данных Spark в контейнер Azure Cosmos DB

В этом примере вы запишете кадр данных Spark в контейнер Azure Cosmos DB. Эта операция влияет на производительность рабочих нагрузок транзакций и на использование единиц запросов, подготовленных в контейнере Azure Cosmos DB или общей базе данных.

Синтаксис на языке **Python** будет следующим:
```python
# Write a Spark DataFrame into an Azure Cosmos DB container
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

YOURDATAFRAME.write.format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .option("spark.cosmos.write.upsertEnabled", "true")\
    .mode('append')\
    .save()
```

Эквивалентный синтаксис на языке **Scala** будет следующим:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

import org.apache.spark.sql.SaveMode

df.write.format("cosmos.oltp").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>"). 
    option("spark.cosmos.write.upsertEnabled", "true").
    mode(SaveMode.Overwrite).
    save()
```

## <a name="load-streaming-dataframe-from-container"></a>Загрузка DataFrame для потоковой передачи из контейнера
В этом жесте вы будете использовать возможность потоковой передачи Spark для загрузки данных из контейнера в DataFrame. Данные будут храниться в основной учетной записи Data Lake (и файловой системе), подключенной к рабочей области. 
> [!NOTE]
> Если вы хотите ссылаться на внешние библиотеки в Synapse Apache Spark, просмотрите дополнительные сведения [здесь](#external-library-management). Например, если вы хотите принять кадр данных Spark в контейнере Cosmos DB API для Mongo DB, можно использовать соединитель Mongo DB для Spark, описанный [здесь](https://docs.mongodb.com/spark-connector/master/).

## <a name="load-streaming-dataframe-from-azure-cosmos-db-container"></a>Загрузка кадра данных для потоковой передачи из контейнера Azure Cosmos DB
В этом примере вы будете использовать функцию структурированной потоковой передачи Spark для загрузки данных из контейнера Azure Cosmos DB в кадр данных потоковой передачи Spark с помощью функции канала изменений в Azure Cosmos DB. Данные контрольных точек, используемые Spark, будут храниться в основной учетной записи озера данных (и файловой системе), подключенной к рабочей области.

Если папка */localReadCheckpointFolder* отсутствует (в примере ниже), она будет создана автоматически. Эта операция влияет на производительность рабочих нагрузок транзакций и на использование единиц запросов, подготовленных в контейнере Azure Cosmos DB или общей базе данных.

Синтаксис на языке **Python** будет следующим:
```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

dfStream = spark.readStream\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .option("spark.cosmos.changeFeed.readEnabled", "true")\
    .option("spark.cosmos.changeFeed.startFromTheBeginning", "true")\
    .option("spark.cosmos.changeFeed.checkpointLocation", "/localReadCheckpointFolder")\
    .option("spark.cosmos.changeFeed.queryName", "streamQuery")\
    .load()
```

Эквивалентный синтаксис на языке **Scala** будет следующим:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val dfStream = spark.readStream.
    format("cosmos.oltp").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>").
    option("spark.cosmos.changeFeed.readEnabled", "true").
    option("spark.cosmos.changeFeed.startFromTheBeginning", "true").
    option("spark.cosmos.changeFeed.checkpointLocation", "/localReadCheckpointFolder").
    option("spark.cosmos.changeFeed.queryName", "streamQuery").
    load()
```

## <a name="write-streaming-dataframe-to-azure-cosmos-db-container"></a>Запись кадра данных для потоковой передачи в контейнер Azure Cosmos DB
В этом примере вы запишете кадр данных для потоковой передачи в контейнер Azure Cosmos DB. Эта операция влияет на производительность рабочих нагрузок транзакций и на использование единиц запросов, подготовленных в контейнере Azure Cosmos DB или общей базе данных. Если папка */localWriteCheckpointFolder* отсутствует (в примере ниже), она будет создана автоматически. 

Синтаксис на языке **Python** будет следующим:

```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

# If you are using managed private endpoints for Azure Cosmos DB analytical store and using batch writes/reads and/or streaming writes/reads to transactional store you should set connectionMode to Gateway. 

streamQuery = dfStream\
        .writeStream\
        .format("cosmos.oltp")\
        .outputMode("append")\
        .option("checkpointLocation", "/localWriteCheckpointFolder")\
        .option("spark.synapse.linkedService", "<enter linked service name>")\
        .option("spark.cosmos.container", "<enter container name>")\
        .option("spark.cosmos.connection.mode", "gateway")\
        .start()

streamQuery.awaitTermination()
```

Эквивалентный синтаксис на языке **Scala** будет следующим:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

// If you are using managed private endpoints for Azure Cosmos DB analytical store and using batch writes/reads and/or streaming writes/reads to transactional store you should set connectionMode to Gateway. 

val query = dfStream.
            writeStream.
            format("cosmos.oltp").
            outputMode("append").
            option("checkpointLocation", "/localWriteCheckpointFolder").
            option("spark.synapse.linkedService", "<enter linked service name>").
            option("spark.cosmos.container", "<enter container name>").
            option("spark.cosmos.connection.mode", "gateway").
            start()

query.awaitTermination()
```

## <a name="external-library-management"></a>Управление внешними библиотеками

Из этого примера вы узнаете, как ссылаться на внешние библиотеки из JAR-файлов при использовании записных книжек Spark в рабочих областях Synpase Apache Spark. JAR-файлы можно поместить в контейнер в основной учетной записи озера данных, подключенной к рабочей области, а затем добавить в записную книжку Spark следующую инструкцию `%configure`:

```cmd
%%configure -f
{
    "jars": [
        "abfss://<storage container name>@<data lake account name>.dfs.core.windows.net/<path to jar>"
    ]
}
```
Если вы хотите отправить определения удаленных заданий Spark в пул бессерверный пул Apache Spark, узнайте, как ссылаться на внешние библиотеки, следуя инструкциям из этого [руководства](../spark/apache-spark-job-definitions.md).

## <a name="next-steps"></a>Следующие шаги

* [Примеры для начала работы с Azure Synapse Link на сайте GitHub](https://aka.ms/cosmosdb-synapselink-samples).
* [Поддерживаемые функции Azure Synapse Link для Azure Cosmos DB](./concept-synapse-link-cosmos-db-support.md)
* [Подключитесь к Synapse Link для Azure Cosmos DB](../quickstart-connect-synapse-link-cosmos-db.md).
