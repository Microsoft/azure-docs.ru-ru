---
title: Перенос данных из Apache Cassandra в Azure Cosmos DB API Cassandra с использованием кирпичей данных (Spark)
description: Узнайте, как перенести данные из базы данных Apache Cassandra в Azure Cosmos DB API Cassandra с помощью Azure Databricks и Spark.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 03/10/2021
ms.author: thvankra
ms.reviewer: thvankra
ms.openlocfilehash: caf9cbb0ca017ee00c5061d94e0d37703194943d
ms.sourcegitcommit: 87a6587e1a0e242c2cfbbc51103e19ec47b49910
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103573386"
---
# <a name="migrate-data-from-cassandra-to-azure-cosmos-db-cassandra-api-account-using-azure-databricks"></a>Перенос данных из Cassandra в учетную запись API Cassandra Azure Cosmos DB с помощью Azure Databricks
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

API Cassandra в Azure Cosmos DB стали отличным выбором для корпоративных рабочих нагрузок, работающих на Apache Cassandra, по различным причинам: 

* **Отсутствие дополнительных издержек при управлении и мониторинге:** Она устраняет издержки на управление множеством параметров в файлах ОС, ВИРТУАЛЬНОЙ машины Java и YAML и их взаимодействиях.

* **Значительные сокращения затрат:** Вы можете сэкономить затраты с Azure Cosmos DB, включая стоимость виртуальных машин, пропускной способности и применимых лицензий. Кроме того, вам не нужно управлять центрами обработки данных, серверами, хранилищем SSD, сетью и расходами электроэнергии. 

* **Возможность использования существующего кода и средств:** Azure Cosmos DB обеспечивает совместимость на уровне кабельных протоколов с существующими пакетами SDK и средствами Cassandra. Благодаря этой совместимости можно использовать имеющуюся базу кода с API Cassandra для Azure Cosmos DB, внеся простые изменения.

