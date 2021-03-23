---
title: Перенос данных из Apache Cassandra в API Cassandra Azure Cosmos DB с помощью блоков данных (Spark)
description: Узнайте, как перенести данные из базы данных Apache Cassandra в API Cassandra Azure Cosmos DB с помощью Azure Databricks и Spark.
author: TheovanKraay
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: how-to
ms.date: 03/10/2021
ms.author: thvankra
ms.reviewer: thvankra
ms.openlocfilehash: caedefbf3887205b68bcd5de5e7cd5f1f7d7f53c
ms.sourcegitcommit: ba3a4d58a17021a922f763095ddc3cf768b11336
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104801015"
---
# <a name="migrate-data-from-cassandra-to-an-azure-cosmos-db-cassandra-api-account-by-using-azure-databricks"></a>Перенос данных из Cassandra в учетную запись API Cassandra Azure Cosmos DB с помощью Azure Databricks
[!INCLUDE[appliesto-cassandra-api](includes/appliesto-cassandra-api.md)]

API Cassandra в Azure Cosmos DB стали отличным выбором для корпоративных рабочих нагрузок, работающих на Apache Cassandra по нескольким причинам:

* **Отсутствие дополнительных издержек при управлении и мониторинге:** Она устраняет издержки на управление и мониторинг параметров в файлах операционной системы, ВИРТУАЛЬНОЙ машины Java и YAML и их взаимодействии.

* **Значительные сокращения затрат:** Вы можете сэкономить затраты с Azure Cosmos DB, включая затраты на виртуальные машины, пропускную способность и применимые лицензии. Вам не нужно управлять центрами обработки данных, серверами, хранилищем SSD, сетью и расходами электроэнергии.

* **Возможность использования существующего кода и средств:** Azure Cosmos DB обеспечивает совместимость на уровне протоколов с существующими пакетами SDK и средствами Cassandra. Такая совместимость гарантирует, что вы сможете использовать существующую базу кода с Azure Cosmos DB API Cassandra с тривиальными изменениями.