Существует несколько способов миграции рабочих нагрузок базы данных с одной платформы на другую. [Azure Databricks](https://azure.microsoft.com/services/databricks/) — это платформа в качестве предложения услуги для [Apache Spark](https://spark.apache.org/) , которая предлагает способ выполнения автономных миграций в крупном масштабе. В этой статье описываются шаги, необходимые для переноса данных из собственных пространств ключей и таблиц Apache Cassandra в Azure Cosmos DB API Cassandra с помощью Azure Databricks.

## <a name="prerequisites"></a>Предварительные требования

* [Предоставление учетной записи API Cassandra для Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account)

* [Просмотр основных сведений о подключении к API Cassandra для Azure Cosmos DB](cassandra-spark-generic.md)

* Чтобы обеспечить совместимость, ознакомьтесь с [поддерживаемыми функциями Azure Cosmos DB API Cassandra](cassandra-support.md) .

* Убедитесь, что вы уже создали пустые пространства ключей и таблицы в целевой Azure Cosmos DB учетной записи API Cassandra

* [Использование cqlsh или размещенной оболочки для проверки](cassandra-support.md#hosted-cql-shell-preview)

## <a name="provision-an-azure-databricks-cluster"></a>Предоставление кластера Azure Databricks

Вы можете выполнить инструкции по [подготовке кластера Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal). Рекомендуется выбрать среду выполнения для модуля, версия 7,5, которая поддерживает Spark 3,0:

:::image type="content" source="./media/cassandra-migrate-cosmos-db-databricks/databricks-runtime.png" alt-text="Среда выполнения модулями":::


## <a name="add-dependencies"></a>Добавление зависимостей

Чтобы подключиться как к собственным, так и Cosmos DBным конечным точкам Cassandra, необходимо добавить в кластер библиотеку соединителей Apache Spark Cassandra. В кластере выберите библиотеки. > установить New-> Maven. добавить `com.datastax.spark:spark-cassandra-connector-assembly_2.12:3.0.0` в Maven координатах:

:::image type="content" source="./media/cassandra-migrate-cosmos-db-databricks/databricks-search-packages.png" alt-text="Пакеты поиска в модулях":::

Выберите установить и при завершении установки убедитесь, что кластер перезапускается. 

> [!NOTE]
> Убедитесь, что кластер кирпичей перезапускается после установки библиотеки соединителей Cassandra.

## <a name="create-scala-notebook-for-migration"></a>Создание записной книжки Scala для миграции

Создайте записную книжку Scala в модулях, используя следующий код. Замените исходные и целевые конфигурации Cassandra соответствующими учетными данными, исходными и целевыми пространств ключей и таблицами, а затем выполните:

```scala
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql._
import org.apache.spark.SparkContext

// source cassandra configs
val nativeCassandra = Map( 
    "spark.cassandra.connection.host" -> "<Source Cassandra Host>",
    "spark.cassandra.connection.port" -> "9042",
    "spark.cassandra.auth.username" -> "<USERNAME>",
    "spark.cassandra.auth.password" -> "<PASSWORD>",
    "spark.cassandra.connection.ssl.enabled" -> "false",
    "keyspace" -> "<KEYSPACE>",
    "table" -> "<TABLE>"
)

//target cassandra configs
val cosmosCassandra = Map( 
    "spark.cassandra.connection.host" -> "<USERNAME>.cassandra.cosmos.azure.com",
    "spark.cassandra.connection.port" -> "10350",
    "spark.cassandra.auth.username" -> "<USERNAME>",
    "spark.cassandra.auth.password" -> "<PASSWORD>",
    "spark.cassandra.connection.ssl.enabled" -> "true",
    "keyspace" -> "<KEYSPACE>",
    "table" -> "<TABLE>",
    //throughput related settings below - tweak these depending on data volumes. 
    "spark.cassandra.output.batch.size.rows"-> "1",
    "spark.cassandra.output.concurrent.writes" -> "1000",
    "spark.cassandra.concurrent.reads" -> "512",
    "spark.cassandra.output.batch.grouping.buffer.size" -> "1000",
    "spark.cassandra.connection.keep_alive_ms" -> "600000000"
)

//Read from native Cassandra
val DFfromNativeCassandra = sqlContext
  .read
  .format("org.apache.spark.sql.cassandra")
  .options(nativeCassandra)
  .load
  
//Write to CosmosCassandra
DFfromNativeCassandra
  .write
  .format("org.apache.spark.sql.cassandra")
  .options(cosmosCassandra)
  .mode(SaveMode.Append)
  .save
```

> [!NOTE]
> Значения параметров `spark.cassandra.output.batch.size.rows` и `spark.cassandra.output.concurrent.writes` , а также число рабочих ролей в кластере Spark — это важные конфигурации, которые необходимо настроить, чтобы избежать [ограничения скорости](/samples/azure-samples/azure-cosmos-cassandra-java-retry-sample/azure-cosmos-db-cassandra-java-retry-sample/), которое происходит, когда запросы Azure Cosmos DB превышают подготовленную пропускную способность/([единицы запроса](./request-units.md)). Может потребоваться изменить эти параметры в зависимости от количества исполнителей в кластере Spark и, возможно, размера (и, следовательно, стоимости на ЕДИНИЦу) каждой записи, записываемой в целевые таблицы.

## <a name="troubleshooting"></a>Устранение неполадок

### <a name="rate-limiting-429-error"></a>Ограничение скорости (429 ошибка)
Вы можете увидеть код ошибки 429 или `request rate is large` текст ошибки, несмотря на то, что указанные выше параметры имеют минимальные значения. Ниже перечислены некоторые сценарии.

- **Пропускная способность, выделенная для таблицы, меньше 6000 [единиц запросов](./request-units.md)**. Даже при минимальных настройках Spark сможет выполнять операции записи с частотой около 6000 единиц запросов. Если вы подготовили таблицу в пространства ключей с подготовленной пропускной способностью, возможно, что эта таблица имеет менее 6000 в секунду, доступной во время выполнения. Убедитесь, что в таблице, в которую выполняется миграция, имеется по крайней мере 6000. д. при выполнении миграции, а также при необходимости выделения выделенных единиц запросов для этой таблицы. 
- **Чрезмерное смещение данных с большим объемом данных**. Если имеется большой объем данных (т. е. строк таблицы) для переноса в заданную таблицу, но в данных имеется значительный отклонения (т. е. большое количество записей, записываемых для одного и того же значения ключа секции), вы можете по-прежнему использовать ограничение скорости, даже если в таблице было подготовлено большое количество [единиц запросов](./request-units.md) . Это обусловлено тем, что единицы запросов делятся между физическими секциями, а высокая интенсивность данных может привести к узким местам запросов к одной секции, что приводит к ограничению частоты. В этом случае рекомендуется сократить до минимальных настроек пропускной способности в Spark, чтобы избежать ограничения скорости и заставить миграцию выполняться медленно. Этот сценарий может оказаться более распространенным при переносе ссылочных таблиц или элементов управления, где доступ менее частый, но может быть очень высок. Однако если в любой другой тип таблицы имеется значительный отклонения, может быть целесообразно изучить модель данных, чтобы избежать проблем с горячей секцией для рабочей нагрузки во время операций стабильного состояния. 



## <a name="next-steps"></a>Дальнейшие действия

* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md) 
* [Рекомендации по использованию ключа секции](partitioning-overview.md#choose-partitionkey)
* [Оценка единиц запросов в секунду с помощью планировщика ресурсов Azure Cosmos DB](estimate-ru-with-capacity-planner.md)
* [Эластичное масштабирование в Azure Cosmos DB API Cassandra](manage-scale-cassandra.md)