Существует множество способов переноса рабочих нагрузок базы данных с одной платформы на другую. [Azure Databricks](https://azure.microsoft.com/services/databricks/) является предложением платформы как услуги (PaaS) для [Apache Spark](https://spark.apache.org/) , которое позволяет выполнять автономную миграцию крупным образом. В этой статье описываются шаги, необходимые для переноса данных из собственных пространств ключейов Apache Cassandra и таблиц в Azure Cosmos DB API Cassandra с помощью Azure Databricks.

## <a name="prerequisites"></a>Предварительные требования

* [Подготавливает учетную запись API Cassandra Azure Cosmos DB](create-cassandra-dotnet.md#create-a-database-account).

* [Ознакомьтесь с основами подключения к API Cassandra Azure Cosmos DB](cassandra-spark-generic.md).

* Чтобы обеспечить совместимость, ознакомьтесь с [поддерживаемыми функциями Azure Cosmos DB API Cassandra](cassandra-support.md) .

* Убедитесь, что вы уже создали пустые пространств ключей и таблицы в целевой Azure Cosmos DB API Cassandra учетной записи.

* [Используйте cqlsh или размещенную оболочку для проверки](cassandra-support.md#hosted-cql-shell-preview).

## <a name="provision-an-azure-databricks-cluster"></a>Предоставление кластера Azure Databricks

Вы можете выполнить инструкции по [подготовке кластера Azure Databricks](/azure/databricks/scenarios/quickstart-create-databricks-workspace-portal). Рекомендуется выбрать среду выполнения для модуля, версия 7,5, которая поддерживает Spark 3,0.

:::image type="content" source="./media/cassandra-migrate-cosmos-db-databricks/databricks-runtime.png" alt-text="Снимок экрана, на котором показано, как найти версию среды выполнения для кирпичей.":::

## <a name="add-dependencies"></a>Добавление зависимостей

Необходимо добавить Apache Spark библиотеку соединителей Cassandra в кластер для подключения к собственным и Azure Cosmos DB конечным точкам Cassandra. В кластере выберите **библиотеки**  >  **Установка New**  >  **Maven**, а затем добавьте `com.datastax.spark:spark-cassandra-connector-assembly_2.12:3.0.0` в координаты Maven.

:::image type="content" source="./media/cassandra-migrate-cosmos-db-databricks/databricks-search-packages.png" alt-text="Снимок экрана, показывающий Поиск пакетов Maven в модулях.":::

Нажмите кнопку **установить**, а затем перезапустите кластер после завершения установки.

> [!NOTE]
> Убедитесь, что кластер кирпичей перезапускается после установки библиотеки соединителей Cassandra.

## <a name="create-scala-notebook-for-migration"></a>Создание записной книжки Scala для миграции

Создайте записную книжку Scala в модулях. Замените исходные и целевые конфигурации Cassandra соответствующими учетными данными, а также исходными и целевыми пространств ключей и таблицами. Затем выполните следующий код:

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
> `spark.cassandra.output.batch.size.rows`Значения и `spark.cassandra.output.concurrent.writes` и количество рабочих ролей в кластере Spark являются важными настройками, чтобы избежать [ограничения скорости](/samples/azure-samples/azure-cosmos-cassandra-java-retry-sample/azure-cosmos-db-cassandra-java-retry-sample/). Ограничение скорости происходит, когда запросы на Azure Cosmos DB превышает подготовленную пропускную способность или [единицы запроса](./request-units.md) (RUs). Может потребоваться изменить эти параметры в зависимости от количества исполнителей в кластере Spark и потенциального размера (и, следовательно, стоимости единиц) каждой записи, записываемой в целевые таблицы.

## <a name="troubleshoot"></a>Диагностика

### <a name="rate-limiting-429-error"></a>Ограничение скорости (429 ошибка)

Может появиться сообщение об ошибке 429 или "частота запросов слишком велика", даже если вы сократили настройки до минимальных значений. Следующие сценарии могут привести к ограничению скорости:

* **Пропускная способность, выделенная для таблицы, меньше 6 000 [единиц запросов](./request-units.md)**. Даже при минимальных настройках Spark может писаться с частотой около 6 000 единиц запросов. Если вы подготовили таблицу в пространства ключей с общей пропускной способностью, возможно, что в этой таблице доступно менее 6 000.

    При выполнении миграции убедитесь, что в таблице, в которую выполняется миграция, имеется по крайней мере 6 000. При необходимости выделите выделенные единицы запроса для этой таблицы.

* **Чрезмерное смещение данных с большим объемом данных**. Если у вас есть большой объем данных для переноса в заданную таблицу, но имеется значительный отклонения в данных (то есть большое количество записей, записываемых для одного и того же значения ключа секции), вы можете использовать ограничение скорости, даже если в таблице подготовлено несколько [единиц запросов](./request-units.md) . Единицы запросов делятся между физическими секциями, а высокая интенсивность данных может привести к узким местам запросов к одной секции.

    В этом случае Сократите до минимальных настроек пропускной способности в Spark и принуждает миграцию работать медленно. Этот сценарий может оказаться более распространенным при переносе ссылок или элементов управления, когда доступ к ним происходит реже, и его размер может быть высоким. Однако если в других типах таблиц имеется значительный сдвиг, может возникнуть необходимость изучить модель данных, чтобы избежать проблем с горячей секцией для рабочей нагрузки во время операций стабильного состояния.

## <a name="next-steps"></a>Следующие шаги

* [Обеспечение необходимой пропускной способности для контейнеров и баз данных](set-throughput.md)
* [Рекомендации по использованию ключа секции](partitioning-overview.md#choose-partitionkey)
* [Оценка единиц запросов в секунду с помощью планировщика ресурсов Azure Cosmos DB](estimate-ru-with-capacity-planner.md)
* [Эластичное масштабирование в Azure Cosmos DB API Cassandra](manage-scale-cassandra.md)
